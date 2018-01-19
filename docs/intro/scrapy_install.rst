.. _docs-intro-scrapy-install:

=========
安装指南
=========


安装Scrapy
=============

Scrapy运行在Cpython解释器（默认python实现)和PyPy解释器(从PyPy5.9版本开始)下的Python 2.7和Python 3.3或更高版本上。
如果你在使用 Anaconda_ 或者 Miniconda_，你可以通过 conda-forge_ 渠道安装软件包，该渠道包含适用于Linux，Windows和OS X的最新软件包。
通过运行以下命令，可以使用conda安装Scrapy: ::

    > conda install -c conda-forge scrapy


同样的，如果你熟悉Python包安装方式，你可以通过以下命令从PyPI安装Scrapy和它的依赖包。 ::

    > pip install Scrapy


注意，对于某些Scrapy依赖项的编译问题的解决方法，有时候可能需要根据你的操作系统来决定，因此请务必查看 平台特定的安装说明 `install guide`_ 。

我们强烈建议你将Scrapy安装在专用的虚拟环境 virtualenv_link_ 中，以免与你的系统包发生冲突。

有关更多关于平台特定的介绍和细节，请继续阅读。

.. _Anaconda: https://docs.anaconda.com/anaconda/
.. _Miniconda: https://conda.io/docs/user-guide/install/index.html
.. _virtualenv_link: https://docs.scrapy.org/en/latest/intro/install.html#intro-using-virtualenv
.. _conda-forge: https://conda-forge.org/
.. _`install guide`: https://docs.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes

一些更有益于学习scrapy的知识
----------------------------

Scrapy是一个用纯python编写的，并依赖于几个关键的Python包（其中包含如下几个):

* lxml_,一个高效的XML和HTML解析器。

* parsel_,一个编写于lxml之上的HTML/XML数据提取库。

* w3lib_,一个处理URL和网页编码的多用途辅助库。

* twisted_,一个异步的网络框架。

* cryptography_ 和 pyOpenSSL_,用来处理各种网络级别的安全需要。

.. _lxml: http://lxml.de/
.. _parsel: https://pypi.python.org/pypi/parsel
.. _w3lib: https://pypi.python.org/pypi/w3lib
.. _twisted: https://twistedmatrix.com/
.. _cryptography: https://cryptography.io/
.. _pyOpenSSL: https://pypi.python.org/pypi/pyOpenSSL


Scrapy测试的最低版本需求:
    
* Twisted 14.0
* lxml 3.4
* pyOpenSSL 0.14 
   
Scrapy可能在这些包的老版本上也可以运行， 但因为Scrapy没有在这些版本上测试过，所以并不能保证在这些版本上Scrapy能够正常工作。

其中一些软件包本身依赖于非Python包，可能需要额外的安装步骤，具体取决于您的平台。请查看下面的特定平台下安装指南 install_plateform_guide_ 。

以防止出现任何有关依赖的问题，请参考他们各自的安装介绍。

 `lxml installation`_

 `cryptography installation`_
 
.. _`lxml installation`: http://lxml.de/installation.html
.. _`cryptography installation`: https://cryptography.io/en/latest/installation/
.. _install_plateform_guide: https://docs.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes


使用虚拟环境(建议)
===================

我们建议在所有平台上都将Scrapy安装在一个虚拟环境中。

尽管可以在全局（又名系统范围）或用户空间安装Python包，但我们不建议直接在系统环境下安装Scrapy。
相反，我们建议在“虚拟环境”(virtualenv)中安装Scrapy。 虚拟环境不会与系统环境中安装的Python包冲突(如果发生冲突可能会破坏系统工具和脚本)，并且仍然可以正常的使用pip(不包括 sudo)安装软件包。

要使用虚拟环境，请参阅virtualenv安装说明。要在全局安装它(全局安装在这里实际上是有帮助)，可以通过运行下面命令. ::

    > [sudo] pip install virtualenv

查看用户指南 `user guide`_ 来学习如何创建你的虚拟环境。
 
.. note::
    如果您使用Linux或OSX，那么`virtualenvwrapper`是创建虚拟环境比较方便的工具。

当你创建了一个虚拟环境，你可以在里面用pip安装Scrapy,就像安装其他的Python包一样,（对于需要事先安装的非python依赖，请参阅下面的特定平台安装指南 `install tutorial`_
  
 
Python 虚拟环境默认使用Python2或者Python3创建。

* 如果你想通过py3安装Scrapy,你可以在py3虚拟环境下安装Scrapy。
* 如果你想通过py3安装Scrapy,你可以在py2虚拟环境下安装Scrapy。
 
.. _`user guide`: https://virtualenv.pypa.io/en/stable/userguide/
.. _`install tutorial`: https://doc.scrapy.org/en/latest/intro/install.html#intro-install-platform-notes


