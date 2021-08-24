# Oracle数据库：

## mysql和oracle的区别

1、在select子句中，聚合函数不能和其他非分组字段同时存在

2、oracle需要使用序列，来实现字段自动增长

3、oracle分页处理更加繁琐（oracle 12c得到改善）

4、

6.日期字段的处理

MYSQL日期字段 分DATE和TIME两种，ORACLE日期字段只有DATE，包含年月日时分秒信息，用当前数据库的系统时间为 SYSDATE，精确到秒，或者用字符串转换成日期型函数TO_DATE(‘2001-08-01’,’YYYY-MM-DD’)年-月-日24小时:分 钟:秒的格式YYYY-MM-DD HH24:MI:SS TO_DATE()还有很多种日期格式，可以参看ORACLE DOC.日期型字段转换成字符串函数TO_CHAR(‘2001-08-01’,’YYYY-MM-DD HH24:MI:SS’)

日期字 段的数学运算公式有很大的不同。MYSQL找到离当前时间7天用DATE_FIELD_NAME > SUBDATE(NOW()，INTERVAL 7 DAY)ORACLE找到离当前时间7天用 DATE_FIELD_NAME >SYSDATE - 7;

MYSQL中插入当前时间的几个函数是：NOW()函数以`'YYYY-MM-DD HH:MM:SS'返回当前的日期时间，可以直接存到DATETIME字段中。CURDATE()以’YYYY-MM-DD’的格式返回今天的日期，可以 直接存到DATE字段中。CURTIME()以’HH:MM:SS’的格式返回当前的时间，可以直接存到TIME字段中。例：insert into tablename (fieldname) values (now())

而oracle中当前时间是sysdate

7.空字符的处理

MYSQL的非空字段也有空的内容，ORACLE里定义了非空字段就不容许有空的内容。按MYSQL的NOT NULL来定义ORACLE表结构，导数据的时候会产生错误。因此导数据时要对空字符进行判断，如果为NULL或空字符，需要把它改成一个空格的字符串。

8.字符串的模糊比较

MYSQL里用字段名like%‘字符串%’，ORACLE里也可以用字段名like%‘字符串%’但这种方法不能使用索引，速度不快，用字符串比较函数instr(字段名，‘字符串’)>0会得到更精确的查找结果。

9.程序和函数里，操作数据库的工作完成后请注意结果集和指针的释放。

## Oracle12c和Oracle11g

## sql语句分类

SQL(Structured Query Language) 是结构化查询语言的缩写。

SQL是在关系数据库上执行数据操作、检索及维护所使用的标准语言,可以用来查询数据，操纵数据，定义数据，控制数据，所有数据库都使用相同或者相似的语言。

SQL可分为:

数据定义语言（DDL） : Data Definition Language
数据操纵语言（DML） : Data Manipulation Language
事务控制语言（TCL）：Transaction Control Language)
数据查询语言（DQL）：Data Query Language
数据控制语言（DCL） : Data Control Language
执行SQL语句时，用户只需要知道其逻辑含义，而不需要知道SQL语句的具体执行步骤

##### 4.2.1. 数据定义语言（DDL）

用于建立、修改、删除数据库对象，包括创建语句(CREATE)、修改语句(ALTER)、删除语句(DROP)，比如使用CREATE TABLE创建表，使用ALTER TABLE修改表，使用DROPTABLE删除表等动作。这类语言不需要事务的参与，自动提交。

##### 4.2.2. 数据操作语言（DML）

用于改变数据库数据，包括INSERT、UPDATE、DELETE三条语句。其中，INSERT语句用于将数据插入到数据库中，UPDATE语句用于更新数据库中已存在的数据，DELETE用于删除数据库中已存在的数据。DML语言和事务是相关的，执行完DML操作后必须经过事务控制语句提交后才真正的将改变应用到数据库中。

##### 4.2.3. 事务控制语言（TCL）

用来维护数据一致性的语句，包括提交(COMMIT)、回滚(ROLLBACK)、保存点(SAVEPOINT)三条语句，其中COMMIT用来确认已经进行的数据库改变， ROLLBACK语句用来取消已经进行的数据库改变，当执行DML操作后(也就是上面说的增加、修改、删除等动作)，可以使用COMMIT语句来确认这种改变，或者使用ROLLBACK取消这种改变。SAVEPOINT语句用来设置保存点，使当前的事务可以回退到指定的保存点，便于取消部分改变。

##### 4.2.4. 数据查询语言（DQL）

用来查询所需要的数据。使用最广泛，语法灵活复杂。

##### 4.2.5. 数据控制语言（DCL）

用于执行权限的授予和收回操作、创建用户等，包括授予(GRANT)语句，收回(REVOKE)语句，CREATE USER语句，其中GRANT用于给用户或角色授予权限， REVOKE用于收回用户或角色已有的权限。DCL语句也不需要事务的参与，是自动提交的。

## Oracle启动服务：

默认oracle安装后会有7个服务：

**1、OracleServiceORCL：**

数据库服务（数据库实例），是Oracle数据库核心服务，是数据库启动基础

*****ORCL是数据库实例名，默认的数据库是ORCL，你可以创建其他的，即OracleService+数据库名。

**2、OracleOraDb11g_home1TNSListener:**

监听器服务，在oracle数据库需要远程访问时需要启动（本地访问（sqlpuls）不需要启动，但本地使用工具进行连接时，还是需要启动，在oracle9i版本后，默认包含XDB特性，当oracle监听服务启动时，XDB的Http服务会默认占用8080端口）

解决方式：

​	1、禁用XDB服务：

```
在ORACLE_HOME/dbs/initSID.ora文件，去除如下行：
dispatchers='(PROTOCOL=TCP) (SERVICE=XDB)'
```

**3、OracleOraDb11g_home1ClrAgent：**

Oracle数据库的.NET扩展服务的一部分（非必须启动）

**4、OracleMTSRecoveryService：**

服务端控制服务，作用是允许数据库充当一个微软事务服务器MTS、COM/COM+对象和分布式环境下的事务的资源管理器（非必需启动）

**5、OracleDBConsoleorcl：**

Oracle数据库控制台服务，在运行Enterprise Manager（企业管理器OEM）的时候，需要启动这个服务。（非必须启动）

**6、Oracle ORCL VSS Writer Service：**
Oracle卷映射拷贝写入服务，它可以在多卷或者单个卷上创建映射拷贝，同时不会影响到系统的系统能。（非必须启动）

**7、OraclJobSchedulerORCL:**
Oracle作业调度（定时器）服务，ORCL是Oracle实例标识。（非必须启动）



## Oracle数据库体系结构：

实例（Instance）：一组Oracle后台进程以及在服务器中分配的共享内存区域

数据库（Database）：是由磁盘的数据文件、控制文件、日志文件、参数文件和归档日志文件等组成的物理文件集合

数据库服务器（Database Server）:管理数据库的软件工具（如sqlPLUS）

**其关系：**

实例用于管理控制数据库，数据库为实例提供数据

一个数据库可以被多个实例装载使用

一个实例在生命周期中只能装置打开一个数据库

#### orcale数据储存结构：

从逻辑上：一个数据库（database）下面可以分为多个表空间（tablespace）；一个表空间下面又分为多个段（segment）；一个段又分为多个区间（extent）；一个区间又有一组连续的数据块（data block）组成

**段：一个数据表占一个段、一个索引占一个段，段是指占用数据文件空间的通称，或数据库对象使用的空间的集合；段可以有表段、索引段、回滚段、临时段和高速缓存段等。**

**区：一个区中的数据块是连续的，但在物理磁盘上可能是分散的，分配给对象（如表）的任何连续块叫区间；区间也叫扩展，因为当它用完已经分配的区间后，再有新的记录插入就必须在分配新的区间（即扩展一些块）；一旦区间分配给某个对象（表、索引及簇），则该区间就不能再分配给其它的对象.**

从物理上：一个表空间由多个数据文件构成，数据文件是实实在在存在的磁盘上的文件，这些数据文件由系统磁盘的块组成

#### 表空间对区间的管理：

**词典管理表空间**(Dictionary-managed tablespaces)

​    在表空间里，有的区间被占用了，有的没被占用，这些数据是放在数据字典里的。当你对这个表空间进行分配或释放的时候，数据文件里相关的表就会做修改

 

**本地管理表空间**（locally managed tablespace）

​      本地管理表空间不是在数据词典里存储表空间的，由自由区管理的表空间。用位图来自由的管理区间。一个区间对一个位，如果这个位是1表示已经被占用，0表示未被占用

　　词典管理空间表示“中央集权治”，本地管理表空间表示“省市自治区”，一个databases表示中国，tablespaces表示一个省或直辖市。词典管理统一由中央调配。而本地管理表示有高度的自治权利，自已各种资源的分配不用上报中央。

#### 表空间类型：

**Undo  tablespace**

　　Undo 类型的表空间（还原表空间），当你对一张表或一条记录进行修改的时候，它会对修改之前的信息进行保存，这样可以保证数据的回滚。Undo 只包含undo类型的对象，不能包含任何其他对象，只适合于数据文件和区间管理。

创建undo 类型的表空间：

 SQL>create undo tablespace  undo1 datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m;

**Temporary  Tablespaces**

 　　临时表空间，相当于一个临时的垃圾场。用于排序操作，比如你要做一次大数据量的查询，但在内存无法存储这么大量的数据，然后会在磁盘上建立一个临时的表空间用记存放这些数据。Oracle就会用这个临时表空间做排序，存储中间结果。

一个全局的临时表空间，可以由多个用户共享，谁需要谁使用。但它只能存放临时的数据，不能包含任何永久性对象。 建议用本地管理方式创建这个表空间。

创建临时表空间：

 SQL>create temporary tablespace  temp datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m  extent management local uniform size 4m;

**Tablespaces**

​         用户数据表空间， 最重要,也是用于存放用户数据表空间

SQL>create undo tablespace  tablespace datafile '/ora10/product/oradata/ora10/paul01.dbf' size 20m;

#### 表空间的创建：

create  tablespace(表空间类型)   tablespacename（表空间名）  datafile  '/ora10/product/oradata/ora10/paul01.dbf'（表空间数据文件及路径） size 20m（数据文件最大值，即表空间大小）

*****表空间数据文件路径一定要是本来存在的，否则部分创建

除了最基本的属性外，还可以设置其他属性：

表空间的区大小   uniform  size  128k

表空间的数据文件大小自动增长（并可以设置最大值） autoextend on next 10m maxsize 500m

**这些属性出来在创建表空间时添加外，还可以之后通过修改表空间属性来设置：**

如：alter tablespace sp01（表空间名） 'D:\dev\oracle\product\10.2.0\dada01.dbf' autoextend on next 10m maxsize 500m;

#### 表空间状态：

当建立表空间时，表空间处于联机的(online)状态，此时该表空间是可以访问的，并且该表空间是可以读写的，即可以查询该表空间的数据，而且还可以在表空间执行各种语句。但是在进行系统维护或是数据维护时，可能需要改变表空间的状态。一般情况下，由特权用户或是dba来操作。

1)、使表空间脱机
alter tablespace 表空间名 offline;
eg、alter tablespace data01 offline;--表空间名不能加单引号
2)、使表空间联机
alter tablespace 表空间名 online;
eg、alter tablespace data01 online;
3)、只读表空间
当建立表空间时，表空间可以读写，如果不希望在该表空间上执行update，delete，insert操作，那么可以将表空间修改为只读
alter tablespace 表空间名 read only;

4)、可读写表空间

alter tablespace 表空间名 read write;

**只读或可读写状态时，表空间一定也为online**

#### 删除表空间：

一般情况下，由特权用户或是dba来操作，如果是其它用户操作，那么要求用户具有drop tablespace 系统权限。
drop tablespace ‘表空间’ including contents and datafiles;

**including contents表示删除表空间时，删除该空间的所有数据库对象，而datafiles表示将数据库文件也删除。**

#### 扩展表空间：

表空间是由数据文件组成的，表空间的大小实际上就是数据文件相加后的大小。

因此扩展表空间，就是扩展表空间的数据文件，有三种方式：

1、设置表空间数据文件自动增长

```
SQL> alter tablespace sp01 'D:\dev\oracle\product\10.2.0\dada01.dbf' autoextend on next 10m maxsize 500m;
```

2、修改表空间数据文件的大小

```
SQL> alter tablespace sp01 'D:\dev\oracle\product\10.2.0\dada01.dbf' resize 4m
```

3、添加表空间的数据文件

```
SQL> alter tablespace sp01 add datafile 'D:\dev\oracle\product\10.2.0\dada02.dbf' size 1m;
```

#### 表空间数据文件移动

有时，如果你的数据文件所在的磁盘损坏时，该数据文件将不能再使用，为了能够重新使用，需要将这些文件的副本移动到其它的磁盘，然后恢复。

1. 确定数据文件所在的表空间

```
select tablespace_name from dba_data_files where file_name=upper('D:\dev\oracle\product\10.2.0\dada01.dbf');
```

2. 使表空间脱机

--确保数据文件的一致性，将表空间转变为offline的状态。

alter tablespace sp01 offline;

3. 使用命令移动数据文件到指定的目标位置

host move D:\dev\oracle\product\10.2.0\dada01.dbf （要移动的文件）    c:\dada01.dbf（移动后的文件）

4. 执行alter tablespace 命令
    在物理上移动了数据后，还必须执行alter tablespace命令对数据库文件进行逻辑修改：
    alter tablespace sp01 rename datafile 'D:\dev\oracle\product\10.2.0\dada01.dbf' to 'c:\sp01.dbf';
5. 使得表空间联机
    在移动了数据文件后，为了使用户可以访问该表空间，必须将其转变为online状态。
    alter tablespace sp01 online;

#### 在表空间中创建表

create table mypart(
   deptno number(4), 
   dname varchar2(14), 
   loc varchar2(13)
) tablespace sp01;



## Oracle-DBA常用操作

### 1、视图：

视图是一张虚拟表，其内容由查询定义，同真实的表一样，视图包含一系列带有名称的列和行数据。但是，视图并不在数据库中以存储的数据值集形式存在。行和列数据来自由定义视图的查询所引用的表，并且在引用视图时动态生成。(视图不是真实存在磁盘上的)

二、视图与表的区别
1、表需要占用磁盘空间，视图不需要
2、视图不能添加索引(所以查询速度略微慢点)
3、使用视图可以简化，复杂查询
4、视图的使用利于提高安全性
比如：不同用户查看不同视图

三、创建/修改视图
1、创建视图
create view 视图名 as select 语句 [with read only]
2、创建或修改视图
create or replace view 视图名 as select 语句 [with read only]
3、删除视图
drop view 视图名



### 5、权限角色

#### 1、系统权限

1)、**系统权限是指执行特定类型sql命令的权利。它用于控制用户可以执行的一个或是一组数据库操作。**比如当用户具有create table权限时，可以在其方案中建表，当用户具有create any table权限时，可以在任何方案中建表。oracle提供了100多种系统权限。
常用的有：
create session 连接数据库 
create table 建表
create view 建视图 
create public synonym 建同义词
create procedure 建过程、函数、包 
create trigger 建触发器
create cluster 建簇

2、显示系统权限
oracle提供了100多种系统权限，而且oracle的版本越高，提供的系统权限就越多，我们可以查询数据字典视图system_privilege_map，可以显示所有系统权限。
select * from system_privilege_map order by name;

3、授予系统权限

一般情况，授予系统权限是由dba完成的，如果用其他用户来授予系统权限，则要求该用户必须具有grant any privilege的系统权限。

4、回收系统权限
一般情况下，回收系统权限是dba来完成的，如果其它的用户来回收系统权限，要求该用户必须具有相应系统权限及转授系统权限的选项(with admin option)。

#### 2、对象权限

**指访问其它方案对象的权利，用户可以直接访问自己方案的对象，但是如果要访问别的方案的对象，则必须具有对象的权限。**
比如smith用户要访问scott.emp表(scott：方案，emp：表)
常用的有：
insert 添加
delete 删除 
alter 修改 
select 查询 
index 索引 
references 引用 
execute 执行

2)、显示对象权限
通过数据字段视图可以显示用户或是角色所具有的对象权限。视图为dba_tab_privs
SQL> conn system/manager;
SQL> select distinct privilege from dba_tab_privs;
SQL> select grantor, owner, table_name, privilege from dba_tab_privs where grantee = 'BLAKE';

3)、授予对象权限
在oracle9i前，授予对象权限是由对象的所有者来完成的，如果用其它的用户来操作，则需要用户具有相应的(with grant option)权限，从oracle9i 开始，dba用户(sys，system)可以将任何对象上的对象权限授予其它用户。授予对象权限是用grant 命令来完成的。对象权限可以授予用户，角色，和public。在授予权限时，如果带有with grantoption 选项，则可以将该权限转授给其它用户。但是要注意with grant option选项不能被授予角色。
1.monkey 用户要操作scott.emp 表，则必须授予相应的对象权限
1). 希望monkey可以查询scott.emp 表的数据，怎样操作？
grant select on emp to monkey;
2). 希望monkey可以修改scott.emp 的表数据，怎样操作？
grant update on emp to monkey;
3). 希望monkey可以删除scott.emp 的表数据，怎样操作？
grant delete on emp to monkey;
4). 有没有更加简单的方法，一次把所有权限赋给monkey？
grant all on emp to monkey;

2.能否对monkey访问权限更加精细控制。(授予列权限)
1). 希望monkey只可以修改scott.emp的表的sal字段，怎样操作？
grant update on emp(sal) to monkey
2).希望monkey 只可以查询scott.emp 的表的ename，sal 数据，怎样操作？
grant select on emp(ename,sal) to monkey

3.授予alter权限
如果black用户要修改scott.emp表的结构，则必须授予alter对象权限
SQL> conn scott/tiger
SQL> grant alter on emp to blake;
当然也可以用system，sys 来完成这件事。
4.授予execute权限
如果用户想要执行其它方案的包/过程/函数，则须有execute权限。
比如为了让ken可以执行包dbms_transaction，可以授予execute 权限。
SQL> conn system/manager
SQL> grant execute on dbms_transaction to ken;
5.授予index权限
如果想在别的方案的表上建立索引，则必须具有index 对象权限。
如果为了让black 可以在scott.emp 表上建立索引，就给其index 的对象权限
SQL> conn scott/tiger
SQL> grant index on scott.emp to blake;
6.使用with grant option 选项
该选项用于转授对象权限。但是该选项只能被授予用户，而不能授予角色
SQL> conn scott/tiger;
SQL> grant select on emp to blake with grant option;
SQL> conn black/shunping
SQL> grant select on scott.emp to jones;

#### 3、角色

**角色。角色是一组权限的集合，将角色赋给一个用户，这个用户就拥有了这个角色中的所有权限。不同的角色有不同的权限，分为预定义角色和自定义角色**

**预定义角色**：是指oracle所提供的角色，每种角色都用于执行一些特定的管理任务，下面我们介绍常用的预定义角色connect、resource、dba。

1)、connect角色
connect角色具有一般应用开发人员需要的大部分权限，当建立了一个用户后，多数情况下，只要给用户授予connect和resource角色就够了，那么connect角色具有哪些系统权限呢？
create cluster
create database link
create session
alter session
create table
create view
create sequence

2)、resource角色
resource角色具有应用开发人员所需要的其它权限，比如建立存储过程，触发器等。这里需要注意的是resource角色隐含unlimited tablespace系统权限。
resource角色包含以下系统权限：
create cluster
create indextype
create table
create sequence
create type
create procedure
create trigger

3)、dba角色
dba角色具有所有的系统权限，及with admin option选项，默认的dba用户为sys和system，它们可以将任何系统权限授予其他用户。但是要注意的是dba角色不具备sysdba和sysoper的特权（启动和关闭数据库）。

1)、建立角色（不验证）
如果角色是公用的角色，可以采用不验证的方式建立角色。
create role 角色名 not identified;
2)、建立角色（数据库验证）
采用这样的方式时，角色名、口令存放在数据库中。当激活该角色时，必须提供口令。在建立这种角色时，需要为其提供口令。
create role 角色名 identified by 密码;

2、角色授权
1)、给角色授权
给角色授予权限和给用户授权没有太多区别，但是要注意，系统权限的unlimited tablespace和对象权限的with grant option选项是不能授予角色的。
SQL> conn system/oracle;
SQL> grant create session to 角色名 with admin option
SQL> conn [scott/oracle@orcl](mailto:scott/oracle@orcl);
SQL> grant select on scott.emp to 角色名;
SQL> grant insert, update, delete on scott.emp to 角色名;
通过上面的步骤，就给角色授权了。

2)、分配角色给某个用户
一般分配角色是由dba来完成的，如果要以其它用户身份分配角色，则要求用户必须具有grant any role的系统权限。
SQL> conn system/oracle;
SQL> grant 角色名 to blake with admin option;
因为我给了with admin option选项，所以，blake可以把system分配给它的角色分配给别的用户。

3、删除角色
使用drop role，一般是dba来执行，如果其它用户则要求该用户具有drop any role系统权限。
SQL> conn system/oracle;
SQL> drop role 角色名;
问题：如果角色被删除，那么被授予角色的用户是否还具有之前角色里的权限？
答案：不具有了

4、显示角色信息
1)、显示所有角色
SQL> select * from dba_roles;
2)、显示角色具有的系统权限
SQL> select privilege, admin_option from role_sys_privs where role='角色名';
3)、显示角色具有的对象权限
通过查询数据字典视图dba_tab_privs可以查看角色具有的对象权限或是列的权限。
4)、显示用户具有的角色，及默认角色
当以用户的身份连接到数据库时，oracle 会自动的激活默认的角色，通过查询数据字典视图dba_role_privs 可以显示某个用户具有的所有角色及当前默认的角色。
SQL> select granted_role, default_role from dba_role_privs where grantee = ‘用户名’;

五、精细访问控制
精细访问控制是指用户可以使用函数，策略实现更加细微的安全访问控制。如果使用精细访问控制，则当在客户端发出sql语句(select，insert，update，delete)时，oracle会自动在sql语句后追加谓词(where子句)，并执行新的sql语句，通过这样的控制，可以使得不同的数据库用户在访问相同表时，返回不同的数据信息，如：
用户 scott blake jones
策略 emp_access
数据库表 emp
如上图所示，通过策略emp_access，用户scott，black，jones在执行相同的sql语句时，可以返回不同的结果。
例如：当执行select ename from emp时，根据实际情况可以返回不同的结果。

**自定义角色**：顾名思义就是自己定义的角色，根据自己的需要来定义。一般是dba来建立，如果用别的用户来建立，则需要具有create role的系统权限。在建立角色时可以指定验证方式(不验证，数据库验证等)。



### 数据库基本使用：

1、Oracle安装默认自动生成sys用户和system用户

sqlPLUS常用命令：

#### 1、连接命令：

**登入数据库（可以在登入后继续使用，来切换不同用户）：**

conn   用户名（本地连接）      /        conn 密码@网络用户名 （远程连接）   

**显示该数据库的用户名 ：**

show   用户名      

**该命令用来断开与当前数据库的连接：**

disc/disconn/disconnect

**修改用户密码：**

passw   （修改当前用户密码）      password   用户名   （可以修改其他用户）

**清空sqlPLUS屏幕：**

clear screen

#### 2、文件操作命令

**运行sql脚本**

@/start    脚本路径

将sqlPLUS内容输出到指定文件

spool   文件路径

sql代码

spool   off



**sqlPLUS工具作用**：主要用于执行sql语句，pl\sql块

*****PL\SQL：过程化sql语句，是Oracle数据库对sql语句的扩展，在普通sql语句上增加使用了编程语言的特点，

通过一些逻辑语言（条件判断、循环等）来执行sql语句，实现复杂功能的数据库查询

*****Mysql不支持PL/SQL,但支持Navicat Premium

**PL\SQL和存储过程的区别：**

功能实现一样，但PL\SQL本质上还是sql语句，只能一次性使用；存储过程是将一系列语句保存在数据库中，可以重复调用



## Oralce数据类型：

### 1、字符类型

- char

  固定长度，不足向右补足空格，存储效率更高；对于数据长度不定的字段，浪费空间

  **最大长度**：2000字节

  **默认长度**：1字节

- nchar

  和char相同，区别是通过unicode字符集存储，长度单位为字符

  **最大长度**：2000字符

  **默认长度**：1字符

- VARCHAR、VARCHAR2

  可变长度，节省空间

  **VARCHAR最大长度为2000字节，VARCHAR2最大长度为4000字节**

- NVARCHAR2

  和VARCHAR2相同，区别是通过unicode字符集存储，长度单位为字符

  **最大长度：2000字符**

### 2、数值型

- NUMBER

  NUMBER范围-10的38次方到10的38次方，可以表示整数，也可以表示小数

  NUMBER（P,S），P表示有效位数，S表示小数有效位数，比如NUMBER（5,2）其范围为：-999.99 ~ 999.99；NUMBER(5)其范围为：99999 ~ -99999

  **默认P为38，S在0-38中取值**

  **P的取值范围为1-38,S为38-P**

### 3、日期、时间类型

- DATE

  年月日、时分秒

  当不指定时分秒时，默认为00:00:00

  **长度固定为7**

- TIMESTAMP

  是对DATE类型的扩展，支持小数秒

  其长度就是所支持小数秒的小数位，默认为6

### 4、大对象类型

| 大对象类型 | 描述            |
| ---------- | --------------- |
| BLOB       | 二进制数据      |
| CLOB       | 字符数据        |
| NCLOB      | Unicode字符数据 |

**这些大对象，支持最大10G，单位为KB**

## Oracle日常问题集合

### 1、pl/sql工具对表的属性设置：

**一般情况下，都是用默认值**

  %free 10 --块保留10%的空间留给更新该块数据使用 
  initrans 1 --初始化事务槽的个数
  maxtrans 255 --最大事务槽的个数

```
initial 64k --区段(extent)一次扩展64k
minextents 1 --最小区段数
maxextents unlimited --最大区段无限制 
```

### 2、Oracle客户端连接注意事项：

查询当前oracle服务端的字符集编码：

select * from nls_database_parameters where parameter='NLS_CHARACTERSET';

查询当前连接oracle客户端的字符集编码：

select   userenv('language')  from   dual;不配置环境变量，默认为 SIMPLIFIED CHINESE_CHINA.AL32UTF8

1、数据库连接工具进行oracle数据库连接时，需要使用instantclient（oracle即时客户端），并且配置该客户端使用的环境变量（保证所有路径都能执行该客户端）：

Path：   instantclient路径

**由于数据库连接工具都可以直接配置oracle客户端环境，因此不需要该配置**

2、为保证oracle客户端连接所使用的字符集编码与服务器数据库一致，应配置环境变量

NLS_LANG： oracle服务器对应的字符集编码

3、当使用TNS连接时，需要在instantclient安装目录下，创建Network\Admin目录（必须与oracle原客户端目录一致，否则连接工具无法检测到该文件），并添加配置文件tnsnames.ora；（**也可以通过配置TNS_ADMIN环境变量来指定tnsnames.ora文件的路径：只对plsqDeveloper有效**）

4、一定要保证instantclient版本和数据库连接工具的版本一致

1. plsqlDeveloper使用TNS作为连接方式，它会自动扫描本地oracle安装文件中的客户端配置文件；因此在进行oracl远程连接时，需要进行TNS连接配置，此时就会扫描配置好的tnsnames.ora；当然也可以直接使用正常连接：ip:端口/oracle服务名  （oracle8i之后，一般数据库远程连接，对外都使用服务名）

   **plsqDeveloper在help->support info，可以指定所有配置路径**

2. Navicat Premium 12，自带一个精简版instantclient_10_2，但该版本不支持GBK编码，只支持Unicode（AL32UTF8属于unicode编码）、ASCII以及西欧字符集；因此需要下载一个完整版的instantclient客户端

### 3、Oracle创建自增字段：

1、创建oracle序列

create sequence 序列名  
       increment by 1 --每次增加几个，我这里是每次增加1 
       start with 1   --从1开始计数 
       nomaxvalue      --不设置最大值 
       nocycle         --一直累加，不循环 
       nocache        --不建缓冲区 

2、创建触发器

CREATE TRIGGER ContestDB_trigger 
    BEFORE INSERT ON 表名
    FOR EACH ROW 
    WHEN (new.主键名 is null) --只有在tid为空时，启动该触发器生成tid号 
 begin 
    select 序列名.nextval into :new.主键名 from sys.dual; 
 end; 

怎样重置自增：

1、删除当前序列，然后重新创建

2、通过执行自增来调整当前序列值（操作比较麻烦）

### 4、Oracle分析函数，以及常用场景：

在Oracle8.16后，提供一种分析函数语法，用于计算基于一组数据为结果的聚合函数（计算参数为字段）：

1、非聚合函数返回的是一组数据；而聚合函数返回的是一行数据

2、分析函数需要搭配over（）开窗函数一起使用，over（）开窗函数：

其中包括三个分析子句：**分组(partition by), 排序(order by), 窗口(rows)**

​		**开窗函数是对于分析函数来说，就是对于除分析函数外的sql语句查询进行进一步数据处理，分组、排序、窗口；sql 语句中存在order by时例外，当sql语句中的orderby和窗口函数中的orderby相同时，可以看作为sql语句数据先进行了处理，然后分析函数在对数据进行处理；当两者不相同时，sql语句中的排序，会最后执行**

**窗口代表了分析函数处理当前数据的选择范围，默认为全部，根据是否使用partition by 语句，还有所不同：**

​	1、当窗口函数中出现分组子句（partition by ）时，属性对应于分组后，每组数据中的第一行、当前行、最后行

​	2、当窗口函数中没有分组子句时，属性对应于查询表中数据的第一行、当前行、最后行	

**窗口子句语法为：rows between xxx  and xxx**

常用属性有：

第一行是 unbounded preceding，

前一行：1 preceding

当前行是 current row，

最后一行是 unbounded following,

 后一行：1following

**窗口子句不能单独出现，必须有order by子句时才能出现；当没有窗口子句，但有order by 时，默认为当组中的第一行到当前行，因此需要使用全组数据时，要么省略order by，要么在order by基础上指定窗口**

3、常用分析函数有：

1、计算：sum（column）、avg（column）、max（column）、min（column）##它们本身也是聚合函数

2、排名：row_number()   、rank（）、dense_rank（）

row_number() 在出现相同数据时，按照数据出现前后排名，不产生重复值

rank（），在出现相同数据时，排名会相同，但之后出现不同数据的排名不再连续，而是根据前面数据总数进行排名

dense_rank(),在出现相同数据时，排名会相同,之后出现不同数据的排名会紧跟其后，连接

**排名默认需要使用全部数据，因此无法指定窗口函数中的窗口子句**

3、取值：first_value(column)、last_value()、lag（）、lead（）

**first_value(column)、last_value(column)：**

​		取开窗函数处理后的第一行/最后一行数据对应的字段值，当开窗函数进行分组时，会取每组中的第一个值；**一般情况下，我们取的第一个值都是通过排序得到的，使用了order by,并且我们需要保证排序的数据总是为当前全部数据，因此需要添加窗口子句（防止只有order by时，窗口默认为当组第一行到当前行数据）**

**lag（）、lead（）：**

一次查询中取出同一字段的前n行的数据和后n行的值；语法从理论上是省略一部分数据，从而获取需要的数据

lag（column，n，‘null’）：获取column字段，省略全部数据中的后n行数据，并且可以指定省略数据的代替值为null（当然也可以不指定，从而为null）

lead（column，n，‘null’）：获取column字段，省略全部数据中的前n行数据，并且指定省略数据代替值为null（当然也可以不指定，从而为null）

**该分析函数的处理前体是数据有一定的排序，因此必须使用order by子句，由于需要全部数据，因此不能指定rows窗口子句**

4、常用使用场景：

1、first_value(column)、last_value(column)，可以用于当前场景：

- 查询消费表中，所有人员消费的最大值(最小值)，追加在所有数据后：

````sql
select t.*,FIRST_value(name) over(partition by name order by value rows between unbounded preceding and unbounded following) max_value 
from Test t
````

- 查询当前月份的前一个月和后一个月的销售额

````java
select t.*,
first_value(password) over(order by id rows between 1 preceding and current row),
last_value(password) over (order by id rows between current row and 1 following)
from test t 
````

2、row_number()   、rank（）、dense_rank（），可以用于场景：查询消费表中，所有人员各最大消费的那条记录（rank（）、dense_rank(),会由于最大消费数有相同，而出现多条）

````sql
select * from 
(select t.* ,row_number() over(partition by name  order by password) mm from test t) where mm=1
````

3、sum（column）、avg（column）、max（column）、min（column），可以用于场景：

- 每月销售额记录表中，查询每月数据和今年一月到当前月的总销售额、平均销售额、最大销售额、最小销售额

````sql
select t.*,sum(month_value) over(order by month) from test t where time =trunc(SYSDATE)
````

- 每月销售额记录表中，今年所有月份的总销售额、平均销售额、最大销售额、最小销售额

````sql
select t.*,sum(month_value) over() from test t where time =trunc(SYSDATE) order by month
````

4、lag（）、lead（），可以用于场景：

- 查询当前月份的前一个月和后一个月的销售额（相对于first_value(),不会由于第一行和最后一行，而产生错误数据，而是用其他数据填充）

````sql
select t.*,
lag(value,1) over (order by month ),
lead(value,1) over (order by month)
from test t 
````

**lag(value ,1)与t.* 结合展示时，相当于是将value本来的排序数据，往下一行开始进行展示，并缺少最后一条数据**

### 5、Oracle DBlink

**用于在本地数据库，远程访问另外一个数据库中的数据，并进行增删改操作**

在实际开发过程中，会直接创建两个数据库的连接，然后进行操作访问；但有了DBlink后，就可以把两个不同数据库的表数据，交给oracl进行sql处理（数据库处理效率和便利性，远大于JAVA）

### 6、Oracle索引：

**索引分类：**

B树索引（普通索引）、位图索引、唯一索引、函数索引

- B数索引结构：

  每个节点保留索引关键值和执行下个子节点的指针；而最后一行则保存索引key值以及rowID（实际数据的物理存储地址）；并且最后一行时连续的，而索引本身有序，可以实现索引范围查找

- 唯一索引：

  和B数索引一样，只是所有节点的关键字没有重复值，因此速度更快（默认将主键使用唯一索引）

- 位图索引：

  根据数据对于每一个rowID提供一个位图值(BIT 值)，对应当前一个或多个字段的索引key，当进行条件查询时，就对位图进行位运算，取出bit值符合实际条件结构的rowID

  优点：存储空间小，执行速度快

  缺点：在插入、更新时需要计算位图值，因此不适合频繁更新的字段；列的特征值过多，导致位图向量太多，从而bit值种类太多，索引太多得不偿失

