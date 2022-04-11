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

### 3、spring-context

​		spring-context为springFramework核心包之一，除了实现IOC容器的核心功能外，还集成了一些JAVAEE开发常用功能：

- JMS（Java消息服务）
- 电子邮件
- 任务调度
- 缓存抽象

**注意：spring-context完整功能还依赖spring-context-support包**

#### 3.1、电子邮件

##### 3.1.1、电子邮件协议

​		电子邮件的发送与接收基于电子邮件协议来完成，目前共有三种，都是TPC/IP协议簇中的应用层协议：

- SMTP：

  ​		简单邮件传输协议，建立在FTP文件传输服务之上，定义了邮件传送到目的地的规则，帮助邮件找到目的地；SMTP服务器就是用于发送或转换电子邮件；默认使用TCP端口25

- POP3：

  ​		邮局协议版本3，定义了客户端接收电子邮件的规则，POP3服务器保存邮件，并给客户端登录，离线下载自己的电子邮件；默认使用TCP端口53

- IMTP：

  ​		交互邮件访问协议，和POP3类似，用于客户端接收电子邮件，但IMTP可以将客户端和邮件服务器双向通信，不仅支持电子邮件的离线下载，还可以直接在服务器上操作邮件；默认使用TCP端口143

##### 3.1.2、电子邮件发送接收过程

​		通过SMTP-POP3/IMTP服务器，两个客户端就能完成电子邮件的通信，过程如下：

1、用户A连接登录SMTP服务器，编写邮件、设置邮件目的地

2、SMTP服务器基于SMTP协议进行邮件发送

3、根据目的地，找到POP3/IMTP服务器，进行邮件接收

4、用户B连接登录POP3/IMTP服务器，获取自己的邮件，进行离线下载（IMTP则支持在线管理）

##### 3.1.3、JavaMail使用

​		spring对于电子邮件的支持，基于JAVAEE规范JSR-919的实现JavaMail，因此需要额外引入依赖：

```xml
<dependency>
    <groupId>com.sun.mail</groupId>
    <artifactId>jakarta.mail</artifactId>
    <version>1.6.5</version>
</dependency>
```

​		由于版权问题，在JAVAEE8后，JavaMail改为jakarta.mail代替，但本质上是一个项目；目前spring5只支持1.6.x版本，不支持最新的2.0

​		spring-context提供JavaMailSenderImpl类，简化封装邮件发送功能

- SimpleMailMessage

  SimpleMailMessage简单消息邮件，提供如下常用属性：

  | 属性     | 作用                 |
  | -------- | -------------------- |
  | subject  | 邮件主题             |
  | text     | 邮件内容             |
  | from     | 邮件发送人           |
  | to       | 邮件目的地，可以多个 |
  | sentDate | 发送时间             |
  | replyTo  | 回复人               |
  |          |                      |

  ```java
  public void sendSimpleMail() {
      	JavaMailSenderImpl sender = new JavaMailSenderImpl();
  		// 简单消息邮件
  		SimpleMailMessage simpleMailMessage = new SimpleMailMessage();
  		simpleMailMessage.setSubject("TEST");
  		simpleMailMessage.setText("今天星期几");
  		simpleMailMessage.setTo("yuanhuanpeipei@gmail.com");
  		simpleMailMessage.setFrom("756116832@qq.com");
  
  		//发送简单消息邮件
  		mailSender.send(simpleMailMessage);
  	}
  ```

- MimeMessage、MimeMessageHelper

  MimeMessage复杂消息邮件，MimeMessageHelper用于配置MimeMessage；MimeMessage额外提供Attachment（附件）功能

  ```java
  	public  void sendMimeMail() throws MessagingException {
  		JavaMailSenderImpl sender = new JavaMailSenderImpl();
          //复杂消息邮件
  		MimeMessage mimeMessage = sender.createMimeMessage();
          //true表示邮件支持附件
  		MimeMessageHelper mimeMessageHelper = new MimeMessageHelper(mimeMessage, true);
  		mimeMessageHelper.setSubject("TEST");
  		mimeMessageHelper.setText("今天星期几");
  		mimeMessageHelper.setTo("yuanhuanpeipei@gmail.com");
  		mimeMessageHelper.setFrom("756116832@qq.com");
  		mailSender.send(mimeMessage);
  	}
  ```

