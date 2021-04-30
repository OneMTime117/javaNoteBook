## Spring-data

spring数据访问模块由如下几个包构成：spring-jdbc、spring-tx、spring-orm、spring-oxm

spring-jdbc：封装jdbc操作，简化使用jdbc对数据库的访问；（依赖spring-tx，提供其功能）

spring-tx：提供对事务的管理，并统一所有DAO层异常类结构

spring-orm：用于spring整合JPA框架，实现对资源、事务的管理和DAO访问（类似于mybatis-spring .jar）

spring-oxm：提供java对象和xml文件之间的映射，提供映射转化一致接口和异常结构，方便使用（目前没有使用需求）

### spring JDBC：

spring JDBC用于简化jdbc操作，减低开发者编写JDBC中的一些简单固定API

#### jdbc操作点的工作分配：

| JDBC操作点         | spring | 开发者 |
| ------------------ | ------ | ------ |
| 定义连接参数       |        | X      |
| 打开连接           | X      |        |
| 指定sql语句        |        | X      |
| 声明参数并提供参数 |        | X      |
| 准备和执行sql语句  | X      |        |
| 遍历处理结果集     | X      |        |
| 处理异常           | X      |        |
| 处理事务           | X      |        |
| 关闭资源           | X      |        |

因此可以看出，spring-JDBC包，可以让开发者以非常优雅的方式使用jdbc，进行数据库交互

#### Jdbc核心操作类的使用：

##### JdbcTemplate

​		JdbcTemplate是JDBC包中的核心类，主要完成如下工作：

- 处理资源的创建和释放
- 执行sql语句，并获取结果集
- 捕获JDBC异常，转换为DataAccessException

**JdbcTemplate是线程安全的，可以用于多个sql执行，在使用时，一个DataSource对应一个JdbcTemplate；JdbcTemplate，每执行依次CRUD操作，都会从连接池中获取connection，但是会优先获取当前线程中创建的connection，当线程工作完成后，则关闭该connection，并且在不使用事务管理器时，固定为自动提交**

可以看出，JdbcTemplate的线程安全和获取connection方式和myabtis中的SqlSessionTemplate一致

###### JdbcTemplate配置：

需要手动配置dataSource、JdbcTemplate，spring-JDBC包中，提供DriverManagerDataSource类，来简单创建DataSource，当然也可以使用**带有连接池功能第三方dataSource**

```java
@Component
public class DataSourceConfig {
	@Value("${db1.username}")
	String username;
	
	@Value("${db1.password}")
	String password;
	
	@Value("${db1.url}")
	String url;
	
	@Value("${driver}")
	String driverClassName;

	@Bean
	public DataSource dataSource() {
		DriverManagerDataSource dataSource = new DriverManagerDataSource(url, username, password);
		dataSource.setDriverClassName(driverClassName);
		return dataSource;
	}
	
	@Bean
	public JdbcTemplate jdbcTemplate(@Qualifier("dataSource") DataSource dataSource) {
		JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
		return jdbcTemplate;
	}
}
```

###### JbdcTemplate的使用：

1、查询（select）

- 使用Object和基本数据类型包装类进行接收

```java
queryForObject（sql，String.class）//查询一个数据

Map<String,Object> map = queryForMap（sql）//查询一条数据

List<Map<String,Object>> list= queryForList（sql）//查询一组数据
```

- 使用实体对象，手动映射接收

```java
//查询一条数据
User query = jdbcTemplate.query("sql...",new ResultSetExtractor<User>() {
	@Override
	public User extractData(ResultSet rs) throws SQLException, DataAccessException {
		User user = new User();
		user.setName(rs.getString(""));
		user.setPassword(rs.getString(""));
		return user;
		}
	});

//查询一组数据
List<User> query2 = jdbcTemplate.query("sql...",new RowMapper<User>() {
	@Override
	public User mapRow(ResultSet rs, int rowNum) throws SQLException {
		User user = new User();
		user.setName(rs.getString(""));
		user.setPassword(rs.getString(""));
		return user;
		}
	});
```

