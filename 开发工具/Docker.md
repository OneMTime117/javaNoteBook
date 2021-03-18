# Docker

## 1、什么是Docker

​	Docker是一个开源的应用容器引擎。可以让开发者将自己的应用和依赖打包到一个轻量级、可移植的容器中，然后可以发布到任何流行的Linux机器（也可以是windows）上运行，并可以实现虚拟化。

​	Docker提出了**集装箱理念**，每个应用在系统中都是一个独立的、标准化集装箱，不会互相影响，并且系统不需要对每个集装箱的安放进行考虑（不需要配置运行环境），并且其虚拟化的实现，相对于虚拟机来说，更加节省物理机的性能资源（相当于一个轻量化的虚拟机，容器内部是一个简易版的Linux环境）

​	**Docker给开发者带来的好处：**

- 简化应用程序的安装和配置，做到一次搭建，到处能用
- 作为轻量化的虚拟机，其启动时间非常快、性能损坏小（方便微服务集群搭建）
- 方便各种应用程序的打包、部署（方便微服务集群部署）

## 2、Docker核心概念

​	docker主机（Host）：安装了Docker程序的机器（安装在服务器上）

​	docker客户端（Client）：通过客户端来使用docker主机

​	docker仓库：存放docker镜像

​	docker镜像：软件打包好的镜像

​	docker容器：docker镜像启动后的实例称之为docker容器；容器是独立运行的一个应用

使用docker的步骤：

1、安装docker

2、去docker仓库下载对应软件的docker镜像

3、使用docker运行镜像，生成一个docker容器，并运行

4、对容器停止，就是关闭对应软件

## 3、安装Docker

- 首先查看Centos版本，要求内核版本高于3.10  通过   **uname  -r** 进行查看

- 版本过低时，可以通过**yum  update**  进行升级

  按照yum-utils， 提供yum-config-manager功能

  ```
   yum install -y yum-utils device-mapper-persistent-data lvm2
  ```

  添加yum镜像源：

  **yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo**

- 通过 **yum list docker-ce --showduplicates | sort -r** 查询docker版本

- 通过**yum install xxx**进行docker安装

- 通过 **docker -v** 可以查看docker的版本号，可以检测docker是否成功安装

- 通过**systemctl start docker** 启动docker应用程序（可以通过 **systemctl enable docker** 设置程序开机自启动）

- 通过 **systemctl stop docker** 关闭docker应用程序

## 4、docker的基本使用

### 1、docker镜像操作（镜像获取部分）

- **docker search **  镜像名

  docker镜像查询    在dockerHub网站中搜索指定镜像，获取查询列表

  列表详情如下：

  | 列名        | 说明                                       |
  | ----------- | ------------------------------------------ |
  | NAME        | 镜像名                                     |
  | DESCRIPTION | 描述                                       |
  | STARTS      | 标星数                                     |
  | OFFICIAL    | 是否为官方镜像                             |
  | AUTOMATED   | 是否为自动配置（官方镜像一般需要手动配置） |

- **docker pull ** 镜像名：标签名

  docker镜像拉取（下载）如果不指定标签名，默认拉取latest 最新版本）

- **docker images**  

  查看docker所有本地镜像

  包括如下几个信息：

  | 列名       | 说明         |
  | ---------- | ------------ |
  | REPOSITORY | 镜像仓库名   |
  | TAG        | 镜像标签     |
  | IMAGE ID   | 镜像id       |
  | CREATED    | 镜像创建时间 |
  | SIZE       | 镜像大小     |

  -a 	列出所有镜像（镜像是存在嵌套，这样能查出所有中间层镜像）

  -q	只显示IMAGE ID

- **docker rmi  **     镜像仓库名/镜像ID

  删除指定本地镜像（通过空格可以实现多个删除）

  **docker rmi  ${docker images -aq}**   删除全部镜像	

### 2、docker 容器操作

