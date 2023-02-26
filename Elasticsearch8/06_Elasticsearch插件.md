## 插件

### 插件简介

插件是用户以自定义的方式增强Elasticsearch功能的一种方法。它们可以为es添加自定义映射类型、自定义分词器、原生脚本、自动伸缩等等扩展功能。插件包含jar文件，可能包含脚本和配置文件，并且必须安装在群集中的每个节点上。安装后，必须重新启动每个节点才能看到插件。主要包含**核心插件**和**社区贡献插件（第三方插件）**。

**注意：es的插件安装需要使用root权限，因为会涉及到一些文件写入的权限操作。**

#### 核心插件

核心插件属于Elasticsearch项目的插件。与Elasticsearch项目同时发布，并且其版本号始终与Elasticsearch发布的版本号相同。这些插件由Elastic官方团队维护。在使用过程中，如果遇到问题，可以在Github项目页面上报告问题和错误报告（GitHub搜索Elasticsearch即可查看）。

在GitHub的Elasticsearch中，核心插件列表如下：

https://github.com/elastic/elasticsearch/tree/main/plugins

#### 第三方插件

第三方插件属于Elasticsearch项目外部的插件，由个人或者第三方公司自主开发，并拥有各自许可证以及各自运行的版本控制系统。在使用第三方插件时，如遇到问题则在第三方社区网站上进行查询或者提交issue。



### 插件管理

官方插件相关的文档地址：https://www.elastic.co/guide/en/elasticsearch/plugins/master/plugin-management.html，具体插件的安装、删除、列表等都可以在此查看。

插件的命令位于elasticsearch的bin目录下，执行以下命令即可查看帮助文档。

```shell
[es@elasticsearch1 elasticsearch-8.4.3]$ pwd
/home/elasticsearch/elasticsearch-8.4.3
[es@elasticsearch1 elasticsearch-8.4.3]$ bin/elasticsearch-plugin -h
A tool for managing installed elasticsearch plugins

Commands
--------
list - Lists installed elasticsearch plugins
install - Install a plugin
remove - removes plugins from Elasticsearch

Non-option arguments:
command

Option             Description
------             -----------
-E <KeyValuePair>  Configure a setting
-h, --help         Show help
-s, --silent       Show minimal output
-v, --verbose      Show verbose output

```

#### 安装插件

**（1）命令安装**

安装核心插件时，可以使用如下命令

插件的命令位于**elasticsearch的bin目录**下，执行以下命令即可查看帮助文档。

```shell
bin/elasticsearch-plugin install [plugin_name]
```

