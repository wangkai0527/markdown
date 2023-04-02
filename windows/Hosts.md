[TOC]

# Hosts

## Hosts文件的位置
很多用户都知道在Window系统中有个Hosts文件（没有后缀名），在Windows 98系统下该文件在Windows文件夹。
在Windows 2000/XP系统中位于\%Systemroot%\System32\Drivers\Etc 文件夹中，其中，%Systemroot%指系统安装路径。例如，Windows XP 安装在C:\WINDOWS,那么Hosts文件就在C:\WINDOWS\system32\drivers\etc中。
你也可以用windows自带的查找功能搜索找到hosts文件。
该文件其实是一个纯文本的文件，用普通的文本编辑软件如记事本等都能打开和编辑。

## Hosts文件的基本内容和语法
用记事本打开hosts文件，就可以看见了微软对这个文件的说明。Hosts文件文一般有如下面的基本内容

```
# Copyright (c) 1993-1999 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
#
# This file contains the mappings of IP addresses to host names. Each
# entry should be kept on an individual line. The IP address should
# be placed in the first column followed by the corresponding host name.
# The IP address and the host name should be separated by at least one
# space.
#
# Additionally, comments (such as these) may be inserted on individual
# lines or following the machine name denoted by a '#' symbol.
#
# For example:
#
# 102.54.94.97 rhino.acme.com # source server
# 38.25.63.10 x.acme.com # x client host

127.0.0.1 localhost
```

这个文件是根据TCP/IP for Windows 的标准来工作的。它的作用是定义IP地址和
Host name(主机名)的映射关系，是一个映射IP地址和Host name (主机名) 的规定。这个规定中，要求每段只能包括一个映射关系，也就是一个IP地址和一个与之有映射关系的主机名。 IP地址要放在每段的最前面，映射的Host name(主机名)在IP后面，中间用空格分隔。对于这段的映射说明，用“#”分割后用文字说明。

## Hosts文件的工作方式
现在让我们来看看Hosts在Windows中是怎么工作的。
我们知道在网络上访问网站，要首先通过DNS服务器把要访问的网络域名（XXXX.com）解析成XXX.XXX.XXX.XXX的IP地址后，计算机才能对这个网络域名作访问。
要是对于每个域名请求我们都要等待域名服务器解析后返回IP信息，这样访问网络的效率就会降低，因为DNS做域名解析和返回IP都需要时间。
为了提高对经常访问的网络域名的解析效率，可以通过利用Hosts文件中建立域名和IP的映射关系来达到目的。根据Windows系统规定，在进行DNS请求以前，Windows系统会先检查自己的Hosts文件中是否有这个网络域名映射关系。如果有则，调用这个IP地址映射，如果没有，再向已知的DNS服务器提出域名解析。也就是说Hosts的请求级别比DNS高。

## Hosts文件的具体作用
现在来看一下Hosts文件的工作方式以及它在具体使用中起哪些作用。

1. 加快域名解析
对于要经常访问的网站，我们可以通过在Hosts中配置域名和IP的映射关系，提高域名解析速度。由于有了映射关系，当我们输入域名计算机就能很快解析出IP，而不用请求网络上的DNS服务器。

2. 方便局域网用户
在很多单位的局域网中，会有服务器提供给用户使用。但由于局域网中一般很少架设DNS服务器，访问这些服务器时，要输入难记的IP地址。这对不少人来说相当麻烦。现在可以分别给这些服务器取个容易记住的名字，然后在Hosts中建立IP映射，这样以后访问的时候，只要输入这个服务器的名字就行了。

3. 屏蔽网站
现在有很多网站不经过用户同意就将各种各样的插件安装到你的计算机中，其中有些说不定就是木马或病毒。对于这些网站我们可以利用Hosts把该网站的域名映射到错误的IP或本地计算机的IP，这样就不用访问了。在WINDOWSX系统中，约定127.0.0.1为本地计算机的IP地址, 0.0.0.0是错误的IP地址。
如果，我们在Hosts中，写入以下内容：

127.0.0.1 # 要屏蔽的网站 A

