## RabbitMQ高性能的原因

### Erlang

- 由爱立信公司开发（Ericsson Language）

- 一门为交换机软件开发诞生的编程语言

### Erlang简史

- 始于1987年，1998年爱立信公司开源Erlang VM、OTP源代码

- 1999年，Erlang Solutions公司在伦敦成立

- 2005年，Alexey Shchepin发布了一个基于Erlang的IM服务，随后催生了Facebook Chat 和WhatsApp

- 2006年，基于Erlang的RabbitMQ诞生

### Erlang特点

- 通用的面向并发的编程语言，适用于分布式系统
- 基于虚拟机解释运行，跨平台部署
- 进程间上下文切换效率远高于C语言

- 有着和原生Socket一样的延迟



## AMQP协议介绍

### AMQP协议的作用

- AMQP 是一个协议，规定了软件之间的行为

- RabbitMQ 是AMQP协议的具体实现

- AMQP协议还有其他实现，如RocketMQ等

![image-20230225105534567](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225105534567.png)

- Broker: 接收和分发消息的应用，RabbitMQ 就是Message Broker

- Virtual Host: 虚拟Broker，将多个单元隔离开

- Connection: publisher／consumer和broker之间的TCP连接

- Channel: connection内部建立的逻辑连接，通常每个线程创建单独的channel

- Routing Key: 路由键，用来指示消息的路由转发，相当于快递的地址

- Exchange: 交换机，相当于快递的分拨中心，发送给Queue，是AMQP中的核心概念

- Queue: 消息队列，消息最终被送到这里等待consumer取走

- Binding: exchange和queue之间的虚拟连接，用于message的分发依据



AMQP协议的核心概念-Exchange

- 在AMQP协议或者是RabbitMQ实现中，最核心的组件是Exchange

- Exchange承担RabbitMQ的核心功能——路由转发

- Exchange有多个种类，配置多变



## RabbitMQ核心 - Exchange解析

### Exchange的作用

- Exchange是AMQP协议和RabbitMQ的核心组件

- Exchange的功能是根据绑定关系和路由键为消息提供路由，将消息转发至相应的队列

- Exchange有4种类型：Direct / Topic / Fanout / Headers，其中Headers使用很少，以前三种为主



### Direct Exchang

- Message中的Routing Key如果和Binding Key一致， Direct Exchange则将message发到对应的

queue中

![image-20230225111337735](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225111337735.png)

### Fanout Exchange

- 每个发到Fanout Exchange的message都会分发到所有绑定的queue上去

![image-20230225111701356](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225111701356.png)

### Topic Exchange

- 根据Routing Key及**通配规则**，Topic Exchange将消息分发到目标Queue中

- 全匹配：与Direct类似

- Binding Key中的**#**：匹配任意个数的word

- Binding Key中的*****：匹配任意1个word
- 示例：
  - 咖啡含有咖啡因，冷热均可，苦甜均可。使用Binding Key表示：caffeine.#
  - 奶茶不含有咖啡因，热饮，甜味。使用Binding Key表示：nocaffeine.hot.sweet
  - 果汁不含有咖啡因，冷热均可，甜味。使用Binding Key表示：nocaffeine.*.sweet

<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225112752453.png" alt="image-20230225112752453" style="zoom:50%;" />

<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225112715661.png" alt="image-20230225112715661" style="zoom:50%;" />

<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225112840069.png" alt="image-20230225112840069" style="zoom:50%;" />

示例网站：http://tryrabbitmq.com/

### Exchange总结

- AMQP协议直接决定了RabbitMQ的内部结构和外部行为

- 对于发送者来说，将消息发给特定的Exchange

- 消息经过Exchange路由后，到达具体队列

- 消费者将消息从监听的队列中取走

- Exchange主要有3种类型：Direct / Topic / Fanout

- Direct（直接路由）：Routing Key = Binding Key，容易配置和使用

- Fanout（广播路由）：群发绑定的所有队列，适用于消息广播

- Topic（话题路由）：功能较为复杂，但能降级为Direct，建议优先使用，为以后拓展留余地



## RabbitMQ快速安装

### 安装注意事项

- RabbitMQ是基于Erlang的跨平台应用，Windows / Linux / MacOS 都可以安装
- 生产环境绝大多数都是Linux操作系统，Windows / MacOS 一般作为本地开发使用
- 一定要在官网或官方渠道下载安装，一旦有后门的应用进入生产环境后果不堪设想

### Windows安装

- 下载并安装Erlang OTP (Open Telecom Platform)  https://www.erlang.org/downloads

- 下载并安装RabbitMQ: https://www.rabbitmq.com/

- 安装完成后，查看系统服务中会出现RabbitMQ

### MacOS安装

brew update

brew install rabbitmq

brew 工具会自动安装OTP依赖

### Linux安装(Docker)

docker run -it --rm --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:3-management



## 网页端管理工具

### 网页端管理工具介绍

- RabbitMQ网页端管理工具也叫管理控制台、管控台
- 管理控制台是RabbitMQ最常用的管理、配置工具
- 管理控制台对于业务的开发、调试也非常有用

### 管控台的功能页面

- 概览：查看节点/集群状态

- 连接：查看connection

- 通道：查看channel

- 交换机：查看、操作交换机状态

- 队列：查看、操作队列状态

- 管理：高级管理功能

启动应用：rabbitmq-plugins enable rabbitmq_management （启动应用后需要重启或启动rabbitmq）

浏览器打开：http://127.0.0.1:15672

默认用户名：guest 	默认密码：guest

![image-20230225170242190](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230225170242190.png)



## 命令行工具

### 使用场景

生产环境、端口限制等不便打开网页端工具的场景

使用脚本自动化配置RabbitMQ



### 使用口诀

- 想看什么就List什么

- 想清空什么就purge什么

- 想删除什么就Delete什么

- 一切问题记得使用 --help



### 状态查看

- 查看状态：rabbitmqctl status

- 查看绑定：rabbitmqctl list_bindings

- 查看channel：rabbitmqctl list_channels

- 查看connection：rabbitmqctl list_connections

- 查看消费者：rabbitmqctl list_consumers

- 查看交换机：rabbitmqctl list_exchanges

- 查看队列：rabbitmqctl list_queues

- 删除队列：rabbitmqctl delete_queue

- 清空队列：rabbitmqctl purge_queue



### 用户相关

- 新建用户：rabbitmqctl add_user

- 修改用户密码：rabbitmqctl change_password

- 删除用户：rabbitmqctl delete_user

- 查看用户：rabbitmqctl list_users

- 设置用户角色：rabbitmqctl rabbitmqctl set_user_tags



### 应用启停

启动应用：rabbitmqctl start_app

关闭应用：rabbitmqctl stop_app，保留Erlang虚拟机（暂停）

关闭应用：rabbitmqctl stop，并关闭Erlang虚拟机



### 集群相关

加入集群：rabbitmqctl join_cluster

离开集群：rabbitmqctl reset



### 镜像队列

设置镜像队列：rabbitmqctl sync_queue

取消镜像队列：rabbitmqctl cancel_sync_queue



### 总结

命令行工具强大

命令行工具的功能基本可以在管控台找到，但有其特殊场景

命令不需要死记硬背，需要时查询文档即可，记住万能的–help