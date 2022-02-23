SpringFramework

## 1、spring概念

​		spring是一个轻量级的开源JAVAEE框架，用于解决企业应用开发的复杂性。有两个核心部分：

- IOC：控制反转，将创建对象的过程交给spring进行管理
- AOP：面向切面编程，通过动态代理，将一些模块化代码分离，不写入源代码

### 1.1、spring特点：

- 方便解耦；自动依赖注入，从而简化开发
- 面向切面编程，使开发者专注于业务逻辑代码的编写
- 支持对各种框架的集成
- 支持各种JAVAEE规范，并提供更加简化的API

### 1.2、spring模块划分：

spring是一个轻量化框架，通过多个模块来给开发者提供自己应用程序所需要的功能，实现JAVAEE平台的所有规范。spring主要分成六个模块：

- IOC容器
- AOP
- 数据访问与集成
- Web和远程调用
- spring测试
- Instrumentation和Messaging（使用比较少）

![](..\Typora图片\spring模块架构.png)

### 1.3、spring核心容器模块：

核心容器模块就是spring框架的基础，用于整个spring容器的搭建和使用，也是整合所有其他模块的基础，它包括：

- Beans：访问配置文件、管理创建bean、IOC容器的依赖注入
- Core：spring基本核心工具类，其他spring组件的基本核心
- Context：springIOC功能上的扩展服务：邮件服务、任务调度、JNDI定位、缓存、远程访问、视图层框架的封装等
- SpEL：spring表达式语言，spring3.0中提出，使用#{}作为定界符，用于配置Bean对象的属性注入

对应四个spring依赖包：spring-beans、spring-core、spring-context、spring-expression

## 2、IOC容器

​		IOC容器，即spring容器，实现spring的依赖注入，控制反转，整个容器的创建和使用依赖于spring的核心容器模块，因此只需要导入基础的四个依赖包

### 2.1、基本概念：

1、IOC容器设计和目的：

​		将对象创建和对象调用的过程，交给spring进行管理；从而降低对象之间的耦合

2、IOC容器的底层技术：

​			xml解析、工厂模式、反射

3、IOC容器底层原理：

- 在spring的xml配置文件中，配置IOC容器需要创建管理的对象	

- spring通过对xml解析，获取需要创建对象的信息、实例化方式和某些特定行为

- 通过反射技术，使用xml解析得到的信息创建该实例；

- 最后使用工厂模式，来提供获取该实例的方法

  因此实际上，IOC容器底层就是一个对象工厂，使用java反射技术，读取xml配置文件，提供实例化对象的方法

### 2.2、IOC容器入门：

- 整个IOC容器的搭建，需要使用spring配置文件来完成：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="http://www.springframework.org/schema/beans    
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd 
                        http://www.springframework.org/schema/context    
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd">
    
    <!-- 配置User对象 -->                    
    <bean id="user" class="com.yh.User"></bean>                                           
</beans>  
```

**根据需要使用spring模块的标签，来引入对应的XML Schema文件，定义springXML配置文件的文档结构**

- 获取IOC容器上下文，然后依赖注入获取IOC容器中的实例

```java
	public static void main(String[] args) {
		//获取IOC容器上下文
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
		User user = context.getBean("user", User.class);
	}
```

- 手写IOC容器工厂（简单模拟实现IOC容器）

```java
public class IocFactory {

	private static HashMap<String, Object> singleBeanMap=new HashMap<String, Object>(10);
	
    //注入Bean，省略了xml解析获取bean全类名的过程
	public static void setBean(String beanName,String className) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
		Class clazz = Class.forName(className);
		boolean contains = singleBeanMap.keySet().contains(beanName);
		//保证bean对象的单例创建
		if (!contains) {
			synchronized (IocFactory.class) {
				if (!contains) {
					Object newInstance = clazz.newInstance();
					singleBeanMap.put(beanName, newInstance);
				}
			}
		}
	}
	
    //获取IOC容器中的Bean
	public static <T> T getBean(String beanName,Class<T> clazz) {
		if (!singleBeanMap.keySet().contains(beanName)) {
			System.out.println("IOC容器中不存在该"+beanName+"Bean");
		}
		T bean = (T)singleBeanMap.get(beanName);
		return bean;
	}
}
```

### 2.3、核心接口：

spring容器的实现基于BeanFactory和ApplicationContext两个接口

#### 2.3.1、BeanFactory

BeanFactory是访问IOC容器的根接口，其实现类为`DefaultListableBeanFactory`,其层次结构图为：

![](../Typora图片/BeanFactory实现类UML图.jpg)

相关主要接口作用:

- BeanFactory：实现对Bean的访问
- HierarchicalBeanFactory：继承BeanFactory接口，实现IOC容器父子关系结构
- AutowireCapableBeanFactory：继承BeanFactory接口，实现Bean自动装配功能
- ListableBeanFactory：继承BeanFactory接口，实现对Bean集合的操作
- ConfigurableBeanFactory：继承HierarchicalBeanFactory接口，额外实现SingletonBeanRegistry接口，提供单例bean的注册功能
- BeanDefinitionRegistry：实现Bean的注册
- FactoryBeanRegistrySupport ：实现FactoryBean的注册

**BeanFactory一般用于spring内部对IOC容器的管理，其实例对象使用延迟初始化策略，当程序需要访问IOC容器时，才会进行初始化；**

#### 2.3.2、ApplicationContext

ApplicationContext是BeanFactory的子接口，通过spring-context包对其进行企业化应用功能扩展，其实现类根据应用场景（是否为web应用、web实现方式）和配置实现（xml和注解）分为多种，最基础的实现类为`GenericApplicationContext`，其层次结构图为：

![](../Typora图片/ApplicationContext实现类UML图.jpg)

额外实现接口有：

- EnvironmentCapable：用于spring获取环境变量（如System配置、Servlet配置）
- MessageSource：支持国际化信息获取
- ResourceLoader：加载各种类型资源
- ApplicationEventPubisher：实现事件发布与监听功能
- LifeCycle：实现开启或关闭Bean的生命周期
- ConfigurableApplicationContext：继承ApplicationContext接口，提供一系列Application属性的配置和容器刷新、关闭功能

**ApplicationContext面向使用spring的开发者，是spring应用上下文，提供IOC容器的高级功能；默认在应用启动时，直接初始化**

GennericApplicationContext的最终实现：

- 搭配AnnotationConfigRegistry接口，实现以java注解方式配置容器

- 搭配ConfigurableWebApplicationContext接口转为webApplciationContext实例

### 2.4、IOC容器配置（XML）：

Spring支持两种IOC容器的配置方式：

- 基于XML配置

- 基于java代码和java注解配置（spring3.0引入）

#### 2.4.1、Bean注册

IOC容器提供两种类型Bean的创建，Bean和FactoryBean

##### 2.4.1.1、Bean标签

通过构造方法来创建Bean实例，默认使用无参构造方法

```xml
<bean id="user" class="com.yh.User" name="userA;userB"></bean>
```

**bean标签基本属性：**

| 属性  | 描述                                                        |
| ----- | ----------------------------------------------------------- |
| id    | bean的唯一标识符                                            |
| name  | bean的别名，可以指定多个别名（spring5后，name别名不能重复） |
| class | bean的全类名，必填                                          |

注意：

- id的默认值：

  1、id和name都不指定时，IOC容器默认使用class作为id

  2、只指定name时，IOC容器使用第一个nane作为id

- id和name都可以作为getBean方法参数，来获取Bean，一般推荐使用id

##### 2.4.1.2、Property标签：

property标签用于bean属性注入，但需要保证bean提供属性的set方法

```xml
<bean id="user" class="com.yh.User">
	<property name="name" vlaue"yh"/>
    <property name="gread" ref="gread">
</bean>
```

**property标签属性：**

| 属性  | 描述                                                      |
| ----- | --------------------------------------------------------- |
| name  | 属性名，支持级联注入，但需要保证级联对象已被注入          |
| value | 属性值，只支持基础数据类型                                |
| ref   | 属性值，用于将IOC容器的bean作为属性进行注入，通过id来引入 |
| idref | 属性值，代替ref，在xml中校验指定bean是否在IOC容器中存在   |

**property标签嵌套bean标签：**

property标签嵌套bean标签，来直接创建引用数据类型属性的值，但该bean只作用于上层bean，无法从IOC容器中获取

```xml
<property name="gread" >
    <bean  class="com.yh.Gread">
		 <property name="id" value="1" />
		 <property name="math" value="98" />
		 <property name="english" value="100" />
    </bean>
</property>
```

spring提供一系列常用集合、特殊值的标签，方便属性注入：

```xml
<property name="name">
	<null/>
</property>

<property name="arrayId">
	<array>
		<value>111</value>
	</array>
</property>

<property name="listId">
	<list>
		<value>11</value>
	</list>
