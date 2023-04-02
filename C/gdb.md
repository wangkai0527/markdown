[TOC]

# gdb
GDB（GNU Debugger）是UNIX及UNIX-like下的强大调试工具，可以调试ada, c, c++, asm, minimal, d, fortran, objective-c, go, java,pascal等语言。
对于C程序来说，需要在编译时加上-g参数，保留调试信息，否则不能使用GDB进行调试。


## 编译源码
sudo yum -y install texinfo
wget http://ftp.gnu.org/gnu/gdb/gdb-8.2.tar.gz
tar xvf gdb-8.2.tar.gz
cd gdb-8.2
mkdir build && cd build
../configure --with-python=yes
make -j16
sudo make install




## GUI
在Linux上通常使用gdb命令行调试，但该方式调试不太直观，且命令行长时间不用，容易忘记，不如GUI直观和容易上手，下面介绍基于GUI的方式调试Linux。

### Linux调试GUI方案简介
1. Visual studio 远程调试Linux
在VS2015版本以后Visual studio就支持Linux的编译和调试。使用熟悉的windows界面开发和调试Linux，极大的提高了开发效率，可以广泛应用的Linux服务器开发和嵌入式Linux开发。 
遗憾的是该方式需要基于VS工程来调试，旧的项目是基于Makefile的工程，VS不能调试Makefile工程，通过工具将Makefile工程转VS工程，然后用VS来调试，详见https://github.com/robotdad/vclinux。

2. 基于eclipse 本地调试Linux
因为eclipse是跨平台的，安装一个带GUI的linux系统，就可以像VS一样开发和调试Linux

3. 基于eclipse 远程调试Linux
gdb+gdbserver方式，远程有一个gdbserver,本地机器通过网络发指令给gdbserver完成调试

4. 基于QtCreator 本地调试Linux
因为QtCreator是跨平台的，安装一个带GUI的linux系统，就可以像VS一样开发和调试

5. 基于QtCreator 远程调试Linux
gdb+gdbserver方式，远程有一个gdbserver,本地机器通过网络发指令给gdbserver完成调试

6. 基于vscode 远程调试Linux
该方法支持调试Linux程序，不要编译器参与，可以完美的将Makefile工程简单的接管起来调试,可以是基于ssh+vscode方式或者gdb+vscode+gdbserver方式。本文重点介绍该ssh+vscode方法的使用。

### 基于vscode ssh远程调试Linux实战

1. 软件安装
1)服务器安装gdb
注意我们是ssh+vscode方式，没有用到gdbserver,故不需要安装gdbserver。
gdb+gdbserver方式，在宿主机还需要安装一个交叉编译的gdb，目标机起一个gdbserver去接收和解析指令，详见https://blog.csdn.net/zhaoxd200808501/article/details/77838933。

2)VScode 输入ctrl+shift+x
在扩展专栏安装Remote Development和C/C++,安装后完毕后产生一个SSH工具和debug工具。     


2. 建立ssh连接
这里以简单起见，使用密码账号登录。当然也可以使用公钥登录。详解官方文档
1）选择ssh target
2）点击+，输入 ssh root@IP  –A，选择配置文件路径即可
3） 选择连接服务器，右键---connect host in current window，输入相应密码即可连接SSH服务器


3. 配置debug 参数，并进行调试
1） 点击debug工具栏，选择Open a file用来指定远程服务器debug源文件
即选择远程服务器的debug文件，指定目录和源文件。如C/C++文件
2）选择远程配置文件目录，并创建默认的launch.json文件
3） 修改配置文件
1.可执行文件路径
"program": "${workspaceFolder}/hello",
2.在main入口断点住
"stopAtEntry": true,

4. 选择gdb launch 调试器，就可以启动远程的hello可执行文件，并进行单步，断点等各种调试


5. vscode同样支持attach到某个进程进行在线调试，对线上正在运行的进程进行各种调试和状态查看等





## 开启调试

    gdb HelloWorld

### 运行程序

    run
    run 686

### 设置参数

    set args 686
    run

### attach

    ps -ef|grep HelloWorld
    gdb HelloWorld 20829
    gdb HelloWorld --pid 20829
    
    gdb
    file HelloWorld
    attach 20829

    Could not attach to process.  If your uid matches the uid of the target
    process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
    again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
    ptrace: Operation not permitted.

    将 /etc/sysctl.d/10-ptrace.conf 中的 kernel.yama.ptrace_scope = 1 修改为 kernel.yama.ptrace_scope = 0

### 调试core文件

    gdb HelloWorld core_dump.txt
    //输入bt后，就可以看到调用栈了,出错位置在哪一行
    bt

