# Linux

**是在Unix系统的基础上修改研发的系统，一套免费使用和自由传播的类Unix操作系统**

## Unix：

​	是最早期的多用户、多任务操作系统，但是该系统需要与硬件配套使用，且为商业软件，需要付费使用

cmd：命令提示符，为windows系统操作DOS系统的窗口

dos：磁盘操作系统，用于个人PC上,它是一个单用户单任务的操作系统

BIOS:基本输入输出系统，用于提供计算机最底层、最直接的硬件设置和控制

Android：基于Linux的自由及开放源代码的操作系统，主要使用于移动设备

ios/Mac os X：类Unix的商业操作系统

windows：窗口可视化操作系统，使用GUI来操作计算机

GUI：图形用户界面

## 操作系统版本：

### 1、桌面操作系统：

桌面操作系统主要用于个人计算机上，从软件上分为两大类：

Unix操作系统、windows操作系统

硬件架构分为：

PC版、MAC版

　　1、Unix和类Unix操作系统：Mac OS X，Linux发行版（如Debian，Ubuntu，Linux Mint，openSUSE，Fedora等）；

　　2、微软公司Windows操作系统 ：Windows XP，Windows Vista，Windows 7，Windows 8等。

### 2、服务器操作系统：

服务器操作系统一般指的是安装在大型计算机上的操作系统，如web服务器、应用服务器、数据库服务器

分为三大系列：

　    1、Unix系列：SUNSolaris，IBM-AIX，HP-UX，FreeBSD等；

　　2、Linux系列：Red Hat Linux，CentOS，Debian，Ubuntu等；

　　3、Windows系列：Windows Server 2003，Windows Server 2008，Windows Server 2008 R2等。

### 3、嵌入式操作系统：

应用在嵌入式系统的操作系统，广泛用于生活中：数码相机、手机、平板电脑、家用电器、医疗设备、交通灯、航空电子设备和工厂控制设备等

操作系统有：嵌入式Linux、Windows Embedded、VxWorks等、手机平板端：Android、iOS

**嵌入式计算机系统：**

是一种“完全嵌入受控器件内部，为特定应用而设计的专用计算机系统”；

为控制、监视或辅助设备、机器或用于工厂运作的设备。与通用计算机系统（不是操作系统，如pc）嵌入式系统通常执行的是带有特定要求的预先定义的任务。嵌入式系统通常执行的是带有特定要求的预先定义的任务，通常进行大量生产，所以单个的成本节约，能够随着产量进行成百上千的放大

**计算机系统：**
指用于数据库管理的计算机硬软件及网络系统，由计算机硬件和软件组成，从而使用进行计算

# 1、Linux服务器操作系统：

### Linux发行版：

​		就Linux的本质来说，它只是操作系统的核心，负责控制硬件、管理文件系统、程序进程等，并不给用户提供各种工具和应用软件。所谓工欲善其事，被必先利其器，一套在优秀的操作系统核心，若没有强大的应用软件可以使用，如C/C++编译器、C/C++库、系统管理工具、网络工具、办公软件、多媒体软件、绘图软件等，也无法发挥它强大的功能，用户也无法仅仅使用这个系统核心进行工作，因此人们以Linux内核为中心，再集成搭配各种各样的系统管理软件或应用工具软件组成一套完整的操作系统，如此的组合便称为Linux发行版。

### CentOS发行版：

1、CentOS-7 启动时有两个启动项，前一个为正常启动，后一个为修复模式（当CentOS有问题，不能正常启动时使用）

2、网卡配置操作

cd  /etc/sysconfig/network-scripts/   转到该文件夹（网卡配置文件目录）

ls     获取所有文件列表（网卡配置文件列表）

ip  addr 获取网卡配置信息

systemctl restart network    重启网络服务（用于修改网卡配置后执行，使修改生效）

dhclient      自动获取ip（网卡配置为ip动态分配DHCP） 

查看防火墙状态：systemctl status firewalld.service

​				关闭：systemctl stop firewalld		
​				开启：systemctl start firewalld
​				开机自动关闭：systemctl disable firewalld
​				开机自动启动：systemctl enable firewalld

# 2、基本操作

## 1、Linux入门

### 1.1、Linux系统优点：

开源、安全、高效、**稳定、处理高并发性能优秀**（区别于window server）

### 1.2、GNU计划（人机交互）：

在计算机硬件基础上，添加操作系统内核；然后提供使用操作系统内核的shell程序；最后通过软件实现对shell程序的使用

### 1.3、虚拟机网络搭建：

**虚拟机的三种网络连接：**

- 桥连接：会占用当前本机所在网段的IP地址，并且可以和其他主机的互相通讯（ 因此需要对虚拟机中的网卡进行配置，保证其IP与主机IP在同一网段，使用同一网关；**并且桥接模式需要使用有线网卡，才能访问外网**）
- NAT（网络地址转换）模式：为虚拟机单独指定一个非本机网段IP地址，不会占用当前本机所在网段的IP地址，但是只能单向访问其他主机（Virtual Box无法进行主机到虚拟机的访问；VMware可以本地主机到虚拟机的访问）
- 主机模式：虚拟机是一个独立的主机，不能访问外网；只能和本机进行相互通讯（两者通讯是使用主机的额外虚拟网卡，因此虚拟机ping的主机IP，应该为其虚拟网卡的ip）

**因为为了保证远程访问，且可以访问外网，应该使用桥连接(无线网卡，只能使用nat模式访问外网)**

### 1.4、CentOS7图形化界面：

- 终端窗口:

  Linux终端的复制粘贴：

  -   复制命令：Ctrl + Shift + C  组合键.

  -   粘贴命令：Ctrl + Shift + V  组合键.

### 1.5、vmTools使用：

VMware虚拟机提供工具，来方便Linux系统的基本操作：

- 实现文档复制粘贴
- 实现文件夹共享

## 2、Linux目录结构

- 根目录：  /             （window会根据盘符，分为多个根目录）

- 通过根目录，使用树型结构进行展开，定义其他子目录及文件
- Linux中，一切皆文件；Linux将所有内容都以文件形式保存管理，包括文件、目录、硬件设备、网络通讯

**根目录下，默认子目录作用与特点：**

- /bin         Binary，用于存放常用命令
- /sbin       SuperUser Binary，用于存放系统管理员权限（Root）使用的程序
- /home    存放每个普通用户的主目录，并且目录名为该用户名
- /root       系统管理员主目录
- /lib           library，系统所需要的所有动态连接库，几乎所有Linux程序都会用到
- /etc         etcetra（附加）， 所有系统管理所需要的配置文件
- /usr          unix  system  resource,存放系统程序的安装文件
- /boot        存放启动Linux时的核心文件
- /proc       process 系统内存映射，是一个虚拟目录
- /srv           service，存放服务启动后需要读取的数据
- /sys          system，系统文件
- /tmp         temp，存放临时文件
- /dev          device，设备目录，将所有设备以文件形式进行存储（类似于window设备管理器）
- /media      自动挂载目录，当识别外部设备（U盘、光驱），则将在该目录下临时挂载文件
- /mnt          mount（挂载）,手动挂载目录，用于给用户手动挂载
- /opt            optional（可选择的），存放软件安装包
- /usr/local   存放非自带软件的安装文件
- /var             variable，用于存放不断变化的文件，如日志文件
- /selinux      security-enhanced Linux，存放安全系统

**注意：**

- **Linux默认目录已经规划好了存放内容，因此在使用时应该严格遵守**

- **当用户登入后，默认进入当前用户的家目录；如root用户，进入/root**

