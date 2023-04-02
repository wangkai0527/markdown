[TOC]

# CMake

你或许听过好几种 Make 工具，例如 GNU Make ，QT 的 qmake ，微软的 MSnmake，BSD Make（pmake），Makepp，等等。
这些 Make 工具遵循着不同的规范和标准，所执行的 Makefile 格式也千差万别。
这样就带来了一个严峻的问题：如果软件想跨平台，必须要保证能够在不同平台编译。而如果使用上面的 Make 工具，就得为每一种标准写一次 Makefile ，这将是一件让人抓狂的工作。

CMake就是针对上面问题所设计的项目构建工具：
它首先允许开发者编写一种平台无关的 CMakeList.txt 文件来定制整个编译流程，
然后再根据目标用户的平台进一步生成所需的本地化 Makefile 和工程文件，如 Unix 的 Makefile 或 Windows 的 Visual Studio 工程。
从而做到“Write once, run everywhere”。显然，CMake 是一个比上述几种 make 更高级的编译配置工具。
一些使用 CMake 作为项目架构系统的知名开源项目有 VTK、ITK、KDE、OpenCV、OSG 等。

CMake是kitware公司以及一些开源开发者在开发几个工具套件(VTK)的过程中所产生的衍生品。后来经过发展，最终形成体系，在2001年成为一个独立的开放源代码项目。

