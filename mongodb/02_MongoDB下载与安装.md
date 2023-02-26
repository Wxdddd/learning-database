## MongoDB 的下载与安装

### 1、下载MongoDB

下载地址：https://www.mongodb.com/try/download/community

### 2、安装MongoDB 

#### 2.1 Linux 安装

在linux平台的MongoDB为解压版，我们只要戒烟tgz文件就可以使用。

![image-20220726125817969](https://winnxudong.oss-cn-shanghai.aliyuncs.com/images/image-20220726125817969.png)

**命令下载：**

```shell
[root@VM-0-6-centos ~]# wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.28.tgz
--2022-07-26 13:00:39--  https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.28.tgz
[root@VM-0-6-centos mongodb]# ll
total 84068
-rw-r--r-- 1 root root 86084214 Jan 25 07:51 mongodb-linux-x86_64-4.0.28.tgz
```

**解压并更换文件位置**

```shell
[root@VM-0-6-centos mongodb]# tar -zxvf mongodb-linux-x86_64-4.0.28.tgz 
[root@VM-0-6-centos mongodb]# ll
total 84072
drwxr-xr-x 3 root root     4096 Jul 26 13:04 mongodb-linux-x86_64-4.0.28
-rw-r--r-- 1 root root 86084214 Jan 25 07:51 mongodb-linux-x86_64-4.0.28.tgz
[root@VM-0-6-centos mongodb]# mv mongodb-linux-x86_64-4.0.28 /usr/local/mongodb
[root@VM-0-6-centos mongodb]# cd  /usr/local/mongodb/
[root@VM-0-6-centos mongodb]# ll
total 200
drwxr-xr-x 2 root root  4096 Jul 26 13:04 bin
-rw-r--r-- 1 root root 30608 Jan 25 07:17 LICENSE-Community.txt
-rw-r--r-- 1 root root 16726 Jan 25 07:17 MPL-2
-rw-r--r-- 1 root root  2601 Jan 25 07:17 README
-rw-r--r-- 1 root root 60005 Jan 25 07:17 THIRD-PARTY-NOTICES
-rw-r--r-- 1 root root 81355 Jan 25 07:17 THIRD-PARTY-NOTICES.gotools
[root@VM-0-6-centos mongodb]# pwd
/usr/local/mongodb
```

**数据持久化设置**

```shell
[root@VM-0-6-centos mongodb]# mkdir -p data/db
[root@VM-0-6-centos mongodb]# ll
total 204
drwxr-xr-x 2 root root  4096 Jul 26 13:04 bin
drwxr-xr-x 3 root root  4096 Jul 26 16:06 data
-rw-r--r-- 1 root root 30608 Jan 25 07:17 LICENSE-Community.txt
-rw-r--r-- 1 root root 16726 Jan 25 07:17 MPL-2
-rw-r--r-- 1 root root  2601 Jan 25 07:17 README
-rw-r--r-- 1 root root 60005 Jan 25 07:17 THIRD-PARTY-NOTICES
-rw-r--r-- 1 root root 81355 Jan 25 07:17 THIRD-PARTY-NOTICES.gotools
```

> MongoDB的数据存储在data目录的db目录下，但是这个目录在安装过程中不会自动创建，需要手动创建data/db目录。data目录可以创建在任何位置。

### 3、MongoDB的启动与关闭

#### 3.1 启动MongoDB

> 1. 前置启动
>
> 2. 后置启动
>
>    无论哪种启动方式都需要执行bin目录中的mongod命令。MongoDB在启动时默认的查找数据库的路径为/data/db。如果我们路劲有变化，需要在该命令中通过--dbpath参数来指定db目录的路劲（该路劲可以是相对路径，也可以是绝对路径）。

##### **前置启动**

```shell
[root@VM-0-6-centos mongodb]# bin/mongod --dbpath data/db/
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] MongoDB starting : pid=12499 port=27017 dbpath=data/db/ 64-bit host=VM-0-6-centos
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] db version v4.0.28
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] git version: af1a9dc12adcfa83cc19571cb3faba26eeddac92
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] allocator: tcmalloc
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] modules: none
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] build environment:
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten]     distarch: x86_64
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten]     target_arch: x86_64
2022-07-26T16:17:34.233+0800 I CONTROL  [initandlisten] options: { storage: { dbPath: "data/db/" } }
2022-07-26T16:17:34.233+0800 I STORAGE  [initandlisten] 
2022-07-26T16:17:34.234+0800 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2022-07-26T16:17:34.234+0800 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2022-07-26T16:17:34.234+0800 I STORAGE  [initandlisten] wiredtiger_open config: create,cache_size=256M,cache_overflow=(file_max=0M),session_max=20000,eviction=(threads_min=4,threads_max=4),config_base=false,statistics=(fast),log=(enabled=true,archive=true,path=journal,compressor=snappy),file_manager=(close_idle_time=100000),statistics_log=(wait=0),verbose=(recovery_progress),
2022-07-26T16:17:34.920+0800 I STORAGE  [initandlisten] WiredTiger message [1658823454:920432][12499:0x7fa084985a40], txn-recover: Set global recovery timestamp: 0
2022-07-26T16:17:34.942+0800 I RECOVERY [initandlisten] WiredTiger recoveryTimestamp. Ts: Timestamp(0, 0)
2022-07-26T16:17:34.958+0800 I STORAGE  [initandlisten] Starting to check the table logging settings for existing WiredTiger tables
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: Access control is not enabled for the database.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          Read and write access to data and configuration is unrestricted.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: This server is bound to localhost.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          Remote systems will be unable to connect to this server. 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          Start the server with --bind_ip <address> to specify which IP 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          addresses it should serve responses from, or with --bind_ip_all to
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          bind to all interfaces. If this behavior is desired, start the
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **          server with --bind_ip 127.0.0.1 to disable this warning.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/defrag is 'always'.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] ** WARNING: soft rlimits too low. rlimits set to 3878 processes, 100001 files. Number of processes should be at least 50000.5 : 0.5 times number of files.
2022-07-26T16:17:34.968+0800 I CONTROL  [initandlisten] 
2022-07-26T16:17:34.968+0800 I STORAGE  [initandlisten] createCollection: admin.system.version with provided UUID: ac2b97fe-4328-47d2-afaa-150ace40616b
2022-07-26T16:17:34.996+0800 I COMMAND  [initandlisten] setting featureCompatibilityVersion to 4.0
2022-07-26T16:17:34.997+0800 I STORAGE  [initandlisten] Finished adjusting the table logging settings for existing WiredTiger tables
2022-07-26T16:17:34.997+0800 I STORAGE  [initandlisten] createCollection: local.startup_log with generated UUID: 1317a1c8-7f98-4be6-9812-d933e592054e
2022-07-26T16:17:35.021+0800 I FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory 'data/db/diagnostic.data'
2022-07-26T16:17:35.025+0800 I NETWORK  [initandlisten] waiting for connections on port 27017
2022-07-26T16:17:35.027+0800 I CONTROL  [LogicalSessionCacheReap] Sessions collection is not set up; waiting until next sessions reap interval: config.system.sessions does not exist
2022-07-26T16:17:35.028+0800 I STORAGE  [LogicalSessionCacheRefresh] createCollection: config.system.sessions with generated UUID: 118fa036-d361-49bf-aa99-a49b67d860fc
2022-07-26T16:17:35.059+0800 I INDEX    [LogicalSessionCacheRefresh] build index on: config.system.sessions properties: { v: 2, key: { lastUse: 1 }, name: "lsidTTLIndex", ns: "config.system.sessions", expireAfterSeconds: 1800 }
2022-07-26T16:17:35.059+0800 I INDEX    [LogicalSessionCacheRefresh]     building index using bulk method; build may temporarily use up to 500 megabytes of RAM
2022-07-26T16:17:35.059+0800 I INDEX    [LogicalSessionCacheRefresh] build index done.  scanned 0 total records. 0 secs
```

##### **后置启动**

>所谓后置启动就是以守护进程的方式启动MongoDB。我们需要在执行mongod命令中添加 --fork参数。需要注意的是，--fork参数需要配合着--logpath 或者是-syslog参数使用。 --logpath与--syslog参数是指定MongoDB的日志文件。MongoDB的日志文件可以在系统中的任意位置。

```shell
# 创建log目录与存储日志的文件
[root@VM-0-6-centos mongodb]# mkdir log
[root@VM-0-6-centos mongodb]# cd log
[root@VM-0-6-centos log]# ls
[root@VM-0-6-centos log]# touch mongo.log
[root@VM-0-6-centos log]# ls
mongo.log
[root@VM-0-6-centos log]# cd ..
[root@VM-0-6-centos mongodb]# bin/mongod --dbpath data/db/ --logpath log/mongo.log --fork
about to fork child process, waiting until server is ready for connections.
forked process: 14453
child process started successfully, parent exiting
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     14453  2.0  4.9 1008136 49744 ?       Sl   16:29   0:01 bin/mongod --dbpath data/db/ --logpath log/mongo.log --fork
root     14688  0.0  0.0 112812   980 pts/1    S+   16:30   0:00 grep --color=auto mongod
```

##### **常见的启动参数**

| 参数        | 安静输出                                                     |
| ----------- | ------------------------------------------------------------ |
| --quiet     | 安静输出                                                     |
| --port      | 指定服务端口号，默认27017                                    |
| --bind      | 绑定服务ip，若绑定127.0.0.1，则只能本机访问                  |
| --logpath   | 指定MongoDB日志文件，注意是指定日志文件不是目录              |
| --logappend | 使用追加日志的方法                                           |
| --fork      | 守护进程的方式运行MongoDB，创建服务器进程                    |
| --auth      | 启用验证                                                     |
| --config    | 指定配置文件的路径，注意时指定配置文件不是目录               |
| --journal   | 启用日志选项，MongoDB的数据操作将会写入到journal文件夹的文件里 |

##### **配置文件启动**

> 如果觉得在启动MongoDB时给定的参数项太多，那么我们也可以通过配置文件来配置启动参数，配置文件可以在任意目录中，配置文件的扩展名应为conf，配置文件中使用 key-value结构。在执行MongoDB 时通过--config参数来指定需要加载的配置文件。

```shell
[root@VM-0-6-centos etc]# mkdir etc
[root@VM-0-6-centos etc]# cd etc/
[root@VM-0-6-centos etc]# vim mongo.conf
[root@VM-0-6-centos etc]# cd ..
# 查看MongoDB是否已经启动，若已启动先停止掉，不建议直接kill，kill时mongodb无法持久化数据。
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     14453  0.3  4.9 1008136 49940 ?       Sl   16:29   0:04 bin/mongod --dbpath data/db/ --logpath log/mongo.log --fork
root     17761  0.0  0.0 112812   980 pts/1    S+   16:48   0:00 grep --color=auto mongod
[root@VM-0-6-centos mongodb]# kill 14453
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     17811  0.0  0.0 112812   980 pts/1    S+   16:49   0:00 grep --color=auto mongod
[root@VM-0-6-centos mongodb]# bin/mongod --config etc/mongo.conf 
about to fork child process, waiting until server is ready for connections.
forked process: 17937
child process started successfully, parent exiting
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     17937  2.5  4.9 1008140 49984 ?       Sl   16:49   0:01 bin/mongod --config etc/mongo.conf
root     18154  0.0  0.0 112812   976 pts/0    S+   16:50   0:00 grep --color=auto mongod
```

```shell
#mongo.log 内容
port=27017
dbpath=/usr/local/mongodb/data/db
logpath=/usr/local/mongodb/log/mongo.log
fork=true
```

##### 配置环境变量

```shell
[root@VM-0-6-centos mongodb]# vim /etc/profile
# 添加内容
export PATH=/usr/local/mongodb/bin:$PATH
#  source /etc/profile 另外一种写法: . /etc/profile
[root@VM-0-6-centos mongodb]# source /etc/profile
[root@VM-0-6-centos mongodb]# mongod --help
Options:

General options:
  -v [ --verbose ] [=arg(=v)]           be more verbose (include multiple times
```

#### 3.2 关闭MongoDB

##### 使用 Ctrl+C 关闭

如果我们的启动方式是前置启动，那么直接使用快捷键Ctrl+C就可以关闭MongoDB这种关闭方式会等待当前进行中的的操作完成，所以依然是安全的关闭方式。

##### 使用kill 命令关闭

我们可以通过Linux的kill命令结束MongoDB进程，然后删除data目录中的mongod.lock文件，否则下次无法启动。但是此方法不建议使用，因为会造成数据损坏现象。

##### 使用MongoDB的函数关闭

在MongoDB中提供了两个关闭数据库的函数： 

db.shutdownServer()
db.runCommand("shutdown")
如上两个方法都需要在admin库中执行，并且都是安全的关闭方式。

```shell
[root@VM-0-6-centos mongodb]# bin/mongo
MongoDB shell version v4.0.28
connecting to: mongodb://127.0.0.1:27017/?gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("4b56aad4-705b-4b2b-a7f8-d3b018c638bb") }
MongoDB server version: 4.0.28
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
        http://docs.mongodb.org/
Questions? Try the support group
        http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2022-07-26T17:07:27.146+0800 I CONTROL  [initandlisten] 
# > use admin
switched to db admin
# > db.shutdownServer()
2022-07-26T17:09:33.265+0800 I NETWORK  [js] DBClientConnection failed to receive message from 127.0.0.1:27017 - HostUnreachable: Connection closed by peer
server should be down...
2022-07-26T17:09:33.267+0800 I NETWORK  [js] trying reconnect to 127.0.0.1:27017 failed
2022-07-26T17:09:33.267+0800 I NETWORK  [js] reconnect 127.0.0.1:27017 failed failed
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     21449  0.0  0.0 112812   976 pts/0    S+   17:10   0:00 grep --color=auto mongod
```

##### 使用MongoDB命令关闭

mongod --shutdown --dbpath <数据库路径>

```shell
[root@VM-0-6-centos mongodb]# ps aux | grep mongod
root     17937  0.3  4.9 1008140 50372 ?       Sl   16:49   0:03 bin/mongod --config etc/mongo.conf
root     20638  0.0  0.0 112812   980 pts/0    S+   17:05   0:00 grep --color=auto mongod
[root@VM-0-6-centos mongodb]# bin/mongod --shutdown --dbpath data/db/
killing process with pid: 17937
[root@VM-0-6-centos mongodb]# 
```

