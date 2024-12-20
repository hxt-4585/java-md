# Nginx基础篇

## 一、目录结构与基本运行原理

### 1.1 目录结构

 进入Nginx的主目录我们可以看到这些文件夹  

 **client_body_temp  conf  fastcgi_temp  html logs proxy_temp sbin scgi_temp uwsgi_temp**  



 其中这几个文件夹在刚安装后是没有的，主要用来存放运行过程中的临时文件  

 **client_body_temp fastcgi_temp proxy_temp scgi_temp**  



-  **conf**  

 用来存放配置文件相关  

-  **html**  

 用来存放静态文件的默认目录 html、css等  

-  **sbin**  

 nginx的主程序  



![image-20241124153245849](.\img\image-20241124153245849.png)



### 1.2 基本运行原理

![](.\img\image.png)

## 二、Nginx基础配置

### 2.1 最小配置文件

```
# nginx.conf，删除了注释部分

worker_processes  1;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;


    sendfile        on;


    keepalive_timeout  65;

    server {
        listen       80;
        server_name  localhost;


        location / {
            root   html;
            index  index.html index.htm;
        }



        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


    }

}
```



`work_processes 1` 默认为1，表示开启一个业务进程，根据内核数进行配置

`worker_connections` 每一个worker进程可以创建多少连接

`include mime.types` 把另外一个配置文件引入当前的配置文件，`mime.types` 标注发送的是什么类型

`sendfile on` 数据零拷贝，免除中间拷贝的过程

​		未开启 `sendfile` ：`nginx` 是一个软件，会向操作系统注册某个端口，当有请求发送过来的时候，`linux` 网络接口将请求转发给 `nginx` ，`nginx` 会从固态硬盘中读取请求的内容，并加载到**应用程序的内存**，应用程序读取后发送给**网络接口缓存**

![](.\img\image (1).png)

​		开启 `sendfile` ：**应用内存**不去加载磁盘上的文件，通过发送 `sendfile()` 直接让**网络接口**读取内容，减少了一次数据拷贝的过程

![](.\img\image (2).png)

`keepalive_timeout 65` 保持连接，超时时间

`server` ：可以配置多个主机，通过修改端口号

```
server {
    listen 80; #监听端口号
    server_name localhost; #域名、主机名
    
    # location指uri，/xxoo/index
    # http://atguigu.com/xxoo/index
    location / { #匹配路径
        root html; #文件根目录，html目录
        index index.html index.htm; #默认页名称
    }
    error_page 500 502 503 504 /50x.html; #报错编码对应页面
    location = /50x.html {
    		root html;
		}
}
```



## 三、虚拟主机与域名解析

### 3.1 浏览器、Nginx与Http协议

![image-20241124163652601](.\img\image-20241124163652601.png)



### 3.2 虚拟主机原理

在一台物理服务器上划分出多个**虚拟的独立服务器**，每个虚拟主机可以独立运行，拥有自己的文件目录、操作系统和配置文件

```
# 在 hosts 文件中添加
192.168.116.128 s.com
```



### 3.3 多站点配置

创建www和vod目录，并分别创建两个index.html

![image-20241125223657886](.\img\image-20241125223657886.png)



```
    # 上面的server优先匹配
    server {
        listen       80;
        # server_name  localhost;
        server_name  www.mmban.com;

        location / {
            # root   html;
            root   /www/www;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


    }

    server {
        listen       88;
        #server_name  localhost;
        server_name  vod.mmban.com;

        location / {
            # root   html;
            root   /www/vod;
            index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```



### 3.4 ServerName匹配规则

```
# 配置两个域名指向同一个主机
server_name  vod.mmban.com vod1.mmban.com;
# 通配符
server_name *.mmban.com;
server_name vod.*;
# 正则匹配
server_name ~^[0-9]+\.mmban\.com$;
```



### 3.5 多用户二级域名

![image-20241127204124354](.\img\image-20241127204124354.png)

`ym.weibo.com` 其中 `ym` 为二级域名，泛解析：`*.weibo.com` 所有符合规则的域名都解析到这一台服务器上，`nginx` 服务器将请求转发到业务服务器，业务服务器通过查询 DB 找到相关信息，通过反向代理返回给 `nginx`

### 3.6 短网址

**短网址的作用**：

1、减少字节数：有些URL是很长的但URL本身往往不是用户关注的，在微博和短信等场景中如果URL占了长长的篇幅这是很影响阅读体验的；使用短网址有效控制了URL的长度又能保证用过可通过URL到达设定页面。

![image-20241127204714942](.\img\image-20241127204714942.png)

2、隐藏真实网站：长网址保存了真实域名，比如一条短信说点链接可领某宝红包，如果使用长网址域名根本就不是淘宝的用户可能就不会点。使用短连接就能将真实域名隐藏起来，单从短网址上看用户不能知道是不是指向淘宝页面。如下图所示：

![image-20241127204730430](.\img\image-20241127204730430.png)

![image-20241127205150971](.\img\image-20241127205150971.png)



### 3.7 HTTPDNS

DNS走UDP协议，HTTPDNS走HTTP协议，一般给手机的APP或者CS架构



**传统 DNS 存在的问题**

当我们发出请求解析 DNS 的时候，首先，会先连接到运营商本地的 DNS 服务器，由这个服务器帮我们去整棵 DNS 树上进行解析，然后将解析的结果返回给客户端。但是本地的 DNS 服务器，作为一个本地导游，往往有自己的“小心思”。

一个最令人头痛的问题，相信每个人都遇到，就是**域名劫持**，指通过攻击域名解析服务器（DNS）或伪造域名解析服务器的方法，将目标网站的域名解析到错误的地址，从而使用户无法访问目标网站。

**HTTPDNS**

