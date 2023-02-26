## Elastic App Search



### 介绍

通过一套完善的 API 以及直观的仪表板，Elastic App Search 为公司网站、电子商务网站或应用带来了 Elasticsearch 的强大功能。获得无缝的可扩展性、可调的相关性控制、详尽的文档、经过良好维护的客户端和强大的分析能力，为您的客户打造领先的搜索体验。https://www.elastic.co/guide/en/app-search/8.4/getting-started.html

![image-20221229092101044](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229092101044.png)

### 特点

简单理解就是：工具箱，提供各种搜索需要的功能；开箱即用，快速创建搜索服务，具体如下：

特点：

- 快速搭建：不限平台，不限数据，不限位置
- 功能强大：相关性高，容错性优异
- 搜索分析：实时数据，可付诸实践的见解
- 相关性调整：使用直观界面调校搜索的相关性。创建同义词组，针对特定查询对结果进行重排序，以及分配权重和系数，这些都有助于对整体精准度进行微调。



### 功能

- 分析功能：App Search会收集用户搜索时留下的大量行为数据，经过分析能够帮助我们优化搜索服务。App Search可捕获关键指标，并提供各种用于将数据可视化的工具。您不仅可以通过低廉的价格存储数据，改进搜索体验并对其进行个性化设置，还能充分利用各种趋势，并消除产品、服务或内容方面的空白。
- Machine Learning：App Search 采用的是最新的 Machine Learning 和自然语言处理技术。提供的功能灵活且易于实施，您可以借助工具在应用程序中构建语义和图像搜索、个性化和问题回答功能，从而显著改善搜索体验。
- 数据采集：App Search不仅支持用户自己上传JSON文档数据，而且支持网络爬虫抓取数据和编写上传脚本
- 相关性和个性化搜索：凭借强大的开箱即用型搜索相关性，Elastic 提供了构建卓越搜索体验所需的所有工具，可以帮助用户准确找到所需的内容。丰富的相关性调优工具和先进的 Machine Learning，可帮助您深入进行分析、微调和个性化定制。
- 部署灵活性：只需单击几下，即可在您钟爱的云中或跨多个云为您的应用程序添加搜索功能，您也可以探索 App Search功能强大的部署选项。支持在云中、本地部署或混合环境中托管，并可灵活地构建完全契合当前（或所需）基础架构设置的搜索体验。
- Search UI：只需几行代码，即可创建搜索界面和可配置的搜索组件。利用丰富的筛选功能，您可以构建完全可定制的搜索体验，帮助用户准确找到所需的内容。



### 使用场景

![image-20221229093103445](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229093103445.png)



### Elastic App Search 架构

![image-20221229093142621](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229093142621.png)

![image-20221229093208585](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229093208585.png)

### 操作流程

![image-20221229093239102](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229093239102.png)



## 部署 App Search（Kibana中访问）

部署 App Search，它由 Enterprise Search、Kibana 和 Elasticsearch Servers 组成，因此首先把Elasticsearch， Kibana安装好，https://www.elastic.co/guide/en/app-search/current/installation.html，然后它还依赖java环境，所以**要安装jdk11**。

1. 下载地址：https://www.elastic.co/cn/downloads/enterprise-search，可以使用`wget`进行下载
2. 解压：`tar -zxvf`
3. 配置App Search：`vim ./config/enterprise-search.yml`

```yaml
allow_es_settings_modification: true

elasticsearch.host: http://192.168.52.129:9200
#
# Elasticsearch credentials:
#
elasticsearch.username: elastic
elasticsearch.password: AAAaaa1234 # 注意这里密码不能是纯数字

# kibanavim
kibana.host: http://192.168.52.129:5601

# 定义用户访问应用程序搜索的公开URL
ent_search.external_url: http://192.168.52.129:3002
# 监听ip和端口设置
ent_search.listen_host: 192.168.52.129
ent_search.listen_port: 3002
```

