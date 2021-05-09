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

- 启动nginx,cd /usr/local/ginx/sbin,执行niginx shell脚本

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
      worker_connections  1024;   //每个工作进程的最大连接数
  }
  ```

- http模块部分

  ​	nginx实现http服务器功能的核心配置，包括反向代理、缓存、日志等配置。内部又可以分为http（全局http服务器配置）和server（单个http虚拟服务器配置）：

  - http：

    ​	定义http服务器的文件引入、MIME-TYPE、http访问日志文件、连接超时时间、负载均衡等

  - server：

    ​	定义单个http虚拟服务器的端口、url、主机名等

## 3、配置实例

​	1、反向代理

​	2、负载均衡

​	3、动静分离

​	4、高可用集群

## 4、nginx原理