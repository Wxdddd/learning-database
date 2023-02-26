## Elasticsearch8集群安装

### 修改elasticsearch.yml

```properties
cluster.name: my-es-cluster
node.name: elasticsearch-1
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["192.168.52.129"]
cluster.initial_master_nodes: ["elasticsearch-1"]
xpack.security.enabled: true

# https证书配置
xpack.security.http.ssl:
  enabled: false
  verification_mode: certificate
  truststore.path: certs/elastic-certificates.p12
  keystore.path: certs/elastic-certificates.p12

# 传输层ssl证书
xpack.security.transport.ssl:
  enabled: true
  verification_mode: certificate
  keystore.path: certs/elastic-certificates.p12
  truststore.path: certs/elastic-certificates.p12

# 启用Elasticsearch API密钥服务
xpack.security.authc.api_key.enabled: true
# 允许自动创建索引
action.auto_create_index: ".app-search-*-logs-*,-.app-search-*,+*"

# 允许跨域请求
http.cors.enabled: true
http.cors.allow-origin: "*"
```

其中：verification_mode有三个值：

- full：它验证所提供的证书是否由受信任的权威机构(CA)签名，并验证服务器的主机名（或IP地址）是否与证书中识别的名称匹配。
- certificate：它验证所提供的证书是否由受信任的机构(CA)签名，但不执行任何主机名验证。
- none：它不执行服务器证书的验证。



### 修改jvm.options

因为虚拟机的内存不是特别高，所以根据实际情况将jvm.options的Xms和Xmx参数调小一些

```properties
# vim jvm.options
-Xms1g
-Xmx1g
```



### 系统设置

```shell
# 修改limits.conf文件，用于实现对用户资源进行限制,如进程数/文件数等
vim /etc/security/limits.conf
# soft指的是当前系统生效的设置值，软限制也可以理解为警告值。hard表明系统中所能设定的最大值。soft的限制不能比hard限制高
# nofile 文件描述符
* hard nofile 65536 # 表示任何一个用户可以打开的最大的文件描述符数量
* soft nofile 65536

# 修改sysctl.conf
vim /etc/sysctl.conf 
vm.max_map_count=655360 # max_map_count文件包含限制一个进程可以拥有的VMA(虚拟内存区域)的数量
# 加载参数vim
sysctl -p
```



### 安装证书

#### 创建证书

##### 创建一个证书颁发机构

```shell
./bin/elasticsearch-certutil ca
```

提示命名文件：直接回车，默认文件名elastic-stack-ca.p12文件 提示输入密码：可以直接回车，也可以输入密码进行设置

##### 为节点生成证书和私钥

```shell
./bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
```

提示命名文件，直接回车，默认文件名elastic-certificates.p12文件 提示输入密码：可以直接回车，也可以输入密码进行设置

##### config目录下创建下certs目录

```bash
mkdir config/certs 
```

##### 将文件可拷贝到certs目录下

```bash
mv elastic-certificates.p12 config/certs/
```

##### 给keystore和truststore设置密码

keystore是存储密钥（公钥、私钥）的容器。truststore其本质都是keystore。只不过二者盛放的密钥所有者不同而已，对于keystore一般存储自己的私钥和公钥，而truststore则用来存储自己信任的对象的公钥。

如果在创建证书的过程中加了密码，需要输入这个密码。每个节点都需要

```bash
# https证书
./bin/elasticsearch-keystore add xpack.security.http.ssl.keystore.secure_password
./bin/elasticsearch-keystore add xpack.security.http.ssl.truststore.secure_password

# 传输层ssl证书
./bin/elasticsearch-keystore add xpack.security.transport.ssl.keystore.secure_password
./bin/elasticsearch-keystore add xpack.security.transport.ssl.truststore.secure_password
```

这样就会在config目录下keystore文件了



### 启动服务

```shell
# su es 注意当前账号
./bin/elasticsearch -d
```



### 创建用户密码

集群中的节点都按照上面的方式完成配置并启动后，就可以设置账号密码了

1、自动创建密码

```shell
./bin/elasticsearch-setup-passwords auto
```

2、手动输入密码

```shell
./bin/elasticsearch-setup-passwords interactive
```

![image-20221228150433356](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221228150433356.png)

3、重置用户密码（随机密码）

```bash
./bin/elasticsearch-reset-password -u elastic
```

4、重置用户密码（指定密码）

```bash
./bin/elasticsearch-reset-password -u elastic -i <password>
```



### 修改kibana 认证

kibana中配置ES中配置的kibana账号密码即可连接ES认证

```yaml
elasticsearch.username: "kibana_system"
elasticsearch.password: "AAAaaa1234"
```

登录Kibana：http://192.168.52.129:5601/，账号/密码：elastic / AAAaaa1234