##### 3.1.4、springBoot自动配置

​		springBoot提供了MailProperties属性类，实现通过配置文件设置IOC容器中自动配置的JavaMailSenderImpl实例属性

​		@ConditionalOnProperty(prefix = "spring.mail", name = "host")对JavaMailSenderImpl自动配置添加了约束条件，必须在配置文件中定义spring.mail.host属性

​		mail功能常用配置：

```yaml
  #mail
spring: 
  mail:
    protocol: smtp
    host: smtp.qq.com
    username: 756116832@qq.com
    #邮箱授权码
    password:
    #开启ssl安全协议,此时SMTP服务器端口将为465
    properties:
      mail:
        smtp:
          ssl:
            enable: true
```

##### 3.1.5、多邮件发送人配置

​		SMTP服务商，对于每个邮箱账号都会限制每日发送邮件数，因此在实际商用环境中，需要配置多个邮件发送账号，轮询使用：

​		此时就需要自定义创建JavaMailSenderImpl实例：

```java
@Configuration
public class EmailUtil extends JavaMailSenderImpl {

	// 保存多个用户名和密码的队列（spring.mail.username=、
	// spring.mail.password=中定义多个客户端账号、密码，使用","分隔）
	private ArrayList<String> usernameList;
	private ArrayList<String> passwordList;
	// 轮询标识
	private int currentMailId = 0;


	public EmailUtil(MailProperties properties){
		applyProperties(properties);
		//获取所有账号
		usernameList=new ArrayList<>(Arrays.asList(properties.getUsername().split(",")));
		passwordList=new ArrayList<>(Arrays.asList(properties.getPassword().split(",")));
		if (usernameList.size()!=passwordList.size()){
			throw new BusinessException("EmailUtil创建失败,请检查相关配置");
		}
	}

	@Override
	protected void doSend(MimeMessage[] mimeMessages, Object[] originalMessages) throws MailException {
		int size = usernameList.size();
		//轮询
		currentMailId = (currentMailId + 1) % size;
		this.setUsername(usernameList.get(currentMailId));
		this.setPassword(passwordList.get(currentMailId));
		this.doSend(mimeMessages, originalMessages);
	}

	@Override
	public String getUsername() {
		return usernameList.get(currentMailId);
	}

	private void applyProperties(MailProperties properties) {
		this.setHost(properties.getHost());
		if (properties.getPort() != null) {
			this.setPort(properties.getPort());
		}
		this.setProtocol(properties.getProtocol());
		if (properties.getDefaultEncoding() != null) {
			this.setDefaultEncoding(properties.getDefaultEncoding().name());
		}
		if (!properties.getProperties().isEmpty()) {
			this.setJavaMailProperties(asProperties(properties.getProperties()));
		}
	}

	private Properties asProperties(Map<String, String> source) {
		Properties properties = new Properties();
		properties.putAll(source);
		return properties;
	}

}
```

#### 3.2、任务调度

​		spring使用`TaskExecutor`、`TaskScheduler` 接口，为任务的异步执行和调度提供抽象；支持对线程池、CommonJ规范的实现，同时可以体现出JDK各版本的差异性

##### 3.2.1、TaskExecutor

​		TaskExecutor继承java.util.concurrent.Executor接口，本质上等同于该接口，提供了对线程池的抽象。spring内置了许多TaskExecutor接口的实现：

- SyncTaskExecutor：

  ​		使用当前线程，同步执行任务，一般用于测试执行简单任务

- SimpleAsyncTaskExecutor：

  ​		每次执行任务都会启动一个新线程，可以设置最大线程数，如果达到最大值，则进行等待

- ThreadPoolTaskExecutor：

  ​		对java.util.concurrent.ExecutorService类的封装，作为Bean进行参数配置，适合大部分情况

- ConcurrentTaskExecutor：

  ​		对java.util.concurrent.Executor接口实现类的封装，作为不同类型的`java.util.concurrent.Executor`的适配器，默认使用 java.util.concurrent.ThreadPoolExecutor

- WorkManagerTaskExecutor：

  ​		用于集成CommonJ线程池

- DefaultManagedTaskExecutor：

  ​		使用JNDI获取ManagedExecutorService

##### 3.2.2、TaskScheduler

​		spring3.0引入了TaskScheduler接口，用于定义任务调度抽象；通过Duration（周期）、Date（时间）、Trigger（触发器）三种形式进行调度

