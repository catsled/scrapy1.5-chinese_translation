.. _docs-topics-spiders:

=======
Spiders
=======

Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页
的内容中提取结构化数据(爬取item)。 换句话说，Spider就是您定义爬取的动作及分析某个网页(或
者是有些网页)的地方。

对spider来说，爬取循环做着下面的事:

1. 首先通过初始化 requests 抓取第一个 URL，并设置回调函数。 当该 requests 下载完毕并返回时，
   将生成 response ，然后作为参数传给该回调函数。

   初始的 requests 是通过调用 :meth:`~scrapy.spiders.Spider.start_requests` 方法(默认情况下)为
   :attr:`~scrapy.spiders.Spider.start_urls` 中的URL生成初始 :meth:`~scrapy.spiders.Spider.start_requests` 请求
   并且设置 :attr:`~scrapy.spiders.Spider.parse` 方法作为请求的回调函数。

2. 在回调函数中，您将解析 Response（网页）并返回带有提取后的数据的 dict ，:class:`~scrapy.item.Item` 对象，:class:`~scrapy.http.Request` 对象
   或这些对象的可迭代容器。这些请求还将包含回调（可能是相同的），然后由 Scrapy 下载，然后由指定
   的回调处理它们的响应。

3. 在回调函数中，您通常使用 :ref:`docs-topics-selectors` 来解析页面内容（但您也可以使用BeautifulSoup，lxml或您喜欢的任何解析器），
   并使用解析的数据生成 Item。

4. 最后，由spider返回的 item 通常会持久化存储到数据库中（在某些 :ref:`Item Pipeline <topics-item-pipeline>` 中进行存储操作）或者
   使用 :ref:`topics-feed-exports` 写入到文件中。
   
虽然这个循环（或多或少）适用于任何种类的 spider，Scrapy 实现了不同种类的默认 spider 用于不同的需求。
我们将在这里谈论这些类型。

.. module:: scrapy.spiders
   :synopsis: Spiders base class, spider manager and spider middleware

.. _topics-spiders-ref:

scrapy.Spider
=============

