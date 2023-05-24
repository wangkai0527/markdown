[TOC]

# Zygote


## Zygote 是什么？

在Android中，负责孵化新进程的这个进程叫做Zygote，安卓上其他的应用进程都是由它孵化的。
众所周知，安卓是Linux内核，安卓系统上运行的一切程序都是放在Dalvik虚拟机上的，Zygote也不例外，事实上，它是安卓运行的第一个Dalvik虚拟机进程。
既然Zygote负责孵化其他的安卓进程，那么它自己是由谁孵化的呢？既然Android是基于Linux内核，那么Zygote当然就是Linux内核启动的用户级进程Init创建的了。


## Zygote 的作用是什么？
对于 Zygote 的作用实际上可以概括为以下两点：
1. 创建 SystemServer 进程
2. 孵化应用进程


## Zygote 的启动过程

Zygote进程在Init进程启动过程中被以service服务的形式启动：
```
service zygote /system/bin/app_process -Xzygote /system/bin --zygote --start-system-server
class main
socket zygote stream 660 root system 
```

调用 app_main.cpp 的 main 函数中的 AppRuntime 的 start 方法来启动 Zygote 进程

调用 startVm 函数来创建虚拟机，调用 startReg 函数为java虚拟机注册JNI方法

通过 toSlashClassName 找到 ZygoteInit ，通过 GetStaticMethedID 函数找到 main 方法然后调用，ZygoteInit 的 main 方法是由Java语言编写的，当前的运行逻辑在Native中，这就需要JNI来调用Java，这样 Zygote 就从Native层进入了Java框架层。

frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
```Java
    public static void main(String argv[]) {
       //1、创建ZygoteServer
        ZygoteServer zygoteServer = new ZygoteServer();
      .......
        try {
             .......
            boolean startSystemServer = false;
            String socketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
           // 2、解析app_main.cpp传来的参数
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    socketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[I]);
                }
            }

            if (abiList == null) {
                throw new RuntimeException("No ABI list supplied.");
            }
            //3、创建一个Server端的Socket
            zygoteServer.registerServerSocket(socketName);
            // In some configurations, we avoid preloading resources and classes eagerly.
            // In such cases, we will preload things prior to our first fork.
            if (!enableLazyPreload) {
                bootTimingsTraceLog.traceBegin("ZygotePreload");
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_START,
                    SystemClock.uptimeMillis());
               //4、加载进程的资源和类
                preload(bootTimingsTraceLog);
                EventLog.writeEvent(LOG_BOOT_PROGRESS_PRELOAD_END,
                    SystemClock.uptimeMillis());
                bootTimingsTraceLog.traceEnd(); // ZygotePreload
            } else {
                Zygote.resetNicePriority();
            }
              ........
            if (startSystemServer) {
                //5、开启SystemServer进程，这是受精卵进程的第一次分裂
                startSystemServer(abiList, socketName, zygoteServer);
            }

            Log.i(TAG, "Accepting command socket connections");
           //6、启动一个死循环监听来自Client端的消息
            zygoteServer.runSelectLoop(abiList);
           //7、关闭SystemServer的Socket
            zygoteServer.closeServerSocket();
        } catch (Zygote.MethodAndArgsCaller caller) {
           //8、这里捕获这个异常调用MethodAndArgsCaller的run方法。
            caller.run();
        } catch (Throwable ex) {
            Log.e(TAG, "System zygote died with exception", ex);
            zygoteServer.closeServerSocket();
            throw ex;
        }
    }
```


总结下来的流程：
1. 通过 registerServerSocket 方法来创建一个Server端的socket，这个name为 zygote 的socket用于等待ActivityManagerService 请求 Zygote 来创建新的应用程序进程
2. 预加载，预加载项如下：
```Java
preloadClasses();
preloadResources();
preloadOpenGL();
preloadSharedLibraries();
preloadTextResources();
WebViewFactory.prepareWebViewInZygote();
...
```

