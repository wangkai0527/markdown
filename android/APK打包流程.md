[TOC]

# APK打包流程

## 创建android项目

### android

### android.bat create project

Android SDK工具软件包提供了一个android.bat批处理工具，位于以下位置：android_sdk/tools/

>Action "create project":
Creates a new Android project.
Options:
-n --name     应用程序的名字
-t --target   SDK Target ID
-p --path     应用程序的工作目录
-k --package  应用程序的包名
-a --activity 默认创建的Activity的名字

执行命令 `android create project -n LBSDemo -p LBSDemo -k com.lbs -a LBSDemo -t 7`

Note: As this command is deprecated now so it is already removed from sdk with tools version 25.3.0.

## 生成R.java










E:\Android\sdk\tools>android --help

       Usage:

       android [global options] action [action options]

       Global options:

  -h --help       : Help on a specific command.

  -v --verbose    : Verbose mode, shows errors, warnings and all messages.

     --clear-cache: Clear the SDK Manager repository manifest cache.

  -s --silent     : Silent mode, shows errors only.

                     Valid

                     actions

                     are

                     composed

                     of a verb

                     and an

                     optional

                     direct

                     object:

-    sdk              : Displays the SDK Manager window.

                      SDK管理界面

-    avd              : Displays the AVD Manager window.

                       AVD管理界面

-   list              : Lists existing targets or virtual devices.

                       列出AVD和手机设备（已经连接上，并开启了adb服务的）

-   list avd          : Lists existing Android Virtual Devices.

                      列出AVD

-   list target       : Lists existing targets.

                      列出手机设备

-   list sdk          : Lists remote SDK repository.

                      获取SDK的代码库信息

- create avd          : Creates a new Android Virtual Device.

                       创建AVD

-   move avd          : Moves or renames an Android Virtual Device.

                       移动或重命名AVD

- delete avd          : Deletes an Android Virtual Device.

                       删除AVD

- update avd          : Updates an Android Virtual Device to match the folders

                        of a new SDK.

                       更新AVD信息，如文件配置目录

- create project      : Creates a new Android project.

                        创建android项目

- update project      : Updates an Android project (must already have an

                        AndroidManifest.xml).

- create test-project : Creates a new Android project for a test package.

- update test-project : Updates the Android project for a test package (must

                        already have an AndroidManifest.xml).

- create lib-project  : Creates a new Android library project.

                   创建Lib类型的Android项目

- update lib-project  : Updates an Android library project (must already have

                        an AndroidManifest.xml).

- create uitest-project: Creates a new UI test project.

- update adb          : Updates adb to support the USB devices declared in the

                        SDK add-ons.

                        更新ADB，以支持USB设备

- update sdk          : Updates the SDK by suggesting new platforms to install

                        if available.

 

想获取具体命令更详细的参数，如创建AVD的详细参数：

输入 android create avd

 

E:\Android\sdk\tools>android create avd

Error: The parameters --name, --target must be defined for action 'create avd'

       Usage:

       android [global options] create avd [action options]

       Global options:

  -h --help       : Help on a specific command.

  -v --verbose    : Verbose mode, shows errors, warnings and all messages.

     --clear-cache: Clear the SDK Manager repository manifest cache.

  -s --silent     : Silent mode, shows errors only.

                     Action "create avd":

  Creates a new Android Virtual Device.

Options:

  -c --sdcard  : Path to a shared SD card image, or size of a new sdcard for

                 the new AVD.

创建SDCARD，-c后面是SDCARD的本地路径或者大小，如果是路径，则指向一个sdcard.img,否则在默认位置产生一个新指定大小(MB)的sdcard.img