.. class:: Spider()

   这是最简单的spider，其他的spider必须继承该类（包括 Scrapy 自带的一些爬虫，以及你自己写的爬虫）。它不提供
   任何特殊功能。它只是提供了一个默认的 :meth:`start_requests` 方法实现，它读取并请求爬虫的 :attr:`start_urls` 属性，并为
   每个结果响应调用爬虫的 ``parse`` 方法。

   .. attribute:: name

       定义spider名字的字符串(string)。name定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。 
       不过您可以生成多个相同的spider实例(instance)，这没有任何限制。 name是spider最重要的属性，而且是必须的。

       如果spider抓取单个网站（single domain），一个常见的做法是以该网站(domain)(加或不加 `TLD`_ )来命名spider。
       例如，如果爬取``mywebsite.com`` ，该spider通常会被命名为 ``mywebsite`` 。

       .. note:: 在Python 2中，``name`` 必须是 ASCII 格式。

   .. attribute:: allowed_domains

       可选项。包含了spider允许抓取的域名列表。当 :class:`~scrapy.spidermiddlewares.offsite.OffsiteMiddleware` 启用时，不在域名列表中的URL不会被请求。
       
       比如您的目标网址是 ``https://www.example.com/1.html`` ，然后将 ``'example.com'`` 添加到列表中。
      
   .. attribute:: start_urls

       URL列表。当没有指定特定 URL 时，爬虫将从从该列表中开始抓取。因此，爬取的第一个页面将是这里列出的某个URL。
       后续的 URL 将根据从起始 URL 中获得的数据连续生成。

   .. attribute:: custom_settings

      该设置是一个dict。运行spider时改设置会覆盖项目级的设置。因为设置在实例化（instantiation）之前更新，所以它必须定义为类属性。

      有关内置可用设置的列表，请参阅：:ref:`topics-settings-ref`。

   .. attribute:: crawler

      该属性在初始化类之后由 :meth:`from_crawler` 类方法设置，并链接到该spider实例绑定的 :class:`~scrapy.crawler.Crawler` 对象上。

      Crawlers 在项目中封装了很多组件，作为单一入口访问（例如扩展，中间件，信号管理器等）。有关详情，请参阅 :ref:`topics-api-crawler`。

   .. attribute:: settings

      spider 运行时的配置。这是一个 :class:`~scrapy.settings.Settings` 实例，详细了解请参考 :ref:`topics-settings` 。
      :class:`~scrapy.settings.Settings` instance, see the
      :ref:`topics-settings` topic for a detailed introduction on this subject.

   .. attribute:: logger

      Python Logger 使用 Spider 的 :attr:`name` 创建。您可以通过它发送日志消息，如 :ref:`topics-logging-from-spiders`。
      
   .. method:: from_crawler(crawler, \*args, \**kwargs)

       这是 Scrapy 用来创建 spider 的类方法。

       您可能不需要直接重写它，因为默认实现充当 :meth:`__init__` 方法的代理，用给定的参数`args`和命名参数`kwargs`来调用它。

       尽管如此，此方法会在新实例中设置 :attr:`crawler` 和 :attr:`settings` 属性，以便稍后在spider代码中访问它们。

       :param crawler: spider将绑定到crawler
       :type crawler: :class:`~scrapy.crawler.Crawler` 实例

       :param args: 传递给 :meth:`__init__` 方法的参数
       :type args: list

       :param kwargs: 传递给 :meth:`__init__` 方法的关键字参数
       :type kwargs: dict

   .. method:: start_requests()

       此方法必须返回一个可迭代对象（iterable），该对象包含了 spider 用于爬取的第一个Request。
       当 spider 启动时，Scrapy 会调用该方法。此方法仅被调用一次，因此将 :meth:`start_requests` 作为生成器实现是安全的。
       
       默认实现为:attr:`start_urls`中的每个url生成``Request(url, dont_filter=True)``。


       如果您想要修改最初爬取某个域名的Request对象，您可以重写(override)该方法。
       例如，如果您需要在启动时使用POST请求登录，您可以::

           class MySpider(scrapy.Spider):
               name = 'myspider'

               def start_requests(self):
                   return [scrapy.FormRequest("http://www.example.com/login",
                                              formdata={'user': 'john', 'pass': 'secret'},
                                              callback=self.logged_in)]

               def logged_in(self, response):
                   # here you would extract links to follow and return Requests for
                   # each of them, with another callback
                   pass

   .. method:: parse(response)

       当 requests 请求没有指定回调函数时，这是Scrapy用来处理下载后的response的默认方法。

       ``parse`` 方法负责处理 response 并返回所抓取的数据以及(或者)后续的URL。:class:`Spider` 对其他的
       Request的回调函数也有相同的要求。

       此方法以及任何其他Request的回调函数必须返回一个可迭代的 :class:`~scrapy.http.Request` 或 dict 或 :class:`~scrapy.item.Item` 对象。

       :param response: 将要解析的 response 对象
       :type response: :class:`~scrapy.http.Response`

   .. method:: log(message, [level, component])

       通过 Spider 的 :attr:`logger` 发送日志消息，保留向后兼容性。有关详细信息，请参阅
       :ref:`topics-logging-from-spiders`.

   .. method:: closed(reason)

       当spider关闭时，该函数被调用。 该方法提供了一个替代调用signals.connect()来监听 :signal:`spider_closed` 信号的快捷方式。

让我们看个例子::

    import scrapy


    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            self.logger.info('A response from %s just arrived!', response.url)

在单个回调函数中返回多个Request以及Item的例子::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = [
            'http://www.example.com/1.html',
            'http://www.example.com/2.html',
            'http://www.example.com/3.html',
        ]

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield {"title": h3}

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

