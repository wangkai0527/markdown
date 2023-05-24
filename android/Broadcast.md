[TOC]

# Broadcast


Android应用可以通过广播从系统或其他App接收或发送消息。类似于订阅-发布设计模式。当某些事件发生时，可以发出广播。 系统在某些状态改变时会发出广播，例如开机、充电。App也可发送自定义广播。广播可用于应用间的通讯，是IPC的一种方式。
广播的种类
广播的种类也可以看成是广播的属性。


标准广播（Normal Broadcasts）
完全异步的广播。广播发出后，所有的广播接收器几乎同时接收到这条广播。 不同的App可以注册并接到标准广播。例如系统广播。


有序广播（Ordered Broadcasts）
同步广播。同一时刻只有一个广播接收器能接收到这条广播。这个接收器处理完后，广播才会继续传递。 有序广播是全局的广播。


本地广播（Local Broaddcasts）
只在本App发送和接收的广播。注册为本地广播的接收器无法收到标准广播。


带权限的广播
发送广播时可以带上相关权限，申请了权限的 App 或广播接收器才能收到相应的带权限的广播。 如果在 manifest 中申请了相应权限，接收器可以不用再申请一次权限即可接到相应广播。


接收广播
创建广播接收器，调用onReceive()方法，需要一个继承 BroadcastReceiver 的类。
注册广播
代码中注册称为动态注册。在AndroidManifest.xml中注册称为静态注册。动态注册的刚波接收器一定要取消注册。在onDestroy()方法中调用unregisterReceiver()方法来取消注册。
不要在onReceive()方法中添加过多的逻辑操作或耗时的操作。因为在广播接收器中不允许开启线程，当onReceive()方法运行较长时间而没结束时，程序会报错。因此广播接收器一般用来打开其他组件，比如创建一条状态栏通知或启动一个服务。
新建一个MyExampleReceiver继承自BroadcastReceiver。
scala复制代码public class MyExampleReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        Toast.makeText(context,"Got it",Toast.LENGTH_SHORT).show();
        //abortBroadcast();
    }
}

abortBroadcast()可以截断有序广播
在AndroidManifest.xml中注册广播接收器；android:name里填接收器的名字。 可以设置广播接收器优先级：
ini复制代码<intent-filter android:priority="100">

<receiver android:name=".MyExampleReceiver">
    <intent-filter>
        <action android:name="com.rust.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>

让接收器接收到一条"com.rust.broadcasttest.MY_BROADCAST"广播。
发送自定义广播（标准广播）时，要传送这个值。例如：
ini复制代码Intent intent = new Intent("com.rust.broadcasttest.MY_BROADCAST");
sendBroadcast(intent);

发送有序广播，应当调用sendOrderedBroadcast()
ini复制代码Intent intent = new Intent("com.rust.broadcasttest.MY_BROADCAST");
sendOrderedBroadcast(intent，null);

发送广播
App有3种发送广播的方式。发送广播需要使用Intent类。
sendOrderedBroadcast(Intent, String)
发送有序广播。每次只有1个广播接收器能接到广播。 接收器接到有序广播后，可以完全地截断广播，或者传递一些信息给下一个接收器。 有序广播的顺序可受android:priority标签影响。同等级的接收器收到广播的顺序是随机的。
sendBroadcast(Intent)
以一个未定义的顺序向所有接收器发送广播。也称作普通广播。 这种方式更高效，但是接收器不能给下一个接收器传递消息。这类广播也无法截断。
**LocalBroadcastManager.sendBroadcast
广播只能在应用程序内部进行传递，并且广播接收器也只能接收到来自本应用程序发出的广播。 这个方法比全局广播更高效（不需要Interprocess communication，IPC），而且不需要担心其它App会收到你的广播以及其他安全问题。
广播与权限
发送带着权限的广播
当你调用sendBroadcast(Intent, String)或sendOrderedBroadcast(Intent, String, BroadcastReceiver, Handler, int, String, Bundle)时，你可以指定一个权限。
接收器在manifest中申请了相应权限时才能收到这个广播。
例如发送一个带着权限的广播
arduino复制代码sendBroadcast(new Intent("com.example.NOTIFY"), Manifest.permission.SEND_SMS);

