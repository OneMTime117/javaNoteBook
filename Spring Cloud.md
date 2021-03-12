Spring cloud（分布式(微服务)系统工具集）：

## 1、基础概念

**微服务架构：**

​	提倡将单一应用程序划分为一组小的服务，服务之间互相协调配合；每个服务运行在独立的进程中，服务之间采用轻量级的通讯机制，独立部署到生产环境，实现用户最终功能

**spring cloud：**

​	分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合，俗称微服务全家桶

### 1.1 、spring cloud常用技术：

- 微服务网关：

  ​	为所有微服务提供一个统一入口，并对所有请求进行鉴权校验；使用动态路由将请求分发到不同的微服务模块

- 服务注册与发现

  ​	将所有微服务统一管理，感知它们的上下线，方便消费者能够远程调用正常的服务提供者，避免单点故障对微服务系统的影响

- 服务负载与调用

  ​	对服务的远程调用提供负载均衡与重试策略，更好的实现微服务系统的高可用；

  ​	提供高效、简洁的远程调用编码方式，提供开发效率

- 服务熔断降级

  ​	最大程度上保证微服务系统的高可用，避免在高并发下，单点故障导致整个微服务系统蔓延雪崩

- 服务分布式配置

- 服务总线

- 服务开发

### 1.2、spring cloud和spring boot版本选择

springboot使用数字进行版本命名，目前推荐使用2.x版本；

springcloud使用字母顺序命名（单词为伦敦地铁站名），目前推荐H版；

每当一个重大bug被解决，都会发布一个SR X后缀版本，表示service releases（修复），作为目前正式版；GA表示当前稳定版；SNAPSHOT（快照）表示为快照版，没有重大更新随时被修改；

**两者版本依赖大致版本对应关系:**

| springCloud | springBoot              |
| ----------- | ----------------------- |
| Hoxton      | 2.2.X，2.3.X（SR5开始） |
| Greenwich   | 2.1.X                   |
| Finchley    | 2.0.X                   |
| Edgware     | 1.5.X                   |

实际中，前者每一个小版本都有对应当前后者大版本中的一段区间小版本，实际中应根据官方文档进行范围确认：

**Hoxton.SR9---->2.3.5.RELEASE**

### 1.3、spring cloudH版，对于各个技术组件的选取：

![](F:\Desktop\javaNoteBook\Typora图片\springCloud组件选择.jpg)

## 2、springCloud项目简单底层搭建

- 建立父POM：

  ​	默认情况下，spring给我们提供了springboot、springcloud的父POM，但是并没有将两者整合一起，并且有时我们项目需要自定义一些非spring整合的包，因此可以给所有微服务定义一个共同的父POM，用于全局管理依赖版本

  其中springboot、springcloud的父POM是必须的：

  ```java
  	<!-- 定义全局变量,统一jar包版本 -->
  	<properties>
  		<spring.boot-version>2.3.5.RELEASE</spring.boot-version>
  		<spring.cloud-version>Hoxton.SR9</spring.cloud-version>
  		<spring.cloud.alibaba-version>2.2.1.RELEASE</spring.cloud.alibaba-version>
  	</properties>
  
  	<!-- 锁定子模块的依赖所有版本，在父POM中，用于子POM继承；子类不需要指定版本号；并且只用于声明依赖，并不会引用导入依赖 -->
  	<dependencyManagement>
  		<dependencies>
  			<!-- 定义springboot、springcloud、springcloud-Ali的父POM，用于定义其相关所有依赖版本 -->
  			<dependency>
  				<groupId>org.springframework.boot</groupId>
  				<artifactId>spring-boot-starter-parent</artifactId>
  				<version>${spring.boot-version}</version>
  				<type>pom</type>
  				<scope>import</scope>
  			</dependency>
  			<dependency>
  				<groupId>org.springframework.cloud</groupId>
  				<artifactId>spring-cloud-dependencies</artifactId>
  				<version>${spring.cloud-version}</version>
  				<type>pom</type>
  				<scope>import</scope>
  			</dependency>
  			<dependency>
  				<groupId>com.alibaba.cloud</groupId>
  				<artifactId>spring-cloud-alibaba-dependencies</artifactId>
  				<version>${spring.cloud.alibaba-version}</version>
  				<type>pom</type>
  				<scope>import</scope>
  			</dependency>
  		</dependencies>
  	</dependencyManagement>
  ```

- 建立微服务项目（Maven Module），定义POM文件

  ​	将主项目作为父POM引用，此时对于公共依赖，则可以直接在父POM中定义版本，而子项目中就直接声明groupId和artifactId

- 定义springboot配置文件

  ​	一般情况下，至少要定义server.port（服务端口）、spring.application.name（项目名）

- 定义springboot应用启动类

  ```java
  @SpringBootApplication
  public class OrderMain {
  	public static void main(String[] args) throws Exception {
  		SpringApplication.run(OrderMain.class, args);
  	}
  }
  ```

- 编写业务

  - 服务提供者

    ```java
    	@PostMapping("/payment/create")
    	public Result insertPayment(@RequestBody Payment payment,@RequestHeader("content-Type") String contentType) {
    		int addPayment = paymentService.addPayment(payment);
    		System.out.println(addPayment);
    		if (addPayment > 0) {
    			return ResultUtil.sussess("成功");
    		} else {
    			return ResultUtil.error("失败");
    		}
    	}
    ```

  - 服务消费者

    ```java
    	@PostMapping("/spring/create")
    	public Result createPayment(Payment payment) {
    		//调用者以键值对方式，接收参数，
    		//使用restTemplate进行http调用时，默认将参数作为请求body，
    		HttpEntity<Payment> httpEntity = new HttpEntity<>(payment);
    		return restTemplate.postForObject(url+"payment/create",httpEntity, Result.class);
    	}
    ```

    **整个微服务调用过程是基于RESTapi，使用springMVC提供了RESTClient来完成**

- 代码重构：

  实际上，对于多个微服务项目通用的代码，可以使用一个子项目进行封装，然后部署到maven仓库，最后进行作为依赖进行导入（一般情况下，所有项目导入所有相同依赖，这样方便依赖版本管理；而**理论上，作为依赖导入的项目中，其所有maven引入的包，都可以供其他微服务使用**）

### 微服务项目搭建步骤

