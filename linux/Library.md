[TOC]


# Library

使用GNU的汇编工具集Binutils


## 静态库

编译静态库时只有编译过程，没有链接过程，静态库引用其它库并不会在编译的时候把引用的库函数编译到生成的 lib 中，只是简单的将编译后的中间文件打包。


### ar

ar命令可以用来创建、修改库，也可以从库中提出单个模块。

https://blog.csdn.net/crazyhacking/article/details/7281023

创建库文件
ar crv liberrno.a
ar crv liberrno.a errno.o

加入新成员errno.o
默认的加入方式为append，即加在库的末尾
ar rv liberrno.a errno.o
将新成员errno.o加在指定成员atest.o之后
ar rav atest.o liberrno.a errno.o
将新成员errno.o加在指定成员atest.o之前
ar rbv atest.o liberrno.a errno.o
ar riv atest.o liberrno.a errno.o

列出库中已有成员
ar tv libc.a

删除库中成员
ar dv libc.a errno.c

从库中解出成员
ar xv libc.a errno.o

调整库中成员的顺序
将errno.o移动到atest.o之后
ar mav atest.o liberrno.a errno.o
将errno.o移动到atest.o之前
ar mbv atest.o liberrno.a errno.o
ar miv atest.o liberrno.a errno.o

更新库的符号索引表，对静态库的成员“__.SYMDEF”进行更新
ar s liberrno.a
ranlib liberrno.a


## 动态库

和Windows下的PE文件类似，ELF文件是linux系统下可执行文件、动态库文件、静态库文件的标准格式。有时候我们需要查看ELF文件的头信息，或者动态库文件的导出函数等。
和PE文件类似，ELF文件也是分成一个个段的，比如代码段，数据段等等。


动态库可以控制对外的符号表
静态库只能混淆符号表，因为静态库没有连接过程，只是打包.o


动态库隐藏函数符号

    .mk:
    LOCAL_CPPFLAGS += -fvisibility=hidden
    CMAKE_CXX_FLAGS += -fvisibility=hidden

    .h:
    #define MYDCIR_API __attribute__((visibility ("default")))  //Linux动态库(.so)
    MYDCIR_API int test_fun();






## 查看库信息

### file
    $ file xxx.so
    xxx.so: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, not stripped
    xxx.so: ELF 32-bit LSB shared object, QUALCOMM DSP6, version 1 (SYSV), dynamically linked, no section header
    xxx.so: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, BuildID[sha1]=cadb1242415ccd72b9658b100baaa457ad582a9f, stripped

    $ file libc.a
    libc.a: current ar archive
    $ ar xv libc.a errno.o
    $ file errno.o
    errno.o: ELF 64-bit LSB relocatable, ARM aarch64, version 1 (SYSV), not stripped

    $ file HelloWorld
    -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
    -DANDROID_ABI=arm64-v8a \
    HelloWorld: ELF 64-bit LSB shared object, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /system/bin/linker64, BuildID[sha1]=2e611121c470a7178667beae5926cd29e67afccd, with debug_info, not stripped

    -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
    -DANDROID_ABI=x86_64 \
    HelloWorld: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /system/bin/linker64, BuildID[sha1]=e3a7a6636cdb228ccad23ecbf5259cd8dadb7502, with debug_info, not stripped

    gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc
    HelloWorld: ELF executable, 64-bit LSB arm64, dynamic (/lib/ld-linux-aarch64.so.1), BuildID=cd2691e4d92347bca6f288f37ee35116aab9af61, not stripped
    
    /usr/bin/x86_64-linux-gnu-gcc
    HelloWorld: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=00748eb6d9c8c65f002ea37f21ee3c6b9ffabf24, for GNU/Linux 3.2.0, with debug_info, not stripped


### ldd
ldd 用于打印程序或者库文件的依赖库关系
ldd不是一个可执行程序，只是一个shell脚本，如果程序执行时，依赖的某个库找不到，通过这个命令可以迅速定位问题所在。
如果某个依赖的库不存在，会打印类似“xxx.so not found”的提示
-d是ldd的缩写

ldd能够显示可执行模块的dependency，其原理是通过设置一系列的环境变量，如下：LD_TRACE_LOADED_OBJECTS、LD_WARN、LD_BIND_NOW、LD_LIBRARY_VERSION、LD_VERBOSE等。
当`LD_TRACE_LOADED_OBJECTS`环境变量不为空时，任何可执行程序在运行时，它都会只显示模块的dependency，而程序并不真正执行。要不你可以在shell终端测试一下，如下：

    export LD_TRACE_LOADED_OBJECTS=1

再执行任何的程序，如ls等，看看程序的运行结果。
ldd显示可执行模块的dependency的工作原理，其实质是通过ld-linux.so（elf动态库的装载器）来实现的。我们知道，ld-linux.so模块会先于executable模块程序工作，并获得控制权，因此当上述的那些环境变量被设置时，ld-linux.so选择了显示可执行模块的dependency。

实际上可以直接执行ld-linux.so模块，如：/lib/ld-linux.so.2 --list program（这相当于ldd program）

    $ ldd -v /usr/bin/chmod

    linux-vdso.so.1 (0x00007ffe3ef2d000)
    libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff16da82000)
    /lib64/ld-linux-x86-64.so.2 (0x00007ff16dc95000)

    Version information:
    /usr/bin/chmod:
            libc.so.6 (GLIBC_2.3) => /lib/x86_64-linux-gnu/libc.so.6
            libc.so.6 (GLIBC_2.3.4) => /lib/x86_64-linux-gnu/libc.so.6
            libc.so.6 (GLIBC_2.14) => /lib/x86_64-linux-gnu/libc.so.6
            libc.so.6 (GLIBC_2.4) => /lib/x86_64-linux-gnu/libc.so.6
            libc.so.6 (GLIBC_2.2.5) => /lib/x86_64-linux-gnu/libc.so.6
    /lib/x86_64-linux-gnu/libc.so.6:
            ld-linux-x86-64.so.2 (GLIBC_2.3) => /lib64/ld-linux-x86-64.so.2
            ld-linux-x86-64.so.2 (GLIBC_PRIVATE) => /lib64/ld-linux-x86-64.so.2

其中信息解释如下：
第一列显示需要的类库。
第二列显示实际提供的类库。
第三列显示提供类库的开始地址空间

    $ ldd --help
    Usage: ldd [OPTION]... FILE...
        --help              print this help and exit //打印帮助
        --version           print version information and exit // 查看指令版本信息
    -d, --data-relocs       process data relocations //执行重定位和报告任何丢失的对象；
    -r, --function-relocs   process data and function relocations //执行数据对象和函数的重定位，并且报告任何丢失的对象和函数；
    -u, --unused            print unused direct dependencies // 打印未直接使用的依赖
    -v, --verbose           print all information //详细信息模式，打印所有相关信息；

    For bug reporting instructions, please see:
    <https://bugs.launchpad.net/ubuntu/+source/glibc/+bugs>.


