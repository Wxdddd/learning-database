## 容器编排Docker Swarm

### 1. Docker Swarm介绍

- 怎么去管理多个容器？
- 怎么能去方便横向扩展
- 如果容器down了，怎么能自动恢复？
- 如何去更新容器而不影响业务？
- 如何去监控追踪这些容器？
- 怎么去调度容器的创建？
- 保护隐私数据？

> Docker Swarm就是为了解决这些等等问题而诞生的。



#### Docker Swarm两种角色Manager与Worker

![image-20191117093158710](images/容器编排Docker Swarm/image-20191117093158710.png)

#### Docker Swarm两种概念Service和Replicas

- 一个Service代表一个container，这个container可以从dockerhub拉取的image创建，或者从本地的Dockerfile build出来的image创建
- Replicas表示横向扩展，一个Replicas表示横向扩展一个容器

#### ![一个Service扩展三个Replicas](images/容器编排Docker Swarm/image-20191117093942954.png)

> 一个nginx的Service横向扩展三个Replicas，上图实际过程中会产生三个容器，这三个容器会通过我们的调度系统计算哪一个node负载较轻，然后调度到不同的node上面去。

- swarm 服务创建和调度的过程

  ![image-20191117094524432](images/容器编排Docker Swarm/image-20191117094524432.png)

### 2. 安装

Docker Swarm 由docker公司开发，存在于docker中，安装好docker docker swarm也就安装好了。

### 3. 创建一个三节点的swarm集群

#### 使用Vagrantfile创建

- Vagrantfile文件

```shell
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.6.0"

boxes = [
    {
        :name => "swarm-manager",
        :eth1 => "192.168.205.10",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "swarm-worker1",
        :eth1 => "192.168.205.11",
        :mem => "1024",
        :cpu => "1"
    },
    {
        :name => "swarm-worker2",
        :eth1 => "192.168.205.12",
        :mem => "1024",
        :cpu => "1"
    }
]

Vagrant.configure(2) do |config|

  config.vm.box = "centos/7"

  boxes.each do |opts|
      config.vm.define opts[:name] do |config|
        config.vm.hostname = opts[:name]
        config.vm.provider "vmware_fusion" do |v|
          v.vmx["memsize"] = opts[:mem]
          v.vmx["numvcpus"] = opts[:cpu]
        end

        config.vm.provider "virtualbox" do |v|
          v.customize ["modifyvm", :id, "--memory", opts[:mem]]
          v.customize ["modifyvm", :id, "--cpus", opts[:cpu]]
        end

        config.vm.network :private_network, ip: opts[:eth1]
      end
  end

  config.vm.synced_folder "./labs", "/home/vagrant/labs"
  config.vm.provision "shell", privileged: true, path: "./setup.sh"

end
```

- 创建虚拟主机

```shell
192:chapter7 webxudong$ vagrant up
192:chapter7 webxudong$ vagrant status
Current machine states:

swarm-manager             running (virtualbox)
swarm-worker1             running (virtualbox)
swarm-worker2             running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```

#### 进入三台虚拟主机

```
192:chapter7 webxudong$ vagrant ssh swarm-manager
[vagrant@swarm-manager ~]$ 

192:chapter7 webxudong$ vagrant ssh swarm-worker1
[vagrant@swarm-worker1 ~]$ 

192:chapter7 webxudong$ vagrant ssh swarm-worker2
[vagrant@swarm-worker2 ~]$ 
```

#### 三台虚拟主机初始化docker swarm

- swarm-manager节点上运行

  ```sh
  [vagrant@swarm-manager ~]$ docker swarm init --advertise-addr 192.168.205.10
  Swarm initialized: current node (j60cxemf5q7twe6p65qpzhpv4) is now a manager.
  
  To add a worker to this swarm, run the following command:
  
      docker swarm join --token SWMTKN-1-07gcjx7j45mcjut8aswso26hb5x1futswqbua47pt6b1ik2xlq-aazwz5ra9z4v4erxr8qzixp1o 192.168.205.10:2377
  
  To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
  ```