- 搭建父POM项目，定义所需依赖以及对应版本（其中可以引入springBoot、springCloud对应的父POM依赖，从而方便实现版本其组件依赖的版本统一）
- 创建MavenModule项目，即微服务：
  - POM文件：指定父POM（创建Maven项目时就可以指定生成）、导入需要的依赖（由于父POM已经声明了所有的依赖版本，因此不需要指定version；如果指定则覆盖）
  - 主启动类：main方法、@SpringBootApplication注解、@EnableEurekaClient（非Eureka服务端实力；否则使用@EnableEurekaServer）
  - application配置文件：server.port（端口号）、spring.application.name（服务名）、数据源配置（单数据源、多数据源）、druid连接池配置、eureka配置

- 对于微服务中通用代码，可以单独使用一个MavenModule项目编写，然后在其他微服务pom中依赖引入（同样可以使用父POM统一指定版本）

## 3、Eureka组件

### 1、基本概念

#### 1.1、服务治理理念

​	在微服务调用过程中，每个服务之间都可以存在依赖关系，而随着服务的增多和集群化，对这种依赖关系的管理就变得负载，此时就需要使用**服务治理**，**实现服务发现、注册、调用、负载均衡和容错**

#### 1.2、Eureka系统架构

​	Eureka采用C/S架构，Eureka作为服务注册功能的服务器，微服务系统中，所有服务作为客户端，与Eureka进行心跳连接，从而实现Eureka对所有微服务的监控；

​	服务提供者会将自己接口的通讯地址注册到Eureka中，而服务消费者则通过接口别名在Eureka中获取实际服务的通讯地址，从而在本地实现RPC远程调用

#### 1.3、Eureka和Dubbo的区别

​	从架构上，两种没有本质的区别，同时通过一个观察者模式来实现服务注册，并管理服务之间的依赖关系。但Eureka在进行RPC远程调用时，是基于RESTapi实现；而Dubbo是基于更底层的网络通讯（dubbo协议等）

#### 1.4、Eureka组件

​	Eureka采用C/S架构，因此在整个API的使用中提供两个组件：

- Eureka Server，用于服务注册，使用注册表管理所有的可用服务节点信息

- Eureka Client，用于服务发现，访问Eureka 服务端获取服务注册表，并进行心跳连接

  因此整个服务注册发现过程如下：

  1. 启动eureka服务端，提供面向Eureka客户端的注册服务
  2. 启动服务提供者，将服务注册到eureka服务端的服务注册表中（以服务别名：RPC调用地址  键值对形式保存）
  3. 启动服务消费者，从eureka服务端中通过服务别名获取相应的RPC调用地址，保存在缓存中，每30s进行一次更新；当需要使用时，则使用HttpClient进行远程调用

1.5、Eureka心跳

默认心跳周期30s，最长心跳时间90s

### 2、Eureka单机服务搭建

1、Eureka服务端

- pom文件:Eureka服务端一般不会参与服务注册和调用，因此不进行逻辑代码编写，也就不需要引入其他依赖（只需要构建基本springBoot项目、eureka服务端组件）

```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
```

- 主启动类

```java
@EnableEurekaServer
@SpringBootApplication
public class EurekaApplication {
	public static void main(String[] args) throws Exception {
		SpringApplication.run(EurekaApplication.class, args);
	}
}
```

- application配置

```java
server.port=7001
#eureka服务端实例名
eureka.instance.hostname=localhost

#表示自身作为客户端，不注册到eureka服务注册表中(默认true)
eureka.client.register-with-eureka=false
#表示不会检索eureka服务注册表中服务实例(默认true)
eureka.client.fetch-registry=false

#eureka服务端访问地址
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka
```

2、Eureka客户端：

- pom文件

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

- 主启动类

```java
@EnableEurekaClient
@SpringBootApplication
public class OrderApplication {
	public static void main(String[] args) throws Exception {
		SpringApplication.run(OrderApplication.class, args);
	}
}
```

- application配置

```properties
server.port=80
spring.application.name=order

#eureka服务端访问地址
eureka.client.service-url.defaultZone=http://localhost:7001/eureka
```

**注意：**

1、eureka.client.service-url对应属性是一个键值对，提供一个默认key：defaultZone，默认value：http://localhost:8761/eureka

2、所有Eureka客户端都会被服务端管理，并且通过http://localhost:7001/进行管理页面访问

3、eureka.instance.hostname就是网络映射当前的网络地址，localhost、127.0.0.1都指向同一个IP，即本机IP；在Eureka客户端中，就需要指定真实的ip（当然同一台机器就可以都使用localhost）

4、在http://localhost:7001/页面中，所有Eureka客户端都会显示实例名：    IP名:应用名:应用端口名；当eureka客户端和服务端为同一个主机，则默认使用主机名，可以使用配置进行修改显示当前ip（**此为客户端配置**）

```properties
eureka.instance.prefer-ip-address=true
eureka.instance.ip-address=	192.168.0.138
```

自定义整个客户端实例名：（**这个只能修改eureka监控页面的名称，不能实现ip显示**，因此需要两个同时使用）

```properties
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
```

### 3、Eureka集群搭建

Eureka集群是为了实现服务注册中心的高可用，防止单点故障导致整个微服务应用崩溃，并能实现Eureka的负载均衡

Eureka集群中，每个节点都会互相注册管理，建立心跳连接

1、设置Eureka服务器集群各个实例名

```properties
server.port=7001
eureka.server.instance.hostname:eureka7001.com
```

2、Eureka服务器之间进行相互注册（即eureka服务端访问地址为其他服务端的集合，不用包含自己），并且也不需要作为客户端注册服务

```properties
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
#eureka服务端访问地址
eureka.client.service-url.defaultZone=http://eureka7002.com:7002/eureka
```

3、设置hosts网络映射配置（测试阶段，所有eureka服务都在同一台主机上时，实现一台主机拥有多个eureka.instance.hostname，就是在本地主机中，自定义域名地址），**如果为多主机，则可以省略**，并且Eureka服务器集群各个实例名直接使用其主机ip地址

文件：C:\Windows\System32\drivers\etc\hosts

```properties
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
127.0.0.1 eureka7003.com
```

4、设置所有客户端的eureka服务端访问地址（包含所有erurka集群）

```properties
#eureka服务端访问地址
eureka.client.service-url.defaultZone=http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
```

5、服务调用修改：

- 使用服务提供者的应用名，作为别名替代实际调用接口的ip+端口（别名不区分大小写）

```java
//url="http://localhost:8001/payment/user"	
private String url="http://PAYMENT/payment/user";
```

