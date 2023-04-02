[TOC]

# Binder






![](Binder实现原理.image)






发送数据:通过copyfrom_user将数据拷贝到 binder驱动中，binder驱动存在于内核空间
获取数据:会提前打开 binder 驱动，并通进过alloc_page分配一页物理页面(4K)，而物理页指向的就是物理内存，又与 service端和内核空间存在 mmap 映射关系，所以service端可以直接访问到物理内存中的数据，也就不需要再次进行拷贝了



这里用户空间 mmap(1M-8K)的空间，为什么要减去8K，而不是直接用1M?
Android的 gitcommit 记录:
Modify the binder to request 1M-2 pages instead of1M.The backing store in the kernel requires a guard pageso 1M
allocations fragment memory very badly.Subtracting a couple of pages so that they fit in a power of two allows the kernel to make more efficient use of itsvirtual address space.
大致的意思是:kernel的“backingstore”需要一个保护页，这使得1M用来分配碎片内存时变得很差，所以这里减去两页来提高效率，因为减去一页就变成了奇数。

系统定义:BINDER VMSIZE((1*1024*1024)-
sysconf(SC PAGE SIZE)*2)=(1M-sysconf(SC PAGE SIZE)*2)
这里的8K，其实就是两个PAGE的SIZE，物理内存的划分是按 PAGE(页)来划分的，一般情况下，一个Page的大小为4K。
内核会增加一个 guard page，再加上内核本身的guardpage，正好是两个page的大小，减去后，就是用户空间可用的大小。














https://blog.csdn.net/qq_27672101/article/details/108186072

https://blog4jimmy.com/2018/01/350.html

https://www.jianshu.com/p/15c6167cc666

https://blog.csdn.net/u010128475/article/details/125699297