除了 :attr:`~.start_urls` ，你也可以直接使用 :meth:`~.start_requests` ; 您也可以使用 :ref:`topics-items` 
来给予数据更多的结构性(give data more structure)::

    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)

.. _spiderargs:

Spider arguments
================

Spider 可以接收参数来修改其行为。spider 参数的一些常见用法是定义起始URL或限制要爬取一部分网站，
但实际上它们可以配置 spider 的任何功能。

Spider参数使用 ``-a`` 选项通过 :command:`crawl` 命令传递。例如::

    scrapy crawl myspider -a category=electronics

Spiders 可以通过 `__init__` 方法访问参数::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def __init__(self, category=None, *args, **kwargs):
            super(MySpider, self).__init__(*args, **kwargs)
            self.start_urls = ['http://www.example.com/categories/%s' % category]
            # ...

默认的 `__init__` 方法将获得所有的 spider 参数，并将参数作为 spider 的属性。
上面的例子也可以写作如下::

    import scrapy

    class MySpider(scrapy.Spider):
        name = 'myspider'

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/categories/%s' % self.category)

请记住，spider的参数只能是字符串。spider自己不会做任何解析。
如果打算通过命令行设置 `start_urls` 属性,你必须使用类似`ast.literal_eval <https://docs.python.org/library/ast.html#ast.literal_eval>`_
或`json.loads <https://docs.python.org/library/json.html#json.loads>`_ 的方式将它解析到列表中，然后将其设置为属性。
否则，你会迭代一个 start_urls 字符串（一个非常常见的python陷阱），导致每个字符被看作一个单独的url。 

一个有用的例子是通过 :class:`~scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware` 设置 http auth 证书或
通过 :class:`~scrapy.downloadermiddlewares.useragent.UserAgentMiddleware` 设置 user agent::

    scrapy crawl myspider -a http_user=myuser -a http_pass=mypassword -a user_agent=mybot

Spider 参数也可以通过 Scrapyd ``schedule.json`` API 传递。 
查看 `Scrapyd documentation`_.

.. _builtin-spiders:

Generic Spiders
===============

Scrapy 自带一些有用的通用爬虫，你可以将自己的spider作为它们的子类。他们的目的是为一些常见的抓取案例提供方便的功能，
例如根据某些规则跟踪网站上的所有链接，从 `Sitemaps`_ 抓取或解析XML / CSV Feed。

对于在下面的爬虫中使用的示例，我们假设你有一个项目，在 ``myproject.items`` 模块中声明一个 ``TestItem``::

    import scrapy

    class TestItem(scrapy.Item):
        id = scrapy.Field()
        name = scrapy.Field()
        description = scrapy.Field()


.. currentmodule:: scrapy.spiders

CrawlSpider
-----------

.. class:: CrawlSpider

   这是最常用的爬取常规网站的spider，它提供了一个方便的机制，通过定义一组 rules 来跟踪链接。
   它可能不是完全适合您的特定网站或项目，但它有几种通用例子，因此您可以以此为起点，
   根据需要重写更多的自定义功能，当然也可以实现自己的spider。

   除了从 Spider 继承的属性外（你必须指定），这个类还支持一个新的属性:

   .. attribute:: rules

       它是一个(或多个) :class:`Rule` 对象的 List 列表. 每个 :class:`Rule` 定义了爬取网址的特定行为。
       Rule 对象的描述如下。如果多个 rules 匹配相同的链接，则会根据它在属性中定义的顺序使用第一个规则。

   该spider也提供了一个可重写的方法:

   .. method:: parse_start_url(response)

      这个方法是start_urls的响应。它允许解析初始响应，并且必须得返回一个 :class:`~scrapy.item.Item` 对象，
      一个:class:`~scrapy.http.Request` 对象或者一个包含其中任何一个对象的迭代器。

Crawling rules
~~~~~~~~~~~~~~