- swarm-worker1节点上运行

  ```sh
  [vagrant@swarm-worker1 ~]$ docker swarm join --token SWMTKN-1-07gcjx7j45mcjut8aswso26hb5x1futswqbua47pt6b1ik2xlq-aazwz5ra9z4v4erxr8qzixp1o 192.168.205.10:2377
  This node joined a swarm as a worker.
  ```

- swarm-worker1节点上运行

  ```sh
  [vagrant@swarm-worker2 ~]$ docker swarm join --token SWMTKN-1-07gcjx7j45mcjut8aswso26hb5x1futswqbua47pt6b1ik2xlq-aazwz5ra9z4v4erxr8qzixp1o 192.168.205.10:2377
  This node joined a swarm as a worker.
  ```

- swarm-manager节点上查看swarm集群

  ```sh
  [vagrant@swarm-manager ~]$ docker node ls
  ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
  j60cxemf5q7twe6p65qpzhpv4 *   swarm-manager       Ready               Active              Leader              19.03.5
  j362tummq26toh1sr6rfarfl0     swarm-worker1       Ready               Active                                  19.03.5
  o59j6gjuy45p9ix1rlv127bdx     swarm-worker2       Ready               Active                                  19.03.5
  ```

### 4. Service的创建维护和水平扩展

#### 创建Service

- swarm-manager节点上运行

  ```shell
  [vagrant@swarm-manager ~]$ docker service create --name demo busybox sh -c "while true;do sleep 3600;done"
  [vagrant@swarm-manager ~]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  24jntr5hdpxp        demo                replicated          1/1                 busybox:latest 
  # 查看demo container运行在哪个node
  [vagrant@swarm-manager ~]$ docker service ps demo
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
  ivvn28n2ss52        demo.1              busybox:latest      swarm-manager       Running             Running about a minute ago 
  [vagrant@swarm-manager ~]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  24jntr5hdpxp        demo                replicated          1/1                 busybox:latest 
  ```

  > `MODE`: 模式
  >
  > `REPLICAS`: 水平扩展 1/1 分母表示水平扩展个数，分子表示运行的个数

#### Service水平扩展

- swarm-manager节点上运行

  ```sh
  [vagrant@swarm-manager ~]$ docker service scale demo=5
  demo scaled to 5
  overall progress: 5 out of 5 tasks 
  1/5: running   [==================================================>] 
  2/5: running   [==================================================>] 
  3/5: running   [==================================================>] 
  4/5: running   [==================================================>] 
  5/5: running   [==================================================>] 
  verify: Service converged 
  [vagrant@swarm-manager ~]$ docker service ps demo
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
  ivvn28n2ss52        demo.1              busybox:latest      swarm-manager       Running             Running 13 minutes ago                       
  u5typ20d0y4v        demo.2              busybox:latest      swarm-worker2       Running             Running 11 seconds ago                       
  ialhh3zp6j44        demo.3              busybox:latest      swarm-worker2       Running             Running 11 seconds ago                       
  ujz455zw5e72        demo.4              busybox:latest      swarm-manager       Running             Running 24 seconds ago                       
  aqoucqu52nhp        demo.5              busybox:latest      swarm-worker1       Running             Running 10 seconds ago   
  ```

  > 通过`NODE`节点观察，可发现有水平扩展后的5个demo container，2个在swarm-manager节点上，2个在swarm-worker2节点上，1个在swarm-worker1节点上。

- swarm-worker2节点上运行

  ```sh
  [vagrant@swarm-manager ~]$ docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
  f4081ed632ba        busybox:latest      "sh -c 'while true;d…"   6 minutes ago       Up 6 minutes                            demo.4.ujz455zw5e7223txnxv9s7ppw
  284153a86f6b        busybox:latest      "sh -c 'while true;d…"   20 minutes ago      Up 20 minutes                           demo.1.ivvn28n2ss52uzeryn9nodkm1
  # 在swarm-worker2 node上强制删除一个demo container
  [vagrant@swarm-worker2 ~]$ docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
  a915a5b0b425        busybox:latest      "sh -c 'while true;d…"   9 minutes ago       Up 8 minutes                            demo.2.u5typ20d0y4vqn2kky0c7r2ib
  7356c9c03d67        busybox:latest      "sh -c 'while true;d…"   9 minutes ago       Up 8 minutes                            demo.3.ialhh3zp6j44mbojzyv82j84l
  [vagrant@swarm-worker2 ~]$ docker rm -f 7356c9c03d67
  7356c9c03d67
  ```