</property>

<property name="mapId">
 	<map> 
 		<entry key="1">
            <value>abc</value>
        </entry>
	</map>
</property>  

<property name="listId" >
 	<map> 
 		<entry key="1">
            <ref bean="user"/>
        </entry>
	</map>
</property>  

<property name="propId">
    <props>
         <prop key="name">yh</prop>
    </props>
</property>
```

##### 2.4.1.3、Constructor-arg标签

通过有参构造方法创建Bean，从而完成属性注入

```xml
<bean id="user" class="com.yh.User"  name="userA;userB" >
      <constructor-arg name="name" type="String" value="yh"></constructor-arg>
</bean> 
```

Constructor-arg提供两种参数指定方式：

- name：指定参数名来匹配构造方法
- index：通过参数列表索引匹配构造方法

两种方式可以同时使用，但可能存在无法确认唯一构造方法的情况，因此一般情况下，推荐额外添加type属性来指定参数类型

**constructor-arg属性值配置方式和property一致**

##### 2.4.1.4、FactoryBean

bean标签支持对FactoryBean的创建，并分为两种方式：

- 工厂静态方法实例化

```xml
<bean id="user" class="com.yh.UserFactory" factory-method="createInstance"/>
```

````java
public class UserFactory {
	private static User user = new User();

	public static User createInstance() {
		return user;
	};
}
````

factroy-method用于指定Bean对应工厂类的实例获取方法，并需要保证该方法为static

- 工厂对象方法实例化：

```xml
<bean id="userFactory" class="com.yh.UserFactory"/>

<!-- 使用factory-bean时，需要保证class属性为空，即当前bean的类型，交给工厂Bean决定 -->
<bean id="user" factory-bean="userFactory" factory-method="createInstance"/>
```

```java
public class userFactory {
    private User User = new User();

    public  User createInstance() {
        return clientService;
    }
}
```

- 通过constructor-arg子标签，完成工厂类中实例获取方法的属性注入

```xml
<bean id="user" class="com.yh.UserFactory" factory-method="createInstance">
    <constructor-arg name="name" value="yh111"></constructor-arg>
</bean
```

```java
public class UserFactory {
	private static User user = new User();

	public static User createInstance(String name) {
        user.setName(name);
		return user;
	};
}
```

##### 2.4.1.5、额外命名空间

spring对于xml方式的配置，提供一系列属性注入方式的简化：

- 引入P命名空间：简化xml中property标签的使用，实现属性注入时根据实现类的set方法，进行xml属性自动提示
- 引入util命名空间：用于在IOC容器中配置集合Bean，来代替ListFactoryBean、MapFactoryBean、SetFactoryBean、PropertiesFactoryBean对象方式创建集合Bean
- 引入C命名空间：简化constructor-arg标签的使用，实现在Bean标签中，使用内联属性进行构造函数参数的声明（spring3.1引入）

#### 2.4.2、Bean行为配置

IOC容器对Bean提供了作用域、生命周期、依赖关系的行为配置

##### 2.4.2.1、作用域

spring提供Bean的六个作用域，通过Bean标签的属性Scope设置

| 作用域                    | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| singleton（单实例）       | 默认值，作用在spring容器中，保证实例唯一                     |
| prototype（原型，多实例） | 作用在spring容器中，每次获取bean都会实例化一个新对象         |
| request                   | 作用在当前web应用的request请求中，每个request请求都会实例化一个新对象 |
| session                   | 作用在当前web应用的session中，每个session都会实例化一个新对象 |
| application               | 作用在当前web应用中，保证实例唯一                            |
| websocket                 | 作用在在当前Websocket连接中，即每个Websocket连接都会实例化一个新对象 |

**单例和原型使用场景：**

1、单例：

​		适合于无状态Bean，即Bean实例化对象不保存当前线程使用的数据，线程安全；如DAO层（数据方法对象）、service层（业务逻辑对象）

2、原型：

​		适合于有状态Bean，每个Bean实例化对象都保存了当前线程的数据信息，伴随调用者的消亡而消亡；并且在被IOC容器创建后，就不会被IOC容器管理，因此其Bean定义的销毁方法也不会被执行，需要开发者手动进行原型Bean中的资源释放

注意：

- request、session、application和websocket作用域并不能在spring常规IOC容器中定义，否则会抛出IllegalStateException异常，未知bean作用域；它们只用于web项目中的IOC容器

- 

- 单例Bean依赖于原型Bean时，会将原型Bean的作用域改为单例

  可以通过手动注入方式，来避免IOC容器修改原型Bean的作用域

- 原型Bean依赖于单例Bean时，始终使用同一个单例Bean

##### 2.4.2.2、生命周期

Bean的生命周期简单分为5个过程：

| 生命周期       |
| -------------- |
| 调用构造方法   |
| 属性注入       |
| 调用初始化方法 |
| 调用销毁方法   |
| 销毁           |

IOC容器能够实现Bean生命周期的方法回调：

- 初始化方法回调：

  在Bean完成属性注入后，实现无参初始化方法的回调

  ```xml
  <bean id="user" class="com.yh.User" init-method="init" />
  ```

- 销毁方法回调：

  在Bean销毁之前，实现无参销毁方法的回调

  ```xml
  <bean id="user" class="com.yh.User" destroy-method="destroy" />
  ```

bean标签的父标签beans，提供全局初始化方法、销毁方法的配置：

```xml
<beans default-init-method="init" default-destroy-method="destroy">
```

**IOC容器关闭时，会销毁所有Bean，然后调用registerShutdownHook方法**

##### 2.4.2.3、循环依赖、延迟加载、强制依赖

bean的加载顺序由bean直接的依赖关系决定：

- IOC容器初始化时，会验证每一个Bean的配置，并优先加载所有的非延迟加载的Bean

  注意：原型Bean为懒加载，单例Bean默认为非延迟加载

- 但两个Bean存在依赖关系时，IOC容器会优先实例化被依赖的Bean，从而完成另外一个Bean的属性注入

**在Bean依赖关系中，存在循环依赖问题：**

​		A的创建依赖于B，B的创建又依赖于A；spring对这种问题提供了解决方式，但前提是循环依赖关系不能在bean的实例化阶段构造，只能在属性注入阶段，否则将抛出BeanCurrentlyInCreationException异常

**延迟加载：**

​		Bean标签提供lazy-init属性，指定bean是否延迟加载；

```xml
<bean id="user" class="com.yh.User" lazy-init="true"/>
```

注意：

- 延迟加载并不会影响bean的注册，当该bean被需要加载的bean依赖时，则按顺序进行正常加载

**强制依赖：**

​		当Bean依赖关系不太直接时，IOC容器则无法严格确定bean的加载顺序，比如：

A依赖于B，B相关属性的有效值通过C的初始化方法来实现重新赋值，此时B优先于A加载，而B和C没有直接的依赖关系，但却需要保证C加载优先A，否则会导致A初始化使用B相关属性的值

​		Bean标签提供depends-on属性，可以显式强制当前Bean要在指定Bean后面加载，如user需要在gread后加载：

```xml
<bean id="user" class="com.yh.User" depends-on="gread"/>
<bean id="gread" class="com.yh.Gread" />
```

#### 2.4.3、Bean自动装配:

​		Bean标签通过Property、Constructor-arg子标签来完成手动属性注入，但也可以通过autowire属性来进行自动装配，即根据某种规则，自动匹配IOC容器中的Bean来作为value进行属性注入

​		和手动属性注入类似，spring提供两种自动装配方式：基于属性的自动装配、基于构造方法的自动装配

##### 2.4.3.1、基于属性的自动装配

​		IOC容器会根据当前Bean类中**提供了setter方法的属性**，来对其进行自动注入：

```xml
<Bean id="gread" class="com.yh.Gread"/>  

<!-- 自动注入gread-->
<Bean id="user" class="com.yh.User" autowire="byNmae"/>
<Bean id="user" class="com.yh.User" autowire="byType"/>
```

1、autowire为byName时，寻找IOC容器中bean id和属性名相同的Bean实例，进行注入

2、autowire为byType时，寻找IOC容器中bean class和属性类型相同的Bean实例，进行注入；

注意：

- 自动装配不能和手动注入同时使用，因此**基于属性的自动装配只能使用无参构造方法**，如果没有则会抛出异常

##### 2.4.3.2、基于构造方法的自动装配

​		spring会根据如下策略，选取构造方法，根据参数类型匹配bean，完成自动装配：

1、获取所有构造方法，进行排序：参数多的优先、权限修饰符为public的优先

2、对于参数个数相同、权限修饰符相同的构造方法，比较参数类型的差异性（差异性高的优先）

3、对所有构造方法进行遍历，找到第一个满足IOC容器注入条件的构造方法，进行实例化和自动装配

```xml
<Bean id="gread" class="com.yh.Gread"/> 

