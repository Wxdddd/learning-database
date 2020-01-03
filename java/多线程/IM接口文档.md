## IM接口文档 - V1.0

### 1. 获取会话列表

请求方式：post

请求地址：/im/conv/list

请求参数：

```json
{
	"from_id": "CKD_P_903000000000001"
}
```

| 参数    | 说明     |
| ------- | -------- |
| from_id | 发送方id |

返回参数：

```json
[
    {
        "conv_id": "xxxxx1",
		"conv_type": 0,
        "conv_users": ["CKD_P_903000000000001","CKD_P_903000000000002"],
        "users_type": ["P","D"],
        "last_msg": "测试信息",
        "last_msg_type": 2,
        "last_user_name": "白亚玲",
        "last_time": 1577433689000,
        "unread_count": 0
    },
    {
        "conv_id": "xxxxx2",
		"conv_type": 1,
        "conv_users": ["CKD_P_903000000000001","CKD_P_903000000000003"],
        "last_msg": "测试信息",
        "last_msg_type": 2,
        "last_user_name": "白亚玲",
        "last_time": 1577433689000,
        "unread_count": 0
    }
    ...
]
```

| 参数           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| conv_id        | 会话id                                                       |
| conv_type      | 会话类型（0人 1 群）                                         |
| conv_users     | 会话相关用户                                                 |
| last_msg       | 会话的最后一条消息                                           |
| last_msg_type  | 会话最后一条消息的类型（0 文字 1 图片 2 语音 3 视频 4 文件） |
| last_user_name | 会话最后一条消息的发送人                                     |
| last_time      | 会话最后一条消息的时间                                       |
| unread_count   | 当前用户未读消息数量                                         |

### 2. 通过会话id获取会话消息列表

请求方式：post

请求地址：/im/conv/{conv_id}

请求参数：

```json
{
	"page_no": 1,
    "page_size": 10
}
```

| 参数      | 说明         |
| --------- | ------------ |
| conv_id   | 会话ID       |
| page_no   | 分页页数     |
| page_size | 分页每页数量 |

返回参数：

```json
[
    {
        "msg_id": "xxxxx1"
        "msg_type": 0,
        "msg": "文本测试",
        "conv_id": "xxxx1",
        "from_id": "CKD_P_903000000000001",
        "from_time": 1577336309973,
        "user" : {
        	"id" : "903000000000001",
        	"type": "P",
        	"img": "http://xxx.png",
        	"name": "安利英"
    	}
    },
    {
        "msg_id": "xxxxx2"
        "msg_type": 0,
        "msg": "http://leancloudfile.xuetoutong.com/8f01e7daaaee84d289cd.png",
        "conv_id": "xxxx1",
        "from_id": "CKD_P_903000000000002",
        "from_time": 1577336309973,
    	"user" : {
        	"id" : "903000000000001",
    		"type": "D",
        	"img": "http://xxx.png",
        	"name": "安利英"
    	}
    }
    ...
]
```

| 参数         | 说明                                           |
| ------------ | ---------------------------------------------- |
| msg_id       | 消息id                                         |
| msg_type     | 消息类型（0 文字 1 图片 2 语音 3 视频 4 文件） |
| msg          | 消息内容                                       |
| conv_id      | 会话id                                         |
| from_id      | 消息发送人id                                   |
| from_time    | 消息发送时间                                   |
| user -> id   | 用户id                                         |
| user -> type | 用户类型                                       |
| user -> img  | 用户头像                                       |
| user -> name | 用户名称                                       |

### 3. 通过发送者id与接受者id获取会话消息列表

请求方式：post

请求地址：/im/from/to

请求参数：

```json
{
    "from_id": "CKD_P_903000000000001",
    "to_id": "CKD_P_903000000000002",
	"page_no": 1,
    "page_size": 10
}
```

| 参数      | 说明         |
| --------- | ------------ |
| from_id   | 发送者id     |
| to_id     | 接受者id     |
| page_no   | 分页页数     |
| page_size | 分页每页数量 |

返回参数：

```json
[
    {
        "msg_id": "xxxxx1"
        "msg_type": 0,
        "msg": "文本测试",
        "conv_id": "xxxx1",
        "from_id": "CKD_P_903000000000001",
        "from_time": 1577336309973,
        "user" : {
        	"id" : "903000000000001",
        	"type": "P",
        	"img": "http://xxx.png",
        	"name": "安利英"
    	}
    },
    {
        "msg_id": "xxxxx2"
        "msg_type": 0,
        "msg": "http://leancloudfile.xuetoutong.com/8f01e7daaaee84d289cd.png",
        "conv_id": "xxxx1",
        "from_id": "CKD_P_903000000000002",
        "from_time": 1577336309973,
    	"user" : {
        	"id" : "903000000000001",
    		"type": "D",
        	"img": "http://xxx.png",
        	"name": "安利英"
    	}
    }
    ...
]
```

| 参数         | 说明                                           |
| ------------ | ---------------------------------------------- |
| msg_id       | 消息id                                         |
| msg_type     | 消息类型（0 文字 1 图片 2 语音 3 视频 4 文件） |
| msg          | 消息内容                                       |
| conv_id      | 会话id                                         |
| from_id      | 消息发送人id                                   |
| from_time    | 消息发送时间                                   |
| user -> id   | 用户id                                         |
| user -> type | 用户类型                                       |
| user -> img  | 用户头像                                       |
| user -> name | 用户名称                                       |

### 4. 消息发送

请求方式：post

请求地址：/im/send/msg

请求参数：

```json
{
	"conv_id": "xxxxx1",
    "msg": "文本测试",
    "msg_type": 0,
    "from_id": "CKD_P_903000000000001",
    "to_id": "CKD_P_903000000000002"
}
```

| 参数     | 说明     |
| -------- | -------- |
| conv_id  | 会话ID   |
| msg      | 消息内容 |
| msg_type | 消息类型 |
| from_id  | 发送者id |
| to_id    | 接受者id |

返回参数：

```json
{}
```

### 5. 创建群

请求方式：post

请求地址：/im/creat/group

请求参数：

```json
{
	"owner_id": "CKD_P_903000000000002",
    "name": "群1",
    "member": ["CKD_P_903000000000001","CKD_P_903000000000002",....],
    "img": "http://xxx.png",
    "description": "群1xxx"
    
}
```

| 参数        | 说明          |
| ----------- | ------------- |
| owner_id    | 创建者id      |
| name        | 群名称        |
| member      | 群成员 非必填 |
| img         | 群头像        |
| description | 群描述        |

返回参数：

```json
{}
```

### 6. 添加群成员

请求方式：post

请求地址：/im/add/group/merber

请求参数：

```json
{
	"conv_id": "CKD_P_903000000000002",
    "member": ["CKD_P_903000000000001","CKD_P_903000000000002",....]
}
```

| 参数    | 说明   |
| ------- | ------ |
| conv_id | 会话id |
| member  | 群成员 |

返回参数：

```json
{}
```

### 7. 查看群成员

请求方式：post

请求地址：/im/group/{conv_id}/list

请求参数：

```json
{
}
```

返回参数：

```json
[
    {
        "id": "CKD_P_903000000000002"
		"name": "xxx1",
        "alias": "yyy1",
	}
]
```



| 参数  | 说明     |
| ----- | -------- |
| id    | 成员id   |
| name  | 成员名称 |
| alias | 成员别名 |

