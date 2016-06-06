---
layout: post
title: "根据经纬度计算之间距离"
date: 2014-05-21 09:37:19 +0800
comments: true
categories:
- Dev
tags:
- Distance
- Geo

---

网上搜集的两种计算方法，

PHP版

<!-- more -->

```
/**
*  @desc 根据两点间的经纬度计算距离
*  @param float $lat 纬度值
*  @param float $lng 经度值
*/
 function getDistance($lat1, $lng1, $lat2, $lng2)
 {
     $earthRadius = 6367000; //approximate radius of earth in meters

     /*
       Convert these degrees to radians
       to work with the formula
     */

     $lat1 = ($lat1 * pi() ) / 180;
     $lng1 = ($lng1 * pi() ) / 180;

     $lat2 = ($lat2 * pi() ) / 180;
     $lng2 = ($lng2 * pi() ) / 180;

     /*
       Using the
       Haversine formula

       http://en.wikipedia.org/wiki/Haversine_formula

       calculate the distance
     */

     $calcLongitude = $lng2 - $lng1;
     $calcLatitude = $lat2 - $lat1;
     $stepOne = pow(sin($calcLatitude / 2), 2) + cos($lat1) * cos($lat2) * pow(sin($calcLongitude / 2), 2);  $stepTwo = 2 * asin(min(1, sqrt($stepOne)));
     $calculatedDistance = $earthRadius * $stepTwo;

     return round($calculatedDistance);
 }
 ```

 Javascript版：

 ```
 function GetDistance(lat1, lng1, lat2, lng2)
{  
    if( ( Math.abs( lat1 ) > 90 ) ||( Math.abs( lat2 ) > 90 ) ){  
        return false;  
    }  
    if( ( Math.abs( lng1 ) > 180 ) ||( Math.abs( lng2 ) > 180 ) ){  
        return false;  
    }  
    var radLat1 = rad(lat1);  
    var radLat2 = rad(lat2);  
    var a = radLat1 - radLat2;  
    var b = rad(lng1) - rad(lng2);  
    var s = 2 * Math.asin(Math.sqrt(Math.pow(Math.sin(a/2),2) +  
    Math.cos(radLat1)*Math.cos(radLat2)*Math.pow(Math.sin(b/2),2)));  
    s = s *6378.137 ;// EARTH_RADIUS;  
    s = Math.round(s * 10000) / 10000;  
    return s;  
}
function rad(d)
{  
    return d * Math.PI / 180.0;  
}
```