0.0.0.0 # 要屏蔽的网站 B

这样，计算机解析域名 A和 B时，就解析到本机IP或错误的IP，达到了屏蔽网站A 和B的目的。

例1.
在 hosts文件中加入如下内容就可以屏蔽文件中定义的对应的网址。
127.0.0.1 localhost
127.0.0.1 download.3721.com
127.0.0.1 3721.com #3721网络实名
127.0.0.1 3721.net #3721网络实名
127.0.0.1 cnsmin.3721.com #3721网络实名
127.0.0.1 cnsmin.3721.net #3721网络实名
127.0.0.1 download.3721.com #3721网络实名
127.0.0.1 download.3721.net #3721网络实名
127.0.0.1 www.3721.com #3721网络实名
127.0.0.1 www.3721.net #3721网络实名

例2.
在 hosts文件中加入如下内容就可以屏蔽文件中定义的对应的网址。
127.0.0.1 localhost
127.0.0.1 bar.baidu.com #百度IE搜索伴侣
127.0.0.1 www.baidu.com #百度IE搜索伴侣
127.0.0.1 baidu.com #百度IE搜索伴侣


4. 顺利连接系统
对于Lotus的服务器和一些数据库服务器，在访问时如果直接输入IP地址那是不能访问的，只能输入服务器名才能访问。那么我们配置好Hosts文件，这样输入服务器名就能顺利连接了。




最后要指出的是，Hosts文件配置的映射是静态的，如果网络上的计算机更改了请及时更新IP地址，否则将不能访问。
 

它的作用: 
是把IP和网址映射起来。访问网站时必须通过DNS服务器把域名解析为IP地址，这样浏览器才能知道连接到哪里才是我们要的网站，如果每个域名请求都要等待域名服务器解析后返回IP地址，就会降低访问网络的效率.为了提高访问效率， 

在Windows的处理逻辑里，它总是先在HOSTS文件里查找这个域名和IP的对应关系， 

如果对应关系存在，Windows就直接连接HOSTS表里描述的IP地址，只有在找不到的时候才向DNS服务器发送解析域名的请求，这个逻辑关系在某些程度上的确方便了用户，因为HOSTS表的优先度比任何一个DNS服务器都高，我们能用它跳过域名解析这一步，访问网站的速度就能提高，也不怕DNS服务器出故障时叫天不应叫地不灵了；局域网用户还能通过HOSTS表用自己设定的域名访问本网段内某台机器提供的网站，而不用记忆复杂的IP； 

鉴于HOSTS表的优先度，还能用它屏蔽恶意站点。 
当有IP在DNS上不能解析时,也直接在hosts表中加入,就可以访问该网站,不然输入域名无反应.







host是一个没有扩展名的系统文件，可以用记事本等工具打开，其作用就是将一些常用的网址域名与其对应的IP地址建立一个关联“数据库”，当用户在浏览器中输入一个需要登录的网址时，系统会首先自动从Hosts文件中寻找对应的IP地址，一旦找到，系统会立即打开对应网页，如果没有找到，则系统再会将网址提交DNS域名解析服务器进行IP地址的解析。现在笔者就向大家介绍该文件的三个特殊妙用。


1. 重新找回“失效”网址
提起这事笔者非常惭愧，前些天登录了几次搜狐的站点，可首页就是打不开，于是便料定搜狐可能由于内部什么调整而将服务器关了，笔者甚至还幸灾乐祸的发短信向朋友们报告自己发现的“惊爆新闻”！
当笔者知道在其他几乎所有的电脑上均能打开传说中的搜狐网站而只有自己打不开时，笔者傻了，难道真的是朋友们说的“人品问题”……
按照解决问题的常规，在运行框中输入“ping www.sohu.com”，发现其返回的IP地址不是搜狐对应的“220.181.26.133”，而是莫名其妙的“127.0.0.1”!至此真相大白，
原来一些网页恶意脚本将笔者的Hosts文件进行了修改，即在Hosts文件中添加了一条“127.0.0.1www.sohu.com ”记录，当笔者在地址栏中输入搜狐的网址时，被系统解析出来的IP地址不是正确的“220.181.26.133”而是“127.0.0.1”，所以自然就打不开了。
解决方法很简单，在c:\windows\system32\drivers\etc文件夹中找到Hosts文件并用记事本打开(Windows 9x/Me系统在C:\Windows文件夹中找)，
将其中的错误记录(如“127.0.0.1www.sohu.com”)或者全部记录删去，保存文件退出，这时再登录搜狐的站点就应该畅通无阻了。
提示：如果用户感觉手工寻找Hosts文件及手工指定记事本工具打开Hosts文件