- swarm-manager节点上运行

  ```sh
  [vagrant@swarm-manager ~]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  24jntr5hdpxp        demo                replicated          5/5                 busybox:latest  
  [vagrant@swarm-manager ~]$ docker service ps demo
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR                         PORTS
  ivvn28n2ss52        demo.1              busybox:latest      swarm-manager       Running             Running 23 minutes ago                                 
  u5typ20d0y4v        demo.2              busybox:latest      swarm-worker2       Running             Running 9 minutes ago                                  
  rhsryeg7n21u        demo.3              busybox:latest      swarm-worker1       Running             Running 21 seconds ago                                 
  ialhh3zp6j44         \_ demo.3          busybox:latest      swarm-worker2       Shutdown            Failed 26 seconds ago    "task: non-zero exit (137)"   
  ujz455zw5e72        demo.4              busybox:latest      swarm-manager       Running             Running 9 minutes ago                                  
  aqoucqu52nhp        demo.5              busybox:latest      swarm-worker1       Running             Running 9 minutes ago 
  ```

  > 可以观察出 我们在swarm-worker2上删除一个demo container之后，在swarm-worker1上多出了一个demo container。
  >
  > `这就说明swarm scale不仅可以做到横向扩展，而且可以确保我们一定数目的扩展是有效的。`

#### 删除Service

- swarm-manager节点上运行

  ```sh
  [vagrant@swarm-manager ~]$ docker service rm demo
  demo
  [vagrant@swarm-manager ~]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  ```

### 5. swarm集群里通过service部署wordpress

#### 创建网络

- 在swarm-manager节点上创建一个overlay的网络，使集群之间的container网络互通

  ```sh
  [vagrant@swarm-manager ~]$ docker network create -d overlay demo
  jd53h59af2rjd12t0s7ibuwha
  [vagrant@swarm-manager ~]$ docker network ls
  NETWORK ID          NAME                DRIVER              SCOPE
  fbbe012e1221        bridge              bridge              local
  jd53h59af2rj        demo                overlay             swarm
  594d137fcf21        docker_gwbridge     bridge              local
  1df5f5c0357a        host                host                local
  m0nxe0fd0k7p        ingress             overlay             swarm
  8b8926c5e805        none                null                local
  ```

#### 创建service容器

- 在swarm-manager节点上 创建mysql容器 

  ```sh
  [vagrant@swarm-manager ~]$ docker service create --name mysql --env MYSQL_ROOT_PASSWORD=root --env MYSQL_DATABASE=wordpress --network demo --mount type=volume,source=mysql-data,destination=/var/lib/mysql mysql:5.7
  [vagrant@swarm-manager ~]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  5veg8wfapdki        mysql               replicated          1/1                 mysql:latest        
  [vagrant@swarm-manager ~]$ docker service ps mysql
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE           ERROR               PORTS
  vpgxe1saj60e        mysql.1             mysql:latest        swarm-manager       Running             Running 2 minutes ago     
  ```

- 在swarm-manager节点上 创建wordpress容器

  ```sh
  # 这里 WORDPRESS_DB_HOST 地址一定要填写Container容器的名称来访问。
  [vagrant@swarm-manager ~]$ docker service create --name wordpress -p 80:80 --env WORDPRESS_DB_PASSWORD=root --env WORDPRESS_DB_HOST=mysql:3306 --network demo wordpress
  [vagrant@swarm-manager ~]$ docker service ps wordpress
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
  pb8xcsubwxrc        wordpress.1         wordpress:latest    swarm-worker1       Running             Running about a minute ago     
  ```

- 访问测试

  > 输入任意虚拟主机IP地址即可访问
  >
  > 例如： 192.168.250.11



### 6. 集群服务间通信之Routing mesh