接收广播的app必须注册相应的权限
ini复制代码<uses-permission android:name="android.permission.SEND_SMS"/>

当然也可以使用自定义。在 manifest 中使用 permission 标签
ini复制代码<permission android:name="custom_permission" />

添加后编译一下。即可调用Manifest.permission.custom_permission
接收带权限的广播
若注册广播接收器时申明了权限，那么只会接收到带着相应权限的广播。
在配置文件中声明权限，程序才能访问一些关键信息。 例如允许查询系统网络状态。
xml复制代码<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>

<!-- 机器开机广播 -->
<uses-permission android:name="android.permission.BOOT_COMPLETED">

如果没有申请权限，程序可能会意外关闭。
使用示例
发送和接收广播。分为发送和接收方2个App。
使用带权限的广播。系统权限与自定义权限。 使用权限需要在AndroidManifest.xml中声明。如果是自定义权限，需要先添加自定义权限。
xml复制代码<!-- 自定义的权限  给广播用 -->
<permission android:name="com.rust.permission_rust_1" />
<uses-permission android:name="com.rust.permission_rust_1" />

发送广播时带上权限声明。接收方（不论是否己方App）需要在AndroidManifest.xml中申请权限。 注册接收器时也需要声明权限。
发送不带权限的有序广播
ini复制代码Intent intent = new Intent(MSG_PHONE);
sendOrderedBroadcast(intent, null);
Log.d(TAG, "[RustFisher-App1] 发送不带权限的有序广播, " + intent.getAction());

发送方App1代码
scss复制代码private static final String TAG = "rustApp";
    public static final String MSG_PHONE = "msg_phone";
    public static final String PERMISSION_RUST_1 = "com.rust.permission_rust_1";

        // onCreate注册广播接收器
        registerReceiver(mStandardReceiver1, makeIF());
        registerReceiver(mStandardReceiver2, makeIF());
        registerReceiver(mStandardReceiver3, makeIF());

        registerReceiver(mStandardReceiverWithPermission, makeIF(),
                Manifest.permission.permission_rust_1, null);  // 带上权限

        LocalBroadcastManager.getInstance(getApplicationContext())
                .registerReceiver(mLocalReceiver1, makeIF());
        LocalBroadcastManager.getInstance(getApplicationContext())
                .registerReceiver(mLocalReceiver2, makeIF());
        LocalBroadcastManager.getInstance(getApplicationContext())
                .registerReceiver(mLocalReceiver3, makeIF());

        // 解除接收器
        unregisterReceiver(mStandardReceiver1);
        unregisterReceiver(mStandardReceiver2);
        unregisterReceiver(mStandardReceiver3);

        unregisterReceiver(mStandardReceiverWithPermission);

        LocalBroadcastManager.getInstance(getApplicationContext())
                .unregisterReceiver(mLocalReceiver1);
        LocalBroadcastManager.getInstance(getApplicationContext())
                .unregisterReceiver(mLocalReceiver2);
        LocalBroadcastManager.getInstance(getApplicationContext())
                .unregisterReceiver(mLocalReceiver3);

    // 发送标准广播
    private void sendStandardBroadcast() {
        Intent intent = new Intent(MSG_PHONE);
        sendBroadcast(intent);
        Log.d(TAG, "[RustFisher-App1] Dispatcher 发送标准广播");
    }

    // 发送带权限的标准广播
    private void sendStandardBroadcastWithPermission() {
        Intent intent = new Intent(MSG_PHONE);
        sendBroadcast(intent, PERMISSION_RUST_1);
        Log.d(TAG, "[RustFisher-App1] Dispatcher 发送带权限的标准广播");
    }

    // 发送本地广播
    private void sendAppLocalBroadcast() {
        Intent intent = new Intent(MSG_PHONE);
        LocalBroadcastManager.getInstance(getApplicationContext()).sendBroadcast(intent);
        Log.d(TAG, "[RustFisher-App1] Dispatcher 发送本地广播");
    }

    private IntentFilter makeIF() {
        IntentFilter intentFilter = new IntentFilter(MSG_PHONE);
        intentFilter.addAction(Intent.ACTION_TIME_TICK);
        intentFilter.addAction(Intent.ACTION_TIME_CHANGED);
        return intentFilter;
    }

    // 标准接收器  用context来注册
    private BroadcastReceiver mStandardReceiver1 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 标准接收器1 收到: " + intent.getAction());
        }
    };

    // 标准接收器  用context来注册
    private BroadcastReceiver mStandardReceiver2 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 标准接收器2 收到: " + intent.getAction());
            if (intent.getAction().endsWith(MSG_PHONE)) {
                abortBroadcast(); // 截断有序广播
                Log.d(TAG, "[RustFisher-App1] 标准接收器2截断有序广播 " + intent.getAction());
            }
        }
    };

    // 标准接收器  用context来注册
    private BroadcastReceiver mStandardReceiver3 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 标准接收器3 收到: " + intent.getAction());
        }
    };

    // 注册的时候给它带权限  标准接收器
    private BroadcastReceiver mStandardReceiverWithPermission = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 带权限的标准接收器收到: " + intent.getAction());
        }
    };

    /**
 * 用LocalBroadcastManager来注册成为本地接收器
 * 收不到标准广播 - 不论是本app发出的还是别的地方发出来的
 */
    private BroadcastReceiver mLocalReceiver1 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 本地接收器1 收到: " + intent.getAction());
        }
    };

    private BroadcastReceiver mLocalReceiver2 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 本地接收器2 收到: " + intent.getAction());
        }
    };

    private BroadcastReceiver mLocalReceiver3 = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[RustFisher-App1] 本地接收器3 收到: " + intent.getAction());
        }
    };

