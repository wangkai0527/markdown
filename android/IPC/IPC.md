[TOC]

# IPC

Inter-Process Communication
进程间通信 ，是指在不同进程之间传播或交换信息。



Linux 进程间通信方式:
管道
共享内存
Socket
File

Android 进程间通信方式:
Binder


### 为什么Android要采用Binder做进程通信

|  | 管道 | 共享内存 | Socket | Binder |
| :---: | :---: | :---: | :---: | :---: |
| 性能 | 两次拷贝 | 无需拷贝 | 两次拷贝 </br>用户和内核的cpu状态切换 | 一次拷贝 |
| 安全性 | 安全 | 依赖上层协议 </br>访问接入点是开放的 </br>不安全 | 依赖上层协议 </br>访问接入点是开放的 </br>不安全 | 为每个APP分配UID </br>同时支持实名和匿名 |
| 特点 | 1v1 | NvN </br>控制复杂，易用性差 | 基于C/S 架构 </br>易用性差 | 基于C/S 架构 </br>易用性高 |





https://www.cnblogs.com/CheeseZH/p/5264465.html