比如：安装 [ICU plugin](https://www.elastic.co/guide/en/elasticsearch/plugins/master/analysis-icu.html)（ICU Analysis插件是一组将Lucene ICU模块集成到Elasticsearch中的库。 本质上，ICU的目的是增加对Unicode和全球化的支持，以提供对亚洲语言更好的文本分割分析），只需要执行以下命令：

```shell
bin/elasticsearch-plugin install analysis-icu
```

**注意：插件安装好以后一定要记得重启elasticsearch的服务才能生效**

**（2）离线安装**

在安装命令后面，也可以跟一个url的参数，这个url可以是本地路径，也可以是一个http的访问路径，命令如下：

```shell
bin/elasticsearch-plugin install [path]
```

其中url如果为本地路径需要注意：

- Windows环境下具体命令为：

```shell
bin\elasticsearch-plugin install file:///C:/path/to/plugin.zip
```

其中`C:/path/to/plugin.zip`指创建的存放目录

- Unix环境，则命令为：

```shell
bin/elasticsearch-plugin install file:///path/to/plugin.zip
```

其中`path/to/plugin.zip`为插件存放目录

**注意：**

 **1、这两种安装方式，如果后面的路径包含空格，那么必须用引号把路径引起来**

 **2、如果采用离线安装，那么插件不得包含在要安装插件的节点的插件目录中，否则安装将失败**

**（3）URL安装**

在上述离线（本地）安装的命令中后面的path可以为一个url地址

```shell
bin/elasticsearch-plugin install [url]
```

比如采用url地址安装ICU plugin插件，那么命令如下：

```shell
bin/elasticsearch-plugin install https://artifacts.elastic.co/downloads/elasticsearch-plugins/analysis-icu/analysis-icu-7.3.2.zip
```

注意：如果插件地址是不受信任的证书与HTTPS ，那么插件脚本将拒绝使用。要使用自签名HTTPS证书，您需要将CA证书添加到本地Java信任库，并将位置传递到脚本，比如：

```shell
ES_JAVA_OPTS="-Djavax.net.ssl.trustStore=/path/to/trustStore.jks" bin/elasticsearch-plugin install https://host/plugin.zip
```

#### 插件列举、删除

1. 列举安装好的插件

```shell
bin/elasticsearch-plugin list
```

2. 移除插件

```shell
bin/elasticsearch-plugin remove [plugin_name]
```

注意：

1、移除以后记得重启elasticsearch服务，才会生效

2、默认情况下，插件配置文件（如果有）保存在磁盘上；这是为了在升级插件时不会丢失配置。如果希望在删除插件时清除配置文件，请使用-p或–purge。删除插件后，可以使用此选项删除任何遗留的配置文件。

### Head插件

在学习和使用Elasticsearch的过程中，必不可少需要通过一些工具查看es的运行状态以及数据。如果都是通过rest请求，未免太过麻烦，而且也不够人性化。head可以完美帮我们快速学习和使用es。官网：

- https://github.com/mobz/elasticsearch-head
- https://github.com/mobz/elasticsearch-head/releases

从官方的文档中我们知道，在5.0版本以上不在提供插件安装形式，而是独立的运行项目。

#### 安装

第一步：下载源码

https://github.com/mobz/elasticsearch-head

第二步：安装node环境

```shell
[root@elasticsearch1 elasticsearch-8.4.3]# mkdir /opt/softwares
mkdir: cannot create directory ‘/opt/softwares’: File exists
[root@elasticsearch1 elasticsearch-8.4.3]# mkdir /opt/softwares/plugins
[root@elasticsearch1 elasticsearch-8.4.3]# mkdir /opt/softwares/plugins/node
## 下载并解压node，本地目录(/opt/softwares)，如果没联网也可以从软件包中，本地上传到此目录
cd /opt/softwares/plugins/node
wget https://nodejs.org/download/release/latest-v14.x/node-v14.21.1-linux-x64.tar.xz
## 解压
tar -xvf node-v14.21.1-linux-x64.tar.xz

## 配置环境变量
vi /etc/profile
export NODE_HOME=/opt/softwares/plugins/node/node-v14.21.1-linux-x64
export PATH=$PATH:$NODE_HOME/bin

## 配置生效
source /etc/profile

## 查看版本
node -v
```

第三步：修改配置

```shell
mkdir -p /usr/local/elasticsearch/plugins
cd /usr/local/elasticsearch/plugins
git clone https://github.com/mobz/elasticsearch-head.git
cd elasticsearch-head
vi Gruntfile.js ## 97行添加 hostname: '*',
vi _site/app.js ## 4374行 将localhost修改为 192.168.52.129（服务ip）
```

第四步：启动head插件

编辑elasticsearch.yml文件，在文件中添加

```shell
http.cors.enabled: true
http.cors.allow-origin: "*"
```

允许es跨域访问配置。重启elasticsearch，然后在执行

```shell
cd elasticsearch-head
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install --ignore-scripts -- ## 打包命令
npm run start ## 启动命令
```

第四步：访问http://192.168.52.129:9100

#### Head插件使用

##### 连接ES服务器

在界面中右侧输入需要连接的ES集群服务器地址和端口，然后点击【连接】如果右侧出现green意味着ES为正常健康状态



##### 创建student索引

点击【索引】菜单 -> 新建索引 ->输入索引名称（student）-> 点击OK即可

![image-20220408065908933](http://www.imooc.com/wiki/第3章-Elasticsearch项目实战-音乐曲库搜索.assets/image-20220408065908933.png)

**注意：当ES集群为打个节点时，创建完索引以后集群健康状态为黄色，因为单点部署Elasticsearch, 默认的分片副本数目配置为1，而相同的分片不能在一个节点上，所以就存在副本分片指定不明确的问题，所以显示为yellow，可以通过在Elasticsearch集群上添加一个节点来解决问题。**

##### 删除student索引

在概览中，找到student索引，然后点击【动作】 -> 【删除】

##### 添加数据

在【复合查询】中输入插入的api地址以及对应的请求参数



```json
{
    "took": 27,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 3,
            "relation": "eq"
        },
        "max_score": 1,
        "hits": [
            {
                "_index": "user",
                "_id": "--EaUoUBIl0Jt_AkviKZ",
                "_score": 1,
                "_source": {
                    "uname": "admin",
                    "passwd": "123456",
                    "introduce": "勇敢 坚强 正直",
                    "createtime": "2020-11-11 11:11:11"
                }
            },
            {
                "_index": "user",
                "_id": "_OEaUoUBIl0Jt_Ak9yIB",
                "_score": 1,
                "_source": {
                    "uname": "zhangsan",
                    "passwd": "111111",
                    "introduce": "勇敢 果断 正义",
                    "createtime": "2020-11-11 11:11:11"
                }
            },
            {
                "_index": "user",
                "_id": "_eEbUoUBIl0Jt_AkDSJp",
                "_score": 1,
                "_source": {
                    "uname": "lisi",
                    "passwd": "222222",
                    "introduce": "聪明 坚强 善良",
                    "createtime": "2020-11-11 11:11:11"
                }
            }
        ]
    }
}
```

##### 插入数据指定ID

默认\_id是自动生成的，但如果需要指定，那么在_doc后面添加id值

```json
{
  "name": "赵六",
  "gender": "男",
   "age": 58,        
  "introduce": "我是赵六",
  "createtime": "2030-04-30"
}
```

![image-20221227154817866](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221227154817866.png)

##### 查询数据

**（1）must，must not，should的区别**

- **must** 返回的文档必须满足must子句的条件，类似于 **== and**
- **must not**返回的文档必须不满足must not 子句的条件 类似于 **!= not**
- **should** 返回的文档只要满足should中的一个条件即可 类似于 **|| or**

**（2）各类查询参数**

- **match**查询的时候，elasticsearch会根据你给定的字段提供合适的分析器
- **term** 把检索串当作一个整体来执行检索，即不会对检索串分词
- **text** 会将检索词进行分词，然后对数据进行匹配
- **prefix** 就是前缀检索。比如商品name中有多个以"Java"开头的document, 检索前缀"Java"时就能检索到所有以"Java"开头的文档
- **wildcard** 通配符查询，通配符检索，例如 java\*，\*java*
- **fuzzy** 纠错检索，其中min_similarity设置匹配的最小相似度，默认值为0.5，对于字符串，取值为0-1(包括0和1)，对于数值，取值可能大于1，对于日期型取值为1d，1m等，1d就代表1天；max_expansions：查询中的词项可以扩展的数目，默认可以无限大。例如：检索name中包含"Java"的文档， Java中缺失了一个字母a
- **range** 区间查询，如果type是时间类型，可用内置now表示当前，-1d/h/m/s来进行时间操作
- **query_string**  可以对int，long，string查询，对int，long只能本身查询，对string进行分词和本身查询，比如 name:张三 或者 name.keyword:张三
- **missing**  返回没有字段或值为null的文档

### Kibana插件

Kibana 是一款开源的数据分析和可视化平台，它是 Elastic Stack 成员之一，设计用于和 Elasticsearch 协作。https://www.elastic.co/guide/en/kibana/current/get-started.html

您可以使用 Kibana 对 Elasticsearch 索引中的数据进行搜索、查看、交互操作。可以很方便的利用图表、表格及地图对数据进行多元化的分析和呈现。

Kibana 可以使大数据通俗易懂。它很简单，基于浏览器的界面便于您快速创建和分享动态数据仪表板来追踪 Elasticsearch 的实时数据变化。

搭建 Kibana 非常简单。可以分分钟完成 Kibana 的安装并开始探索 Elasticsearch 的索引数据——没有代码、不需要额外的基础设施。

> graph
> 在Elasticsearch数据中显示并分析相关关系。
>
> discover
> 通过查询和过滤原始文档以交互方式浏览数据。
>
> visualize
> 在Elasticsearch索引中创建可视化并聚合数据存储。
>
> dashboard
> 显示并共享可视化和保存的搜索的集合。
>
> canvas
> 以像素完美的方式展示您的数据。
>
> maps
> 探索来自Elasticsearch和Elastic Maps Service的地理空间数据。
>
> machine learning
> 自动对时间序列数据的正常行为建模以检测异常。
>
> infrastructure
> 探索基础结构指标和常见服务器，容器和服务的日志。
>
> logs
> 实时流式记录日志或在类似控制台的体验中滚动浏览历史视图。
>
> APM
> 从应用程序内部自动收集深入的性能指标和错误。
>
> uptime
> 执行端点运行状况检查和正常运行时间监视。
>
> SIEM
> 探索安全指标并记录事件和警报。
>
> Console
> 跳过cURL并使用此JSON接口直接处理您的数据。
>
> Index Patterns
> 管理有助于从Elasticsearch检索数据的索引模式。
>
> Monitoring
> 跟踪弹性堆栈的实时运行状况和性能。
>
> Rollups
> 将历史数据汇总并存储在较小的索引中，以供将来分析。
>
> Saved Objects
> 导入，导出和管理您保存的搜索，可视化和仪表板。
>
> Security Settings
> 保护您的数据并轻松管理哪些用户可以访问用户和角色。
>
> Spaces
> 将仪表板和其他保存的对象组织到有意义的类别中。
>
> Watcher
> 通过创建，管理和监视警报来检测数据中的更改。
>
> Dev Tools
> 开发工具
>
> metrics
> 从服务器上运行的操作系统和服务收集指标。
>
> management
> 管理索引，索引模式，保存的对象，Kibana设置等

官网：https://www.elastic.co/products/kibana

下载地址：https://www.elastic.co/cn/downloads/kibana

#### 安装

https://www.elastic.co/guide/en/kibana/current/install.html

第一步：上传（下载）并解压（目录：/usr/local/elasticsearch/plugins）

```shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.1.2-linux-x86_64.tar.gz
tar -zxvf kibana-8.1.2-linux-x86_64.tar.gz
cd kibana-8.1.2
```

第二步：修改Kibana配置文件

```shell
vi config/kibana.yml
```

修改一下内容：

```yaml
# 服务端口，默认5601
server.port: 5601
# 允许访问IP
server.host: "0.0.0.0"
# 设置 elasticsearch 节点及端口
elasticsearch.hosts: ["http://192.168.52.129:9200"]
# 设置中文界面
i18n.locale: "zh-CN"
```

第三步：启动kibana（需要先启动es）

```shell
 ./bin/kibana --allow-root
```

注意：可以将Kibana添加到系统服务，通过系统服务方式启动

```shell
nohup ./bin/kibana --allow-root > ./logs/kibana-nhup.log 2>&1 &
```

第四步：访问 http://192.168.52.129:5601/

查看进程：`ps -ef|grep node`
