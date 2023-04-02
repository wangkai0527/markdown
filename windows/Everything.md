[Toc]

# Everything


[官方网站](https://www.voidtools.com/zh-cn)


[搜索语法](https://zhuanlan.zhihu.com/p/409431144)


## Everything文件检索软件为什么比系统检索快?

Everything并没有全部逐一扫描我们硬盘上的文件，而是通过读取NTFS文件系统中的USN日志来完成的，只能检索文件名。
USN是系统日志的一部分，是Update Service Number Journal or Change Journal的英文缩写，直译为“更新序列号”，是对NTFS卷里所修改过的信息进行相关记录的功能。
当年微软发布Windows 2000时，建立NTFS 5.0的同时，加入了一些新功能和改进了旧版本的文件系统，它可以在分区中设置监视更改的文件和目录的数量，记录下监视对象修改时间和修改内容。
当这个功能启用时，对于每一个NTFS卷，当发生有关添加、删除和修改文件的信息时，NTFS都使用USN日志记录下来。
USN日志的工作方式，相对来说很简单，所以非常的高效。它开始的时候是一个空文件，包括NTFS每个卷的信息。每当NTFS卷有改变的时候，所改变的信息会马上被添加到这个文件里。这其中，每条修改的记录都使用特定符号来标识为日志形式，也就是USN日志。每条日志，记录了包括文件名、文件信息做出的改变，日志里包括发生了什么变化（添加、删除或其他操作）。
USN日志相当于一本书的索引，当然书里面内容发生添加、修改或删除的时候，USN日志会记录下来何时做了修改，并使用特定序列号来标识，但它并不会记录里面具体修改了什么东西，所以索引文件很小。而当你想查找某一篇文章时，你就不用一页一页去翻书，可以直接通过查找USN日志（也就是建立的索引）就知道这篇文章是否存在。
综上： NTFS文件系统中的USN日志，是一项系统管理功能，能够记录卷上文件和文件夹的所有更改。“Everything”的搜索功能也是基于这个日志，只是在索引当中根据文件名过滤出符合条件的文件或文件夹故而十分迅速。随着版本的优化，其速度也到了目前秒开的水平，确实是Windows的文件名检索利器。

Windows搜索用的是普通的文件系统遍历查找。用的应该是标准WIN32 API，例如FindFirstFile/FindNextFile之类。
当然，Windows7之后的版本现在已经内置带索引的搜索功能，这项功能非常复杂，不仅可以搜索文件名，还可以搜索文件内容，而且适用任意文件系统。
但缺点就是需要一个后台服务爬虫不停的对文件系统进行索引，比较耗资源。

因为mac os的文件系统也没有这样的文件数据库，所以mac平台下也没有everything和listary这样高效的检索工具，只能挨个找