#### 2.1、容器创建

- **docker run **      镜像仓库名：版本号     command（一般情况下，不在这指定命令，而使用通过Dockerfile完成）

  运行镜像，从而创建出一个容器，并启动后，执行相应指令

   --name	指定容器名（没有则随机分配） 

  -p	主机端口与容器端口之间的端口映射，如8888:8080  访问宿主机端口8888，则会映射到docker的8080端口 

   --restart always	该容器伴随docker一起自启动

  -d	后台运行，容器启动但不进入，称之为**启动守护式容器**，**注意：如果容器中不存在前台进程，则容器默认自杀停止运行，这是docker自身的一个机制**

  -it         i 以交互模式运行容器，t 为容器分配一个伪终端;两者一般联合使用，在容器启动的同时，进入容器进行终端操作，称之为**启动交互式容器**

  -v	指定容器数据卷

  --restart  always	总是伴随docker一起启动

- **docker ps**  

  查询系统正在运行的docker容器信息

  列表信息如下：

  | 列名         | 说明                                             |
  | ------------ | ------------------------------------------------ |
  | CONTAINER ID | 容器ID                                           |
  | IMAGE        | 镜像名                                           |
  | COMMAND      | 容器启动时，执行的命令                           |
  | CREATED      | 创建时间                                         |
  | STATUS       | 容器状态                                         |
  | PORTS        | 端口映射（不指定则只能本地连接，无法暴露到外网） |
  | NAMES        | 容器名                                           |

  -a	查询系统正在运行的容器信息+历史运行的容器信息

  -l	查询最近创建的容器

  -n	查询最近n个创建的容器

  -q	只显示容器ID

#### 2.2、容器停止、删除、启动、重启

- **docker stop**   容器id/容器name  

  停止容器运行

  **docker kill 容器id/容器name**	强制关闭容器

- **docker rm**   容器id/容器name   

  删除已停止的容器

  -f 	可以删除运行中的容器

  **docker container prune**	删除所有未运行的容器 

  **docker rm -f  ${docker ps -aq}**	删除所有容器

- **docker start**  容器id/容器name 

   启动容器

- **docker restart**	 容器id/容器name	

  重启容器

#### 2.3、容器进入、退出

- **docker  attach**	容器id/容器name

  进入到容器，但是不会以一个新的终端方式交互，因此在终端下执行exit、Ctrl+C会关闭容器

- **docker exec**   容器id/容器name  Command

  在容器中执行命令，但会马上退出容器

  -it	则会以交互式方式执行命令，一般情况下，搭配执行/bin/bash命令用于进入容器：

  **docker exec  -it  容器id/容器name	/bin/bash**

- exit（退出当前终端），如果在交互式启动的终端中执行，则直接会关闭容器；如果是在**docker exec  -it  容器id/容器name	/bin/bash**方式的终端中，则只是退出当终端，不会关闭容器

  ctrl+P+Q快捷键（退出容器，但不关闭），**推荐使用**

#### 2.4、查看容器中的日志、进程、内部细节、文件操作

- **docker logs**   容器id/容器name 

  查看容器中的日志

  -f	跟随最新日志实时追加打印

  -t	显示时间戳

  -- tail  n	显示最后N条日志

- **docker top**   容器id/容器name 

  查看容器中所有的进程

- **docker inspect**     容器id/容器name 

  查看容器中的内部细节，以JSON格式打印

- **docker cp**       容器id/容器name: 文件路径      复制到宿主机外的路径

  用于将docker中的文件，复制到宿主机中（同理对于linux的文件命令，都可以通过docker  + Command来实现，只不过docker中的文件需要添加 **容器id/容器name:** 前缀） 

**对于官方docker镜像。都有使用文档进行参考，获取更多的docker命令操作**

## 5、docker镜像原理

### 1、docker镜像：

