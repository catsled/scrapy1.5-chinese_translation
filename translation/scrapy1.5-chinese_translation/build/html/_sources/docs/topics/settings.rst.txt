.. _docs-topics-settings:

==============
配置(Settings)
==============

Scrapy settings 允许你定制所有Scrapy组件的行为，包括: core, extensions, pipelines 和 spiders。

settings 的基础架构提供了一个可供其他代码获取配置属性的全局键值对(key-value)映射命名空间。settings也
可以通过其他途径进行配置。

同样地，settings也提供了选择当前正在运行的Scrapy project(如果你有很多project)。

查看可用的内置settings列表: :ref:`docs-topics-settings-ref`

.. _docs-topics-settings-module-envvar:

指定settings
======================

当使用Scrapy时， 你需要指定你要使用哪一个settings文件。可以使用环境变量 ``SCRAPY_SETTINGS_MODULE``
来实现该功能。

``SCRAPY_SETTINGS_MODULE`` 的值应该使用Python路径语法(Python path syntax)编写, 比如说: ``myproject.settings``。
注意，settings模块应该包含在Python的查找路径中 `import search path`_ 。

.. _import search path: https://docs.python.org/2/tutorial/modules.html#the-module-search-path


填充settings
==============

Settings可以通过不同的方法进行填充，每一种方法拥有不同的优先级。优先级顺序列表:

 1. 命令行选项 (Commoand line options) -- 优先级最高
 2. 每个爬虫的配置 (Settings pre-spider)
 3. 项目配置模块 (Project settings module)
 4. 每条命令的默认配置 (Default settings per-command)
 5. 默认全局配置 (Default global settings) -- 优先级最低

填充这些配置源是由内部机制谨慎完成的，但是也可以通过调用API进行手动处理。参考 :ref:`docs-topics-api-settings`。

更多方法的详细描述。

1. 命令行选项(Command line options)
-------------------------------------

通过命令行提供的参数拥有最高的优先级，可以用来覆盖其他的任何选项。你可以使用 ``-s`` (或 ``--set``) 命令行选项
来明确的指定要覆盖哪一个或者哪些配置信息。

.. highlight:: sh

例::

    scrapy crawl myspider -s LOG_FILE=scrapy.log

2. 配置每一个spider (Settings per-spider)
------------------------------------------

Spiders 可以定义自己的配置信息，该配置信息的优先级高于项目(project)的配置信息，并且会覆盖项目的配置信息。
通过设置spiders的 :attr:`~scrapy.spiders.Spider.custom_settings` 属性实现 ::

    class MySpider(scrapy.Spider):
        name = 'myspider'

        custom_settings = {
            'SOME_SETTING': 'some value',
        }

3. 项目配置模块(Project settings module)
----------------------------------------

项目配置模块是Scrapy项目(Scrapy prject)的标准配置文件，你的大部分配置信息都会写在该文件中。
对于一个标准的Scrapy项目，这意味着你将会在对应的 ``settings.py`` 文件中添加或修改配置信息。

4. 每条命令的默认配置(Default settings per-command)
---------------------------------------------------

每一个 :doc:`Scrapy tool </topics/commands>` 命令都可以拥有自己的默认配置，该配置可以覆盖全局
默认配置。这些自定义配置可以通过命令类(command class)的 ``default_settings`` 属性进行指定。

5. 默认全局配置(Default global settings)
-----------------------------------------

默认全局配置位于 ``scrapy.settings.default_settings`` 模块中。相关文档 :ref:`docs-topics-settings-ref` 。


如何获取配置信息
===================

.. highlight:: python

在一个spider中，可以同过 ``self.settings`` 获取settings信息 ::

    class MySpider(scrapy.Spider):
        name = 'myspider'
        start_urls = ['http://example.com']

        def parse(self, response):
            print("Existing settings: %s" % self.settings.attributes.keys())

.. note::
    ``settings`` 只有在spider初始化以后才会被设置到基础的Spider类中。如果你想在初始化之前获取
    配置信息(如: 在你的spider中的 ``__init__()`` 方法中使用)，你需要重写 
    :meth:`~scrapy.spiders.Spider.from_crawler` 方法。

可以通过向extensions, middlewares 和 item pipelines中的 ``from_crawler`` 方法传递的Crawler
对象的 :attr:`scrapy.crawler.Crawler.settings` 属性获取配置信息。 ::

    class MyExtension(object):
        def __init__(self, log_is_enabled=False):
            if log_is_enabled:
                print("log is enabled!")

        @classmethod
        def from_crawler(cls, crawler):
            settings = crawler.settings
            return cls(settings.getbool('LOG_ENABLED'))

你可以像使用字典一样使用配置对象(如: ``settings['LOG_ENABLED']`` )，但是，为了避免类型错误，
更好的做法是使用特定的格式来提取你需要的信息。对应方法参考: :class:`~scrapy.settings.Settings` API。


合理地命名配置名称(Retionale for settings names)
====================================================

配置名称通常以对应的配置组件名称作为前缀。比如说: 一个虚构的 robots.txt 扩展的配置名称应该为 ``ROBOTSTXT_ENABLED``，
``ROBOTSTXT_OBEY``，``ROBOTSTXT_CACHEDIE``，等。

