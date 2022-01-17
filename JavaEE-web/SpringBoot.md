# SpringBoot

## 1、自动配置

### 1、自动配置开启

​		springboot基于spring-boot-autoconfigure包，提供系列模块自动配置类，通过添加`@SpringBootApplication`或者@EnableAutoConfiguration来开启自动配置。并且可以通过exclude、excludeName属性，来排除指定自动配置类

​		@SpringBootApplication是一个复合注解，包含@EnableAutoConfiguration、@ComponentScan、@SpringBootConfiguration，其作用分别为：

- 开启自动配置（@EnableAutoConfiguration）
- Bean组件扫描（@ComponentScan）
- 和@Configuration一致，标记当前类为配置类（@SpringBootConfiguration）

在spring-boot-autoconfigure包下的META-INF/spring.factories文件，定义了所有的自动配置类，在springBoot应用启动时，进行加载

### 2、自动配置约束

​		springboot通过约束注解，来控制配置类的生效场景；包含如下注解：

- class约束：

  **@ConditionalOnClass/@ConditionalOnMissingClass**

  ​		判断如果classpath路径下包含/不包含指定类时，才会实例化当前bean，注入到IOC容器中

  **注意：**

  ​		class约束不能直接搭配@Bean使用，因此类初始化时，JVM就加载了类方法中的引用，导致约束条件不满足时，方法引用失败

- Bean约束：

  **@ConditionalOnBean/@ConditionalOnMissingBean**

  ​		根据注解的value或type值，判断如果IOC容器中存在/不存在指定beanName或beanType时，才会实例化当前bean，注入到IOC容器中

- 配置文件属性约束：

  **@ConditionalOnProperty**

  根据prefix、name属性获取配置文件中定义的属性，判断是否和havingValue值相同，如果相同则注入当前bean

  ```java
  @ConditionalOnProperty(prefix = "scheduling", name = "enabled", havingValue = "true")
  @Configuration
  public class TaskConfig {
  }
  ```

- 资源条件

  **@ConditionalOnResource**

  判断指定资源是否存在，如果存在则注入当前bean

  ```java
  @ConditionalOnResource(resources = {"file:/home/user/test.dat"})
  @Configuration
  public class TaskConfig {
  }		
  ```

## 2、配置属性

### 1、自定义配置属性

​	springboot基于模块，本身在配置文件中提供了许多模块可配置属性入口，方便开发者在不同环境下，进行相应配置；

对于开发者手动创建的Bean，其属性同样也可以在配置文件中配置，避免进行硬编码，通过`@ConfigurationProperties`来实现：

根据Bean的创建方式，`@ConfigurationProperties`使用分为两种：

- 搭配@Bean

```java
@ConfigurationProperties(prefix="bean.user-info")
@Bean
public UserInfo getUserInfo(){
    return new UserInfo();
}
```

- 搭配@Component

```java
@ConfigurationProperties(prefix="bean.user-info")
@Component
public UserInfo{
    private String name;
    
    private String password;
    
    public void setName(String name){
        this.name=name;
    }
    
    public void setPassword(String password){
        this.password=password;
    }
}
```

**注意 ：**

- 由于属性自动注入基于setter方法，因此必须包装自定义配置的属性提供对应的setter方法

- 相对于@Value进行属性注入,@ConfigurationProperties必须保证application配置文件中的自定义数据配置key的书写规范,**全小写,使用 - 代替驼峰模式中的大写**

  ```yaml
  user-info:
  	name: yh
  ```

### 2、声明配置属性元数据

​		在配置文件中，对于自定义属性，IDE会抛`Cannot resolve configuration property`警告，原因是自定义属性缺失元数据，不能提示开发者该属性的注入去向

​		通过springToolSuite插件生成src/main/resource/META-INF/additional-spring-configuration-metadata.json文件，用于自定义配置属性元数据

```java
{
  "properties": [
    {
      "name": "scheduling.enabled",
      "type": "java.lang.String",
      "description": "是否开启springboot任务模块"
    }
  ] 
}
```