当前目录下有个mksdcard.exe可以创建sdcard文件

  -n --name    : Name of the new AVD. [required]  AVD的命名【必要】

  -a --snapshot: Place a snapshots file in the AVD, to enable persistence.

              快照功能，当下次重新启动AVD时可以直接加载退出时的状态和应用

  -p --path    : Directory where the new AVD will be created.

              AVD文件所在目录

  -f --force   : Forces creation (overwrites an existing AVD)

              覆盖已有AVD文件

  -s --skin    : Skin for the new AVD.

              指定skin,就是分辨率参数

  -t --target  : Target ID of the new AVD. [required]

             就是API Level【必要】

  -b --abi     : The ABI to use for the AVD. The default is to auto-select the

                 ABI if the platform has only one ABI for its system images.

            硬件配置，system.img,如指定机型HTC的system.img，

示例:android create avd -n AVD23 -t 10 -s WVGA –c 512

创建一个Android 2.3.3的AVD,支持SDCARD，大小512MB，屏幕分辨率WVGA，命名为AVD23

提示：可以通过android avd调出avd  GUI编辑界面来熟悉每个命令对应的参数。









第一步 生成R.java资源文件

       @aapt package -f -m -J %ROOT%\gen -S %PATH_PROJECT%\res -I %PATH_SDK_PLATFORM%\android.jar -M %PATH_PROJECT%\AndroidManifest.xml
      
这里我们自动将资源文件res(-S)生成 R.java文件并存放到 gen目录下(-m -J)，当然我们还需要android.jar的API库(-I)跟android的配置文件AndroidManifest.xml(-M)的协助，并且设置为强制覆盖不询问形式(-f)

第二步 编译java文件

      @javac -encoding GB18030 -target 1.6 -bootclasspath %PATH_SDK_PLATFORM%\android.jar -d %ROOT%\classes %PATH_PROJECT%\src\%PATH_PACKAGE%\*.java %ROOT%\gen\%PATH_PACKAGE%\R.java

生 成class文件，我们需要借助jdk来完成。将java文件与生成的资源ID文件R.java一起编译成.class文件，并存放到classes目录中(-d)。

第三步 打包.class文件

         @call dx --dex --output=%ROOT%\classes.dex %ROOT%\classes
        
将编译好的.class文件打包成classes.dex二进制执行文件。

第四步 打包资源文件

       @aapt package -f -M %PATH_PROJECT%\AndroidManifest.xml -S %PATH_PROJECT%\res -A %ROOT%\assets -I %PATH_SDK_PLATFORM%\android.jar -F %ROOT%\resources.ap_

选中AndroidManifest.xml (-M), 资源文件夹res(-S) 跟 assets(-A)，加上Android的API库(-I)，一起打包输出到resources.ap_ 文件中(-F)。

第五步 打包APK文件

        @call apkbuilder %ROOT%\unsigned.apk -u -z %ROOT%\resources.ap_ -f %ROOT%\classes.dex
       
将资源文件包resources.ap_跟二进制文件包classes.dex一起打包成未签名的apk压缩包 unsign.apk。

第六步 签名

      @java -jar %PATH_SIGN%\signapk.jar %PATH_SIGN%\testkey.x509.pem %PATH_SIGN%\testkey.pk8 unsigned.apk android.apk

这里我们使用的是platform里面两个testkey，调用signapk.jar来执行，最终生成android.apk的签名apk包，这时候就可以安装到android系统中了。

目录结构
 
         +autoPackage (根目录)
        
 
         -+androidProject (工程目录)
        
 
         ---(一个普通的工程目录，不赘述)
        
 
         -+platform (Android.jar目录)
        
 
         ---android.jar (用了Android 2.2的platform)
        
 
         -+sign (签名文件目录)
        
 
         ---signapk.jar (在源码\platform\build\tools\signapk目录下)
        
 
         ---testkey.pk8 (在源码\platform\build\target\product\security目录下)
        
 
         ---testkey.x509.pem (在源码\platform\build\target\product\security目录下)
        
 
         --run.bat (批处理文件)
        
 
         --icon-36.png (低分辨率ICON)
        
 
         --icon-48.png (中分辨率ICON)
        
 
         --icon-72.png (高分辨率ICON)
        
