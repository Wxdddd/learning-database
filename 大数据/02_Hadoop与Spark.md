## Hadoop与Spark

### 1. Spark是什么

Spark是基于内存计算的大数据并行计算框架。Spark基于内存计算的特性，提高了在大数据环境下数据处理的实时性，同时保证了高容错性和高可伸缩性，允许用户将Spark部署在大量的廉价硬件之上形成集群，从而提高并行计算能力。

Spark于2009年诞生于加州大学伯克利分校AMP Lab，在开发以Spark为核心的BDAS时，AMP Lab提出的目标是：one stack to rule them all，也就是说在一套软件栈内完成各种大数据分析任务。目前，Spark已经成为Apache软件基金会旗下的顶级开源项目。

### 2. Spark的特点

- 运行速度快

  Spark源码是由Scala语言编写的，Scala语言非常简洁并具有丰富的表达力。 Spark充分利用和集成了Hadoop等其他第三方组件，同时着眼于大数据处理，那么数据处理速度是至关重要的，Spark通过将中间结果缓存在内存从而减少磁盘I/O来达到性能的提升。

- 易用性

  ​	Spark支持Java、Python和Scala的API，还支持超过80种高级算法，使用户可以快速构建不同的应用。而且Spark支持交互式的Python和Scala的shell，可以非常方便地在这些shell中使用Spark集群来验证解决问题的方法。

- 支持复杂查询

  ​	除了简单的map及reduce操作之外，Spark还支持复杂查询。Spark支持SQL查询、流式计算、机器学习和图算法，同时用户可以在同一个工作流中无缝地搭配这些计算范式。

- 实时的流处理

  ​	与Hadoop相比，Spark不仅支持离线计算还支持实时流计算。Spark Streaming主要用来对数据进行实时处理，而Hadoop在拥有了YARN之后，也可以借助其他框架进行流式计算。

- 容错性

  ​	Spark引入了弹性分布式数据集RDD(Resilient Distributed Dataset)，它是分布在一组节点中的只读对象集合，这些集合是弹性的，如果数据集的一部分丢失，则可以根据“血统”对它们进行重建。另外在对RDD进行计算时可以通过CheckPoint机制来实现容错。

### 3. Hadoop VS Spark

![image-20221203144208478](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221203144208478.png)