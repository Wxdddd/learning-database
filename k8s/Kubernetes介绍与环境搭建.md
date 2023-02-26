### 1. Kubernetes介绍

![image-20230219123314426](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230219123314426.png)



<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230219131029276.png" alt="image-20230219131029276" style="zoom:50%;" />

- Image：docker 镜像

- Container：docker 容器

- POD

  - pod中可以有一个或多个容器
  - pod中所有的容器都运行在一个机器上
  - pod中所有的容器都有一个唯一的ip
  - 每一个pod中都有一个Pause容器，作用是作为pod中的根容器把其他容器链接到一起。还有一个功能负责pod的健康检查汇报给k8s。

- ReplicaSet（RS副本集）

  - 负责同一个应用需要运行几个实例
  - 如果有实例停止运行会自动重启或重新部署应用

- Deploment

  - 负责部署更新
  - 更新时会先创建一组新的RS，新的RS创建完成后才会清除旧的RS

  

<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230219131002582.png" alt="image-20230219131002582" style="zoom:50%;" />

- lable（标签）
  - Deploment，ReplicaSet，POD等都可以进行标签
- Service
  - 通过Selector标识是哪一个标签，将对应标签的内容（POD）进行对外映射。
  - ClusterIP是Service对外的ip，提供给其他容器或者其它客户端访问。



<img src="https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20230219130722978.png" alt="image-20230219130722978" style="zoom:50%;" />

- Master
  - 主节点
  - 管理Worker节点
- Worker
  - 工作节点
- ETCD
  - 进行部署配置的存储
- ApiServer
  - 操作K8S接口
- Scheduler
  - 调度器，负责Node与Pod关联配置
- ControllerManager
  - 内部控制中心，负责维护各种K8S对象
- Kubelet
  - 负责POD在Worker上的运行

### 2. 认证与授权

