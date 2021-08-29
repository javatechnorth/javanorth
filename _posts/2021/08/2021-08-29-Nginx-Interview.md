---
layout: post
title:  Nginx面试题
tagline: by 揽月中人
categories: Nginx面试题
tags:
- 揽月中人
---

Nginx面试题

<!--more-->

### 1 什么是Nginx？

Nginx---Ngine X，是一款免费的、自由的、开源的、高性能HTTP服务器和反向代理服务器；也是一个IMAP、POP3、SMTP代理服务器；Nginx以其高性能、稳定性、丰富的功能、简单的配置和低资源消耗而闻名。

也就是说Nginx本身就可以托管网站（类似于Tomcat一样），进行Http服务处理，也可以作为反向代理服务器 、负载均衡器和HTTP缓存。

Nginx 解决了服务器的C10K（就是在一秒之内连接客户端的数目为10k即1万）问题。它的设计不像传统的服务器那样使用线程处理请求，而是一个更加高级的机制—事件驱动机制，是一种异步事件驱动结构。

### 2 为什么要用Nginx？
1. 跨平台：可以在大多数Unix like 系统编译运行。而且也有Windows的移植版本。 
2. 配置异常简单：非常的简单，易上手。 
3. 非阻塞、高并发连接：数据复制时，磁盘I/O的第一阶段是非阻塞的。官方测试能支持5万并发连接，实际生产中能跑2~3万并发连接数（得益于Nginx采用了最新的epoll事件处理模型（消息队列）。 
4. Nginx代理和后端Web服务器间无需长连接； 
5. Nginx接收用户请求是异步的，即先将用户请求全部接收下来，再一次性发送到后端Web服务器，极大减轻后端Web服务器的压力。 
6. 发送响应报文时，是边接收来自后端Web服务器的数据，边发送给客户端。 
7. 网络依赖性低，理论上只要能够ping通就可以实施负载均衡，而且可以有效区分内网、外网流量。 
8. 支持内置服务器检测。Nginx能够根据应用服务器处理页面返回的状态码、超时信息等检测服务器是否出现故障，并及时返回错误的请求重新提交到其它节点上。 
9. 此外还有内存消耗小、成本低廉（比F5硬件负载均衡器廉价太多）、节省带宽、稳定性高等特点。

### 3 为什么Nginx性能这么高？

Nginx事件处理机制：异步非阻塞事件处理机制：运用了epoll模型，提供了一个队列，排队解决。

### 4 Nginx怎么处理请求的？

nginx接收一个请求后，首先由listen和server_name指令匹配server模块，再匹配server模块里的location，location就是实际地址。

```yaml
   server {            						# 第一个Server区块开始，表示一个独立的虚拟主机站点
        listen       80；      					# 提供服务的端口，默认80
        server_name  localhost；       			# 提供服务的域名主机名
        location / {            				# 第一个location区块开始
            root   html；       				# 站点的根目录，相当于Nginx的安装目录
            index  index.html index.htm；      	# 默认的首页文件，多个用空格分开
        }          								# 第一个location区块结果

```

### 5 什么是正向代理和反向代理？

**正向代理（forward proxy）**：是一个位于客户端和目标服务器之间的服务器(代理服务器)，为了从目标服务器取得内容，客户端向代理服务器发送一个请求并指定目标，然后代理服务器向目标服务器转交请求并将获得的内容返回给客户端。

**反向代理（reverse proxy）**：是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。

#### 区别：

虽然正向代理服务器和反向代理服务器所处的位置都是客户端和真实服务器之间，所做的事情也都是把客户端的请求转发给服务器，再把服务器的响应转发给客户端，但是二者之间还是有一定的差异的。

1、**正向代理其实是客户端的代理**，帮助客户端访问其无法访问的服务器资源。**反向代理则是服务器的代理**，帮助服务器做负载均衡，安全防护等。

