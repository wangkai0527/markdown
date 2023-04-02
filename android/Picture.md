[TOC]

# Picture



## 9-Patch 
[创建可调整大小的位图（9-Patch 文件）](https://developer.android.google.cn/studio/write/draw9patch?hl=zh-cn)



### FAQ

-----------------------------------------------------------------------
Android Studio提示：
        
        Error:found an invalid color

查看build日志：

        .9.png error: file failed to compile

1. 在build.gradle里添加:

        aaptOptions.cruncherEnabled = false
        aaptOptions.useNewCruncher = false
依旧报错

2. 在gradle.properties里面添加

        android.enableAapt2= false
依旧报错

3. 然后又直接在AS中把后缀名.9.png改成.png，没有报错了。但是图片使用却不能按照.9图片的拉伸效果处理了，图片出现变形，同时在图片周围还出现了一些黑色的边线。

重新认识：
后面在网上去查找了关于.9图片资料，原来.9图片会在左边和上班标注一条线段，表示在图片需要扩展或者收缩的时候，这些片段是可以在纵向或者横向拉伸缩小的；同时会在右边和下班标注一条线段，表示显示的内容区域是在什么区域段。

**最后解决：**
我在AS中打开了美工给我的那张.9图片查看，发现它的右边和下边的黑色线条是占满了整个图片的边框，然后我去手动修改了一下右边和下边黑色线条的长度，保存之后，编译就通过了，可以正常使用了。

防坑注意点：
如果是把.9图片作为背景图，比如作为弹窗的背景图片，有时候会发现自己的弹窗怎么设置长宽属性，还有设置布局XML都没有作用，还以为中邪了．后面经过调查发现，原来是自己在调整.9图片显示区域的时候，把显示区域变窄了，导致显示的内容区域和图片的边距margin变大了导致的．后面去调节了.9图片的显示区域，尽量把显示区域接近整个图片的大小就好了。

-----------------------------------------------------------------------





## WebP

[创建 WebP 图片](https://developer.android.google.cn/studio/write/convert-webp?hl=zh-cn)













