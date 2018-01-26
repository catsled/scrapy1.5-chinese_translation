.. _docs-intro-scrapy-shell:

========================
 命令行工具
========================

Scrapy是通过scrapy命令行工具来控制的，在这里被称为“Scrapy工具”，可以将它与子命令区分开来，我们称之为“命令”或“Scrapy命令”。

Scrapy工具提供了多个命令，用于多个目的，每个命令都接受不同的参数和选项。

(scrapy deploy 命令已经在 1.0版本中删除，取而代之的是独立的scrapyd-deploy， 参见 deploy章节)

========================
 配置设置
========================

Scrapy将在scrapy.cfg中查找配置参数，默认位置:

#. /etc/scrapy.cfg 或 c:\scrapy\scrapy.cfg (系统范围),

#. ~/.config/scrapy.cfg ($XDG_CONFIG_HOME) 和 ~/.scrapy.cfg ($HOME) 全局 (用户范围) settings

#. Scrapy项目的根目录中 scrapy.cfg (看下部分).

这些文件中的配置项将按优先级顺序合并成列表:用户定义的值比系统级的默认值优先级更高并且当配置项被定义时，项目范围内的配置将覆盖所有其他的值。

Scrapy也可以被找到并且可以通过一些环境变量，如下:

* SCRAPY_SETTINGS_MODULE (see Designating the settings)
* SCRAPY_PROJECT
* SCRAPY_PYTHON_SHELL (see scrapy shell)

========================
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

在项目目录中，有一个众所周知存放scrapy.cfg文件的目录叫项目的根目录。 该文件包含定义项目配置的python模块名称  ::

	[settings]
	default = myproject.settings
	

========================
 用Scrapy工具
========================

你可以先运行一个没有参数的Scrapy工具，它会打印一些有用的帮助和可用的命令：  ::

	Scrapy X.Y - no active project

	Usage:
	scrapy <command> [options] [args]

	Available commands:
		crawl         Run a spider
		fetch         Fetch a URL using the Scrapy downloader
	[...]
	

第一行将打印当前活动的项目。在本例中，它是从项目外部运行的。如果从项目内部运行它会打印出这样的东西:  ::

	Scrapy X.Y - project: myproject

	Usage:
		scrapy <command> [options] [args]

	[...]

========================
 创建项目
========================

通常你使用scrapy工具首先会做的事应该就是创建你的Scrapy项目:  ::

	scrapy startproject myproject [project_dir]
	

这样就会在project_dir目录下创建一个Scrapy项目。 如果project_dir没有定义的话，project_dir会默认和myproject相同。

下一步，你需要先进入到项目目录中：  ::

	cd project_dir
	

然后你就可以在这里使用scrapy命令来管理和控制你的项目了。

========================
 控制项目
========================

你在你的项目中使用 scrapy 工具来控制和管理它们

例如，创建一个新爬虫:  ::

	scrapy genspider mydomain mydomain.com

一些Scrapy命令（像 crawl）必须在Scrapy程序内部才能运行。 请参阅下面的 命令参考，以获得更多关于哪些命令必须从项目内部、哪些不必须的信息。

还要注意的是，有些命令在从项目内部运行时可能会产生些许不同。 例如，如果获取的url与某些特定的爬虫相关联，fetch 命令将使用spider - overridden（爬虫 - 覆盖）行为(比如 user_agent属性覆盖user-agent)。这是有意的，因为 fetch 命令是用来检查爬虫如何下载页面的。

========================
 可用的工具命令
========================

这个章节包含一个可用的内置命令列表中关于命令的描述和用法示例。 记住，你可以通过运行下面这个命令获得更多关于每个命令的信息:  ::

	scrapy <command> -h
	
你也可以使用这个命令获取全部可用的命令：  ::

	scrapy -h
	
共有两类命令，一类只在Scrapy项目内有效（特定于项目的命令），另一类是不需要Scrapy项目也可工作的命令(全局命令)，尽管它们在项目内中运行时可能稍有不同(因为将使用项目被覆盖的设置)。

全局命令:

* startproject
* genspider
* settings
* runspider
* shell
* fetch
* view
* version

仅项目命令:

* crawl
* check
* list
* edit
* parse
* bench

**开始项目**

* 语法: scrapy startproject <project_name> [project_dir]
* 需要项目: no

在 project_dir 目录下创建一个名为 project_name 的新项目。 如果没有指定 project_dir，project_dir 将与project_name相同。

**genspider**

* 语法: scrapy genspider [-t template] <name> <domain>
* 需要项目 project: no

如果从项目内部调用，则在当前文件夹或当前项目的spider文件夹中创建一个新的spider。<name> 参数设置为爬虫的 name, 而 <domain> 用于生成 allowed_domains 和 start_urls 这两个爬虫属性。

用例：  ::

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

**crawl**

* 语法: scrapy crawl <spider>
* 需要项目: yes

使用爬虫开始抓取。

用例:  ::

	$ scrapy crawl myspider
	[ ... myspider starts crawling ... ]
	
	
**check**

* 语法: scrapy check [-l] <spider>
* 需要项目: yes

运行检查.

用例:  ::

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
	
**list**

* 语法: scrapy list
* 需要项目: yes

列出当前项目中所有可用的爬虫。输出的每一行就是可用爬虫的name。

用例:  ::

	$ scrapy list
	spider1
	spider2
	
**edit**

* Syntax: scrapy edit <spider>
* Requires project: yes

使用EDITOR环境变量中设定的编辑器来编辑指定的爬虫，或是使用Editor设置的编辑器。