有时候我们的程序core dump了却没有生成core文件，很可能是我们设置的问题：

    ulimit -c                   #查看core文件配置，如果结果为0，程序core dump时将不会生成core文件
    ulimit -c unlimited         #不限制core文件生成大小
    ulimit -c 10                #设置最大生成大小，单位为块，一块默认为4K


## 断点调试
break(可简写为b)

### 查看已设置的断点
    
    info breakpoints
    
### 根据行号设置断点
    
    b 9
    b test.c:9
    
### 根据函数名设置断点
    
    b printNum
    
### 根据条件设置断点
    
    break test.c:23 if b==0
    //会使得b等于0时，产生断点1。可以很方便地用来改变断点产生的条件
    condition 1 b==0
    
### 根据规则设置断点
    
    rbreak printNum*            #对所有调用printNum函数都设置断点
    rbreak .                    #对所有函数设置断点
    rbreak test.c:.             #对test.c中的所有函数设置断点
    rbreak test.c:^print        #对以print开头的函数设置断点
    
### 设置临时断点
只生效一次
    
    tbreak test.c:l0            #在第10行设置临时断点

### 跳过多次设置断点

    ignore 1 30                 #其中，1是你要忽略的断点号，可以通过前面的方式查找到，30是需要跳过的次数。这样设置之后，会跳过前面30次。
    
### 根据表达式值变化产生断点
    
    watch a
    
程序必须运行起来，否则会出现: `No symbol "a" in current context.`
因为程序没有运行，当前上下文也就没有相关变量信息。
rwatch和awatch同样可以设置观察点前者是当变量值被读时断住，后者是被读或者被改写时断住。

### 禁用或启动断点
    
    disable                     #禁用所有断点
    disable bnum                #禁用标号为bnum的断点
    enable                      #启用所有断点
    enable bnum                 #启用标号为bnum的断点
    enable delete bnum          #启动标号为bnum的断点，并且在此之后删除该断点

### 断点清除
    
    clear                       #删除当前行所有breakpoints
    clear function              #删除函数名为function处的断点
    clear filename:function     #删除文件filename中函数function处的断点
    clear lineNum               #删除行号为lineNum处的断点
    clear f:lename：lineNum     #删除文件filename中行号为lineNum处的断点
    delete                      #删除所有breakpoints,watchpoints和catchpoints
    delete bnum                 #删除断点号为bnum的断点


## 变量查看
print(可简写为p)
打印基本类型，数组，字符数组等直接使用p 变量名

    (gdb) p a
    $1 = 10
    (gdb) p b
    $2 = {1, 2, 3, 5}
    (gdb) p c
    $3 = "hello,shouwang"
    (gdb)

有时候，多个函数或者多个文件会有同一个变量名，这个时候可以在前面加上函数名或者文件名来区分
    
    (gdb) p 'testGdb.h'::a
    $1 = 11
    (gdb) p 'main'::b
    $2 = {1, 2, 3, 5}
    (gdb)

### 查看指针地址

    (gdb) p d
    $1 = (int *) 0x602010
    (gdb)
    
### 查看指针内容
打印指针指向的内容，需要解引用
    
    (gdb) p *d
    $2 = 0
    (gdb) p *d@10
    $3 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    (gdb)
    //仅仅使用*只能打印第一个值，如果要打印多个值，后面跟上@并加上要打印的长度。或者@后面跟上变量值
    (gdb) p *d@a
    $2 = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
    (gdb)
    //由于a的值为10，并且是作为整型指针数据长度，因此后面可以直接跟着a，也可以打印出所有内容。

    //$可表示上一个变量，而假设此时有一个链表linkNode，它有next成员代表下一个节点，则可使用下面方式不断打印链表内容：
    (gdb) p *linkNode
    (这里显示linkNode节点内容)
    (gdb) p *$.next
    (这里显示linkNode节点下一个节点的内容)

    //查看数组的内容，你可以将下标一个一个累加，还可以定义一个类似UNIX环境变量，这样就不需要每次修改下标去打印啦。
    (gdb) set $index=0
    (gdb) p b[$index++]
    $11 = 1
    (gdb) p b[$index++]
    $12 = 2
    (gdb) p b[$index++]
    $13 = 3