以下是Android应用进程共享内存图：
![](Android应用进程共享内存.awebp)

通过上图可以很容易理解在Zygote进程预加载系统资源后，然后通过它孵化出其他的虚拟机进程，进而共享虚拟机内存和框架层资源，这样大幅度提高应用程序的启动和运行速度。
3. 启动 SystemServer 进程
4. 执行 runSelectLoop() 方法等待消息去创建应用进程


## Zygote注意细节
Zygote进行fork的时候要是单线程，为了避免造成死锁或者状态不一致等问题
Zygote的跨进程通信没有采用Binder机制，而是采用本地socket


## 孵化应用进程这种事为什么不交给SystemServer来做，而专门设计一个Zygote？

我们知道，应用在启动的时候需要做很多准备工作，包括启动虚拟机，加载各类系统资源等等，这些都是非常耗时的，如果能在zygote里就给这些必要的初始化工作做好，子进程在fork的时候就能直接共享，那么这样的话效率就会非常高。
这个就是zygote存在的价值，这一点呢SystemServer是替代不了的，主要是因为SystemServer里跑了一堆系统服务，这些是不能继承到应用进程的。
而且我们应用进程在启动的时候，内存空间除了必要的资源外，最好是干干净净的，不要继承一堆乱七八糟的东西。
所以呢，不如给SystemServer和应用进程里都要用到的资源抽出来单独放在一个进程里，也就是这的zygote进程，然后zygote进程再分别孵化出SystemServer进程和应用进程。孵化出来之后，SystemServer进程和应用进程就可以各干各的事了。


## Zygote的IPC通信机制为什么不采用binder？如果采用binder的话会有什么问题么？
第一个原因，我们可以设想一下采用binder调用的话该怎么做，首先zygote要启用binder机制，需要打开binder驱动，获得一个描述符，再通过mmap进行内存映射，还要注册binder线程，这还不够，还要创建一个binder对象注册到serviceManager，另外AMS要向zygote发起创建应用进程请求的话，要先从serviceManager查询zygote的binder对象，然后再发起binder调用，这来来回回好几趟非常繁琐，
相比之下，zygote和SystemServer进程本来就是父子关系，对于简单的消息通信，用管道或者socket非常方便省事。

第二个原因，如果zygote启用binder机制，再fork出SystemServer，那么SystemServer就会继承了zygote的描述符以及映射的内存，这两个进程在binder驱动层就会共用一套数据结构，这显然是不行的，所以还得先给原来的旧的描述符关掉，再重新启用一遍binder机制，这个就是自找麻烦了。


Zygote工作流程图：
![](Zygote流程图.awebp)






## Zygote 源码分析
AOSP/android10/system/core/rootdir/init.rc
```
import /init.environ.rc
import /system/etc/init/hw/init.usb.rc
import /init.${ro.hardware}.rc
import /vendor/etc/init/hw/init.${ro.hardware}.rc
import /system/etc/init/hw/init.usb.configfs.rc
import /system/etc/init/hw/init.${ro.zygote}.rc

    start ueventd

    # Start logd before any other services run to ensure we capture all of their logs.
    start logd
    # Start lmkd before any other services run so that it can register them
    chown root system /sys/module/lowmemorykiller/parameters/adj
    chmod 0664 /sys/module/lowmemorykiller/parameters/adj
    chown root system /sys/module/lowmemorykiller/parameters/minfree
    chmod 0664 /sys/module/lowmemorykiller/parameters/minfree
    start lmkd

    # Start essential services.
    start servicemanager
    start hwservicemanager
    start vndservicemanager
```
start 会去 /system/bin 目录下找环境变量，启动相应服务

AOSP/android10/system/core/rootdir/init.zygote64.rc
```
service zygote /system/bin/app_process64 -Xzygote /system/bin --zygote --start-system-server
    class main
    priority -20
    user root
    group root readproc reserved_disk
    socket zygote stream 660 root system
    socket usap_pool_primary stream 660 root system
    onrestart exec_background - system system -- /system/bin/vdc volume abort_fuse
    onrestart write /sys/power/state on
    onrestart restart audioserver
    onrestart restart cameraserver
    onrestart restart media
    onrestart restart netd
    onrestart restart wificond
    writepid /dev/cpuset/foreground/tasks
```



