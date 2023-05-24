[TOC]

# Android系统框架



Android的系统架构采用了分层架构的思想。下图为谷歌官方提供的经典分层架构图，从下往上依次分为Linux内核、硬件抽象层（HAL）、系统Native库和Android运行时环境、Java框架层以及应用层这5层架构，其中每一层都包含大量的子模块或子系统。


![](Android系统框架.png)


Android底层内核空间以Linux Kernel作为基石，上层用户空间由Native系统库、虚拟机运行环境、框架层组成，通过系统调用(Syscall)连通系统的内核空间与用户空间。对于用户空间主要采用C++和Java代码编写，通过JNI技术打通用户空间的Java层和Native层(C++/C)，从而连通整个系统。

## Application 应用程序层
AOSP/android10/packages

这一层装配了一个核心应用程序集合，包括浏览器、联系人、电话、日历、相机等应用；
除此之外，用户开发的Android应用也处于这一层，比如微博、微信、QQ等；
总的来说，应用程序层包括了所有的Android应用程序，其可以分为系统自带的核心应用程序以及用户自行开发的应用程序。


## Framework 应用程序框架层
AOSP/android10/frameworks

它提供了大量可供开发人员使用的应用程序接口（Application Programming Interface，API），Android自带的很多核心应用也是使用这些API完成的。
应用程序框架层主要提供的组件如下表所示：

| Framework | 功能 |
| :---: | :---: |
| Activity Manager 	|	 |
| Location Manager 位置管理器	|	提供地理位置及定位功能服务 |
| Package Manager 包管理器	|   管理所有安装在Android系统中的应用程序 |
| Notification Manager 通知管理器   |	使应用程序可以在状态栏中显示自定义的提示信息 |
| Resource Manager 资源管理器  | 提供应用程序使用的各种非代码资源，如本地字符串、图片、布局文件、颜色文件等 |
| Telephony Manager 电话管理器	|	管理所有的移动设备功能  |
| Window Manager 窗口管理器	|	管理所有开启的窗口程序  |
| Content Provider 内容提供者	|	使不同应用程序之间可以共享数据  |
| View System 视图系统	|	丰富的、可扩展的视图集合，可用于构建一个应用程序，包括列表(Lists)、网格(Grids)、文本框 (TextBoxes)、按钮(Buttons)，甚至是内嵌的Web浏览器    |


## Native 系统库
AOSP/android10/external

Android包含一些原生C/C++库，这些库能够被安卓系统的不同组件使用。它们通过Android应用框架为开发者提供服务。
开发者可以通过调用Java API Framework来使用原生库的功能，也可以用Android NDK直接调用原生库。

系统库包括九个子系统，分别是：
| Native | 功能 |
| :---: | :---: |
| 系统C语言库 | 标准C语言系统库 (libc) 的 BSD 衍生，调整为基于嵌入式 Linux 设备 |
| 多媒体库 | 基于 PacketVideo 的 OpenCORE，这些库支持播放和录制许多流行的音频和视频格式，以及静态图像文件，包括 MPEG4、H.264、MP3、AAC、AMR、JPG、PNG |
| 图层管理（Surface Manager） | 对显示子系统的管理，并且为多个应用程序提供 2D 和 3D 图层的无缝融合 |
| SQLite | 轻量级关系数据库引擎 |
| 3D库 | 基于OpenFLES1.0 APLs实现，该库可以使用硬件3D加速或者使用高度优化3D软加速 |
| FreeType | 位图与矢量字体显示渲染 |
| LibWebCore | 新式的 Web 浏览器引擎，支持 Android 浏览器和内嵌的 Web 视图 |
| SGL | 内置的2D图形引擎 |
| SSL | 支持数据通信 |


## Android Runtime
AOSP/android10/art
AOSP/android10/dalvik

Android运行时包括核心库以及Dalvik虚拟机（Android 5.0以后更改为ART虚拟机）。
前者既兼容了大多数Java语言所需要的功能函数，又包括了Android的核心库，比如android.os、http://android.net、android.media等。
后者是一种基于寄存器的Java虚拟机，每一个Android应用都运行在单独的进程中，拥有一个独立的Dalvik虚拟机实例。


Android Runtime是Android系统里面的核心模块之一。在编译Android代码后会生成APK文件，如果我们打开APK文件，会发现.dex后缀的文件，这些文件包含了了我们APP运行的所有源码，它们的表现形式为—— 字节码(byte code)。而字节码是无法直接被机器执行的，所以Android Runtime的作用是将.dex文件的字节码(byte code)翻译成机器可以直接执行的机器码(machine code)。机器码(machine code)是一系列可以直接被机器理解和CPU执行的指令。

https://blog.csdn.net/hp910315/article/details/49095903


### Dalvik虚拟机概述
Google为Android平台专门设计的一套虚拟机来运行Android应用程序，那就是Dalvik虚拟机（Dalvik Virtual Machine，DVM）。

早期Android系统Android Runtime的具体实现，直到在Android K发布后，光荣退役。
受限于早期智能手机的存储空间和RAM，Dalvik的出现便是为了解决优化一个核心指标——RAM的使用。
不同于现在动辄6G，8G的RAM，早期智能手机的RAM最低可以低到200MB。
所以为了避免编译整个APP，Dalvik采用了 Just In Time （JIT）的编译策略。

