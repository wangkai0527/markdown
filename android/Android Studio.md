[TOC]

# Android Studio

## 安装
1. [官网下载Android Studio安装包](https://developer.android.google.cn/studio/)。
2. 点击进行安装，选择安装路径。
3. 指定SDK的本地路径。如果不存在，开始自动下载SDK。
4. 进入AS的欢迎界面
5. 新建一个工程。如果卡在该界面，是因为在从国外站点下载Gradle构建工具，点击取消，采用手动配置Gradle。
6. Build成功。


## 升级
### 检测并更新最新版本
1. `Help` > `Check for Updates`...  
2. `File` > `Settings` > `System Settings` > `Updates` > `选择版本类型`：如Stable Channel > `Check Now`

        版本类型说明：
        1、Canary Channel：金丝雀版（包含最新功能，但存在较多bug）
        2、Dev Channel：开发者版（已解决大部分Bug）
        3、Beta Channel：测试版（部分bug已解决）
        4、Stable Channel：正式稳定版

3. 点击`Check Now`后开始检测版本更新，几秒后弹出弹窗
4. 如果是小版本更新，直接点击`Update and Restart`，不需要重新下载安装包。
如果是大版本更新，点击`Download`，会跳转到浏览器，这里可能需要翻墙
5. 下载最新版本后更新安装即可，可以选择使用之前版本的配置

### 更新到指定版本
1. 查看官网历史版本：[Android Studio 下载文件归档  |  Android 开发者  |  Android Developers](https://developer.android.google.cn/studio/archive)
2. 显示所有Android Studio 版本的归档文件
3. 点击自己想要安装的版本展开
4. 下载自己系统对应的程序或者安装包
5. 下载好后点击安装
6. 可以选择卸载旧的版本，建议不卸载，以后不需要了再卸载


#### 命令行升级

1. 检查自己当前的版本号
`Help` > `Check for Updates`...  

        Build  #AI-162.3573574


2. 查询目前Android Studio的最新版本号是多少
访问 <https://dl.google.com/android/studio/patches/updates.xml> 查看最新的版本号

3. 下载更新包

更新包下载地址为：
`https://dl.google.com/android/studio/patches/AI-162.3573574-171.3829324-patch-win.jar`

请根据自己的Android Studio的build number下载相应的更新包，格式为AI-\$FROM-\$TO-patch-win.jar，其中\$FROM为你当前android studio的build number，\$TO为最新的android studio 的build number.

4. 命令升级版本

将下载的更新包拷贝到任何一个目录下，最好不要是AS的安装目录。否则更新会出现问题，不会成功。

然后打开命令行提示符，进入AS的安装目录，键入如下命令：

        java -classpath D:\svn\AI-162.3573574-162.3742087-patch-win.jar com.intellij.updater.Runner install .

（记住最后面有个点，意思是将这个文件解压安装到当前目录）

安装完毕后，你可以重新启动Android Studio，然后`Help` > `About`查看是不是更新了！


出现的问题：

1. 更新完了,启动的时候卡主了,进不去了。
解决方案：
a.进入安装的Android Studio目录下的bin目录。找到idea.properties文件，随便用一个编辑器打开。
b.在idea.properties文件末尾另起一行添加： disable.android.first.run=true ，然后保存文件。
c.重启Android Studio，这样就可以进入界面。
然后进来了,版本也更新了。

2. 无法下载增量包，404错误：因为版本跨度太大，需要分多段下载，具体可参考<https://dl.google.com/android/studio/patches/updates.xml> 中from标签所指示的版本。

3. 下载后无法解压，提示被JAVA锁定：原因，JAR文件放置位置错误，要放置与Android Studio同一目录下。

4. ADB connection error: windows API的WaitForMultipleObjects所支持的最大句柄数是MAXIMUM_WAIT_OBJECTS, 即64，ddms调用adb时当同时运行进程数大于64则会出错
解决方法：可尝试DDMS的DEVICES窗口中reset ADB，若问题依旧可网上下载修改过的adb.exe替换。



## 快捷键
File-->Settings...-->Keymap-->Editor Actions
例如：
                                                Alt+Shift+↑/↓           上下移动本行
                                                Ctrl+F4                 关闭当前窗口（close tab）
                                                Ctrl+B(Ctrl+左键)       跳转到定义
                                                Ctrl+F(Alt+F7)          查找 (Find对话框可以只显示变量赋值和读取)
                                                Ctrl+R                  替换
Duplicate Line or Selection                     Ctrl+D                  复制本行到下一行
Extend Selection                                Ctrl+W                  选中变量
Type Hierarchy                                  Ctrl+H                  继承关系
Split Lines                                     Ctrl+回车               在当前行下面插入一空行，光标停留在本行
Start New line                                  Shift+回车              在当前行下面插入一空行，光标停留在下一行
Toggle Case                                     Ctrl+Shift+U            大小写转换

https://blog.csdn.net/qq_15807167/article/details/51578339

https://www.cnblogs.com/Seachal/p/5591600.html

https://blog.csdn.net/u012917700/article/details/52437763

https://blog.csdn.net/h_025/article/details/72674653

https://www.cnblogs.com/xmyjcs/p/10423201.html#:~:text=31.Ctrl%2BAlt%2BT%EF%BC%9A%E9%80%89%E4%B8%AD%E4%B8%80%E5%9D%97%E4%BB%A3%E7%A0%81%EF%BC%8C%E6%8C%89%E6%AD%A4%E7%BB%84%E5%90%88%E9%94%AE%EF%BC%8C%E5%8F%AF%E5%BF%AB%E9%80%9F%E6%B7%BB%E5%8A%A0if%20%E3%80%81for%E3%80%81try%2Fcatch%E7%AD%89%E8%AF%AD%E5%8F%A5%E3%80%82,32.Ctrl%2BTab%EF%BC%9A%E6%89%93%E5%BC%80%E7%95%8C%E9%9D%A2%E5%88%87%E6%8D%A2%E7%AA%97%E5%8F%A3%EF%BC%8C%E4%BF%9D%E6%8C%81%E6%8C%89%E4%BD%8FCtrl%E9%94%AE%EF%BC%8C%E9%80%89%E4%B8%AD%E7%9B%B8%E5%BA%94%E7%9A%84%E8%A6%81%E6%89%93%E5%BC%80%E7%9A%84%E7%AA%97%E5%8F%A3%E3%80%82%2033.Ctrl%2BW%EF%BC%9A%E9%80%89%E4%B8%AD%E5%85%89%E6%A0%87%E6%89%80%E5%9C%A8%E7%9A%84%E6%89%80%E5%9C%A8%E7%9A%84%E5%8D%95%E8%AF%8D%EF%BC%88%E4%B8%80%E4%B8%AA%E6%88%90%E5%91%98%E5%8F%98%E9%87%8F%E6%88%96%E8%80%85%E6%98%AF%E4%B8%80%E4%B8%AA%E6%96%B9%E6%B3%95%E5%90%8D%EF%BC%89%EF%BC%8C%E5%A4%9A%E6%8C%89%E4%B8%80%E6%AC%A1%E4%BC%9A%E9%80%89%E4%B8%AD%E6%89%80%E5%9C%A8%E7%9A%84%E8%AF%AD%E5%8F%A5%EF%BC%8C





## 工具

### Device Manager
+ Virtual 虚拟机

+ Physical 真机
    - View Details 关于手机
    - Device File Explorer 文件管理器

### Profile

自带抓包功能。默认是关闭的，需要手动打开：进入`Run` > `Edit Configurations` > `Profiling`。

需要注意的是，你项目中的API版本得是API26以下，而且你的手机版本得是Android5.0以上才能使用抓包功能。

在手机上发起一次网络请求，NETWORK那一栏会发生明显的变化，我们选择那个区域进行抓包，可以看到请求了一个接口MainServlet（如果该区域下会请求多个接口，则会一一列出来），然后我们点击MainServlet，就会出现后台传过来的Json，Header之类的信息。
CPU和MEMORY也一样，都具有记录当前页面的数据，你也可以根据它所记录的数据进行相应的分析。
最后需要注意的是开启Profile之后会降低应用程序的构建速度，因此只有在你要开始对应用程序进行概要分析时，再启用它。


### Lambda
Android Studio3.0后支持Java8，可以直接将Java代码格式成lambda格式，但是你得给你的项目设置成支持Java8，右键你的module，选择`open Module Settings`，把`Source Compatibility`和`Target Compatibility`都设置成1.8。

会自动提醒哪些代码可以转换成lambda表达式。


### Debug

https://developer.android.google.cn/studio/debug?hl=zh-cn#java




## 代码跳转

https://blog.csdn.net/Wbl752134268/article/details/117357177









-----------------------------------------------------------------------
证书不可信任，老弹出 `Server's certificate is not trusted`

解决方法：
点击android studio左上角的 `File` > `Settings` > `Tools` > `Server Certificates` > `Accept non-trusted certificates automatically` -> `OK`

意思为自动接受不可信的证书，将不再弹窗提醒。

-----------------------------------------------------------------------






