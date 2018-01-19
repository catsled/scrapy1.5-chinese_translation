.. _topics-feed-exports:

=========
feed导出
=========

.. 新版本:: 0.10

实现爬虫最常需要的功能之一是能够正确地存储所爬到的的数据，这通常意味着生成一个带有爬取到的数据的“导出文件”（通常成为“导出feed”），以供给其他系统使用。

Scrapy自带了feed导出，并且支持多种序列化格式以及存储后端来生成已抓取项目的feed。

.. _topics-feed-format:

序列化格式
===========

为了序列化爬取到的数据，feed导出使用了 :ref:`Item exporters<topics-exporters>`。其自带支持的类型有：

 * :ref:`topics-feed-format-json`
 * :ref:`topics-feed-format-jsonlines`
 * :ref:`topics-feed-format-csv`
 * :ref:`topics-feed-format-xml`

你也可以通过 :setting:`FEED_EXPORTERS` 设置扩展支持的属性。

.. _topics-feed-format-json:

JSON
----

 * :setting:`FEED_FORMAT`: ``json``
 * 使用的 Exporter: :class:`~scrapy.exporters.JsonItemExporter`
 * 数据量大的情况下使用 JSON，请参见 :ref:`this warning <json-with-large-data>`

.. _topics-feed-format-jsonlines:

JSON lines
----------

 * :setting:`FEED_FORMAT`: ``jsonlines``
 * 使用的 Exporter: :class:`~scrapy.exporters.JsonLinesItemExporter`

.. _topics-feed-format-csv:

CSV
---

 * :setting:`FEED_FORMAT`: ``csv``
 * 使用的 Exporter: :class:`~scrapy.exporters.CsvItemExporter`
 * 要指定所要导出的列及其顺序，请使用 :setting:`FEED_EXPORT_FIELDS`。其他feed导出程序也可以使用此选项，但这对于CSV来说很重要，因为与许多其他导出格式不同，CSV使用固定标题。

.. _topics-feed-format-xml:

XML
---

 * :setting:`FEED_FORMAT`: ``xml``
 * 使用的 Exporter: :class:`~scrapy.exporters.XmlItemExporter`

.. _topics-feed-format-pickle:

Pickle
------

 * :setting:`FEED_FORMAT`: ``pickle``
 * 使用的 Exporter: :class:`~scrapy.exporters.PickleItemExporter`

.. _topics-feed-format-marshal:

Marshal
-------

 * :setting:`FEED_FORMAT`: ``marshal``
 * 使用的 Exporter: :class:`~scrapy.exporters.MarshalItemExporter`


.. _topics-feed-storage:

存储
=====

使用feed导出时可以通过使用URI（通过 FEED_URI 设置）来定义存储端。feed导出支持URI方式定义的多种存储后端类型。

自带支持的存储后端有：

 * :ref:`topics-feed-storage-fs`
 * :ref:`topics-feed-storage-ftp`
 * :ref:`topics-feed-storage-s3` (需要 botocore_ 或者 boto_ 库)
 * :ref:`topics-feed-storage-stdout`

有些存储后端也许会因为外部库未安装而无法使用。例如，S3只有在 botocore_ 或者 boto_ 库安装的情况下才可用。

.. _topics-feed-uri-params:

存储 URI 参数
==============

存储URI也可以包含参数，当feed被创建时这些这些参数可以被覆盖：

 * ``%(time)s`` - 当feed被创建时被 timestamp 覆盖
 * ``%(name)s`` - 被 spider 的名字覆盖

其它命名的参数会被spider同名的属性所覆盖。例如，当feed被创建时，``%(site_id)s``将会被``spider.site_id``属性所覆盖。

这里有一些例子用于说明：

 * 每个spider使用一个目录存储在FTP中：

   * ``ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json``

 * 每个spider使用一个目录存储在S3中：

   * ``s3://mybucket/scraping/feeds/%(name)s/%(time)s.json``


.. _topics-feed-storage-backends:

存储后端
=========

.. _topics-feed-storage-fs:

本地文件系统
----------------

将feeds存储在本地文件系统中。

 * URI scheme: ``file``
 * URI 样例: ``file:///tmp/export.csv``
 * 需要的外部依赖库: none

注意：（只有）存储在本地文件系统时，你可以指定一个绝对路径``/tmp/export.csv``并忽略协议（scheme）。不过这仅仅适用于Unix系统中。

.. _topics-feed-storage-ftp:

FTP
---

将feeds存储在FTP服务器中。

 * URI scheme: ``ftp``
 * URI 样例: ``ftp://user:pass@ftp.example.com/path/to/export.csv``
 * 需要的外部依赖库: none

.. _topics-feed-storage-s3:

S3
--

将feeds存储在 `Amazon S3`_ 中.

 * URI scheme: ``s3``
 * URIs 样例:

   * ``s3://mybucket/path/to/export.csv``
   * ``s3://aws_key:aws_secret@mybucket/path/to/export.csv``

 * 需要的外部依赖库: `botocore`_ 或者 `boto`_

你可以通过在URI中传递 user/password 来完成AWS认证，或者也可以通过以下设置来完成：

 * :setting:`AWS_ACCESS_KEY_ID`
 * :setting:`AWS_SECRET_ACCESS_KEY`

.. _topics-feed-storage-stdout:

标准输出
---------

feeds被写入到Scrapy进程的标准输出。

 * URI scheme: ``stdout``
 * URI 样例: ``stdout:``
 * 需要的外部依赖库: none


设定（Settings）
================

