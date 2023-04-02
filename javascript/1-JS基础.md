[TOC]

#JavaScript基础
***

##什么是JavaScript？
JavaScript是网景（Netscape）公司开发的一种基于客户端浏览器、基于面向对象、事件驱动式的网页脚本语言。  
JavaScript的前身是Livescipt，在1995年时，由Netscape公司的Brendan Eich，在网景导航者浏览器上首次设计实现而成。因为Netscape与Sun合作，Netscape管理层希望它外观看起来像Java，因此取名为JavaScript。但实际上它的语法风格与Self及Scheme较为接近。  
JavaScript正式名称是**ECMAScript**。这个标准由ECMA组织发展和维护。ECMA-262是正式的JavaScript标准，基于JavaScript（Netscape）和JavaScript（Microsoft）。
***

##JavaScript特点
* 基于对象的解释性脚本语言（代码不进行预编译）
* 简单性
* 安全性
* 动态性
* 跨平台性，JavaScript是属于web的语言，适用于PC、平板和手机。
***

##JavaScrit作用
HTML
***描述网页内容。***
CSS
***定义网页样式。***
JavaScript
***设计网页交互式操作、表单验证、网页特效、Web游戏、服务器脚本开发等。***
例如：
+ 嵌入动态文本于HTML页面。
+ 对浏览器事件做出响应。 
+ 读写HTML元素。 
+ 在数据被提交到服务器之前验证数据。
+ 检测访客的浏览器信息。 
+ 控制cookies，包括创建和修改等。 
+ 基于Node.js技术进行服务器端编程。
***

##JavaScript组成
- **ECMAScript**，语法和基本对象
- 文件对象模型(**DOM**)，处理网页内容的方法和接口
- 浏览器对象模型(**BOM**)，与浏览器进行交互的方法和接口
***

##JavaScript如何运行？
浏览器中的JS引擎解释执行。
浏览器的两个引擎：
1. **内容排版引擎**
解析HTML和CSS。   

2. **脚本解释引擎**
解释并执行JS脚本，无需编译直接执行。
解释执行：默认执行顺序从上而下逐行执行。
直译语言的弱点是安全性较差，而且在JavaScript中，如果一条运行不了，那么下面的语言也无法运行。而其解决办法就是使用try{}catch(){}。
***

##JavaScript数据类型
###原始数据类型
* String
* Number
* Boolean
* Undefined
* null
###引用类型（对象）
* Function
* Array
* Data
* Math
* Object
***

##JavaScript数据类型转换
###隐式转换
不需要程序员干预，JS可以自动完成的类型转换。
1. 计算算数中，一切数据都默认转换成数字，再计算。
* boolean类型true --> 1, false --> 0
* 一些数据无法转成数字类型，则会转成NaN-->Not a Number
* NaN不等于，不大于，不小于任何值
* NaN == NaN --> false
* NaN参与的所有计算结果永远是NaN
2. 在+运算中，只要碰到字符串，+就变成了字符串的拼接。而且，另一个不是字符串的数据会变成字符串。

###强制转换
程序员调用了专门的API（应用程序接口--一个方法），执行转换。
1. 任意类型-->字符串
x.toString()
- **x不能是undefined和null**的时候才能使用
String(x)
- 相当于隐式转换，万能的
2. 将字符串转为数字
parseInt(str)
+ 把字符串转换为整数时使用
+ 从第一个字符开始，依次读取每个数字
+ 只要碰上第一个非数字的字符就停止
+ 自动跳过开头的空字符
parseFloat(str)
+ 把字符串转换为浮点数时使用
+ 只认识第一个小数点
***
