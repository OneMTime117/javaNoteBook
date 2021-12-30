# JDBC核心技术

## 1、数据库四要素：

username：用户名

password：密码

url：JDBC连接地址，包含数据库地址、端口、实例名（服务名）、数据库名，然后一些连接属性：是否使用Unicode字符集，Unicode字符集的编码格式、连接使用的时区（useUnicode=true、characterEncoding=utf8、serverTimezone=GMT）

driver：JDBC数据库连接驱动，是JDBC原生api的实现层，用于操作指定数据库

### **常用数据库的url和driver：**

**1、mysql：**

​      url：jdbc:mysql://localhost:3306/hibernatedemo

​	  driver: com.mysql.cj.jdbc.Driver（mysql 6.x及以上）/com.mysql.jdbc.Driver(mysql 5.x及以下)

**2、Oracle：**

​      url：jdbc:oracle:thin:@localhost:1521/XE（服务名）

​				jdbc:oracle:thin:@localhost:1521:XE （实例名）

​		        jdbc:oracle:thin:@ (DESCRIPTION =

​				(ADDRESS = (PROTOCOL = TCP)(HOST = DESKTOP-B9FUSHL)(PORT = 1521))   
​				(CONNECT_DATA =               
​			    (SERVER = DEDICATED)          
​				(SERVICE_NAME = XE)         )  )   （TNS连接）

​     driver： oracle.jdbc.driver.OracleDriver

**3、sqlServer：**

​		url：jdbc:sqlserver://localhost:2433; DatabaseName=test

​		driver: com.microsoft.sqlserver.jdbc.SQLServerDriver

**4、Postgresql：**

​		url： jdbc:postgresql://10.19.0.11:5432/authorization

​		driver：org.postgresql.Driver

## 2、JDBC三对象：

### 1、JDBC使用过程

````java
        //通过配置文件读取四要素
		String url= PropertiesUtil.getPropertiesValue("DBConnection", "url");
		String username= PropertiesUtil.getPropertiesValue("DBConnection", "username");
		String password= PropertiesUtil.getPropertiesValue("DBConnection", "password");
		String driver= PropertiesUtil.getPropertiesValue("DBConnection", "driver");
		
		//创建JDBC三对象
		Connection conn=null;
		PreparedStatement pre = null;
		ResultSet res = null;
		try {
			Class.forName(driver);//驱动加载后，自动注册驱动管理器对象
			conn = DriverManager.getConnection(url, username, password);
			String sql = "xxxxx";
			pre = conn.prepareStatement(sql);
			res = pre.executeQuery();
			
		} catch (SQLException e) {
			e.printStackTrace();
		}finally {
			ConnectionClose.close(conn, pre, res);
		}
````

### 2、JDBC三对象作用

Connection：数据库连接对象，用于创建可以执行sql语句的Statement对象

Statement：用于执行sql语句，获得查询结果，分为三种实现类

​		1、Statement   正常执行sql语句，无法预编译，因此不能进行#{}参数传递；只能直接拼接sql（使用${}）从而可能导致sql注入

​		2、prepatedStatement    先对sql语句进行预编译，能进行#{}参数传递（来代替拼接sql、${},放在sql注入）；由于预编译，可以重复高效执行一条sql语句（批量插入）；可以操作流（Stream）等其他数据类型插入到sql中（Blob类型数据读取、插入）

​		3、CallableStatement   用于执行存储过程语句

ResultSet： 用于保存Statement执行sql的查询结果

**##JDBC预编译：**

 	对于预编译的sql语句，数据库驱动会先将带有？占位符的sql语句编译，生成对应的sql执行代码，并且缓存起来；然后进行？占位符传参后，就开始sql执行

​	优点：对于预编译语句，可以重复利用，可以减少sql语句的编译过程；并且预编译可以有效避免sql注入

## 3、数据库事务：

**事务：**一组逻辑操作单元，使数据从一种状态转化为另一种状态

