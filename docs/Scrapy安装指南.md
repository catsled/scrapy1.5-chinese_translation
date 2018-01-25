# 安装指南

## 安装Scrapy
Scrapy运行在Cpython（默认python实现)中的python2.7和python3.4或者上下版本和PyPy（开始PyPy5.9)

如果你在使用[Anaconda](https://docs.anaconda.com/anaconda/)或者 [Miniconda](https://conda.io/docs/user-guide/install/index.html)，你可以从这conda-[forge](https://conda-forge.org/)通道安装这包，这里有最新的对应Linux，windows，或者Mac平台的安装包。
用conda安装Scrapy，运行下面命令:
> conda install -c conda-forge scrapy

---
或者如果你依然熟悉Python包安装，你可以安装Scrapy和它的依赖从 PyPI上用以下命令:
> pip install Scrapy

---
注意在有些时候这或许需要解决一些因为取决于你的运行操作系统时一些Scrapy依赖的编译问题,因此一定要检查确定[这平台特定安装纪要](https://docs.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes)。

我们强烈建议你安装Scrapy在[一个专用的虚拟环境](https://docs.scrapy.org/en/latest/intro/install.html#intro-using-virtualenv)中，以免与你的系统包发生冲突。

对于更多关于平台特定的介绍和细节，情继续阅读下去。

###  这些都是很好去知道的事情
Scrapy时一个纯粹用python和依赖一些关键python包（尤其以下):

[lxml](http://lxml.de/),一个高效的XML和HTML解析器。

[parsel](https://pypi.python.org/pypi/parsel),一个在lxml上封装写的HTML/XML数据提取库。

[w3lib](https://pypi.python.org/pypi/w3lib),一个多功能的来解决URLs和web页面编码的帮助者。

[twisted](https://twistedmatrix.com/),一个异步的网络框架。

[cryptography](https://cryptography.io/)和[pyOpenSSL](https://pypi.python.org/pypi/pyOpenSSL),来解决各种各样的网络级别的安全需要。

这Scrapy测试最低版本需要是:
    
    Twisted 14.0
    lxml 3.4
    pyOpenSSL 0.14
Scrapy可能在这些包的老版本上也可以运行，但是它不能够保证它会继续工作，因为它没用在这些版本上测试过。

一些包它们自己依赖在没有python包的情况,这样需要额外的取决于你所使用平台的不同安装步骤，请检查[特定平台下安装指南](https://docs.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes)。

以防止依靠这些依赖出现的任何问题，请参考他们各自的安装介绍。

 [lxml installation](http://lxml.de/installation.html)

 [cryptography installation](https://cryptography.io/en/latest/installation/)
 
##  使用虚拟环境(建议)

我们建议在所有的平台上安装Scrapy在一个虚拟环境中。

 尽管可以在全局或用户空间安装Python包，但我们不建议直接在系统环境下安装Scrapy。 相反，我们建议在“虚拟环境”(virtualenv)中安装Scrapy。 virtualenv环境不会与系统环境中安装的Python包冲突(这可能会破坏系统工具和脚本)，并且仍然可以正常地使用pip(不包括 sudo)安装包。

要使用虚拟环境，请参阅virtualenv安装说明。要在全局安装它(在全局安装实际上是有很有用的)，它应该是一个运行的问题。

> [sudo] pip install virtualenv

检查这[用户指南](https://virtualenv.pypa.io/en/stable/userguide/)来学习如何创建你的虚拟环境。
 
 如果您使用Linux或OSX，那么virtualenvwrapper是创建虚拟环境比较方便的工具。
 一旦你创建了一个虚拟环境，你依然可以在里面用pip安装Scrapy,就像任何其他的Python包一样,（参见：特定平台安装指南包含可能需要预先安装的非python依赖项的说明）
 
 Python 虚拟环境可以默认使用Py2或者Py3。
 如果你想在py3下安装Scrapy,你可以在一个py3虚拟环境下安装Scrapy。
 并且如果你想要在py2下安装Scrapy,你可以在一个py2虚拟环境下安装Scrapy。
 
##  特定平台安装指南

### windows

虽然可以在Windows上使用pip安装Scrapy，但是我们推荐您安装“Anaconda”或“Miniconda”，并使用来自“conda-forge”渠道的该包，这样就可以避免大多数安装问题。

一旦你安装了“Anaconda”或“Miniconda”，就可以用命令进行安装了:

> conda install -c conda-forge scrapy

###    Ubuntu 14.04 or above

目前，Scrapy已经用了足够多版本的lxml、twisted和pyOpenSSL进行了测试，Scrapy已经与最新的Ubuntu发行版兼容。但它应该也是支持旧版本的Ubuntu的，比如Ubuntu 14.04，尽管存在TLS连接的潜在问题。

千万不要使用 Ubuntu提供的python-scrapy的Python包,它们都太老太慢了，已经跟不上最新版的Scrapy了！

要在Ubuntu系统(或基于ubuntubased)的系统上安装“Scrapy”，你需要安装这些依赖项:
> sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

python-dev, zlib1g-dev, libxml2-dev 和 libxslt1-dev 都被 lxml 依赖
libssl-dev 和 libffi-dev 被 cryptography 依赖

如果您想在python3上安装Scrapy，您还需要Python 3发展header:

> sudo apt-get install python3 python3-dev

在安装完虚拟环境之后你就可以使用 pip 来安装Scrapy:

> pip install scrapy

同样的非python依赖项也可以用于Debian jessie (8.0)及新版环境中安装Scrapy

### Mac OS X

构建Scrapy的依赖关系需要有一个C编译器和development header。 在OS X上，这通常是由苹果的Xcode开发工具提供的。要安装Xcode命令行工具，打开一个控制台并运行:
> xcode-select --install

有一个[已知的问题](https://github.com/pypa/pip/issues/2468)来防止“pip”更新系统包。 这是成功安装Scrapy及其依赖必须要解决的问题。这里有一些建议解决方案:

(推荐) 千万不要 使用系统的Python,安装一个不和系统部分冲突的全新或更新的版本。 下面就是如何使用homebrew包管理器来安装：
    
    按照 http://brew.sh/ 的指示安装homebrew
    更新 PATH 变量确保homebrew包应该在系统包之前引用(如果你使用zsh作为默认shell，请将 .bashrc 对应改为 .zshrc )
    
> echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

重新加载 .bashrc ，确保已经修改成功::
> source ~/.bashrc

安装python::
> brew install python

最新版本的Python已经将pip进行了捆绑，所以你不必再分别安装它们了。如果不是这样的，请更新你的python:
> brew update; brew upgrade python

(可选) 在一个隔离的环境中安装Python.这种方法是解决上述OSX问题的一种变通方法，但它是管理依赖关系的总体良好实践，并且可以作为第一个方法的补充。

虚拟环境是一个可以用来在python中创建虚拟环境的工具。我们建议你可以阅读类似的教程 http://docs.python-guide.org/en/latest/dev/virtualenvs/ 来进行学习.

在完成这些工作之后，你应该能够成功的安装Scrapy了:
> pip install Scrapy

### PyPy

我们建议使用最新的PyPy版本，这最新测试的PyPy3版本是5.9,只在linux上安装测试过。

许多的scrapy依赖现在有了对于cpython的二进制车轮,而不是PyPy,这意味着这些依赖会在安装时构建,在OSX上，你更可能会面对构建Cryptography依赖的一个问题,解决这个问题的方法描述在[这里](https://github.com/pyca/cryptography/issues/2692#issuecomment-272773481).这是去brew install openssl，然后记录这个标记该命令推荐只在安装scrapy时需要，在linux上除了安装构建依赖没有其他特别的问题。在windows上通PyPy来安装scrapy没有测试过。

你可以检车scrapy是安装成功了通过运行命令:scrapy bench,如果命令输完，报了以下错误比如TypeError: ... got 2 unexpected keyword arguments,这意味着这个安装工具没有能够找到一个PyPy特定依赖,为了解决这个问题,运行该命令:
> pip install 'PyPyDispatcher>=2.1.0'

    