​		Trigger接口定义方法通过TriggerContext来获取任务执行时间，而TriggerContext接口方法定义了任务将要执行时间、最后一次执行和完成时间，spring提供了对Trigger接口的两种实现：

- CronTrigger：通过cron表达式来定义任务调度规则

- PeriodicTrigger：通过定义固定周期、固定初始延迟、是否固定速率三个属性来定义任务调度规则

  TaskScheduler和TaskExecutor类似，提供了一系列对TaskExecutor的变种，在执行任务功能的基础上，额外提供任务调度功能，通常使用`ThreadPoolTaskScheduler`

##### 3.2.3、任务调度注解

​		通过在配置类上添加@EnableAsync、@EnableScheduling注解，来开启spring任务调度功能：

```java
@EnableScheduling
@EnableAsync
@Configuration
public class TaskConfig {

```

​		在方法上使用@Async或@Scheduling注解，来进行任务的声明

- @Async

  1、@Async基于AOP动态代理实现，因此在本地调用@Async方法时，并不会进行异步处理

  2、@Async用于异步执行任务，因此方法返回值应该为void

  3、@Async提供value，通过BeanName获取所使用TaskExecutor实例，作为异步任务使用的线程池

  4、@Async和Bean的生命周期回调注解一起使用，并不会进行异步处理

- @Scheduling

  @Scheduling提供如下属性，来定义任务调度规则：

  | 属性         | 作用                                                         |
  | ------------ | ------------------------------------------------------------ |
  | cron         | cron表达式                                                   |
  | zone         | 时区，用于解析cron表达式，默认使用服务器时区                 |
  | fixedDelay   | 固定延迟，任务执行完成后，下次任务执行间隔时间               |
  | fixedRate    | 固定速率，任务开始执行后，下次任务执行间隔时间（任务执行完成时间大于间隔时间时，会导致单线程阻塞） |
  | initialDelay | 初始化延迟，项目启动后，第一次任务执行延迟时间               |

##### 3.2.4、SpringBoot自动配置

​		springBoot提供了TaskExecutor和TaskScheduler默认配置：

- TaskExecutor：

  ​		使用TreadPoolTaskExecutor实现类，具体属性值如下，并通过`spring.task.execution`进行配置：

  | 属性                            | 作用                                     | 默认值            |
  | ------------------------------- | ---------------------------------------- | ----------------- |
  | threadNamePrefix                | 线程名前缀                               | task-             |
  | shutdown.awaitTermination       | 线程池关闭时，是否等待任务完成           | false             |
  | shutdown.awaitTerminationPeriod | 线程池关闭时，允许等待任务完成的最长时间 |                   |
  | pool.queueCapacity              | 任务队列容量                             | Integer.MAX_VALUE |
  | pool.coreSize                   | 核心线程数                               | 8                 |
  | pool.maxSize                    | 最大线程数                               | Integer.MAX_VALUE |
  | pool.allowCoreThreadTimeout     | 允许核心线程池超时空闲（一直存活）       | true              |
  | pool.keepAlive                  | 空闲时间                                 | 60s               |

  ​		TreadPoolTaskExecutor内部默认使用AbortPolicy（中止策略）：线程池队列满后，直接拒绝新任务，抛出RejectedExecutionException异常

- TaskScheduler：

  ​		使用ThreadPoolTaskScheduler实现类，具体属性如下，并通过`spring.task.scheduling`进行配置：

  | 属性                            | 作用                                     | 默认值      |
  | ------------------------------- | ---------------------------------------- | ----------- |
  | threadNamePrefix                | 线程名前缀                               | scheduling- |
  | shutdown.awaitTermination       | 线程池关闭时，是否等待任务完成           | false       |
  | shutdown.awaitTerminationPeriod | 线程池关闭时，允许等待任务完成的最长时间 |             |
  | pool.size                       | 允许的最大线程数                         | 1           |

  ​		ThreadPoolTaskScheduler为固定大小的线程池，大小默认为1，所有任务调度都使用该线程池完成

注意：

- 当@Scheduling和@Async一起使用时，则任务将会将给异步线程池来完成，任务调度使用的线程池就不会发生阻塞，但也带来了一个问题：

  ​		所有任务调度执行时间都近似于0，导致fixedDelay设置失效；如果任务实际执行时间小于fixedDelay间隔的话，会导致一个任务同时在多个线程中执行

