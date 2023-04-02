[TOC]

# File




## 文件权限
Linux命令:修改文件权限命令chmod、chgrp、chown的区别
chmod是更改文件的权限   chown是改改文件的属主与属组  chgrp只是更改文件的属组。
 

（1）chmod是修改文件/目录的权限。可以有文字修改和数字修改。

    chmod 777 /home/berry

    chmod u+x /home/berry

操作对象who可是下述字母中的任一个或者它们的组合：

u 表示“用户（user）”，即文件或目录的所有者。
g 表示“同组（group）用户”，即与文件属主有相同组ID的所有用户。
o 表示“其他（others）用户”。
a 表示“所有（all）用户”。它是系统默认值。

    chmod ug+w，o-x text

（2）chgrp是修改文件或目录所属组。（只是更改文件的属组）

    chgrp -R guest /var/tmp/f.txt

    chgrp - R root /home/berry/file/a.txt

（3）chown修改文件/目录所属拥有者和组。（改文件的属主与属组）

    chown guest:guest a.txt

    chown -R guest /home/berry (把berry文件下的所有文件都改成guest这个组）


## 文件压缩与解压
### tar
tar是在Linux中使用得非常广泛的文档打包格式。它的好处就是它只消耗非常少的CPU以及时间去打包文件，但它仅仅只是一个打包工具，并不负责压缩。

而实际使用中，除了要打包之外，往往还需要进行一下压缩，提高空间利用率。因此，很多时候，tar命令并不是单独出现的，而是伴随着其他的压缩命令一起出现。
比如：tar.gz格式，tar.bz2格式，tar.xz格式等等。
另外，也有可能会缩写合并之后出现，比如：tgz格式。对于这些文件应该如何处理呢，下面就进行详细的分析

压缩
tar -cvf target_name.tar dir_or_file					# 将目标打包成一个*.tar格式的文件
tar -zcvf target_name.tgz dir_or_file					# 将目标打包成一个*.tgz格式的文件
tar -zcvf target_name.tar.gz dir_or_file				# 将目标打包并压缩成一个*.tar.gz格式的文件
tar -zcvf target_name.tar.Z dir_or_file					# 将目标打包并压缩成一个*.tar.Z格式的文件
tar -jcvf target_name.tar.bz2 dir_or_file				# 将目标打包并压缩成一个*.tar.bz2格式的文件

解压
可以用参数"-C"来更改解包的路径
-C 指定目录

tar -xvf pakage_name.tar						        # 解开一个*.tar的文件包内容到当前目录下
tar -zxvf pakage_name.tgz						        # 解开一个*.tgz的文件包内容到当前目录下
tar -zxvf pakage_name.tar.gz						    # 解开一个*.tar.gz的压缩包内容到当前目录下
tar -zxvf pakage_name.tar.Z						        # 解开一个*.tar.Z的压缩包内容到当前目录下
tar -jxvf pakage_name.tar.bz2						    # 解开一个*.tar.bz2的压缩包内容到当前目录下

参数说明
命令类型参数：
-c或–create：建立新的备份文件。
-x或–extract或–get：从备份文件中还原文件。

压缩方式参数：
-j或–bzip2：以bz2的算法来压缩或者解压文件。
-z或–gzip或–ungzip：通过 gzip 指令处理备份文件。

其他常用参数：
-v或–verbose：显示指令执行过程。
-C<目的目录>或–directory=<目的目录>：切换到指定的目录。


### xz
xz 命令用来解压缩 .xz 文件，压缩率比较高。

xz -z 要压缩的文件
xz -d 要解压的文件
-k 表示不删除原文件
-0 到 -9调节压缩率，默认为6.

压缩
tar -cvf target_name.tar dir_or_file					# 1.将目标打包成一个*.tar格式的文件
xz -z target_name.tar.xz						# 2.将打包好的文件压缩成一个*.tar.xz格式的文件

解压
xz -d pakage_name.tar.xz					# 1.解开tar.xz压缩包为tar格式包
tar -xvf pakage_name.tar					# 2.解开tar格式包到当前目录下
或者
tar -Jxvf **.tar.xz


### gzip
gzip 是 Linux 系统中经常用来对文件进行压缩和解压缩的命令，通过此命令压缩得到的新文件，其扩展名通常标记为“.gz”。
再强调一下，gzip 命令只能用来压缩文件，不能压缩目录，即便指定了目录，也只能分别压缩目录内的所有文件。

压缩
gzip file

解压
gzip -dv xxx.gz
或者
gunzip xxx.gz
gunzip 是个使用广泛的解压缩程序，它用于解开被 gzip 压缩过的文件，这些压缩文件预设最后的扩展名为 .gz。事实上 gunzip 就是 gzip 的硬连接。


### bzip2
bzip2采用新的压缩演算法，压缩效果比传统的LZ77/LZ78压缩演算法来得好。若没有加上任何参数，bzip2压缩完文件后会产生.bz2的压缩文件，并删除原始的文件。
bzip2 命令同 gzip 命令类似，只能对文件进行压缩（或解压缩）。

压缩
bzip2 -ckv file
注意，gzip 只是不会打包目录，但是如果使用“-r”选项，则可以分别压缩目录下的每个文件；而 bzip2 命令则根本不支持压缩目录，也没有“-r”选项。

解压
bzip2 -dkv xxx.bz2
bunzip2 -kv xxx.bz2
bunzip2可解压缩.bz2格式的压缩文件。bunzip2实际上是bzip2的符号连接，执行bunzip2与bzip2 -d的效果相同。

检查文件完整性
bzip2 -t xxx.bz2


### compress
Linux compress命令是一个相当古老的 unix 档案压缩指令，压缩后的档案会加上一个 .Z 延伸档名以区别未压缩的档案，压缩后的档案可以以 uncompress 解压。
若要将数个档案压成一个压缩档，必须先将档案 tar 起来再压缩。由于 gzip 可以产生更理想的压缩比例，一般人多已改用 gzip 为档案压缩工具。
sudo apt-get install ncompress

压缩
compress -b 7 file
compress -rf dir

解压
compress -d xxx.Z
uncompress xxx.Z


### 7z
sudo apt-get install p7zip

压缩
7za a -t7z -r -mx=9 Mytest.7z /opt/temp/*

a 代表添加文件／文件夹到压缩包。
t 是指定压缩类型，这里定为7z，可不指定，因为7za默认压缩类型就是7z。
r 表示递归所有的子文件夹。
Mytest.7z 是压缩好后的压缩包名。
/opt/temp/*：是压缩目标。
-mx=9 表明我们需要极限的压缩比。默认为5。

解压
7za x test.7z -r -o./

x 代表解压缩文件，并且是按原始目录树解压（还有个参数 e 也是解压缩文件，但其会将所有文件都解压到根下，而不是自己原有的文件夹下）。
test.7z 是压缩文件。
r 表示递归解压缩所有的子文件夹。
o 是指定解压到的目录，-o后是没有空格的，直接接目录。这一点需要注意。


### zip

压缩
zip -r target.zip dir_or_file

A 调整可执行的自动解压缩文件。
d 从压缩文件内删除指定的文件。
D 压缩文件内不建立目录名称。
F 尝试修复已损坏的压缩文件。
g 将文件压缩后附加在既有的压缩文件之后，而非另行建立新的压缩文件。
j 只保存文件名称及其内容，而不存放任何目录名称。
m 将文件压缩并加入压缩文件后，删除原始文件，即把文件移到压缩文件中。
o 以压缩文件内拥有最新更改时间的文件为准，将压缩文件的更改时间设成和该文件相同。
q 不显示指令执行过程。
r 递归处理，将指定目录下的所有文件和子目录一并处理。
S 包含系统和隐藏文件。
v 显示指令执行过程或显示版本信息。
<压缩效率> 压缩效率是一个介于 1-9 的数值。

解压
unzip xxx.zip -d xxx
f 更新现有的文件。
l 显示压缩文件内所包含的文件。
v 执行是时显示详细的信息。
C 压缩文件中的文件名称区分大小写。
j 不处理压缩文件中原有的目录路径。
L 将压缩文件中的全部文件名改为小写。
n 解压缩时不要覆盖原有的文件。
o 不必先询问用户，unzip 执行后覆盖原有文件。
P<密码> 使用 zip 的密码选项。
q 执行时不显示任何信息。
[.zip 文件] 指定.zip 压缩文件。
[文件] 指定要处理.zip 压缩文件中的哪些文件。
d<目录> 指定文件解压缩后所要存储的目录。
x<文件> 指定不要处理.zip 压缩文件中的哪些文件。


### rar
sudo apt-get install rar
sudo apt-get install unrar

压缩
rar a abc.rar dir_or_file

解压
rar x fileName.rar




### 压缩比率和占用时间对比
压缩比率=原内容大小/压缩后大小，压缩比率越大，则表明压缩后占用空间的压缩包越小。
为了保证能够让压缩比率较为明显，需选取一个内容较多、占用空间较大的目录作为本次实验的测试。
找了一个大概有23G的目录来测试,首先要明确由于执行环境的变化，误差在所难免，使用time命令。


|               |   .tar          |   .tgz          |   .tar.bz2      |
|   :--------   |   ---:          |   :---:         |   :---:         |
|   打包前大小   |   23214680      |   23214680      |   23214680      |
|   打包后大小   |   22202985      |   18949032      |   18728904      |
|   打包耗时     |   3m20.709s     |   16m30.767s    |   80m39.422s    |
|   解压耗时     |   2m47.548s     |   3m52.418s     |   27m54.525s    |
|   压缩比率     |   0.956419989   |   0.816379292   |   0.806895539   |




综合起来，在压缩比率上： tar.bz2>tgz>tar
占用空间与压缩比率成反比： tar.bz2<tgz<tar
耗费时间（打包，解压）
打包：tar.bz2>tgz>tar
解压： tar.bz2>tar>tgz
从效率角度来说，当然是耗费时间越短越好

因此，Linux下对于占用空间与耗费时间的折衷多选用tgz格式，不仅压缩率较高，而且打包、解压的时间都较为快速，是较为理想的选择。

结论：
再一次印证了物理空间与时间的矛盾（想占用更小的空间，得到高压缩比率，肯定要牺牲较长的时间；反之，如果时间较为宝贵，要求快速，那么所得的压缩比率一定较小，当然会占用更大的空间了）。






### 批量压缩
linux批量压缩当前目录中文件后，删除原文件
for i in `ls|awk -F " " '{print $NF}'`; do tar -zcvf $i.tar.gz $i --remove-files;done



下面的脚本可以压缩日志并删除原始文件
```bash
#!/bin/bash
yesterday=`date -d '1days ago' +%Y_%m_%d`
cd $1
find . -name "*$yesterday*.log" -type f | xargs -I {} tar -zcvf {}.tar.gz {} --remove-files
```

说明加上参数--remove-files，tar命令可以压缩并删除的源文件
这样只能删除文件，如果删除源文件夹，可以使用以下方法
tar -zcvf aaa.tar.gz aaa && rm -rf aaa




假设我们压缩文件文件aaa.log 为aaa.log.tar.gz ，归档压缩之后，并删除文件aaa.log。请参阅下面的命令：
 
tar -zcvf aaa.log.tar.gz aaa.log --remove-files
 
可以看出，主要是使用了--remove-files 这个命令参数选项。
能不能拓展下：解压 aaa.log.tar.gz之后，并删除 aaa.log.tar.gz？我看了一遍又一遍帮助，一直没有发现合适的命令参数选项。不过，完全可以通过一种变通的方法来实现：
 
tar -zxvf aaa.txt.tar.gz && rm -rf aaa.txt.tar.gz
对于上面两种应用，是不是可以进一步拓展出以下两种比较有实际意义的应用：
1、遍历压缩归档日志文件：
 
find . -name "*.log" -type f -exec tar -zcvf {}.tar.gz {} --remove-files > /dev/null \;
2、遍历解压tar.gz文件，并删除tar.gz文件
 
find . -name "*.tar.gz" -type f -exec tar -zxvf {} \; -exec rm -rf {} \; > /dev/null           







## 文件系统

linux和windows的磁盘文件格式不一样，能实现文件共享，需要安装ntf驱动，不同的文件系统把同一个文件转换成字节流


### 扇区与块（sectors,block）

先来说说硬盘吧
最终文件总还是要储存在硬盘上的嘛。

    # fdisk -l
    Disk /dev/cciss/c0d0: 146.7 GB, 146778685440 bytes
    255 heads, 63 sectors/track, 17844 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes

可以看到几个名词：heads/sectors/cylinders，分别就是磁头/扇区/柱面，每个扇区512byte（现在新的硬盘每个扇区有4K）了
硬盘容量就是heads*sectors*cylinders*512=255*63*17844*512=146771896320b=146.7G
注意：硬盘的最小存储单位就是扇区了，而且硬盘本身并没有block的概念。

操作系统的文件系统
文件系统不是一个扇区一个扇区的来读数据，太慢了，所以有了block（块）的概念，它是一个块一个块的读取的，block才是文件存取的最小单位。
先来知道是哪种文件系统

    # df -T
    Filesystem           Type   Size       Used      Avail     Use%  Mounted on
    /dev/cciss/c0d0p5    ext3   112738028  81733116  25185772  77%   /

OK，ext3文件系统 。

    # tune2fs -l /dev/cciss/c0d0p5 | grep "Block size"
    Block size:               4096

一个block是4K，也就是说我所使用的文件系统中1个块是由连续的8个扇区组成。

简单的说扇区是对硬盘而言，块是对文件系统而言。



### 扇区，sector
硬盘的读写以扇区为基本单位。磁盘上的每个磁道被等分为若干个弧段，这些弧段称之为扇区。硬盘的物理读写以扇区为基本单位。通常情况下每个扇区的大小是 512 字节。linux 下可以使用 fdisk -l 了解扇区大小：

    # sudo fdisk -l
    $ sudo /sbin/fdisk -l
    Disk /dev/sda: 20 GiB, 21474836480 bytes, 41943040 sectors
    Units: sectors of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disklabel type: dos
    Disk identifier: 0x7d9f5643

其中 Sector size，就是扇区大小，本例中为 512 bytes。

注意，扇区是磁盘物理层面的概念，操作系统是不直接与扇区交互的，而是与多个连续扇区组成的磁盘块交互。由于扇区是物理层面的概念，所以无法在系统中进行大小的更改。



### 磁盘块，IO Block
文件系统读写数据的最小单位，也叫磁盘簇。扇区是磁盘最小的物理存储单元，操作系统将相邻的扇区组合在一起，形成一个块，对块进行管理。每个磁盘块可以包括 2、4、8、16、32 或 64 个扇区。磁盘块是操作系统所使用的逻辑概念，而非磁盘的物理概念。磁盘块的大小可以通过命令 stat /boot 来查看：

    $ sudo stat /boot
    File: /boot
    Size: 4096        Blocks: 8          IO Block: 4096   directory
    Device: 801h/2049d  Inode: 655361      Links: 3
    Access: (0755/drwxr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
    Access: 2019-07-06 20:19:45.487160301 +0800
    Modify: 2019-07-06 20:19:44.835160301 +0800
    Change: 2019-07-06 20:19:44.835160301 +0800
    Birth: -

其中 IO Block 就是磁盘块大小，本例中是 4096 Bytes，一般也是 4K。

为了更好地管理磁盘空间和更高效地从硬盘读取数据，操作系统规定一个磁盘块中只能放置一个文件，因此文件所占用的空间，只能是磁盘块的整数倍，那就意味着会出现文件的实际大小，会小于其所占用的磁盘空间的情况。

test2.txt是一个只包含一个字母的文本文档。它的理论大小是一个字节，但是由于系统的磁盘块大小是4KB（文件的最小存储大小单位），所以test2.txt占据的磁盘实际空间是4KB

操作系统不能对磁盘扇区直接寻址操写，主要原因是扇区数量庞大，因此才将多个连续扇区组合一起操作。磁盘块的大小是可以通过blockdev命令更改的。





### 页，page
内存的最小存储单位。页的大小通常为磁盘块大小的 2^n 倍，可以通过命令 getconf PAGE_SIZE 来获取页的大小：

    $sudo getconf PAGE_SIZE
    4096

本例中为 4096 Bytes，与磁盘块大小一致。

总结两个逻辑单位：

页，内存操作的基本单位
磁盘块，磁盘操作的基本单位






## 查看磁盘空间
Linux 查看磁盘空间可以使用 df 和 du 命令。

### df
df 以磁盘分区为单位查看文件系统，可以获取硬盘被占用了多少空间，目前还剩下多少空间等信息。

$ df -h
$ df -hT
Filesystem           Type  Size  Used Avail Use% Mounted on
D:/Program Files/Git ntfs  463G  6.6G  456G   2% /
C:                   ntfs  233G   78G  156G  34% /c
E:                   ntfs  470G  300G  170G  64% /e
O:                   cifs   20T  4.0T   17T  20% /o

显示内容参数说明：
Filesystem：文件系统
Type：文件系统类型
Size： 分区大小
Used： 已使用容量
Avail： 还可以使用的容量
Use%： 已用百分比
Mounted on： 挂载点


### du
du 的英文原义为 disk usage，含义为显示磁盘空间的使用情况，用于查看当前目录的总大小。

du -sh [目录名或文件名]：返回该目录的大小
du -sm [目录名或文件名]：返回该文件夹总M数
du -h [目录名或文件名]：查看指定文件夹下的所有文件大小（包含子文件夹）

du -h --max-depth=0
du -h --max-depth=1 文件名
`--max-depth=`指定深入目录的层数（如果不使用该参数，则会循环列出当前目录下所有文件及文件夹使用的空间大小，文件多时会很乱）：
(1) --max-depth=0：查看当前目录使用的总空间大小；
(2) --max-depth=1：查看当前目录使用总空间的大小以及当前目录下一级文件及文件夹各自使用的总空间大小；
--max-depth 等价于 -d
du -h --max-depth=0 等价于 du -hd0







## 快速创建新文件

在Linux中，我们可以从命令行或桌面文件管理器创建一个新文件。

在开始之前
要创建一个新文件，您需要对父目录具有写权限。否则，您将收到一个权限被拒绝的错误。


### 使用touch命令创建文件
touch命令可以让我们来更新现有的文件和目录以及创建新的空文件的时间戳。

创建新的空文件的最简单，最难忘的方法是使用touch命令。

要创建新文件，只需运行touch命令，然后输入要创建的文件名即可：

touch file1.txt


如果文件file1.txt不存在，则上面的命令将创建该文件，否则，它将更改其时间戳。

要一次创建多个文件，请指定文件名，并用空格分隔：

touch file1.txt file2.txt file3.txt


### 使用重定向运算符创建文件
重定向允许您捕获命令的输出，并将其作为输入发送到另一个命令或文件。有两种方法可以将输出重定向到文件。> 操作符将覆盖现有文件，而>> 操作符将追加输出到文件中。

要创建一个空的零长度文件，只需在重定向操作符之前指定要创建的文件名即可：

> file1.txt


这是在Linux中创建新文件的最短命令。

使用重定向创建文件时，请注意不要覆盖现有的重要文件。

### 使用cat命令创建文件
该cat命令主要用于读取和连接文件，但它也可以用于创建新的文件。

要创建新文件，请运行cat命令，后跟重定向操作符>和要创建的文件名。按Enter输入文字，完成后按CRTL+D保存文件。

cat > file1.txt


### 使用echo命令创建文件
所述echo命令的标准输出，其传递的字符串可以被重定向到文件。

要创建新文件，请运行echo命令，后跟要打印的文本，然后使用重定向操作符>将输出写入要创建的文件。

echo "Some line" > file1.txt


如果要创建一个空文件，只需使用：

echo > file1.txt


### 使用Heredoc创建文件
这里document或Heredoc是一种重定向类型，允许您将多行输入传递给命令。

当您要从Shell脚本创建包含多行文本的文件时，通常使用此方法。

例如，要创建一个新文件，file1.txt您将使用以下代码：

<< EOF > file1.txtSome lineSome other lineEOF

Heredoc的正文可以包含变量，特殊字符和命令。

### 使用dd命令创建一个大文件
有时，出于测试目的，您可能需要创建一个大数据文件。当您要测试驱动器的写入速度或测试连接的下载速度时，此功能很有用。

dd命令主要用于转换和复制文件。

要创建一个1G.test大小为1GB 的文件，请运行：

dd if=/dev/zero of=1G.test bs=1 count=0 seek=1G


### 使用fallocate命令创建一个大文件
fallocate 一个命令行实用程序，用于为文件分配实际磁盘空间。

以下命令将创建一个名为1G.test1GB 的新文件：

fallocate -l 1G 1G.test



https://zhuanlan.zhihu.com/p/57091723






## 文件读写
在做U盘存储时，每次write()之后做fsync()，每次存储的数据都为固定的8K数据，为什么每次执行write()和fsync()消耗的时间多数约为十几毫秒，但也会出现一两百毫秒的情况，为什么时间差别这么大?
1.因为写数据快，但寻道或分配磁盘块比较慢。
如果当前文件系统为这个文件分配的文件存储布局有空余空间，磁头又恰好在要写的位置，很快，反之要花更多的时间。

2.这和系统缓存机制及MMC有关,
在调用write把数据写入以后,实际上是写入的缓存中.
有3种情况下系统会把缓存中数据写会磁盘中:
1,调用fsync() ,sync()等强制同步函数。调用fclose函数
2,缓存已满.
3.MMC管理分配内存是发现内存不足,也会把缓冲区内数据写出而收会缓冲区.
对于你所遇到的情况,在你调用fsync时,2,3两中情况如果已发生,那时间就很短,如果没有,往u盘上8K数据有时是很慢的.
这个时间差别,也和你的u盘写入速度有关.



fwrite是写缓冲的，由系统决定什么时候写磁盘或调用flush，所以当写磁盘时，一个线程的时间片不能够把缓冲数据完全写入磁盘时(数据量大)，同时十几个线程都是这种状态轮询切换，就导致了堵塞(最低性能)。
fwrite写缓冲是一种提升性能的方式，所以没有错，那么就要在写磁盘的时间上控制了，也就是多久调用一次flush。
这个从两方面考虑：
一是找到最合适的时间间隔，时间太短浪费，太长则数据堵塞。
二是写磁盘的数据块大小，按照磁盘写扇区写块的大小是最优的。最优情况是：在单位时间片内刚好写完不需要重新调度的数据块。
不知说明白了没有，比如磁盘块的大小为4096，那么就按照4096的倍数去写数据，但还要考虑线程切换时间内写完。
比如每100、200毫秒写 4096 * 100 的数据。



当调用fwrite，需要一次性写入100M数据时，速度超慢，10Min还未写完。开始怀疑代码问题，不断跟踪代码， 发现当写入几十KB文件大小时，速度很快，没问题。一旦写入的数据上百MB，速度奇慢无比。
由于我的开发和调试时在VMware虚拟机下，并且同WIN7，共享。而生成的文件也在共享目录，/mnt/hgfs/linux/下。
突然想到，很有可能是磁盘文件格式不同，导致的原因。因为在共享目录下，既支持windows的文件格式，又支持虚拟机下linux的文件磁盘格式，会产生冲突。
将生成的文件目录拷贝/home/workspace/下，问题解决，升级文件快速生成。


### open与fopen的区别
1. 来源
open是UNIX系统调用函数（包括LINUX等），返回的是文件描述符（File Descriptor），它是文件在文件描述符表里的索引。
fopen是ANSIC标准中的C语言库函数，在不同的系统中应该调用不同的内核api。返回的是一个指向文件结构的指针。 
 PS:从来源来看，两者是有千丝万缕的联系的，毕竟C语言的库函数还是需要调用系统API实现的。
 
2. 移植性
这一点从上面的来源就可以推断出来，`fopen`是C标准函数，因此拥有良好的移植性；而`open`是UNIX系统调用，移植性有限。如windows下相似的功能使用API函数`CreateFile`。

3. 适用范围
open返回文件描述符，而文件描述符是UNIX系统下的一个重要概念，UNIX下的一切设备都是以文件的形式操作。如网络套接字、硬件设备等。当然包括操作普通正规文件（Regular File）。
fopen是用来操纵普通正规文件（Regular File）的。
 
4. 文件IO层次
如果从文件IO的角度来看，open属于低级IO函数，fopen属于高级IO函数。低级和高级的简单区分标准是：谁离系统内核更近。低级文件IO运行在内核态，高级文件IO运行在用户态。
 
5. 缓冲
缓冲文件系统 
缓冲文件系统的特点是：在内存开辟一个“缓冲区”，为程序中的每一个文件使用；当执行读文件的操作时，从磁盘文件将数据先读入内存“缓冲区”，装满后再从内存“缓冲区”依此读出需要的数据。执行写文件的操作时，先将数据写入内存“缓冲区”，待内存“缓冲区”装满后再写入文件。由此可以看出，内存“缓冲区”的大小，影响着实际操作外存的次数，内存“缓冲区”越大，则操作外存的次数就少，执行速度就快、效率高。一般来说，文件“缓冲区”的大小随机器 而定。
fopen, fclose, fread, fwrite, fgetc, fgets, fputc, fputs, freopen, fseek, ftell, rewind等。

非缓冲文件系统 
缓冲文件系统是借助文件结构体指针来对文件进行管理，通过文件指针来对文件进行访问，既可以读写字符、字符串、格式化数据，也可以读写二进制数据。非缓冲文件系统依赖于操作系统，通过操作系统的功能对文件进行读写，是系统级的输入输出，它不设文件结构体指针，只能读写二进制文件，但效率高、速度快，由于ANSI标准不再包括非缓冲文件系统，因此建议大家最好不要选择它。
open, close, read, write, getc, getchar, putc, putchar等。

一句话总结一下，就是open无缓冲，fopen有缓冲。前者与read, write等配合使用， 后者与fread,fwrite等配合使用。

使用fopen函数，由于在用户态下就有了缓冲，因此进行文件读写操作的时候就减少了用户态和内核态的切换（切换到内核态调用还是需要调用系统调用API:read，write）；
而使用open函数，在文件读写时则每次都需要进行内核态和用户态的切换；
表现为，如果顺序访问文件，fopen系列的函数要比直接调用open系列的函数快；如果随机访问文件则相反。

这样一总结梳理，相信大家对于两个函数及系列函数有了一个更全面清晰的认识，也应该知道在什么场合下使用什么样的函数更合适，效率更高。


```C++
int open(const char *path, int access,int mode)  
    path 要打开的文件路径和名称                             
    access 访问模式，宏定义和含义如下：                          
        O_RDONLY         1    只读打开                           
        O_WRONLY         2    只写打开                           
        O_RDWR           4    读写打开                       
        还可选择以下模式与以上3种基本模式相与：                      
            O_CREAT     0x0100   创建一个文件并打开                  
            O_TRUNC     0x0200   打开一个已存在的文件并将文件长度设置为0，其他属性保持           
            O_EXCL      0x0400   未使用                              
            O_APPEND    0x0800   追加打开文件                       
            O_TEXT      0x4000   打开文本文件翻译CR-LF控制字符       
            O_BINARY    0x8000   打开二进制字符，不作CR-LF翻译                                                          
    mode 该参数仅在access=O_CREAT方式下使用，其取值如下：        
        S_IFMT      0xF000   文件类型掩码                        
        S_IFDIR     0x4000   目录                                
        S_IFIFO     0x1000   FIFO 专用                           
        S_IFCHR     0x2000   字符专用                            
        S_IFBLK     0x3000   块专用                              
        S_IFREG     0x8000   只为0x0000                          
        S_IREAD     0x0100   可读                                
        S_IWRITE    0x0080   可写                                
        S_IEXEC     0x0040   可执行 

FILE *fopen(char *filename, char *mode)  
    filename 文件名称  
    mode 打开模式：                                              
        r   只读方式打开一个文本文件                             
        rb  只读方式打开一个二进制文件                           
        w   只写方式打开一个文本文件                             
        wb  只写方式打开一个二进制文件                           
        a   追加方式打开一个文本文件                             
        ab  追加方式打开一个二进制文件                           
        r+  可读可写方式打开一个文本文件                         
        rb+ 可读可写方式打开一个二进制文件                       
        w+  可读可写方式创建一个文本文件                         
        wb+ 可读可写方式生成一个二进制文件                       
        a+  可读可写追加方式打开一个文本文件                     
        ab+ 可读可写方式追加一个二进制文件 

```



