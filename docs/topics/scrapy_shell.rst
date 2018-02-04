.. _docs-topics-scrapy-shell:

========================
 Scrapy shell
========================

在 Scrapy shell 中，你可以在不运行爬虫的情况下调试代码。这样就可以测试提取数据的代码，事实上你可以测试任何代码，因为 Scrapy shell 其实也是一种 Python shell 。

在代码中使用 XPath 或 CSS 表达式的时候，可以在 Scrapy shell 中看到爬虫是怎么工作的，还有从 web 页面爬取的数据。在 shell 中，可以交互式的测试你的表达式而不是运行爬虫来测试每一个改变。

一旦你熟悉了 Scrapy shell, 你会觉得这真是个有用的调试爬虫的工具。


配置 shell
========================

如果你安装了 `IPython`_ ，Scrapy shell 会使用 IPython （而不是标准的 Python 控制台）。 `IPython`_  是个强大的控制台，提供智能自动补全和高亮输出。




我们强烈推荐你安装 `IPython`_ ，尤其是使用 Unix 系统的用户（ `IPython`_ 擅长 Unix 系统）。查看 `IPython 安装指南`_ 获取更多信息。



Scrapy 也支持 `bpython`_ ，如果 `IPython`_ 没有安装，Scrapy shell 会使用 `bpython`_ 。

.. _`IPython`: https://ipython.org/
.. _`IPython 安装指南`: https://ipython.org/install.html
.. _`bpython`: https://www.bpython-interpreter.org/


不管安装的是 ``ipython`` 、 ``bpython``  还是标准的 ``python`` 控制台，都可以通过设置 setting 文件中的环境变量 ``SCRAPY_PYTHON_SHELL`` 的值自主选择，或者也可以在 `scrapy.cfg`_ 文件中定义这个变量： ::

    [setting]
    shell = bpython

.. _`scrapy.cfg`: https://doc.scrapy.org/en/latest/topics/commands.html#topics-config-settings


运行 shell
========================
在命令行中运行 Scrapy shell： ::

    scrapy shell <url>

``<url>`` 是你要抓取数据的页面 URL。

`Scrapy shell` 也可以处理本地文件，就是说可以使用 `shell` 处理你想抓取页面的本地拷贝文件。 `shell` 处理文件的语法如下： ::

    # UNIX-style
    scrapy shell ./path/to/file.html
    scrapy shell ../other/path/to/file.html
    scrapy shell /absolute/path/to/file.html

    # File URI
    scrapy shell file:///absolute/path/to/file.html

.. note::

    使用文件的相对路径容易理解，记得使用前缀 ``./`` （或者 ``../``）。使用 ``scrapy shell index.html`` 会报错（这是设计好的，不是 bug ）。

    对于文件 URIs 来说， `shell` 更喜欢 HTTP URLs ， 按照语法，``index.html`` 会被看成像 ``example.com`` 的这种 URL ， shell 会把 ``index.html`` 做为一个域名然后发出一个 DNS lookup 错误。 

    ::
    
        $ scrapy shell index.html
        [ ... scrapy shell starts ... ]
        [ ... traceback ... ]
        twisted.internet.error.DNSLookupError: DNS lookup failed:
        address 'index.html' not found: [Error -5] No address associated with hostname.
        
    当然，如果在当前文件夹中有 ``index.html`` 文件，`shell` 就会正常处理不会报错。


使用 Scrapy shell
========================
Scrapy shell 是一个标准的 Python 控制台（或者你安装了 `IPython`_ 控制台 ），它提供了一些附加的有用函数。

.. _`IPython`: https://ipython.org/

快捷函数
^^^^^^^^^^^^^^^^^^^^^^^^

- ``shelp()`` - 输出所有的对象和函数
- ``fetch(url[, redirect=True])`` -使用提供的 URL 获取一个新的请求并相应更新所有相关对象。设置 ``redirect=False`` 拒绝获取 HTTP 3xx 状态码。
- ``fetch(request)`` - 从提供的请求获取响应并相应的更新所有相关的对象。
- ``view(response)`` - 在本地浏览器中打开获取的响应来查看数据。为了更好的体现从外部引入的链接（比如图片和 style 样式文件）会增加一个 `<base>`_ 标签到响应体。但是，同时你的电脑上会产生一个临时文件，Scrapy 不会自动删除它。

.. _`<base>`: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base


Scrapy 对象
^^^^^^^^^^^^^^^^^^^^^^^

Scrapy shell 会自动从下载的页面中创建一些有用的对象，比如 Response 和 Selector （都是 HTML 和 XML 内容）。


同样的对象还有这些：

- ``crawler`` - 当前 Crawler 对象。
- ``spider`` - 处理 URL 的爬虫，或者在没有处理当前 URL 的爬虫时它是一个 Spider 对象。
- ``request`` - 获取最后页面的 Request 对象。可以用 replace() 方法修改这个请求，或者在 shell 中使用 fetch 获取一个新请求。
- ``response`` - 包含最后获取的页面的 Response 对象。
- ``setting`` - 当前 Scrapy 设置。


