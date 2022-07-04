### Doker快速应用搭建

```shell
# 查看是否有运行的docker镜像
[root@VM-0-6-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 查看docker镜像
[root@VM-0-6-centos ~]# docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
```

### 1. HelloWorld

```shell
# 直接运行hello world镜像，首先找本地镜像有没有hello world，如果本地没有则从网上去拉取hello world
[root@VM-0-6-centos ~]# docker run hello-world

# 因为hello-world是执行完成后就停止的容器，所以docker ps查询不到
[root@VM-0-6-centos ~]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

# 查询docker曾经存留过的容器以及这个容器的运行状态及其结果
[root@VM-0-6-centos ~]# docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED         STATUS                     PORTS     NAMES
d8d91b116fcf   hello-world   "/hello"   7 minutes ago   Exited (0) 7 minutes ago             objective_lehmann
```

### 2. Redis

```shell
# 使redis3.2版本以后台的方式运行。-d：后台方式运行。:3.2表示镜像版本号（不指定版本号会默认下载最新的版本）。redis-server：redis以server的方式运行起来。
[root@VM-0-6-centos ~]# docker run -d redis:3.2 redis-server

[root@VM-0-6-centos ~]# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS      NAMES
27e81846c296   redis:3.2   "docker-entrypoint.s…"   22 seconds ago   Up 21 seconds   6379/tcp   dreamy_buck

# 使用redis-cli与docker镜像里面的redis进行交互。如果希望使用bash交互将redis-cli替换为bash即可。 27e81846c296：镜像id。
[root@VM-0-6-centos ~]# docker exec -it 27e81846c296 redis-cli
127.0.0.1:6379>

```

### 3. Nginx

```shell
# -p 80:80 ：-p表示端口映射，第一个80表示服务器端口，第二个80是容器内部端口，也就是镜像80端口映射到服务器80端口
[root@VM-0-6-centos ~]# docker run -d -p 80:80 nginx

[root@VM-0-6-centos ~]# docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED          STATUS          PORTS                               NAMES
01a754814f4c   nginx       "/docker-entrypoint.…"   2 minutes ago    Up 2 minutes    0.0.0.0:80->80/tcp, :::80->80/tcp   eager_dijkstra
```

### 4. Dockerfile

vi Dockerfile

```shell
# 容器基本镜像
FROM debian
# 作者
MAINTAINER winn
# 执行mkdir test1命令
RUN mkdir test1
RUN touch test2
# 将本地test3文件添加到容器的根目录下
COPY test3 .
# 将本地test4.rar文件添加到容器的根目录下，并且会解压
ADD test4.tar.gz .
# 打开容器的80端口 一般在容器启动的过程中设定
#EXPOSE 80
ENTRYPOINT ["/bin/sh"]
CMD ["-c", "ls -l"]
```

```shell
[root@VM-0-6-centos home]# touch test
[root@VM-0-6-centos home]# touch test3
[root@VM-0-6-centos home]# mkdir test4
[root@VM-0-6-centos home]# tar -czvf test4.tar.gz test4
[root@VM-0-6-centos home]# docker build -t mysh .
[root@VM-0-6-centos home]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
mysh          latest    bf3a997b1a5a   6 seconds ago   124MB
debian        latest    d2780094a226   6 days ago      124MB
hello-world   latest    feb5d9fea6a5   9 months ago    13.3kB
[root@VM-0-6-centos home]# docker run mysh
total 72
drwxr-xr-x  2 root root 4096 Jun 22 00:00 bin
drwxr-xr-x  2 root root 4096 Mar 19 13:46 boot
drwxr-xr-x  5 root root  340 Jun 29 06:23 dev
drwxr-xr-x  1 root root 4096 Jun 29 06:23 etc
drwxr-xr-x  2 root root 4096 Mar 19 13:46 home
drwxr-xr-x  8 root root 4096 Jun 22 00:00 lib
drwxr-xr-x  2 root root 4096 Jun 22 00:00 lib64
drwxr-xr-x  2 root root 4096 Jun 22 00:00 media
drwxr-xr-x  2 root root 4096 Jun 22 00:00 mnt
drwxr-xr-x  2 root root 4096 Jun 22 00:00 opt
dr-xr-xr-x 95 root root    0 Jun 29 06:23 proc
drwx------  2 root root 4096 Jun 22 00:00 root
drwxr-xr-x  3 root root 4096 Jun 22 00:00 run
drwxr-xr-x  2 root root 4096 Jun 22 00:00 sbin
drwxr-xr-x  2 root root 4096 Jun 22 00:00 srv
dr-xr-xr-x 13 root root    0 Jun 29 05:52 sys
drwxr-xr-x  2 root root 4096 Jun 29 06:23 test1
-rw-r--r--  1 root root    0 Jun 29 06:23 test2
-rw-r--r--  1 root root    0 Jun 29 06:18 test3
drwxr-xr-x  2 root root 4096 Jun 29 06:18 test4
drwxrwxrwt  2 root root 4096 Jun 22 00:00 tmp
drwxr-xr-x 11 root root 4096 Jun 22 00:00 usr
drwxr-xr-x 11 root root 4096 Jun 22 00:00 var
[root@VM-0-6-centos home]# docker run mysh -c date
Wed Jun 29 06:26:02 UTC 2022
# -c date 会抵消Dockerfile中的 CMD ["-c", "ls -l"]
```