.. _docs-topics-settings-ref:


内置配置参考(Built-in settings reference)
============================================

这里以字母序列出了所有可用的Scrapy配置列表，以及默认值和使用范围。

如果绑定了特定组件，给出的范围以及是否可用将会表示该配置会被在何处使用。在这种情况下，
将会给出该组件的模块，该组件通常是 extensions, middleware 或 pipeline。这也意味着，
为了使该配置生效，我们必须启用该组件。

.. setting:: AWS_ACCESS_KEY_ID

AWS_ACCESS_KEY_ID
-------------------

默认值: ``None``

AWS访问秘钥(access key) 将会被用于需要连接 `Amazon Web services`_ 的代码中，
如: :ref:`S3 feed storage backend <topics-feed-storage-s3>`。

.. setting:: AWS_SECRET_ACCESS_KEY

AWS_SECRET_ACCESS_KEY_ID
----------------------------

默认值: ``None``

AWS加密访问秘钥(secret access key) 将会被用于需要连接 `Amazon Web services`_ 的代码中，
如: :ref:`S3 feed storage backend <topics-feed-storage-s3>`。

.. settings:: BOT_NAME

BOT_NAME
---------

默认值: ``'scrapybot'``

爬虫机器人的名称通常以爬虫项目的名称命名。该名称将会被用来构建默认的User-Agent和记录日志。

当使用 :command:`startproject` 命令创建项目时，BOT_NAME将会自动根据项目名称自动填充。

.. settings:: CONCURRENT_ITEMS


CONCURRENT_ITEMS
-------------------

默认值: ``100``

可同时并行处理的最大items数量 (参考 :ref:`Item Pipeline <topics-item-pipeline>`)。

.. setting:: CONCURRENT_REQUESTS

CONCURRENT_REQUESTS
-----------------------

默认值: ``16``

可同时被Scrapy 下载器(downloader) 处理的最大请求数。

.. setting:: CONCURRENT_REQUESTS_PER_DOMAN

CONCURRENT_REQUESTS_PER_DOMAN
-------------------------------

默认值: ``8``

可同时向单个域名请求的最大数量。

参考: :ref:`docs-topics-autothrottle` 和对应的设置选项 :settings:`AUTOTHROTTLE_TARGET_CONCURRENCY` 。

.. setting:: CONCURRENT_REQUESTS_PER_IP

CONCURRENT_REQUESTS_PER_IP
----------------------------

默认值: ``0``

可同时向单个IP请求的最大数量。如果值非零，那么配置 :setting:`CONCURRENT_REQUESTS_PER_DOMAN` 将会被忽略，
也就是并发量将会被 ``CONCURRENT_REQUESTS_PER_IP`` 限制，而不会被 ``CONCURRENT_REQUESTS_PER_DOMAN``
限制。

该配置同样会影响 :setting:`DOWNLOAD_DELAY` 以及 :ref:`docs-topics-autothrottle`: 如果，:settings:`CONCURRENT_REQUESTS_PER_IP`
的值非零，下载延时(download delay) 将会被IP限制，而不是被域名限制。


.. setting:: DEFAULT_ITEM_CLASS

DEFAULT_ITEM_CLASS
----------------------

默认值: ``'scrapy.item.Item'``

该默认类将会被用来在 :ref:`the Scrapy shell <docs-topics-shell>` 中实例化items。

.. setting:: DEFAULT_REQUEST_HEADERS

DEFAULT_REQUEST_HEADERS
---------------------------

默认值 ::

    {
        'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8',
        'Accept-Language': 'en',
    }

默认头信息将会在Scrapy的HTTP请求(Scrapy HTTP Requests)中使用。这些信息在 :class:`~scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware`
中被填充。

.. setting:: DEPTH_LIMIT

DEPTH_LIMIT
-------------

默认值: ``0``

作用域: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

允许对每一个网站爬取的最大深度。如果值为0，则不会有任何限制。

.. settings:: DEPTH_PRIORITY

DEPTH_PRIORITY
----------------

默认值: ``0``

作用域: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

使用一个整数调整基于深度的请求优先级:

- 0(默认值), 不会对深度进行优先级调整
- **正数将会对优先级进行降序调整(深度越大优先级越低)**; 该设置通常用来实现广度优先爬取 (BFO)
- 负数会对优先级进行升序调整(深度越大优先级越高), 该设置通常用来实现深度优先爬取 (DFO)

参考: :ref:`faq-bfo-dfo`

.. note::

    该设置对于优先级的调整与其他优先级设置的策略刚好相反。如: :settings:`REDIRECT_PRIORITY_ADJUST`
    和 :setting:`RETRY_PRIORITY_ADJUST`

.. setting:: DEPTH_STATS

DEPTH_STATS
-------------

默认值: ``True``

作用域: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

是否收集最大深度的统计信息。

.. setting:: DEPTH_STATS_VERBOSE

DEPTH_STATS_VERBOSE
---------------------

默认值: ``False``

