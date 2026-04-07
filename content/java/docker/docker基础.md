---
date: '2026-03-30T13:22:41+08:00'
draft: false
slug: '8e44ea1e'
type: posts
title: 'Docker基础'
collections: ""
categories: []
---

<!--more-->

## 基本概念

### 镜像
镜像是一个固定的只读模版，包含运行某个应用程序所需要的所有内容，包括运行时环境，系统依赖等。

类比 Java 的话，镜像就是 Java 里的类，容器就是 Java 里的对象

#### 镜像的本质

镜像由多层构成，包括基础运行层、运行时层、依赖层、应用层，
然后容器在镜像层之上再加一层可写层


#### 基础命令

拉取镜像： `docker pull nginx:latest`

删除镜像： `docker rmi nginx:latest`  需要先删除使用该镜像构建的容器

查看本地镜像： `docker images`

查看镜像详细信息：`docker inspect nginx`


#### 镜像分层与缓存机制

Dockerfile的每一个命令呢都会形成一层镜像的镜像层，对于指令相同且上下文相同的镜像层将不再构建，而是复用缓存,因此Dockerfile，它构建的一个原则就是变化少的步骤放在前面，变化多的步骤放在后面



#### docker file
构建镜像的文件。通过这个文件就可以构建出一个镜像

docker engine 会从上到下一行一行读取文件内容，
每条指令都会创建一个新的**镜像层（Layer）**。
一步一步打包成一个标准的docker镜像

##### 语法
基础指令
`FROM` 选择一个镜像作为基础镜像。 这是任何一个dockerfile的开头，比如`FROM python:3.9`

`WORKDIR`  设定工作目录， 指定在容器里的那个目录进行操作 `WORKDIR /app`


文件操作
`COPY`  将本地文件复制到容器工作目录内， `COPY . .` （把当前目录的所有文件，拷贝到镜像的当前工作目录下） 

`VOLUME` 声明挂载点，用于持久化数据  `VOLUME ["/data"]`

 执行与构建
`RUN`  构建时执行的命令 比如安装依赖 `RUN pip install -r requirements.txt` 
`ENV`设置环境变量  `ENV APP_HOME=/app`


启动与网络

`CMD` 容器启动时的 默认执行的第一条命令  `CMD ["python", "main.py"]`   一个 Dockerfile 里通常只有最后一个 `CMD` 会生效。

`ENTRYPOINT` 容器启动时的入口命令

构建命令:
`docker build -t my-app:v1 .`  -t 执行镜像名与标签  `.` 表示 当前目录下所有的文件都交给docker engine 去作为上下文进行构建







###  容器
容器是镜像的运行实例，也就是一个，运行起来的应用程序。

容器之间相互隔离，拥有自己独立的文件系统，网络和进程空间


本质是镜像层加可写层




创建并运行容器： `docker run -d -p 8080:80 --name my-nginx nginx`  -d detach 分离 是后台运行 -p 宿主机：容器， 指定端口。   --name 指定容器名称  


run 命令常用参数：
```
-d          # 后台运行
-p 宿主:容器  # 端口映射
-v 宿主:容器  # 数据卷挂载
--name      # 指定容器名
--network   # 指定网络
--env / -e  # 设置环境变量
--rm        # 退出后自动删除
--restart   # 重启策略（no/always/on-failure）
--memory    # 内存限制
--cpus      # CPU 限制
-u          # 指定用户
```



查看运行中的容器： `docker ps`

查看所有容器（包括停止的）： `docker ps -a`     -a all 全部

查看容器日志： `docker logs -f my-nginx`     -f  follow 跟随，持续显示终端的新日志

进入容器内部： `docker exec -it my-nginx /bin/bash`  -i  interactive 交互模式  -t tty 终端模式进入


启动已停止的容器： `docker start my-nginx`

停止容器；  `docker stop my-nginx`  

删除容器： `docker rm my-nginx`

### 仓库
用于集中管理、存放镜像的服务器。 
我们可以通过它来获取别人分享的镜像


### docker engine

![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260407100025044.png)


Docker的架构主要包括Docker client，它是Docker命令行工具，，用户交互的入口将命令转化为 RESTful 发给守护进程