​		然后引入`pring-boot-configuration-processor`，配置文件-元数据处理器，在项目Maven编译时，对生成了json文件处理，实现对配置文件内属性的提示和补全

maven依赖引入：

```xml
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-configuration-processor</artifactId>
     <optional>true</optional>
</dependency>
```

## 3、profile

### 1、外部化配置优先级

​		springboot允许进行外部化配置，以便于开发者在不同环境下使用相同代码。其中外部配置来源包括：

application配置文件、java配置文件、环境变量、命令行参数

​		而这些配置文件属性值，可以如下方式注入：

@Value、@ConfigurationProperties、@PropertySource、Environment对象

​		通过这些配置来源和方式，springboot有对应的优先级，从而可以实现外部配置的覆盖，优先级从小到大：

- 默认属性（通过SpringAppliction.setDefaultPropertes设置）
- @PropertySource引入外部配置文件（注意，该属性只能在应用上下文刷新后，才会生效），搭配@Value、@ConfigurationProperties使用
- 配置文件
- 系统环境变量
- java系统属性（System.getProperties）
- ServletContext、ServletConfig初始化参数
- 命令行参数
- 测试应用程序时，@SpringBootTest中的属性

### 2、profile多环境配置

​		springboot支持多个配置文件，其配置文件名称必须为：

application.yml\application-{profile}.yml;其中application.yml为主配置文件，通过在其中声明spring.profiles.active属性，来选择加载其他配置文件

​		**注意：spring.profiles.active属性为数组，因此可以指定多个，即多个配置文件同时生效**

springboot默认优先./config/目录下的配置文件，因此一般推荐配置文件保存在resources/config/中

### 3、@Profile环境约束

​		通过profile环境配置，可以直接在代码中，使用@Profile注解，来约束Bean的创建注入

```java
@Profile({"!prod","dev"})  //profile不包含prod，且包含dev时，创建Bean
@Component
@Data
public class UserInfo {
	@Value("${beanIoc.userInfo.name}")
	private  String name;

	private String password;
}
```

## 4、开发者工具

​		springboot提供一系列额外开发者工具，在应用程序开发中提供生产效率，其重要功能就是实现热部署：

当开发者修改classpath下的文件后，快速重新启动应用

- 快速重启原理

  ​	spring-boot-devtools使用两个类加载其来重启应用：

  - base ClassLoader：基础类加载器，用于加载第三方依赖jar包
  - restart ClassLoader：重启类加载器，用于加载本地classpath下的类文件，当重启后，会自动销毁restart ClassLoader以及其加载的本地class文件，重新创建该类加载器实例，从而重新加载本地class文件

  因此，当使用热部署重启时，不需要重新加载第三方依赖jar包

- maven依赖引入：


```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-devtools</artifactId>
   <optional>true</optional>
</dependency>
```

- 注意：

  ​		热部署重启的触发，是通过监控类路径资源的变化。对于IDEA，通过buildProject触发热部署，对于Eclipse保存后，直接会编译修改后的文件，从而触发热部署

  ​		对于/META-INF/maven，/META-INF/resources，/resources，/static，/templates，/public这些目录下的文件修改，默认不会触发热部署

  ​		使用java -jar允许打包jar文件时，会自动关闭热部署功能

- devtools相关配置

```yaml
spring:
	devtools:
		restart:
			#是否开启监控
			enabled= ture
			#排除监控路径
			exclude: static/**,public/**
			#添加监控路径
			additional-paths:/data/file
```

- 实时重载

  ​		devtools还提供一个嵌入式liveReload服务器，用于触发浏览器刷新，从而实时加载变更资源；由于目前都是前后端分离架构，因此选择直接关闭该功能

```yml
spring:
	livereload:
    	enabled: false
```

## 5、springboot集成模块

### 1、spring-jdbc

​		spring JDBC用于简化jdbc操作，减少开发者编写JDBC中的一些简单固定API