**事务处理原则：**一个事务中执行多个操作时，即使出现故障，也要保证这个事务操作全部执行或者都不执行；保证数据库数据的一致性。

### **1、JDBC事务基本操作：**

1、默认情况下，JDBC执行sql后，会直接提交事务，因此为保证事务处理原则，应该关闭自动提交

````java
conn.setAutoCommit(false);
````

2、当执行完事务的所有操作后，手动提交事务

````java
conn.commit();
````

3、对于这些操作进行try-catch，当发生异常后，在catch进行事务回滚

````java
conn.rollback();
````

****当使用数据库连接池时，连接不会关闭，而是之后在继续使用。因此对于进行事务操作的conn对象，应在使用完后，恢复到默认设置，保证不影响再次使用。（将conn的自动提交事务打开，然后归还conn对象）

### **2、事务的ACID属性：**

1、原子性：事务时不可分割的工作单位，所有操作要么都执行，要么都不执行

2、一致性：事务提交会保证数据库从一个状态转化为另一个状态；不会出现其他的中间状态

3、隔离性：并发执行多个事务时，不能被相互干扰

4、持久性：事务一旦被提交，其数据的修改就是永久性的

### **3、事务并发问题：**

当同时执行多个事务，并且多个事务访问相同数据时，若不采取一定的隔离机制，会导致一些并发问题：

1、脏读：T1事务读取的T2事务更新的字段数据，但此时T2事务又发送了回滚，导致T1事务读取的数据是临时且无效的

2、不可重复读：T1读取一个字段数据，T2又更新该字段数据，并提交，之后T1在未提交前又读取该字段数据，导致事务前后两次相同操作的结果不同（破坏了事务本身的一致性）

3、幻读：T1读取一个字段的数据，T2又插入了多行该字段数据，并提交，之后T1在未提交前又读取字段数据时，导致多出多个查询数据（破坏了事务本身的一致性，但一般允许该问题产生）

### **4、四种事务隔离级别：**

针对于解决事务并发问题，数据库设置了四种隔离级别，来保证不同需求下，数据库事务的隔离性

1、Read Uncommited（可读未提交的事务）：会导致三种并发问题出现，因此一般不会使用

2、Read Commited（可读已提交事务）：只允许事务读取被提交的变更数据，解决了脏读问题

3、Repeatable Read （可重复读）：在事务执行期间，禁止其他事务对同行数据进行修改，解决脏读、不可重复读问题 

4、Serializable（串行化）：禁止多个事务同时修改（增删改）同一个表数据，解决所有并发问题，但是这样会大大降低并发性能，不推荐使用

Oracle支持2种事务隔离级别：Read Commited、Serializable，默认为 Read Commited

Mysql支持4种事务隔离级别，默认为Repeatable Read

**JDBC隔离级别设置：**

````java
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
````

**并不会改变数据库默认的隔离级别，只是修改当前连接操作的隔离级别**

### 5、只读事务：

​		在多条查询语句执行中，如果不使用事务，则会导致前后读取数据不一致，就比如分页操作；为了保证数据一致性，就需要使用只读事务		

- 优点：

  ​		在开启数据库事务的同时，告诉数据库当前连接会话只进行读操作，数据库从而提供一些优化手段

- 不同数据库，实现只读事务的方式也不同：
  - mysql：不能进行增删改操作，否则数据库会报错；在事务开始到结束时间内，对其他事务操作不可见，并不记录事务id和回滚log
  - oracel：不支持

**JDBC只读事务设置：**

```java
conn.setReadOnly=true;
```

## 4、数据库连接池：

### 1、普通JDBC数据库连接缺点：

1、JDBC使用DriverManager来进行来Connection连接对象创建，每次连接数据库都需要将 conn 加载到内存中，然后验证用户名密码，执行完sql后断开连接。每次使用数据库都需要这个过程，导致高并发时，占用大量系统资源，导致服务器崩溃

2、每次连接Conn创建都需要关闭，否则会发送数据库系统内存泄漏，最终导致数据崩溃重启