然后Docker的守护进程负责Docker的协调整体运行，包括镜像管理、容器管理、网络管理、数据源管理

OCI 运行时层，负责容器生命周期的管理

内核支持层（Linux Kernel） 




### 与虚拟机的区别

虚拟机用虚拟化技术模拟硬件，在硬件的基础上安装一个完整的操作系统，资源占用较大，启动缓慢

而docker不需要硬件和完整操作系统，所有容器通过docker engine 共享宿主机操作系统内核。是隔离进程、网络、文件系统，启动更快，资源占用小













## 构建一个自己的镜像

新建目录并进入：
`mkdir docker-demo & cd docker-demo`
`nano index.html` 
填入 
```html
<!-- index.html -->
<html>
  <body>
    <h1>我的第一个 Docker 镜像！</h1>
  </body>
</html>
```

`nano Dockerfile` 
填入
```
FROM nginx:latest
COPY index.html /usr/share/nginx/html/index.html
```

`docker build -t my-nginx:v1 .`

`docker run -d -p 8080:80 --name docker-demo my-nginx:v1`

访问 8080端口



## docker 镜像分层

是指，docker的镜像是文件系统一层一层堆叠而成。
dockerfile中，每一个命令都是一次对文件系统的修改，都是一层

镜像一旦构建好，这些层次就固定了，是只读的。
在容器里也是 镜像层 （只读）+ 可读写层 
```
|-------------------------|
|  层 4 (最顶层)           |  <- RUN apt-get install vim
|-------------------------|
|  层 3                    |  <- COPY . /app
|-------------------------|
|  层 2                    |  <- ENV APP_HOME=/app
|-------------------------|
|  层 1 (基础层)           |  <- FROM ubuntu:20.04
|-------------------------|
```

### 优点
- 复用：这样实现了一些镜像的复用，比如基础层 ubuntu只需要下载一次，而且只会占用一个ubuntu的内存内存 ，并且在构建时，已构建的层就不需要再构建了。 极大提高了构建速度和内存利用

### 写时复制机制

那镜像层是只读的，但是容器要写入数据，要怎么写呢。
docker 在所有镜像层的最上层添加了一个可读写层

那如果要修改比较底层镜像层的文件呢？
docker会先把底层镜像层的文件复制到可读写层
对这个复制的文件进行修改
下层的文件保持不动
```
|-------------------------|
|  容器层 (可读写)         |  <- 修改后的文件 (副本)
|-------------------------|
|  镜像层 3 (只读)         |  <- 原始文件
|-------------------------|
|  镜像层 2 (只读)         |
|-------------------------|
|  镜像层 1 (只读)         |
|-------------------------|
```


### 相关命令

`docker history my-nginx:v1` 查看镜像构建历史（看分层）

`docker inspect my-nginx:v1` 查看镜像详细信息


## 数据卷

问题： 容器删除后，数据全部都被删除
解决：使用docker数据卷

数据卷的好处：
可以直接实现持久化以及共享，使用数据卷后，数据就不会随着容器的删除而被删除，而是会保存在设定的路径中。如果容器共享数据卷，多个容器可以指向同一个数据卷，从而实现数据的共享。


数据卷的挂载方式推荐总共有两种，

### 数据卷挂载
第一种是定义出数据卷之后再去绑定  叫做Named Volume 数据卷挂载

这种方式呢由docker来对数据卷进行全权的管理，包括该数据卷的创建，删除，备份，迁移等
在宿主机的绑定路径都是由Docker进行管理
你只需要去填入这个数据卷叫什么名字即可，其他的都不需要你来管

这样交给docker来进行管理的好处是
- 第一它的安全性非常高，因为由docker来决定绑定的路径在哪里，那么他就不会去选择一些比较危险的文件，而如果你自己去指定的话，是有可能会选择一些比较危险的文件
- 第二个是它的可移植性非常好，比如说你的这个服务现在要迁移到另一个服务器上，那么你只需要在这个服务器上再执行一次创建同名的数据卷的命令即可，而不需要去找这个服务器上是否有这个与同名的路径

