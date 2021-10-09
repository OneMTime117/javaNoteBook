# Nginx

## 1、基本概念

​	nginx（engine x）是一个高性能的HTTP和反向代理的web服务器，同时可以作为电子邮件代理服务器，提供IMAP/POP3/SMTP服务；也可以作为通用TCP/UDP代理服务器（2004年，由俄罗斯人使用C语言开发）

​	优点：内存占用少、并发能力高，是最常用的web服务器，最高能支持50000个并发连接数

### 1、反向代理

**正向代理：**

​	作用在客户端，通过代理服务器，代理客户端情况向目标服务器发送请求，并将结果转发给客户端，从而解决客户端无法正常访问目标服务器的问题，比如使用VNP代理翻墙

**反向代理：**

​	作用在服务端，隐藏目标服务器的存在，对外暴露代理服务器的地址，当客户端对代理服务器发送请求时，会将其转发给目标服务器进行处理，然后把响应结果拿到后，又返回给客户端

**反向代理的作用：**

​	1、保证内网安全，将反向代理服务器作为公网地址暴露，而实际的目标服务器在内网中与代理服务器进行数据交互

​	2、反向代理服务器作为请求唯一入口，简化访问控制

​	3、用于实现目标服务器集群的负载均衡

### 2、负载均衡

​	在高并发场景下，需要多个服务器来共同处理请求，因此就需要一个负载均衡机制，将所有请求均匀分发到每个服务器上。nginx作为反向代理服务器，就可以实现负载均衡

​	根据不同的负载均衡算法，可以实现不同场景下的需求，需要注意的是，负载均衡并不是代表每个服务器处理器请求数平均，而是希望达到每个服务器能够处理与之能力相匹配的负载数，并尽可能减少服务器长期高负载的情况

### 3、动静分离

​	nginx作为web服务器，有很强大的并发能力，因此为了减轻应用服务器的请求数，我们可以将静态资源交给nginx处理，动态资源交给应用服务器（Tomcat）处理；从而实现动静分离，充分发挥nginx作为高性能http服务器能力，也符合现在web项目前后端分离的做法。

## 2、基本使用

### 1、nginx在Linux环境下的安装

- 保证Linux下安装了**gcc、pcre-devel、zlib-devel、openssl-devel**软件包（用于nginx软件包的编译）

  ```shell
  yum -y install gcc pcre-devel zlib-devel openssl openssl-devel
  ```

- 在Nginx官网下载安装包、并进行解压

- 进入nginx安装包目录下，执行configure shell脚本，完成nginx编译参数配置

  ```shell
  ./configure
  ```

  nginx默认配置，会将编译文件安装到Linux系统的**/usr/local/nginx**目录下

- 执行make命令（C语言的编译命令），根据配置好的编译参数，完成nginx软件包的编译，将其变为可执行文件

  ```shell
  make
  ```

- 执行make install，将编译好的执行文件安装到指定目录

  ```shell
  make install
  ```

- 启动nginx,cd /usr/local/nginx/sbin,执行niginx shell脚本

  ```jav a
  ./nginx
  ```

### 2、nginx常用命令

​	由于没有配置环境变量，因此需要在Nginx bin目录下执行，即**/usr/local/niginx**

- 查看版本：

  ```shell
  ./nginx -v
  ```

- 关闭

  ```shell
  ./nginx -s stop
  ```

- 启动

  ```启动
  ./nginx
  ```

- 重新加载（在修改conf文件后，重新生效）

  ```shell
  ./nginx -s reload
  ```

### 3、nginx配置文件

​	配置文件位置：**/usr/local/nignx/nginx.conf**

**nginx.conf 配置文件内容分为三个部分：（仅作为反向代理、http服务器的配置）**

- 全局配置部分：

  ​	从配置文件开始到events模块之前的内容，用于设置一些nginx服务器整体运行配置

  ```java
  user	nobody		//nginx作为后台进程，所显示的执行用户
  pid		4455		//nginx作为后台进程，所显示的pid
  worker_process 		auto	//nginx工作进程数，一般设置为当前主机cpu的物理核心数（auto，则会根据实际主机，进行计算）
  worker_priority		0		//nginx工作进程优先级，默认为0
      
  error_log  logs/error.log  warn	//nginx日志文件，并指定日记输出级别
  ```

