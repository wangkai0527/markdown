[TOC]

# Window模块



## PhoneWindow
一个 Activity 对应一个 PhoneWindow 
PhoneWindow 实例化是在 Activity 的 attach()

一个 PhoneWindow 对应一个 DecorView 
DecorView 管理
LayoutInflater 解析xml

AOSP/android10/frameworks/base/core/java/com/android/internal/policy/PhoneWindow.java
```Java
public class PhoneWindow extends Window implements MenuBuilder.Callback {
    // This is the top-level view of the window, containing the window decor.
    private DecorView mDecor;

    private LayoutInflater mLayoutInflater;

    /**
     * Constructor for main window of an activity.
     */
    public PhoneWindow(Context context, Window preservedWindow,
            ActivityConfigCallback activityConfigCallback) {
        this(context);

        if (preservedWindow != null) {
            mDecor = (DecorView) preservedWindow.getDecorView();
        }
    }
}
```



AOSP/android10/frameworks/base/core/java/android/view/Window.java
```Java
public abstract class Window {
    private WindowManager mWindowManager;

    public void setWindowManager(WindowManager wm, IBinder appToken, String appName) {
        setWindowManager(wm, appToken, appName, false);
    }

    /**
     * Set the window manager for use by this Window to, for example,
     * display panels.  This is <em>not</em> used for displaying the
     * Window itself -- that must be done by the client.
     *
     * @param wm The window manager for adding new windows.
     */
    public void setWindowManager(WindowManager wm, IBinder appToken, String appName,
            boolean hardwareAccelerated) {
        mAppToken = appToken;
        mAppName = appName;
        mHardwareAccelerated = hardwareAccelerated;
        if (wm == null) {
            wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
        }
        mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
    }

    /**
     * Return the window manager allowing this Window to display its own
     * windows.
     *
     * @return WindowManager The ViewManager.
     */
    public WindowManager getWindowManager() {
        return mWindowManager;
    }
}
```


AOSP/android10/frameworks/base/core/java/android/view/WindowManager.java
```Java
/**
 * The interface that apps use to talk to the window manager.
 */
@SystemService(Context.WINDOW_SERVICE)
public interface WindowManager extends ViewManager {

}
```

AOSP/android10/frameworks/base/core/java/android/view/ViewManager.java
```Java
public interface ViewManager
{
    /**
     * Assign the passed LayoutParams to the passed View and add the view to the window.
     * @param view The view to be added to this window.
     * @param params The LayoutParams to assign to view.
     */
    public void addView(View view, ViewGroup.LayoutParams params);
    public void updateViewLayout(View view, ViewGroup.LayoutParams params);
    public void removeView(View view);
}
```


## WindowManagerImpl
作用是隔离 ActivityThread 和 Window 模块。

AOSP/android10/frameworks/base/core/java/android/view/WindowManagerImpl.java
```Java
public final class WindowManagerImpl implements WindowManager {
    private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
    public final Context mContext;
    private final Window mParentWindow;

    private IBinder mDefaultToken;

    public WindowManagerImpl(Context context) {
        this(context, null);
    }

    private WindowManagerImpl(Context context, Window parentWindow) {
        mContext = context;
        mParentWindow = parentWindow;
    }

    public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
        return new WindowManagerImpl(mContext, parentWindow);
    }

    @Override
    public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
        applyDefaultToken(params);
        mGlobal.addView(view, params, mContext.getDisplayNoVerify(), mParentWindow,
                mContext.getUserId());
    }

    @Override
    public void removeView(View view) {
        mGlobal.removeView(view, false);
    }
}
```




## WindowManagerGlobal
一个App对应一个 WindowManagerGlobal
对应一个 WindowSession

1. 缓存App所有的 DecorView
2. 进程通信和 WindowManagerService
3. 对外提供查找服务

addView() --> new ViewRootImpl() --> 
WindowManagerGlobal.getWindowSession() --> IWindowManager.openSession() --> new Session()

ViewRootImpl.setView() --> IWindowSession.addToDisplayAsUser() -->
WindowManagerService.addWindow() --> WindowState.attach() --> 
Session.windowAddedLocked --> new SurfaceSession() --> nativeCreate() --> 
SurfaceComposerClient::getDefault() --> new SurfaceComposerClient


