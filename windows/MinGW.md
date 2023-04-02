[TOC]

# MinGW

MinGW，是Minimalist GNU for Windows的缩写。它是一个可自由使用和自由发布的Windows特定头文件和使用GNU工具集导入库的集合，允许你在GNU/Linux和Windows平台生成本地的Windows程序而不需要第三方C运行时（C Runtime）库。
MinGW 是一组包含文件和端口库，其功能是允许控制台模式的程序使用微软的标准C运行时（C Runtime）库（MSVCRT.DLL）,该库在所有的 NT OS 上有效，在所有的 Windows 95发行版以上的 Windows OS 有效，使用基本运行时，你可以使用 GCC 写控制台模式的符合美国标准化组织（ANSI）程序，可以使用微软提供的 C 运行时（C Runtime）扩展，与基本运行时相结合，就可以有充分的权利既使用 CRT（C Runtime）又使用 WindowsAPI功能（该段内容来自百度百科）。


[MinGW安装](https://www.w3cschool.cn/c/install-mingw.html)


在命令提示符中输入​`gcc -v`​或者`​g++ -v`​，如果有输出`gcc version 8.1.0`，则证明配置成功。
建议安装新版本，低版本中可能没有g++.exe











