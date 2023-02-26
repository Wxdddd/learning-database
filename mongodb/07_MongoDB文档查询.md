## 查询文档

MongoDB是通过findOne() 和 find() 函数来实现文档查询的。

### findOne 函数

findOne函数用于查询集合中的一个文档。语法如下: db.集合名称.findOne({<query>},{<projection>});

**参数解释：**

query：可选，代表查询条件。

projection：可选，代表查询结果的投影字段名。即查询结果需要返回哪些字段或不需要返回哪些字段。

**新增一批数据做测试**

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
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }
```

测试查询

```shell
> db.stu.find()
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }
> db.stu.findOne()
{
        "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"),
        "name" : "zhangsan",
        "age" : 20
}
> db.stu.findOne({'name':'wangwu'})
{
        "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"),
        "name" : "wangwu",
        "age" : 25
}
# 返回结果不包含_id值
> db.stu.findOne({'name':'wangwu'}, {_id:0})
{ "name" : "wangwu", "age" : 25 }
# 返回结果只显示_id值
> db.stu.findOne({'name':'wangwu'}, {_id:1})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2") }
# 返回结果不包含_id值，只显示name字段
> db.stu.findOne({'name':'wangwu'}, {_id:0, name:1})
{ "name" : "wangwu" }
# 默认情况下_id一定会显示
```

> 注意：在projection中不能使用{name:0,age:1}这种语法格式，这是错误的语法。
>
>  projection只能定义要返回的字段或不返回的字段。
>
> _id字段是MongoDB维护的字段，是唯一可以在projection 中独立使用的。如：{ _id:0,'name':1, 'age':1}

### find函数

find函数用于查询集合中的若干文档。

**语法**： db.stu.find({<query>,<projection>});

**参数解释**：
query：可选，代表查询条件。
projection：可选，代表查询结果的投影字段名。即查询结果需要返回哪些字段或不需要返回哪些字段。

参数用法与findOne一致

### pretty 函数

pretty函数用于格式化find函数查询结果。让查询结果更易查看。findOne函数自动附带格式化查询结果的能力。

语法**：

db.stu.find0.pretty();

### 单条件逻辑运算符

如果你熟悉常规的SQL 数据，通过下表可以更好的理解MongDB 的条件语句查询:

| 操作       | 格式                                    | 范例                                                         | RDBMS 中的类似语句 |
| ---------- | --------------------------------------- | ------------------------------------------------------------ | ------------------ |
| 等于       | {key: value}<br />或{key: {$eq: value}} | db.col.find({by: "winn")).pretty()<br />db.col.find({by: {&#36;eq:"winn"}}) | where by = 'winn'  |
| 小于       | {key: {$lt: value}}                     | db.col.find({"age":{&#36;It:50}}).pretty()                   | where age < 50     |
| 小于或等于 | {key: {$lte: value}}                    | db.col.find({"age":{&#36;Ite:50}}).pretty()                  | where age <= 50    |
| 大于       | {key: {$gt: value}}                     | db.col.find({"age":{&#36;gt:50}}).pretty()                   | where age > 50     |
| 大于或等于 | {key: {$gte: value}}                    | db.col.find({"age":{&#36;gte:50}}).pretty()                  | where age >= 50    |
| 不等于     | {key: {$ne: value}}                     | db.col.find({"age":{&#36;ne:50}}).pretty()                   | where age != 50    |

### 多条件逻辑运算符

#### And

MongoDB 的 find 和 findOne函数可以传入多个键(key)，每个键(key)以逗号隔开即常规 SQL 的 AND 条件。

查询 stu 集合中 name 字段为lisi，且 age 字段大于20的文档: 

```shell
> db.stu.find({'name':'lisi', 'age': {'$gt':20}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
```

#### Or

MongoDB OR 条件语句使用了关键字 $or。

语法格式如下: 

db.集合名称.find({$or: [

​	{key1: value1}, {key2:value2}

]})

```shell
> db.stu.find({$or: [{name: 'wangwu'}, {age: {$lt: 21}}]})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }
```

#### And和Or配合使用

```shell
# name张三 并且满足（name为wangwu或者年龄小于22）
> db.stu.find({'name':'zhangsan', $or: [{name: 'wangwu'}, {age: {$lt: 21}}]})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
```

### $type操作符

在MongoDB 中根据字段的数据类型来查询数据使用$type 操作符来实现，具体使用法语：

db.集合名.find({$type:类型值}) 		//这里的类型值能使用Number 也能使用alias

| Type                    | Number | Alias                 | Notes              |
| ----------------------- | ------ | --------------------- | ------------------ |
| Double                  | 1      | 'double'              |                    |
| String                  | 2      | 'string'              |                    |
| Object                  | 3      | 'object'              |                    |
| Array                   | 4      | 'array'               |                    |
| Binary data             | 5      | 'binData'             | 二进制数据         |
| Undefined               | 6      | 'undefined'           | 未知类型，已过时   |
| ObjectId                | 7      | 'objectId'            | 数据主键           |
| Boolean                 | 8      | 'bool'                |                    |
| Date                    | 9      | 'date'                |                    |
| Null                    | 10     | 'null'                | value为null时      |
| Regular Expression      | 11     | 'regex'               |                    |
| DBPointer               | 12     | 'dbPointer'           | 数据库指针，已过时 |
| JavaScript              | 13     | 'javascript'          |                    |
| Symbol                  | 14     | 'symbol'              | 已过时             |
| JavaScript（with scop） | 15     | 'javaScriptWithScope' |                    |
| 32-bit integer          | 16     | 'int'                 |                    |
| Timestamp               | 17     | 'timestamp'           |                    |
| 64-bit integer          | 18     | 'long'                |                    |
| Decimal128              | 19     | 'decimal'             | 3.4版本以上        |
| Min key                 | -1     | 'minKey'              |                    |
| Max key                 | 127    | 'maxKey'              |                    |

```shell
> db.stu.find({'name': {$type:'int'}})
> db.stu.find({'name': {$type:'2'}})
Error: error: {
        "ok" : 0,
        "errmsg" : "Unknown type name alias: 2",
        "code" : 2,
        "codeName" : "BadValue"
}
> db.stu.find({'name': {$type:2}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }
```

### 正则查询

MongoDB中查询条件也可以使用正则表达式作为匹配约束。

**语法**:

- db.集合名称.find({字段名:正则表达式}); 

- db.集合名称.find({字段名:{&#36;regex: 正则表达式[&#36;options: 正则选项]}});

**正则表达式格式**：/xxx/

**正则选项**：

- **i**  不区分大小写以匹配大小写的情况。

- **m**  多行匹配模式（\n 换行符）。对于包含锚点的模式（即^对于开始，$对于结尾），在每行的开头或结尾处匹配具有多行值的字符串（可解析单字段中的换行符\n）。如果没有此选项，这些锚点将在字符串的开头或结尾处匹配。

- **x**  设置x选项后，正则表达式中的**非转义**的**空白字符将被忽略**。需要&#36;regex与&#36;options语法
- **s**  允许点字符（即 . ）匹配包括换行符在内的所有字符。需要$regex与&#36;options语法

 i，m，x，s可以组合使用。

```shell
# 查询集合中name字段以‘z’开头的数据
> db.stu.find({'name': /^z/})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
> db.stu.find({'name': {$regex:/^z/}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }

# 查询集合中name字段以’n‘结尾的数据
> db.stu.find({'name': {$regex:/n$/}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
> db.stu.find({'name': /n$/})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }

# 查询集合中name字段包含’a‘的数据
> db.stu.find({'name': /a/})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }

# 查询集合中name字段以’L‘开头的数据，并且忽略大小写
> db.stu.find({'name': {$regex:/^L/i}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
> db.stu.find({'name': /^L/i})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
> db.stu.find({'name': {$regex:/^L/, $options: 'i'}})
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }

# 查询集合中name字段以'z'或l开头的数据 
db.stu.find({'name': {$in:[/^z/, /^l/]} });
# 查询集合中name字段不以z开头的数据 
db.stu.find({'name': {$not:/^z/}});
# 查询集合中name字段不以z或l开头的数据 
db.stu.find( {'name': {$nin:[/^z/,/^l/]}});
```

### 分页查询

在 MongoDB 中，使用函数 limit()和 skip0来实现分页数据查询。

#### limit函数

在MongoDB 中读取指定数量的数据记录，可以使用 MongoDB 的 Limit 方法，limit()方法接受一个数字参数，该参数指定从 MongoDB 中读取的记录条数。

**语法**：db.集合名称.find().limit(查询数量);

```shell
#查询 stu 集合中的前 2 条数据: 
> db.stu.find().limit(2);
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf0"), "name" : "zhangsan", "age" : 20 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
```

> 不使用 limit 函数，默认查询集合中全部文档。如果limit 函数不传递参数，也是查询集合中的全部文档

#### skip 函数

在MongoDB 中使用 skip()方法来跳过指定数量的文档，skip 方法同样接受一个数字参数作为跳过的文档条数。

**语法**：db.集合名称.find().limit(查询数量).skip(跳过数量);

```shell
> db.stu.find().limit(2).skip(1);
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf1"), "name" : "lisi", "age" : 22 }
{ "_id" : ObjectId("62ee0de2ed1e3c2d7ec4acf2"), "name" : "wangwu", "age" : 25 }
```

### 排序

在MongoDB 中使用sort方法对数据进行排序，sort()方法可以通过参数指定排序的字段，并使用1和-1来指定排序的方式，其中1为升序排列，而-1是用于降序排列。

**语法**：

db.集合名称.find().sort({key1:sortTypel,key2:sortType2});

```shell
# 查询stu集合中全部文档，以age字段升序排列: 
db.stu.find().sort({'age':1});
# 查询stu集合中全部文档，以age字段升序排列，如果age相同，则以name字段降序排列:
db.stu.find().sort( {'age':1, 'name':-1 });
```

