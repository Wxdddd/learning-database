## 基本概念

| ElasticSearch | 索引（Index）      | 类型（Type）    | 属性（Field）  | 文档（Document） | 映射（Mapping）      |
| :------------ | :----------------- | :-------------- | :------------- | :--------------- | :------------------- |
| 关系型数据库  | 数据库（Database） | 数据表（Table） | 字段（Column） | 数据行（Row）    | 表定义文件（Schema） |

注意：从ElasticSearch6.X开始，官方准备废弃Type了，官方7.0.0以后不建议使用，8.0.0 以后完全不支持。原因如下：

通俗理解Index比作 SQL 的 Database，Type比作SQL的Table。但这并不准确，因为如果在SQL中,Table 之前相互独立，同名的字段在两个表中毫无关系。但是在ES中，同一个Index 下不同的 Type 如果有同名的字段，他们会被 Lucence 当作同一个字段 ，并且他们的定义必须相同。所以官方认为Index现在更像一个表，而Type字段并没有多少意义。目前Type已经被Deprecated，在7.0开始，一个索引只能建一个Type为_doc

### 索引Index

索引是文档（Document）的容器，是一类文档的集合。

- 索引（名词）
  - 类比传统的关系型数据库领域来说，索引相当于SQL中的一个数据库(Database)。
  - 索引由其名称(必须为全小写字符)进行标识。
- 索引（动词）
  - 保存一个文档到索引(名词)的过程。
  - 这非常类似于SQL语句中的 INSERT关键词。如果该文档已存在时那就相当于数据库的UPDATE。
- 索引（倒排索引）
  - 关系型数据库通过增加一个B+树索引到指定的列上，以便提升数据检索速度。
  - 索引ElasticSearch 使用了一个叫做 倒排索引 的结构来达到相同的目的。

### 文档Document

Document：Index 里面单条的记录称为Document（文档）。等同于关系型数据库表中的行。

![image-20221216155532573](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221216155532573.png)

```properties
`_index` 文档所属索引名称。

`_type` 文档所属类型名。

_id Doc的主键。在写入的时候，可以指定该Doc的ID值，如果不指定，则系统自动生成一个唯一的UUID值。

_version 文档的版本信息。Elasticsearch通过使用version来保证对文档的变更能以正确的顺序执行，避免乱序造成的数据丢失。

_seq_no 严格递增的顺序号，每个文档一个，Shard级别严格递增，保证后写入的Doc的_seq_no大于先写入的Doc的_seq_no。

primary_term primary_term也和_seq_no一样是一个整数，每当Primary Shard发生重新分配时，比如重启，Primary选举等，_primary_term会递增1

found 查询的ID正确那么ture, 如果 Id 不正确，就查不到数据，found字段就是false。

_source 文档的原始JSON数据。
```



### 映射Mapping

从7.x开始，一个Mapping只属于一个索引的type 默认type 为：_doc。es字段类型主要有：核心类型、复杂类型、地理类型以及特殊类型

![image-20221216155952214](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221216155952214.png)

#### 核心类型

![image-20221216160026255](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221216160026255.png)

- 字符串类型
  - text ：
    - 类型适用于需要被全文检索的字段，例如新闻正文、邮件内容等比较长的文字，text 类型会被 Lucene 分词器（Analyzer）处理为一个个词项，并使用 Lucene 倒排索引存储，text 字段不能被用于排序，如果需要使用该类型的字段只需要在定义映射时指定 JSON 中对应字段的 type 为 text。
    - 非结构化的文本数据
  - keyword：
    - 不会被分词，适合简短、结构化字符串，例如主机名、姓名、商品名称等，可以用于过滤、排序、聚合检索
    - 也可以用于精确查询。
      - 包括数字、日期、具体的字符串（如“192.168.0.1”）
- 数字类型
  - 数字类型分为 long、integer、short、byte、double、float、half_float、scaled_float。
  - 数字类型的字段在满足需求的前提下应当尽量选择范围较小的数据类型，字段长度越短，搜索效率越高。
  - 对于浮点数，可以优先考虑使用 scaled_float 类型，该类型可以通过缩放因子来精确浮点数，例如 12.34 可以转换为 1234 来存储。