AOSP/android10/frameworks/base/cmds/app_process/app_main.cpp
```C++
namespace android {
class AppRuntime : public AndroidRuntime
{
public:
public:
    AppRuntime(char* argBlockStart, const size_t argBlockLength)
        : AndroidRuntime(argBlockStart, argBlockLength)
        , mClass(NULL)
    {
    }

    //外部调用，className 传进来值是 ActivityThread 
    void setClassNameAndArgs(const String8& className, int argc, char * const *argv) {
        mClassName = className;
        for (int i = 0; i < argc; ++i) {
             mArgs.add(String8(argv[i]));
        }
    }

    virtual void onVmCreated(JNIEnv* env)
    {
        if (mClassName.isEmpty()) {
            return; // Zygote. Nothing to do here.
        }

        /*
         * This is a little awkward because the JNI FindClass call uses the
         * class loader associated with the native method we're executing in.
         * If called in onStarted (from RuntimeInit.finishInit because we're
         * launching "am", for example), FindClass would see that we're calling
         * from a boot class' native method, and so wouldn't look for the class
         * we're trying to look up in CLASSPATH. Unfortunately it needs to,
         * because the "am" classes are not boot classes.
         *
         * The easiest fix is to call FindClass here, early on before we start
         * executing boot class Java code and thereby deny ourselves access to
         * non-boot classes.
         */
        char* slashClassName = toSlashClassName(mClassName.string());
        mClass = env->FindClass(slashClassName);
        if (mClass == NULL) {
            ALOGE("ERROR: could not find class '%s'\n", mClassName.string());
        }
        free(slashClassName);

        mClass = reinterpret_cast<jclass>(env->NewGlobalRef(mClass));
    }

    virtual void onStarted()
    {
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        proc->startThreadPool();

        //调用 ActivityThread 的 main() 函数，并阻塞
        //父进程是 Zygote 
        AndroidRuntime* ar = AndroidRuntime::getRuntime();
        ar->callMain(mClassName, mClass, mArgs);

        //APP应用退出，ActivityThread 才退出
        IPCThreadState::self()->stopProcess();
        hardware::IPCThreadState::self()->stopProcess();
    }

    virtual void onZygoteInit()
    {
        //启动APP应用时，把进程加入线程池。因为需要进程间通信。Binder 是多线程的。
        sp<ProcessState> proc = ProcessState::self();
        ALOGV("App process: starting thread pool.\n");
        proc->startThreadPool();
    }

    virtual void onExit(int code)
    {
        if (mClassName.isEmpty()) {
            // if zygote
            IPCThreadState::self()->stopProcess();
            hardware::IPCThreadState::self()->stopProcess();
        }

        AndroidRuntime::onExit(code);
    }


    String8 mClassName;
    Vector<String8> mArgs;
    jclass mClass;
};
}

using namespace android;

#if defined(__LP64__)
static const char ABI_LIST_PROPERTY[] = "ro.product.cpu.abilist64";
static const char ZYGOTE_NICE_NAME[] = "zygote64";
#else
static const char ABI_LIST_PROPERTY[] = "ro.product.cpu.abilist32";
static const char ZYGOTE_NICE_NAME[] = "zygote";
#endif

int main(int argc, char* const argv[])
{
    AppRuntime runtime(argv[0], computeArgBlockSize(argc, argv));

    // Parse runtime arguments.  Stop at first unrecognized option.
    bool zygote = false;
    bool startSystemServer = false;
    bool application = false;
    String8 niceName;
    String8 className;

    ++i;  // Skip unused "parent dir" argument.
    while (i < argc) {
        const char* arg = argv[i++];
        if (strcmp(arg, "--zygote") == 0) {
            zygote = true;
            niceName = ZYGOTE_NICE_NAME;
        } else if (strcmp(arg, "--start-system-server") == 0) {
            startSystemServer = true;
        } else if (strcmp(arg, "--application") == 0) {
            application = true;
        } else if (strncmp(arg, "--nice-name=", 12) == 0) {
            niceName.setTo(arg + 12);
        } else if (strncmp(arg, "--", 2) != 0) {
            className.setTo(arg);
            break;
        } else {
            --i;
            break;
        }
    }

    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else {
        fprintf(stderr, "Error: no class name or --zygote supplied./n");
        app_usage();
        LOG_ALWAYS_FATAL("app_process: no class name or --zygote supplied.");
    }
}
```