- 针对于服务消费者的RestTemplate组件，添加@LoadBalanced注解，实现eureka服务调用的负载均衡（其作用是在RestTemplate进行REST远程调用前，使用拦截器进行额外处理：别名替换、负载均衡）


```java
	@LoadBalanced
	@Bean
	public RestTemplate getRestTemplate() {
		return new RestTemplate();
	}
```

**注意：**

1、如果不添加@LoadBalanced注解，则RestTemplate无法基于eureka使用别名完成服务发现，原因是：不添加@LoadBalanced，则不会对url进行别名处理

2、当添加@LoadBalanced注解，则RestTemplate就无法使用实际服务访问地址，完成REST调用，原因是：添加@LoadBalanced后，会根据eureka服务端提供的服务注册表进别名处理，生成新的访问地址

### 4、Discovery服务发现

​	通过@EnableDiscoveryClient注解，开启Discovery功能，然后在任意一个微服务中，都可以暴露当前整个微服务注册中心eureka中的服务信息：

-  通过spring直接导入org.springframework.cloud.client.discovery.DiscoveryClient  Bean（注意导入正确包，是基于springcloud-client-discovery）

- @EnableEurekaClient默认带有开启@EnableDiscoveryClient注解的功能，因此可以省略@EnableDiscoveryClient注解

- DiscoveryClient  提供一系列API，来获取整个微服务系统中eureka保存的服务信息

  ```java
  List<ServiceInstance> instancesById = discoveryClient.getInstances("payment");
  for (ServiceInstance instanceInfo : instancesById) {
  	String hostName = instanceInfo.getHost();
  	String instanceId = instanceInfo.getInstanceId();
  }
  ```

### 5、Eureka自我保护

​	对于某个时刻微服务不可用时，Eureka并不会将该服务的注册信息全部清理，任然进行**一定时间的**保存（认为该微服务可用），这样就可以有效避免网络分区，导致服务可以可以使用的情况下，被舍弃（如服务提供者只是暂时和Eureka服务端网络不通，但和服务消费者网络通畅），即宁愿保存错误的注册信息，也不会误删可用服务（保证AP理论，可用、分区容错）

​	**自我保护触发方式：**

Eureka服务端默认最长心跳时间为90s，如果心跳超过该时间，则认为出现网络分区故障，而自我保护的触发条件为：15分钟内心跳失败比例低于85%

​	**自我保护关闭（默认开启）：**

```properties
#禁用自我保护
eureka.server.enable-self-preservation=false
#非自我保护模式下，心跳超时后自动清理间隔时间
eureka.server.eviction-interval-timer-in-ms=5000
```

​	**控制eureka心跳连接时间：**

在客户端中配置：

```properties
#心跳时间间隔(默认30s)
eureka.instance.lease-renewal-interval-in-seconds=30
#心跳过期时间(默认90s)
eureka.instance.lease-expiration-duration-in-seconds=90
```

Eureka自我保护缺点：eureka的自我保护机制会导致服务消费者拿到一个错误的服务实例进行消费，导致远程调用失败，因此就需要搭配服务调用的重试机制和服务熔断使用

## 4、Zookeeper组件（跳过）

## 5、Consul组件（跳过）

## 6、Ribbon组件

### 1、基础概念

​	Ribbon是有Netflix发布的开源项目，实现客户端服务调用的负载均衡，目前处于维护模式

**Load Balance：**

​	负载均衡，将用户请求平摊到多个服务处理，实现系统的高可用

**Ribbon和Nginx负载均衡的区别：**

​	Nginx为服务器负载均衡，客户端所有请求都会先交给Nginx，然后由Nginx实现请求转发，因此负载均衡是由服务端完成

​	Ribbon为本地负载均衡，在进行微服务远程调用时，实现对多个服务提供者远程调用的负载均衡

**因此Niginx是请求级别上的负载均衡；而Ribbon是服务级别上的负载均衡**

### 2、Ribbon组件搭建

- Ribbo默认在导入spring-cloud-starter-netflix-eureka-client包时，会自动依赖导入:

  spring-cloud-starter-netflix-ribbon包

- @LoadBalanced注解，实现服务调用的负载均衡（@LoadBalanced注解由spring.cloud.common 包提供，而实现方式根据负载均衡组件来实现，如Ribbon）

- Ribbon配置：

  Ribbon提供Irule组件，来配置相应负载均衡策略

  注意：Ribbon的配置类，不能放在ComponentScan扫描包下，即**不能与sringBoot应用启动类同包，如果同包后，则该配置类将作为Ribbon的全局负载均衡机制使用**

  ```java
  @Configuration
  public class RibbonConfig {
  	@Bean
  	public IRule getIRule() {
  		return new RandomRule();//随机
  	}
  }
  ```

  在启动类设置Ribbon负载均衡策略：

  name：指定消费的服务应用名 ，**name必须全部大写**（和eureka监控页面一致）

  configuration：通过@Bean注册均衡负载策略组件Irule实现类Bean的配置类

  **因此一个负载均衡策略对应一个配置类**

   ```java
  @RibbonClient(name = "PAYMENT",configuration = RibbonConfig.class)
  public class OrderApplication {
  	public static void main(String[] args) throws Exception {
  		SpringApplication.run(OrderApplication.class, args);
  	}
  }
   ```

### 3、Ribbon负载均衡策略配置方式：

**只针对于所有使用了@LoadBalanced注解定义的RestTemplate的接口调用**

- 通过配置注册IRule组件Bean，来定义Ribbon的全局负载均衡策略（RibbonConfig.class配置类需要自动注册到spring容器中）

- 通过@RibbonClient(name = "PAYMENT",configuration = RibbonConfig.class)，定义单个服务的负载均衡策略，并保证RibbonConfig.class配置类不会自动注册到spring容器中

- 基于配置文件定义Ribbon的单个微服务负载均衡策略：**（同理，服务名必须大写才会生效）**

  ```properties
  #PAYMENT 为服务名，value为策略类名
  PAYMENT.ribbon.NFLoadBalancerRuleClassName:com.ribbon.MyRibbonRule
  ```

注意：三种配置优先级：配置文件>@RibbonClient>全局>默认（轮询）

### 4、Ribbon负载均衡策略

所有均衡负载策略都有一个算法配置类，统一接口为IRule：

```java
public interface IRule{

    public Server choose(Object key);//选择一个RestTmplate可用的远程调用的服务
    
    //set、get ILoadBalancer用于获取服务信息（服务总数、可用服务总数） 
    public void setLoadBalancer(ILoadBalancer lb);   
    public ILoadBalancer getLoadBalancer();    
}
```