接收方App2代码
xml复制代码<!-- 自定义的权限  给广播用 -->
<permission android:name="com.rust.permission_rust_1" />
<uses-permission android:name="com.rust.permission_rust_1" />

java复制代码public static final String MSG_PHONE = "msg_phone";

    // onCreate里注册接收器
    registerReceiver(mDefaultReceiver, makeIF());
    LocalBroadcastManager.getInstance(getApplicationContext())
            .registerReceiver(mLocalReceiver, makeIF());

    unregisterReceiver(mDefaultReceiver);
    LocalBroadcastManager.getInstance(getApplicationContext())
            .unregisterReceiver(mLocalReceiver);

    private BroadcastReceiver mDefaultReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[App2] standard receive: " + intent.getAction());
        }
    };

    private BroadcastReceiver mLocalReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.d(TAG, "[App2] local receive: " + intent.getAction());
        }
    };

    private IntentFilter makeIF() {
        IntentFilter intentFilter = new IntentFilter(MSG_PHONE);
        intentFilter.addAction(Intent.ACTION_TIME_TICK);
        intentFilter.addAction(Intent.ACTION_TIME_CHANGED);
        return intentFilter;
    }

使用LocalBroadcastManager发出的本地广播，另一个App是接收不到的。 要收到本地广播，同样需要LocalBroadcastManager来注册接收器。
可以把本地广播看成是一个局部的，App内的广播体系。
实验中我们注意到，Intent.ACTION_TIME_TICK广播是可以截断的。
监听屏幕亮灭
使用广播监听设备屏幕亮灭状态。这个是系统发出来的广播。
使用的action是

Intent.ACTION_SCREEN_ON 亮屏
Intent.ACTION_SCREEN_OFF 灭屏

scss复制代码    private void registerScreenListener() {
        IntentFilter filter = new IntentFilter();
        filter.addAction(Intent.ACTION_SCREEN_ON);
        filter.addAction(Intent.ACTION_SCREEN_OFF);
        registerReceiver(mScreenReceiver, filter);
    }

    private BroadcastReceiver mScreenReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            final String action = intent.getAction();
            if (Intent.ACTION_SCREEN_ON.equals(action)) {
                // 屏幕亮
            } else if (Intent.ACTION_SCREEN_OFF.equals(action)) {
                // 屏幕灭
            }
        }
    };