<!-- 自动注入gread-->
<Bean id="user" class="com.yh.User" autowire="constructor"/>    
```

**注意：**

- 自动装配无法完成简单数据类型属性的注入

- 使用property、constructor-arg标签，进行手动属性注入后，会使自动装配失效

- 当使用属性类型（构造方法参数类型）进行Bean匹配时，如果IOC容器匹配到多个相同类型bean，则会抛出异常；可以使用Bean标签的primary，来标记在该类型所有Bean中，当前bean优先匹配（注意同类型bean中，只能指定一个）

  ```java
  <Bean id="gread" class="com.yh.Gread" primary="true"/>
  ```

#### 2.4.4、IOC容器基本注解

spring2.0中，引入一系列java注解，来简化Bean的注入，需要额外在spring.xml引入context命名空间：

```xml
<context:annotation-config />
```

此时IOC容器中，会自动创建如下Bean，用于开启IOC容器注解：

| Bean类型                                 | 作用                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| internalConfigurationAnnotationProcessor | 处理用于注册Bean的注解（**@Component、@ComponentScan、@Bean . . **） |
| internalAutowiredAnnotationProcessor     | 处理**@Autowired注解**                                       |
| internalCommonAnnotationProcessor        | 处理带有**JSR-250规范注解**                                  |
| internalEventListenerProcessor           | 事件监听处理器                                               |
| internalEventListenerFactory             | 事件监听工厂                                                 |

##### 2.4.4.1、@Component系列注解

spring2.0提供@Component系列相关注解，来注册bean：

| 注解        | 描述              |
| ----------- | ----------------- |
| @Component  | 注册通用Bean      |
| @Controller | MVC控制层Bean     |
| @Service    | MVC服务层Bean     |
| @Reposity   | MVC数据访问层Bean |

注意：

- @Controller、@Service、@Reposity为包含@Component的组合注解，实际还是通过@Component注解来起到注册bean的作用；通过导入spring-web模块，IOC容器会通过它们进行AOP代理，从而对Bean进行额外处理
- @Component系列注解默认使用类名（转为驼峰模式）的作为bean的id，可以使用value属性自定义bean的id

**开启@Component注解：**

```xml
<context:component-scan base-package="com.yh" />
```

- 开发者需要手动指定扫描哪些包内的@Compoent注解，否则所有@Component注解都不会生效

- 在开启Component扫描时，默认也会隐式开启IOC容器注解功能（context:annotation-config）

- 在context:component-scan标签下，spring提供context:include-filter、context:exclude-filter子标签，来选择或过滤@Component类型：

  一般情况下，两种方式任选其一，以web项目为例，需要对单独扫描@Controller注解放入IOC-web子容器，非@Controller注解放入IOC父容器：

  ```xml
  <context:component-scan base-package="com.yh.controller"  use-default-filters="false">
  	   <context:include-filter type="annotation" 		expression="org.springframework.stereotype.Component"/>
  </context:component-scan>
  
  <context:component-scan base-package="com.yh"  >
  	   <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
  </context:component-scan>
  ```

  use-default-filters默认为ture，即匹配扫描@Component系列注解，一般搭配context:exclude-filter来排除其中的某些注解

  use-default-filters=false则搭配context:include-filter，给开发者手动选择需要匹配的@Component系列注解

#### 2.4.5、导入外部properties文件

spring通过context:property-placeholde标签，来导入properties文件，同时IOC容器中会创建`PropertySourcesPlaceholderConfigurer`实例，用于使用@Value注解进行外部属性注入

```xml
<context:property-placeholder location="classpath:jdbc.properties">
```

注意：

- 当所有properties文件不存在@Value指定值时，会报错；出现多个时，会按照文件加载顺序，选取优先值

### 2.5、IOC容器配置（注解）：

spring3.0中，提供java注解的方式来代替xml配置，搭配一系列配置注解，来省略spring.xml

#### 2.5.1、@Configuration

@Configuration注解为@Component的组合注解，它们都为配置类注解，用于Bean注册；@Component为Lite模式，@Configuration为Full模式

**Lite模式和Full模式的区别：**

@Configuration注解的类，在注册到IOC容器时，默认会通过CGLIB进行@Bean注解方法的动态代理，来处理带有@Bean注解方法的依赖调用问题：

```java
	@Bean("userA")
	public  User getUserA() {
		User user = new User();
		user.setName("A");
		return user;
	}

	@Bean("userB")
	public  User getUserB() {
        User user = getUserB();
		user.setName("B");
		return user;
	}
```

1、如果使用@Component注解，则userA和userB并不是同一个Bean

2、如果使用@Configuration注解，则userA和userB为同一个Bean，这样便于在同一个配置类的多个方法中，进行bean的传递

注意：

- @Component注解可以使用@Qualifier，来将所依赖方法返回值对应的Bean作为参数进行传递，避免重复创建对象
- @Configuration注解提供proxyBeanMethods属性，默认为ture；**由于动态代理会导致一定的性能损失，在spring5.2后，推荐使用@Configuration(proxyBeanMethods=false)，将其设置为Lite模式**

**加载配置类：**

```java
public static void main(String[] args) {
    ApplicationContext ctx = new AnnotationConfigApplicationContext();
    ctx.register(AppConfig.class);
    ctx.register(OtherConfig.class,AdditionalConfig.class);
    ctx.refresh();
}
```

使用ApplicationContext实现类（`AnnotationConfigApplicationContext`）进行配置类的加载

#### 2.5.2、@Bean：

除了spring2.0提供的@Component系列注解进行Bean注册外，spring3.0提供@Bean来进行Bean注册：

 ```java
@Component
public class UserFactory {
	
	@Bean
	public static User getUser() {
		User user = new User();
		user.setName("yh123");
		return user;
	}

	@Bean
	public  User createInstance(@Value("${name}")String name,@Qualifier("user")User user) {
		user.setName(name);
		return user;
	};
}
 ```

注意：

- @Bean注解的方法参数会使用IOC容器进行依赖注入，默认通过参数名进行匹配Bean的id，也可以使用@Qualifier自定义
- @Bean默认使用方法名作为Bean的name（别名），可以通过name属性指定多个；根据别名和id的关系，默认选择第一个作为Bean的id
- 一个类中所有@Bean注释的方法，会按照其声明顺序进行bean注册；当@Bean方法之间存在依赖注入时，则按照依赖关系进行bean的注册
- @Bean注解方法不能为private，因为方法可能会搭配@Configuration进行动态代理
- @Bean注解提供initMethod、destroyMethod属性，指定该Bean生命周期的回调方法

**@Bean和@Component的区别：**

​		@Component用于开发者编写类的Bean注册；而@Bean一般用于第三方类的Bena注册，并且可以对其实例进行额外配置

**@Bean使用在静态方法上：**

​		@Bean允许注解到静态方法上，从而允许在当前配置类未实例化的情况下被调用；让这些bean在IOC容器生命周期的早期完成初始化，提前于其他普通bean；并且由于为静态方法，不会被IOC容器拦截进行动态代理

​		作用：可以用于BeanPostProcessor、BeanFactoryPostProcessor这些需要早期完成初始化的Bean的注册（但一般情况下，使用@Component方式就可以完成，因为spring会对该种类型的Bean进行提前初始化处理）

#### 2.5.3、@Autowired

@Autowired用于代替Bean标签的autowired属性，来实现自动装配；并根据注解位置的不同，分为三种类型:

| 注解位置 | 类型                                          | 注入时间     |
| -------- | --------------------------------------------- | ------------ |
| 构造方法 | 基于构造方法自动注入                          | bean实例化时 |
| set方法  | 基于set方法自动注入                           | bean初始化时 |
| 成员变量 | 基于反射方式进行自动注入（XML配置方式不支持） | bean初始化时 |

- 基于set方法自动注入
- 基于反射进行自动注入,将注入

可以注解在构造方法、普通方法和成员变量上，来完成Bean的自动装配

```java
@Component
public class User {

    @Autowired
    private Gread gread;
    
    @Autowired
    public User(Gread gread){
     	this.gread=gread;   
    }
    
