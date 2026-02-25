---
date: '2026-02-25T10:55:10+08:00'
draft: false
slug: '40bd625f'
type: posts
title: 'Nginx入门'
collections: "nginx"
categories: [java]
---

<!--more-->

## 什么是nginx -- what

nginx 是 一个 轻量，高性能 ，http， 反向代理服务器



### 什么是反向代理

代理可以类比中介，你租房子，不是直接找房主，而是找中介。中介帮你去与房主交涉

正向代理：是客户端进行代理，客户端==知道==自己进行了代理，客户端将数据发送给了代理服务器，代理服务器转发给真实服务器。真实服务器==不知道==有代理服务器，以为代理服务器就是客户端。 这就是正向代理。 类比海外代购

反向代理： 是服务端的代理，客户端==不知道==有代理服务器，以为自己访问的是真实服务器，实际上访问的是代理服务器。



正向代理的好处：帮你做你不能做的事

反向代理的好处： 隐藏真实地址，负载均衡。



## 为什么需要nginx -- why

既然django，tomcat都能提供web服务，为什么还需要nginx？

<br/>

虽然这些服务器可以提供web服务，但是他们更擅长处理后端逻辑。而nginx更擅长处理网络流量。专业的事交给专业的人。

<br/>

nginx擅长处理网络流量的体现：

1. nginx高并发能力强，处理特别快
   1. nginx事件驱动机制，底层采用多路复用技术，一个worker可以在一个线程中处理数万个连接
   2. master-worker进程模型：
      1. master进程： 主进程，主要处理配置文件，管理worker
      2. worker进程：可以有多个，处理网络请求

2. 支持反代
3. 支持负载均衡
4. 处理静态文件极快



## nginx命令

环境 ： debian 12

1. 安装： `apt install nginx -y`
2. 启动： `systemctl start nginx`
3. 打开防火墙： `ufw allow 'nginx full'`
   1. `ufw reload`
4. 重启nginx： `systemctl reload nginx`
5. 检查语法： `nginx -t`



## nginx 要如何使用呢 -- how

nginx的 功能：

1. 反向代理
2. 负载均衡
3. 处理静态资源很快



### nginx 的配置文件

nginx的配置文件有两种写法：

- 所有的配置写到一个文件中
- 分两个文件写： 一个是主配置文件，一个模块配置文件

==其实对后端来讲：也就部署网站会用，配置文件看得懂就行，不会让AI改就完了，所以这一部分可以先过，直接看实战==
<br/>

#### 配置文件的层级：

```txt
全局块
event块
http块
	-http全局配置
	-upstream块
	-server块
		-server全局配置
		-location块
			location配置指令
mail块
```

如果是将所有配置写到一个文件的写法，就是将这些块都写到一个文件中
如果是 模块化的配置方法：
- 全局，event，http块里的配置等 写到一个文件中，同时在这个文件中导入模块配置文件
	- `include /etc/nginx/conf.d/*.conf;`
- 在模块配置文件中， 写 upstream块，server块等


#### 全局块：
 ![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225140541294.png)
<br/>

#### event块：
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225140548689.png)
<br/>

#### http块：
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225140604955.png)

![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225141454741.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225141519313.png)

#### server块和location块：
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225142351565.png)

![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225142401934.png)
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225142526384.png)


### 实战

#### http反向代理配置实战
环境： debian 12
确保你已经打开了nginx： `systemctl status nginx` 

1. 开启一个后端服务
```shell
1. 创建文件夹
mkdir -p ~/nginx-test
cd ~/nginx-test

2. 编写一个页面
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>后端页面 - 被 Nginx 代理</title>
</head>
<body style="background:#112233;color:#00ff88;font-family:monospace;text-align:center;padding-top:80px;">
    <h1>🎉 成功！这是后端返回的内容（端口 8080）</h1>
    <p>你现在看到的内容经过 Nginx 反向代理转发。</p>
    <p>代理功能正常工作！</p>
</body>
</html>
EOF

3. 启动后端服务（后台运行）
nohup python3 -m http.server 8080 > backend.log 2>&1 &
echo "后端已启动在 8080 端口！"


```

 2. 在/etc/nginx/sites-enabled/default 这个模块配置文件中写
