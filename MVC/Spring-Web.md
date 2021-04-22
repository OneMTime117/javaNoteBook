# Spring-Web

**spring-web模块包括三个部分：SpringMVC、Spring WebFlux、Websocket**

## springMVC：

- springMVC的依赖包：

  - spring-webMVC包：该包会依赖导入spring-web，一般情况下两者需要同时使用
  - servlet-api.jar：springMVC是基于servlet api来实现的

  **spring-webmvc和spring-web两个依赖包的作用：**

  ​       spring-web提供了核心HTTP集成，和一些便捷的servlet过滤器，可以整合其他基础的web框架技术（Hessian，Burlap）

  ​       spring-webmvc是对springMVC功能的实现，依赖于spring-web；一般只需要显式添加spring-webmvc依赖

- springMVC基本概念：

  springMVC是基于Servlet API构建的原始Web框架，使得开发者能够简单高效地实现了MVC分层架构，并且有很强地扩展性

### 1、前端控制器：

​		springMVC和其他web框架一样，围绕前端控制器模式来操作servlet API；**DispatcherServlet**就是springMVC的前端控制器；并且springMVC搭配IOC容器，完成所有对象的管理

#### springMVC启动过程：

- web容器初始化过程：

web容器启动时，会先初始化Listener，执行其contextinitialized方法；然后初始化filter，执行init方法；最后初始化servlet，执行init方法；从而完成整个web容器的初始化过程

- IOC容器初始化过程：

springMVC提供ContextLoaderListener、DispatcherServlet来伴随web容器的初始化，完成整个IOC容器的初始化

1、web容器初始化ContextLoaderListener，执行contextinitialized方法，使用ContextLoader读取ServletContext中的初始化参数（web容器的初始化参数），创建Root webApplicationContext（spring容器）；最后将创建好的webApplicationContext放入ServletContext中属性中（web容器中）

2、web容器进一步初始化DispatcherServlet，执行init方法：

​	1、获取ServletContext中的webApplicationContext,作为父容器（spring容器）

​	2、然后读取DispatcherServlet属性，创建一个新的webApplicationContext，作为子容器（springMVC容器）

​	3、之后执行DispatcherServlet其他初始化方法，在子容器中注册多个Bean，实现DispatcherServlet功能

#### spring容器和springMVC容器关系：

- springMVC容器为子容器，可以访问父容器中的Bean；而父容器不能访问子容器中的Bean；

- 父容器通过ApplicationContext来自动注入获取；
- 子容器通过webApplicationContext自动注入获取
- WebApplicationContext是ApplicationContext的子接口

- WebApplicationContext子容器可以获取到父容器上下文：


```java
ApplicationContext parent = webApplicationContext.getParent();
```

- springMVC提供**RequestContextUtils**工具类，可以通过reuqest获取子容器：

```java
RequestContextUtils.findWebApplicationContext(request);
```

- 通过依赖注入，在Bean中可以直接获取子容器webApplicationContext、父容器ApplicationContext对象

```java
	@Autowired
	WebApplicationContext webApplicationContext;

	@Autowired
	ApplicationContext ApplicationContext;
```

#### dispatcherServlet配置：

- xml方式：

```xml
<!-- 配置servelt容器初始化参数 -->
<context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>classpath:spring-mybatis.xml</param-value>
</context-param>

<!-- 配置servelt容器监听器，当项目启动时，自动初始化spring容器 -->
<listener>
	<listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>

<!-- 配置dispatcherServlet参数 -->
<servlet>
	<servlet-name>springMVC</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:springMVC-config.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup><!-- 让springMVC的servlet在项目启动时就加载 -->
	<async-supported>true</async-supported><!-- 开启servlet异步线程处理 -->
</servlet>

<!-- 配置dispatcherServlet映射规则，指定需要拦截的请求url -->
<servlet-mapping>
     <servlet-name>springMVC</servlet-name>
     <url-pattern>/</url-pattern>     <!-- /,拦截所有请求 -->
</servlet-mapping>
```

- spring配置类方式

  springMVC提供**WebApplicationInitializer**接口，来通过ServletContext来初始化当前web项目的Servlet容器，配置Servlet容器组件(Servlet、Listener、Filter)（实际上就是替代了web.xml文件，将该文件的配置信息交给springMVC管理，**但Servelt容器在启动时，还是会先读取web.xml，然后交给springMVC配置**

  ），此时可以通过在pom文件中添加如下配置来消除报错：

  ```xml
  <properties>
      <failOnMissingWebXml>false</failOnMissingWebXml>
  </properties>
  ```

  ```java
  public class WebConfig implements WebApplicationInitializer{
  	@Override
  	public void onStartup(ServletContext servletContext) throws ServletException {
  		//初始化spring容器 (ContextLoaderListener只能通过读取配置文件来完成spring容器的创建)
  		servletContext.setInitParameter("contextConfigLocation", "classpath:spring.xml");
  		ContextLoaderListener contextLoaderListener = new ContextLoaderListener();
  		servletContext.addListener(contextLoaderListener);
  
  		// 创建springMVC容器（可以使用xml或者注解）
  		AnnotationConfigWebApplicationContext appContext = new AnnotationConfigWebApplicationContext();
  		appContext.register(MvcConfig.class);
  //		XmlWebApplicationContext appContext = new XmlWebApplicationContext();
  //		appContext.setConfigLocation("/WEB-INF/classes/springMVC-config.xml");
  
  		// 注册DispatcherServlet，并进行初始化配置（初始化springMVC容器，并注册springMVC需要的Bean）
  	DispatcherServlet servlet = new DispatcherServlet(appContext);
  		Dynamic addServlet = servletContext.addServlet("springMVC", servlet);
  	addServlet.setLoadOnStartup(1);
  		addServlet.addMapping("/");
  	addServlet.setAsyncSupported(true);
  	}
  }
  ```

  2、springMVC还提供**AbstractAnnotationConfigDispatcherServletInitializer**类，为**WebApplicationInitializer**接口的实现类，简化springMVC的配置：

  只需要提供spring容器配置类、springMVC容器配置类，就可以完成整个springMVC的配置；并且对于DispatcherServlet属性，默认添加了如下设置：（通过重写其方法来改变）

  addServlet.setLoadOnStartup(1);

  addServlet.setAsyncSupported(true);

  ```java
  public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
  
      //spring容器配置类
  	@Override
  	protected Class<?>[] getRootConfigClasses() {
  		return new Class<?>[] { springTest.class };
  	}
  
      //springMVC容器配置类
  	@Override
  	protected Class<?>[] getServletConfigClasses() {
  		return new Class<?>[] { WebConfig.class };
  	}
  
      //dispatcherServlet映射
  @Override
  	protected String[] getServletMappings() {
  	return new String[] { "/" };
  	}    
  }
  ```

  springMVC配置类：一般情况下，springMVC容器只需要扫描Controller注解的类

  ```java
  @EnableWebMvc
  @ComponentScan(basePackages = { "com.yh.mvc" }, useDefaultFilters = false, includeFilters = {
  		@Filter(type = FilterType.ANNOTATION, classes = { Controller.class }) })
  @Configuration
  public class MvcConfig {
  }	
  ```

- **配置方式选择：**