    @Autowired
    public void setGread(Gread gread) {
        this.gread=gread; 
    }
}
```

注意：

- 对于成员变量的自动装配，通过java反射实现依赖注入，不需要提供当前成员变量的set方法

- @Autowired不能声明在静态方法和成员变量上，否则无法生效（但并不影响IOC容器的初始化）；对于静态变量，可以使用ApplicationContext进行手动注入

- @Autowired提供属性required，默认为true，表示IOC容器中必须存在匹配的Bean；当设置required=false时，则允许IOC容器中没有匹配的Bean

- @Autowired默认使用类型匹配唯一Bean，如果匹配到多个Bean，则抛出异常；可以通过@Qualifier注解来声明Bean的id，或者使用@primary搭配Bean注册，来定义当前类型所有Bean中，该bean进行类型优先匹配

  ```java
     	@Qualifier("gread")
  	@Autowired
      private Gread gread;
      
      @Autowired
      public User(@Qualifier("gread") Gread gread){
       	this.gread=gread;   
      } 
  ```

**@Autowired基于构造方法自动装配时，构造方法的选择选择策略：**

- 当不使用@Autowired不指定构造方法时：

  1、只有一个构造方法，则直接使用该构造方法

  2、有多个构造方法，则使用无参构造方法，没有就会报错

- 当使用@Autowired指定构造方法时：

  1、只指定一个构造方法，则使用该构造方法，如果IOC容器无法完成依赖注入，则会抛出异常；如果额外设置required=false，则如果IOC容器无法完成依赖注入，将改为无参构造方法（如果没有则会抛出异常）

  2、指定了多个构造方法，则必须保证所有构造方法的@Autowried属性required=false；然后IOC容器根据匹配策略，选择对于构造方法完成自动装配（**策略和xml方式的自动装配一致**）

#### 2.5.4、@Value

@Value和@Autowired类似，用于通过配置文件，进行基础数据类型的自动装配，提供两种类型：

- 注解在方法参数上（包括构造方法）
- 注解在成员变量上，基于java反射完成，因此不需要提供set方法，并且不支持静态成员变量

@Value注解通过value属性值，来匹配配置文件中的key，根据vlaue值的表达方式，分为两种：

- ${}表达式：

```java
@Value("${name}")
private String name;
```

- #{}表达式，即Spring-EL表达式，进行动态属性注入

  通过Spring-EL表达式，可以是实现复杂数据结构对象的属性注入

  ```java
  @Value("#{{'math':100,'english':99}}")  
  private  Map<String, Integer> map;
  ```

**@Vlaue配置文件key的匹配策略：**

- 当在所有配置文件、环境变量中无法匹配到key时，则抛出异常
- 当出现多个匹配值时，根据配置文件、环境变量的优先级顺序确定

#### 2.5.5、配置类相关注解

spring3.0通过@Configuration、@Component注解来搭配其他配置类相关注解，代替spring.xml配置文件

##### 2.5.6.1、@ComponentScan

@ComponentScan代替context:component-scan标签，开启@Component注解功能

常用属性：

| 属性名            | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| basePackages      | 定义扫描路径，为string[]，可以定义多个路径                   |
| value             | basePackages属性别名，作为默认值                             |
| excludeFilters    | 黑名单过滤器                                                 |
| includeFilters    | 白名单过滤器                                                 |
| useDefaultFilters | 是否使用默认过滤器；默认为true，注册路径中所有带有@Component及其子注解的类 |

```java
//如：排除controller注解的类
@Configuration
@ComponentScan(basePackages= {"com.yh"},excludeFilters= {
		@Filter(type=FilterType.ANNOTATION,classes= {Controller.class})})
public class myConfig{
}
```

##### 2.5.6.2、@PropertySource

@PropertySource代替context:property-placeholder标签，导入外部properties文件

常用属性：

| 属性名                 | 描述                                                         |
| ---------------------- | ------------------------------------------------------------ |
| name                   | 资源命名（一般不需要）                                       |
| value                  | 外部资源路径                                                 |
| encoding               | 编码格式，默认使用项目编码格式                               |
| ignoreResourceNotFound | 是否忽略找不到指定值；默认false，不忽略                      |
| factory                | 定义PropertySources的生成规则，默认PropertySourceFactory.class |

```java
@Configuration
@PropertySource("classpath:/my.properties")
public class myConfig{
}
```

注意：

- ignoreResourceNotFound，一般使用默认值false，降低异常发生时的排除难度

##### 2.5.6.3、@Import

一种区别于@Component、@Bean的，额外注册Bean的注解，用于直接将第三方类实例加载到IOC容器中，使用类名（转为驼峰模式）作为Bean的id

**相对于@Bean更加简单，但只能使用默认构造方法进行实例化，没有则会报错**

```java
@Configuration
@Import({User.class})
public class myConfig{
}
```

#### 2.5.6、JSR-250规范注解

spring兼容JSR-250规范，来控制Bean的注入和生命周期回调

##### 2.5.6.1、@Resource

作用和@Autowired+@Qualifie类似，通过id、类型从IOC容器中获取bean

```java
public class SimpleMovieLister {

    @Resource
    private MovieFinder movieFinder;

    @Resource
    public void setMovieFinder(MovieFinder movieFinder) {
        this.movieFinder = movieFinder;
    }
}
```

@Resource提供name、type属性来指定Bean的id和类型，策略如下：

- 同时指定name、type，则使用id和类型匹配唯一Bean，找不到则抛出异常
- 只指定name，则使用id匹配唯一Bean，找不到则抛出异常
- 只指定type，则使用类型匹配唯一Bean，找到多个或0个则抛出异常
- 都不指定时，则默认使用属性名/参数名作为id匹配唯一Bean，如果匹配不到则使用属性类型/参数类型匹配，找到多个或0个则抛出异常

##### 2.5.6.2、@PostConstruct、@PreDestroy

定义Bean生命周期中初始化和销毁的回调方法

```java
public class CachingMovieLister {

    @PostConstruct
    public void init() {
        
    }

    @PreDestroy
    public void destroy() {
       
    }
}
```

- 它们和xml中Bean生命周期回调方法配置一样， @PostConstruct注解方法在Bean完成属性注入后执行，@PreDestroy在Bean销毁前执行
- 它们注解的方法，可以和xml中的Bean生命周期回调方法同时存在并执行（两种配置不冲突，但不推荐，因为不太确定它们的执行顺序）

#### 2.5.7、其他注解（不常用）

##### 2.5.7.1、Bean行为相关注解

用于代替Bean标签中Scope、Lazy和DependsOn属性，搭配@Component、@Bean一起使用

| 注解       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| @Scope     | 和@Component、@Bean一起使用，定义bean注册时的作用域，默认为singleton |
| @Lazy      | 和@Component、@Bean一起使用，使bean的初始化延迟，getBean时才进行加载，默认为true |
| @DependsOn | 和@Component、@Bean一起使用，定义该bean的初始化，要在指定bean完成之后进行 |
| @Order     | 和@Component、@Bean一起使用，定义bean在被使用时的优先级      |

@Order应用场景：

​		在AOP中，对于一个切点存在多个切面时，可以通过@Order定义切面方法执行的优先级

##### 2.5.7.2、JSR-330注解

| 注解                  | 等效spring注解      |
| --------------------- | ------------------- |
| @Inject               | @Autowired          |
| @Named / @ManagedBean | @Component          |
| @Singleton            | @Scope("singleton") |
| @Qualifier / @Named   | @Qualifier          |

##### 2.5.7.3、@Required

@Autowired的required属性作用类似，只能注解在set方法上，搭配xml配置方式使用，表示当前Bean使用xml注册时，必须保证该set方法属性被注入，默认为true

### 2.6、IOC容器扩展接口：

springIOC容器在设计上，为开发者提供了强大的自定义功能，允许对Bean的整个生命周期过程进行扩展处理；**它们本身也需要注入到IOC容器中**

#### 2.6.1、FactoryBean接口

​		spring提供FactoryBean接口，来简化FactoryBean配置：

```java
	//获取该工厂Bean生产的Bean对象
	T getObject() throws Exception;

	//获取生产Bean的类型
	Class<?> getObjectType();

	//生产Bean是否为单例（默认ture）
	default boolean isSingleton() {
		return true;
	}