- events模块部分：

  ​	用于配置nginx服务器于用户的网络连接

  ```java
  events {
      worker_connections  1024;   //每个工作进程的最大连接数，但实际上，nginx并发数受到Linux系统最大开启的文件数限制（65535）
  }
  ```

- http模块部分

  ​	nginx实现http服务器功能的核心配置，包括反向代理、缓存、日志等配置。内部又可以分为http（全局http服务器配置）和server（单个http虚拟服务器配置）：

  - http：

    ​	定义http服务器的其他配置文件、http访问日志文件、连接超时时间、负载均衡等

    ```JAVA
    http{
        include		mime.types;		//引入MIME-TYPE配置文件，支持多种MIME消息类型
        default_type	application/octet-stream;	//定义默认MIME消息类型
        access_log  logs/access.log; 
        
        //定义日志格式main（name）
        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_forwarded_for"';
        
        //指定访问日志路径、格式(main)
        access_log logs/access.log	main;
        
        //连接超时时间（s）
        keepalive_timeout		65；	
        
        //调用sendfile指令，提供文件磁盘输出速度；但如果为较重的磁盘IO应用，则需要关闭（因为有网络io的瓶颈，因此没必要加快系统的磁盘io速度）
        sendfile	on；
        tcp_nopush     on；//必须配合sendfile使用，在tcp数据包达到一定大小后，进行传输（减轻网络io压力） 
        
        //开启gzip压缩
        gzip  on;
    }
    ```

  - server：

    ​	定义单个http虚拟服务器的端口、url、主机名、动态代理等；location外部为全局配置，内部为指定正则表达式匹配的url处理

    ```java
    server{
        listen	80;//监听端口
        server_name		www.cn；	//设置服务器名称（用于进行host域名匹配）
        root	html;	//设置服务器根目录位置
        index	index.html；//服务器index页面
        
        access_log  logs/nginx.access.log  main;//设置服务器的访问日志
        
        error_page	500 502 503 504 /50x.html;//指定nginx响应状态码，所返回的错误页面
        
        location	/	{	//进行正则表达式匹配url        
            proxy_pass http://127.0.0.1:8080;//反向代理    
        }
        
        location = /50x.html {	//完全匹配访问50x.html页面
            root   html;
        }
    }
    ```

## 3、web服务器的功能和配置

### 1、http服务器搭建

#### 1、http服务监听和请求处理

​	nging通过http模块配置http虚拟服务器，并且至少需要一个server指令：

```java
http{
    server{
        #服务器配置
    }
}
```

​	可以在http中添加多个server指令，来定义多个虚拟服务器：

- **listen指令**

  ​	定义服务器监听所使用的ip和端口：

  ```java
  server{
      listen	127.0.0.1:8080;
  }
  ```

  - 当不指定ip时，监听所有主机网卡的ip地址（IPv4、IPv6）
  - 当不指定端口时，默认为80/8000（根据用户权限决定）

- **server_name指令**

  ​	当nginx多个虚拟服务器监听的Ip、端口一致时，就会根据server_name来匹配请求的host请求头:

  ```java
  server{
      listen 127.0.0.1:8080;
      server_name	localhost;
  }
  ```

  匹配规则和优先级为：

  - 完全匹配
  - 星号开头的最长通配符匹配，如* localhost
  - 星号结尾的最长通配符匹配，如localhost*
  - 没有匹配的，则选择第一个服务器（或在`listen`指令指定了default_server后缀的服务器）

- **location指令**

  ​	定义http服务器对不同url请求的处理方式；nginx作为http服务器，即可以进行请求的反向代理，也可以作为文件服务器

  ​	location可以指定两种类型参数：优先进行前缀匹配，如果不满足，则进行正则匹配

  - 前缀匹配：

  ```java
  location	/some/path/ {
      
  }
  ```

  此时，/some/path/index.html 可以被匹配；但/one/some/path/index.html无法匹配

  - 正则表达式匹配：

  ```java
  location	~ \.html? {
      
  }
  ```

  对于正则表达式，则提供一些匹配符，该定义正则表达式的规则

  | 匹配符 | 规则                     |
  | ------ | ------------------------ |
  | =      | 精确匹配                 |
  | ^~     | 以某个字符串开头         |
  | ~      | 区分大小写的正则匹配     |
  | ~*     | 不区分大小写的正则匹配   |
  | !~     | 区分大小写不匹配的正则   |
  | !~*    | 不区分大小写不匹配的正则 |

  **注意：**

  普通匹配时，会根据最长匹配原则，匹配所有的location参数

  正则匹配则找到第一匹配项后，就会停止,因此location声明顺序，会影响所有正则匹配的优先级

