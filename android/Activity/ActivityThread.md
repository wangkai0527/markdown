[TOC]

# ActivityThread

ActivityManagerService 负责管理 Activity 的生命周期，发送信号给 ActivityThread 。
ActivityThread 负责实际执行。
main() --> attach() --> 
ActivityManagerService.attachApplication() --> ActivityManagerService.attachApplicationLocked()--> 
1. bindApplication() --> handleBindApplication()
2. 
android10之前
scheduleLaunchActivity --> handleLaunchActivity --> performLaunchActivity() --> 通过反射实例化 Activity， Activity.attach() 
android10
--> ActivityTaskManagerInternal.attachApplication() --> 
ActivityTaskManagerService.attachApplication() -->
RootWindowContainer.attachApplication --> 
RootWindowContainer.startActivityForAttachedApplicationIfNeeded() -->
ActivityStackSupervisor.realStartActivityLocked()

AOSP/android10/frameworks/base/services/core/java/com/android/server/am/ActivityManagerService.java
```Java
public class ActivityManagerService extends IActivityManager.Stub
        implements Watchdog.Monitor, BatteryStatsImpl.BatteryCallback {

    public ActivityTaskManagerInternal mAtmInternal;

    private boolean attachApplicationLocked(@NonNull IApplicationThread thread,
            int pid, int callingUid, long startSeq) {
    
        thread.bindApplication();

        boolean badApp = false;
        boolean didSomething = false;

        // See if the top visible activity is waiting to run in this process...
        if (normalMode) {
            didSomething = mAtmInternal.attachApplication(app.getWindowProcessController());
        }

        // Find any services that should be running in this process...
        if (!badApp) {
            didSomething |= mServices.attachApplicationLocked(app, processName);
        }

        // Check if a next-broadcast receiver is in this process...
        if (!badApp && isPendingBroadcastProcessLocked(pid)) {
            didSomething |= sendPendingBroadcastsLocked(app);
        }
    }

    public final void attachApplication(IApplicationThread thread, long startSeq) {
        attachApplicationLocked(thread, callingPid, callingUid, startSeq);
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/ActivityTaskManagerService.java
```Java
/**
 * System service for managing activities and their containers (task, stacks, displays,... ).
 */