##### 3.2.5、taskExecutor异常管理

1、spring支持使用Future作为返回值，通过get方法，实现原线程获取异步线程中抛出的异常

2、对于void返回值，则可以通过配置AsyncUncaughtExceptionHandler异常处理器，来进行异常处理

```java
	@Bean
	public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
		return new AsyncUncaughtExceptionHandler() {
			@Override
			public void handleUncaughtException(Throwable ex, Method method, Object... params) {
				log.warn(method.getName()+"    "+ex.getMessage()+"    "+params.toString());
			}
		};
	}
```

##### 3.2.6、cron表达式

​		cron表达式由六个以空格分隔的时间/日期字段组成：

```java
 ┌────────────── 秒 (0-59)
 │ ┌───────────── 分钟 (0 - 59)
 │ │ ┌───────────── 小时 (0 - 23)
 │ │ │ ┌───────────── 一个月中的某天 (1 - 31)
 │ │ │ │ ┌───────────── 月（1-12）（或JAN-DEC）
 │ │ │ │ │ ┌───────────── 星期几（0 - 7,0 或 7 为星期日，或 MON-SUN）
 │ │ │ │ │ │ 
 │ │ │ │ │ │
 * * * * * *
```

并定义了如下规则：

- *表示任意范围
- ？在指定月份和星期时，由于月份和星期是冲突的，为了保证语义的正确性，其中一个指定后，如果另一个为任务范围，则应该使用？代替* （实际上* 和?没有区别）
- ，分割多个元素，表示并集
- -连接两个数字，表示闭区间范围
- / 指定在范围和*中的周期间隔
- **L表示在该月的最后一天、该月最后一个星期几（根据测试并不支持）**

**常用例子:**

| cron表达式      | 意义                     |
| --------------- | ------------------------ |
| 0 0 * * * *     | 每小时                   |
| */10 * * * * *  | 每10秒                   |
| 0 0  8-10 * * * | 每天8、9、10点           |
| 0 0 6,19 * * *  | 每天6、19点              |
| 0 0 0 L * *     | 每个月最后一天凌晨       |
| 0 0 0 L-3 * *   | 每个月倒数第三个凌晨     |
| 0 0 0 * * 5     | 每个星期五凌晨           |
| 0 0 0 * * 5L    | 每个月最后一个星期五凌晨 |
| 0 0 0 25 12 ？  | 每年12-25凌晨            |

spring提供对常用cron表达式的宏：**（根据测试并不支持）**

| 宏       | 意义                          |
| -------- | ----------------------------- |
| @yearly  | 每年1月1号凌晨（0 0 0 1 1 *） |
| @monthly | 每月1号凌晨（0 0 0 1 * *）    |
| @weekly  | 每周星期日凌晨（0 0 0 * * 0） |
| @daily   | 每天凌晨（0 0 0 * * *）       |
| @hourly  | 每小时（0 0 * * * *）         |

##### 3.2.7、定时任务和spring-websocket冲突

​		在项目引入spring-websocket模块，spring会默认创建一个TaskScheduler实例Bean（当使用SockJS时，才会初始化该实例，否则默认为null）；当开启定时任务后，spring就会使用IOC容器中的TaskScheduler，作为定时任务的线程池，此时就会报错：

​		为了避免spring定时任务使用spring-websocket模块中的TaskScheduler，就需要开发者手动创建TaskScheduler，此时spring会获取所有TaskScheduler实例，并排除为null的，之后通过@Primary确定最终候选人

#### 4.3、缓存抽象

​		spring提供了缓存抽象，对JAVA方法参数列表，对其返回值进行缓存处理

##### 3.3.1、缓存注解

​		spring缓存抽象提供了一组java注解，用于声明式缓存处理

注意：

- spring缓存注解的依赖于AOP动态代理，因此需要保证方法可见性为public

###### 3.3.1.1、@Cacheable

​		用于缓存填充，当方法被调用时，先检查缓存是否存在；不存在，则调用方法进行缓存填充；存在，则获取缓存作为返回值，无需调用方法

​		@Cacheable属性：