2. 自动屏蔽网页恶意插件
上网观看免费影视剧是笔者的一大爱好，前段时间经一个大侠朋友推荐，笔者终于又找到了一个很不错的免费在线电影站点，不过在打开每一部电影播放页面前，
站点均会dan出一个要求安装百度工具条的网页并且不等用户同意便立即进入下载安装状态！尽管这个网页窗口可以一关了之，但要看的电影不是一部，每次都要连接下载肯定会影响正常网页的打开速度及正在播放视频的流畅。
通过观察，在各个电影播放页面中dan出的这个百度工具条安装窗口都是同一网址，由于原网址较长，我们用http://www.123.com/aa.exe代替，
下面我们打开系统文件夹中的Hosts文件，在文件中新开启一行，输入“0.0.0.0http://www.123.com/aa.exe”(输入内容没有引号，但IP地址与网址间有空格)，
接下来将文件保存退出，当电影站点试图打开http://www.123.com/aa.exe页面时，系统会自动将其解析到“0.0.0.0”这样一个不可能存在的IP地址上，这样也就屏蔽了该网页插件。
提示：1.用户可以用同样的方法将其他恶意插件、各种网页dan出广告和一些非法网站添加进Hosts文件进行彻底屏蔽。
另外，如果用户需要经常向Hosts文件添加屏蔽网址，则也可以不用每次进入系统目录中寻找Hosts文件：同样将“雅虎助手”切换到“编辑Hosts表”选项卡，单击“添加”按钮，这样便可以批量向Hosts文件添加屏蔽网址了。
2.大家是否经历过这样的怪事：在正常浏览网页或进行其他办公操作的过程中，IE每隔一段时间就会自动dan出整屏的网页广告并且这些网页广告内容还会自动随机变换！
不过网址的形式比较固定：比如http://www.xxx.net、http://www.xxx.net/v/和http://www.xxx.net/new／等，
其实这是一种类似“Win32.Troj.PopWeb”的系列木马病毒，大家也可以按照上面的方法将这些网址解析成“０.０.０.０”，从而摆脱病毒的骚扰。


3. 一键登录局域网指定服务器
单位的“高工”在公司的局域网中建了个CS对战服务器，于是我们这些一人吃饱全家皆饱的单身汉便又有了在下班时间消遣的好去处。
不过有一点美中不足，局域网中没有再架设DNS服务器，所以我们每次只能输入IP地址进行登录，尽管只是数量不算多的一串数字，但毕竟数字枯燥啊！
在这种情况下，我们可以通过修改Hosts文件来达到一键登录局域网CS服务器的目的：打开Hosts文件，
同样在新开启的空白行中输入“221.555.78.122 aa.com”(假定221.555.78.122是CS服务器在局域网中的IP地址)，这样我们以后只要输入“aa.com”就可以直接登录局域网CS服务器了。



在Windows 98系统下该文件在Windows目录，在Windows 2000/XP系统中位于C:\Winnt\System32\Drivers\Etc 目录中。该文件其实是一个纯文本的文件，用普通的文本编辑软件如记事本等都能打开。 
用记事本打开hosts文件，首先看见了微软对这个文件的说明。这个文件是根据TCP/IP for Windows 的标准来工作的，
它的作用是包含IP地址和Host name(主机名)的映射关系，是一个映射IP地址和Host name(主机名)的规定，规定要求每段只能包括一个映射关系，
IP地址要放在每段的最前面，空格后再写上映射的Host name(主机名)。对于这段的映射说明用“#”分割后用文字说明。 






