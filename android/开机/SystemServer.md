[TOC]

# SystemServer

添加自定义系统服务
1. mSystemServiceManager.startService("全类名")
2. 添加自定义 Service 类继承 SystemService
3. 重写 onStart() 方法，启动 Service


AOSP/android10/frameworks/base/services/java/com/android/server/SystemServer.java
```Java
public final class SystemServer {

    private static final String CAR_SERVICE_HELPER_SERVICE_CLASS =
            "com.android.internal.car.CarServiceHelperService";
    
    // maximum number of binder threads used for system_server
    // will be higher than the system default
    private static final int sMaxBinderThreads = 31;

    private Context mSystemContext;
    private SystemServiceManager mSystemServiceManager;
    
    // TODO: remove all of these references by improving dependency resolution and boot phases
    private PowerManagerService mPowerManagerService;
    private ActivityManagerService mActivityManagerService;
    private WindowManagerGlobalLock mWindowManagerGlobalLock;


    /**
     * The main entry point from zygote.
     */
    public static void main(String[] args) {
        new SystemServer().run();
    }

    private void run() {
        try {
            // Increase the number of binder threads in system_server
            BinderInternal.setMaxThreads(sMaxBinderThreads);

            // Create the system service manager.
            mSystemServiceManager = new SystemServiceManager(mSystemContext);
            mSystemServiceManager.setStartInfo(mRuntimeRestart,
                    mRuntimeStartElapsedTime, mRuntimeStartUptime);
            LocalServices.addService(SystemServiceManager.class, mSystemServiceManager);
        }

        try {
            //启动引导服务
            startBootstrapServices(t);
            //启动核心服务
            startCoreServices(t);
            //启动其他服务
            startOtherServices(t);
        }
    }

    /**
     * Starts the small tangle of critical services that are needed to get the system off the
     * ground.  These services have complex mutual dependencies which is why we initialize them all
     * in one place here.  Unless your service is also entwined in these dependencies, it should be
     * initialized in one of the other functions.
     */
    private void startBootstrapServices(@NonNull TimingsTraceAndSlog t) {
        // TODO: Might need to move after migration to WM.
        ActivityTaskManagerService atm = mSystemServiceManager.startService(
                ActivityTaskManagerService.Lifecycle.class).getService();
        mActivityManagerService = ActivityManagerService.Lifecycle.startService(
                mSystemServiceManager, atm);
        mActivityManagerService.setSystemServiceManager(mSystemServiceManager);
        mActivityManagerService.setInstaller(installer);

        // Power manager needs to be started early because other services need it.
        // Native daemons may be watching for it to be registered so it must be ready
        // to handle incoming binder calls immediately (including being able to verify
        // the permissions for those calls).
        mPowerManagerService = mSystemServiceManager.startService(PowerManagerService.class);

        // Now that the power manager has been started, let the activity manager
        // initialize power management features.
        mActivityManagerService.initPowerManagement();

        // Set up the Application instance for the system process and get started.
        mActivityManagerService.setSystemProcess();

        // Complete the watchdog setup with an ActivityManager instance and listen for reboots
        // Do this only after the ActivityManagerService is properly started as a system process
        watchdog.init(mSystemContext, mActivityManagerService);
    }

    /**
     * Starts some essential services that are not tangled up in the bootstrap process.
     */
    private void startCoreServices(@NonNull TimingsTraceAndSlog t) {
        mSystemServiceManager.startService(SystemConfigService.class);

        // Tracks the battery level.  Requires LightService.
        mSystemServiceManager.startService(BatteryService.class);

        // Tracks whether the updatable WebView is in a ready state and watches for update installs.
        if (mPackageManager.hasSystemFeature(PackageManager.FEATURE_WEBVIEW)) {
            mWebViewUpdateService = mSystemServiceManager.startService(WebViewUpdateService.class);
        }

        // Tracks cpu time spent in binder calls
        mSystemServiceManager.startService(BinderCallsStatsService.LifeCycle.class);

        // Tracks time spent in handling messages in handlers.
        mSystemServiceManager.startService(LooperStatsService.Lifecycle.class);

        // Service to capture bugreports.
        mSystemServiceManager.startService(BugreportManagerService.class);

        // Serivce for GPU and GPU driver.
        mSystemServiceManager.startService(GpuService.class);
    }

    /**
     * Starts a miscellaneous grab bag of stuff that has yet to be refactored and organized.
     */
    private void startOtherServices(@NonNull TimingsTraceAndSlog t) {
        // We now tell the activity manager it is okay to run third party
        // code.  It will call back into us once it has gotten to the state
        // where third party code can really run (but before it has actually
        // started launching the initial applications), for us to complete our
        // initialization.
        mActivityManagerService.systemReady(() -> {
            if (mPackageManager.hasSystemFeature(PackageManager.FEATURE_AUTOMOTIVE)) {
                mSystemServiceManager.startService(CAR_SERVICE_HELPER_SERVICE_CLASS);
            }

            try {
                startSystemUi(context, windowManagerF);
            } catch (Throwable e) {
                reportWtf("starting System UI", e);
            }
        }, t);
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java
```Java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {
    
    public void setSystemProcess() {
        try {
            ServiceManager.addService(Context.ACTIVITY_SERVICE, this, /* allowIsolated= */ true,
                    DUMP_FLAG_PRIORITY_CRITICAL | DUMP_FLAG_PRIORITY_NORMAL | DUMP_FLAG_PROTO);
            ServiceManager.addService(ProcessStats.SERVICE_NAME, mProcessStats);
            ServiceManager.addService("meminfo", new MemBinder(this), /* allowIsolated= */ false,
                    DUMP_FLAG_PRIORITY_HIGH);
            ServiceManager.addService("gfxinfo", new GraphicsBinder(this));
            ServiceManager.addService("dbinfo", new DbBinder(this));
            if (MONITOR_CPU_USAGE) {
                ServiceManager.addService("cpuinfo", new CpuBinder(this),
                        /* allowIsolated= */ false, DUMP_FLAG_PRIORITY_CRITICAL);
            }
            ServiceManager.addService("permission", new PermissionController(this));
            ServiceManager.addService("processinfo", new ProcessInfoService(this));
            ServiceManager.addService("cacheinfo", new CacheBinder(this));

            ApplicationInfo info = mContext.getPackageManager().getApplicationInfo(
                    "android", STOCK_PM_FLAGS | MATCH_SYSTEM_ONLY);
            mSystemThread.installSystemApplicationInfo(info, getClass().getClassLoader());

            synchronized (this) {
                ProcessRecord app = mProcessList.newProcessRecordLocked(info, info.processName,
                        false,
                        0,
                        new HostingRecord("system"));
                app.setPersistent(true);
                app.pid = MY_PID;
                app.getWindowProcessController().setPid(MY_PID);
                app.maxAdj = ProcessList.SYSTEM_ADJ;
                app.makeActive(mSystemThread.getApplicationThread(), mProcessStats);
                addPidLocked(app);
                mProcessList.updateLruProcessLocked(app, false, null);
                updateOomAdjLocked(OomAdjuster.OOM_ADJ_REASON_NONE);
            }
        } catch (PackageManager.NameNotFoundException e) {
            throw new RuntimeException(
                    "Unable to find android system package", e);
        }

        // Start watching app ops after we and the package manager are up and running.
        mAppOpsService.startWatchingMode(AppOpsManager.OP_RUN_IN_BACKGROUND, null,
                new IAppOpsCallback.Stub() {
                    @Override public void opChanged(int op, int uid, String packageName) {
                        if (op == AppOpsManager.OP_RUN_IN_BACKGROUND && packageName != null) {
                            if (getAppOpsManager().checkOpNoThrow(op, uid, packageName)
                                    != AppOpsManager.MODE_ALLOWED) {
                                runInBackgroundDisabled(uid);
                            }
                        }
                    }
                });

        final int[] cameraOp = {AppOpsManager.OP_CAMERA};
        mAppOpsService.startWatchingActive(cameraOp, new IAppOpsActiveCallback.Stub() {
            @Override
            public void opActiveChanged(int op, int uid, String packageName, boolean active) {
                cameraActiveChanged(uid, active);
            }
        });
    }

    // Lifecycle 是 ActivityManagerService 的包装类
    public static final class Lifecycle extends SystemService {
        private final ActivityManagerService mService;
        private static ActivityTaskManagerService sAtm;

        public Lifecycle(Context context) {
            super(context);
            mService = new ActivityManagerService(context, sAtm);
        }

        public static ActivityManagerService startService(
                SystemServiceManager ssm, ActivityTaskManagerService atm) {
            sAtm = atm;
            return ssm.startService(ActivityManagerService.Lifecycle.class).getService();
        }

        @Override
        public void onStart() {
            mService.start();
        }

        @Override
        public void onBootPhase(int phase) {
            mService.mBootPhase = phase;
            if (phase == PHASE_SYSTEM_SERVICES_READY) {
                mService.mBatteryStatsService.systemServicesReady();
                mService.mServices.systemServicesReady();
            } else if (phase == PHASE_ACTIVITY_MANAGER_READY) {
                mService.startBroadcastObservers();
            } else if (phase == PHASE_THIRD_PARTY_APPS_CAN_START) {
                mService.mPackageWatchdog.onPackagesReady();
            }
        }

        @Override
        public void onCleanupUser(int userId) {
            mService.mBatteryStatsService.onCleanupUser(userId);
        }

        public ActivityManagerService getService() {
            return mService;
        }
    }

    private void start() {
        removeAllProcessGroups();
        mProcessCpuThread.start();

        mBatteryStatsService.publish();
        mAppOpsService.publish();
        Slog.d("AppOps", "AppOpsService published");
        LocalServices.addService(ActivityManagerInternal.class, mInternal);
        mActivityTaskManager.onActivityManagerInternalAdded();
        mPendingIntentController.onActivityManagerInternalAdded();
        // Wait for the synchronized block started in mProcessCpuThread,
        // so that any other access to mProcessCpuTracker from main thread
        // will be blocked during mProcessCpuTracker initialization.
        try {
            mProcessCpuInitLatch.await();
        } catch (InterruptedException e) {
            Slog.wtf(TAG, "Interrupted wait during start", e);
            Thread.currentThread().interrupt();
            throw new IllegalStateException("Interrupted wait during start");
        }
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/SystemServiceManager.java
```Java
public class SystemServiceManager {

    // Services that should receive lifecycle events.
    private final ArrayList<SystemService> mServices = new ArrayList<SystemService>();

    /**
     * Creates and starts a system service. The class must be a subclass of
     * {@link com.android.server.SystemService}.
     *
     * @param serviceClass A Java class that implements the SystemService interface.
     * @return The service instance, never null.
     * @throws RuntimeException if the service fails to start.
     */
    public <T extends SystemService> T startService(Class<T> serviceClass) {
        try {
            final String name = serviceClass.getName();
            Slog.i(TAG, "Starting " + name);
            Trace.traceBegin(Trace.TRACE_TAG_SYSTEM_SERVER, "StartService " + name);

            // Create the service.
            if (!SystemService.class.isAssignableFrom(serviceClass)) {
                throw new RuntimeException("Failed to create " + name
                        + ": service must extend " + SystemService.class.getName());
            }
            final T service;
            try {
                Constructor<T> constructor = serviceClass.getConstructor(Context.class);
                service = constructor.newInstance(mContext);
            } catch (InstantiationException ex) {
                throw new RuntimeException("Failed to create service " + name
                        + ": service could not be instantiated", ex);
            } catch (IllegalAccessException ex) {
                throw new RuntimeException("Failed to create service " + name
                        + ": service must have a public constructor with a Context argument", ex);
            } catch (NoSuchMethodException ex) {
                throw new RuntimeException("Failed to create service " + name
                        + ": service must have a public constructor with a Context argument", ex);
            } catch (InvocationTargetException ex) {
                throw new RuntimeException("Failed to create service " + name
                        + ": service constructor threw an exception", ex);
            }

            startService(service);
            return service;
        } finally {
            Trace.traceEnd(Trace.TRACE_TAG_SYSTEM_SERVER);
        }
    }

    public void startService(@NonNull final SystemService service) {
        // Register it.
        mServices.add(service);
        // Start it.
        long time = SystemClock.elapsedRealtime();
        try {
            service.onStart();
        } catch (RuntimeException ex) {
            throw new RuntimeException("Failed to start service " + service.getClass().getName()
                    + ": onStart threw an exception", ex);
        }
        warnIfTooLong(SystemClock.elapsedRealtime() - time, service, "onStart");
    }
}
```

AOSP/android10/frameworks/base/core/java/android/content/Context.java
```Java
public abstract class Context {

    /**
     * Use with {@link #getSystemService(String)} to retrieve a
     * {@link android.app.ActivityManager} for interacting with the global
     * system state.
     *
     * @see #getSystemService(String)
     * @see android.app.ActivityManager
     */
    public static final String ACTIVITY_SERVICE = "activity";
    
}
```

AOSP/android10/frameworks/base/core/java/android/os/ServiceManager.java
```Java
public final class ServiceManager {

    
    private static IServiceManager sServiceManager;

    
    private static IServiceManager getIServiceManager() {
        if (sServiceManager != null) {
            return sServiceManager;
        }

        // Find the service manager
        sServiceManager = ServiceManagerNative
                .asInterface(Binder.allowBlocking(BinderInternal.getContextObject()));
        return sServiceManager;
    }

    
    public static void addService(String name, IBinder service) {
        addService(name, service, false, IServiceManager.DUMP_FLAG_PRIORITY_DEFAULT);
    }

    
    public static void addService(String name, IBinder service, boolean allowIsolated) {
        addService(name, service, allowIsolated, IServiceManager.DUMP_FLAG_PRIORITY_DEFAULT);
    }

    /**
     * Place a new @a service called @a name into the service
     * manager.
     *
     * @param name the name of the new service
     * @param service the service object
     * @param allowIsolated set to true to allow isolated sandboxed processes
     * @param dumpPriority supported dump priority levels as a bitmask
     * to access this service
     */
    
    public static void addService(String name, IBinder service, boolean allowIsolated,
            int dumpPriority) {
        try {
            getIServiceManager().addService(name, service, allowIsolated, dumpPriority);
        } catch (RemoteException e) {
            Log.e(TAG, "error in addService", e);
        }
    }

}
```

AOSP/android10/frameworks/base/core/java/android/os/ServiceManagerNative.java
```Java
public final class ServiceManagerNative {
    private ServiceManagerNative() {}

    /**
     * Cast a Binder object into a service manager interface, generating
     * a proxy if needed.
     *
     * TODO: delete this method and have clients use
     *     IServiceManager.Stub.asInterface instead
     */
    
    public static IServiceManager asInterface(IBinder obj) {
        if (obj == null) {
            return null;
        }

        // ServiceManager is never local
        return new ServiceManagerProxy(obj);
    }
}

// This class should be deleted and replaced with IServiceManager.Stub whenever
// mRemote is no longer used
class ServiceManagerProxy implements IServiceManager {
    public ServiceManagerProxy(IBinder remote) {
        mRemote = remote;
        mServiceManager = IServiceManager.Stub.asInterface(remote);
    }

    public IBinder asBinder() {
        return mRemote;
    }

    
    public IBinder getService(String name) throws RemoteException {
        // Same as checkService (old versions of servicemanager had both methods).
        return mServiceManager.checkService(name);
    }

    public void addService(String name, IBinder service, boolean allowIsolated, int dumpPriority)
            throws RemoteException {
        mServiceManager.addService(name, service, allowIsolated, dumpPriority);
    }
}
```






