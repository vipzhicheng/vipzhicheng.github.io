---
layout: post
title: "MYSQL随机结果集方案比较"
date: 2014-01-06 01:21:31 +0800
comments: true
categories:
- Dev
tags:
- 数据库
- MySQL
- 随机结果集

---

在我们的业务需求当中，经常有需要取得随机结果的需求，比如随机会员，随机文章列表，随机文章跳转等等，我们大家都知道MYSQL的ORDER BY RAND()有性能问题，本文翻译自国外的一篇博文，大家来学习一下作者是如何解决这个问题的，这个解决方案具有在生产环境中实施的可行性。

<!--more-->

译文开始：

作为第一个例子，我们假设数据的ID从1开始，并且在1和最大值之间是连续的。

## 把事情交给应用层（PHP, JSP, Python, Ruby ...)

第一个思路：我们可以简化整个工作，如果我们可以预先在应用层计算出随机ID
```
SELECT MAX(id) FROM random;
## 在应用层生成随机ID <random-id>
SELECT name FROM random WHERE id = <random-id>
```
因为MAX(id) == COUNT(id), 我们仅仅是在1和最大值之间生成了随机数，然后传给数据库取出随机记录。

第一个SELECT是已经被优化好的，不需要任何计算。第二个是eq_ref(参见MYSQL EXPLAN语句）是一个常量，所以也非常快。

## 把事情交给数据库

在应用层做这件事真的是必要的么？我们不能在数据库中做么？

### 生成一个随机的ID
```
> SELECT RAND() * MAX(id) FROM random;
+------------------+
| RAND() * MAX(id) |
+------------------+
|  689.37582507297 |
+------------------+
```
### 哇，是一个浮点型数字，我们需要的是一个整型的随机数
```
> SELECT CEIL(RAND() * MAX(id)) FROM random;
+-------------------------+
| CEIL(RAND() * MAX(id)) |
+-------------------------+
|                    1000000  |
+-------------------------+
```
### 看起来好了一些，但性能呢？
```
> EXPLAIN
   SELECT CEIL(RAND() * MAX(id)) FROM random;
+----+-------------+-------+-------+------+-------------+
| id | select_type | table | type  | rows | Extra       |
+----+-------------+-------+-------+------+-------------+
|  1 | SIMPLE      | random  | index | 1000000  | Using index |
+----+-------------+-------+-------+------+-------------+
```
### 需要扫描索引 ? 我们还没有对 MAX()进行优化。
```
> EXPLAIN
   SELECT CEIL(RAND() * (SELECT MAX(id) FROM random));
+----+-------------+-------+------+------+------------------------------+
| id | select_type | table | type | rows | Extra                        |
+----+-------------+-------+------+------+------------------------------+
|  1 | PRIMARY     | NULL  | NULL | NULL | No tables used               |
|  2 | SUBQUERY    | NULL  | NULL | NULL | Select tables optimized away |
+----+-------------+-------+------+------+------------------------------+
```
## 一个简单的子查询让我们重新获得了我们想要的性能。
好吧，现在我知道了怎么获得一个随机数，那么怎么获得一条随机记录呢？
```
> EXPLAIN
SELECT name
  FROM random
 WHERE id = (SELECT CEIL(RAND() *
                         (SELECT MAX(id)
                            FROM random));
+----+-------------+--------+------+---------------+------+---------+------+---------+------------------------------+
| id | select_type | table  | type | possible_keys | key  | key_len | ref  | rows    | Extra                        |
+----+-------------+--------+------+---------------+------+---------+------+---------+------------------------------+
|  1 | PRIMARY     | random | ALL  | NULL          | NULL | NULL    | NULL | 1000000 | Using where                  |
|  3 | SUBQUERY    | NULL   | NULL | NULL          | NULL | NULL    | NULL |    NULL | Select tables optimized away |
+----+-------------+--------+------+---------------+------+---------+------+---------+------------------------------+
> show warnings;
+-------+------+------------------------------------------+
| Level | Code | Message                                  |
+-------+------+------------------------------------------+
| Note  | 1249 | Select 2 was reduced during optimization |
+-------+------+------------------------------------------+
```
**不，不**，一定不要这样做。这虽然是最显而易见的做法，但同时也是最错误的做法，原因是： 子查询里的select将在外层查询时对每一行分别执行，行数越多，性能越差。
我们需要找到一种方法确保随机ID只生成一次:
```
SELECT name
  FROM random JOIN
       (SELECT CEIL(RAND() *
                    (SELECT MAX(id)
                       FROM random)) AS id
        ) AS r2
       USING (id);
+----+-------------+------------+--------+------+------------------------------+
| id | select_type | table      | type   | rows | Extra                        |
+----+-------------+------------+--------+------+------------------------------+
|  1 | PRIMARY     | <derived2> | system |    1 |                              |
|  1 | PRIMARY     | random     | const  |    1 |                              |
|  2 | DERIVED     | NULL       | NULL   | NULL | No tables used               |
|  3 | SUBQUERY    | NULL       | NULL   | NULL | Select tables optimized away |
+----+-------------+------------+--------+------+------------------------------+
```
内层SELECT生成了一个常量临时表，JOIN也仅仅是JOIN一行，完美,没有Sorting, 没有用应用层，大部分的查询都是优化了的。

## 如果ID不是连续的呢...?

根据上面得到的结论，我们可以这样做
```
SELECT name
  FROM random AS r1 JOIN
       (SELECT (RAND() *
                     (SELECT MAX(id)
                        FROM random)) AS id)
        AS r2
 WHERE r1.id >= r2.id
 ORDER BY r1.id ASC
 LIMIT 1;
+----+-------------+------------+--------+------+------------------------------+
| id | select_type | table      | type   | rows | Extra                        |
+----+-------------+------------+--------+------+------------------------------+
|  1 | PRIMARY     | <derived2> | system |    1 |                              |
|  1 | PRIMARY     | r1         | range  |  689 | Using where                  |
|  2 | DERIVED     | NULL       | NULL   | NULL | No tables used               |
|  3 | SUBQUERY    | NULL       | NULL   | NULL | Select tables optimized away |
+----+-------------+------------+--------+------+------------------------------+
```
JOIN语句找出所有比随机ID大的数据，并且在没有直接匹配的情况下会选择最接近的一个，一旦找到我们就停止（LIMIT 1), 我们读取数据时对索引字段进行了排序，由于我们用的是>=,所以我们也就不必再使用CEIL函数来得到整型随机ID了。我们做了更少的事情，但效果是一样的。

