[TOC]

# Instrumentation

声明用于监控应用与系统交互的 Instrumentation 类。

https://developer.android.google.cn/guide/topics/manifest/instrumentation-element?hl=zh-cn

https://zhuanlan.zhihu.com/p/522415346

AOSP/android10/frameworks/base/core/java/android/app/Instrumentation.java
```Java
/**
 * Base class for implementing application instrumentation code.  When running
 * with instrumentation turned on, this class will be instantiated for you
 * before any of the application code, allowing you to monitor all of the
 * interaction the system has with the application.  An Instrumentation
 * implementation is described to the system through an AndroidManifest.xml's
 * &lt;instrumentation&gt; tag.
 */
public class Instrumentation {

    public void callApplicationOnCreate(Application app) {
        app.onCreate();
    }

    public void callActivityOnCreate(Activity activity, Bundle icicle) {
        prePerformCreate(activity);
        activity.performCreate(icicle);
        postPerformCreate(activity);
    }

    /**
     * Perform calling of an activity's {@link Activity#onCreate}
     * method.  The default implementation simply calls through to that method.
     *  @param activity The activity being created.
     * @param icicle The previously frozen state (or null) to pass through to
     * @param persistentState The previously persisted state (or null)
     */
    public void callActivityOnCreate(Activity activity, Bundle icicle,
            PersistableBundle persistentState) {
        prePerformCreate(activity);
        activity.performCreate(icicle, persistentState);
        postPerformCreate(activity);
    }

    public void callActivityOnResume(Activity activity) {
        activity.mResumed = true;
        activity.onResume();
    }
}
```



