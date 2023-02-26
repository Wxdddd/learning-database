## MongoDB的用户管理

### 1、MongoDB用户权限列表

| 命令                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| read                 | 允许用户读取指定数据库                                       |
| readWrite            | 允许用户读写指定数据库                                       |
| dbAdmin              | 允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问systemprofile |
| userAdmin            | 允许用户向systemusers集合写入，可以在指定数据库里创建、删除和管理用户 |
| elusterAdmin         | 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限 |
| readAnyDatabase      | 只在admin数据库中可用，赋予用户所有数据库的读权限            |
| readWriteAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读写权限          |
| userAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限     |
| dbAdminAnyDatabase   | 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限       |
| root                 | 只在admin数据库中可用。超级账号，超级权限                    |

### 2、MongoDB用户使用

> mongodb有一个用户管理机制，简单描述为，有一个管理用户组，这个组的用户是专门为管理普通用户而设的，暂且称之为管理员。
>
> 管理员通常没有数据库的读写权限，只有操作用户的权限，因此我们只需要赋予管理员userAdminAnyDatabase角色即可。
>
> 另外管理员账户必须在admin数据库下创建，3.0版本后没有admin数据库，但我们可以手动use一个。
>
> 注：use命令在切换数据库时，如果切换到一个不存在的数据库，MongodDB会自动创建该数据库。

##### 切换到admin数据库

```shell
[root@VM-0-6-centos ~]# mongo
MongoDB shell version v4.0.28
ould be at least 50000.5 : 0.5 times number of files.
2022-07-27T11:59:51.672+0800 I CONTROL  [initandlisten] 
> use admin
switched to db admin
```

##### 查看admin中的用户

```shell
> use admin
switched to db admin
> show users
> db.system.users.find()
```

> 目前admin库中没有用户，所以查询无结果。

##### 创建用户

在MongoDB 中可以使用db.createUser({用户信息})函数来创建用户

```json
db.createUser({
	user: "<name>",
	pwd: "<cleartext password>",
	customData: { <any information> }， 
  roles:[
			{ role: "<role>", db: "<database>" } | "<role>",
      ....
	]
})
```

- user：新建用户名。
- pwd：新建用户密码。
- customData：存放一些用户相关的自定义数据，该属性也可忽略。
- roles：数组类型，配置用户的权限。db表示哪个数据库，role表示什么权限。

```json
> db.createUser({
 user: "winn",
 pwd: "AAAaaa123", 
   roles:[
 { 'role': "userAdminAnyDatabase", 'db': "admin" }
 ]
 })
Successfully added user: {
        "user" : "winn",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ]
}
```

开启权限认证模式

> mongo.conf 文件中添加auth=true参数

```shell
[root@VM-0-6-centos ~]# vim /usr/local/mongodb/etc/mongo.conf
port=27017
dbpath=/usr/local/mongodb/data/db
logpath=/usr/local/mongodb/log/mongo.log
fork=true
auth=true
[root@VM-0-6-centos ~]# mongod --shutdown --dbpath /usr/local/mongodb/data/db/
killing process with pid: 10573
[root@VM-0-6-centos ~]# mongod -config /usr/local/mongodb/etc/mongo.conf
about to fork child process, waiting until server is ready for connections.
forked process: 15378
child process started successfully, parent exiting
[root@VM-0-6-centos ~]# ps aux | grep mongod
root     15378 15.2  4.8 1008140 49436 ?       Sl   12:28   0:01 mongod -config /usr/local/mongodb/etc/mongo.conf
root     15438  0.0  0.0 112812   980 pts/0    S+   12:29   0:00 grep --color=auto mongod
```

验证是否启动认证模式

```shell
[root@VM-0-6-centos ~]# mongo
MongoDB shell version v4.0.28
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("0461203a-eb0c-4364-b688-682745266e58") }
MongoDB server version: 4.0.28
> use test
switched to db test
> db.tt.insert({'a':'123'})
WriteCommandError({
        "ok" : 0,
        "errmsg" : "command insert requires authentication",
        "code" : 13,
        "codeName" : "Unauthorized"
})
```