2、**正向代理一般是客户端架设的**，比如在自己的机器上安装一个代理软件。**而反向代理一般是服务器架设的**，比如在自己的机器集群中部署一个反向代理服务器。

3、**正向代理中，服务器不知道真正的客户端到底是谁**，以为访问自己的就是真实的客户端。而在**反向代理中，客户端不知道真正的服务器是谁**，以为自己访问的就是真实的服务器。

4、正向代理和反向代理的作用和目的不同。**正向代理主要是用来解决访问限制问题**。而**反向代理则是提供负载均衡、安全防护等作用**。二者均能提高访问速度。



### 6 使用“反向代理服务器的优点是什么?

反向代理服务器可以隐藏源服务器的存在和特征。它充当互联网云和web服务器之间的中间层。这对于安全方面来说是很好的，特别是当您使用web托管服务时。

### 7 Nginx的优缺点？

#### 优点：

1. 占内存小，可实现高并发连接，处理响应快
2. 可实现http服务器、虚拟主机、方向代理、负载均衡
3. Nginx配置简单
4. 可以不暴露正式的服务器IP地址

#### 缺点：

动态处理差：Nginx处理静态文件好,耗费内存少，但是处理动态页面则很鸡肋，现在一般前端用Nginx作为反向代理抗住压力.

### 8 Nginx应用场景？

1. HTTP服务器。Nginx本身也是一个静态资源的服务器，当只有静态资源的时候，就可以使用Nginx来做服务器，如果一个网站只是静态页面的话，那么就可以通过这种方式来实现部署。
2. 静态服务器。在公司中经常会遇到静态服务器，通常会提供一个上传的功能，其他应用如果需要静态资源就从该静态服务器中获取。
3. 动静分离。动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们就可以根据静态资源的特点将其做缓存操作，这就是网站静态化处理的核心思路。
4. 反向代理。反向代理应该是Nginx使用最多的功能了，反向代理(Reverse Proxy)方式是指以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器。
5. 负载均衡。负载均衡也是Nginx常用的一个功能，负载均衡其意思就是分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其它关键任务服务器等，从而共同完成工作任务。
6. Nginx 中也可以配置安全管理、比如可以使用Nginx搭建API接口网关,对每个接口服务进行拦截。
   

### 9 Nginx目录结构有哪些？

```yaml
[root@localhost ~]# tree /usr/local/nginx
/usr/local/nginx
├── client_body_temp
├── conf                             # Nginx所有配置文件的目录
│   ├── fastcgi.conf                 # fastcgi相关参数的配置文件
│   ├── fastcgi.conf.default         # fastcgi.conf的原始备份文件
│   ├── fastcgi_params               # fastcgi的参数文件
│   ├── fastcgi_params.default       
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types                   # 媒体类型
│   ├── mime.types.default
│   ├── nginx.conf                   # Nginx主配置文件
│   ├── nginx.conf.default
│   ├── scgi_params                  # scgi相关参数文件
│   ├── scgi_params.default  
│   ├── uwsgi_params                 # uwsgi相关参数文件
│   ├── uwsgi_params.default
│   └── win-utf
├── fastcgi_temp                     # fastcgi临时数据目录
├── html                             # Nginx默认站点目录
│   ├── 50x.html                     # 错误页面优雅替代显示文件，例如当出现502错误时会调用此页面
│   └── index.html                   # 默认的首页文件
├── logs                             # Nginx日志目录
│   ├── access.log                   # 访问日志文件
│   ├── error.log                    # 错误日志文件
│   └── nginx.pid                    # pid文件，Nginx进程启动后，会把所有进程的ID号写到此文件
├── proxy_temp                       # 临时目录
├── sbin                             # Nginx命令目录
│   └── nginx                        # Nginx的启动命令
├── scgi_temp                        # 临时目录
└── uwsgi_temp                       # 临时目录
```

### 10 Nginx配置文件nginx.conf有哪些属性模块?

