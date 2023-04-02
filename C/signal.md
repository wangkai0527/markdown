[TOC]

# signal


signal()信号函数是设置某一个信号的对应动作，它的声明如下 ：

    #include <signal.h>
    typedef void (*sighandler_t)(int);
    sighandler_t signal(int signum, sighandler_t handler);

第一个参数signum为指定要处理信号的类型，可以设置成除SIGKILL和SIGSTOP以外的任何信号。

本例中参数类型设置为SIGCHLD，意为当子进程终止的时候，系统内核会给父进程发送信号。

第二个参数是和信号关联的动作，就是当信号发生并由系统通知进程后，进程需要做什么处理，一般可以取三种值：
1）SIG_IGN，表示忽略该信号；
2）SIG_DFL，对该信号进行默认处理；
3）由程序设计人员设定的信号处理函数sighandler_t handler（）。

注意，类型sighandler_t，表示指向返回值为void型（参数为int型）的函数（的）指针。它可以用来声明一个或多个函数指针，举例如下：

    sighandler_t sig1, sig2; 这个声明等价于下面这种晦涩难懂的写法：

    void (*sig1)(int), (*sig2)(int);




另外，进程中要是没有对信号进行signal（）操作，则进程会对信号采用系统默认的处理方式进行操作。例如程序（进程）产生Segmentation fault错误时，系统内核程序会发送一个SIGSEGV信号通知程序有不合法内存引用的事件发生。若我们在程序中没有编写任何针对该信号的处理函数，系统则按照默认的方式处理传过来的信号（终止程序运行）。



一些常用的C信号处理 signal.h signum.h
主要信号及说明：
SIGHUP 挂起信号
SIGINT 中断信号
SIGQUIT 退出信号
SIGILL 非法指令
SIGTRAP 跟踪/断点中断
SIGABRT 放弃
SIGFPE 浮点异常
SIGKILL 删除（不能捕获或者忽略）
SIGBUS 总线错误
SIGEGV分段错误
SIGSYS 系统调用错误参数
SIGPIPE 管道错误
SIGALRM 闹钟
SIGTERM 软件终止
SIGUSR1 用户信号1
SIGUSR2 用户信号2
SIGCHLD子状态改变
SIGPWR 功能失败/重新启动
SIGWINCH 窗口大小改变
SIGUGR 紧急网络界面接口条件
SIGPOLL 可修改的事件发生
SIGSTOP 停止（不能捕获或忽略）
SIGTSTP 用户停止请求
SIGCONT停止的进程继续进行




-----------------------------------------------------------------------
error: ‘SIGIO’ undeclared

#include <signal.h>

-----------------------------------------------------------------------



https://www.cnblogs.com/yikoulinux/p/14999960.html




