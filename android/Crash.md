[TOC]

# Crash




[Android下打印调试堆栈方法](https://blog.csdn.net/freshui/article/details/9456889)

https://juejin.cn/post/7018153262482194446





# Downtime



死机问题我们分两步走
1：先尝试关闭子系统异常重启
/sys/bus/msm_subsys/devices
restart_leve	system的话就是子系统发生异常就会重启整个系统,改为related
cat /sys/bus/msm_subsys/devices/subsys0/name					modem
cat /sys/bus/msm_subsys/devices/subsys1/name					adsp
cat /sys/bus/msm_subsys/devices/subsys2/name					cdsp


2：手机在关机后时候会进入download模式。如果会我们就需要通过高通的工具QPSTdump内存分析
adb shell "echo 1 > /sys/module/msm_poweroff/parameters/download_mode"
adb shell "cat /sys/module/msm_poweroff/parameters/download_mode"
这个就是使能ramdump的节点

vendor/etc/init/hw/init.msm.usb.configfs.rc
adb shell "getprop sys.usb.config"
adb shell "setprop sys.usb.config diag"


复现之后 如果是底层重启，手机会进入黑屏状态，连上linux lsusb 查看 会有一个 900e 或者 9091的设备
此时用高通qpst configuration 抓dump 就行了


只是上层重启
上层系统有watchdog，当超时会重启system服务
adb shell "getprop persist.sys.crashOnWatchdog"
adb shell "setprop persist.sys.crashOnWatchdog true"
adb shell "getprop persist.sys.crashOnWatchdog"
这样的话，发生watchdog问题的时候会自动进入到抓ramdump的模式下，然后就能最大限度的保留现场，以便后续分析。


QCAP
zhaojunhai@huaqin.com
yao105a~Z








adb pull data/tombstones/tombstone_09
E:\baidu\android\AS\Sdk\ndk\21.1.6352462\ndk-stack -sym D:\Desktop\adb_xroot\test_ndk_stack -dump D:\Desktop\adb_xroot\test_ndk_stack\tombstone_09

backtrace:
E:\baidu\android\AS\Sdk\ndk\21.1.6352462\toolchains\llvm\prebuilt\windows-x86_64\bin\x86_64-linux-android-nm.exe D:\Desktop\adb_xroot\test_ndk_stack\base\lib\arm64-v8a\libUdJni.so > nm_func.txt
E:\baidu\android\AS\Sdk\ndk\21.1.6352462\toolchains\llvm\prebuilt\windows-x86_64\bin\x86_64-linux-android-addr2line.exe -f -e D:\Desktop\adb_xroot\test_ndk_stack\base\lib\arm64-v8a\libUdJni.so 000000000005157c







int create_compute_ftr_thread(pThreadInfo_t pInfo) {
#if 0
    pthread_attr_t attr;
    struct sched_param scparam;
    scparam.sched_priority = sched_get_priority_max(SCHED_RR);
    pthread_attr_init(&attr);
    pthread_attr_setschedpolicy(&attr, SCHED_RR);
    pthread_attr_setschedparam(&attr, &scparam);
    return pthread_create(&(pInfo->pid), &attr, compute_ftr_thread_func, pInfo);
#else
    return pthread_create(&(pInfo->pid), NULL, compute_ftr_thread_func, pInfo);
#endif
}



//        List<String> thermalList = ThermalInfoUtil.getThermalInfo();
//        Log.d(TAG, "thermalList:");
//        for (int i = 0; i < thermalList.size(); i++) {
//            Log.d(TAG, i + ":" + thermalList.get(i));
//        }





1、try catch

try {
    if ((mDialog != null) && mDialog.isShowing()) {
      mDialog.dismiss();
    }
} catch (Exception e) {
       
} finally {
    mDialog = null;
}  
2、判断上下文，有效时再关闭

if (progressDialog != null) {
       if (progressDialog.isShowing()){
           Context context = ((ContextWrapper)progressDialog.getContext()).getBaseContext();
       if(context instanceof Activity) {
           if(!((Activity)context).isFinishing() && !((Activity)context).isDestroyed())
                  progressDialog.dismiss();
           } else {
                  progressDialog.dismiss();
           }
        }
       progressDialog = null;
	}
}




修改init.qcom.test.rc后，里面语句没有执行，原因可能是init.qcom.test.rc的文件权限不一致

:/ # ls -Zl vendor/etc/init/hw/*
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 81930 2009-01-01 00:00 vendor/etc/init/hw/init.msm.usb.configfs.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 40540 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.dl.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0  7670 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.factory.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 42210 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.rc
-rw-rw-rw- 1 root root u:object_r:vendor_configs_file:s0  2224 2022-01-07 15:24 vendor/etc/init/hw/init.qcom.test.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 92933 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.usb.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 13335 2009-01-01 00:00 vendor/etc/init/hw/init.target.dl.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 13926 2009-01-01 00:00 vendor/etc/init/hw/init.target.rc
:/ # chmod 0644 vendor/etc/init/hw/init.qcom.test.rc
chmod: chmod 'vendor/etc/init/hw/init.qcom.test.rc' to 100644: Read-only file system

:/ # exit

D:\Desktop\adb_xroot>adb remount
remount succeeded

D:\Desktop\adb_xroot>adb shell
:/ # chmod 0644 vendor/etc/init/hw/init.qcom.test.rc
:/ # ls -Zl vendor/etc/init/hw/*
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 81930 2009-01-01 00:00 vendor/etc/init/hw/init.msm.usb.configfs.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 40540 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.dl.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0  7670 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.factory.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 42210 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0  2224 2022-01-07 15:24 vendor/etc/init/hw/init.qcom.test.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 92933 2009-01-01 00:00 vendor/etc/init/hw/init.qcom.usb.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 13335 2009-01-01 00:00 vendor/etc/init/hw/init.target.dl.rc
-rw-r--r-- 1 root root u:object_r:vendor_configs_file:s0 13926 2009-01-01 00:00 vendor/etc/init/hw/init.target.rc







init.qcom.test.rc里面有自动启动fingerprintd，第一次push fingerprintd可以正常启动，后面修改fingerprintd不能正常启动，因为fingerprintd的标签不对，需要restorecon

:/ # ls -Zl system/bin/fingerprintd
-rwxrwxrwx 1 root shell u:object_r:system_file:s0             14248 2022-01-29 10:12 system/bin/fingerprintd

:/ # restorecon system/bin/fingerprintd
SELinux: Loaded file_contexts
SELinux: Could not set context for /system/bin/fingerprintd:  Read-only file system
restorecon: restorecon failed: system/bin/fingerprintd: Read-only file system

:/ # mount -o rw,remount /

:/ # restorecon system/bin/fingerprintd
SELinux: Loaded file_contexts

:/ # ls -Zl system/bin/fingerprintd
-rwxrwxrwx 1 root shell u:object_r:fingerprintd_exec:s0       14248 2022-01-29 10:12 system/bin/fingerprintd






















