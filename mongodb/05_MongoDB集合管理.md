## MongoDB集合管理

>MongoDB中的集合是一组文档的集，相当于关系型数据库中的表。

### 1、创建集合

MongoDB使用db.createCollection()函数来创建集合。语法格式:db.createCollection(name, options)。

 name：要创建的集合名称。

options:可选参数，指定有关内存大小及索引的选项。

**options 可以是如下参数：**

| 字段        | 类型 | 描述                                                         |
| ----------- | ---- | ------------------------------------------------------------ |
| capped      | 布尔 | （可选）如果为true，则创建固定集合。<br />固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。<br/>**当该值为true 时，必须指定 size参数。** |
| autoindexid | 布尔 | 布尔（可选）如为true，自动在id字段创建索引。默认为false。    |
| size        | 数值 | （可选）为固定集合指定一个内存储存的最大值。超出自动覆盖最早的文档。（以字节为单位）。<br />**如果capped为true，也需要指定该字段。** |
| max         | 数值 | （可选）指定固定集合中包含文档的最大数量。超出自动覆盖最早的文档。 |

> 在插入文档时，MongoDB首先检查固定集合size字段，然后检查max字段。

```shell
> use  admin
switched to db admin
> db.auth("winn", "123123")
1
> use test
> show collections
> db.createCollection("coll1")
{ "ok" : 1 }
> show collections
coll1
> db.createCollection('coll2', {'capped': true, 'size': 2000000, 'max': 10000})
{ "ok" : 1 }
```

**使用默认集合**

在MongoDB中，我们也可以不用创建集合，当我们插入一些数据时，会自动创建集合，并且会使用文档管理命令中的集合名称作为集合的名称。

```shell
>db.createUser({'user':'test', 'pwd':'AAAaaa123','roles':[{'role':'readWrite', 'db':'test'}]})
# 退出后重新登录客户端
> use  test
switched to db test
> db.auth("test", "AAAaaa123")
1
> db.coll3.insert({"abc": "456"})
WriteResult({ "nInserted" : 1 })
> show collections
coll1
coll2
coll3
```

### 2、查看集合

- show collections

  ```shell
  > show collections
  coll1
  coll2
  coll3
  ```

- show  tables

  ```shell
  > show  tables
  coll1
  coll2
  coll3
  ```

- 查看集合详情

  > 语法：db.集合名称.stats()

  ```shell
  > db.coll2.stats()
  {
          "ns" : "test.coll2",
          "size" : 0,
          "count" : 0,
          "storageSize" : 4096,
          "capped" : true,
          "max" : 10000,
          "maxSize" : 2000128,
          "sleepCount" : 0,
          "sleepMS" : 0,
          ....
  ```

### 3、删除集合

如果我们要删除集合，需要先切换到需要删除集合所在的数据库，使用drop()函数删除集合即可。

删除集合的语法格式为：db.集合名称drop()

```shell
> db.coll2.drop()
true
> show tables
coll1
coll3
```