**XML配置和springMVC配置的两种方式，三个只能同时存在其一，否则会导致Servelt容器启动报错：web.xml中存在多个web应用程序上下文的加载定义，推荐使用AbstractAnnotationConfigDispatcherServletInitializer，更加简单方便**

#### dispatcherServlet属性：

dipatcherServlet初始化参数：（用于在web.xml中配置dipatcherServlet使用）

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| contextClass                   | 用于实例化的参数class对象，默认XmlWebApplicationContext，即springMVC容器的上下文对象 |
| contextConfigLocation          | pringMVC容器的上下文对象的配置文件路径                       |
| namespace                      | XmlWebApplicationContext的命名空间，默认为**[servlet-name]-servlet**，在web.xml中使用 |
| throwExceptionIfNoHandlerFound | 在找不到请求的处理程序时是否抛出NoHandlerFoundException异常；默认为false |

dipatcherServlet常用属性：

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| isAsyncSupported | 默认为false，让dipatcherServlet对请求进行异步多线程处理      |
| loadOnStartup    | 默认-1，让dipatcherServlet懒加载；可以使用设置为1，让其在项目启动时就加载 |

#### dispatcherServlet组件：

dispatcherServlet会依赖一系列springMVC容器中的Bean，来实现其功能：

| Bean                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| HandlerMapping                        | 将请求、拦截器与对应处理程序进行映射，用于进行请求的预处理和后置处理 |
| HandlerAdapter                        | 帮助DispatcherServlet调用请求映射的处理程序                  |
| HandlerExceptionResolver              | 用于解析异常，将其映射到对应处理程序、HTML错误视图或其他目标中 |
| ViewResolver                          | 视图解析器，用于提供视图名称和视图之间的映射                 |
| LocaleResolver、LocaleContextResolver | 提供国际化视图                                               |
| ThemeResolver                         | 提供web程序多主题布局                                        |
| MultipartResolver                     | 处理post请求中的multipart/form-data类型的表单提交，用于处理二进制数据（文件） |
| FlashMapManager                       | 管理FlashMap对象，实现在重定向时，属性从要给请求传递到另一个请求 |

#### multipart/form-data处理：

​		在Servelt中，对于multipart/form-data的表单数据，只能手动解析body获取，springMVC提供MultipartResolver组件来处理，对HttpServletRequest进行封装，生成MultipartHttpSerletRequest，方便@RequestParam、@RequestPart获取请求数据；springMVC默认没有配置MultipartResolver

​		MultipartResolver配置：

1、在Servelt3.0中，已经提供了对应的解析处理和针对于ServletRequest操作其数据的API。但是需要进行一定的配置，才能开启；springMVC提供standardServletMultipartResolver来使用Servelt3.0这个原生功能

```java
public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
	xxxx
	@Override
	protected void customizeRegistration(Dynamic registration) {
        //配置Servelt提供的解析器的本地缓存路径（解析请求中的multipart/form-data数据，需要进行缓存，才能多次操作）
		registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
	}
}
```

**此时，ServletRequest就可以使用getPart，操作Part对象来处理multipart/form-data的表单数据****

```java
	@Bean("standardServletMultipartResolver")
	public MultipartResolver multipartResolverConfig() {
		StandardServletMultipartResolver standardServletMultipartResolver = new StandardServletMultipartResolver();
		return standardServletMultipartResolver;
	}
```

**目前，standardServletMultipartResolver并不需要注册为Bean，dispatcherServelt会自动获取该组件，来处理request请求**

2、使用Commons-FileUpload包实现，springMVC提供CommonsMultipartResolver类来封装使用，需要导入额外的包，在项目、Tomcat不支持Servlet3.0时使用（不进行配置介绍）

### 2、RequestContextHolder

​	springMVC是基于ServletAPI来进行请求处理，因此对于请求、响应处理的关键，还是在于HttpServletRequest、HttpServletRespone对象。springMVC提供RequestContextHolder工具类，在方便在Service层获取当前线程中的Servelt对象，以及它们保存的属性（request、session保存的attributes）：

​	springMVC定义了RequestAttributes，使用K-V保存请求中的重要对象；ServletRequestAttributes继承该接口来快速获取HttpServletRequest、HttpServletResponse：

```java
//通过RequestContextHolder来获取RequestAttributes
//两个方法在没有使用JSF的项目中是没有区别的
RequestAttributes requestAttributes = RequestContextHolder.currentRequestAttributes();
//RequestAttributes requestAttributes =                                 RequestContextHolder.getRequestAttributes();

//RequestAttributes向下转型为ServletRequestAttributes，然后获取其中的request、response对象
HttpServletRequest request = ((ServletRequestAttributes)requestAttributes).getRequest();
HttpServletResponse response = ((ServletRequestAttributes)requestAttributes).getResponse();

```

- **RequestContextHolder怎样使用RequestAttributes封装请求对象及其属性的：**

  当请求被DispatcherServlet拦截后，在FrameworkServlet的doGet/doPost方法中，调用processRequest（request, response）来进行请求对象的预处理：

  **RequestContextHolder使用ThreadLocal来保存RequestAttributes对象，实现多线程间的可见性**

  - **从RequestContextHolder中获取上一个请求的RequestAttributes对象**
  - 通过当前请求的request和response，以及前一个RequestAttributes对象，来创建新的RequestAttributes（封装当前请求对象及其属性）
  - 将新的RequestAttributes设置到RequestContextHolder的ThreadLoacl<RequestAttributes>中

### 3、springMVC过滤器：

Servlet容器提供三大组件：Servelt、Filter、Listener；它们实现了整个Servlet容器对于Web请求的处理

#### 过滤器、监听器配置：

springMVC本身提供dispatcherServelt来拦截处理所有的请求，因此一般情况下，Servlet容器不会进行其他Servelt的配置；而对于Filter、Listener，在springMVC项目下，提供多种配置方式：

- XML配置：

  Servlet容器会默认读取项目下的WEB-INF/web.xml文件，来进行Servelt组件配置；对于**Filter执行顺序，会根据在web.xml中的声明顺序决定**

- 基于Servelt3.0注解配置：

  在Servelt3.0,为了减少xml配置的繁琐，提供了@WebFilter、@WebServlet、@WebListener来配置Servelt组件，对于**Filter执行顺序，会根据@WebFilter所注解的类名自然排序，因此推荐使用Filter01作为前缀来控制**，此时就可以省略web.xml

- springMVC配置：

  springMVC提供**WebApplicationInitializer**、**AbstractAnnotationConfigDispatcherServletInitializer**接口，来获取servelt容器的上下文对象ServeltContext，配置其Servelt组件，对于Filter执行顺序，会根据配置顺序决定

  ```java
  public class WbeConfigSimple extends AbstractAnnotationConfigDispatcherServletInitializer {
  	xxxx	
  	@Override
  	protected Filter[] getServletFilters() {
          //按照数组排序顺序执行
  		return new Filter[] {new MyFilter()};
  	}
  }
  ```

由于Servelt容器，会优先读取WEB-INF/web.xml文件，然后再将配置好的ServeltContext交给springMVC处理，因此其Filter优先级为：

XML配置    >  注解配置  >   springMVC配置，**推荐单独使用一种进行配置，减少不必要的处理**

