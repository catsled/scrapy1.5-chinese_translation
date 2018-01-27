.. _docs-intro-scrapy-shell:

========================
命令行工具
========================

.. versionadded:: 0.10

Scrapy是通过scrapy命令行工具来控制的，为了将它与子命令(我们称之为“命令”或“Scrapy命令”)区分开来，在这里我们称它为“Scrapy工具”。

Scrapy工具提供了多个命令，为了不同的目的，每个命令都接受一系列不同的参数和选项。

( ``scrapy deploy`` 命令已经在 1.0版本中删除，取而代之的是独立的 ``scrapyd-deploy`` ， 参见 `deploy`_ 章节)

.. _docs-config-settings:

配置 settings
========================

Scrapy将在scrapy.cfg中查找配置参数，默认位置:

1.  ``/etc/scrapy.cfg`` 或 ``c:\scrapy\scrapy.cfg`` (系统范围),

2. ``~/.config/scrapy.cfg`` ( ``$XDG_CONFIG_HOME`` ) 和 ``~/.scrapy.cfg`` ( ``$HOME`` ) 全局 (用户范围) settings

3. ``scrapy.cfg`` 包含在Scrapy项目的根目录中 (看下部分).

这些文件中的配置项将按优先级顺序合并成列表:用户定义的值比系统级的默认值优先级更高并且当配置项被定义时，项目范围内的配置将覆盖所有其他的值。

Scrapy可以通过一些环境变量来进行设置，如下:

* ``SCRAPY_SETTINGS_MODULE`` (see :ref:`docs-topics-settings-module-envvar`)
* ``SCRAPY_PROJECT``
* ``SCRAPY_PYTHON_SHELL`` (see :ref:`docs-topics-shell`)

.. _docs-topics-project-structure:

Scrapy项目的默认结构
========================

在深入了解命令行工具和它的子命令之前，让我们先了解一下Scrapy项目的目录结构。

尽管它是可以被修改的，但是所有的Scrapy项目都默认生成相同的文件结构，类似于这样  ::

    scrapy.cfg
	myproject/
		__init__.py
		items.py
		middlewares.py
		pipelines.py
		settings.py
		spiders/
			__init__.py
			spider1.py
			spider2.py
			...

在项目目录中，有一个众所周知存放 ``scrapy.cfg`` 文件的目录叫项目的根目录。 该文件包含定义项目配置的python模块名称  ::

	[settings]
	default = myproject.settings
	
使用Scrapy工具
========================

你可以先运行一个没有参数的Scrapy工具，它会打印一些有用的帮助和可用的命令：  ::

	Scrapy X.Y - no active project

	Usage:
	scrapy <command> [options] [args]

	Available commands:
		crawl         Run a spider
		fetch         Fetch a URL using the Scrapy downloader
	[...]
	

第一行将打印当前活动的项目。在本例中，它是从项目外部运行的。如果从项目内部运行它会打印出下面的信息:  ::

	Scrapy X.Y - project: myproject

	Usage:
		scrapy <command> [options] [args]

	[...]

创建项目
-------------

当使用 ``scrapy`` 工具时，第一件要做事情常常是创建一个Scrapy项目:  ::

	scrapy startproject myproject [project_dir]
	

这样就会在 ``project_dir`` 目录下创建一个Scrapy项目。 如果 ``project_dir`` 没有定义的话，``project_dir`` 会默认和 ``myproject`` 相同。

下一步，你需要先进入到项目目录中：  ::

	cd project_dir
	

然后你就可以在这里使用 ``scrapy`` 命令来管理和控制你的项目了。


控制项目
---------------

在你的项目中使用 scrapy 工具来控制和管理它们

例如，创建一个新爬虫:  ::

	scrapy genspider mydomain mydomain.com

一些Scrapy命令（如 :command:`crawl`）必须在Scrapy项目内部运行。 参阅 :ref:`commands reference <topics-commands-ref>`，来获取详细信息。

