.. docs-intro-scrapy-tutorial:

===========
Scrapy 教程
===========

在本节教程中，我们认为你已经成功的安装了scrapy. 如果你还没有安装scrapy,请查看安装教程 :ref:`docs-intro-scrapy-install` 。

接下来，我们将要爬取 quotes.toscrape.com，该网站列出了一些名人名言。

跟着教程熟悉接下来的任务:

1. 创建一个新的Scrapy项目
2. 编写一个spider类进行网站爬取和提取数据
3. 使用命令行导出爬取到的数据
4. 使用follow links将爬虫改为可递归的
5. 使用spider参数

Scrapy完全使用python编写.如果你并不熟悉python编程语言，没关系，除了Scrapy之外你可以从很多地方了解到这门语言。

如果你有其他语言的编程经验，并且想要快速学习python，我们推荐你阅读一下Dive Into Python 3. 当然，我们也推荐Python Turorial。

如果你没有编程经验，并且想要以python开始你的编程生涯，这本Learn Python The Hard Way对你来说应该用处很大.这里列出了一些对你或许有用的Python学习资源。


创建一个项目
====================

在爬取数据之前，你应该先建立一个新的Scrapy项目.输入一个你喜欢的目录名称，接下来你的代码将放在这个目录下面::

    scrapy startproject tutorial

执行这个命令,你将会得到一个名为 ``tutorial`` 的目录，具体结构如下::

     tutorial/
        scrapy.cfg            # deploy configuration file

        tutorial/             # project's Python module, you'll import your code from here
            __init__.py

            items.py          # project items definition file
            
            middlewares.py    # project middlewares file

            pipelines.py      # project pipelines file

            settings.py       # project settings file

            spiders/          # a directory where you'll later put your spiders
                __init__.py


第一个Spider
====================

接下来，我们会定义一个Spider类，`Scrapy` 将会使用这个类去爬取网站上的信息.这个类必须继承自 ``scrapy.Spider`` 同时，
你还需要定义一些其他的方法，比如说，创建初始化的请求，你也可以选择如何去跟进这个网页的其他链接，以及如何从已下载的网页中解析提取相应的数据。

这是我们的第一个Spider代码.将这些代码保存至 ``tutorial/spiders`` 下的 ``quotes_spider.py`` 文件中::

    import scrapy


    class QuotesSpider(scrapy.Spider):
        name = "quotes"

        def start_requests(self):
            urls = [
                'http://quotes.toscrape.com/page/1/',
                'http://quotes.toscrape.com/page/2/',
            ]
            for url in urls:
                yield scrapy.Request(url=url, callback=self.parse)

        def parse(self, response):
            page = response.url.split("/")[-2]
            filename = 'quotes-%s.html' % page
            with open(filename, 'wb') as f:
                f.write(response.body)
            self.log('Saved file %s' % filename)

如你所见，我们的Spider继承了 ``scrapy.Spider`` 同时还定义了一些属性和方法:

* name: 该属性是用来唯一标识一个Spider的，所以你不能在不同的Spider中使用相同的name.

* start_requests(): 该方法必须返回一个可迭代的Requests（你可以返回一个列表或者编写一个生成器函数），保证一连串的请求将会被这个方法成功的创建，随后，``Spider`` 将会从这里开始他的爬虫之旅.

* parse(): 


追踪链接
========

并不是只抓取网站 http://quotes.toscrape.com 的第一二页，你一定想要抓取网站中的所有页面。

知道了怎么从页面中提取数据，现在，我们看一下怎么从提取的数据中追踪链接。

首先就是要从页面中提取你想要追踪的链接。查看页面代码可以从类似于下面这种标记中的代码中找到链接到下一页的 URL 。::

    <ul class="pager">
    	<li class="next">
        	<a href="/page/2/">Next <span aria-hidden="true">&rarr;</span></a>
    	</li>
	</ul>


在 shell 中提取链接::


	>>> response.css('li.next a').extract_first()
	'<a href="/page/2/">Next <span aria-hidden="true">→</span></a>'


此命令会得到一个 html 元素，但是我们需要得到它的属性`href`的值。很幸运，Scrapy 刚好支持 CSS 扩展，使用 CSS 扩展可以获取元素的属性值，如下：::


	>>>response.css('li.next a::attr(href)').extract_first()
	'/page/2/'"


现在，爬虫可以通过追踪链接递归抓取每个页面的数据：::

	import scrapy
	class QuotesSpider(scrapy.Spider):
    		name = "quotes"
    		start_urls = [
        	'http://quotes.toscrape.com/page/1/',
    		]
    		def parse(self, response):
        		for quote in response.css('div.quote'):
            			yield {
					'text': quote.css('span.text::text').extract_first(),
					'author': quote.css('small.author::text').extract_first(),
					'tags': quote.css('div.tags a.tag::text').extract(),
				    }
			next_page = response.css('li.next a::attr(href)').extract_first()
			if next_page is not None:
			    next_page = response.urljoin(next_page)
			    yield scrapy.Request(next_page, callback=self.parse)


提取数据后，`parse()` 方法会通过链接请求下一个页面，它会使用 `urljoin()` 方法生成一个绝对路径（抓取的链接是相对路径的）并且请求下一页，这个方法注册自己为回调函数完成下一页的数据提取，从而实现爬取所有的页面。

Scrapy 追踪链接的机制：当你在一个回调方法中发起一个 Request 的时候，Scrapy 会确保请求发送并且当请求完成的时候会注册一个可以执行的回调方法。

使用这种方法，你可以构建复杂的抓取器去根据你定义的规则追踪链接，并且从不同的页面中提取多种数据。

