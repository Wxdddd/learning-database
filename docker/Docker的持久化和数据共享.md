## Docker的持久化和数据共享

### 1. Docker持久化数据的方案

- `基于本地文件系统的Volume`。可以在执行Docker create或Docker run时，通过`-v`参数将主机的目录作为容器的数据卷。这部分功能便是基于本地文件系系统的volume管理。

- `基于plugin的Volume`，支持第三方的存储方案，比如NAS，aws。

### 2. Volume的类型

- 受管理的data Volume，由docker后台自动创建。
- 绑定挂载的Volune，具体挂载位置可以由用户指定。

### 3. Data Volume 

#### 创建mysql容器

```shell
[vagrant@docker-host ~]$ sudo docker run -d --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
```

#### 查看docker volume

```shell
[vagrant@docker-host ~]$ docker volume ls
DRIVER              VOLUME NAME
local               f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee
[vagrant@docker-host ~]$ docker volume inspect f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee
[
    {
        "CreatedAt": "2019-11-16T02:41:04Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee/_data",
        "Name": "f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee",
        "Options": null,
        "Scope": "local"
    }
]
```

> 会发现多出一个VOLUME，这个就是mysql容器的外部数据卷。
>
> mysql容器外部数据卷目录：
>
> /var/lib/docker/volumes/f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee/_data

#### 删除mysql容器

```shell
[vagrant@docker-host ~]$ docker stop mysql
mysql
[vagrant@docker-host ~]$ docker rm mysql
mysql
```

#### 再次查看volume

```shell
[vagrant@docker-host ~]$ docker volume ls
DRIVER              VOLUME NAME
local               f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee
```

> 可以发现容器删除后，volume还是存在的。这样确保了数据丢失的问题。

#### 删除volume

```shell
[vagrant@docker-host ~]$ docker volume rm f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee
f1c4118d0960cb957470a149af3c4f46fcb1f501ed7e675c0babb6dd364b5eee
```

#### 创建容器并且指定volume名称与容器内部存放数据的路径

```shell
[vagrant@docker-host ~]$ sudo docker run -d -v mysql:/var/lib/mysql --name mysql -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
f57c6983d307a66db5597a2720236321b95d3744455b07770784499cc11f6c8c
[vagrant@docker-host ~]$ docker volume ls
DRIVER              VOLUME NAME
local               mysql
[vagrant@docker-host ~]$ docker volume inspect mysql
[
    {
        "CreatedAt": "2019-11-16T02:53:14Z",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/mysql/_data",
        "Name": "mysql",
        "Options": null,
        "Scope": "local"
    }
]
```

#### volume的复用

```shell
[vagrant@docker-host ~]$ docker stop mysql
mysql
[vagrant@docker-host ~]$ docker rm mysql
mysql
[vagrant@docker-host ~]$ sudo docker run -d -v mysql:/var/lib/mysql --name mysql2 -e MYSQL_ALLOW_EMPTY_PASSWORD=true mysql
9334e575f98766a56b37d84a800c2673137289d3cc49e7e74c200d9b9173b167
[vagrant@docker-host ~]$ docker volume ls
DRIVER              VOLUME NAME
local               mysql
```

>删除mysql容器 创建mysql2容器指定使用mysql容器的volume
>
>实践步骤：
>
>1. 进入mysql容器的mysql数据库中添加一个`数据库`
>2. 退出mysql容器并删除mysql容器
>3. 创建mysql2容器并指定volume为mysql的volume
>4. 进入mysql2容器的mysql数据库中查看 1 操作的数据库是否存在

### 4. Bind Mouting

> `将本地文件目录与容器文件目录进行绑定/映射，本地文件目录下文件修改 容器文件目录下文件也会修改，容器文件目录下文件修改 本地文件目录下文件也会修改。`

#### Dockerfile创建镜像

##### dockerfile文件

```shell
# this same shows how we can extend/change an existing official image from Docker Hub
FROM nginx:latest
# highly recommend you always pin versions for anything beyond dev/learn
WORKDIR /usr/share/nginx/html
# change working directory to root of nginx webhost
# using WORKDIR is prefered to using 'RUN cd /some/path'
COPY index.html index.html
# I don't have to specify EXPOSE or CMD because they're in my FROM
```

##### index.html文件

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>hello</title>
</head>
<body>
  <h1>Hello Docker! </h1>
</body>
</html>
```

##### 创建镜像

```shell
[vagrant@docker-host docker-nginx]$ docker build -t wenxudong/nginx .
Sending build context to Docker daemon  3.072kB
Step 1/3 : FROM nginx:latest
latest: Pulling from library/nginx
8d691f585fa8: Pull complete 
5b07f4e08ad0: Pull complete 
abc291867bca: Pull complete 
Digest: sha256:922c815aa4df050d4df476e92daed4231f466acc8ee90e0e774951b0fd7195a4
Status: Downloaded newer image for nginx:latest
 ---> 540a289bab6c
Step 2/3 : WORKDIR /usr/share/nginx/html
 ---> Running in 602a1a4c4dec
Removing intermediate container 602a1a4c4dec
 ---> 745743e7e234
Step 3/3 : COPY index.html index.html
 ---> c8191f8fedba
Successfully built c8191f8fedba
Successfully tagged wenxudong/nginx:latest
[vagrant@docker-host docker-nginx]$ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED              SIZE
wenxudong/nginx     latest              c8191f8fedba        About a minute ago   126MB
nginx               latest              540a289bab6c        3 weeks ago          126MB
```

#### 创建容器并且映射文件

```shell
# 查看当前目录下的文件
[vagrant@docker-host docker-nginx]$ ls
Dockerfile  index.html
# 创建目录
[vagrant@docker-host docker-nginx]$ docker run -d -p 80:80 -v $(pwd):/usr/share/nginx/html --name web wenxudong/nginx
35eb4452adcea5e6932838a69d840efe7ed0fe065535f4d18e98ed47074821e5
```

> `-v $(pwd):/usr/share/nginx/html`将当前目录映射到web容器的/usr/share/nginx/html目录

#### 验证Bind Mouting

```shell
# 进入容器
[vagrant@docker-host docker-nginx]$ docker exec -it web /bin/bash
root@35eb4452adce:/usr/share/nginx/html# cd /usr/share/nginx/html
root@35eb4452adce:/usr/share/nginx/html# ls
Dockerfile  index.html
root@35eb4452adce:/usr/share/nginx/html# touch test.txt
root@35eb4452adce:/usr/share/nginx/html# ls
Dockerfile  index.html	test.txt
# 退出容器
root@35eb4452adce:/usr/share/nginx/html# exit
exit
# 再次查看当前目录下的文件
[vagrant@docker-host docker-nginx]$ ls
Dockerfile  index.html  test.txt
[vagrant@docker-host docker-nginx]$ touch test2.txt
# 进入容器
[vagrant@docker-host docker-nginx]$ docker exec -it web /bin/bash
root@35eb4452adce:/usr/share/nginx/html# ls
Dockerfile  index.html	test.txt  test2.txt
```

