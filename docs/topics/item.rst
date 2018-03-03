.. _docs-topics-items:

======
Items
======

.. module:: scrapy.item
   :synopsis: Item and Field classes

爬取的主要目标是从非结构化的数据源（通常是网页）中提取结构化数据。Scrapy可以将提取的数据以python字典形式返回。
这种方式虽然比较方便而且是我们所熟悉，但是Python字典缺乏结构性，很容易出现字段拼写错误或者是返回的数据不一致情况，特别是在有大量爬虫的项目中。

Scrapy提供了一个定义公共输出数据格式的 :class:`Item` 类。:class:`Item` 对象是一种简单的容器，用来保存爬取到的数据。
Item提供了一个用于声明可用字段的 `dictionary-like`_ API ，并提供了方便的语法。

许多Scrapy组件使用Items提供的额外信息:
导出器查看已声明的字段以此弄清楚需要导出哪些字段，序列化可以使用Item字段中的metadata进行定制，
:mod:`trackref` Item实例可以用来寻找内存泄露 (查看 :ref:`dosc-topics-leaks-trackrefs`) 。

.. _dictionary-like: https://docs.python.org/2/library/stdtypes.html#dict

.. _topics-items-declaring:

声明Item
=============

Item使用简单的类定义语法以及 :class:`Field`对象来声明。下面是一个示例: ::

    import scrapy

    class Product(scrapy.Item):
        name = scrapy.Field()
        price = scrapy.Field()
        stock = scrapy.Field()
        last_updated = scrapy.Field(serializer=str)

.. note:: 熟悉 `Django`_ 的朋友一定会注意到Item定义方式与 `Django Models`_ 很类似，
   不同的在于Scrapy Item更为简单，没有不同字段类型的概念。

.. _Django: https://www.djangoproject.com/
.. _Django Models: https://docs.djangoproject.com/en/dev/topics/db/models/

.. _topics-items-fields

Item 字段
==========

:class:`Field`对象指明了每个字段的元数据。例如，在上面的示例中``last_updated``指明了该字段的序列化函数。

您可以为每个字段指定任何类型的元数据。:class:`Field`对象对接受值没有限制。出于这个原因，无法提供所有可用的元数据的键参考列表。
在:class:`Field`对象中定义的每个键都可以被不同的组件使用，只有这些使用键组件才知道键的存在。您也可以根据自己的需求定义和使用任何其他:class:`Filed`键。
设置:class:`Field`对象的主要目的是提供一种方法来在一个地方定义所有的字段元数据。

通常情况下，依赖每个字段的组件是使用了特定的字段键来配置该行为。您必须参考其文档以查看每个组件使用的元数据键。

需要注意的是，用来声明item的 :class:`Field` 对象并没有被赋值为class属性。不过可以通过 :attr:`Item.filed` 属性进行访问。


使用Items
=============

下面使用 ``Product`` item :ref:`declared above <topics-items-declaring>` 来演示一些items执行常见任务示例。
你会注意到API与 `dict API`_ 非常相似。


创建Item
-----------

::

    >>> product = Product(name='Desktop PC', price=1000)
    >>> print product
    Product(name='Desktop PC', price=1000)

获取字段值
------------

::

    >>> product['name']
    Desktop PC
    >>> product.get('name')
    Desktop PC

    >>> product['price']
    1000

    >>> product['last_updated']
    Traceback (most recent call last):
        ...
    KeyError: 'last_updated'

    >>> product.get('last_updated', 'not set')
    not set

    >>> product['lala'] # getting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'lala'

    >>> product.get('lala', 'unknown field')
    'unknown field'

    >>> 'name' in product  # is name field populated?
    True

    >>> 'last_updated' in product  # is last_updated populated?
    False

    >>> 'last_updated' in product.fields  # is last_updated a declared field?
    True

    >>> 'lala' in product.fields  # is lala a declared field?
    False

设置字段值
------------

::

    >>> product['last_updated'] = 'today'
    >>> product['last_updated']
    today

    >>> product['lala'] = 'test' # setting unknown field
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'


访问所有填充的值
-----------------

如果要访问所有填充值，只需使用典型的`字典API`_::

    >>> product.keys()
    ['price', 'name']

    >>> product.items()
    [('price', 1000), ('name', 'Desktop PC')]

.. _字典API: https://docs.python.org/2/library/stdtypes.html#dict

其他常见的任务
---------------

复制item::

    >>> product2 = Product(product)
    >>> print product2
    Product(name='Desktop PC', price=1000)

    >>> product3 = product2.copy()
    >>> print product3
    Product(name='Desktop PC', price=1000)

根据item创建字典::

    >>> dict(product) # create a dict from all populated values
    {'price': 1000, 'name': 'Desktop PC'}

根据字典创建item::

    >>> Product({'name': 'Laptop PC', 'price': 1500})
    Product(price=1500, name='Laptop PC')

    >>> Product({'name': 'Laptop PC', 'lala': 1500}) # warning: unknown field in dict
    Traceback (most recent call last):
        ...
    KeyError: 'Product does not support field: lala'

扩展items
===========

您可以通过继承item基类来扩展item(以添加更多字段或更改某些字段的某些元数据）。

例如::

    class DiscountedProduct(Product):
        discount_percent = scrapy.Field(serializer=str)
        discount_expiration_date = scrapy.Field()


您也可以通过使用原字段的元数据，添加新的值或修改原来的值来扩展字段的元数据::

    class SpecificProduct(Product):
        name = scrapy.Field(Product.fields['name'], serializer=my_serializer)

上面示例保留了原来的元数据值，同时增添（或者是覆盖）了``name``字段的``serializer``


Item 对象
==========

.. class:: scrapy.item.Item([arg])

    根据给定的参数中返回一个可选初始化的item。

    item复制了标准的字典API，包括其构造函数。Item提供的唯一附加属性是：

    .. attribute:: fields

        一个包含这个Item的*所有声明字段*的字典，不仅仅是包含那些填充的字段。键是字段名称，值是在:ref:`Item`声明中使用的 :class:`Field`对象。

.. _dict API: https://docs.python.org/2/library/stdtypes.html#dict
.. _Item声明:

字段对象
==========

.. class:: scrapy.item.Field([arg])

    :class:`Field`类仅仅是内置`dict`_类的一个别名，并没有提供额外的方法或者属性。换句话说，:class:`Field`对象是普通的Python字段，一个单独的类，用于支持基于类属性的项目声明语法。

.. _dict: https://docs.python.org/2/library/stdtypes.html#dict