springMVC配置的过滤器、Servelt会自动开启非懒加载和异步处理模式；而ServeltAPI 方式需要手动配置相应参数

#### springMVC常用过滤器：

​	springMVC提供一套基于Filter接口的过滤器类架构，springMVC在Filter基础上，提供了几个重要的类：

- GenericFilterBean：Filter的子类，提供了Filter访问web.xml文件中Filter标签下的子标签init-param值，是实现Filter的初始化
- OncePerRequestFilter：GenericFilterBean的子类，实现了对于request只进行一次过滤（解决转发时，会导致request再次被过滤器处理的问题）
- AbstractRequestLoggingFilter：OncePerRequestFilter的子类，提供额外beforeRequest、afterRequest方法，提供对过滤前后的编写相应代码（将整个过滤器代码分步化，增强过滤器全局处理请求的能力）

对于这几个类，springMVC提供了几个使用的实现类：

- HiddenHttpMethodFilter：可以将form表单的请求转化为DELETE、PUT（form表单只支持put、post）
- ShallowEtagHeaderFilter：实现对HTTP缓存中Etag请求头的处理
- CorsFilter：实现CORS处理
- CharacterEncodingFilter：实现对请求数据参数的编码处理（**非常重要**），注意get传参数据，会被servlet容器处理，并不会被CharacterEncodingFilter处理；但springMVC在进行参数封装时，会自动根据Servlet容器编码来进行转化（tomcat8.0 默认为utf-8）

### 4、springMVC拦截器：

​	拦截器是web框架的必备功能，增强开发者对请求全局的控制处理，相对于Servelt api提出的过滤器理念，功能更加强大

#### 拦截器使用：

​	springMVC提供HandlerInterceptor接口，对所有被HandlerMapping处理的请求进行拦截处理：

```java
public interface HandlerInterceptor {
    default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)throws Exception {
     	return true;
	} 
    
    default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable ModelAndView modelAndView) throws Exception {
        
	}
    
    default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,@Nullable Exception ex) throws Exception {
        
	}
```

HandlerInterceptor提供三个方法：

- preHandle：在控制器方法（Handler）执行前执行，返回值为一个布尔类型，true则将请求交给下一拦截器或Handler处理，false则认为请求已被处理，不在进行任何后面处理。可以用于身份认证和授权
- postHandle：在控制器方法返回ModelAndView之前执行，这里并不是控制器方法的实际返回值，springMVC提供对控制器方法各种返回值的支持，但无论是否为Model、View等返回值类型，最后反射调用后都会通过Hander返回值处理器处理，返回一个ModelAndView对象，此时就会开始执行postHandle，可用于对MV对象进行额外处理(**当Handler执行出现异常时，则不会执行postHandle方法**)
- afterCompletion：执行完控制器方法并返回ModelAndView后执行，一般用于对控制层的统一异常处理、日志处理（**无论Handler执行是否发生异常，都会执行该方法**）

#### **拦截器配置：**

有两种配置方式：

1、使用springMVC配置类，进行手动配置

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    	//拦截器配置
	@Override
	public void addInterceptors(InterceptorRegistry registry) {
		InterceptorRegistration addInterceptor = registry.addInterceptor(new MyInterceptor());
		addInterceptor.addPathPatterns("/**");//拦截器映射匹配路径
		addInterceptor.excludePathPatterns("/testSync");//拦截器排除路径
	}
}
```

2、注入到IOC容器中，自动完成拦截器注册

#### 拦截器和过滤器的区别：

1、拦截器基于JAVA反射机制，过滤器是基于函数回调

2、过滤器依赖于Servlet容器，拦截器不依赖

3、拦截器只会对映射了控制器方法的请求进行处理；过滤器会对所有请求进行处理

4、拦截器可以访问springMVC处理请求所提供的各对象（handle、ModelAndView、Exception）；过滤器只能获取Servelt请求的相应对象

5、在一个请求中，只会调用一次过滤器；但对于拦截器，会在控制器中进行映射方法处理器的跳转，从而调用多次拦截器（请求每映射一次处理器，都会调用拦截器）

### 5、springMVC控制器：

springMVC提供基于注解的编程模型，来实现控制器功能，处理被前端控制器拦截映射的请求，其方法叫做请求映射处理器（**Handler**）

#### @Controller、@RestController

用于注册控制器bean：

- @Controller

  ​	使用上和spring的@Component一致，通过@ComponentScan来将被@Controller注解的类注册到springMVC容器中

- @RestController

  ​	是一个组合注解，@Controller+@ResponseBody，让当前类中所有的方法都继承@ResponseBody，表示所有方法的返回值都直接写入responseBody中，不进行String类型的视图解析，

  ​	**对于javaBean，会通过HttpMessageConverter进行数据转化，默认使用JSON序列化的形式，将JAVAbean转化为JSON字符串，并且修改response.ContentType="application/json;charset=UTF-8"**；

#### HttpMessageConverter：

HttpMessageConverter，http消息转化器，有两个使用场景：

1、获取requestBody中的数据，将其转化为javaBean；

2、将带有@ResponseBody注解的Handler的返回值，进行类型转化，并写入到responseBody中，并指定其Content-Type响应头

**springMVC对Content-Type响应头处理：**

对于Content-Type响应头时，会受到请求头中Accept属性的影响，也叫做MediaType。而requestMapping中的produces就是用于设置请求中的Accept属性，其原理为：

​		在springMVC中，response.ContentType会被HttpMessageConverter单独处理，首先根据requestMapping中的produces确定MediaType的范围，然后根据MediaType和javaBean类型，选择合适的HttpMessageConverter，此时来最终确定response.ContentType，因此在springMVC中response.ContentType无法被手动修改

springMVC提供多个默认HttpMessageConverter，常用有两个：

- MappingJackson2HttpMessageConverter：支持javaType=Object、MediaType=“application/json；charset=UTF-8”；但需要导入Jackson的jar包（jackson-databind）
- StringHttpMessageConverter：支持javaType=String、MediaType=="text/plain;charset=ISO-8859-1"；

**注意：**

​	由于StringHttpMessageConverter的先声明，因此对于String类型的javaBean，优先被StringHttpMessageConverter处理，这样就会导致String类型无法被json序列化，并导致中文乱码，对于这种问题有两种解决方法：

1、我们可以通过@RequestMapping的Produces属性，来控制跳过StringHttpMessageConverter，选择MappingJackson2HttpMessageConverter进行处理；

2、设置StringHttpMessageConverter的supportedMediaTypes属性为“application/json；charset=UTF-8”

#### Converter:

​	springMVC对于HTTP参数，会分为两种，

1、Body传参：基于json格式使用HttpMessageConverter进行反序列化，

2、url、form表单传参，通过springMVC内置的一系列处理器，根据接受参数类型，进行对应的转换

因此，可以自定义Converter进行一些复杂参数类型的转换，方便参数接受：

**在自定义converter解析失败抛出异常后，会先被springMVC默认解析器处理，如果无法处理，则再抛出异常**

- **Converter<S,T>**

  使用Converter<S,T>接口,来定义converter，一般S使用String类型，表示源数据类型，T为接受转换的数据类型

  以date类型为例

  ```java
  @Configuration
  public class DateTimeConverter {
  
  	@Bean
  	public Converter<String, Date> DateConverter() {
  		return new Converter<String, Date>() {
  			@Override
  			public Date convert(String source) {
  				//使用Hutool-DateUtil工具类，支持多种时间类型的格式化
  				return DateUtil.parse(source);
  			}
  		};
  	}
  }
  ```

- **ConverterFactory<S,R>**

  直接Converter接口存在一个问题，无法动态获取T类型，需要手动写死，即每个Converter只能处理一种数据类型，而使用ConverterFactory<S,R>,R指定一个接受参数类型的范围,从而对实现了某种特殊接口的类型参数，进行处理单独

  以枚举为例

  ```java
  //converterFactory工厂
  public class CommonEnumConverterFactory implements ConverterFactory<String, IEnum<String>> {
  
      //提供创建converter对象方法，并以目标参数类型作为入参
  	@Override
  	public <T extends IEnum<String>> Converter<String, T> getConverter(Class<T> targetType) {
  		return new CommonEnumConverter<>(targetType);
  	}
  
      //对应的converter类，通过构造方法获取的目标参数类型，进行额外处理
  	public class CommonEnumConverter<T extends IEnum<String>> implements Converter<String, T> {
  
  		private final T[] values;
  
  		public CommonEnumConverter(Class<T> targetType) {
  			//获取当前枚举类型的所有枚举值
  			values = targetType.getEnumConstants();
  		}
  
  		@Override
  		public T convert(String source) {
  			if (StrUtil.isBlank(source.trim())) {
  				return null;
  			}
  			for (T t : values) {
  				if (Objects.equals(source, t.getValue())) {
  					return t;
  				}
  			}
  			throw new BusinessException("枚举类型转换失败");
  		}
  	}
  }
  ```

  **注意：**

  ​	converterFactory区别于Converter，它并不会被spring扫描后直接生效，必须在WebMvcConfigurer配置接口继承类中，手动注册：

  ```java
  	//注册格式化器、转换器工厂
  	public void addFormatters(FormatterRegistry registry) {
  		registry.addConverterFactory(new CommonEnumConverterFactory());
  	}
  ```


#### 映射注解

springMVC提供**@RequestMapping**注解，将请求映射到指定控制器方法中；并提供各种属性，通过URL、http方法、请求参数、Headers来进行匹配

@RequestMapping属性：

| 属性名   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| name     | 该控制器的映射Name                                           |
| value    | path的别名                                                   |
| path     | 所映射请求的url，支持相对路径匹配、REST风格传参              |
| method   | HTTP方法，RequestMethod[]，可以匹配多个HTTP方法              |
| params   | 请求参数列表，不是固定匹配，包括所有就满足，**即请求参数中必须包含当前规定的所有参数** |
| produces | 指定处理后响应头中，Content-Type类型值，**通过该属性可以定义响应数据的编码格式** |
| headers  | 请求中必须包含指定请求头                                     |

springMVC还提供@RequestMapping多个变体注解，用于方便定义Http方法类型：

**@GetMapping、@PostMapping**

注意：@RequestMapping可以注解在类上，定义所有方法的映射规则，此时还可以使用@RequestMapping注解在方法上，来进一步缩小映射匹配范围：

- 对于Path，会将类和方法上的值进行组合：

```java
@RestController
@RequestMapping(path="/data")
public class DataController {
	
