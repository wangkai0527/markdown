[TOC]

# FTP

文件传送协议FTP是互联网上使用得最广泛的文件传送协议。

FTP屏蔽了各计算机系统的细节，因而适合于在异构网络中任意计算机之间传送文件。

FTP：基于TCP

TFTP:简单文件传送协议，基于UDP

 

文件传送协议FTP只提供文件传送的一些基本的服务，它使用TCP可靠的运输服务。FTP的主要功能树减少或消除在不同操作系统下

处理文件的不兼容性。

FTP使用客户服务器方式。一个FTP服务器进程可同时为多个客户进程提供服务。FTP的服务器进程由两大部分组成：一个主进程，

负责接收新的请求；另外有若干个从属进程，负责处理单个请求。

FTP的客户和服务器之间要建立两个并行的TCP连接：“控制连接”和“数据连接”。

 

当客户进程向服务器进程发出建立连接请求时，要寻找连接服务器进程的熟知端口21，同时还要告诉服务器进程自己的另一个端口号码，用于

建立数据传送连接。接着，服务器进程用自己传送数据的熟知端口20与客户进程所提供的端口号建立数据传送连接。由于FTP使用了两个

不同的端口号，所以数据连接与控制连接不会发生混乱。

TFTP熟知端口号69。




问题： 项目中有个功能模块是需要用到FTP协议传输文件，但发现传输成功的文件比原文件大的情况。

首先我先修改了传输的代码，部分源代码如下：

```java
    byte[] buffer = new byte[1024];           
        while (inputStream.read(buffer) != -1) {
            out.write(buffer);
            ......

```

修改后的代码，部分源代码如下：

```java
    byte[] buffer = new byte[1024];    
    int numberRead = 0;       
        while ((numberRead = inputStream.read(buffer)) != -1) {
            out.write(buffer,0,numberRead);
            out.flush();
            ......

```


经过以上代码的修改后，其他协议的传输文件大小不一致的问题得到了解决，但是FTP协议传输仍然存在问题，查了很久发现是编码的原因，问题出在FTP传输模式

FTP的传输方式
FTP的传输有两种方式： ASCII传输模式和二进制数据传输模式。

**1．ASCII传输方式：**假定用户正在拷贝的文件包含的简单ASCII码文本，如果在远程机器上运行的不是UNIX，当文件传输时ftp通常会自动地调整文件的内容以便于把文件解释成另外那台计算机存储文本文件的格式。

但是常常有这样的情况，用户正在传输的文件包含的不是文本文件，它们可能是程序，数据库，字处理文件或者压缩文件（尽管字处理文件包含的大部分是文本，其中也包含有指示页尺寸，字库等信息的非打印字符）。在拷贝任何非文本文件之前，用binary 命令告诉ftp逐字拷贝，不要对这些文件进行处理，这也是下面要讲的二进制传输。

2．二进制传输模式： 在二进制传输中，保存文件的位序，以便原始和拷贝的是逐位一一对应的。即使目的地机器上包含位序列的文件是没意义的。例如，macintosh以二进制方式传送可执行文件到Windows系统，在对方系统上，此文件不能执行。

如果你在ASCII方式下传输二进制文件，即使不需要也仍会转译。这会使传输稍微变慢 ，也会损坏数据，使文件变得不能用。（在大多数计算机上，ASCII方式一般假设每一字符的第一有效位无意义，因为ASCII字符组合不使用它。如果你传输二进制文件，所有的位都是重要的。）

创建FTP连接和登录时需要加以下代码：

```java
    FTPClient ftpClient = new FTPClient();
    //连接ftp服务器
    ftpClient.connect(host, Integer.parseInt(port));
    ftpClient.setControlEncoding("gbk");

    ftpClient.setConnectTimeout(0);
    // set timeout to 5 minutes
    ftpClient.setControlKeepAliveTimeout(300);
    // 设置以字节流传输模式
    ftpClient.setFileTransferMode(FTP.STREAM_TRANSFER_MODE);
    // 设置为被动模式登陆
    ftpClient.enterLocalPassiveMode();
    ftpClient.setAutodetectUTF8(true);
    //登录ftp服务器
    ftpClient.login(username, password);
    ftpClient.setFileType(FTP.BINARY_FILE_TYPE);
    ......
```