- RoundRobinRule（全局默认配置），简单轮询策略

  rest接口第几次请求 % 服务提供者集群总数 = 实际调用服务提供者的下标

  服务重启后，rest接口请求次数会重置为1

- RandomRule，随机策略

- BestAvailableRule，最大可用策略，先过滤不可用服务实例，然后选择当前并发请求数最小的

- WeightedResponseTimeRule，响应加权轮询策略，根据各服务实例的响应时间，进行加权处理。然后再采用轮询方式调用

- AvailabilityFilteringRule，可用过滤策略，先过滤不可用和并发请求过大的服务实例，然后对剩余部分实例进行线性轮询

- ZoneAvoidanceRule，区域感知策略，选择符合server所在区域的性能和server的可用性实例

### 5、自定义负载均衡策略

策略思想：让服务实例进行轮询，并每轮连续三次

- 自定义Rule组件

  ```java
  /**
   * @author yuanhuan
   * @date 2021年1月4日上午11:24:31
   * @Description 自定义ribbon负载均衡策略：每个服务轮询访问3次
   */
  public class MyRibbonRule extends AbstractLoadBalancerRule {
  	private AtomicInteger nextServerCyclicCounter;
  	// 定义该服务连续轮询次数
  	private AtomicInteger totle;
  
  	public MyRibbonRule() {
  		nextServerCyclicCounter = new AtomicInteger(0);
  		totle = new AtomicInteger(0);
  	}
  
  	public MyRibbonRule(ILoadBalancer lb) {
  		this();
  		setLoadBalancer(lb);
  	}
  
  	public Server choose(ILoadBalancer lb, Object key) {
  		if (lb == null) {
  //	            getLog().warn("no load balancer");
  			return null;
  		}
  
  		Server server = null;
  		int count = 0;
  		while (server == null && count++ < 10) {
  			List<Server> reachableServers = lb.getReachableServers();//可用服务
  			List<Server> allServers = lb.getAllServers();//所有注册服务
  			int upCount = reachableServers.size();
  			int serverCount = allServers.size();
  
  			if ((upCount == 0) || (serverCount == 0)) {//没有服务可用
  				return null;
  			}
  
  			//获取当前轮询的服务实例
  			int nextServerIndex = myincrementAndGetModulo(serverCount);
  			server = allServers.get(nextServerIndex);
  
  			if (server == null) {
  				Thread.yield();
  				continue;
  			}
  
  			if (server.isAlive() && (server.isReadyToServe())) {
  				return (server);
  			}
  
  			// Next.
  			server = null;
  		}
  
  		if (count >= 10) {
  //	            getLog().warn("No available alive servers after 10 tries from load balancer: " + lb);
  		}
  		return server;
  	}
  
  	/**
  	 * 具体实现算法。可能写的不好，实现效果
  	 */
  	private int myincrementAndGetModulo(int modulo) {
  		for (;;) {
  			int current = nextServerCyclicCounter.get();//获取当前服务轮询的次数
  			int next = (current + 1) % modulo;
  			// 预期一样的话就把value更新为next
  			if (totle.get() < 3 && nextServerCyclicCounter.compareAndSet(current, current)) {
  				// 获取并自增
  				totle.getAndIncrement();
  				if (totle.get() == 3) {
  					// 重新赋值
  					totle.getAndSet(0);
  					nextServerCyclicCounter.getAndSet(next);
  				}
  				return next;
  			}
  		}
  	}
  
  	public Server choose(Object key) {
  		return choose(getLoadBalancer(), key);
  	}
  
  	public void initWithNiwsConfig(IClientConfig clientConfig) {
  	}
  }
  ```

- 然后进行负载均衡策略配置（全局或指定某个服务的负载均衡策略）

### 6、超时控制、重试机制

同负载均衡策略配置相同，Ribbon提供服务和全局的参数配置：

```properties
ribbon.ConnectionTimeout=1000   //连接超时时间，默认1000
ribbon.ReadTimeout=5000			//数据传输超时时间（并不是响应时间），默认1000
ribbon.MaxAutoRetries=1			//同一台实例最大重试次数,不包括首次调用，默认1
ribbon.MaxAutoRetriesNextServer=1 //切换实例后的最大重试次数,不包括首次调用，默认0
ribbon.OkToRetryOnAllOperations=false //默认false,只会对GET请求进行重试(防止幂等操作)

PAYMENT.ribbon.ConnectionTimeout=1000  //单个服务的配置
```

- Ribbon的重试机制仅仅只是针对于REST的调用（请求连接和传输超时、报错；或返回指定需要重试的状态码）

## 7、OpenFeign组件

### 1、基础概念

- Feign：是一个声明式的HTTP客户端，让HTTP客户端的编写更加简单，并且**集成了Ribbon进行负载均衡，搭配eureka更方便的进行微服务调用**；（相对于Ribbon，将restTempate的接口调用方式，转变为面向接口编程）
- OpenFeign，是springCloud在Feign的基础上的增强版，基本代替了Feign进行使用；

- Feign和OpenFeign的区别：

  Feign是springCloud组件中的轻量化RESTful风格的Http服务客户端，并且Feign内置了Ribbo实现负载均衡；通过注解定义Http服务端接口，然后将Http服务端接口的调用，转变为面向接口编程方式；

  OpenFeign是springCloud在Feign的基础上支持springMVC相关注解，方便搭配springMVC使用

### 2、OpenFeign的使用：

- 导入相关依赖：

  ```xml
  <dependency>
  	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-starter-openfeign</artifactId>
  </dependency>
  ```

- 主启动类开启@EnableFeignClients注解

- 将需要调用的Http服务（@Controller方法）定义为接口，并完全符合springMVC注解形式，并声明微服务名称

  ```java
  //feign服务调用接口
  @Component
  @FeignClient("payment")//对应调用的服务名
  public interface PaymentFeignService {
  	@GetMapping("/user")
  	R getUser(@RequestParam("id") String id);
  }
  ```

  注意：默认情况下，每一个@FeignClient注解对应一个微服务，也就对应一个接口类，不能存在一个接口对应相同微服务；即不能存在@FeignClient注解的value值相同（**可以修改Feign配置或手动进行feign接口代理来实现**）