​	是一种轻量级、可执行的独立软件包，用于打包软件运行环境和基于运行环境开发的软件，包含运行软件所需的所有内容，包括代码、依赖库、环境变量和配置文件

​	镜像文件，也被称之为UnionFS（联合文件系统）是docker镜像的基础，是一种分层、轻量级并且高性能的文件系统，它支持对文件系统的修改作为一次提交来层层叠加，并可以将不同目录挂载到同一个虚拟文件系统下。因此就可以实现多个文件系统联合加载在一起，对外只提供一个镜像文件

### 2、docker镜像中重要的几个文件系统：

- bootFS： 包含boot加载器和内核，在Linux系统启动是，就会加载一个bootfs文件系统，因此一般情况下，所有Docker镜像都可以共用这个Linux内核系统
- rootFS：包含典型Linux系统中的标准目录和文件，相当于Linux操作系统，而rootfs可以非常精简

### 3、docker镜像分层

通过联合文件系统，docker镜像可以进行类似于JAR包的依赖分层，以tomcat镜像为例，包含如下层级：
bootFs（kernel内核）、rootFS（centos系统）、JDK、tomcat

优点：对于一些基础镜像，通过分层能够很好的实现资源共享，避免重复下载两个镜像所共用的文件系统

### 4、docker镜像文件特定

- docker镜像都是只读

- 在一个容器被启动时，会有一个可写层（文件系统）加载到镜像顶部，也被称之为容器层，而容器层下面的文件系统都叫做镜像层

### 5、docker提交镜像文件

- **docker  commit**   容器ID	新镜像名：新镜像tag

  -a	指定作者

  -m	镜像描述

  在当前镜像创建容器后，我们可以在容器层中对文件系统进行修改，从而改变当前容器的特征；但是在容器被删除后，这些修改并不会保存到原镜像文件中，因此我们就可以通过commit，创建一个新的镜像，并且保存对当前容器的修改

## 6、docker容器数据卷

​	docker容器的生命周期和运行程序一致，其内部数据会随着容器的关闭而消失，因此需要通过数据卷来持久化容器中的数据

​	容器数据卷是一个特殊的目录，单独存放在宿主机中，它可以绕过联合文件系统，为一个和多个容器提供数据，从而实现**容器间的数据共享和数据持久化**

在**docker run**命令中，使用**-v**来指定容器数据卷：

**docker run  -v  宿主机目录：容器目录    镜像id**

此时宿主机目录的数据和容器目录数据互通，可以实现相互读写，当然也可以设置docker容器对数据卷只能读（only reader）

**docker run  -v    宿主机目录：容器目录：or	镜像id**  

**docker run  --volumes-from    A容器id	镜像id**

将A容器作为当前容器的数据卷容器，从而实现将A容器中的数据卷复制到当前容器中，从而实现多个容器使用同一个数据卷，挂载到宿主机的同一个目录中

## 7、Dockerfile

​	将一个应用、软件进行docker构建镜像时，需要有个构建文件来控制整个docker构建过程。**因此Dockerfile就是docker镜像的构建文件**

- **docker  history**   镜像名

  查看指定镜像的构建过程，即Dockerfile构建文件的内容

- **docker  build**   -t     构建的镜像名       执行目录          

  -t	用于指定构建的镜像名和tag

  .	表示在当前目录下执行构建（会影响到Dockerfile文件中所使用到的相对路径）

  -f	用于指定Dockerfile文件路径（不指定，则默认在执行目录下，寻找name=Dockerfile的文件）

  **一般情况下，将Dockerfile文件（注意大小写）和其他需要ADD文件放在当前目录下，dockerfile内部使用相对路径，然后执行：           docker  build   -t       构建的镜像名       .**         

### 1、Dockerfile文件语法特点：

1、指令按照从上到下书写，每一行表示一条执行指令，没有语法结束符，并按顺序执行

2、每条语句的保留字必须大写，并且格式为 ： **保留字    参数**

