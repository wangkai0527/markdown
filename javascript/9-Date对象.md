[TOC]

#Date对象

Date对象封装的是1970年1月1日0时0分0秒到当前时间的毫秒差！是一个巨大的毫秒数！
+ 提供对时间的操作方法

***

##创建Date对象
1. var now = new Date();
2. var date = new Date("year/month/date");
3. var date = new Date("year/month/date[hours:minutes:seconds]");
4. var date = new Date(year, month, date, hours, minutes, seconds);
5. var date = new Date(milliseconds);
6. var date = new Date("year/month/date");
var date1 = new Date(date.getTime());
date.getTime() 表示取出了date 距离1970 到date目标时间的毫秒差

+ 所有的日期对象set操作都会直接修改原日期对象，一旦计算完毕，旧日期就会被覆盖。
所以，在计算之前都要先复制日期对象的一个副本进行计算操作
+ 计算机中的日期跟现实的日期有差别：
现实的月份-1=计算机中的月份
[1,2,3,4,5,6,7,8,9,10,11,12]  
&#160;0 1 2 3 4 5 6 7 8 &#160;&#160;9 10 11
[日,1,2,3,4,5,6]
&#160;&#160;0 &#160;1 2 3 4 5 6

```javascript
    // 现在的时间距离生日那天的毫秒差
    var now=new Date();
    console.log(now);
    var birth=new Date("1983/12/16");
    var mils=now-birth;
    console.log(mils);
```

```javascript
    // 把毫秒转化为天数
    var days=Math.floor(mils/1000/60/60/24);
    console.log("我已经活了"+days+"天");
    var age=Math.ceil(days/365);
    console.log("年龄："+age);
```

***

##Date API

|    |  属性  |
|  :---:  |  :---:  |
|  年  |  FullYear  |
|  月  |  Month  |
|  日  |  Date  |
|  星期  |  Day  |
|  时  |  Hours  |
|  分  |  Minutes  |
|  秒  |  Seconds  |
|  毫秒  |  Milliseconds  |

+ 每个属性都有一对儿get和set方法
get方法用来获得属性的值
set方法用来设置属性的值
+ 星期 Day 没有set方法

```javascript
    var now = new Date();
    console.log(now);
    // now.toLocaleDateString() 将时间对象转化为本地时间字符串输出
    console.log(now.toLocaleDateString());
```

>Fri Sep 06 2019 14:38:53 GMT+0800 (中国标准时间)
2019/9/6

***