linux-vdso.so.1为一个虚拟的库Virtual Dynamic Shared Object所以没有路径,包含一些内核api(比如syscall()这个函数就是在linux-vdso.so.1里头,内核把包含某.so的内存页在程序启动的时候映射入其内存空间，对应的程序就可以当普通的.so来使用里头的函数)
/lib64/ld-linux-x86-64.so.2为链接器,是程序中第一个被加载的(然后通过ldopen来动态加载剩下的库?)
此外: -v 选项可以打印完整信息,包括链接的库的版本信息.

libstdc++.so.6和libc.so.6应该是c++和c的标准库.
libgcc_s.so.1 是gcc运行时库,提供一些gcc支持的api.



### nm
nm命令是Linux下自带的强大的文本分析工具，是命令来源于name的简写。
该命令用来列出指定文件中的符号（如常用的函数名、变量等，以及这些符号存储的区域）。
它显示指定文件中的符号信息，文件可以是对象文件、可执行文件或对象文件库。
如果文件中没有包含符号信息，nm报告该情况，单不把他解释为出错。
nm缺省情况下报告十进制符号表示法下的数字值。

| 参数 | 说明 |
| :---: | :---: |
| -A/-o/–print-file-name | 在输出时加上文件名； |
| -a/–debug-syms | 输出所有符号，包含debugger-only symbols; |
| -B/–format=bsd | BSD码显示，兼容MIPS nm； |
| -C/–demangle | 将低级符号名解析为用户级名字，可以使得C++函数名更具可读性； |
| -D/–dynamic | 显示动态符号。该选项只对动态目标（如特定类型的共享库）有意义； |
| -f format/–format=format | 使用format格式输出。format可以选取bsd、sysv或posix，该选项在GNU的nm中有用。默认为bsd |
| -g/–extern-only | 只显示外部符号； |
| -l/–line-numbers | 对于每个符号，使用debug信息找到文件名和行号； |
| -n/-v/–numeric-sort | 按符号对应地址的顺序排序，而非按符号名字字符顺序排序； |
| -P/–portability | 按照POSIX2.0标准格式输出，等同于使用 -f posix； |
| -p/–no-sort | 按照目标文件中遇到的符号顺序显示，不排序； |
| -r/–reverse-sort | 反转排序； |
| -s/–print-armap | 当列出库成员符号时，包含索引。索引的内容：模块和其包含名字的映射； |
| -u/–undefined-only | 只显示未定义符号； |
| –defined-only | 只显示定义了的符号。 |

如上面nm列出的结果中每个符号意义如下：
对于每个符号，nm 显示：

• 符号值，由选项选择的基数（见下文），或默认为十六进制。

• 符号类型。至少使用以下类型；其他的也取决于对象文件格式。如果是小写，符号通常是本地的；如果是大写，则符号是全局的（外部的）。然而，对于特殊的全局符号（“u”、“v”和“w”）显示了一些小写符号

| 符号 | 描述 |
| :---: | :---: |
| A	 | 符号的值是绝对值，不会通过进一步的链接而改变。 |
| B/b | 符号在 BSS 数据段中。本节通常包含零初始化或未初始化的数据，尽管确切的行为取决于系统。 |
| C	 | 这个符号很常见。常用符号是未初始化的数据。链接时，多个常用符号可能以相同的名称出现。如果符号在任何地方定义，则常用符号被视为未定义的引用。 |
| D/d | 符号在初始化数据段中。 |
| G/g | 符号在小对象的初始化数据段中。一些目标文件格式允许更有效地访问小型数据对象，例如全局int变量，而不是大型全局数组。 |
| i | 对于PE格式文件，这表示符号位于特定于实现的部分中 DLL。对于ELF格式文件，这表明该符号是一个间接函数。这是一个GNU对标准ELF 符号类型集的扩展。它表示一个符号，如果被一个重定位不计算其地址，而是必须在运行时调用。运行时然后执行将返回要在重定位中使用的值。 |
| I	 | 该符号是对另一个符号的间接引用。 |
| N	 | 该符号是调试符号。 |
| n	 | 符号在只读数据段中。 |
| p	 | 符号在堆栈展开部分中。 |
| R/r | 该符号位于只读数据段中。 |
| S/s | 该符号在小对象的未初始化或零初始化数据段中。 |
| T/t | 符号在文本（代码）部分。 |
| U | 符号未定义。 |
| u | 该符号是唯一的全局符号。这是标准ELF符号集的GNU扩展绑定。对于这样的符号，动态链接器将确保在整个过程中只有一个具有此名称和类型的符号正在使用中。 |
| V/v | 符号是弱对象。当弱定义符号与正常定义符号链接时，使用正常定义的符号没有错误。当一个弱的未定义符号被链接并且该符号没有定义，弱符号的值变为零，没有错误。在某些系统上，大写表示已指定默认值。 |
| W/w | 该符号是一个弱符号，没有被专门标记为弱对象符号。当一个弱定义符号与正常定义符号链接，正常定义符号与没有错误。当一个弱的未定义符号被链接并且该符号未定义时，符号以系统特定的方式确定，没有错误。在某些系统上，大写表示已指定默认值。 |
| - | 该符号是a.out目标文件中的stabs符号。在这种情况下，打印的下一个值是stabs other字段、stabs desc字段和stab类型。刺符号用于保持调试信息。 |
| ？ | 	符号类型未知，或目标文件格式为sp |


寻找特殊标识
有时会碰到一个编译了但没有链接的代码，那是因为它缺失了标识符；这种情况，可以用nm和objdump、readelf命令来查看程序的符号表；所有这些命令做的工作基本一样；
比如连接器报错有未定义的标识符；大多数情况下，会发生在库的缺失或企图链接一个错误版本的库的时候；浏览目标代码来寻找一个特殊标识符的引用:

    $ nm -uCA *.o | grep foo

-u选项限制了每个目标文件中未定义标识符的输出。-A选项用于显示每个标识符的文件名信息；对于C++代码，常用的还有-C选项，它也为解码这些标识符；

objdump、readelf命令可以完成同样的任务。等效命令为： 

    $ objdump −t 
    $ readelf -s

只显示外部的定义的符号

    $ nm add.o --defined-only -g

按照地址顺序列出符号信息

    $ nm -n HelloWorld
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
                 U _ZNSolsEPFRSoS_E@@GLIBCXX_3.4
                 U _ZNSolsEi@@GLIBCXX_3.4
                 U _ZNSt8ios_base4InitC1Ev@@GLIBCXX_3.4
                 U _ZNSt8ios_base4InitD1Ev@@GLIBCXX_3.4
                 U _ZSt4cout@@GLIBCXX_3.4
                 U _ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@@GLIBCXX_3.4
                 U _ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@@GLIBCXX_3.4
                 U __cxa_atexit@@GLIBC_2.2.5
                 w __cxa_finalize@@GLIBC_2.2.5
                 w __gmon_start__
                 U __libc_start_main@@GLIBC_2.2.5
                 U __stack_chk_fail@@GLIBC_2.4
                 U memmove@@GLIBC_2.2.5
                 U printf@@GLIBC_2.2.5