作用域: ``scrapy.spidermiddlewares.depth.DepthMiddleware``

是否收集冗余的深度统计信息。如果启用该配置，将会收集每一个深度对应的请求统计信息。

.. setting:: DNSCACHE_ENABLED

DNSCACHE_ENABLED
------------------

默认值: ``True``

是否启用缓存中的DNS。

.. setting:: DNSCACHE_SIZE

DNSCACHE_SIZE
---------------

默认值: ``1000``

缓存中DNS的大小。

.. setting:: DNS_TIMEOUT

DNS_TIMEOUT
-------------

默认值: ``60``

DNS查询过期时间(单位: 秒)，支持小数。

.. setting:: DOWNLOADER

DOWNLOADER
------------

默认值: ``'scrapy.core.downloader.Downloader'``

爬取时用到的下载器。

.. setting:: DOWNLOADER_HTTPCLIENTFACTORY

DOWNLOADER_HTTPCLIENTFACTORY
-------------------------------

默认值: ``'scrapy.core.downloader.webclient.ScrapyHTTPClientFactory'``

定义一个用来进行 HTTP/1.0 连接( ``HTTP10DownloadHandler`` )的Twisted ``protocol.ClientFactory`` 类。

.. note::

    目前,HTTP/1.0几乎不会被使用，所以你可以放心的忽略此配置。除非你使用的Twisted的版本小于 11.1版本，
    或者你真的想要使用HTTP/1.0，你可以通过使用 ``'scrapy.core.downloader.handlers.http.HTTP10DownloadHandler'``
    覆盖 ``http(s)`` 策略对应的 :setting:`DOWNLOAD_HANDLERS_BASE` 。

.. setting:: DOWNLOADER_CLIENTCONTEXTFACTORY

DOWNLOADER_CLIENTCONTEXTFACTORY
-----------------------------------

默认值: ``'scrapy.core.downloader.contextfactory.ScrapyClientContextFactory'``

代表了ContextFactory将会使用的类路径(classpath)

这里的"ContextFactory" 是指Twisted中关于SSL/TLS上下文环境的一个术语，该配置定义了TLS/SSL协议使用
的版本，是否需要证书认证，或是否启用客户端认证，以及其他的功能。

.. note::

    Scrapy的默认上下文工厂并不理会远程服务器证书认证(remote server certitficate verification)。
    对于网站爬取来说，这总是一个不错的选择。

    如果你确实需要启用远程服务器证书认证，Scrapy同样也提供了另一个上下文工厂类供你设置，
    ``'scrapy.core.downloader.contextfactory.BrowserLikeContextFactory'`` 使用了平台的
    证书去验证远程的端点。**要使用该功能，Twisted>=14.0.**

如果你确实要使用自定义的ContextFactory，确保在初始化时接受一个方法参数，该方法是映射了 
:settings:`DOWNLOADER_CLIENT_TLS_METHOD` 的 ``OpenSSL.SSL`` 方法。

.. setting:: DOWNLOADER_CLIENT_TLS_METHOD

DOWNLOADER_CLIENT_TLS_METHOD
---------------------------------

默认值: ``'TLS'``

在使用默认的 HTTP/1.1 下载器之前使用该配置来自定义 TLS/SSL 方法。

该设置的值必须为下列字符串值中的一个:

- ``'TLS'``: 映射到OpenSSL的 ``TLS_method()`` (又名 ``SSlv23_method()`` )，
  允许协议协商，并且可以从平台的最高层开始支持； **默认，推荐**
- ``'TLSv1.0'``: 该值强制HTTPS使用1.0版本的TLS进行连接；在Scrapy<1.1的版本中使用
- ``'TLSv1.1'``: 强制使用1.1版本的TLS
- ``'TLSv1.2'``: 强制使用1.2版本的TLS
- ``'SSLv3'``: 强制使用3版本的SSL (**不推荐使用**)

.. note::

    我们推荐你使用 PyOpenSSl>=0.13的版本，Twisted>=0.13或更高的版本。

.. setting:: DOWNLOADER_MIDDLEWARES

DOWNLOADER_MIDDLEWARES
------------------------

默认值:: ``{}``

一个包含了可以在你的项目中使用的中间件以及对应顺序的字典。更多信息 :ref:`docs-topics-downloader-middleware-setting`。

.. setting:: DOWNLOADER_MIDDLEWARES_BASE

DOWNLOADER_MIDDLEWARES_BASE
-------------------------------

默认值::

    {
        'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
        'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
        'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
        'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
        'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
        'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
        'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
        'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
        'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
        'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
        'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
        'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
        'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
        'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
    }

一个包含了在Scrapy中默认启用的下载器中间件的字典。顺序较低的接近Scrapy引擎，顺序较高的接近下载器中间件。
永远不要在你的项目中修改该配置，但是你可以通过修改 :setting:`DOWNLOADER_MIDDLEWARES` 来实现相应的功能。
查看更多信息 :ref:`docs-topics-downloader-middleware-setting` 。

.. setting:: DOWNLOADER_STATS

DOWNLOADER_STATS
------------------

默认值: ``True``

是否收集下载器统计信息。