public class ActivityTaskManagerService extends IActivityTaskManager.Stub {
    RootWindowContainer mRootWindowContainer;
    public boolean attachApplication(WindowProcessController wpc) throws RemoteException {
        return mRootWindowContainer.attachApplication(wpc);
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/RootWindowContainer.java
```Java
class RootWindowContainer extends WindowContainer<DisplayContent>
        implements DisplayManager.DisplayListener {

    ActivityTaskManagerService mService;
    ActivityStackSupervisor mStackSupervisor;
    WindowManagerService mWindowManager;

    boolean attachApplication(WindowProcessController app) throws RemoteException {
        for (int displayNdx = getChildCount() - 1; displayNdx >= 0; --displayNdx) {
            final PooledFunction c = PooledLambda.obtainFunction(
                    RootWindowContainer::startActivityForAttachedApplicationIfNeeded, this,
                    PooledLambda.__(ActivityRecord.class), app, stack.topRunningActivity());
        }
    }

    private boolean startActivityForAttachedApplicationIfNeeded(ActivityRecord r,
            WindowProcessController app, ActivityRecord top) {
        if (mStackSupervisor.realStartActivityLocked(r, app, top == r /*andResume*/,
                true /*checkConfig*/)) {
            mTmpBoolean = true;
        }
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/ActivityStackSupervisor.java
```Java
public class ActivityStackSupervisor implements RecentTasks.Callbacks {
    final ActivityTaskManagerService mService;

    boolean realStartActivityLocked(ActivityRecord r, WindowProcessController proc,
            boolean andResume, boolean checkConfig) throws RemoteException {
    
    }
}
```

AOSP/android10/frameworks/base/core/java/android/app/ActivityThread.java
```Java
public final class ActivityThread extends ClientTransactionHandler {
    final ApplicationThread mAppThread = new ApplicationThread();
    final Looper mLooper = Looper.myLooper();
    final H mH = new H();
    final Executor mExecutor = new HandlerExecutor(mH);

    Instrumentation mInstrumentation;

    class H extends Handler {
        public static final int BIND_APPLICATION        = 110;
        public static final int EXIT_APPLICATION        = 111;
        public static final int RECEIVER                = 113;
        public static final int CREATE_SERVICE          = 114;
        public static final int SERVICE_ARGS            = 115;
        public static final int STOP_SERVICE            = 116;

        public static final int CONFIGURATION_CHANGED   = 118;
        public static final int CLEAN_UP_CONTEXT        = 119;
        public static final int GC_WHEN_IDLE            = 120;
        public static final int BIND_SERVICE            = 121;
        public static final int UNBIND_SERVICE          = 122;
        public static final int DUMP_SERVICE            = 123;
        public static final int LOW_MEMORY              = 124;
        public static final int PROFILER_CONTROL        = 127;
        public static final int CREATE_BACKUP_AGENT     = 128;
        public static final int DESTROY_BACKUP_AGENT    = 129;
        public static final int SUICIDE                 = 130;
        public static final int REMOVE_PROVIDER         = 131;
        public static final int DISPATCH_PACKAGE_BROADCAST = 133;
        public static final int SCHEDULE_CRASH          = 134;
        public static final int DUMP_HEAP               = 135;
        public static final int DUMP_ACTIVITY           = 136;
        public static final int SLEEPING                = 137;
        public static final int SET_CORE_SETTINGS       = 138;
        public static final int UPDATE_PACKAGE_COMPATIBILITY_INFO = 139;
        public static final int DUMP_PROVIDER           = 141;
        public static final int UNSTABLE_PROVIDER_DIED  = 142;
        public static final int REQUEST_ASSIST_CONTEXT_EXTRAS = 143;
        public static final int TRANSLUCENT_CONVERSION_COMPLETE = 144;
        public static final int INSTALL_PROVIDER        = 145;
        public static final int ON_NEW_ACTIVITY_OPTIONS = 146;
        public static final int ENTER_ANIMATION_COMPLETE = 149;
        public static final int START_BINDER_TRACKING = 150;
        public static final int STOP_BINDER_TRACKING_AND_DUMP = 151;
        public static final int LOCAL_VOICE_INTERACTION_STARTED = 154;
        public static final int ATTACH_AGENT = 155;
        public static final int APPLICATION_INFO_CHANGED = 156;
        public static final int RUN_ISOLATED_ENTRY_POINT = 158;
        public static final int EXECUTE_TRANSACTION = 159;
        public static final int RELAUNCH_ACTIVITY = 160;
        public static final int PURGE_RESOURCES = 161;
        public static final int ATTACH_STARTUP_AGENTS = 162;

        public void handleMessage(Message msg) {
            switch (msg.what) {
                case BIND_APPLICATION:
                    Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "bindApplication");
                    AppBindData data = (AppBindData)msg.obj;
                    handleBindApplication(data);
                    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
                    break;
                case EXIT_APPLICATION:
                    if (mInitialApplication != null) {
                        mInitialApplication.onTerminate();
                    }
                    Looper.myLooper().quit();
                    break;
            }
            Object obj = msg.obj;
            if (obj instanceof SomeArgs) {
                ((SomeArgs) obj).recycle();
            }
        }
    }

    public final Activity startActivityNow(Activity parent, String id,
        Intent intent, ActivityInfo activityInfo, IBinder token, Bundle state,
        Activity.NonConfigurationInstances lastNonConfigurationInstances, IBinder assistToken) {
        return performLaunchActivity(r, null /* customIntent */);
    }

    private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
        Activity activity = null;
        try {
            activity = mInstrumentation.newActivity(
                    cl, component.getClassName(), r.intent);
        }

        try {
            Application app = r.packageInfo.makeApplication(false, mInstrumentation);
            if (activity != null) {
                Window window = null;
                if (r.mPendingRemoveWindow != null && r.mPreserveWindow) {
                    window = r.mPendingRemoveWindow;
                    r.mPendingRemoveWindow = null;
                    r.mPendingRemoveWindowManager = null;
                }

                activity.attach(appContext, this, getInstrumentation(), r.token,
                        r.ident, app, r.intent, r.activityInfo, title, r.parent,
                        r.embeddedID, r.lastNonConfigurationInstances, config,
                        r.referrer, r.voiceInteractor, window, r.configCallback,
                        r.assistToken);

                int theme = r.activityInfo.getThemeResource();
                if (theme != 0) {
                    activity.setTheme(theme);
                }

                activity.mCalled = false;
                if (r.isPersistable()) {
                    mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
                } else {
                    mInstrumentation.callActivityOnCreate(activity, r.state);
                }
                if (!activity.mCalled) {
                    throw new SuperNotCalledException(
                        "Activity " + r.intent.getComponent().toShortString() +
                        " did not call through to super.onCreate()");
                }
                r.activity = activity;
                mLastReportedWindowingMode.put(activity.getActivityToken(),
                        config.windowConfiguration.getWindowingMode());
            }
            r.setState(ON_CREATE);
        }
    }