3、在更具Dockerfile文件进行docker镜像构建时，每条语句都会创建一个中间镜像层，并层层提交覆盖

4、通过**#** 进行语句注释

### 2、docker相关保留字指令：

| 保留字     | 使用说明                                                     |
| ---------- | ------------------------------------------------------------ |
| FROM       | 指定基础镜像,指定当前镜像是在哪个镜像的基础上创建的,FROM  centos |
| MAINTAINER | 镜像维护者的姓名、邮箱地址,MAINTAINER   yh< xxxxx.@e-mail >  |
| RUN        | 镜像构建时，需要执行的命令,   RUN  yum -y install vim        |
| EXPOSE     | 声明容器创建后，对外暴露的端口号（只是声明，用于告诉开发者，容器启动后应该映射的端口，并能控制-P 随机映射所暴露的端口号,EXPOSE   8008 |
| WORKDIR    | 镜像运行创建容器并进入后，默认的工作目录（不指定，则为/）,WORKDIR  /local |
| ENV        | 声明变量，使用在RUN  所声明的命令中，通过**$var** 引入,ENV   path   /usr/tmp |
| ADD        | 复制指定文件，并自动进行解压缩tar包, ADD  jdk.jar.tar    jdk.jar |
| COPY       | 复制指定文件（一般直接使用ADD）,COPY    jdk.jar   jdk.jar    |
| VOLUME     | 指定容器创建的数据卷，但只需要指定容器目录（原因：docker在镜像构建时，并不知道当前系统目录结构，因此会使用docker默认宿主机挂载目录）,VOLUME   /data |
| CMD        | 指定容器运行时，要执行的启动命令(虽然CMD能够指定多个，但只有最后一个会生效，并且可以被docker run命令最后声明的命令 **所替换**),CMD ["each" "hello world"] |
| ENTRYPOINT | 指定容器运行时，要执行的启动命令（同样，ENTRYPOINT可以指定多个，但是只有最后一个会生效，但它所指定的命令会追加到  docker run 命令最后,CMD指令前面）ENTRYPOINT ["each" "hello world"] |
| ONBUILD    | 为一个触发器，并不会按照顺序执行，当该镜像作为基础镜像被继承构建后，则会执行当前对应命令，进行一些额外的工作 |

**CMD和ENTRYPOINT书写格式：**

虽然可以直接字符串形式书写（**shell格式**），但是这样在执行时，会默认在命令前面添加 **/bin/sh -c**，有时会导致命令执行错误，因此一般情况下，使用字符串数组（**exec格式**）来声明，将空格分隔的命令，作为每个单独的字符串数组元素

**当启动命令需要多条时，可以使用sh脚本或者使用&&符号连接**

### 3、Dockerfile镜像构建案例：

**构建springboot的jar镜像：**

1、创建一个文件夹，放入jar、创建Dockerfile文件

2、编写Dockerfile文件，test.jar为j对于jar文件名、app.jar为打包到镜像中的jar包文件名

```dockerfile
FROM openjdk:8
ADD  test.jar  app.jar
EXPOSE 8001
ENTRYPOINT java - jar app.jar
```

3、在当前文件夹下构建镜像,system为构建的镜像名

```she
docker build  -t  system   .
```

4、运行镜像，创建容器

```shell
docker run --name system -d -p 8001:8001
```

## 8 、docker各种常用开发软件镜像启动规范：

- mysql： docker run --name mysql01  -e MYSQL_ROOT_PASSWORD=123456  -p 3306:3360 -d   -restart always  mysql:5.5

指定root用户登入的密码（否则mysql启动失败）、指定端口映射（用于访问mysql）

- redis :     docker run --name redis01    -p 6379:6379 -d   --restart always  redis

- zookeeper:  docker run --name zookeeper01 -p 2181:2181   -d  --restart always zookeeper

  其中--restart  always 指该容器伴随docker启动，而启动

# Jenkins



# K8S