```yaml
worker_processes  1；                		   # worker进程的数量
events {                              			# 事件区块开始
    worker_connections  1024；            	   	# 每个worker进程支持的最大连接数
}                                    			# 事件区块结束
http {                               			# HTTP区块开始
    include       mime.types；            		# Nginx支持的媒体类型库文件
    default_type  application/octet-stream；     # 默认的媒体类型
    sendfile        on；       					# 开启高效传输模式
    keepalive_timeout  65；       				# 连接超时
    server {            						# 第一个Server区块开始，表示一个独立的虚拟主机站点
        listen       80；      					# 提供服务的端口，默认80
        server_name  localhost；       			# 提供服务的域名主机名
        location / {            				# 第一个location区块开始
            root   html；       					# 站点的根目录，相当于Nginx的安装目录
            index  index.html index.htm；      	# 默认的首页文件，多个用空格分开
        }          								# 第一个location区块结果
        error_page   500502503504  /50x.html；   # 出现对应的http状态码时，使用50x.html回应客户
        location = /50x.html {          		# location区块开始，访问50x.html
            root   html；      					# 指定对应的站点目录为html
        }
    }  
    ......


```

### 11 Nginx静态资源?

静态资源访问，就是存放在Nginx的html页面，我们可以自己编写

### 12 如何用Nginx解决前端跨域问题？

使用Nginx转发请求。把跨域的接口写成调本域的接口，然后将这些接口转发到真正的请求地址。

### 13 Nginx虚拟主机怎么配置?

1. 基于域名的虚拟主机，通过域名来区分虚拟主机——应用：外部网站
2. 基于端口的虚拟主机，通过端口来区分虚拟主机——应用：公司内部网站，外部网站的管理后台
3. 基于ip的虚拟主机。

### 14 限流怎么做的？

Nginx限流就是限制用户请求速度，防止服务器受不了

限流有3种

1.  正常限制访问频率（正常流量）
2.  突发限制访问频率（突发流量）
3.  限制并发连接数

Nginx的限流都是基于漏桶流算法，底下会说道什么是桶铜流

### 15 为什么要做动静分离？

1. Nginx是当下最热的Web容器，网站优化的重要点在于静态化网站，网站静态化的关键点则是是动静分离，动静分离是让动态网站里的动态网页根据一定规则把不变的资源和经常变的资源区分开来，动静资源做好了拆分以后，我们则根据静态资源的特点将其做缓存操作。

2. 让静态的资源只走静态资源服务器，动态的走动态的服务器

3. Nginx的静态处理能力很强，但是动态处理能力不足，因此，在企业中常用动静分离技术。

4. 对于静态资源比如图片，js，css等文件，我们则在反向代理服务器nginx中进行缓存。这样浏览器在请求一个静态资源时，代理服务器nginx就可以直接处理，无需将请求转发给后端服务器tomcat。
   若用户请求的动态文件，比如servlet,jsp则转发给Tomcat服务器处理，从而实现动静分离。这也是反向代理服务器的一个重要的作用。

### 16 Nginx负载均衡的算法怎么实现的?策略有哪些?

####  1. RR(round robin :轮询 默认)

每个请求按时间顺序逐一分配到不同的后端服务器，也就是说第一次请求分配到第一台服务器上，第二次请求分配到第二台服务器上，如果只有两台服务器，第三次请求继续分配到第一台上，这样循环轮询下去，也就是服务器接收请求的比例是 1:1， 如果后端服务器down掉，能自动剔除。轮询是默认配置，不需要太多的配置

同一个项目分别使用8081和8082端口启动项目

```yaml
upstream web_servers { 
  server localhost:8081; 
  server localhost:8082; 
}

server {
  listen    80;
  server_name localhost;
  \#access_log logs/host.access.log main;



  location / {
    proxy_pass http://web_servers;
    \# 必须指定Header Host
    proxy_set_header Host $host:$server_port;
  }
 }
```

#### 2. 权重