特定平台安装指南
====================


windows
-------------------

虽然可以在Windows上使用pip安装Scrapy，但是我们推荐您安装`Anaconda`或`Miniconda`，并使用来自`conda-forge`渠道的该包，这样就可以避免大多数安装问题。

你安装了`Anaconda`或`Miniconda`后，可以通过以下命令安装Scrapy: ::

    > conda install -c conda-forge scrapy


Ubuntu 14.04 or above
---------------------------

目前，Scrapy正在使用最新版本的lxml，twisted和pyOpenSSL进行测试，并已经与最新的Ubuntu发行版兼容。但它应该也支持旧版本的Ubuntu的，比如Ubuntu 14.04，尽管TLS连接可能存在问题。

不要使用 Ubuntu提供的python-scrapy的Python包,它们一般是比较老而且缓慢，无法跟上最新版的Scrapy。

在Ubuntu(或基于ubuntubased)系统上安装“Scrapy”，你还需要安装这些依赖项: ::

    > sudo apt-get install python-dev python-pip libxml2-dev libxslt1-dev zlib1g-dev libffi-dev libssl-dev

lxml需要的 python-dev, zlib1g-dev, libxml2-dev 和 libxslt1-dev依赖项
cryptography需要的libssl-dev 和 libffi-dev 依赖项

如果你想要安装基于python3版本的scrapy，你还需要一些python3版本的开发头文件: ::

    > sudo apt-get install python3 python3-dev

在安装完虚拟环境之后你就可以使用 pip 来安装Scrapy: ::

    > pip install scrapy

.. note:: 注意：在Debian Jessie（8.0）及以上版本中，可以使用相同的非Python依赖项来安装Scrapy。


Mac OS X
------------------

构建Scrapy的依赖关系需要有一个C编译器和头开发文件。 在OS X上，通常是由苹果的Xcode开发工具提供的。如果要安装Xcode命令行工具，请打开一个控终端并以下运行: ::

    > xcode-select --install

如何防止 `pip` 更新系统包，是一个现有的[已知问题] know_question_ 。 这是成功安装Scrapy及其依赖必须要解决的问题。下面是一些建议性的解决方案:

* (推荐) 不使用系统自带的Python,安装一个不和系统部分冲突的新的，更新的版本。 以下是如何使用homebrew包管理器来安装的方法：
    
   * 按照 http://brew.sh/ 的说明安装homebrew
   * 更新 PATH 变量，确保homebrew包应该在系统包之前引用(如果你使用zsh作为默认shell，请将 .bashrc 对应改为 .zshrc )::
    
        > echo "export PATH=/usr/local/bin:/usr/local/sbin:$PATH" >> ~/.bashrc

   * 重新加载 .bashrc ，确保已经修改成功::

        > source ~/.bashrc

* 安装python::

    > brew install python

最新版本的Python已经将pip进行了捆绑，所以你不必再分别安装它们了。如果不是这种情况，请更新你的python:
> brew update; brew upgrade python

(可选) 在一个独立的环境中安装Python.这是解决上述OSX问题的一种变通方法，但它是管理依赖关系的总体良好实践，并且可以作为第一个方法的补充。

virtualenv是一个在python中创建虚拟环境的工具。我们建议你可以阅读类似的教程 `virtualenv_guide`_ 来进行学习。

在完成这些工作之后，你应该能够成功的安装Scrapy了: ::
    > pip install Scrapy

.. _`virtualenv_guide`: http://docs.python-guide.org/en/latest/dev/virtualenvs/
.. _know_question: https://github.com/pypa/pip/issues/2468


PyPy
----------

我们建议使用最新的PyPy版本，最新测试的版本是5.9。对于PyPy3，这个版本只在linux上安装测试过。

目前，更多的scrapy二进制依赖文件针对的是CPython，而不是PyPy ,这意味着这些依赖会在安装时构建,在OSX上，你可能会遇到构建Cryptography依赖性的问题,这里 here_ 描述了解决这个问题的方法.接下来将该标志导出（export）,该命令只推荐在安装scrapy时使用，除了安装构建依赖外，在linux上安装没有其他特别的问题。在windows上使用PyPy来安装scrapy还未经测试。

你可以通过运行命令: ::

    scrapy bench

来检测scrapy是否安装成功,如果运行后出现类似如下的错误提示：TypeError: ... got 2 unexpected keyword arguments。说明这个安装工具没有能够找到一个PyPy特定依赖,为了解决这个问题,需要运行以下命令: ::

    > pip install 'PyPyDispatcher>=2.1.0'

.. _here: https://github.com/pyca/cryptography/issues/2692#issuecomment-272773481 