上述代码将创建一个循环,跟进所有没有抓取过的下一页的链接, 包括论坛和有分页的网站。



创建多个请求的快捷方式
=======================


你可以使用 `response.follow` 方法创建 Request 对象：::


	import scrapy
	class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/page/1/',
        ]
        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.css('span small::text').extract_first(),
                    'tags': quote.css('div.tags a.tag::text').extract(),
                }
            next_page = response.css('li.next a::attr(href)').extract_first()
            if next_page is not None:
                yield response.follow(next_page, callback=self.parse)

和 `scrapy.Request` 不同， `response.follow` 支持相对 URL 路径——不需要调用 `urljoin` .但是 `response.follow`  仅仅返回一个`Request`接口,你仍然要发起这个请求当然，`response.follow` 的第一个参数不一定是字符串也可以是一个选择器,这个选择器应该提供必须的属性: ::

	for href in response.css('li.next a::attr(href)'):
    		yield response.follow(href, callback=self.parse)

对于一个 `<a>` 元素： `response.follow` 会自动使用它的 `href` 属性。所以，代码可以更短: ::

	for a in response.css('li.next a'):
    		yield response.follow(a, callback=self.parse)

>!注意
>`response.follow(response.css('li.next a'))` 是错误的，因为`response.css`返回一个类似列表的对象，这个对象包括这个选择器的所有结果，它并不是一个单选择器。使用上面例子中的`for`循环或者`response.follow(response.css('li.next a')[0])`是不错的选择。
>


更多的例子和模式
=================

下面是另一个列举回调和链接跟踪的爬虫，抓取作者信息: ::

    import scrapy
    class AuthorSpider(scrapy.Spider):
    name = 'author'

    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        # follow links to author pages
        for href in response.css('.author + a::attr(href)'):
            yield response.follow(href, self.parse_author)

        # follow pagination links
        for href in response.css('li.next a::attr(href)'):
            yield response.follow(href, self.parse)

    def parse_author(self, response):
        def extract_with_css(query):
            return response.css(query).extract_first().strip()

        yield {
            'name': extract_with_css('h3.author-title::text'),
            'birthdate': extract_with_css('.author-born-date::text'),
            'bio': extract_with_css('.author-description::text'),
        }

这个爬虫会从主页开始调用`parse_author`跟踪所有的与作者信息有关的页面，和上面的例子一样，分页链接会调用`parse`方法。

相比`scrapy.Request`，使用`response.follow`可以写更少的代码。

.. `parse_author`回调定义了一个很有用的函数，它可以从一个 CSS 查询中提取或清理数据，并且生成一个带有作者信息的 Python 字典。

更有趣的是，我们可以看到：即使同样的作者有很多名言，我们却不需要考虑同一个作者的页面被多次抓取。Scrapy 默认匹配已经抓取过的 URL 来过滤重复的请求，这也避免了因为程序错误而多次请求服务器的问题。你可以设置 `DUPEFILTER_CLASS` 的值决定是否过滤。


希望你已经理解了 Scrapy 跟踪链接和回调函数的机制。

另一个利用爬虫的追踪链接机制的例子是 `CrawlSpider` 类，你可以在任何爬虫中使用它应用一个小型的规则引擎，然后在它的基层构建你的爬虫。



当然，可能经常需要用多个页面的数据来构建一个抓取条目，这时就可以[设法给回调函数传递参数] pass_params_ 。
 
.. _pass_params: https://doc.scrapy.org/en/latest/topics/request-response.html#topics-request-response-ref-request-callback-arguments

使用Spider的参数
=================

当你在命令行(cmd)运行你的爬虫时,你可以通过使用 -a 选项来向你的爬虫提供一些参数: ::

    scrapy crawl quotes -o quotes-humor.json -a tag=humor

这些参数会被传递到当前爬虫的 Spider类中的`__init__`方法中, 同时这些参数会默认的成为该爬虫的属性。

在本例中, 你可以通过`self.tag` 来使用通过tag参数提供的值.同时利用这个特性来构建你的URL,让爬虫去爬取你想要的数据.:: 

    import scrapy 

    class QuotesSpider(scrpay.Spider):
        name = "quotes"

        def start_requests(self):
            url = "http://quotes.toscrape.com/"
            tag = getattr(self, 'tag', None)
            if tag is not None:
                url = url + 'tag/' + tag
            yield scrapy.Request(url, self.parse)

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                        'text': quote.css('span.text::text').extract_first(),
                        'author': quote.css('small.author::text').extract_first()
                }
        
        next_page = response.css('li.next a::attr(href)').extract_first()
        if next_page is not None:
            yield response.follow(next_page, self.parse)

如果你将tag=humor这个参数传递给了这个爬虫, 那么该爬虫只会获取与humor这个标签相关的url， 比如说: `http://quotes.toscrape.com/tag/humor`。

获取更多关于爬虫参数的信息 link_

.. _link: https://doc.scrapy.org/en/latest/topics/spiders.html#spiderargs


下一步
==========

对于Scrapy来说,这只是一个很基础的教程, 有很多其他的特性在本节并没有提到.你可以在 `Scrapy at a glance`_ 这一章节查看[What else?]()
来获取更多有关Scrapy的重要信息。

你可以通过 `Basic concepts`_ 这一章节继续学习更多有关于命令行工具, spiders, 选择器(用来提取数据),和对提取的数据进行规范化等一系列在本章没有涉及到的内容.如果你更迫不及待的想去试一下案例项目, 请查看 Examples_。

.. _`Basic concepts`: http://www.baidu.com
.. _`Scrapy at a glance`: http://www.baidu.com
.. _Examples: :https://www.baidu.com
