[TOC]

#数组API
***

##数组API分类

####不会修改原数组
join
concat
slice

####修改原数组
splice
reverse
sort
push
pop
unshift
shift

***

##join

数组的字符串拼接
arr.join("连接符")

使用场景
1. 将字符数组，无缝拼接成单词
2. 将单词数组，拼接成句子
3. 将数组转化为页面上的列表/选择元素

``` javascript
    var chars=["张","三","丰"];
    console.log(chars.join("+"));
```

>张+三+丰

***

##concat
连接数组
不修改原数组对象，返回的是新数组对象
var newArr=arr.concat(值1, 值2, [值3,值4], ...);

``` javascript
    var arr=[2,3];
    var arr1=[22,33];
    var student={"sname":"张三","age":18};
    var newArr=arr.concat(44,55,arr1,student);
    console.log(newArr);
```
>(7) [2, 3, 44, 55, 22, 33, {sname:"张三",age:18}]

***

##slice
获取子数组
var subArr=arr.slice(starti,endi);
+ starti - 开始的下标位置
+ endi   - 结束的小标位置，如果省略这个参数，表示从起始下标开始截取到数组最后
+ ***含头不含尾***
+ 支持倒数下标

``` javascript
    var arr=[1, 2, 3, 4, 5];
    //       0  1  2  3  4
    //      -5 -4 -3 -2 -1
    console.log(arr.slice(1));
    console.log(arr.slice(1, -1));
    console.log(arr.slice(-5, -1));
```

>(4) [2, 3, 4, 5]
(3) [2, 3, 4]
(4) [1, 2, 3, 4]

***

##splice
删除、插入、替换任意位置的任意个数的元素
直接修改原数组对象

###删除
1. 2个参数
2. 要删除的第一项的位置坐标 index
3. 要删除的元素个数 n
splice(starti, n)

###插入
1. 3个参数
2. 起始位置坐标 index
3. 要删除的个数 - 0
4. 要插入的元素

###替换
1. 3个参数
2. 起始位置坐标 index
3. 要替换的个数 - n
4. 要插入的元素

``` javascript
    var arr=[1, 2, 3, 4, 5];
    //    删除操作
    arr.splice(1, 2);
    console.log(arr);
    //    插入操作
    var arr=[1, 2, 3, 4, 5];
    arr.splice(1, 0, 7, 8, 9);
    console.log(arr);
    //    替换操作
    var arr=[1, 2, 3, 4, 5];
    arr.splice(1, 4 , 6, 7, 8, 9);
    console.log(arr);
```

>(3) [1, 4, 5]
(8) [1, 7, 8, 9, 2, 3, 4, 5]
(5) [1, 6, 7, 8, 9]

***

##reverse
颠倒数组中所有元素的位置
直接修改原数组对象

``` javascript
    var arr=[1, 2, 3, 4, 5];
    arr.reverse();
    console.log(arr);
```
>(5) [5, 4, 3, 2, 1]

***

##sort
数组的排序方法
arr.sort()  直接修改元素数组对象  

默认的sort方法，只能将所有的元素转为字符串之后再**根据unicode号进行排序**
解决：自定义一个比较规则 -- 比较器函数
1. 定义一个比较器规则 比如升序
2. 把比较器函数的引用作为参数传递给sort方法即可  

把一个函数的引用作为参数给另外一个函数调用时使用，这个函数是**回调函数**

###比较器函数
+ 专门封装任意两数的比较逻辑的函数
+ 参数2个 :  a,b  
a始终指当前值，b始终指下一个值
+ 函数体：
    如果a>b 就交换两值位置 a-b>0 必须返回>0的数  - 升序
    如果a<b 就交换两值位置 a-b<0 必须返回<0的数  - 降序
    如果a=b  a-b=0 必须返回=0的数
+ 升序：function compare(a,b){return a-b}
+ 降序：function compare(a,b){return b-a}
+ 始终a是前一个，b是下一个，所以a,b的顺序不能变

``` javascript
    var arr=[3,22,1,2,34,11];
    function compareUp(a,b){return a-b}
    function compareDown(a,b){return b-a}

    console.log(arr.sort(compareUp));
    console.log(arr.sort(compareDown));
    console.log(arr.sort(function(a,b){return a-b}));
```

>(6) [1, 2, 3, 11, 22, 34]
(6) [34, 22, 11, 3, 2, 1]
(6) [1, 2, 3, 11, 22, 34]

***

##栈和队列

都是用数组来模拟的结构

###栈
一端封闭，只能从另一端进出的数组
特点：FILO - first in last out  先进后出

从数组的末尾入栈，其他元素的下标不会受到影响
末尾入栈： arr.push(新值)
末尾出栈： var last=arr.pop()

从数组的开头入栈：每次入栈，数组下标都会改变
开头入栈：arr.unshift(新值)
开头出栈：var first=arr.shift()

``` javascript
    var bus=[1, 2, 3, 4, 5];
    bus.push(6);
    console.log(bus);

    var last = bus.pop();
    console.log(last);
    console.log(bus);

    bus.unshift(7);
    console.log(bus);

    var first = bus.shift();
    console.log(first);
    console.log(bus);
```

>(6) [1, 2, 3, 4, 5, 6]
6
(5) [1, 2, 3, 4, 5]
(6) [7, 1, 2, 3, 4, 5]
7
(5) [1, 2, 3, 4, 5]



###队列
从一端进，从另一端出的数组
特点：FIFO - 先进先出

***

