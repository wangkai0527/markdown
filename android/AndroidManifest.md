[TOC]

# AndroidManifest.xml



### android:process
如果我们需要让一个服务在一个远端进程中运行（而不是标准的它所在的apk的进程中运行），我们可以在声明文件中这个服务的标签中通过android:process属性为其指定一个进程。

在Android的帮助文档中我们可以了解到，一般情况下一个服务没有自己独立的进程，它一般是作为一个线程运行于它所在的应用的进程中。
但是也有例外，Android声明文件中的android:process属性却可以为任意组件包括应用指定进程，换句话说，通过在声明文件中设置android:process属性,我们可以让组件（例如Activity, Service等）和应用(Application)创建并运行于我们指定的进程中。

注意：这里选择”remote”这个名字是随意主观的，你能用其他名字来让这个服务在另外的进程中运行。冒号’:’这个前缀将把这个名字附加到你的包所运行的标准进程名字的后面作为新的进程名称。

例如：一个应用的包名为com.aoyousatuo.example, 则本例中服务将运行的新进程的名称为com.aoyousatuo.example:remote.(注意，如果声明文件中的组件或者应用没有指定这个属性则默认应用和其组件将相应运行在以其包名命名的进程中).


服务所在进程的名字。通常，一个应用的所有组件都运行在系统为这个应用所创建的默认进程中。这个默认进程是用这个应用的包名来命名的。

标签的process属性可以设置成和所有组件都不同的默认值。但是这些组件可以通过设置自己的process值来覆写这个默认值，这样可以让你的应用跨多进程运行。

如果被设置的进程名是以一个冒号开头的，则这个新的进程对于这个应用来说是私有的，当它被需要或者这个服务需要在新进程中运行的时候，这个新进程将会被创建。
如果这个进程的名字是以小写字符开头的，则这个服务将运行在一个以这个名字命名的全局的进程中，当然前提是它有相应的权限。这将允许在不同应用中的各种组件可以共享一个进程，从而减少资源的占用。

例如一个应用运行在进程com.aoyousatuo.example中，android:process属性设置为com.rabbit.man,则新的进程名字为com.rabbit.man。


android:process=":remote"，代表在应用程序里，当需要该service时，会自动创建新的进程。
android:process="remote"，没有“:”分号的，则创建全局进程，不同的应用程序共享该进程。








### android:testOnly
The application could not be installed: INSTALL_FAILED_TEST_ONLY

Android Studio 3.0之后，在打包生成debug apk时，在apk的manifest文件的application标签里自动添加 android:testOnly="true"属性。
当您点击 Run 图标  时，Android Studio 会自动添加此属性。

[application属性](https://developer.android.com/guide/topics/manifest/application-element)


1.在项目中的gradle.properties全局配置中设置：

    android.injected.testOnly=false

这样设置后，反编译就没有属性testOnly=true了

2.adb install 加-t:

    adb install -t app-debug.apk

3.manifest文件application属性添加:

    android:testOnly="false"

4.
使用AS菜单Build->Make Project来编译项目，生成的apk是无该标记位的。
使用菜单Build里面的Build APK(s)，生成的apk也是无该标记位的。