> swarm集群之间的通信主要是由Routing Mesh实现的。
>
> 实例：`第5点的创建网络`

#### Routing Mesh的两种体现

##### `Internal`

`Internal` -- **Container和Container之间**的访问通过overlay网络（通过VIP也就是虚拟IP）,然后经过LVS映射到具体哪一个node的Container上访问。已达到负载均衡。

![image-20191117164742048](images/容器编排Docker Swarm/image-20191117164742048.png)

![request访问过程](images/容器编排Docker Swarm/image-20191117165846145.png)

##### `Ingress`

`Ingress` -- 如果服务有绑定接口，则此服务可以通过任意swarm节点的相应接口访问。

- 比如A、B两个node做了swarm集群，若A node有一个开放了8080端口的容器，则通过B node的8080端口也可以进行访问。

Ingress的三个特性

- 外部访问的负载均衡
- 服务端口被暴露到各个swarm节点
- 内部通过IPVS进行负载均衡

<img src="images/容器编排Docker Swarm/image-20191117170651834.png" alt="image-20191117170651834"  />

> 上图表示 当一个request请求到host3节点时，因为host3没有对应的服务容器，所以host3节点的IPV6就会将该请求转发到有服务容器的host2节点进行处理请求并返回消息。



![image-20191117171802547](images/容器编排Docker Swarm/image-20191117171802547.png)

> 具体讲解：https://coding.imooc.com/lesson/189.html#mid=11817

### 7. Docker Stack部署Wordpress

Docker Compose方式部署Docker Swarm集群

`官方文档`: https://docs.docker.com/compose/compose-file/#deploy

#### docker-compose.yml文件

```yaml
version: '3'

services:

  web:
    image: wordpress
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: mysql
      WORDPRESS_DB_PASSWORD: root
    networks:
      - my-network
    depends_on:
      - mysql
    deploy:
      mode: replicated
      replicas: 3
      restart_policy:
        condition: on-failure
        delay: 5s
        max_attempts: 3
      update_config:
        parallelism: 1
        delay: 10s

  mysql:
    image: mysql:5.7
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: wordpress
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - my-network
    deploy:
      mode: global
      placement:
        constraints:
          - node.role == manager

volumes:
  mysql-data:

networks:
  my-network:
    driver: overlay
```