#### 2、http变量

​	nginx预定义许多变量，大多数变量都是在运行中动态生成的，其中就包含了一些常用的http变量：

| 变量         | 含义               |
| ------------ | ------------------ |
| $remote_addr | http请求的客户端ip |
| $uri         | http请求路径       |
| $host        | 请求头host         |

#### 3、返回状态码

​	nginx对于特定的uri请求，可以通过**return指令**来返回状态码，对于30x请求，则可指定重定向地址

- **return指令**

```java
location /home.html	{
    return	301		http://baidu.com
}
```

#### 4、uri重写

- **rewrite指令**

```java
location /user/ {
    rewrite ^/user/(.*)  $/show?user=$1	break;
}
```

rewrite在一个作用域中可以指定多条，从上到下依次执行并覆盖；第三个参数，break：停止后续rewrite指令执行

#### 5、http响应重写

​	对于html静态文件，可以通过**sub_filter指令**（内容过滤器）来实现指定内容的替换：

#### 6、错误处理

- **error_page指令** 

  对于错误状态码（4xx、5xx）,返回错误页面：

```java
location /old/path.html {
    error_page 404	= 301 http:/example.com/new/path.html
}
```

### 2、静态资源处理

#### 1、静态资源访问控制

在不同操作系统下，资源路径也有所不同：

1、window绝对路径要以磁盘开头；Linux则以 / 符号

2、当以目录名开头时，则为相对路径，从nginx目录下匹配

- **root指令**

  用于指定静态资源根目录,可以指定在http\server\location各个级别，遵循就近原则

  ```java
  location /images/ {
      root /data;
  }
  ```

  uri进行location匹配时，会将根目录放在uri前，来访问静态资源，即 root+loaction；如/images/home.html 将转为/data/images/home.html

  **没有强制目录路径以/结尾的要求**

- **alias指令**

  和root指令一样，用于指定静态资源目录

  ```java
  location /images/ {
      alias /data/
  }
  ```

  区别于root，直接在指定目录下寻找资源，不需要拼接上loaction，如/images/home.html将转化为/data/home.html

  **目录路径必须以/结尾**

- **index指令**

  当请求以/结尾时，则视为对目录的请求，并尝试访问目录下的索引文件（默认为index），通过index指令来定义多个索引文件，并按顺序依次匹配

  ```java
  location /images/ {
      root /data;
     	index	index.html index.tml home.page;
  }
  ```

- **autoindex指令**

  对应访问目录请求，当无法访问到目录中的索引文件时，可以使用autoindex指令，来返回目录文件列表

  ```java
  location /images/ {
      root /data;
     	index	index.html index.tml home.page;
      autoindex on;
  }
  ```

- **try_files指令**

  用于检查文件或目录是否存在，可以指定多个参数，并进行重定向，当所有参数都无法匹配时，返回状态码；当可以匹配时，则进行该uri的访问

  ```java
  location / {
      try_files $uri	/images/default.gif
  }
  ```

#### 2、静态资源传输性能优化

- **sendfile指令**

  默认请求下，nginx进行网络文件传输时，会先将文件复制到缓冲区；当使用sendfile指令后，就可以省略这一部，直接将文件复制到传输协议（socket）缓冲区，等待进行网络数据传输

  同时可以通过sendfile_max_chunk来现在sendfile复制文件数据量最大值，避免大文件传输占用整个工作线程，长时间堵塞

  ```java
  localtion /images/ {
      sendfile on;
      sendfile_max_chunk 1M
  }
  ```

- **tcp_nopush指令**

  搭配sendfile一起使用，在协议缓冲区出现大块数据后，再进行网络传输，减少网络I/O次数

#### 3、静态缓存

​	为了减少静态资源的重复访问，基于http与浏览器的缓存机制，nginx可以设置静态资源的缓存时间	

- **expires指令**

  ​	nginx默认，静态缓存是关闭

  ```java
  expires	24h;
  ```

### 3、反向代理

​		nginx支持多种协议的反向代理，并不仅仅限制于http服务器：