## 数据分布导致的问题

一旦ID的分布不平均，我们得到的就不是真正的随机数据，通过统计我们可以看出：
```
> select * from holes;
+----+----------------------------------+----------+
| id | name                             | accesses |
+----+----------------------------------+----------+
|  1 | d12b2551c6cb7d7a64e40221569a8571 |      107 |
|  2 | f82ad6f29c9a680d7873d1bef822e3e9 |       50 |
|  4 | 9da1ed7dbbdcc6ec90d6cb139521f14a |      132 |
|  8 | 677a196206d93cdf18c3744905b94f73 |      230 |
| 16 | b7556d8ed40587a33dc5c449ae0345aa |      481 |
+----+----------------------------------+----------+
```
如果随机ID是9到15之间的数字，则我们总会取出id=16的那一条数据。

针对这一问题，有一种不是真正的解决方案，但当你的数据大部分是不经常改变时，你可以添加一个map表，他分配一个连续ID给真正的ID。
```
> create table holes_map ( row_id int not NULL primary key, random_id int not null);
> SET @id = 0;
> INSERT INTO holes_map SELECT @id := @id + 1, id FROM holes;
> select * from holes_map;
+--------+-----------+
| row_id | random_id |
+--------+-----------+
|      1 |         1 |
|      2 |         2 |
|      3 |         4 |
|      4 |         8 |
|      5 |        16 |
+--------+-----------+
```
现在row_id就是我们的连续ID了，我们可以这样写我们的查询：
```
SELECT name FROM holes
  JOIN (SELECT r1.random_id
         FROM holes_map AS r1
         JOIN (SELECT (RAND() *
                      (SELECT MAX(row_id)
                         FROM holes_map)) AS row_id)
               AS r2
        WHERE r1.row_id >= r2.row_id
        ORDER BY r1.row_id ASC
        LIMIT 1) as rows ON (id = random_id);
```
通过1000次查询，我们得到如下统计，可以看出这次我们的随机访问是均匀的了。
```
> select * from holes;
+----+----------------------------------+----------+
| id | name                             | accesses |
+----+----------------------------------+----------+
|  1 | d12b2551c6cb7d7a64e40221569a8571 |      222 |
|  2 | f82ad6f29c9a680d7873d1bef822e3e9 |      187 |
|  4 | 9da1ed7dbbdcc6ec90d6cb139521f14a |      195 |
|  8 | 677a196206d93cdf18c3744905b94f73 |      207 |
| 16 | b7556d8ed40587a33dc5c449ae0345aa |      189 |
+----+----------------------------------+----------+
```