## 3、Linux远程操作

Linux远程操作一般包括：文件上传下载和终端访问

**如果进行远程访问，则需要Linux已启动SSHD服务，该服务用于监听22端口，用于创建SSH连接**

- Xshell5:

  用于进行SSH协议的远程登录会话，SSH为安全协议，使用非对称加密方式连接，需要双方安装openssh客户端和服务端，并保证服务端启动ssh服务

  - 可以通过设置快捷键来实现Ctrl C、Ctrl X、Ctrl V，来复制粘贴剪切

- Xftp5：

  基于windows平台的SFTP\FTP软件，进行windows与Linux直接的文件传输

  SFTP是基于SSH安全连接的基础上实现操作，因此默认端口为22；FTP默认端口为21；一般情况下，Linux服务器只暴露22端口，进行远程访问，即使用SFTP

  **Xftp5默认使用系统编码，因此在连接选项中，需要选择使用utf-8**

  **默认新创建用户没有FTP权限，只能使用ROOT**

## 4、Linux开机、重启、用户切换和注销

- **关机和重启**

  - shutdown

    | 命令行            | 作用            |
    | ----------------- | --------------- |
    | shutdown -h now   | 立即关机        |
    | shutdown -h 20:20 | 在今天20:20关机 |
    | shutdown -h +10   | 10分钟后关机    |
    | shutdown -r now   | 立即重启        |
    | shutdown -r +10   | 10分钟后重启    |

  - halt

    直接使用，没有参数，等同于shutdown -h now

  - reboot

    直接使用，没有参数，等同于shutdown -r now


  ​	一般情况下，Linux基本不需要关机、重启；但如果需要关机时，应该执行syn指令，将内存数据同步到磁盘中，防止未写入的数据丢失

- **用户切换和注销**

  ​	尽量避免使用Root账号进行登录，因为Root账号权限太大，避免意外执行某些危险操作；当即使需要Root权限时，也可以在登录后进行用户切换

  **su -l 用户名             切换用户** ；-l（login）可以省略，用户名省略时，默认为root（root切换其他用户时，不需要输入密码）

  logout		注销用户，退出登录

  exit                     返回上一个用户

## 5、用户、组、权限管理

​	Linux在用户创建设计上，每个用户都有自己对应的**一个或多个组**和**一个家目录**

### 5.1、用户

- **添加用户**

  **useradd**	[选项]	yh    指定一个yh用户，并新建一个组以及对于家目录 /home/yh

  -  -d	/home/hh	手动指定家目录（**必须保证该目录之前不存在，因为每个用户的家目录是唯一的**）

    -  -g	用户组	手动指定用户组（默认会新建与用户名同名的组）

- **指定用户密码**

  **passwd**   yh   指定/修改yh用户密码（如果不进行密码指定，则无法正常登录）

- **删除用户**

  **userde**l	[选项]	yh	删除用户（用户删除了，其家目录依然存在）

  - -r	删除用户以及家目录

  一般情况下，删除用户应该保留家目录

- **查询用户信息**

  **id**    yh    查询用户信息：uid（用户id）、gid（组id）、组（组名称）

  **who**   am   i       查询当前终端登录用户名

### 5.2、组

- **创建组**

  **groupadd**    yyy

- **修改用户所在组（主要组）**

  **usermod**  -g     组    用户

- **删除组**

  **groupdel**    yyy （如果存在用户使用了yyy组，则无法直接删除）

### 5.3、用户和组的配置

- 用户配置文件

  **/etc/passwd文件**：用户配置文件，用户名：口令：用户ID：组ID：注解描述：家目录：使用的shell解释器

  **/etc/shadow文件**：口令配置文件，用户名：加密口令：密码最后一次修改实际：最小时间间隔：最大时间间隔：警告时间：不活动时间：失效时间：标志

- 组配置文件

  **/etc/group文件**：组配置文件，组名：口令：组ID：组内用户列表

### 5.4、Linux文件权限管理

在Linux系统中，所有文件都有三个属性：所有者、所在组、其他组（系统中除所有者外，其他用户的组都是该文件的其他组）

**1、查看文件的所有者和所在组**

通过 ls -l	可以查询文件的详细信息，也就包括**文件所有者、所在组**，比如：

```shell
-rw-------. 1 root root 1572 12月  1 14:48 anaconda-ks.cfg
-rw-r--r--. 1 root root 1620 12月  1 14:52 initial-setup-ks.cfg
drwxr-xr-x. 2 root root    6 2月  25 21:47 yh
drwxr-xr-x. 2 root root    6 12月  1 15:48 公共
drwxr-xr-x. 2 root root    6 12月  1 15:48 模板
```

文件详细信息按顺序为：权限、连接数（1，表示为硬连接，为一个文件；多个则表示为一个目录）、所有者、所在组、文件大小、创建时间、文件名 

**2、修改文件的所有者和所在组**

**chown**	yh	apple.txt	修改apple.txt文件的所有者为yh（change owner）

**当一个用户创建一个文件后，该文件的所有者和所在组就是该用户和用户所在组，但在修改文件所有者用户后，其所在组并不会随之改变**

**chgrp**	root	apple.txt	修改apple.txt文件的所在组为root（change  group）

chown   yh：root apple.txt	修改文件所有者和所在组（使用更多，因此一般情况下，所有者的组就是所在组，因此修改所有者时，需要一起改变）

- -R ：对于目录，可以进行递归操作，批量修改目录下所有文件权限的使用者或所在组

  chown    -R    yh：root  home

**3、文件权限解析**

文件权限字符串通过10位字符来进行表示

第一位：表示文件类型，-普通文件、d目录、l 软连接、 c 字符设备（键盘鼠标） b块文件（硬盘）

第二、三、四位：表示文件所有者的权限：第一个为r/- 表示有无读权限；第二个为w/-表示有无写权限；第三个为

x/-表示是否文件是否能被执行，目录是否允许进入

第五、六、七位：表示文件所在组的权限，同理

第八、九、十位：表示文件其他组的权限，同理

**4、文件权限修改**

- chmod指令   执行操作	指定文件	修改指定文件的权限

  - u、g、o、a表示需要修改的权限属性：

  u：所有者 user

  g：所在组 group

  o：其他组 other	 

  a：三个权限属性总和 all

  **修改方式一：**

  通过+、-、= 配合r、w、x,来表示权限增删修

  chmod  u=rwx    hello.java  表示给hello.java文件的所有者提供读、写、执行权限

  chmod  g+r	hello.java 表示给hello.java文件的所在组添读权限

  chomd a-w	hello.java 表示给hello.java文件的其他组取消写权限

  chmod u=rw-,g=rw-,o=--x hello.java  通过逗号，可以对三个属性同时修改

  **修改方式二：**

  通过数字来表示u、g、o三个属性的实际值（这种方式更加方便）

  r 表示4、w表示2、x表示1，因此：

  rwx=7、rw-=6、r-x=5、r--=4、-wx=3、-w-=2、--x=1、---=0

  chmod 764 hello.java  表示所有者有读写执行权限、所在组有读写权限、其他组有读权限

## 6、Linux系统运行级别

Linux在设计上，提供了7个级别：

| 运行级别 |                                        |
| -------- | -------------------------------------- |
| 0        | 关机（相当于halt指令）                 |
| 1        | 单用户（不需要密码直接登录）           |
| 2        | 多用户无网络（类似于window的安全模式） |
| 3        | 多用户有网络（最常使用）               |
| 4        | 保留（没有详细定义，一般不使用）       |
| 5        | 图形界面                               |
| 6        | 重启（相当于reboot指令）               |

