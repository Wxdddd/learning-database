## 聚合查询

> MongoDB中聚合（aggregate）主要用于处理数据（诸如统计平均值求和等），并返回计算后的数据结果。聚合框架是MongoDB的高级查询语言，允许我们通过转化合并由多个文档的数据来生成新的在单个文档里不存在的文档信息。通俗一点来说，可以把MongoDB的聚合查询等价于SQL 的GROUP BY语句。

### aggregate函数

MongoDB中聚合的方法使用aggregate。

**语法**：
db.集合名称.aggregate(<agg_options>)

**参数解释**:
agg_options：数组类型参数，传入具体的聚合表达式要求，来计算聚合结果。此参数代表聚合规则，如计算总和、平均值、最大最小值等。在MongoDB的聚合查询中，操作复杂度都在这里。

```shell
# 聚合统计stu集合中文档数量:
> db.stu.aggregate([{$group:{_id:null, count:{$sum:1}}}]);
{ "_id" : null, "count" : 3 }
# 上述操作类似SQL 中的:select coun(*) from stu
```

### 常见的聚合表达式

提供测试数据: 

db.agg.insert([{'name':'zhangsan', 'age':20, 'tags':['JavaSE', 'JDBC'], 'birth': ISODate('2000-01-01')},{'name':'lisi', 'age':20, 'tags':['JavaSE', 'JavaEE'], "birth":ISODate('2000-02-01')},{'name':'wangwu', 'age':21, 'tags':['JavaSE', 'JDBC', 'JavaEE'],"birth": ISODate('2000-03-01')},{'name':'zhaoliu', 'age':21, 'birth': ISODate('2000-04-01')}]);

#### 求和 - $sum

统计agg集合中所有文档age字段的总和:

```shell
> db.agg.aggregate([{$group:{_id:null, sum_age: {$sum: '$age'}}}]);
{ "_id" : null, "sum_age" : 82 }
```

**语法解释:**

$group：分组，代表聚合的分组条件

id：分组的字段。相当于 SQL 分组语法 group by column_name 中的 column_name 部分。

如果根据某字段的值分组，则定义为_id:'$字段名'。所以此案例中的null 代表一个固定的字面值'null'。

sum_age：返回结果字段名。可以自定义，类似 SQL 中的字段别名。

$sum：求和表达式。相当于 SQL 中的sum()。

$age：代表文档中的age字段的值。

> 此案例中的 &#36;sum:&#36;age 代表求文档中age字段的和。
>
> 如果案例定义为$sum:age，代表求age字面值的和。在MongoDB中，字符串是不会进行数学相关运算的。得到的求和结果一定是0。

```shell
# 根据相同的年龄进行分组
> db.agg.aggregate([{$group:{_id:"$age", count: {$sum: 1}}}]);
{ "_id" : 21, "count" : 2 }
{ "_id" : 20, "count" : 2 }
```

#### 条件筛选 - $match

统计 agg 集合中，年龄小于21 的文档数量:

```shell
> db.agg.aggregate([{$match: {'age':{$lt:21}}},{$group:{_id:null, doc_num: {$sum:1}}}]);
{ "_id" : null, "doc_num" : 2 }
```

**语法解释**:

$match：匹配条件，相当于SQL 中的where子句，代表聚合之前进行条件筛选。

{'age':{$lt:21}}：age 字段小于 21。

统计agg集合中，相同年龄的文档数量，且要求统计结果中，同年龄的文档数量大于1: 

```shell
> db.agg.aggregate([{$group:{_id:'$age', nums:{$sum:1}}},{$match:{nums:{$gt:1}}}]);
{ "_id" : 21, "nums" : 2 }
{ "_id" : 20, "nums" : 2 }
```

