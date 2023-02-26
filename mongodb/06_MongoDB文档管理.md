## MongoDB文档管理

> 在MongoDB 中文档是指多个键及其关联的值有序地放置在一起就是文档，其实指的就是数据，也是我们平时操作最多的部分。
>
> MongoDB中的文档的数据结构和JSON 基本一样。所有存储在集合中的数据都是 BSON 格式。
>
> BSON 是一种类似JSON的二进制形式的存储格式，是Binary JSÓN的简称。

### 1、插入文档

#### 单个文档

- insert函数

  语法：db.COLLECTION_NAME.insert(document)

  ```shell
  > use test
  switched to db test
  > db.auth("test", "AAAaaa123")
  1
  > db.coll1.insert({"name":"winn"})
  WriteResult({ "nInserted" : 1 })
  > db.coll1.find()
  { "_id" : ObjectId("62e13a38b550ddd37b6cb889"), "name" : "winn" }
  ```

- save函数

  语法：db.COLLECTION_NAME.save(document)

  ```shell
  > db.coll1.save({"name":"zhangsan"})
  WriteResult({ "nInserted" : 1 })
  > db.coll1.find()
  { "_id" : ObjectId("62e13a38b550ddd37b6cb889"), "name" : "winn" }
  { "_id" : ObjectId("62e13ac0b550ddd37b6cb88a"), "name" : "zhangsan" }
  ```

- insertOne函数

  Mongodb 3.2版本以后支持

  语法：db.COLLECTION_NAME.insertOne(document)

  ```shell
  > db.coll1.insertOne({"name":"lisi"})
  {
          "acknowledged" : true,
          "insertedId" : ObjectId("62e13b0eb550ddd37b6cb88b")
  }
  > db.coll1.find()
  { "_id" : ObjectId("62e13a38b550ddd37b6cb889"), "name" : "winn" }
  { "_id" : ObjectId("62e13ac0b550ddd37b6cb88a"), "name" : "zhangsan" }
  { "_id" : ObjectId("62e13b0eb550ddd37b6cb88b"), "name" : "lisi" }
  ```

#### 多个文档

- insert

  语法：db.COLLECTION_NAME.insert([{},{},{}...])

  ```shell
  > db.coll1.insert([{"name":"winn1"}, {"name": "winn2"}])
  BulkWriteResult({
          "writeErrors" : [ ],
          "writeConcernErrors" : [ ],
          "nInserted" : 2,
          "nUpserted" : 0,
          "nMatched" : 0,
          "nModified" : 0,
          "nRemoved" : 0,
          "upserted" : [ ]
  })
  > db.coll1.find()
  ...
  { "_id" : ObjectId("62e13c16b550ddd37b6cb88c"), "name" : "winn1" }
  { "_id" : ObjectId("62e13c16b550ddd37b6cb88d"), "name" : "winn2" }
  ```

- save

  语法：db.COLLECTION_NAME.save([{},{},{}...])

  ```shell
  > db.coll1.save([{"name":"zhangsan1"}, {"name": "zhangsan2"}])
  BulkWriteResult({
          "writeErrors" : [ ],
          "writeConcernErrors" : [ ],
          "nInserted" : 2,
          "nUpserted" : 0,
          "nMatched" : 0,
          "nModified" : 0,
          "nRemoved" : 0,
          "upserted" : [ ]
  })
  > db.coll1.find()
  ...
  { "_id" : ObjectId("62e13c7cb550ddd37b6cb88f"), "name" : "zhangsan1" }
  { "_id" : ObjectId("62e13c7cb550ddd37b6cb890"), "name" : "zhangsan2" }
  ```

