[TOC]

# Tengine

[Tengine GitHub](https://github.com/OAID/Tengine)

[文档](https://tengine.readthedocs.io/zh_CN/latest/index.html)










ssh test@192.168.1.125
test


/home/test/roy/roy_lr/tengine/Tengine

cd build-an
rm -rf *
cmake -DCMAKE_TOOLCHAIN_FILE=/home/test/roy/roy_lr/tengine/ndk/android-ndk-r21e/build/cmake/android.toolchain.cmake -DANDROID_ABI="arm64-v8a" -DANDROID_PLATFORM=android-21 ..
make tengine-lite-static -j4
ll
cd source/