- 引入maven依赖

  ```xml
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-jdbc</artifactId>
      </dependency>
  ```

#### 1.1、jdbc简化细节

| JDBC操作点         | spring-jdbc | 开发者 |
| ------------------ | ----------- | ------ |
| 定义连接参数       |             | X      |
| 打开连接           | X           |        |
| 指定sql语句        |             | X      |
| 声明参数并提供参数 |             | X      |
| 准备和执行sql语句  | X           |        |
| 遍历处理结果集     | X           |        |
| 处理异常           | X           |        |
| 处理事务           | X           |        |
| 关闭资源           | X           |        |

因此可以看出，spring-jdbc可以让开发者以非常优雅的方式使用jdbc，进行数据库交互

#### 1.2、jdbcTemplate

​		spring-jdbc提供封装模板，jdbcTemplate完成如下工作：

1、JDBC资源的创建和释放

2、执行sql语句，获取结果集

3、捕获jdbc异常，转化为DataAccessException

​		**jdbcTemplate线程安全，每个DataSource对应一个jdbcTemplate，执行jdbc操作时，都会从连接池中获取connection对象，并在线程工作完成后，关闭connection对象；在未使用事务管理器时，默认自动提交事务**

- **配置：**

springboot根据dataSource和相关配置文件提供默认配置(JdbcTemplateConfiguration)

spring-jdbc包提供`DriverManagerDataSource`来创建dataSource,也可以其他依赖的实现类进行创建（如`DruidDataSource`）

```java
@Component
public class DataSourceConfig {
	@Value("${db.username}")
	String username;
	
	@Value("${db.password}")
	String password;
	
	@Value("${db.url}")
	String url;
	
	@Value("${db.driver}")
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

- **使用：**

  1、查询

  - 使用集合接收

    ```java
    queryForObject(sql,String.class)//查询一条单字段数据
        
    Map<String,Object> map = queryForMap（sql）//查询一条数据
    
    List<Map<String,Object>> list= queryForList（sql）//查询一组数据
    ```

  - 实体类接收

    ```java
    //查询一条数据
    User query = jdbcTemplate.query(sql,new ResultSetExtractor<User>() {
    	@Override
    	public User extractData(ResultSet rs) throws SQLException, DataAccessException {
    		User user = new User();
    		user.setName(rs.getString(""));
    		user.setPassword(rs.getString(""));
    		return user;
    		}
    	});
    
    
    //查询一组数据
    List<User> query2 = jdbcTemplate.query(sql,new RowMapper<User>() {
    	@Override
    	public User mapRow(ResultSet rs, int rowNum) throws SQLException {
    		User user = new User();
    		user.setName(rs.getString(""));
    		user.setPassword(rs.getString(""));
    		return user;
    		}
    	});
    ```

    注意：

    - 使用queryForMap、queryForList接收数据时，key默认为结果集中的字段名，区分大小写
    - 使用ResultSetExtractor、RowMapper进行实体对象映射时，使用resultSet.getString获取数据，不区分大小写

  2、更新

  ```java
  int update = jdbcTemplate.update(sql);
  ```

  3、占位符参数绑定

  jdbcTemplate默认使用PreparedStatement，提供占位符参数绑定功能

  ```java
  //按照sql中的占位符顺序，在参数列表最后面，依次添加参数
  jdbcTemplate.queryForMap("selsect * from user where id=?",1);
  ```

  4、批处理操作

  jdbcTemplate提供BatchPreparedStatementSetter接口，来定义批处理执行规则：

  ```java
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
  ```

  对于list数据较多时，应该使用多次批处理操作，使用ParameterizedPreparedStatementSetter进行定义：

  ```java
  jdbcTemplate.batchUpdate("update User set name=? where id=?",users,100,new ParameterizedPreparedStatementSetter<User>() {
  	@Override
  	public void setValues(PreparedStatement ps, User user) throws SQLException {
  		ps.setString(1, user.getName());
  		ps.setString(2, user.getId());
  	}
  });
  ```

#### 1.3、nameParameterJdbcTemplate

​		jdbcTemplate的进一步封装，更好的进行jdbc参数绑定，实现以map集合或实体类来绑定参数，提高代码的可阅性、维护性

- map绑定

```java
String sql="select * from where id=:id";
Map<String, Integer> singletonMap = Collections.singletonMap("id",1);
paramJdbcTemplate.queryForList(sql, singletonMap);
```

- 实体类绑定

```java
String sql="select * from User where name=:name";
User user = new User();
user.setName("yh");
//将实体类封装为BeanPropertySqlParameterSource
BeanPropertySqlParameterSource beanPropertySqlParameterSource = new BeanPropertySqlParameterSource(user);
List<Map<String, Object>> queryForList = paramJdbcTemplate.queryForList(sql, beanPropertySqlParameterSource);
```

#### 1.4、其他

​		spring-jdbc还提供`SimpleJdbcInsert`、`SimpleJdbcCall`：

- SimpleJdbcInsert用于简化insert操作，提供调用数据库主键自增功能
- simpleJbdcCall用于简化数据库存储过程调用

#### 1.5、数据源和连接池

​		springboot提供spring.datasource.*来配置jdbc数据源，同时提供了hikari、tomcat、dbcp2和oracleucp四种连接池的自动配置，默认引入了hickari第三方依赖，实现数据库来连接池功能

​		开发者可以通过手动创建DataSource，来自定义配置文件参数的引入和连接池的选择，比如Druid连接池：

```java
	@ConfigurationProperties(prefix="spring.datasource")
	@Bean
	public DataSource dataSource(){
		return new DruidDataSource();    //使用Druid线程池
		//return DataSourceBuilder.create().build();使用springboot默认线程池
	}