- insertMany

  Mongodb 3.2版本以后支持

  语法：db.COLLECTION_NAME.insertMany([{},{},{}...])

  ```shell
  > db.coll1.insertMany([{"name":"lisi1"}, {"name": "lisi2"}])
  {
          "acknowledged" : true,
          "insertedIds" : [
                  ObjectId("62e13d29b550ddd37b6cb891"),
                  ObjectId("62e13d29b550ddd37b6cb892")
          ]
  }
  > db.coll1.find()
  ...
  { "_id" : ObjectId("62e13d29b550ddd37b6cb891"), "name" : "lisi1" }
  { "_id" : ObjectId("62e13d29b550ddd37b6cb892"), "name" : "lisi2" }
  ```

#### 通过变量插入

>Mongo Shell工具允许我们定义变量。所有的变量类型为var类型。也可忽略变量类型。变量中赋值符号后侧需要使用小括号来表示变量中的值。我们可以将变量作为任意插入文档的函数的参数。
>
>语法格式:变量名=({变量值})

```shell
# 单个插入
> name=({'name': 'winn3'})
{ "name" : "winn3" }
> db.coll1.insert(name)
WriteResult({ "nInserted" : 1 })
> db.coll1.find()
....
{ "_id" : ObjectId("62e13f7eb550ddd37b6cb893"), "name" : "winn3" }
# 多个插入
> names=([{'name': 'winn4'}, {'name': 'winn5'}])
[ { "name" : "winn4" }, { "name" : "winn5" } ]
> db.coll1.insert(names)
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 2,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.coll1.find()
...
{ "_id" : ObjectId("62e14012b550ddd37b6cb894"), "name" : "winn4" }
{ "_id" : ObjectId("62e14012b550ddd37b6cb895"), "name" : "winn5" }
```

### 2、更新文档

> MongoDB 通过 update函数与save函数来更新集合中的文档

#### update函数

update()函数用于更新已存在的文档。

语法格式:

db.集合名称.update(
	<query>,

​	<update>,

​	<upsert: boolean>

​	<multi: boolean>
)
**参数说明:**
query：update 的查询条件，类似 sql update 查询内 where 后面的。

update：update的对象和一些更新的操作符等，也可以理解为sql update 查询内 set 后面的。

upsert：可选，这个参数的意思是，如果不存在update的记录，是否插入这个document。true 为插入，默认是 false，不插入。

multi：可选，mongodb默认是false，只更新找到的第一条记录，如果这个参数为 true，就把按条件查询出来的多条记录全部更新。

>在MongoDB中的update是有两种更新方式，一种是覆盖更新，一种是表达式更新。
>
>覆盖更新：顾名思义，就是通过某条件，将新文档覆盖原有文档。
>
>表达式更新：这种更新方式是通过表达式来实现复杂更新操作，如：字段更新、数值计算、数组操作、字段名修改等。

插入一条新数据

```shell
> db.col.insert({
	title: 'MongoDB 教程', 
	description: 'MongoDB是一个 Nosql数据库', 
	by:'ciicsh',
	tags: ['mongodb', 'database', 'NoSQL'], 
	likes: 100
})
WriteResult({ "nInserted" : 1 })
> db.col.find()
{ "_id" : ObjectId("62e14c04b550ddd37b6cb896"), "title" : "MongoDB 教程", "description" : "MongoDB是一个 Nosql数据库", "by" : "ciicsh", "tags" : [ "mongodb", "database", "NoSQL" ], "likes" : 100 }

```

##### 覆盖更新

> 覆盖更新不能将multi参数设置为true。因为MongoDB要求multi参数为true只能使用表达式更新操作。

```shell
> db.col.update({"title" : "MongoDB 教程"}, { "name": "winn", "title" : "MongoDB 教程", "description" : "MongoDB是一个 Nosql数据库", "by" : "ciicsh", "likes"
 : 100 })
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.col.find()
{ "_id" : ObjectId("62e14c04b550ddd37b6cb896"), "name" : "winn", "title" : "MongoDB 教程", "description" : "MongoDB是一个 Nosql数据库", "by" : "ciicsh", "likes" : 100 }
```