```shell
# 创建一个命名 Volume
docker volume create mydata

# 查看 volume 列表
docker volume ls

# 查看 volume 详情（注意 Mountpoint 字段）
docker volume inspect mydata

# 运行容器并挂载 volume
docker run -it --name vol-test -v mydata:/app/data ubuntu:latest /bin/bash

# 在容器内写入数据
mkdir -p /app/data
echo "我是持久化数据" > /app/data/important.txt
cat /app/data/important.txt
exit

# 删除容器
docker rm vol-test

# 用新容器挂载同一个 volume
docker run -it --name vol-test2 -v mydata:/app/data ubuntu:latest /bin/bash

# 数据还在！
cat /app/data/important.txt
exit

# 清理
docker rm vol-test2
```







### 绑定挂载
第二种是直接通过-V参数来直接进行绑定，这种方式叫做绑定挂载


绑定挂载这种方式与前面的数据卷挂载方式最大的不同是，它需要你去指定该数据卷所在在宿主机的目录，需要你手动去指定一个绝对的路径。

这种方式其实并不安全，因此不推荐在生产上使用，一般是在开发中会比较常使用

在开发中可以直接知道这个数据源所在宿主机的目录，然后就可以直接对目录中的文件进行修改，从而实现同步。

```shell
# 在宿主机创建目录和文件
mkdir ~/docker-web
echo "<h1>版本 1.0</h1>" > ~/docker-web/index.html

# 挂载本地目录到容器
docker run -d -p 8080:80 --name bind-test \
  -v ~/docker-web:/usr/share/nginx/html \
  nginx:latest

# 验证
curl http://localhost:8080
# 输出：<h1>版本 1.0</h1>

# 在宿主机修改文件（不需要重启容器！）
echo "<h1>版本 2.0 - 热更新！</h1>" > ~/docker-web/index.html

# 再次验证，内容已更新
curl http://localhost:8080
# 输出：<h1>版本 2.0 - 热更新！</h1>

# 清理
docker stop bind-test && docker rm bind-test
```


### 实战：MySQL 数据持久化

```
# 创建 volume 存储 MySQL 数据
docker volume create mysql-data

# 运行 MySQL 容器
docker run -d \
  --name my-mysql \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e MYSQL_DATABASE=testdb \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# 等待几秒让 MySQL 启动
docker logs my-mysql   # 看到 ready for connections 即可

# 进入 MySQL 创建数据
docker exec -it my-mysql mysql -u root -p123456

# 在 MySQL 内执行：
USE testdb;
CREATE TABLE users (id INT, name VARCHAR(50));
INSERT INTO users VALUES (1, 'Docker 学习者');
SELECT * FROM users;
exit

# 删除容器（模拟灾难）
docker stop my-mysql && docker rm my-mysql

# 用同一个 volume 恢复容器
docker run -d \
  --name my-mysql-restored \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -v mysql-data:/var/lib/mysql \
  -p 3306:3306 \
  mysql:8.0

# 等待启动后验证数据
docker exec -it my-mysql-restored mysql -u root -p123456

# 在 MySQL 内执行：
USE testdb;
SELECT * FROM users;   # 数据还在！
exit

# 清理
docker stop my-mysql-restored && docker rm my-mysql-restored
	docker volume rm mysql-data
```

### 相关命令
```
docker volume create mydata
docker run -v mydata:/data nginx
docker run -v $(pwd)/data:/data nginx    # Bind Mount
docker volume ls / inspect / rm
```


## 网络体系

### 为什么需要Docker网络

因为容器本身是相互隔离的，但在实际的应用环境中，我们更希望容器之间可以进行通信。比如说，我们希望后端应用可以访问数据库，那么这两个容器之间就需要能够通信，因此就需要容器之间可以直接进行网络连接。


那么这就是Docker网络要解决的第一个问题，容器如何访问容器


我们还希望能够看容器内部的实际情况，因此我们需要这个宿主机访问容器


第二个问题是，宿主机如何去访问容器？


第三个问题，容器如何访问宿主机




### Docker的四个网络模式
第一个呢是桥接模式啊，bridge。嗯，这个模式的话，它类似于在你们公司呢去装一个这种内部电话交换机，然后所有在这个模式下的每一个房间呢都接了这个电话，然后每个房间通过这个拨打这个内部电话号（ip）呢来实现这个各个房间之间的进行这个通信，外部是无法进行这个通信的，
这种方式呢，是指就是在docker容器启动时，它会生成一个叫bridge的虚拟网桥，然后给每个容器分配一个IP地址，各个容器之间可以通过这个IP地址进行通信，其他的外部容器无法与内部容器进行通信


