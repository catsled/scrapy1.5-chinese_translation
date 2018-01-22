.. _docs-intro-selectors:

=========
选择器
=========

当你正在爬取网站时，最常见的任务就是从HTML源码中提取数据。下面这些库可以帮助你完成这些任务:

 * `BeautifulSoup`_ : 在Python编程人员眼中，这是一个非常流行的网页数据提取库，它通过HTML代码来构建一个
   Python对象，并且可以合理的处理一些HTML代码中的错误标记，当然，它的缺点就是处理速度太慢。

 * `lxml`_ : 这是一个XML解释库，它基于一个非常 pythonic 的接口 `ElementTree`_。(lxml 并不是Python
    标准库中的一部分。)

Scrapy中提供了一套自己的数据提取机制。因为它通过 `Xpath`_ 或 `CSS`_ 表达式来对HTML文档中指定的部分进行
选择提取，所以它被称作选择器(selecotrs)。

`XPath`_ 是一款为了在XML文档中提取节点的语言，当然，它也可以被用在HTML中。`CSS`_ 是一款适用于HTML文档
风格的语言。它定义了与这些特定HTML元素风格相关联的选择器。

Scrapy选择器建立在 `lxml`_ 库之上，这就意味着它在解析速度和精确度上与lxml很相似。

本节将会解释选择器如何工作，并且描述它们的API(应用程序接口)相比于 `lxml`_的API是多么的小和方便(lxml API由于它不仅仅可以处理标记文档[HTML，XML]的选择，而且可以处理更多其他的任务，所以相较于选择器他的体积更大。)

获取完整的选择器接口文档
:ref: `Selector reference <docs-topics-selectors-ref>`

.. _BeautifulSoup: https://www.crummy.com/software/BeautifulSoup/
.. _lxml: http://lxml.de/
.. _ElementTree: https://docs.python.org/2/library/xml.etree.elementtree.html
.. _cssselect: https://pypi.python.org/pypi/cssselect/
.. _XPath: https://www.w3.org/TR/xpath
.. _CSS: https://www.w3.org/TR/selectors


使用选择器
=============

构建选择器
-------------

.. highlight:: python

Scrapy选择器是一些 :class:`~scrapy.selecotr.Selector` 类的实例，构建该实例可以通过传入 **text** 参数
或者 :class:`~scrapy.http.TextResponse`对象。它将会自动对比传入的参数，然后选择一个最好的解析规则(XML vs HTML)::

    >>> from scrapy.selector import Selector
    >>> from scrapy.http import HtmlResponse

使用text构建::

    >>> body = '<html><body><span>good</span></body></html>'
    >>> Selector(text=body).xpath('//span/text()').extract()
    [u'good']

使用响应对象构建::

    >>> response = HtmlResponse(url='http://example.com', body=body)
    >>> Selector(response=response).xpath('//span/text()').extract()
    [u'good']

为了方便使用，响应(response)对象通过 `.selector` 属性暴露出一个选择器，在合适的情境下，使用
下面这种快捷方式也是完全可以的::

    >>> response.selector.xpath('//span/text()').extract()
    [u'good']


使用选择器
-----------

为了解释如何使用选择器，我们将会使用 `Scrapy shell` (提供一个可交互的测试环境) 和一个定位在Scrapy文档
服务器上的案例页:

    https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

.. _docs-topics-selectors-htmlcode:

这是他的HTML代码:

.. literalinclude:: ../_static/selectors-sample1.html
   :language: html

.. highlight:: sh

首先，打开 scrapy shell::

    scrapy shell https://doc.scrapy.org/en/latest/_static/selectors-sample1.html

接下来，当shell加载完毕，你将会获取一个可用的response对象，在shell中，使用 ``response`` 来调用它，
同时，在 ``response.selector`` 这个属性中携带着一个选择器(selector)。

当我们处理HTML时，选择器将会自动使用HTML解释器。

.. highlight:: python

因此，通过观察这个网页 :ref:`HTML code <topics-selectors-htmlcode>`， 我们构建一个Xpath选择器来
从标题标签(title tag)中提取文本信息::

    >>> response.selector.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]

