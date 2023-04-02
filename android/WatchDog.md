[TOC]

# WatchDog



## 在嵌入式设备上的应用

Watch Dog的中文意思是“看门狗”，最初是应用子啊嵌入式设备上的，目的是为了防止程序跑飞，所以专门设置了一个硬件看门狗，每隔一段时间，看门狗就会去检查一下某个参数是不是被设置了，如果发现该参数设置出错，就会强制重启程序。



## 在Android上的应用

Android对SystemServer参数是否被设置时很谨慎的，所以专门为它增加了看门狗，可他到底看哪个门呢？就是看几个重要的Service门，如果Android一旦发现几个重要的Service门出问题，就会马上杀掉进程system_server，而这也会使zygote一起自杀，最后导致重启Java世界。

### Android中的Watchdog分析

[Android中的Watchdog](https://jinzhuojun.blog.csdn.net/article/details/46552397?spm=1001.2101.3001.6650.9&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-9-46552397-blog-121949418.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-9-46552397-blog-121949418.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=10)

代码位置:

    frameworks/av/media/libwatchdog/include/watchdog/Watchdog.h
    frameworks/av/media/libwatchdog/Watchdog.cpp

    frameworks/base/services/core/java/com/android/server/Watchdog.java
    frameworks/base/core/jni/android_server_Watchdog.cpp

Watchdog是SystemServer中的独立线程，它隔一定时间间隔会向各监视线程调度一次检查操作。这个检查操作当中会调用已注册的Monitor对象。如果Monitor对象上产生死锁，或是关键线程hang住，那么该检查必定不能按时结束，这样就被Watchdog检查到。


我们先把SystemServer使用Watchdog的调用流程总结一下，然后以为为切入点来分析Watchdog。system_server和Watchdog的交互流程可以总结为：
```java
Watchdog.getInstance().init();

Watchdog.getInstance().start();

Watchdog.getInstance().addMonitor();
```

SystemServer调用Watchdog的start函数，这将导致Watchdog的run在另外一个线程中被执行。
隔一段时间给另外一个线程发送一条MONITOR消息，那个线程将检查各个Service的健康情况。而看门狗会等待检查结果，如果第二次还没有返回结果，那么就会杀死进程systen_server。


这么多的Service,看门狗到底是检查哪个呢？一共有三个Service需要交给Watchgod检查：
ActivityManagerService
PowerManagerService
WindowManagerService
在构造函数中把自己加入Watchdog的检查队列。
`Watchdog.getInstance().addMonitor(this);`

原来Watchdog最怕系统服务死锁了，对于这种情况也只能采取杀死进程的办法了。

```C++
#include <time.h>
#include "Watchdog.h"
 
int main(int argc, char** argv)
{
    // Watchdog在下面的代码块有效，
    // 如果在5s内，没有出代码块，就会狗叫，定时器就会给自己发送SIGABRT信号。
    // 如果在5s内出下面的代码块，就会调用Watchdog析构函数,timer_delete删除定时器。
    // 要旨：主要针对代码块，出代码块会析构
    {
        android::Watchdog watchdog(std::chrono::seconds(5));
        // 具体工作XXX
        sleep(6);
    }
    return 0;
}
```



## 手动实现Watchdog


### 脚本实现

```shell
#!/bin/sh

while true
do
ps -ef | grep "test(程序名)" | grep -v "grep"

if ["$?" -eq 0]
then
./test
echo "wath process has been restarted! "

else
echo "watch process already started ! "

fi

sleep 1
done

```


### 在可执行程序中包含Watchdog

通过父进程监控子进程（任务进程）的运行状态来判断子进程是否崩溃，父进程相当于watchdog。

```C
#include <unistd.h>
#include <signal.h>
#include <sys/wait.h>
#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <assert.h>
#include <pthread.h>

#define ture 1
#define false -1

void *ServerFunc(void *);
void ChildForkReload(int);
int initServer();
int watchdog();

void *ServerFunc(void *avg)
{
    int i = 0;
    while (true) {
        for (i ; i < 5 ; i ++) {
            printf("Func running while i = %d\n", i);
            sleep(1);
        }
        char *p = NULL;
        *p = 1;
    }
}

void ChildForkReload(int n)
{
    int status = 0;
    int pid = wait(&status);
    if (pid < 0) {
        printf("Child Reload err : %s\n", strerror(errno));
        return;
    }

    if (WIFSIGNALED(status)) {
        int signalid = WTERMSIG(status);
        printf("Child exit ,signal = %d\n", signalid);
    }

    if (WCOREDUMP(status)) {
        printf("Core Dumped.\n");
    }

    sleep(2);
    pid = fork();
    if (pid < 0) {
        printf("Child Reload fail.\n");
    } else if (pid == 0) {
        int ret = initServer();
        if (ret == false) {
            printf("Child Reload fail.\n");
        } else {
            printf("Child Reload Success.\n");
        }
    }
    return;
}

int initServer()
{
    int ret = 0;
    pthread_t threadid;
    
    printf("Child thread start.\n");
    ret = pthread_create(&threadid, NULL, ServerFunc, NULL);
    if (ret != 0) {
        printf("Thread create fail.\n");
        return false;
    }
    return true;
}

int watchdog()
{
    int pid = fork();
    if (pid < 0) {
        printf("Watchdog fork fail.\n");
        return false;
    }

    if (pid) {
        while (true) {
            assert(signal(SIGCHLD, ChildForkReload) != SIG_ERR);
            pause();
        }
    }

    return true;
}

int main()
{
    printf("Program Start.\n");
    int ret = 0;
    ret = watchdog();
    if (ret == false) {
        printf("Watchdog start fail.\n");
        return false;
    }

    ret = initServer();
    return 0;
}

```






WIFSIGNALED(status)为非0 表明进程异常终止。

若宏为真，此时可通过WTERMSIG(status)获取使得进程退出的信号编号 。

WCOREDUMP（status）检测是否生成了core文件

wait()就是得到子进程的返回码，防止子进程变成僵尸进程。