> 报错 command insert requires authentication 认证开启成功

登录认证并验证写权限

```shell
# 在test库中认证失败，必须在创建时指定的admin数据库中认证。
> db.auth('winn', 'AAAaaa123')
Error: Authentication failed.
0
> use admin
switched to db admin
> db.auth('winn', 'AAAaaa123')
1
> show users
{
        "_id" : "admin.winn",
        "userId" : UUID("05e5f319-22bf-422b-8b39-77a912df8e81"),
        "user" : "winn",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
> db.tt.insert({'a':'123'})
WriteCommandError({
        "ok" : 0,
        "errmsg" : "not authorized on admin to execute command { insert: \"tt\", ordered: true, lsid: { id: UUID(\"0461203a-eb0c-4364-b688-682745266e58\") }, $db: \"admin\" }",
        "code" : 13,
        "codeName" : "Unauthorized"
})
```

> 因为winn账户只设定了账号管理权限，所以它无法插入数据。

创建只针对test数据库有读写权限的用户

```shell
> db.createUser({'user':'test', 'pwd':'AAAaaa123','roles':[{'role':'readWrite', 'db':'test'}]})
Successfully added user: {
        "user" : "test",
        "roles" : [
                {
                        "role" : "readWrite",
                        "db" : "test"
                }
        ]
}
# 先退出再登录认证
> exit
bye
[root@VM-0-6-centos ~]# mongo
MongoDB shell version v4.0.28
> use test
switched to db test
> db.auth('test',  'AAAaaa123')
1
> db.tt.insert({'a':'123'})
WriteResult({ "nInserted" : 1 })
> db.tt.find()
{ "_id" : ObjectId("62e0c320010e9d0454e26faf"), "a" : "123" }
```

##### 更新用户

> 如果我们需要对已存在的用户的角色做修改，那么我们可以使用db.updateUser()函数来更新用户。该函数需要当前用户具有userAdminAnyDatabase或者更高级别的权限才可以进行操作。

语法格式

```shell
db.updateUser("roles":[{"role": "角色名称"}],  {"更新项2": "更新内容"})
```

winn账户只具备userAdminAnyDatabase用户管理角色，我们为其添加一个dbAdminAnyDatabase数据库管理角色。

```shell
> use admin
switched to db admin
> db.auth("winn", "AAAaaa123")
1
> db.updateUser("winn", {"roles": [{"role": "userAdminAnyDatabase", "db": "admin"}, {"role": "dbAdminAnyDatabase",  "db": "admin"}]})
> show users
{
        "_id" : "admin.winn",
        "userId" : UUID("05e5f319-22bf-422b-8b39-77a912df8e81"),
        "user" : "winn",
        "db" : "admin",
        "roles" : [
                {
                        "role" : "userAdminAnyDatabase",
                        "db" : "admin"
                },
                {
                        "role" : "dbAdminAnyDatabase",
                        "db" : "admin"
                }
        ],
        "mechanisms" : [
                "SCRAM-SHA-1",
                "SCRAM-SHA-256"
        ]
}
```

更新密码

> 两种方式：
>
> 1. db.updateUser("用户名", {"pwd": "新密码"})
> 2. db.changeUserPassword("用户名", "新密码")

```shell
#重新登录mongo客户端
> use admin
switched to db admin
> db.auth("winn", "AAAaaa123")
1
> db.changeUserPassword("winn", "123123")
> exit
bye
[root@VM-0-6-centos ~]# mongo
> use admin
switched to db admin
> db.auth("winn", "123123")
1
```

##### 删除用户

> 通过db.dropUser("用户名")函数可删除指定用户。删除成功后会返回true。
>
> 在删除用户时需要切换到创建时用户所指定的数据库中才可以删除。
>
> 注意：需要使用具有userAdminAnyDatabse角色管理员用户才可以删除其他用户。

删除test数据库的test用户

```shell
> use admin
switched to db admin
> db.auth("winn", "123123")
1
> use test
switched to db test
> db.dropUser("test")
true
```