- 以接口形式编程，调用REST服务：

  ```java
  @Autowired
  PaymentFeignService feignService;
  //基于feign进行Http接口调用	
  @GetMapping("/feign/user")
  public R getFeignUser(@RequestParam String id) {
  	R user = feignService.getUser(id);
  	return user;
  }
  ```

**OpenFeign在使用springMVC注解时，注意事项：**

- @RequestParam、@PathVariable等注解的value不能省略
- 不能使用@RequestMapping注解在类上，用于拼接url
- 无法使用@ModelAttribute注解，实现复杂对象接受多个简单参数；如果使用复杂对象，则默认使用POST请求，通过json序列化方式解析body数据（因此对于多参数，必须使用@RequestParam、@PathVariable）

注意：在feign在接口中的springMVC注解时，所有value值都不能省略（如，而在controller方法中却可以省略）

### 3、OpenFeign的超时控制和日志打印：

- 超时控制：

OpenFeign所有Http接口调用**默认等待1s**，超时则抛出调用超时异常

由于OpenFeign对于负载均衡是基于Ribbon，因此其超时控制也是由Ribbon完成；（参考Ribbon的属性配置），其自身有超时控制，但是由于ribbon和Hystrix就存在超时冲突，因此默认取消

- 日志打印：

  - 定义feign配置类：配置feign的日志级别，共有4种：

    - NONE（无，不打印）
    - BASIC（只打印状态码、执行时间、HTTP方法、URL）
    - HEADERS（在BASIC基础上，打印请求、响应Header）
    - FULL（在HEADERS基础上，打印请求、响应的body和元数据）

    ```java
    @Configuration
    public class FeignConfig {
    	@Bean
    	feign.Logger.Level feignLoggerLever(){
    		return feign.Logger.Level.FULL;
    	} 
    }
    ```

  - 在服务消费者中，将feign接口包的日志级别改为debug（实现日志框架对其日志的打印）

    ```properties
    logging.level.com.yh.feign.service:debug
    ```

### 4、OpenFeign的负载均衡

OpenFeign负载均衡完全使用Ribbon来实现，但需要注意：

配置Ribbon某个服务的负载均衡时，需要保证服务名和@FeignClient注解中的服务名都为大写（**遵循Ribbon服务名大写的配置原则**），否则feign的负载均衡将使用默认负载均衡策略

## 8、Hystrix组件

### 1、基本概念

- 分布式系统面临的问题

  ​	对于复杂的分布式体系结构中，应用程序可能会存在数十个依赖关系，而这些依赖不可避免的会发生调用失败的问题；而当其中一个服务响应过长或不可用后，在高并发下就会导致服务雪崩

  ​	Hystrix就是用于解决该问题，是用于处理分布式系统的延迟和容错的开源框架，保证在微服务调用过程中，如果出现其中一个依赖调用超时或异常，不会导致整个服务失败，避免级联故障，提供分布式系统的弹性

- 断路器

  ​	Hystrix提供断路器概念，当某个服务单元发生故障后，通过断路器进行故障监控（类似于电源保险丝），向调用方返回一个预期、可处理的备选响应（FallBack），而不是长时间等待或抛出无法处理的异常，这样就能保证调用方线程不会一直阻塞，避免故障在分布式系统中的蔓延，防止服务雪崩

- Hystrix功能：

  - 服务降级：FallBack

    服务降级是面对服务调用的处理方式，既可以被动（如果服务调用发生故障，则进行降级处理）；也可以主动（当触发服务熔断后，默认在一段时间内，对服务调用进行降级处理）

    触发场景：针对于当前服务调用，发生程序运行异常、运行超时、线程池/信号量打满和服务熔断

  - 服务熔断：break

    当服务调用失败次数达到最大后，触发熔断，并在一段时间内对服务进行降级处理；并在之后可以尝试恢复，关闭服务降级

  - 服务限流：flowlimit

    对于秒杀高并发场景，保证最大服务调用数，超出的请求进行排队或丢弃处理

  - 服务故障实时监控

  **服务熔断和降级的区别：**

### 2、Hystrix的使用

- 额外添加Hystrix依赖

  spring-cloud-starter-netflix-hystrix包

- 启动类添加开启熔断器（Hystrix）功能的注解：

  @EnableCircuitBreaker/@EnableHystrix

  @EnableHystrix是对@EnableCircuitBreaker的封装，两者功能一致

- 通过@HystrixCommand注解配置指定方法出现异常、或超时后，执行fallback方法，保证当前方法能够正常返回

  ```java
  	@HystrixCommand(fallbackMethod = "hystrixTestFallback", commandProperties = {
  			@HystrixProperty(name = "execution.timeout.enabled", value = "true"),
  			@HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds", value = "2000"),
  			@HystrixProperty(name = "circuitBreaker.enabled", value = "true"),
  			@HystrixProperty(name = "circuitBreaker.sleepWindowInMilliseconds", value = "5000") })
  	@GetMapping("/hystrixTest")
  	public R HystrixTest(String id) {
  		if ("1".equals(id)) {
  			throw new RuntimeException();
  		} else if (Objects.equal(1, id)) {
  			try {
  				Thread.sleep(3000);
  			} catch (InterruptedException e) {
  				e.printStackTrace();
  			}
  		}
  		return R.ok();
  	}
  
  	//fallback方法
  	public R hystrixTestFallback(String id) {
  		return R.error("已超时，请重试");
  	}
  
  ```

注意：

1、@HystrixCommand服务降级只是针对于方法的执行，本身和微服务系统没有必要的联系，因此既可以放在服务提供方，也可以放在服务调用方；但一般情况下，放在服务调用方上进行处理，原因是：消费者更加清楚对当前调用的服务限制要求（等待时间，调用失败后的fallback操作）和fallback处理方式

2、@HystrixCommand中fallback方法的返回值和入参要与原监控降级方法一致；并且不能使用在抽象方法上

### 3、Hystrix的使用优化

​	按正常的@HystrixCommand的使用方式，每个服务的接口方法都需要对应一个@HystrixCommand注解和fallback方法，这样大大增加了相似代码量；因此需要对所有的服务接口方法进行分类：**通用全局降级方法和独立的自定义降级方法**

