## 高效简洁的数据处理方式

- 认识Transformation算子
- map
- filter
- flatMap
- keyBy
- union
- connect

https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/operators/overview/

### map

**DataStream → DataStream**

> 每个元素都作用上一个方法，得到新的元素

```java
DataStreamSource<String> source = env.readTextFile("data/access.log");
SingleOutputStreamOperator<Access> mapStream = source.map(new AccessConvertFunction());
mapStream.print();
```



### filter

**DataStream → DataStream**

> 过滤出满足条件的元素

```java
DataStreamSource<String> source = env.readTextFile("data/access.log");
source.map(new AccessConvertFunction())
	.filter(x -> x.getTraffic() > 4000).print();
```



### flatMap

**DataStream → DataStream**

>可以是一对一、一对多、一对0  一个元素进来，可以出去0、1、多个元素

```java
DataStreamSource<String> source = env.readTextFile("data/access.log");
source.map(new AccessConvertFunction())
        .flatMap(new AccessFlatMapFunction())
        .print();
```



### KeyBy

**DataStream → KeyedStream**

> 对数据流进行聚合操作，操作后一般接上sum，reduce等操作

```java
DataStreamSource<String> source = env.readTextFile("data/access.log");
source.map(new AccessConvertFunction())
        .keyBy(x -> x.getDomain()).sum("traffic")
        .print();
```



### Union

**DataStream* → DataStream**

> 多个数据流合并为一个数据流，包含所有流里面的所有的元素。如果你自己合并自己那么流中会有两份数据
>
> 数据类型问题：union的多个流中数据类型是需要相同的

```java
DataStreamSource<Integer> stream1 = env.fromElements(1, 2, 3);
DataStreamSource<Integer> stream2 = env.fromElements(11, 12, 13);
DataStreamSource<String> stream3 = env.fromElements("A", "B", "C");

stream1.union(stream2).map(x -> "PK_" + x).print();
stream1.union(stream1).print();
stream1.union(stream1, stream2).print();
```