**语法解释**：
{&#36;group:{id:'&#36;age',nums:{&#36;sum:1}}}：统计每个age字段值的文档数量。

{$match:{nums:{$gt:1}}}：匹配条件，相当于 SQL 中的 having 子句。代表聚合之后进行条件筛选。返回nums字段值大于1的结果。

> 注意：&#36;match表达式**定义的位置不同，代表的筛选时机不同**，根据具体情况，合理的定义$match表达式，可以有效提升聚合查询效率。
>
> 尽可能先筛选过滤更多的无效数据，再进行分组，效率最高。

#### 最大值 - $max

```shell
# 查询agg集合中age字段最大的文档：
> db.agg.aggregate([{$group:{_id:null, max_age: {$max:'$age'}} } ]);
{ "_id" : null, "max_age" : 21 }
```

#### 最小值- $min

```shell
# 查询agg集合中birth字段最小的文档：
> db.agg.aggregate([{$group:{_id:null, min_birth: {$min:"$birth"} } } ]);
{ "_id" : null, "min_birth" : ISODate("2000-01-01T00:00:00Z") }
```

#### 平均值 - $avg

```shell
# 查询agg集合中所有文档age 字段的平均值：
> db.agg.aggregate([{$group:{_id:null, avg_age:{$avg:'$age'}}}]);
{ "_id" : null, "avg_age" : 20.5 }
```

#### 统计结果数组返回-$push

```shell
# 查询agg集合中age字段每个值的对应的文档的name字段值: 
> db.agg.aggregate([{$group:{_id:'$age', names:{$push:'$name'}}}]);
{ "_id" : 21, "names" : [ "wangwu", "zhaoliu" ] }
{ "_id" : 20, "names" : [ "zhangsan", "lisi" ] }
```

**语法解释**:

```
names:{$push:'$name'}:
```

将分组结果中每个文档的name字段的值载入一个数组。返回字段名为names。

```shell
# 查询agg集合中age字段每个值的对应的文档数据:
> db.agg.aggregate([{$group: {_id:'$age', docs: {$push:'$$ROOT'}}}]);
{ "_id" : 21, "docs" : [ { "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : [ "JavaSE", "JDBC", "JavaEE" ], "birth" : ISODate("2000-03-01T00:00:00Z") }, { "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "age" : 21, "birth" : ISODate("2000-04-01T00:00:00Z") } ] }

{ "_id" : 20, "docs" : [ { "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : [ "JavaSE", "JDBC" ], "birth" : ISODate("2000-01-01T00:00:00Z") }, { "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : [ "JavaSE", "JavaEE" ], "birth" : ISODate("2000-02-01T00:00:00Z") } ] }
```

#### 数组字段拆分 - $unwind



```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果
> db.agg.aggregate([{$unwind:'$tags'}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaSE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JDBC", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaEE", "birth" : ISODate("2000-03-01T00:00:00Z") }
```

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，并包含对应数据在数组中的索引
> db.agg.aggregate([{$unwind:{path:'$tags', includeArrayIndex:'arrayIndex'} }]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z"), "arrayIndex" : NumberLong(0) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z"), "arrayIndex" : NumberLong(1) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z"), "arrayIndex" : NumberLong(0) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z"), "arrayIndex" : NumberLong(1) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaSE", "birth" : ISODate("2000-03-01T00:00:00Z"), "arrayIndex" : NumberLong(0) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JDBC", "birth" : ISODate("2000-03-01T00:00:00Z"), "arrayIndex" : NumberLong(1) }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaEE", "birth" : ISODate("2000-03-01T00:00:00Z"), "arrayIndex" : NumberLong(2) }
```

**语法解释**：

path:'$tags' ：要拆分的数组字段。 

includeArrayIndex：包含数组下标。

'arrayIndex'：返回结果中数组下标的字段名，可自定义。

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，并包含tags字段不存在、字段值为null或空数组的文档
> db.agg.aggregate([{$unwind:{path:'$tags',preserveNullAndEmptyArrays:true }}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaSE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JDBC", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaEE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "age" : 21, "birth" : ISODate("2000-04-01T00:00:00Z") }
```