- 全局通用降级方法使用：

  ```java
  @DefaultProperties(defaultFallback = "paymentGlobelFallbackMethod")
  @RestController
  @RequestMapping("/order")
  public class OrderController {
  
  	//feign接口服务，并使用hystrix进行服务容错	
  	@Autowired
  	PaymentFeignService service;
  	
  
  	@HystrixCommand
  	@GetMapping("/feign/userWaitTime")
  	public R getFeignUserWaitTime(@RequestParam String id) {
  		R user = service.getHystrixUserWaitTime(id);
  		return user;
  	}
  	
  	public R paymentGlobelFallbackMethod() {
  		return R.ok("payment服务全局异常处理");
  	}
  }
  ```

  此时，paymentGlobelFallbackMethod全局fallback方法不需要与原方法入参一致

  **但这种方式只解决的代码过多的问题，但是fallback方法会和调用者本地逻辑代码耦合**

### 4、基于feign进一步使用优化

​	对于feign的引入，就很好的将远程调用代码和本地service代码解耦分离，同样的Hystrix的fallback方法也同样可以配合feign，与本地代码解耦，将fallback方法针对于每个微服务模块的feign接口进行全局处理：

- 实现feign接口，编写指定fallback方法

```java
@Component
public class PymentFeignServiceFallback implements PaymentFeignService {
    
	public R getHystrixUser(String id) {
		return R.ok("没有找到，请稍后重试");
	}

	public R getHystrixUserWaitTime(String id) {
		return R.ok("没有找到，请稍后重试");
	}
}
```

- 服务调用方，修改配置文件

```properties
#开启feign对hystrix的支持,运行hystrix的fallback方法能针对于feign接口实现
feign.hystrix.enabled=true
```

- 在feign接口的@FeignClient注解上，指定fallback实现类

```java
@FeignClient(value="PAYMENT",fallback = PymentFeignServiceFallback.class)
```

### 5、Hystrix原理

Hystrix提供一个核心对象：HystrixCommand/HystrixObservableCommand命令对象，基于命令模式来实现对指定方法的调用与监控，并在底层使用了大量的RxJava响应式编程（观察者---订阅者）

#### 工作流程：

1、HystrixCommand/HystrixObservableCommand分别提供两种调用方式，是执行整个Hystrix的功能入口

- HystrixObservableCommand：
  - toObservable（）方法：当对依赖项订阅后，返回一个观察者对象（Observable），并且Hystrix命令不会立即执行，而是当所有订阅者订阅后才执行
  - observeable（）方法：相对于toObservable()，在调用后会直接执行Hystrix命令

- HystrixCommand：
  - queue（）方法，toObservable().toBlocking().toFuture()，异步执行获取future对象
  - execute（）方法，queue().get()，同步执行获取结果

一般情况下，对外就是使用HystrixCommand来完成hystrix对依赖（调用方法）的异步、同步命令处理;而Hystrix也只提供HystrixCommand方式的直接@HystrixCommand

2、Hystrix对于多个相同依赖，可以使用同一个HystrixCommand对象，即缓存机制，来直接返回结果

3、如果没有缓存，则判断熔断器状态：开启（熔断发生），则直接调用getFallback（）方法，进行降级处理；关闭，则进行下一步判断

4、判断当前依赖隔离的线程池和信号量是否已满（当前方法调用的并发量是否达到指定值），是，则进行降级处理；没有，则执行run（）方法，

5、在run（）方法执行过程中，如果执行失败或超时，则进行降级处理

#### Hystrix依赖隔离：

Hystrix提供两种方式资源隔离计算，来单独限制每个依赖的并发量，防止请求阻塞，导致服务器线程资源全部浪费在该依赖上：

- 线程池资源隔离

  对于每一个依赖，其HystrixCommand都运行在一个单独的线程中，并通过其依赖的线程池进行并发控制，适合大部分依赖调用情景，如网络请求（远程调用），进行超时处理；缺点：隔离线程和调用线程的切换，增加了CPU的损耗

- 信号量资源隔离

  对于那些内部逻辑复杂，耗时长的依赖，其HystrixCommand在当前请求的线程中执行，通过信号量大小来进行并发控制，避免线程切换，同时也就不能支持超时调用、异步调用；但是可以动态调整并发量大小，

### 6、Hystrix配置

Hystrix的服务熔断建立在服务降级基础之上，之上触发条件有所不同,Hystrix配置如下：

1、Execution，用于控制Hystrixcommand.run()如何执行

| 参数                                                | 作用                                                         | 默认值 |
| --------------------------------------------------- | ------------------------------------------------------------ | ------ |
| execution.isolation.strategy                        | 隔离策略：THREAD（线程隔离），在单独的线程上执行，并发请求受该线程池的线程数影响；SEMAPHORE（信号隔离），在方法调用的线程上执行，并发请求受信号量限制 | THREAD |
| execution.isolation.thread.timeoutInMilliseconds    | 超时时间                                                     | 1000ms |
| execution.timeout.enabled                           | 是否开启超时中断                                             | true   |
| execution.isolation.thread.interruptOnTimeout       | 发生超时时，是否中断（Thread有效，SEMAPHORE必须执行完成，才会判断超时） | true   |
| execution.isolation.thread.interruptOnCancel        | 发生取消是，是否中断（Thread有效）                           | false  |
| execution.isolation.semaphore.maxConcurrentRequests | 并发最大请求数（SEMAPHORE有效）                              | 10     |

2、Fallback，控制HystrixCommand.getFallback()如何执行

| 参数                                               | 作用                                                   | 默认值 |
| -------------------------------------------------- | ------------------------------------------------------ | ------ |
| fallback.enabled                                   | 是否开启执行备用方法                                   | true   |
| fallback.isolation.semaphore.maxConcurrentRequests | 运行执行getFallback()方法的最大并发数（SEMAPHORE有效） | 10     |

3、CircuitBreaker，控制HysertixCircuitBreaker（断路器）行为

| 参数                                     | 作用                                                         | 默认值 |
| ---------------------------------------- | ------------------------------------------------------------ | ------ |
| circuitBreaker.enabled                   | 开启断路器功能                                               | true   |
| circuitBreaker.requestVolumeThreshold    | 熔断触发的最小失败数  N/10s                                  | 20/10s |
| circuitBreaker.errorThresholdPercentage  | 熔断触发的最小失败率 %                                       | 50     |
| circuitBreaker.sleepWindowInMilliseconds | 熔断重试窗口期                                               | 5000   |
| circuitBreaker.forceOpen                 | 熔断器强制打开（跳闸），拒绝服务调用（优先级高于forceClosed） | false  |
| circuitBreaker.forceClosed               | 熔断器强制关闭，允许服务调用失败，不进行熔断处理             | false  |

**其他配置基本不会使用，需要则可以查看官方文档**