对于ResultSetExtractor<T>、RowMapper<T>，可以使用Lambda表达式进行简化，并在当前DAO中，jdbcTemplate多个query使用相同的ResultSetExtractor<T>、RowMapper<T>，可以将其提取到dao类的成员变量中，供多个方法使用：

```java
private final RowMapper<Actor> actorRowMapper = (resultSet, rowNum) -> {
    Actor actor = new Actor();
    actor.setName(resultSet.getString("name"));
    actor.setPassword(resultSet.getString("password"));
    return actor;
};
```

**注意：**

- 当使用queryForMap、queryForList方法，将结果集封装为map时，我们使用列名进行数据取出时，spring已经对结果集中的数据进行了提取封装，因此只能使用sql中定义的大小写格式获取（**oracle在不使用双引号情况下，默认全部大写**）
- 当使用ResultSetExtractor、RowMapper进行实体对象映射时，使用resultSet.getString获取数据，不区分大小写（结果集获取数据不需要区分列名的大小写）

2、更新（insert、update、delete）

```java
int update = jdbcTemplate.update("sql...");
```

3、执行存储过程

```java
jdbcTemplate.execute("sql...");
```

3、绑定参数

**jdbcTemplate默认使用PreparedStatement**，来进行sql语句的预编码，并绑定参数;按照sql中的占位符顺序，在参数列表最后面，依次添加参数

```java
jdbcTemplate.queryForMap("selsect * from user where id=?",1);
```

4、单次批处理操作

提供一个BatchPreparedStatementSetter接口，来定义批处理的执行规则

```java
public void batchUpdate(final List<User> users) {		
		jdbcTemplate.batchUpdate("update User set name=? where id=?", new BatchPreparedStatementSetter() {
			@Override
			public void setValues(PreparedStatement ps, int i) throws SQLException {
                //遍历定义所有sql语句参数
				User user = users.get(i);
				ps.setString(1, user.getName());
				ps.setString(2, user.getId());
			}
			
			@Override
			public int getBatchSize() {
				return users.size();//批处理数
			}
		})
	}
```

或使用spring提供的SqlParameterSourceUtils来创建SqlParameterSource接口实现类数组，定义批处理对象

```java
//创建的SqlParmeterSource的实现类数组会在内部批处理绑定所有sql参数；但要保证有批处理需要的list集合
paramJdbcTemplate.batchUpdate("update User set name=? where id=?",SqlParameterSourceUtils.createBatch(users));
```

5、多次批处理

提供ParameterizedPreparedStatementSetter接口来定义

```java
jdbcTemplate.batchUpdate("update User set name=? where id=?",users,100,new ParameterizedPreparedStatementSetter<User>() {
	@Override
	public void setValues(PreparedStatement ps, User user) throws SQLException {
		ps.setString(1, user.getName());
		ps.setString(2, user.getId());
	}
});
```

##### NameParameterJdbcTemplate

​		该类对JdbcTemplate进行进一步封装，提供了一种新的绑定参数的编程方式(本质上还是使用了sql占位符)，解决了JdbcTemplateAPI使用时的弊端：

**jdbcTemplate在参数很多时，只能按顺序一一对应sql占位符，代码阅读性、维护性非常低**

1、map方式绑定参数：

```java
String sql="select * from where id=:id";
Map<String, Integer> singletonMap = Collections.singletonMap("id",1);
paramJdbcTemplate.queryForList(sql, singletonMap);
```

2、javaBean方式绑定参数：提供BeanPropertySqlParameterSource接口来实现

```java
String sql="select * from User where name=:name";
User user = new User();
user.setName("yh");
BeanPropertySqlParameterSource beanPropertySqlParameterSource = new BeanPropertySqlParameterSource(user);
List<Map<String, Object>> queryForList = paramJdbcTemplate.queryForList(sql, beanPropertySqlParameterSource);
```

##### SimpleJdbcInsert、SimpleJdbcCall（一般不使用）

​		这两个类可以通过JDBC驱动来操作数据库中的元数据，从而简化对insert操作和存储过程的调用：

1、简化insert插入表数据

2、insert时使用主键自增

3、简化调用存储过程，并处理返回值

