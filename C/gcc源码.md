[TOC]

# gcc源码


## gcc源码编译

编译最新的gcc依赖老的gcc编译器
前期依赖库：
apt-get install bison flex m4 texinfo autoconf build-essential gcc gcc-multilib g++ make gawk


1. 下载gcc源码
gcc官网：https://gcc.gnu.org/
由于官网在国外，下载速度较慢
https://gcc.gnu.org/wiki/InstallingGCC

清华镜像站：
https://mirrors.tuna.tsinghua.edu.cn/
https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-9.1.0/

南京大学开源镜像站：
http://mirrors.nju.edu.cn/
http://mirrors.nju.edu.cn/gnu/glibc/


2. 下载gcc依赖的三个源码库
gmp
mpc
mpfr

a.
apt-get install libgmp-dev libmpfr-dev libmpc-dev

b.自动下载
./contrib/download_prerequisites

c.手动下载
https://mirrors.tuna.tsinghua.edu.cn/gnu/gmp/
tar -xjvf gmp-6.2.1.tar.bz2
cd gmp
mkdir build && cd build
../configure  --prefix=/home/wk/tools/gmp
make && make install

https://mirrors.tuna.tsinghua.edu.cn/gnu/mpfr/
tar -xzvf mpfr-4.1.1.tar.gz
cd mpfr
mkdir build && cd build
../configure  --prefix=/home/wk/tools/mpfr --with-gmp=/home/wk/tools/gmp
make && make install

https://mirrors.tuna.tsinghua.edu.cn/gnu/mpc/
tar -xzvf mpc-1.3.0.tar.gz
cd mpc
mkdir build && cd build
../configure  --prefix=/home/wk/tools/mpc --with-gmp=/home/wk/tools/gmp --with-mpfr=/home/wk/tools/mpfr
make && make install

https://gcc.gnu.org/pub/gcc/infrastructure/
tar -xjvf isl-0.24.tar.bz2
cd isl
mkdir build && cd build
../configure --prefix=/home/wk/tools/isl --with-gmp-prefix=/home/wk/tools/gmp
make && make install

https://gcc.gnu.org/pub/gcc/infrastructure/
tar -xzvf cloog-0.18.1.tar.gz
cd cloog
mkdir build && cd build
../configure  --prefix=/home/wk/tools/cloog --with-gmp-prefix=/home/wk/tools/gmp
make && make install





3. configuare
mkdir build && cd build

../configure --prefix=/home/wk/tools/gcc-7.5.0 --enable-checking=release --disable-multilib --enable-languages=c,c++ --enable-bootstrap
../configure --prefix=/home/wk/tools/gcc-linaro-7.5 --enable-checking=release --disable-multilib --enable-languages=c,c++ --enable-bootstrap

// --with-gmp=/home/wk/tools/gmp --with-mpfr=/home/wk/tools/mpfr --with-mpc=/home/wk/tools/mpc --with-isl=/home/wk/tools/isl

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/wk/tools/gmp/lib:/home/wk/tools/mpfr/lib:/home/wk/tools/mpc/lib:/home/wk/tools/isl/lib:/home/wk/tools/cloog/lib






–prefix=/opt/gcc-9.1.0 (指定安装路径)
–enable-checking=release （增加编译过程中的一些检查）
–disable-multilib （ 取消多目标库编译，取消32位库编译，在64位机器上默认为enable-multilib）
-enable-languages=c,c++,fortran,go （编译器支持编译的语言）
–enable-bootstrap （进行冗余的编译检查工作）
--program-suffix=-4.7 (gcc being installed as /usr/local/bin/gcc-4.7)

configuare选项
https://gcc.gnu.org/install/configure.html





4. make
make -j8