#### Hystrix配置方式

在搭配feign使用时，对于feign接口的hystrix参数配置，只能使用全局配置进行设置：

```properties
#Hystrix配置
hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds=5000
```

而当使用@HystrixCommand注解时，除了在该注解中声明配置属性外，也可以使用配置文件：

```java
#Hystrix配置
hystrix.command.xxxx.execution.isolation.thread.timeoutInMilliseconds=5000
```

xxxx为@HystrixCommand的commandKey属性属性值

**注意：**

- **Ribbon的超时时间必须小于Hystrix的超时时间，否则Ribbon会提前抛出超时异常，导致Hystrix不能进行服务降级处理**

### 7、Hystrix的服务图形监控界面

​	Hystrix提供准实时的调用监控，Hystrix Dashboard,来持续性记录所有通过Hystrix发起的请求的执行信息，并以表报和图形的形式展现给用户，如每秒执行多少请求、成功多少、失败多少等；

​	如果Hystrix在服务消费者进行处理，则就对服务消费者进行监控，**因此对于使用feign整合Hystrix，则需要对服务消费者进行监控**

- 导入图形化工具包：hystrix-dashboard

  ```properties
  <dependency>
  	<groupId>org.springframework.cloud</groupId>
  	<artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
  </dependency>
  ```

- 启动类 :添加@EnableHystrixDashboard额外注解

  ```java
  @EnableHystrixDashboard
  @SpringBootApplication
  public class HystrixDashboardApplication {
  	
  	public static void main(String[] args) throws Exception {
  		SpringApplication.run(HystrixDashboardApplication.class, args);
  	}
  }
  ```

- 对于监控的微服务，需要导入actuator服务监控驱动包，从而为hystrix-dashboard提供监控数据

- 在新版本springcloud中，没有提供hystrix.stream的监控servlet端口，因此需要在被监控的微服务中手动配置:

  ```java
  	// springCloud新版本，需要手动创建hystrix监控页面的servlet
  	@Bean
  	public ServletRegistrationBean getServlet() {
  		HystrixMetricsStreamServlet hystrixMetricsStreamServlet = new HystrixMetricsStreamServlet();
  		ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(hystrixMetricsStreamServlet);
  		servletRegistrationBean.setLoadOnStartup(1);
  		servletRegistrationBean.addUrlMappings("/hystrix.stream");
  		servletRegistrationBean.setName("HystrixMetricsStreamServlet");
  		return servletRegistrationBean;
  	}
  ```

- 通过localhost:9001/ hystrix进入hystrix-dashboard配置页面，然后添加监控端口：http://localhost:82/hystrix.stream

？？ 目前，配置页面正常、hystrix.stream返回值正常，但监控页面报错

## 9、Gateway组件

### 1、基本概念

- Gateway，新一代API网关，用于代替zuul，原因是:

1、zuul1.x,在springCloud F版作为推荐网关，其底层是基于Servlet2.5的阻塞式web架构（每个请求创建一个线程），并且不支持长连接（如websocket），和Nginx类似只是使用java实现，因此性能相对较差；zuul2.x,提供了基于Netty非阻塞和长连接支持，但是出来太慢，被springCloud推出的Gateway代替

2、Gateway由springcloud推出，基于springBoot2.x、使用了spring5的新特性：spring WebFlux、Reactor-Netty，通过Netty实现**异步非阻塞式**web框架，**利用多核 CPU 的硬件资源去处理大量的并发请求，提升系统的吞吐量和伸缩性，但并不会使接口请求响应时间缩短**，因此可以利用在高并发、磁盘IO密集型,、网络IO密集型的使用场景，如api网关，从而提升转发下游服务的吞吐量；**并且支持websocket**

- Gateway功能：

  提供简单有效的方式，对api进行路由，并基于Filter链提供一些强大的过滤器功能，如安全、监控和限流

  - Route（路由）：由ID、目标URI和一系列断言和过滤器组成，通过路由在来讲请求点位到真正的服务节点，并在转发的前后，进行精细化控制，并且**屏蔽所有微服务对外暴露的端口，统一使用gateway的端口，实现线程内的反向代理**
  - Predicate（断言）：JDK8中，提供java.util.function.Predicate，用于实现匹配HTTP请求中所有内容的断言，用于匹配路由
  - Filter（过滤）：通过过滤器，在请求被路由前后，进行修改

- GateWay对请求的处理流程：
  1. 客户端发起请求，交给GateWay处理，通过GateWayHandlerMapping找到与请求相匹配的路由
  2. 然后通过过滤链，对请求进行额外处理（可以在路由转发请求前后执行，即被代理业务逻辑的执行前后）
  3. 在转发前可以实现：参数校验、权限校验、流量监控、日志输出、协议转换等
  4. 在转发后可以实现：响应修改、日志输出、流量监控等

### 2、Gateway的使用

- 搭建GateWay微服务：

  - POM：

    gateWay本身就是一个网关微服务，因此需要导入eureka依赖（**但是不能导入spring-web模块，因为gateWay内部本身就已经基于spring-web模块搭建，官方默认认为两者会冲突**），然后额外导入spring-cloud-starter-gateway包

  - 主启动类：

    ```java
    @EnableEurekaClient
    @EnableDiscoveryClient
    @SpringBootApplication
    public class GateWayApplication {
    	public static void main(String[] args) throws Exception {
    		SpringApplication.run(GateWayApplication.class, args);
    	}
    }
    ```

  - 配置文件：

    ```properties
    server.port=9527
    spring.application.name=Gateway
    
    eureka.client.register-with-eureka=true
    eureka.client.fetch-registry=true
    
    #单机
    #eureka.client.service-url.defaultZone=http://localhost:7001/eureka
    #集群，注册到所有eureka服务端中
    eureka.client.service-url.defaultZone=http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
    eureka.instance.instance-id=gateway9527
    
    #gateway网关配置，可配置多个
    spring.cloud.gateway.routes[0].id=pyment_route
    spring.cloud.gateway.routes[0].uri=http://localhost:8001
    spring.cloud.gateway.routes[0].predicates=Path=/**
    ```

- 此时，就可以直接通过localhost：9527/xxxx，直接转发请求到指定的微服务中：localhost:8001/xxxx

### 3、Gateway基于硬编码实现网关配置

通过RoutLocator（路由定位器）对象，来对Gateway进行相应网关配置，并且**可以和配置文件的路由配置同时存在**

