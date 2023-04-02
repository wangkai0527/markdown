[TOC]

# Vistual Studio


[最新版本官方下载地址](https://visualstudio.microsoft.com/zh-hans/downloads/)


|Community  | Professional | Enterprise|
| :---: | :---: | :---: |
|社区版 | 专业版 | 企业版|
|免费 | 收费 | 收费|



[Vistual Studio 2015 Community下载地址](http://download.microsoft.com/download/B/4/8/B4870509-05CB-447C-878F-2F80E4CB464C/vs2015.com_chs.iso)


专业版批量激活秘钥：HMGNV-WCYXV-X7G9W-YCX63-B98R2


## Windows Kits


mklink /J "C:\Program Files (x86)\Windows Kits" "D:\Windows Kits"
或者修改注册表：
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Microsoft SDKs\Windows\v10.0


## FAQ

-----------------------------------------------------------------------
如果Visual Studio安装路径是C盘，此时卸载重装后可能无法重新更改共享组件、工具和SDK的路径。
因为已经写入到注册表中，需要删除注册表中历史路径才可。

1. WIN + R, 输入regedit
2. 计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\VisualStudio\Setup
3. 右键 SharedInstallationPath 和 CachePath删除(D)

-----------------------------------------------------------------------
Visual Studio Installer找不到已安装版本。
如果要更新、修改或卸载产品，必须要下载缓存，缓存默认存储路径：
`C:\ProgramData\Microsoft\VisualStudio\Packages`
缓存文件建议保存到安装目录的同级目录

-----------------------------------------------------------------------
默认的HelloWorld程序报错，无法打开源文件。

工具 --> 获取工具和功能 --> 打开Visual Studio Installer，选择安装SDK版本10.0.20348.0
项目 --> 属性 --> 常规 --> Windows SDK版本 --> 设置对应的版本

-----------------------------------------------------------------------
C4996：'xx':Use xx or xx instead or define _WINSOCK_DEPRECATED_NO_WARINNGS to disable deprecated API warings

在VS2013版本以上开始支持新API，使用旧API会报错。推荐使用新API。
也可以在项目 --> 属性 --> C/C++ --> 常规 --> SDL检查 --> 否


-----------------------------------------------------------------------