#### 排序 - $sort

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，并根据拆分后的 tags 升序排序:
> db.agg.aggregate([{$unwind:'$tags'},  {$sort:{'tags':1}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JDBC", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaEE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaSE", "birth" : ISODate("2000-03-01T00:00:00Z") }
```

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，并根据拆分后的 tags升序排序，如果tags值相同，则按照age字段降序排列:
> db.agg.aggregate([{$unwind:'$tags'}, {$sort:{'tags':1, 'age':-1}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JDBC", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaEE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "age" : 21, "tags" : "JavaSE", "birth" : ISODate("2000-03-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
```



#### 分页相关 - &#36;limit和&#36;skip

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果。假设分页显示结果，每页显示2条数据，查询第一页
> db.agg.aggregate([{$unwind:'$tags'},{$limit:2} ]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-01-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "age" : 20, "tags" : "JDBC", "birth" : ISODate("2000-01-01T00:00:00Z") }
```

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果。假设分页显示结果，每页显示2条数据，查询第二页
> db.agg.aggregate([{$unwind:'$tags'},{$limit:4},{$skip:2}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
# 第二种写法
> db.agg.aggregate([{$unwind:'$tags'},{$skip:2},{$limit:2}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaSE", "birth" : ISODate("2000-02-01T00:00:00Z") }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "age" : 20, "tags" : "JavaEE", "birth" : ISODate("2000-02-01T00:00:00Z") }
```

**语法解释**：

 {$limit:2}：查询前2条数据。如果没有排序，则按照存储顺序计算。

{$skip:2}：跳过前2条数据。如果没有排序，则按照存储顺序计算。

> 备注：{&#36;skip:} 与  {&#36;limit:} 函数，哪个函数在前就先执行哪个函数。 

#### 管道操作（聚合投影约束）- $project

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，只显示返回结果的 name 和tags 字段的值:
> db.agg.aggregate([{$unwind:'$tags'},{$project: {'name':1, 'tags':1} }]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "tags" : "JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "tags" : "JDBC" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "tags" : "JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "tags" : "JavaEE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "tags" : "JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "tags" : "JDBC" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "tags" : "JavaEE" }
```

**语法解释**:
$project:{name:1, tags:1}：返回结果包含name字段和tags字段。如果不包含name字段，则定义为name:0。在聚合查询返回结果中id字段是默认包含的。

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，只显示返回结果的 name和tags字段的值，且不包含默认的_id 字段:
> db.agg.aggregate([{$unwind:'$tags'},{$project: {'_id':0, 'name':1, 'tags':1}}])
{ "name" : "zhangsan", "tags" : "JavaSE" }
{ "name" : "zhangsan", "tags" : "JDBC" }
{ "name" : "lisi", "tags" : "JavaSE" }
{ "name" : "lisi", "tags" : "JavaEE" }
{ "name" : "wangwu", "tags" : "JavaSE" }
{ "name" : "wangwu", "tags" : "JDBC" }
{ "name" : "wangwu", "tags" : "JavaEE" }

# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，返回结果包含name和tags 字段的值，并将age字段和birth字段的值保持在一个命名为age_birth的数组中:
> db.agg.aggregate([{$unwind:'$tags'},{$project:{'_id':0,'name':1,'tags':1,'age_birth':['$age','$birth']}}]);
{ "name" : "zhangsan", "tags" : "JavaSE", "age_birth" : [ 20, ISODate("2000-01-01T00:00:00Z") ] }
{ "name" : "zhangsan", "tags" : "JDBC", "age_birth" : [ 20, ISODate("2000-01-01T00:00:00Z") ] }
{ "name" : "lisi", "tags" : "JavaSE", "age_birth" : [ 20, ISODate("2000-02-01T00:00:00Z") ] }
{ "name" : "lisi", "tags" : "JavaEE", "age_birth" : [ 20, ISODate("2000-02-01T00:00:00Z") ] }
{ "name" : "wangwu", "tags" : "JavaSE", "age_birth" : [ 21, ISODate("2000-03-01T00:00:00Z") ] }
{ "name" : "wangwu", "tags" : "JDBC", "age_birth" : [ 21, ISODate("2000-03-01T00:00:00Z") ] }
{ "name" : "wangwu", "tags" : "JavaEE", "age_birth" : [ 21, ISODate("2000-03-01T00:00:00Z") ] }
```

**语法解释**：
age birth:['&#36;age', '&#36;birth']：定义字段名agebirth的数组，并将age和birth字段的值载入数组中返回。

#### 管道操作（字符串函数）

```shell
# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，返回结果包含name和tags 字段的值，并将 name 字段转大写，tags 字段转小写:
> db.agg.aggregate([{$unwind:'$tags'},{$project: {'new_name':{$toUpper:'$name'},'new_tags':{$toLower:'$tags'}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "new_name" : "ZHANGSAN", "new_tags" : "javase" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "new_name" : "ZHANGSAN", "new_tags" : "jdbc" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "new_name" : "LISI", "new_tags" : "javase" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "new_name" : "LISI", "new_tags" : "javaee" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "new_name" : "WANGWU", "new_tags" : "javase" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "new_name" : "WANGWU", "new_tags" : "jdbc" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "new_name" : "WANGWU", "new_tags" : "javaee" }

# 语法解释:
'new_name':{$toUpper:'$name'}：将name字段数据转大写，并显示字段名为newname
'new_tags':{$toLower:'$tags'}：将tags字段值转小写，并显示字段名为new_tags

# 拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，将name字段和tags字段的值拼接为一个完整字符串:
> db.agg.aggregate([{$unwind:'$tags'},{$project:{name_tags: {$concat:['$name', '-', '$tags']}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name_tags" : "zhangsan-JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name_tags" : "zhangsan-JDBC" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name_tags" : "lisi-JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name_tags" : "lisi-JavaEE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_tags" : "wangwu-JavaSE" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_tags" : "wangwu-JDBC" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_tags" : "wangwu-JavaEE" }

# 语法解释：
name_tags:{$concat:['$name'，'$tags']}：将字段name，字符串'-'和字段tags的值拼接为新的字符串，并显示字段名为name_tags

#  拆分agg集合中tags数组字段的值，使数组中每个值对应一个结果，只显示name字段的前3个字符:
> db.agg.aggregate([{$unwind:'$tags'},{$project:{name_prefix: {$substr:['$name',0,3]}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name_prefix" : "zha" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name_prefix" : "zha" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name_prefix" : "lis" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name_prefix" : "lis" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_prefix" : "wan" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_prefix" : "wan" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name_prefix" : "wan" }
```

#### 管道操作（数学运算函数）

```shell
# 查询agg集合中数据，显示name和age字段，并为age字段数据做加1操作 
> db.agg.aggregate([{$project:{name:1, new_age:{$add:['$age',1]}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "new_age" : 21 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "new_age" : 21 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "new_age" : 22 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "new_age" : 22 }

# 查询agg集合中数据，显示name和age字段，并为age字段数据做减1操作
> db.agg.aggregate([{$project:{name:1,new_age:{$subtract:['$age',1]}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "new_age" : 19 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "new_age" : 19 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "new_age" : 20 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "new_age" : 20 }

#  查询agg集合中数据，显示name和age字段，并为age字段数据做乘2操作: 
> db.agg.aggregate([{$project:{name:1, new_age: {$multiply:['$age', 2]} } }]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "new_age" : 40 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "new_age" : 40 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "new_age" : 42 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "new_age" : 42 }

# 查询agg集合中数据，显示name和age字段，并为age字段数据做除2操作: 
> db.agg.aggregate([{$project:{name:1, new_age:{$divide:['$age', 2]}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "new_age" : 10 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "new_age" : 10 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "new_age" : 10.5 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "new_age" : 10.5 }

# 查询agg集合中数据，显示name和age字段，并为age字段数据做模2操作: 
> db.agg.aggregate([{$project: {name:1, new_age: {$mod:['$age', 2]}}}]);
{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "name" : "zhangsan", "new_age" : 0 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "name" : "lisi", "new_age" : 0 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "name" : "wangwu", "new_age" : 1 }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "name" : "zhaoliu", "new_age" : 1 }
```

#### 管道操作（日期函数）

```shell
查询agg 集合中的数据，显示birth字段的各部分数据，包括:年、月、日等。
> db.agg.aggregate({$project: {"年份":{$year:"$birth"},"月份":{$month:"$birth"}, "一年中第几周":{$week:"$birth"},"日期":{$dayOfMonth: "$birth"},"星期": {$day
OfWeek:"$birth"}, "一年中第几天":{$dayOfYear:"$birth"},"时":{$hour:"$birth"},"分":{$minute:"$birth"},"秒":{$second:"$birth"}," 毫秒":{$millisecond:"$birth"},
"自定义格式化时间": {$dateToString: {format: "%Y 年%m 月%d 日 %H:%M:%S",date:"$birth"}}}})

{ "_id" : ObjectId("62ee81a41149d2ef699ece07"), "年份" : 2000, "月份" : 1, "一年中第几周" : 0, "日期" : 1, "星期" : 7, "一年中第几天" : 1, "时" : 0, "分" : 0, "秒" : 0, " 毫秒" : 0, "自定义格式化时间" : "2000 年01 月01 日 00:00:00" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece08"), "年份" : 2000, "月份" : 2, "一年中第几周" : 5, "日期" : 1, "星期" : 3, "一年中第几天" : 32, "时" : 0, "分" : 0, "秒" : 0, " 毫秒" : 0, "自定义格式化时间" : "2000 年02 月01 日 00:00:00" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece09"), "年份" : 2000, "月份" : 3, "一年中第几周" : 9, "日期" : 1, "星期" : 4, "一年中第几天" : 61, "时" : 0, "分" : 0, "秒" : 0, " 毫秒" : 0, "自定义格式化时间" : "2000 年03 月01 日 00:00:00" }
{ "_id" : ObjectId("62ee81a41149d2ef699ece0a"), "年份" : 2000, "月份" : 4, "一年中第几周" : 13, "日期" : 1, "星期" : 7, "一年中第几天" : 92, "时" : 0, "分" : 0, "秒" : 0, " 毫秒" : 0, "自定义格式化时间" : "2000 年04 月01 日 00:00:00" }
语法解释:
{$week:"$birth"}:每年的周计数从0开始。
{$dayOfWeek:"$birth"}:星期日为1，星期六为 7。
($dateToString:{format:"%Y年%m 月%d日%H:%M:%S"date:"$birth"}}：使用MongoDB中的日期占位表达式格式化birth字段的数据。%Y年、%m月、%d 日、%H24小时制、%M分钟、%S 秒。
```