```java
@Configuration
public class GatewayConfig {
	@Bean
	public RouteLocator getRouteLocator(RouteLocatorBuilder builder) {
		// 创建路由
		Builder routes = builder.routes();
        //path为localhost:9527后面的路径,uri为转发地址
		RouteLocator routeLocator = routes.route("baidu_route", r -> r.path("/free").uri("http://www.doyoudo.com")).build();
		return routeLocator;
	}
}
```

### 4、Gateway整合eureka和ribbon，实现动态路由

#### 1、基本概念：

- 前提背景：

  ​	在没有网关前，所有微服务的远程调用都是通过eureka服务发现与ribbon负载均衡来实现。但现在添加gateWay后，所有RESTapi的请求都需要经过网关，因此远程调用也就需要被网关处理后，进行转发

- 结论：

  ​	因此，对于网关路由配置，就不再针对于某个ip、端口（服务器）进行，而是通过微服务名，进行动态路由配置。从而实现请求被网关拦截处理的同时，ribbon进行负载均衡，即gateway指定当前请求转发给某个微服务，而erurka和ribbon在该微服务集群中，选择一个服务器

#### 2、整合实现：

- 修改配置文件：


```properties
#gateway基于微服务的动态网关配置
spring.cloud.gateway.discovery.locator.enabled=true
```

- 配置PAYMENT微服务动态路由：

```properties
spring.cloud.gateway.routes[0].id=pyment
# lb://表示使用Ribbon负载均衡的方式来进行实际ip端口路由转发  PAYMENT表示通过eureka服务名代替实际的ip端口
spring.cloud.gateway.routes[0].uri=lb://PAYMENT   
spring.cloud.gateway.routes[0].predicates=Path=/payment/**
```

### 5、Predicate断言的使用

Gateway除了使用**Path匹配url外**，可以通过predicate断言，进行一些默认提供的请求拦截处理：

- 通过请求时间来限制：

  指定时间格式需要为：2021-02-25T18:28:22.421+08:00[Asia/Shanghai]，可以调用API：

  ```java
  ZonedDateTime now = ZonedDateTime.now();
  System.out.println(now);
  ```

  - After   请求时间必须在指定时间后
  - Before  请求时间必须在指定时间前
  - Between 请求时间必须在指定时间之间

- 通过HTTP规则来限制：
  - Cookie  指定Cookie内容
  - Header  指定HTTP请求头
  - Method  指定HTTP的方法
  - Host  指定域名路径
  - Qurey  指定接口参数

配置文件Predicate多种限制同时使用的编写规则：

- properties

```properties
#gateway网关配置
spring.cloud.gateway.routes[0].id=pyment
# lb://表示使用Ribbon负载均衡的方式来进行实际ip端口路由转发  PAYMENT表示通过eureka服务名代替实际的ip端口
spring.cloud.gateway.routes[0].uri=lb://PAYMENT
spring.cloud.gateway.routes[0].predicates[0]=Path=/payment/**
spring.cloud.gateway.routes[0].predicates[1]=After=2021-02-25T18:28:22.421+08:00[Asia/Shanghai]
```

- yml（配置更加简洁）

```yaml
serer:
  port: 9527
  
spring:
  application:
    name: Gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
      - id: payment
        uri: lb://PAYMENT
        predicates:
        - Path=/payment/**
        - After=2021-02-25T18:28:22.421+08:00[Asia/Shanghai]
    
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001:7001/eureka,http://eureka7002:7002/eureka
  instance:
    instance-id: gateway9527
```

### 6、Filter过滤器使用

Gateway提供一系列默认内置Filter，分为两类：

- GatewayFilter：单一过滤器，只针对于某个路由转发的请求，一般直接使用Gateway内置提供的。和predicate类似，通过提供一系列**GatewayFilterFactory**，在配置文件中使用

  常用GatewayFilter工厂如下：

  - AddRequestHeaderGatewayFilterFactory，添加请求头

    ```yml
    filters:
    	- AddRequestHeader=foo, bar-{segment}
    ```

  - AddResponseHeaderGatewayFilterFactory，添加响应头

    ```yaml
    filters:
        - AddResponseHeader=X-Response-Red, Blue
    ```

  - AddRequestParameterGatewayFilterFacotry，添加请求K-V参数

    ```yaml
    filters:
        - AddRequestParameter=color, blue
    ```

  - DedupeResponseHeaderGatewayFilterFactory，删除重复响应头，有时网关和下游服务会设置相同的请求头，因此需要删除重复的ResponseHeader：

    可以指定多个，使用空格分隔，并且可以指定可选参数strategy，用逗号隔开：

    RETAIN_FIRES（保留第一个，即下游服务的），RETAIN_LAST(保留最后一个，即网关的)

    ```yaml
    filters:
    	- DedupeResponseHeader= Access-Control-Allow-Credentials Access-Control-Allow-Origin ，RETAIN_FIRST
    ```

    

    

  | 工厂名                                   | 关键字（用于配置）   | 作用                                                         | 参数        |
  | ---------------------------------------- | -------------------- | ------------------------------------------------------------ | ----------- |
  | AddRequestHeaderGatewayFilterFactory     | AddRequestHeader     |                                                              | name，value |
  | AddResponseHeaderGatewayFilterFactory    | AddResponseHeader    | 添加响应头                                                   | name,value  |
  | AddRequestParameterGatewayFilterFacotry  | AddRequestParameter  | 添加请求KV参数                                               |             |
  | DedupeResponseHeaderGatewayFilterFactory | DedupeResponseHeader | 删除重复的响应头（有时可能网关和下游服务会同时添加相同的响应头，因此就可以选择保留第一个或最后一个，默认保留第一个，即下游服务的） |             |
  |                                          |                      |                                                              |             |
  |                                          |                      |                                                              |             |

  配置方式：

  关键字=key，value

  ```yml
  spring:
    cloud:
      gateway:
        routes:
        - id: add_request_header_route
          uri: https://example.org
          filters:
          - AddRequestHeader=X-Request-color, blue
          - AddResponseHeader=X-Response-color, red
  ```

  

- GlobalFilter：全局过滤器，针对所有路由转发的请求，一般用于开发者自定义，实现日志记录、网关鉴权等

feign的远程调用，本质上和网关gateway没有联系，gateway只是对外进行反向代理，而feign是通过服务注册中心来进行http api的远程调用







Gateway路由转发的负载均衡策略






P73

springCloud和Dubbo的区别

Ribbon对于服务集群的信息，则通过discvorey进行获取**