.. setting:: DOWNLOAD_DELAY

DOWNLOAD_DELAY
---------------

默认值: ``0``

在下载同一网站的连续页面时，下载器的等待时间(单位: 秒)。使用该配置对爬取速度进行限制，以免对目标服务器造成过大的压力。
(支持小数)。 例 ::

    DOWNLOAD_DELAY = 0.25  # 延迟 250毫秒

该配置也被 :setting:`RANDOMIZE_DOWNLOAD_DELAY` (默认开启) 影响。默认情况下，Scrapy不会在两次请求之间
等待一个固定的时间，而是在 0.5 * :setting:`DOWNLOAD_DELAY` 和 1.5 * :setting:`DOWNLOAD_DELAY` 
之间随机选取一个时间值。

当 :setting:`CONCURRENT_REQUESTS_PER_IP` 的值不为0时，请求延时将会被每个ip限制，而不是每一个域名(domain)。

你也可以通过修改每一个spider中的 ``download_delay`` spider属性对该配置进行修改。

.. setting:: DOWNLOAD_HANDLERS

DOWNLOAD_HANDLERS
-------------------

默认值: ``{}``

一个包含了在你的项目中所启用的请求下载器处理器(request downloader handlers)。
参考 :setting:`DOWNLOAD_HANDLERS_BASE`。

.. setting:: DOWNLOAD_HANDLERS_BASE

DOWNLOAD_HANDLERS_BASE
------------------------

默认值 ::

    {
        'file': 'scrapy.core.downloader.handlers.file.FileDownloadHandler',
        'http': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
        'https': 'scrapy.core.downloader.handlers.http.HTTPDownloadHandler',
        's3': 'scrapy.core.downloader.handlers.s3.S3DownloadHandler',
        'ftp': 'scrapy.core.downloader.handlers.ftp.FTPDownloadHandler',
    }

一个包含了在Scrapy中默认启用的请求下载处理器的字典。你不应该在你的项目中修改该配置，但是你可以通
过修改 :setting:`DOWNLOAD_HANDLERS` 来实现相应的功能。

你可以通过将 :setting:`DOWNLOAD_HANDLERS` 中的URI设定为 ``None`` 来关闭这些下载处理器(download handlers)。
比如说，关闭内置的FTP处理器，可以在你的 ``settings.py`` 中这样做 ::

    DOWNLOAD_HANDLERS = {
        'ftp': None,
    }

.. setting:: DOWNLOAD_TIMEOUT

DOWNLOAD_TIMEOUT
-------------------

默认值: ``180``

为下载器设定超时时间(单位: 秒)。

.. note::

    你可以通过spider的 :attr:`download_timeout` 属性为每一个spider设置超时时间，也可以通过
    使用 :reqmeta:`download_timeout` 在Request.meta 中对每一个请求设定一个超时时间。

.. setting:: DOWNLOAD_MAXSIZE

DOWNLOAD_MAXSIZE
------------------

默认值: `1073741824` (1024MB)

下载器下载的最大响应大小(单位: 字节)。

如果你想要禁用该功能，将其值设定为0。

.. reqmeta:: download_maxsize

.. note::

    你可以通过spider的 :attr:`download_maxsize` 属性为每一个spider设置下载大小，也可以通过
    使用 :reqmeta:`download_maxsize` 在Request.meta 中对每一个请求设定一个下载大小。

    需要Twisted >= 11.1. 

.. setting:: DOWNLOAD_WARNSIZE

DOWNLOAD_WARNSIZE
--------------------

默认值: `33554432` (32MB)

下载器将会对响应的大小进行提示。

如果你想要禁用该功能，将其值设定为0。

.. note::

    你可以通过spider的 :attr:`download_warnsize` 属性为每一个spider设置下载大小，也可以通过
    使用 :reqmeta:`download_warnsize` 在Request.meta 中对每一个请求设定一个下载大小。

    需要Twisted >= 11.1. 

.. setting:: DOWNLOAD_FAIL_ON_DATALOSS

DOWNLOAD_FAIL_ON_DATALOSS
---------------------------

默认值: ``True``

中断的响应是否失败。也就是声明的 ``Content-Length`` 与服务器传送过来的不一致，或者是一部分响应
没有正常结束。如果将该值设置为 ``True``，响应将会引起(抛出)一个 ``ResponseFailed([_DataLoss])``
错误。如果设置为 ``False``，将会略过响应并为其添加一个 ``dataloss`` 标签，即: ``'dataloss' in response.flags`` 为 ``True`` 。

作为一个可选项，你可以通过将Request.meta中的 :reqmeta:`download_fail_on_dataloss` 设置为 ``False``
来实现对于每一个请求的设置。

.. note::

    从服务器的配置错误到网络错误再到数据错误都有可能会引起响应的中断，或者数据丢失。考虑到中断的响应中可能会
    包含部分不完整的内容，用户可以决定处理这些中断的响应是否有意义。如果 :setting:`RETRY_ENABLED` 为 ``True``
    并且该配置( ``DOWNLOAD_FAIL_ON_DATALOSS`` ) 也为 ``True``， 那么 ``ResponseFailed([_DataLoss])``
    错误仍然会被重试。

