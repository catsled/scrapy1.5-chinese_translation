.. _docs-intro-item-pipeline:

==========================
Item Pipeline
==========================

Spider 抓取的 Item 会被发送到 Item Pipeline，Pipeline 会使用一些按顺序执行的组件处理 Item 。

每个 item pipeline 组件(有时称之为“Item Pipeline”)都是一个 Python 类，这个类有一些简单的方法。组件接收到 Item 后对其进行处理，然后决定 Item 是传递给下一个 pipeline，还是直接丢弃。

典型应用：

 - 清洗 HTML 数据
 - 验证爬取的数据(检查item是否包含你抓取的字段)
 - 查重(丢弃)
 - 将爬取的数据保存到数据库中


编写 item pipeline
==========================
每个 item pipiline 组件都是一个实现以下方法的Python类： 

.. method:: process_item(self, item, spider)

    每个 item pipeline 组件都会调用这个方法。 :meth:`process_item`
    必须返回字典形式的数据, 返回一个 :class:`~scrapy.item.Item` 
    (或者一个可以继承的) 对象, 返回一个 `Twisted Deferred`_ 或者抛出一个
    :exc:`~scrapy.exceptions.DropItem` 异常。丢弃的 item 不会被下一个 pipeline 
    组件处理。

    .. _`Twist Deferred`: https://twistedmatrix.com/documents/current/core/howto/defer.html 

    :param item: 爬取的 item
    :type item: :class:`~scrapy.item.Item` 对象或字典

    :param spider: 爬取 item 的爬虫
    :type spider: :class:`~scrapy.spiders.Spider` 对象

 


其他方法：

.. method:: open_spider(self, spider)

    启动爬虫的时候调用。

    :param spider: 启动的爬虫
    :type spider: :class:`~scrapy.spiders.Spider` 对象

.. method:: close_spider(self, spider)

    停止爬虫的时候调用。
    :param spider: 停止的爬虫
    :type spider: :class:`~scrapy.spiders.Spider` 对象


.. method:: from_crawler(cls, crawler)

    如果存在, 组件会用这个方法从一个 :class:`~scrapy.crawler.Crawler` 
    类中创建一个 pipeline 实例，而且必须返回一个新的 pipeline 实例。Crawler 对象
    为所有 Scrapy 核心组件提供入口，比如设置和信号组件。 pipeline 用这种方法与
    核心组件通信并且把它的功能挂载到 Scrapy 上。

    :param crawler: 使用本组件的 crawler
    :type crawler: :class:`~scrapy.crawler.Crawler` 对象
    


Item pipeline 实例
==========================

删除没有 price 属性的 item 
^^^^^^^^^^^^^^^^^^^^^^^^^^

假设有一个 pipeline ，现在检查不包括 VAT （ ``price_excludes_vat`` 属性）的 item 是否存在 ``price`` 属性，然后删除没有 price 属性的 item。::

    from scrapy.exceptions import DropItem

    class PricePipeline(object):

        vat_factor = 1.15

        def process_item(self, item, spider):
            if item['price']:
                if item['price_excludes_vat']:
                    item['price'] = item['price'] * self.vat_factor
                return item
            else:
                raise DropItem("Missing price in %s" % item)

item 写入 json 文件
^^^^^^^^^^^^^^^^^^^^^^^^^^


下面的 pipeline 把爬取的 item （所有爬虫中爬取的）存到一个 ``items.jl`` 文件中，每行都使用 json 格式序列化一个item。 ::

    import json

    class JsonWriterPipeline(object):

        def open_spider(self, spider):
            self.file = open('items.jl', 'w')

        def close_spider(self, spider):
            self.file.close()

        def process_item(self, item, spider):
            line = json.dumps(dict(item)) + "\n"
            self.file.write(line)
            return item


.. Note:: 
    
    例子中类 JsonWriterPipeline 的目的只是为了介绍怎么编写 item pipelines 。如果你真的想把爬取到的 item 存到 json 文件中，你应该使用  `Feed exports`_  。

.. _`Feed exports`: https://docs.scrapy.org/en/latest/topics/feed-exports.html#topics-feed-exports