还要注意的是，有些命令在从项目内部运行时可能会产生些许不同。 例如，当获取一些与特定 ``spider`` 相关联的url时，fetch命令将会产生覆盖(spider-overridden)行为(例如: ``use_agent`` 属性将会覆盖 user-agent)，这样做的意图是为了使用 ``fetch`` 命令去检测spiders是如何下载页面的。

.. _docs-topics-commands-ref:

 可用的工具命令
========================

本节包含了一个可用的内置命令列表及其描述和一些使用案例。 记住，你可以通过运行下面这个命令获得更多关于每个命令的信息:  ::

	scrapy <command> -h
	
你也可以使用这个命令获取全部可用的命令：  ::

	scrapy -h
	
共有两类命令，一类只在Scrapy项目内有效（特定于项目的命令），另一类是不需要Scrapy项目也可工作的命令(全局命令)，尽管它们在项目内中运行时可能稍有不同(因为将使用项目中的设置)。

全局命令:

* :command:`startproject`
* :command:`genspider`
* :command:`settings`
* :command:`runspider`
* :command:`shell`
* :command:`fetch`
* :command:`view`
* :command:`version`

仅项目命令:

* :command:`crawl`
* :command:`check`
* :command:`list`
* :command:`edit`
* :command:`parse`
* :command:`bench`

.. command:: startproject

startproject
--------------

