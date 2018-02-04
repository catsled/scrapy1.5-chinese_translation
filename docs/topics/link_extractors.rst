.. _docs-topics-link-extractors:

===============
链接提取器
===============

链接提取器的设计目的是通过 :class:`scrapy.http.Response` 对象从网页中提取链接，以供后续使用。

Scrapy内置有 ``scrapy.linkextractors.LinkExtractor`` 对象，不过你可以通过实现一个非常简单的接口来自定义满足你自己需求的链接提取器。

每一个链接提取器唯一的公共方法是 ``extract_links``，用于接收一个 :class:`~scrapy.http.Response` 对象并返回一个 :class:`scrapy.link.Link` 对象的列表。链接提取器意在只实例化一次，而其 ``extract_links`` 方法会根据不同的响应被多次调用来提取链接。

链接提取器通过一组规则在 :class:`~scrapy.spiders.CrawlSpider` 类（Scrapy内置）中使用，但即使你不从 :class:`~scrapy.spiders.CrawlSpider` 继承，你也可以在你的爬虫中使用它，因为它的目的非常简单：提取链接。


.. _docs-topics-link-extractors-ref:

内置链接提取器参考
==================================

.. module:: scrapy.linkextractors
   :synopsis: Link extractors classes

在 :mod:`scrapy.linkextractors` 模块中提供了与Scrapy捆绑在一起的链接提取器类。

默认的链接提取器是 ``LinkExtractor``，与 :class:`~.LxmlLinkExtractor` 相同::

    from scrapy.linkextractors import LinkExtractor

在先前的Scrapy版本中，有其它的链接提取器类，但现在已经被弃用了。

LxmlLinkExtractor
-----------------

.. module:: scrapy.linkextractors.lxmlhtml
   :synopsis: lxml的基于HTMLParser的链接提取器


.. class:: LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href',), canonicalize=False, unique=True, process_value=None, strip=True)

    LxmlLinkExtractor是一个值得推荐的链接提取器，具有方便的过滤器选项。它使用lxml强大的HTMLParser实现。

    :param allow: 一个正则表达式（或者正则表达式列表），这个（绝对）url必须匹配才能被提取。如果没有提供这个参数（或者为空），它将匹配所有链接。
    :type allow: 一个正则表达式（或者正则表达式列表）

    :param deny: 一个正则表达式（或者正则表达式列表），这个（绝对）url必须匹配才能被排除。它的优先级别高于参数 ``allow``。如果没有提供这个参数（或者为空），它将不会排除任何链接。
    :type deny: 一个正则表达式（或者正则表达式列表）

    :param allow_domains: 包含将被考虑用于提取链接的域的字符串的单个值列表
    :type allow_domains: str 或 list

    :param deny_domains: 包含将不会被考虑用于提取链接的域的字符串的单个值或列表
    :type deny_domains: str 或 list

    :param deny_extensions: 包含当提取链接时应该被忽略的插件的字符串的单个值或列表。如果没有提供这个参数，则默认为 `scrapy.linkextractors`_ package 中定义的 ``IGNORED_EXTENSIONS`` 列表。
    :type deny_extensions: list

    :param restrict_xpaths: 是一个XPath（或XPath的列表），它定义了响应中应从中提取链接的区域。如果提供了这个参数，只有由那些XPath选中的文本将会被扫描链接。请看以下的例子。
    :type restrict_xpaths: str 或 list

    :param restrict_css: 是一个CSS选择器，它定义了相应中应从中提取链接的区域。具有与 ``restrict_xpaths`` 相同的功能。
    :type restrict_css: str 或 list

    :param tags: 提取链接时应考虑的单个标签或标签列表，默认为 ``('a', 'area')``。
    :type tags: str 或 list

    :param attrs: 查找提取链接时应考虑的单个属性或属性列表（仅适用于 ``tags`` 参数中指定的那些标签），默认为 ``('href',)``
    :type attrs: list

    :param canonicalize: 规范每个提取的 url (使用 w3lib.url.canonicalize_url)。 默认为 ``False``。请注意，canonicalize_url 用于重复检查；它可以改变服务器端可见的URL，因此对于使用规范化和原始URL的请求，响应可能会有所不同。如果你使用LinkExtractor来跟踪链接，那么保持默认的 ``canonicalize=False`` 可以使程序更加健壮。
    :type canonicalize: boolean

    :param unique: 是否应该对提取的链接进行重复过滤。
    :type unique: boolean

    :param process_value: 接收从标签提取的每个值和被扫描的属性的函数，并且可以修改该值并返回一个新的值，或返回 ``None`` 来忽略该链接。如果没有提供这个参数，``process_value`` 默认为 ``lambda x: x``

        .. highlight:: html

        例如，从这个代码中提取链接::

            <a href="javascript:goToPage('../other/page.html'); return false">Link text</a>

        .. highlight:: python

        你可以在 ``process_value`` 中使用以下函数::

            def process_value(value):
                m = re.search("javascript:goToPage\('(.*?)'", value)
                if m:
                    return m.group(1)

    :type process_value: callable

    :param strip: 是否从提取的属性中去除空格。根据HTML5标准，前导和尾随空格必须从 ``<a>``， ``<area>`` 和其它许多元素的 ``href`` 属性，``<img>``， ``<iframe>`` 元素的 ``src`` 等等的属性中去除。所以LinkExtractor默认去除空格字符。可通过设置 ``strip=False`` 来关闭它（例如，如果你从允许前/后空格的元素或属性中提取URL的时候）。
    :type strip: boolean

.. _scrapy.linkextractors: https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py
