[TOC]

# WSL

[安装WSL](https://learn.microsoft.com/zh-cn/windows/wsl/install)

https://blog.csdn.net/sexyluna/article/details/106432454

https://zhuanlan.zhihu.com/p/224753478

必须运行 Windows 10 版本 2004 及更高版本（内部版本 19041 及更高版本）或 Windows 11 才能使用以下命令。
通过按 Windows 徽标键 + R， 检查你的 Windows 版本，然后键入winver，选择“确定” 。

`wsl --install`

`wsl -l -v`

wsl是系统中单纯的按linux的方法操作并不能修改hostname主机名。
即使修改/etc/hostname配置文件也一样无法修改主机名，正确的作法是修改windows计算机的hostname主机名。


## 文件资源管理器
Windows访问WSL2子系统的文件夹
在文件资源管理器输入\\wsl$

其他软件访问，映射网络驱动器Z:到\\wsl$\Ubuntu


## WSL + VSCode

vscode打开wsl目录的几种方法：
1. ctrl + shift + p --> >WSL: Open Folder in WSL...
2. ctrl + shift + p --> >WSL: New WSL Window --> Remote Explorer
这两种方法cmake插件没生效
3. 映射网络驱动器Z:到\\wsl$\Ubuntu，打开Z:\home\wk\Tengine



## WSL + docker

WSL2中的软件配置开机自启，例如启动 Docker

    alias sds="sudo service docker start"


https://blog.xhyeax.com/2020/04/15/wsl2-docker/

https://blog.csdn.net/XhyEax/article/details/105560143


## FluentTerminal

比Windows Terminal更好用的win10终端

安装:
1. 下载安装包：https://github.com/felixse/FluentTerminal/releases/download/0.6.1.0/FluentTerminal.Package_0.6.1.0_Test.zip
2. 在下完zip压缩包后，解压
3. 打开程序文件夹，右键点击install.ps1 ,选择在power shell 运行。之后按照提示操作即可。

注：FluentTerminal现在已经可以直接在win10 应用商店安装(没有)。


## 端口

    powershell
    wsl -- ifconfig eth0

    netsh interface portproxy add v4tov4 listenport=[win10端口] listenaddress=0.0.0.0 connectport=[虚拟机的端口] connectaddress=[虚拟机的ip]
    netsh interface portproxy add v4tov4 listenport=80 listenaddress=0.0.0.0 connectport=80 connectaddress=172.29.41.233

    netsh interface portproxy show all


[Windows 10 配置端口转发到WSL2](https://cloud.tencent.com/developer/article/1913920)

[如何在局域网的其他主机上中访问本机的WSL2](https://zhuanlan.zhihu.com/p/425312804)




## FAQ

-----------------------------------------------------------------------
vs code server for wsl closed unexpectedly

1. 使用管理员权限打开powershell，执行`netsh winsock reset`，重置wsl和主机间的网络协议并重启电脑 ，但不成功。
2. 找到vscode和wsl之间的配置文件`find / -name '.vscode-server' -type d`
3. 删除配置文件`rm -rf .vscode-server`
4. 还是不成功，发现终端日志`Download xxx failed`, 打开Internet连接，windows重启vscode(linux执行`code .`)，成功

-----------------------------------------------------------------------
最近在使用的 wsl2 的时候突然发现 wsl2 无法正常联网，即 ping 不通外网以及宿主机的 wsl 网卡。但是将 wsl 版本设置为 1 就可以联网了。

如果你是正常使用的时候，并且自己没有手动修改过 主机 和 WSL2 的网络配置，然后就忽然发现 WSL2 不能正常访问网络了，这个时候你重启一下 WSL2 大概率就可以工作了。

    # 重启 WSL 指令 
    $ wsl --shutdown 
    # 之后就重新启动进入即可 
    $ wsl 

如果还不行就参考下面的过程一个个的排查吧。
[WSL2 网络异常排查 [ping 不通、网络地址异常、缺少默认路由、被宿主机防火墙拦截]](https://www.jianshu.com/p/ba2cf239ebe0)

-----------------------------------------------------------------------