第二个是主机模式host，它主要容器不会获得独立的IP，而是直接使用宿主机的网络
优点是性能高，没有网络转换，缺点的话会有可能触发端口冲突，比如说宿主机有80端口，那容器就不能再使用80端口了

第三个呢是无网络模式啊，这种就是不连接任何的网络

第四个是Overlay 覆盖网络。 类比 跨城市的vpn

当你有不同的宿主机，也就是不同的电脑，它们组成一个集群时，通过这个网络模式，让所有的容器假装它们是处在一个局域网里






### Docker网络的最佳实践

我们一般使用的网络模式就是第一个桥接模式，它是Docker默认的网络模式，但是我们不会直接去使用它，而是去自定义一个bridge

默认的桥接模式缺点是容器间需要通过IP通信，但容器IP不固定，每次重启后会随机重新分配，导致之前配置的IP失效，需要重新配置，非常麻烦，所以我们不会使用默认的桥接模式，而是自定义。

自定义桥接模式的优点在于，Docker在你自定义的网络中内置了一个轻量DNS服务器，当容器进行通信的时候，它就可以不去使用IP，而是使用容器的名字进行通鑫。


==因此，最佳实践是永远使用自定义的桥接网络模式==

实现的命令是先自定义一个桥接网络模式，然后让容器使用你自定义的这种网络模式
```
# 1. 创建一个自定义网络
docker network create my_app_net

# 2. 运行容器时加入这个网络
docker run -d --name my_mysql --network my_app_net mysql
docker run -d --name my_web --network my_app_net nginx

# 3. 在 my_web 容器里，可以直接 ping my_mysql，畅通无阻！
```

更加进阶的就是使用Docker Compose进行多容器编排时去使用你自定义的桥接网络模式



### 宿主机如何访问容器

前面我们解决了容器与容器之间相互访问的问题，可以通过调节网络模式，然后使用自定义网络模式，以容器名称作为IP进行相互访问

那么宿主机要如何去访问容器呢，它是通过端口映射来进行


```
docker run -d -p 8080:80 --name my_nginx nginx
```

这里呢是将容器的80端口映射到宿主机的8080端口，这样的话我们在宿主机访问本机的8080端口呢就可以访问到容器端口80端口


### 容器如何访问宿主机？

当你宿主机里运行了MySQL，希望容器内的Python程序能访问宿主机数据库，这时候容器需要访问宿主机，但容器内部的localhost是指容器本身，不是宿主机，所以在容器内部使用localhost无法访问宿主机，为了解决这个问题，Docker给出了一个特殊域名 `host.docker.internal`

只要在容器里访问 **`host.docker.internal`**，它就会自动解析到你真实的宿主机 IP 上。

### Docker网络常用命令

```
docker network ls
docker network inspect mynet
docker network rm mynet
```


## 多容器编排（Docker Compose）

### 为什么需要多容器编排

假如没有docker compose，你要部署一个网站的话，需要部署前端后端数据库，至少是这三样服务，那在处理的时候呢，你就需要使用 Docker 先将这个数据库先启动起来，然后再去启动后端，再去启动前端，你要处理它们之间的网络连接，然后处理它每一个的数据卷，去处理它的这个挂载的端口啊等等，而且呢你每一次重启时呢都需要去重复这个上面的命令，这些命令都是很长的，有可能会出错，就是导致这样的话其实非常的不方便，而且你要处理这个环境问题，呃，那么多任务时间盘呢就是把这些你的需要的这些配置呢全部都写到一个文件里，通过执行这个文件呢就可以实现了快速的去把这个多个服务呢给运行起来



### 什么是Docker Compose

比如说，你开了一家餐厅，然后如果是使用Docker的情况下，那么你需要去招聘厨师、招服务员，然后去购买餐具、购买厨房用具等等，都是你自己去亲自做。
那么使用Docker Compose呢，它能将你所需要的一切东西列出来都交给一个中介，然后中介去帮你把这些事情去完成。