| 属性名              | 作用                                                         |
| ------------------- | ------------------------------------------------------------ |
| value（cacheNames） | 缓存名称，可以指定多个从而生成多份缓存                       |
| key                 | 缓存key，支持SpEL表达式                                      |
| keyGenerator        | 缓存key生成器，用于通过返回参数生成缓存key                   |
| cacheManager        | 缓存管理器，定义spring缓存的实现方式，管理缓存               |
| cacheResolver       | 缓存解析器，内部定义了的缓存管理器，通过缓存上下文，获取所操作的缓存 |
| condition           | 缓存条件，通过SpEL表达式指定参数需要满足的条件，满足才进行缓存处理 |

注意：

- key和keyGenerator不能同时指定，否则会抛出异常
- cacheManager和cacheResolver不能同时指定，否则会抛出异常；

###### 3.3.1.2、@CachePut

​		用于缓存更新，始终调用方法，并将结果放入缓存中；建议@CachePut不要和@Cacheable同时使用，前者会强制调用方法更新缓存，导致@Cacheable不能正常生效（当然，可以通过条件将两者分离，但不推荐）

###### 3.3.1.3、@CacheEvict

​		用于缓存驱逐，当方法被调用时，删除指定缓存；

注意：

- @CacheEvict额外提供allEntries属性，当为true时，则会将当前缓存中的所有key-value全部删除，此时key、keyGrenerator失效
- @CacheEvict额外提供beforeInvocation属性，当为true时，则缓存驱逐发生在方法调用前，此时即使方法调用发生异常，也会执行缓存驱逐；默认false，在方法调用成功后执行缓存驱逐

###### 3.3.1.4、@Caching

​		用于声明多个相同类型的缓存注解，实现同一个方法根据条件对多个不同缓存进行操作，比如：

```java
//同时驱逐primary、secondary两个缓存
@Caching(evict={@CacheEvit("primary"),@CacheEvict(cacheNames="secondary")})
public void remove(String xzqh)
```

###### 3.3.1.5、@CacheConfig

​		类级别注解，支持全局配置当前类下，所有缓存注解的value、keyGenerator、CacheManager、CacheResolver属性，并可以被局部配置替代

###### 3.3.1.6、@EnableCache

​		用于开启spring声明式缓存抽象功能

##### 3.3.2、keyGenerator

​		keyGrnerator用于根据参数列表自动生成缓存key，spring默认提供`SimpleKeyGenerator`来生成缓存key，其规则如下：

- 单个参数：new SimpleKey
- 一个参数：直接为参数本身
- 多个参数：new SimpleKey(params)

key生成时，则调用key对象的toString方法，对于SimpleKey.toString：

```java
	@Override
	public String toString() {
		return getClass().getSimpleName() + " [" + StringUtils.arrayToCommaDelimitedString(this.params) + "]";
	}
```

##### 3.3.3、CacheManager

​		spring-context提供多个对CacheManager的实现：

- 基于JDK concurrentMap，注册ConcurrentMapCacheManager实例
- 基于Ehcache，注册EhCacheCacheManager实例，需要额外导入ehcache库
- 基于Caffeine，注册CaffeineCacheManager实例，需要额外导入caffeine库
- 基于JSR-107缓存规范，注册jCacheManager实例，需要额外导入cache-api库

### 4、spring-redis

spring-redis用于提供spring程序配置访问redis的功能，简化开发者对redis的使用

- 引入maven依赖

```xml
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```

#### 4.1、redis客户端

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

#### 4.2、Lecttuce客户端连接配置

​		spring-redis提供`RedisConnection`、`RedisConnectionFactory`来进行redis连接的抽象。开发者只需要注入`RedisConnectionFactory`对象，就能完成redis连接配置

##### 4.2.1、springBoot自动配置：

​		springBoot基于`JedisConnectionConfiguration`和`LettuceConnectionConfiguration`来完成`RedisConnectionFactory`对象的注入；并根据引入的redis客户端类型来选择配置

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

#### 4.3、redisTemplate

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

#### 4.4、redis实现spring-cache抽象

​		spring-redis提供`RedisCacheManager`，来实现spring-cache抽象

##### 4.4.1、springBoot自动配置

​		springBoot基于RedisCacheConfiguration，来实现RedisCacheManager的Bean注入

RedisCacheManager的自动配置需要建立在RedisConnectionFactory被注入的基础上，通过配置好的RedisCacheConfiguration来创建RedisCacheManage

​		在CahceProperties配置文件中，通过spring.cache.redis进行RedisCacheConfiguration的配置：

