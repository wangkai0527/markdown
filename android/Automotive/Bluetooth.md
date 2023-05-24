[TOC]

# Bluetooth

[蓝牙](https://developer.android.google.cn/guide/topics/connectivity/bluetooth?hl=zh-cn)

蓝牙技术提供了短距离无线功能，两个设备可以相互进行数据传输。多年来，蓝牙历经了多个版本，包括蓝牙v1.0、v1.1、1.2、v2.0+EDR、v2.1+EDR、v3.0+HS、v4.0、v4.1、v4.32、v5.0、v5.1和v5.2。它们都工作在2.4GHz ISM频段，技术标准都是以IEEE 802.15.1标准为基础，在这个无线个人区域网（WPAN）标准中，我们定义了蓝牙的PHY和MAC层。

各个蓝牙版本都有不同的速度和数据速率要求，但是它们保持与以前的版本的兼容性，这样可以确保不同版本的设备可以相互通讯。

版本	发布时间	最大传输速度	传输距离	其他
V1.0	1998	723.1Kb/s	10米	BR采用GFSK编码
V1.1	2002	810Kb/s	10米	开始支持Stereo传输
V1.2	2003	1Mb/s	10米	增加抗干扰跳频功能
V2.0	2004	2.1Mb/s	10米	EDR采用DQPSK和8DPSK编码
V2.1	2007	3Mb/s	10米	增加Sniff省电功能
V3.0	2009	24Mb/s	10米	集成802.11 PAL
V4.0	2010	24Mb/s	50米	增加低功耗模式
V4.1	2013	24Mb/s	50米	增加802.11n PAL
V4.2	2014	24Mb/s	50米	增加LE Data Packet Length Extensin，扩展链路层PDU长度
V5.0	2016	48Mb/s	300米	2Ms/s PHY，功率可达20dBm，灵敏度达-82dBm，8位前向纠错编码
V5.1	2019	48Mb/s	300米	提高了定位精度，配对和通讯速度
V5.2	2020	48Mb/s	300米	提出了LE Audio


Bluetooth 比 Wifi 通信速率低，但更稳定。
Bluetooth 属于设备驱动层。


```sequence
车机->手机: 蓝牙芯片扫描10m以内波
Note right of 手机: 蓝牙链接
Note left of 车机: 蓝牙链接
手机-->车机: 返回蓝牙设备信息
车机->手机: 链接指定的蓝牙设备
手机-->车机: 链接成功
手机-->车机: 我需要密码
车机->手机: 密码
手机-->车机: 密码验证成功
车机->手机: 数据通信
```



## 传统蓝牙

1.1
1.2
4.2

BluetoothManager
蓝牙管理服务
管理蓝牙驱动

BluetoothAdapter
蓝牙适配器
监听，开启蓝牙

BluetoothLeScanner
扫描

ScanCallback
回调扫描结果

BluetoothDevice
蓝牙设备

BluetoothGatt
通信




## 蓝牙BLE

4.0
4.1

低功耗，传输速率快，覆盖范围更广，安全性更高







https://blog.csdn.net/qq_32115439/article/details/80379262


