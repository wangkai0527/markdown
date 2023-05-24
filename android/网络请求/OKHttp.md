[TOC]

# OKHttp

http协议
第一行：请求行
请求行下面的：请求属性集


选择网络访问框架的时候，为什么要选择OkHttp而不是其他框架；
明确一点：并不期待，你将市面上所有的框架都全部搞得非常清楚，优缺点全部列出来；你是否具备掌控网络访问框架的能力；
这个问题没有标准答案，最好是带点主观意识；
OkHttp
XUtil      支持网络请求，图片加载，甚至还能操作数据库；就我个人而言，我认为，一个好的网络访问框架应该只专注一件事
Retrofit  肯定这个框架不错，它封装了OkHttp，所以我暂时没有去深入了解它，
Volley  官方出品，官方介绍适合小中型app，接口比较多，访问量比较大；基于HttpUrlConnection封装，（HttpUrl。。。 android 2.3以下api）
就我个人而言，我更希望能够更加深入的去了解网络访问框架

OkHttp基于Socket通信，它更倾向于底层，会对Http协议进行完全的封装，我在学习这个框架的时候，可以更加底层的了解；我相信只要我能搞定okhttp，那么其他的
访问框架，都很容易懂；

OkHttp中为什么使用构建者模式？
使用多个简单的对象一步一步构建成一个复杂的对象；
优点：当内部数据过于复杂的时候，可以非常方便的构建出我们想要的对象，并且不是所有的参数我们都需要进行传递；
缺点：代码会有冗余


怎么设计一个自己的网络访问框架，为什么这么设计？
我目前还没有正式设计过网络访问框架，
如果是我自己设计的话，我会从以下两个方面考虑
1：先参考现有的框架，找一个比较合适的框架作为启动点，比如说，基于上面讲到的okhttp的优点，选择okhttp的源码进行阅读，并且将主线的流程抽取出，为什么这么做，因为okhttp里面虽然涉及到了很多的内容，但是我们用到的内容并不是特别多；保证先能运行起来一个基本的框架；
2：考虑拓展，有了基本框架之后，我会按照我目前在项目中遇到的一些需求或者网路方面的问题，看看能不能基于我这个框架进行优化，比如服务器它设置的缓存策略，
我应该如何去编写客户端的缓存策略去对应服务器的，还比如说，可能刚刚去建立基本的框架时，不会考虑HTTPS的问题，那么也会基于后来都要求https，进行拓展；

为什么要基于Okhttp，就是因为它是基于Socket，从我个人角度讲，如果能更底层的深入了解相关知识，这对我未来的技术有很大的帮助；




okhttp流程 原理 关键类

okhttp 关键类
OKHttpClient Request Call-RealCall.enqueue
synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }

synchronized void enqueue(AsyncCall call) {
当运行的队列中的数值小于64， 并且同时访问同一个机器目标HOST请求书小于5
直接加入到运行队列
不然的话就加入到等待队列
    if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      runningAsyncCalls.add(call);
      executorService().execute(call);
    } else {
      readyAsyncCalls.add(call);
    }
  }

Response response = getResponseWithInterceptorChain();
获取响应的数据







1: 先搞懂责任链是个啥？基于责任链搞清楚reponse  okhttp
2：搞清楚拦截器，
  a 重试/重定向：
  b 桥拦截器：封装header属性 host keep-live gzip    
     header 进行基本设置，
  c 缓存拦截器
  d 连接拦截器
  e CallServerInterceptor 

 executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
SynchronousQueue<Runnable> 这个参数是线程池等待队列，
 
1：核心线程数 保持在线程池中的线程数量
2：线程池最大可容纳的线程数  
3/4参数：当线程池中的线程数量大于核心线程数，空闲线程就会等待60s才会被终止，如果小于就会立刻停止；
SynchronousQueue

okhttp完全流程
1：OkHttpClient okHttpClient = new OkHttpClient.Builder()
构建一个okhttpClient对象，传入你想传入的对象，不传就是默认的；
2：构建request对象
Request request = new Request.Builder()  
3：okHttpClient.newCall  实际上返回的realCall类 继续调用RealCall.newRealCall
4：调用enqueue方法，传入我们需要的回调接口，而且会判断，
synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
如果当前这个call对象已经被运行的话，则抛出异常；
5：继续调用dispatcher的enqueue方法，如果当前运行队列<64并且正在运行，访问同一个服务器地址的请求<5
就直接添加到运行队列，并且开始运行；
不然就添加到等待队列；
6：运行AsyncCall，调用它的execute方法
7：在execute方法中处理完response之后，会在finally中调用dispathcer的finished方法；
8：当当前已经处理完毕的call从运行队列中移除掉；并且调用promoteCalls方法
9：promoteCalls方法中进行判断，
如果运行队列数目大于等于64，如果等待队列里啥都没有，也直接return？
循环等待队列，
将等待队列中的数据进行移除，移除是根据运行队列中还能存放多少来决定；
移到了运行队列中，并且开始运行；



如何考虑app的安全性
1：使用https协议进行交互
2：数据交互时，根据业务分出哪些是敏感信息，凡是敏感信息使用对称加密方式，如果是类似密码的，则使用不可逆的加密方式；md5
3：考虑跟钱相关，或者同等重要的数据接口，需要做多重验证，比如：前端加密请求参数，合并请求参数生成MD5码，服务器端做多重认证，最好能对比本地数据库或者缓存之类的信息；
4：混淆，
5: app加固，dex文件进行加密，这种方式，可以通过”内存下载“，不安全，也只是为了增加破解难度；
6：将加密算法，一些核心数据添加到so文件中；