```

### 2、spring-tx

​		spring-tx实现spring事务管理

spring-tx基于spring-jdbc依赖引入

#### 2.1、spring事务管理抽象

​		spring提供了`PlatformTransactionManager `接口，来定义事务管理抽象

```java
public interface PlatformTransactionManager extends TransactionManager {

    //获取当前事务状态
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException;

    //提交事务
    void commit(TransactionStatus status) throws TransactionException;

    //事务回滚
    void rollback(TransactionStatus status) throws TransactionException;
}
```

- TransactionDefinition接口

  定义事务基本属性：

  1、事务传播行为

  2、事务隔离级别

  3、事务超时时间

  4、事务只读状态

  ```java
  public interface TransactionDefinition {
      int getPropagationBehavior(); // 返回事务的传播行为
      int getIsolationLevel(); // 返回事务的隔离级别
      int getTimeout();  // 返回事务超时时间
      boolean isReadOnly(); // 返回事务只读状态
  } 
  ```

- TransactionStatus接口

  定义事务执行、查询事务状态的方法

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

在JavaEE中，事务管理分为全局事务和局部事务：

- 全局事务可以实现对多个事务资源的同时管理，通常为关系型数据库和消息队列，基于JTA（Java Transaction API）规范实现
- 本地事务实现对单一事务资源的管理，如JDBC事务

spring基于事务管理抽象，从而提供在任何平台下的一致编程模型：

![](../Typora图片/spring事务抽象.png)

#### 2.2、jdbc事务管理

​		spring-jdbc提供`DataSourceTransactionManager`来实现对jdbc事务管理,从而支持jdbcTemplate和myBatis

- **配置：**

  springboot提供自动配置（`DataSourceTransactionManagerAutoConfiguration`）

  在多数据源情况下，需要根据不同数据源进行手动配置：

  ```java
      @Bean(name = "baseTransactionManager")
      @Primary
      public DataSourceTransactionManager setTransactionManager(@Qualifier("baseDataSource") DataSource dataSource) {
          return new DataSourceTransactionManager(dataSource);
      }
  ```

  在配置类中，使用`@EnableTransactionManagement`开启事务管理

- **使用：**

  通过在Service层方法添加@Transaction注解，来实现该方法内jdbc事务

  ```java
  @Transaction
  public void update(){
      
  }
  ```


#### 2.3、spring事务的传播行为

​		事务传播行为：

​				用于定义事务方法被调用时，事务的生效方式

​		spring事务定义了7种传播类型,根据当前调用事务方法的环境是否存在事务,分为两种情况：

- A类：当前环境没有事务
- B类：当前环境有事务

| 传播行为属性                     | 描述                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| REQUIRED（必需使用事务，默认值） | A类：调用方法自己创建一个事务，仅用于自己<br />B类：调用方法使用当前环境的事务 |
| SUPPORTS（只支持外部事务）       | A类：调用方法以非事务方式执行<br />B类：调用方法使用当前环境的事务 |
| MANDATORY（强制使用外部事务）    | A类：调用方法抛出异常<br />B类：调用方法使用当前环境的事务   |
| NEVER（永不使用事务）            | A类：调用方法以非事务方式执行<br />B类：调用方法抛出异常     |
| NESTED（允许嵌套使用外部事务）   | A类：调用方法自己创建一个事务，仅用于自己<br />B类：调用方法使用当前环境的事务，并进行嵌套处理（调用方法发生异常回滚后，并不会影响外层方法的事务处理） |
| NOT_SUPPORTED（不支持外部事务）  | 总是非事务执行，不会受到当前环境事务的影响                   |
| REQUIRES_NEW（必须使用新事务）   | 总是创建一个新事务作用于自己，不会受到当前环境事务的影响     |

**注意：**

对于事务隔离级别，超时和只读设置，在当前环境有事务时，只有调用方法创建自己的新事务才会生效，（即传播行为：REQUIRED、REQUIRES_NEW）

**在测试REQUIRED和NESTED区别时，外层方法需要使用try-catch捕捉导致内层方法事务回滚的异常，从而判断事务嵌套处理时，事务只进行内层方法的部分回滚**

#### 2.4、声明式事务管理

​		spring-tx提供@Transactional注解，来进行声明式事务管理，省去重复的事务管理逻辑，减少对业务代码的侵入性

​		@Transactional可以注解在方法、接口或者类上

**原理：**

​		通过AOP切面进行环绕通知，在方法前后进行事务开启、提交或回滚操作。通过代理对象对原方法进行try-catch，判断事务是否回滚

```java
  		//开启事务
      TransactionInfo txInfo = createTransactionIfNecessary(tm, txAttr, joinpointIdentification);
      Object retVal = null;
      try {
         // This is an around advice: Invoke the next interceptor in the chain.
         // This will normally result in a target object being invoked.
          //方法调用
         retVal = invocation.proceedWithInvocation();
      }
      catch (Throwable ex) {
         // target invocation exception
   		//回滚事务
         completeTransactionAfterThrowing(txInfo, ex);
         throw ex;
      }
      finally {
         cleanupTransactionInfo(txInfo);
      }
       //提交事务
      commitTransactionAfterReturning(txInfo);
      return retVal;