你只需要写一个叫 `docker-compose.yml` 的配置文件，把所有容器的配置写进去，然后敲一行命令，它就能帮你把所有容器按顺序拉起，并让它们互相通信。


### Docker compose的语法

这个文件的骨架是
```
# 1. 核心：定义所有的服务（容器）
services:
  服务A的名字:
    # ... 各种配置
  服务B的名字:
    # ... 各种配置

# 2. 声明数据卷（可选，如果有容器需要存数据就要写）
volumes:
  数据卷名字:

# 3. 声明网络（可选，一般不写 Compose 也会自动创建一个默认网络）
networks:
  网络名字:
```


可以看到主要是分三部分，第一个部分是定义服务，第二部分是定义数据源，第三部分是定义网络。其实跟前面的命令是相对应的，这个数据源也就是前面提到的数据源的这个数据源挂载的链接方式，然后网络的话也是自定义网络

然后呢，在这个定义服务中，会用到下面声明的数据卷和网络


然后定义服务，它跟之前运行容器的run命令是非常相似的


1. 基础配置：从哪来，叫什么

```
services:
  web:
    image: nginx:latest       # 【拉取现成镜像】用哪个基础镜像
    # build: .                # 【本地构建镜像】如果不用 image，可以用 build 基于当前目录的 Dockerfile 打包
    container_name: my-web    # 【自定义容器名】不写的话，会自动叫 "文件夹名_web_1"
```

2. 映射端口

```
    ports:
      - "8080:80"   # 【固定格式：宿主机端口:容器内端口】。意思是对外暴露 8080，转发到容器里的 80
      # - "443:443" # 可以映射多个端口
```

3. 挂载数据卷

```
    volumes:
      - ./html:/usr/share/nginx/html   # 【相对路径映射】把当前电脑的 html 文件夹，映射进容器里（适合开发时改代码）
      - db_data:/var/lib/mysql         # 【命名卷映射】(冒号左边的 db_data 是个名字，不是路径，数据交由 Docker 统一保管)
```

4. 环境变量
```
    environment:
      - MYSQL_ROOT_PASSWORD=123456     # 【列表格式】直接写 键=值
      # 或者用字典格式（二选一，效果一样）：
      # MYSQL_ROOT_PASSWORD: "123456"  
    env_file:
      - .env                           # 【推荐】如果变量太多，可以写在一个 .env 文件里，这里直接引入
```

5. 依赖关系: 谁先启动谁后启动

```
    depends_on:
      - db           # 意思是：启动 web 之前，必须先启动 db 服务
      - redis
```



6. 重启策略：挂了怎么办？
```
    restart: always   # 只要挂了就自动重启（服务器断电重启后也会跟着启动）
    # 可选值：
    # no: 默认值，不重启
    # on-failure: 只有报错退出时才重启
    # unless-stopped: 除非你手动停止它，否则一直重启（最常用！）
```

7. 容器健康检查
```
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"] # 怎么检查健康（这里用 curl 访问自己）
      interval: 10s    # 每隔 10 秒检查一次
      timeout: 3s      # 超过 3 秒算失败
      retries: 3       # 连续失败 3 次算不健康
```

8. 工作目录
```
    working_dir: /app  # 相当于进入容器后先 cd /app
```


一个例子

```
services:
  # 服务1：你的 Python 代码
  web:
    build: .                 # 用当前目录的 Dockerfile 构建镜像
    container_name: flask-app
    ports:
      - "5000:5000"
    volumes:
      - .:/app               # 把当前目录所有代码挂进容器的 /app 里（改了代码不用重新构建镜像）
    working_dir: /app        # 进去后先 cd 到 /app
    command: python app.py   # 覆盖镜像默认的启动命令，执行你的代码
    restart: unless-stopped
    depends_on:
      redis:
        condition: service_healthy  # 🌟 高级用法：等 redis 健康检查通过后，再启动 web

  # 服务2：Redis 缓存
  redis:
    image: redis:alpine      # alpine 版本非常小巧
    container_name: my-redis
    restart: unless-stopped
    healthcheck:             # 给 redis 加个健康检查
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 3s
      retries: 5
```

## Docker与K8s













## docker面试题