AOSP/android10/frameworks/base/core/jni/AndroidRuntime.cpp
```C++
using namespace android;
using android::base::GetProperty;

namespace android {
/*
 * JNI registration.
 */

int register_com_android_internal_os_RuntimeInit(JNIEnv* env)
{
    const JNINativeMethod methods[] = {
            {"nativeFinishInit", "()V",
             (void*)com_android_internal_os_RuntimeInit_nativeFinishInit},
            {"nativeSetExitWithoutCleanup", "(Z)V",
             (void*)com_android_internal_os_RuntimeInit_nativeSetExitWithoutCleanup},
    };
    return jniRegisterNativeMethods(env, "com/android/internal/os/RuntimeInit",
        methods, NELEM(methods));
}

int register_com_android_internal_os_ZygoteInit_nativeZygoteInit(JNIEnv* env)
{
    const JNINativeMethod methods[] = {
        { "nativeZygoteInit", "()V",
            (void*) com_android_internal_os_ZygoteInit_nativeZygoteInit },
    };
    return jniRegisterNativeMethods(env, "com/android/internal/os/ZygoteInit",
        methods, NELEM(methods));
}

// ----------------------------------------------------------------------

/*static*/ JavaVM* AndroidRuntime::mJavaVM = NULL;

AndroidRuntime::AndroidRuntime(char* argBlockStart, const size_t argBlockLength) :
        mExitWithoutCleanup(false),
        mArgBlockStart(argBlockStart),
        mArgBlockLength(argBlockLength)
{
    init_android_graphics();

    // Pre-allocate enough space to hold a fair number of options.
    mOptions.setCapacity(20);

    assert(gCurRuntime == NULL);        // one per process
    gCurRuntime = this;
}

AndroidRuntime::~AndroidRuntime()
{
}

/*
 * Start the Android runtime.  This involves starting the virtual machine
 * and calling the "static void main(String[] args)" method in the class
 * named by "className".
 *
 * Passes the main function two arguments, the class name and the specified
 * options string.
 */
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote)
{
    /* start the virtual machine */
    JniInvocation jni_invocation;
    jni_invocation.Init(NULL);
    JNIEnv* env;
    if (startVm(&mJavaVM, &env, zygote, primary_zygote) != 0) {
        return;
    }
    onVmCreated(env);

    /*
     * Register android functions.
     */
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives/n");
        return;
    }
}

/*static*/ JavaVM* AndroidRuntime::getJavaVM() {
    return AndroidRuntime::mJavaVM;
}

/*
 * Get the JNIEnv pointer for this thread.
 *
 * Returns NULL if the slot wasn't allocated or populated.
 */
/*static*/ JNIEnv* AndroidRuntime::getJNIEnv()
{
    JNIEnv* env;
    JavaVM* vm = AndroidRuntime::getJavaVM();
    assert(vm != NULL);

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK)
        return NULL;
    return env;
}

static const RegJNIRec gRegJNI[] = {
        REG_JNI(register_com_android_internal_os_RuntimeInit),
        REG_JNI(register_com_android_internal_os_ZygoteInit_nativeZygoteInit),
};

/*
 * Register android native functions with the VM.
 */
/*static*/ int AndroidRuntime::startReg(JNIEnv* env)
{
    if (register_jni_procs(gRegJNI, NELEM(gRegJNI), env) < 0) {
        env->PopLocalFrame(NULL);
        return -1;
    }
}

}
```