对应需求、目录及流程写出批处理脚本 run.bat

       @title Auto package And Sign
      
       @echo *******************************  start *********************************
 
       @set ROOT=C:\autoPackage
 
       @set PATH_PACKAGE=com\isfeel\autopackage
 
       @set PATH_SDK_PLATFORM=%ROOT%\platform
 
       @set PATH_SIGN=%ROOT%\sign
 
       @set PATH_PROJECT=%ROOT%\androidProject
      
 
       @del %ROOT%\android.apk
      
       @mkdir bin gen classes assets


       @copy /Y icon-72.png %PATH_PROJECT%\res\drawable-hdpi\icon.png
 
       @copy /Y icon-48.png %PATH_PROJECT%\res\drawable-mdpi\icon.png
      
       @copy /Y icon-36.png %PATH_PROJECT%\res\drawable-ldpi\icon.png
      

       @aapt package -f -m -J  %ROOT%\gen -S %PATH_PROJECT%\res -I %PATH_SDK_PLATFORM%\android.jar -M %PATH_PROJECT%\AndroidManifest.xml
      
 
       @javac -encoding GB18030 -target 1.6 -bootclasspath %PATH_SDK_PLATFORM%\android.jar -d %ROOT%\classes %PATH_PROJECT%\src\%PATH_PACKAGE%\*.java %ROOT%\gen\%PATH_PACKAGE%\R.java 
      
 
       @call dx --dex --output=%ROOT%\classes.dex %ROOT%\classes
      
 
       @aapt package -f -M %PATH_PROJECT%\AndroidManifest.xml -S %PATH_PROJECT%\res -A %ROOT%\assets -I %PATH_SDK_PLATFORM%\android.jar -F %ROOT%\resources.ap_
         

       @call apkbuilder %ROOT%\unsigned.apk -u -z %ROOT%\resources.ap_ -f %ROOT%\classes.dex
      
 
       @java -jar %PATH_SIGN%\signapk.jar %PATH_SIGN%\testkey.x509.pem %PATH_SIGN%\testkey.pk8 unsigned.apk android.apk
      
 
       @del %ROOT%\unsigned.apk %ROOT%\classes.dex %ROOT%\resources.ap_
      
 
       @rmdir /s/q %ROOT%\bin %ROOT%\gen %ROOT%\classes %ROOT%\assets
      
 
       @echo *********************** finished success! ******************************
      
       @pause
      
       @exit



















1、生成R.java

2、编译*.java

3、生成classes.dex

4、将assets、res打包

5、生成未签名的apk

6、将apk签名

基于这些步骤，我们使用bat来一步一步完成，下面是一个脚本文件截图。

212833447.jpg

1、生成R.java

新建一个AntDemo工程，然后在该工程的根目录下编写第一步的bat脚本1_genR.bat，代码如下。

aapt package -f -m -J gen -S res -I D:/android-sdk-windows/platforms/android-16/android.jar -M AndroidManifest.xml

注：执行后生成R.java文件。用eclipse直接新建一个AntDemo工程后，gen目录下已经有R.java文件，如果再执行这个脚本会看到修改时间会改变。如果不确定的，可以将R.java删除，然后再执行脚本以便观看效果。

下面是脚本执行后的图。

212833448.jpg

2、编译*.java

编译时调用的是JDK下的javac命令，所以需要安装JDK，建议使用jdk1.6(1.7有可能出问题)。安装完成后将JDK所在目录下的bin目录设置成环境变量，然后编译脚本2_compile.bat.

mkdir bin

javac -target 1.6 -bootclasspath D:/android-sdk-windows/platforms/android-17/android.jar -d bin gen\com\example

\antdemo\*.java src\com\example\antdemo\*.java注：

(1)、mkdir bin是创建bin目录。

(2)、javac中的target指定的是jdk的版本。

(3)、android.jar需要对应到相应的版本，这里选的是android-17。

(4)、gen\com\example指定java文件所在的目录。

执行该脚本后，会在bin目录下生成一系列的class文件，如下图。

212833449.jpg

3、生成classes.dex

dx命令是一个dx.bat的脚本，位于android-sdk/plaform-tools下。为了能直接执行dx命令，需要将dx.bat所在的目录设置到环境变量中。下面是编写的脚本3_dex.bat.

