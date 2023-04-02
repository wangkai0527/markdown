[TOC]

# GNU






## libc

1.Glibc
glibc = GNU C Library
是GNU项（GNU Project）目，所实现的 C语言标准库（C standard library）。
目前，常见的桌面和服务器中的GNU/Linux类的系统中，都是用的这套C语言标准库。
其实现了常见的C库的函数，支持很多种系统平台，功能很全，但是也相对比较臃肿和庞大。

2.uClibc
一个小型的C语言标准库，主要用于嵌入式。
其最开始设计用于uClinux（注：uClinux不支持MMU），因此比较适用于微处理器中。
对应的，此处的u意思是μ，Micro，微小的意思。

uClibc的特点：
(1)uClibc比glibc要小很多。
(2)uClibc是独立的，为了应用于嵌入式系统中，完全重新实现出来的。和glibc在源码结构和二进制上，都不兼容。

3.EGLIBC
EGLIBC = Embedded GLIBC
EGLIBC是，（后来）glibc的原创作组织FSF所（新）推出的，glibc的一种变体，目的在于将glibc用于嵌入式系统。
EGLIBC的目标是：
(1)保持源码和二进制级别的兼容于Glibc
源代码架构和ABI层面兼容
如果真正实现了这个目标，那意味着，你之前用glibc编译的程序，可以直接用eglibc替换，而不需要重新编译。
这样就可以复用之前的很多的程序了。
(2)降低(内存)资源占用/消耗
(3)使更多的模块为可配置的（以实现按需裁剪不需要的模块）
(4)提高对于交叉编译(cross-compilation)和交叉测试(cross-testing)的支持

Eglibc的最主要特点就是可配置，这样对于嵌入式系统中，你所不需要的模块，比如NIS，locale等，就可以裁剪掉，不把其编译到库中，使得降低生成的库的大小了。
更多特点，可以去看： Eglibc的特点

【glibc, uClibc, Elibc的渊源/历史/区别/联系】
1. 写程序，需要用到很多c语言的库函数。所有的库函数加起来，就是对应的C语言（标准）函数库。
2. 目前在普通GNU/Linux系统中所用的C语言标准库，叫做glibc。其功能很全，函数很多，但是代码太多，编译出来的函数库的大小也很大，即资源占用也很多。
3. 而嵌入式系统中，也需要C语言写代码实现特定功能，也需要用到C语言函数库，但是由于嵌入式系统中，一般资源比较有限，所以不适合直接使用（太占用资源的）gLibc。
4. 所以有人就又（没有参考glibc，而是从头开始，）重新实现了一个用于嵌入式系统中的，代码量不是很大的，资源占用相对较少的，C语言函数库，叫做uClibc。并且，uClibc不支持MMU（内存管理单元）。
5. 而后来，glibc的开发者，又推出个Embedded glibc，简称eglibc，其主要目的也是将glibc用于嵌入式领域。
相应最大的改动就在于，把更多的库函数，改为可配置的，这样，如果你的嵌入式系统中不需要某些函数，就可以裁剪掉，不把该函数编译到你的eglibc库中，使得最终生成的eglibc库的大小变小，最终符合你的嵌入式系统的要求（不能超过一定的大小），这样，就实现了，把glibc引用于嵌入式系统中的目的了。

可以简单的理解为：
glibc，uClibc，eglibc都是C语言函数库：
1. uClibc是嵌入式系统中用的，glibc是桌面系统用的
2. eglibc也是嵌入式系统中用的，是glibc的嵌入式版本，和glibc在源码和二进制上兼容。



4. Musl-libc C语言标准库Musl-libc项目发布了1.0版。Musl是一个轻量级的C标准库，设计作为GNU C library (glibc)、 uClibc或Android Bionic的替代用于嵌入式操作系统和移动设备。它遵循POSIX 2008规格和 C99 标准，采用MIT许可证授权，使用Musl的Linux发行版和项目包括sabotage，bootstrap-linux，LightCube OS等等。

目前openwrt LEDE默认使用Musl-libc了。。。CC分支还是uclibc











当你在linux下写C/C++代码的时候，是不是会遇到许多编译链接的问题？ 时不时报个glibc,gcc，g++等相关的错误？ 很多时候都无从下手，而且比较混乱。 这也是编译链接过程中经常出现的问题。

这篇文章不是去介绍如何编译链接，而是理清编译链接过程中碰到的一些概念和出现的问题。尤其是，libc,glib,glibc,eglibc,libc++,libstdc++,gcc,g++。

从libc说起。
libc是Linux下原来的标准C库，也就是当初写hello world时包含的头文件#include < stdio.h> 定义的地方。

后来逐渐被glibc取代，也就是传说中的GNU C Library,在此之前除了有libc，还有klibc,uclibc。现在只要知道用的最多的是glibc就行了，主流的一些linux操作系统如 Debian, Ubuntu，Redhat等用的都是glibc（或者其变种，下面会说到).

那glibc都做了些什么呢？ glibc是Linux系统中最底层的API，几乎其它任何的运行库都要依赖glibc。 glibc最主要的功能就是对系统调用的封装，你想想看，你怎么能在C代码中直接用fopen函数就能打开文件？ 打开文件最终还是要触发系统中的sys_open系统调用，而这中间的处理过程都是glibc来完成的。这篇文章详细介绍了glibc是如何与上层应用程序和系统调用交互的。除了封装系统调用，glibc自身也提供了一些上层应用函数必要的功能,如string,malloc,stdlib,linuxthreads,locale,signal等等。