这些是配置feed导出的设定：

 * :setting:`FEED_URI` (必须)
 * :setting:`FEED_FORMAT`
 * :setting:`FEED_STORAGES`
 * :setting:`FEED_EXPORTERS`
 * :setting:`FEED_STORE_EMPTY`
 * :setting:`FEED_EXPORT_ENCODING`
 * :setting:`FEED_EXPORT_FIELDS`
 * :setting:`FEED_EXPORT_INDENT`

.. currentmodule:: scrapy.extensions.feedexport

.. setting:: FEED_URI

FEED_URI
--------

Default: ``None``

导出feed的URI。支持的URI协议请参见存储后端 :ref:`topics-feed-storage-backends`。

为了启用feed导出，该设定是必须的。

.. setting:: FEED_FORMAT

FEED_FORMAT
-----------

feed的序列化格式。可用的值请参见序列化格式 :ref:`topics-feed-format`。

.. setting:: FEED_EXPORT_ENCODING

FEED_EXPORT_ENCODING
--------------------

Default: ``None``

feed使用的编码

如果未设置或设置为 ``None`` (默认)，则除JSON导出因历史原因使用安全的数字编码（``\uXXXX`` 序列）以外，其余都使用UTF-8。

如果你想让JSON使用UTF-8，也可以使用 ``utf-8``。

.. setting:: FEED_EXPORT_FIELDS

FEED_EXPORT_FIELDS
------------------

Default: ``None``

要导出的字段列表，可选。
示例: ``FEED_EXPORT_FIELDS = ["foo", "bar", "baz"]``.

使用 FEED_EXPORT_FIELDS 选项定义要导出的字段及其顺序。

当 FEED_EXPORT_FIELDS 为空或者None（默认值）时，Scrapy使用在字典中定义好的字段或子类 :class:`~.Item` 正在迭代的spider。

如果一个导出程序要求一组固定的字段（如：:ref:`CSV <topics-feed-format-csv>` 导出格式）并且FEED_EXPORT_FIELDS为空或者为None，则Scrapy会尝试从导出数据中推断字段名称————当前它使用第一项中的字段名称。

.. setting:: FEED_EXPORT_INDENT

FEED_EXPORT_INDENT
------------------

Default: ``0``

用于在每一级别缩进输出的空格量。如果``FEED_EXPORT_INDENT``是一个非负整数，那么数据元素和成员对象将与该缩进级别相匹配。缩进级别为``0``(默认值)或负值，则会把每个项目放在新的一行上。``None`` 选择最紧凑的表示。

目前仅通过 :class:`~scrapy.exporters.JsonItemExporter` 和 :class:`~scrapy.exporters.XmlItemExporter` 实现，即当你导出为 ``.json`` 或者 ``.xml`` 时。

.. setting:: FEED_STORE_EMPTY

FEED_STORE_EMPTY
----------------

Default: ``False``

是否导出空feeds（即没有项目的feeds）。

.. setting:: FEED_STORAGES

FEED_STORAGES
-------------

Default: ``{}``

包含你项目支持的额外的存储后端的字典。字典的键(keys)是URI协议，值(values)是存储类的路径。

.. setting:: FEED_STORAGES_BASE

FEED_STORAGES_BASE
------------------

Default::

    {
        '': 'scrapy.extensions.feedexport.FileFeedStorage',
        'file': 'scrapy.extensions.feedexport.FileFeedStorage',
        'stdout': 'scrapy.extensions.feedexport.StdoutFeedStorage',
        's3': 'scrapy.extensions.feedexport.S3FeedStorage',
        'ftp': 'scrapy.extensions.feedexport.FTPFeedStorage',
    }

包含Scrapy支持的内置feed存储后端的字典。你可以通过在 :setting:`FEED_STORAGES` 中为他们的URI scheme分配 ``None``来禁用这些后端中的任意一个。例如，要禁用内置FTP存储后端(无需替换)，将其放置在你的 ``settings.py`` 中::

    FEED_STORAGES = {
        'ftp': None,
    }

.. setting:: FEED_EXPORTERS

FEED_EXPORTERS
--------------

Default: ``{}``

包含你的项目支持的额外导出程序的字典。字典的键(keys)是序列化格式的，值(values)是 :ref:`Item exporter <topics-exporters>` 类的路径。

.. setting:: FEED_EXPORTERS_BASE

FEED_EXPORTERS_BASE
-------------------
Default::

    {
        'json': 'scrapy.exporters.JsonItemExporter',
        'jsonlines': 'scrapy.exporters.JsonLinesItemExporter',
        'jl': 'scrapy.exporters.JsonLinesItemExporter',
        'csv': 'scrapy.exporters.CsvItemExporter',
        'xml': 'scrapy.exporters.XmlItemExporter',
        'marshal': 'scrapy.exporters.MarshalItemExporter',
        'pickle': 'scrapy.exporters.PickleItemExporter',
    }

包含Scrapy支持的内置feed导出程序的字典。你可以通过在 :setting:`FEED_EXPORTERS` 中为其序列化格式分配 ``None`` 来禁用这些导出程序中的任意一个。例如，要禁用内置的CSV导出程序(无需替换)，将其放置在你的 ``settings.py`` 中::

    FEED_EXPORTERS = {
        'csv': None,
    }

.. _URI: https://en.wikipedia.org/wiki/Uniform_Resource_Identifier
.. _Amazon S3: https://aws.amazon.com/s3/
.. _boto: https://github.com/boto/boto
.. _botocore: https://github.com/boto/botocore