指定轮询几率，weight和访问比率成正比, 也就是服务器接收请求的比例就是各自配置的weight的比例，用于后端服务器性能不均的情况,比如服务器性能差点就少接收点请求，服务器性能好点就多处理点请求。

```yaml
upstream test {
  server localhost:8081 weight=1;
  server localhost:8082 weight=3;
  server localhost:8083 weight=4 backup;
}
```

示例是4次请求只有一次被分配到8081上，其他3次分配到8082上。backup是指热备，只有当8081和8082都宕机的情况下才走8083

#### 3. ip_hash

上面的2种方式都有一个问题，那就是下一个请求来的时候请求可能分发到另外一个服务器，当我们的程序不是无状态的时候(采用了session保存数据)，这时候就有一个很大的很问题了，比如把登录信息保存到了session中，那么跳转到另外一台服务器的时候就需要重新登录了，所以很多时候我们需要一个客户只访问一个服务器，那么就需要用iphash了，iphash的每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

```yaml
upstream test {
  ip_hash;
  server localhost:8080;
  server localhost:8081;
}
```

#### 4. fair(第三方)

按后端服务器的响应时间来分配请求，响应时间短的优先分配。这个配置是为了更快的给用户响应

```yaml
upstream backend {
  fair;
  server localhost:8080;
  server localhost:8081;
}
```



#### 5. url_hash(第三方)

按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。在upstream中加入hash语句，server语句中不能写入weight等其他的参数，`hash_method`是使用的hash算法

```yaml
upstream backend {
    hash $request_uri;
    hash_method crc32;
    server localhost:8080;
    server localhost:8081;
}
```

### 17 Nginx location uri正则表达式

location uri正则表达式

- `.` ：匹配除换行符以外的任意字符
- `?` ：重复0次或1次
- `+` ：重复1次或更多次
- `*` ：重复0次或更多次
- `\d` ：匹配数字
- `^` ：匹配字符串的开始
- `$` ：匹配字符串的结束
- `{n}` ：重复n次
- `{n,}` ：重复n次或更多次
- `[c]` ：匹配单个字符c
- `[a-z]` ：匹配a-z小写字母的任意一个
- `(a|b|c)` : 属线表示匹配任意一种情况，每种情况使用竖线分隔，一般使用小括号括括住，匹配符合a字符 或是b字符 或是c字符的字符串
- `\` 反斜杠：用于转义特殊字符

小括号()之间匹配的内容，可以在后面通过`$1`来引用，`$2`表示的是前面第二个()里的内容。正则里面容易让人困惑的是`\`转义特殊字符。

### 18 Nginx配置高可用性怎么配置？

当上游服务器(真实访问服务器)，一旦出现故障或者是没有及时相应的话，应该直接轮训到下一台服务器，保证服务器的高可用

```yaml
server {
        listen       80;
        server_name  www.javanorth.cn;
        location / {
		    ### 指定上游服务器负载均衡服务器
		    proxy_pass http://backServer;
			###nginx与上游服务器(真实访问的服务器)超时时间 后端服务器连接的超时时间_发起握手等候响应超时时间
			proxy_connect_timeout 1s;
			###nginx发送给上游服务器(真实访问的服务器)超时时间
            proxy_send_timeout 1s;
			### nginx接受上游服务器(真实访问的服务器)超时时间
            proxy_read_timeout 1s;
            index  index.html index.htm;
        }
    }
```

### 19 Nginx怎么判断别IP不可访问？

```yaml
 # 如果访问的ip地址为192.168.3.123,则返回403
     if  ($remote_addr = 192.168.3.123) {  
         return 403;  
     }  
```

### 20 怎么限制浏览器访问？

```yaml
## 不允许谷歌浏览器访问 如果是谷歌浏览器返回500
 	if ($http_user_agent ~ Chrome) {   
        return 500;  
    }
```

### 21 Rewrite全局变量是什么？

![image-20210829211717118](http://www.javanorth.cn/assets/images/2021/Nginx/Nginx Rewrite.png)