```

**@Transaction注解属性：**

| 属性                     | 作用描述                                                     |
| ------------------------ | ------------------------------------------------------------ |
| value/transactionManager | 指定使用的事务管理器，默认使用Primary的Bean                  |
| propagation              | 事务传播，默认为PROPAGATION_REQUIRED                         |
| isolation                | 事务隔离级别，默认为Isolation.DEFAULT                        |
| timeout                  | 事务超时时间，默认为TransactionDefinition.TIMEOUT_DEFAULT（-1） |
| readOnly                 | 是否为只读事务，默认为false                                  |
| rollbackFor              | 引起事务回滚的异常calss对象数组                              |
| rollbackForClassName     | 引起事务回滚的异常类名数组                                   |
| noRollbackFor            | 不会导致事务回滚的异常calss对象数组                          |
| noRollbackForClassName   | 不会导致事务回滚的异常类名数组                               |

**@Transactional失效场景：**

- 方法非public访问权限：

  ​		aop代理对象会校验带有@Transactional的方法是否被public修饰

- 在同一类中，非事务方法调用了事务方法：

  ​		事务管理依赖于aop代理，当事务方法在对象内部被调用时，并不会从IOC容器中获取带有事务管理功能的代理对象，而是使用原对象Bean执行该事务方法

  ​		解决方法：

  ​		手动获取该类的AOP代理对象：

  ```java
  //1、通过AopContext获取当前类的代理对象
  UserServiceImpl currentProxy = (UserServiceImpl) AopContext.currentProxy();
  currentProxy.testA();
  
  //2、通过ApplicationContext，获取指定类的最终Bean（代理对象）
  applicationContext.getBean(clazz);
  ```

**@Transactional事务回滚：**

​		当方法发生异常时，就会触发事务回滚。有时，我们需要对异常进行日志处理，进行手动try-catch，此时就会导致事务无法回滚。因此就需要进行编程式手动回滚：

```java
    try {
        // some business logic...
    } catch (NoProductInStockException ex) {
        // trigger rollback programmatically
        TransactionAspectSupport.currentTransactionStatus().setRollbackOnly();
    }