​		使用注意：由于这两个类使用线程不安全，因此一般情况下，直接通过dataSource对象，在DAO层的初始化方法中创建，即因此DAO层进行数据访问都需要创建一个新对象

### spring tx：

​		spring tx包实现spring对事务的管理，并统一所有DAO层异常类结构

#### 正常本地事务和全局事务的局限(不使用spring事务管理)：

本地事务：单个特定资源的事务，如JDBC事务；它无法跨多个事务资源工作，并且在使用上会入侵编程代码

全局事务：javaEE规范中提供了JTA（Java Transaction API），来管理多个事务资源协调工作，如关系型数据库和消息队列。但是JTA非常繁琐，通常还需要使用JNDI从J2EE容器中获取外部数据源

spring事务提供了TransactionManager接口，来定义事务管理器，分为两种实现方式：

声明式事务管理：PlatformTransactionManager

反应式事务管理：ReactiveTransactionManager（spring5.2开始提供）

#### PlatformTransactionManager：

##### 声明式事务管理器基本概念：

```java
public interface PlatformTransactionManager extends TransactionManager {

    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

TransactionDefinition接口定义了事务的基本属性，来描述当前事务：

- 事务传播行为
- 事务隔离级别
- 事务超时时间
- 是否为只读事务

TransactionStatus接口提供控制事务执行和查询事务状态的简单方法：

```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {

    @Override
    boolean isNewTransaction();//是否为新创建的事务（事务嵌套时，判断是否使用了外层事务）

    boolean hasSavepoint();//是否存在回滚节点（用于嵌套事务）

    @Override
    void setRollbackOnly();//事务回滚

    @Override
    boolean isRollbackOnly();//是否可以是否回滚

    void flush();//事务刷新

    @Override
    boolean isCompleted();//是否完成回滚或提交
}
```

##### 声明式事务管理器的使用及其原理：

1、注册PlatformTransactionManager接口的实现类DataSourceTransactionManager，创建事务管理器

```java
@Bean
public DataSourceTransactionManager txManager(@Qualifier("dataSource") DataSource dataSource) {
	DataSourceTransactionManager dataSourceTransactionManager = new DataSourceTransactionManager(dataSource);
	return dataSourceTransactionManager;
}
```

2、使用AOP来实现事务管理

- 定义指定事务管理器的AOP切面策略，进行切入点的方法增强

  ```java
  <tx:advice id="txAdvice" transaction-manager="txManager">
      <tx:attributes>
      <tx:method name="get*" read-only="true" rollback-for="NoProductInStockException"/>
      <tx:method name="*"/>
      </tx:attributes>
  </tx:advice>
  ```

- 定义切入点，并指定该切入点的切面策略

  ```java
  <aop:config>
      <aop:pointcut id="fooServiceMethods" expression="execution(* x.y.service.*.*(..))"/>
      <aop:advisor advice-ref="txAdvice" pointcut-ref="fooServiceMethods"/>
  </aop:config>
  ```

**实际上，该AOP切面过程，就是以环绕通知的方式进行切入，对整个过程进行try-catch，当前方法发生异常、或事务管理发生异常时，则进行该方法中的事务回滚;没有异常抛出，则进行事务提交**

##### @Transactional

​		spring提供注解的方式，来更加直接的实现对指定方法的事务管理器的AOP，消除AOP切入点的定义

需要在配置类上添加@EnableTransactionManagement，来使@Transactional生效；或者在spring.xml中，添加**<tx:annotation-driven transaction-manager="txManager"/>**标签；该设置只在当前IOC容器生效，因此在使用spring-MVC时，需要注意该配置应添加到spring ioc容器中

**@Transactional注解属性：**

| 属性                     | 作用描述                                                  |
| ------------------------ | --------------------------------------------------------- |
| value/transactionManager | 指定使用的事务管理器，默认使用Primary的Bean               |
| propagation              | 事务传播，默认为PROPAGATION_REQUIRED                      |
| isolation                | 事务隔离级别，默认为Isolation.DEFAULT                     |
| timeout                  | 事务超时时间，默认为TransactionDefinition.TIMEOUT_DEFAULT |
| readOnly                 | 是否为只读事务，默认为false                               |
| rollbackFor              | 引起事务回滚的异常calss对象数组                           |
| rollbackForClassName     | 引起事务回滚的异常类名数组                                |
| noRollbackFor            | 不会导致事务回滚的异常calss对象数组                       |
| noRollbackForClassName   | 不会导致事务回滚的异常类名数组                            |

所有异常必须为Throwable的子类

##### spring事务的传播行为：

**传播行为值的是当一个事务方法被另一个事务方法调用时，改事务方法改如何进行。因此其配置作用是为了改变事务方法被嵌套在其他事务方法内部时，事务生效方式**

根据实际情况有七种类型，以如下模板为例：

存在两个类分别有一个增删改操作的方法，此时方法A内部调用方法B，我们需要关注方法B的传播行为属性：

| 传播行为属性     | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| REQUIRED（默认） | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，创建一个事务（该事务只会作用于方法B） |
| SUPPORTS         | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，方法B则会非事务执行 |
| MANDATORY        | 当方法A本身有事务时，方法B会使用方法A中的事务<br />当方法A没有事务时，方法B则会抛出异常 |
| NEVER            | 总是非事务执行，当方法A存在事务时，则抛出异常                |
| NESTED           | 当方法A本身有事务时，则使用方法A事务进行嵌套事务（即方法A发生异常，会导致方法B事务回滚，当方法B异常不会影响到方法A<br />当方法A没有事务时，创建一个事务（该事务只会作用于方法B） |
| NOT_SUPPORTED    | 总是非事务执行，当方法A存在事务时，则将其挂起（两个方法不会相互影响） |
| REQUIRES_NEW     | 总是使用新事务执行，当方法A存在事务时，将其挂起（两个方法不会相互影响） |

在进行事务传播中，内部事务（方法B事务）设置参数（隔离级别，超时和只读设置），只有当创建自己事务时，才会生效；否则就继承外部事务（方法A事务）特征；**因此这些参数只有在REQUIRED、REQUIRES_NEW中才有可能生效（创建自己新事务时生效）**

**目前NESTED、NOT_SUPPORTED、REQUIRES_NEW没有测试成功？？？？**

spring多数据源事务管理：

##### spring事务手动操作：

spring提供一个静态方法，获取当前线程中的事务管理器的TransactionStatus，来控制事务执行和查询事务状态

```java
TransactionStatus currentTransactionStatus = TransactionAspectSupport.currentTransactionStatus();
```

最常用的操作，回滚事务：(**其他方法，查看TransactionStatus接口**)

```java
TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
```

##### spring事务使用注意事项：

1、在同一个类中，事务方法A调用事务方法B，事务的实现情况：

事务方法A执行时，触发AOP动态代理，进行切面事务处理，之后调用目标Bean的方法A，从而执行目标Bean的方法B，此时并不会触发AOP动态代理，因此**方法A、方法B都被同一个事务管理**

###### 2、在同一个类中，非事务方法调用事务方法时，会导致事务失效

- 原因：

事务管理依赖于AOP，spring只会在调用事务方法时才进行AOP动态代理，在执行非事务方法时实际还是使用的原对象调用方法，因此内部调用事务方法时，会使用当前原对象，导致没有触发代理、事务失效

- 解决方式：

1、避免这种情况的方法调用，将事务方法单独写在一个类中（**推荐**）

2、在非事务方法调用事务方式时，先手动获取其类的AOP代理对象，然后调用事务方法

前提：@EnableAspectJAutoProxy(exposeProxy=true)

```java
UserServiceImpl currentProxy = (UserServiceImpl) AopContext.currentProxy();
currentProxy.testA();
```

3、在非事务方法调用事务方式时，利用一个工具类，自动获取目标类的AOP代理对象，然后调用事务方法(**和方法2基本一致**)

```java
ServiceUtil.getUserService().testA()
```

4、将非事务方法添加事务注解，使得其类方法统一进行AOP增强，从而在调用时使用同一个AOP代理对象（**但这样会增加事务管理的成本，在代码量很大的情况下，这种方式不可取**）

###### 3、只读事务和不使用事务的区别：

当只执行单条查询语句时，则不需要使用事务；当执行多条查询语句时，需要通过事务来确保整体数据的一致性，一致性的程度和当前事务的隔离级别有关；比如查询所有数据和所有数据量时，当出现并发两条sql之间数据被其他用户修改时，则导致结果不统一

###### 4、手动回滚对事务传播的影响：

当对内部方法代码进行异常捕捉手动回滚时，由于事务传播原因，当前方法使用的是外部方法事务，当手动在内部方法提交事务后，外部方法会再提交一次，因此会报错事务已被回滚

当对外部方法代码进行异常捕捉手动回滚时，在它们抛出异常时，内部方法会和外部方法一起进行手动回滚；

#### JtaTransactionManager：

JtaTransactionManager用于分布式事务管理器，对JTA进行了进一步的封装，简化了其API的使用，由于JTA只是J2EE规范接口，其实现交给第三方来完成：Atomikos是一个实现了JTA规范的框架，并且不需要在J2EE容器中使用（解决了JTA使用的弊端），用于在单个应用中，解决多个事务资源同步问题（包括多种资源的事务，如JDBC、消息队列）

##### **XA（分布式事务规范）：**

​	定义了全局事务管理器（TM）和局部资源管理器（RM）之间的接口；使得事务管理器可以和多个资源管理器相互通信，来协调不同资源的事务管理，主流的关系型数据库都实现了XA接口

​	XA使用**二阶段提交**方式来管理全局事务，第一阶段为准备阶段，各个资源管理器将参与事务的资源锁住，并响应回告给事务管理器；第二阶段为提交阶段，事务管理器通知资源管理器进行事务提交；并且**XA不能够进行自动提交**

​	XA的缺点：对于提交方式的准备阶段，维护成本非常高，性能与本地事务相差10倍

##### spring+mybaits+Atomikos的整合使用：

这些框架可以共同实现对多事务资源的同步管理

#### 自定义spring事务注解：

​		我们在使用@Transaction注解时，可以知道一个方法中，只能定义一个事务管理器，当方法中需要调用多个数据源的sql，当前方式就无法满足我们的使用；除了使用本地分布式事务管理器外，我们还可以使用最原始的方式，重新定义一个可以操作多个数据源事务管理器进行提交、回滚的切面，即自定义spring事务注解：

1、自定义事务管理注解类(模仿@Transaction)：

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface MyTransaction {

		//事务管理器Name集合
		String[] name() default {""};

		//所有事务管理器的传播行为
		Propagation propagation() default Propagation.REQUIRED;
		
		//所有事务管理器的隔离级别
		Isolation isolation() default Isolation.DEFAULT;

		//所有事务管理器不触发回滚的异常
		Class<? extends Throwable>[] noRollbackFor() default {};
}
```