- 日期类型
  - ES 底层依然采用的是时间戳的形式存储
  - “format”: “yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis”
- 布尔类型
  - JSON 文档中同样存在布尔类型，不过 JSON 字符串类型也可以被 ES 转换为布尔类型存储，前提是字符串的取值为 true 或者 false，布尔类型常用于检索中的过滤条件。
- 二进制类型
  - 二进制类型 binary 接受 BASE64 编码的字符串，默认 store 属性为 false，并且不可以被搜索。
- 范围类型
  - 范围类型可以用来表达一个数据的区间，可以分为5种：
    - integer_range、float_range、long_range、double_range 以及 date_range。

#### 复杂类型

复合类型主要有**对象类型（object）**和**嵌套类型（nested）**

- 对象类型：JSON 字符串允许嵌套对象，一个文档可以嵌套多个、多层对象。可以通过对象类型来存储二级文档，不过由于 Lucene 并没有内部对象的概念，ES 会将原 JSON 文档扁平化

**字段值为json数据**

```shell
# 插入数据
{
    "name":"张三",
    "description":{
        "date":"2030-12-12",
        "title":"Watching TV"
    }
}
```

等同于

```shell
{
    "name":"张三",
    "description.date":"2030-12-12",
    "description.title":"Watching TV"
}
```

**字段值为json数组**

```shell
# 插入数据
{
    "name":"张三",
    "userid":3,
    "description":[
        {
           "date":"2030-12-12",
        	"title":"Watching TV"
        },
        {
            "date":"2032-12-13",
        	"title":"Shoping"
        }
    ]
}
```

等同于

```shell
{
    "name":"张三",
    "userid":1,
    "description.date":["2030-12-12","2032-12-13"],
    "description.title":["Watching TV","Shopping"]
}
```

此时搜索：“2030年” 且“包含Shopping”的记录时，这条记录仍然可以被搜索出来。就是因为**存储对象数组时使用object会丢失层级关系**，解决方式就是需要Nested格式解决。

- 嵌套类型：可以看成是一个特殊的对象类型，可以让对象数组独立检索

```shell
# 插入数据
{
    "name":"张三",
    "userid":3,
    "description":[
        {
           "date":"2030-12-12",
        	"title":"Watching TV"
        },
        {
            "date":"2032-12-13",
        	"title":"Shoping"
        }
    ]
}
```

等同于一个根记录

```shell
{
    "name":"张三",
    "userid":1
}
```

两条嵌套文档

```shell
{
    "description.date":"2030-12-12",
    "description.title":"Watching TV"
}

{
    "description.date":"2032-12-13",
    "description.title":"Shoping"
}
```

#### 地理数据类型

- geo_point类型：可用于存储某个地理坐标的经纬度。
- geo_shape类型：用于存储地理多边形形状

#### 特殊数据类型

- ip类型：
  - 用于表示IPv4与IPv6的地址
- completion类型：
  - 提供自动输入关联完成功能，如常见的baidu搜索框。
- token_count类型：
  - 用于计算字符串token的长度，使用时需提供"analyzer"定义。
- percolate类型：一个索引中只能有一个percolator字段类型
  - 定义为percolate类型的字段会被ES分析为一个查询并保存下来，并可用在后继对文档的查询中。Percolate可以理解为一个预置的查询。
- alias类型：
  - 定义一个已存在字段的别名。
- join类型：
  - 该类型定义了文档对象之间的父子关系，即同一索引中多个文档对象可以存在依赖关系，如互联网应用常见的博客文章与回复，问题与回答之间的关系



### Dynamic Mapping

- 动态映射时Elasticsearch的一个重要特性:
  - 不需要提前创建index、定义mapping信息和type类型,
  - 可以 直接向ES中插入文档数据时, ES会根据每个新field可能的数据类型, 自动为其配置type等mapping信息
  - 这个过程就是动态映射(dynamic mapping).

![image-20221216160206516](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221216160206516.png)

- 约束策略
  - dynamic设为true时，新增字段的文档写入时，Mapping同时被更新
  - dynamic设为false时，Mapping不会被更新，新增字段的数据无法被索引，但是会出现在_source中，忽略
  - dynamic设为strict，新增字段时文档将写入失败