[TOC]


# SSH


https://info.support.huawei.com/info-finder/encyclopedia/zh/SSH.html







## scp

//使用方法：scp 源文件路径 账户@地址:目的路径
scp C:\Users\zbh\Desktop\1.txt  lucas@192.168.11.150:/home/lucas/

scp lucas@192.168.110.128:/home/lucas/world.txt C:\Users\zbh\Desktop\

从linux系统复制文件到windows系统：scp /oracle/a.txt  administrator@192.168.3.181:/d:/

在linux环境下，将windows下的文件复制到linux系统中：scp administrator@192.168.3.181:/d:/test/config.ips  /oracle

请注意：因为windows系统本身不支持ssh协议，所以，要想上面的命令成功执行，必须在windows客户端安装ssh for windows的客户端软件，比如winsshd，使windows系统支持ssh协议才行。










