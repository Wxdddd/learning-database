## ES的REST API接口

Elasticsearch公开了UI组件使用的REST API，可以直接调用这些API来配置和访问Elasticsearch功能。Elasticsearch所有的API接口地址：

https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html

### 索引CRUD命令

- 创建时命名全部小写，不能使用_开头，中间不能使用,

- 创建Index

  ```shell
  [es@elasticsearch1 bin]$ curl -XPUT http://elasticsearch1:9200/lucky?pretty
  {
    "acknowledged" : true,
    "shards_acknowledged" : true,
    "index" : "lucky"
  }
  
  ```

- 获取索引

  ```shell
  [es@elasticsearch1 bin]$ curl -XGET http://elasticsearch1:9200/lucky?pretty
  {
    "lucky" : {
      "aliases" : { },
      "mappings" : { },
      "settings" : {
        "index" : {
          "routing" : {
            "allocation" : {
              "include" : {
                "_tier_preference" : "data_content"
              }
            }
          },
          "number_of_shards" : "1",
          "provided_name" : "lucky",
          "creation_date" : "1672119533893",
          "number_of_replicas" : "1",
          "uuid" : "F8ZFue0GQZWmVo_05Oqmng",
          "version" : {
            "created" : "8040399"
          }
        }
      }
    }
  }
  
  ```

- 删除索引

  ```shell
  [es@elasticsearch1 bin]$ curl -XDELETE http://elasticsearch1:9200/lucky?pretty
  {
    "acknowledged" : true
  }
  ```



### 文档CRUD命令

第一步：创建user索引

```shell
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json" -XPUT http://elasticsearch1:9200/user?pretty -d '{"settings":{"number_of_shards":1, "number_of_replicas":1}, "mappings":{"dynamic":"false","properties":{"uname": {"type":"keyword"},"passwd":{"type":"keyword","index":false},"introduce": {"type":"text"},"createtime":{"type":"date","format":"yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"}}}}'
{
  "acknowledged" : true,
  "shards_acknowledged" : true,
  "index" : "user"
}
```

第二步：插入数据

```shell
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json" -XPOST http://elasticsearch1:9200/user/_doc?pretty -d \
> '{"uname" : "admin","passwd" : "123456","introduce" : "勇敢 坚强 正直","createtime":"2020-11-11 11:11:11"}'

{
  "_index" : "user",
  "_id" : "--EaUoUBIl0Jt_AkviKZ",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json" -XPOST http://elasticsearch1:9200/user/_doc?pretty -d \
> '{"uname" : "zhangsan","passwd" : "111111","introduce" : "勇敢 果断 正义","createtime" :"2020-11-11 11:11:11"}'

{
  "_index" : "user",
  "_id" : "_OEaUoUBIl0Jt_Ak9yIB",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json" -XPOST http://elasticsearch1:9200/user/_doc?pretty -d \
>  '{"uname" : "lisi","passwd" : "222222","introduce" : "聪明 坚强 善良","createtime":"2020-11-11 11:11:11"}'

{
  "_index" : "user",
  "_id" : "_eEbUoUBIl0Jt_AkDSJp",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 2,
  "_primary_term" : 1
}

```

第三步：插入数据指定ID

```shell
[es@elasticsearch1 bin]$  curl -H "Content-Type: application/json" -XPUT http://elasticsearch1:9200/user/_doc/1?pretty -d '{
"uname" : "wangwu",
"passwd" : "333333",
"introduce" : "聪明 坚强 善良",
"createtime" : "2020-11-11 11:11:11"
}'

{
  "_index" : "user",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 3,
  "_primary_term" : 1
}
```

第四步：查询数据信息