**对于单用户运行级别的使用：**

​	当我们直接使用主机重启时，在加载Linux内核前，修改其运行级别为1，此时就可以进入单用户模式运行，从而实现**root密码找回**（远程登录方式重启时，无法进入Linux内核加载前的界面，因此也就保证了单用户运行级别的安全性，即使用者必须在主机上操作）

### 6.1、设置系统运行级别

**init   Num        切换当前运行级别**

- centOS6中，是提供**/etc/inittab文件**进行默认运行级别的配置；但centOS7中，则使用systemctl命令行形式：

  - systemctl    get-default                       获取当前系统默认运行级别

  - systemctl     set-default       grahical.target(multi-user.target)       设置当前系统默认运行级别为5（3）              

```
0 shutdown.target
1 emergency.target
2 rescure.target
3 multi-user.target
5 graphical.target
```

## 7、vi/vim编辑器

- 所有的Linux系统都内置vi文本编辑器，类似于windows中的记事本；都具有程序编辑能力
- Vim是Vi的增强版，可以通过字体颜色来分区语法结构，方便程序编程。

**vim   xxx             使用vim进行当前文件编辑**

### 7.1、vi/vim三种编辑模式

- 正常模式：

  ​	当使用vim打开一个文档后，就直接进行正常模式（默认），可以通过键盘来移动光标，并可以使用快捷键进行复制、粘贴、删除字符和整行删除

- 插入模式：

  ​	通过输入i/I、o/O、a/A进入插入模式，此时就可以直接通过键盘进行文本编辑；当编辑完成后，通过ESC退出插入模式

- 命令行模式：

  ​	输入：进入命令行模式，此时就可以通过输入命令行，来实现显示行号、光标快速移动（第一行、最后一行）、保存、退出等操作；每当执行一次命令行操作后，就会自动退出进入正常模式

### 7.2、vi/vim常用快捷键和使用场景

- 正常模式：

  | 快捷键    | 作用                                                         |
  | --------- | ------------------------------------------------------------ |
  | i/I       | 进入插入模式，并在光标前插入；在光标当前行的第一个字符前插入 |
  | o/O       | 进入插入模式，并在光标下一行新建空白行插入；上一行新建空白行 |
  | a/A       | 进入插入模式，并在光标后插入；在光标当前行的最后插入         |
  | G         | 移动光标到第一行开头                                         |
  | gg        | 移动光标到最后行结尾                                         |
  | dd        | 删除当前行；5dd：删除当前行开始的往下5行                     |
  | yy        | 拷贝当前行；5yy：拷贝当前行开始的往下5行                     |
  | p         | 将拷贝的数据插入到当前行的下面                               |
  | u         | 撤销操作（CTRL+R 取消撤销）                                  |
  | ZZ        | 保存并退出                                                   |
  | /xx +回车 | 查找，之后通过n/N跳转到下一个匹配值所在节点                  |

- 命令行模式

  | 快捷键   | 作用            |
  | -------- | --------------- |
  | set nu   | 显示行号        |
  | set nonu | 不显示行号      |
  | 1        | 移动到第1行开头 |
  | w        | 保存            |
  | q        | 退出            |
  | q！      | 强制退出        |
  | wq       | 保存且退出      |

## 8、Linux常用实用指令

### 1、Linux帮助指令

​	**Linux提供帮助指令，让我们了解所以指令的使用、参数功能描述**

- man

  **man     指令**

  Linux提供一本帮助手册，它会根据指定的指令进行搜索查找，并展示详细的描述

- help

  **help     指令**

  用于**获取shlle内置指令**的帮助信息，更加简洁

- help外部命令

  **指令     -help（h）**  

  所有外部指令都提供了 -h参数，用于查询当前外部指令的帮助信息

### 2、文件目录操作指令

- **pwd指令**

  pwd             显示当前目录的绝对路径(**print work directory  打印工作路径**)

- **ls指令**

  ls		显示当前目录或文件信息（**list files  文件列表**）

  - -a		显示所有包括隐藏的文件、目录
  - -l		以列表的形式显示信息（并额外显示文件相关信息）
  - -R             recursively（递归），递归列出所有子文件

**指令参数使用时，直接在 -后面进行参数选项拼接，如 ：**

ls  -al展示当前目录下所有文件、目录,包括隐藏文件，并且以列表形式展现

- **cd指令**

  cd     /home/yh		切换目录（**change directory**）

  **常用目录表的方式：**

  - ~ 		表示用户家目录

  - .	          表示当前目录		


  - ..		表示上一级目录  ../.. 就表示上上级目录

- **mkdir指令**

  mkdir	[选项]	/home/yh		创建目录（**make directory**）

  - -p		可以创建多级目录（默认情况下，无法创建多级目录）

- **rmdir指令**

  rmdir	/home/yh		删除空目录（**remove directory**）

- **rm指令**vi

  rm	[选项]	/home/yh		删除一个文件或目录（**remove**），默认是无法删除目录的

  - -r		递归删除（实现非空目录的删除，但每个文件和目录的删除都会询问）

  - -f		强制删除（不询问）

  **一般情况下，直接使用rm   -rf   进行非空目录的删除**

- **touch指令**

  touch     文件名称		创建一个空文件（可以指定绝对路径，但必须保证目录存在）

  通过指定多个文件名称，可以实现一次性创建多个文件

  文件重复时，会自动覆盖

- **cp指令**

  cp	[选项]	文件	/home/yh		拷贝文件到指定目录下、或指定文件实现复制重命名（**copy**）

  - -r	递归拷贝所有文件以及目录

  \cp	xxx	对于重名文件强制覆盖

- **mv指令**

  mv	旧文件路径	新文件路径	移动文件（**move file**），将原文件移动到新文件目录下，并进行重命名

  - 当新文件路径不指定文件名时，则单纯进行文件移动，如：

    mv	/home/yh/test.txt	/home/hhh			将test.txt移动到/home/hhh目录下

  - 当旧文件路径的目录和新文件路径目录相同时，则单纯进行文件重命名，如：

    mv	/home/yh/test.txt	/home/yh/test2.txt		将test.txt重命名为test2.txt

- **cat指令**

  cat	/home/test.txt		连接文件并打印（**concatenate**）

  -n		显示行号

  cat指令直接使用，会直接在所有内容进行打印，因此无法很好了逐行查看，可以通过**| more（管道命令**）来实现：**（本质上就是将cat指令查询的所有内容，交个more指令处理，从而实现分页）**

  cat	/home/test.txt	| more		但是无法向上翻页

- **more指令**

  more	/home/test.txt		类似于cat，但会以全屏方式一页一页进行展示

  **全屏模式快捷键：**

  空格下一页；b上一页；Enter下一行；q退出全屏模式（当展示完后，自动退出全屏模式）；v 使用vim进行文本编辑

- **less指令**

  less	/home/test.txt		类似于more，但是不会直接全屏加载所有内容，然后进行分页查看；而是分屏每下一页，读取一次数据进行展示，因此效率非常高，适合查看大型文件（日志文件）

  **分屏模式快捷键：**

  空格下一页；b上一页；上下方向键跳转行，q退出分屏模式（不会自动退出）；v 使用vim进行文本编辑

**cat、more、less都能够在分页查询状态下，使用/ xxx,进行字符串查找**