```
server{
	listen 80;  #监听端口
	server_name _;  #域名匹配
	
	location / {
		proxy_pass http://localhost:8080;  #将80端口的请求转发到后端8080
		charset utf-8;   # 防止乱码
	}
	
}
```
保存
3. 检查语法并重启nginx
```shell

nginx -t  # 检查语法

systemctl reload nginx # 重启nginx
```

打开服务器的ip地址，即可看到
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225163709166.png)


在location 块中除了 proxy_pass 外还可以配：
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225164322989.png)






#### 负载均衡配置实战

1. 准备两个后端服务

```shell
# 1 创建文件夹
mkdir -p ~/backend-1 ~/backend-2

# 2 创建两个页面
cat > ~/backend1/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>Backend-1</title></head>
<body style="background:#112233;color:#00ff88;text-align:center;padding-top:100px;">
    <h1>我是 Backend-1 (端口 8080)</h1>
    <p>这是第一台后端服务器</p>
</body>
</html>
EOF

cat > ~/backend2/index.html << 'EOF'
<!DOCTYPE html>
<html>
<head><meta charset="UTF-8"><title>Backend-2</title></head>
<body style="background:#331122;color:#ff8800;text-align:center;padding-top:100px;">
    <h1>我是 Backend-2 (端口 8081)</h1>
    <p>这是第二台后端服务器</p>
</body>
</html>
EOF

# 3. 启动两个后端（后台运行） 
cd ~/backend1 && nohup python3 -m http.server 8080 > backend1.log 2>&1 & cd ~/backend2 && nohup python3 -m http.server 8081 > backend2.log 2>&1 &
echo "两个后端已启动！" 


```

2. 配置nginx

在/etc/nginx/sites-available/default 文件中修改

```shell

#定义upstream
upstream backend_pool{
	server localhost:8081; 
	server localhost:8082;
}

server{
	listen 80;
	server_name _;
	
	location / {
		proxy_pass http://backend_pool/;
		proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        charset utf-8;
	}
	
	
}



```
3. 重启nginx

访问页面 并 ctrl + f5
可以看到：
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225165757067.png)

与
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225165808450.png)
交替出现



>负载均衡策略
1. 轮询（默认） 轮流分配
2. 给连接最少的后端服务
3. 根据用户ip算哈希
4. 按权重分配
5. 完全随机





#### 静态页面配置

1. 创建静态页面
```shell

# 1. 进入默认静态目录
cd /var/www/html

# 2. 创建一个漂亮的静态首页
cat > index.html << 'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>我的第一个 Nginx 静态网站</title>
    <style>
        body { font-family: Arial; text-align: center; padding: 100px; background: linear-gradient(135deg, #667eea, #764ba2); color: white; }
        h1 { font-size: 3em; }
        .card { background: rgba(255,255,255,0.1); padding: 30px; border-radius: 15px; display: inline-block; margin-top: 30px; }
    </style>
</head>
<body>
    <div class="card">
        <h1>🎉 恭喜！你用 Nginx 成功运行了静态网站！</h1>
        <p>这是纯静态页面，由 Nginx 直接提供。</p>
        <p>当前时间：$(date)</p>
    </div>
</body>
</html>
EOF
```

2. 修改配置文件

 修改 /etc/nginx/sites-available/default 文件
```shell
upstream backend_pool {
    least_conn;
    server 127.0.0.1:8080;
    server 127.0.0.1:8081;
}

server {
    listen 80;

    server_name _;

    # 静态页面根目录配置（重点！）
    root /var/www/html;      # 静态文件根目录
    index index.html;        # 默认访问的文件名

    # 静态页面（根路径）
    location / {
        try_files $uri $uri/ =404;   # 优先找文件，找不到返回404
    }

}
```

3. 重启

可以看到
![image.png](https://cdn.jsdelivr.net/gh/word1230/image-ob@main/image/20260225170744193.png)


