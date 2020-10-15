# Model One-to-One Relationships with Embedded Documents 一对一嵌套关系模型[¶](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/)

On this page

- [Overrivew](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#overview)
- [Embedded Document Pattern](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#embedded-document-pattern)
- [Subset Pattern](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#subset-pattern)

在本页面

- [概述](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#overview)
- [嵌套文档模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#embedded-document-pattern)
- [子集模式](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#subset-pattern)



## Overview [概述¶](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#overview)

This page describes a data model that uses [embedded](https://docs.mongodb.com/v4.2/core/data-model-design/#data-modeling-embedding) documents to describe a one-to-one relationship between connected data. Embedding connected data in a single document can reduce the number of read operations required to obtain data. In general, you should structure your schema so your application receives all of its required information in a single read operation.

这章节使用嵌套文档的模型描述了具有一对一关系的数据实体。把有关联的数据嵌套在单个文档中，可以减少读操作的次数。通常来说，如果你按嵌套文档模式来设计你的数据结构，那么在一次读取操作里，你的应用程序会接收文档所有的信息。



## Embedded Document Pattern 嵌套文档模式[¶](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#embedded-document-pattern)

Consider the following example that maps patron and address relationships. The example illustrates the advantage of embedding over referencing if you need to view one data entity in context of the other. In this one-to-one relationship between `patron` a nd `address` data, the `address` belongs to the `patron`.

考虑下面 patron 和 address 映射关系的示例。这个示例说明嵌套文档优于引用：你需要在一个数据实体的内部查看另一个数据实体的信息。数据实体 `patron` 和 数据实体`address` 的关系是一对一的，一个 `address` 数据实体属于一个 `patron` 数据实体。



In the normalized data model, the `address` document contains a reference to the `patron` document.

在标准化数据模型里，`address` 文档包含了 `patron`文档的引用。

```
// patron document
// patron 文档
{
   _id: "joe",
   name: "Joe Bookreader"
}

// address document
// address 文档
{
   patron_id: "joe", // reference to patron document // patron文档的引用
   street: "123 Fake Street",
   city: "Faketon",
   state: "MA",
   zip: "12345"
}
```

If the `address` data is frequently retrieved with the `name` information, then with referencing, your application needs to issue multiple queries to resolve the reference. The better data model would be to embed the `address`data in the `patron` data, as in the following document:

如果以引用的方式频繁读取`name` 和 `address` 的数据，那么你的应用程序需要查询多次才能获取信息。更好的方式应该把 `address` 实体嵌套到 `patron` 实体内，像下面这个示例。

```
{
   _id: "joe",
   name: "Joe Bookreader",
   address: {
              street: "123 Fake Street",
              city: "Faketon",
              state: "MA",
              zip: "12345"
            }
}
```

With the embedded data model, your application can retrieve the complete patron information with one query.

在嵌套文档模型里，你的应用程序查询一次就能获取 `patron` 的完整信息。



## Subset Pattern 子集模式[¶](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/#subset-pattern)

A potential problem with the [embedded document pattern](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-many-relationships-between-documents/#one-to-many-embedded-document-pattern) is that it can lead to large documents that contain fields that the application does not need. This unnecessary data can cause extra load on your server and slow down read operations. Instead, you can use the subset pattern to retrieve the subset of data which is accessed the most frequently in a single database call.

嵌套文档模型有一个潜在问题是：当文档包含了应用程序不需要的字段时，它会导致文档过大。这些冗余的数据会造成服务器的额外开销从而降低读的性能。相反，你可以把频繁被访问的数据子集放在单独的数据库中，以子集模式去方式去获取。



Consider an application that shows information on movies. The database contains a `movie` collection with the following schema:

考虑一个应用会呈现电影的信息。数据库包含了一个 `moive` 集合，`movie` 的模式如下。

```
{
  "_id": 1,
  "title": "The Arrival of a Train",
  "year": 1896,
  "runtime": 1,
  "released": ISODate("01-25-1896"),
  "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
  "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
  "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
  "lastupdated": ISODate("2015-08-15T10:06:53"),
  "type": "movie",
  "directors": [ "Auguste Lumière", "Louis Lumière" ],
  "imdb": {
    "rating": 7.3,
    "votes": 5043,
    "id": 12
  },
  "countries": [ "France" ],
  "genres": [ "Documentary", "Short" ],
  "tomatoes": {
    "viewer": {
      "rating": 3.7,
      "numReviews": 59
    },
    "lastUpdated": ISODate("2020-01-09T00:02:53")
  }
}
```

Currently, the `movie` collection contains several fields that the application does not need to show a simple overview of a movie, such as `fullplot` and rating information. Instead of storing all of the movie data in a single collection, you can split the collection into two collections:

目前，在展示一部电影的简介时，`movie` 集合包含了应用程序不需要的几个的字段，比如像 `fullplot` 和 `rating` 的值。并不是要把电影的所有的数据都存储在单个集合里，你可以把单个集合分离成两个集合：

- The `movie` collection contains basic information on a movie. This is the data that the application loads by default:

- `moive` 集合包含了一部电影的基本信息。应用程序会默认加载这个文档数据。

  ```
  // movie collection
  // movie 集合
  
  {
    "_id": 1,
    "title": "The Arrival of a Train",
    "year": 1896,
    "runtime": 1,
    "released": ISODate("1896-01-25"),
    "type": "movie",
    "directors": [ "Auguste Lumière", "Louis Lumière" ],
    "countries": [ "France" ],
    "genres": [ "Documentary", "Short" ],
  }
  ```

- The `movie_details` collection contains additional, less frequently-accessed data for each movie:

- `movie_details` 集合包含了每部电影额外的，较少访问的数据。

  ```
  // movie_details collection
  // movie_details 集合
  
  {
    "_id": 156,
    "movie_id": 1, // reference to the movie collection
    "poster": "http://ia.media-imdb.com/images/M/MV5BMjEyNDk5MDYzOV5BMl5BanBnXkFtZTgwNjIxMTEwMzE@._V1_SX300.jpg",
    "plot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, ...",
    "fullplot": "A group of people are standing in a straight line along the platform of a railway station, waiting for a train, which is seen coming at some distance. When the train stops at the platform, the line dissolves. The doors of the railway-cars open, and people on the platform help passengers to get off.",
    "lastupdated": ISODate("2015-08-15T10:06:53"),
    "imdb": {
      "rating": 7.3,
      "votes": 5043,
      "id": 12
    },
    "tomatoes": {
      "viewer": {
        "rating": 3.7,
        "numReviews": 59
      },
      "lastUpdated": ISODate("2020-01-29T00:02:53")
    }
  }
  ```

This method improves read performance because it requires the application to read less data to fulfill its most common request. The application can make an additional database call to fetch the less-frequently accessed data if needed.

这种方法提高了读取性能，因为它要求应用程序读取更少的数据来满足最常见的需求。如果需要那些较少被访问的数据，应用程序会调用其他的数据库。



> TIP
>
> When considering where to split your data, the most frequently-accessed portion of the data should go in the collection that the application loads first.
>
> 提示：
>
> 当考虑在哪里分离你的数据时，在文档中频繁被访问的部分应该被分离出来，因为它会被应用程序首先加载。





SEE ALSO

To learn how to use the subset pattern to model one-to-many relationships between collections, see [Model One-to-Many Relationships with Embedded Documents](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-many-relationships-between-documents/#data-modeling-example-one-to-many).

另请参阅

了解如何使用子集模式到一对多关系集合模型中，参阅 [Model One-to-Many Relationships with Embedded Documents](https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-many-relationships-between-documents/#data-modeling-example-one-to-many)





### Trade-Offs of the Subset Pattern 子集模式的权衡

Using smaller documents containing more frequently-accessed data reduces the overall size of the working set. These smaller documents result in improved read performance and make more memory available for the application.

使用包含频繁被访问数据的小文档会减少工作集的大小。这些较小的文档集会提升了读取性能，并为应用程序提供更多内存可用。



However, it is important to understand your application and the way it loads data. If you split your data into multiple collections improperly, your application will often need to make multiple trips to the database and rely on `JOIN` operations to retrieve all of the data that it needs.

然而，理解你的应用程序及其加载数据的方式是重要的。如果分离数据不当，你的应用程序会经常需要多次访问数据库和依赖 `JOIN` 操作才能获取应用程序需要的全部数据。



In addition, splitting your data into many small collections may increase required database maintenance, as it may become difficult to track what data is stored in which collection.

另外，你的数据被分离成很多个集合，会增加数据库的维护成本。因为它会使数据存储和数据查询变得困难。



原文链接：https://docs.mongodb.com/v4.2/tutorial/model-embedded-one-to-one-relationships-between-documents/

译者：朱俊豪

