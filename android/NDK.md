[TOC]

# NDK


https://developer.android.google.cn/ndk/guides/ndk-build?hl=zh-cn

需要代理：
http://developer.android.com/sdk/ndk/
网友翻译：
https://www.cnblogs.com/qq78292959/archive/2011/11/02/2232962.html

[Android NDK 从入门到精通（汇总篇）](https://blog.csdn.net/afei__/article/details/81290711)



mips 、mips64 和 armeabi 在 ndk r17 中已经移除，再使用独立交叉工具链的时候不能指定 --arch=mips/mips64/armeabi。
另外从 ndk r19 已经内置交叉编译工具链，不需要使用独立工具链。








## Android工程配置NDK

在 jdk10 之前的版本可以通过 javah 来生成，但是在 jdk10 已经移除了 javah 工具，
现在可以通过 javac -h 来生成 JNI 头文件，使用如下的命令。

javac -h ./ xxx.java


1. 修改顶层 local.properties 文件中 ndk 的路径

    sdk.dir=E\:\\baidu\\android\\AS\\sdk
    ndk.dir=E\:\\baidu\\android\\AS\\sdk\\ndk\\21.1.6352462

2. app层 build.gradle 中指定 ndk 版本
自动编译
```
android {
    defaultConfig {
        externalNativeBuild {
            ndkBuild {
                //arguments "-DANDROID_ARM_NEON=TRUE", "-DANDROID_PLATFORM=android-21", "-DANDROID_STL=c++_shared"
                // Sets an optional flag for the C compiler.
                cFlags "-D__STDC_FORMAT_MACROS"
                // Sets optional flags for the C++ compiler.
                cppFlags "-fexceptions", "-frtti", "-std=c++11", "-DDEBUG=1"
                abiFilters "arm64-v8a"
                //abiFilters += listOf("x86", "x86_64", "armeabi", "armeabi-v7a", "arm64-v8a")
            }
        }
    }
    ...
    externalNativeBuild {
        ndkBuild {
            path file('src/main/jni/Android.mk')
        }
    }
    ndkVersion '21.1.6352462'
}
```

手动编译
```
android {
    defaultConfig {
        ndk { moduleName "GoogleLib" }
        sourceSets.main {
            jni.srcDirs = []
            jniLibs.srcDir "src/main/libs"
        }
    }
}

    //cd app/src/main
    //ls jni/
    //ndk-build
```



## Application.mk
Application.mk 指定了 ndk-build 的项目范围设置。下面是项目常用到的变量表。


### APP_ABI
APP_ABI  编译代码输出的 ABI 列表, 可以指指定的值：
armeabi-v7a, 
arm64-v8a, 
x86_64, 
x86, 
mips(在 ndk r17 移除), 
mips64（ndk r17 移除）,
all (当前 ndk 支持的所有 abi 列表)

值得注意的是：中间用过空格隔开，列入 armeabi-v7a x86，例如：

APP_ABI := armeabi-v7a arm64-v8a x86_x64 x86


### APP_MODULES
APP_MODULES 编译模块列表, 指定 Android.mk 中模块的名称，控制那些模块需要被编译。

APP_MODULES := hello


### APP_OPTIM
APP_OPTIM 可选命令，如果没有设置，会根据应用是否属于调试模式而自动设置 debug 或 release。


### APP_CFLAGS
APP_CFLAGS 指定 (C/C++) 优化代码的参数，用于指定头文件路径便于编译器
包含文件（例如：APP_CFLAGS := -I$(LOCAL_PATH)/include）。

一般与优化相关的就是指定 Ox，其中 x 的优化级别为 0,1,2(或者-Os)


### APP_CPPFLAGS
APP_CPPFLAGS 和 APP_CFLAGS 类似，只指定 c++ 编译器的优化参数


### APP_LDFLAGS
APP_LDFLAGS 关联可执行文件和共享库时指定，例如可执行文件中依赖其他动态库，对静态库没有影响


### APP_PLATFORM
APP_PLATFORM 声明编译此应用所面向的 Android API 级别，对应于应用的 minSdkVersion。
如果未指定，ndk-build 将以 NDK 支持的最低 API 级别为目标。

需要注意的是，你当前的 ndk 可能不一定支持你所设置的平台，你可以参考你使用的 ndk 
版本 platform 下是否包含指定的平台。例如我这里是 ndk r20 最低为 android-16


### APP_STL
用于此应用的 C++ 标准库。默认情况下使用 system STL。
其他选项包括 c++_shared、c++_static 和 none。

值得注意的是： 
1. 系统运行时指的是 /system/lib/libstdc++.so。请勿将该库与 GNU 的全功能
libstdc++ 混淆。在 Android 系统中，libstdc++ 只是 new 和 delete。对于全功能 C++
标准库，请使用 libc++，libc++ 的共享库为 libc++_shared.so，静态库为 libc++_static.a。

2. 使用静态运行时（以及一般静态库）要特别小心，例如：
```
# Application.mk
    APP_STL := c++_static // 指定 c++_static 静态运行库
    
# Android.mk
    include $(CLEAR_VARS)
    LOCAL_MODULE := foo
    LOCAL_SRC_FILES := foo.cpp
    include $(BUILD_SHARED_LIBRARY) // 动态库 foo 

    include $(CLEAR_VARS)
    LOCAL_MODULE := bar
    LOCAL_SRC_FILES := bar.cpp
    LOCAL_SHARED_LIBRARIES := foo
    include $(BUILD_SHARED_LIBRARY) // 动态库 bar 依赖 foo
``` 

它会导致两个 foo 和 bar 都依赖 c++_static, 增加了应用体积不说，还会有如下风险：
. 内存在一个库中分配，而在另一个库中释放，从而导致内存泄漏或堆损坏。
. libfoo.so 中引发的异常在 libbar.so 中未被捕获，从而导致应用崩溃。
. std::cout 的缓冲未正常运行。


### APP_BUILD_SCRIPT
APP_BUILD_SCRIPT 要从其他位置加载 Android.mk 文件，请将 APP_BUILD_SCRIPT 设置为
Android.mk 文件的绝对路径。


## Android.mk
Android.mk 文件位于项目 jni/ 目录的子目录中。Application.mk 是对于整个应用共同的配置，而 Android.mk 指定构建的模块如何构建。


### LOCAL_PATH
LOCAL_PATH Android.mk 路径，设置为 $(call my-dir)

my-dir 是一个宏函数，用来返回当前 Android.mk 的路径，例如：

LOCAL_PATH := $(call my-dir)


### CLEAR_VARS
CLEAR_VARS CLEAR_VARS 变量指向一个特殊的 GNU Makefile，后者会清除许多 LOCAL_XXX
变量，例如 LOCAL_MODULE、LOCAL_SRC_FILES 和 LOCAL_STATIC_LIBRARIES。请注意，GNU
Makefile 不会清除 LOCAL_PATH。

include $(CLEAR_VARS)


### LOCAL_MODULE
LOCAL_MODULE 模块名称

值得注意的是：每个模块名称必须唯一，且不含任何空格。编译系统在生成最终共享库文件时
，会对您分配给 LOCAL_MODULE的名称自动添加正确的前缀和后缀。
例如

LOCAL_MODULE := hello-jni

上述示例会生成名为 libhello-jni.so 的库。


### LOCAL_SRC_FILES
LOCAL_SRC_FILES  C 和/或 C++ 源文件列表

例如：

LOCAL_SRC_FILES := a.c b.c 

记住文件列表中间用空格隔开


### BUILD_SHARED_LIBRARY
BUILD_SHARED_LIBRARY 将所有内容连接到一起,BUILD_SHARED_LIBRARY 变量指向一个 GNU Makefile 脚本，
该脚本会收集您自最近 include 以来在 LOCAL_XXX变量中定义的所有信息。此脚本确定要编译的内容以及编译方式。

最终会生成一个共享库。 

include $(BUILD_SHARED_LIBRARY)


### BUILD_STATIC_LIBRARY
BUILD_STATIC_LIBRARY 生成的是一个静态库，可以用来链接到其他共享库中。

include $(BUILD_STATIC_LIBRARY)


### PREBUILT_SHARED_LIBRARY
PREBUILT_SHARED_LIBRARY 指向用于指定预编译共享库的编译脚本。与 BUILD_SHARED_LIBRARY 和 BUILD_STATIC_LIBRARY 的情况不同，
这里的 LOCAL_SRC_FILES 值不能是源文件，而必须是指向预编译共享库的一个路径。
例如 foo/libfoo.so。使用此变量的语法为：

include $(PREBUILT_SHARED_LIBRARY)


### PREBUILT_STATIC_LIBRARY
PREBUILT_STATIC_LIBRARY 跟 PREBUILT_SHARED_LIBRARY 类似


### LOCAL_C_INCLUDES
LOCAL_C_INCLUDES  追加到 include 搜索列表路径，用法如下：

LOCAL_C_INCLUDES := sources/foo


### LOCAL_CFLAGS
LOCAL_CFLAGS 这个在前面 Application.mk 中已经讲过，传递的一组 （C/C++）可选编译器标记。 但是如果你只是
指定头文件，最好使用 LOCAL_C_INCLUDES。

LOCAL_CFLAGS := -I$(LOCAL_PATH)/myinclude


### LOCAL_CPPFLAGS
LOCAL_CPPFLAGS 和 LOCAL_CFLAGS 类似，不过只针对 C++ 的可选编译器标记。


### LOCAL_LDLIBS
此变量列出了在编译共享库或可执行文件时使用的额外链接器标记。利用此变量，
您可使用 -l 前缀传递特定系统库的名称。
例如，以下示例指示链接器生成在加载时链接到 /system/lib/libz.so 的模块: 

LOCAL_LDLIBS := -lz


### LOCAL_LDFLAGS
此变量列出了编译系统在编译共享库或可执行文件时使用的其他链接器标记。例如，要在 ARM/X86 上使用 ld.bfd 链接器：

LOCAL_LDFLAGS += -fuse-ld=bfd

值得注意的是：如果为静态库定义此变量，编译系统会忽略此变量，并且 ndk-build 会显示一则警告。


### LOCAL_SHARED_LIBRARIES
LOCAL_SHARED_LIBRARIES: 当前模块依赖的共享库列表


### LOCAL_STATIC_LIBRARIES
LOCAL_STATIC_LIBRARIES: 存储当前模块依赖的静态库模块列表







## Android.bp

https://blog.csdn.net/tkwxty/article/details/105142182

https://blog.csdn.net/tahlia_/article/details/105494446









## C++库支持
[NDK 支持多种 C++ 运行时库](https://developer.android.google.cn/ndk/guides/cpp-support?hl=zh-cn#hr)






## build

显示参数
ndk-build V=1 ..



编译可执行文件时，可以加static选项，打包依赖的库，这样会不依赖于平台的基础库
android上(ndk)重写了很多基础函数头文件和实现，和linux上的不一样。
比如：
/mnt/d/Android/Sdk/ndk/21.1.6352462/sysroot/usr/include/bits/fortify/string.h
/usr/include/string.h









## 尝试APK链接gcc编译的库
NDK编译规则，库名要以lib开头，不能带版本号
LOCAL_SRC_FILES should point to a file ending with ".so" 
LOCAL_SRC_FILES points to a missing file

E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "libc.so.6" not found
cd /data/app/com.gc.udxxx/lib/arm64
mv libc.so libc.so.6
E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: library "ld-linux-aarch64.so.1" not found
mv libld-linux-aarch64.so ld-linux-aarch64.so.1

E AndroidRuntime: java.lang.UnsatisfiedLinkError: dlopen failed: cannot find verneed/verdef for version index=32770 referenced by symbol "_res" at "/data/app/com.gc.ud-hS0uu9lDhP3l-eChFtrpRw==/lib/arm64/libc.so.6"


### 在手机上尝试运行gcc编译的HelloWorld
gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-gcc

PD2020:/data # ./HelloWorld
/system/bin/sh: ./HelloWorld: No such file or directory

1. 使用静态链接`-static`

2. 将依赖的动态库复制到android系统中的/lib目录下

        file build_linux_aarch64/out/HelloWorld
        build_linux_aarch64/out/HelloWorld: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 3.7.0, BuildID[sha1]=7bf69e1b4efb49cf0eb7a21a74fcf6dad403b3e4, with debug_info, not stripped

        adb root
        adb remount
        mount -o rw,remount /
        adb push E:\baidu\linux\gcc\gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu\aarch64-linux-gnu\libc\lib\ld-2.25.so /lib/