AOSP/android10/frameworks/base/core/java/android/view/WindowManagerGlobal.java
```Java
public final class WindowManagerGlobal {
    private static WindowManagerGlobal sDefaultWindowManager;
    private static IWindowManager sWindowManagerService;
    private static IWindowSession sWindowSession;

    public static void initialize() {
        getWindowManagerService();
    }

    public static WindowManagerGlobal getInstance() {
        synchronized (WindowManagerGlobal.class) {
            if (sDefaultWindowManager == null) {
                sDefaultWindowManager = new WindowManagerGlobal();
            }
            return sDefaultWindowManager;
        }
    }

    public static IWindowManager getWindowManagerService() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowManagerService == null) {
                sWindowManagerService = IWindowManager.Stub.asInterface(
                        ServiceManager.getService("window"));
                try {
                    if (sWindowManagerService != null) {
                        ValueAnimator.setDurationScale(
                                sWindowManagerService.getCurrentAnimatorScale());
                        sUseBLASTAdapter = sWindowManagerService.useBLAST();
                    }
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
            }
            return sWindowManagerService;
        }
    }

    public static IWindowSession getWindowSession() {
        synchronized (WindowManagerGlobal.class) {
            if (sWindowSession == null) {
                try {
                    // Emulate the legacy behavior.  The global instance of InputMethodManager
                    // was instantiated here.
                    // TODO(b/116157766): Remove this hack after cleaning up @UnsupportedAppUsage
                    InputMethodManager.ensureDefaultInstanceForDefaultDisplayIfNecessary();
                    IWindowManager windowManager = getWindowManagerService();
                    sWindowSession = windowManager.openSession(
                            new IWindowSessionCallback.Stub() {
                                @Override
                                public void onAnimatorScaleChanged(float scale) {
                                    ValueAnimator.setDurationScale(scale);
                                }
                            });
                } catch (RemoteException e) {
                    throw e.rethrowFromSystemServer();
                }
            }
            return sWindowSession;
        }
    }

    public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) {
        
        final WindowManager.LayoutParams wparams = (WindowManager.LayoutParams) params;

        ViewRootImpl root;
        View panelParentView = null;

        synchronized (mLock) {
            root = new ViewRootImpl(view.getContext(), display);

            view.setLayoutParams(wparams);

            mViews.add(view);
            mRoots.add(root);
            mParams.add(wparams);

            // do this last because it fires off messages to start doing things
            try {
                root.setView(view, wparams, panelParentView, userId);
            }
        }
    }

    public void removeView(View view, boolean immediate) {
        synchronized (mLock) {
            int index = findViewLocked(view, true);
            View curView = mRoots.get(index).getView();
            removeViewLocked(index, immediate);
            if (curView == view) {
                return;
            }
        }
    }

    private void removeViewLocked(int index, boolean immediate) {
        ViewRootImpl root = mRoots.get(index);
        View view = root.getView();

        if (root != null) {
            root.getImeFocusController().onWindowDismissed();
        }
        boolean deferred = root.die(immediate);
        if (view != null) {
            view.assignParent(null);
            if (deferred) {
                mDyingViews.add(view);
            }
        }
    }
}
```




AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/WindowManagerService.java
```Java
public class WindowManagerService extends IWindowManager.Stub
        implements Watchdog.Monitor, WindowManagerPolicy.WindowManagerFuncs {
    /**
     * All currently active sessions with clients.
     */
    final ArraySet<Session> mSessions = new ArraySet<>();

    public int addWindow(Session session, IWindow client, int seq,
            LayoutParams attrs, int viewVisibility, int displayId, Rect outFrame,
            Rect outContentInsets, Rect outStableInsets,
            DisplayCutout.ParcelableWrapper outDisplayCutout, InputChannel outInputChannel,
            InsetsState outInsetsState, InsetsSourceControl[] outActiveControls,
            int requestUserId) {

        synchronized (mGlobalLock) {
            final WindowState win = new WindowState(this, session, client, token, parentWindow,
                    appOp[0], seq, attrs, viewVisibility, session.mUid, userId,
                    session.mCanAddInternalSystemWindow);
            
            win.attach();
            mWindowMap.put(client.asBinder(), win);
            win.initAppOpsState();
        }

    }

    // -------------------------------------------------------------
    // IWindowManager API
    // -------------------------------------------------------------

    @Override
    public IWindowSession openSession(IWindowSessionCallback callback) {
        return new Session(this, callback);
    }
}
```

AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/WindowState.java
```Java
/** A window in the window manager. */
class WindowState extends WindowContainer<WindowState> implements WindowManagerPolicy.WindowState,
        InsetsControlTarget {
    final Session mSession;

    void attach() {
        if (DEBUG) Slog.v(TAG, "Attaching " + this + " token=" + mToken);
        mSession.windowAddedLocked(mAttrs.packageName);
    }
}
```


AOSP/android10/frameworks/base/services/core/java/com/android/server/wm/Session.java
```Java
/**
 * This class represents an active client session.  There is generally one
 * Session object per process that is interacting with the window manager.
 */
class Session extends IWindowSession.Stub implements IBinder.DeathRecipient {
    final WindowManagerService mService;
    SurfaceSession mSurfaceSession;

    void windowAddedLocked(String packageName) {
        mPackageName = packageName;
        mRelayoutTag = "relayoutWindow: " + mPackageName;
        if (mSurfaceSession == null) {
            if (DEBUG) {
                Slog.v(TAG_WM, "First window added to " + this + ", creating SurfaceSession");
            }
            mSurfaceSession = new SurfaceSession();
            ProtoLog.i(WM_SHOW_TRANSACTIONS, "  NEW SURFACE SESSION %s", mSurfaceSession);
            mService.mSessions.add(this);
            if (mLastReportedAnimatorScale != mService.getCurrentAnimatorScale()) {
                mService.dispatchNewAnimatorScaleLocked(this);
            }
        }
        mNumWindow++;
    }
}
```


