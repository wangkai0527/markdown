# GLSL语言

**(一)名词解释：**

> 1、图元：图形软件用来描述各种图形的函数。
> 2、OpenGL渲染管线
> 渲染流水线：是显卡芯片内部处理图形信号相互独立的并行处理单元。简单理解就是把数据转化到OpenGL并且生成最终图像的一个过程！
> 3、GLSL是什么？
> GLSL是一门专门为图形开发设计的编程语言。
> 4、可编程管线的编程阶段
> ![这里写图片描述](https://img-blog.csdn.net/20180418222130814?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 顶点处理阶段：由顶点着色器、细分着色器和几何着色器组成。主要功能是决定了图元的位置。
> 顶点着色器：当缓存数据初始化完成之后，顶点着色开始接受顶点数据，并单独处理每个顶点，顶点处理完之后，开始激活细分着色器。
> 细分着色器：分为细分控制着色器和细分计算着色器。用来描述物体的形状，在管线中生成新的几何体，并把这个几何体处理的更加平顺，然后就会成为最终状态。
> 几何着色器：细分着色器将最终的模型传给了几何着色器，其实在管线内部会对这些图元都进行修改，因为有的合适有的不合适，所以到了几何着色器就开始对所有的图像进行修改，改变几何图元的类型或者放弃掉没有用的图元。
> 片元处理阶段：由片元裁剪、光栅化和片元着色器组成。主要决定了片元的存在以及片元的颜色。
> 图元裁剪：剪切掉光栅化之前我们看不到（视口以外）的图元。
> 光栅化：将输入图元的数学描述转换为与屏幕位置对应的像素片元的过程。
> 片元着色器：主要处理光栅化之后的单独片元，主要计算了片元的颜色和深度值，然后开始传递到片元测试和混合模块。这里像光照，雾化处理都在这一步。

**（二）GLSL变量**

> (1)GLSL支持的基本数据类型：
> float、int、bool、uint/double。
> (2)标量：只有大小没有方向
> (3)变量命名规范:
> \1. 数字和“_”不能作为开头
> \2. 变量名不能包含连续的下划线
> \3. 用户变量不能用“gl”开头
> \4. 见名识意
> (4)变量的初始化:
> float f=1.0；
> bool b=true；
> int i=15；
> (5)类型转换(win):
> float f=1.0；
> bool b=bool（f）
> (6)聚合类型——向量
> 向量 :有方向的量
> 向量在着色器中的作用：位置，颜色，纹理
> 基本类型
> ![这里写图片描述](https://img-blog.csdn.net/20180523004417962?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 向量初始化：
> 1、声明变量的初始化
> vec3 xyz=vec3（1.0，1.0，1.0）;
> float x= xyz.x;
> vec3 rgb=vec3（1.0，1.0，1.0）;
> float r=rgb.r;
> float g=rgb.g;
> 当三个值相等的时候
> vec3 rgb=vec3（1.0 ）;等价于vec3 rgb=vec3（1.0，1.0，1.0）;
> 2、类型之间等价交换
> vec3 xyz1=vec3（1.0，1.0，1.0）;
> vec3 xyz2=vec3（xyz1）;
> 3、截短
> vec3 xyz1=vec3（1.0，1.0，1.0）;
> vec2 xy=vec2（xyz1）;
> 4、加长
> vec2 xy=vec2（xyz1）;
> vec3 xyz1=vec3（xy ，1.0）;
> （7）矩阵
> 矩阵在3D开发中的作用：位移，旋转，缩放
> .矩阵类型
> ![这里写图片描述](https://img-blog.csdn.net/20180523010001439?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 矩阵初始化
> 只传一个值：m=mat3（4.0）;
> ![这里写图片描述](https://img-blog.csdn.net/20180523010915308?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 以下三种写法得到效果是一样的：
> mat3 m=（ 1.0,2.0,3.0,4.0,5.0,6.0,7.0,8.0,9.0）;
>
> vec3 f1=vec3(1.0,2.0,3.0);
> vec3 f2=vec3(4.0,5.0,6.0);
> vec3 f3=vec3(7.0,8.0,9.0);
> mat3 m=mat3( f1,f2,f3 );
>
> vec2 f1=vec2(1.0,2.0);
> vec2 f2=vec2(4.0,5.0);
> vec2 f3=vec2(7.0,8.0);
> mat3 m=mat3(f1,3.0,f2,6.0,f3 ,9.0
> GLSL首先填充列 然后填充行：
> ![这里写图片描述](https://img-blog.csdn.net/20180523011152267?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
>
> （8）向量和矩阵内的元素访问
> 1、什么是分量？
> 聚合类型的元素
> 例如：vec3 xyz = vec3(1.0,1.0,1.0);
> xyz.x：分量，也是聚合类型的元素
> 2、分量名称三种形式的集合
> ![这里写图片描述](https://img-blog.csdn.net/20180529000607728?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 3、向量与矩阵中的元素访问方式？
> 向量：使用分量的名称
> 矩阵：可以以二维数组的方式进行访问
> 4、向量内元素访问（详解）
> 例如：vect3 rgb = vec3(1.0,1.0,1.0);
> float r = rgb.r;
> float g = rgb.g;
> 数组方式获取：
> float r1 = rgb[0];
> 5、矩阵内元素的访问（详解）
> 例如：mat4 m = mat4(2.0);
> ![这里写图片描述](https://img-blog.csdn.net/20180529001225466?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 获取矩阵的第二列：vect4 xyzw = m[2];
> 获取具体的标量：float f = m[2][2];//也可以这样写：m[2].z;
> 6、齐次坐标：就是将n维的向量用一个n+1维来表示。
> (4,2,1)和(8,4,2)都表示的二维点(4,2)。
> （9）结构体和采样器
> 什么是采样器？
> 专门来进行纹理采样的。
> 一般情况下一个采样器代表了一套或者一副纹理。
> ![这里写图片描述](https://img-blog.csdn.net/20180529003118374?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 注意：1、采样器的变量不能在着色器中初始化。
> 一般情况下采样器的变量都是用uniform这个限定符来修饰的，从宿主语言（java）接收传递进来的着色器的值。
> 3、sampler3D并不是在所有的OpenGLES实现中都是支持的，如果非要用，在着色器代码中打开相应的拓展。
> 1、结构体：
> 就是从逻辑上将不同类型的数据组合到一个数据集合中。
> 结构体的作用
> 简化运算。
> 定义一个结构体
> `struct info{vec3 color;//颜色vec3 xyz; //位置vec3 st; //纹理}`
> 4、怎么调用结构体元素作为输入参数
> info cxs = info(vec3(1.0,2.0,3.0),vec3(4.0,5.0,6.0),vec3(7.0,8.0,9.0));
> vec3 color = info.color;
> (10)数组
> 1、数组：有闲个类型相同的变量集合；
> 2、GLSL数组特性以及注意事项
> GLSL支持任意类型的数组，包括结构体数组。
> 索引从零开始
> 没有负数索引
> 数组可以定义为有大小或者没有大小
> 数组数GLSL第一等类型
> GLSL里面数组有构造函数，并且用函数的参数作为返回类型。
> 可以静态初始化一个数组
> float c[3] = float[3](https://blog.csdn.net/xuyankuanrong/article/details/1.0,1.0,1.0);
> GLSL有一个隐式的方法，可以返回元素个数(length)。
> int t = c.length;
> 多维数组：float c[3][5];
> (11)类型修饰符
> 1、类型修饰符作用
> ![这里写图片描述](https://img-blog.csdn.net/20180529005910284?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> ![这里写图片描述](https://img-blog.csdn.net/20180529005923874?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（12）语句
1.GLSL操作符与优先级
![这里写图片描述](https://img-blog.csdn.net/20180719235533292?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（12）操作符重载
GLSL里大部分操作符都是经过重载的，也就是说他们可以用作多种数据类型操作
1.向量和矩阵相乘
例如:vec3 xyz;
mat3 m;
vec3 xyz1 = xyz*m;
注意：要求向量的维度和矩阵的维度必须匹配！
2向量相乘：逐个分量相乘
vec2 a,b,c;
c=a*b;//计算 c=( a.x*b.x , a.y*b.y );
3矩阵相乘： 得到是通常矩阵相乘的结果；
例如：mat2 m,u,v;
m=u*v;
//m=(u00*v00+u01*v10 u00*v01+u01*v11
u01*v00+u11*v10 u10*v01+u11*v11);
4、控制流
if(){}else{}
switch(){}
5、循环语句
for（）{}
while(){}
do{}while()
6、流控制语句
![这里写图片描述](https://img-blog.csdn.net/20180720000553431?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
函数声明
注意:
1.函数声明，变量名需要添加访问修饰符
2.GLSL支持用户自定义函数，同时它定义了一些内置函数
3.函数名称可以是任何字符、数字、下划线，但是不能使用数字，连续下划线或者gl_作为函数的开始
4.返回值可以是任何内置的GLSL类型，或者用户定义的结构体和数组类型。
5.返回值是数组时，必须现实的指定大小。函数返回类型是void则没有返回值
6.函数的参数也可以是任何类型的函数，包括数组(这里数组必须设置大小)
7.在使用一个函数前必须声明他的原型或者直接给出函数体。
8.GLSL的编译器在使用函数前必须找到函数的声明，否则会产生错误
9.函数原型只是给出了函数的形式，但是并没有给出具体的实现内容
例如:
float HornerEvalPolynomial（float coeeff[10] ,float x）；
（13)程序的基本结构
1.程序基本组成
\2. 案例分析
案例：

```
uinfrom mat4 uMVPMatrix;
 attruibte vec3 aPosition;
 varying vec2 vTextureCoord;
  void positionShift(){ 
   gl_Position = uMVPMatrix * vec4(aPosition,1.0);
   }  
      void main(){
        positionShift(); 
            vTextureCoord = aTexCoor; 
      }
1234567891011
```

（14）内建变量
![这里写图片描述](https://img-blog.csdn.net/20180720001038973?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（15）顶点着色器

![这里写图片描述](https://img-blog.csdn.net/20180720001200799?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180720001943352?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（16）片元着色器
片元着色器中的内建变量分为输入变量以及输出变量。
一、输入变量
片元着色器中的内建输入变量主要有 gl_FragCoord 和 gl_FrontFacing。这两个内建变量都是只读的，由渲染管线中片元着色器之前阶段生成。
![这里写图片描述](https://img-blog.csdn.net/20180720002130709?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
二、输出变量
片元着色器中的内建输出变量主要有gl_fragColor与gl_FragData
![这里写图片描述](https://img-blog.csdn.net/20180720002216700?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
三、内置变量表格
1.定点着色器的内置变量
![这里写图片描述](https://img-blog.csdn.net/20180720002307579?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2.片元着色器的内置变量
![这里写图片描述](https://img-blog.csdn.net/20180720002344534?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（17）内建函数

> 与其他高级语言类似，为了方便开发，OpenGL ES着色语言也提供了很多的内置函数。
> 这些函数大都已经被重载，一般具有4种变体，分别用来接收和返回float、vec2、vec3以及vec4类型的值。
> 为什么使用内置函数：
> 1.内置函数都是以最优的方式实现的，有部分函数甚至由硬件直接支持，大大提高了执行效率！
> 注意：大部分内置函数同时适用于顶点作色器与片元着色器，但也有部分内置函数只适合顶点着色器或者片元着色器的
> 内置函数按照设计目的分为3个类型：
> ![这里写图片描述](https://img-blog.csdn.net/20180720002448643?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3h5MTIxMzIzNjExMw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarThumbUp.png)点赞2

- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarComment.png)评论2](https://blog.csdn.net/xuyankuanrong/article/details/79998061#commentBox)

- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarShare.png)分享](javascript:;)

- [![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)收藏2](javascript:;)

- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReward.png)打赏

- ![img](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarReport.png)举报

- [关注](javascript:;)

- 一键三连

  点赞Mark关注该