- **>指令和   >>指令**

  xxxxx    >/>>   hello.java 

  前者是将内容覆盖写入到指定文件中；后者是将内容追加到指定文件的末尾

  可以搭配一些信息查询指令，将查询到的数据写入指定文件中，如ls、cat、echo:

  ls  -l  >> hello.java    将当前目录列表信息追加到hello.java文件的末尾

  **注意:**

  **1、该指令只能对打印到控制台的数据进行操作，无法直接处理字面量或变量中的数据，因此对于变量中数据的处理，需要搭配echo指令**

  **2、先执行指令前的命令，然后将命令打印结果写入指定文件，并需要确保文件存在**

- **echo指令**

  echo   xxxx   用于控制台输出，一般情况下用于输出变量,如：

  echo $PATH   输出系统环境变量

- **head指令**

  head   hello.java   用于输出指定文件的前10行内容

  -n  5   指定输出前5行数据

- **tail指令**

  tail  hello.java  用于输出指定文件的后10行内容

  -n 5  指定输出后5行

  -f     实时追踪文档所有更新，一般用于实时监控日志文件

- **ln指令**

  - ln  /root/hello.java    hello.java        创建当前路径下的hello.java硬链接（**Link**），引向/root/hello.java；用于实现一个文件存在多个有效路径名。

  硬链接操作元素只能是文件，源文件自身就是一个硬链接，当所有硬链接被删除时，该文件才会被删除。**并且多个硬链接共用同一个文件，相对于复制减少磁盘占用，避免文件被误删除**

  - ln -s  /root/hello.java    hello.java       创建将当前路径下的hello.java软连接（类似于windows的快捷方式），引向/root/hello.java

  软连接操作元素可以是文件或者目录，当源文件删除后，软连接文件还是会一直存在，只是指向一个无效链接，它们是不同的两个文件，只是软连接文件存储了源文件的真实路径；在操作软连接文件时，当前路径为软连接文件路径

- **history指令**

  history    用于获取当前系统执行过的所有指令

  history   10   获取后10行历史指令

  ！10    执行第10行历史指令

### 3、时间日期指令

- **date指令**

  date           显示当前时间

  通过 +  来设置时间显示格式，如显示日期

  date  +%Y:%m:%d   :只是自定义的分割符，可以随意代替或省略

  **注意：对于格式，如果存在空格时，则需要使用引号括起来**

  | 参数 | 描述 |
  | ---- | ---- |
  | %Y   | 年   |
  | %m   | 月   |
  | %d   | 日   |
  | %H   | 时   |
  | %M   | 分   |
  | %S   | 秒   |

  date -s  时间字符串            设置当前系统时间,如 date  -s   “2018-10-19 11:22:22" 

- **cal指令**

  cal        显示当前日历图表（**calendar**）

  cal     2020   显示2020年12个月的日历图表

### 4、搜索查询指令

- **find指令**

  find [搜索范围]   [选项]   在指令目录下向下递归变量所有子目录，找到满足条件的文件或目录显示在终端

  常用选项：

  - -name xxx    通过文件名查找

    find /home   -name hello        在/home目录下寻找文件名为hello的文件或目录（打印其绝对路径）

  - -user  xxx     通过文件所属用户名查找

    find  /home    -user   root     查找属于root用户的文件、目录

  - -size   xxx     通过文件大小查找

    find  /home -size  +20M      查找大于20M的文件，同理-20M 为小于20M； 20M 为等于20M

- **locate指令**

  locate也是文件查找指令，但是相对于find指令的速度更快，原因是：Linux系统会为系统所有文件和目录建立一个locate数据库，用于通过**文件名快速查找指定文件路径**，无需对系统文件进行遍历，速度较快，但是需要管理员定期更新locate数据库

  - updatedb          在使用locate指定前，需要先创建locate数据库
  - locate    hello    在linux中查询所有文件路径中含有hello关键字的文件

- **grep指令和管道符号|**

  **Globally search a Regular Expression and Print，全局搜索正则表达式并打印**

  grep   [选项]  查找内容  源文件    过滤查找，在指定文件中查询相应内容（会显示当前匹配内容行的所有数据）

  - grep -n   helloword   hello.java   过滤查找，并显示查询内容所在行数
  - grep  -i    helloword  hello.java    过滤查找，并忽略查询内容的大小写，进行匹配

  **管道符号：|，用于将前面指令要打印的数据交给后面指令处理**

  grep指令在使用时，一般搭配管道符号，对指定数据进行过滤查找：

  - ls -l    |  grep -n    hello         查找当前目录列表中含有hello的部分行内容

### 5、压缩和解压缩指令

- **gzip/gunzip指令**

  gzip	hello.java	压缩文件，将文件压缩为*.gz压缩文件（在源文件名.gz后缀）

  gunzip	hello.java.gz	解压缩文件，解压*.gz压缩文件

  **它们进行文件压缩、解压缩时，并不会保留源文件；并且不能对目录进行操作**

- **zip/unzip指令**

  zip	[选项]	hello.zip	hello	压缩文件或目录，压缩为hello.zip压缩文件(zip必须手动设置生成的zip文件名)	

  - -r	对于目录，进行递归压缩（必须有，否则只会压缩hello该目录）

  unzip	[选项]	hello.zip   解压*.zip文件

  - -d    指定解压缩后，文件的存放目录

  - -n    对相同文件不进行覆盖
  - -o    对相同文件进行覆盖

- **tar指令**

  tar	[选项]	/home	对指定文件进行打包、压缩，或者解压、解包（tape archive磁带归档）

  - -c	生成*.tar打包文件，即打包指定文件或目录
  - -v        显示执行信息
  - -f         指定压缩后的文件或解压的文件（**必需指定**）
  - -z         搭配-c：打包同时进行gzip压缩，生成*.tar.gz；搭配-x：解包前，先进行gzip解压
  - -x         解包.tar文件

一般情况下，打包和压缩同时进r行，因此一般都是操作的*.tar.gz文件：

**注意：cz、zx的顺序有严格要求，即先打包后压缩、先解压后解包**

打包压缩文件

​	tar	-cvfz	myhome.tar.gz	/home

解压、解包tar.gz：

​	tar	-zxvf	myhome.tar.gz                    

​	**在整个解压指令结尾，可以添加	-C	/home来指定解压后，文件存放的目录（需要保证该目录存在）**

- **.tar、.zip、.gz三种后缀文件的区别：**

  - 磁盘归档文件，实现将多个文件（目录）打包到单个文件中，并提高在磁盘的中高效写入

  tar，tape archive磁带归档，就是Linux中的一种磁盘归档文件，实现将目录的打包为单个文件。

  zip和gz（gzip）是针对于文件压缩，实现减少文件的磁盘占用。但前者zip本身也实现的磁盘归档的功能，而gz只是单纯的文件压缩，因此只能操作单个文件。

## 9、Linux定时任务调度

任务调度：系统在某个特定时间点执行特定命令或程序

任务调度分类：

1. 系统工作，如系统病毒扫描
2. 个别用户工作，如mysql数据库备份

Linux所有任务调度都是通过crond服务来控制，并且默认是开启启动的，每一分钟会检查一次Linux所有cron调度文件中需要执行的命令。

- **crontab指令**

  crontab  [-u user] [选项]	针对于个别用户工作，进行定时任务调度操作（crond table）

  - -e	编辑crontab文件内容
  - -l         显示crontab文件内容
  - -r         删除当前用户的crontab文件

  [-u  user]    用于指定某个用户的crontab服务，可以省略，表示操作的是当前用户

  crontab操作的crontab文件会自动保存到**/var/spool/cron目录**下，文件名和所操作的用户同名。我们可以直接操作该文件来设置任务，但是并不会保存后就生效，而是需要重启crond任务服务

  系统任务调度的crontab文件为**/etc/crontab**,前四行为用于配置crond任务运行的环境变量，后面添加需要的系统任务（定时重启等）