### 按照特定格式打印变量
常见格式控制字符如下：
x 按十六进制格式显示变量。
d 按十进制格式显示变量。
u 按十六进制格式显示无符号整型。
o 按八进制格式显示变量。
t 按二进制格式显示变量。
a 按十六进制格式显示变量。
c 按字符格式显示变量。
f 按浮点数格式显示变量。

    (gdb) p c
    $18 = "hello,shouwang"
    (gdb) p/x c
    $19 = {0x68, 0x65, 0x6c, 0x6c, 0x6f, 0x2c, 0x73, 0x68, 0x6f, 0x75, 0x77, 0x61, 
    0x6e, 0x67, 0x0}
    (gdb)
    //如果我们想用这种方式查看浮点数的二进制格式是怎样的是不行的，因为直接打印它首先会被转换成整型
    (gdb) p e
    $1 = 8.5
    (gdb) p/t e
    $2 = 1000
    (gdb)


### 查看内存内容
examine(简写为x)可以用来查看内存地址中的值。语法如下：
x/[n][f][u] addr
其中：
n 表示要显示的内存单元数，默认值为1
f 表示要打印的格式，前面已经提到了格式控制字符
u 要打印的单元长度
addr 内存地址

单元类型常见有如下：
b 字节
h 半字，即双字节
w 字，即四字节
g 八字节
我们通过一个实例来看，假如我们要把float变量e的四个字节按照二进制方式打印，并且打印单位是一字节：

    (gdb) x/4tb &e
    0x7fffffffdbd4:    00000000    00000000    00001000    01000001
    (gdb)



### 自动显示变量内容
假设我们希望程序断住时，就显示某个变量的值，可以使用display命令。

    //每次程序断住时，就会打印e的值。
    (gdb) display e
    1: e = 8.5

    //要查看哪些变量被设置了display，可以使用：
    (gdb)info display
    Auto-display expressions now in effect:
    Num Enb Expression
    1:   y  b
    2:   y  e

    //如果想要清除可以使用
    delete display num #num为前面变量前的编号,不带num时清除所有。

    //或者去使能：
    disable display num  #num为前面变量前的编号，不带num时去使能所有


### 查看寄存器内容

    (gdb)info registers
    rax            0x0    0
    rbx            0x0    0
    rcx            0x7ffff7dd1b00    140737351850752
    rdx            0x0    0
    rsi            0x7ffff7dd1b30    140737351850800
    rdi            0xffffffff    4294967295
    rbp            0x7fffffffdc10    0x7fffffffdc10
    (内容过多未显示完全)


## 单步调试

### 单步执行-next
next命令（可简写为n）用于在程序断住后，继续执行下一条语句，

    //假设已经启动调试，并在第12行停住，如果要继续执行，则使用n执行下一条语句，如果后面跟上数字num，则表示执行该命令num次，就达到继续执行n行的效果了：
    $ gdb gdbStep                                   #启动调试
    (gdb)b 25                                       #将断点设置在12行
    (gdb)run                                        #运行程序
    Breakpoint 1, main () at gdbStep.c:25
    25        int b = 7;
    (gdb) n                                         #单步执行
    26        printf("it will calc a + b\n");
    (gdb) n 2                                       #执行两次
    it will calc a + b
    28        printf("%d + %d = %d\n",a,b,c);
    (gdb)

从上面的执行结果可以看到，我们在25行处断住，执行n之后，运行到26行，运行n 2之后，运行到28行，但是有没有发现一个问题，为什么不会进入到add函数内部呢？那就需要用到另外一个命令啦。

### 单步进入-step
对于上面的情况，如果我们想跟踪add函数内部的情况，可以使用step命令（可简写为s），它可以单步跟踪到函数内部，但前提是该函数有调试信息并且有源码信息。

    $ gdb gdbStep    #启动调试
    (gdb) b 25       #在12行设置断点
    Breakpoint 1 at 0x4005d3: file gdbStep.c, line 25.
    (gdb) run        #运行程序
    Breakpoint 1, main () at gdbStep.c:25
    25        int b = 7;
    (gdb) s          
    26        printf("it will calc a + b\n");
    (gdb) s     #单步进入，但是并没有该函数的源文件信息
    _IO_puts (str=0x4006b8 "it will calc a + b") at ioputs.c:33
    33    ioputs.c: No such file or directory.
    (gdb) finish    #继续完成该函数调用
    Run till exit from #0  _IO_puts (str=0x4006b8 "it will calc a + b")
        at ioputs.c:33
    it will calc a + b
    main () at gdbStep.c:27
    27        int c = add(a,b);
    Value returned is $1 = 19
    (gdb) s        #单步进入，现在已经进入到了add函数内部
    add (a=13, b=57) at gdbStep.c:6
    6        int c = a + b;

从上面的过程可以看到，s命令会尝试进入函数，但是如果没有该函数源码，需要跳过该函数执行，可使用finish命令，继续后面的执行。如果没有函数调用，s的作用与n的作用并无差别，仅仅是继续执行下一行。它后面也可以跟数字，表明要执行的次数。