    @RequestMapping(path="/test")
	public String Test() {
		return "succsee";
	}
```

**此时实际请求映射的Path值为 /data/test**

- 对于method、params、produces、headers，方法级别会覆盖类级别

##### 方法参数注解

​	springMVC提供一系列方法参数注解，将springMVC封装好的Http请求数据，注入到控制器方法形参中；在@Controller注解的类被注册到springMVC容器后，会进行自动装配；当其映射方法被执行时，会通过方法参数中的注解，将请求中的参数对应传入，实现对请求数据的自动获取

| 注解              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| @PathVariable     | 获取URL中使用REST风格的传参值                                |
| @MatrixVariable   | 获取URL中矩阵变量传参值（一般不使用）                        |
| @RequestParam     | 获取Servlet中的请求参数（url传参、form-data传参），springmvc对于文件上传，提供MultipartFile对象进行接收（File无法接收） |
| @RequestHeader    | 获取请求头参数值                                             |
| @CookieValue      | 获取Cookie中的参数值                                         |
| @RequestBody      | 获取请求Body中的内容，并通过HttpMessageConverter根据请求头中的Content-Type对内容进行解析，转化为java对象（一般用于json反序列化为java对象） |
| @RequestPart      | 将multipart/form-data中的json数据直接反序列化为java对象（避免后台手动反序列化，但是需要前端定义当前数据的content-type=application/json） |
| @ModelAttribute   | 将所有URL获取的参数和form-data获取的参数，绑定到java对象中；必须保证java对象提供要给无参构造方法，无需保证至少有一个属性注入 |
| @SessionAttribute | 获取HttpSession对象中的属性，如sessionID                     |
| @RequestAttribute | 获取HttpServeltRequest对象中的属性（用于请求转发，但在前后端分离的项目中不会使用） |

**注意：**

- 除@ModelAttribute外，所有方法参数注解都有一个**required**属性，默认为true，即不能够忽略：

  ​	当请求参数中不包含当前指定的参数时，springMVC就会抛出异常**响应400**，提示当前类型的参数没有被绑定，即请求中没有包含当前数据；当required属性为false时，springMVC就会忽略没有绑定的参数，默认为null

- 以@RequestParam为例，方法参数注解的使用：

  ​	@RequestParam提供name、value两个属性值，来对应获取http请求数据中的参数名，从而获取其参数值；**当不进行两个属性值声明时，默认使用方法形参名匹配**

  ​	@RequestParam单独还提供defaultValue属性，定义当参数没有被绑定时的默认值（不设置，则默认为null）；并且defaultValue只能提供String类型值，需要保证所对应形参类型可以进行有效转化

- 当映射方法参数没有使用注解时，首先会与springMVC默认定义的一些早期值进行匹配，当没有匹配值时，则通过BeanUtils.isSimpleProperty方法判断当前参数类型是否为简单数据类型；是，则自动添加一个属性required=false的@RequestParam注解；不是，则自动添加@ModelAttribute注解

- @RequestParam和@RequestParth都可以获取multipart/form-data数据，但是两者有区别：

  @RequestParam只能将text解析为String类型

  @RequestParth还可以解析如json字符串，将其转化为javaBean；但需要配合数据Content-Type值来解析（并不是请求头中的Content-Type），因此一般不使用，这样只是让后台代码更加优雅，但是增加了不必要的前后端传参定义规范

#### 方法参数早期匹配值

​		spring提供一系列对象，方便操作servelt API；这些值的注入不需要特殊注解，springMVC会在早期，根据参数类型进行匹配注入：

| 对象类型                                                    | 作用                                                         |
| ----------------------------------------------------------- | ------------------------------------------------------------ |
| WebRequest、NativeWebRequest                                | springMVC封装好的对象，用于简化servletAPI操作，如对请求参数、请求属性和会话属性的访问 |
| javax.servlet.ServletRequest、javax.servlet.ServletResponse | 获取Servlet API提供的请求和响应对象                          |
| javax.servlet.http.HttpSession                              | 获取Serlvet API提供的请求会话对象；该对象的线程不安全，因此要避免业务层缓存数据放在Session属性中 |
| HttpMethod                                                  | HttpMethod的枚举值                                           |
| java.io.InputStream、java.io.Reader                         | 获取Servlet API的 请求输入流                                 |
| java.io.OutputStream、java.io.Writer                        | 获取Servlet API的 响应输出流                                 |
| java.util.Locale                                            | 当前请求的语言环境（Http请求并不会带有当前客户端的语言环境，需要通过LocaleContextResolver来解析请求中的特定数据，来设置） |
| java.util.TimeZone、java.time.ZoneId                        | 当前请求的时区（同上）                                       |
| HttpEntity<B>                                               | 获取请求中的body和headers，使用                              |
| UriComponentsBuilder                                        | 用于构造和编码URI                                            |
| Errors、BindingResult                                       | 用于获取springMVC参数绑定的错误信息，需要保证它们在所有绑定数据的最后 |
| javax.servlet.http.PushBuilder                              | 获取Servlet 4.0 提供用于访问HTTP/2资源推送的对象             |
| java.security.Principal                                     | 当使用spring-security时，获取用于验证用户身份的Principal实现类 |

另外有关视图解析、重定向的对象，无需了解

#### 方法返回值和相关注解

映射方法返回值用于定义springMVC处理请求后，响应数据的传递方式和响应信息的设置：

| 返回值类型                                                   | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| HttpEntity<B>、ResponseEntity<B>                             | 用于定义完整的响应信息（body与Headers），返回值会通过HttpMessageConverter写入Response对象中 |
| HttpHeaders                                                  | 定义Headers信息，即无正文数据                                |
| String、View、java.util.Map、org.springframework.ui.Model、ModelAndView | 定义视图、视图模型数据（前后端分离不会使用）                 |
| void                                                         | 默认响应已经被处理，若没有则会抛出异常响应500                |

除了常规响应信息外，还提供对异步请求、HTTP流的处理，在此不进行了解

​	springMVC提供@ResponseBody注解，使用**HttpMessageConverter**将返回值转化为ResponseBody，默认使用jackson JSON来实现对象序列化，简单数据类型则转化为String

##### @ExecptionHandler

​		当请求映射处理期间发生异常时（控制层抛出异常），DispatcherServlet会将异常交给一系列HandlerExceptionResolver类型Bean，以责任链模式进行异常处理，即**请求错误响应**

springMVC提供如下HandlerExceptionResolver实现类：

| HandlerExceptionResolver实现类    | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 异常类名称和错误视图名称之间的映射，定义响应错误页面（一般用于前后端不分离项目，使用Servlet进行页面跳转） |
| DefaultHandlerExceptionResolver   | springMVC默认提供的异常解析器，解决所有springMVC引发的异常，并映射响应对应的HTTP状态代码；它的解析结果可以被ResponseEntityExceptionHandler、REST API异常所覆盖 |
| ResponseStatusExceptionResolver   | 提供@ResponseStatus注解，来映射指定HTTP状态代码              |
| ExceptionHandlerExceptionResolver | 提供@ExceptionHandler注解，来定义指定异常的处理方法          |

我们可以自定义多个HandlerExceptionResolver类型Bean，根据order来设置它们的优先级，即在责任链中的优先级；当所有HandlerExceptionResolver都没有解决该异常时，则交给servlet容器处理

**springMVC推荐使用@ExceptionHandler来定义异常处理：**

```java
@RestController
public class TestController {
    