```

​		该接口一般用于复杂对象Bean的创建，并且FactoryBean本身也会注册到IOC容器中，但beanId会添加&前缀表示，其生产的Bean则为普通Bean

#### 2.6.2、BeanFactoryPostProcessor接口

​		用于在BeanFacotry（IOC容器）实例化后，对其中已加载的BeanDefinition（Bean注册信息）进行修改

```java
//获取已实例化的BeanFacotry，对其进行操作
void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException;
```

注意：

- BeanFacotryPostProcessor可以存在多个，可以同@Order或者Ordered接口来控制它们的执行顺序

#### 2.6.3、BeanPostProcessor接口

​		用于对已完成属性注入的Bean，在执行初始化方法前后，对其进行额外处理

```java
	//Bean执行初始化之前
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	//Bean执行初始化之后
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}
```

注意：

- BeanPostProcessor可以存在多个，可以同@Order或者Ordered接口来控制它们的执行顺序

- **BeanPostProcessor接口和InitializingBean、DisposableBean接口区别：**

  1、前者是通过BeanPostProcessor的执行方式，作用于所有Bean的初始化阶段

  2、后者是提供针对于实现该接口的单个Bean，编写初始化和销毁方法，无法作用于所有Bean（XML中的init-method、destroy-method就是使用这种方式）

  3、@PostConstruct发生在BeanPostProcessor接口方法后；@Processor发生在DisposableBean接口前

#### 2.6.4、Aware系列接口

​		aware系列接口用于为当前Bean进行指定对象或数据的注入：

| 接口                           | 作用                                 |
| ------------------------------ | ------------------------------------ |
| BeanClassLoaderAware           | 获取当前Bean使用的类加载器           |
| ResourceLoaderAware            | 获取资源加载器                       |
| BeanNameAware                  | 获取当前Bean的id                     |
| BeanFactoryAware               | 获取当前Bean所在的BeanFacotry        |
| ApplicationContextAware        | 获取当前Bean所在的ApplicationContext |
| MessageSourceAware             | 获取消息资源处理器                   |
| ApplicationEventPublisherAware | 获取事件发布器                       |

##### Aware接口的回调执行点：

1、BeanNameAware、BeanClassLoaderAware、BeanFactoryAware接口，会在Bean完成属性注入后，执行初始化回调前，被回调处理

2、EnvironmentAware、EmbeddedValueResolverAware、ResourceLoaderAware、ApplicationEventPublisherAware、MessageSourceAware、ApplicationContextAware接口，会在`ApplicationContextAwareProcessor`的初始化回调方法中，进行回调处理

### 2.7、IOC容器的内部实现

​		BeanFacotry是实现IOC容器的核心接口，内部实现了对Bean的加载全过程

#### 2.7.1、BeanDefinition

​		beanDefinition为Bean的元数据，定义了Bean创建所需要的所有信息，其实现类为`GennericBeanDefinition`

**GennericBeanDefinition重要属性：**

| 属性              | 作用                                             |
| ----------------- | ------------------------------------------------ |
| beanClass         | bean类型                                         |
| scope             | bean作用域                                       |
| lazyInit          | 是否延迟加载                                     |
| autowiredMode     | 自动注入模式（NO、ByName、ByType）               |
| dependsOn         | 依赖于指定bean的优先创建                         |
| autowireCandidate | false时,自动装配不考虑将该bean作为依赖注入的对象 |
| primary           | 自动装配出现多个匹配项时,该bean优先              |
| qualifiers        | 内部所依赖的bean                                 |
| initMethodName    | 初始化方法名                                     |
| destroyMethodName | 销毁方法名                                       |

#### 2.7.2、DefaultListableBeanFactory

​	`BeanFactory`的实现类，IOC容器的核心类，实现Bean的管理

实现Bean管理相关属性：

- beanDefinitionNames：所有bean注册名称列表，按注册顺序排序
- singletonBeanNamesByType：所有单例bean其类型和beanNames的 K-V映射
- allBeanNamesByType：所有bean其类型和beanNames的 K-V映射
- beanDefinitionMap：所有bean其beanName和BeanDefintion的K-V映射

beanName和beanNames的区别：

​	beanName为bean在IOC容器中的唯一标识符（即ID），beanNames为bean的id和别名的集合（默认使用第一个值作为id）

##### 2.7.2.1、DefaultSingletonBeanRegistry

​	`DefaultListableBeanFactory`子类，用于处理单例Bean的注册销毁和依赖关系

- singletonObjects：存放所有单例Bean的初始化对象（初始化完成的），也称之为IOC容器的单例池
- earlySingletonObjects：存放所有单例Bean的实例化对象（没有进行属性注入的）
- singletonFactories ：存放已实例化Bean的引用ObjectFactories

IOC容器通过这三个Map集合，来构成Bean创建过程中的三级缓存，解决循环依赖问题

#### 2.7.3、初始化IOC容器

​		IOC容器的启动，基于AnnotationConfigApplicationContext实例化来完成：

```java
AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(springTest.class);
```

并在实例化过程中，完成如下操作：

- 创建`DefaultListableBeanFactory`(BeanFactory)
- 创建AnnotatedBeanDefinitionReader，用于后续Bean注册，并在初始化时注册spring一系列内置Bean
- 创建ClassPathBeanDefinitionScanner，用于开发者手动注册Bean
- 调用refresh方法，初始化IOC容器上下文

##### 2.7.3.1、内置bean

​		AnnotatedBeanDefinitionReader初始化时，注册如下spring内置Bean

```java
//注册spring内置的注解处理器（会进行初始化）
AnnotationConfigUtils.registerAnnotationConfigProcessors(this.registry);
```

| 内置Bean类型                         | 接口类型                 | 作用                                       |
| ------------------------------------ | ------------------------ | ------------------------------------------ |
| ConfigurationAnnotationProcessor     | BeanFactoryPostProcessor | 用于完成Bean的注册                         |
| AutowiredAnnotationBeanPostProcessor | BeanPostProcessor        | 处理@Autowired注解，完成Bean的属性注入     |
| CommonAnnotationBeanPostProcessor    | BeanPostProcessor        | 处理@PostConstruct和@PreDestroy、@Resource |
| PersistenceAnnotationProcessor       | BeanPostProcessor        | 处理JPA注解（需要导入spirng-orm包）        |
| EventListenerMethodProcessor         | BeanFactoryPostProcessor | 处理事件监听                               |
| DefaultEventListenerFactory          | EventListenerFactory     | 事件监听工厂                               |

##### 2.7.3.2、refresh

**refresh方法源码解析：**

```java
	@Override
	public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
		//为初始化上下文做准备
		prepareRefresh();

        //创建BeanFactory（对于注解方式，则会使用原来的，原因：注解版需要提前创建一个beanFactory来注册配置类Bean；而xml方式，则是在此时进行beanFactory的注册）
		ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
            
        //配置beanFactory：
   		//配置beanFactory内部使用的类加载器、Bean表达式解析器。。
        //添加ApplicationContextAwareProcessor、ApplicationListenerDetector两个Bean后置处理器
        //添加beanFactory相关依赖的其他Bean（MessageSource、Environment）。。。   
        //忽略Aware系列接口的自动装配
		prepareBeanFactory(beanFactory);

		try {
            //spring5没有做任何操作
			postProcessBeanFactory(beanFactory);
            
			//调用所有的BeanFactoryPostProcessors（内置和自定义）
			invokeBeanFactoryPostProcessors(beanFactory);
            
			//注册所有的BeanPostProcessor（自定义）
			registerBeanPostProcessors(beanFactory);

			//初始化上下文消息源（直接注册初始化相应bean，不需要创建BD对象，然后进行Bean初始化）
			initMessageSource();

			//初始化上下文事件广播器（直接注册初始化相应bean）
			initApplicationEventMulticaster();

			//注册其他特殊bean
			onRefresh();

			//注册监听器bean
			registerListeners();

			//初始化IOC容器中的所有单例bean（同时调用所有BeanPostProcessor）	
			finishBeanFactoryInitialization(beanFactory);

			//完成其他初始化上下文工作
			finishRefresh();
			}
        。。。。
		}
	}
