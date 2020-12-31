Spring cloud（分布式(微服务)系统工具集）：

## 1、基础概念

**微服务架构：**

​	提倡将单一应用程序划分为一组小的服务，服务之间互相协调配合；每个服务运行在独立的进程中，服务之间采用轻量级的通讯机制，独立部署到生产环境，实现用户最终功能

**spring cloud：**

​	分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合，俗称微服务全家桶

### 1.1 、spring cloud常用技术：

- 微服务网关：

  ​	为所有微服务提供一个统一入口，并对所有请求进行鉴权校验；使用动态路由将请求

- 服务注册与发现
- 服务负载与调用
- 服务熔断降级
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

![](C:\Users\OneMTime\Desktop\笔记\Typora图片\springCloud组件选择.jpg)

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

## 3、微服务项目搭建步骤

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

  Ribbon提供Irule组件，来配置吸怪负载均衡策略

  注意：Ribbon的配置类，不能放在ComponentScan扫描包下，即**不能与sringBoot应用启动类同包**

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

### 3、Ribbon负载均衡策略

所有均衡负载策略都有一个算法配置类，统一接口为IRule：

```java
public interface IRule{

    public Server choose(Object key);//选择一个RestTmplate可用的远程调用的服务
    
    //set、get ILoadBalancer用于获取服务信息（服务总数、可用服务总数） 
    public void setLoadBalancer(ILoadBalancer lb);   
    public ILoadBalancer getLoadBalancer();    
}
```

- 轮询，RoundRobinRule（全局默认配置）

  rest接口第几次请求 % 服务提供者集群总数 = 实际调用服务提供者的下标

  服务重启后，rest接口请求次数会重置为1

- 随机，RandomRule

- 

### 4、自定义负载均衡策略

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

OpenFeign所有Http接口调用**默认等待1s**，超时则抛出调用超时异常

由于OpenFeign对于负载均衡是基于Ribbon，因此其超时控制也是有Ribbon完成：

- 日志打印：

  - 定义feign配置类：配置feign的日志级别，共有4种：NONE（无），BASIC（只打印状态码、执行时间、HTTP方法、URL）、HEADERS（在BASIC基础上，打印请求、响应Header）、FULL（在HEADERS基础上，打印请求、响应的body和元数据）

    ```java
    @Configuration
    public class FeignConfig {
    	@Bean
    	feign.Logger.Level feignLoggerLever(){
    		return feign.Logger.Level.FULL;
    	} 
    }
    ```

  - 





#####46







Ribbon对于服务集群的信息，则通过discvorey进行获取**

P40

springCloud和Dubbo的区别