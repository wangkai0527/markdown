[TOC]

# synchronized

```Java
public class UserManger {
    private  volatile  static   UserManger ourInstance ;

    private UserManger() {

    }

    public static UserManger getInstance(){
        if (ourInstance == null) {
            synchronized (UserManger.class) {
                if (ourInstance == null) {
                    ourInstance = new UserManger();
                }
            }
        }
        return ourInstance;
    }

    public void getPerson() {
        Person person = new Person();
        synchronized (person) {
            person.hashCode();
        }
    }
}
```


## 锁的底层实现

```Java
public class Demo {
    public void foo() {
        int a = 100;
    }
}


public class Demo {
    public synchronized void foo() {
        int a = 100;
    }
}
```
Demo.java-->Demo.class---->dex文件

dex查看命令

    dx --dex --verbose --dump-to=Demo.dex.txt --dump-method=Demo.foo --verbose-dump Demo.class


Demo.dex.txt 字节码指令
```
Demo.foo:()V:
regs: 0002; ins: 0001; outs: 0000
  0000: const/16 v0, #int 100 // #0064
  0002: return-void
  0003: code-address

  debug info
    line_start: 3
    parameters_size: 0000
    0000: prologue end
    0000: line 3
    0002: line 4
    0002: +local v0 a int
    end sequence
  source file: "Demo.java"


Demo.foo:()V:
regs: 0002; ins: 0001; outs: 0000
  0000: monitor-enter v1
  0001: const/16 v0, #int 100 // #0064
  0003: monitor-exit v1
  0004: return-void
 
  debug info
    line_start: 4
    parameters_size: 0000
    0000: prologue end
    0000: line 4
    0003: line 5
    0003: +local v0 a int
    end sequence
  source file: "Demo.java"
```

```C++
hotspot/src/share/vm/interpreter/interpreterRuntime.cpp
hotspot/src/share/vm/interpreter/interpreterRuntime.hpp

IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorenter(JavaThread* thread, BasicObjectLock* elem))

IRT_ENTRY_NO_ASYNC(void, InterpreterRuntime::monitorexit(JavaThread* thread, BasicObjectLock* elem))

```

​在理解锁实现原理之前先了解一下Java的对象头和 Monitor，在JVM中，对象是分成**三部分**存在的：

> 对象头 
> 实例数据 
> 对其填充

![](对象实例.jpg)

> 实例数据和对其填充与synchronized无关，。

### Java对象在对内存分布

​实例数据存放类的属性数据信息，包括父类的属性信息，如果是数组的实例部分还包括数组的长度，这部分内存按4字节对齐；对其填充不是必须部分，由于虚拟机要求对象起始地址必须是8字节的整数倍，

**对齐填充**仅仅是为了使字节对齐。

​对象头是我们需要关注的重点，它是synchronized实现锁的基础，因为synchronized申请锁、上锁、释放锁都与对象头有关。

​对象头主要结构是由**Mark Word** 和 **Class Metadata Address** 组成，其中Mark Word存储对象的hashCode、锁信息或分代年龄或GC标志等信息，Class Metadata Address是类型指针指向对象的类元数据，JVM通过该指针确定该对象是哪个类的实例。

### 锁也分不同状态

​JDK6之前只有两个状态：**无锁、有锁（重量级锁）**，

而在JDK6之后对synchronized进行了优化，新增了两种状态，总共就是四个状态：

> 1. 无锁状态、
> 2. 偏向锁、
> 3. 轻量级锁、
> 4. 重量级锁，

​其中无锁就是一种状态了。锁的类型和状态在对象头Mark Word中都有记录，在申请锁、锁升级等过程中JVM都需要读取对象的Mark Word数据。

​每一个锁都对应一个monitor对象，在HotSpot虚拟机中它是由ObjectMonitor实现的（C++实现）。每个对象都存在着一个monitor与之关联，对象与其monitor之间的关系有存在多种实现方式，如monitor可以与对象一起创建销毁或当线程试图获取对象锁时自动生成，但当一个monitor被某个线程持有后，它便处于锁定状态。

```ObjectMonitor() {
    _header       = NULL;
    _count        = 0;  //锁计数器
    _waiters      = 0,
    _recursions   = 0;
    _object       = NULL;
    _owner        = NULL;
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ;
    FreeNext      = NULL ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
  } 
```

### ObjectMonitor 工作机制

ObjectMonitor中有两个队列

> **_WaitSet**     等待锁的线程集合
>
> **_EntryList**，进入锁范围的线程集合

### 监视器工作流程