.. class:: Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)

   ``link_extractor`` 是一个链接提取器 :ref:`Link Extractor <topics-link-extractors>` 对象，定义了如何从爬取的页面提取链接。

   ``callback`` 是一个 callable 或 string (此时，该 spider 中同名的函数将会被调用)，link_extractor 从 Response 对象中提取的
   每个链接都会调用该函数。该回调函数接收一个 response 对象作为它的第一个参数，并且必须返回一个包含 :class:`~scrapy.item.Item` 
   及（或）:class:`~scrapy.http.Request` 对象（或它们的任何子类）的列表。

   .. warning:: 当编写爬虫规则时，避免使用 ``parse`` 作为回调，因为 :class:`CrawlSpider` 本身使用 ``parse`` 方法来实现其逻辑。
       因此，如果您重写 ``parse`` 方法，crawl spider 将会运行失败。
    

   ``cb_kwargs`` 包含传递给回调函数的参数(keyword argument)的字典

   ``follow`` 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。 如果 ``callback`` 为None， ``follow`` 默认设置为 ``True`` ，
   否则默认为 ``False`` 。

   ``process_links`` 是一个 callable 或 string（此时，该spider中同名的函数将会被调用），使用 ``link_extractor`` 从 Response 对象中
   提取的每个链接列表调用它。主要用来过滤链接。

   ``process_request`` 是一个 callable 或 string（此时，该spider中同名的函数将会被调用），它将被此规则提取的每个 request 调用，并且必须
   返回一个 request 或 None（过滤掉该request）。

CrawlSpider example
~~~~~~~~~~~~~~~~~~~

