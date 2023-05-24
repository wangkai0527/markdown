[TOC]

# Android SDK
Android SDK 中包含了开发应用所需的多个软件包。
[本页列出了可供使用的最重要的命令行工具（按提供这些工具的软件包整理）](https://developer.android.com/studio/command-line)

[使用 SDK 管理器更新工具](https://developer.android.com/studio/intro/update#sdk-manager)


## Android SDK 构建工具

必需。包含构建 Android 应用的工具。

位置：_`android_sdk/build-tools/version`_

[SDK Build Tools 版本说明](https://developer.android.com/studio/releases/build-tools)


## Android SDK 平台工具

必需。包含 Android 平台所需的各种工具，包括 adb 工具。

位置：_`android_sdk/platform-tools/`_

[SDK Platform Tools 版本说明](https://developer.android.com/studio/releases/platform-tools)


## Android SDK 命令行工具

位置：*`android_sdk/cmdline-tools/version/bin/`*

[Android SDK 命令行工具版本说明](https://developer.android.com/studio/releases/cmdline-tools)

***注意：Android SDK 命令行工具软件包（位于 cmdline-tools）取代了 SDK 工具软件包（位于 tools）。***


## Android 模拟器
位置：_`android_sdk/emulator/`_

[模拟器版本说明](https://developer.android.com/studio/releases/emulator)


## Android SDK 工具

包括 ProGuard 等基本工具。

位置：
*`android_sdk/tools`*
*`android_sdk/tools/bin`*

[SDK 工具版本说明](https://developer.android.com/studio/releases/sdk-tools)

***注意：此 SDK 工具软件包已废弃，不会再收到更新。请改用新的 Android SDK 命令行工具软件包***




# 下载方式

## [网站下载](https://androidsdkmanager.azurewebsites.net/SDKPlatform)

## 配置HTTP proxy
File -> Settings -> Appearance & Behavior -> System Settings -> HTTP proxy
选择Manual proxy configuration
设置 Host name 为：mirrors.opencas.cn
设置 Port number 为：80
Android SDK -> SDK Update sites，勾选上Force Https://…，点击Apply

        Android SDK在线更新镜像服务器

        南阳理工学院镜像服务器地址:
        mirror.nyist.edu.cn 端口:80

        中国科学院开源协会镜像站地址:
        IPV4/IPV6: mirrors.opencas.cn 端口:80
        IPV4/IPV6: mirrors.opencas.org 端口:80
        IPV4/IPV6: mirrors.opencas.ac.cn 端口:80

        上海GDG镜像服务器地址:
        sdk.gdgshanghai.com 端口:8000

        北京化工大学镜像服务器地址:
        IPv4: ubuntu.buct.edu.cn/ 端口:80
        IPv4: ubuntu.buct.cn/ 端口:80
        IPv6: ubuntu.buct6.edu.cn/ 端口:80

        大连东软信息学院镜像服务器地址:
        mirrors.neusoft.edu.cn 端口:80

        腾讯Bugly 镜像:
        android-mirror.bugly.qq.com 端口:8080
        [腾讯镜像使用方法](http://android-mirror.bugly.qq.com:8080/include/usage.html)


-----------------------------------------------------------------------
第一次启动 Android Studio，可以关闭HTTP proxy配置
打开 bin\idea.properties 文件，在底部加上一行

        disable.android.first.run=true

重新打开 Android Studio 就可进入主界面。

-----------------------------------------------------------------------

代理停用后，有时候会遇到同步失败的问题，这是因为Android Studio的 gradle会把手工配置的代理信息保存至当前用户目录的`gradle.properties`配置文件中，具体路径为`C:\Users[USER].gradle\gradle.properties`

        systemProp.http.proxyHost=127.0.0.1
        systemProp.http.proxyPort=1080
        systemProp.https.proxyHost=
        systemProp.https.proxyPort=80

把systemProp.*相关信息都删除掉即可。


## 更改hosts文件
1. 取消设置的HTTP Proxy，选择No proxy
2. 进入网站http://ping.chinaz.com/，进行 dl.google.com ping检查，选择大陆响应时间最短的IP地址
3. 进入cmd对此IP地址进行ping测试，如果可以将（IP地址 dl.google.com）加入hosts文件中
hosts文件地址：C:\WINDOWS\System32\drivers\etc\hosts（快捷键查询WINDOW+R 输入drivers）
Linux: /etc/hosts

        127.0.0.1 localhost
        203.208.41.73 dl.google.com

4. 点击Apply、OK，重新打开Android SDK，可以看到列表已经获得