0000000000001000 t _init
0000000000001140 T _start
0000000000001170 t deregister_tm_clones
00000000000011a0 t register_tm_clones
00000000000011e0 t __do_global_dtors_aux
0000000000001220 t frame_dummy
000000000000122a t _ZZ4mainENKUliiE_clEii
0000000000001243 T main
00000000000013ac t _ZSt4sortIPiZ4mainEUliiE_EvT_S2_T0_
00000000000013d7 t _ZN9__gnu_cxx5__ops16__iter_comp_iterIZ4mainEUliiE_EENS0_15_Iter_comp_iterIT_EES4_
0000000000001424 t _ZSt6__sortIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_T0_
000000000000147f t _ZSt4moveIRZ4mainEUliiE_EONSt16remove_referenceIT_E4typeEOS3_
000000000000148e t _ZN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EC1ES2_
000000000000148e t _ZN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EC2ES2_
00000000000014a9 t _ZSt16__introsort_loopIPilN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_T0_T1_
000000000000152e t _ZSt22__final_insertion_sortIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_T0_
0000000000001596 t _ZSt14__partial_sortIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_S6_T0_
00000000000015db t _ZSt27__unguarded_partition_pivotIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEET_S6_S6_T0_
0000000000001654 t _ZSt16__insertion_sortIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_T0_
0000000000001727 t _ZSt26__unguarded_insertion_sortIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_T0_
0000000000001764 t _ZSt13__heap_selectIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_S6_T0_
00000000000017e2 t _ZSt11__sort_heapIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_RT0_
000000000000182a t _ZSt22__move_median_to_firstIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_S6_S6_T0_
0000000000001952 t _ZSt21__unguarded_partitionIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEET_S6_S6_S6_T0_
00000000000019e0 t _ZN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EclIPiS5_EEbT_T0_
0000000000001a10 t _ZN9__gnu_cxx5__ops15__val_comp_iterIZ4mainEUliiE_EENS0_14_Val_comp_iterIT_EENS0_15_Iter_comp_iterIS4_EE
0000000000001a63 t _ZSt25__unguarded_linear_insertIPiN9__gnu_cxx5__ops14_Val_comp_iterIZ4mainEUliiE_EEEvT_T0_
0000000000001b09 t _ZSt11__make_heapIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_RT0_
0000000000001bd1 t _ZSt10__pop_heapIPiN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_S6_S6_RT0_
0000000000001c67 t _ZSt4moveIRN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEONSt16remove_referenceIT_E4typeEOS7_
0000000000001c76 t _ZN9__gnu_cxx5__ops14_Val_comp_iterIZ4mainEUliiE_EC1EONS0_15_Iter_comp_iterIS2_EE
0000000000001c76 t _ZN9__gnu_cxx5__ops14_Val_comp_iterIZ4mainEUliiE_EC2EONS0_15_Iter_comp_iterIS2_EE
0000000000001c96 t _ZN9__gnu_cxx5__ops14_Val_comp_iterIZ4mainEUliiE_EclIiPiEEbRT_T0_
0000000000001cc6 t _ZSt13__adjust_heapIPiliN9__gnu_cxx5__ops15_Iter_comp_iterIZ4mainEUliiE_EEEvT_T0_S7_T1_T2_
0000000000001e7e t _ZN9__gnu_cxx5__ops14_Iter_comp_valIZ4mainEUliiE_EC1EONS0_15_Iter_comp_iterIS2_EE
0000000000001e7e t _ZN9__gnu_cxx5__ops14_Iter_comp_valIZ4mainEUliiE_EC2EONS0_15_Iter_comp_iterIS2_EE
0000000000001e9d t _ZSt11__push_heapIPiliN9__gnu_cxx5__ops14_Iter_comp_valIZ4mainEUliiE_EEEvT_T0_S7_T1_RT2_
0000000000001f98 t _ZN9__gnu_cxx5__ops14_Iter_comp_valIZ4mainEUliiE_EclIPiiEEbT_RT0_
0000000000001fc8 t _Z41__static_initialization_and_destruction_0ii
0000000000002015 t _GLOBAL__sub_I_HelloWorld.cpp
000000000000202e W _ZSt4__lgl
0000000000002053 W _ZSt4moveIRiEONSt16remove_referenceIT_E4typeEOS2_
0000000000002065 W _ZSt13move_backwardIPiS0_ET0_T_S2_S1_
00000000000020b5 W _ZSt9iter_swapIPiS0_EvT_T0_
00000000000020df W _ZSt12__miter_baseIPiET_S1_
00000000000020f1 W _ZSt23__copy_move_backward_a2ILb1EPiS0_ET1_T0_S2_S1_
000000000000215f W _ZSt4swapIiENSt9enable_ifIXsrSt6__and_IJSt6__not_ISt15__is_tuple_likeIT_EESt21is_move_constructibleIS4_ESt18is_move_assignableIS4_EEE5valueEvE4typeERS4_SE_
00000000000021d2 W _ZSt12__niter_baseIPiET_S1_
00000000000021e4 W _ZSt22__copy_move_backward_aILb1EPiS0_ET1_T0_S2_S1_
0000000000002219 W _ZSt12__niter_wrapIPiET_RKS1_S1_
000000000000222f W _ZNSt20__copy_move_backwardILb1ELb1ESt26random_access_iterator_tagE13__copy_move_bIiEEPT_PKS3_S6_S4_
00000000000022b0 T __libc_csu_init
0000000000002320 T __libc_csu_fini
0000000000002328 T _fini
0000000000003000 R _IO_stdin_used
0000000000003004 r _ZStL19piecewise_construct
0000000000003038 r __GNU_EH_FRAME_HDR
00000000000037fc r __FRAME_END__
0000000000004d50 d __frame_dummy_init_array_entry
0000000000004d50 d __init_array_start
0000000000004d60 d __do_global_dtors_aux_fini_array_entry
0000000000004d60 d __init_array_end
0000000000004d68 d _DYNAMIC
0000000000004f68 d _GLOBAL_OFFSET_TABLE_
0000000000005000 D __data_start
0000000000005000 W data_start
0000000000005008 D __dso_handle
0000000000005010 D __TMC_END__
0000000000005010 B __bss_start
0000000000005010 D _edata
0000000000005010 b completed.8061
0000000000005011 b _ZStL8__ioinit
0000000000005018 B _end


查看函数符号

    $ nm -D xxx.so

（-D或-dynamic选项表示：显示动态符号。该选项仅对于动态库有意义）

得到的结果中以T开头的就是函数符号

使用awk命令筛选出第二列为-T的行

    $ nm -D xxx.so | awk '{if($2=="T"){print $3}}'