```

#### 2.7.4、Bean注册

在Refresh方法中，调用所有的BeanFactoryPostProcessor，从而完成Bean的注册

```java
invokeBeanFactoryPostProcessors(beanFactory);
```

BeanFactoryPostProcessor分为两种类型：

- BeanDefinitionRegistryPostProcessor：用于注册Bean的BeanFactory后置处理器,优先执行
- BeanFactoryPostProcessor：常规BeanFactory后置处理器,用于对beanDefinition信息进行修改

默认情况下,spring只提供一个BeanDefinitionRegistryPostProcessor类型的Bean,用于完成bean注册;该bean为`ConfigurationClassPostProcessor`

**ConfigurationClassPostProcessor完成Bean注册的过程如下:**

- 获取调用BeanFacotry.register方法手动注册的Bean，判断它们是否为配置类（被@Component注解），同时设置其类型为Full或Lite

- 初始化配置类解析器ConfigurationClassParser

- 对当前配置类进行解析，处理其中的注解（@PropertySource、@ComponentScan、@Import、@Bean、@ImportResource），使用`AnnotatedBeanDefinitionReader`完成注册

- 对于Full配置类，进行动态代理，实现配置类中@Bean方法间的依赖注入

#### 2.7.5、Bean初始化

在refresh方法中，完成所有单例Bean的初始化（如果非单例bean被依赖，则也会初始化）

```java
finishBeanFactoryInitialization(beanFactory);
```

Bean初始化过程如下：

- 创建bean的实例化对象,并保证所依赖的bean被创建
- 调用所有BeanPostProcessor,完成bean的初始化（属性注入、初始化回调）

**spring提供4个BeanPostProcessor子接口，来完成整个初始化：**

**1、InstantiationAwareBeanPostProcessor**

提供三个方法：

- postProcessBeforeInstantiation，Bean实例化前，是否提供其他对象，作为其实例化对象
- postProcessAfterInstantiation，Bean实例化后，执行属性注入前，设置Bean是否正常执行自动装配
- postProcessPropertys，Bean属性注入前，对PropertyValues对象（已创建好的属性值）进行修改

实现类和作用有：

`AutowiredAnnotationBeanPostProcessor`  对Bean中@Autowired、@Value注解进行处理，完成依赖注入

`CommonAnnotationBeanPostProcessor`  对Bean中@Resource注解进行处理，完成依赖注入

**2、SmartInstantiationAwareBeanPostProcessor**

提供三个方法：

- predictBeanType，获取bean的类型
- determineCandidateConstructors，自定义bean的构造方法
- getEarlyBeanReference，获取提前暴露bean的引用

实现类和作用有：

`AnnotationAwareAspectJAutoProxyCreator`用于对bean进行动态代理，实现aop

**3、MergedBeanDefinitionPostProcessor**

提供一个方法：

- postProcessMergedBeanDefinition，在bean实例化之后，对当前Bean的BeanDeinition对象进行修改

实现类和作用有：

`AutowiredAnnotationBeanPostProcessor`  对@Autowired、@Value注解处理进行准备操作

`CommonAnnotationBeanPostProcessor`  对@Resource、@PostConstruct、注解进行祖辈操作

**4、DestructionAwareBeanPostProcessor**

bean销毁前执行，用于Bean声明周期的销毁回调

```java
public interface DestructionAwareBeanPostProcessor extends BeanPostProcessor {
    //这里实现销毁对象的逻辑
    void postProcessBeforeDestruction(Object bean, String beanName) throws BeansException;
    //判断是否需要处理这个对象的销毁
    default boolean requiresDestruction(Object bean) {
        return true;
    }
}
```

##### 2.7.5.1、Bean初始化过程

基于所有BeanPostProcessor方法调用，Bean初始化过程分为八步：

| 序号 | BeanPostProcessor接口类型和方法                              | 执行点和作用                                                 |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 1    | InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation | Bean实例化前（提供实例对象（如代理对象），否则交给spring进行实例化） |
| 2    | SmartInstantiationAwareBeanPostProcessor.determineCandidateConstructors | Bean实例化前（确定构造方法）                                 |
| 3    | MergedBeanDefinitionPostProcessor.postProcessMergedBeanDefinition | Bean实例化后（检测和修改当前Bean的BeanDefinition属性）       |
| 4    | InstantiationAwareBeanPostProcessor.postProcessAfterInstantiation | Bean属性注入前（确定是否执行Bean属性注入）                   |
| 5    | InstantiationAwareBeanPostProcessor.postProcessPropertyValues | Bean属性注入后（对Bean的属性进行最终确认）                   |
| 6    | BeanPostProcessor.postProcessBeforeInitialization            | Bean初始化回调前                                             |
| 7    | BeanPostProcessor.postProcessAfterInitialization             | Bean初始化回调后                                             |
| 8    | DestructionAwareBeanPostProcessor.postProcessBeforeDestruction | 执行Bean的销毁回调                                           |

spring初始化过程对应处理入口：

| 初始化过程       | 入口                                                         |
| ---------------- | ------------------------------------------------------------ |
| 实例化           | 实现构造方法的属性注入（@AutoWired注解在构造方法上）         |
| 属性注入         | @AutoWired、@Value                                           |
| Bean初始化回调前 | BeanPostProcessor.postProcessBeforeInitialization、@PostConstruct |
| Bean初始化回调   | 实现InitializingBean接口                                     |
| Bean初始化回调后 | BeanPostProcessor.postProcessAfterInitialization             |
| Bean销毁回调前   | @Processor                                                   |
| Bean销毁回调     | 实现DisposableBean接口                                       |



### ApplicationContext其他功能：

#### 资源加载：

- spring提供了Resource接口，重新定义了java中各种资源获取的API，功能更加强大，且使用方便；其实现类包括：

  | 类名                   | 作用                                                         |
  | ---------------------- | ------------------------------------------------------------ |
  | UrlResource            | 对java.net.URL进行了包装，通过不同前缀，来进行不同类型的资源访问 |
  | ClassPathResource      | 从当前类路径中获取资源                                       |
  | FileSystemResource     | 从文件系统中获取资源                                         |
  | ServletContextResource | 从web应用根目录中获取资源                                    |
  | InputStreamResource    | 通过一个输入流来创建资源对象                                 |
  | ByteArrayResource      | 通过一个字节数组来创建资源对象                               |

- UrlResource，前缀和访问资源类型的匹配策略：

  | 前缀           | 访问资源类型          |
  | -------------- | --------------------- |
  | classpath：    | 类路径                |
  | file：         | 文件系统              |
  | https：/http： | URL                   |
  | 无             | 根据ReourceLoader决定 |

- ReourceLoader接口资源加载器

  ApplicationContext就实现了这个接口，根据不同了Reource实现类，加载特定的Resource资源

- spring提供ResourceLoaderAware接口，用于获取ApplicationContext中的ReourceLoader,手动进行外部资源加载

#### 国际化功能（i18n）

spring提供了一个**MessageSource接口**，用于提供国际化功能（i18n）。同时还提供了**HierarchicalMessageSource接口**，用于分层解析消息，**根据不同的地区，对消息进行不同的解析**

**自定义MessageSource：**

提供三种解析消息的方法，以消息编码、参数、默认消息、地区四个参数进行自定义逻辑代码解析，实现国际化

```java
public class MyMessage implements MessageSource{

	@Override
	public String getMessage(String code, Object[] args, String defaultMessage, Locale locale) {
	}

	@Override
	public String getMessage(String code, Object[] args, Locale locale) throws NoSuchMessageException {
		return null;
	}

	@Override
	public String getMessage(MessageSourceResolvable resolvable, Locale locale) throws NoSuchMessageException {
		return null;
	}
}
```

spring提供一些MessageSource实现类，按照相应配置实现国际化功能（由于一般用于JSP页面，因此不进行深入）

#### 事件监听

spring提供ApplicationEvent类和ApplicationListener接口提供事件监听处理，在spring4.2后，提供基于注解的方式发布事件：

spring提供多个默认事件:

| 事件                       | 作用                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ContextRefreshedEvent      | applicationContext刷新（只支持可以热刷新的applicationContext，如XmlWebApplicationContext） |
| ContextStartedEvent        | applicationContext调用Start方法                              |
| ContextStoppedEvent        | applicationContext调用Stop方法                               |
| ContextClosedEvent         | applicationContext调用close方法                              |
| RequestHandledEvent        | web应用中，spring的DispatcherServlet拦截到http请求           |
| ServletRequestHandledEvent | RequestHandledEvent的子类，添加了拦截http请求的servlet信息   |

通过实现ApplicationListener接口的Bean来监听这些事件，执行相应代码：

- 自定义事件（不需要放到spring容器中）

```java
public class MyEvent extends ApplicationEvent {
	
	public MyEvent(Object source) {
		super(source);
	}
}
```

- 自定义事件监听器（被spring容器管理）

```java
@Component
public class MyEventListener implements ApplicationListener<MyEvent>{