## 通过触发器维护连续ID映射表

首先我们先初始化表:
```
DROP TABLE IF EXISTS r2;CREATE TABLE r2 (
  id SERIAL,
  name VARCHAR(32) NOT NULL UNIQUE
);

DROP TABLE IF EXISTS r2_equi_dist;
CREATE TABLE r2_equi_dist (
  id SERIAL,
  r2_id bigint unsigned NOT NULL UNIQUE
);
```
当我们在r2中改变了一些记录，我们希望r2_equi_dist也能随之更新
```
DELIMITER $$
DROP TRIGGER IF EXISTS tai_r2$$
CREATE TRIGGER tai_r2
 AFTER INSERT ON r2 FOR EACH ROW
BEGIN
  DECLARE m BIGINT UNSIGNED DEFAULT 1;

  SELECT MAX(id) + 1 FROM r2_equi_dist INTO m;
  SELECT IFNULL(m, 1) INTO m;
  INSERT INTO r2_equi_dist (id, r2_id) VALUES (m, NEW.id);
END$$
DELIMITER ;

DELETE FROM r2;

INSERT INTO r2 VALUES ( NULL, MD5(RAND()) );
INSERT INTO r2 VALUES ( NULL, MD5(RAND()) );
INSERT INTO r2 VALUES ( NULL, MD5(RAND()) );
INSERT INTO r2 VALUES ( NULL, MD5(RAND()) );

SELECT * FROM r2;
+----+----------------------------------+
| id | name                             |
+----+----------------------------------+
|  1 | 8b4cf277a3343cdefbe19aa4dabc40e1 |
|  2 | a09a3959d68187ce48f4fe7e388926a9 |
|  3 | 4e1897cd6d326f8079108292376fa7d5 |
|  4 | 29a5e3ed838db497aa330878920ec01b |
+----+----------------------------------+

SELECT * FROM r2_equi_dist;
+----+-------+
| id | r2_id |
+----+-------+
|  1 |     1 |
|  2 |     2 |
|  3 |     3 |
|  4 |     4 |
+----+-------+
```
INSERT是非常简单的，当DELETE时我们必须更新映射表来维护重建针对新不连续集合的映射
```
DELIMITER $$DROP TRIGGER IF EXISTS tad_r2$$
CREATE TRIGGER tad_r2
 AFTER DELETE ON r2 FOR EACH ROW
BEGIN
  DELETE FROM r2_equi_dist WHERE r2_id = OLD.id;
  UPDATE r2_equi_dist SET id = id - 1 WHERE r2_id > OLD.id;
END$$
DELIMITER ;

DELETE FROM r2 WHERE id = 2;

SELECT * FROM r2;
+----+----------------------------------+
| id | name                             |
+----+----------------------------------+
|  1 | 8b4cf277a3343cdefbe19aa4dabc40e1 |
|  3 | 4e1897cd6d326f8079108292376fa7d5 |
|  4 | 29a5e3ed838db497aa330878920ec01b |
+----+----------------------------------+

SELECT * FROM r2_equi_dist;
+----+-------+
| id | r2_id |
+----+-------+
|  1 |     1 |
|  2 |     3 |
|  3 |     4 |
+----+-------+
```
UPDATE 也是很直接的，我们只需要维护外键的约束。
```
DELIMITER $$DROP TRIGGER IF EXISTS tau_r2$$
CREATE TRIGGER tau_r2
 AFTER UPDATE ON r2 FOR EACH ROW
BEGIN
  UPDATE r2_equi_dist SET r2_id = NEW.id WHERE r2_id = OLD.id;
END$$
DELIMITER ;

UPDATE r2 SET id = 25 WHERE id = 4;

SELECT * FROM r2;
+----+----------------------------------+
| id | name                             |
+----+----------------------------------+
|  1 | 8b4cf277a3343cdefbe19aa4dabc40e1 |
|  3 | 4e1897cd6d326f8079108292376fa7d5 |
| 25 | 29a5e3ed838db497aa330878920ec01b |
+----+----------------------------------+

SELECT * FROM r2_equi_dist;
+----+-------+
| id | r2_id |
+----+-------+
|  1 |     1 |
|  2 |     3 |
|  3 |    25 |
+----+-------+
```