4. 修改env.sh的jvm改小些

```shell
vim ./config/env.sh
export JAVA_OPTS=${JAVA_OPTS:-"-Xms1g -Xmx1g"}
```

5. 配置Elasticsearch：`vim ./config/elasticsearch.yml`

```shell
# 启用Elasticsearch API密钥服务
xpack.security.authc.api_key.enabled: true
# 允许自动创建索引
action.auto_create_index: ".app-search-*-logs-*,-.app-search-*,+*"
```

6. 启动Elasticsearch服务
7. 在Kibana添加如下配置

```yaml
enterpriseSearch.host: 'http://192.168.52.129:3002'
```

8. 启动Kibana
9. 启动app seasrch 服务

```shell
./bin/enterprise-search
```

启动时，会提示如下错误：

![image-20221229100945672](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229100945672.png)

这时在修改app search的配置文件，添加请求秘钥

```shell
secret_management.encryption_keys: [dca4ad4d1b32575afcb65c623e894b7c90d0559e222a5f5f2d9297e552ffd38e]
```

再重新启动

10. Kibana中查看





## 检索（API接口）



### API鉴权

https://www.elastic.co/guide/en/app-search/8.4/authentication.html

当其他应用比如搜索UI或与App Search集成的另一个应用程序通过API去访问App Search时都必须要要经过权限校验才能访问。

此处的API Keys有两种：

- Private API Key：私有API密钥，默认的API访问密钥可以对除凭据和Web爬网程序用户代理之外的所有可用API端点进行读写。前缀为private-。
- Public Search Key：公共搜索密钥，默认API读取密钥对搜索端点具有只读搜索访问权限。前缀为search-

### 搜索API

API文档地址 https://www.elastic.co/guide/en/app-search/8.4/search.html，搜索的API接口为：

```http
GET <ENTERPRISE_SEARCH_BASE_URL>/api/as/v1/engines/<engine_name>/search
```

或者

```http
POST <ENTERPRISE_SEARCH_BASE_URL>/api/as/v1/engines/<engine_name>/search
```

其中：

- **ENTERPRISE_SEARCH_BASE_URL**： 为请求URL地址
- **engine_name**：为App Search中引擎名字

**访问时都要携带访问Token**

#### 请求参数

- `query` ：查询条件，**必填**
- `page` ：分页数据，选填，是JSON对象，https://www.elastic.co/guide/en/app-search/8.4/search-guide.html#search-guide-paginate，比如：

```json
"page": {
	"current": 10, # 当前页
	"size": 1000	# 每页多少条，默认10条
}
```

- `sort` ：排序，https://www.elastic.co/guide/en/app-search/8.4/sort.html，也是一个JSON对象：

```json
"sort": {
    "title": "desc" # title为排序字段，desc倒序，asc顺序
}

# 或者多个条件
"sort": [
    { "_score": "desc" }, _score是一个匹配评分数字
    { "title": "desc" }
]
```

- `search_fields`：搜索字段 https://www.elastic.co/guide/en/app-search/8.4/search-fields-weights.html，里面可以包含查询过滤的字段，还可以设置字段权重提高搜素结果准确度，比如：

```json
"query": "everglade"，	# 检索内容
"search_fields": { # 从如下字段中进行检索
    "title": {
       "weight": 6 # 设置权重，0-10之间
     },
    "description": {
       	"weight": 2
    },
    "states": {
       	"weight": 1
    }
 }
```

- `result_fields`： 结果显示的字段https://www.elastic.co/guide/en/app-search/8.4/result-fields-highlights.html，包含如下两种：

   （1）Raw（原始）: 字段中值的精确表示。这是准确的！它不是HTML转义的。

   （2）Snippet（高亮）: 代码段是字段中值的HTML转义表示，其中查询匹配项在**标记中捕获。**