	@Override
	public void onApplicationEvent(MyEvent event) {
		System.out.println(event.getSource());
	}
}
```

- 使用ApplicationContext发布事件，监听器触发执行代码

```java
context.publishEvent(new MyEvent("sd"));
```

#### 在web应用快速创建ApplicationContext实例化

ApplicationContext提供声明式的方式来创建其实例，用于web应用中，通过监听器伴随web应用的启动，初始化spring容器：

在web项目的web.xml中进行配置：

 ```java
	<!-- 定义上下文参数，用于指定spring的xml配置文件 -->
	<context-param>
    	<param-name>contextConfigLocation</param-name>
    	<param-value>/WEB-INF/daoContext.xml /WEB-INF/applicationContext.xml</param-	value>
	</context-param>

	<!-- 配置spring容器监听器，当项目启动时，自动初始化spring容器 -->
	<listener>
		<listener-class>
			org.springframework.web.context.ContextLoaderListener
		</listener-class>
	</listener>
 ```

当**contextConfigLocation**参数没有指定时，spring该监听器会默认扫描**/WEB-INF/applicationContext.xml**路径

### spring实现Bean的手动注册：

1、直接通过ApplicationContext，在IOC容器初始化后手动注册；该方式并不会将该Bean交给spring管理，因此**不会遵循springBean的生命周期，只是单纯用于spring将其依赖注入到其他bean中**（一般不使用）

```java
context.getBeanFactory().registerSingleton("lisr", new HelloWord());
```

2、在IOC容器初始化前，创建指定类的BeanDefinition对象，手动注册

```java
RootBeanDefinition bean = new RootBeanDefinition(HelloWord.class);
registry.registerBeanDefinition("helloWord", bean);
```

可以通过两种方式完成：

- 使用BeanDefinitionRegistryPostProcessor接口，在执行完默认注册处理器后，执行手动注册处理器

  ```java
  public class MapperBeanDefinitionRegistart implements BeanDefinitionRegistryPostProcessor{
  
  	@Override
  	public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
  	}
  
  	@Override
  	public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
  		RootBeanDefinition bean = new RootBeanDefinition(HelloWord.class);
          registry.registerBeanDefinition("helloWord", bean);
  	}
  }
  ```

- 使用ImportBeanDefinitionRegistrar接口，创建一个动态注册Bean的实现类

  使用方式：

  - 实现ImportBeanDefinitionRegistrar接口

    registerBeanDefinitions方法提供两个参数：

    - AnnotationMetadata  使用@Import导入自身的配置类的所有注解信息
    - BeanDefinitionRegistry spring的Bean注册类（BeanFactory实现类）

    ```java
    public class MyBeanDefinitionRegistart implements ImportBeanDefinitionRegistrar{
    
    	@Override
    	public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry registry) {
            //获取当前配置类的注解信息（可以创建一个自定义注解，来获取需要注册的全类名，从而获取class对象）
    	AnnotationAttributes annoAttrs = AnnotationAttributes.fromMap(annotationMetadata.getAnnotationAttributes(ComponentScan.class.getName()));
    	String[] basePackages = annoAttrs.getStringArray("basePackages");
    		
    	//手动注册Bean（通过class对象创建）
    	RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(TestMapper.class);
    	registry.registerBeanDefinition("testMapper", rootBeanDefinition);
    	}
    }
    ```

  - 在配置类上使用@Import注解到，来导入该实现类（使用@Component注册不会生效）

    ```java
    @Configuration
    @Import(ImportBeanDefinitionRegistrarTest.class) 
    public class ConfigurationTest {
     	
    }
    ```

**也可以将@Import声明到自定义注解上，然后通过自定义注解声明到配置类上，来实现整个功能（即利用组合注解，来使自定义注解具有动态注册Bean的能力，mybatis整合spring中，@MapperScan就利用了这种方式）**

## 3、AOP

### 3.1、基本概念

- AOP的定义：

  ​		面向切面编程，提供了另一种编写程序结构的方式，是对面向对象编程（OOP）的补充

- AOP的作用：

  ​		AOP可以对业务逻辑的各个部分进行隔离，使得业务逻辑各部分之间的耦合度降低，提高程序的可重用性，提升开发效率；因此也就能实现，在不改变原来业务代码的基础上，添加新功能，并且可以通过配置随时删除

- AOP的实际应用：

  ​		安全控制、事务处理、异常处理、日志打印、性能统计等

- AOP在spring中的实现应用：
  1. 提供声明式的企业服务，如spring的声明式事务管理
  2. 为用户提供切面编程开发手段，即spring aop

### 3.2、AOP原理-动态代理

​	AOP底层原理技术为动态代理，分为JDK动态代理和CGLIB动态代理

- JDK动态代理：

  ​		创建接口实现类的代理对象，从而增强实现类的方法

  代码实现过程如下：

  ​		1、通过JDK提供的java.lang.reflect.Proxy对象newProxyInstance方法来创建代理对象

  ​		2、方法参数包括：目标类的类加载器、目标类所实现的接口、InvocationHandler对象

  ​		3、在InvocationHandler中，重写invoke方法，完成目标类指定方法的拦截处理

  ​		4、在invoke方法中，proxy为代理对象，method为目标类方法对象，args为目标类方法参数；通过method.invoke(目标类实例，args)来执行原方法

  ```java
  		//动态代理
  		Singer singerProxy1 = (Singer)Proxy.newProxyInstance(singerL.getClass().getClassLoader(),Singer.classs
  				new InvocationHandler() {
  					@Override
  					public Object invoke(Object proxy, Method method, Object[] args){
  						System.out.println("进行动态代理1");
  						Object invoke = method.invoke(singerL, args);
  						System.out.println("进行动态代理2");
                          //invoke为singerL实现类的当前方法的返回值；当return null时，代理类执行返回值就为null
  						return invoke;
  					}
  				});
  		singerProxy1.sing();
  ```

- CGLIB动态代理：

  ​		创建当前类子类的代理对象，从而增强当前类的方法

  代码实现过程如下：

  1、首先需要导入额外依赖包：cglib（在spring-core中已经包含了该工具包）

  2、创建一个enhancer对象，用于设置动态代理的目标类、方法拦截器
  
  3、最后在生成以目标类作为父类继承的子类代理类
  
  ```java
  		Enhancer enhancer = new Enhancer();
  		enhancer.setSuperclass(SingerL.class);
  		enhancer.setCallback(new MethodInterceptor() {
  			@Override
  			public Object intercept(Object Proxy, Method method, Object[] arg, MethodProxy methodProxy) throws Throwable {
  				System.out.println("进行动态代理1");
  				Object invokeSuper = methodProxy.invokeSuper(Proxy, args);
  				System.out.println("进行动态代理2");
  				return invokeSuper;
  			}
  		});
		
  		SingerL singerLProxy = (SingerL)enhancer.create();
		singerLProxy.sing();
  ```

**spring动态代理方式的选择：**

​		spring默认使用JDK动态代理，由于JDK动态代理需要基于目标类的接口来实现，因此要确保目标类有对应的接口；当没有接口时，spring则使用CGlib动态代理的方式实现

springAOP在使用动态代理实现切面编程中，是基于IOC容器的依赖注入，实现动态代理类的创建和使用的；**因此在使用AOP时，需要保证被增强类和增强类都被IOC容器管理**

**动态代理对于final或private修饰方法的处理：**

- 对于JDK动态代理，需要基于目标类接口，而接口方法只能为public（非final），因此不支持final或private的动态代理
- 对于CGLIB动态代理，则通过动态创建目标类子类方式进行代理，因此只会继承public（非final）方法，，因此不支持final或private的动态代理

### 3.3、springAOP的使用

​		通过导入spring-aop包，来引入springAop模块；同时springAop基于IOC容器来进行对象管理，因此还需要导入springIOC容器相关依赖

​		spring-aop提供三种使用方式，xml、注解、接口编程式；一般推荐使用注解方式，更加简洁；而注解方式需要基于AspectJ完成（单独的AOP框架）

#### 3.2.1、AspectJ

​		AspectJ是一个AOP框架，它扩展了java语言，定义了AOP语法

**实现原理：**

​		通过专门的编译器，解读AOP语法，在编译过程中，对目标类进行切面代码织入，生成class文件；因此被称之为静态织入

**优缺点：**

- 缺点：需要专门的编译器ajc；新增增强代码时，需要重新编译被增强目标类
- 优点：从class文件上，就进行了AOP的实现，性能更好；支持多个所有级别的切入点，AOP能力更加强大

#### 3.2.2、AOP术语

​		AOP作为一种编程思想，相对于OOP（面向对象编程），提供了一系列新的术语

- 切面（Aspect）：声明当前类可以用于AOP，包含通知和切入点

- 连接点（JointPoint）：进行AOP增强的目标方法
- 切入点（Pointcut）：表示一组连接点，通过切入点表达式进行匹配，找到满足条件的连接点

- 通知（Advice）：定义具体的增强处理，并声明通知类型

  | 通知类型 | 代码执行时间点                                               |
  | -------- | ------------------------------------------------------------ |
  | 前置通知 | 在原方法执行前                                               |
  | 后置通知 | 在原方法执行后                                               |
  | 环绕通知 | 在原方法执行前后                                             |
  | 异常通知 | 当原方法出现异常时                                           |
  | 最终通知 | 类似于JAVA异常中的finally，无论方法是否发生异常，都会在最后被执行 |

- 织入（Weaving）：表示完成AOP增强操作的过程

#### 3.2.3、springAop（注解版）

​		springAop（注解）基于AspectJ实现，spring利用AspectJ框架的配置模式，使用其aop相关注解和切入点表示，来完成AOP的定义；内部则完全通过动态代理，在运行期实现对目标类的增强；因此被称之为动态织入

**优缺点：**

- 缺点：依赖于springIOC容器，由于是动态代理实现方法增强，因此性能不如AspectJ；只支持方法级别的切入点
- 优点：纯java代码实现，无需额外编译，使用更加简单

**依赖：**

​		spring-aop默认不会引入AspectsJ模块依赖，需要额外导入spring-aspects（提供spring对于AspectJ配置驱动的整合）、aspectjweaver（提供aop相关注解和切入点表达式）

##### 3.2.3.1、springAop切面注解

- @EnableAspectJAutoProxy：开启springAop功能

- @Aspect：注解在指定类上，将其声明切面类（需要保证该类被IOC容器管理）

- @Pointcut：注解在指定方法上，用于声明切入点，定义切入点表达式；此时该方法签名在aspectJ语法中，就表示为该切入点，实现切入点表达式的复用（需要保证方法返回值为void，无参）

- 通知注解：注解在指定方法上，搭配切入点，定义通知内容和通知类型

  | 通知注解        | 对应通知类型           | 通知执行点                                               |
  | --------------- | ---------------------- | -------------------------------------------------------- |
  | @Before         | 前置通知               | 连接点方法执行前                                         |
  | @AfterReturning | 后置通知（返回前通知） | 连接点方法执行、返回结果前                               |
  | @Around         | 环绕通知               | 控制整个连接点方法的执行（执行完成的前后处理、是否执行） |
  | @AfterThrowing  | 异常通知               | 在连接点方法发生异常后执行（可以匹配指定异常）           |
  | @After          | 最终通知               | 连接点方法执行、返回结果后                               |

##### 3.2.3.2、切入点表达式

AspectJ切入点表达式语法，提供9种匹配、限制语句

| 表达式类型  | 规则                                                         |
| ----------- | ------------------------------------------------------------ |
| execute     | 匹配连接点方法（灵活性高，最常用）                           |
| within      | 限制连接点所在的包、类                                       |
| this        | 限制AOP代理类的类型                                          |
| target      | 限制AOP目标类的类型                                          |
| args        | 限制连接点方法的参数类型；同时可以搭配通知方法进行连接点实参传递 |
| @target     | 限制AOP目标类的类型上必须包含指定注解                        |
| @args       | 限制连接点方法参数类型的类上，包含指定注解                   |
| @within     | 限制连接点所在类上，包含指定注解                             |
| @annotation | 限制连接点方法必须包含指定注解                               |

**execution表达式格式：**

​		execution（[权限修饰符]    [返回类型]    [包名.类名].[方法名] ([参数列表]) throws [异常类型]）  

注意：

- 由于AOP基于动态代理，因此连接点方法必须为public，所以权限修饰符可以省略
- 包名和类名可以省略，表示任意类
- throws一般情况下，推荐省略

通配符：

- *匹配所有支付

- ..   匹配多个包、多个参数
- +表示类和其子类

**可以搭配&&将execution和其他限制语句一起使用：**

```java
//com.yh.common.aop包下任意public方法
execution(* *(..))&&within(com.yh.common.aop..*)
```

**限制语句也可以单独使用：**

```java
//com.yh.common.aop包下任意方法
within(com.yh.common.aop..*)
```

##### 3.2.3.3、五种通知使用规范和使用场景：

- @Before：

  通知方法返回值应该为void（允许设置返回值，但没有意义），参数列表只能引入JoinPoint对象（保证第一个参数），用于获取连接点信息（代理对象、目标对象、连接点方法信息）

  **一般用于方法调用前的日志记录**

- @After：

  和@Before一致

  **一般用于方法调用后的日志记录、资源关闭**

- @AfterReturning

  通知方法返回值应该为void（允许设置返回值，但没有意义），参数列表可以引入JoinPoint对象（保证第一个参数）；搭配returning属性，可以通过通知方法参数接收连接点返回值对象，需要保证通知方法参数名和reurning属性值一致；从而实现对其返回值对象的处理（对于final对象，无法进行修改）

  **一般用于对方法返回值进行额外处理，如对结果进行缓存**

- @AfterThrowing

  通知方法返回值应该为void（允许设置返回值，但没有意义），参数列表可以引入JoinPoint对象（保证第一个参数）；搭配throwing属性，可以通过通知方法参数来接收连接点抛出的异常对象，需要保证通知方法参数名和throwing属性值一致；同时连接点只有抛出该异常类型，才会进行该通知处理

  **一般用于方法调用异常的日志记录（并不能对捕获异常）**

- @Around

  通知方法返回值应该为Object，参数列表可以引入ProceedingJoinPoint对象（保证第一个参数），在JoinPoint对象基础上提供了执行连接点方法功能；可以对整个连接点方法执行前后进行处理、控制连接点方法是否执行、自定义返回值

  **一般用于统一异常处理、缓存控制**

可以看出，@Around功能最为强大，实际上可以实现其他4中通知功能；但在满足需求的情况下，应该选择更加简单的通知

**五种通知同时声明时，其执行顺序：**

| 通知处理流程                     |
| -------------------------------- |
| 环绕通知-方法执行前              |
| 前置通知                         |
| 方法执行中                       |
| 后置通知（异常通知）             |
| 最终通知                         |
| 环绕通知-后（环绕通知-异常处理） |

**通知优先级和执行顺序：**

​		当一个切入点有多个同类型通知时，其优先级则根据IOC容器注册顺序决定；如果通知都在同个Bean中，优先级根据通知方法的声明顺序决定

​		优先级越高，增强代码越靠近目标源代码，即前置通知优先级越高，越后执行；后置通知优先级越高，越前执行

##### 3.2.3.4、通知方法的参数传递

​		所有通知方法都默认允许接收JoinPoint对象，用于获取连接点信息；并且需要作为第一参数

JoinPoint常用方法：

| 方法名           | 作用               |
| ---------------- | ------------------ |
| getArgs（）      | 返回方法参数       |
| getThis（）      | 返回代理对象       |
| getTarget（）    | 返回目标对象       |
| getSignature（） | 返回连接点方法信息 |

​		基于**args**切入点表达式，通知方法还可以直接接收连接点实参对象：

```java
	@Before("name()&&args(fixed)")
	public void beforeAop(JoinPoint joinPoint,String fixed) {
		System.out.println(fixed);
		System.out.println("前置通知");
	}