| 属性            | 作用         | 默认值                    |
| --------------- | ------------ | ------------------------- |
| timeToLive      | 过期时间     | Duration.ZERO（永不过期） |
| cacheNullValues | 允许缓存null | true                      |
| keyPrefix       | key前缀      | 缓存名：：                |
| useKeyPrefix    | 启用key前缀  | true                      |

​		除此之外，RedisCacheConfiguration还需要额外配置redis序列化器：

| 属性                   | 作用              | 默认值         |
| ---------------------- | ----------------- | -------------- |
| keySerializationPair   | 缓存key序列化器   | String序列化器 |
| valueSerializationPair | 缓存value序列化器 | JDK序列化器    |

##### 4.4.2、redisCacheManage手动配置

```java
@Bean
public CacheManager cacheManager(RedisConnectionFactory factory) {
    	//配置Json序列化器所使用的ObjectMapper
		ObjectMapper objectMapper = new ObjectMapper();
		JacksonConfig.configureDate(objectMapper);
		JacksonConfig.registerJavaTimeModule(objectMapper);
		//添加额外属性@Class,实现复杂对象的序列化
		objectMapper.activateDefaultTyping(objectMapper.getPolymorphicTypeValidator(), ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.WRAPPER_OBJECT);
		//反序列化时,允许未知属性
		objectMapper.configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES,false);

		Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
		jackson2JsonRedisSerializer.setObjectMapper(objectMapper);	
    //创建一个默认配置的ReidsCacheConfiguration，修改其属性
		RedisCacheConfiguration cacheConfig = RedisCacheConfiguration.defaultCacheConfig();
    	//设置缓存过期时间（1天）
		cacheConfig.entryTtl(Duration.ofDays(1))
        		//不允许缓存有空值
				.disableCachingNullValues()
				//设置缓存管理器key-value数据序列化策略（key为String，value为json）
           	                               	.serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))		.serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new Jackson2JsonRedisSerializer(Object.class)));
    
		//通过RedisConnectionFactory、RedisCacheConfiguration来创建RedisCacheManager
		return RedisCacheManager.builder(factory).cacheDefaults(cacheConfig).build();
}
```

### 5、spring-web

​		springboot对整个web模块进行了一系列自动配置和优化，来方便快速构建web应用

- 引入依赖

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  ```

springBoot会额外引入如下模块:

- spring-boot-starter-json
- spring-boot-starter-tomcat

#### 5.1、springboot自动配置

##### 5.1.1、HttpMessageConverter

​		springBoot默认提供了两个HttpMessageConverter实例：

- StringHttpMessageConverter
- MappingJackson2HttpMessageConverter，默认使用依赖使用Jackson作为json库，通过替换Json库，springBoot额外支持Jsonb、Gson的自动配置

##### 5.1.2、@JsonComponent

​		springBoot提供@JsonComponent注解，通过使用内部类来定义一个类的Json序列化器和反序列化器，并注入到IOC容器中：

```java
@JsonComponent
@Data
@AllArgsConstructor
@NoArgsConstructor
public class JsonObject {

	private String name;

	private Integer age;

	public static class Serializer extends JsonSerializer<JsonObject> {
		@Override
		public void serialize(JsonObject value, JsonGenerator jgen, SerializerProvider serializers) throws IOException {
			jgen.writeStartObject();
			jgen.writeStringField("name", value.getName());
			jgen.writeNumberField("age", value.getAge());
			jgen.writeEndObject();
		}
	}

