## 快速便捷的接入各种不同数据源的数据

https://nightlies.apache.org/flink/flink-docs-master/docs/dev/datastream/overview/

> 大数据处理的三段论： 输入  处理  输出

### 编程规范

1. 获取执行环境
2. 加载或创建初始数据
3. 针对数据做处理操作
4. 结果输出
5. 触发程序

DataStream表示的是Flink程序中要执行的数据集合，是不可变的数据集合（数据是可以重复的）。可以是无界的也可以使有界的。

#### 1. 获取执行环境

StreamExecutionEnvironment：

```java
getExecutionEnvironment() 这是我们用的最多的一种
createLocalEnvironment()  这种仅限于本地开发使用
createRemoteEnvironment()  知道就行，开发不用
```

#### 2. 数据源

**添加数据源**

```java
StreamExecutionEnvironment.addSource(sourceFunction)
```

**内置数据源**

File-based：

- readTextFile(path)
- readFile(fileInputFormat, path)
- readFile(fileInputFormat, path, watchType, interval, pathFilter, typeInfo)

Socket-based：

- socketTextStream

Collection-based：

- fromCollection(Collection)
- fromCollection(Iterator, Class)
- fromElements(T ...)
- fromParallelCollection(SplittableIterator, Class)
- generateSequence(from, to)

**制定数据源**

> 如果内置的Source不能满足我们的需求时，那么我们就可以通过SourceFunction（单并行度）、ParallelSourceFunction（多并行度）或RichParallelSourceFunction（更高级别多并行度）接口自定义Source来实现。

示例：`addSource(new FlinkKafkaConsumer<>(...))` See [connectors](https://nightlies.apache.org/flink/flink-docs-master/docs/connectors/datastream/overview/) for more details

> 单并行度：fromElements  fromCollection  socketTextStream 等
>
> 多并行度：readTextFile fromParallelCollection generateSequence  readFile 等

**Flink中datasource的使用**

```java
public class FlinkDataSourceApp {
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();

//        DataStreamSource<String> source = env.readFile(new TextInputFormat(null), "data/wc.data");
//
//        // 这个readTextFile方法底层其实调用的就是readFile
////        DataStreamSource<String> source = env.readTextFile("data/wc.txt");
//        System.out.println(source.getParallelism());  // 8
//
//        SingleOutputStreamOperator<String> mapStream = source.map(String::toUpperCase);
//        System.out.println(mapStream.getParallelism());
//        mapStream.print();

//        DataStreamSource<Access> source = env.addSource(new AccessSourceV2()).setParallelism(3); // 对于ParallelSourceFunction是可以根据具体情况来设定并行度的
//        System.out.println(source.getParallelism());
//        source.print();

        /**
         * 使用Flink自定义MySQL的数据源，进而读取MySQL里面的数据
         */

        env.addSource(new PKMySQLSource()).setParallelism(3).print();


        /**
         * 单并行度：fromElements  fromCollection  socketTextStream
         * 多并行度：readTextFile fromParallelCollection generateSequence  readFile
         * 自定义：
         */
        env.execute("作业名字");
    }
}
```