HTTPNDS 不走传统的 DNS 解析，而是自己搭建基于 HTTP 协议的 DNS 服务器集群，分布在多个地点和多个运营商。当客户端需要 DNS 解析的时候，直接通过 HTTP 协议进行请求这个[服务器集群](https://zhida.zhihu.com/search?content_id=148262835&content_type=Article&match_order=2&q=服务器集群&zhida_source=entity)，得到就近的地址。



## 四、反向代理

###  4.1 网关、代理与反向代理

#### 4.1.1 网关

访问网络的入口就是网关，代理服务器也是网关。所有的请求都要经过网关。所以就出现了一个缺点：如果网关的带宽不够，就算你网络的带宽再高也没用。这个问题可用lvs的DR模型解决，DR模型是请求进入时经过网关，但是从服务器向客户端传数据时就不经过网关了。

#### 4.1.2 代理

![image-20241127211526208](.\img\image-20241127211526208.png)

#### 4.1.3 反向代理

![image-20241127211552156](.\img\image-20241127211552156.png)

### 4.2 反向代理应用

**传统架构**

![image-20241127213419149](.\img\image-20241127213419149.png)

![image-20241127213526144](.\img\image-20241127213526144.png)

![image-20241127213540550](.\img\image-20241127213540550.png)

**中小型互联网项目**

url rewrite

![image-20241127213738973](.\img\image-20241127213738973.png)

![image-20241127213758930](.\img\image-20241127213758930.png)

![image-20241127213938877](.\img\image-20241127213938877.png)



### 4.3 反向代理实战

```
    server {
        listen       80;
        # server_name  localhost;
        server_name  www.mmban.com;

        location / {
        	# 一旦配置proxy_pass，root和index就不会再去找
        	proxy_pass http://www.atguigu.com;
            # root   html;
            # root   /www/www;
            # index  index.html index.htm;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }


    }

```

访问 `192.168.116.129` 

![image-20241128203055652](F:\java\Nginx\img\image-202411272047304323.png)

```
proxy_pass http://192.168.116.130;
```

![image-20241128204553697](F:\java\Nginx\img\image-20241128204553697.png)

## 五、负载均衡

### 5.1 简易版本（轮询）

![image-20241127214333683](.\img\image-20241127214333683.png)



```
worker_processes  1;
events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;

    keepalive_timeout  65;

    upstream httpds{
        server 192.168.116.130:80;
        server 192.168.116.131:80;
    }

    server {
        listen       80;
        
        location / {
	proxy_pass http://httpds;
        }

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

}

```

轮询的方式

![image-20241128205329031](F:\java\Nginx\img\image-20241127204730435.png)

轮询的方式无法保持会话



### 5.2 权重配置

```
upstream httpds{
    server 192.168.116.130:80 weight=4;
    server 192.168.116.131:80 weight=1;
}
```

五次内：先连续访问 `192.168.116.130:80` 4次，在访问 `192.168.116.131:80` 一次



#### 5.2.1 down

```
upstream httpds{
    server 192.168.116.130:80 weight=4 down;
    server 192.168.116.131:80 weight=1;
}
```

让某些机器不参与负载均衡



#### 5.2.2 backup

```
upstream httpds{
    server 192.168.116.130:80 weight=4 down;
    server 192.168.116.131:80 weight=1;
    server 192.168.116.132:80 weight=1 backup;
}
```

`backup` 备用机

如果 `192.168.116.131:80` 挂了，那就没有服务器可以使用，`192.168.116.132:80` 就会上线



### 5.3 其他负载均衡策略

#### 5.3.1 ip_hash

根据客户端的ip地址转发到同一台服务器

​		已经不太适应现在这个移动互联网时代了。因为咱们手机客户端移动时IP是会动态变动的，有的时候信号是好是坏 IP 也会变化。所以这种方式不太合适。没法保持会话



#### 5.3.2 least_conn

最少连接访问

​		使后端会话连接数保持一个均衡。但是一般会话连接数比较少的原因是咱们给某台机器的权重比较低，所以连接少。而且也不适合保证服务的上下线

1. 因为我们增加一台新的机器时，连接会话数最少，那么后面是不是所有的连接都会在这台机器上。
2. 配置这个策略需要reload，reload的原理是需要将老的线程kill掉。所以咱们后端服务器的会话数都变为0了



#### 5.3.3 url_hash

根据用户访问的url定向转发请求

​		每个URL地址会被Nginx解析为一个hash值。我们知道注册、登录、首页的url地址是不一致的，如果登录后访问其它页面地址变化了那么url_hash值也会变化，有可能会转发在不同的服务器。

![image-20241129195600585](F:\java\Nginx\img\image-20241129195600585.png)

#### 5.3.4 fair

根据后端服务器响应的时间转发请求

​		需要下载第三方插件，根据后端服务器的响应时间转发请求。也不合理。会出现流量倾斜的情况。因为有可能因为网络原因导致



## 六、动静分离

### 6.1 动静分离原理

适合中小型网站，并发量不高、且文件不多

​		Nginx动静分离的原理主要是基于Nginx作为反向代理服务器的角色，利用其高效处理静态内容的能力，将网站的动态请求和静态请求分发到不同的后端服务器或处理器上，以此来优化资源的加载速度和提升整个系统的性能。

![image-20241129200816519](F:\java\Nginx\img\image-20241129200816519.png)

![image-20241129201037207](F:\java\Nginx\img\image-20241129201037207.png)

没有动静分离的项目，用户键入网址后，会有高并发请求打到nginx服务器，随后nginx服务器向tomcat服务器请求所需资源。实施动静分离后，项目将静态资源打包到nginx服务器



### 6.2 动静分离配置



## 七、URLRewrite





## 七、防盗链
