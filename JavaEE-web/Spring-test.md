# Spring-test

spring测试，相对于普通单元测试，提供了基于IOC容器的单元测试和多种模拟对象来提供测试运行环境，提高单元测试速度，通过**spring-test**包来实现

### springTest容器上下文管理

springTest会对spring的ApplicationConext、WebApplicationContext实例进行缓存，即对spring容器父、子上下文进行缓存和管理，有效避免每次测试时，重复对spring容器中的Bean进行实例化

#### 使用注解配置spring容器上下文：

- @ContextConfiguration注解，用于配置在测试类上，定义如何加载和配置spring容器，提供两种方法：locations（指定XML文件）、classes（指定注解配置类）

- @WebApplicationConfiguration注解，用于配置在测试类上，在@ContextConfiguration注解的基础上，定义当前spring容器需要加载WebApplicationContext，即基于web项目的spring容器；同时定义Web项目的资源加载路径：

  默认为"src/main/webapp"，一般情况下不需要进行重新指定

- @ContextHierarchy注解，用于配置在测试类上，定义ApplicationContext实例的层次结构；实际使用场景就是分别定义ApplicationConext、WebApplicationContext的配置

- @DirtiesContext，定义在方法或类上，控制清除spring容器上下文的缓存；通过classMode进行缓存清除时间点：测试类执行前后，测试方法执行前后；使用场景为：有些测试方法中，可能存在对ApplicationContext进行修改操作，从而隐式导致之后的测试被影响（如修改某个Bean单例状态），因此就需要重新创建容器

#### 整合测试框架：

​	springTest可以与JUnit、TestNG等任何单元测试框架整合，来编写单元测试。以整合JUnit4为例：

通过@RunWith(SpringJUnit4ClassRunner.class)或者**@RunWith(SpringRunner.class)简写变体**实现对JUnit4的完成集成

```java
@RunWith(SpringRunner.class)//整合Juni框架
@ContextHierarchy({ //配置spring容器上下文（默认只加载ApplicationConext）
		@ContextConfiguration(classes = springTest.class),
		@ContextConfiguration(classes = MvcConfig.class) })
@WebAppConfiguration //加载WebAppConfiguration
@DirtiesContext(classMode=ClassMode.AFTER_CLASS)//配置容器上下文销毁时间
public class SpringTest {
    
}    
```

- spring2.2版本后，将JUnit5作为单元测试默认包，因此整合方式也有变化：

  **@RunWith(SpringRunner.class)**直接替换为**@ExtendWith(SpringExtension.class)**

### springTest事务管理：

springTest支持使用@Transactional注解，来进行事务管理。但默认情况下，事务在测试完成后会自动回滚，即不会影响数据库中的数据，我们可以通过@Commit或@Rollback（false），但它们都会导致**事务无法进行异常回滚**

当service层方法进行事务管理，然后在测试方法中调用时，**根据spring事务的传播行为，测试方法的事务和service层方法的事务不会相互影响**，从而可以通过编写service类的方式，来实现在单元测试中，事务的异常回滚

### springTest测试监听：

​		springTest框架核心包括TestContext、TestContextManager和TestExecutionListener三个对象：

- TestContext

  ​	封装运行测试的上下文（与实际整合的单元测试框架无关），为测试提供spring容器上下文管理和缓存支持

- TestContextManager

  ​	是实现springTest框架的核心入口，负责管理所有TestContext事件，从而触发相应TestExecutionListener

- TestExecutionListener

  ​	实现springTest功能的核心类，springTest提供一系列默认Listener：

  | 监听器                                                       | 作用                                       |
  | ------------------------------------------------------------ | ------------------------------------------ |
  | ServletTestExecutionListener                                 | 配置Servlet API模拟 WebApplicationContext  |
  | DirtiesContextBeforeModesTestExecutionListener、DirtiesContextTestExecutionListener | 处理@DirtiesContext注解                    |
  | DependencyInjectionTestExecutionListener                     | 处理@Autowried注解                         |
  | TransactionalTestExecutionListener                           | 处理@Transactional注解                     |
  | SqlScriptsTestExecutionListener                              | 处理@Sql注解                               |
  | EventPublishingTestExecutionListener                         | 将测试执行事件发布到测试 ApplicationContex |

### springMVC测试：

springTest提供对Servlet API的环境模拟对象，实现不运行Servlet容器的情况下，提供完整的springMVC运行环境：

```java
@RunWith(SpringRunner.class)
@ContextHierarchy({ 
		@ContextConfiguration(classes = springTest.class),
		@ContextConfiguration(classes = MvcConfig.class) })
@WebAppConfiguration
public class MvcTest {
	 
	@Autowired
	WebApplicationContext context;
	 
	MockMvc mockMvc;

	@Before
	public void initMokcMvc() {
		mockMvc = MockMvcBuilders.webAppContextSetup(context).build();
	}
```

然后在测试中，使用MockMvc来进行Servlet API操作