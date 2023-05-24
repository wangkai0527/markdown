[TOC]

# USB


需要驱动
四根线
支持热插拔，携带方便，协议统一

1.0     12 M/s
2.0     480 M/s
3.0     10 G/s


反向不归零编码
变化 0
不变化 1




https://blog.csdn.net/dxjshr/article/details/80612261

https://blog.csdn.net/kjunchen/article/details/52244617








## FTDI

http://www.ftdichip.cn/Android.htm

https://www.ftdichip.com/index.html

https://www.ftdichip.com/Products/ICs.htm






## FAQ

-----------------------------------------------------------------------
设备管理器，发现【通用串行总线控制器】下有一项带有黄色【！】未知USB设备（设备描述符请求失败）。

解决方法如下：

1.点击Windows键 +R或者（点击系统桌面左下角【开始】，在开始菜单中点击【运行】），在运行对话框中输入：services.msc命令，打开服务窗口；

2.在服务窗口中找到Plug and Play服务，双击Plug and Play，打开Plug and Play的属性窗口：

首先将Plug and Play服务的启动类型为：禁用，停止服务，点击应用，确定，此时启动类型为：禁用，服务状态为：已停止，

然后再点击 Plug and Play服务 设置重新启动，将Plug and Play服务的启动类型为：手动或自动，启动服务，点击应用，确定，此时启动类型为：手动或自动，服务状态为：正在运行；

3.回到服务窗口，可以看到：Plug and Play服务的  启动类型：自动，运行状态：正在运行 ，关 点击窗口左上角的【文件】，在下拉菜单中点击【退出】，退出服务窗口或者直接关掉服务窗口。


4.再插拔一次USB设备，这时进入设备管理器，可以看到：【通用串行总线控制器】下有一项带有黄色【！】未知USB设备（设备描述符请求失败）项消失了或者

【通用串行总线控制器】下带有黄色【！】的Unknown USB Device（Device Descniptor Request Failed）项消失。同时发现原来查不到的设备也有了，

5.选择其他设备下的未知设备，右键 更新驱动，即可以使用了。

-----------------------------------------------------------------------