## 一次取出多行随机结果

如果你想取出多行结果，你可以这样做

* 执行这个查询多次
* 写存储过程，把结果存入临时表
* 使用UNION语句

## 存储过程的方法

存储过程提供给你一些你从其他喜欢的编程语言中了解的结构

* 循环
* 控制
* 顺序执行

针对这个目标，我们只需要一个循环
```
DELIMITER $$
DROP PROCEDURE IF EXISTS get_rands$$
CREATE PROCEDURE get_rands(IN cnt INT)
BEGIN
  DROP TEMPORARY TABLE IF EXISTS rands;
  CREATE TEMPORARY TABLE rands ( rand_id INT );

loop_me: LOOP
    IF cnt < 1 THEN
      LEAVE loop_me;
    END IF;

    INSERT INTO rands
       SELECT r1.id
         FROM random AS r1 JOIN
              (SELECT (RAND() *
                            (SELECT MAX(id)
                               FROM random)) AS id)
               AS r2
        WHERE r1.id >= r2.id
        ORDER BY r1.id ASC
        LIMIT 1;

    SET cnt = cnt - 1;
  END LOOP loop_me;
END$$
DELIMITER ;

CALL get_rands(4);
SELECT * FROM rands;
+---------+
| rand_id |
+---------+
|  133716 |
|  702643 |
|  112066 |
|  452400 |
+---------+
```
这里作者留给读者去修复一些仍然存在的问题

* 使用动态SQL来确定要操作的临时表，以便于让同一个存储过程适用于多种随机需求
* 使用UNIQUE 索引来避免随机数据的重复

## 性能

现在，让我们来看一看性能怎么样，我们有3个不同的查询来解决我们的问题。

* Q1. ORDER BY RAND()
* Q2. RAND() * MAX(ID)
* Q3. RAND() * MAX(ID) + ORDER BY ID

Q1 预期的代价是 N * log2(N), Q2 and Q3 都接近常数。

我们用真实数据来测试，从100行到100万行，并且执行1000次做如下统计
```
   100        1.000      10.000     100.000    1.000.000
Q1  0:00.718s  0:02.092s  0:18.684s  2:59.081s  58:20.000s
Q2  0:00.519s  0:00.607s  0:00.614s  0:00.628s   0:00.637s
Q3  0:00.570s  0:00.607s  0:00.614s  0:00.628s   0:00.637s
```
正如你所看到的，Q1在仅仅100行数据时就已经落后于我们优化后的SQL了。

笔者会尽最大的努力翻译出原文中所有要表达的意思，但错误之处在所难免，请大家不吝指出。

对原文感兴趣的读者，请参考作者博文原文
http://jan.kneschke.de/projects/mysql/order-by-rand/