```

​		**注意:**

​		由于事务传播行为的存在,进行手动回滚时,需要保证当前事务只针对于该方法;如果外层方法也使用该事务,就会导致事务重复提交或回滚,抛出异常

**@Transaction触发回滚的异常选择:**

​		默认情况下，spring事务只会对RuntimeException进行处理（Error默认会可以），因此对于方法抛出的检查性异常不能导致事务回滚

​		为了避免这种情况的发生,推荐使用@Transactional(rollbackFor = Exception.class)进行注解

```java
	@Transactional(rollbackFor = Exception.class)
	public void update() throws IOException {
		// some business logic...
		throw new IOException();
	}
```

#### 2.5、编程式事务管理

​		spring还提供编程式事务管理,通过`TransactionTemplate`封装模板或者PlatformTransactionManager事务管理对象进行操作

**TransactionTemplate:**

​		线程安全,springboot在单数据源环境下,默认提供;但由于不同业务下,使用的事务属性不同,因此一般情况下,需要开发者手动创建多个不同属性的transactionTemplate

```java
	@Bean("default")
	public TransactionTemplate  transactionTemplate(PlatformTransactionManager transactionManager){
		TransactionTemplate transactionTemplate = new TransactionTemplate(transactionManager);
		//transactionTemplate属性设置
		return transactionTemplate;
	}
```

**使用:**

```java
public void update() {
		transactionTemplate.execute(new TransactionCallbackWithoutResult() {
			@Override
			protected void doInTransactionWithoutResult(TransactionStatus status) {
				try {
					SysUser sysUser = new SysUser();
					sysUser.setUsername("1111");
					sysUser.setPassword("1");
					sysUserMapper.insert(sysUser);
				} catch (Exception ex) {
					status.setRollbackOnly();
				}
			}
		});
	}
```

transactionTemplate.exectue提供两种回调函数:

- TransactionCallbackWithoutResult  无返回值
- TransactionCallback<String>  有返回值

**PlatformTransactionManager:**

```java
	//创建事务
    DefaultTransactionDefinition transDef = new DefaultTransactionDefinition(); 
	//定义事务属性
    transDef.setPropagationBehavior(DefaultTransactionDefinition.PROPAGATION_REQUIRED); 
	//获得事务状态
    TransactionStatus status = dataSourceTransactionManager.getTransaction(transDef);  	
    try {
        // 数据库操作
        dataSourceTransactionManager.commit(status);// 提交
    } catch (Exception e) {
        dataSourceTransactionManager.rollback(status);// 回滚
    }
