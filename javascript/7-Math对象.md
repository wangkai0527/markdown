[TOC]

#Math对象

封装数学计算常用的常量和计算方法的全局对象
+ 不能new，所有的API都直接使用 Math.xxx
+ 只要进行数学计算， 优先考虑 Math对象的方法

***

##Math API


|  功能  |  Math API  |    |
|  :---:  |  :---:  |  :---:  |
|  上取整  |  Math.ceil(n)  |    |
|  下取整  |  Math.floor(n)  |    |
|  四舍五入  |  Math.round(n)  |    |
|  乘方  |  Math.pow(底数，指数)  |  Math(10,2) 10的2次方  |
|  开平方  |  Math.sqrt(n)  |    |
|  获取最大值  |  Math.max(n1, n2, n3,...)  |  不支持数组作为参数  |
|  获取最小值  |  Math.min(n1, n2, n3,...)  |  不支持数组作为参数  |
|  随机数  |  var r=Math.random()  |  0<=r<1  |


```javascript
    var n=3.00000001;
    var n1=3.999999;
    var n2=3.5000001;
    console.log(Math.ceil(n));
    console.log(Math.floor(n1));
    console.log(Math.round(n2));
```
>4
3
4

###任意min和max之间的随机数
>Math.floor( parseInt(Math.random()*(max-min+1)+min) );

比如： 
|  60~100 的随机数  |  60<=r<=100  |
|  :---:  |  :---:  |
|  Math.random()  |   0<=r<1  |
|  Math.random()*41  |  0<=r<41  |
|  Math.random()*41+60  |  60<=r<101  |
|  Math.floor(Math.random()*41+60)  |  60<=r<=100  |



```javascript
    /*  随机生成验证码 */
    var chars=[];   //备选字符：数字+英文字符（大小写） 10+26+26=62
    //    从48开始到57结束，顺序生成0-9十个字符
    for(var c=48;c<58;c++){
        chars.push(String.fromCharCode(c));
    }
    //    从65号开始到90结束，顺序生成A-Z 26个字符
    for(var c=65;c<91;c++)
        chars.push(String.fromCharCode(c));
    //从97开始到122技术，顺序生成a-z 26个字符
    for(var c=97;c<123;c++)
        chars.push(String.fromCharCode(c));
    
    function getCode(){// 获得4位字符组成的随机验证码
        var code=[];
    //    反复生成随机数，直到code中的元素个数为4，不再生成
        while(code.length<4){
        //    从 0~61 之间 随机生成一个下标位置i
            var i=Math.floor(parseInt(Math.random()*62) );
        //    将chars中i位置的元素压入到code中
            code.push(chars[i]);
        }
        return code.join("");
    }
    var code=getCode();
    var input="";
    //    请用户反复输入验证码（code），保存在input中，直到输入正确为止
    while((input=prompt("请输入验证码 : "+code)).toUpperCase()
        !=code.toUpperCase()){
        alert("验证码错误，请重新输入")
    //    重新获得一个新的验证码
        code=getCode();
    }
    //    退出循环后，提示验证通过
    alert("验证通过");
```