* 语法: ``scrapy startproject <project_name> [project_dir]``
* 是否需要项目: *no* `

在 ``project_dir`` 目录下创建一个名为 ``project_name`` 的新项目。 如果没有指定 ``project_dir`` ，``project_dir`` 将与 ``project_name`` 相同。

用法示例： ::
	
	$ scrapy startproject myproject

.. command:: genspider

genspider
-------------

* 语法: ``scrapy genspider [-t template] <name> <domain>``
* 是否需要项目: *no*

如果从项目内部调用，则在当前文件夹或当前项目的 ``spiders`` 文件夹中创建一个新的spider。``<name>`` 参数设置为爬虫的 ``name``, 而 ``<domain>`` 用于生成spider的 ``allowed_domains`` 和 ``start_urls`` 属性。

用法示例： ::

	$ scrapy genspider -l
	Available templates:
	  basic
	  crawl
	  csvfeed
	  xmlfeed
	
	$ scrapy genspider example example.com
	Created spider 'example' using template 'basic'
	
	$ scrapy genspider -t crawl scrapyorg scrapy.org
	Created spider 'scrapyorg' using template 'crawl'
	

这仅仅是一个方便创建基于预定义模板爬虫的快捷命令，但这肯定不是创建爬虫的唯一方式。你也可以不使用这个命令，直接自己创建爬虫的源代码python文件。

.. command:: crawl

crawl
--------

* 语法: ``scrapy crawl <spider>``
* 是否需要项目: *yes*

使用爬虫开始抓取。

用法示例： ::

	$ scrapy crawl myspider
	[ ... myspider starts crawling ... ]
	
	
.. command:: check
	
check
-------

* 语法: ``scrapy check [-l] <spider>``
* 是否需要项目: *yes*

运行检查.

用法示例:  ::

	$ scrapy check -l
	first_spider
	 * parse
 	 * parse_item
	second_spider
 	 * parse
	 * parse_item

	$ scrapy check
	[FAILED] first_spider:parse_item
	>>> 'RetailPricex' field is missing

	[FAILED] first_spider:parse
	>>> Returned 92 requests, expected 0..4
	
.. command:: list	

list
------

* 语法: ``scrapy list``
* 是否需要项目: ``yes``

列出当前项目中所有可用的爬虫。输出的每一行就是可用爬虫的name。

用法示例: ::

	$ scrapy list
	spider1
	spider2
	
.. command:: edit
	
edit
------

* 语法: ``scrapy edit <spider>``
* 是否需要项目: ``yes``

使用EDITOR环境变量中设定的编辑器来编辑指定的爬虫，或是使用Editor设置的编辑器。

这个命令是为常见情况所提供的便利的快捷方式，开发人员当然也可以自由选择任何工具或IDE来编写和调试爬虫。

用法示例: ::

	$ scrapy edit spider1
	
.. command:: fetch
	
fetch
---------

* 语法: ``scrapy fetch <url>``
* 是否需要项目: *no*

使用Scrapy下载器下载指定的URL，并将内容写入标准输出中。

这个命令有趣的地方在于，爬虫怎样下载，就会怎样获取这个页面。 例如，如果爬虫有一个会覆盖用户代理的USER_AGENT属性，那么这个命令也会使用这个属性。

因此，这个命令可以用来“查看”你的爬虫如何获取某个页面

如果在一个项目之外使用，就不需要使用特定的Scrapy内容，它只会使用默认的Scrapy下载器设置。

支持选项:

* ``--spider=SPIDER``: 通过爬虫自动检测和强制使用特定的爬虫；

* ``--headers``: 打印响应的HTTP头，而不是响应的正文；

* ``--no-redirect``: 不追踪 HTTP 3xx的重定向(默认是追踪)；

用法示例:  ::

	$ scrapy fetch --nolog http://www.example.com/some/page.html
	[ ... html content here ... ]
	
	$ scrapy fetch --nolog --headers http://www.example.com/
	{'Accept-Ranges': ['bytes'],
	'Age': ['1263   '],
	'Connection': ['close     '],
	'Content-Length': ['596'],
	'Content-Type': ['text/html; charset=UTF-8'],
	'Date': ['Wed, 18 Aug 2010 23:59:46 GMT'],
	'Etag': ['"573c1-254-48c9c87349680"'],
	'Last-Modified': ['Fri, 30 Jul 2010 15:30:18 GMT'],
	'Server': ['Apache/2.2.3 (CentOS)']}
	
.. command:: view
	
view
-----

* 语法: ``scrapy view <url>``
* 是否需要项目: *no*

在浏览器中打开给定的URL，就像你的Scrapy“看到”一样。 有时，爬虫与普通用户的查看到的页面不同，因此这可以用来检查爬虫“看到”的内容，并确认它是你所期望的那样。

支持选项:

* ``--spider=SPIDER``: 通过爬虫自动检测和强制使用特定的爬虫；

* ``--no-redirect``: 不追踪 HTTP 3xx的重定向(默认是追踪)；

用法示例:  ::

	$ scrapy view http://www.example.com/some/page.html
	[ ... browser starts ... ]

.. command:: shell

shell
-------

* 语法: ``scrapy shell [url]``
* 是否需要项目: *no*

启动Scrapy shell访问URL(如果给定的话)，如果没有给定URL，则为空。 同样也支持Unix风格的本地文件路径，``./`` ， ``../`` 。 更多信息请参见 :ref:`docs-topics-shell` 。

支持选项:

* ``--spider=SPIDER``: 通过爬虫自动检测和强制使用特定的爬虫；

* ``-c code``: 计算shell中的代码，打印结果并退出；

* ``--no-redirect``: 不追踪 HTTP 3xx的重定向(默认是追踪); 这只会影响在命令行上作为参数传递的URL; 一旦你在shell中，默认情况 fetch(url)就会追踪HTTP重定向。

用法示例: ::

	$ scrapy shell http://www.example.com/some/page.html
	[ ... scrapy shell starts ... ]
	
	$ scrapy shell --nolog http://www.example.com/ -c '(response.status, response.url)'
	(200, 'http://www.example.com/')
	
	# shell follows HTTP redirects by default
	$ scrapy shell --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
	(200, 'http://example.com/')
	
	# you can disable this with --no-redirect
	# (only for the URL passed as command line argument)
	$ scrapy shell --no-redirect --nolog http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F -c '(response.status, response.url)'
	(302, 'http://httpbin.org/redirect-to?url=http%3A%2F%2Fexample.com%2F')
	
.. command:: parse
	
parse
-------

* 语法: ``scrapy parse <url> [options]``
* 是否需要项目: *yes*

获取给定的url，通过 ``--callback`` 选项向spider中传入函数，或者默认的 ``parse`` 方法来处理响应 。

支持选项:

* ``--spider=SPIDER``: 绕过spider自动检测并且强制使用特定的spider

* ``--a NAME=VALUE``: 设置爬虫参数(可以重复)

* ``--callback or -c``: 用于解析响应的爬虫方法

* ``--meta`` or ``-m``: 附加的request meta值将会被传递到回调函数的request中。该值必须是一个可用的json字符串。 如: --meta='{"foo": "bar"}'

* ``--pipelines``: 指定pipeline

* ``--rules`` or ``-r``: 使用 :class:`~scrapy.spiders.CrawlSpider` 规则来去发现用来解析响应的回调方法(即spider方法)。

* ``--noitems``: 不显示抓取到的item

* ``--nolinks``: 不显示提取的链接

* ``--nocolour``: 避免对输出进行着色

* ``--depth`` or ``-d``: 需要递归请求的深度级别(默认:1)

* ``--verbose`` or ``-v``: 显示每个深度级别的信息

用法示例: ::

	$ scrapy parse http://www.example.com/ -c parse_item
	[ ... scrapy log lines crawling example.com spider ... ]
	
	>>> STATUS DEPTH LEVEL 1 <<<
	# Scraped Items  ------------------------------------------------------------
	[{'name': u'Example item',
	'category': u'Furniture',
	'length': u'12 cm'}]

	# Requests  -----------------------------------------------------------------
	[]
	
.. command:: settings
	
settings
----------

* 语法: ``scrapy settings [options]``
* 是否需要项目: *no*

从Scrapy setting获取值。

如果在项目中使用，它将显示项目设置值，否则它将显示Scrapy的默认值。

用法示例： ::

	$ scrapy settings --get BOT_NAME
	scrapybot
	$ scrapy settings --get DOWNLOAD_DELAY
	0
	

.. commands:: runspider
	
runspider
------------

* 语法: ``scrapy runspider <spider_file.py>``
* 是否需要项目: *no*

运行一个包含爬虫的Python文件，而不再需要创建一个项目。

用法示例: ::
	$ scrapy runspider myspider.py
	[ ... spider starts crawling ... ]
	
.. command:: version
	
version
--------

* 语法: ``scrapy version [-v]``
* 是否需要项目: *no*

打印Scrapy版本。 如果scrapy与 ``-v`` 一起使用时，它还会打印Python、Twisted和Platform 的信息，这对bug报告非常有用。

.. command:: bench

bench
-------

.. versionadded:: 0.17

* 语法: ``scrapy bench``
* 是否需要项目: *no*

快速启动一个benchmark测试。 :ref:`benchmarking`


自定义项目命令
========================

你也可以使用 :setting:`COMMANDS_MODULE` 设置来添加你自定义的项目命令。
有关如何实现您的命令的示例，请参阅 `scrapy/commands`_ 中的Scrapy命令。

.. _scrapy/commands: https://github.com/scrapy/scrapy/tree/master/scrapy/commands
.. setting:: COMMANDS_MODULE

命令模块
-----------

默认: ``''`` (空字符串)

用于查找Scrapy自定义命令和用于为自己的Scrapy项目添加自定义的命令模块。

用法示例： ::
	COMMANDS_MODULE = 'mybot.commands'
	
通过setup.py注册命令
------------------------------

.. note:: 这是一个经验性的特性,请小心使用。
	
通过向 ``setup.py`` 库文件的接入点(entry point)添加一个 ``scrapy.commands`` 来引入一个外部库，。

下面的示例是添加一个名为 my_command 的命令:  ::

	from setuptools import setup, find_packages

	setup(name='scrapy-mymodule',
	entry_points={
		'scrapy.commands': [
		'my_command=my_scrapy_module.commands:MyCommand',
		],
	   },
	)