```

### 3、spring-redis

spring-redis用于提供spring程序配置访问redis的功能，简化开发者对redis的使用

- 引入maven依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

#### 3.1、redis客户端

​		spring-redis支持使用Lettuce、Jedis两种客户端进行redis连接，它们对redis功能支持性有所不同：

| 功能               | Lettuce                              | Jedis                      |
| ------------------ | ------------------------------------ | -------------------------- |
| 独立连接           | X                                    | X                          |
| 主从模式           | X                                    |                            |
| reids哨兵模式      | 主节点查找、哨兵身份验证、子节点读取 | 主节点查找                 |
| redis集群模式      | 集群连接、集群节点连接、子节点读取   | 集群连接、集群节点连接     |
| 单连接实例并发使用 | X                                    | 需要基于JedisShardInfo实现 |
| 反应式API          | X                                    |                            |

综上所述，在redis功能支持和性能上，Lettuce客户端更加强大，同时spring-redis**默认引入了对Lettuce客户端依赖（Lettuce-core）**

**注意：**

​		如果两个客户端同时导入，根据顺序springboot自动配置会优先使用Lettuce

#### 3.2、Lecttuce客户端连接配置

​		spring-redis提供`RedisConnection`、`RedisConnectionFactory`来进行redis连接的抽象。开发者只需要注入`RedisConnectionFactory`对象，就能完成redis连接配置

- springBoot自动配置：

  springBoot基于`JedisConnectionConfiguration`和`LettuceConnectionConfiguration`来完成`RedisConnectionFactory`对象的注入,根据引入的redis客户端类型进行选择注入

  **LettuceConnectionFactory:**

  ​		`LettuceConnectionFactory`通过`RedisConfiguration`和`LettuceClientConfiguration`来进行构建：

  - RedisConfiguration：

    redis连接配置类，用于定义redis连接方式和连接属性

    spring-redis提供RedisConfiguration接口的三种实现类，对于不同的连接方式，根据spring配置文件定义的属性，来选择实现

    | RedisConfiguration实现类     | 连接方式 |
    | ---------------------------- | -------- |
    | RedisStandaloneConfiguration | 独立连接 |
    | RedisSentinelConfiguration   | 哨兵模式 |
    | RedisClusterConfiguration    | 集群模式 |

  - LettuceClientConfiguration：

    Lettuce客户端配置类，定义Lettuce客户端相关属性和连接池配置

    spring-redis提供`DefaultLettuceClientConfiguration`类来实现该接口

  **注意：**

  ​		spring-redis默认不开启连接池，并且不提供连接池依赖；如果使用连接池，则需要开发者手动导入commons-pool2依赖：

  ```xml
       <dependency>
              <groupId>org.apache.commons</groupId>
              <artifactId>commons-pool2</artifactId>
       </dependency>
  ```

- 属性配置：

  基于`RedisProperties`类,进行属性注入
  
  ```yml
    redis:
      host: localhost
      port: 6379
      password: yh9527
      database: 0
      lettuce:
        pool:
          min-idle: 0		#最小空闲连接数
          max-active: 8	#最大连接数
          max-idle: 8		#最大空间连接数
          max-wait: 3000  #空闲最大时间
    timeout: 5000    #连接超时时间
  ```

#### 3.3、redisTemplate

​		spring-redis通过redisTemplate来获取redis数据的操作类Operation，并分为两种类型：

1、key类型Operation：用于操作指定类型key

2、key绑定Operation：用于操作指定类型、指定key（适用于对一个key多次操作）

| 数据类型    | key类型Operation      | key绑定Operation     |
| ----------- | --------------------- | -------------------- |
| value       | ValueOperations       | BoundValueOperations |
| hash        | HashOperations        | BoundHashOperations  |
| list        | ListOperations        | BoundListOperations  |
| set         | SetOperations         | BoundSetOperations   |
| zset        | ZSetOperations        | BoundZSetOperations  |
| 地理空间    | GeoOperations         | BoundGeoOperations   |
| HyperLogLog | HyperLogLogOperations |                      |

BoundKeyOperations用于通用key绑定，不提供对value的操作

​		redisTemplate线程安全，每个`redisConnectionFactory`对于一个redisTemplate模板，为其提供redisConnection实例

​		redisTemplate实例化时，除了需要提供`redisConnectionFactory`外，还需要指定数据的序列化器，根据数据结构分为Value和Hash，因此需要指定redisTmeplate对于key、Value、HashKey、HashValue的序列化器

springBoot默认自动配置了StringRedisTemplate、RedisTemplate：

- StringRedisTempalte：

  ​		key、value使用StringRedisSerializer，默认UTF-8编码；**当value为String类型时使用**

- RedisTemplate：

  ​		key、value使用JdkSerializationRedisSerializer，需要数据实体类继承Serizlizable接口；**当value为复杂数据类型时使用**

- redisTemplate手动配置：

  ​		一般情况下，推荐key使用String序列化，value使用json序列化，spring-redis提供了Jackson2JsonRedisSerializer类，基于jackson包实现json序列化：

  ```java
  	@Bean
  	public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory, ObjectMapper objectMapper) {
  		Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
  		//redis中对象序列化方式使用jackson,并依赖于项目配置的objectMapper
  		jackson2JsonRedisSerializer.setObjectMapper(objectMapper);
  		RedisTemplate<String, Object> redisTemplate = new RedisTemplate<>();
  		redisTemplate.setConnectionFactory(factory);
  		//定义序列化方式
  		redisTemplate.setValueSerializer(jackson2JsonRedisSerializer);
  		redisTemplate.setHashValueSerializer(jackson2JsonRedisSerializer);
  		redisTemplate.setKeySerializer(new StringRedisSerializer());
  		redisTemplate.setHashKeySerializer(new StringRedisSerializer());
  		return redisTemplate;
  	}
  ```

- Jackson2JsonRedisSerializer序列化问题:

  ​		为了redis序列化器的通用性,一般开发者会将Jackson2JsonRedisSerializer中的javaType设置为Object.class,从而导致reids反序列化数据都为Object对象;此时jackson就会使用默认类型（LinkedHashMap）作为Object对象的原始数据类型,导致后续无法进行类型强行转化
  
  - 两种解决方法:
  
    1、通过objectMapper.convertValue()方法进行类型转化
  
    2、配置objectMapper，开启jackson多态类型处理，即在序列化时会记录当前对象及其属性的类型信息，从而在反序列化时能够直接使用该类型作为默认类型，实现强行转化
  
    ```java
    objectMapper.activateDefaultTyping(objectMapper.getPolymorphicTypeValidator(),
    				ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
    ```
  
    - objectMapper.getPolymorphicTypeValidator()为类型校验器，直接使用默认值
    - ObjectMapper.DefaultTyping用于指定记录哪些类型信息，包含如下类型：
  
    | 枚举                    | 作用                              |
    | ----------------------- | --------------------------------- |
    | JAVA_LANG_OBJECT        | Object类型                        |
    | OBJECT_AND_NON_CONCRETE | Object或接口/抽象类               |
    | NON_CONCRETE_AND_ARRAYS | Object、接口/抽象类或所有数组类型 |
    | NON_FINAL               | 所有非Final属性和自身             |
  
    - JsonTypeInfo.Id，用于指定类型信息描述方式，默认直接使用CLASS，即序列化时额外添加一个@class属性，来描述当前对象的全类名
    - JsonTypeInfo.As，用于指定类型信息描述在Json字符串中的结构位置，包含如下类型：
  
    | 枚举              | 作用                                                     |
    | ----------------- | -------------------------------------------------------- |
    | PROPERTY          | 作为内部属性存放                                         |
    | EXISTING_PROPERTY | 作为已存在的属性                                         |
    | EXTERNAL_PROPERTY | 作为外部属性存放（不能使用，直接抛出异常）               |
    | WRAPPER_OBJECT    | 作为一个包装对象（额外添加一个key-value层级）            |
    | WRAPPER_ARRAY     | 作为一个包装数组（额外添加一个数组层级，描述和数据同级） |

### 4、spring-context

