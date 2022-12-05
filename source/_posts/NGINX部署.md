---
title : Nginx部署只是
categories : 运维
tags : nginx
cover: https://source.wjwsm.top/97.jpg
swiper_index: 5
date: 2022-12-03 10:20:00
---
# *NGINX*部署

## *NGINX*介绍

`作用：http代理，反向代理，负载均衡，动静分离，作为web服务器最常用的功能之一，尤其反向代理，也可以解决浏览器同源问题。`

## 正反代理

### 正向代理

![正向代理](https://source.wjwsm.top/%E6%AD%A3%E5%90%91%E4%BB%A3%E7%90%86.png)

### 反向代理

![](https://source.wjwsm.top/%E5%8F%8D%E5%90%91%E4%BB%A3%E7%90%86.png)

> 正向代理，代理的是客户端方向的，反向代理代理的是服务端方法的。

## 负载均衡

> 负载均衡分类 硬件和软件负载均衡
>
> 1. 硬件负载均衡（贵，性能高，并发量大）
> 2. 软件负载均衡（便宜，性能低）
> 3. NGINX    1w-5w

> Nginx提供的负载均衡策略有2种：内置策略和扩展策略。内置策略为轮询，加权轮询，iphash，扩展有响应时间最小，url哈希

### 轮询

![轮询](https://source.wjwsm.top/%E8%BD%AE%E8%AF%A2.png)

~~~
nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	server localhost:8081;
	server localhost:8082;
	server localhost:8083;
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表

}
~~~



### 加权轮询

![](https://source.wjwsm.top/%E5%8A%A0%E6%9D%83%E8%BD%AE%E8%AF%A2.png)

~~~
nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	server localhost:8081 weight=1;
	server localhost:8082 weight=2;
	server localhost:8083 weight=3;
	# 1/6  1/3 1/2的权重概率
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表
}
~~~



### iphash（客户端）

![](https://source.wjwsm.top/iphash.png)

~~~
nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	ip_hash; #保证每个访客访问固定服务器
	server localhost:8081 weight=1;
	server localhost:8082 weight=2;
	server localhost:8083 weight=3;
	# 1/6  1/3 1/2的权重概率
	#ip_hash可以配权重 
	# ip_hash性能低，一般不推荐
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表
}
~~~



`iphash对客户端请求的ip进行hash操作，然后根据hash结果将同一个客户端ip的请求分发给同一台服务器进行处理，可以解决session不共享的问题。`

### 最小连接数

将请求发给连接数最少的服务器

~~~
nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	least_conn;
	server localhost:8081;
	server localhost:8082;
	server localhost:8083;
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表

}
~~~

### 响应时间最少（第三方）

~~~
 nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	fair; #实现响应时间短的优先分配
	server localhost:8081;
	server localhost:8082;
	server localhost:8083;
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表

}
~~~

### url哈希（服务端）

> url哈希之后指向对应的固定服务器，统一资源多次请求，可能会到不同的服务器，导致不必要的多次下载，缓存命中率不高，以及一些资源时间的浪费。

~~~
 nginx.conf文件中
#负载均衡：目标服务器列表
upstream host_list{
	hash $request_url; #实现url_hash
	server localhost:8081;
	server localhost:8082;
	server localhost:8083;
}

location /nginx{
	proxy_pass:http://host_list;  
	#代理转发给服务器列表

}
~~~



## 动静分离

+ 跨域解决方案

jsonp、后端程序（允许跨域）、前端代理服务器、nginx服务器

>动静分离，在我们的软件开发中，有些请求是需要后台处理的，有些请求是不需要经过后台处理的（如：css、html、jpg、js等等文件），这些不需要经过后台处理的文件称为静态文件。让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作。提高资源响应的速度。