8 是你允许运行的最大任务数, 建议和 CPU 的线程数量保持一致.
这个过程耗时非常长. 我的电脑上只编译 c++ 都花了二十分钟. 它会编译自己三遍.
第一遍用你的编译器编译它, 生成第一版.
第二遍用第一版编译它, 生成第二版.
第三遍用第二版编译它, 生成第三版.
从理论上分析, 第一版和第二版的二进制代码可能会不同, 但他们的使用效果应该是一样的.
所以就会我们预期是第二版和第三版的 hash 值一致. 如果一致, 就会使用第三版进行后续库的编译.
这样可以确保每一个编译器都是通过自己编译自己的.
(这个 bootstrap 功能可以通过在 configure 添加参数 --disable-bootstrap 来关闭. )

make install
等这个步骤完成, 使用 make install 即可安装到刚刚的 prefix 的地方



-----------------------------------------------------------------------
sanitizer_internal_defs.h:261:72: error: size of array ‘assertion_failed__1150’ is negative

修改下列文件，删除一行 target-libsanitizer \

gcc-7.5.0/configure:

target_libraries=
#        target-libsanitizer \

-----------------------------------------------------------------------


-----------------------------------------------------------------------
configure: WARNING: trying to bootstrap a cross compiler
checking for objdir... .libs
checking for the correct version of gmp.h... yes
checking for the correct version of mpfr.h... yes
checking for the correct version of mpc.h... no
configure: error: Building GCC requires GMP 4.2+, MPFR 2.4.0+ and MPC 0.8.0+.
Try the --with-gmp, --with-mpfr and/or --with-mpc options to specify
their locations.  Source code for these libraries can be found at
their respective hosting sites as well as at
ftp://gcc.gnu.org/pub/gcc/infrastructure/.  See also
http://gcc.gnu.org/install/prerequisites.html for additional info.  If
you obtained GMP, MPFR and/or MPC from a vendor distribution package,
make sure that you have installed both the libraries and the header
files.  They may be located in separate packages.

-----------------------------------------------------------------------

-----------------------------------------------------------------------
conftest.c:10:10: fatal error: isl/schedule.h: No such file or directory
   10 | #include <isl/schedule.h>
      |          ^~~~~~~~~~~~~~~~
compilation terminated.

-----------------------------------------------------------------------

-----------------------------------------------------------------------
make[2]: warning: Clock skew detected. Your build may be incomplete.

检测到时钟偏差，主要是两个设备系统之间的时间上存在差距，导致编译失败。

cd build文件夹的上一级源码目录
将所有文件都重新touch一遍，更新到本地系统的时间
find ./ -type f | xargs touch

-----------------------------------------------------------------------







## 交叉工具链

交叉工具链主要包括Binutils(汇编工具)、GCC(编译器)和Glibc(标准C函数库)，主要用于把源代码编译连接生成可执行程序。

### Glibc
在执行辅助命令make命令时，会调用工具链里的编译器GCC进行编译，使用汇编器Binutils链接到C函数库Glibc，将源代码转换成可执行程序。

Glibc是C函数库是内核与应用程序的中间部分，主要提供C函数库文件。安装Glibc就是在/lib安装一系列的库文件，直接在链接阶段就将库文件链接到可执行文件中，叫静态库。静态库文件是/lib目录下的.a文件；在程序运行时才被载入叫动态库，动态库文件是/lib下的.so文件。Glibc是C函数库，Linux的命令执行过程中都要调用Glibc，其他的函数库也会调用Glibc，Glibc再去调用系统中的内核，内核再进行资源分配。Gcc和Binutils是应用程序，会引用里面的C函数库的库文件，同时使用工具链编译出来的程序软件也会调用Glibc函数库里的库文件。