AOSP/android10/frameworks/base/core/java/com/android/internal/os/ZygoteInit.java
```Java
package com.android.internal.os;

public class ZygoteInit {

    /**
     * The path of a file that contains classes to preload.
     */
    private static final String PRELOADED_CLASSES = "/system/etc/preloaded-classes";

    private static boolean sPreloadComplete;

    static void preload(TimingsTraceLog bootTimingsTraceLog) {
        Log.d(TAG, "begin preload");
        beginPreload();
        preloadClasses();
        cacheNonBootClasspathClassLoaders();
        preloadResources();
        nativePreloadAppProcessHALs();
        maybePreloadGraphicsDriver();
        preloadSharedLibraries();
        preloadTextResources();
        // Ask the WebViewFactory to do any initialization that must run in the zygote process,
        // for memory sharing purposes.
        WebViewFactory.prepareWebViewInZygote();
        endPreload();
        warmUpJcaProviders();
        Log.d(TAG, "end preload");

        sPreloadComplete = true;
    }

    private static void beginPreload() {
        ZygoteHooks.onBeginPreload();
    }

    private static void preloadSharedLibraries() {
        Log.i(TAG, "Preloading shared libraries...");
        System.loadLibrary("android");
        System.loadLibrary("compiler_rt");
        System.loadLibrary("jnigraphics");

        try {
            System.loadLibrary("sfplugin_ccodec");
        } catch (Error | RuntimeException e) {
            // tolerate missing sfplugin_ccodec which is only present on Codec 2 devices
        }
    }

    native private static void nativePreloadAppProcessHALs();

    /**
     * This call loads the graphics driver by making an OpenGL or Vulkan call.  If the driver is
     * not currently in memory it will load and initialize it.  The OpenGL call itself is relatively
     * cheap and pure.  This means that it is a low overhead on the initial call, and is safe and
     * cheap to call later.  Calls after the initial invocation will effectively be no-ops for the
     * system.
     */
    static native void nativePreloadGraphicsDriver();

    private static void maybePreloadGraphicsDriver() {
        if (!SystemProperties.getBoolean(PROPERTY_DISABLE_GRAPHICS_DRIVER_PRELOADING, false)) {
            nativePreloadGraphicsDriver();
        }
    }

    private static void preloadTextResources() {
        Hyphenator.init();
        TextView.preloadFontCache();
    }

    /**
     * Performs Zygote process initialization. Loads and initializes commonly used classes.
     *
     * Most classes only cause a few hundred bytes to be allocated, but a few will allocate a dozen
     * Kbytes (in one case, 500+K).
     */
    private static void preloadClasses() {
        final VMRuntime runtime = VMRuntime.getRuntime();

        InputStream is;
        try {
            is = new FileInputStream(PRELOADED_CLASSES);
        } catch (FileNotFoundException e) {
            Log.e(TAG, "Couldn't find " + PRELOADED_CLASSES + ".");
            return;
        }

        try {
            BufferedReader br =
                    new BufferedReader(new InputStreamReader(is), Zygote.SOCKET_BUFFER_SIZE);

            int count = 0;
            int missingLambdaCount = 0;
            String line;
            while ((line = br.readLine()) != null) {
                // Skip comments and blank lines.
                line = line.trim();
                if (line.startsWith("#") || line.equals("")) {
                    continue;
                }

                try {
                    // Load and explicitly initialize the given class. Use
                    // Class.forName(String, boolean, ClassLoader) to avoid repeated stack lookups
                    // (to derive the caller's class-loader). Use true to force initialization, and
                    // null for the boot classpath class-loader (could as well cache the
                    // class-loader of this class in a variable).
                    Class.forName(line, true, null);
                    count++;
                }
            }
        }
    }


    /**
     * Prepare the arguments and forks for the system server process.
     *
     * @return A {@code Runnable} that provides an entrypoint into system_server code in the child
     * process; {@code null} in the parent.
     */
    private static Runnable forkSystemServer(String abiList, String socketName,
            ZygoteServer zygoteServer) {
        int pid;
        try {
            /* Request to fork the system server process */
            pid = Zygote.forkSystemServer(
                    parsedArgs.mUid, parsedArgs.mGid,
                    parsedArgs.mGids,
                    parsedArgs.mRuntimeFlags,
                    null,
                    parsedArgs.mPermittedCapabilities,
                    parsedArgs.mEffectiveCapabilities);
        }
    }

    /**
     * This is the entry point for a Zygote process.  It creates the Zygote server, loads resources,
     * and handles other tasks related to preparing the process for forking into applications.
     *
     * This process is started with a nice value of -20 (highest priority).  All paths that flow
     * into new processes are required to either set the priority to the default value or terminate
     * before executing any non-system code.  The native side of this occurs in SpecializeCommon,
     * while the Java Language priority is changed in ZygoteInit.handleSystemServerProcess,
     * ZygoteConnection.handleChildProc, and Zygote.usapMain.
     *
     * @param argv  Command line arguments used to specify the Zygote's configuration.
     */
    
    public static void main(String argv[]) {
        ZygoteServer zygoteServer = null;

        try {
            boolean startSystemServer = false;
            String zygoteSocketName = "zygote";
            String abiList = null;
            boolean enableLazyPreload = false;
            for (int i = 1; i < argv.length; i++) {
                if ("start-system-server".equals(argv[i])) {
                    startSystemServer = true;
                } else if ("--enable-lazy-preload".equals(argv[i])) {
                    enableLazyPreload = true;
                } else if (argv[i].startsWith(ABI_LIST_ARG)) {
                    abiList = argv[i].substring(ABI_LIST_ARG.length());
                } else if (argv[i].startsWith(SOCKET_NAME_ARG)) {
                    zygoteSocketName = argv[i].substring(SOCKET_NAME_ARG.length());
                } else {
                    throw new RuntimeException("Unknown command line argument: " + argv[i]);
                }
            }

            // In some configurations, we avoid preloading resources and classes eagerly.
            // In such cases, we will preload things prior to our first fork.
            if (!enableLazyPreload) {
                preload(bootTimingsTraceLog);
            }

            zygoteServer = new ZygoteServer(isPrimaryZygote);

            //创建 SystemServer 进程
            if (startSystemServer) {
                Runnable r = forkSystemServer(abiList, zygoteSocketName, zygoteServer);

                // {@code r == null} in the parent (zygote) process, and {@code r != null} in the
                // child (system_server) process.
                if (r != null) {
                    r.run();
                    return;
                }
            }
        }
    }
}
```