[所有版本下载地址](https://cmake.org/files/)

源码编译安装：
#tar -xvf cmake-3.22.0-rc2.tar.gz        
#cd cmake-3.22.0-rc2
#./bootstrap --prefix=/usr/local/cmake
#make
#make install

Linux:
`sudo apt install cmake`

[官方参考手册](https://cmake.org/cmake/help/v3.12/index.html)

[CMake 入门实战 （比较详细的一篇入门教程）](http://www.hahack.com/codes/cmake/)


优点：
1、开源代码，使用类BSD许可发布。
2、跨平台，并可以生成native编译配置文件，在linux/Unix平台，生成makefile,在苹果平台可以生成Xcode,在windows平台，可以生成MSVC的工程文件。
3、能够管理大型项目。
4、简化编译构建过程和编译过程。cmake的工具链：cmake+make。
5、高效率，因为cmake在工具链中没有libtool。
6、可扩展，可以为cmake编写特定功能的模块，扩展cmake功能。

缺点：
1、文档太差。
2、弱变量及未定义的变量导致非预期行为。
3、调试困难。



## CMakeLists.txt

```
#cmake最低版本需求，不加入此行会受到警告信息
cmake_minimum_required(VERSION 3.12)

project(hello)

set(CMAKE_CXX_STANDARD 11)

find_package(OpenCV REQUIRED)

include_directories(${OpenCV_INCLUDE_DIRS})
set(CMAKE_CXX_STANDARD 11)

add_executable(hello main.cpp)

target_link_libraries(hello ${OpenCV_LIBS})
```

```C++
#include <opencv2/opencv.hpp>
#include <iostream>
#include <string>
using namespace cv;

void ImageThreshold(String str)
{
    Mat image = imread(str);

    imshow("test_opencv", image);
    waitKey(0);
}

int main()
{
    String str = "/Users/xxx/Pictures/me.jpg";
    std::cout << "Hello, world" << std::endl;
    ImageThreshold(str);
    std::cout << str << std::endl;
    return 0;
}
```


## 手动build

1. 外部编译方式
其实cmake可以直接在当前目录进行编译，无须建立build目录。但是，这种做法会将所有生成的中间文件和源代码混在一起，而且cmake生成的makefile无法跟踪所有的中间文件，即无法使用”make distclean”命令将所有的中间文件删除。因此，我们推荐建立build目录进行编译，所有的中间文件都会生成在build目录下，需要删除时直接清空该目录即可。
2. 根据CMakeLists.txt生成makefile
3. 编译工程


Windows 
`mkdir build && cd build`
`cmake .. -G"MinGW Makefiles"`
Debug `cmake --build .`
Release `cmake --build . -- /p:Configuration=Release`

CMakeCache.txt:
//Path to CMake executable.
CMAKE_COMMAND:INTERNAL=E:/baidu/cmake/cmake-3.25.0-rc1-windows-x86_64/bin/cmake.exe

Linux
`mkdir build && cd build`
`cmake ..`
`make -j4`

CMakeCache.txt:
//Path to CMake executable.
CMAKE_COMMAND:INTERNAL=/usr/bin/cmake


## 制定toolchain.cmake
.cmake文件的好处是一次编写多次使用，不同平台架构的交叉编译工具链可以编写一个独立的toolchain.cmake文件，而工程的CMakeLists.txt可以编写为通用格式，对工具链不可见。

`mkdir build && cd build && cmake -DCMAKE_TOOLCHAIN_FILE=../toolchain/ndk32-toolchain.cmake .. && make -j4`


toolchain.cmake:

```
set(CMAKE_SYSTEM_NAME Android)

set(CMAKE_ANDROID_API 21)
set(CMAKE_ANDROID_ARCH_ABI aarch64)
set(CMAKE_ANDROID_STL_TYPE gnustl_static)

set(TOOLCHAIN_PATH /opt/sdk/android-aarch64)
# set(ANDROID_LIB_PATH ${TOOLCHAIN_PATH}/sysroot/usr/lib)

set(CMAKE_C_COMPILER ${TOOLCHAIN_PATH}/bin/aarch64-linux-android-gcc)
set(CMAKE_C_FLAGS "-D__ANDROID_API__=21  -fno-exceptions -O2 -fpie -fpic -fPIE -fPIC -pie -lm -Wl,-llog" CACHE STRING "" FORCE)

set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PATH}/bin/aarch64-linux-android-g++)
set(CMAKE_CXX_FLAGS "-D__ANDROID_API__=21 -DANDROID_STL=gnustl_static -fno-exceptions -O2 -fpie -fpic -fPIE -fPIC -pie -std=c++11 -lm -lstdc++ -Wl,-llog" CACHE STRING "" FORCE)
```


## build error

-----------------------------------------------------------------------
Windows执行命令`cmake .`，报错

    Running
    'nmake' '-?'
    failed with:
    系统找不到指定的文件。

    CMake Error: CMAKE_C_COMPILER not set, after EnableLanguage
    CMake Error: CMAKE_CXX_COMPILER not set, after EnableLanguage

总结一下，此类问题两个可能原因，一个是未正确配置编译器，另外一个是未正确配置生成器（Generators），生成器可以是 MinGW Makefiles，也可以是其他Ninja、CodeBlocks 等等。
问题是因为cmake默认使用windows的nmake程序（本机没有所以提示找不到nmake），因此需要指明cmake要生成mingw make使用的makefile文件。
1. 执行命令`cmake -G"MinGW Makefiles"`，但是又有报错

    CMake Error: CMake was unable to find a build program corresponding to "MinGW Makefiles".  
                CMAKE_MAKE_PROGRAM is not set.  You probably need to select a different build tool.
    CMake Error: CMAKE_C_COMPILER not set, after EnableLanguage
    CMake Error: CMAKE_CXX_COMPILER not set, after EnableLanguage


cmake会在前次执行结果文件的基础上直接执行，但是发现上次用的是NMake，与本次指定的MinGW不一致，所以报错。

2. 将生成的中间文件全部删除，再次执行ok

3. 如果还报错，请先安装MinGW，再设置环境变量

-----------------------------------------------------------------------


## Android工程配置CMake

https://developer.android.google.cn/ndk/guides/cmake?hl=zh-cn

1. 修改顶层local.properties文件中CMake的路径

>cmake.dir=cmake.dir=D\:\\Android\\Sdk\\cmake\\3.18.1

2. app层build.gradle中指定CMake版本

```
android {
    defaultConfig {
        externalNativeBuild {
            cmake {
                abiFilters "arm64-v8a"
            }
        }
    }
    ...
    externalNativeBuild {
        cmake {
            path file('src/main/cpp/CMakeLists.txt')
            version '3.18.1'
        }
    }
}
```



https://www.bookstack.cn/read/CMake-Cookbook/README.md

https://www.zhihu.com/column/c_1490802622991306752

https://juejin.cn/post/6844903557183832078

https://zhuanlan.zhihu.com/p/393316878

https://www.cnblogs.com/juzaizai/category/1993745.html

https://juejin.cn/post/6844903565803126798



## cmake_policy
https://cmake.org/cmake/help/latest/manual/cmake-policies.7.html
https://zhuanlan.zhihu.com/p/489148298






## 软件发布前的库优化与裁剪

https://zhuanlan.zhihu.com/p/72475595

https://zhuanlan.zhihu.com/p/73592450

sudo apt-get update
sudo apt install graphviz

cmake .. --graphviz=./target_deps_graphviz

dot -Tpng -o target.png ./target_deps_graphviz
dot -Tpdf -o target.pdf ./target_deps_graphviz







shell+cmake
https://rangaofei.github.io/2018/02/22/shell%E8%84%9A%E6%9C%AC%E7%94%9F%E6%88%90%E5%AE%89%E5%8D%93%E5%85%A8abi%E5%8A%A8%E6%80%81%E5%BA%93%E4%B8%8E%E9%9D%99%E6%80%81%E5%BA%93/#more