##### 表达式更新

通过 update() 方法来更新标题

```shell
> db.col.update({"title": "MongoDB 教程"}, {&#36;set: {'title':'MongoDB'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.col.find()
{ "_id" : ObjectId("62e14c04b550ddd37b6cb896"), "name" : "winn", "title" : "MongoDB", "description" : "MongoDB是一个 Nosql数据库", "by" : "ciicsh", "likes" : 100 }
```

**更多案例:**

- 只更新第一条满足条件的记录:
  db.col.update( { "count": { &#36;gt:1}} ，{ &#36;set :{ "test2" : "OK"} } );
- 更新全部满足条件的记录:
  db.col.update({"count": { &#36;gt :3 } }，{ &#36;set:{ "test2":"OK"}},false,true );
- 如果没有符合条件的数据，则添加一条:
  db.col.update({"count": { &#36;gt : 4 } } ，{ &#36;set : { "test5" : "OK"} }, true, false );
- 如果没有符合条件的数据，则全部添加进去:
  db.colupdate({"count": { &#36;gt : 5 } } ，{ &#36;set:{ "test5" : "OK"} }, true, true );

新增一批数据为下面表达式更新做测试

```shell
> db.stu.insert([{'name':'zhangsan', 'age':20},{'name':'lisi', 'age':22},{'name':'wangwu','age':25}])
BulkWriteResult({
        "writeErrors" : [ ],
        "writeConcernErrors" : [ ],
        "nInserted" : 3,
        "nUpserted" : 0,
        "nMatched" : 0,
        "nModified" : 0,
        "nRemoved" : 0,
        "upserted" : [ ]
})
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }

```

###### &#36;inc

用法：{$inc:{field:value}}

作用：对一个数字字段的某个field增加 value

示例：将name为zhangsan的学生的age增加5

```shell
> db.stu.update({name:"zhangsan"},{$inc:{age:5}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "age" : 25 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
# -5相当于减去5
> db.stu.update({name:"zhangsan"},{$inc:{age:-5}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;set

用法：{$set:{field:value}}

作用：把文档中某个字段field设置为value，如果filed不存在，则增加新的filed并赋值为value

示例：把zhangsan的年龄设为23岁

```shell
> db.stu.update({name:"zhangsan"},{$set:{age:23}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "age" : 23 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;unset

用法：{$unset:{field:value}}

作用：删除某个字段field

示例：把zhangsan的年龄字段删除

```shell
# age:1 这个age后面的赋值为任何内容都可以
> db.stu.update({name:"zhangsan"},{$unset:{age:1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan" }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;push

用法：{$push:{field:value}}

作用：把value追加到field里，field必须是数组类型，如果field不存在，则会自动新增一个数组类型的field并且追加value值。

示例：给zhangsan添加别名"xiaozhang"

```shell
> db.stu.update({name:"zhangsan"},{$push: {"ailas":"xiaozhang"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ "xiaozhang" ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;addToSet

用法：{$addToSet:{field:value}}

作用：加一个值到数组内，而且只有当这个值在数组中不存在时才增加。

示例：往zhangsan的别名字段里添加两个别名A1、A2

```shell
> db.stu.update({name:"zhangsan"},{$addToSet:{"ailas":["A1","A2"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ "xiaozhang", [ "A1", "A2" ] ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

注意：此处加入的数据是一个数据为 A1和A2的 数组对象。并不是将两个数据依次加入到ailas数组中。

###### &#36;pop

用法：删除数组内第一个值:{&#36;pop:{field:-1}}、删除数组内最后一个值:{&#36;pop:{field:1}}

作用：用于删除数组内的一个值

示例：删除 zhangsan 记录中alias 字段中最后一个别名

```shell
> db.stu.update({name:"zhangsan"},{$pop:{"ailas":1}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ "xiaozhang" ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;pull

用法：{&#36;pull:{field:_value}}

作用：从数组field 内删除一个等于_value的值

示例：删除 zhangsan 记录中的别名xiaozhang

```shell
> db.stu.update({name:"zhangsan"},{$pull: {"ailas":"xiaozhang"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;pullAll

用法：{&#36;pullAll:value_array}

作用：用法同$pull一样，可以一次性删除数组内的多个值。

示例：删除zhangsan记录内的A1和A2别名

```shell
> db.stu.update({name:"zhangsan"},{$push: {"ailas":"A1"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.update({name:"zhangsan"},{$push: {"ailas":"A2"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ "A1", "A2" ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
> db.stu.update({name:"zhangsan"},{$pullAll: {"ailas":["A1","A2"]}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "name" : "zhangsan", "ailas" : [ ] }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

###### &#36;rename

用法：{&#36;rename:{old_field_name: new_field_name}}

作用：对字段进行重命名

示例：把zhangsan记录的name字段重命名为sname

```shell
> db.stu.update({name: "zhangsan"}, {$rename: {"name": "sname"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb897"), "ailas" : [ ], "sname" : "zhangsan" }
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 }
```

##### save函数

> save()函数的作用是保存文档，如果文档存在则覆盖，如果文档不存在则新增。
>
> save()函数对文档是否存在的唯一判断标准是"_id"系统唯一字段是否匹配。
>
> 所以使用save()函数实现更新操作，则必须提供"id"字段数据。

save()函数的语法是： db.集合名称.save(<document>)

>参数document代表要修改的文档内容，要求必须提供”id"字段数据。

使用save函数实现更新操作：

```shell
db.col.save({
	"id": ObjectId("5d0207e460ad10791be757d2")
	"name": "winn",
	....
})
```

#### 3、删除文档

> MongoDB是用过 remove()函数、deleteOne()函数、deleteMany()函数来删除集合中的文档。

##### remove函数

语法格式:

db.集合名称.remove(
	<query>,
	<justOne: boolean>

);

参数说明:

query：可选参数，要删除的文档条件，相当于 SQL 语法中的 where 子句作用。

justOne：可选参数，布尔类型，代表是否只删除第一个匹配条件满足的文档。默认值为false，代表删除全部满足匹配条件的文档。

> 注意：此方法已经过时，官方推荐使用deleteOne()和deleteMany()函数来实现删除操作。且remove()函数并不会真正的释放存储空间，需要使用db.repairDatabase()函数来释放存储空间。

###### remove - 删除全部

删除test集合中的全部文档：db.test.remove({});

###### remove -  条件删除

删除test集合中年龄为20的文档：db.test.remove({"age":  20});

删除test集合中年龄小于20，并且第一个匹配的文档：db.test.remove({"age":  {'$lt', 20}},  true);

```shell
大于符号：$gt、小于符号：$lt、大于等于：$gte、小于等于：$lte、等于：$eq、不等于：$neq
```

##### deleteOne函数

语法格式：db.集合名称.deleteOne({<query>});

参数解释：

query：要删除的文档条件，相当于SQL 语法中的where 子句作用。

删除stu集合中 name 字段为zhangsan 的第一个文档：

```shell
> db.stu.deleteOne({'sname':'zhangsan'});
{ "acknowledged" : true, "deletedCount" : 1 }
> db.stu.find()
{ "_id" : ObjectId("62e15773b550ddd37b6cb898"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62e15773b550ddd37b6cb899"), "name" : "wangwu", "age" : 25 } 
```

##### deleteMany函数

语法格式：db.集合名称.deleteMany({<query>});

参数解释:

query：要删除的文档条件，相当于 SQL 语法中的where 子句作用。

删除stu集合中字段大于10的所有文档: 

```shell
> db.stu.deleteMany({'age':{'$gt':10} });
{ "acknowledged" : true, "deletedCount" : 2 }
> db.stu.find()
> 
```