	public static class Deserializer extends JsonDeserializer<JsonObject> {
		@Override
		public JsonObject deserialize(JsonParser jsonParser, DeserializationContext ctxt)
				throws IOException, JsonProcessingException {
			ObjectCodec codec = jsonParser.getCodec();
			JsonNode tree = codec.readTree(jsonParser);
			String name = tree.get("name").textValue();
			int age = tree.get("age").intValue();
			return new JsonObject(name, age);
		}
	}
}
```

##### 5.1.3、静态内容

​		springboot默认按照顺序，从而如下类路径下获取静态资源：/static、/public、/resources、/META-INF/resources；并且提供spring.mvc.resources.static-locations属性来自定义静态资源路径

​		开发者可以通过WebMvcConfigurer.addResourceHandlers方法，自定义静态资源处理器，来修改此行为；也可以通过spring.mvc.static-path-pattern来修改静态资源匹配路径

​		spring.mvc.resources.static-locations和spring.mvc.static-path-pattern区别：

- 前者是定义静态资源目录，从而该目录下匹配静态资源
- 否则是定义静态资源路径匹配的url格式，只有url满足该模式，才能进行静态资源获取；同时在匹配静态资源时，会将spring.mvc.static-path-pattern作为前缀去除

实际场景：

spring.mvc.resources.static-locations="/static"，spring.mvc.static-path-pattern="/resource/**"

只有当请求为xxx/resource/index.htm时，才会从而static目录下匹配index.html

##### 5.1.4、错误处理

​		springboot提供/error来处理所有错误的映射，并提供了一个全局的错误页面，以HTML格式展示响应的json数据

​		可以在/static/error/目录下，自定义错误页面，并以对应的响应码作为HTML文件名

#### 5.2、嵌入式Servlet容器

​		springboot支持嵌入式Servlet容器，包含：Tomcat、Jetty、Undertow服务器，默认监听http请求的8080端口；所有注册到IOC容器中的Servlet组件，都会交给嵌入式Servlet容器

##### 5.2.1、初始化ServletContext

​		springMVC默认使用外部Servlet容器时，通过执行javax.servlet.ServletContainerInitializer来调用`org.springframework.web.WebApplicationInitializer`接口onStart方法，来初始化ServletContext；

​		而springBoot嵌入式Servlet容器是通过`org.springframework.boot.web.servlet.ServletContextInitializer`接口来单独实现

##### 5.2.2、嵌入式Servlet容器配置

​		springBoot通过配置文件，提供了如下嵌入式Servlet容器配置：

- 网络设置：server.port（Http请求监听端口）、server.address（Http请求监听的主机IP）
- 会话设置：server.servlet.session.persistent（会话时候持久）、server.servlet.session.timeout（会话超时时间）、会话数据位置（server.servlet.session.store-dir）、会话cookie配置（server.servlet.session.cookie.*）
- 错误页面位置：server.error.path
- SSL
- HTTP压缩

注意：

- Undertow服务器不支持JSP

### 6、spring-test

### 7、spring-boot-actuator

​		springboot额外提供actuator模块（执行器），帮助开发者在生产环境下，通过JMX、HTTP来管理监控springboot应用程序

- 引入依赖

  ```xml
  <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-actuator</artifactId>
  </dependency>
  ```

#### 7.1、执行器端点

​		执行器端点可以使开发者监视并交互springboot应用程序；在springBoot中内置了许多端点，将其启用、公开，来实现数据监控

​		在使用HTTP方式暴露端口时，通过/actuator/端点id 来进行访问

执行器端点：

| 端点           | 描述                                     |
| -------------- | ---------------------------------------- |
| auditevents    | 获取所有已触发的审计事件的报告           |
| beans          | springIOC容器中的所有bean                |
| conditions     | 获取所有自动配置条件生效或不通过的报告   |
| configprops    | 获取所有配置属性及其值                   |
| env            | 公开spring应用中所有属性                 |
| health         | 获取spring应用的健康状态                 |
| heapdump       | 下载堆的dump文件                         |
| httpTrace      | 获取最近100Http请求的跟踪结果            |
| info           | 获取开发者所定义的该spring应用信息       |
| loggers        | 显示和修改应用中的日志配置               |
| mappings       | 获取所有HTTP映射以及对应处理器方法的报告 |
| metrics        | 获取当前应用的指标信息                   |
| scheduledtasks | 获取当前应用的所有调度任务               |
| threaddump     | 下载线程的dump文件                       |
| shutdown       | 手动正常关闭spring应用                   |

##### 7.1.1、启用和暴露端点

​		springboot默认情况下，启用除了shutdown外的所有端点，但由于端点可能包含敏感信息，因此springboot默认基于HTTP方式只会暴露health、info端点，其他端点则使用JMX方式暴露

​		通过management.endpoints.web.exposure.include可以实现使用HTTP方式暴露指定端点，来替换默认值

注意：

​		当使用springSecurity来保证springBoot应用的网络安全时，所有以HTTP方式暴露的端点，都会被springSecurity保护，此时就需要自定义SecurityFilterChain ，在请求映射到任意端点时，运行任意身份访问：

```java
@Configuration(proxyBeanMethods = false)
public class MySecurityConfiguration {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.requestMatcher(EndpointRequest.toAnyEndpoint())
                .authorizeRequests((requests) -> requests.anyRequest().permitAll());
        return http.build();
    }
}
```

##### 7.1.2、发现页面

​		默认，在端点发现页面/actuator，会返回所有http公开的断点信息

management.endpoints.web.discovery.enabled=false用于禁用发现页面

##### 7.1.3、CORS支持

​		基于HTTP暴露的端点访问，同样支持CORS，通过如下属性来配置：

```java
management.endpoints.web.cors.allowed-origins=https://example.com
management.endpoints.web.cors.allowed-methods=GET,POST
```

##### 7.1.4、自定义端点

- 端点类注解：

  需要额外搭配@Component注解才能生效

  | 注解         | 作用                        |
  | ------------ | --------------------------- |
  | @Endpoint    | 可以同时被JMX、HTTP方式暴露 |
  | @JmxEndpoint | 只能被JMX方式暴露           |
  | @WebEndpoint | 只能被HTTP方式暴露          |

- 操作方法注解：

  在端点类中声明方法，来定义端点操作，根据操作类型，分为三类：

  | 注解             | 作用                                     | HTTP方法 |
  | ---------------- | ---------------------------------------- | -------- |
  | @ReadOperation   | 读取数据，将方法返回值作为数据返回       | GET      |
  | @WriteOperation  | 修改数据，使用JSON格式传参，作为方法参数 | POST     |
  | @DeleteOperation | 删除数据，使用JSON格式传参，作为方法参数 | DELETE   |

  ```java
  @Component
  @Endpoint(id ="test")
  public class MyEndpoint {
  
  	@ReadOperation
  	public String getData(){
  		return "hello world";
  	}
  }
  ```

##### 7.1.5、健康信息（health）

​		健康信息通过management.endpoint.health.show-details进行配置，来定义是否显示细节，提供如下三种

- never：从不显示细节

- when-authorized：向指定授权用户显示，management.endpoint.health.roles来配置授权的角色

- always：先所有用户显示细节

  springboot提供了多个健康指标，通过management.health.key.enabled来指定key进行启用，默认开启的：

  | key       | 描述                               |
  | --------- | ---------------------------------- |
  | db        | 检查是否可以获得可连接的datasource |
  | diskspace | 检查磁盘空间是否不足               |
  | redis     | 检查redis服务器是否启动            |
  | mail      | 检查mail服务器是否启动             |

###### 7.1.5.1、自定义健康信息

​		通过注册实现HealthIndicator接口的Bean，来自定义健康信息

```java
@Component
public class MyHealthIndicator  implements HealthIndicator {

