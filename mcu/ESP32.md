[TOC]

# ESP32

[乐鑫科技](https://www.espressif.com.cn/zh-hans)

[ESP32-S2-Pico](https://www.waveshare.net/wiki/ESP32-S2-Pico#.E8.AF.B4.E6.98.8E)



# Arduino

[Arduino官网](https://www.arduino.cc/reference/en/)

[太极创客_Arduino 教程及下载](http://www.taichi-maker.com/homepage/arduino-basic-tutorial-index/#index)

官方网站服务器不在中国，下载很慢，建议使用以上资源下载



### Arduino IDE安装ESP32 SDK
1. 先安装好ArduinoIDE

2. 打开Arduino IDE菜单 > 文件 >首选项，在 附加开发板管理器网址 输入框中，填入以下网址：
https://www.arduino.me/package_esp32_index.json

3. 下载社区打包的esp32安装包，直接运行，程序会自动解压到相应位置。

[下载地址1 阿里云盘](https://www.aliyundrive.com/s/hf8f7JScwD2)

[下载地址2 社区成员 鱼小黑 提供](https://cloud.codess-nas.top:5213/s/2Ocn)

如果安装过其他版本的esp32 package，请先删除，再使用本安装包，删除方法：文件管理器地址栏输入 %LOCALAPPDATA%/Arduino15/packages，回车进入，然后删除掉其中的esp32文件夹

4. 解压完成后，再打开Arduino IDE，即可在 菜单栏>工具>开发板 中找到你使用的esp32开发板




注意开启USB CDC和选择USB update模式Internal USB



## FAQ

-----------------------------------------------------------------------
端口呈现灰色，无法选择端口

1. 先断电，按住BOOT，再上电
2. 电脑上选择换一个端口进行插入，看看会不会有端口出来。
3. 端口接触不良，拔出来再重新插入即可。

-----------------------------------------------------------------------
上传出现ERROR:
To suppress this error, set --after option to 'no_reset'.

提示下载完后需要手动复位才能运行程序

-----------------------------------------------------------------------


[Arduino专栏（创客）](https://blog.csdn.net/weixin_44940488/category_11486890.html)

[Arduino从入门到放肆](https://blog.csdn.net/shuige2215/category_10469198.html)





# VSCode+platformIO

[参考1](https://blog.csdn.net/qq_45516773/article/details/115406420)
[参考2](https://blog.csdn.net/xingxingdiandeng69/article/details/123523601?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-123523601-blog-109233133.pc_relevant_3mothn_strategy_and_data_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-4-123523601-blog-109233133.pc_relevant_3mothn_strategy_and_data_recovery&utm_relevant_index=5)
[参考3](https://blog.csdn.net/happyjoey217/article/details/113177118)
[参考4](https://zhuanlan.zhihu.com/p/509527710)



-----------------------------------------------------------------------
In function setup : error :Serial undeclared

`main.c` rename `main.cpp`
-----------------------------------------------------------------------

-----------------------------------------------------------------------
Serial Monitor串口一直重复打印以下LOG:
rst:0x10 (RTC_WDT_RTC_RESET),boot:0x13(SPI_FAST_FLASH_BOOT)
csum err:0xa9!=0x03
flash read err, 1000
ets_main.c 371

GPIO脚配置不正确，需要使用没有被占用的pin脚
-----------------------------------------------------------------------



# Mircopython
[参考1](https://blog.csdn.net/qq_45516773/article/details/116589749?spm=1001.2014.3001.5502)