	@ExceptionHandler(Exception.class)
	public String exceptionHandler(Exception exception) {
		return exception.toString();
	}
    
    @RequestMapping("/test")
	public String  test() {
		int a =1/0;
        return "hh";
    }
}
```

**@ExceptionHandler注解的异常处理器只能作用当前Controller类中**

##### @ModelAttribute

​	**dispatcherServlet在处理每个映射请求时，都提供了一个Model对象，用于作为数据中间层，存储和访问数据**；**在前后端不分离的情况下，Model对象可以被视图层访问**

​	@ModelAttribute注解就是用于开发者操作Model对象数据，它可以注解在三种位置：

- 映射方法上，将当前方法的返回值作为Model中数据（前后端分离不会使用）
- 映射方法参数上，首先根据Model中的已有数据进行Name匹配注入，不存在匹配值则会将参数放入Model中，使用**WebDataBinder**进行参数绑定；（**可以直接通过org.springframework.ui.Model进行早期值参数注入，获取Model对象**）
- 控制器的非映射方法上，用于执行@RequestMapping方法前，初始化Model对象

```java
	@ModelAttribute("userA")
	public User initModel(String id,String name) {
		User user = new User();
		user.setId(id);
		user.setName(name);
		return user;
	}

	@RequestMapping("/test")
	public String  test(Model model) {//通过Model对象获取其中数据
		User user = (User)model.getAttribute("userA");
		System.out.println(user.toString());
        return "hh";
    }

	@RequestMapping("/testA")
	public String  test(@ModelAttribute("userA") User user) {//直接匹配Model对象中的userA参数镀锡，进行注入
		xxx
    }
	
	@RequestMapping("/testA")
	public String  test(@ModelAttribute User user) {//此时会通过WebDataBinder，获取servelt参数值，进行参数绑定
		xxx
    }
```

##### @InitBinder

​	@InitBinder用于初始化当前控制器的WebDataBinder，该对象有如下作用：

- 将请求参数绑定到Model对象中
- 将所有的String类型请求数据（请求参数、url、Headers、Cookie等）转化为控制器方法参数
- 将Model对象中的数据转化为String值

```java
    @InitBinder 
    public void initBinder(WebDataBinder binder) {
        SimpleDateFormat dateFormat = new SimpleDateFormat("yyyy-MM-dd");
        dateFormat.setLenient(false);
        binder.registerCustomEditor(Date.class, new CustomDateEditor(dateFormat, false));
    }
```

通过 @InitBinder 可以定义WebDataBinder 一些基本属性，如date转化器

##### @ ControllerAdvice

​	@ControllerAdvice为增强版控制器，用于全局处理请求数据，搭配@ExceptionHandler、@ModelAttribut、@InitBinder使用；

**@ControllerAdvice并不是@Controller的子注解，并不能搭配@requestMapping使用，实现控制器功能；而是@Component子注解，会被注册到spring容器中**

- 全局异常处理：

  @ControllerAdvice和@ExecptionHandler搭配使用，全局处理控制层抛出的所有异常；当发生发生指定异常时，会被相应Handler方法处理，从而指定当前请求的响应内容

  ```java
  @ControllerAdvice
  public class CustomExceptionHandler {
  