Socket Server 端

AOSP/android10/frameworks/base/core/java/com/android/internal/os/ZygoteServer.java
```Java
/**
 * Server socket class for zygote processes.
 *
 * Provides functions to wait for commands on a UNIX domain socket, and fork
 * off child processes that inherit the initial state of the VM.%
 */
class ZygoteServer {

    /**
     * Listening socket that accepts new server connections.
     */
    private LocalServerSocket mZygoteSocket;
    /**
     * The name of the unspecialized app process pool socket to use if the USAP pool is enabled.
     */
    private final LocalServerSocket mUsapPoolSocket;

    /**
     * File descriptor used for communication between the signal handler and the ZygoteServer poll
     * loop.
     * */
    private final FileDescriptor mUsapPoolEventFD;

    /**
     * Initialize the Zygote server with the Zygote server socket, USAP pool server socket, and USAP
     * pool event FD.
     *
     * @param isPrimaryZygote  If this is the primary Zygote or not.
     */
    ZygoteServer(boolean isPrimaryZygote) {
        mUsapPoolEventFD = Zygote.getUsapPoolEventFD();

        if (isPrimaryZygote) {
            mZygoteSocket = Zygote.createManagedSocketFromInitSocket(Zygote.PRIMARY_SOCKET_NAME);
            mUsapPoolSocket =
                    Zygote.createManagedSocketFromInitSocket(
                            Zygote.USAP_POOL_PRIMARY_SOCKET_NAME);
        } else {
            mZygoteSocket = Zygote.createManagedSocketFromInitSocket(Zygote.SECONDARY_SOCKET_NAME);
            mUsapPoolSocket =
                    Zygote.createManagedSocketFromInitSocket(
                            Zygote.USAP_POOL_SECONDARY_SOCKET_NAME);
        }

        mUsapPoolSupported = true;
        fetchUsapPoolPolicyProps();
    }
}
```