https://blog.csdn.net/qq_28087491/article/details/121437727



### objdump
objdump：通过反汇编，显示目标文件中的详细信息

反汇编整个程序
    $ objdump -d HelloWorld

但是如果程序较大，那么反汇编时间将会变长，而且反汇编文件也会很大。如果我们已经知道了问题在某个函数，只想反汇编某一个函数，怎么处理呢？
我们可以利用前面介绍的nm命令获取到函数test的地址，然后使用下面的方式反汇编：

    $ objdump -d HelloWorld --start-address=0x1243 --stop-address=0x13ac

HelloWorld:     file format elf64-x86-64

Disassembly of section .text:

0000000000001243 <main>:
    1243:       f3 0f 1e fa             endbr64
    1247:       55                      push   %rbp
    1248:       48 89 e5                mov    %rsp,%rbp
    124b:       48 83 ec 40             sub    $0x40,%rsp
    124f:       64 48 8b 04 25 28 00    mov    %fs:0x28,%rax
    1256:       00 00
    1258:       48 89 45 f8             mov    %rax,-0x8(%rbp)
    125c:       31 c0                   xor    %eax,%eax
    125e:       48 8d 3d a0 1d 00 00    lea    0x1da0(%rip),%rdi        # 3005 <_ZStL19piecewise_construct+0x1>
    1265:       b8 00 00 00 00          mov    $0x0,%eax
    126a:       e8 51 fe ff ff          callq  10c0 <printf@plt>
    126f:       48 8d 35 9f 1d 00 00    lea    0x1d9f(%rip),%rsi        # 3015 <_ZStL19piecewise_construct+0x11>
    1276:       48 8b 05 53 3d 00 00    mov    0x3d53(%rip),%rax        # 4fd0 <_ZSt4cout@GLIBCXX_3.4>
    127d:       48 89 c7                mov    %rax,%rdi
    1280:       e8 5b fe ff ff          callq  10e0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
    1285:       48 89 c2                mov    %rax,%rdx
    1288:       48 8b 05 39 3d 00 00    mov    0x3d39(%rip),%rax        # 4fc8 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
    128f:       48 89 c6                mov    %rax,%rsi
    1292:       48 89 d7                mov    %rdx,%rdi
    1295:       e8 56 fe ff ff          callq  10f0 <_ZNSolsEPFRSoS_E@plt>
    129a:       48 8d 35 82 1d 00 00    lea    0x1d82(%rip),%rsi        # 3023 <_ZStL19piecewise_construct+0x1f>
    12a1:       48 8b 05 28 3d 00 00    mov    0x3d28(%rip),%rax        # 4fd0 <_ZSt4cout@GLIBCXX_3.4>
    12a8:       48 89 c7                mov    %rax,%rdi
    12ab:       e8 30 fe ff ff          callq  10e0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
    12b0:       48 89 c2                mov    %rax,%rdx
    12b3:       48 8b 05 0e 3d 00 00    mov    0x3d0e(%rip),%rax        # 4fc8 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
    12ba:       48 89 c6                mov    %rax,%rsi
    12bd:       48 89 d7                mov    %rdx,%rdi
    12c0:       e8 2b fe ff ff          callq  10f0 <_ZNSolsEPFRSoS_E@plt>
    12c5:       48 8d 35 61 1d 00 00    lea    0x1d61(%rip),%rsi        # 302d <_ZStL19piecewise_construct+0x29>
    12cc:       48 8b 05 fd 3c 00 00    mov    0x3cfd(%rip),%rax        # 4fd0 <_ZSt4cout@GLIBCXX_3.4>
    12d3:       48 89 c7                mov    %rax,%rdi
    12d6:       e8 05 fe ff ff          callq  10e0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
    12db:       48 89 c2                mov    %rax,%rdx
    12de:       48 8b 05 e3 3c 00 00    mov    0x3ce3(%rip),%rax        # 4fc8 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
    12e5:       48 89 c6                mov    %rax,%rsi
    12e8:       48 89 d7                mov    %rdx,%rdi
    12eb:       e8 00 fe ff ff          callq  10f0 <_ZNSolsEPFRSoS_E@plt>
    12f0:       c7 45 e0 04 00 00 00    movl   $0x4,-0x20(%rbp)
    12f7:       c7 45 e4 02 00 00 00    movl   $0x2,-0x1c(%rbp)
    12fe:       c7 45 e8 03 00 00 00    movl   $0x3,-0x18(%rbp)
    1305:       c7 45 ec 01 00 00 00    movl   $0x1,-0x14(%rbp)
    130c:       48 8d 45 e0             lea    -0x20(%rbp),%rax
    1310:       48 83 c0 10             add    $0x10,%rax
    1314:       48 8d 55 e0             lea    -0x20(%rbp),%rdx
    1318:       48 89 c6                mov    %rax,%rsi
    131b:       48 89 d7                mov    %rdx,%rdi
    131e:       e8 89 00 00 00          callq  13ac <_ZSt4sortIPiZ4mainEUliiE_EvT_S2_T0_>
    1323:       48 8d 45 e0             lea    -0x20(%rbp),%rax
    1327:       48 89 45 d0             mov    %rax,-0x30(%rbp)
    132b:       48 8b 45 d0             mov    -0x30(%rbp),%rax
    132f:       48 89 45 c8             mov    %rax,-0x38(%rbp)
    1333:       48 8b 45 d0             mov    -0x30(%rbp),%rax
    1337:       48 83 c0 10             add    $0x10,%rax
    133b:       48 89 45 d8             mov    %rax,-0x28(%rbp)
    133f:       48 8b 45 c8             mov    -0x38(%rbp),%rax
    1343:       48 3b 45 d8             cmp    -0x28(%rbp),%rax
    1347:       74 48                   je     1391 <main+0x14e>
    1349:       48 8b 45 c8             mov    -0x38(%rbp),%rax
    134d:       8b 00                   mov    (%rax),%eax
    134f:       89 45 c4                mov    %eax,-0x3c(%rbp)
    1352:       8b 45 c4                mov    -0x3c(%rbp),%eax
    1355:       89 c6                   mov    %eax,%esi
    1357:       48 8b 05 72 3c 00 00    mov    0x3c72(%rip),%rax        # 4fd0 <_ZSt4cout@GLIBCXX_3.4>
    135e:       48 89 c7                mov    %rax,%rdi
    1361:       e8 ca fd ff ff          callq  1130 <_ZNSolsEi@plt>
    1366:       48 8d 35 c9 1c 00 00    lea    0x1cc9(%rip),%rsi        # 3036 <_ZStL19piecewise_construct+0x32>
    136d:       48 89 c7                mov    %rax,%rdi
    1370:       e8 6b fd ff ff          callq  10e0 <_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc@plt>
    1375:       48 89 c2                mov    %rax,%rdx
    1378:       48 8b 05 49 3c 00 00    mov    0x3c49(%rip),%rax        # 4fc8 <_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_@GLIBCXX_3.4>
    137f:       48 89 c6                mov    %rax,%rsi
    1382:       48 89 d7                mov    %rdx,%rdi
    1385:       e8 66 fd ff ff          callq  10f0 <_ZNSolsEPFRSoS_E@plt>
    138a:       48 83 45 c8 04          addq   $0x4,-0x38(%rbp)
    138f:       eb ae                   jmp    133f <main+0xfc>
    1391:       b8 00 00 00 00          mov    $0x0,%eax
    1396:       48 8b 4d f8             mov    -0x8(%rbp),%rcx
    139a:       64 48 33 0c 25 28 00    xor    %fs:0x28,%rcx
    13a1:       00 00
    13a3:       74 05                   je     13aa <main+0x167>
    13a5:       e8 56 fd ff ff          callq  1100 <__stack_chk_fail@plt>
    13aa:       c9                      leaveq
    13ab:       c3                      retq

