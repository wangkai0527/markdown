[TOC]

# Native

动态库的查找路径可以在路径 /system/etc/ 的ld.config*上配置

http://aospxref.com/android-11.0.0_r21/xref/bionic/linker/linker.cpp




libdvm.so 是 Android 的 Dalvik 虚拟机使用的动态库 ; Android 5.0 及以下系统使用 Dalvik 虚拟机 ;

libart.so 是 Android 的 Art 虚拟机使用的动态库 ; Android 5.1 及以上系统使用 Art 虚拟机 ;

目前所有的模拟器 , 真机获取的虚拟机动态库都是 libart.so  ; 5.0 以下的 Android 设备 , 现在很少了 ;



libandroid_runtime.so 是 Android 运行时相关的函数库 ; 如 : Java 层与 Native 层交互的 JNI 机制 , 系统控制机制 , 获取硬件设备 ( GPS , 陀螺仪 ) 数据 等 ;


libandroidfw.so 是 Android 的 Framework 层的 Native 实现部分的动态库 ,










