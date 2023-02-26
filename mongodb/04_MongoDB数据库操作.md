## MongoDB数据库操作

### 1、创建数据库

在MongoDB中创建数据库的命令使用的是use命令。该命令有两层含义：

- 切换到指定数据库。
- 如果切换的数据库不存在，则创建该数据库。

我们使用use命令创建一个名为test的数据库

```shell
> use test
switched to db sxttest
```

### 2、查看所有数据库

我们可以通过show dbs命令查看当前MongoDB中的所有数据库。

如果开启了用户认证，则需要先登录方可查看到结果，否则不显示任何信息。

如果使用的是具备数据库管理员角色的用户，那么则可以看到MongoDB中的所有数据库，如果使用的普通用户登录的那么只能查询到该用户所拥有的数据库。 

```shell
> use admin
switched to db admin
> db.auth("winn", "123123")
1
> show  dbs
admin   0.000GB
config  0.000GB
local   0.000GB
test    0.000GB
```

### 3、删除数据库

> 在MongoDB 中使用db.dropDatabase()函数来删除数据库。
>
> 在删除数据库之前，需要使用具备dbAdminAnyDatabase角色的管理员用户登录，然后切换到需要删除的数据库，执行 db.dropDatabase()函数即可。
>
> 删除成功后会返回一个{"ok":1 }的 JSON 字符串。

我们现在将刚刚创建的test删除。

```shell
> use admin
switched to db admin
> db.auth("winn", "123123")
1
> use test
switched to db test
> db.dropDatabase()
{ "dropped" : "test", "ok" : 1 }
> show  dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```

