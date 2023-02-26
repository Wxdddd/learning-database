### RabbitMQ集群模式与原理解析

#### RabbitMQ四种集群模式

- 主备模式：热备份，一个master一个slave，正常master提供读写slave提供备份，master异常情况下slave升级为master。
- 远程模式：主要做数据异地的容灾，异地转移，提升性能。当单个集群处理不过来时可以转移到下游的某个集群去处理。
- 镜像模式：业界流行，消息非常可靠。
- 多活模式：与远程模式类似。

#### 主备模式

- **warren（兔子窝）**，一个主/备方案（主节点如果挂了，从节点提供服务，和**ActiveMQ**利用zookeeper做主备一样，主备切换秒级。）
- 主备架构模型

![RabbitMQ-主备模式](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/RabbitMQ-%E4%B8%BB%E5%A4%87%E6%A8%A1%E5%BC%8F.png)

- 主备模式HaProxy配置

  ```shell
  listen rabbitmq_cluster #定义主备集群的名称：rabbitmq_cluster
  	bind 0.0.0.0:5672	#绑定端口：5672
  	mode tcp #配置TCP模式
  	balance roundrobin	#简单轮训
  	server bhz76 192.168.11.71:5672 check inter 5000 rise 2 fall 2	#主节点
  	server bhz77 192.168.11.72:5672 backup check inter 5000 rise 2 fall 2 #备用节点
  ```

#### 远程模式

- 远距离通信和复制，可以实现双活的一种模式，简称Shovel模式。

- 所谓的Shovel就是我们可以把消息进行不同数据中心的复制工作，可以跨地域的让两个mq集群互联。

- Shovel架构模型

  ![image-20211209153527939](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20211209153527939.png)

  > 在使用了shovel插件后，模型变成了近端同步确认，远端异步确认的方式，大大提高了订单确认速度，并且还能保证可靠性。

- Shovel集群的拓扑图

  ![image-20211209154106847](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20211209154106847.png)

  > 此MQ集群可以帮我们做集群陆游的转换。第一个集群消费不过来时转移到第二个，第一个集群出现问题时可以转换到第二个。

- 此模式因配置过于繁琐，使用较少，也**不推荐使用**。

#### 镜像模式

- 集群模式非常经典的就是Mirror镜像模式，保证100%数据不丢失。
- 在实际工作中用的最多，并且实现集群非常的简单，一般互联网大厂都会构建这种镜像集群模式。
- Mirror镜像队列：高可靠 -》数据同步 -》可靠性（奇数节点）。

- RabbitMQ集群架构图

  ![image-20211209155149359](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20211209155149359.png)

  > 三个mirror queue数据都保持一致。

- 缺点不支持横向扩展。

#### 多活模式

- 这种也是实现异地数据复制的主流模式，因为Shovel模式配置比较复杂，所以一般来说实现异地集群都是使用这种双活 或者 多活模型来实现的。

- 这种模型需要依赖RabbitMQ的**federation插件**，可以实现持续的可靠的AMQP数据通信，多活模式实际配置与应用非常简单。

- RabbitMQ部署架构采用双中心模式（多中心），那么在两套（或多套）数据中心中各部署一套RabbitMQ集群，各中心的RabbitMQ服务除了需要为业务提供正常的消息服务外，中心之间还需要实现部分队列消息共享。

- 多活集群架构模式

  ![image-20211209160627488](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20211209160627488.png)

- federation插件：是一个不需要构建Cluster，而是在Brokers之间传输消息的高性能插件，federation插件可以再Brokers或者Cluster之间传输消息，连接的双方可以使用不同的users和virtual hosts，双方可以使用版本不同的RabbitMQ和Erlang。federation插件使用AMQP协议通讯，可以接受不连续的传输。

- federation exchanges，可以看成Downstream从Upstream主动拉取消息（下游主动从上游拉取消息），但并不是拉取所有消息，必须是在Downstream上已经明确定义Bindings（绑定）关系的Exchange（通道），也就是有实际的物理Queue（队列）来接收消息，才会从Upstream拉取消息到Downstream。使用AMQP协议实施代理间通信，Downstream会将绑定关系组合在一起，绑定/解除绑定命令将发送到Upstream交换机。因此，federation exchanges 只接受具有订阅的消息，来自官方说明。

![image-20211209201354090](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20211209201354090.png)