这个命令是为常见情况所提供的便利的快捷方式，开发人员当然也可以自由选择任何工具或IDE来编写和调试爬虫。

用例:  ::

	$ scrapy edit spider1
	
**fetch**

* Syntax: scrapy fetch <url>
* Requires project: no

使用Scrapy下载器下载指定的URL，并将内容写入标准输出中。

这个命令有趣的地方在于，爬虫怎样下载，就会怎样获取这个页面。 例如，如果爬虫有一个会覆盖用户代理的USER_AGENT属性，那么这个命令也会使用这个属性。

因此，这个命令可以用来“查看”你的爬虫如何获取某个页面

如果在一个项目之外使用，就不需要使用特定的Scrapy内容，它只会使用默认的Scrapy下载器设置。

支持选项:

* --spider=SPIDER: 通过爬虫自动检测和强制使用特定的爬虫；

* --headers: 打印响应的HTTP头，而不是响应的正文；

* --no-redirect: 不追踪 HTTP 3xx的重定向(默认是追踪)；

用例:  ::

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
	
**view**

* Syntax: scrapy view <url>
* Requires project: no

在浏览器中打开给定的URL，因为你的Scrapy将“看到”它。 有时，爬虫与普通用户的查看到的页面不同，因此这可以用来检查爬虫“看到”的内容，并确认它是你所期望的那样。

支持选项:

* --spider=SPIDER: 通过爬虫自动检测和强制使用特定的爬虫；

* --no-redirect: 不追踪 HTTP 3xx的重定向(默认是追踪)；

用例:  ::

	$ scrapy view http://www.example.com/some/page.html
	[ ... browser starts ... ]
	
**shell**

* Syntax: scrapy shell [url]
* Requires project: no

启动Scrapy shell访问URL(如果给定的话)，如果没有给定URL，则为空。 也支持Unix风格的本地文件路径，./ 、 ../前缀或绝对文件路径。 更多信息请参见shell。

支持选项:

* --spider=SPIDER: 通过爬虫自动检测和强制使用特定的爬虫；

* -c code: 计算shell中的代码，打印结果并退出；

* --no-redirect: 不追踪 HTTP 3xx的重定向(默认是追踪); 这只会影响在命令行上作为参数传递的URL; 一旦你在shell中，默认情况 fetch(url)就会追踪HTTP重定向。

用例:  ::

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
	
**parse**

* Syntax: scrapy parse <url> [options]
* Requires project: yes

传入--callback选项，让爬虫获取给定的URL处理并解析，如果不传，则使用parse

支持选项:

* --spider=SPIDER: 通过蜘蛛自动检测和强制使用特定的蜘蛛

* --a NAME=VALUE: 设置爬虫参数(可以重复)

* --callback or -c: 用于解析响应的爬虫方法

* --pipelines: 指定pipeline

* --rules or -r: 使用CrawlSpider规则来发现回调(即蜘蛛方法)来解析响应。

* --noitems: 不显示抓取到的item

* --nolinks: 不显示提取的链接

* --nocolour: 避免对输出进行着色

* --depth or -d: 需要递归请求的深度级别(默认:1)

* --verbose or -v: 显示每个深度级别的信息

用例:  ::

	$ scrapy parse http://www.example.com/ -c parse_item
	[ ... scrapy log lines crawling example.com spider ... ]
	
	>>> STATUS DEPTH LEVEL 1 <<<
	# Scraped Items  ------------------------------------------------------------
	[{'name': u'Example item',
	'category': u'Furniture',
	'length': u'12 cm'}]

	# Requests  -----------------------------------------------------------------
	[]
	
**settings**

* Syntax: scrapy settings [options]
* Requires project: no

获取Scrapy设置的属性值。

如果在项目中使用，它将显示项目设置值，否则它将显示Scrapy的默认值。

示例:  ::

	$ scrapy settings --get BOT_NAME
	scrapybot
	$ scrapy settings --get DOWNLOAD_DELAY
	0
	
**runspider**

* Syntax: scrapy runspider <spider_file.py>
* Requires project: no

运行一个包含爬虫的Python文件，而不再需要创建一个项目。

示例:  ::
	$ scrapy runspider myspider.py
	[ ... spider starts crawling ... ]
	
**version**

* Syntax: scrapy version [-v]
* Requires project: no

打印Scrapy版本。 如果scrapy与-v一起使用时，它还会打印Python、Twisted和Platform info，这对bug报告非常有用。

**bench**

在新版本0.17

* Syntax: scrapy bench
* Requires project: no

快速启动一个benchmark测试。

========================
 自定义项目命令
========================

你也可以使用COMMANDS_MODULE设置来添加你自定义的项目命令。有关如何实现您的命令的示例，请参阅scrapy/commands中的Scrapy命令。

**命令模块**

默认: '' (空字符串)

用于查找Scrapy自定义命令和用于为自己的Scrapy项目添加自定义命令的模块。

示例:  ::

	COMMANDS_MODULE = 'mybot.commands'
	
**通过setup.py来注册命令**

.. note::

	这是一个经验性的特性,请小心使用。
	
你还可以通过引入一个外部库，在库中的setup.py文件的entry_points中添加scrapy.commands部分以此来添加Scrapy命令。

下面的示例是添加一个名为 my_command 的命令:  ::

	from setuptools import setup, find_packages

	setup(name='scrapy-mymodule',
	entry_points={
		'scrapy.commands': [
		'my_command=my_scrapy_module.commands:MyCommand',
		],
	},
	)