```json
"query": "everglade",
"result_fields": {
    "title": {
        "raw": {} # 原始文本
    },
    "description": {
        "raw": {
            "size": 50	# 长度为50字符，可选，设置size时，至少要20字符，默认是整个文本长度
        },
        snippet": {
        	"size": 20, # 可选，设置size时，至少要20字符，默认是100字符
        	"fallback": true # 如果为true，则如果未找到snippet代码段，则返回原始文本字段内容。如果为false，则仅使用snippet代码片段，如果没有则为null
    	}
	}
}
```

或者

```json
"query": "everglade",
"result_fields": {
    "title": {
        "snippet": {
            "size": 20, # 可选，设置size时，至少要20字符，默认是100字符
            "fallback": true # 如果为true，则如果未找到snippet代码段，则返回原始文本字段内容。如果为false，则仅使用snippet代码片段，如果没有则为null
        }
    },
    "description": {
        "raw": {
            "size": 200
        },
        "snippet": {
            "size": 100
        }
    },
    "states": {
        "raw" : {},
        "snippet": {
            "size": 20,
            "fallback": true
        }
    }
}
```

#### 响应结果

- `results`：搜索结果。与搜索匹配的结果数组。每个结果包括一个文档和元数据。当分页超过10000个结果时，数组为空。

- `meta`：元数据信息。有如下属性：

   （1）meta.request_id：表示请求的字符串ID。保证独一无二。将ID与API日志API一起使用以搜索API请求。

   （2）meta.warnings：查询的警告数组。包含在错误响应和成功响应中，因此请检查所有响应中的警告。

   （3）meta.alerts：部署的警报阵列。包括错误响应和成功响应，因此请检查所有警报响应。

   （4）meta.page：分页信息。meta.page.current：当前页；meta.page.size：每页数量；meta.page.total_pages：总页数，meta.page.total_results：结果总数，**当结果总数超过10000时，搜素结果为空。**

### 曲库搜索（示例）

通过匹配歌曲名、歌手、歌词、专辑名、作词作曲人等字段，搜索出包含关键字的歌曲，同时结果只显示歌曲、播放地址、歌手、专辑信息

![image-20221229135353149](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229135353149.png)

```http
curl -X POST 'http://192.168.52.129:3002/api/as/v1/engines/music-search/search' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer search-h58drmesqkqxz9971krrqt6g' \
-d '{
	"query": "寻找周杰伦",
	"search_fields": {"name": {"weight": 6},"singer": {"weight": 3}, "lyric": {"weight": 1},"album": {"weight": 1},"co": {"weight": 2}},
	"result_fields": {"id": {"raw": {} },"name": {"raw": {}},"url": {"raw": {} }, "singer": {"raw": {}},"album": {"raw": {} }},"sort": [{ "_score": "desc" },{"location": "asc"},{"date": "desc"}]}'
```

### 自动补全功能

![image-20221229135458384](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20221229135458384.png)

自动补全的文档：https://www.elastic.co/guide/en/app-search/8.4/query-suggestion.html

API接口：

```http
GET/POST <ENTERPRISE_SEARCH_BASE_URL>/api/as/v1/engines/{ENGINE_NAME}/query_suggestion
```

适用范围：只适用于文本字段

参数：

**`query` (required)**：匹配关键字，必填

**`types` (optional)**：匹配的文本字段，默认是所有文本字段

**`size` (optional)**：返回结果的数量，必须是1-20之间，默认是10

比如：

```http
curl -X POST 'http://192.168.52.129:3002/api/as/v1/engines/national-parks-demo/query_suggestion' \
-H 'Content-Type: application/json' \
-H 'Authorization: Bearer search-h58drmesqkqxz9971krrqt6g' \
-d '{
 	"query": "晴天",
	"types": {
		"documents": {
	      "fields": [
	        "name",
	    	"singer",
	    	"co"
	      ]
	    }
	},
  	"size": 3
}'
```