2、定义处理自定义事务注解所在方法的AOP切面类：

```java
@Component
@Aspect
public class TransactionAop {
	@Autowired
	ApplicationContext context;

	// 定义公共切入点(声明被该注解标记的方法)
	@Pointcut(value = "@annotation(com.yh.jdbc.MyTransaction)")
	public void pointCut() {
	};

	// 进行环绕通知
	@Around(value = "pointCut()")
	public Object transactionalGroupAspectArround(ProceedingJoinPoint pjp) throws Throwable {
		// 原方法返回值
		Object obj = null;
		// 是否进行了事务管理
		boolean doit = false;

		// 获取当前切入点方法对象
		MethodSignature signature = (MethodSignature) pjp.getSignature();
		Method method = signature.getMethod();

		// 获取该方法中事务注解元数据
		MyTransaction annotation = method.getAnnotation(MyTransaction.class);

		// 初始化事务管理器相关对象集合
		List<TransactionParam> tsParams = null;

		if (annotation != null) {
			Isolation isolation = annotation.isolation(); // 事务隔离级别
			Propagation propagation = annotation.propagation(); // 传播行为
			// 不触发回滚的异常类名set
			Set<String> noRollbackForNameSet = new HashSet<>();
			Class<? extends Throwable>[] noRollbackFor = annotation.noRollbackFor();
			if (noRollbackFor != null && noRollbackFor.length > 0) {
				for (Class<? extends Throwable> clazz : noRollbackFor) {
					noRollbackForNameSet.add(clazz.getSimpleName().toLowerCase());
				}
			}

			// 获取所有事务管理器Name数组
			String[] transactionNames = annotation.name();
			try {
				if (transactionNames.length > 0) {
					doit = true;// 执行当前方法的事务管理

					tsParams = new ArrayList<TransactionAop.TransactionParam>(transactionNames.length);
					for (String transactionName : transactionNames) {
						// 获取当前事务管理器,并定义其默认事务对象，获取事务状态
						TransactionParam tsParam = new TransactionParam();
						DataSourceTransactionManager tm = context.getBean(transactionName,
								DataSourceTransactionManager.class);
						DefaultTransactionDefinition def = new DefaultTransactionDefinition();
						def.setIsolationLevel(isolation.value());
						def.setPropagationBehavior(propagation.value());
						TransactionStatus transaction = tm.getTransaction(def);

						// 将所有事务管理器信息封装到TransactionParam对象中
						tsParam.def = def;
						tsParam.status = transaction;
						tsParam.tm = tm;
						tsParams.add(tsParam);
					}

					// 执行原方法代码
					obj = pjp.proceed();

					// 提交所有事务
					for (TransactionParam tsParam : tsParams) {
						tsParam.tm.commit(tsParam.status);
					}
				}
			} catch (Throwable e) {
				boolean rollback = true;// 事务回滚

				String errName = e.getClass().getName().toLowerCase();

				if ("transactionexception".equals(errName)) { // 这是sql异常，救不了了
					rollback = true;
				} else {
					if (noRollbackForNameSet.contains(errName)) { // 发生异常，不回滚
						rollback = false;
					}
				}

				if (!rollback) { // 事务不回滚
					try {
						for (TransactionParam tsParam : tsParams) {
							tsParam.tm.commit(tsParam.status);
						}
						throw e; // 直接抛出异常，end
					} catch (org.springframework.transaction.TransactionException e1) {// 如果这步还出现sql异常，只能回滚事务了
						e1.printStackTrace();
					}
				}

				// 事务回滚(在循环回滚时,第一次回滚成功后，会将事务同步锁处于非活跃状态，导致当前线程无法继续操作其他事务)
				for (TransactionParam tsParam : tsParams) {
					if (!TransactionSynchronizationManager.isSynchronizationActive()) {
						TransactionSynchronizationManager.initSynchronization();
					}
					tsParam.tm.rollback(tsParam.status);
				}

				throw e;
			}
		}
		if (!doit) {// 不需要进行方法增强时，则执行原方法代码
			obj = pjp.proceed();
		}
		return obj;
	}

	class TransactionParam {
		// 事务服务类
		DataSourceTransactionManager tm;
		// 默认事务对象
		DefaultTransactionDefinition def;
		// 事务状态
		TransactionStatus status;
	}
}
```