当然它还有一个选项，用来设置当遇到没有调试信息的函数，s命令是否跳过该函数，而执行后面的。默认情况下，它是会跳过的，即step-mode值是off：

    (gdb) show step-mode 
    Mode of the step operation is off.
    (gdb) set step-mode on
    (gdb) set step-mode off

还有一个与step相关的命令是stepi（可简写为si），它与step不同的是，每次执行一条机器指令：

    (gdb) si
    0x0000000000400573    6        int c = a + b;
    (gdb) display/i $pc
    1: x/i $pc
    => 0x400573 <add+13>:    mov    -0x18(%rbp),%eax
    (gdb)

### 继续执行到下一个断点-continue
我们可能打了多处断点，或者断点打在循环内，这个时候，想跳过这个断点，甚至跳过多次断点继续执行该怎么做呢？可以使用continue命令（可简写为c）或者fg，它会继续执行程序，直到再次遇到断点处：

    $ gdb gdbStep
    (gdb)b 18    #在count函数循环内打断点
    (gdb)run
    Breakpoint 1, count (num=10) at gdbStep.c:18
    18            i++;
    (gdb) c      #继续运行，直到下一次断住
    Continuing.
    1

    Breakpoint 1, count (num=10) at gdbStep.c:18
    18            i++;
    (gdb) fg     #继续运行，直到下一次断住
    Continuing.
    2

    Breakpoint 1, count (num=10) at gdbStep.c:18
    18            i++;
    (gdb) c 3    #跳过三次
    Will ignore next 2 crossings of breakpoint 1.  Continuing.
    3
    4
    5

    Breakpoint 1, count (num=10) at gdbStep.c:18
    18            i++;

### 继续运行到指定位置-until
假如我们在25行停住了，现在想要运行到29行停住，就可以使用until命令（可简写为u）：

    $ gdb gdbStep
    (gdb)b 25
    (gdb)run
    (gdb) u 29
    it will calc a + b
    3 + 7 = 10
    main () at gdbStep.c:29
    29        count(c);

可以看到，在执行u 29之后，它在29行停住了。它利用的是临时断点。

### 跳过执行—skip
skip可以在step时跳过一些不想关注的函数或者某个文件的代码:

    $ gdb gdbStep
    (gdb) b 27
    Breakpoint 1 at 0x4005e4: file gdbStep.c, line 27.
    (gdb) skip function add    #step时跳过add函数
    Function add will be skipped when stepping.
    (gdb) info skip   #查看step情况
    Num     Type           Enb What
    1       function       y   add
    (gdb) run
    Starting program: /home/hyb/workspaces/gdb/gdbStep 
    it will calc a + b

    Breakpoint 1, main () at gdbStep.c:27
    27        int c = add(a,b);
    (gdb) s
    28        printf("%d + %d = %d\n",a,b,c);

可以看到，再使用skip之后，使用step将不会进入add函数。
step也后面也可以跟文件：

    (gdb)skip file gdbStep.c
    这样gdbStep.c中的函数都不会进入。

其他相关命令：

    skip delete [num] 删除skip
    skip enable [num] 使能skip
    skip disable [num] 去使能skip

其中num是前面通过info skip看到的num值，上面可以带或不带该值，如果不带num，则针对所有skip，如果带上了，则只针对某一个skip。



## 源码查看
list命令（可简写为l），它用来打印源码

### 直接打印源码

    $ gdb main
    (gdb) l

直接输入l可从第一行开始显示源码，继续输入l，可列出后面的源码。后面也可以跟上+或者-，分别表示要列出上一次列出源码的后面部分或者前面部分。

### 列出指定行附近源码
l后面可以跟行号

    (gdb) l 9

### 列出指定函数附近的源码
    
    (gdb) l printNum

### 设置源码一次列出行数
不知道你有没有发现，在列出函数源码的时候，它并没有列全，因为l每次只显示10行，那么有没有方法每次列出更多呢？
我们可以通过listsize属性来设置，例如设置每次列出20行：

    (gdb) set listsize 20
    (gdb) show listsize
    Number of source lines gdb will list by default is 20.

这样每次就会列出20行，当然也可以设置为0或者unlimited，这样设置之后，列出就没有限制了，但源码如果较长，查看将会不便。

### 列出指定行之间的源码

    (gdb) l 3,15
    (gdb) list 3,

### 列出指定文件的源码
可以是文件名加行号或函数名，或者指定文件指定行之间

    (gdb) l test.c:1
    (gdb) l test.c:printNum1
    (gdb) l test.c:1,test.c:3