### Binutils
Binutils 是GNU工具之一，它包括连接器、汇编器和其他用于目标文件和档案的工具，它是二进制代码的处理维护工具。安装Binutils工具包含的程序有：
● addr2line 把程序地址转换为文件名和行号。在命令行中给它一个地址和一个可执行文件名，它就会使用这个可执行文件的调试信息指出在给出的地址上是哪个文件以及行号。
● ar 建立、修改、提取归档文件。归档文件是包含多个文件内容的一个大文件，其结构保证了可以恢复原始文件内容。
● as 主要用来编译GNU C编译器gcc输出的汇编文件，产生的目标文件由连接器ld连接。
● c++filt 连接器使用它来过滤 C++ 和 Java 符号，防止重载函数冲突。
● gprof 显示程序调用段的各种数据。
● ld 是连接器，它把一些目标和归档文件结合在一起，重定位数据，并连接符号引用。通常，建立一个新编译程序的最后一步就是调用ld。
● nm 列出目标文件中的符号。
● objcopy 把一种目标文件中的内容复制到另一种类型的目标文件中。
● objdump 显示一个或者更多目标文件的信息。使用选项来控制其显示的信息，它所显示的信息通常只有编写编译工具的人才感兴趣。
● ranlib 产生归档文件索引，并将其保存到这个归档文件中。在索引中列出了归档文件各成员所定义的可重分配目标文件。
● readelf 显示elf格式可执行文件的信息。
● size 列出目标文件每一段的大小以及总体的大小。默认情况下，对于每个目标文件或者一个归档文件中的每个模块只产生一行输出。
● strings 打印某个文件的可打印字符串，这些字符串最少4个字符长，也可以使用选项-n设置字符串的最小长度。默认情况下，它只打印目标文件初始化和可加载段中的可打印字符；对于其他类型的文件它打印整个文件的可打印字符。这个程序对于了解非文本文件的内容很有帮助。
● strip 丢弃目标文件中的全部或者特定符号。
● libiberty 包含许多GNU程序都会用到的函数，这些程序有getopt、obstack、strerror、strtol和strtoul。
● libbfd 二进制文件描述库。
● libopcode 用来处理opcodes的库，在生成一些应用程序的时候也会用到它。


### 构建gcc交叉工具链
参考文章：
https://wiki.osdev.org/GCC_Cross-Compiler

https://preshing.com/20141119/how-to-build-a-gcc-cross-compiler/
https://blog.csdn.net/fickyou/article/details/51671006

http://blog.chinaunix.net/uid-24943863-id-3997047.html
https://blog.csdn.net/ComputerInBook/article/details/105510452




tar -xzvf binutils-2.39.tar.gz (编译报错 unknown type name 'class')
tar -xzvf binutils-2.30.tar.gz
tar -zxvf glibc-2.9.tar.gz (编译报错 These critical programs are missing or too old: as ld gcc make)
tar -zxvf glibc-2.20.tar.gz
tar -zxvf linux-6.1.tar.gz

mkdir build_binuntils && cd build_binuntils
../binutils-2.30/configure --prefix=/home/wk/tools/gcc-linaro-7.5_aarch64 -target=aarch64-linux --disable-multilib --disable-werror --with-isl=/home/wk/tools/isl


https://mirrors.edge.kernel.org/pub/linux/kernel/v6.x/
cd linux-6.1/
make ARCH=arm64 INSTALL_HDR_PATH=/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux headers_install

mkdir build_gcc-linaro-7.5 && cd build_gcc-linaro-7.5
../configure --prefix=/home/wk/tools/gcc-linaro-7.5 --build=$MACHTYPE --target=arm-linux-gnueabihf --enable-shared --disable-multilib
../configure --prefix=/home/wk/tools/gcc-linaro-7.5 --with-sysroot=/usr/local/arm-linux/arm-linux-gnueabi/libc --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=arm-linux-gnueabi
../gcc-linaro-7.5-2019.12/configure --prefix=/home/wk/tools/gcc-linaro-7.5_aarch64 --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=aarch64-linux-gnu --enable-checking=release --disable-multilib --enable-languages=c,c++ --enable-bootstrap
../gcc-linaro-7.5-2019.12/configure --prefix=/home/wk/tools/gcc-linaro-7.5_aarch64 --target=aarch64-linux --disable-multilib --enable-languages=c,c++