.. setting:: DUPEFILTER_CLASS

DUPEFILTER_CLASS
-------------------

默认值: ``'scrapy.dupefilters.RFPDupeFilter'``

该类被用来检测并过滤重复的请求。

默认的过滤器(``RFPDupeFilter``)基于使用 ``scrapy.utils.request.request_fingerprint`` 函数
的请求指纹。为了改变重复检测的方法，你可以继承 ``RFPDupeFilter`` 并重写它的 ``request_fingerprint`` 方法。
你怎么小心都不为过，因为你可能会陷入到循环爬取中。一个常用的好的建议是在 :class:`~scrapy.http.Request` 中
将 ``dont_filter`` 参数的值设置为 ``True`` 让该请求不被过滤。

.. setting:: DUPEFILTER_DUBUG

DUPEFILTER_DUBUG
-------------------

默认值: ``False``

默认情况下， ``RFPDupeFilter`` 只记录第一条重复的请求。将 :setting:`DUPEFILTER_DEBUG` 设置为 ``True`` 来记录所有的重复请求。

.. setting:: EDITOR

EDITOR
--------

默认值: ``vi`` (Unix系统环境) IDLE编辑器 (Windows系统环境)

通过 :command:`edit` 命令，可以使用该编辑器编辑spiders。
此外，如果已经设定了环境变量 ``EDITOR``，那么，使用 :command:`edit` 命令将不会调用默认编译器。

.. setting:: EXTENSIONS

EXTENSIONS
-----------

默认值:: ``{}``

一个包含了可以在你的项目中启用的扩展及其执行顺序的字典。

.. setting:: EXTENSIONS_BASE

EXTENSIONS_BASE
----------------

默认值 ::

    {
        'scrapy.extensions.corestats.CoreStats': 0,
        'scrapy.extensions.telnet.TelnetConsole': 0,
        'scrapy.extensions.memusage.MemoryUsage': 0,
        'scrapy.extensions.memdebug.MemoryDebugger': 0,
        'scrapy.extensions.closespider.CloseSpider': 0,
        'scrapy.extensions.feedexport.FeedExporter': 0,
        'scrapy.extensions.logstats.LogStats': 0,
        'scrapy.extensions.spiderstate.SpiderState': 0,
        'scrapy.extensions.throttle.AutoThrottle': 0,
    }

一个包含了在Scrapy中默认可用的扩展及其顺序的字典。该配置中包含了所有稳定的内置扩展。
注意，其中的一些需要在配置中启用。

更多信息 :ref:`extensions user guide <tpoics-extensions>` 以及 :ref:`list of available extensions <topics-extensions-ref>` 。


.. setting:: FEED_TEMPDIR

FEED_TEMPDIR
--------------

Feed Temp 目录允许你设定一个自定义的目录用来保存爬虫临时文件，用于接下来将文件上传至 :ref:`FTP feed storage <topics-feed-storage-ftp>` 和 :ref:`Amazon S3 <topics-feed-storage-s3>` 。

.. setting:: FTP_PASSIVE_MODE

FTP_PASSIVE_MODE
------------------

默认值: ``True``

初始化FTP传输时，是否启用被动模式。

.. setting:: FTP_PASSWORD

FTP_PASSWORD
--------------

默认值: ``"guest"``

当使用FTP连接并且在 ``Request`` 的meta中不存在 ``"ftp_password"`` 时触发使用。

.. note::
    参考 `RFC 1635`_ ，尽管通常使用"guest"作为密码或者邮箱地址登录匿名FTP，
    但有一些FTP服务器明确的需要用户的邮箱地址，并且不允许使用"guest"作为登录密码。

.. RFC 1635: https://tools.ietf.org/html/rfc1635

.. setting:: FTP_USER

FTP_USER
---------

默认值: ``"anonymous"``

当没有在 ``Request`` 的meta属性中设置 ``"ftp_user"`` 时，使用该配置的值作为FTP连接的用户名。

.. setting:: ITEM_PIPELINES

ITEM_PIPELINES
----------------

默认值: ``{}``

一个包含了 item pipelines及其顺序的字典。顺序值可以在 0-1000 内任意选择。值越小执行优先级越高。

例 ::

    ITEM_PIPELINES = {
       'mybot.pipelines.validate.ValidateMyItem': 300,
       'mybot.pipelines.validate.StoreMyItem': 800,
   }

.. setting:: ITEM_PIPELINES_BASE

ITEM_PIPELINES_BASE
---------------------

一个包含了在Scrapy中默认启用的pipelines的字典。你永远都不应该在你的项目中修改此配置，但你可以
通过修改 :setting: `ITEM_PIPELINES` 来实现相应的功能。

.. setting:: LOG_ENABLED

LOG_ENABLED
------------

默认值: ``True``

是否启用日志功能。

.. setting:: LOG_ENCODING

LOG_ENCODING
-------------

默认值: ``'utf-8'``

日志记录使用的编码格式。

.. setting:: LOG_FILE

LOG_FILE
----------

默认值: ``None``