### 指定源码路径
可以使用dir命名指定源码路径

    (gdb) dir /home/hyb/workspaces/gdb/sourceCode/temp
    Source directories searched: /home/hyb/workspaces/gdb/sourceCode/temp:$cdir:$cwd

    $ readelf main -p .debug_str
    [     0]  long unsigned int
    [    12]  short int
    [    1c]  /home/hyb/workspaces/gdb/sourceCode
    [    40]  main.c
    （显示部分内容）

    (gdb) set substitute-path /home/hyb/workspaces/gdb/sourceCode /home/hyb/workspaces/gdb/sourceCode/temp
    (gdb) show substitute-path
    List of all source path substitution rules:
    `/home/hyb/workspaces/gdb/sourceCode' -> `/home/hyb/workspaces/gdb/sourceCode/temp'.
    (gdb)

设置完成后，可以通过show substitute-path来查看设置结果。这样它也能在正确的路径查找源码啦。
需要注意的是，这里对路径做了字符串替换，那么如果你有多个路径，可以做多个替换。甚至可以对指定文件路径进行替换。
最后你也可以通过unset substitute-path [path]取消替换。

### 编辑源码
为了避免已经启动了调试之后，需要编辑源码，又不想退出，可以直接在gdb模式下编辑源码，它默认使用的编辑器是/bin/ex，但是你的机器上可能没有这个编辑器，或者你想使用自己熟悉的编辑器，那么可以通过下面的方式进行设置：

    $ EDITOR=/usr/bin/vim
    $ export EDITOR

/usr/bin/vim可以替换为你熟悉的编辑器的路径，如果你不知道你的编辑器在什么位置，可借助whereis命令或者witch命令查看：

    $ whereis vim
    vim: /usr/bin/vim /usr/bin/vim.tiny /usr/bin/vim.basic /usr/bin/vim.gnome /etc/vim /usr/share/vim /usr/share/man/man1/vim.1.gz
    $ which vim
    /usr/bin/vim

设置之后，就可以在gdb调试模式下进行编辑源码了，使用命令edit location，例如：

    (gdb)edit 3  #编辑第三行
    (gdb)edit printNum #编辑printNum函数
    (gdb)edit test.c:5 #编辑test.c第五行

可自行尝试，这里的location和前面介绍的一样，可以跟指定文件的特定行或指定文件的指定函数。
编辑完保存后，别忘了重新编译程序：

    (gdb)shell gcc -g -o main main.c test.c

这里要注意，为了在gdb调试模式下执行shell命令，需要在命令之前加上shell，表明这是一条shell命令。这样就能在不用退出GDB调试模式的情况下编译程序了。

另外一种模式
启动时，带上tui(Text User Interface)参数，会有意想不到的效果，它会将调试在多个文本窗口呈现：

    gdb main -tui



## 条件断点
http://c.biancheng.net/view/8255.html

https://blog.csdn.net/whahu1989/article/details/125464867






## C/C++目标文件运行段和debug段分离
在C/C++调试过程中不免会遇到crash的情况，这时候一个有debug section的可执行文件+core文件可以方便定位问题。
但是，一个没有去除debug section和其他一些符号段的ELF会很大，所以往往为了节约空间，在编译时不会带-g，甚至strip掉其他的不需要的段，但是这也意味着调试的时候无法精确定位。

其实，上面两个痛点GDB早就考虑到了，可以通过strip+objcopy轻松的做到轻量级上线，并且又保留debug能力。

具体的步骤为：

    $ objcopy --only-keep-debug <obj> <obj>.debug ： 将obj的debug相关的sections拷贝到obj.debug
    $ strip --strip-debug <obj> ：将obj中的重定向期间不需要的section（例如：debug段, symbol段）去除
    $ objcopy --add-gnu-debuglink=<obj>.debug <obj> ：在<obj>文件中加入.gnu_debuglink section，它记录了debug section存在哪个文件里（并且通过CRC保证是出自同一个build）。

这里需要注意的是，<obj>.debug可以是带路径的（相对/绝对），但是建议使用文件名，因为这样的话gdb会去几个默认路径去寻找这个文件，包括：
<obj>所在目录
<obj>下的debug目录
/usr/lib/debug


    debug:
    @objcopy --only-keep-debug bin/Release/Server bin/Release/Server.symbol
    release:
    @objcopy --strip-debug bin/Release/Server

gdb调试:
1. gdb启动时制定symbol文件

    $ gdb -s Server.symbol -e Server -c core


2. gdb运行过程中加载
    
    $gdb Server core
    #(这里中间略去gdb启动的信息)
    (gdb) symbol-file Server.symbol