好了，那eglibc又是什么？ 这里的e是Embedded的意思，也就是前面说到的变种glibc。eglibc的主要特性是为了更好的支持嵌入式架构，可以支持不同的shell(包括嵌入式)，但它是二进制兼容glibc的，就是说如果你的代码之前依赖eglibc库，那么换成glibc后也不需要重新编译。ubuntu系统用的就是eglibc（而不是glibc）,不信，你执行 ldd –version 或者 /lib/i386-linux-gnu/libc.so.6
(64位系统运行/lib/x86_64-linux-gnu）看看，便会显示你系统中eglibc/glibc的版本信息。 这里提到了libc.so.6,这个文件就是eglibc/glibc编译后的生成库文件。

还有一个glib看起来也很相似，那它又是什么呢？glib也是个c程序库，不过比较轻量级，glib将C语言中的数据类型统一封装成自己的数据类型，提供了C语言常用的数据结构的定义以及处理函数，有趣的宏以及可移植的封装等(注：glib是可移植的，说明你可以在linux下，也可以在windows下使用它）。那它跟glibc有什么关系吗？其实并没有，除非你的程序代码会用到glib库中的数据结构或者函数，glib库在ubuntu系统中并不会默认安装(可以通过apt-get install libglib2.0-dev手动安装)，著名的GTK+和Gnome底层用的都是glib库。想更详细了解glib？ 可以参考这里

看到这里，你应该知道这些库有多重要了吧？ 你写的C代码在编译的过程中有可能出现明明是这些库里面定义的变,却量还会出现’Undefined’, ‘Unreference’等错误，这时候你可能会怀疑是不是这些库出问题了？ 是不是该动手换个gilbc/eglibc了？ 这里强调一点，在你准备更换/升级这些库之前，你应该好好思考一下，你真的要更换/升级吗？你要知道你自己在做什么！你要时刻知道glibc/eglibc的影响有多大，不管你之前部署的什么程序，linux系统的ls,cd,mv,ps等等全都得依赖它，很多人在更换/升级都有过惨痛的教训，甚至让整个系统奔溃无法启动。所以，强烈不建议更换/升级这些库！

当然如果你写的是C++代码，还有两个库也要非常重视了，libc++/libstdc++,这两个库有关系吗？有。两个都是C++标准库。libc++是针对clang编译器特别重写的C++标准库，那libstdc++自然就是gcc的事儿了。libstdc++与gcc的关系就像clang与libc++. 其中的区别这里不作详细介绍了。

再说说libstdc++，glibc的关系。 libstdc++与gcc是捆绑在一起的，也就是说安装gcc的时候会把libstdc++装上。 那为什么glibc和gcc没有捆绑在一起呢？
相比glibc，libstdc++虽然提供了c++程序的标准库，但它并不与内核打交道。对于系统级别的事件，libstdc++首先是会与glibc交互，才能和内核通信。相比glibc来说，libstdc++就显得没那么基础了。

说完了这些库，这些库最终都是拿来干嘛的？当然是要将它们与你的程序链接在一起！ 这时候就不得不说说gcc了(当然还有前文提到的clang以及llvm等编译器，本文就不细说它们的区别了)。

你写的C代码.c文件通过gcc首先转化为汇编.S文件，之后汇编器as将.S文件转化为机器代码.o文件，生成的.o文件再与其它.o文件，或者之前提到的libc.so.6库文件通过ld链接器链接在一块生成可执行文件。当然，在你编译代码使用gcc的时候，gcc命令已经帮你把这些细节全部做好了。

那g++是做什么的? 慢慢说来，不要以为gcc只能编译C代码，g++只能编译c++代码。 后缀为.c的，gcc把它当作是C程序，而g++当作是c++程序；后缀为.cpp的，两者都会认为是c++程序，注意，虽然c++是c的超集，但是两者对语法的要求是有区别的。在编译阶段，g++会调用gcc,对于c++代码，两者是等价的，但是因为gcc命令不能自动和C++程序使用的库联接，需要这样，gcc -lstdc++, 所以如果你的Makefile文件并没有手动加上libstdc++库，一般就会提示错误，要求你安装g++编译器了。

好了，就说到这，理清这些库与编译器之间的关系，相信会对你解决编译链接过程中遇到的错误起到一点帮助。

如果你的编译器不支持一些新的C/C++特性，想升级gcc/g++, 这里也给出一个基于ubuntu系统的参考方法。

添加ppa

    sudo add-apt-repository ppa:ubuntu-toolchain-r/test
    sudo apt-get update

添加ppa，是因为你所用的ubuntu版本的更新源中可能并没有你想要的gcc/g++版本。

安装新版gcc/g++

    sudo apt-get install gcc-4.8
    sudo apt-get install g++-4.8

可以到/usr/bin/gcc查看新安装的gcc,g++

配置系统gcc/g++

使用update-alternatives,统一更新gcc/g++

    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.6
    sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 80 --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
    sudo update-alternatives --config gcc

数字优先级(如60，80)高的会被系统选择为默认的编译器,也可以执行第三条命令就是来手动配置系统的gcc,此处按照提示,选择4.8版本的即可。


strings /lib/x86_64-linux-gnu/libc.so.6 | grep GLIBC


https://blog.csdn.net/wq_0708/article/details/121105055





## LFS
Linux From Scratch，简称 LFS，不同于其它的 Linux 发行版，它是一种给使用者指导建议，由使用者自行从头开始自己构建的发行版。

https://linux.cn/article-5865-1.html

https://lctt.github.io/LFS-BOOK/


