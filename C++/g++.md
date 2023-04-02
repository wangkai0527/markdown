[TOC]





# g++

## 后缀名
这是现在不同标准下给出的扩展名：
Unix： C, cc, cxx

GNU C++： C, cc, cxx, cpp, c++

Digital Mars： cpp, cxx

Borland： C++ cpp

Watcom： cpp

Microsoft Visual C++： cpp, cxx, cc

Metrowerks CodeWarrior： cpp, cp, cc, cxx, c++


从上可看出对于Unix系统通常C++源文件后缀为`.cc `，非Unix系统则为`.cpp`。





### 编译器根据后缀名选择编译方式

test.c:
```C
#include <stdio.h>

void func()
{
    printf("hello world \n");
}

int main()
{
    func();

    return 0;
}
```

    $gcc -S test.c
    $more test.s
            .file   "test.c"
            .text
            .section        .rodata
    .LC0:
            .string "hello world "
            .text
            .globl  func
                .type   func, @function
    .LFE0:
            .size   func, .-func
            .globl  main
            .type   main, @function

    $rm test.s && mv test.c test.cpp
    $gcc -S test.cpp
    $more test.s
        .file   "test.cpp"
        .text
        .section        .rodata
    .LC0:
            .string "hello world "
            .text
            .globl  _Z4funcv
            .type   _Z4funcv, @function
    .LFE0:
            .size   _Z4funcv, .-_Z4funcv
            .globl  main
            .type   main, @function

对于.c文件，gcc编译后的func的.type为func；而对于.cpp文件gcc编译后的func的.type为_Z4funcv， 则编译器会根据文件后缀名对函数或变量名对某些修正，一个是C的编译方式，一个是C++的编译方式。

    $rm test.s && mv test.cpp test.c
    $g++ -S test.c
    $more test.s
        .file   "test.c"
        .text
        .section        .rodata
    .LC0:
            .string "hello world "
            .text
            .globl  _Z4funcv
            .type   _Z4funcv, @function
    .LFE0:
            .size   _Z4funcv, .-_Z4funcv
            .globl  main
            .type   main, @function

可以看到g++无论是对.c文件还是.cpp文件都是按C++的方式编译的，这是和gcc是有区别的。gcc会根据文件后缀名来确定编译方式，而g++只有C++的编译方式。





## extern 关键字


## extern "C"

https://www.cnblogs.com/xiangtingshen/p/10980055.html

https://zhuanlan.zhihu.com/p/123269132

```C
#ifdef __cplusplus
extern "C"
{
#endif


#ifdef __cplusplus
}
#endif
```


    vim /usr/include/x86_64-linux-gnu/sys/cdefs.h +114

```C
#ifdef  __cplusplus
# define __BEGIN_DECLS  extern "C" {
# define __END_DECLS    }
#else
# define __BEGIN_DECLS
# define __END_DECLS
#endif
```









## build error

-----------------------------------------------------------------------
xx.c换成xx.cpp，可能会报错：

    error: no matching function for call to 'xxx'


删除编译生成的中间文件

-----------------------------------------------------------------------




