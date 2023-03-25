## Filnk常用命令

### 1. 查看正在运行的任务

```shell
root@79200e52ad69:/opt/flink# flink list -r
Waiting for response...
No running jobs.
```



### 2. 查看所有任务

```shell
root@79200e52ad69:/opt/flink# flink list -a
Waiting for response...
No running jobs.
No scheduled jobs.
---------------------- Terminated Jobs -----------------------
05.03.2023 10:49:08 : f1705248f96363ebe846707741aaea85 : WordCount (FINISHED)
--------------------------------------------------------------
```



### 3. 查看计划任务

```shell
root@79200e52ad69:/opt/flink# flink list -s
Waiting for response...
No scheduled jobs.
```



### 4. 停止任务

```shell
root@79200e52ad69:/opt/flink# flink cancel f1705248f96363ebe846707741aaea85 (任务id)
Cancelling job f1705248f96363ebe846707741aaea85.
```



### 5. 提交任务

```shell
  Syntax: run [OPTIONS] <jar-file> <arguments>
  "run" action options:

	 -c,--class <classname>                     main() 包路径	
     
     -C,--classpath <url>                       依赖包路径
     
     -d,--detached                              
     -n,--allowNonRestoredState                 
     
     -p,--parallelism <parallelism>             并行度
     
     -s,--fromSavepoint <savepointPath>         
     
root@79200e52ad69:/opt/flink# flink run -c com.winn.flink.basic.PKFlinkSocketWCApp -C /home/jar/winn-flink-basic-1.0-SNAPSHOT.jar --host hadoop --port 9527
```



## Flink安装

https://nightlies.apache.org/flink/flink-docs-master/docs/deployment/resource-providers/standalone/overview/