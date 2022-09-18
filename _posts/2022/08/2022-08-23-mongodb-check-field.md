---
layout: post
title:  如何在MongoDB中检查字段的存在？
tagline: by feng
categories: java
tags: 
    - feng
---

大家好，我是指北君。

在本文中，我们来看看如何在 MongoDB 中检查字段是否存在。

首先，我们创建一个简单的Mongo数据库, 然后放入一些假数据，以便在以后的例子中使用。之后，我们将展示如何在本地 Mongo 查询以及Java中检查字段是否存在。
<!--more-->
### 示例配置

在我们开始检查字段是否存在之前，我们需要一个现有的数据库、集合，以及供以后使用的假数据。我们将使用Mongo shell来实现。

首先，让我们把Mongo shell上下文切换到一个存在的数据库。

```bash
use javanorth
```

值得指出的是，MongoDB只在你第一次在该数据库中存储数据时创建数据库。我们将在`users`集合中插入一个用户。

```bash
db.users.insert({name: "java", surname: "north" })
```

现在我们已经做好了一些准备工作，接下去就讲讲如何检查字段是否存在。

### 在Mongo Shell中检查字段是否存在

有时我们需要通过基本的查询来检查特定字段的存在，例如在Mongo Shell或其他数据库控制台。幸运的是，Mongo提供了一个特殊的查询操作符，`$exists`，用于该目的。

```bash
db.users.find({ 'name' : { '$exists' : true }})
```

我们使用一个标准的`find` Mongo方法，在这个方法中，我们指定我们要寻找的字段，并使用`$exists`查询操作符。如果`name`字段在`users`集合中存在，所有包含该字段的记录都将被返回。

```bash
[
  {
    "_id": {"$oid": "6115ad91c4999031f8e6f582"},
    "name": "java",
    "surname": "north"
  }
]
```

如果该字段丢失，我们将得到一个空的结果。

### 在Java中检查字段的存在性

在我们研究在Java中检查字段是否存在的可能方法之前，让我们为我们的项目添加必要的[Mongo driver](https://search.maven.org/search?q=a:mongo-java-driver%20AND%20g:org.mongodb)。这里是Maven的依赖性。

```xml
<dependency>
    <groupId>org.mongodb</groupId>
    <artifactId>mongo-java-driver</artifactId>
    <version>3.12.10</version>
</dependency>
```

这里是Gradle的版本。

```bash
implementation group: 'org.mongodb', name: 'mongo-java-driver', version: '3.12.10'
```

最后，让我们连接到`existence`数据库和`users`集合。

```java
MongoClient mongoClient = new MongoClient();
MongoDatabase db = mongoClient.getDatabase("existence");
MongoCollection<Document> collection = db.getCollection("users");
```

### 使用过滤器

The`com.mongodb.client.model.Filters`是Mongo依赖的一个util类，包含了很多有用的方法。我们将在我们的例子中使用`exists()`方法。

```java
Document nameDoc = collection.find(Filters.exists("name")).first();
assertNotNull(nameDoc);
assertFalse(nameDoc.isEmpty());
```

首先，我们尝试从`users`集合中寻找元素，并得到第一个找到的元素。如果指定的字段存在，我们得到一个`nameDoc`文档作为响应。它不是空的，也不是空的。

现在，让我们看看当我们试图找到一个不存在的字段时会发生什么。

```java
Document nameDoc = collection.find(Filters.exists("non_existing")).first();
assertNull(nameDoc);
```

如果没有找到元素，我们会得到一个 null 的Document作为响应。

### 使用 Document 查询

`com.mongodb.client.model.Filters`类并不是检查字段存在的唯一方法。我们可以使用`com.mongodb.BasicDBObject:`的一个实例。

```java
Document query = new Document("name", new BasicDBObject("$exists", true));
Document doc = collection.find(query).first();
assertNotNull(doc);
assertFalse(doc.isEmpty());
```

其行为与前面的例子相同。如果元素被找到了，我们会收到一个非`null`的`Document`，它是空的。

当我们试图找到一个不存在的字段时，代码的行为也是一样的。

```java
Document query = new Document("non_existing", new BasicDBObject("$exists", true));
Document doc = collection.find(query).first();
assertNull(doc);
```

如果没有找到任何元素，我们会得到一个null 的 Document 作为响应。

### 总结

在这篇文章中，我们讨论了如何在MongoDB中检查字段是否存在。首先，我们展示了如何创建一个Mongo数据库、集合，以及如何插入假数据。然后，我们解释了如何使用一个基本的查询来检查一个字段在Mongo shell中是否存在。最后，我们解释了如何使用`com.mongodb.client.model.Filters`和`Document`查询方法来检查字段的存在。
