[TOC]


# Library





有时候我们想查看一个exe引用了哪些动态库，或者我们想看某个动态库包含哪些接口函数，这个时候可以使用dumpbin.exe工具：

+ 输入Dumpbin -imports calldll.exe查看它的输入信息，可以看到它加载了***.dll
+ 输入dumpbin –exports dlltest.dll,列出导出函数

/DEPENDENTS: 
 查看依赖项；    如: dumpbin /dependents vlc.exe

ps:
1. 如果有Image has the following delay load dependencies，列出的为 运行时动态加载的dll。
2. 如果有Image has the following dependencies，列出的为载入程序时加载的dll。


如果提示'dumpbin' 不是内部或外部命令，也不是可运行的程序或批处理文件。
解决方法：
1. Everything找到dumpbin.exe
2. Everything找到vcvars32.bat，只要将vcvars32.bat拖放至cmd，按回车。然后再运行dumpbin -exports XXX.dll即可。
3. 开始->所有程序->Microsoft Visual Studio 2010->Visual Studio Tools ->“Visual Studio 命令提示(2010)”后，就像普通的cmd一样的命令行环境，就可以正常使用VS的一些工具，其中就包括dumpbin。

 


进程查看器（ProcessExplorer）可以用来查看进程（实时运行）依赖的dll文件；

DependencyWalker可以用来查看dll或exe依赖的dll文件。
官网： http://www.dependencywalker.com/




https://zhuanlan.zhihu.com/p/57279354

## 动态加载

Windows下搜索路径次序
1. 可执行文件所在的目录（当前目录）；
2. Windows的系统目录（该目录可以通过GetSystemDirectory函数获得）；
3. 16位的系统目录（Windows目录下的system子目录）；
4. Windows目录（该目录可以通过GetWindowsDirectory函数获得）；
5. 进程当前所在的工作目录；
6. PATH环境变量中所列出的目录。
注：工作目录位于Windows目录之后，这一改变开始于Windows XP SP2之后。