注意：尽管Android应用程序是通过Java语言来开发的，但其并不是运行在标准的Java虚拟机上。
DalVik虚拟机作为Android平台的核心组件，拥有如下特点： 
1. 体积小，占用内存空间小； 
2. 执行专有的DEX（Dalvik Executable）文件格式，体积更小，执行速度更快； 
3. 常量池采用32位索引值，寻址类方法名、字段名、常量更快； 
4. 基于寄存器架构，并拥有一套完整的指令系统； 
5. 提供对生命周期的管理、堆栈的管理、线程的管理、安全和异常的管理以及垃圾回收等重要功能； 
6. 所有Android应用程序都运行在Android系统进程里，每个进程对于一个Dalvik虚拟机实例。
Dalvik虚拟机采用的是JIT（Just-in-time Compilation，即时编译）技术，在运行应用时将字节码翻译为机器码，从而使程序的执行速度更快。但随着硬件水平的不断发展以及人们对更高性能的需求，Dalvik虚拟机的不足日益突出。
Google在2014年推出了新的虚拟机ART，并从Android5.0开始废弃了Dalvik，全面推行ART。
ART虚拟机采用AOT（Ahead-of-time）技术，在应用程序安装时就会将字节码转换为机器码，从而优化了应用运行的速度。
在内存管理方面，ART也有比较大的改进，对内存分配和回收都做了算法优化，降低了内存碎片化程度，回收时间也得以缩短。


### Dalvik虚拟机与Java虚拟机的区别
|  | Dalvik虚拟机 | Java虚拟机 |
| :---: | :---: | :---: |
| 虚拟机架构 | 基于寄存器架构，数据访问通过寄存器间直接传递，编译和运行都会更快一些	| 基于栈架构，编译和运行会慢一些 |
| 字节码 | 执行.dex格式的Dalvik字节码，由.class文件通过Android SDK目录下名为dx的工具压缩后产生的，体积更小 | 执行.class格式的Java字节码 |
| 运行环境 | 一个应用启动会运行一个单独的虚拟机，运行在独立的进程中	| 所有应用都运行在同一个虚拟机中 |


### JIT(Just in Time)
JIT的核心原理是编译器仅仅只在APP运行到相关代码的时候才会去编译当前运行的一小部分代码。也正是因为这一原理，Dalvik虚拟机仅仅只在运行时编译他所需要执行的代码，这样就可以节省下大量的RAM。
但是我们很快就能想到，这种策略有一个致命的问题，因为一切都发生在运行时，当我们运行时虚拟机才会去编译我们所需要的代码，编译完成后，我们才能执行。这样就会严重影响APP运行时候的性能。
虽然Google为Dalvik这种机制做了非常多的优化，比如对一些常用的代码进行缓存，不用频繁编译，但是这些优化手段能够起到的作用非常有限。
随着时间的推移，我们的手机性能变得越来越强劲，RAM也变得越来越大，存储空间也越来越大。
我们日常所使用的APP，随着功能的增加，也变得越来越大。开发者们开始发现JIT带来的性能影响也越来越大。

Google便在Android L中引入了新的Android Runtime 虚拟机实现——ART


### ART虚拟机
ART虚拟机的实现与Dalvik虚拟机的实现是完全不同的。它引入了一种新的编译策略——Ahead of Time（AOT） 而不是继续使用或者优化JIT。


### AOT（Ahead of Time）
AOT的核心原理是，在APP安装时便将字节码编译成机器码。
这样的实现使得APP在运行时可以直接使用机器码，而不是在运行时再进行翻译，这种方法可以将APP的运行性能提高到使用JIT的20倍甚至更多。
但同样的它所带来的问题是 安装APP时间会大大拉长，因为系统需要将字节码翻译成机器码。同时翻译后的产物也会占用更多的存储空间。
随着ART的推广，开发者们发现对于一个APP的大部分功能，用户几乎很少使用（二八定律）。
所以预编译整个APP，在目前来看是没有必要的，所以Google在Android N中重新引入了 JIT，同时引入了另一种编译策略profile-guide optimization（PGO）。


### PGO（Profile-Guide Optimization）
它的原理就是，当ART发现热点代码的时候，这些代码便会被直接翻译成机器码，并被缓存下来。
随着执行次数的增加，用户的常用功能被直接探测，那么一个符合用户使用习惯的profile文件便会被生成。
用户APP的执行性能也会越来越高。而这一部分的操作也并不是在运行时执行，而是在用户手机空闲的时候根据profile文件做预编译工作。
它所带来的缺点就是，系统需要根据用户的使用习惯和方法来进行预编译，也就是说用户在第一次使用的时候只会使用JIT，导致首次使用的性能较差。Google为了解决这种问题在Android P中引入了新的编译策略 profiles in the cloud。


### PIC（profiles in the cloud）
它的核心原理是，大部分用户的使用模式趋同，所以对于已经安装了该APP的用户，Google会根据大部分人的行为习惯生成profile文件并存储在云端，所以在新用户安装时，这份文件会随着APK文件同时下发，进行预编译的处理。然后在用户实际使用时，还会不断优化profile文件，进行预编译处理。



## HAL 硬件抽象层
AOSP/android10/hardware

硬件抽象层处于应用程序框架层和Linux内核驱动之间，用于将硬件抽象化。简单来说，就是对内核驱动程序进行封装，向上提供接口，向下屏蔽具体实现细节。
系统抽象层包含多个库模块，每个模块都为特定类型的硬件组件实现接口，例如相机、蓝牙模块。
当应用程序框架层API要访问设备硬件时，Android系统会为该硬件组件加载库模块。


## Linux Kernel Linux内核层
AOSP/android10/bionic           (Drivers调用Linux内核API)

Android基于Linux提供核心系统服务，例如安全、内存管理、进程管理、网络堆栈、驱动模型。
除了标准的 Linux 内核外，Android 还增加了内核的驱动程序，如Binder(IPC)驱动、显示驱动、输入设备驱动、音频系统驱动、摄像头驱动、WiFi驱动、蓝牙驱动、电源管理。