由于经常需要在responses中使用Xpath和CSS，所以，resonses中包含了两个非常方便的快捷方式: ``response.xpath()``
和 ``response.css()``::

    >>> response.xpath('//title/text()')
    [<Selector (text) xpath=//title/text()>]
    >>> response.css('title::text')
    [<Selector (text) xpath=//title/text()>]

如你所见，``.xpath()`` 和 ``.css()`` 方法将会返回一个 :class:`~scrapy.selector.SelectorList` 类的实例,
该实例又是一个新的选择器列表。所以，使用这个API可以快速的从嵌套的结构中选择到想要的数据::

    >>> response.css('img').xpath('@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

要真正的提取到这个文本数据，你必须调用选择器中的 ``.extract()``方法(该方法将会返回一个数据列表)::

    >>> response.xpath('//title/text()').extract()
    [u'Example website']

如果你只想提取匹配到的第一个数据，你可以使用选择器中的 ``.extract_first()``方法

    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()
    u'Name: My image 1 '

如果找不到匹配的元素，该方法将会返回一个 ``None``::

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first() is None
    True

使用默认的返回值来替代返回的 ``None`` 值::

    >>> response.xpath('//div[@id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'

注意，CSS选择器可以使用CSS3语法来选择文本或属性节点::

    >>> response.css('title::text').extract()
    [u'Example website']

接下来，我们要获取一些简单的URL和一些图片的链接::

    >>> response.xpath('//base/@href').extract()
    [u'http://example.com/']

    >>> response.css('base::attr(href)').extract()
    [u'http://example.com/']

    >>> response.xpath('//a[contains(@href, "image")]/@href').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.css('a[href*=image]::attr(href)').extract()
    [u'image1.html',
     u'image2.html',
     u'image3.html',
     u'image4.html',
     u'image5.html']

    >>> response.xpath('//a[contains(@href, "image")]/img/@src').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

    >>> response.css('a[href*=image] img::attr(src)').extract()
    [u'image1_thumb.jpg',
     u'image2_thumb.jpg',
     u'image3_thumb.jpg',
     u'image4_thumb.jpg',
     u'image5_thumb.jpg']

.. _docs-topics-selectors-nesting-selectors:


嵌套选择器
------------

选择方法 (``.xpath()`` or ``.css()``) 返回一个相同类型的选择器列表，所以，你仍然可以对这些选择器调用
选择方法::

    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    [u'<a href="image1.html">Name: My image 1 <br><img src="image1_thumb.jpg"></a>',
     u'<a href="image2.html">Name: My image 2 <br><img src="image2_thumb.jpg"></a>',
     u'<a href="image3.html">Name: My image 3 <br><img src="image3_thumb.jpg"></a>',
     u'<a href="image4.html">Name: My image 4 <br><img src="image4_thumb.jpg"></a>',
     u'<a href="image5.html">Name: My image 5 <br><img src="image5_thumb.jpg"></a>']

    >>> for index, link in enumerate(links):
    ...     args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract))
    ...     print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
    Link number 3 points to url [u'image4.html'] and image [u'image4_thumb.jpg']
    Link number 4 points to url [u'image5.html'] and image [u'image5_thumb.jpg']


在选择器中使用正则表达式
------------------------

:class:`~scrapy.selector.Selector` 类也包含一个 ``.re()`` 方法，该方法可以使用正则表达式来提取数据。
尽管如此，不同与 ``.xpath()`` 或 ``.css()``方法， ``.re()`` 将会返回一个unicode字符串列表。所以你不能
嵌套的使用 ``.re()`` 。

下面这个例子将会从 :ref:`HTML code<docs-topics-selectors-htlcode>` 网站上提取图片名称::

    >>> response.xpath('//a[@contains(@href, "iamge")]/text()').re(r'Name:\s*(.*)')
    [u'My image 1',
     u'My image 2',
     u'My image 3',
     u'My image 4',
     u'My image 5']

.. _docs-selectors-relative-xpaths:


使用Xpaths相对路径
--------------------

时刻谨记，如果你正在使用嵌套的选择器，并且使用以 ``/`` 开头的XPath语法，你将不会从你之前的选择器中继续查找，
而是以绝对路径从整个文档中搜索。

举个例子，假设你想要从 ``<div>`` 元素中提取所有的 ``<p>`` 元素。首先，你应该获取所有的 ``<div>`` 元素::

    >>> divs = response.xpath('//div')

然后，你可能会想要使用接下来的方法，但是，它是错误的，这个方法实际上提取的是文档中所有的``<p>`` 元素，而不是
仅仅是包含在 ``<div>`` 元素中的 ``<p>`` 元素::

    >>> for p in divs.xpath('//p'):  # this is wrong - gets all <p> from the whole document
    ...     print p.extract()

正确的方法是这样的(注意 ``.//p`` 的前缀 ``.`` )::

    >>> for p in divs.xpath('.//p'):  # extracts all <p> inside
    ...     print p.extract()

另一个常用的方法就是，直接提取所有的 子元素 ``<p>``::

    >>> for p in divs.xpath('p'):
    ...     print p.extract()

查看更多关于XPaths相关路径的问题 `Location Paths`_ 。

.. _Location Paths: https://www.w3.org/TR/xpath#location-paths

.. _docs-topics-selectors-xpath-variables:


在XPath表达使用使用变量
--------------------------

XPath允许你在XPath表达式中使用参考变量，用 ``$somevariable`` 这样的语法来实现。这有些像参数化查询，或是在你的
SQL语句中使用 ``?`` 来作为传递参数的占位符(该占位符将会被传递进去的参数替换)。

在这个案例中，我们将会使用元素的"id"属性去匹配一个元素，但是我们不会使用硬编码的形式去实现它::

    >>> # `$val` used in the expression, a `val` argument needs to be passed
    >>> response.xpath('//div[@id=$val]/a/text()', val='images').extract_first()
    u'Name: My image 1 '

接下来这个案例中，我们将会去寻找一个包含5个子元素 ``<a>`` 的 ``<div>`` 元素(这里，我们传递一个整型值 ``5``)::

    >>> response.xpath('//div[count(a)=$cnt]/@id', cnt=5).extract_first()
    u'images'

在使用``.xpath()`` 时，所有的参考变量都需要被一个实际的值替换,否则你将会得到一个 ``ValueError: XPath error:`` 异常。

使用`parsel`_ 库来增强Scrapy selector，查看更多关于 `XPath variables`_ 的细节和案例。

.. _parsel: https://parsel.readthedocs.io/
.. _XPath variables: https://parsel.readthedocs.io/en/latest/usage.html#variables-in-xpath-expressions


使用EXSLT扩展
-----------------

Scrapy选择器建立在 `lxml`_ 之上， 它也支持使用一些预注册的命名空间，用来在XPath表达式中使用 `EXSLT`_ 扩展:


======  =====================================    =======================
prefix  namespace                                usage
======  =====================================    =======================
re      \http://exslt.org/regular-expressions    `regular expressions`_
set     \http://exslt.org/sets                   `set manipulation`_
======  =====================================    =======================


正则表达式
~~~~~~~~~~~

这个例子中， ``test()``函数可以证明，当XPath中的 ``starts-with()`` 或 ``contains()`` 方法能力不足时，
正则表达式是多么的有用。

选择所有包含以数字结尾的"class"属性的list元素的链接::

    >>> from scrapy import Selector
    >>> doc = """
    ... <div>
    ...     <ul>
    ...         <li class="item-0"><a href="link1.html">first item</a></li>
    ...         <li class="item-1"><a href="link2.html">second item</a></li>
    ...         <li class="item-inactive"><a href="link3.html">third item</a></li>
    ...         <li class="item-1"><a href="link4.html">fourth item</a></li>
    ...         <li class="item-0"><a href="link5.html">fifth item</a></li>
    ...     </ul>
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> sel.xpath('//li//@href').extract()
    [u'link1.html', u'link2.html', u'link3.html', u'link4.html', u'link5.html']
    >>> sel.xpath('//li[re:test(@class, ""item-\d$")]//@href').extract()
    [u'link1.html', u'link2.html', u'link4.html', u'link5.html']
    >>>

.. warning:: C库中的 ``libxslt`` 本身并不支持EXSLT正则表达式，所以 `lxml`_ 通过对Python中的 ``re`` 模块
    实现一个钩子函数来实现这个功能。因此，在你的XPath表达式中使用正则函数将会带来较小的性能降低。


集合操作
~~~~~~~~~~~~

该操作可以很方便的在接下来的案例中提取文本元素时排除一部分文档树

提取一组包含itemscopes和相对应的itemprops的microdata(案例内容来自 http://schema.org/Product)::

    >>> doc = """
    ... <div itemscope itemtype="http://schema.org/Product">
    ...   <span itemprop="name">Kenmore White 17" Microwave</span>
    ...   <img src="kenmore-microwave-17in.jpg" alt='Kenmore 17" Microwave' />
    ...   <div itemprop="aggregateRating"
    ...     itemscope itemtype="http://schema.org/AggregateRating">
    ...    Rated <span itemprop="ratingValue">3.5</span>/5
    ...    based on <span itemprop="reviewCount">11</span> customer reviews
    ...   </div>
    ...
    ...   <div itemprop="offers" itemscope itemtype="http://schema.org/Offer">
    ...     <span itemprop="price">$55.00</span>
    ...     <link itemprop="availability" href="http://schema.org/InStock" />In stock
    ...   </div>
    ...
    ...   Product description:
    ...   <span itemprop="description">0.7 cubic feet countertop microwave.
    ...   Has six preset cooking categories and convenience features like
    ...   Add-A-Minute and Child Lock.</span>
    ...
    ...   Customer reviews:
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Not a happy camper</span> -
    ...     by <span itemprop="author">Ellie</span>,
    ...     <meta itemprop="datePublished" content="2011-04-01">April 1, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1">
    ...       <span itemprop="ratingValue">1</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">The lamp burned out and now I have to replace
    ...     it. </span>
    ...   </div>
    ...
    ...   <div itemprop="review" itemscope itemtype="http://schema.org/Review">
    ...     <span itemprop="name">Value purchase</span> -
    ...     by <span itemprop="author">Lucas</span>,
    ...     <meta itemprop="datePublished" content="2011-03-25">March 25, 2011
    ...     <div itemprop="reviewRating" itemscope itemtype="http://schema.org/Rating">
    ...       <meta itemprop="worstRating" content = "1"/>
    ...       <span itemprop="ratingValue">4</span>/
    ...       <span itemprop="bestRating">5</span>stars
    ...     </div>
    ...     <span itemprop="description">Great microwave for the price. It is small and
    ...     fits in my apartment.</span>
    ...   </div>
    ...   ...
    ... </div>
    ... """
    >>> sel = Selector(text=doc, type="html")
    >>> for scopr in sel.xpath('//div[@itemscope]')
    ...     print "current scope:", scope.xpath('@itemtype').extract()
    ...     props = scope.xpath('''
                        set:difference(./descendant::*/@itemprop,
                        .//*[@itemscope]/*/@itemprop)''')
    ...     print "     properties:", props.extract()
    ...     print

    current scope: [u'http://schema.org/Product']
        properties: [u'name', u'aggregateRating', u'offers', u'description', u'review', u'review']

    current scope: [u'http://schema.org/AggregateRating']
        properties: [u'ratingValue', u'reviewCount']

    current scope: [u'http://schema.org/Offer']
        properties: [u'price', u'availability']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    current scope: [u'http://schema.org/Review']
        properties: [u'name', u'author', u'datePublished', u'reviewRating', u'description']

    current scope: [u'http://schema.org/Rating']
        properties: [u'worstRating', u'ratingValue', u'bestRating']

    >>>

我们首先遍历每一个 ``itemscope`` 元素， 然后查找所有的 ``itemprops`` 元素，同时排除那些子元素中包含 ``itemscope`` 
的元素。

.. _EXSLT: http://exslt.org/
.. _regular expressions: http://exslt.org/regexp/index.html
.. _set manipulation: http://exslt.org/set/index.html


关于XPath的一些小提示
------------------------

当你在Scrapy选择器中使用XPath时，这里有一些小提示: `this post from ScrapingHub's blog`_, 如果你
并不熟悉XPath，你可以看一下 `XPath tutorial`_。

.. _`XPath tutorial`: http://www.zvon.org/comp/r/tut-XPath_1.html
.. _`this post from ScrapingHub's blog`: https://blog.scrapinghub.com/2014/07/17/xpath-tips-from-the-web-scraping-trenches/


在条件语句中使用文本节点
~~~~~~~~~~~~~~~~~~~~~~~~~~

当你在 `XPath string function`_ 中使用文本内容作为参数时，最好避免使用 ``.//text()`` ，而是使用 ``.``来作为替代。
这是因为， 使用表达式 ``.//text()`` 将会产出一个文本元素(节点)集合( *node-set* )，将该节点集合转换为一个字符串时，
该节点将会作为参数传递给像: ``contains()`` 或 ``starts-with()`` 这样的字符串函数，但是只会返回第一个元素的结果。

例::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<a href="#">Click here to go to the <strong>Next Page</strong></a>')

将一个节点集合 **node-set** 转换为字符串::

    >>> sel.xpath('//a//text()').extract() # take a peek at the node-set
    [u'Click here to go to the ', u'Next Page']
    >>> sel.xpath("string(//a[1]//text())").extract() # convert it to string
    [u'Click here to go to the ']

获取一个节点以及所有子节点中的字符串::

    >>> sel.xpath("//a[1]").extract() # select the first node
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']
    >>> sel.xpath("string(//a[1])").extract() # convert it to string
    [u'Click here to go to the Next Page']

所以，使用 ``.//text()`` 并不会选择到所有的节点中的文本::

    >>> sel.xpath("//a[contains(.//text(), 'Next Page')]").extract()
    []

但是，使用 ``.`` 可以匹配到标签内的所有内容::

    >>> sel.xpath("//a[contains(., 'Next Page')]").extract()
    [u'<a href="#">Click here to go to the <strong>Next Page</strong></a>']

.. _`XPath string function`: https://www.w3.org/TR/xpath/#section-String-Functions


注意 //node[1] 与 (//node)[1]的区别
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``//node[1]`` 选择每一个节点下的第一个子元素。

``(//node)[1]`` 选择所有的节点，然后只获取第一个。

例::

    from scrapy import Selector
    >>> sel = Selector(text="""
    ....:     <ul class="list">
    ....:         <li>1</li>
    ....:         <li>2</li>
    ....:         <li>3</li>
    ....:     </ul>
    ....:     <ul class="list">
    ....:         <li>4</li>
    ....:         <li>5</li>
    ....:         <li>6</li>
    ....:     </ul>""")
    >>> xp = lambda x: sel.xpath(x).extract()

这样操作将会获取到所有 ``<li>`` 元素下的第一个子元素::

    >>> xp("//li[1]")
    [u'<li>1</li>', u'<li>4</li>']

获取整个文档中的第一个 ``<li>``::

    >>> xp("(//li)[1]")
    [u'<li>1</li>']

获取所有 ``<ul>`` 下的第一个 ``<li>`` 元素::

    >>> xp("//ul/li[1]")
    [u'<li>1</li>', u'<li>4</li>']

在整个文档中获取第一个 ``<ul>`` 下的第一个 ``<li>`` 元素::

    >>> xp("(//ul/li)[1]")
    [u'<li>1</li>']


使用CSS进行class属性查询
~~~~~~~~~~~~~~~~~~~~~~~~~~~

因为一个元素可以包含很多CSS classes，所以在XPath中通过class进行选择是一个更加繁琐的方式::

    *[contains(concat(' ', normalize-space(@class), ' '), ' someclass ')]

如果使用 ``@class='someclass'`` 你或许或遗漏一些拥有其他`classes`的元素，如果你使用 ``contains(@class, 'someclass`)``
可能会获取到一些你不想要的元素(该元素中拥有一些其他的通用class)。

就结果而言，Scrapy选择器允许你链式调用，所以，大多数时间你都会将CSS与XPath结合使用::

    >>> from scrapy import Selector
    >>> sel = Selector(text='<div class="hero shout"><time datetime="2014-07-23 19:00">Special date</time></div>')
    >>> sel.css('.shout').xpath('./time@datetime').extract()
    [u'2014-07-23 19:00']

相较于使用上面那种繁琐的XPath方法，这种方法更加简洁。记住，在XPath表达式中使用 ``.`` 来从上一个选择器中
继续提取。

.. _docs-topics-selectors-ref:


内置选择器参考
================


.. module:; scrapy.selector
    :synopsis: Selector class


选择器对象(Selector objects)
-----------------------------

.. class:: Selector(response=None, text=None, type=None)

  一个可以从当前内容中进行选择(select)，并且是封装在response之上的 :class:`Selector` 实例。

  ``response`` 是一个可以用来进行选择和数据提取的 :class:`~scrapy.http.HtmlResponse` 或
  :class:`~scrapy.http.XmlResponse` 的对象。

  ``text`` 是一个uicode字符串，或者是一个utf-8编码的文档。应该在 ``response`` 不可用的情况下
  使用它，但是 ``text`` 和 ``response`` 两者同时使用是未定义的(不要同时使用)。

  ``type`` 用来定义选择器的类型, 它可以是 ``"html"``, ``"xml"`` 或 ``None`` (default)。

    当 ``type`` 为 ``None`` 时，选择器会根据 ``response`` 自动选择最好的选择器类型，或者，当
    使用 ``text`` 时， 会默认的使用 ``"html"``。

    如果 ``type`` 为空，并且传递了 ``response`` ，那么，选择器将会依据响应(response)类型来推断使用哪种类型的选择器:

        * ``"html"`` for :class:`~scrapy.http.HtmlResponse` type
        * ``"xml"`` for :class:`~scrapy.http.XmlResponse` type
        * ``"html"`` for anything else

    否则，如果 ``type`` 被设定好了， 选择器将不会进行检测，而是使用设定值。

  .. method:: xpath(query)

     查找包含所有匹配xpath ``query`` 的单一化元素，并且以 :class:`SelectorList` 的实例作为返回值。
     该实例列表中同样实现了 :class:`Selector` 接口。

     ``query`` 是一个包含了可以被XPath使用字符串。

     .. note::

         方便起见，该方法可以通过 ``response.xpath()`` 进行调用

  .. method:: css(query)

      同样的，使用CSS选择器并返回一个 :class:`SelectorList` 实例。
      ``query`` 是一个包含了可以被CSS选择器使用的字符串。

      在程序执行的背后，CSS查询被 `cssselect`_ 转换为XPath查询并且以 ``.xpath()``方法运行。

      .. note::

          方便起见，该方法可以通过 ``response.css()`` 调用。

  .. method:: extract()

     序列化并将匹配到的节点作为unicode字符串列表返回。

  .. method:: re(regex)
    
     使用正则表达式(regex)将匹配到的内容以unicode字符串列表返回。

     ``regex`` 是一个被编译过得正则表达式，或者是一个将要被 ``re.compile(regex)`` 编译的
     正则表达式(正则表达式字符串)。

    ..note::

        注意， ``re()`` 和 ``re_first()`` 对HTML实体进行解码(除了 ``&lt;`` 和 ``&amp;``).

  .. method:: register_namespace(prefix, uri)

     在 :class:`Selector`中使用已注册的命名空间。你不能从未注册的非标准命名空间中进行数据的选择
     和提取。

  .. method:: remove_namespaces()

     移除所有的命名空间，允许使用 namespace-less xpaths 进行文档的遍历。

  .. method:: __nonzero__()

     如果有任意一个被选择到的内容，该方法将会返回一个 ``True`` 否则返回一个 ``False``。换句话说，
     就是 :class:`Selector` 的布尔值是通过选择的内容决定的。


选择器列表对象(SelectorList objects)
--------------------------------------

.. class:: SelectorList

   :class:`SelectorList` 类是Python内置类 ``list` 的一个子类，同时提供了一些附加的方法。

   .. method:: xpath(query)

       对列表中的每一个元素调用 ``.xpath()`` 方法，并且返回一个一维的结果来作为另一个 :class:`SelectorList`。

       ``query`` 作为 :meth:`Selector.xpath` 中的一个参数被使用。

   .. method:: css(query)

       对每一个元素调用 ``.extract()`` 方法，然后以一个包含unicode字符串的一维列表作为结果返回。

   .. method:: re()

       对每一个元素调用 ``.re()`` 方法，然后以一个包含unicode字符串的一维列表作为结果返回。


HTML响应类型选择器案例
-------------------------

这里有一些 :class:`Selector` 案例用来解释一些概念。
我们假设接下来的每个 :class:`Selector` 案例都已经被 :class:`~scrapy.http.HtmlResponse` 对象
初始化过了。 ::

    sel = Selector(html_response)

1. 从HTML响应体中选择所有的 ``<h1>`` 元素，并返回一个 :class:`Selector` 对象列表 (或者说是一个 :class:`SelectorList` 对象)::

    sel.xpath("//h1")

2. 从HTML响应体中提取所有的 ``<h1>`` 元素，并返回一个unicode字符串列表::

    sel.xpath("//h1").extract()   # 包含h1标签
    sel.xpath("//h1/text()").extract()  # 不包含h1标签

3. 迭代所有的 ``<p>`` 标签，并打印他们的类(class)属性::

    for node in sel.xpath("//p"):
        print node.xpath("@class").extract()


XML类型选择器案例
-----------------------

这里有一些 :class:`Selector` 案例用来解释一些概念。
我们假设接下来的两个 :class:`Selector` 案例都已经被 :class:`~scrapy.http.XmlResponse` 对象
初始化过了。 ::

    sel = Selector(xml_response)

1. 从XML响应体中选择所有的 ``<product>`` 元素，并返回一个 :class:`Selector` 对象列表(或者说是一个 :class:`SelectorList` 对象)::

    sel.xpath("//product")

2. 从一个需要注册命名空间的 `Google Base XML feed`_ 中提取所有的价格::

    sel.register_namespace("g", "http://base.google.com/ns/1.0")
    sel.xpath("//g:price").extract()

.. _removing-namespaces:


移除命名空间
-------------

在处理scapy项目时，去除命名空间，只使用元素名称是非常方便的。使用 :meth:`Selector.remove_namespaces`
方法，你可以写出更简单方便的XPaths。

让我们通过 GitHub blog atom feed 作为一个案例来进行解释。

.. highlight:: sh

首先，通过 ``scrapy shell`` 打开我们想要爬取的网站url::

    $ scrapy shell https://github.com/blog.atom

.. highlight:: python

当进入 shell 以后，我们尝试选择所有的 ``<link>`` 对象，但是它并不起作用(因为 Atom XML 的命名空间将会
混淆这些节点)::

    >>> response.xpath("//link")
    []

但是，当我们调用 :meth:`Selector.remove_namespaces` 方法，所有的节点都可以直接通过他们的名字获取::

    >>> response.selector.remove_namespaces()
    >>> response.xpath("//link")
    [<Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     <Selector xpath='//link' data=u'<link xmlns="http://www.w3.org/2005/Atom'>,
     ...

你可能会想，为什么不能自动的调用该方法去移除命名空间。有以下两个原因:

1. 删除命名空间需要迭代并修改文档中的所有节点，对于所有被Scrapy爬取的文档来说，该操作对性能的消耗太过昂贵。

2. 尽管很少有元素名称与命名空间冲突的情况，但我们仍需要应对这种情况的发生。

.. _Google Base XML feed: https://support.google.com/merchants/answer/160589?hl=en&ref_topic=2473799


     
        

