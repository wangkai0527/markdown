[TOC]

# pthread







# TLS

Thread-local storage (TLS)
TLS is a computer programming method that uses static or global memory local to a thread.

This is sometimes needed because normally all threads in a process share the same address space, which is sometimes undesirable. In other words, data in a static or global variable is normally always located at the same memory location, when referred to by threads from the same process. Variables on the call stack however are local to threads, because each thread has its own stack, residing in a different memory location.

Sometimes it is desirable that two threads referring to the same static or global variable are actually referring to different memory locations, thereby making the variable thread-local, a canonical example being the C error code variable errno.

If it is possible to make at least a memory address sized variable thread-local, it is in principle possible to make arbitrarily sized memory blocks thread-local, by allocating such a memory block and storing the memory address of that block in a thread-local variable.

 

例如:
Fatal: errno: TLS reference is libxxx mismatches non-TLS definition in xxx.

部分变量errno(注意是errno, 不是error)这样声明：

    #ifndef errno
    extern int errno;
    #endif

但是多线程环境中errno并不一定都是int，改用：

    #include <errno.h>



__thread关键字
Thread Local Storage
线程局部存储(tls)是一种机制,通过这一机制分配的变量,每个当前线程有一个该变量的实例.
gcc用于实现tls的运行时模型最初来自于IA-64处理器的ABI,但以后被用到其它处理器上。它需要链接器（ld），动态连接器（ld.so)和系统库（libc.so,libpthread.so)的全力支持.因此它不是到处可用的。
在用户层，用一个新的存储类型关键字：__thread表示这一扩展。例如：

    __thread int i;
    extern __thread struct state s;
    static __thread char *p;

__thread限定符(specifier)可以单独使用，也可带有extern或static限定符，但不能带有其它存储类型的限定符。
__thread可用于全局的静态文件作用域，静态函数作用域或一个类中的静态数据成员。不能用于块作用域，自动或非静态数据成员。

当应用address-of操作符于一个线程局部存储变量时，它被在运行时求值，返回该变量当前线程实例的地址。这样得到的地址可以被其它任何线程使用。当一个线程终止时，任何该线程中的线程局部存储变量都不再有效。静态初始化不会涉及到任何线程局部存储变量的地址。

在c++中，如果一个线程局部存储变量有一个初始化器，它必须是常量表达式。
翻译的不好，直接看英文原版—https://gcc.gnu.org/onlinedocs/gcc-3.4.1/gcc/Thread-Local.html


```C++
#include <iostream>  
#include <pthread.h>  
#include <unistd.h>

using namespace std; 

const int i=5;  
__thread int var=i;//两种方式效果一样  

static __thread int var2 = 15;//  

static void* worker1(void* arg);  
static void* worker2(void* arg);  

int main(){  
    pthread_t pid1,pid2;  

    static __thread  int temp=10;//修饰函数内的static变量  

    pthread_create(&pid1,NULL,worker1,NULL);  
    pthread_create(&pid2,NULL,worker2,NULL);  
    pthread_join(pid1,NULL);  
    pthread_join(pid2,NULL);  

    cout<<"main var :" << ++var<<endl;
    cout<<"main var2 :" << &var<<endl;
    cout<<"main var addr :" << &var<<endl;
    cout<<"main var2 addr :" << &var2<<endl;
    cout<<temp<<endl;//输出10  
    return 0;  
}  

static void* worker1(void* arg){  
    cout<<"worker1 var :" << ++var<<endl;//输出 6  
    cout<<"worker1 var addr :" << &var<<endl;
    cout<<"worker1 var2 :" << ++var2<<endl;   
    cout<<"worker1 var2 addr :" << &var2<<endl;
}  

static void* worker2(void* arg){  
    sleep(1);//等待线程1改变var值，验证是否影响线程2  
    cout<< "worker2 var :" << --var<<endl;
    cout<<"worker2 var addr :" << &var<<endl;

    cout<<"worker2 var2 :" << --var2<<endl;   
    cout<<"worker2 var2 addr :" << &var2<<endl;
}
```

    [james_xie@james-desk myCode]$ g++ threadvar.c -o threadvar -lpthread
    [james_xie@james-desk myCode]$ ./threadvar 
    worker1 var :6
    worker1 var addr :0x7f73a20f86f4
    worker1 var2 :16
    worker1 var2 addr :0x7f73a20f86f8
    worker2 var :4
    worker2 var addr :0x7f73a18f76f4
    worker2 var2 :14
    worker2 var2 addr :0x7f73a18f76f8
    main var addr :0x7f73a30f0734
    main var2 addr :0x7f73a30f0738
    10


上面简单的事例中，总共有三个线程（包括主线程），可以看到通过__thread 修饰的变量，在线程中地址都不一样，__thread变量每一个线程有一份独立实体，各个线程的值互不干扰。

