[TOC]


# Socket

[socket send和recv正确用法](https://blog.csdn.net/XiaoWhy/article/details/105122044)

## 如何解决windows.h和WinSocket2.h的冲突
原因是windows.h头文件中已经包含了很多关于socket的宏定义，但是当我们引入winsocket2.h头文件的时候呢，这些宏定义已经跟之前的不一样了，也就产生了冲突。

两种解决办法：
1. 将winsocket2头文件的引入放在windows头文件之前。

2. 如果我们的项目工程很大的话，不能确保这两个头文件的引入顺序一定是正确的，检查起来也比较麻烦。
#define WIN32_LEAN_AND_MEAN
这个宏是用来减少我们不常用到的宏定义，来起到提高编译速度的。


## Linux下常用的C/C++开源Socket库
单谈网络通讯模块，我们可能既手工封装操作系统api，又使用第三方库。
在unix/linux系统上，直接调用socket()系列。
在windows上，直接调用winsock2提供的API。
使用MFC开发的，可能直接使用MFC提供的那个框架。
使用了boost库的，可能喜欢使用那个异步Asio库。
还有著名的ACE库，以及类似的跨平台库。

1. Linux Socket Programming In C++ : http://tldp.org/LDP/LG/issue74/tougher.html

2. ACE: http://www.cs.wustl.edu/~schmidt/ACE.html
ACE采用ACE_OS适配层屏蔽各种不同的、复杂繁琐的操作系统API。

ACE是一个大型的中间件产品，代码20万行左右，过于宏大，一堆的设计模式，架构了一层又一层。它庞大、复杂，适合大型项目。开源、免费，不依赖第三方库。使用的时候，要根据情况，看你从哪一层来进行使用。支持跨平台。

ACE超重量级的网络通信开发框架。ACE自适配通信环境（AdaptiveCommunication Environment）是可以自由使用、开放源代码的面向对象框架，在其中实现了许多用于并发通信软件的核心模式。ACE提供了一组丰富的可复用C++包装外观（Wrapper Facade）和框架组件，可跨越多种平台完成通用的通信软件任务，其中包括：事件多路分离和事件处理器分派、信号处理、服务初始化、进程间通信、共享内存管理、消息路由、分布式服务动态（重）配置、并发执行和同步，等等

3. C++ Sockets Library: http://www.alhem.net/Sockets/index.html
它是一个跨平台的Sockets库，实现包括TCP、UDP、ICMP、SCTP协议。已实现的应用协议包括有SMTP、HTTP(S)、Ajp。具有SOCKS客户端实现以及匿名DNS，支持HTTP的GET/POST/PUT以及WebServer的框架。
它封装了sockets C API的C++类库。支持SSL, IPv6, tcp和udp sockets, sctp sockets, http协议, 高度可定制的错误处理。

4. Asio C++ Library: http://think-async.com/
它是一个基于Boost开发的异步IO库，封装了对Socket的常用操作，简化了基于Socket程序的开发。它开源、免费、支持跨平台。

5. libevent: http://libevent.org/
它是一个C语言写的网络库，主要支持的是类Linux 操作系统，最新的版本添加了对Windows的IOCP的支持。由于IOCP是异步IO，与Linux下的POLL模型，EPOLL模型，还有freebsd的KQUEUE等这些同步模型在用法上完全不一致，所以使用方法也不一样，就好比ACE中的Reactor和Proactor模式一样，使用起来需要转变思路。如果对性能没有特别的要求，那么使用libevent中的select模型来实现跨平台的操作，select模型可以横跨Windows，Linux，Unix，Solaris等系统。

Libevent是一个轻量级的开源高性能网络库，它的机制是采用事件触发，封装了以下三种事件的响应:IO事件,定时器事件,信号事件。select模型来实现跨平台的操作，Windows环境下支持IOCP。Google的开源WEB浏览器Chromium在Mac和Linux版本中，也使用了Libevent，足见该库的质量。

6. libev: http://software.schmorp.de/pkg/libev.html
它是一个C语言写的，只支持Linux系统的库，以前的时候只封装了EPOLL模型.使用方法类似libevent，但是非常简洁，代码量是最少的一个库，也就几千行代码。显然这样的代码跨平台肯定是无法支持的了，如果你只需要在Linux下面运行，那用这个库也是可以的。

libev和libevent很像，按照作者的介绍，可以作为libevent的替代者，能够提供更高的性能。libev是一个高性能事件循环，所实现的功能就是一个强大的reactor。

7. SimpleSocket: http://home.kpn.nl/lcbokkers/simsock.htm
这个类库让编写基于Socket的客户/服务器程序更加容易。

8. simple-socket: http://sourceforge.net/projects/simple-socket/
An easy to use C++ socket andnetwork library, mainly for UNIX systems.

9. POCO: http://pocoproject.org/
POCO C++ Libraries提供一套C++的类库用以开发基于网络的可移植的应用程序，功能涉及线程、线程同步、文件系统访问、流操作、共享库和类加载、套接字以及网络协议包括：HTTP、FTP、SMTP等；其本身还包含一个HTTP服务器，提供XML的解析和SQL数据库的访问接口。POCO库的模块化、高效的设计及实现使得POCO特别适合嵌入式开发。在嵌入式开发领域，由于C++既适合底层（设备I/O、中断处理等）和高层面向对象开发，越来越流行。

10. libcurl: http://curl.haxx.se/libcurl/
libcurl是免费的轻量级的客户端网络库，支持DICT, FILE, FTP, FTPS, Gopher, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS,POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMTP, SMTPS, Telnet, TFTP.支持SSL, HTTPPOST,HTTPPUT, FTP上传, HTTP form上传，代理，cookies, 用户名与密码认证。

如果你开发的是客户端，libcurl是一个不错的选择。

11. libiop: http://sourceforge.net/projects/libiop/
一个c语言开发的跨平台网络IO库。

功能特性：c/c++api, 底层支持epoll, select,poll等io模型；异步事件模型；任务池模型，跨平台线程接口；跨平台(Linux/windows)；日志服务；稳定，支持7*24小时无间断运行，自动处理异常状态；高并发与快速响应；API简洁，学习成本底。



## JAVA 通过 Socket 实现 TCP 编程

https://blog.csdn.net/qq_23473123/article/details/51461894?spm=1001.2101.3001.6650.6&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-51461894-blog-7342991.pc_relevant_3mothn_strategy_recovery&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EBlogCommendFromBaidu%7ERate-6-51461894-blog-7342991.pc_relevant_3mothn_strategy_recovery&utm_relevant_index=7


## PC端通过USB线与Android设备通信

通过 Socket 连接通信，将 PC 端作为客户端，Android 设备作为服务端
127.0.0.1 为设备本地地址

`adb forward tcp:8000 tcp:9000`
将 PC 的8000端口的数据发送到 Android 设备9000端口
执行该指令之后才能通信
注意，该指令只能由 PC 端执行，故 Socket 方式不支持两台 Android 设备间通过 USB 通信



PC
```java
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.net.Socket;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ScheduledFuture;
import java.util.concurrent.ScheduledThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.logging.Logger;
 
public class SocketTest {
	private ScheduledThreadPoolExecutor mScheduledThreadPoolExecutor = null;
	private Runnable mRunnable = null;
	private ScheduledFuture<?> mScheduledFuture = null;
	private Socket mSocket = null;
 
	private Logger mLogger = Logger.getLogger(SocketTest.class.getName());
 
	private void session() {
		DataInputStream dis = null;
		DataOutputStream dos = null;
		try {
			dis = new DataInputStream(mSocket.getInputStream());
			dos = new DataOutputStream(mSocket.getOutputStream());
 
			while (true) {
				String data = "PC时间:" + System.currentTimeMillis();
				dos.writeUTF(data);
				dos.flush();
 
				String s = dis.readUTF();
				mLogger.info("收到数据:" + s);
 
				Thread.sleep(5000);
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			try {
				mSocket.close();
			} catch (Exception e) {
				e.printStackTrace();
			}
 
			mSocket = null;
		}
	}
 
	public SocketTest() {
		mScheduledThreadPoolExecutor = new ScheduledThreadPoolExecutor(1);
 
		mRunnable = new Runnable() {
			@Override
			public void run() {
				if (mSocket == null || !mSocket.isConnected()) {
					mLogger.info("尝试建立连接...");
					try {
						mSocket = new Socket("localhost", 18000);
						mLogger.info("建立新连接:" + mSocket.toString());
 
						CompletableFuture.runAsync(new Runnable() {
							@Override
							public void run() {
								session();
							}
						});
					} catch (Exception e) {
						mLogger.info("连接异常");
					}
				} else {
					mLogger.info("连接心跳检测:当前已经建立连接，无需重连");
				}
			}
		};
 
		// 每隔3秒周期性的执行心跳检测动作。
		mScheduledFuture = mScheduledThreadPoolExecutor.scheduleAtFixedRate(mRunnable, 0, 3, TimeUnit.SECONDS);
	}
 
	public static void main(String[] args) {
		new SocketTest();
	}
}
```

Android
```java
package zhangphil.socket;
 
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
 
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
 
 
public class MainActivity extends AppCompatActivity {
    private String TAG = "Android端";
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        new ServerThread().start();
    }
 
    private class ServerThread extends Thread {
 
        @Override
        public void run() {
            ServerSocket serverSocket = null;
            try {
                serverSocket = new ServerSocket(19000);
                while (true) {
                    Socket socket = serverSocket.accept();
                    Log.d(TAG, "接受连接");
 
                    new ClientThread(socket).start();
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (serverSocket != null) {
                    try {
                        serverSocket.close();
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
 
    private class ClientThread extends Thread {
        private Socket socket;
 
        public ClientThread(Socket socket) {
            this.socket = socket;
            Log.d(TAG, "当前Socket:" + socket.toString());
        }
 
        @Override
        public void run() {
            try {
                DataInputStream dis = new DataInputStream(socket.getInputStream());
                DataOutputStream dos = new DataOutputStream(socket.getOutputStream());
 
                while (true) {
 
                    String data = dis.readUTF();
                    Log.d(TAG, "收到数据:" + data);
 
                    //回写给客户端。
                    String s = "手机时间:" + System.currentTimeMillis();
                    dos.writeUTF(s);
                    dos.flush();
                }
 
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    socket.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```




-----------------------------------------------------------------------
socket.error: [Errno 98] Address already in use

当该端口被其他进程占用时，修改一个未使用的端口号重新运行
执行`netstat -nlp | grep :端口号`，查看当前使用该端口的进程，得到进程号
`sudo kill 进程号`
当一个进程持续杀不死，杀死之后换一个PID继续占用该端口时，可能是有一个父进程持续生成占用该端口的子进程。这时，采用以下方法：
执行 `ps -ef|grep 子进程号`，根据返回信息获取父进程号，杀死该父进程。

-----------------------------------------------------------------------
socket error：[Errno 111]Connection refused

1.确保服务端在相应的端口监听；
2.关闭防火墙（ubuntu下面的命令：sudo ufw disable）;
3.而且server端要 sudo 运行；


-----------------------------------------------------------------------