- **proxy_pass指令**

  proxy_pass指令	【被代理的目标服务器Ip地址端口/域名】

   	proxy_pass指令用于将请求代理到http服务器上（tomcat虽然是应用服务器，当也可作为http服务器）

  如果location指定了匹配路径，则会将匹配的url部分进行截取，替换为所代理的的服务器地址，比如：

  ```java
  location /some/path/	{
      proxy_pass	 http://www.example.com/link/ ; 
  }
  ```

  则会将请求/some/path/page.html代理到http://www.example.com/link/page.html上

  **注意:**

  ​		**proxy_pass中,不能使用localhost代替本地ip**

- **proxy_set_header指令**

   proxy_set_header      【请求头】	【值】

  nginx在进行请求代理时，会重新定义两个http请求头：Host=$proxy_host（代理的服务器ip地址），Connection=close，并且删除所有值为空的请求头，我们可以通过proxy_set_header指令，自定义请求头

- **proxy_buffering指令**

  ```java
  location /some/path/ {
      proxy_buffers	16	4k;		//缓冲区数量、大小
      proxy_buffer_size	2k;		//请求第一部分的缓冲区大小，一般小于默认缓冲区（主要用于缓存响应头）
      proxy_pass	http://localhost:8000;
  }
  ```

  ​	nginx在进行反向代理时，会对代理服务器的响应进行缓冲，当接受完整个响应后，再发送给客户端，这样可以优化低性能客户端导致的响应时间延长问题，避免浪费代理服务器的请求处理时间，让其能够和nginx快速响应。

  ​	默认情况下，反向代理缓冲是开启的，使用**proxy_buffering  on/off**  进行配置

- **proxy_bind指令**

  ​	如果代理服务器域名对应有多个ip时，可以通过proxy_bind进行指定ip地址的绑定

  ```java
  location	/some/path/
  ```

### 4、负载均衡

​		基于反向代理，nginx可以作为非常有效的Http负载均衡器

- **upstream指令**

  ​	在http模块下，可以通过upstream指令定义一组服务器server，并指定负载均衡策略

  ```java
  http{
      upstream	backed	{		//backed 为组名
          server backend1.example.com  weight = 5 ; 	//server	可以为ip+端口或者是域名
          server backend2.example.com ; 
          server 192 .0.0.1 backup; 
      }
  }
  ```

  ​	然后，通过proxy_pass将请求传递给这组名为backed的服务器组

  ```java
  server{
      location	/	{
          proxy_pass	http://backend;			//backend对于负载均衡服务器的组名
      }
  }
  ```

  **nginx负载均衡策略：**

  | 调度算法   | 概述                                                         | 使用场景                |
  | ---------- | ------------------------------------------------------------ | ----------------------- |
  | 轮询       | 按时间顺序逐一分配到不同的后端服务器**(默认)**               | 一般情况                |
  | 加权轮询   | 加权轮询,weight值越大,分配到的访问几率越高                   | 服务器集群性能不均      |
  | ip_hash    | 每个请求按访问IP的hash结果分配,同一IP的固定访问同一个后端服务器 | 解决session会话保持问题 |
  | url_hash   | 按照访问URL的hash结果来分配请求,同一个URL定向到同一个后端服务器 | 很少用                  |
  | 最小连接数 | 最少连接数,分发到主机连接数最小的那台                        | 很少用                  |

  ```java
  	upstream	systemServer{					
          server 127.0.0.1:8001 weight=5;			//加权
          server 127.0.0.1:8002;					//不加权，则默认为1
          server 127.0.0.1:8003 down;				//手动关闭不参与负载均衡
          server 127.0.0.1:8004 backup;			//备用，当其他所有主服务器无法使用时，参与请求处理
      }    
  
  	upstream	systemServer{					//ip hash
          ip	hash	
          server 127.0.0.1:8001 ;
          server 127.0.0.1:8002;
      }
  ```

### 5、压缩和解压缩

​	nginx通过压缩响应，可以减少传输数据的大小，但是压缩会有很大的性能开销；如果响应已经被压缩时（代理服务器处理过的响应），则nginx不会再进行压缩

- **gzip指令**

  ​	默认，nginx不开启压缩功能，通过命令进行开启

  ```java
  gzip	on;
  ```

- **gzip_types指令**

  ​	默认，nginx只会压缩MIME类型为text/html的响应，可以通过gzip_types指令进行其他类型的指定

  ```java
  gzip_types	test/plain	application/xml;
  ```