```

##### 3.2.3.4、使用Demo

**切面：**

```java
@Component
@Aspect
public class AopAspect {

	@Pointcut("execution(public * com.yh.common.demo.aop.AopTarget.getName(..))")
	public void name() {
	}

	@Before("name()")
	public void beforeAop(JoinPoint joinPoint) {
		System.out.println("前置通知");
	}

	@After(value = "name()")
	public void afterAop(JoinPoint joinPoint) {
		System.out.println("最终通知");
	}

	@AfterReturning(value = "name()", returning = "retVal")
	public void afterAop(JoinPoint joinPoint, Object retVal) {
		System.out.println("返回值:"+retVal);
		((Map<String, String>) retVal).put("name","yh");
		System.out.println("返回前通知");
	}

	@AfterThrowing(value = "name()",throwing = "e")
	public void afterAop(JoinPoint joinPoint, BusinessException e) {
		System.out.println("异常处理");
	}

	@Around(value = "name()")
	public Object around(ProceedingJoinPoint proceedingJoinPoint)  {
		System.out.println("环绕通知-方法执行前");

		Object proceed = null;
		try {
			proceed = proceedingJoinPoint.proceed();
		} catch (Throwable throwable) {
			System.out.println("环绕通知-处理异常");
		}
		System.out.println("环绕通知-方法执行前");
		return proceed;
	}
}
```

目标类：

```java
@Component
public class AopTarget {

	public Map<String, String> getName(String fixed) throws Exception{
		System.out.println("目标方法执行中");
		Map<String,String> map  = new HashMap<>();
		map.put("name",fixed+"Name");
		if (Objects.equals(fixed,"1")){
			throw new BusinessException("");
		}else {
			return map;
		}
	}
}
```

### 3.4、springAOP实际应用场景

## spring面试问题：

1、spring三级缓存：

- 一级缓存：存储完整Bean
- 二级缓存：存储未注入属性的Bean
- 三级缓存：使用该Bean的工厂对象，提供beanName、BD、未注入属性的Bean；（当进行获取后，则进行二级缓存；为了是在该缓存中判断是否存在循环依赖的AOP代理）

1、spring为什么要使用三级缓存解决循环依赖

- 本身只需要使用二级缓存就可以解决循环依赖，将为完成属性注入的Bean防止二级缓存中，从而保证循环依赖注入

- 但如果只使用二级缓存，来保存实例化对象和初始化完成对象:

  1. 创建A的实例化对象，放入二级缓存
  2. A进行AOP代理，此时由于依赖注入，则进行B的初始化
  3. 此时B进行依赖注入时，使用二级缓存中的A对象
  4. 此时A再进行B的依赖注入，此时又需要进行AOP代理，而导致A出现两个相同类型实例（B中依赖的A实例、A的代理）

  即会导致，进行aop代理后，B所依赖的A并不是代理实例，而是原生实例

- 当使用三级缓存时
  1. 创建A的实例对象，放入三级缓存，
  2. 进行代理时，依赖B，完成B的注入（当出现循环依赖时）使用三级缓存；
  3. 如果调用三级缓存，则此时就会判断A是否存在动态代理，存在则提前进行A的动态代理，然后转移到二级缓存中
  4. 保证B中存储A的动态代理对象引用
  5. B初始化完成后，开始进行A的动态代理

**即三级缓存机制是为了保证，存在AOP代理的循环引用正常进行**