3、开发过程中不能控制连接Conn对象创建数据，这样会导致系统资源毫无顾虑的分配。当连接过多时，导致内存溢出，服务器崩溃

### 2、数据库连接池技术原理：

1、为解决传统开发中数据库连接问题，可以采用数据库连接池技术

2、**数据库连接池基本思想和java线程池类似**。保证连接池中放入一定数量的连接Conn，当需要使用数据库时，则冲连接池中取出，使用完后再放回去。

3、数据库连接池负责分配、管理、释放数据库连接，它允许应用程序重复使用一个现有的数据库连接。

4、数据库连接池可以设置最小连接数数和最大连接数。

### 3、数据库连接池技术优点：

1、数据库连接资源重用

2、数据库连接池在项目启动时，会进行初始化，同时创建最小数的连接对象，备用用于数据库连接操作，并且使用完后不用释放。避免了数据库来连接初始化和释放的过程，减少了系统的响应时间

3、可以管理数据库连接资源的分配

4、有完善的数据库连接创建、释放机制，避免了数据库连接操作可能出现的资源泄漏

### 4、连接池对于资源的关闭：

当使用连接池后，还是需要对于JDBC三对象进行关闭，减少资源占用

对于conn对象的close，连接池对于该方法进行了重写，close并不会释放数据库连接，而是改变该连接在连接池中的状态，有占用（活跃）转变为队列（休眠）状态，从而让连接池去判断是否释放不必要的连接

## 5、batch批量操作和大对象处理

1、Batch批量插入、更新    

````java
			String sql = "INSERT INTO user(id,name,password) values (?,?,?)";
			pre = conn.prepareStatement(sql);
			for (int i = 0; i < 10000; i++) {
				pre.setObject(1, "id_"+i);
				pre.setObject(2, "name_"+i);
				pre.setObject(3, "password_"+i);
				
//				通过pre的Batch对象，来实现批量执行sql，减少sql执行与数据库的交互次数（大大提升性能）
//				将该sql保存到Batch中，但不执行
				pre.addBatch();
				if ((i!=0 && i%500==0) || i=10000-1) {
					//将Batch中所有的sql一起执行
					pre.executeBatch();
					//清空Batch，用于后面保存sql
					pre.clearBatch();
				}
			}
````

2、大对象类型（blob、clob、text）处理

由于大对象类型数据可能很大，因此需要配置数据库插入的最大数据量（mysql默认为1M）

- 插入大对象类型

```java
		String sql="insert into phtot(id,`blob`,`clob`) values(1,?,?)";
		pre = conn.prepareStatement(sql);
//		将文件转化为字节流
		FileInputStream fileInputStream = new FileInputStream(new File("C:\\Users\\OneMTime\\Desktop\\Typora图片\\1566201801526.png"));
//		将文件转化为字符流
		FileReader fileReader = new FileReader(new File("C:\\Users\\OneMTime\\Desktop\\Mysql1.txt"));
		
		pre.setBlob(1, fileInputStream);
		pre.setClob(2, fileReader);//Oracl 大文本为clob类型
		pre.setCharacterStream(2, fileReader); //mysql 大文本为text类型
		pre.executeUpdate();
```

- 读取大对象类型

````java
		String sql = "select * from phtot where id =1";
		pre=conn.prepareStatement(sql);
		ResultSet executeQuery = pre.executeQuery();
		while (executeQuery.next()) {
			Blob blob = executeQuery.getBlob("blob");
//			Clob clob = executeQuery.getClob("clob");
//			//将blob转化为字节流
//			InputStream binaryStream = blob.getBinaryStream();
//			将clob转化为字符流
//			Reader characterStream = clob.getCharacterStream();
			
			//txet转化为字符流
			Reader characterStream2 = executeQuery.getCharacterStream("clob");
			if (characterStream2!=null) {
				BufferedReader bufferedReader = new BufferedReader(characterStream2);
				
				String s="";
				while((s=bufferedReader.readLine())!=null) {
					System.out.println(s);
				}
			}

		}
````