mkdir build_glibc && cd build_glibc
export LD_LIBRARY_PATH=

../glibc-2.20/configure --prefix=/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux --build=$MACHTYPE --target=aarch64-linux --with-headers=/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/include --disable-multilib libc_cv_forced_unwind=yes
../glibc-2.20/configure --prefix=/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux --build=$MACHTYPE --target=aarch64-linux --with-sysroot=/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux --disable-multilib libc_cv_forced_unwind=yes --disable-shared

make install-bootstrap-headers=yes install-headers

make -j4 csu/subdir_lib

echo 'csu/init-first.o csu/libc-start.o csu/sysdep.o csu/version.o csu/check_fds.o csu/libc-tls.o csu/elf-init.o csu/dso_handle.o csu/errno.o csu/init-arch.o csu/errno-loc.o' > /mnt/e/gcc/build_glibc/csu/stamp.oT
mv -f /mnt/e/gcc/build_glibc/csu/stamp.oT /mnt/e/gcc/build_glibc/csu/stamp.o
echo 'csu/init-first.os csu/libc-start.os csu/sysdep.os csu/version.os csu/check_fds.os csu/dso_handle.os csu/unwind-resume.os csu/errno.os csu/init-arch.os csu/errno-loc.os' > /mnt/e/gcc/build_glibc/csu/stamp.osT
echo 'csu/elf-init.oS' > /mnt/e/gcc/build_glibc/csu/stamp.oST
mv -f /mnt/e/gcc/build_glibc/csu/stamp.osT /mnt/e/gcc/build_glibc/csu/stamp.os
gcc -nostdlib -nostartfiles -r -o /mnt/e/gcc/build_glibc/csu/crt1.o /mnt/e/gcc/build_glibc/csu/start.o /mnt/e/gcc/build_glibc/csu/abi-note.o /mnt/e/gcc/build_glibc/csu/init.o
mv -f /mnt/e/gcc/build_glibc/csu/stamp.oST /mnt/e/gcc/build_glibc/csu/stamp.oS


error: ‘__NR_arch_prctl’ undeclared
error: ‘ARCH_SET_FS’ undeclared

cp /mnt/e/gcc/linux-6.1/arch/x86/include/uapi/asm/prctl.h /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/include/asm/

E:\gcc\glibc-2.20\sysdeps\x86_64\nptl\tls.h:
# include <asm/prctl.h>	    /* For ARCH_SET_FS.  */



install csu/crt1.o csu/crti.o csu/crtn.o /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib

aarch64-linux-gcc -nostdlib -nostartfiles -shared -x c /dev/null -o /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib/libc.so

touch /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/include/gnu/stubs.h




/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib/crti.o: error adding symbols: File in wrong format

cp /mnt/e/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/libc/usr/lib/crt* /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib
cp /mnt/e/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/libc/usr/lib/libc.a /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib
cp /mnt/e/gcc/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/aarch64-linux-gnu/libc/usr/lib/libc.so /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib
cp ./aarch64-linux-gnu/libc/lib/libc.so.6 /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib
cp ./aarch64-linux-gnu/libc/lib/libc-2.25.so /home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/lib

/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/bin/ld: cannot find /lib/libc.so.6
/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/bin/ld: cannot find /usr/lib/libc_nonshared.a
/home/wk/tools/gcc-linaro-7.5_aarch64/aarch64-linux/bin/ld: cannot find /lib/ld-linux-aarch64.so.1








深入分析GCC
https://book.douban.com/subject/26984172/
https://developer.aliyun.com/article/90401?spm=a2c6h.12873639.article-detail.38.49a32510CcEtHU&scm=20140722.ID_community@@article@@90401._.ID_community@@article@@90401-OR_rec-V_1

https://compcert.org/

https://zhuanlan.zhihu.com/p/190961262