日志输出的文件名。如果设置为 ``None``，将会使用标准错误流。

.. setting:: LOG_FORMAT

LOG_FORMAT
-----------

默认值: ``'%(asctime)s [%(name)s] %(levelname)s: %(message)s'``

日志消息的格式字符串。参考 `Python logging documentation`_ 获取更多信息。

.. _Python logging documentation: https://docs.python.org/2/library/logging.html#logrecord-attributes

.. setting:: LOG_DATEFORMAT

LOG_DATEFORMAT
---------------

默认值: ``'%Y-%m-%d %H:%M:%S'``

时间格式的字符串，可以作为 :setting:`LOG_FORMAT` 中 ``%(asctime)s`` 占位符的扩充。
参考 `Python datetime documentation`_ 获取更多信息。

.. _Python datetime documentation: https://docs.python.org/2/library/datetime.html#strftime-and-strptime-behavior

.. setting:: LOG_LEVEL

LOG_LEVEL
----------

默认值: ``'DEBUG'``

日志记录的最小等级。可用的等级有: CRITICAL，ERROR，WARNING，INFO，DEBUG。更多信息 :ref:`docs-topics-logging` 。

.. setting:: LOG_STDOUT

LOG_STDOUT
------------

默认值: ``False``

如果该值为 ``True`` ，程序中的所有标准输出(错误输出) 都会被重定向值日志中。比如，如果执行 ``print 'hell'`` 
打印值将会出现在Scrapy的日志中。

.. setting:: LOG_SHORT_NAMES

LOG_SHORT_NAMES
-----------------

默认值: ``False``

如果值为 ``True`` ，日志只会包含根路径。如果值为 ``False`` 则会显示负责日志输出的组件。

.. setting:: MEMDEBUG_ENABLED

MEMDEBUG_ENABLED
------------------

默认值: ``False``

是否启用内存调试。

.. setting:: MEMDEBUG_NOTIFY

MEMDEBUG_NOTIFY
-------------------

默认值: ``[]``

如果启用了内存调试，并且该值为空，则内存报告将会被发送到指定的路径，否则，报告将会被写入日志。

例 ::

    MEMDEBUG_NOTIFY = ['user@example.com']

.. setting:: MEMUSAGE_ENABLED

MEMUSAGE_ENABLED
-----------------

默认值: ``True``

作用域: ``scrapy.extensions.memusage``

是否启用内存使用扩展。该扩展会追踪此进程使用内存的峰值(将写入统计信息中)。作为可选项，
当Scrapy进程占用的内存超过限定时，可以关闭该进程(查看 :setting:`MEMUSAGE_LIMIT_MB`)，
并通过邮件通知(查看 :setting:`MEMUSAGE_NOTIFY_MAIL`)。

参考 :ref:`docs-topics-extensions-ref-memusage` 。

.. setting:: MEMUSAGE_LIMIT_MB

MEMUSAGE_LIMIT_MB
-------------------

默认值: ``0``

作用域: ``scrapy.extensions.memusage``

在Scrapy关闭前允许使用的最大内存(单位: MB)，需要设置MEMUSAGE_ENABLED为 ``True`` 。如果
值为0，将不会执行内存使用检测。

参考 :ref:`docs-topics-extensions-ref-memusage` 。

.. setting:: MEMUSAGE_CHECK_INTERVAL_SECONDS

MEMUSAGE_CHECK_INTERVAL_SECONDS
--------------------------------

.. versionadded:: 1.1

默认值: ``60.0``

作用域: ``scrapy.extensions.memusage``