- **crond服务操作**

  crontab指令的实现依赖于Linux系统中的crond任务服务，而作为服务就可以进行相应的stop、start、restart和配置文件重载，如：

  systemctl restart crond   重启crond服务

- **cron表达式**

  在crontab文件中，通过cron表达式+command 来实现配置定时任务，其中cron表达式就是用于表示定时策略:

  cron表达式由5个占位符组成：

  ```java
  * * * * *		分别表示分钟（0-59）、小时（0-23）、日期（1-31）、月份（1-12）、星期（0-6,0为星期天）
  ```

**特殊符号含义：**

| 符号 | 含义             |
| ---- | ---------------- |
| *    | 表示任何时间     |
| ，   | 表示不连续时间   |
| -    | 表示连续时间，   |
| */n  | 每隔多久执行一次 |

实例：

```java
* * * * * command	每分钟执行
3,15 * * * * command	每小时的3分、15分执行
3,15 8-11 * * * command	每天8点到11点钟的3分、15分执行
3,15 8-11 * * 1 command	每周1的8点到11点钟的3分、15分执行
3,15 8-11 */2  *  * command 每隔两天的8点到11点钟的3分、15分执行
```

## 10、磁盘分区、挂载

### 10.1、磁盘分区

- mbr分区：
  - 最多支持四个主分区
  - 系统只能安装在主分区中
  - 扩展分区要占一个主分区，用于划分成多个逻辑分区
  - MBR一个主分区最大只支持2TB，但拥有较好的兼容性

- gtp分区
  - 支持无限多个主分区（但会被操作系统限制，windows最多128个分区）
  - 分区最大支持18EB的容器（1024*1024GB）
  - windows7后支持gtp

### 10.2、硬盘挂载

1、虚拟机添加硬盘（物理机）

2、磁盘分区

​	**fdisk**	/dev/sdb1           给/dev下的磁盘硬件sdb1进行磁盘分区

会进入到一个引导页面，一般步骤为 n：新增分区	p：选择为主分区类型	回车：默认使用全部容量	w：将进行分区并退出

3、格式化磁盘设备(make file system)

​	**mkfs**	-t	ext4	/dev/sdb1           给/dev下的磁盘硬件sdb1格式化为 ext4（文件系统类型，扩展文件系统）

4、创建挂载目录、手动挂载

​	mkdir	/mnt/newDisk	新建一个用于挂载磁盘的目录（一般在/mnt目录下创建）

​	**mount**	/dev/sdb1	/mnt/newDisk	将以格式化分区好的磁盘，挂载到指定目录下，从而进行使用

- **umount**	目录/设备路径	用于取消指定挂载目录或设备的挂载

5、可以设置成自动永久挂载，在linux系统启动时，和系统盘类似，自动挂载

​	在/etc/fstab配置文件，添加相应信息

**注意：对于U盘，不需要分区，可以直接进行挂载**

### 10.3、磁盘信息指令

- **df指令**

  df	查询系统磁盘使用情况（disk free）

  -h	适合人类可读的格式（human readable）

- **du指令**

  du	/home	查询指定目录/文件磁盘占用大小（disk ），目录会自动进行递归操作

  -h	适合人类可读的格式（human readable）

  -a	查询所有文件

  -s	目录不进行递归操作（文件则不能使用）

## 11、个数统计

- **wc指令**

  wc	helloworld.java	统计指定内容的字符数、字节数、行数

  可以搭配|管道符号使用	

  -c	只显示字节数

  -w	只显示单词数

  -l	只显示行数

- 统计字符个数：

```shell
wc	-c	helloworld.java
```

- 统计文件数：^-       表示每行开头第一个字符为-

```java
ls -l |grep ^-	|wc -l
```

- 统计目录数：^d	表示每行开头第一个字符为d

```shell
ls -l |grep ^d	|wc -l
```

# 3、进程管理

## 1、Linux进程基本概念

1、每一个执行的程序、代码都称之为一个进程，每个进程都分配一个ID号

2、每个进程都对应要给父进程，一个父进程可以有多个子进程

3、进程存在的方式有两种：前台和后台，前台进程用户可以直接在屏幕上看到并操作；后台进程则无法显示在屏幕中，以后台方式执行

4、一般系统服务都是以后台进程方式存在，知道系统关机时才结束运行

## 2、基本进程指令

- **PS指令**

  ps	[选项]	查看当前终端下，正在执行的进程信息（**process 运行、进程**）

  进程信息：

  | 字段 | 说明                                                         |
  | ---- | ------------------------------------------------------------ |
  | PID  | 进程识别号（唯一id）                                         |
  | TTY  | 终端机号（远程或本地操作系统的终端窗口唯一id），**Teletype**电传打字机，用于输入指令字符的设备，本地tty也叫做控制台（console） |
  | TIME | 进程所消耗CPU的执行时间                                      |
  | CMD  | 当前进程所执行的命令（指令代码）或进程名（程序），（太长会被截取） |

  -a	显示当前终端下的所有进程信息（包括其他用户的程序）

  -u	以用户格式显示进程信息（会显示额外信息：用户名user、占用CPU量CPU、占用内存量MEM、虚拟内存量VSZ、物理内存量RSS、进程状态STA（s表示休眠、r表示运行））

  -x	显示额外没有控制终端的进程

  **一般情况下，我们只需要查看指定进程的状态，因此需要配置| grep管道符+过滤指令，来筛选：**

  **ps	-aux	|grep	xxx**

  **ps	-ef	用于查看所有进程，并且显示其父进程**

- **kill、killall指令**

  kill	[选项]	用于通过进程号杀死进程

  killall	进程名	用于通过进程名杀死进程，且支持通配符，一次性杀死多个满足匹配条件的进程

  -9	强制杀死进程

- **pstree指令**

  pstree	[选项]	以树形图方式查看进程信息

  -p	显示进程pid

  -u	显示进程所属用户

## 3、服务管理指令

​	服务也是一种进程，运行在后台中，也称之为守护进程

### 1、Linux系统启动过程

- 内核引导，BIOS进行开启自检，启动设置，之后引入硬盘中/boot目录下的内核文件
- 运行init进程，确定系统的运行级别
- 系统初始化，根据init进程运行的脚本，完成系统的初始化
- 建立终端
- 用户登录系统

对应init进程，在CentOS不同版本中，也对应不同：

- CentOS5，使用SystemV
- CentOS7，使用Systemd

### 2、SystemV下的服务管理

**service指令：**

​	service	服务名	start|stop|restart|reload|status	服务启动、停止、重启、重新加载配置文件、服务状态查询

**chkconfig指令：**

​	chkconfig	--list	显示所有SysV服务在每个级别下的自启动状态（check  config）

​	chkconfig	--level	5	服务名	on/off	指定服务在5运行级别下的自启动状态

### 3、Systemd介绍

#### 1、由来

​	Linux的启动采用init的进程（SystemV）来完成，但这样会有两个缺点：

- 启动时间长，init进程的代码是串行执行的，执行时间长

- 启动脚本复杂，init进程本质上就是执行了一段启动脚本，因此如果需要进行一些特殊处理，则要编写很长的脚本

  因此Systemd就由此诞生，它设计目标就是为系统的启动和管理，提供一套完整的解决方案；d是守护进程（daemon）的缩写，Systemd，就是系统的守护进程，成为系统的第一个进程（PID为1），而后面运行所有的进程，都是它的子进程

#### 2、特点：

​	Systemd功能强大，使用方便，但是体系庞大复杂，与操作系统的其他部分耦合性很高，违反了keep simple, keep stupid的Unix 哲学