dx --dex --output=G:\Code\Android\Workspace\AntDemo\bin\classes.dex G:\Code\Android\Workspace\AntDemo\bin

注：

(1)、output指定的是输出dex的位置。

(2)、G:\Code\Android\Workspace\AntDemo\bin指定的是class文件的目录。

执行脚本后生成的文件见下图。

212833450.jpg.jpg

4、将assets、res打包

编写4_package.bat，内容如下。

aapt package -f -A assets -S res -I D:/android-sdk-windows/platforms/android-17/android.jar -M AndroidManifest.xml -F bin/AntDemo

注：将assets和res中的文件打包，生成AntDemo文件。见下图。

212833450.jpg.jpg

5、生成未签名的apk

编写5_unsigned.bat，内容如下。

apkbuilder G:\Code\Android\Workspace\AntDemo\bin\AntDemo_unsigned.apk -v -u -z G:\Code\Android\Workspace\AntDemo\bin\AntDemo -f G:\Code\Android\Workspace\AntDemo\bin\classes.dex -rf G:\Code\Android\Workspace\AntDemo\src

注：

(1)、需要调用apkbuilder命令，该命令是一脚本，在android-sdk/tools下，所以需要将该目录设置成环境变……

(2)、需要将之前生成的AntDemo、classes.dex和src下的文件一起构建出apk。

下面是生成的结果图。

212833451.jpg

6、将apk签名

签名apk前，需要先生成一个签名文件*.keystore。该签名文件可以通过JDK下的bin目录下的keytool.exe来生成。可以使用如下脚本：

keytool -genkey -alias ant_test -keyalg RSA -validity 20000 -keystore my.keystore执行该脚本后，按提示输出即可。这里的测试版本使用的密码是123456，其他的直接回车(即使用默认的unknown)。具体的可以参看此文《Android之apk文件签名》

有了keystore之后，就可以进行apk签名了，编写脚本6_signed.bat，内容如下：

jarsigner -keystore G:\Code\Android\Workspace\AntDemo\build\my.keystore -storepass 123456 -keypass 123456 -signedjar G:\Code\Android\Workspace\AntDemo\bin\AntDemo_signed.apk G:\Code\Android\Workspace\AntDemo\bin\AntDemo_unsigned.apk ant_test注：

(1)、jarsigner在JDK的bin目录下，该目录如果没有设置成环境变量的，需要设置成环境变量。

(2)、输入生成my.keystore中所使用的密码和别名(生成签名文件时alias所指定的参数即是)。

下面是执行脚本后的图。

212833452.jpg

上面的过程是一步一步分开的，如果需要一起执行，可以将脚本合并成一个脚本，或者再建一个脚本build.bat来调用各个脚本，下面是build.bat的脚本代码。

call 1_genR

call 2_compile

call 3_dex

call 4_package

call 5_unsigned

call 6_signed

这样，使用bat脚本打包就完毕了。bat脚本的执行就是对一系列任务的执行，而ant也是对一系列任务的执行，所以可以将这些脚本转换成ant的脚本。











将本bat放到cocos2dx目录下你的工程的project.android下(需修改变量)。

ASmaker 用来将Resources文件夹下的lua文件批量加密 算法参考我之前的rc4算法实现。

每次打包apk前 svn 最新的工程代码 和 cocos2dx引擎代码。

@echo off

rem 工具路径

set JAVA_HOME = "C:\Program Files\Java\jdk1.8.0_05"

set ANT_HOME = "D:\ProgramSoftware\apache-ant-1.9.4"

set ANDROID_HOME = "D:\ProgramSoftware\android sdk\sdk"

set NDK_HOME = "D:\ProgramSoftware\android-ndk-r9d-windows-x86_64\android-ndk-r9d"

set SVN_HOME = "C:\Program Files\TortoiseSVN\bin\"

rem 目标路径

set WORK_DIR = "D:\engine\projects\XXXXX\proj.android"

rem set PRO_DIR = "D:\engine\projects\XXXXX"

set RESOURCES_DIR= %WORK_DIR%\..\Resources