	@Override
	public Health health() {
		return Health.down().withDetail("Error Code","500").build();
	}

}
```

​		此时在监控信息中，components会包含`my`子监控信息

##### 7.1.6、应用信息（info）

​		springboot自动配置了4类info信息：

| id    | 描述                                             |
| ----- | ------------------------------------------------ |
| build | 构建信息，获取META-INF/build-info.propreties资源 |
| env   | 环境变量，获取在配置文件中所有info.xx信息        |
| git   | git信息，获取git.properties资源                  |
| java  | java运行信息（被删除）                           |

​		通过management.info.<id>.enabled来公开指定类型信息，默认会开启env，java；build和git则根据资源是否存在来决定

###### 7.1.6.1、生成构建信息文件

​		基于spring-boot-maven-plugin插件，添加execution配置，在编译、打包文件中生成build-info.propreties文件

```XML
<build>
        <plugins>
            <!--springboot打包插件,用于maven生成可运行的jar包(如果没有,则jar无法运行)-->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.3.5.RELEASE</version>
                <!-- 在maven构建时,生成build-info的构建信息-->
                <executions>
                    <execution>
                        <goals>
                            <goal>build-info</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```

#### 7.2、HTTP端点管理

springboot提供如下属性，来自定义HTTP端点访问配置：

| 属性                               | 默认值      | 作用     |
| ---------------------------------- | ----------- | -------- |
| management.endpoints.web.base-path | /actuator   | 访问前缀 |
| management.server.port             | server.port | 访问端口 |
| management.server.address          | 127.0.0.1   | 监听ip   |

#### 7.3、JMX端点管理和使用