现在让我们来看看一个带有 rule 的 CrawlSpider 示例::

    import scrapy
    from scrapy.spiders import CrawlSpider, Rule
    from scrapy.linkextractors import LinkExtractor

    class MySpider(CrawlSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com']

        rules = (
            # Extract links matching 'category.php' (but not matching 'subsection.php')
            # and follow links from them (since no callback means follow=True by default).
            Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

            # Extract links matching 'item.php' and parse them with the spider's method parse_item
            Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
        )

        def parse_item(self, response):
            self.logger.info('Hi, this is an item page! %s', response.url)
            item = scrapy.Item()
            item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
            item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
            item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
            return item


该spider将从 example.com 的首页开始爬取，获取category以及item的链接，后者使用 ``parse_item`` 方法解析响应。在 parse_item 方法中，
每个 response 将会被XPath处理，从HTML解析成数据并填入 :class:`~scrapy.item.Item` 中。

XMLFeedSpider
-------------

.. class:: XMLFeedSpider

    XMLFeedSpider被设计用于通过迭代各个节点来分析XML源(XML feed)。迭代器可以从: ``iternodes``, ``xml``,
    和 ``html`` 选择。出于性能的考虑，建议使用 iternodes 迭代器，因为``xml`` 和 ``html``迭代器会一次生成整个DOM，然后在解析它。
    但是，使用 ``html`` 作为迭代器可以有效应对有错误标记的XML。

    要设置 iterator 迭代器和 tag name 标签名称，您必须定义以下类属性:

    .. attribute:: iterator

        定义要使用的迭代器的string。可选项有:

           - ``'iternodes'`` - 一个高性能的基于正则表达式的迭代器

           - ``'html'`` - 使用 :class:`~scrapy.selector.Selector` 的迭代器。需要注意的是该迭代器使用DOM进行分析，
             其需要将所有的DOM载入内存，当数据量很大的时候会产生问题

           - ``'xml'`` - 使用 :class:`~scrapy.selector.Selector` 的迭代器。需要注意的是该迭代器使用DOM进行分析，
             其需要将所有的DOM载入内存，当数据量很大的时候会产生问题

        它的默认值: ``'iternodes'``.

    .. attribute:: itertag

        一个包含开始迭代的节点名的string。例如::

            itertag = 'product'

    .. attribute:: namespaces

        一个由 ``(prefix, uri)`` 元组(tuple)所组成的list。 其定义了在该文档中会被spider处理的可用的namespace。 
        ``prefix`` 及 ``uri`` 会被自动调用 :meth:`~scrapy.selector.Selector.register_namespace` 生成namespace。

        您可以通过在 :attr:`itertag` 属性中制定节点的namespace。

        例如::

            class YourSpider(XMLFeedSpider):

                namespaces = [('n', 'http://www.sitemaps.org/schemas/sitemap/0.9')]
                itertag = 'n:url'
                # ...

    除了这些新的属性之外，该spider也有以下可以覆盖(overrideable)的方法:

    .. method:: adapt_response(response)

        该方法在spider的middleware(中间件)收到response时调用。您可以在 response 被解析之前使用该函数来修改响应的内容。 
        该方法接受一个response并返回一个response(可以相同也可以不同)。

    .. method:: parse_node(response, selector)

        当节点符合提供的标签名时(``itertag``)该方法被调用。该方法接收 response 以及相应的 :class:`~scrapy.selector.Selector` 作为参数。重写此方法是强制性的，
        否则，你的spider将无法工作。该方法返回一个 :class:`~scrapy.item.Item` 对象或者 :class:`~scrapy.http.Request` 对象 或者一个包含二者的可迭代对象(iterable)。

    .. method:: process_results(response, results)
  
        当spider返回结果(item或request)时该方法被调用。 设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理， 例如设定item的ID。
        其接受一个结果的列表(list of results)及对应的response。 其结果必须返回一个结果的列表(list of results)(包含Item或者Request对象)。

XMLFeedSpider example
~~~~~~~~~~~~~~~~~~~~~

很容易使用的spider。看个例子::

    from scrapy.spiders import XMLFeedSpider
    from myproject.items import TestItem

    class MySpider(XMLFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.xml']
        iterator = 'iternodes'  # This is actually unnecessary, since it's the default value
        itertag = 'item'

        def parse_node(self, response, node):
            self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

            item = TestItem()
            item['id'] = node.xpath('@id').extract()
            item['name'] = node.xpath('name').extract()
            item['description'] = node.xpath('description').extract()
            return item

简单来说，我们做的就是创建一个spider，从给定的``start_urls``下载一个 feed.xml 文件，然后遍历每个``item``标签，
打印出来，并将一些随机数据存储在一个 :class:`~scrapy.item.Item` 中。

CSVFeedSpider
-------------

.. class:: CSVFeedSpider

   该spider和XMLFeedSpider十分类似, 不同在于这个spider是按行遍历而不是节点遍历。每次迭代调用的方法是 :meth:`parse_row`.

   .. attribute:: delimiter

       在CSV文件中用于区分字段的分隔符。类型为string。 默认为 ``','`` (逗号)。

   .. attribute:: quotechar

       在CSV文件中每个字段的外围字符的字符串默认是 ``'"'`` (引号)。

   .. attribute:: headers

       CSV文件中列名称的列表。

   .. method:: parse_row(response, row)

       该方法接收一个response对象及一个以提供或检测出来的header为键的字典(代表每行)。 该spider中，您也可以
       重写 ``adapt_response`` 及 ``process_results`` 方法来进行预处理(pre-processing)及后处理(post-processing)。

CSVFeedSpider 例子
~~~~~~~~~~~~~~~~~~~~~

下面的例子和之前的例子很像，但使用了
:class:`CSVFeedSpider`::

    from scrapy.spiders import CSVFeedSpider
    from myproject.items import TestItem

    class MySpider(CSVFeedSpider):
        name = 'example.com'
        allowed_domains = ['example.com']
        start_urls = ['http://www.example.com/feed.csv']
        delimiter = ';'
        quotechar = "'"
        headers = ['id', 'name', 'description']

        def parse_row(self, response, row):
            self.logger.info('Hi, this is a row!: %r', row)

            item = TestItem()
            item['id'] = row['id']
            item['name'] = row['name']
            item['description'] = row['description']
            return item


SitemapSpider
-------------

.. class:: SitemapSpider

    SitemapSpider使您爬取网站时可以通过 Sitemaps 来发现爬取的URL。其支持嵌套的sitemap，并能从robots.txt中获取 `Sitemaps`_ 的url。

    .. attribute:: sitemap_urls

        包含您要爬取的url的sitemap的url列表(list)。 
        您也可以指定为一个 robots.txt ，spider会从中分析并提取url。

    .. attribute:: sitemap_rules

        一个包含 ``(regex, callback)`` 元组的 list 列表:

        * ``regex`` 是一个用于匹配从sitemap提取的url的正则表达式
          ``regex`` 可以是一个字符串或者编译过的正则对象

        * callback指定了匹配正则表达式的url的处理函数。 ``callback`` 可以是一个字符串(spider中的方法名)或者是callable。

        例如::

            sitemap_rules = [('/product/', 'parse_product')]

        规则按顺序进行匹配，之后第一个匹配才会被应用。

        如果您忽略该属性，sitemap中发现的所有url将会被 ``parse`` 函数处理。

    .. attribute:: sitemap_follow

        一个用于匹配要跟进的sitemap的正则表达式的列表(list)。这仅适用于使用 `Sitemap index files`_ 来指向其他sitemap文件的站点。  

       默认情况下，所有的sitemap都会被跟进。

    .. attribute:: sitemap_alternate_links

        指定是否应该跟进一个url的可选链接。有些网站会在一个 url 块内提供其他语言的网站链接。

        例如::

            <url>
                <loc>http://example.com/</loc>
                <xhtml:link rel="alternate" hreflang="de" href="http://example.com/de"/>
            </url>

        当 ``sitemap_alternate_links`` 被启用时, 两个URL都会被获取。当``sitemap_alternate_links`` 关闭时, 只有 ``http://example.com/`` 会被获取

        ``sitemap_alternate_links`` 默认是关闭的。


SitemapSpider examples
~~~~~~~~~~~~~~~~~~~~~~

简单的例子: 使用 ``parse`` 回调函数处理通过sitemap发现的所有url::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']

        def parse(self, response):
            pass # ... scrape item here ...

针对不同的 URL 使用不同的callback回调函数::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/sitemap.xml']
        sitemap_rules = [
            ('/product/', 'parse_product'),
            ('/category/', 'parse_category'),
        ]

        def parse_product(self, response):
            pass # ... scrape product ...

        def parse_category(self, response):
            pass # ... scrape category ...

跟进 `robots.txt`_ 文件定义的sitemap并只跟进包含有 ``/sitemap_shop`` 的url::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]
        sitemap_follow = ['/sitemap_shops']

        def parse_shop(self, response):
            pass # ... scrape shop here ...

将SitemapSpider与其他URL来源相结合::

    from scrapy.spiders import SitemapSpider

    class MySpider(SitemapSpider):
        sitemap_urls = ['http://www.example.com/robots.txt']
        sitemap_rules = [
            ('/shop/', 'parse_shop'),
        ]

        other_urls = ['http://www.example.com/about']

        def start_requests(self):
            requests = list(super(MySpider, self).start_requests())
            requests += [scrapy.Request(x, self.parse_other) for x in self.other_urls]
            return requests

        def parse_shop(self, response):
            pass # ... scrape shop here ...

        def parse_other(self, response):
            pass # ... scrape other here ...

.. _Sitemaps: https://www.sitemaps.org/index.html
.. _Sitemap index files: https://www.sitemaps.org/protocol.html#index
.. _robots.txt: http://www.robotstxt.org/
.. _TLD: https://en.wikipedia.org/wiki/Top-level_domain
.. _Scrapyd documentation: https://scrapyd.readthedocs.io/en/latest/