set ASSETS_DIR = %WORK_DIR%\assets

rem 先删除旧的assets

if exist D:\engine\projects\XXXXX\proj.android\assets (

echo "deleting assets"

rd /q /s D:\engine\projects\XXXXX\proj.android\assets

)

rem 再删除旧有的Resources

if exist D:\engine\projects\XXXXX\Resources (

echo "deleting Resources"

rd /q /s D:\engine\projects\XXXXX\Resources

)

rem 删除旧的APK

if exist D:\engine\projects\XXXXX\proj.android\bin\XXXXX-release.apk (

echo "deleting old APK"

del /q /f D:\engine\projects\XXXXX\proj.android\bin\XXXXX-release.apk

)

rem call ant clean

rem svn

"C:/Program Files/TortoiseSVN/bin/TortoiseProc.exe" /command:update /path:"D:\engine\projects\XXXXX" /closeonend:1

"C:/Program Files/TortoiseSVN/bin/TortoiseProc.exe" /command:update /path:"D:\engine" /closeonend:1

pushd D:\engine\projects\XXXXX\proj.android

rem luajit Resources

for /r D:\engine\projects\XXXXX\Resources %%i in (*.lua) do (

echo %%i

luajit.exe -b %%i %%i

)

rem ASmaker assets

ASmaker -i D:\engine\projects\XXXXX\Resources -o D:\engine\projects\XXXXX\proj.android\assets

rem ndk

call "D:\ProgramSoftware\android-ndk-r9d-windows-x86_64\android-ndk-r9d\ndk-build" -C "D:\engine\projects\XXXXX\proj.android" "NDK_MODULE_PATH=D:\engine;D:\engine\cocos2dx\platform\third_party\android\prebuilt"

rem ant release

call "D:\ProgramSoftware\android sdk\sdk\tools\android" update project -p "D:\engine\projects\XXXXX\proj.android"

call ant release

popd

pause












1. 把android命令行工具所在的路径添加到path环境变量中，主要包括：

D:\adt-bundle-windows-x86_64-20131115\sdk\tools;

D:\adt-bundle-windows-x86_64-20131115\sdk\platform-tools;

D:\adt-bundle-windows-x86_64-20131115\sdk\build-tools\19.0.1;


2. 我们在eclipse中新建一个android项目，内容非常简单，只有一个MainActivity。



3. 下面我们把这个项目的源码拷贝到其他的路径，然后用命令行进行打包，比如本文是拷贝到D:\work\taobao-wireless\android\安全\命令行打包\hellodemo。

4
（1）生成R文件。在命令行输入：

aapt package -f -m -J ./gen -S res -M AndroidManifest.xml -I D:\adt-bundle-windows-x86_64-20131115\sdk\platforms\android-19\android.jar


（2）生成class文件。在命令行输入：

javac -bootclasspath D:\adt-bundle-windows-x86_64-20131115\sdk\platforms\android-19\android.jar -d bin src\com\example\hello\*.java gen\com\example\hello\R.java  


3）把class文件打成jar包。在命令行输入：

cd bin

jar cvf hello.jar *



（4）生成dex文件。在命令行输入：

cd ..

dx --dex --output=bin\classes.dex bin\hello.jar


（5）打包资源。在命令行输入：

aapt package -f -M AndroidManifest.xml -S res -I D:\adt-bundle-windows-x86_64-20131115\sdk\platforms\android-19\android.jar -F bin\resources.ap_  

6）生成未签名的apk。在命令行输入：

java -cp D:\adt-bundle-windows-x86_64-20131115\sdk\tools\lib\sdklib.jar com.android.sdklib.build.ApkBuilderMain hello.apk  -v -u -z bin\resources.ap_ -f bin\classes.dex -rf src


7）对apk进行签名。在命令行输入：

cd ../../Auto-sign

java -jar signapk.jar testkey.x509.pem testkey.pk8 ../命令行打包/hellodemo/hello.apk ../命令行打包/hellodemo/hellosign.apk





## AAPT


https://www.jianshu.com/p/8d691b6bf8b4

