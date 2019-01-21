---
layout: post
title: "实践MongoDB之三Java驱动"
date: 2018-04-27 10:12:13 +0800
categories: 数据库
tags: document-database driver nosql mongodb database java
---

[MongoDB Java Driver](http://mongodb.github.io/mongo-java-driver/)

mongodb-driver-core

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-core</artifactId>
        <version>3.6.3</version>
    </dependency>
```

## MongoDB Driver

`mongodb-driver-sync`是`3.7`版本新增的

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-sync</artifactId>
        <version>3.7.0-rc0</version>
    </dependency>
```

`mongodb-driver`是`3.0`版本新增的

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver</artifactId>
        <version>3.6.3</version>
    </dependency>
```

`mongo-java-driver` legacy uber-jar，集成了`mongodb-driver`、`mongodb-driver-core`和`bson`，符合OSGi bundle标准。

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongo-java-driver</artifactId>
        <version>3.6.3</version>
    </dependency>
```

### 样例

```java
// make a connection
MongoClient mongoClient = new MongoClient( "localhost" , 27017 );
// access a database
MongoDatabase database = mongoClient.getDatabase("mydb");
// access a collection
MongoCollection<Document> collection = database.getCollection("test");
// create a document
Document doc = new Document("name", "MongoDB")
                .append("type", "database")
                .append("count", 1)
                .append("versions", Arrays.asList("v3.2", "v3.0", "v2.6"))
                .append("info", new Document("x", 203).append("y", 102));
// insert a document
collection.insertOne(doc);
// create an ascending index on the name field
collection.createIndex(new Document("name", 1));
```

### POJO样例

```java
// Creating a Custom CodecRegistry
CodecRegistry pojoCodecRegistry = fromRegistries(MongoClient.getDefaultCodecRegistry(),
                fromProviders(PojoCodecProvider.builder().automatic(true).build()));
// using the CodecRegistry
 MongoClient mongoClient = new MongoClient("localhost", MongoClientOptions.builder().codecRegistry(pojoCodecRegistry).build());
```

## MongoDB Async Driver

`mongodb-driver-async`是`3.0`版本新增的

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-async</artifactId>
        <version>3.6.3</version>
    </dependency>
```



```java
SingleResultCallback<Document> callbackPrintDocuments = new SingleResultCallback<Document>() {
   @Override
   public void onResult(final Document document, final Throwable t) {
       System.out.println(document.toJson());
   }
};

SingleResultCallback<Void> callbackWhenFinished = new SingleResultCallback<Void>() {
    @Override
    public void onResult(final Void result, final Throwable t) {
        System.out.println("Operation Finished!");
    }
};

Block<Document> printDocumentBlock = new Block<Document>() {
    @Override
    public void apply(final Document document) {
        System.out.println(document.toJson());
    }
};
```

### 样例

```java
MongoClient mongoClient = MongoClients.create();
collection.insertOne(doc, new SingleResultCallback<Void>() {
    @Override
    public void onResult(final Void result, final Throwable t) {
        System.out.println("Inserted!");
    }
});
```



### MongoDB RxJava Driver

[MongoDB RxJava Driver](http://mongodb.github.io/mongo-java-driver-rx/)

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-rx</artifactId>
        <version>1.5.0</version>
    </dependency>
```



### MongoDB Reactive Streams Java Driver

[MongoDB Reactive Streams Java Driver](http://mongodb.github.io/mongo-java-driver-reactivestreams/)

```xml
<dependency>
        <groupId>org.mongodb</groupId>
        <artifactId>mongodb-driver-reactivestreams</artifactId>
        <version>1.7.1</version>
    </dependency>
```



```java
// insert a document
collection.insertOne(doc).subscribe(new OperationSubscriber<Success>());
```

