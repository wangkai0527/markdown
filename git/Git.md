[TOC]

# Git
Git 是一个开源的分布式版本控制系统，用于敏捷高效地处理任何或小或大的项目。
Git 是 Linus Torvalds 为了帮助管理 Linux 内核开发而开发的一个开放源码的版本控制软件。
Git 与常用的版本控制工具 CVS, Subversion 等不同，它采用了分布式版本库的方式，不必服务器端软件支持。

[Git - Book](https://git-scm.com/book/en/v2)
[Pro Git](https://gitee.com/progit/)
[Pro Git第二版](https://bingohuang.gitbooks.io/progit2/content/)
[45个 GIT 经典操作场景](https://blog.csdn.net/xinzhifu1/article/details/123271097)


## 基于 Git 的代码托管和研发协作平台
Gitee和GitHub都提供了基于SSH协议的Git服务，在使用SSH协议访问仓库之前，需要先配置好账户/仓库的SSH公钥。
[Gitee 帮助中心](https://gitee.com/help/articles/4181#article-header0)
[GitHub Docs](https://docs.github.com/cn/authentication/connecting-to-github-with-ssh)

[Gitee 最有价值开源项目](https://gitee.com/gvp)
[Gitee 上的优秀开源项目](https://gitee.com/explore)


### WSL配置ssh key
1. 切换到ssh配置目录
`cd ~/.ssh`

2. 生成秘钥对
可以使用下面的语句生成自己想要的秘钥对，其中，xxxxx可以不是自己的邮箱名，gitee中会把此部分识别为公钥标题，便于自己记忆即可；id_rsa_xxx代表生成的公钥对的私钥文件名，也是随意设置，便于自己记忆即可；
`ssh-keygen -t rsa -C "xxxxx" -f "id_rsa_xxx"`
生成了两个文件
`id_rsa `         //private key
`id_rsa.pub`      //public key

3. 添加公钥至gitee或github等平台
`cat id_rsa.pub`
复制生成后的 ssh key，通过仓库主页 「管理」->「部署公钥管理」->「添加部署公钥」 ，添加生成的 public key 添加到仓库中。
标题可以设置成gcore_dekstop或gcore_notebook，以区分哪台电脑。

4. 如果你为不同平台创建了独立的公钥，可以创建config文件解决ssh冲突
在.ssh文件夹下执行命令`vi config`

        # gitee
        Host gitee.com
        HostName gitee.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/gitee_id_rsa

        # github
        Host github.com
        HostName github.com
        PreferredAuthentications publickey
        IdentityFile ~/.ssh/github_id_rsa

5. 验证ssh公钥是否配置成功
`ssh -T git@gitee.com`
`ssh -T git@github.com`

>ssh: connect to host gitee.com port 22: Connection refused
fatal: 无法读取远程仓库。

不成功未必是前面的操作错误，有可能是自己的网络问题。有些校园网限制了SSH，只要切换成手机热点就会验证成功。

github 默认新建仓库的主分支名是 main，最好修改成 master。
点击用户头像下拉列表 --> settings --> Repository --> Repository default branch



### Windows 上的 git bash 与 WSL

WSL 是一个类似于 Linux 的兼容层。
您在 WSL 之上运行 Linux 应用程序，他们认为它们在 Linux 上本地运行，而系统调用正在转换为 Windows 操作系统系统调用。
您确实可以通过/mnt/c/... 访问 Windows 文件，但这就是您在 Windows-Linux 互操作性方面所能期望的全部内容。

WSL replaced by WSL2 ，它使用全新的架构，使用 real Linux kernel .
WSL2模拟实际的 Linux 发行版，是在 Windows 中运行的完整 Linux 内核。


Git-bash 是一个 Windows 应用程序——一个运行 Windows 应用程序的 shell 。
其中一些可能是用 Linux 兼容库 (Cygwin) 编译的。但他们不必如此。
这提供了较少“类似 Linux”的体验，但如果您只需要一些 Linux 命令行工具并且不需要安装成熟的 Linux 可执行文件，这可能是一个很好的折衷方案。

Git for Windows正在使用 mingw-w64项目(如 illustrated here)和 msys2 .
适用于 Windows 的 Git 基于 POSIX兼容层，which has limitations :POSIX support is deprecated since Windows 8 .


在WSL里生成的公钥和私钥，在Git-bash里也可以用，反之也是。
WSL:

	cp ~/.ssh/id_rsa* /mnt/d/Desktop/

Git-bash:

	mv /d/Desktop/id_rsa* ~/.ssh



## Git安装
Git 目前支持 Linux/Unix、Solaris、Mac和 Windows 平台上运行。
Git 各平台安装包下载地址为：http://git-scm.com/downloads

Git 的工作需要调用 curl，zlib，openssl，expat，libiconv 等库的代码，所以需要先安装这些依赖工具。

Linux

	$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev
	$ apt-get install git
	$ git --version

	$ yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel
	$ yum -y install git-core
	$ git --version

源码安装
最新源码包下载地址：https://git-scm.com/download

先安装上面的依赖包

	$ tar -zxf git-1.7.2.2.tar.gz
	$ cd git-1.7.2.2
	$ make prefix=/usr/local all
	$ sudo make prefix=/usr/local install

Windows
安装包下载地址：https://gitforwindows.org/
官网慢，可以用国内的镜像：https://npm.taobao.org/mirrors/git-for-windows/

完成安装之后，就可以使用命令行的 Git Bash（自带了 ssh 客户端），和带图形界面的 Git GUI 项目管理工具。

-----------------------------------------------------------------------
Git GUI启动时禁用警告对话框
This repository currently has approximately 1500 loose objects.

如果是Git v1.7.9或更高版本，则可以使用以下命令禁用警告对话框：
git config --global gui.gcwarning false
如果使用的是较旧的版本，则可以编辑/lib/git-core/git-gui并删除after 1000 hint_gc行，或者编辑/usr/share/git-gui/lib/database.tcl并删除hint_gc过程的正文。 
(这些文件路径在Cygwin上-在其他环境下，文件可能位于不同的位置。对于Windows，它是c:\Program Files\Git\mingw64\libexec\git-core\git-gui.tcl)

-----------------------------------------------------------------------

SmartGit
比较好用的带图形界面的Git管理工具。

Mac
在 Mac 平台上安装带图形界面的 Git 安装工具，下载地址为：http://sourceforge.net/projects/git-osx-installer/


## Git配置
Git 提供了一个叫做 `git config` 的工具，专门用来配置或读取相应的工作环境变量。

这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：
1. `/etc/gitconfig`文件
系统中对所有用户都普遍适用的配置。若使用 `git config` 时用 `--system` 选项，读写的就是这个文件。
2. `~/.gitconfig` 文件
用户目录下的配置文件只适用于该用户。若使用 `git config` 时用 `--global` 选项，读写的就是这个文件。
3. `.git/config` 文件
当前项目的.git目录中的配置文件，仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 `.git/config` 里的配置会覆盖 `/etc/gitconfig` 中的同名变量。

在 Windows 系统上，Git 会找寻用户主目录下的 .gitconfig 文件。主目录即 `$HOME` 变量指定的目录，一般都是 `C:\Documents and Settings\$USER`。

此外，Git 还会尝试找寻 `/etc/gitconfig` 文件，只不过看当初 Git 装在什么目录，就以此作为根目录来定位。

https://cloud.tencent.com/developer/article/1565474

### 查看配置信息
要检查已有的配置信息，可以使用 `git config --list `命令：

	$ git config --list
	http.postbuffer=2M
	user.name=wangkai
	user.email=kai.wang@gcoreinc.com

有时候会看到重复的变量名，那就说明它们来自不同的配置文件，不过最终 Git 实际采用的是最后一个。

这些配置我们也可以在 `~/.gitconfig` 或 `/etc/gitconfig` 看到，如下所示：

	$ vim ~/.gitconfig 
	[http]
		postBuffer = 2M
	[user]
		name = wangkai
		email = kai.wang@gcoreinc.com


查看`.git/config`文件
```
[core]
	repositoryformatversion = 0
	filemode = false
	bare = false
	logallrefupdates = true
	symlinks = false
	ignorecase = true
[remote "origin"]
	url = ssh://git@192.168.1.110/volume1/gcore.git
	fetch = +refs/heads/*:refs/remotes/origin/*
[branch "master"]
	remote = origin
	merge = refs/heads/master
[taggrouping]
	groups =
	singles =
[branch "S7_07S1_Samsung"]
	remote = origin
	merge = refs/heads/S7_07S1_Samsung
[pull]
	rebase = true
```


### 用户信息
用户名和邮箱地址是本地git客户端的一个变量，不随git库而改变。
每次commit都会用用户名和邮箱纪录。
github的contributions统计就是按邮箱来统计的。

查看用户名和邮箱地址：

	$ git config user.name
	$ git config user.email

修改用户名和邮箱地址：

	$ git config --global user.name "wangkai"
	$ git config --global user.email "kai.wang@gcoreinc.com"

如果用了 --global 选项，那么更改的配置文件就是位于你用户主目录下的那个，以后你所有的项目都会默认使用这里配置的用户信息。
如果要在某个特定的项目中使用其他名字或者电邮，只要去掉 --global 选项重新配置即可，新的设定保存在当前项目的 .git/config 文件里。

取消每次都要输入用户名和密码
Windows:
在 `C:\Users\your name\` 目录下找到到 `.gitconfig` 这个文件，使用编辑器打开

	[user]
		name = ****
		email = ********.com

文件中增加 [credential]配置

	[credential]
		helper = store

配置完成保存退出，在项目文件夹打开`git bash`，使用`git pull`再输入一次账号密码。
最后一次使用`git pull`

Linux:

	git config --global credential.helper store
	git pull 或者 git push (第一次输入，后续就不用再次输入)




### 文本编辑器
设置Git默认使用的文本编辑器, 一般可能会是 Vi 或者 Vim。如果你有其他偏好，比如 Emacs 的话，可以重新设置：

	$ git config --global core.editor emacs

### 差异分析工具
还有一个比较常用的是，在解决合并冲突时使用哪种差异分析工具。比如要改用 vimdiff 的话：

	$ git config --global merge.tool vimdiff

Git 可以理解 kdiff3，tkdiff，meld，xxdiff，emerge，vimdiff，gvimdiff，ecmerge，和 opendiff 等合并工具的输出信息。



## Git流程
![](Git流程图.drawio.png)


## 下载代码
大概分三种情况

1. 下载的代码不需要包含git管理既不包含.git文件
这个时候直接在github上面点击download下载即可 这时候下载的代码不包含git管理既不包含.git文件

2. 下载的代码要包含git管理但是没有配置ssh
如果需要git管理但是没有ssh权限可以使用git clone repo_url 来下载代码 repo_url为https卡头的url，这个时候下载的代码没有push的权限 如果需要提交修改通过pull request。

3. 下载的代码要包含git管理配置的有ssh
如果需要git管理并且有ssh权限可以使用git clone git@github.com:xxx/xxx.git来下载代码，这样下载的代码有push的权限。


git clone http://git.gcoreinc.com/wangkai0527/GCDemo.git
git clone git@git.gcoreinc.com:wangkai0527/GCDemo.git
git clone git@gitee.com:wangkai0527/gcore.git
git clone ssh://git1@116.247.75.250:27186/home/git1/oppo/mk110.git myproj
执行该命令后，会在当前目录下创建一个myproj项目目录，其中包含一个 .git 的目录，用于保存下载下来的所有版本记录


-----------------------------------------------------------------------
git clone权限不够
如 fatal: unable to access : The requested URL returned error: 403

可能原因是，你之前在本电脑使用过git。但是以前和现在又不是同一个账户。所以当你现在使用 git clone url 时，默认使用以前的账户信息。所以出现没有权限的状况。

解决方法：
重置本机保留的git config 信息。
命令如下：

	git config --system --unset credential.helper

然后你再次克隆的时候，就会让你输入用户名和密码了。

-----------------------------------------------------------------------

-----------------------------------------------------------------------
错误1：OpenSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github.com:443
错误2：github.com port 443: Timed out

查看自己的git代理

	git config --global --get http.proxy
	git config --global --get https.proxy

取消代理

	git config --global --unset http.proxy
	git config --global --unset https.proxy

设置自己的git代理

	git config --global http.proxy http://127.0.0.1:8000
	git config --global https.proxy http://127.0.0.1:8000

添加远程代理

	git config --global --add remote.origin.proxy ""

取消SSL校验

	env GIT_SSL_NO_VERIFY=true

https://www.ipaddress.com/site/github.com
ipadress.com查询ip
github.com
github.global.ssl.fastly.net
把上面查询出来的ip追加到hosts文件里面
140.82.113.4 github.com
199.232.69.194 github.global.ssl.fastly.net

解决办法：

	git config --global http.proxy '127.0.0.1:8000'
	git config --global https.proxy '127.0.0.1:8000'
	git config --global --unset http.proxy
	git config --global --unset https.proxy

-----------------------------------------------------------------------



## 上传代码
大概分三种情况

1. Create a new repository
	git clone git@10.20.60.243:kai.wang/test.git	//拷贝一份远程仓库
	cd test
	touch README.md
	git status										//查看本地仓库当前的状态，显示有变更的文件
	git diff xxx.c									//比较文件的不同，即暂存区和工作区的差异
	git add README.md								//添加文件到暂存区
	git commit -m "add README"						//提交暂存区到本地仓库
	git push -u origin master						//上传远程代码并合并
	git log											//查看历史提交记录
	git log xxx.c
	git log -3 xxx.c

2. Existing folder
	cd existing_folder
	git init										//初始化本地仓库，包含一个.git 目录，该目录包含了资源的所有元数据
	git remote add origin git@10.20.60.243:kai.wang/test.git		//与远程服务器仓库建立连接
	git add .
	git commit -m "Initial commit"
	git push -u origin branch_name					//第一次需要加分支名，以后默认当前分支

3. Existing Git repository
	cd existing_repo
	git remote rename origin old-origin
	git remote add origin git@10.20.60.243:kai.wang/test.git
	git push -u origin --all						//推送所有分支
	git push -u origin --tags


### 远程仓库管理
绑定

	git remote rename origin old-origin
	git remote add origin xxx.git						//与远程服务器仓库建立连接
	git remote remove origin							//解除远程仓库绑定
	git remote -v										//查看远程仓库地址
	git remote show origin


获取更新

	git pull											//下载远程代码并合并
	git pull --rebase									//以变基的方式，避免如果本地与远程commit不同步，会出现自动生成的merge commit
	git pull --rebase origin master						//git pull 拉取不到最新时

执行 git pull --rebase 的时候必须保持本地目录干净。即：不能存在状态为 modified 的文件。（存在Untracked files是没关系的）

	$ git pull --rebase
	error: cannot pull with rebase: You have unstaged changes.
	error: please commit or stash them.

	$ git status .
	On branch master
	Your branch is up to date with 'origin/master'.

	Changes not staged for commit:
	(use "git add <file>..." to update what will be committed)
	(use "git restore <file>..." to discard changes in working directory)
			modified:   .vscode/settings.json

因为本地有文件改动尚未提交造成的。此时有两种做法：
如果本次修改已经完成，则可以先提交一下

	git add .
	git commit -m "xxx"
	git pull --rebase

如果本次修改尚未完成，则可以先贮藏：

	git stash
	git pull --rebase
	git stash pop

如果出现冲突，可以选择手动解决冲突后继续 rebase，

	git rebase –continue

也可以放弃本次 rebase

	git rebase --abort

Git使用过程中出现(master|REBASE 1/10)

	git rebase --abort

提交

	git push -u origin branch_name
	git push --force origin master		//强制推送，慎用



-----------------------------------------------------------------------
在使用git更新或提交项目时候出现 "fatal: The remote end hung up unexpectedly " 原因是推送的文件太大。

那就简单了，要么是缓存不够，要么是网络不行，要么墙的原因
特别是资源库在国外的情况下。此问题可能由网络原因引起。

方法一：
修改提交缓存大小为500M，或者更大的数字

	git config --global http.postBuffer 524288000
	git config --global http.postBuffer 1048576000

或者在克隆/创建版本库生成的 .git目录下面修改生成的config文件增加如下：

[http]  
postBuffer = 524288000


方法二：
配置git的最低速度和最低速度时间：

	git config --global http.lowSpeedLimit 0
	git config --global http.lowSpeedTime 999999  单位 秒

--global配置对当前用户生效，如果需要对所有用户生效，则用--system


方法三：

	fatal: The remote end hung up unexpectedlyB | 2.00 KiB/s

我的是这样的，后面带| 2.00KiB/s  
这句显示 远程结束挂起 |2KiB/s
应该是墙的原因导致网速太慢，且项目有点大上传不上
解决办法：翻墙 或者换个网速好的网络重新再push一遍 就没问题了.我是到公司用公司的网络解决的.

-----------------------------------------------------------------------



## 分支管理

创建分支

	git branch <name>								//创建叫name的分支，但仍然停留在当前分支。

删除分支

	git branch -d <name>							//参数为-D则为强制删除。
	git push origin --delete <name>					//删除远程仓库的叫name的分支，同名的本地分支并不会被删除，所以还需要单独删除本地同名分支
	git branch -dr <remote>/<branch-name>			//没有删除远程分支，只是删除 git branch -r 列表中的追踪分支。一般只有git push命令可以修改远程仓库。

切换分支

	git switch <name>
	git checkout <name>

如果分支不存在，会报错。
可见git switch和git checkout在分支操作方面的用处完全一样。那么可以在分支操作上尽量用git branch和git switch。
因为git checkout除了可以操作分支，它还可以操作文件。这条命令可以重写工作区，是一个很危险的命令。

	git checkout xxx/xxx.c							//恢复暂存区的指定文件到工作区
	git checkout -- xxx/xxx.c
	git checkout [commit] [file]					//恢复某个commit的指定文件到暂存区和工作区
	git checkout .									//恢复暂存区的所有文件到工作区
	git checkout tag_name
	git checkout -b branch_name tag_name

创建+切换分支

	git switch -c <name>
	git checkout -b <name>

如果分支存在则只切换分支。不存在则创建叫name的分支，然后切换到该分支。相当于两条命令：git branch <name>，git checkout <name>

创建本地跟踪分支并从远程分支拉取代码

	git branch --track demo origin/feature/demo
	git switch -c demo origin/feature/demo
	git checkout -b demo origin/feature/demo

建立当前分支与指定远程分支的追踪关系

	git branch -u origin/feature/demo
	git branch --set-upstream-to origin/feature/demo

查看分支

	git branch：查看本地分支，当前分支前面会标一个*号。
	git branch -r：查看远程分支。
	git branch -a：查看本地分支和远程分支，远程分支会用红色表示出来（如果你开了颜色支持的话）。
	git branch -vv：查看本地分支对应的远程分支。

重命名分支

	git branch -m oldName newName

合并分支
当master代码改动了，需要更新开发分支（dev）上的代码

	git checkout master 
	git pull 
	git checkout dev
	git merge master


## 版本管理
如果你有的修改已经加入了暂存区

	git reset --hard
	git reset --hard HEAD
	git clean -xdf

	git回滚到某一个版本
	git reset --hard a136c6923d882ffc9065439f33412936902a1f5d

强制提交

	git push -f origin master

强制覆盖本地文件

	git fetch --all
	git reset --hard origin/master
	git pull

删除暂存区文件

	git rm -r .setting 									//会把工作区和暂存区都删除
	git rm -r --cached .setting 						//--cached不会把本地的删除
	git commit -m 'delete .setting dir'
	git push -u origin master




git reset 之后切换到原来的commit
git reset的语法：
git reset [--hard|soft|mixed|merge|keep] [<commit>或HEAD]
作用：将当前分支reset到指定的commit或者HEAD(默认为最新的一次提交，即重设到最新一次提交之前的版本)
那使用git reset命令之后，想回到以前怎么处理呢？

第一种方法：

	git reflog
	git reset --hard commitid

第二种方法：

	git reflog
	git checkout commitid
	git rebase HEAD branchName

解释下：
git reset之后，你通过git log看不到某些提交的记录了，可以使用git reflog来查看git的所有记录。
第一种方法，使用的就是git reset原理。
第二种方法，先将head指向commitid，之后，再将branch指定到head

	rm -rf .git/rebase-apply






## Git标签
如果你达到一个重要的阶段，并希望永远记住那个特别的提交快照，你可以使用 git tag 给它打上标签。

查看所有标签

	git tag

查看版本信息

	git show tag_name

查看标签,可加上参数-l(列表形式列出） -n(附加说明)

	git tag [-l -n]

查看符合检索条件的标签 

	git tag -l 1.*.* 

创建标签(给最新一次提交HEAD)

	git tag 1.0.0

创建带备注标签(推荐) 

	git tag -a 1.0.0 -m "这是备注信息" 

针对特定commit版本SHA创建标签 

	git tag -a 1.0.0 0c3b62d -m "这是备注信息" 

删除标签(本地) 

	git tag -d tag_name

将本地所有标签发布到远程仓库

	git push origin --tags 

指定版本发送

	git push origin 1.0.0 

删除远程仓库对应标签（Git版本 > V1.7.0）

	git push origin --delete 1.0.0 

旧版本Git 

	git push origin :refs/tags/tag_name

获取远程标签

	git fetch origin tag "标签名称"







## 如何修改Git已提交的日志
情况一：最后一次提交且未push

	git commit --amend
git会打开$EDITOR编辑器，它会加载这次提交的日志，这样我们就可以在上面编辑，编辑后保存即完成此次的修改。

情况二：最后一次提交且已push到服务器

	git commit --amend
	git push origin master --force
	或者git push --force origin HEAD

和情况一的做法一样。使用push推送到远程服务器是需要加上--force，让服务器更新历史记录。

需要注意的是：把修改后的日志强制push到Git服务器，如果别人本地的副本有修改，很有可能会导致他们同步不了，所以最好和他们核对下。

情况三：旧的提交且未推送
假设commit是倒数第3次提交，这个可以使用git log查看，

	$ git log
	commit b1b451d218cc23b6c769f373164f2b89cf54d0aa
	Author: clcaza <clcaza@sina.cn>
	Date:  Sat Mar 10 19:09:08 2018 +0800

	添加内容d

	commit 04f0d1809d5d31cc6e930efcba47a5f3f7e93319
	Author: clcaza <clcaza@sina.cn>
	Date:  Sat Mar 10 19:08:24 2018 +0800

	添加内容c

	commit 94fc8feb916442d56b558d5c370f18f057298921
	Author: clcaza <clcaza@sina.cn>
	Date:  Sat Mar 10 19:07:08 2018 +0800

	添加内容a

	commit fd517efa9faf6a5ec71d0eac38fbcfa0cd689f40
	Author: clcaza <clcaza@sina.cn>
	Date:  Sat Mar 10 19:06:21 2018 +0800


revert是放弃指定提交的修改，但是会生成一次新的提交，需要填写提交注释，以前的历史记录都在，而reset是指将HEAD指针指到指定提交，历史记录中不会出现放弃的提交记录。

	git revert <commitid>
	git revert HEAD
	或者
	//(数字代表回退几个版本)
	git reset --hard HEAD~3
	或者
	执行rebase
	git rebase -i HEAD~3

它会打开一个编辑器，它会把最后前3次的提交显示出来，类似于：

	pick 94fc8fe 添加内容a
	pick 04f0d18 添加内容c
	pick b1b451d 添加内容d

你会看到，它是按提交的顺序显示的，与git log显示的顺序相反。定位到你要编辑日志的那一行，把pick修改为edit，然后保存。

接着就是修改日志内容了

	git commit --amend

完成编辑日志后，记得执行：

	git rebase --continue

Rebase目的是打开提交的历史记录，让您选择要修改的内容。 Git会让你在一个新的分支修改内容。 git rebase --continue则是让你重新回到之前的分支。


情况四：旧的提交且已push到服务器
前面编辑日志的操作是和情况三是一样的：

	git rebase -i HEAD~X
	git commit --amend
	git rebase --continue
X表示倒数第几次提交。

完成编辑日志后，执行push：

	git push origin master --force








## .gitignore文件
```
*.class

#package file
*.war
*.ear

#kdiff3 ignore
*.orig

#maven ignore
target/

#eclipse ignore
.settings/
.project
.classpatch

#idea
.idea/
/idea/
*.ipr
*.iml
*.iws

#temp file
*.log
*.cache
*.diff
*.patch
*.tmp

#system ignore
.DS_Store
Thumbs.db
```


### .gitignore不生效问题解决方法

第一种方法
.gitignore中已经标明忽略的文件目录下的文件，git push的时候还会出现在push的目录中，或者用git status查看状态，想要忽略的文件还是显示被追踪状态。
原因是因为在git忽略目录中，新建的文件在git中会有缓存，如果某些文件已经被纳入了版本管理中，就算是在.gitignore中已经声明了忽略路径也是不起作用的，
这时候我们就应该先把本地缓存删除，然后再进行git的提交，这样就不会出现忽略的文件了。

解决方法: git清除本地缓存（改变成未track状态），然后再提交:

	git rm -r --cached .
	git add .
	git commit -m 'update .gitignore'
	git push -u origin master

需要特别注意的是：
1）.gitignore只能忽略那些原来没有被track的文件，如果某些文件已经被纳入了版本管理中，则修改.gitignore是无效的。
2）想要.gitignore起作用，必须要在这些文件不在暂存区中才可以，.gitignore文件只是忽略没有被staged(cached)文件，
对于已经被staged文件，加入ignore文件时一定要先从staged移除，才可以忽略。

第二种方法
在每个clone下来的仓库中手动设置不要检查特定文件的更改情况。

	git update-index --assume-unchanged PATH     //在PATH处输入要忽略的文件

在使用.gitignore文件后如何删除远程仓库中以前上传的此类文件而保留本地文件
在使用git和github的时候，之前没有写.gitignore文件，就上传了一些没有必要的文件，在添加了.gitignore文件后，就想删除远程仓库中的文件却想保存本地的文件。这时候不可以直接使用"git rm directory"，这样会删除本地仓库的文件。可以使用"git rm -r –cached directory"来删除缓冲，然后进行"commit"和"push"，这样会发现远程仓库中的不必要文件就被删除了，以后可以直接使用"git add -A"来添加修改的内容，上传的文件就会受到.gitignore文件的内容约束。


### Git仓库中文件的大致4种状态
1. Untracked:
未跟踪, 此文件在文件夹中, 但并没有加入到git库, 不参与版本控制. 通过git add 状态变为Staged.

2. Unmodify:
文件已经入库, 未修改, 即版本库中的文件快照内容与文件夹中完全一致. 这种类型的文件有两种去处, 如果它被修改,
而变为Modified. 如果使用git rm移出版本库, 则成为Untracked文件

3. Modified:
文件已修改, 仅仅是修改, 并没有进行其他的操作. 这个文件也有两个去处, 通过git add可进入暂存staged状态,
使用git checkout 则丢弃修改过, 返回到unmodify状态, 这个git checkout即从库中取出文件, 覆盖当前修改

4. Staged:
暂存状态. 执行git commit则将修改同步到库中, 这时库中的文件和本地文件又变为一致, 文件为Unmodify状态.
执行git reset HEAD filename取消暂存, 文件状态为Modified

Git 状态 untracked 和 not staged的区别
1）untrack 表示是新文件，没有被add过，是为跟踪的意思。
2）not staged 表示add过的文件，即跟踪文件，再次修改没有add，就是没有暂存的意思





## .gitmodules
子模块允许你将一个 Git 仓库作为另一个 Git 仓库的子目录。 它能让你将另一个仓库克隆到自己的项目中，同时还保持提交的独立。

	git submodule add https://github.com/XXX

.gitmodules文件
```
[submodule "abc"]
        path = abc
        url = http://github.xxx.xxx/xxxx
        branch = release
```







## Git中CRLF与LF的转换
1.换行符在不同的操作系统上的表示
首先要理解的一点是，对于不同的操作系统，对于换行符的表示是不一样的。也就是说当我们在编辑一个文件，在键盘上按下回车键的时候，对于不同的操作系统保存到文件中的换行符是不一样的。见下表：

CR:表示回车\r
LF:表示换行\n
CRLF:表示回车换行\r\n

敲下回车键，不同的操作系统保存到文件中的值：
Windows：使用的是CRLF ==> 即\r\n，文件中保存的是\r\n
Linux/Unix: 使用的是LF ==> 即\n，文件中保存的是\n
Mac OS: 使用的是CR ==> 即\r，文件中保存的是\r
Mac OS X系统：使用的是LF ==> 即\n，文件中保存的是\n（Mac OS X已经改成和Unix/Linx一样使用LF）
问题: 既然不同的操作系统，对于换行符使用不同的表示形式，如果一个团队在开发一个共同的项目，如果你使用的是windows系统，而你的小伙伴用的是Mac的话，当你们使用git协同开发软件时，就会出现换行符不统一的问题。

虽然对于不同的操作系统，默认的换行符的表示方法不一样，但是编辑器是可以设置在敲下回车键的时候保存的换行符是什么的，比如常用的vscode就可以进行设置。直接点击编辑器右下角的LF或者CRLF，出现如下图所示的设置，直接选择即可。在设置完成之后，在敲回车键，保存在文件中的换行符就是你设置的（CRLF或者是LF，设置什么就是什么）。

2.Git会自动对换行符进行转换
Git为了解决上面提出的问题，会自动对换行符进行转换。转换的方案有3种：

在提交时将CRLF转换为LF，在拉取（检出checkout）时将UNIX换行符（LF）替换成CRLF。（Windows系统推荐使用，我们在windows上安装git的时候，如果一路next，默认是使用这个方案）
在提交时将CRLF转换为LF，在拉取（检出checkout）时不进行转换。（Linux/Unix和Mac OS和Mac OS X推荐使用，在Unix或者类Unix操作系统上安装git，默认使用这种方案）
不进行转换（这种方案对于跨平台项目不推荐使用）。
可以发现，如果不使用第3种方案，那么在Git仓库（包括本地仓库和GitHub远程仓库）中保存的文件的换行符都是LF表示的。

3.自己指定换行符转换方案
我们自己在开发过程中，是可以修改/设置Git的换行符转换方案的。修改/设置的方法有2种。

3.1 通过Git的全局配置进行修改
设置autoclf属性，在控制台直接运行如下的一条命令就可以设置了：

	// 提交时转换为LF，检出时转换为CRLF
	git config --global core.autocrlf true   

	// 提交时转换为LF，检出时不转换
	git config --global core.autocrlf input   

	// 提交检出均不转换
	git config --global core.autocrlf false

上述命令运行之后，会修改.gitconfig文件。

一般在项目中，为了避免项目中同时出现CRLF和LF，还可以开启safecrlf检查。当然，如果你的项目自己定义了语法检查规则，例如使用eslint去约束换行符必须是LF，那么当你的文件中出现CRLF的时候，eslint会给你错误提示信息，告诉你不能包含CRLF，这时候，不开启safecrlf也是可以的 （一般建议开启）。

开启方法如下第一条命令：

// 拒绝提交包含混合换行符的文件 （一般设置为true）
git config --global core.safecrlf true   

// 允许提交包含混合换行符的文件
git config --global core.safecrlf false   

// 提交包含混合换行符的文件时给出警告
git config --global core.safecrlf warn
上述命令运行之后，也会修改.gitconfig文件。

3.2 通过.gitattributes进行修改
参考：https://git-scm.com/docs/gitattributes
注意.gitattributes是针对一个单一的仓库的，也就是说每一个代码仓库都可以包含一个.gitattributes文件。这种方式设置之后，不需要一个项目组里面的同事分别再去修改自己电脑的git的全局配置。

对于通过.gitattributes设置换行符的转换方案，可以使用如下的命令：

1. text=auto：采用git认为最好的方式来处理文件，未在.gitattributes中
设置的项默认按照这种方式处理；（If Git decides that the content
is text, its line endings are converted to LF on checkin.
When the file has been committed with CRLF, no conversion 
is done.）git发现是文本文件,那么在checkin的时候，会将文件结尾符转
换为LF。果文件已经被已CRLF的形式提交（就是说已经在Gti仓库中的文件，如
果结束符是CRLF，不会有任何的转换），不会有任何转换。
 
2. -text 表示让git在checkin以及checkout的时候，对end-of-line不做任何转换。

3. text 表示在checkin的时候会被转换为LF（在repository中的文件结束符是LF），如果需要控制在checkout的时候的换行符，
需要结合eol进行设置（也就是  控制working tree中的文件的结尾符，需要通过eol设置）。 
text=auto和text的区别在于，text=auto由git来确定是不是文本文件，从而进行转换；
而text表示，你确定这个path就是文本文件，会直接对这个path进行转换，而不是有git来decides是否转换。

4. 如果没有指定text，git会使用全局配置中的core.autocrlf来进行eol的转换。core.autocrlf需要自己在自己的电脑上进行配置。

5. eol=crlf：对左边匹配的文件统一使用CRLF换行符格式，如果有文件中出现LF将会转换成CRLF;也就是说，在checkin和checkout的时候，文件中都是CRLF，LF会被转换为CRLF。

7. eol=lf：对左边匹配的文件统一使用LF换行符格式，如果有文件中出现CRLF将会转换成LF;也就是说，在checkin和checkout的时候，文件中都是LF，CRLF会被转换为LF。

8. binary: 告诉git该文件为二进制，防止git修改该文件。git不会对对其中的换行符进行改变。

注意：.gitattributes文件必须要提交之后才能生效。










## Git服务器
### 安装Git

### 创建git用户

	sudo groupadd git
	sudo useradd git -g git

### 创建公钥证书
收集所有需要登录的用户的公钥，公钥位于id_rsa.pub文件中，把我们的公钥导入到`/home/git/.ssh/authorized_keys`文件里，一行一个。

	cd /home/git/
	mkdir .ssh
	chmod 755 .ssh
	touch .ssh/authorized_keys
	chmod 644 .ssh/authorized_keys

### 新增仓库

	mkdir gitrepo
	chown git:git gitrepo/
	cd gitrepo
	git init --bare gcore.git
	chown -R git:git gcore.git

例如:

	ssh -v git@192.168.1.110
	pwd:Qq123456
	cd /volumel/
	git init --bare gcore.git
	sudo chmod 777 gcore.git/*
	sudo chown -R git:users gcore.git/

-----------------------------------------------------------------------
git push -u origin --all失败:
error: unable to write file ....:Permission denied
error: failed to push some refs to '.../gcore.git'

git服务器运行sudo chown -R git:users gcore.git/

-----------------------------------------------------------------------




### 有两种方式可以搭建自己的git服务器

#### 在ubuntu18上配置git服务器并配置客户端公钥实现版本控制(成功实现)

首先配置ssh
一、首先操作服务端：
1、安装 ssh
sudo apt-get install openssh-server openssh-client


2、在服务端创建文件 authorized_keys
cd ~/.ssh
touch authorized_keys
chmod 644 authorized_key

3、将客户端的公钥写入文件
gedit authorized_keys


二、操作客户端
1、安装ssh
sudo apt-get install openssh-client

2、生成客户端公钥
ssh-keygen
gedit /home/figo/.ssh/id_rsa
sudo cp id_rsa.pub ~/文档/ssh

3、将公钥写入服务端 authorized_keys 文件
这步生成的公钥写入上述 一的3中。


设置git
一、服务端
1、安装git
sudo apt-get install git

2、创建一个git库
git init --bare my.git/

二、客户端
1、安装git并设置用户名、邮箱
sudo apt-get install git
git config --global user.email "client@example.com"
git config --global user.name "client“

2、从服务端克隆代码(server为git服务端的名字)
git clone server@172.16.xx.xxx:/home/server/gitdir/mygit
从服务端拉取代码
git pull

3、添加文件到缓存
git add .

4、提交并推送到服务器
git commit -m "test first"
git push -u origin maste






#### 在ubuntu18上配置git服务器并通过Gitolite配置用户权限(成功实现)
可以控制不同用户的权限

一：
1、在服务端安装ssh和git
apt-get update  

安装 ssh
apt-get install openssh-server

安装 Git
apt-get install git


2、在服务端创建一个gitadmin用户
sudo useradd -m gitadmin   
sudo passwd gitadmin


3、将gitadmin设置为管理用户

sudo vim /etc/sudoers

    打开sudoers，在下面位置添加gitadmin

    # User privilege specification
    root ALL=(ALL:ALL)  ALL
    gitadmin  ALL=(ALL:ALL)  ALL

然后保存退出



4、公钥文件改名
在服务器切换到gitadmin用户
su git

生成一对 RSA 密钥
ssh-keygen -t rsa

进入密钥的目标，将公钥文件改名(原名为：id_rsa.pub)
cd /home/git/.ssh
mv id_rsa.pub admin.pub


5、安装Gitolite
进入 gitadmin用户主目录
cd /home/gitadmin

下载 gitolite 的仓库
git clone git://github.com/sitaramc/gitolite

创建 bin 文件夹
mkdir -p $home/bin

安装 gitolite
gitolite/install -to $HOME/bin



6、配置Gitolite

进入密钥目录
cd /home/gitadmin/.ssh

将管理的公钥文件 setup 到 gitolite 中
home/gitadmin/bin/gitolite setup -pk admin.pub

切回gitadmin主目录，多出了一个文件、一个文件夹
cd /home/gitadmin


ls
projects.list 文件：仓库列表文件（gitolite自动创建）
repositories 文件夹：存放所有 git 仓库的文件夹
repositories 文件夹已经存在两个仓库 gitolite-admin.git、testing.git
gitolite-admin.git 管理配置权限的仓库
testing.git 测试仓库


二：
7、下载服务器端的远程管理仓库 gitolite-admin

我是在 GitServer 这个电脑上下载 gitolite-admin，也可以在其他电脑上下载 gitolite-admin 进行管理。

注：使用其他电脑进行管理，需要将其他电脑生产的公钥文件 setup 到 gitolite 中。

进入 gitadmin主目录

cd /home/gitadmin
下载远程管理仓库

git clone gitadmin@172.16.xx.xxx:/gitolite-admin
进入 gitolite-admin 目录，可以看到 conf、keydir 两个文件夹

cd gitolite-admin
keydir 用来存放所有用户的pub公钥文件的，当前目录有 admin.pub 文件

conf 用来配置 Git 仓库、用户、用户组权限的，由目录下 gitolite.conf 文件来配置



8、配置gitolite.conf

进入 conf 目录，编辑 gitolite.conf

cd conf
vi gitolite.conf
作如下编辑：

创建管理组 admin，组员有 admin

创建开发组 dev，组员有 figo

创建仓库 testfigo

admin组 拥有 master 分支读写权

dev组 拥有 dev 分支读写权



@admin = admin
@dev = figo
repo gitolite-admin
  RW+     =   admin

repo testing       
  RW+     =   @all

repo testfigo
RW+     =   @admin
RW+  master   =   @admin
RW+  dev   =   @dev


将figo的公钥上传到 /home/gitadmin/gitolite-admin/keydir/figo.pub

应用修改到远程服务器端

刚刚的配置就是修改了 gitolite-admin 仓库的文件，还需要将修改后的文件提交到服务器端

切换到 gitolite-admin 目录

cd /home/gitadmin/gitolite-admin


9、配置 git config，告诉 Git 你是谁 （根据自己情况）

git config --global user.name "admin"
git config --global user.email "admin@example.com"
提交到远程服务器端（提交的文件：gitolite.conf、figo.pub）

git add .
git commit -m "update gitolite-admin"
git push
进入/home/gitadmin/repositories/目录，新增了 testfigo.git 的仓库文件，这是一个空白仓库

10、客户端clone项目

figo用户的客户端可以 clone 服务端仓库 testfigo.git

git clone gitadmin@172.16.xx.xxx:testfigo.git




## Git仓库瘦身

查看历史大文件，显示历史中体积最大的 5 个文件

	git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -5 | awk '{print$1}')"

从历史中删除 target/ 这个文件夹，只是删除了引用

	git filter-branch --force --index-filter 'git rm -r  --cached --ignore-unmatch target/' --prune-empty --tag-name-filter cat -- --all

执行仓库压缩，触发垃圾回收, 删除没有被引用的文件，从物理上减少了磁盘空间使用

	git gc --prune=now

推送到远程仓库

	git push origin --force --all