shell 会话示例
========================
这是一个典型的示例，我们在 shell 会话中爬取 `https://scrapy.org`_ 页面，然后爬取 `https://reddit.com`_ 页面。最后，我们把 （Reddit）的请求方法改为 POST 。在重新发起请求后会报错。最后我们使用 Ctrl-D 关闭 shell 会话（ Unix 系统），在 Windows 系统中使用 Ctrl-Z 。

注意，这些页面并不是静态的，现在可能已经发生了变化，你使用下面的命令提取数据的时候可能会得到不一样的输出。这个示例只是想让你熟悉 Scrapy shell 的工作流程。

.. _`https://scrapy.org`: https://scrapy.org
.. _`https://reddit.com`: https://reddit.org



首先，运行 shell ： ::

    scrapy shell 'https://scrapy.org' --nolog


shell 请求URL（使用 shell 下载器），然后会输出所有的对象和快捷方法（你会发现下面的这些行都是以 ``[s]`` 开头的）。 ::

    [s] Available Scrapy objects:
    [s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
    [s]   crawler    <scrapy.crawler.Crawler object at 0x7f07395dd690>
    [s]   item       {}
    [s]   request    <GET https://scrapy.org>
    [s]   response   <200 https://scrapy.org/>
    [s]   settings   <scrapy.settings.Settings object at 0x7f07395dd710>
    [s]   spider     <DefaultSpider 'default' at 0x7f0735891690>
    [s] Useful shortcuts:
    [s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
    [s]   fetch(req)                  Fetch a scrapy.Request and update local objects
    [s]   shelp()           Shell help (print this help)
    [s]   view(response)    View response in a browser
    
    >>>



然后，开始使用这些对象： ::


    >>> response.xpath('//title/text()').extract_first()
    'Scrapy | A Fast and Powerful Scraping and Web Crawling Framework'
    
    >>> fetch("https://reddit.com")
    
    >>> response.xpath('//title/text()').extract()
    ['reddit: the front page of the internet']
    
    >>> request = request.replace(method="POST")
    
    >>> fetch(request)
    
    >>> response.status
    404
    
    >>> from pprint import pprint
    
    >>> pprint(response.headers)
    {'Accept-Ranges': ['bytes'],
    'Cache-Control': ['max-age=0, must-revalidate'],
    'Content-Type': ['text/html; charset=UTF-8'],
    'Date': ['Thu, 08 Dec 2016 16:21:19 GMT'],
    'Server': ['snooserv'],
    'Set-Cookie': ['loid=KqNLou0V9SKMX4qb4n; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                'loidcreated=2016-12-08T16%3A21%3A19.445Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                'loid=vi0ZVe4NkxNWdlH7r7; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure',
                'loidcreated=2016-12-08T16%3A21%3A19.459Z; Domain=reddit.com; Max-Age=63071999; Path=/; expires=Sat, 08-Dec-2018 16:21:19 GMT; secure'],
    'Vary': ['accept-encoding'],
    'Via': ['1.1 varnish'],
    'X-Cache': ['MISS'],
    'X-Cache-Hits': ['0'],
    'X-Content-Type-Options': ['nosniff'],
    'X-Frame-Options': ['SAMEORIGIN'],
    'X-Moose': ['majestic'],
    'X-Served-By': ['cache-cdg8730-CDG'],
    'X-Timer': ['S1481214079.394283,VS0,VE159'],
    'X-Ua-Compatible': ['IE=edge'],
    'X-Xss-Protection': ['1; mode=block']}
    >>>


从 spiders 中调用 shell 查看响应。

有时候你想查看正在被爬虫处理的响应，只需要检查一下是否是你要处理的响应。


可以用 ``scrapy.shell.inspect_response`` 函数检查响应。


从爬虫中调用： ::

    
    import scrapy
    
    
    class MySpider(scrapy.Spider):
        name = "myspider"
        start_urls = [
            "http://example.com",
            "http://example.org",
            "http://example.net",
        ]
    
        def parse(self, response):
            # We want to inspect one specific response.
            if ".org" in response.url:
                from scrapy.shell import inspect_response
                inspect_response(response, self)
    
            # Rest of parsing code.

运行爬虫的时候，会得到类似于下面的输出： ::


    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.com> (referer: None)
    2014-01-23 17:48:31-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.org> (referer: None)
    [s] Available Scrapy objects:
    [s]   crawler    <scrapy.crawler.Crawler object at 0x1e16b50>
    ...

    >>> response.url
    'http://example.org'

然后，运行下面的代码看是否会得到数据： ::

    >>> response.xpath('//h1[@class="fn"]')
    []


返回了空列表，在浏览器中打开响应，查看这个响应是不是你要处理的响应： ::

    >>> view(response)
    True


最后，按 Ctrl-D (Windows 系统用 Ctrl-Z ) 退出 shell 重新爬取： ::

    >>> ^D
    2014-01-23 17:50:03-0400 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://example.net> (referer: None)
    ...


但是，你不能用 `fetch` 方法，因为 Scrapy 引擎会被 shell 阻塞。当然，你退出 shell 后，爬虫会继续从刚才停止的地方开始爬取，就像上面展示的那样。