      //@ExceptionHandler注解的方法，会自动注入Exception类型参数，即全局控制层抛出的异常对象
  	@ResponseBody
  	@ExceptionHandler(Exception.class)
  	public String exceptionHandler(Exception exception) {
  		return exception.toString();
  	}
  }
  ```

- 全局请求参数绑定

  @ControllerAdvice和@ModelAttribut搭配使用，全局处理所有请求的参数绑定；一般情况下，优先作用局部方法，不使用默认全局值，因为参数绑定的通用性不强，以免干扰过多不相干的映射处理器

- 全局初始化WebDataBinder 

  @ControllerAdvice和@InitBinder 搭配使用，全局初始化WebDataBinder对象，和@ModelAttribut一样，优先作用局部方法

@ControllerAdvice默认作用所有@Controller，但是可以通过指定basePackages，来所需作用范围

### 6、springMVC其他功能：

##### URI处理：

​	springMVC提供UriComponentsBuilder工具类，来构建和编码URI

##### 异步请求处理：

​	在Servlet3.0后，使服务器支持request异步请求处理，使用线程池技术，来多线程异步处理请求，而不是单线程阻塞的形式，导致服务器并发量降低

- Servelt异步请求处理方式：

  1、request.startAsync() 将当前Servelt请求设置为异步处理模式，从而将当前Servelt线程释放，但响应处于处理状态

  2、并且request.startAsync()方法会返回一个AsyncContext对象，用于缓存当前请求对于的reques、response对象，并存放在application作用域中；当请求完成响应后，则会重新将其reques、response对象交给Servelt容器

  3、Servelt会存在一个异步线程池，用于执行所有被延迟响应的异步请求；此时servelt容器就会异步线程池中的servelt线程处理该请求，完成请求响应

​	springMVC提供两种方式来封装实现Servelt异步请求处理：

- DeferredResult：

  springMVC提供DeferredResult对象，来搭配控制器实现Servelt异步请求处理；我们需要手动创建新线程，来执行异步代码，将结果放入DeferredResult对象，进行响应

  ```java
  @RequestMapping("/testSync")
  	public DeferredResult<String> syncTask(){
          xxxxx
  		//定义异步结果集，定义异步处理超时时间
  		DeferredResult<String> deferredResult = new DeferredResult<String>((long) 11000);
  		Runnable runnable = new Runnable() {
  			@Override
  			public void run() {
  				try {
  					Thread.sleep(10000);
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  				deferredResult.setResult("hhhhSync");
  			}
  		};
  		Thread thread = new Thread(runnable);
  		thread.start();
  		return deferredResult;
  	}
  ```

  DeferredResult的工作过程：

  - 控制器返回DeferredResult对象
  - springMVC调用request.startAsync() ，此时当前servelt线程释放（处理当前请求的线程）
  - DeferredResult被其他线程处理，并执行setResult方法
  - 此时springMVC将请求重新交给Servelt容器管理，从而获取一个servelt线程，处理当前请求

- Callable：

  springMVC还支持JDK提供的java.util.concurrent.Callable作为处理器返回值，springMVC会使用一个线程池来处理控制器所返回的Callable任务；springMVC默认提供SimpleAsyncTaskExecutor对象作为线程池，但是它实际并不是一个线程池，每次执行任务都需要创建一个新的线程；

  因此我们需要进行手动配置springMVC的异步线程池，并且可以设置异步请求默认超时时间

  springMVC提供多个AsyncTaskExecutor实现类，推荐使用ThreadPoolTaskExecutor，它是对java.util.concurrent.ThreadPoolExecutor的封装

  ```java
  @Configuration
  public class MvcConfig implements WebMvcConfigurer{	
  	@Override
  	public void configureAsyncSupport(AsyncSupportConfigurer configurer) {
  		ThreadPoolTaskExecutor threadPoolTaskExecutor = new ThreadPoolTaskExecutor();
          threadPoolTaskExecutor.initialize();
  		configurer.setTaskExecutor(threadPoolTaskExecutor);
  		configurer.setDefaultTimeout(10000);
  	}
  }
  ```

  ```java
  @RequestMapping("/testSyncA")
  	public Callable<String> syncTaskA(){
  		xxxx
          return new Callable<String>() {
  			@Override
  			public String call() throws Exception {
  				try {
  					Thread.sleep(10000);
  				} catch (InterruptedException e) {
  					e.printStackTrace();
  				}
  				return "hhhhSyncA";
  			}
  		};
  	}
  ```

  Callable的工作过程：

  - 控制器返回Callable
  - springMVC调用request .startAsync，此时当前servelt线程释放（处理当前请求的线程），并将Callable对象交给AsyncTaskExecutor线程池执行
  - 当Callable执行完成，返回结果后，springMVC再次将请求交给Servlet容器处理，从而获取一个servelt线程，处理当前请求

- 在进行异步请求处理过程中，HTTP客户端会有一个Read超时时间（即HTTP响应时间）；而服务端也可以设置自己处理请求的超时时间，但一般情况下，该超时时间要小于客户端Read超时时间

- **注意，异步请求需要保证处理请求的Servert和Filter都设置为可异步处理：asyncSupported=true**

##### CORS：

​	springMVC的handlerMapping实现了对CORS内置支持，当请求被映射到相应处理器后，HandlerMapping会对请求进行CORS拦截处理，并且开发者可以为url-pattren或每个映射的url设置单独的CORS配置：

**全局配置：**

在**WebMvcConfigurer** 接口中，提供**addCorsMappings**回调方法，来设置指定映射url模板的CORS配置

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {	
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/*")//拦截url-pattern 
			.allowedOrigins("*")	//允许的跨域请求，* 为允许所有(浏览器对于跨域请求，会自动添加origin请求头，value为当前项目的ip+端口)
			.allowedHeaders("*") //允许的请求头
			.allowedMethods("*")  //允许的HTTP请求方法
			.allowCredentials(false) //允许请求发送cookie等安全证书（当allowedOrigins为*时，就不能设置allCrendentials为true）
			.maxAge(3600);//预检请求有效期 单位s
	}
```

**局部配置：**

spring4.2后，提供@CrossOrigin注解，进行局部控制器的CROS处理

@CrossOrigin可以注解在类或方法上，默认使控制器支持跨域请求处理：

@CrossOrigin属性

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| origins          | 指定允许的跨域请求地址，默认为空，允许所有                   |
| allowedHeaders   | 指定允许的请求头，默认空，但CROS规范默认允许一些请求头如：Content-type |
| exposedHeaders   | 指定要暴露的请求头，默认为空                                 |
| methods          | 指定允许的http请求方法，默认为空，为当前控制器方法允许值     |
| allowCredentials | 允许请求发送cookie等安全证书，默认不启用                     |
| maxAge           | 预检请求有效期，默认-1 永不过期                              |

**局部配置会覆盖全局配置**

**当然springMVC也支持原生CROS解决方案，使用CROS过滤器,并且springMVC提供更加简便的配置方法**

##### 网络安全：

**springMVC提供整合spring Security框架，来实现web项目的权限和网络安全**

##### HTTP缓存：

​	HTTP缓存属于客户端缓存，即浏览器缓存，它是基于HTTP协议存在的，并分为两种：强制缓存和协商缓存

HTTP缓存作用过程：

1. 浏览器发出请求，首先判断是否命中强制缓存，命中则从浏览器缓存中读取资源
2. 如果没有命中则将资源请求发送给服务器，此时服务器判断客户端的本地缓存是否失效
3. 没有失效，则服务器不返回任何信息，浏览器转而从缓存中加载资源
4. 如果失效，服务器就会返回完成资源信息，交给浏览器加载

HTTP缓存设置：

通过修改HTML meta标签，来关闭浏览器缓存；

```java
<META HTTP-EQUIV="Pragma" CONTENT="no-store">
```

- 强制缓存：

  当强制缓存命中后，请求会直接返回200，Size会显式from cache；强制缓存根据HTTP协议版本，分两个响应请求头控制：Expires、Cache-Control；它们代表了当前服务器返回资源的缓存时间

  Expires为HTTP1.0定义，Cache-Control为1HTTP1.1定义；它们向上兼容，因此Cache-Control的优先级更高

- 协商缓存

  协商缓存相对于强制缓存，是将浏览器缓存的决定权始终交给服务器完成，当第一次请求被服务器处理时，会在响应头中添加一个缓存标识，再第二请求时，服务器通过判断缓存标识，来决定是否读取缓存数据

  在HTTP1.0中,通过Last-Modified响应头来记录资源上一次访问时间，第二次请求时，就会使用If-Modified-Sinc请求头带上该时间，此时服务器就可以通过该请求头时间，进行逻辑代码处理，判断是否需要使用浏览器缓存；当使用，则直接返回304状态码；不使用，则返回新资源，并使用Last-Modified响应头记录资源访问新时间

  HTTP1.1中提供Etag来作用缓存标识，If-None-mactch作为请求是否使用缓存标识

springMVC提供CacheControl对象，来实现服务器端对Cache-Control响应头的处理，并在控制器返回值中，进行Cache-Control、Etag处理

##### FreeMarker：

​	springMVC内部集成了Apache FreeMarker模板引擎，来配置web项目视图及其数据访问

### 7、springMVC配置：

​		springMVC提供**WebMvcConfigurer**接口，让开发者对springMVC中的内部组件进行配置，常用包括：

```java
public interface WebMvcConfigurer {
    //url路径匹配解析
    default void configurePathMatch(PathMatchConfigurer configurer) {
	}
    
    //内容类型解析
    default void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
	}
    
    //异步请求处理配置
    default void configureAsyncSupport(AsyncSupportConfigurer configurer) {
	}
    
    //默认servetl处理，优先级最低，当没有可匹配的Servet时生效
    default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
	}
    
    //数字、日期格式转化器，通过@NumberFormat、@DateTimeFormat使用，在javaBean接收请求参数时，进行相应类型转化
    //注册格式化器、转换器工厂
    default void addFormatters(FormatterRegistry registry) {
	}
    
    //配置拦截器
    default void addInterceptors(InterceptorRegistry registry) {
	}
    //静态资源映射
    default void addResourceHandlers(ResourceHandlerRegistry registry) {
	}
    //CORS配置
    default void addCorsMappings(CorsRegistry registry) {
	}
    //视图控制器，定义指定处理器返回值对应视图
    default void addViewControllers(ViewControllerRegistry registry) {
	}
    //视图解析器，实现对视图解析和数据渲染
    default void configureViewResolvers(ViewResolverRegistry registry) {
	}
    //Handler参数解析器
    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
	}
    //Handler返回值处理器
    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
	}
    
    //消息转化器，用于处理request、response的Body数据
    default void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
	}
    //
    default void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
	}
    
    //异常处理器
    default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
	}
    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
	}
    
    //用于控制器参数校验，通过@Valid、Validate字段定义使用
    @Nullable
	default Validator getValidator() {
		return null;
	}
    
    //响应码解析器，生成对应错误的响应码
    @Nullable
	default MessageCodesResolver getMessageCodesResolver() {
		return null;
	}
}
```

当然springMVC提供了一个**WebMvcConfigurer**接口的具体实现类：**DelegatingWebMvcConfiguration**，为springMVC提供默认配置，在在其基础上进行扩展、删除

### 8、springMVC请求处理流程：

![](..\Typora图片\springMVC请求处理流程.png)

过程如下：

- Servlet容器启动，初始化Listener、Filter、Servlet，完成整个Servlet容器上下文的配置

  在初始化Servelt时，就会实例化DispatcherServelt，从而创建springIOC容器、初始化springMVC组件：

  包括对控制器中映射处理器方法的获取，保存到HandlerMapping中

- 当用户发送请求后：

  1. 首先被Servlet容器获取，进行Filter处理
  2. 在交给url映射的Servlet中（springMVC的DispatcherServelt一般拦截所有请求）

- DispatcherServelt将拦截到的请求进行处理：

  1. 通过HandlerMapping，找到当前请求的Handler
  2. 根据Handler类型，找到对应的HandlerAdapter
  3. 调用拦截器preHandler方法，对请求进行前置处理
  4. 使用HandlerAdapter对根据Handler对象，对控制器方法进行反射调用

- Handler的执行过程：

  1. 通过Handler参数解析器，完成控制器方法的参数注入（此时回对请求的request对象进行处理）
  2. 反射调用控制器方法
  3. 通过Handler返回值处理器，对控制器方法返回值进行处理（此时会对请求的response对象进行处理）
  4. 最后返回ModelAndView对象

- Handler执行后，DispatcherServlet进行整个处理结果进行进一步处理：（期间当出现异常，则调用springMVC异常处理器，来响应请求）

  1. 调用拦截器的postHandler方法，对请求ModelAndView对象进行处理
  2. 通过视图解析器，完成对MV对象的解析，找到相应视图
  3. 然后将MV对象中的Model数据，渲染到视图中
  4. 在响应请求前，调用拦截器的afterCompletion
  5. 最后响应请求

**对应前后端分离项目，一般不会进行MV对象的视图解析，即直接使用@ResponseBody，将控制器方法的返回值以json形式写入响应Body中，而Handler会返回一个null的MV对象**

### 9、DispatcherServelt源码解析：

DispatcherServelt的继承了一个HttpServelt，当请求被其拦截后，执行其doGet、doPost方法。从而调用DispatcherServelt核心入口方法**doDispatch**:

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
                //检查请求是否存在二进制文件表单提交，并进行缓存处理
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

                //在DispatcherServelt初始化时，会自动完成以下操作：
				//1、获取IOC容器在所有的@Controller注解类
                //2、遍历其所有方法，获取带有@RequestMapping注解的方法
                //3、创建一个Map集合，以@RequestMapping的value作为Key，其方法对应mothod对象为Value（通过HandlerMapping实现）
                	
                //根据请求rul，通过HandlerMapping从Map集合中获取对应的Handler
                //HandlerMapping有多个，对应不同的控制器定义方式：@Cotroller、Cotroller接口、HttpRequestHandler，每个HandlerMapping都有一个Map存储handler和url映射关系
				mappedHandler = getHandler(processedRequest);
                //若没有找到对应Handler，则直接响应404
				if (mappedHandler == null || mappedHandler.getHandler() == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				//将获取该Handler对应的HanlerAdapter，用于调用Handler处理请求
                //同样HanlerAdapter根据控制器的定义方式，也有多个不同的实现，Handler可以是一个类，也可以是一个方法（@Controller定义的）；根据Handler的不同类型，来使用不同的调用方式
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				//如果为GET请求，则默认支持对last-modified请求头的处理，实现HTTP协商缓存
				String method = request.getMethod();
				boolean isGet = "GET".equals(method);
				if (isGet || "HEAD".equals(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (logger.isDebugEnabled()) {
						logger.debug("Last-Modified value for [" + getRequestUri(request) + "] is: " + lastModified);
					}
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}

                //执行拦截器的preHandler方法，并是否直接拦截请求
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				//通过HanlerAdapter实际调用Hanler
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

                //判断请求是否需要进行异步处理，则先返回释放Servelt线程
				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}

                //对Hanler执行结果进行默认视图解析
				applyDefaultViewName(processedRequest, mv);
                //执行拦截器的postHandler方法，对Hanler结果进行进一步处理
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
                //dispatchException会被全局异常处理器处理（@ExceptionHandler）
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
            //完成最后的Disparch处理：异常处理、视图处理、拦截器afterCompletion方法执行
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				//如果进行异步请求处理，则执行异步拦截器的afterConcurrentHandlingStarted方法
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// 清空该请求的multipart缓存
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}
	}
```

## REST客户端

springMVC提供两种REST客户端API调用：

#### RestTemplate：

**以同步请求方式执行REST客户端服务通信，默认是对JDK中java.net包的封装，提供更加方便、高效的API**

##### RestTmplate实例化：

​	RestTmplate默认构造方法，底层使用java.net.HttpURLConnection进行HTTP请求，开发者可以通过ClientHttpRequestFactory接口参数，来通过构造方法来进行配置，springMVC提供如下HTTP客户端库的支持：

- HttpComponentsClientHttpRequestFactory ：用于配置Apache HttpComponents包
- Netty4ClientHttpRequestFactory：用于配置Netty4 包
- OkHttp3ClientHttpRequestFactory：用于配置OkHttp包
- SimpleClientHttpRequestFactory：默认使用，用于配置java.net包

##### RestTemplate常用方法：

| 方法          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| getForObject  | GET请求，获取响应数据，并使用消息转化器，转化为指定JavaBean  |
| getForEntity  | GET请求，获取响应封装对象ResponseEntity（包括响应码、Headers、Body） |
| postForObject | POST请求，获取响应数据，并使用消息转化器，转化为指定JavaBean |
| postForEntity | POST请求，获取响应封装对象ResponseEntity（包括响应码、Headers、Body） |
| exchange      | 请求通用方法，必须使用RequestEntity参数（包括HTTP方法，URL，标头和正文作为输入），获取获取响应封装对象ResponseEntity |

- 对于RestTemplate方法中的url，可以使用字符串，并提供REST风格的传参方式（也可以通过UriComponentsBuilder来创建uri，阅读性更高）

```java
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/bookings/{booking}", String.class, "42", "21");

Map<String, String> vars = Collections.singletonMap("hotel", "42");
String result = restTemplate.getForObject(
        "https://example.com/hotels/{hotel}/rooms/{hotel}", String.class, vars);
```

- RestTempate方法的返回值有两种，javaBean和ResponseEntity，根据方法参数中的responseType来确定它们的泛型
- springMVC提供HttpEntity对象，来定义请求中的HTTP方法，URL，标头和正文，并且使用消息转化器将javaBean转化为对应ContentType类型数据
- springMVC提供MultiValueMap，来定义表单提交数据，并被消息转化器处理
- 当没有使用HttpEntity对象进行body数据包装时，springMVC会在内部进行处理，对于对象使用转换器处理，首先是String消息转换器，之后为json消息转换器；因此对于String类型，则使用String消息转换器，将content-type设置为：text/plain;charset=ISO-8859-1；其他对象，则使用json消息转换器，将content-type设置为：application/json，但是需要保证该对象能够被json序列化（如果为integer就会报错400）

##### RestTemplate常用场景：

- 简单Get请求：

  ```java
  String result = restTemplate.getForObject(uri, String.class);
  ```

- 自定义请求头的Get请求：

  ```java
  HttpHeaders headers = new HttpHeaders();
  httpHeaders.setOrigin("xxx");
  HttpEntity<T> httpEntity = new HttpEntity<T>(headers);
  String result = restTemplate.exchange(url, HttpMethod.GET, httpEntity, String.class)
  ```

- Body传JSON数据的Post请求：

  ```java
  HttpHeaders headers = new HttpHeaders();
  headers.setContentType(MediaType.APPLICATION_JSON_UTF8);
  User user = new User();
  HttpEntity<User> httpEntity = new HttpEntity<User>(user，headers);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

- 表单传参的Post请求：