AOSP/android10/frameworks/base/core/java/com/android/internal/os/Zygote.java
```Java
public final class Zygote {

    static int forkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities) {
        ZygoteHooks.preFork();

        int pid = nativeForkSystemServer(
                uid, gid, gids, runtimeFlags, rlimits,
                permittedCapabilities, effectiveCapabilities);

        // Set the Java Language thread priority to the default value for new apps.
        Thread.currentThread().setPriority(Thread.NORM_PRIORITY);

        ZygoteHooks.postForkCommon();
        return pid;
    }

    private static native int nativeForkSystemServer(int uid, int gid, int[] gids, int runtimeFlags,
            int[][] rlimits, long permittedCapabilities, long effectiveCapabilities);

}
```

AOSP/android10/frameworks/base/core/jni/com_android_internal_os_Zygote.cpp
```C++
namespace android {
NO_PAC_FUNC
static jint com_android_internal_os_Zygote_nativeForkSystemServer(
        JNIEnv* env, jclass, uid_t uid, gid_t gid, jintArray gids,
        jint runtime_flags, jobjectArray rlimits, jlong permitted_capabilities,
        jlong effective_capabilities) {

    pid_t pid = ForkCommon(env, true,
                         fds_to_close,
                         fds_to_ignore,
                         true);
    if (pid == 0) {
    } else if (pid > 0) {
        // The zygote process checks whether the child process has died or not.
        ALOGI("System server process %d has been created", pid);
        gSystemServerPid = pid;
        // There is a slight window that the system server process has crashed
        // but it went unnoticed because we haven't published its pid yet. So
        // we recheck here just to make sure that all is well.
        int status;
        if (waitpid(pid, &status, WNOHANG) == pid) {
            ALOGE("System server process %d has died. Restarting Zygote!", pid);
            RuntimeAbort(env, __LINE__, "System server process has died. Restarting Zygote!");
        }
    }
    return pid;
}

static const JNINativeMethod gMethods[] = {
    {"nativeForkSystemServer", "(II[II[[IJJ)I",
        (void*)com_android_internal_os_Zygote_nativeForkSystemServer},
};

int register_com_android_internal_os_Zygote(JNIEnv* env) {
    gZygoteClass = MakeGlobalRefOrDie(env, FindClassOrDie(env, kZygoteClassName));
    gCallPostForkSystemServerHooks = GetStaticMethodIDOrDie(env, gZygoteClass,
                                                            "callPostForkSystemServerHooks",
                                                            "(I)V");
    gCallPostForkChildHooks = GetStaticMethodIDOrDie(env, gZygoteClass, "callPostForkChildHooks",
                                                    "(IZZLjava/lang/String;)V");

    return RegisterMethodsOrDie(env, "com/android/internal/os/Zygote", gMethods, NELEM(gMethods));
}
}
```







## Zygote 面试题


### `Activity` 等系统类以及 `libandroid.so` 等系统库什么时候被加载？

在 Zygote 启动时被加载。ZygoteInit 的 preload() 函数。


enableLazyPreload = true
懒加载
减少内存占用
不经常用，例如Java 的 class

enableLazyPreload = false
预加载
减少CPU工作符合
加载时间长，频率高










