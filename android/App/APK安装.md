[TOC]

# APK安装


1. 复制 APK 到/data/app 目录下，解压并扫描安装包。
2. 资源管理器解析 APK 里的资源文件。
3. 解析 AndroidManifest 文件，并在/data/data/目录下创建对应的应用数据目录。
4. 然后对 dex 文件进行优化，并保存在 dalvik-cache 目录下。
5. 将 AndroidManifest 文件解析出的四大组件信息注册到 PackageManagerService 中。
6. 安装完成后，发送广播。