Broadcast 相关面试题
1. 广播传输的数据是否有限制，是多少，为什么要限制？

广播是通过Intent携带需要传递的数据的
Intent是通过Binder机制实现的
Binder对数据大小有限制，不同room不一样，一般为1M

2. 广播的分类？

标准广播：通过context. sendBroadcast或者context. sendBroadcastAsUser发送给当前系统中所有注册的接受者，也就是只要注册了就会接收到。应用在需要通知各个广播接收者的情况下使用，如开机启动。
有序广播：接收者按照优先级处理广播，并且前面处理广播的接受者可以中止广播的传递，一般通过context. sendOrderedBroadcast或者context.sendOrderedBroadcastAsUser，在需要有特定拦截的场景下使用，如黑名单短信、电话拦截。
粘性广播：可以发送给以后注册的接受者，意思是系统会将前面的粘性广播保存在AMS中，一旦注册了与以保存的粘性广播符合的广播，在注册结束后会立即收到广播，一般通过context. sendStickyBroadcast或context.sendStickyOrderedBroadcast来发送，从字面上看，可以看出来粘性广播也分为普通粘性广播和有序粘性广播。
本地广播：发出的广播只能在应用程序内部进行传递，广播接收器也只能接受来自本应用程序的广播。
全局广播：系统和广播，发出的广播可以被其他任何应用程序接收到，并且也可以接受到其他任何应用程序的广播。

3. 广播的使用场景，使用方式
广播是一种广泛运用的在应用程序之间传输信息的机制，主要用来监听系统或者应用发出的广播信息，然后根据广播信息作为相应的逻辑处理，也可以用来传输少量、频率低的数据。
在实现开机启动服务和网络状态改变、电量变化、短信和来电时通过接收系统的广播让应用程序作出相应的处理。
使用：
typescript复制代码//在AndroidManifest中静态注册
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.hp.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>

//动态注册，在代码中注册
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    mIntentFilter = new IntentFilter();
    //添加广播想要监听的类型，监听网络状态是否发生变化
    mIntentFilter.addAction("android.net.conn.CONNECTIVITY_CHANGE");
    mNetworkChangeReceiver = new NetworkChangeReceiver();
    //注册广播
    registerReceiver(mNetworkChangeReceiver, mIntentFilter);
}

@Override
protected void onDestroy() {
    super.onDestroy();
    //取消注册广播接收器
    unregisterReceiver(mNetworkChangeReceiver);
}

//发送广播，同样通过Intent
Intent intent = new Intent("com.example.hp.broadcasttest.MY_BROADCAST");
//发送标准广播
sendBroadcast(intent);

//接收广播
public class MyBroadcastReceiver extends BroadcastReceiver {
    public MyBroadcastReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // TODO: This method is called when the BroadcastReceiver is receiving
        // an Intent broadcast.
        Toast.makeText(context, "received", Toast.LENGTH_SHORT).show();
        //将这条广播截断
//        abortBroadcast();
    }
}

4. BroadcastReceiver，LocalBroadcastReceiver 区别
广播接收者：
复制代码（1）用于应用间的传递消息
（2）由于跨应用，存在安全问题

本地广播接收者：
复制代码（1）广播数据在本应用范围内传播。  
（2）不用担心别的应用伪造广播。  
（3）比发送全局广播更高效、安全。  
（4）无法使用静态注册  

5. 在 manifest 和代码中如何注册和使用 BroadcastReceiver

在AndroidManifest中静态注册，然后直接使用。
代码中，通过registerReceiver来注册。
注册发送后，在BroadcastReceiver（自定义一个接收器继承自BroadcastReceiver）的onReceive中接收广播并处理广播。

6. 广播引起 anr 的时间限制

前台广播：BROADCAST_FG_TIMEOUT = 10s
后台广播：BROADCAST_BG_TIMEOUT = 60s




## Broadcast 源码分析


https://juejin.cn/post/6844903990329606158#heading-15