3、使用自定义注解，进行组合事务管理器的AOP，实现多数据源事务同步

```java
@MyTransaction(name= {"txManagerA","txManager"})
	public void testA() {
		jdbcTemplate.update("delete from user where id =?", "36");
		jdbcTemplateA.update("delete from test where id=?","13");
		System.out.println("A删除成功");
		int i =1/0;
	}
```

**该方式无法正确的使用事务的传播行为，因此要避免事务方法的嵌套调用**

#### 对DAO操作的抽象：

spring对数据方法对象（DAO）提供了一个统一的异常抽象层，来轻松了在各种dao层技术（JDBC、hibernate、JPA）中切换使用

- **统一的异常类结构框架：**

对于所有的DAO层操作错误，统一使用**DataAccessException**异常抛出，方便各种dao层技术中产生异常的捕捉；实现方式为：通过@Repository注解，来将dao层类注册到IOC容器中；对于mybatis中的@Mapper注解，和@Repository作用一致

### spring ORM：

​		spring ORM提供了对JPA的集成，用于支持它们的资源管理、dao访问和事务管理，常见实现JPA的框架有：Hibernate、spring data jpa；

​		注意：对于mybatis，并不属于JPA框架，因此不需要spring ORM；由于目前并没使用到这些ORM框架，因此不进行深入了解