item 存到 MongoDB 
^^^^^^^^^^^^^^^^^^^^^^^^^^
下面的例子使用 `pymongo`_  把 item 存到 `MongoDB`_ 中。不一样的是，在 Scrapy 中 MongoDB 集和在 item 类后命名。

.. _`pymongo`: https://api.mongodb.com/python/current/
.. _`MongoDB`: https://www.mongodb.com/

下面的例子展示了怎样使用 `~from_crawel` 方法以及如何清洗数据：::
    
    import pymongo
    
    class MongoPipeline(object):
    
        collection_name = 'scrapy_items'
    
        def __init__(self, mongo_uri, mongo_db):
            self.mongo_uri = mongo_uri
            self.mongo_db = mongo_db
    
        @classmethod
        def from_crawler(cls, crawler):
            return cls(
                mongo_uri=crawler.settings.get('MONGO_URI'),
                mongo_db=crawler.settings.get('MONGO_DATABASE', 'items')
            )

        def open_spider(self, spider):
            self.client = pymongo.MongoClient(self.mongo_uri)
            self.db = self.client[self.mongo_db]
    
        def close_spider(self, spider):
            self.client.close()
    
        def process_item(self, item, spider):
            self.db[self.collection_name].insert_one(dict(item))
            return item

生成 item 快照
^^^^^^^^^^^^^^^^^^^^^^^^^^

下面的例子用 `~process_item()` 方法返回 `Deferred`_ 。用 `Splash`_ 渲染 item url 快照。然后 pipeline 请求本地运行的 `Splash`_ 实例。请求下载后，Deferred 开始运行，把 item 保存到文件中，同时为 item 增加一个保存文件名的字段。::

    import scrapy
    import hashlib
    from urllib.parse import quote
    
    
    class ScreenshotPipeline(object):
        """Pipeline that uses Splash to render screenshot of
        every Scrapy item."""
        
        SPLASH_URL = "http://localhost:8050/render.png?url={}"

        def process_item(self, item, spider):
            encoded_item_url = quote(item["url"])
            screenshot_url = self.SPLASH_URL.format(encoded_item_url)
            request = scrapy.Request(screenshot_url)
            dfd = spider.crawler.engine.download(request, spider)
            dfd.addBoth(self.return_item, item)
            return dfd

        def return_item(self, response, item):
            if response.status != 200:
                # Error happened, return item.
                return item

            # Save screenshot to file, filename will be hash of url.
            url = item["url"]
            url_hash = hashlib.md5(url.encode("utf8")).hexdigest()
            filename = "{}.png".format(url_hash)
            with open(filename, "wb") as f:
                f.write(response.body)

            # Store filename in item.
            item["screenshot_filename"] = filename
            return item

.. _`Deferred`: https://twistedmatrix.com/documents/current/core/howto/defer.html
.. _`Splash`: https://splash.readthedocs.io/en/stable/


过滤重复数据
^^^^^^^^^^^^^^^^^^^^^^^^^^

过滤器会查找重复的 item ，然后删除这些已经处理过的 item 。我们抓取的 item 的 id 值是唯一的，但是 spider 会返回多个 id  值一样的重复数据： ::

    from scrapy.exceptions import DropItem

    class DuplicatesPipeline(object):
    
        def __init__(self):
            self.ids_seen = set()
    
        def process_item(self, item, spider):
            if item['id'] in self.ids_seen:
                raise DropItem("Duplicate item found: %s" % item)
            else:
                self.ids_seen.add(item['id'])
                return item


激活 item pipeline 组件
==========================


你可以在 Scrapy setting 的 `~ITEM_PIPELINE` 中加入你要激活的 item pipeline 的类名，如下： ::
    
    ITEM_PIPELINES = {
        'myproject.pipelines.PricePipeline': 300,
        'myproject.pipelines.JsonWriterPipeline': 800,
    }

上面设置中类名后面的数字决定了相应的 piepline 运行的顺序了： 所有的 item 会根据这个数字从小到大传递给它相应的 pipeline 。它们的取值一般在 0-1000 之间。









