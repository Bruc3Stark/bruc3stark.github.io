---
layout: mypost
title: MongoDB 学习
categories: [数据库]
---

### 安装

windows 系统下的安装比较简单，图省事直接下载 msi 格式的安装包就行了，安装向导会自动把它安装成服务的,推荐这样安装

也可以手动安装成服务，以管理员模式启动 CMD，切换到 MongoDB 的安装目录，并执行命令`mongod --dbpath "D:\mongodb\data\db" --logpath "D:\mongodb\logs\log.txt" --install -serviceName "MongoDB"`

执行手动安装命令之前要先把 data/db 和 logs/logs.txt 等文件夹和文件创建好，不然会报错。

下面是自动安装的截图，最后一步 MongoDB Compass 的勾要去掉，不然要等待下载好久。推荐自己去下载 MongoDB Compass 单独安装

![01](m-01.png)

![02](m-02.png)

![03](m-03.png)

通过`net start MongoDB`启动服务（管理员权限），一般安装好会自动启动的

启动成功浏览器打开`http://127.0.0.1:27017/`会看到效果

### 使用

进入到安装目录下面的 bin 目录，里面有 MongoDB 的工具

为了方便可以把该目录加到环境变量中，下面是两个常用的工具

mongo.exe ，MongoDB 命令行工具，可以执行增删改查等命令

mongodump.exe，备份工具

安装目录下面的 data 目录就是数据库文件了。通过命令行添加数据库和集合会发现文件的变动，不能辨识

![04](m-04.png)

### 介绍

MongoDB 的数据格式和传统的关系型数据库有点差异，不然怎么会叫 NoSQL 呢

传统的数据库，表，记录对应 MongoDB 中的数据库，Collection，Document。主要差异就在 Document 上，它是没有固定结构的一种记录，和 json 很像，同时 MongoDB 的一些操作语法格式也和 json 的格式很像

### 命令行

进入到 MongoDB 的安装目录，执行 mongo 进入 shell 界面

show dbs 显示所有的数据库，及其容量

show collections 显示所有的集合

use [name] 切换数据库，没有也可以切换，在插入数据后会被创建

db.dropDatabase() 删除数据库

db.[name].drop() 删除集合

db.createCollection([name],[options]) 创建集合

db.col.find().pretty() 格式化结果

db.[name].find(query, projection) 查询，为空列出集合内所有文档

db.[name].insert([json]) 插入文档

db.[name].update() 更新文档

db.[name].save() 替换文档

db.[name].remove() 删除文档

### 图形化界面

有好多款，看了下常用的 Navicate 也有 MongoDB 版本的，不过是收费的

其他的也有好多，不过界面都不怎么漂亮

我是用的是 MongoDB 官方出的 MongoDB Compass，界面挺漂亮的，关键还免费，唯一的缺点是用 Electron 写的，运行启动比较慢

![05](m-05.png)

### 在 Java 中使用 MongoDB

先需要驱动程序，jar 包或者 maven.....

这里使用的是 mongo-java-driver-3.9.1.jar

### 连接数据库

如果有设置用户用户名和密码

```java
MongoCredential credential = MongoCredential.createCredential("用户名", "数据库名称", "密码".toCharArray());
MongoClient mongoClient = new MongoClient(new ServerAddress("IP", 端口号), Arrays.asList(credential));
```

没有设置用户名和密码，直接连接

```java
private static MongoClient mongoClient;
private static MongoDatabase mongoDatabase;

public static void main(String[] args) {
    try {
        mongoClient = new MongoClient("127.0.0.1", 27017);
        // 数据库
        mongoDatabase = mongoClient.getDatabase("test");
        // collection
        MongoCollection<Document> collection = mongoDatabase.getCollection("col");
        System.out.println(collection.countDocuments());
    } catch (Exception e) {
        System.err.println(e.getClass().getName() + ": " + e.getMessage());
    } finally {
        // 释放资源
        if (mongoClient != null) {
            mongoClient.close();
        }
    }
}
```

### 插入

```java
Document doc = new Document();
doc.append("title", "标题");
doc.append("description", "描述");
doc.append("likes", 20);

collection.insertOne(doc);
```

### 查询

MongoDB 类似 json 的查询语法，可以使用 Document 嵌套出相同的格式

当然也提供了 Filters 工具来简化操作，见下面的删除操作

```java
// 所有document
FindIterable<Document> docs = collection.find();
// 去除_id,description列
docs = docs.projection(new Document("_id", 0).append("description", 0));
// 根据likes升序排列
docs = docs.sort(new Document("likes", 1));

// 遍历结果，forEach已过时，不推荐使用
MongoCursor<Document> cursor = docs.iterator();
try {
    while (cursor.hasNext()) {
        System.out.println(cursor.next().toJson());
    }
} finally {
    cursor.close();
}
```

### 删除

删除 likes 数目小于 50 的 Document

```java
// collection.deleteOne(new Document("likes", new Document("$lt", 50)));
collection.deleteOne(Filters.lt("likes", 50));
```

### 修改

```java
// 修改一条
collection.updateOne(Filters.eq("title", "标题"), new Document("$set", new Document("title", "新标题")));

// 修改多条，所有点赞数目+1
UpdateResult updateResult = collection.updateMany(new Document(), new Document("$inc", new Document("likes", 1)));
System.out.println(updateResult.getModifiedCount());

```