查看函数符号

    $ objdump -tT xxx.so

查看依赖的库

    $ objdump -x xxx.so | grep NEEDED

查看可执行程序依赖的库

    $ objdump -x 可执行程序名 | grep NEEDED
    $ objdump -p 可执行程序名 | grep NEEDED

查看库所用的编译器及版本

    $ objdump -s --section=.comment libxxx.a
    $ readelf xx -p .comment

    wk@ZJ-DI-1499F:libs_vivo/system/lib64$ readelf libc++_shared.so -p .comment
    String dump of section '.comment':
    [     0]  GCC: (GNU) 4.9 20140827 (prerelease)
    [    25]  clang version 3.6

    wk@ZJ-DI-1499F:libs_vivo/system/lib64$ readelf libandfix.so -p .comment
    String dump of section '.comment':
    [     1]  GCC: (GNU) 4.9.x 20150123 (prerelease)
    [    28]  Android clang version 3.8.256229  (based on LLVM 3.8.256229)

    wk@ZJ-DI-1499F:libs_vivo/system/lib64$ readelf libc.so -p .comment
    String dump of section '.comment':
    [     1]  Snapdragon LLVM ARM Compiler 6.0.7 for Android NDK (based on llvm.org 6.0)
    [    4c]  GCC: (GNU) 4.9.x 20150123 (prerelease)

    wk@ZJ-DI-1499F:libs_vivo/system/lib64$ readelf libart.so -p .comment
    String dump of section '.comment':
    [     1]  Snapdragon LLVM ARM Compiler 6.0.7 for Android NDK (based on llvm.org 6.0)
    [    4c]  GCC: (GNU) 4.9.x 20150123 (prerelease)
    [    73]  Android (4691093 based on r316199) clang version 6.0.2 (https://android.googlesource.com/toolchain/clang 183abd29fc496f55536e7d904e0abae47888fc7f) (https://android.googlesource.com/toolchain/llvm 34361f192e41ed6e4e8f9aca80a4ea7e9856f327) (based on LLVM 6.0.2svn)

    wk@ZJ-DI-1499F:libs_vivo/vendor/lib64$ readelf hw/sensors.sdm710.so -p .comment
    String dump of section '.comment':
    [     1]  Snapdragon LLVM ARM Compiler 6.0.7 for Android NDK (based on llvm.org 6.0)


    wk@ZJ-DI-1499F:libs_oppo/system/lib64$ readelf libc.so -p .comment
    String dump of section '.comment':
    [     1]  Snapdragon LLVM ARM Compiler 4.0.10 for Android NDK (based on llvm.org 4.0+)
    [    4e]  GCC: (GNU) 4.9.x 20150123 (prerelease)

    wk@ZJ-DI-1499F:libs_oppo/system/lib64$ readelf libart.so -p .comment
    String dump of section '.comment':
    [     1]  GCC: (GNU) 4.9.x 20150123 (prerelease)
    [    28]  Android clang version 5.0.300080  (based on LLVM 5.0.300080)
    [    65]  Snapdragon LLVM ARM Compiler 4.0.10 for Android NDK (based on llvm.org 4.0+)

    wk@ZJ-DI-1499F:libs_oppo/vendor/lib64$ readelf libc++_shared.so -p .comment
    String dump of section '.comment':
    [     0]  GCC: (GNU) 4.9.x 20150123 (prerelease)
    [    27]  Android clang version 5.0.300080  (based on LLVM 5.0.300080)

    wk@ZJ-DI-1499F:libs_oppo/vendor/lib64$ readelf libvendor.goodix.hardware.biometrics.fingerprint@2.1.so -p .comment
    String dump of section '.comment':
    [     0]  GCC: (GNU) 4.9.x 20150123 (prerelease)
    [    27]  Android clang version 5.0.300080  (based on LLVM 5.0.300080)



### readelf
功能：用于显示ELF文件（如.so、.a、.o文件等）的相关信息。
readelf命令可显示一个或多个ELF格式对象文件的信息。后面可加一些选项控制要显示的特定信息。
elffile …是要检查的目标文件。支持32位和64位ELF文件，也支持包含ELF文件的文档（如使用ar命令将一些elf文件打包生成的lib*.a之类的文件）。
该程序执行与objdump相似的功能，但更详细，并且独立于BFD库而存在，因此，即使BFD中存在错误，readelf也不会受到影响。


查看ELF文件开始的文件头信息

    $ readelf -h libcdsprpc.so
    $ readelf -h HelloWorld
    ELF Header:
        Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00                    #elf文件魔数字
        Class:                             ELF64                                    #64位 elf文件
        Data:                              2's complement, little endian            #字节序为小端序
        Version:                           1 (current)
        OS/ABI:                            UNIX - System V
        ABI Version:                       0
        Type:                              DYN (Shared object file)                 #目标文件类型
        Machine:                           Advanced Micro Devices X86-64            #目标处理器体系
        Version:                           0x1
        Entry point address:               0x1140                                   #入口地址
        Start of program headers:          64 (bytes into file)
        Start of section headers:          16712 (bytes into file)
        Flags:                             0x0
        Size of this header:               64 (bytes)
        Size of program headers:           56 (bytes)
        Number of program headers:         13
        Size of section headers:           64 (bytes)
        Number of section headers:         29
        Section header string table index: 28

入口地址是 0x1140(_start)，而不是 0x1243(main)。也就是说，我们的程序运行并非从main开始。

查看源码路径（包括库所用的编译器及版本）

    $ readelf helloworld -p .debug_str

查看包含了哪些.o文件

    $ readelf -h libxxx.a | grep "File:"

查看依赖的库

    $ readelf -d libffmpeg.so | grep NEEDED

查看是否带有调试信息.debug_info

    $ readelf -S libxxx | grep debug

    $ gdb HelloWorld
    Reading symbols from HelloWorld...done.
    Reading symbols from HelloWorld...(no debugging symbols found)...done.

    $ file HelloWorld
    
    /usr/bin/x86_64-linux-gnu-gcc
    HelloWorld: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=00748eb6d9c8c65f002ea37f21ee3c6b9ffabf24, for GNU/Linux 3.2.0, with debug_info, not stripped

如果最后是stripped，则说明该文件的符号表信息和调试信息已被去除



    $ readelf -H

    -a 
    --all					显示全部信息,等价于 -h -l -S -s -r -d -V -A -I
    -h 
    --file-header			显示ELF文件开始的文件头信息
    -l 
    --program-headers
    --segments				显示程序头（段头）信息
    -S 
    --section-headers
    --sections				显示节头信息
    -g 
    --section-groups		显示节组信息
    -t 
    --section-details		显示节的详细信息
    -e 
    --headers				显示全部头信息，等价于: -h -l -S
    -s 
    --syms 
    --symbols 				显示符号表节的信息，包含静态符号表（.symtab）和动态符号表（.dynsym）
                            如果只关心动态符号表可以直接使用--dyn-syms
                            如果符号有相应的版本信息，则会显示该版本信息
                            版本字符串显示为符号名称的后缀，并以@字符开头，例如foo@VER_1。 
                            在解析未版本化引用的符号时，如果该版本是要使用的默认版本，则将显示为后缀，其后跟两个@字符，例如foo@@VER_2
    -n 
    --notes 				显示note段/节的信息
    -r 
    --relocs 				显示可重定位节的信息
    -u 
    --unwind 				显示unwind节信息
    -d 
    --dynamic 				显示动态节的信息
    -V 
    --version-info 			显示版本段的信息 
    -A 
    --arch-specific 		显示特定结构体系信息
    -c 
    --archive-index			显示二进制文档的标头部分中包含的文件符号索引信息 
    -D 
    --use-dynamic 			显示符号时，使用动态节中的符号哈希表，而不是符号表节。显示重定位时，使显示动态重定位而不是静态重定位
    -x <number or name>
    --hex-dump=<number or name>	
                            显示指定节的内容为十六进制字节
    -p <number or name>
    --string-dump=<number or name>	
                            显示指定节的内容为可打印的字符串
    -R <number or name>
    --relocated-dump=<number or name>	
                            显示指定节的内容为十六进制字节，并在显示之前重新定位
    -z
    --decompress			要求被x，R或p选项存储的节在显示之前先解压缩。如果未压缩，则将按原样显示
    -w[lLiaprmfFsOoRtUuTgAckK]
    --debug-dump[=rawline,=decodedline,=info,=abbrev,=pubnames,=aranges,=macro,=frames,=frames-interp,=str,=str-offsets,=loc,=Ranges,=pubtypes,=trace_info,=trace_abbrev,=trace_aranges,=gdb_index,=addr,=cu_index,=links,=follow-links]
                            显示文件中DWARF调试节的内容 
    -I 
    --histogram 			显示符号表时，显示bucket list长度的柱状图
    -W 
    --wide 					允许输出宽度超过80个字符显示在一行上
    -H 
    --help 					显示readelf可理解的命令行选项
    -v 
    --version 				显示readelf的版本信息
    @file 					从<file>中获取命令行选项


### strings
打印某个文件的可打印字符串

    $ strings HelloWorld | grep xxx


### size
查看文件段大小
text段：正文段字节数大小
data段:包含静态变量和已经初始化的全局变量的数据段字节数大小
bss段：存放程序中未初始化的全局变量的字节数大小
当我们知道各个段的大小之后，如果有减小程序大小的需求，就可以有针对性的对elf文件进行优化处理。

    $ size HelloWorld
    text    data     bss     dec     hex filename
    8526     704       8    9238    2416 HelloWorld


### strip
去掉elf文件中所有的符号信息

    $ ls -al HelloWorld
    -rwxrwxrwx 1 wk wk 59872 Mar 13 15:51 HelloWorld
    $ strip HelloWorld
    $ ls -al HelloWorld
    -rwxrwxrwx 1 wk wk 18568 Mar 15  2023 HelloWorld
    $ size HelloWorld
    text    data     bss     dec     hex filename
    8526     704       8    9238    2416 HelloWorld



### addr2line
定位crash问题
有时候程序崩溃了但不幸没有生成core文件，是不是就完全没有办法了呢？运行完cmdTest之后，我们通过dmesg命令可以获取到以下内容

    [27153070.538380] traps: cmdTest[2836] trap divide error ip:40053b sp:7ffc230d9280 error:0 in cmdTest[400000+1000]

该信息记录了cmdTest运行出错的基本原因（divide error）和出错位置（40053b）

    addr2line -e cmdTest 40053b
    /home/hyb/practice/cmdTest.c:4



### pmap
pmap既可以获取正在运行时的进程的内存映射信息，也可以获取进程的依赖共享库信息

    PD2020:/ # ps -ef | grep cdsprpcd
    system        1266     1 1 11:06:35 ?     00:26:35 cdsprpcd
    root         31754 31447 1 10:49:04 pts/0 00:00:00 grep cdsprpcd

    PD2020:/ # pmap 1266 | head
    1266: /vendor/bin/cdsprpcd
    00000062984c7000      4K r----  /vendor/bin/cdsprpcd
    00000062984c8000      8K --x--  /vendor/bin/cdsprpcd
    00000062984ca000      4K r----  /vendor/bin/cdsprpcd
    0000007b5945f000     28K r----  /system/lib64/libcutils.so
    0000007b59466000     32K --x--  /system/lib64/libcutils.so
    0000007b5946e000      4K rw---  /system/lib64/libcutils.so
    0000007b5946f000      8K r----  /system/lib64/libcutils.so
    0000007b59471000      4K rw---    [anon:.bss]
    0000007b5949b000     16K r----  /system/lib64/libnetd_client.so







## 动态加载
由于之前进程的整个程序和数据必须处于物理内存当中，因此进程的大小受物理内存大小的限制。为了获得更好的内存空间使用率的话，我们可以去使用动态加载。

采用动态加载，一个子程序只有在调用的时候才会被加载，所有的子程序都以可重定位的方式保存在磁盘上。主程序装入内存并且去执行，当一个子程序需要调用另一个子程序的时候，调用子程序的时候回先去检查另一个子程序是否已经加载，如果没有的话，就让可重定位的链接程序去加载所需要的子程序，并且去更新程序的地址表去反映这一变化。

动态加载的优点其实就是不用的子程序决不会被加载，动态加载不需要操作系统提供特别的支持，利用这种方法来涉及程序主要是用户的责任，不过操作系统可以帮助程序员，如提供子程序库以实现动态加载

动态库的动态加载技术，Windows和Linux都提供了相应的函数来打开动态库、获取函数指针和关闭动态库。


## 动态链接
有的操作系统只支持静态链接，也就是其实就是加载程序合并到二进制程序镜像中。而动态链接其实就是将链接延迟到了运行时，这个被用在系统库是极好的，因为如果说我们有很多应用程序都去调用了系统库的某些子程序，如果没有动态链接，我们就需要在这些应用程序中都去复制一份这些子程序的副本，这无疑是浪费磁盘空间和内存空间的

如果有了动态链接，二进制镜像当中对每个库程序的引用都有一个存根，存根其实就是一小段代码，用来指出如何定位适当的内存驻留库程序，或者说就是用来指出如果该程序不在内存当中应该如何装入库，在执行存根的时候，它会首先去检查子程序是否已在内存当中，如果不在的话就会将子程序装入内存。不管如何，存根都会用子程序的地址去替换自己，并且开始执行子程序，所以下次再指向子程序代码的时候，就可以直接进行执行了。使用系统库的所有的进程只需要有一份系统库代码就够了

动态链接也可用于库更新比如说修复了些许漏洞，一个库可以被新的版本所替代，并且该库的所有程序会自动的使用新的版本，如果没有动态链接，那么所有的这些程序都必须重新去链接这样才可以去访问新的库，以及为了不使程序错用新的、不兼容版本的库，程序和库将包含版本信息，这样的话多个版本就都可以装入内存，然后程序可以通过版本信息去决定使用库的哪个版本

通过man ld可以查到以下链接顺序

       1.  Any directories specified by -rpath-link options.
       2.  Any directories specified by -rpath options.  The difference between -rpath and -rpath-link is that directories specified by -rpath options are included in the executable and used at runtime, whereas the -rpath-link option is only effective at link time. Searching -rpath in this way is only supported by native linkers and cross linkers which have been configured with the --with-sysroot option.
       3.  On an ELF system, for native linkers, if the -rpath and -rpath-link options were not used, search the contents of the environment variable "LD_RUN_PATH".
       4.  On SunOS, if the -rpath option was not used, search any directories specified using -L options.
       5.  For a native linker, the search the contents of the environment variable "LD_LIBRARY_PATH".
       6.  For a native ELF linker, the directories in "DT_RUNPATH" or "DT_RPATH" of a shared library are searched for shared libraries needed by it. The "DT_RPATH" entries are ignored if "DT_RUNPATH" entries exist.
       7.  The default directories, normally /lib and /usr/lib.
       8.  For a native linker on an ELF system, if the file /etc/ld.so.conf exists, the list of directories found in that file.
       If the required shared library is not found, the linker will issue a warning and continue with the link.

注意:有的操作系统(我的ubuntu18.04就是)支持RUNPATH的话,会在RUNPATH设置情况下自动忽略RPATH(后续详解)

现代连接器在处理动态库时将链接时路径（Link-time path）和运行时路径（Run-time path）分开，
用户可以通过-L指定连接时库的路径，通过-R（或-rpath）指定程序运行时库的路径，大大提高了库应用的灵活性.(ld默认搜索/lib和/usr/lib这两个目录)

### 指定链接时路径
链接器选项 -L: 用于链接时指定显示链接的库的搜索路径（优先级高）
也就是所有的 -lxxx选项里的库都会先从 -L 指定的目录去找,然后是默认的地方(/lib and /usr/lib).

rpath-link
链接选项-rpath-link用于在链接时指定间接依赖库的目录（-link的意思为路径并不写入二进制文件因此不做运行时搜索路径使用）。
rpath-link区别于-L用于指定间接依赖的动态库的搜索路径，而-L为直接依赖的搜索路径。


### 指定运行时路径
修改环境变量LD_LIBRARY_PATH
在运行时搜索直接或间接依赖。优先级低于RPATH为第二优先级

增加配置文件ld.so.confg中搜索路径。

这些都可能改变当前的运行环境，可能导致一些意想不到的事情发生，因此不是很好的方法。
对系统影响最小的方法是在编译的时候指定运行时动态加载库的搜索路径。

rpath
1. 用于在链接时指定直接或间接链接的库搜索路径（最高优先级）
2. 指定运行时的本二进制文件的直接或间接依赖的动态加载库搜索路径（最高优先级），写入二进制文件中RPATH。
注：有些系统默认开启链接选项 -enable-new-dtags ,导致 -rpath 生成 RUNPATH。通过指定链接选项 -disable-new-dtags 来使其生成 RPATH。
如果有多个rpath，注意是”:”，和PATH的路径分隔符一样，还要注意中英文字符：
    
    -Wl,-rpath,d1:..:dn

RUNPATH
写入在二进制文件中，用于指定运行时本二进制文件的直接依赖动态加载库搜索路径(优先级低于 LD_LIBRARY_PATH )。存在时覆盖二进制文件中 RPATH。

    //使用DT_RPATH
    gcc main.c -ldl -Wl,--rpath=.,--disable-new-dtags
    //使用DT_RUNPATH
    gcc main.c -ldl -Wl,--rpath=.,--enable-new-dtags



chrpath -r [new_rpath] [lib_name] 注:只能改路径比原来短的.
chrpath -l [lib_name] 显示rpath路径
patchelf --set-rpath [new_rpath] [lib_name] 注:没有限制路径长度，但是修改的是 RUNPATH


### 动态链接的步骤

1. 启动动态连接器本身（自举）
2. 装载所有需要的共享对象
3. 重定位和初始化

#### 动态连接器自举
我们前面说了动态链接器本身也是一个共享对象，但是事实上它有一定的特殊性。对于普通共享对象文件来说，它的重定位工作由动态链接器来完成；共享对象文件可以依赖于其他共享对象，其中的被依赖的共享对象也是由动态链接器负责链接和装载，可是对于动态链接器本身来说，它的重定位工作由谁来完成？它是否可以依赖于其他的共享对象？

这是一个“鸡生蛋，蛋生鸡”的问题，为了解决这种无休止的循环，动态链接器这个“鸡”必须有些特殊性。首先动态链接器本身不可以依赖其他任何共享对象；其次是动态链接器本身所需要的全局和静态变量的重定位工作由本身完成。对于第一个条件我们可以人为地控制，在编写动态链接器时保证不使用任何系统库、运行库；对于第二个条件，动态链接器必须在启动时有一段精巧的代码来完成这项工作同时又不使用到全局和静态变量。这种具有一定限制条件的启动代码往往被称为自举（Bootstrap）。

动态链接器入口地址即是自举代码的入口，当操作系统将进程控制权交给动态链接器时，动态链接器的自举代码开始执行。自举代码会首先找到它自己的GOT。而GOT的第一个项保存的是.dynamic段的偏移地址，由此找到了动态链接器本身的.dynamic段，通过.dynamic段中的信息（.dynamic段中存储了动态链接器重定位表和符号表等等用于动态链接的信息的偏移地址），自举代码便可以获得动态链接器本身的重定表和符号表等信息，从而得到动态链接器本身的重定位入口，先将它们全部重定位。从这一步开始，动态链接器代码中才可以开始使用自己的全局变量和静态变量。

实际上在动态链接器的自举代码中，除了不可以使用全局变量和静态变量之外，甚至不能调用函数，即动态连接器本身的函数也不能调用，这是为什么呢？其实我们在前面分析地址无关代码时已经提到过，使用PIC模式编译的共享对象，对于模块内部的函数调用也是采用跟模块外部函数调用一样的方式，即使用GOT/PLT的方式，所以在GOT/PLT没有被重定位之前，自举代码不可以使用任何全局变量，也不可以调用函数。


Glibc2.6.1源码中的elf/rtld.c
从Glibc源码中自举代码末尾的注释我们可以看到，作者说"Now life is sane"，可以想象动态链接器的作者的如释重负，完成自举之后，可以自由地调用各种函数并且随意访问全局变量了。


#### 装载共享对象
完成基本自举以后，动态链接器将可执行文件和链接器本身的符号表都合并到一个符号表中，我们可以称它为全局符号表（Global Symbol Table）。然后链接器开始寻找可执行文件所依赖的共享对象，依赖对象是存放在.dynamic段中的，在.dynamic段中有一个类型是DT_NEEDED，它所指出该可执行文件（或共享对象）所依赖的共享对象。由此，链接器可以列出可执行文件所需要的所有共享对象，并将这些共享对象的名字放入到一个装载集合中。然后链接器开始从集合里取一个所需要的共享对象的名字，找到相应的文件后打开该文件，读取相应的ELF文件头和.dynamic段，然后将它相应的代码段和数据段映射到进程空间中。如果这个ELF共享对象还依赖其他共享对象，那么将所依赖的共享对象的的名字放到装载集合中。如次往复，直到所有依赖的共享对象都被装载进来为止，当然链接器可以有不同的装载顺序，如果我们把依赖关系看作一个图的话，那么装载过程就是一个图的遍历过程，可以使用深度优先也可以使用广度优先来遍历整个图，这取决于链接器。比较常见的算法一般都是广度优先。

当一个新的共享对象被装载进来的时候，它的符号表就会被合并到全局符号表中，所以当所有的共享对象都被装载进来的时候，全局符号表里面将包含进程中所有的动态链接所需要的符号。

重定位和初始化
当上面的步骤完成之后，链接器开始重新遍历可执行文件和每个共享对象的重定位表，将它们的GOT/PLT中每个需要重定位的位置进行修正。因为此时动态链接器已经拥有了进程的全局符号表，所以这个修正的过程也就比较容易了，跟我们前面提到的地址重定位的原理基本相同。动态链接器会根据重定位类型，来计算相应重定位入口地址，在这里就不做详细介绍了。

重定位完成之后，如果共享对象有.init段，那么动态链接器就会执行.init段中的代码，用以实现共享对象特有的初始化过程，比如最常见的，共享对象中的C++的全局/静态对象的构造就需要通过.init来初始化。相应地，共享对象中还可能有.finit段，当进程退出时会执行.finit段中的代码，可以用来实现类似C++全局对象析构之类的操作。

如果进程的可执行文件也有.init段，动态连接器不会执行它（不是.init段不执行，而是由其他部分的程序执行），因为可执行文件中的.init和.finit段由程序初始化部分代码负责执行。

当完成了重定位和初始化之后，所有的准备工作就宣告完成了，所需要的共享对象也都已经装载并且链接完成了，这时候动态连接器的工作就做完了，将进程的控制权交给程序的入口并且开始执行了。

从上面的介绍，我们不难发现动态链接的最困难的部分就是动态链接器自举的过程。当链接器完成自举后，就会去装载所有的共享库及其依赖的共享库，当装载完所有的共享库后就可以进行对共享库进行重定位和初始化，完成后动态链接的工作就完成了，进程的控制权就交给程序的入口并开始执行。


#### 显式运行时链接
支持动态链接的系统往往都支持一种更加灵活的模块加载方式，叫做显式运行时链接（Explicit Run-time Linking），有时候也叫做运行时加载。也就是让程序自己在运行时控制加载指定的模块，并且可以在不需要该模块时将其卸载。从前面对动态链接器的了解来看，如果动态链接器可以在运行时将共享模块装载进内存并且可以进行重定位等操作，那么这种运行时加载在理论上也是很容易实现的。而且一般的共享对象不需要进行任何修改就可以进行运行时加载，这种共享对象往往被叫做动态装载库（Dynamic Loading Library），其实本质上它跟一般的共享对象没有什么区别，只是程序开发者使用它的角度不同。

这种运行时加载使得程序的模块组织变得很灵活，可以用来实现一些诸如插件、驱动等功能。当程序需要到某个插件或者驱动的时候，才将相应的模块装载进来， 而不需要从一开始就将他们全部装载进来，从而减少了程序的启动时间和内存使用。并且程序可以在运行的时候重新加载某个模块，这样使得程序本身不必重新启动而实现模块的增加、删除、更新等，这对于很多需要长时间运行的程序来说是很大的优势。最常见的例子就是Web服务器程序，对于Web服务器程序来说，它很需要根据配置来选择不同的脚本解释器、数据库连接驱动等，对于不同的脚本解释器分别做成一个独立的模块，当Web服务器需要某种脚本解释器的时候可以将其加载进来（有点类似于操作系统执行支持格式的可执行文件，会根据不同的可执行文件格式来找到对应的加载程序）；这对于数据库连接的驱动程序也是一样的原理。另外对于一个可靠的Web服务器来说，长期运行是必要的保证，如果我们需要增加某种脚本解释器，或者某个脚本解释器模块需要升级，则可以通知Web服务器程序重新装载该共享模块以实现相应的目的。

在Linux中，从文件本的格式上看来，动态库实际上跟一般的共享对象没有区别，主要的区别是共享对象是由动态链接器在程序启动之前负责装载和链接的，这一系列步骤都由动态链接器自动完成，对于程序本身是透明的；而动态库的装载则是通过一系列由动态链接器提供的API玩完成的（程序员自己调用），具体地讲有4个函数：

打开动态库（dlopen）
查找符号（dlsym）
错误处理（dlerror）
关闭动态库（dlclose）
程序可以通过这几个API对动态库进行操作。这几个API的实现是在/lib/libdl.so.2里面，它们的声明和相关常量被定义在系统标准头文件<dlfcn.h>。感兴趣的读者可以去看看这几个函数的具体意义，在这里就不详细介绍了。






 
https://blog.csdn.net/MyArrow/article/details/9625495

https://blog.csdn.net/Windgs_YF/article/details/104004620

https://www.cnblogs.com/ZhaoxiCheung/p/9424930.html

https://www.jianshu.com/p/72cc08405a5a

https://www.cnblogs.com/Anker/p/3746802.html










https://www.jianshu.com/p/90f5ec723175

https://blog.csdn.net/counsellor/article/details/81543197

https://blog.csdn.net/fengel_cs/article/details/60958569

https://blog.csdn.net/weixin_33775582/article/details/86321607