```shell
//查询index
[es@elasticsearch1 bin]$ curl -XGET http://elasticsearch1:9200/user/_search?pretty

{
  "took" : 575,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 4,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "user",
        "_id" : "--EaUoUBIl0Jt_AkviKZ",
        "_score" : 1.0,
        "_source" : {
          "uname" : "admin",
          "passwd" : "123456",
          "introduce" : "勇敢 坚强 正直",
          "createtime" : "2020-11-11 11:11:11"
        }
      },
      {
        "_index" : "user",
        "_id" : "_OEaUoUBIl0Jt_Ak9yIB",
        "_score" : 1.0,
        "_source" : {
          "uname" : "zhangsan",
          "passwd" : "111111",
          "introduce" : "勇敢 果断 正义",
          "createtime" : "2020-11-11 11:11:11"
        }
      },
      {
        "_index" : "user",
        "_id" : "_eEbUoUBIl0Jt_AkDSJp",
        "_score" : 1.0,
        "_source" : {
          "uname" : "lisi",
          "passwd" : "222222",
          "introduce" : "聪明 坚强 善良",
          "createtime" : "2020-11-11 11:11:11"
        }
      },
      {
        "_index" : "user",
        "_id" : "1",
        "_score" : 1.0,
        "_source" : {
          "uname" : "wangwu",
          "passwd" : "333333",
          "introduce" : "聪明 坚强 善良",
          "createtime" : "2020-11-11 11:11:11"
        }
      }
    ]
  }
}

// 按照ID查询文档
[es@elasticsearch1 bin]$ curl -XGET http://elasticsearch1:9200/user/_doc/1?pretty

{
  "_index" : "user",
  "_id" : "1",
  "_version" : 1,
  "_seq_no" : 3,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "uname" : "wangwu",
    "passwd" : "333333",
    "introduce" : "聪明 坚强 善良",
    "createtime" : "2020-11-11 11:11:11"
  }
}

```

第五步：修改数据

```shell
--------------------------------------------------------------直接替换
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json"  -XPUT http://elasticsearch1:9200/user/_doc/1?pretty -d '{ "uname":"world" }'

{
  "_index" : "user",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 4,
  "_primary_term" : 1
}


--------------------------------------------------------------局部替换
[es@elasticsearch1 bin]$ curl -H "Content-Type: application/json"  -XPOST http://elasticsearch1:9200/user/_doc/1?pretty -d '{"uname": "hello world","age":18}'

{
  "_index" : "user",
  "_id" : "1",
  "_version" : 3,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 5,
  "_primary_term" : 1
}

[es@elasticsearch1 bin]$ curl -XGET http://elasticsearch1:9200/user/_doc/1?pretty

{
  "_index" : "user",
  "_id" : "1",
  "_version" : 3,
  "_seq_no" : 5,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "uname" : "hello world",
    "age" : 18
  }
}

```

第六步：删除数据

```shell
[es@elasticsearch1 bin]$ curl -XDELETE http://elasticsearch1:9200/user/_doc/1?pretty
{
  "_index" : "user",
  "_id" : "1",
  "_version" : 4,
  "result" : "deleted",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 6,
  "_primary_term" : 1
}


[es@elasticsearch1 bin]$ curl -XGET http://elasticsearch1:9200/user/_search?pretty

{
  "took" : 840,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "user",
        "_id" : "--EaUoUBIl0Jt_AkviKZ",
        "_score" : 1.0,
        "_source" : {
          "uname" : "admin",
          "passwd" : "123456",
          "introduce" : "勇敢 坚强 正直",
          "createtime" : "2020-11-11 11:11:11"
        }
      },
      {
        "_index" : "user",
        "_id" : "_OEaUoUBIl0Jt_Ak9yIB",
        "_score" : 1.0,
        "_source" : {
          "uname" : "zhangsan",
          "passwd" : "111111",
          "introduce" : "勇敢 果断 正义",
          "createtime" : "2020-11-11 11:11:11"
        }
      },
      {
        "_index" : "user",
        "_id" : "_eEbUoUBIl0Jt_AkDSJp",
        "_score" : 1.0,
        "_source" : {
          "uname" : "lisi",
          "passwd" : "222222",
          "introduce" : "聪明 坚强 善良",
          "createtime" : "2020-11-11 11:11:11"
        }
      }
    ]
  }
}

```