- **gzip_min_length指令**

  ​	默认，进行压缩的最小响应包大小为20KB，可以通过gzip_min_length指定

  ```java
  gzip_min_lenght	1000;
  ```

- **gzip_proxied指令**

  ​	默认，nginx不压缩代理请求的响应，通过gzip_proxied可以对于代理请求的响应具有指定的响应标头时，进行压缩

  ```java
  gzip_proxied	expired no-cache no-store private
  ```

- **gunzip指令**

  ​	由于可能存在部分客户端不支持对压缩响应的处理，因此通过配置解压缩功能，让nginx能够在发送数据时，如果客户端不支持压缩响应，则对响应数据进行解压缩，从而满足对于两种不同客户端的压缩处理

  ```java
  gunzip	on;
  ```

- **gzip_static指令**

  ​	通过该指令，能实现对指定文件的请求，返回其压缩版本xx.gz。如果不存在压缩文件，则使用功能未压缩版本

  ```java
  gzip_static	on;
  ```

### 6、Https服务器

省略...

## 4、常规使用

### 1、反向代理

实例：定义两个应用服务的反向代理，并通过url前缀进行区分

```java
server{
    listen 80;
    server_name	www.system.cn
    
    location ~ /system1	{
        root	/home/system1;
        proxy_pass	http://127.0.0.1:8001/system;
    }
    
    location ~ /system2 {
        root /home/system2;
        proxy_pass http://127.0.0.1:8002/system;
    }
}
```

### 2、负载均衡

实例：对两个相同应用服务进行带有负载均衡功能的反向代理,使用同一个静态资源目录

```java
http{
    
    upstream	systemServer{
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
    }
    
    server{
        listen 80;
        server_name	www.system.cn;
        
        location /	{
            proxy_pass	http://systemServer;
            root /home/system;
        }
    }
}
```

### 3、动静分离

​	基于前后端分离理念，动态、静态资源会放在不同的服务器上，而nginx就可以将请求分为动态和静态请求，对应访问不同的服务器；并且nginx本身就可以作为http服务器，因此就可以同location匹配不同请求，进行静态资源访问，或者反向代理应用服务器

```java
http{
    server{
        listen 80;
        server_name	www.system.cn;
        
        location /api/front	{
            root /home/system;
        }
        
        location	/api	{
            proxy_pass	http://systemServer;
        }
    }
}
```

### 4、高可用集群

​	nginx集群基于keepalived实现靠可用，避免单点故障

#### 1、keepalived

​	keepalived是Linux下的一个轻量级高可用解决方案，用于监控集群系统中各个服务节点状态，自动完成故障节点的转移，防止单点故障；起初是为LVS（linux virtual server 虚拟服务器）设计，后加入VRRP功能（虚拟路由冗余协议，实现多个服务节点，对外保留一个ip，并在能力实现主备模式，只有一个主节点作为工作节点，其他为备份节点，用于容灾备份）

#### 2、高可用集群搭建

​	1、安装keepalived

```java
yum	install keepalived	-y
```

​	2、keepalived会被安装在etc/keepalived中，通过修改keepealived.conf,来搭建nginx的主备集群

**省略。。。**

## 5、nginx原理

### 1、master-worker进程机制

​	nginx运行时，分为1个master进程和多个worker进程，所有请求信息先被master获取，无限循环，等待被分发给其他worker进程，单独进行处理。

​	优点：

​		1、对于每个worker进程，都只有一个线程，因此无需要进行加锁处理（同时为了保证并发能力，需要设置多个worker进程，为了最大限度的减少线程间的切换而导致的资源损耗，一般设置worker进程数为CPU的个数）

​		2、通过master来传递信息，实现nginx热部署，即重新加载指令先交给master，当每个worker进程处理完当前请求后，在进行重启;从而避免一次性关闭所有worker

​		3、每个woker进程独立，不会互相影响

### 2、nginx最大请求并发数

- 对于http请求，每个请求需要消耗一个连接数，因此并发数为：worker_connection*worker_processes
- 对于http1.1，每个请求需要消耗两个连接数：worker_connection*worker_processes/2
- 而对于反向代理的http1.1请求，则需要消耗四个连接数：worker_connection*worker_processes/4

### 3、nginx高并发、高性能原因

​	1、master-worker进程机制的事件驱动，避免加锁并且单线程运行，处理速度更快

​	2、epoll模型，属于IO多路复用机制，从而提供i/o效率