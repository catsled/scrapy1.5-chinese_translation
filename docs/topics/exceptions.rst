.. _docs-topics-exceptions:

==========
异常
==========

.. module:: scrapy.exceptions
   :synopsis: Scrapy exceptions

.. _docs-topics-exceptions-ref:

内置异常参考手册
=============================

下面是Scrapy提供的异常及其用法。

DropItem
--------

.. exception:: DropItem

该异常由 item pipeline 抛出，用于停止处理 Item。详细内容请参考 :ref:`topics-item-pipeline`。

CloseSpider
-----------

.. exception:: CloseSpider(reason='cancelled')

    该异常由 spider 的回调函数（callback）抛出，用于暂停/停止 spider。支持的参数有：

    :param reason: 关闭的原因
    :type reason: str

样例::

    def parse_page(self, response):
        if 'Bandwidth exceeded' in response.body:
            raise CloseSpider('bandwidth_exceeded')

DontCloseSpider
---------------

.. exception:: DontCloseSpider

该异常由 :signal:`spider_idle` 信号处理器抛出，以防止 spider 被关闭。

IgnoreRequest
-------------

.. exception:: IgnoreRequest

该异常由调度器（Scheduler）或下载中间件抛出，用于声明忽略该请求。

NotConfigured
-------------

.. exception:: NotConfigured

该异常由某些组件抛出，用于声明其仍然保持关闭状态。这些组件包括：

 * Extensions
 * Item pipelines
 * Downloader middlewares
 * Spider middlewares

该异常必须由组件的方法  ``__init__`` 抛出。

NotSupported
------------

.. exception:: NotSupported

该异常用于声明一个不支持的特性。