    /**
     * Extended implementation of activity launch. Used when server requests a launch or relaunch.
     */
    @Override
    public Activity handleLaunchActivity(ActivityClientRecord r,
            PendingTransactionActions pendingActions, Intent customIntent) {

        WindowManagerGlobal.initialize();

        final Activity a = performLaunchActivity(r, customIntent);

        return a;
    }

    @VisibleForTesting
    public ActivityClientRecord performResumeActivity(IBinder token, boolean finalStateRequest,
            String reason) {
        final ActivityClientRecord r = mActivities.get(token);
        if (r.getLifecycleState() == ON_RESUME) {
            return null;
        }
        try {
            r.activity.performResume(r.startsNotResumed, reason);
            r.setState(ON_RESUME);
        }
        return r;
    }

    @Override
    public void handleResumeActivity(IBinder token, boolean finalStateRequest, boolean isForward,
            String reason) {
        final ActivityClientRecord r = performResumeActivity(token, finalStateRequest, reason);

        final Activity a = r.activity;

        // If the window hasn't yet been added to the window manager,
        // and this guy didn't finish itself or start another activity,
        // then go ahead and add the window.
        boolean willBeVisible = !a.mStartedActivity;
        if (!willBeVisible) {
            try {
                willBeVisible = ActivityTaskManager.getService().willActivityBeVisible(
                        a.getActivityToken());
            } catch (RemoteException e) {
                throw e.rethrowFromSystemServer();
            }
        }
        if (r.window == null && !a.mFinished && willBeVisible) {
            r.window = r.activity.getWindow();
            View decor = r.window.getDecorView();
            decor.setVisibility(View.INVISIBLE);
            ViewManager wm = a.getWindowManager();
            WindowManager.LayoutParams l = r.window.getAttributes();
            a.mDecor = decor;
            l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
            l.softInputMode |= forwardBit;
            if (r.mPreserveWindow) {
                a.mWindowAdded = true;
                r.mPreserveWindow = false;
                // Normally the ViewRoot sets up callbacks with the Activity
                // in addView->ViewRootImpl#setView. If we are instead reusing
                // the decor view we have to notify the view root that the
                // callbacks may have changed.
                ViewRootImpl impl = decor.getViewRootImpl();
                if (impl != null) {
                    impl.notifyChildRebuilt();
                }
            }
            if (a.mVisibleFromClient) {
                if (!a.mWindowAdded) {
                    a.mWindowAdded = true;
                    wm.addView(decor, l);
                } else {
                    // The activity will get a callback for this {@link LayoutParams} change
                    // earlier. However, at that time the decor will not be set (this is set
                    // in this method), so no action will be taken. This call ensures the
                    // callback occurs with the decor set.
                    a.onWindowAttributesChanged(l);
                }
            }

            // If the window has already been added, but during resume
            // we started another activity, then don't yet make the
            // window visible.
        } else if (!willBeVisible) {
            if (localLOGV) Slog.v(TAG, "Launch " + r + " mStartedActivity set");
            r.hideForNow = true;
        }
    }