AOSP/android10/frameworks/base/core/java/android/view/SurfaceSession.java
```Java
/**
 * An instance of this class represents a connection to the surface
 * flinger, from which you can create one or more Surface instances that will
 * be composited to the screen.
 * {@hide}
 */
public final class SurfaceSession {
    // Note: This field is accessed by native code.
    @UnsupportedAppUsage
    private long mNativeClient; // SurfaceComposerClient*

    private static native long nativeCreate();
    private static native void nativeDestroy(long ptr);
    private static native void nativeKill(long ptr);

    /** Create a new connection with the surface flinger. */
    @UnsupportedAppUsage
    public SurfaceSession() {
        mNativeClient = nativeCreate();
    }

    /* no user serviceable parts here ... */
    @Override
    protected void finalize() throws Throwable {
        try {
            if (mNativeClient != 0) {
                nativeDestroy(mNativeClient);
            }
        } finally {
            super.finalize();
        }
    }

    /**
     * Forcibly detach native resources associated with this object.
     * Unlike destroy(), after this call any surfaces that were created
     * from the session will no longer work.
     */
    @UnsupportedAppUsage
    public void kill() {
        nativeKill(mNativeClient);
    }
}
```



AOSP/android10/frameworks/base/core/jni/android_view_SurfaceControl.cpp
```C++
namespace android {

static jlong nativeCreate(JNIEnv* env, jclass clazz, jobject sessionObj,
        jstring nameStr, jint w, jint h, jint format, jint flags, jlong parentObject,
        jobject metadataParcel) {
    ScopedUtfChars name(env, nameStr);
    sp<SurfaceComposerClient> client;
    if (sessionObj != NULL) {
        client = android_view_SurfaceSession_getClient(env, sessionObj);
    } else {
        client = SurfaceComposerClient::getDefault();
    }
    SurfaceControl *parent = reinterpret_cast<SurfaceControl*>(parentObject);
    sp<SurfaceControl> surface;
    LayerMetadata metadata;
    Parcel* parcel = parcelForJavaObject(env, metadataParcel);
    if (parcel && !parcel->objectsCount()) {
        status_t err = metadata.readFromParcel(parcel);
        if (err != NO_ERROR) {
          jniThrowException(env, "java/lang/IllegalArgumentException",
                            "Metadata parcel has wrong format");
        }
    }

    status_t err = client->createSurfaceChecked(
            String8(name.c_str()), w, h, format, &surface, flags, parent, std::move(metadata));
    if (err == NAME_NOT_FOUND) {
        jniThrowException(env, "java/lang/IllegalArgumentException", NULL);
        return 0;
    } else if (err != NO_ERROR) {
        jniThrowException(env, OutOfResourcesException, NULL);
        return 0;
    }

    surface->incStrong((void *)nativeCreate);
    return reinterpret_cast<jlong>(surface.get());
}

static const JNINativeMethod sSurfaceControlMethods[] = {
    {"nativeCreate", "(Landroid/view/SurfaceSession;Ljava/lang/String;IIIIJLandroid/os/Parcel;)J",
            (void*)nativeCreate },
};

int register_android_view_SurfaceControl(JNIEnv* env)
{
    int err = RegisterMethodsOrDie(env, "android/view/SurfaceControl",
            sSurfaceControlMethods, NELEM(sSurfaceControlMethods));
    return err;
}

}
```


AOSP/android10/frameworks/native/libs/gui/SurfaceComposerClient.cpp
```C++
namespace android {
class DefaultComposerClient: public Singleton<DefaultComposerClient> {
    Mutex mLock;
    sp<SurfaceComposerClient> mClient;
    friend class Singleton<ComposerService>;
public:
    static sp<SurfaceComposerClient> getComposerClient() {
        DefaultComposerClient& dc = DefaultComposerClient::getInstance();
        Mutex::Autolock _l(dc.mLock);
        if (dc.mClient == nullptr) {
            dc.mClient = new SurfaceComposerClient;
        }
        return dc.mClient;
    }
};
ANDROID_SINGLETON_STATIC_INSTANCE(DefaultComposerClient);


sp<SurfaceComposerClient> SurfaceComposerClient::getDefault() {
    return DefaultComposerClient::getComposerClient();
}

}
```





https://juejin.cn/post/7049608618143907877

https://juejin.cn/post/7015973616659464206




