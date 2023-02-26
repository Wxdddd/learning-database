## Elasticsearch安装



### 下载Elasticsearch

官网下载地址：https://www.elastic.co/cn/downloads/elasticsearch，点击进入，选择不同的平台下载不同的安装软件



### 上传并解压

官网安装文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html

下载完elasticsearch-8.4.3的tar包以后，上传到**node1**服务器`/home/elasticsearch`目录中，或者使用`wget`下载到服务器

```shell
[root@elasticsearch1 elasticsearch]# wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.4.3-linux-x86_64.tar.gz
```

解压

```shell
[root@elasticsearch1 elasticsearch]# tar -zxvf elasticsearch-8.4.3-linux-x86_64.tar.gz
```



### 创建es账号

如果不创建非root账号，直接启动会报错，`cannot run elasticsearch as root`故创建es账号，密码为123456，同时要将elasticsearch的解压目录权限赋予es账号

```shell
[root@elasticsearch1 elasticsearch]# useradd es
#授权
[root@elasticsearch1 elasticsearch]# chown -R es:es elasticsearch-8.4.3
[root@elasticsearch1 elasticsearch]# su es
[es@elasticsearch1 elasticsearch]$ 
```

### 开发（学习）模式

开发模式是默认配置（未配置集群发现设置），如果用户只是出于学习目的，而引导检查会把很多用户挡在门外，所以ES提供了一个设置项`discovery.type:single-node`。此项配置为指定节点为单节点发现以绕过引导检查。Elasticsearch8版本默认开启xpack，安全策略。默认密码在启动日志上。因此将`xpack.security.enabled: false`能够更快启动

```shell
[es@elasticsearch1 elasticsearch]$ cd elasticsearch-8.4.3/config/
[es@elasticsearch1 config]$ ll
total 40
-rw-rw----. 1 es es  1042 Oct  4 15:16 elasticsearch-plugins.example.yml
-rw-rw----. 1 es es  2903 Oct  4 15:16 elasticsearch.yml
-rw-rw----. 1 es es  2563 Oct  4 15:16 jvm.options
drwxr-x---. 2 es es     6 Oct  4 15:20 jvm.options.d
-rw-rw----. 1 es es 17417 Oct  4 15:20 log4j2.properties
-rw-rw----. 1 es es   473 Oct  4 15:19 role_mapping.yml
-rw-rw----. 1 es es   197 Oct  4 15:19 roles.yml
-rw-rw----. 1 es es     0 Oct  4 15:19 users
-rw-rw----. 1 es es     0 Oct  4 15:19 users_roles
[es@elasticsearch1 config]$ vim elasticsearch.yml
```

```yaml
# ======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
#
# ---------------------------------- Network -----------------------------------
#
# By default Elasticsearch is only accessible on localhost. Set a different
# address here to expose this node on the network:
# 1. 所有地址都可访问
network.host: 0.0.0.0
#
# By default Elasticsearch listens for HTTP traffic on the first free port it
# finds starting at 9200. Set a specific HTTP port here:
#
#http.port: 9200
#
# For more information, consult the network module documentation.
#
# --------------------------------- Discovery ----------------------------------
#
# Pass an initial list of hosts to perform discovery when this node is started:
# The default list of hosts is ["127.0.0.1", "[::1]"]
#
#discovery.seed_hosts: ["host1", "host2"]
# 2. 单节点运行
discovery.type: single-node
#
# Bootstrap the cluster using an initial set of master-eligible nodes:
#
#cluster.initial_master_nodes: ["node-1", "node-2"]
#
# For more information, consult the discovery and cluster formation module documentation.
#
# --------------------------------- Readiness ----------------------------------
#
# Enable an unauthenticated TCP readiness endpoint on localhost
#
#readiness.port: 9399
#
# ---------------------------------- Various -----------------------------------
#
# Allow wildcard deletion of indices:
#
#action.destructive_requires_name: false
# 3. 关闭安全策略
xpack.security.enabled: false

```



#### 启动

```shell
[es@elasticsearch1 config]$ cd ../bin/
[es@elasticsearch1 bin]$ ./elasticsearch
```

![image-20221215211009319](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221215211009319.png)

#### 环境变量配置

```shell
[es@elasticsearch1 bin]$ su root
Password:
[root@elasticsearch1 bin]# vi /etc/profile
export ES_HOME=/home/elasticsearch/elasticsearch-8.4.3
export PATH=$ES_HOME/bin:$PATH
[root@elasticsearch1 bin]# source /etc/profile
[root@elasticsearch1 bin]# su es
# 后台启动
[es@elasticsearch1 bin]$ elasticsearch -d

```

### 生产模式

当用户修改了有关集群的相关配置会触发生产模式，在生产模式下，服务启动会触发ES的引导检查或者叫启动检查（bootstrap checks），所谓引导检查就是在服务启动之前对一些重要的配置项进行检查，检查其配置值是否是合理的。引导检查包括对JVM大小、内存锁、虚拟内存、最大线程数、集群发现相关配置等相关的检查，如果某一项或者几项的配置不合理，ES会拒绝启动服务，并且在开发模式下的某些警告信息会升级成错误信息输出。引导检查十分严格，之所以宁可拒绝服务也要阻止用户启动服务是为了防止用户在对ES的基本使用不了解的前提下启动服务而导致的后期性能问题无法解决或者解决起来很麻烦。因为一旦服务以某种不合理的配置启动，时间久了之后可能会产生较大的性能问题，但此时集群已经变得难以维护和扩展，ES为了避免这种情况而做出了引导检查的设置，本来在开发模式下为警告的启动日志会升级为报错（Error）。这种设定虽然增加了用户的使用门槛，但是避免了日后产生更大的问题。如下配置会触发生产模式

- discovery.seed_hosts
- cluster.initial_master_nodes

当你配置集群发现相关配置项的时候，Elasticsearch 就会认为你将用于生产环境，并将警告升级为异常。这些异常将阻止 Elasticsearch 节点启动。这是一项重要的安全措施，可确保不会因为服务器配置错误而丢失数据。