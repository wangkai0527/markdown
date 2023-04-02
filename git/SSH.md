[TOC]

# SSH

[SSH工作机制](https://www.cnblogs.com/ftl1012/p/ssh.html)

[SSH基本用法](https://zhuanlan.zhihu.com/p/21999778)

[SSH简介及两种远程登录的方法](https://blog.csdn.net/li528405176/article/details/82810342)


## ssh-keygen

ssh-keygen命令 用于为“ssh”生成、管理和转换认证密钥，它支持RSA和DSA两种认证密钥。

ssh秘钥登录特点：1.安全；2.免输密码。
对于安全级别较高的服务器，建议配好ssh登录后禁掉密码登录。
缺点：略繁琐。如果你的只是临时登录一次，那么还是密码吧。

ssh 公钥认证是ssh认证的方式之一。通过公钥认证可实现ssh免密码登陆, git的ssh方式也是通过公钥进行认证的。
在用户目录的home目录下, 有一个.ssh的目录, 和当前用户ssh配置认证相关的文件, 几乎都在这个目录下。

### 生成密钥

SSH 密钥默认保留在 ~/.ssh 目录中。 如果没有 ~/.ssh 目录，ssh-keygen命令会使用正确的权限创建一个。

使用 ssh-keygen 时，请先进入到 ~/.ssh 目录，不存在的话，请先创建。并且保证 ~/.ssh 以及所有父目录的权限不能大于 711


使用 ssh-kengen 会在~/.ssh/目录下生成两个文件，不指定文件名和密钥类型的时候，默认生成的两个文件是：
id_rsa
id_rsa.pub 
第一个是私钥文件，第二个是公钥文件。


`ssh-keygen -t rsa -C "xxxxx" -f "id_rsa_xxx"`
|命令选项	|   |
| :---  | :---  |
|-b	    |采用长度1024bit的密钥对,b=bits,最长4096，不过没啥必要|
|-C	    |公钥文件中的备注，C=comment|
|-e	    |读取openssh的私钥或者公钥文件|
|-f	    |指定用来保存密钥的文件名,f=output_keyfiles|
|-i 	|读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥|
|-I 	|显示公钥文件的指纹数据|
|-N 	|提供一个新密语|
|-P 	|提供（旧）密语|
|-q 	|静默模式|
|-t 	|采用rsa加密方式,t=type|
更多参数可运行 man ssh-keygen


为了让私钥文件和公钥文件能够在认证中起作用，请确保权限正确。
对于.ssh 以及父文件夹，当前用户一定要有执行权限，其他用户最多只能有执行权限。
对于公钥和私钥文件也是: 当前用户一定要有执行权限，其他用户最多只能有执行权限。

### 在服务器上安装秘钥

把上一步生成的公钥发送到服务器(scp,FillZilla等)上，然后在服务器上执行下面命令:
`cat id_rsa.pub >> ~/.ssh/authorized_keys`

如此便完成了公钥安装，有个小坑值得一提：authenrized_keys的权限必须是600或更小，否则会连接失败。
保险起见，执行下面命令
`chmod 600 ~/.ssh/authorized_keys`
`chmod 700 ~/.ssh`

另外，.ssh目录的owner必须是ssh登录用户，不能是root

服务器ssh配置
修改服务器上的ssh配置文件，位置：/etc/ssh/sshd_config

    RSAAuthentication yes
    PubkeyAuthentication yes

    PermitRootLogin no //禁止root登录
    PasswordAuthentication yes //允许密码登录，根据你的情况设置

然后重启ssh服务

`service sshd restart`


### 连接服务器
方法1： 直接ssh

`ssh -i ~/.ssh/id_rsa -p 22 user@yourservername`

方法2（推荐）：修改~/.ssh/config

    Host server_alias(你的服务器别名)
    HostName test.com/192.168.1.1(域名或IP)
    Port 22
    User user
    IdentityFile id_rsa

保存后，登录时只需执行

`ssh server_alias`

多个服务器另起一行续写就行了，就是这么简单！




## ssh-agent

用于管理 ssh private keys, 目的是对解密的私钥进行高速缓存。
ssh-add 提示并将用户使用的私钥添加到 ssh-agent 维护列表中，此后当公钥连接到远程 SSH 或 SCP 主机时，不再提示信息。

注:如果在使用shh-add的时候提示:
>Could not open a connection to your authentication agent.

则需手动开启ssh:
方式1：
    
    `ssh-agent bash`

方式2：

    eval `ssh-agent -s`

再次执行`ssh-add`即可


## ssh-add

ssh-add命令是把专用密钥添加到ssh-agent的高速缓存中。该命令位置在/usr/bin/ssh-add。

语法:
ssh-add [-cDdLlXx] [-t life] [file …]
ssh-add -s pkcs11

选项
-D:删除ssh-agent中的所有密钥.
-d:从ssh-agent中的删除密钥
-e pkcs11:删除PKCS#11共享库pkcs1提供的钥匙。
-s pkcs11:添加PKCS#11共享库pkcs1提供的钥匙。
-L:显示ssh-agent中的公钥
-l:显示ssh-agent中的密钥
-t life:对加载的密钥设置超时时间, 超时ssh-agent将自动卸载密钥
-X:对ssh-agent进行解锁
-x:对ssh-agent进行加锁

1、把专用密钥添加到 ssh-agent 的高速缓存中:
`ssh-add ~/.ssh/id_dsa`

2、从ssh-agent中删除密钥:
`ssh-add -d ~/.ssh/id_xxx.pub`

3、查看ssh-agent中的密钥:
`ssh-add -l`