> `mode`: 模式，`global`（swarm集群只有一个容器）或`replicated`（容器的指定数量）。默认值为`replicated`。
>
> `constraints`: 指定容器存放在哪一个节点
>
> `replicas`: 初始化容器的数量
>
> `restart_policy`: 配置是否以及如何在容器退出时重新启动容器
>
> - `condition`：重启策略，`none`，`on-failure`（如果失败就重启）或者`any`（默认值：`any`）。
> - `delay`：尝试重新启动等待的时间（默认值：0）。
> - `max_attempts`：在放弃之前尝试重新启动容器的次数（默认值：`never give up`）。如果在配置的`window`内重新启动未成功，则此尝试不计入配置的`max_attempts`值。例如，如果`max_attempts`设置为“ 2”，并且第一次尝试时重新启动失败，则可能会尝试两次以上的重新启动。
> - `window`：决定重新启动是否成功之前要等待的[时间](https://docs.docker.com/compose/compose-file/#specifying-durations)，指定为[持续时间](https://docs.docker.com/compose/compose-file/#specifying-durations)（默认值：立即决定）。
>
> `update_config`: 配置应如何更新服务。
>
> - `parallelism`: 一次更新的容器数量。
> - `delay`: 更新一组容器之间的等待时间。
> - `failure_action`:如果更新失败该怎么办。  `continue`, `rollback`, or `pause`之一 (default: `pause`).
> - `monitor`: 每个任务更新后监视失败的持续时间 `(ns|us|ms|s|m|h)` (default 0s).
> - `max_failure_ratio`: 更新期间容忍的失败率。
> - `order`: 更新期间的操作顺序. One of `stop-first` (旧任务在启动新任务之前停止), or `start-first` (新任务先启动与正在运行的任务短暂重叠) (默认 `stop-first`) **Note**: Only supported for v3.4 and higher

```sh
[vagrant@swarm-manager wordpress]$ ll
-rw-r--r--. 1 vagrant vagrant 797 11月  9 01:36 docker-compose.yml
```

#### 定义一个stack 

- 在swarm-manager虚拟机上运行

```sh
#wordpress 表示stack的name		--compose-file可简化为-c
[vagrant@swarm-manager wordpress]$ docker stack deploy wordpress --compose-file=docker-compose.yml
Creating network wordpress_my-network
Creating service wordpress_web
Creating service wordpress_mysql
[vagrant@swarm-manager wordpress]$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
wordpress           2                   Swarm
[vagrant@swarm-manager wordpress]$ docker stack ps wordpress
ID                  NAME                                        IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
lrrljhecevie        wordpress_mysql.j60cxemf5q7twe6p65qpzhpv4   mysql:5.7           swarm-manager       Running             Running about a minute ago                       
lv8pp54nocv5        wordpress_web.1                             wordpress:latest    swarm-worker2       Running             Running about a minute ago                       
4e5n61cxluc8        wordpress_web.2                             wordpress:latest    swarm-manager       Running             Running 17 seconds ago                           
j5ygi22idfa9        wordpress_web.3                             wordpress:latest    swarm-worker1       Running             Running about a minute ago                       
[vagrant@swarm-manager wordpress]$ docker stack services wordpress
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
5dvzninwbelp        wordpress_web       replicated          3/3                 wordpress:latest    *:8080->80/tcp
kn54k9wirhi6        wordpress_mysql     global              1/1                 mysql:5.7        
```

#### 访问测试

> 输入三台虚拟主机任意ip即可
>
> 例如：http://192.168.205.12:8080/

#### 删除stack

```sh
[vagrant@swarm-manager wordpress]$ docker stack rm wordpress
Removing service wordpress_mysql
Removing service wordpress_web
Removing network wordpress_my-network
[vagrant@swarm-manager wordpress]$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
```

### 8. Docker Secret管理和使用

#### 什么是Secret

- 用户名密码
- SSH key
- TLS认证
- 任何不想让人看到的数据

#### Secret Management

- 存在Swarm Manager节点 Raft database里。
- Secret可以assign给一个service，这个service就能看到这个secret
- 在container内部Secret看起来像文件，但实际是存在内存中的

#### 创建Secret

- 文件创建（swarm-manager node）

  ```sh
  # password 的内容为 admin123
  [vagrant@swarm-manager wordpress]$ vim password
  [vagrant@swarm-manager wordpress]$ docker secret create my-pw password
  k2cdynlhfrndmmmzsdy8ypu08
  [vagrant@swarm-manager wordpress]$ rm -rf password 
  [vagrant@swarm-manager wordpress]$ docker secret ls
  ID                          NAME                DRIVER              CREATED             UPDATED
  k2cdynlhfrndmmmzsdy8ypu08   my-pw                                   42 seconds ago      42 seconds ago
  ```

- 标准输入创建（swarm-manager node）

  ```sh
  [vagrant@swarm-manager wordpress]$ echo "adminadmin" | docker secret create my-pw2 -
  44jotpnunsf6g6r08p5aych2g
  [vagrant@swarm-manager wordpress]$ docker secret ls
  ID                          NAME                DRIVER              CREATED             UPDATED
  k2cdynlhfrndmmmzsdy8ypu08   my-pw                                   2 minutes ago       2 minutes ago
  44jotpnunsf6g6r08p5aych2g   my-pw2                                  3 seconds ago       3 seconds ago
  ```

- 删除secret（swarm-manager node）

  ```sh
  [vagrant@swarm-manager wordpress]$ docker secret rm my-pw2
  my-pw2
  [vagrant@swarm-manager wordpress]$ docker secret ls
  ID                          NAME                DRIVER              CREATED             UPDATED
  k2cdynlhfrndmmmzsdy8ypu08   my-pw                                   3 minutes ago       3 minutes ago
  ```

#### 使用secret

- ##### 示例（swarm-manager node）

```sh
# 传入多个secret时 指定多个--secret即可 如：--secret my-pw --secret my-pw2
[vagrant@swarm-manager wordpress]$ docker service create --name client --secret my-pw busybox sh -c "while true;do sleep 3600; done"
[vagrant@swarm-manager wordpress]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
fxhh0c9onw2i        client              replicated          1/1                 busybox:latest      
[vagrant@swarm-manager wordpress]$ docker service ps client
ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
t7n08iczw9cz        client.1            busybox:latest      swarm-manager       Running             Running about a minute ago                       
[vagrant@swarm-manager wordpress]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
7d8245e2a1ed        busybox:latest      "sh -c 'while true;d…"   About a minute ago   Up About a minute                       client.1.t7n08iczw9czjb3y0tpuxvox4
[vagrant@swarm-manager wordpress]$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS               NAMES
7d8245e2a1ed        busybox:latest      "sh -c 'while true;d…"   About a minute ago   Up About a minute                       client.1.t7n08iczw9czjb3y0tpuxvox4
#进入容器 查看secret
[vagrant@swarm-manager wordpress]$ docker exec -it 7d82 sh
/ # cd /run/secrets/
/run/secrets # ls
my-pw
/run/secrets # cat my-pw
admin123
```

- ##### 实例: mysql

   创建db容器（swarm-manager node）

  ```sh
  [vagrant@swarm-manager wordpress]$ docker service create --name db --secret my-pw -e MYSQL_ROOT_PASSWORD_FILE=/run/secrets/my-pw mysql:5.7
  nsk6nxis00y2iwqbmhkv99voz
  overall progress: 1 out of 1 tasks 
  1/1: running   [==================================================>] 
  verify: Service converged 
  [vagrant@swarm-manager wordpress]$ docker service ls
  ID                  NAME                MODE                REPLICAS            IMAGE               PORTS
  fxhh0c9onw2i        client              replicated          1/1                 busybox:latest      
  nsk6nxis00y2        db                  replicated          1/1                 mysql:5.7           
  [vagrant@swarm-manager wordpress]$ docker service ps db
  ID                  NAME                IMAGE               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
  fn83ntbf4587        db.1                mysql:5.7           swarm-worker1       Running             Running 59 seconds ago 
  ```

  验证db容器（swarm-worker1  db容器创建在了swarm-worker1 node）

  ```sh
  [vagrant@swarm-worker1 ~]$ docker ps
  CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
  03e7d7bca776        mysql:5.7           "docker-entrypoint.s…"   2 minutes ago       Up 2 minutes        3306/tcp, 33060/tcp   db.1.fn83ntbf45873q7vwr4bzov7h
  [vagrant@swarm-worker1 ~]$ docker exex -it 03e7 sh
  # cat /run/secrets/my-pw
  admin123
  # mysql -u root -p 
  Enter password: 
  Welcome to the MySQL monitor.  Commands end with ; or \g.
  Your MySQL connection id is 5
  Server version: 5.7.28 MySQL Community Server (GPL)
  
  Copyright (c) 2000, 2019, Oracle and/or its affiliates. All rights reserved.
  
  Oracle is a registered trademark of Oracle Corporation and/or its
  affiliates. Other names may be trademarks of their respective
  owners.
  
  Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
  
  mysql> 
  ```

- ##### docker stack 使用 secret

  更改之前的dockerfile

  ```yaml
  version: '3'
  
  services:
  
    web:
      image: wordpress
      ports:
        - 8080:80
      secrets:
        - my-pw #指定secret
      environment:
        WORDPRESS_DB_HOST: mysql
        WORDPRESS_DB_PASSWORD_FILE: /run/secrets/my-pw
      networks:
        - my-network
      depends_on:
        - mysql
      deploy:
        mode: replicated
        replicas: 3
        restart_policy:
          condition: on-failure
          delay: 5s
          max_attempts: 3
        update_config:
          parallelism: 1
          delay: 10s
  
    mysql:
      image: mysql
      secrets:
        - my-pw
      environment:
        MYSQL_ROOT_PASSWORD_FILE: /run/secrets/my-pw
        MYSQL_DATABASE: wordpress
      volumes:
        - mysql-data:/var/lib/mysql
      networks:
        - my-network
      deploy:
        mode: global
        placement:
          constraints:
            - node.role == manager
  
  volumes:
    mysql-data:
  
  networks:
    my-network:
      driver: overlay
  
  # secrets:
  #   my-pw:
  #    file: ./password 创建secret不推荐，推荐事先创建好
  ```

  > 其余操作相同

### 9. Service更新

#### 创建overlay网络（swarm-manager node）

```sh
[vagrant@swarm-manager wordpress]$ dcoker network create -d voerlay demo
[vagrant@swarm-manager wordpress]$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
fbbe012e1221        bridge              bridge              local
jd53h59af2rj        demo                overlay             swarm
594d137fcf21        docker_gwbridge     bridge              local
1df5f5c0357a        host                host                local
m0nxe0fd0k7p        ingress             overlay             swarm
8b8926c5e805        none                null                local
```

#### 创建容器（swarm-manager node）

```sh
[vagrant@swarm-manager wordpress]$ docker service create --name web --publish 8080:5000 --network demo xiaopeng163/python-flask-demo:1.0
[vagrant@swarm-manager wordpress]$ docker service ps web
ID                  NAME                IMAGE                               NODE                DESIRED STATE       CURRENT STATE                ERROR               PORTS
c47a9kwyh9gs        web.1               xiaopeng163/python-flask-demo:1.0   swarm-worker2       Running             Running about a minute ago                       
```

#### 更新容器

- 首先横向扩展容器（swarm-manager node）

  ```sh
  [vagrant@swarm-manager wordpress]$ docker service scale web=2
  [vagrant@swarm-manager wordpress]$ docker service ps web
  ID                  NAME                IMAGE                               NODE                DESIRED STATE       CURRENT STATE             ERROR               PORTS
  c47a9kwyh9gs        web.1               xiaopeng163/python-flask-demo:1.0   swarm-worker2       Running             Running 7 minutes ago                         
  ryfpv427p97m        web.2               xiaopeng163/python-flask-demo:1.0   swarm-manager       Running             Preparing 4 minutes ago   
  ```

- 更新两个容器的镜像

  ```sh
  [vagrant@swarm-manager wordpress]$ docker service update --image xiaopeng163/python-flask-demo:2.0 web
  web
  overall progress: 2 out of 2 tasks 
  1/2: running   [==================================================>] 
  2/2: running   [==================================================>] 
  verify: Service converged 
  [vagrant@swarm-manager wordpress]$ docker service ps web
  ID                  NAME                IMAGE                               NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
  88ifmz2u04kb        web.1               xiaopeng163/python-flask-demo:2.0   swarm-worker2       Running             Running 2 minutes ago                        
  c47a9kwyh9gs         \_ web.1           xiaopeng163/python-flask-demo:1.0   swarm-worker2       Shutdown            Shutdown 2 minutes ago                       
  y5476lu7hvgz        web.2               xiaopeng163/python-flask-demo:2.0   swarm-manager       Running             Running 2 minutes ago                        
  ryfpv427p97m         \_ web.2           xiaopeng163/python-flask-demo:1.0   swarm-manager       Shutdown            Shutdown 2 minutes ago   
  ```

  > `docker service update --help` 查看更新命令
  >
  > 更新两个容器时会一个一个更新不会让服务中断。

- 更新容器的端口

```sh
[vagrant@swarm-manager wordpress]$ docker service update --publish-rm 8080:5000 --publish-add 8088:5000 web
web
overall progress: 2 out of 2 tasks 
1/2: running   [==================================================>] 
2/2: running   [==================================================>] 
verify: Service converged 
[vagrant@swarm-manager wordpress]$ docker service ls
ID                  NAME                MODE                REPLICAS            IMAGE                               PORTS                           
kmziylwlje61        web                 replicated          2/2                 xiaopeng163/python-flask-demo:2.0   *:8088->5000/tcp
```

> `Docker Stack`的更新 修改yml文件再次执行
>
> `docker stack deploy wordpress -c=docker-compose.yml` 即可