1. 用来保存ObjectWaiter对象列表(每个等待锁的线程都会被封装ObjectWaiter对象)，
2. owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时， 首先会进入**_EntryList 集合**
3. 当线程获取到对象的monitor 后进入 _Owner 区域并把monitor中的owner变量设置为当前线程同时monitor中的计数器count加1，
4. 若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSe t集合中等待被唤醒。
5. 若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。

>   monitor对象存在于每**个Java对象的对象头**中(存储的指针的指向)，





### synchronized 原理

#### 为什么Java中任意对象可以作为锁？

​因为 synchronized锁便是通过这种方式获取锁的，也是为什么Java中任意对象可以作为锁的原因，同时也是notify/notifyAll/wait等方法存在于顶级对象Object中的原因(关于这点稍后还会进行分析)

#### JVM对synchronized的优化

​从最近几个jdk版本中可以看出，Java的开发团队一直在对synchronized优化，其中最大的一次优化就是在jdk6的时候，新增了两个锁状态，通过锁消除、锁粗化、自旋锁等方法使用各种场景，给synchronized性能带来了很大的提升。

#### 锁膨胀

上面讲到锁有四种状态，并且会因实际情况进行膨胀升级，其膨胀方向是：

**无锁**——>**偏向锁**——>**轻量级锁**——>**重量级锁**，并且膨胀方向不可逆。

![img](锁状态.png)

![img](对象头结构1.png)

![img](对象头结构2.png)





#### 偏向锁

一句话总结它的作用：减少统一线程获取锁的代价。在大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得，那么此时就是偏向锁。

核心思想：
如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word的结构也就变为偏向锁结构，当该线程再次请求锁时，无需再做任何同步操作，即获取锁的过程只需要检查Mark Word的锁标记位为偏向锁以及当前线程ID等于Mark Word的ThreadID即可，这样就省去了大量有关锁申请的操作。

**过程:**  
先说无锁—>偏向锁。锁的标志位都为01。
当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要花费CAS操作来加锁和解锁，而只需简单的测试一下对象头的Mark Word里是否存储着指向当前线程的偏向锁，如果测试成功，表示线程已经获得了锁，如果测试失败，则需要再测试下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁），如果没有设置，则使用CAS竞争锁，如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。

​要注意的是，偏向锁使用了一种**等到竞争出现**才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。**偏向锁的撤销**，需要等待全局安全点（在这个时间点上没有字节码正在执行），它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，**如果线程不处于活动状态**，则将对象头设置成无锁状态，如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的Mark Word要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。下图中的线程1演示了偏向锁初始化的流程， 

### 轻量级锁
轻量级锁是由偏向锁升级而来，当存在第二个线程申请同一个锁对象时，偏向锁就会立即升级为轻量级锁。注意这里的第二个线程只是申请锁，不存在两个线程同时竞争锁，可以是一前一后地交替执行同步块。

### 重量级锁
重量级锁是由轻量级锁升级而来，当同一时间有多个线程竞争锁时，锁就会被升级成重量级锁，此时其申请锁带来的开销也就变大。

重量级锁一般使用场景会在追求吞吐量，同步块或者同步方法执行时间较长的场景。

### 锁消除
**消除锁是虚拟机另外一种锁的优化，这种优化更彻底，在JIT编译时，对运行上下文进行扫描，去除不可能存在竞争的锁。**比如下面代码的method1和method2的执行效率是一样的，因为object锁是私有变量，不存在所得竞争关系。



### 锁粗化
**锁粗化是虚拟机对另一种极端情况的优化处理，通过扩大锁的范围，避免反复加锁和释放锁。**
比如下面method3经过锁粗化优化之后就和method4执行效率一样了。



### 自旋锁与自适应自旋锁
轻量级锁失败后，虚拟机为了避免线程真实地在操作系统层面挂起，还会进行一项称为自旋锁的优化手段。

**自旋锁：**许多情况下，共享数据的锁定状态持续时间较短，切换线程不值得，通过让线程执行循环等待锁的释放，不让出CPU。
如果得到锁，就顺利进入临界区。如果还不能获得锁，那就会将线程在操作系统层面挂起，这就是自旋锁的优化方式。
但是它也存在缺点：如果锁被其他线程长时间占用，一直不释放CPU，会带来许多的性能开销。

自适应自旋锁：这种相当于是对上面自旋锁优化方式的进一步优化，它的自旋的次数不再固定，其自旋的次数由前一次在同一个锁上的自旋时间及锁的拥有者的状态来决定，这就解决了自旋锁带来的缺点。
 