  ```java
  HttpHeaders httpHeaders = new HttpHeaders();
  httpHeaders.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
  MultiValueMap<String, String> map= new LinkedMultiValueMap<>();
  map.add("id", "1");
  HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(map, headers);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

- 表单传二进制数据的Post请求：

  ```java
  HttpHeaders httpHeaders = new HttpHeaders();
  httpHeaders.setContentType(MediaType.APPLICATION_FORM_URLENCODED);
  MultiValueMap<String, Object> map= new LinkedMultiValueMap<>();
  map.add("file", new File("xxx"));
  HttpEntity<MultiValueMap<String, String>> httpEntity = new HttpEntity<>(map, headers);
  String result = restTemplate.postForObject(url, httpEntity, String.class);
  ```

#### WebClient：

​		spring5.0提供的WebClient，来支持HTTP请求的反应式编程、非阻塞的客户端，并有效支持同步、异步和流方案，相对于RestTemplate，支持如下内容：

- 非阻塞式I/O，性能更好
- lambda表达式风格编程
- 支持异步请求调用
- 支持反应式流，通过**Reactor**实现

## WebSocket：

​		webSocket协议提供了一个标准API，通过单个TCP连接在客户端和服务器之间创建全双工双向网络通信通道，相对于HTTP轮询技术，更好的减少了网络资源的浪费，性能更高

​		spring框架提供WebSocket API，用于编写webSocket消息的客户端和服务器端应用程序，通过导入spring-websocket包来实现：

#### spring-webSocket：

​		通过WebSocketHandler和HandshakeInterceptor，来完成整个WebSocket服务器编写：

- WebScoketHandlert：

WebScoketHandlert有两个实现类，对应webSocket通信时的消息类型：

TextWebSocketHandler：文本类型、BinaryWebSocketHandler；二进制数据类型

```java
public class MyWebSocketHandler extends TextWebSocketHandler{
	
    //连接建立后，执行方法
	@Override
	public void afterConnectionEstablished(WebSocketSession session) throws Exception {
		super.afterConnectionEstablished(session);
	}
    
	//客户端发送消息时，执行方法
	@Override
	protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
		
	}
	
	//连接关闭时，执行方法
	@Override
	public void afterConnectionClosed(WebSocketSession session, CloseStatus status) throws Exception {
		super.afterConnectionClosed(session, status);
	}
	
	//websocket发生错误时，执行方法
	@Override
	public void handleTransportError(WebSocketSession session, Throwable exception) throws Exception {
		super.handleTransportError(session, exception);
	}
}
```

- @EnableWebSocket

xxxx测试失败

#### webSocket-api：

​		在Tomcat运行环境中，默认提供了对webSocket的支持，因此在springBoot中，我们可以通过内置Tomcat、或者直接使用外部Tomcat，来实现webSocket：

**参考笔记：JAVA利用WebSocker实现实时通信**