    private void attach(boolean system, long startSeq) {
        sCurrentActivityThread = this;
        mSystemThread = system;
        if (!system) {
            android.ddm.DdmHandleAppName.setAppName("<pre-initialized>",
                                                    UserHandle.myUserId());
            RuntimeInit.setApplicationObject(mAppThread.asBinder());
            final IActivityManager mgr = ActivityManager.getService();
            try {
                mgr.attachApplication(mAppThread, startSeq);
            } catch (RemoteException ex) {
                throw ex.rethrowFromSystemServer();
            }
            // Watch for getting close to heap limit.
            BinderInternal.addGcWatcher(new Runnable() {
                @Override public void run() {
                    if (!mSomeActivitiesChanged) {
                        return;
                    }
                    Runtime runtime = Runtime.getRuntime();
                    long dalvikMax = runtime.maxMemory();
                    long dalvikUsed = runtime.totalMemory() - runtime.freeMemory();
                    if (dalvikUsed > ((3*dalvikMax)/4)) {
                        if (DEBUG_MEMORY_TRIM) Slog.d(TAG, "Dalvik max=" + (dalvikMax/1024)
                                + " total=" + (runtime.totalMemory()/1024)
                                + " used=" + (dalvikUsed/1024));
                        mSomeActivitiesChanged = false;
                        try {
                            ActivityTaskManager.getService().releaseSomeActivities(mAppThread);
                        } catch (RemoteException e) {
                            throw e.rethrowFromSystemServer();
                        }
                    }
                }
            });
        } else {
            // Don't set application object here -- if the system crashes,
            // we can't display an alert, we just want to die die die.
            android.ddm.DdmHandleAppName.setAppName("system_process",
                    UserHandle.myUserId());
            try {
                mInstrumentation = new Instrumentation();
                mInstrumentation.basicInit(this);
                ContextImpl context = ContextImpl.createAppContext(
                        this, getSystemContext().mPackageInfo);
                mInitialApplication = context.mPackageInfo.makeApplication(true, null);
                mInitialApplication.onCreate();
            } catch (Exception e) {
                throw new RuntimeException(
                        "Unable to instantiate Application():" + e.toString(), e);
            }
        }

        ViewRootImpl.ConfigChangedCallback configChangedCallback
                = (Configuration globalConfig) -> {
            synchronized (mResourcesManager) {
                // TODO (b/135719017): Temporary log for debugging IME service.
                if (Build.IS_DEBUGGABLE && mHasImeComponent) {
                    Log.d(TAG, "ViewRootImpl.ConfigChangedCallback for IME, "
                            + "config=" + globalConfig);
                }

                // We need to apply this change to the resources immediately, because upon returning
                // the view hierarchy will be informed about it.
                if (mResourcesManager.applyConfigurationToResourcesLocked(globalConfig,
                        null /* compat */)) {
                    updateLocaleListFromAppContext(mInitialApplication.getApplicationContext(),
                            mResourcesManager.getConfiguration().getLocales());

                    // This actually changed the resources! Tell everyone about it.
                    if (mPendingConfiguration == null
                            || mPendingConfiguration.isOtherSeqNewer(globalConfig)) {
                        mPendingConfiguration = globalConfig;
                        sendMessage(H.CONFIGURATION_CHANGED, globalConfig);
                    }
                }
            }
        };
        ViewRootImpl.addConfigCallback(configChangedCallback);
    }

    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // Install selective syscall interception
        AndroidOs.install();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        // Call per-process mainline module initialization.
        initializeMainlineModules();

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
}
```


## Activity

AOSP/android10/frameworks/base/core/java/android/app/Activity.java
```Java
public class Activity extends ContextThemeWrapper
        implements LayoutInflater.Factory2,
        Window.Callback, KeyEvent.Callback,
        OnCreateContextMenuListener, ComponentCallbacks2,
        Window.OnWindowDismissedCallback,
        AutofillManager.AutofillClient, ContentCaptureManager.ContentCaptureClient {
    
    private Instrumentation mInstrumentation;

    private Window mWindow;
    private WindowManager mWindowManager;
    /*package*/ View mDecor = null;

    /** Retrieve the window manager for showing custom windows. */
    public WindowManager getWindowManager() {
        return mWindowManager;
    }

    /**
     * Retrieve the current {@link android.view.Window} for the activity.
     * This can be used to directly access parts of the Window API that
     * are not available through Activity/Screen.
     *
     * @return Window The current window, or null if the activity is not
     *         visual.
     */
    public Window getWindow() {
        return mWindow;
    }

    final void attach(Context context, ActivityThread aThread,
            Instrumentation instr, IBinder token, int ident,
            Application application, Intent intent, ActivityInfo info,
            CharSequence title, Activity parent, String id,
            NonConfigurationInstances lastNonConfigurationInstances,
            Configuration config, String referrer, IVoiceInteractor voiceInteractor,
            Window window, ActivityConfigCallback activityConfigCallback, IBinder assistToken) {
        attachBaseContext(context);

        mFragments.attachHost(null /*parent*/);

        mWindow = new PhoneWindow(this, window, activityConfigCallback);
        mWindow.setWindowControllerCallback(mWindowControllerCallback);
        mWindow.setCallback(this);
        mWindow.setOnWindowDismissedCallback(this);
        mWindow.getLayoutInflater().setPrivateFactory(this);
        if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
            mWindow.setSoftInputMode(info.softInputMode);
        }
        if (info.uiOptions != 0) {
            mWindow.setUiOptions(info.uiOptions);
        }
        mUiThread = Thread.currentThread();

        mMainThread = aThread;
        mInstrumentation = instr;
        mToken = token;
        mAssistToken = assistToken;
        mIdent = ident;
        mApplication = application;
        mIntent = intent;
        mReferrer = referrer;
        mComponent = intent.getComponent();
        mActivityInfo = info;
        mTitle = title;
        mParent = parent;
        mEmbeddedID = id;
        mLastNonConfigurationInstances = lastNonConfigurationInstances;
        if (voiceInteractor != null) {
            if (lastNonConfigurationInstances != null) {
                mVoiceInteractor = lastNonConfigurationInstances.voiceInteractor;
            } else {
                mVoiceInteractor = new VoiceInteractor(voiceInteractor, this, this,
                        Looper.myLooper());
            }
        }

        mWindow.setWindowManager(
                (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
                mToken, mComponent.flattenToString(),
                (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
        if (mParent != null) {
            mWindow.setContainer(mParent.getWindow());
        }
        mWindowManager = mWindow.getWindowManager();
        mCurrentConfig = config;

        mWindow.setColorMode(info.colorMode);
        mWindow.setPreferMinimalPostProcessing(
                (info.flags & ActivityInfo.FLAG_PREFER_MINIMAL_POST_PROCESSING) != 0);

        setAutofillOptions(application.getAutofillOptions());
        setContentCaptureOptions(application.getContentCaptureOptions());
    }

    final void performResume(boolean followedByPause, String reason) {
        // mResumed is set by the instrumentation
        mInstrumentation.callActivityOnResume(this);
    }
}
```


## Instrumentation
[](Instrumentation.md)





