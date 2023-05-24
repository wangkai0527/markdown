[TOC]

# Gradle

Android Studio 构建系统以 Gradle 为基础，并且 Android Gradle 插件添加了几项专用于构建 Android 应用的功能。虽然 Android 插件通常会与 Android Studio 的更新步调保持一致，但插件（以及 Gradle 系统的其余部分）可独立于 Android Studio 运行并单独更新。

[Android Gradle 插件版本说明](https://developer.android.com/studio/releases/gradle-plugin)

## Android Gradle 插件

Google开发的，Android Studio中的插件，用来调用本地的Gradle构建工具。

安装Android Studio时默认会自带此插件。 
eg:`D:\Android\Android Studio-2021.2.1.15\plugins\gradle\lib`。

### 指定方式
1. 在 Android Studio 的 `File` > `Project Structure` > `Project` 菜单中的`Android Gradle Plugin Version`指定插件版本。
2. 在顶层 build.gradle 文件中进行指定。

```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
        google()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.6.2'
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        jcenter()
        google()
        maven { url "https://raw.githubusercontent.com/Pgyer/mvn_repo_pgyer/master" }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```



## Gradle

开源的自动化构建工具。
[Gradle官网地址](http://gradle.org)

作用：
1. 打包项目(ant)
2. 引入第三方库(maven)
3. 写脚本来定制化自己的操作(groovy)

安装Android Studio时默认会安装Gradle在`C:/Users/用户名/.gradle`此路径下， eg:`C:/Users/1499/.gradle/wrapper/dists/gradle-5.6.4-all`。

### 指定方式

1. 在 Android Studio 的 `File` > `Project Structure` > `Project` 菜单中的`Gradle Version`指定 Gradle 版本。
2. 在 Android Studio 的 `File` > `Settings` > `Build,Execution,Deployment` > `Gradle` 菜单中的`Use Gradle from:`如果选择Specified location，路径直接指定到选择的gradle文件夹。
3. 在 Android Studio 的 `File` > `Settings` > `Build,Execution,Deployment` > `Gradle` 菜单中的`Use Gradle from: `如果选择 `gradle-wrapper.properties` file，会通过在 `gradle/wrapper/gradle-wrapper.properties` 文件中修改 Gradle 分发引用来指定。

Sync Now或者在Terminal执行命令gradlew build

```
#Wed Feb 26 15:52:57 CST 2020
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
#默认的官网下载Gradle地址，如果本地没有会从官网自动下载，需要翻墙
distributionUrl=https\://services.gradle.org/distributions/gradle-5.6.4-all.zip
#本地下载后的Gradle地址
distributionUrl=file:///C:/Users/1499/.gradle/wrapper/dists/gradle-5.6.4-all
#两者选其一即可
```

下表列出了各个 Android Gradle 插件版本所需的 Gradle 版本。为了获得最佳性能，您应使用 Gradle 和插件这两者的最新版本。

|   插件版本	    |   所需的 Gradle 版本  |
|   :---            |   :---            |
|   1.0.0 - 1.1.3	|   2.2.1 - 2.3     |
|   1.2.0 - 1.3.1	|   2.2.1 - 2.9     |
|   1.5.0	        |   2.2.1 - 2.13    |
|   2.0.0 - 2.1.2	|   2.10 - 2.13     |
|   2.1.3 - 2.2.3	|   2.14.1 - 3.5    |
|   2.3.0+	        |   3.3+            |
|   3.0.0+	        |   4.1+            |
|   3.1.0+	        |   4.4+            |
|   3.2.0 - 3.2.1	|   4.6+            |
|   3.3.0 - 3.3.3	|   4.10.1+         |
|   3.4.0 - 3.4.3	|   5.1.1+          |
|   3.5.0 - 3.5.4	|   5.4.1+          |
|   3.6.0 - 3.6.4	|   5.6.4+          |
|   4.0.0+	        |   6.1.1+          |
|   4.1.0+	        |   6.5+            |
|   4.2.0+	        |   6.7.1+          |
|   7.0	            |   7.0+            |
|   7.1	            |   7.2+            |
|   7.2	            |   7.3.3+          |


## [Android Gradle 插件和 Android Studio 兼容性](https://developer.android.google.cn/studio/releases/gradle-plugin.html#agp-studio-compatibility)

Android Studio 构建系统以 Gradle 为基础，并且 Android Gradle 插件添加了几项专用于构建 Android 应用的功能。

|   Android Studio 版本	    |   所需插件版本  |
|   :---                    |   :---        |
|   Arctic Fox / 2020.3.1	|   3.1-7.0     |
|   Bumblebee / 2021.1.1	|   3.2-7.1     |
|   Chipmunk / 2021.2.1	    | 	3.2-7.2     |







## Gradle 执行cmd脚本

https://blog.csdn.net/weixin_42863849/article/details/121611393











## Gradle sync failed

-----------------------------------------------------------------------
>Gradle sync failed: Cause: com.android.build.gradle.api.BaseVariant.getOutputs()Ljava/util/List;
Consult IDE log for more details (Help | Show Log) (8s 123ms)

- 把项目中依赖的ButterKnife降级到8.4.0

        classpath 'com.jakewharton:butterknife-gradle-plugin:8.4.0'

- 把Gradle plugin版本降低至2.3.3 重新编译下就可以了

-----------------------------------------------------------------------

>Unable to resolve dependency for ':app@signingConfigs/compileClasspath': Could not resolve project :appCommon.

* 把项目中的signingConfigs节点删除掉就好了

-----------------------------------------------------------------------

> Eeecution failed for task ':app:transformDexArchiveWithExterLibsDexMergeForDebug'.

+ 将电脑中的.gradle目录删除掉（清除掉gradle缓存）重新build

-----------------------------------------------------------------------

> Error:(56, 0) Cannot set the value of read-only property 'outputFile' for ApkVariantOutputImpl_Decorated{apkData=Main{type=MAIN, fullName=debug, filters=[]}} of type com.android.build.gradle.internal.api.ApkVariantOutputImpl.
\<a href="openFile:D:\eclipseCode\ipay-android\xinlebao\build.gradle" rel="external nofollow" >Open File\</a>

    android.applicationVariants.all { variant ->
        variant.outputs.all {
            outputFileName = "xinlebao_${defaultConfig.versionName}_${releaseTime()}.apk"
        }
    }

-----------------------------------------------------------------------

>Error:java.util.concurrent.ExecutionException: com.android.tools.aapt2.Aapt2Exception: AAPT2 error: check logs for details

+ 在gradle.properties中关闭APPT2 编译

        android.enableAapt2=false

-----------------------------------------------------------------------
apt插件问题

>Error:Cannot choose between the following configurations of project :mylibrary:

- debugApiElements
- debugRuntimeElements
- releaseApiElements
- releaseRuntimeElements
All of them match the consumer attributes:

1. 在project的build.gradle中删除
classpath 'com.neenbedankt.gradle.plugins:android-apt:1.8'
2. 在module的build.gradle中删除
apply plugin: 'android-apt'
3. 将module的build.gradle文件中的dependency
apt 'com.jakewharton:butterknife-compiler:8.1.0'
//改为
annotationProcessor 'com.jakewharton:butterknife-compiler:8.1.0'

-----------------------------------------------------------------------

>Gradle sync failed: Support for builds using Gradle versions older than 2.6 was removed in tooling API version 5.0. You are currently using Gradle version 2.2.1. You should upgrade your Gradle build to use Gradle 2.6 or later.
+ 修改Gradle版本号

-----------------------------------------------------------------------

>Gradle sync failed: Could not find com.android.tools.build:gradle:4.1.1.

可能是Jar包仓库下载的不够完善
+ 在顶层 build.gradle 文件中的buildscript项和allprojects中添加google()

-----------------------------------------------------------------------

>Error:Cause: buildToolsVersion is not specified.
+ 在build.gradle中添加：
buildToolsVersion “25.0.3”

Android Studio3.0，Google把buildToolsVersion构建工具的版本给“干掉了”，在以前的版本中，buildToolsVersion也会给项目的构建带来很多错，

-----------------------------------------------------------------------

>Could not find method dependencyResolutionManagement() for arguments ...

+ 提高Gradle和插件的版本，这是因为导入的项目是在更高版本下编译的。

[settings.gradle 文件中的代码库设置](https://developer.android.google.cn/studio/releases/gradle-plugin.html#settings-gradle)

更新Android Gradle插件7.1.0版本后，顶层build.gradle写法：
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '7.2.1' apply false
    id 'com.android.library' version '7.2.1' apply false
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

更新Android Gradle插件7.1.0版本后，settings.gradle写法：

```
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "TestGradle"
include ':app'
```

+ 如果不想升级，需要修改build.gradle和settings.gradle成旧版写法

更新Android Gradle插件7.1.0版本前，build.gradle写法：
```
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    repositories {
        jcenter()
        google()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.6.2'
    }
}

allprojects {
    repositories {
        jcenter()
        google()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

更新Android Gradle插件7.1.0版本前，settings.gradle写法：
```
include ':app'
rootProject.name = "TestGradle"
```

-----------------------------------------------------------------------

>Invalid revision: 3.18.1-g262b901-dirty

+ 提高Gradle和插件的版本，使其支持cmake 3.18。因为native工程既然要求cmake 3.18，说明用到了3.18之后的cmake特性

-----------------------------------------------------------------------

起因 :需要配置 Android 高性能音频 Oboe 函数库 , 参考 https://github.com/google/oboe/blob/master/docs/GettingStarted.md 文档 , 使用预构建的二进制库和头文件 , 需要配置如下配置 :

    android {
        buildFeatures {
            prefab true
        }
    }

结果出现以下一系列报错信息 , 这是由于 Android Studio 版本 , Gradle 版本 , Gradle 插件版本 配置不匹配导致 ;

报错信息 1 :
>Could not find method buildFeatures() for arguments...

报错信息 2 :
>Caused by: groovy.lang.MissingMethodException: No signature of method: build_90npnf01wae3avkxhn7ts5vqn.android() is applicable for argument types:...

如果要配置 buildFeatures , 必须使用 4.1 以上的 Android Studio 版本 , 这是支持 buildFeatures 的最低版本 ;
+ Android Studio 版本 : 4.1
+ Gradle 版本 : 最低版本 6.6.1
+ Gradle 插件版本配置 : 最低版本 4.1.0

-----------------------------------------------------------------------

Clean Project:
Execution failed for task ':app:externalNativeBuildCleanCustomDebugType'.

rm -rf app/.cxx
rm -rf app/.externalNativeBuild
rm -rf app/build

-----------------------------------------------------------------------

-----------------------------------------------------------------------

cvc-complex-type.2.4.a: 发现了以元素 ‘base-extension‘ 开头的无效内容。应以 ‘{layoutlib}‘ 之一开头

官方其他信息补充：
跟随 Arctic Fox 更新的其中一个重点就是 AGP (Android Gradle plugin) 7.0 的调整 … 
使用 AGP 7.0 构建时需要 JDK 11 才能运行 Gradle … 
并且只要你更新到 Android Studio Arctic Fox ，它是直接捆绑了 JDK 11 并将 Gradle 配置为默认使用它，所以大多数情况下，如果你本地配置正常，是可以直接使用 AGP 7.0的升级。 … 
在 Project Structure 的 SDK Location 栏目，可以看到 JDK 的配置位置已经被移动到Gradle Settings …

AGP 使用 7.0 之前的版本，那么他需要依赖老的 JDK 1.8。

-----------------------------------------------------------------------

-----------------------------------------------------------------------
aapt: error: resource android:color/system_neutral1_1000 not found

compileSdkVersion 31
targetSdkVersion 31
-----------------------------------------------------------------------





