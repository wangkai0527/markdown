[TOC]

# CPU

https://blog.csdn.net/u010476739/article/details/127477977?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-4-127477977-blog-81048857.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EYuanLiJiHua%7EPosition-4-127477977-blog-81048857.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=9



## CPU架构

目前android支持如下7中CPU架构：
armeabi 第5代 ARM v5TE，使用软件浮点运算，兼容所有ARM设备，通用性强，速度慢（只支持armeabi）
armeabi-v7a 第7代 ARM v7，使用硬件浮点运算，具有高级扩展功能（支持 armeabi 和 armeabi-v7a，目前大部分手机都是这个架构）
arm64-v8a 第8代，64位，包含AArch32、AArch64两个执行状态对应32、64bit（支持 armeabi-v7a、armeabi 和 arm64-v8a）
*x86 intel 32位，一般用于平板（支持 armeabi(性能有所损耗) 和 x86）
x86_64 intel 64位，一般用于平板（支持 x86 和 x86_64）
mips 基本没见过（支持 mips）
mips64 基本没见过（支持 mips 和 mips_64）


Android是如何加载So库的
程序对当前手机cpu架构(比如 armeabi-v7a)做了适配，那手机跑程序时候就直接在这个目录下找对应的so库，如果找不到就直接报错
如果只对armeabi的手机cpu做适配，那么支持armeabi的手机都会去armeabi目录下找对应的so库



项目中如何适配
如果适配不止一个cpu架构，比如armeabi、 armeabi-v7a 、arm64-v8a这三个，那么一定要确保三个目录中的so库数目一样；第三方库如果支持者提供这三个cpu架构的so库，那非常理想，对应放到目录就可以；
如果适配的上面三个cpu架构，第三方库只提供了两个cpu（比如armeabi、 armeabi-v7a）的库，那也要提供的armeabi的so库，复制一份（armeabi或者armeabi-v7a的so库，因为arm64-v8a兼容armeabi 和 armeabi-v7a）到没有提供的arm64-v8a这个架构目录下；如果不这么做，当应用跑到arm64-v8a架构的手机上时，找不到对应的so库就会报错
具体自己项目适配几种cpu架构，得看app性质，比如微信，主要考虑到兼容，让几乎所有手机都可以适配，另外也相对减少了apk的大小；而另外一个app，比如游戏或者一些对手机性能有要求的app，这种app就挑用户了，只适配到armeabi-v7a，因为目前主流手机都支持armeabi-v7a，就算app支持到只支持armeabi这种架构的手机，app也未必能运行的起来，体验也未必好，算是app放弃也这些用户吧，再说使用只支持armeabi这种架构的手机估计年纪也大了，也不会使用到这个app；
如果只适配一种cpu架构，armeabi（都兼容，但性能有所损耗，如微信和qq）或者armeabi-v7a（目前大部分手机都支持这种cpu架构（王者荣耀））；目前手头的app目前只支持armeabi，armeabi-v7a，但是现在apk包越来越大，后面也会考虑只支持一个cpu架构的方案（可以减少10M）;
如果app适配了armeabi、 armeabi-v7a 、arm64-v8a三种cpu架构，以我的手机mate9为例， mate9支持 armeabi、 armeabi-v7a 、arm64-v8a，那么app在找so文件时会从最新的一代的cpu 架构(arm64-v8a)找so文件,如果找不到会直接报错，不会再去armeabi-v7a 和armeabi里面找，一定要确保三个目录中的so库数目一样；如果适配armeabi、 armeabi-v7a，mate9手机上app在找so文件时会从armeabi-v7a找对应so，没有就报错；如果只适配一种，那么手机只要支持这种cpu架构，就会去这个文件夹下找对应的so，找不到就报错，如果手机不支持这种cpu架构就报错
项目中遇到的一种情况：默认情况下（不在gradle中设置ndk），某个测试机支持v8a，v7a，armeabi三种cpu架构；代码中只有armeabi和v7a两个包并且两个包内的so包完全相同；在测试机上无论debug还是release测试都不会有问题；但是后续开发中集成了某个第三方arr后，打的包就会默认在v8a中找so文件，导致程序崩溃；所以建议开发时候一定要对cpu架构做适配；仅此记录

具体适配的cpu架构，在app>build.gradle文件中android->defaultConfig 里设置的支持的 ndk 为准：
ndk { 
abiFilters 'armeabi','armeabi-v7a' // , 'arm64-v8a','x86', 'x86_64' 
}

如何查看手机支持的cpu架构和app适配的cpu架构:
下载App：Native Libs Monitor（豌豆荚上可以下载）
https://github.com/zhaobozhen/LibChecker

大公司如何适配的
微信（只适配armeabi）、qq（只适配armeabi）、王者荣耀（只适配armeabi-v7a）、百度地图（只适配armeabi）、大众点评（只适配armeabi）等









# GPU

32个图层



## CUDA
https://cn.bing.com/search?q=CUDA&form=QBLH&sp=-1&pq=cuda&sc=10-4&qs=n&sk=&cvid=0064B7E1CF3342AF906DEE5B39A575D3&ghsh=0&ghacc=0&ghpl=








# TPU


# CPU GPU TPU 的区别

https://www.cnblogs.com/Renyi-Fan/p/9720941.html


https://zhuanlan.zhihu.com/p/154058171

https://zhuanlan.zhihu.com/p/137192727


https://zhuanlan.zhihu.com/p/156171120