​	Systemd将所有脚本称之为unit，分为12类：service、socket、target、path...,并且提供对这些脚本执行的管理操作；每个unit都有一个配置文件，来告诉systemd如何启动；systemd默认从目录**/etc/systemd/system/**读取配置文件，而该目录中大部分都为符号连接（软连接），真正的配置文件存放在**/usr/lib/systemd/system/**中，而**自启动命令的作用就是创建两者的软连接**，因此自启动不会受默认target的影响

#### 3、Systemd常用指令：

**Systemctl**就是Systemd的主命令，用于管理系统资源（Unit）:

- systemctl         [command]         [unit]	管理指定unit

command主要有：

| command                        | 作用                                                    |
| ------------------------------ | ------------------------------------------------------- |
| start/stop/restart/reload/kill | 开启、停止、重启、重载、杀死                            |
| enable/disable                 | 开机自启动开启或停止（Systemd中不存在系统不同运行级别） |
| status                         | 状态                                                    |
| show                           | 查看unit底层参数                                        |
| is-active                      | 是否运行                                                |
| is-enabled                     | 是否自启动                                              |

- systemctl	list-units	[选项]	查看所有unit列表

  --all	列出所有unit，包括启动失败的

  --type=unit	列出所有service类型的unit

- systemctl有关target类型的单位操作

  target就是unit组，包含了需要unit，也就对应了SystemV中的7个运行级别，但是**在设计上，System中7中运行级别是互斥的，但Systemd中的target可以同时指定一起启动**

  systemctl	get-default	获取系统的默认启动组

  systemctl	list-units	--type=target	查询系统所有target

  systemctl	set-default	xxx.target	设置系统默认启动组

  systemctl	isolate	xxx.target	切换组，并关闭前一个组中不需要启动的unit

#### 4、自定义systemd服务

分为三步，脚本内容根据具体程序决定：

- 自定义服务脚本，/usr/lib/systemd/system/tomcat.service

- 添加可执行权限
- 设置该服务为开启自启动

### 4、CentOS7常用服务

- firewall.service	防火墙（IP、端口管理）
- sshd.service         SSH远程连接
- network.service          网络管理（其指令ifconfig，查看当前网络状态）

### 5、常用进程监控指令

- **top指令**

  top	[选项]	和ps指令类似，用于显示正在执行的进程信息，但是top可以进行动态监控，实时更新当前进程状态（**提供整体系统的运行状态**）

  -d	指定top命令每隔几秒更新执行，默认3秒

  -i	top不再显示任何停止（stopped）或僵死（zombie）的进程

  -p	指定pid，动态监控某个进程的状态

  top进行动态监控数据展示时，提供交互操作：

  P	以cpu使用率排序

  M	以内存使用率排序

  N	以PID排序

  q	推出top动态监控

- **netstat指令（属于network服务指令)**

  netstat	[选项]	监控系统所有网络服务状态，即需要暴露监听端口的服务（net status）

  -a	所有tcp、udp、unix网络连接

  -at	所有tcp网络连接

  -p	额外展示PID和服务名

  -n	显示端口号，而不是用服务名代替

  因此一般情况下，使用:

  ```shell
  netstat	-atpn
  ```

  打印的数据中，0.0.0.0表示所有IVP4地址，：：：：表示所有IVP6地址

## 4、普通进程管理

### 1、进程和会话

​	每个进程都对应如下参数：

| 参数 | 作用                                                 |
| ---- | ---------------------------------------------------- |
| uid  | 进程创建的用户id                                     |
| pid  | 进程id                                               |
| ppid | 父进程id                                             |
| pgid | 进程组id，进程组由一些进程组成，并选择其中一个主进程 |
| sid  | 会话id，每个会话包含一个前端进程组和n个后台进程组    |

1、每个控制终端都对应创建一个会话，而会话中的首个进程被称之为控制进程，一般为 bash进程，即shell程序，用于操作linux系统

2、当终端关闭时，会发送 sighup信号给bash控制进程，而bash进程会将sighup传递给其他进程；而sighup信号就会默认让进程执行退出操作	

### 2、进程控制

#### 1、进程启动

​	进程启动分为手工启动和后台启动：

- 手工启动：

  输入命令或执行程序，来实现进程的前台或后台启动

- 后台启动：

  区别与后台进程，只是执行命令行，并在后面添加’&‘，不占用前台界面，但整个进程会和当前控制终端的会话绑定

#### 2、进程停止（挂起）、恢复

​	通过Ctrl + Z组合键，可以是当前进程挂起（调入后台，并停止运行）

- **jobs指令**

  通过jobs命令，可以查看当前终端在后台的进程任务（后台启动和后台挂起的进程）

  jobs  -l	显示后台的进程任务，并且显示pid

- **bg指令**

  通过bg指令（BackGround），可以将后台中挂起的任务恢复，并在后台执行

  bg	pid	需要指定进程对应的pid

- **fg指令**

  通过fg指令（ForeGround），可以将后台任务重新恢复到前台运行

  同样，也需要指定pid

#### 3、进程终止

​	1、通过Ctrl+C组合键，可以直接终止当前进程

​	2、关闭当前终端，此时当前终端对应session下的所有进程会自动杀死

### 3、实现进程后台运行，并不伴随终端关闭，而被杀死

​	首先我们可以得知，后台进程（守护进程）可以实现该目的，其原理如下：

​		由于只有在同一个会话下，才能进行Sinhup信号的传递，因此当修改该进程的sid时，就无法被终端控制，继而变为守护进程，只有主机关闭，才被杀死；因此就是使用了setsid来实现

**注意：**

- 对于长期使用的进程，推荐将其转换为服务，交给systemd管理

- 一段时间允许的进程，使用后台启动	
- 对于周期性执行进程，则使用cron执行启动命令

对于非后台进程，可以通过nohup指令实现:

```shell
nohup	command		&	
```

其中，&表示以后台方式启动，nohup只是忽略了终端传递的SIGHUP信号，导致不会伴随终端被杀死，对于其他终端信号：stdout和stderr，这些信号信息会保存到nohup.out文件中，如果进程运行时间非常长，则该文件会浪费很大的磁盘空间，一般情况下，可以对stdout和stderr输出进行重定向：

```shell
nohup  	command     >/dev/null	2>&1    &
```

将标准输出输入到 /dev/null	将错误输出 输入到 标准输出，从而将所有输出信息写入一个不存在的设备，即不进行输出

#  4、软件包管理

## 1、RPM

​	RPM，即RedHat Package Manager,红帽软件包管理工具，用于互联网下载包的打包和安装工具，生成或执行.RPM拓展文件。被广泛用于Linux发行版本中，已成为行业标准

该工具的常用指令：

### 1、查看（-q query）

- rpm	-qa	查看系统中已安装的所有RPM包（一般情况下，会使用|grep 过滤）

查询信息会显示RPM包的包名、版本和适合的系统版本,如：

```shell
docker-ce-cli-20.10.5-3.el7.x86_64
```

- rpm          -q	软件包名	查询系统是否安装该软件
- rpm          -qi       软件包名         查询系统是否安装该软件，如果安装则显示具体的安装信息和软件包信息
- rpm           -ql        文件名           查看当前文件属于哪个RPM包的文件

### 2、卸载（-e erase）

- rpm	-e	软件包名	卸载指定软件包（如果当前软件包依赖于其他时，会报错）
- rpm         -e       --nodeps      软件包         忽略依赖关系，强制卸载（一般不会使用）

### 3、安装（-ivh）

- rpm	-ivh	软件包路径	install安装、verboset提示、hash进度

## 2、YUM

​	yum（ Yellow dog Updater, Modified），是一个Shell前端软件包管理器（前端即支持图形化界面），基于RPM包管理工具之上，从指定的服务器自动下载RPM包并且安装，同时自动处理RPM包的依赖关系，保证一次性安装所有依赖的RPM包

### 1、yum基本指令

- yum	list	从yum服务器中，查询所有RPM包（一般情况下，搭配|grep过滤）
- yum       search       RPM包         和list类似，不需要使用|grep过滤
- yum        install        RPM包           下载安装指定的RPM包
- yum         list        installed      获取已安装的所有RPM包（一般情况下，搭配|grep过滤）
- yum         remove        RPM包          卸载指定RPM包

### 2、yum实际开发场景中的配置

1、对yum进行更新，保证能获取到最新的RPM包

```shell
yum  update
```

2、安装yum-utils，提供添加yum服务器镜像源功能(yum-config-manager)

```shell
yum install -y yum-utils

yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo           //添加aliyun的yum服务器镜像
```

# 5、开发环境搭建

**Linux进行安装软件有两种方式：**

- 使用yum，下载对应RPM包

  - 优点：安装快捷方便，不需要进行额外配置
  - 缺点：只能选择yum服务器上现有的最新版本

- 在官网下载对应版本的.tar.gz 压缩包，进行解压和运行环境配置

  - 优点：可以随意选择指定版本

  - 缺点：需要手动解压、配置

## 1、JAVAEE开发环境

### 1、JDK1.8

- yum安装：

  使用YUM按照JDK开发环境，即**java-1.8.0-openjdk-devel.x86_64**,他会以来下其他rpm包：

  - **java-1.8.0-openjdk-headless.x86_64**	用于支持headless模式，通过headless工具包，实现对图像、文本、声音的操作
  -  **java-1.8.0-openjdk .x86_64**              JDK运行环境，即JRE
  - **copy-jdk-configs**                   JDK相关配置

  安装后，默认保存在**/usr/lib/jvm**下，可以直接使用（java、javac、java -version）

  ```java
  yum search java-1.8
      
  yum install java-1.8.0-openjdk-devel.x86_64
  ```

