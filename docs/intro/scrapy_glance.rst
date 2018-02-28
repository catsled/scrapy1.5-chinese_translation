.. _docs-intro-scrapy-glance:


===============
Scrapy初见
===============

Scrapy是一个应用程序框架，可用于爬取网站并提取可用于各种应用广泛的程序（如数据挖掘，信息处理或档案处理）的结构化数据。

尽管Scrapy最初是为爬取网页 `wiki`_ 而设计的，但它也可用于调用各种API（如：`Amazon Associates Web Services`_ ）提取数据，或作为一个通用的网络爬虫工具。

.. _`wiki`: https://en.wikipedia.org/wiki/Web_scraping
.. _`Amazon Associates Web Services`: https://affiliate-program.amazon.com/gp/advertising/api/detail/main.html


通过一个爬虫例子来了解
===========================


为了向你展现Scrapy所能带给你的东西，我们来看一下这个Scrapy Spider的例子，让我们使用最简单的方式来运行一个爬虫。

以下是一段爬虫代码，从网站<http://quotes.toscrape.com>爬取名人名言: ::

    import scrapy

    class QuotesSpider(scrapy.Spider):
        name = "quotes"
        start_urls = [
            'http://quotes.toscrape.com/tag/humor/',
        ]

        def parse(self, response):
            for quote in response.css('div.quote'):
                yield {
                    'text': quote.css('span.text::text').extract_first(),
                    'author': quote.xpath('span/small/text()').extract_first(),
                }

            next_page = response.css('li.next a::attr("href")').extract_first()
            if next_page is not None:
                yield response.follow(next_page, self.parse)


将它放到一个文本文件中，并将其命名为 `quotes_spider.py` ，然后使用 `runspider` 命令运行该爬虫： ::

    scrapy runspider quotes_spider.py -o quotes.json

完成之后，你将得到一个 `quotes.json` 的文件，该文件以JSON格式存储了一系列包含文本和作者的名言，如下所示（为了获得更好的可读性，在这里重新格式化）： ::

    [{
        "author": "Jane Austen",
        "text": "\u201cThe person, be it gentleman or lady, who has not pleasure in a good novel, must be intolerably stupid.\u201d"
    },
    {
        "author": "Groucho Marx",
        "text": "\u201cOutside of a dog, a book is man's best friend. Inside of a dog it's too dark to read.\u201d"
    },
    {
        "author": "Steve Martin",
        "text": "\u201cA day without sunshine is like, you know, night.\u201d"
    },
    ...]


刚刚发生了什么？
^^^^^^^^^^^^^^^^^^

当你运行命令： `scrapy runspider quotes_spider.py` 时，Scrapy在代码里面查找Spider的定义，并通过它的爬虫引擎运行它。

通过向 `start_urls` 属性中定义的URL（在本例中，只含有 *humor* 类别中的名言的URL）发出请求，并调用默认的回调方法 `parse`，将响应对象作为参数传递，随即开始抓取。在 `parse` 回调中，我们使用CSS选择器循环遍历quote元素，产生带有提取的名言文本和作者的Python字典，查找指向下一页的链接，并使用相同的回调方法 `parse` 调度另一个请求。

这里你会注意到Scrapy的一个主要优点：请求被异步调度和处理 ( `async`_ )。这意味着Scrapy不需要等待一个请求被完成与处理，它可以同时发送另一个请求或者做其他事情。这也意味着即便某些请求失败或者处理时发生了错误，其他请求也可以继续。

虽然这可以快速执行爬虫（同时以容错方式发送多个并发请求），但Scrapy还可以通过一些设置(`settings_`)控制爬虫的下载策略。你也可以在每个请求之间设置一个下载延迟，限制每个域或每个IP的并发请求量，甚至使用一个自动限速扩展( `auto throttle`_ ) 来自动计算这些延迟。

.. note:: 

    这里使用feed导出 `feed export`_ 生成了JSON格式的文件，你可以轻易地更改导出格式（例如，XML或CSV）或存储在后端（例如，FTP或 `Amazon S3`_ ）。你还可以编写一个项目管道( `item pipeline`_ )将数据存储在数据库中。

.. _`async`: https://doc.scrapy.org/en/latest/topics/architecture.html#topics-architecture

.. _`settings`: https://doc.scrapy.org/en/latest/topics/settings.html#topics-settings-ref
.. _`feed export`: https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports
.. _`Amazon S3`: https://aws.amazon.com/cn/s3/
.. _`auto throttle`: https://doc.scrapy.org/en/latest/topics/autothrottle.html#topics-autothrottle
.. _`item pipeline`: https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline

还有什么？
================

你已经知道了如何使用Scrapy从网站中提取和存储数据，但这仅仅是表面。Scrapy提供了许多强大的功能，使得爬虫更加简单和高效，例如：

1. 内置支持使用扩展CSS选择器和XPath表达式从HTML/XML源选择和提取 `select and extract`_ 数据，使用正则表达式提取帮助器方法。

2. 一个交互式shell控制台 ( `scrapy-shell`_ )（IPython aware），用于尝试使用CSS和XPath表达式来抓取数据，在编写和调试爬虫时非常有用。

3. 内置支持以多种格式（JSON, CSV, XML）生成feed导出 ( `export`_ )并将其存储在多种后端（FTP, S3, 本地文件系统）。

4. 强大的编码支持和自动检测，用于处理外部的，非标准化的和破碎的编码声明。

5. 强大的可扩展性支持 ( `extension`_ )，允许你使用信号( `signal`_ )和定义良好的API（中间件 ( `middleware`_ )和管道( `pipeline`_ )）来插入自己的功能。

6. 广泛的内置扩展和中间件处理：

 - cookies和session处理
 
 - HTTP功能，如压缩，身份认证，缓存
 
 - 用户代理欺骗
 
 - robots.txt
 
 - 爬虫深度限制
 
 - 更多

- 一个远程控制台( `remote console`_ )，用于连接Scrapy进程中运行的Python控制台，以便反编译和调试你的爬虫。

- 再加上其他好处，比如可重复使用的爬虫，可以从 `Sitemaps`_ 和XML/CSV feeds中抓取网站，一个用于自动下载与抓取物品相关的图片( `media-pipeline`_ )（或其他媒体）的媒体管道，缓存DNS解析器等等！


.. _`select and extract`: https://doc.scrapy.org/en/latest/topics/selectors.html#topics-selectors
.. _`scrapy-shell`: https://doc.scrapy.org/en/latest/topics/shell.html#topics-shell
.. _`export`: https://doc.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports
.. _`extension`: https://doc.scrapy.org/en/latest/index.html#extending-scrapy
.. _`signal`: https://doc.scrapy.org/en/latest/topics/signals.html#topics-signals
.. _`middleware`: https://doc.scrapy.org/en/latest/topics/extensions.html#topics-extensions
.. _`pipeline`: https://doc.scrapy.org/en/latest/topics/item-pipeline.html#topics-item-pipeline
.. _`remote console`: https://doc.scrapy.org/en/latest/topics/telnetconsole.html#topics-telnetconsole
.. _`Sitemaps`: https://www.sitemaps.org/index.html
.. _`media-pipeline`: https://doc.scrapy.org/en/latest/topics/media-pipeline.html#topics-media-pipeline


下一步是什么？
================

接下来的步骤是安装Scrapy :ref:`docs-intro-scrapy-install`，按照教程 :ref:`docs-intro-scrapy-tutorial` 学习如何创建一个完整的Scrapy项目并加入社区 community_ 。感谢你的关注！

.. _community: https://scrapy.org/community/