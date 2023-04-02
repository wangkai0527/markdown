[TOC]

# Python

[Python官网](https://docs.python.org/3/)


## Python版本管理
网上有很多教程，讲如何在一台机器上同时安装2.x和3.x两个版本，使用时分别用python、python3区分。但是这种方法有几个明显的缺点：
安装麻烦：源码手动安装，可能需要手动指定安装路径，创建软连接等；
2.x 和 3.x 分别只能安装一个版本：例如不能同时安装 2.6 和 2.7；
需要人工确定项目使用的 Python 版本，指定错误会导致运行报错。

### pyenv
pyenv 用来管理一台机器上的多个 Python 版本。主要解决开发中有的项目需要用Python 2.x，有的项目需要用Python 3.x的场景。

pyenv 支持在一台机器上同时安装 cpython（平时说的 Python）、jython、anaconda、micropython、miniconda、pypy、stackless 等等的任意当前可用版本。
pyenv 使用了垫片的原理，可以做到进入项目目录自动选择 Python 版本，使用极为方便。
[pyenv 神器原理分析](https://cloud.tencent.com/developer/article/1593478?from=10680)

使用方法：
安装 pyenv（推荐方法，此脚本会自动安装若干插件，包括下文即将提到的 pyenv virtualenv）
curl https://pyenv.run | bash

查看所有支持安装的 Python 版本
pyenv install -l

安装 Python 2.7.17 和 3.8.2
pyenv install 2.7.17
pyenv install 3.8.2

指定全局使用 Python 2.7.17
pyenv global 2.7.17

指定 myproject 使用 Python 3.8.2
cd myproject
pyenv local 3.8.2

在当前 shell 中临时使用 Python 3.8.2
pyenv shell 3.8.2

上面在 myproject 项目目录设置了 pyenv local 3.8.2 之后，后续进入该目录及其子目录时，所执行的 python 命令就是 3.8.2 版本，不需手动执行 activate；
离开该目录之后，执行的的 python 命令就是系统安装的或者 pyenv global 指定的版本，不需要手动执行 deactivate。
上述几种用法中，优先级为：pyenv shell > pyenv local > pyenv global > system。即优先使用 pyenv shell 设置的版本，三种级别都没设置时才使用系统安装的版本。


### pyenv virtualenv
前面提到 pyenv 要解决的是多个 Python 的版本管理问题，virtualenv 要解决的是同一个库的版本管理问题。
但如果两个问题都需要解决呢？分别使用不同工具就很麻烦了，而且容易有冲突。
为此，pyenv 引入了了 virtualenv 插件，可以在 pyenv 中解决同一个库的版本管理问题。

通过 pyenv virtualenv 命令，可以与 virtualenv 类似的创建、使用虚拟环境。
但由于 pyenv 的垫片功能，使用虚拟环境跟使用 Python 版本的体验一样，不需要手动执行 activate 和 deactivate，只要进入目录即生效，离开目录即失效。

pyenv virtualenv 的用法和 pyenv 类似（使用上述安装 pyenv 方法会自动安装 virtualenv 插件）：
分别安装基于 Python 2.7.17 和 Python 3.8.2 的虚拟环境
pyenv virtualenv 2.7.17 venv2
pyenv virtualenv 3.8.2 venv3

指定全局使用虚拟环境 venv2
pyenv global venv2

指定 myproject 使用虚拟环境 venv3
cd myproject
pyenv local venv3

在当前 shell 中临时使用虚拟环境 venv3
pyenv shell venv3


### PyCharm配置Python虚拟环境详解
https://blog.csdn.net/didi_ya/article/details/120664678



## 软件包管理
Python拥有众多的软件包/库。
几个构成Python科学计算生态系统的核心成员：

NumPy
NumPy 是 Numerical Python 的简称。 NumPy 是 Python 科学计算最基础的库，涉及数据分析的软件包基本上都基于它来构建。

Pandas
Pandas 的名字来源于 Python 数据分析（ Python data analysis ）和面板数据（ Panel data ）的结合。
该库提供了多个数据存储对象，其中的 DataFrame 对象可以表征数据分析常见的二维表格。除此之外，它还提供了非常多便捷处理结构化数据的函数。

Matplotlib
Matplotlib 起源于矩阵实验室 MATLAB 中的绘图函数，是Python 中非常流行的绘图库，可以轻松实现二维数据甚至多维数据可视化。

SciPy
SciPy 库提供了一组专门用于科学计算的各种标准问题包，如数值积分、微分、信号处理、统计分析，它与 NumPy 的结合可以处理诸多科学计算问题。

Jupyter
Jupyter 是一个交互和探索式计算的高效环境。其中两个组件较为常用，
一是 IPython ，用于编写、测试和调试 Python 代码；
二是 Jupyter Notebook ，它是一个多语言交互式的 Web 笔记本，现在支持运行 Python 、 R 等多种语言。
Jupyter Notebook 中的代码与 Markdown 结合可以创建良好、可重复的动态文档。
这也是读者进行 Python 数据分析的学习环境。

### pypi
pypi 是 Python Package Index 的首字母简写，表示的是 Python 的官方 Package 索引。
可以下载第三方库或上传自己开发的库到PyPI。PyPI推荐使用pip包管理器来下载第三方库。

https://mirrors.tuna.tsinghua.edu.cn/help/pypi/

### pip/pip3
python软件包管理系统，可以利用它安装python包，是easy_install的替代品，英文python install packages.
python 2.7.9 和3.4以后的版本已经内置pip程序，所以不需要安装。默认都安装到当前python版本的python3.7/site-packages文件夹下。

因为我使用的是pyenv来管理python版本，所有通过pip安装的包均放在：
.pyenv/versions/3.7.2/lib/python3.7/site-packages/包名

备注：安装的程序是分开的pip是安装到python2版本对应的目录里，pip3是安装到python3版本对应的目录中。

安装
1 若已经安装easy_install,直接用命令安装
sudo easy_install pip
sudo easy_install install pip

2 若无，前往https://pypi.python.org/pypi/pip 下载pip-18.0.tar.gz 文件，解压下载的文件，
进入解压后的文件夹中，调出命令行窗口或者终端，输入
python setup.py install

3 检验pip是否安装成功
C:\Users\liangxiang>pip --version
pip 18.0 from e:\python\lib\site-packages\pip-18.0-py3.5.egg\pip (python 3.5)


常用命令

pip help

1、查找软件
pip search [包名]

2、安装软件
pip install [包名]

过期库批量更新/安装
pip freeze > xxxx.txt #导出当前系统安装的库，保存为TXT文件
（ pip freeze > E:\\XXX.txt #保存到指定文件夹 ）
pip install -r xxxx.txt

安装指定版本
pip install "软件包名称==版号"

不指定版本号，要求某一个版本之前或之后的版本
pip install "软件包名称>=版号"

3、更新软件
pip install -U [包名]
pip install --upgrade 库名 

4、卸载软件
pip uninstall [包名]

5、列出已安装软件
pip list
pip freeze 
列出所有过期的库
pip list --outdated 

6、查看一个软件包时安装了哪些文件
pip show -f [包名]


### easy_install
easy_install 是一个基于setuptools的工具，帮助我们自动下载、编译、安装和管理python packages.
easy_install需要2.6以上的python版本
[easy_install官网](http://peak.telecommunity.com/DevCenter/EasyInstall)

C:\Users\liangxiang>easy_install --version
setuptools 28.8.0 from e:\python\lib\site-packages (Python 3.5)

安装easy_install的几种方式
1、 源码安装setuptools
安装setuptools之后，easy_install就已经安装好了。
Setuptools下载地址：https://pypi.python.org/pypi/setuptools
Windows下可以直接运行.exe文件，linux下解压，python setup.py install

2、 通过引导程序ez_setup.py安装
引导程序会联网下载最新版本的setuptools,也可以用来更新本地的setuptools.

前往https://pypi.python.org/pypi/ez_setup下载一个名叫ez_setup.py的程序
或者wget http://peak.telecommunity.com/dist/ez_setup.py

安装：
python ez_setup.py
更新:
python ez_setup.py –U setuptools

easy_install的使用
1、 安装python packages
通过包名，从PyPI寻找最新版本，自动下载、编译、安装
easy_install <packagename>

通过包名从指定下载页寻找链接来安装或升级python packages
例：easy_install -f http://pythonpaste.org/package_index.html SQLObject

从具体的URL下载安装
easy_install http://example.com/path/to/MyPackage-1.2.3.tgz

安装一个本地已经存在的.egg文件
easy_install /my_downloads/OtherPackage-3.2.1-py2.3.egg

指定python package的安装目录
添加选项--install-dir=DIR, -d DIR

安装packages到用户目录，非全局安装
添加选项 --user

2、 升级python packages
easy_install --U PyProtocols
easy_install --upgrade PyProtocols
升级到PyPI中存在的最新版

指定升级的版本
$ easy_install "PackageName==2.0"
$ easy_install "PackageName>2.0"

3、 删除python packages
$ easy_install -m PackageName
这样操作之后会将包信息从easy-install.pth文件里删除，不能在python 中使用PackageName，但是删除的不彻底，需要手动删除.egg包和一些其他文件。


### virtualenv
virtualenv 所要解决的是同一个库不同版本共存的兼容问题。例如项目 A 需要用到 requests 的 1.0 版本，项目 B 需要用到 requests 的 2.0 版本。
如果不使用工具的话，一台机器只能安装其中一个版本，无法满足两个项目的需求。

virtualenv 的解决方案是为每个项目创建一个独立的虚拟环境，在每个虚拟环境中安装的库，对其他虚拟环境完全无影响。
所以就可以在一台机器的不同虚拟环境中分别安装同一个库的不同版本。

使用方法：
安装 virtualenv
pip install virtualenv

创建虚拟环境 myenv
virtualenv /path/to/myenv

切换到虚拟环境 myenv，此时命令提示符前会有 (myenv) 提示符
cd /path/to/myenv/ && source bin/activate

安装库，安装到 /path/to/myenv/lib 中，不会与其他虚拟环境冲突
pip install requests

执行 python 相关命令
python demo.py

退出虚拟环境
deactivate


## pyvenv
pyvenv 与 virtualenv 功能和用法类似。不同点在于：

pyvenv 只支持 Python 3.3 及更高版本，而 virtualenv 同时支持 Python 2.x 和 Python 3.x；
pyvenv 是 Python 3.x 自带的工具，不需要安装，而 virtualenv 是第三方工具，需要安装。
pyvenv 实际上是 Python 3.x 的一个模块 venv，等价于 python -m venv。

pyvenv 的用法和 virtualenv 类似：
创建虚拟环境 myenv
pyvenv /path/to/myenv
或者
python -m venv /path/to/myenv

切换到虚拟环境 myenv，此时命令提示符前会有 (myenv) 提示符
cd /path/to/myenv/ && source bin/activate
Windows下执行`\path\to\myenv\Scripts\activate.bat`

安装库，安装到 /path/to/myenv/lib 中，不会与其他虚拟环境冲突
pip install requests

执行 python 相关命令
python demo.py

退出虚拟环境
deactivate
Windows下执行`\path\to\myenv\Scripts\deactivate.bat`






https://github.com/azhan1998/sam_buy


https://github.com/justjavac/free-programming




https://zhuanlan.zhihu.com/p/435398428