- .tar.gz 压缩包安装：

  配置Linux环境变量，修改**/etc/profile文件**，添加如下环境变量：

  ```java
  export JAVA_HOME=/opt/jdk1.8            //为jdk软件压缩包解压后的文件目录
  export PATH=$PATH:$JAVA_HOME/bin	//配置path,$用于引入之前的path，：为分隔符，在后面追加JDK的path	
  ```


### 2、Tomcat

​	tomcat根据所支持的servlet、jdk版本，会有多种选择，一般情况下都会在官网下tar.gz包：

​	**tomcat9，支持JDK8即以上、servlet3.1；**

- 启动tomcat：

  进入到bin目录下，执行startup.sh脚本

  ```java
  ./ startup.sh  在当前目录下，搜索执行指定脚本，如果不指定目录，则默认在环境变量Path下查找
  ```

  此时就可以使用 ip：8080访问tomcat页面（注意需要开放端口）

### 3、Mysql

​	**mysql使用docker进行安装**

由于各种软件包安装都需要进行配置，因此通过使用docker容器化技术，能够有效减少这一部分的工作，并且方便软件管理（更适合应用服务部署）

# 6、shell脚本

**shell：**命令行解释器，提供给用户进行Linux内核的操作；既是命令语言，又是一个程序设计语言，作为命令语言，它可以交互式解释和执行用户输入的命令；作为程序设计语言，它可以定义各种变量、函数、参数和流程控制，实现脚本式的程序调用

## 1、shell脚本创建和执行

​	创建一个简单的shell脚本，文件后缀为sh（**所有文件进行创建后，都没有执行权限**）：

**熟悉格式注意：**

- shell脚本以 **#！/bin/bash**开头，用于指定该shell脚本使用哪种shell版本（因此是可以改变的，但一般都是/bin/bash）
- shell脚本每条语句通过换行分割，不需要使用分号

```shell
#！/bin/bash 
echo helloworld
```

### 1、shell注释

- 单行注释	

```shell
#注释
```

- 多行注释

```shell
:<<!
注释1
注释2
!
```

### 2、shell脚本执行：

提供两种方式：

1. 将shell脚本修改为可执行文件（即赋予文件所有者的执行权限）,此时文件颜色为绿色，然后**使用绝对路径运行文件**

   ```shell
   chmod 755 helloworld.sh    
   
   ./helloworld.sh						./代表当前文件夹路径，本质上也是绝对路径
   /home/java/helloworld.sh			或直接手写绝对路径
   ```

2. 使用sh命令，强制执行该文件（不推荐）,也必须指定绝对路径

   ```java
   sh  ./helloworld.sh        
   ```

**以后台方式运行shell脚本：**

```shell
./helloworld.sh   &              需要保证该文件有可执行权限
```

## 2、shell变量

​	Linux Shell变量分为四种：

- 系统变量：

  ​	通过 **$变量名**   进行引用，如$HOME、$PWD

  ​	通过 **set指令**，可以查看当前Linux所有系统变量

- 用户自定义变量：

  ​	使用在指定的shell脚本中，类似于局部变量：

- 位置参数变量：

  ​	用于获取shell脚本执行命令的参数，类似于函数调用时的入参

- 预定义变量

  ​	是在shell语法设计上，就预先定义好的关键字，代表指定信息

### 1、自定义变量的定义

```java
A=100              //定义变量A为100
echo A=$A          //打印
unset	A          //删除变量A
readonly	A=100  //定义静态变量A为100，静态变量不能重新赋值或删除，否则会报错无法找到A
```

注意：

- 当局部变量和系统变量重名时，优先加载局部变量（就近原则）
- 但进行变量卸载时，如果该变量和系统变量重名，则也无法获取系统变量的值（因此要避免自定义变量名和当前脚本中需要使用的系统变量名重复）
- 进行变量定义时，**=两边不能有空格**
- 变量名，推荐使用大写（因此Linux命令一般都是小写，又便于区分）

**命令结果赋值：**

```java
A=`ls -l`
A=$(ls -l)                   //两种方式作用一致
echo $A                     //此时会打印当前目录下的文件列表信息
```

### 2、系统环境变量设置

​	Linux系统环境变量通过 **/etc/profile**文件，来进行设置

- 基本语法：

  ​	export 变量名=变量值             创建环境变量

  ​	source    /etc/profile		使修改后的配置文件生效

  ​	echo    $变量名			打印环境变量值

  ​	set						查询所有环境变量

### 3、位置参数变量

**作用：**

​	**在进行shell脚本编写时，可能希望能获取执行该脚本的外部参数，实现让执行者能够在脚本外面设置入参，来决定整个脚本的执行逻辑**

- 基本语法：以 ./myshell.sh 100 200 为例

  - $n：通过n来指定获取shell脚本执行命令中，第n个参数值。注意：

    ​	1、第一位数以上时，需要将数字用括号括起来，如$（10）

    ​	2、$0,表示命令行本身，如./myshell.sh

  - $*：获取所有参数，当作一个整体，即 100 200

  - $@：获取所有参数，和$*打印一致，但可以作为数组

  - $#：统计参数个数，即2

### 4、预定义变量

​	shell设计者，方便shell编程，事先定义好的变量：