:ref:`Memory usage extension <topics-extensions-ref-memusage>` 用来检测当前内存使用情况，
相对的， :setting:`MEMUSAGE_LIMIT_MB 与 :setting:`MEMUSAGE_WARNING_MB` 用来设置固定的时间间隔。

该配置用来设置间隔的时长(单位: 秒)。

参考 :ref:`docs-topics-extensions-ref-memusage` 。

.. setting:: MEMUSAGE_NOTIFY_MAIL

MEMUSAGE_NOTIFY_MAIL
---------------------

默认值: ``False``

作用域: ``scrapy.extensions.memusage``

当内存使用到达限制，需要通知的邮件列表。

例 ::

    MEMUSAGE_NOTIFY_MAIL = ['user@example.com']

参考 :ref:`docs-topics-extensions-ref-memusage` 。

MEMUSAGE_WARNING_MB
---------------------

默认值: ``0``

作用域: ``scrapy.extensions.memusage``

触发警告并发送邮件前允许使用的最大内存。如果值为0，不会产生任何警告。

.. setting:: NEWSPIDER_MODULE

NEWSPIDER_MODULE
------------------

默认值: ``''``

使用 :command:`genspider` 命令时创建新的spiders的位置。

例 ::

    NEWSPIDER_MODULE = 'mybot.spiders_dev'

.. setting:: RANDOMIZE_DOWNLOAD_DELAY

RANDOMIZE_DOWNLOAD_DELAY
-------------------------

默认值: ``True``

当启用时，Scrapy在获取相同网站的请求时将会等待的随机时间 (在 0.5 * :setting:`DOWNLOAD_DELAY` 与 1.5 * :setting:`DOWNLOAD_DELAY` 之间)。

进行随机时间延迟选择将会降低爬虫被网站检测到的几率。

该随机选择策略与 `wget`_ 的 ``--random-wait`` 的选项策略相同。

如果 :setting:`DOWNLOAD_DELAY` 的值为0(默认)，该选项不会产生任何作用。

.. _wget: https://www.gnu.org/software/wget/manual/wget.html

.. setting:: REACTOR_THREADPOOL_MAXSIZE

REACTOR_THREADPOOL_MAXSIZE
---------------------------

默认值: ``10``

Twisted Reactor 线程池的最大容量。它是一个被大量Scrapy组件所使用的通用多目的线程池。
线程DNS解析器，BlockingFeedStorage， S3FilesStore仅仅是一小部分。如果你正在遭受
阻塞IO的效率问题，你可以适当的增加该值。

.. setting:: REDIRECT_MAX_TIMES

REDIRECT_MAX_TIMES
--------------------

默认值: ``20``

定义一个请求可以被重定向的最大次数。当超过最大值后，返回该请求目前对应的响应。我们使用Firefox的默认值来应对相同的任务。

.. setting:: REDIRECT_PRIORITY_ADJUST

REDIRECT_PRIORITY_ADJUST
--------------------------

默认值: ``+2``

作用域: ``scrapy.downloadermiddlewares.redirect.RedirectMiddleware``

调整与原请求相关的重定向请求的优先级:

- **一个正数优先级调整意味着更高的优先级。**
- 一个负数优先级调整，意味着更低的优先级。

.. setting:: RETRY_PRIORITY_ADJUST

RETRY_PRIORITY_ADJUST
----------------------

默认值: ``-1``

作用域: ``scrapy.downloadermiddlewares.retry.RetryMiddleware``

调整与原请求相关的重试请求的优先级:

- 一个正数优先级调整意味着更高的优先级。
- **一个负数优先级调整(默认)意味着更低的优先级。**

.. setting:: ROBOTSTXT_OBEY

ROBOTSTXT_OBEY
---------------

默认值: ``False``

作用域: ``scrapy.downloadermiddlewares.robotstxt``

如果启用该配置，Scrapy将会遵守robots.txt政策。更多信息 :ref:`docs-topics-dlmw-robots` 。

.. note::

    由于历史原因，该值被默认设置为 ``False`` ，在使用 ``scrapy startproject`` 命令生成
    settings.py文件时，该组件被默认启用。

.. setting:: SCHEDULER

SCHEDULER
-----------

默认值: ``'scrapy.core.scheduler.Scheduler'``

爬虫的调度程序。

.. setting:: SCHEDULER_DUBUG

SCHEDULER_DUBUG
-----------------

默认值: ``False``

将该值设置为 ``True`` 将会记录请求调度的调试信息。如果请求不能序列化至磁盘，则只记录一次当前的日志信息(仅此一次)。
统计计数器( ``scheduler/unserializable`` ) 会追踪该问题的发生次数。

例 ::

    1956-01-31 00:00:00+0800 [scrapy.core.scheduler] ERROR: Unable to serialize request:
    <GET http://example.com> - reason: cannot serialize <Request at 0x9a7c7ec>
    (type Request)> - no more unserializable requests will be logged
    (see 'scheduler/unserializable' stats counter)

.. setting:: SCHEDULER_DISK_QUEUE

SCHEDULER_DISK_QUEUE
----------------------

默认值: ``'scrapy.squeues.PickleLifoDiskQueue'``

调度器将会使用的磁盘队列(disk queue)类型。其他可用的类型有: ``scrapy.squeues.PickleFifoDiskQueue`` ，
``scrapy.squeues.MarshalFifoDiskQueue``, ``scrapy.squeues.MarshalLifoDiskQueue`` 。

.. setting:: SCHEDULER_MEMORY_QUEUE

SCHEDULER_MEMORY_QUEUE
-------------------------

默认值: ``'scrapy.squeues.LifoMemoryQueue'``

调度器使用的内存队列(in-memory queue)类型。其他可用类型: ``scrapy.squeues.FifoMemoryQueue`` 。

.. setting:: SCHEDULER_PRIORITY_QUEUE

SCHEDULER_PRIORITY_QUEUE
--------------------------

默认值: ``'queuelib.PriorityQueue'``

调度器使用的优先级队列的类型。

.. setting:: SPIDER_CONTRACTS

SPIDER_CONTRACTS
------------------

默认值: ``{}``

一个包含了可以在你的项目中启用的spider条款(spider contracts)的字典，可以使用该配置对spiders进行测试。
更多信息 :ref:`docs-topics-contracts` 。

.. setting:: SPIDER_CONTRACTS_BASE

SPIDER_CONTRACTS_BASE
-----------------------

默认值 ::

    {
        'scrapy.contracts.default.UrlContract' : 1,
        'scrapy.contracts.default.ReturnsContract': 2,
        'scrapy.contracts.default.ScrapesContract': 3,
    }

一个包含了在Scrapy中默认启用的scrapy条款。你永远都不要修改它，但你可以通过修改 :setting:`SPIDER_CONTRACTS`
 实现相应的功能。 更多信息 :ref:`docs-topics-contracts`

你可与通过将 :setting:`SPIDER_CONTRACTS` 中对应的条款值设置为 ``None`` 来禁用对应条款。
比如说禁用内置的 ``ScrapesContract`` ::

    SPIDER_CONTRACTS = {
        'scrapy.contracts.default.ScrapesContract': None,
    }

.. setting:: SPIDER_LOADER_CLASS

SPIDER_LOADER_CLASS
---------------------

默认值: ``'scrapy.spiderloader.SpiderLoader'``

该类被用来加载spiders，且必须执行 :ref:`topics-api-spiderloader` 。

.. setting:: SPIDER_LOADER_WARN_ONLY

SPIDER_LOADER_WARN_ONLY
-------------------------

.. versionadded:: 1.3.3

默认值: ``False``

默认情况下，当scrapy尝试从 :setting:`SPIDER_MODULES` 中导入spider类时，如果出现任何 ``ImportError`` 异常，
程序将显示一个明显的错误。但是，你可以将 ``SPIDER_LOADER_WARN_ONLY`` 的值设置为 ``True`` 来隐藏该异常并发出一个简单的
警告。

.. note::
    一些 :ref:`scrapy commands <docs-topics-command>` 运行时，已经将该配置的值设置为 ``True`` (即它们只会发出警告，并不会产生错误)
    所以，它们不要加载spider类:
    :command:`scrapy runspider <runspider>`,
    :command:`scrapy settings <settings>`,
    :command:`scrapy startproject <startproject>`,
    :command:`scrapy version <version>`.

.. setting:: SPIDER_MIDDLEWARES

SPIDER_MIDDLEWARES
----------------------

默认值: ``{}``

一个包含了可以在你的项目中启用的spider中间件及其顺序的字典。更多信息 :ref:`docs-topics-spider-middleware-setting` 。

.. setting:: SPIDER_MIDDLEWARES_BASE

SPIDER_MIDDLEWARES_BASE
-------------------------

默认值 ::

    {
        'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
        'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
        'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
        'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
        'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
    }

一个包含了在Scrapy中默认启用的spider中间件及其顺序的字典。低顺序的接近scrapy引擎，高顺序的接近spider。
更多信息 :ref:`docs-topics-spider-middleware-setting` 。

.. setting:: SPIDER_MODULES

SPIDER_MODULES
----------------

默认值: ``[]``

一个包含了Scrapy将要查询的spider列表。

例 ::

    SPIDER_MODULES = ['mybot.spiders_prod', 'mybot.spiders_dev']

.. setting:: STATS_CLASS

STATS_CLASS
-------------

默认值: ``'scrapy.statscollectors.MemoryStatsCollector'``

用来收集统计信息的类，必须实现 :ref:`docs-topics-api-stats` 。

.. setting:: STATS_DUMP

STATS_DUMP
-----------

默认值: ``True``

当spider结束时，用来转移 :ref:`Scrapy stats <docs-topics-stats>` 到日志中。

更多信息 :ref:`docs-topics-stats` 。

.. setting:: STATSMAILER_RCPTS

STATSMAILER_RCPTS
-------------------

默认值: ``[]``

当爬取结束时，用来发送统计信息。 更多信息 :class:`~scrapy.extensions.statsmailer.StatsMailer` 。

.. setting:: TELNETCONSOLE_ENABLED

TELNETCONSOLE_ENABLED
-----------------------

默认值: ``True``

是否启用 :ref:`telnet console <docs-topics-telnetconsole>` 。

.. setting:: TELNETCONSOLE_PORT

TELNETCONSOLE_PORT
--------------------

默认值: ``[6023, 6073]``

远程控制使用的端口范围。如果设置为 ``None`` 或 ``0`` ，将会动态的分配一个端口。 更多信息 :ref:`topics-telnetconsole` 。

.. setting:: TEMPLATES_DIR

TEMPLATES_DIR
--------------

默认值: scrapy模块中的 ``templates`` 文件

当使用 :command:`startproject` 命令创建新项目或使用 :command:`genspider` 命令创建新的spider时查找的模板路径。

项目的名称不能与用户文件名或者是项目中的子文件冲突。


.. setting:: URLLENGTH_LIMIT

URLLENGTH_LIMIT
----------------

默认值: ``2083``

作用域: ``spidermiddlewares.urllength``

允许爬取的URL的最长长度。更多信息: https://boutell.com/newfaq/misc/urllength.html

.. setting:: USER_AGENT

USER_AGENT
-----------

默认值: ``"Scrapy/VERSION (+https://scrapy.org)"``

爬取时的默认用户代理。

Settings documented elsehwhere: 
--------------------------------

接下来的配置可以被记录在其他地方，查看更多信息:

.. settingslist::

.. _Amazon web services: https://aws.amazon.com/
.. _breadth-first order: https://en.wikipedia.org/wiki/Breadth-first_search
.. _depth-first order: https://en.wikipedia.org/wiki/Depth-first_search