![动静分离](https://source.wjwsm.top/%E5%8A%A8%E9%9D%99%E5%88%86%E7%A6%BB.png)



## *LINUX*环境下安装nginx

**1、安装gcc**

安装 nginx 需要先将官网下载的源码进行编译，编译依赖 gcc 环境，如果没有 gcc 环境，则需要安装：

```
yum install gcc-c++
```

**2、PCRE pcre-devel 安装**

PCRE(Perl Compatible Regular Expressions) 是一个Perl库，包括 perl 兼容的正则表达式库。nginx 的 http 模块使用 pcre 来解析正则表达式，所以需要在 linux 上安装 pcre 库，pcre-devel 是使用 pcre 开发的一个二次开发库。nginx也需要此库。命令：

```
yum install -y pcre pcre-devel
```

**3、zlib 安装**

zlib 库提供了很多种压缩和解压缩的方式， nginx 使用 zlib 对 http 包的内容进行 gzip ，所以需要在 Centos 上安装 zlib 库。

```
yum install -y zlib zlib-devel
```

**4、OpenSSL 安装**
OpenSSL 是一个强大的安全套接字层密码库，囊括主要的密码算法、常用的密钥和证书封装管理功能及 SSL 协议，并提供丰富的应用程序供测试或其它目的使用。
nginx 不仅支持 http 协议，还支持 https（即在ssl协议上传输http），所以需要在 Centos 安装 OpenSSL 库。

```
yum install -y openssl openssl-devel
```

**5、下载安装包**

手动下载.tar.gz安装包，地址：https://nginx.org/en/download.html

下载完毕上传到服务器上 /root

**6、解压**

```
tar -zxvf nginx-1.18.0.tar.gzcd nginx-1.18.0
```

**7、配置**

使用默认配置，在nginx根目录下执行

```
./configure
make
make install
```

查找安装路径： `whereis nginx`

8、打开阿里云安全组80端口，访问远程主机

![](https://source.wjwsm.top/nginx%E5%AE%89%E8%A3%85%E6%88%90%E5%8A%9Fpng.png)

## *Nginx*常用命令

~~~
cd /usr/local/nginx/sbin/
./nginx  启动
./nginx -s stop  停止
./nginx -s quit  安全退出
./nginx -s reload  重新加载配置文件
ps aux|grep nginx  查看nginx进程
~~~

## *Nginx*配置文件详解

1. worker_processes 1;配置worker进程的数量，建议，最小是1；最大等于cpu的线程数

2. events进程 

   ```
   events {
       worker_connections  1024;
   }
   ```

   每一个events默认并发处理1024个请求

   最大连接数= worker_processes * worker_connections/4

3. http请求模块

   include mime.types  请求内容类型

   default_type  application/octet-stream; 缺省内容类型

   sendfile        on;开启高性能数据处理

   keepalive_timeout  65; 连接保持时间

4. server模块（一个serve代表一个服务器）

   一个http模块中，可以配置多个nginx服务器

   listen 80;通信端口号

   server_name  localhost; 主机ip

   error_page 404 :404页面

   error_page   500 502 503 504  /50x.html;50x错误页面

    location  一个serverr模块包含多个location

       location / {
           root   html;
           index  index.html index.htm;
       }

​		/ 表示 根目录下的所有请求http://location:80......

​		root html、 ；//web资源根目录

​		index index.html index.htm;默认请求的首页

### location详解

1. location

   转发请求

   当nginx接收到客户端请求，获取请求的url和location的表达式进行匹配，进行请求转发。

2. 语法

   location  url表达式{

   ​        server  请求转发目标；

   }

   通过指定模式来与客户端请求的URI相匹配，基本语法如下：**location [=|~|~\*|^~|@] pattern{……}**

   + 没有修饰符 表示：必须以指定模式开始，如：

     ~~~
     server {
     　　server_name baidu.com;
     　　location /abc {
     　　　　……
     　　}
     }
     
     那么，如下是对的：
     http://baidu.com/abc
     http://baidu.com/abc?p1
     http://baidu.com/abc/
     http://baidu.com/abcde
     ~~~

   + =表示：必须与指定的模式精确匹配

     ~~~
     server {
     	server_name sish
     　　location = /abc {
     　　　　……
     　　}
     }
     那么，如下是对的：
     http://baidu.com/abc
     http://baidu.com/abc?p1
     如下是错的：
     http://baidu.com/abc/
     http://baidu.com/abcde
     ~~~

   + ~ 表示：指定的正则表达式要区分大小写

     ~~~
     server {
     	server_name baidu.com;
     　　location ~ ^/abc$ {
     　　　　……
     　　}
     }
     那么，如下是对的：
     http://baidu.com/abc
     http://baidu.com/abc?p1=11&p2=22
     如下是错的：
     http://baidu.com/ABC
     http://baidu.com/abc/
     http://baidu.com/abcde
     ~~~

   + ~* 表示：指定的正则表达式不区分大小写

     ~~~
     server {
     server_name baidu.com;
     location ~* ^/abc$ {
     　　　　……
     　　}
     }
     那么，如下是对的：
     http://baidu.com/abc
     http://baidu..com/ABC
     http://baidu..com/abc?p1=11&p2=22
     如下是错的：
     http://baidu..com/abc/
     http://baidu..com/abcde
     ~~~

   + ^~ 类似于无修饰符的行为，也是以指定模式开始，不同的是，如果模式匹配，
     那么就停止搜索其他模式了。
   + @ ：定义命名location区段，这些区段客户段不能访问，只可以由内部产生的请
     求来访问，如try_files或error_page等

### location查找优先级

查找顺序和优先级
1：带有“=“的精确匹配优先
2：没有修饰符的精确匹配
3：正则表达式按照他们在配置文件中定义的顺序
4：带有“^~”修饰符的，开头匹配
5：带有“~” 或“~\*” 修饰符的，如果正则表达式与URI匹配
6：没有修饰符的，如果指定字符串与URI开头匹配

### location和proxy_pass代理规则（是否以“/”结尾）

1.  配置 **proxy_pass** 时，当在后面的 **url** 加上了 **/**，相当于是绝对路径，则 **Nginx** 不会把 **location** 中匹配的路径部分加入代理 **uri**。

   + 比如下面配置，我们访问 **http://IP/proxy/test.html**，最终代理到 **URL** 是 **http://127.0.0.1/test.html**

   ~~~
   location /proxy/ {
   	proxy_pass http://127.0.0.1/;
   }
   ~~~

2. 如果配置 **proxy_pass** 时，后面没有 **/**，**Nginx** 则会把匹配的路径部分加入代理 **uri**。

   + 比如下面配置，我们访问 **http://IP/proxy/test.html**，最终代理到 **URL** 是 **http://127.0.0.1/proxy/test.html**

   ~~~
   location /proxy/ {
   	proxy_pass http://127.0.0.1;
   }
   ~~~

   

  