| 变量名 | 值                                                       |
| ------ | -------------------------------------------------------- |
| $$     | 当前进程号（PID），即执行当前shell脚本的进程             |
| $!     | **后台运行的**最后一个进程的进程号                       |
| $?     | 最后一个命令执行后的返回状态，0代表执行成功，非0代表失败 |

## 3、shell表达式的赋值

基本语法：

一共有三种方式，进行表达式的赋值

```shell
A=$(((2+3)*4))       注意最外层有连续两个括号，第一个表示将表达式作为一个整体；第二个搭配$使用，将其赋值到变量上/或者用于打印
A=$[(2+3)*4]		使用中括号代替两个小括号（推荐使用）
A=`expr 2 + 3`		使用expr输出表达式的值，赋值到变量中,注意每个字符间需要空格分隔，部分操作符需要反斜杠作为转义符，如\* 乘
```

## 4、变量和字面量的搭配使用

在一个表达式中，如果变量和字面量需要连接使用时，那么需要使用{}来定义变量名的范围，防止编译器无法识别

```java
date=$(date '+%Y-%m-%d %H:%M:%S')
echo ${date}开始备份
```

因此，一般情况下，在表达式中使用变量时，用${}引入

## 5、shell逻辑判断

- if语句：

```shell
if [ condition ]			condition，[]必须使用空格分隔
then					成立，则执行cmd1
     cmd1	
else
	 cmd2				否则，则执行cmd2
fi						和if组合，表示一个完整的if语句
```

- 常用判断条件关键字

  **注意：判000断条件的关键字两端需要有空格**

  - 比较符

    | 条件 | 作用                       |
    | ---- | -------------------------- |
    | -lt  | less than小于              |
    | -le  | less equal than小于等于    |
    | -gt  | geater than 大于           |
    | -ge  | geater equal than 大于等于 |
    | -ne  | not equal  不等于          |
    | =    | 等于                       |

    ```shell
    if [ 3 -lt 4 ]
    then
    echo 大于
    else
    echo 小于等于
    fi
    ```

  - 文件权限判断

    | 条件 | 作用                   |
    | ---- | ---------------------- |
    | -r   | 当前用户是否有读的权限 |
    | -w   | 当前用户是否有写的权限 |
    | -x   | 当前用户是否有执行权限 |

    ```shell
    if [ -r ./myshell.sh ]
    then
    echo 有读的权限
    else
    echo 没有读的权限
    fi
    ```

  - 文件类型判断

    | 条件 | 作用                   |
    | ---- | ---------------------- |
    | -e   | 文件存在               |
    | -f   | 文件存在，并为常规文件 |
    | -d   | 文件存在，并为目录文件 |

    ```shell
    if [ -f ./myshell.sh ]
    then
    echo 该文件存在
    fi
    ```

- if-else-if语句

  ```java
  if [ condition1 ]
  then
       cmd1					如果condition1成立，则执行cmd1
  elif [ condition2 ]			如果condition1不成立、condition2成立，则执行cmd2 
  then
       cmd2
  else						如果condition1、condition2都不成立，则执行cmd3
       cmd3
  fi
  ```

- case语句

  ```shell
  case  $变量名  in
  value1 )			变量值满足value1时，执行cmd1
  	cmd1		
  ;;					；；表示结束break
  value1 )
  	cmd2			变量值满足value1时，执行cmd1
  ;;
  * )
  	cmd3			变量值为其他时，执行cmd3
  ;;
  esac
  
  ```

- for循环语句

  - 方式一，结果集/数组遍历

  ```shell
  for C in 1 2 3 4       in后面可以多个值，使用空格分隔，也可以是一个结果集/数组遍历变量，如$@/$#
  do
          echo $C
  done
  ```

  - 方式二，递增、递减遍历

  ```shell
  SUM=0
  for((i=1;i<=100;i++))    for中，两个括号相邻，即 for（表达式）
  do
          SUM=$(($SUM+$i))
  done
  echo $SUM
  ```

- while循环

  ```shell
  while [ condition ]		如果condition成立，则执行cmd，循环重复
  do
  	CMD					
  done		
  ```

## 6、控制台输入输出

- 输出：

  ​	使用**echo指令**，来完成信息的输出，打印到控制台

- 输入：

  ​	使用**read指令**，来读取控制台的输入，并在shell脚本中，使用变量接受

**read指令：**

read	【选项】	变量名

-p		指定读取值的提示符

-t		指定程序等待控制台输入的时间（秒）,完成一次输入后，不再等待;**不指定时，一直等待**

```java
read -p 请输入一个数：	NUM
echo  $NUM
```

## 7、shell函数

​	shell和其他编程语言一样，可以定义函数，分为系统函数和自定义函数；并且shell函数本身，也是命令

### 1、系统函数

- **basename命令/函数**	

  basename	【String】	【suffix】	删除String中，最后一个/字符前的字符串，用于路径文件名截取；suffix可以选择性指定，对截取后的字符串进行后缀匹配

  ```shell
  basename /home/java/test.txt			结果为test.txt
  basenaem /home/java/test.txt	xt		结果为test.t  		
  ```

- dirname	【String】		删除String中，最后一个/字符后的字符串，用于路径目录截取

  ```shell
  dirname	/home/java/test.txt			结果为 /home/java
  ```

### 2、自定义函数

- shell函数基本语法：

  ```shell
  function  funcName(){		shell函数中，不需要定义形参，只需要保证函数中的实参在脚本中被定义
  	cmd    
  	return  变量/表达式		return可以省略，则使用最后一条命令运行结果作为返回值
  }
  
  A=[ funcName 实参1  实参2 ]   []和里面的内容，要用空格分隔
  ```

  计算两个数的和

  ```shell
  function sum(){						定义sum函数，无返回值
          SUM=$[$n1+$n2]
          echo $SUM
          return	$SUM		
  }
  
  read -p 请输入第一个数 n1				
  read -p 请输入第二个数 n2
  A=[ sum $n1 $n2 ]							调用sum函数，保证当前传递的实参变量与函数中的参数对应
  
  ```

## 8、shell脚本案例使用：

需求、流程：

1、在指定时间备份数据库数据（docker 容器中的，mysql）到指定文件夹

2、创建日志文件，记录备份开始时间和结束时间

3、将备份好的文件以时间命名，并打包为.tar.gz

4、备份成功后，检查所有备份文件，删除十天前的

前提：

- mysql备份数据库语句：

  ```java
  mysqldump -uroot	-p123456 --host=localhost  >  mysql.sql
  ```

- 进入docker执行命令，将获取的数据写入容器外部文件

  ```java
  docker exec -i mysql mysqldump -uroot -p123456 myProject > home/data/mysql.sql
  ```

  **注意：写入的文件地址，为宿主机的，即先执行>前面的语句，然后将结果写入文件**

完整流程：

```java
mysql_container=mysql
mysql_user=root
mysql_password=123456
dbname=myProject
backtime=$(date '+date %Y-%m-%d %H:%M:%S')
logpath='/home/java/log'
datapath='/home/java/data'


if [ ! -d $logpath ]
then
mkdir -p $logpath
fi

if [ ! -d $datapath ]
then
mkdir -p $datapath
fi


echo 备份时间为$backtime,备份数据库为$dbname >>  $logpath/mysqllog.log

for table in $dbname
do
result=$(docker exec -i ${mysql_container} mysqldump -u${mysql_user} -p${mysql_password} ${table} > ${datapath}/mysql.sql)
if [ $? != 0 ]
then
echo 备份失败cho $log >> ./testLog.log
fi
done
```

# 7、系统调优

# 8、深入理解Linux内核

**1、指令别名**

**2、ctrlC\ctrlD\ctrlV\ctrlZ\table**