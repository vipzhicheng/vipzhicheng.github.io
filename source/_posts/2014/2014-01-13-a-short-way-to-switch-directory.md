---
layout: post
title: "Linux系统快速切换目录的神奇脚本"
date: 2014-01-13 10:10:14 +0800
comments: true
categories:
- Ops
tags:
- Linux
- 切换目录
- Shell

---

每天在命令行下，一大部分的工作都是一遍又一遍的输入 cd ~/some/very/deep/often-used/directory这样来切换目录，现在利用一个简单的配置可以实现如下效果:

首先进入我的日常工作目录，标记一个书签mark sanguo

```
cd /Users/kimi/work/123guo/sanguo
mark sanguo
```

以后我再进入这个目录只需要g sanguo即可

```
g sanguo
```

通过输入gs指令来浏览所有的标签

```
gs
#app -> /Applications
#sanguo -> /Users/kimi/work/123guo/sanguo
#web -> /Library/WebServer/Documents
```

<!-- more -->

实现原理如下

```
sudo vim /etc/profile
```
编辑/etc/profile文件并在尾部添加以下内容后强制保存 wq!

```
# mark
export MARKPATH=$HOME/.marks
export MARKDEFAULT=sanguo#设置你的默认书签，可以直接输入g跳转

function g {
    local m=$1
    if [ "$m" = "" ]; then m=$MARKDEFAULT; fi
    cd -P "$MARKPATH/$m" 2>/dev/null || echo "No such mark: $m"
}
function mark {
    mkdir -p "$MARKPATH"
    local m=$1
    if [ "$m" = "" ]; then m=$MARKDEFAULT; fi
    rm -f "$MARKPATH/$m"
    ln -s "$(pwd)" "$MARKPATH/$m"
}
function unmark {
    local m=$1
    if [ "$m" = "" ]; then m=$MARKDEFAULT; fi
    rm -i "$MARKPATH/$m"
}
function gs {
    ls -l "$MARKPATH" | grep ^l | cut -d ' ' -f 13-
}

_completemarks() {
    local curw=${COMP_WORDS[COMP_CWORD]}
    local wordlist=$(ls -l "$MARKPATH" | grep ^l | cut -d ' ' -f 13)
    COMPREPLY=($(compgen -W '${wordlist[@]}' -- "$curw"))
    return 0
}

complete -F _completemarks g unmark
```

转：[在命令行快速切换目录](http://www.ccvita.com/520.html)
