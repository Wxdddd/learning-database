## 一、MongoDB简介

### 1、什么是MongoDB

MongoDB是一个基于分布式文件存储的数据库。由C++语言编写。旨在为WEB应用提供可扩展的高性能数据存储解决方案。
MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。它支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。Mongo最大的特点是它支持的查询语言非常强大，其语法有点类似于面向对象的查询语言，几乎可以实现类似关系数据库单表查询的绝大部分功能，而且还支持对数据建立索引。

### 2、什么是NoSQL

NoSOL（Not OnlySOL)，意即“不仅仅是SOL”，是一项全新的数据库革命性运动，早期就有人提出，发展至2009年趋势越发高涨。NoSQL的拥护者们提倡运用非关系型的数据存储，相对于铺天盖地的关系型数据库运用，这一概念无疑是一种全新的思维的注入。

### 3、NoSQL数据库的分类

#### 3.1 键值(Key-Value)存储数据库

这一类数据库主要会使用到一个哈希表，这个表中有一个特定的键和一个指针指向特定的数据。Key/value模型对于IT系统来说的优势在于简单、易部署。但是如果DBA只对部分值进行查询或更新的时候，Key/value就显得效率低下了。举例如：Tokyo，Cabinet/Tyrant，Redis，Voldemort，Oracle BDB。

#### 3.2 列存储数据库

这部分数据库通常是用来应对分布式存储的海量数据。键仍然存在，但是它们的特点是指向了多个列。这些列是由列家族来安排的。如:Cassandra，HBase，Riak。

#### 3.3 文档型数据库

文档型数据库的灵感是来自于Lotus Notes办公软件的，而且它同第一种键值存储相类似。该类型的数据模型是版本化的文档，半结构化的文档以特定的格式存储，比如JSON。文档型数据库可以看作是键值数据库的升级版，允许之间嵌套键值。而且文档型数据库比键值数据库的查询效率更高。如:CouchDB，MongoDb.国内也有文档型数据库 SequoiaDB，已经开源。

### 4、MongoDB与关系型数据库对比

#### 4.1 术语对比

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接MongoDB不支持                 |
| primary key  | primary key      | 主键，MongoDB自动将id字段设置为主键 |

#### 4.2 存储结构对比

![image-20220725185332216](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20220725185332216.png)

#### 4.3 RDBS与MongoDB 对应术语

| RDBMS    | MongoDB                       |
| -------- | ----------------------------- |
| 数据库   | 数据库                        |
| 表       | 集合                          |
| 行       | 文档                          |
| 列       | 字段                          |
| 表联合   | 嵌入文档                      |
| 主机爱你 | 主键（MongoDB提供了key为_id） |

## 二、MongoDB的 数据类型

| 数据类型           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| String             | 字符串，在MongoDB中UTF-8编码的字符串才是合法的               |
| Integer            | 整数类型。根据您所采用的服务器，可分为32位与64位             |
| Boolean            | 布尔值                                                       |
| Double             | 双精度浮点值                                                 |
| Min/Max keys       | 将一个值与BSON（二进制的JSON）元素的最低值和最高值相对比     |
| Array              | 用于将数组或列表或多个值存储为一个键                         |
| Timestamp          | 时间戳                                                       |
| Object             | 用于内嵌文档                                                 |
| Null               | 用于创建空值                                                 |
| Symbol             | 符号，该数据类型等同于字符串类型，但不同的是，他一般用于采用特殊符号类型的语言 |
| Date               | 日期时间，用UNIX时间格式来存储当前日期或时间                 |
| Object ID          | 对象 id，用于创建文档 的id                                   |
| Binary Data        | 二进制数据，用于存储二进制数据                               |
| Code               | 代码类型。用于在文档中存储java script代码                    |
| Regular Expression | 正则表达式类型。存储正则表达式。                             |

