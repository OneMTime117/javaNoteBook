# Spring-web

**spring-web模块包括四个部分：SpringMVC、RestClient、Websocket、Spring WebFlux**

## SpringMVC：

- springMVC的依赖包：

  - spring-webMVC包：该包会依赖导入spring-web，一般情况下两者需要同时使用
  - servlet-api.jar：springMVC是基于servlet api来实现的

  **spring-webmvc和spring-web两个依赖包的作用：**

  ​       spring-web提供了核心HTTP集成，和一些便捷的servlet过滤器，可以整合其他基础的web框架技术（Hessian，Burlap）

  ​       spring-webmvc是对springMVC功能的实现，依赖于spring-web；一般只需要显式添加spring-webmvc依赖

- springMVC基本概念：

  springMVC是基于Servlet API构建的原始Web框架，使得开发者能够简单高效地实现了MVC分层架构，并且有很强地扩展性

### 1、springMC应用初始化过程：

​		springMVC和其他web框架一样，围绕前端控制器模式来操作servlet API，并搭配springIOC容器，完成对象管理

**1、Servlet容器初始化过程：**

- 初始化Listener，执行其contextinitialized方法
- 初始化filter，执行init方法
- 初始化servlet，执行init方法

**2、springMVC应用的初始化过程：**

​		springMVC提供`ContextLoaderListener`(Listener)、`DispatcherServlet`(servlet)来伴随Servlet容器的初始化，完成整个springMVC应用的初始化

- Servlet容器初始化`ContextLoaderListener`，执行`contextinitialized`方法，从而实现对webApplicationContext（IOC容器）的创建，并放入ServletContext属性中

- webApplicationContext创建过程中，初始化DispatcherServlet：

  ​	1、获取ServletContext中的webApplicationContext，作为父容器（spring容器）

  ​	2、读取DispatcherServlet属性，创建一个新的webApplicationContext，作为子容器（springMVC容器）

  ​	3、将DispatcherServlet相关Bean注入到springMVC容器中

#### 1.1、spring容器和springMVC容器关系

- springMVC容器为子容器，可以访问父容器中的Bean；而父容器不能访问子容器中的Bean；
- 父容器通过ApplicationContext来Bean的管理
- 子容器通过webApplicationContext来完成Bean的管理
- WebApplicationContext是ApplicationContext的子接口，增加了Web应用特性
- webApplicationContext子容器可以获取到ApplicationContext父容器：

```java
ApplicationContext parent = webApplicationContext.getParent();
```

- springMVC提供**RequestContextUtils**工具类，可以通过reuqest获取WebApplicationContext子容器：

```java
RequestContextUtils.findWebApplicationContext(request);
```

#### 1.2、ContextLoaderListener、DispatcherServlet配置

##### 1.2.1、xml配置：

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

##### 1.3.2、WebApplicationInitializer接口配置

​		springMVC提供`WebApplicationInitializer`接口，通过ServletContext来配置Servlet容器组件(Servlet、Listener、Filter)，完成dispatcherServlet配置

​		原理：servlet-API提供ServletContainerInitializer接口；在Servlet容器初始化时，通过SPI机制调用指定ServletContainerInitializer接口实现类的onStartup方法；而springMVC提供SpringServletContainerInitializer，其onStartup方法中，又会调用所有`WebApplicationInitializer`接口实现类的onStartup方法；通过这种形式，springMVC实现对web.xml文件的替代

​		注意：

- Servlet在初始化时，还是会先读取web .xml配置，然后再处理ServletContainerInitializer接口

- 如果没有web.xml文件，POM文件会提示缺失web.xml文件，解决方法：

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

##### 1.3.3、WebApplicationInitializer接口简化版配置

​		springMVC自身提供`WebApplicationInitializer`抽象类，来简化Servlet容器相关配置

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
    
    //DispatcherServlet相关属性设置
    @Override
	protected void customizeRegistration(ServletRegistration.Dynamic registration) {
		super.customizeRegistration(registration);
		registration.setLoadOnStartup(1);
		registration.setAsyncSupported(true);
	}
}
```

注意：

- AbstractAnnotationConfigDispatcherServletInitializer是基于springIOC容器注解形式完成对spring父容器、springMVC子容器的配置，因此需要开发者提供spring容器配置类、springMVC容器配置类

- 配置方式选择：

​		三种配置方式只能同时存在一种；搭配springIOC容器配置注解，选择第三种方式，更加简单明了

### 2、DisparcherServlet

​		前端拦截器，拦截所有Servlet请求，交给springMVC处理

**初始化参数：**

| 参数                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| contextClass                   | 用于实例化的参数class对象，默认XmlWebApplicationContext，即springMVC容器的上下文对象 |
| contextConfigLocation          | pringMVC容器的上下文对象的配置文件路径                       |
| namespace                      | XmlWebApplicationContext的命名空间，默认为**[servlet-name]-servlet**，在web.xml中使用 |
| throwExceptionIfNoHandlerFound | 在找不到请求的处理程序时是否抛出NoHandlerFoundException异常；默认为false |

**常用属性：**

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| isAsyncSupported | 默认为false，开启支持异步请求处理                            |
| loadOnStartup    | 默认-1，让dipatcherServlet懒加载；可以设置为1，让其在项目启动时就加载 |

#### 2.1、DispatcherServlet相关组件：

dispatcherServlet整个功能的实现，依赖于一系列相关组件，并由springMVC容器管理

| 组件                                  | 作用                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| HandlerMapping                        | 将请求与对应处理程序、拦截器进行映射，用于进行请求的预处理和后置处理 |
| HandlerAdapter                        | 调用请求映射的处理程序                                       |
| HandlerExceptionResolver              | 用于解析处理程序中的异常，将其映射到对应处理程序、HTML错误视图或其他目标中 |
| ViewResolver                          | 视图解析器，用于提供视图名称和视图之间的映射                 |
| LocaleResolver、LocaleContextResolver | 国际化解析器，实现模板视图的国际化                           |
| ThemeResolver                         | 样式解析器，实现模板视图动态样式                             |
| MultipartResolver                     | 处理post请求中的multipart/form-data类型的表单提交，用于处理二进制数据（文件） |
| FlashMapManager                       | 管理FlashMap对象，实现在重定向时，属性在请求中的传递         |

#### 2.2、MultipartResolver

​		在Servelt-api中，对于multipart/form-data的表单数据，需要开发者手动操作body输入流；springMVC提供MultipartResolver组件来提供简单处理：

​		将HttpServletRequest封装为MultipartHttpServletRequest，读取body输入流，生成MultipartFile对象

注意：

- springMVC默认没有配置MultipartResolver，需要开发者手动配置

**MultipartResolver配置：**

基于Servlet版本，springMVC提供两种配置方式：

**1、standardServletMultipartResolver**

​		在Servlet3.0后，默认提供了对multipart/form-data的表单数据处理，springMVC通过standardServletMultipartResolver来整合使用

```java
	@Bean("multipartResolver")
	public MultipartResolver multipartResolverConfig() {
		return new StandardServletMultipartResolver();
	}
```

​		StandardServletMultipartResolver内部实现，基于Servlet使用AbstractMultipartHttpServletRequest对象对请求进行封装，读取multipart/form-data的表单数据，将其进行缓存；因此在使用该方式时，需要在Servlet容器启动前，额外配置其缓存路径（默认路径为Tomcat下的tmp目录）：

​		通过AbstractAnnotationConfigDispatcherServletInitializer，对DisparcherServlet属性进行配置

```java
protected void customizeRegistration(Dynamic registration) {
        //配置Servelt提供的解析器的本地缓存路径（解析请求中的multipart/form-data数据，需要进行缓存，才能多次操作）
		registration.setMultipartConfig(new MultipartConfigElement("/tmp"));
	}
}
```

**2、CommonsMultipartResolver**

​		在Servlet3.0之前，需要额外使用Commons-FileUpload包来处理multipart/form-data的表单数据；而springMVC通过CommonsMultipartResolver来整合使用

​		由于目前普遍Servlet版本在3.0以上，因此不进行该配置说明

### 3、springMVC控制器

​		springMVC其核心就是简化MVC应用中，控制层的开发；编写控制器（**Controller**）、请求映射处理器（**Handler**）

#### 3.1、控制器注解

- @Controller

  ​		@Component的组合注解，将@Controller注解类注册到springMVC容器中，作为控制器

- @ResponseBody

  ​		注解在Controller、Handler上，表示将Handler返回值放入ResponseBody中，不进行视图解析

- @RestController

  ​		@Controller、@ResponseBody的组合注解，避免在每个Handler上添加@ResponseBody	

#### 3.2、处理器注解

##### 3.2.1、@RequestMapping

用于声明处理器，将请求映射到指定控制器方法中；并通过属性：URL、http方法、请求参数、Headers来进行匹配

@RequestMapping属性：

| 属性名   | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| name     | 该控制器的映射Name                                           |
| value    | path的别名                                                   |
| path     | 所映射请求的url，支持相对路径匹配、REST风格传参              |
| method   | HTTP方法，RequestMethod[]，可以匹配多个HTTP方法              |
| params   | 请求参数列表，不是固定匹配，包括所有就满足，**即请求参数中必须包含当前规定的所有参数** |
| produces | 指定响应头Content-Type类型值，**通过该属性可以定义响应数据的编码格式** |
| headers  | 请求中必须包含指定请求头                                     |

springMVC提供@RequestMapping多个变体注解，用于方便定义Http方法类型：

**@GetMapping、@PostMapping**

注意：

- @RequestMapping可以注解在类上，定义所有方法的映射规则，但path会通过拼接进行匹配

```java
@RestController
@RequestMapping(path="/data")
public class DataController {
	
    @RequestMapping(path="/test")
	public String Test() {
		return "succsee";
	}
```

此时实际请求映射的path为： /data/test

- 对于method、params、produces、headers属性，方法注解优先级大于类

##### 3.2.2、处理器参数注解

​	springMVC提供一系列方法参数注解，Http请求数据封装注入到控制器方法实参中，实现对请求数据的自动获取

| 注解              | 作用                                                         |
| ----------------- | ------------------------------------------------------------ |
| @PathVariable     | 获取URL中使用REST风格的传参值                                |
| @MatrixVariable   | 获取URL中矩阵变量传参值（一般不使用）                        |
| @RequestParam     | 获取Servlet中的请求参数（url传参、form-data传参）；对于multipart/form-data，搭配MultipartFile对象进行接收 |
| @RequestHeader    | 获取请求头参数值                                             |
| @CookieValue      | 获取Cookie中的参数值                                         |
| @RequestBody      | 获取请求Body中的内容，通过HttpMessageConverter根据请求头中的Content-Type对内容进行解析，转化为java对象 |
| @RequestPart      | 获取multipart/form-data数据（和@RequestParaml类似），但可以额外指定数据的Content-Type，从而将其转化为指定格式类型对象（比如将JSON文件直接转化为java对象） |
| @ModelAttribute   | 将所有URL获取的参数和form-data获取的参数，绑定到java对象中（必须提供无参构造方法） |
| @SessionAttribute | 获取HttpSession对象中的属性，如sessionID                     |
| @RequestAttribute | 获取HttpServeltRequest对象中的属性（用于请求转发，但在前后端分离的项目中不会使用） |

**注意：**

- 除@ModelAttribute外，所有方法参数注解都有一个**required**属性，默认为true，即不能够忽略：

  ​		当请求参数中不包含当前指定的参数时，springMVC就会抛出异常，提示当前类型的参数没有被绑定，当required属性为false时，springMVC就会忽略没有绑定的参数，参数默认为null

- 参数没有注解时，springMVC默认会根据参数类型进行早期值匹配；如果没有匹配到时，则通过BeanUtils.isSimpleProperty方法判断当前参数类型是否为简单数据类型；是，则自动添加一个属性required=false的@RequestParam注解；不是，则自动添加@ModelAttribute注解

- @RequestParam和@RequestParth都可以获取multipart/form-data数据，但是两者有区别：

  @RequestParth额外提供content-Type属性，对文件内容进行解析；但一般情况下不会使用，因为在实际开发中，不会进行这种方式的传参

**@RequestParam注解的使用：**

​		@RequestParam提供name、value两个属性值，来对应获取http请求数据中的参数名，从而获取其参数值；**当不进行两个属性值声明时，默认使用方法形参名匹配**

​		@RequestParam单独还提供defaultValue属性，定义当参数没有被绑定时的默认值

注意：defaultValue只能提供String类型值，需要保证所对应形参类型可以进行有效转化

##### 3.2.3、处理器参数早期匹配值

​		springMVC对于一些特殊对象，并不需要添加参数注解，就能在早期自动完成参数绑定，即参数早期匹配值，它们包含ServletAPI中的常用对象和springMVC封装好的ServletAPI操作对象

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
| HttpEntity<B>                                               | 获取请求中的body和headers                                    |
| UriComponentsBuilder                                        | 用于构造和编码URI                                            |
| Errors、BindingResult                                       | 用于获取springMVC参数绑定的错误信息，需要保证它们在所有绑定数据的最后 |
| javax.servlet.http.PushBuilder                              | 获取Servlet 4.0 提供用于访问HTTP/2资源推送的对象             |
| java.security.Principal                                     | 当使用spring-security时，获取用于验证用户身份的Principal实现类 |

另外有关视图解析、重定向的对象，无需了解

##### 3.2.4、处理器返回值类型

​		在搭配@ResponseBody时，handle能够返回任何java对象；也可以搭配特殊类型，来设置响应头信息：

| 返回值类型                                                   | 作用                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| HttpEntity<B>、ResponseEntity<B>                             | 用于定义完整的响应信息（body与Headers），返回值会通过HttpMessageConverter写入Response对象中 |
| HttpHeaders                                                  | 定义Headers信息，即无正文数据                                |
| String、View、java.util.Map、org.springframework.ui.Model、ModelAndView | 定义视图、视图模型数据（前后端分离不会使用）                 |
| void                                                         | 默认响应已经被处理，若没有则会抛出异常响应500                |

#### 3.3、HttpMessageConverter

http消息转化器，有两个作用位置：

1、读取requestBody数据，将其转化为Java对象

2、将带有@ResponseBody注解的Handler返回值对象，转化为指定类型数据

```java
public interface HttpMessageConverter<T> {
	//是否可以转化requestBody数据
	boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
	
    //是否可以转化responseBody数据
	boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);

    //可以支持的所有MediaType类型
	List<MediaType> getSupportedMediaTypes();

	//requestBody数据转化为java对象
	T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
			throws IOException, HttpMessageNotReadableException;

	//将java对象写入到responseBody中，并设置ConentType
	void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
			throws IOException, HttpMessageNotWritableException;

}
```

​		springMVC会根据需要转化数据的javaType和MediaType来进行选择合适的HttpMessageConverter实现类，最常用的实现类为`StringHttpMessageConverter`、`MappingJackson2HttpMessageConverter`，

- StringHttpMessageConverter：

  ​		javaType=String、MediaType=text/plain、*/ *，将数据以字符串形式处理；MediaType=application/json时，charset=utf-8；其他则charset=ISO-8859-1

- MappingJackson2HttpMessageConverter：

  ​		javaType=Object、MediaType=application/json,application/ +json，会将数据进行Json序列化/反序列化处理；charset=utf-8

注意：

- StringHttpMessageConverter优先级大于MappingJackson2HttpMessageConverter

##### 3.3.1、处理requestBody数据

​		springMVC通过@ReuqestBody来接收由HttpMessageConverter转化来的对象，然后获取所有HttpMessageConverter实现类集合，根据参数类型和请求头ContextType，按顺序进行匹配

##### 3.3.2、处理handle返回值

​		在处理Handle返回值时，HttpMessageConverter首先会根据请求头Accept，来判断客户端支持返回的MediaType类型，然后获取所有HttpMessageConverter实现类集合，根据返回值类型和响应头ContextType，按顺序进行匹配

​		注意：

​		由于StringHttpMessageCoverter优先级大于MappingJackson2HttpMessageConverter，因此如果handle返回值为String类型时，会优先被StringHttpMessageCoverter处理，并使用默认ContextType=text/plain；charset=ISO-8859-1，此时返回结果就会乱码；有两种解决方法：

- 通过@RequestMapping的Produces属性，在Accept请求头的基础上，进一步限制MediaType类型，让StringHttpMessageCoverter使用application/json；charset=utf-8处理
- 修改StringHttpMessageCoverter中的DefaultCharset，默认使用charset=utf-8处理

#### 3.4、Converter

​		springMVC类型转化器，用于对url、form表单传参数据，进行java对象的转化处理

##### 3.4.1、内置转化器

​		springMVC默认提供了对基础类型和集合对象的转化：

- String转数字、boolean、枚举、Locale、char、Properties
- 基础数据类型转包装类
- 数组转任意集合

##### 3.4.2、自定义转化器

​		对于一些复杂参数，开发者可以通过自定义转化器来全局处理，简化参数接收，springMVC提供两个接口来实现：

​		**自定义converter解析失败抛出异常后，会先被springMVC默认解析器处理，如果无法处理，则再抛出异常**

- **Converter<S,T>**

  使用Converter<S,T>接口,原数据为String类型，T为需要转化的对象，以date类型为例

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
  
  Converter可以直接注入到IOC容器中，springMVC会自动获取IOC容器中的所有处理器，进行匹配处理
  
- **ConverterFactory<S,R>**

  使用ConverterFactory<S,R>接口，原数据为String类型，R为某种特殊接口、类，可以处理所有参数类型为R的子类数据

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

  - 相对于Converter，ConverterFactory可以动态处理某种类型的所有子类数据转化

  - converterFactory必须在WebMvcConfigurer中进行手动配置，并不能通过注入IOC容器而生效

  
  ```java
  	//注册格式化器、转换器工厂
  	public void addFormatters(FormatterRegistry registry) {
  		registry.addConverterFactory(new CommonEnumConverterFactory());
  	}
  ```

#### 3.5、WebDataBinder

​		在spring3.0以前，springMVC使用WebDataBinder完成处理器上所有参数数据的绑定、校验和格式化，内部通过PropertyEditor来完成；但该方式有明显的缺点：

- PropertyEditor只能进行Spring和Object的转化（无法完成如，long类型时间戳 转Date类型）

- PropertyEditor线程不安全，每次使用都需要重新创建

- PropertyEditor没有限制转化的参数类型，因此在自定义PropertyEditor，需要手动判断当前请求参数数据是否满足处理条件；也因此不支持细粒化的类型转化或格式化（如使用Date同时接收日期、日期时间）

  

**在spring3.0以后，springMVC在WebDataBinder中添加了Converter组件来进行数据的类型转化和格式化，并使用Validator组件完成数据的校验，最终生成BindingData绑定结构**

​		因此，在webDataBinder中，并不推荐使用PropertyEditor

##### 3.5.1、WebDataBinder参数绑定过程

1、在请求被拦截后，映射到指定处理器处理前，通过request、绑定参数对象、参数名，创建该请求参数的WebDataBinder

2、执行初始化方法，获取注册的所有PropertyEditor集合

3、选择合适的PropertyEditor处理，对处理器参数进行绑定

springMVC中，请求所有参数数据都会交给WebDataBinder进行数据类型的转化、校验和格式化，而相对于Converter转换器，它们的作用时间不同：

​		在webDataBinder进行数据绑定前，内部已经获取了所有Converter，从而遍历所有converter，依次对每个参数进行数据转换

##### 3.5.2、WebDataBinder初始化

​		springMVC提供全局和控制层级别两种方式，来定义WebDataBinder初始化方法；在初始化方法中，可以注册PropertyEditor自动完成数据绑定，并对指定处理器参数手动赋值

- 全局注册：

  通过实现WebBindingInitializer接口，在WebDataBinder创建时，会自动回调该接口方法

  ```java
  public interface WebBindingInitializer {
  
  	void initBinder(WebDataBinder binder);
  }
  ```

  但一般情况下，应该继承其唯一实现类ConfigurableWebBindingInitializer，在springMVC配置基础上，修改WebDataBinder配置

- 控制层级别注册：

  springMVC提供@InitBinder，在控制器中定义当所有处理器参数绑定所使用的WebDataBinder对象的初始化方法

注意：

​		两种方式可以同时存在，全局初始化方法先执行，控制层级别注册后执行，因此后者优先级更高

#### 3.6、控制器异常处理

​		当前处理器执行期间发生异常时,springMVC会将异常交给一系列HandlerExceptionResolver，通过责任链模式进行异常处理

​		HandlerExceptionResolver实现类：

| HandlerExceptionResolver实现类    | 描述                                                         |
| --------------------------------- | ------------------------------------------------------------ |
| SimpleMappingExceptionResolver    | 通过异常名映射返回指定错误视图（一般用于前后端不分离项目）   |
| DefaultHandlerExceptionResolver   | 根据异常类型，返回默认HTTP状态代码                           |
| ResponseStatusExceptionResolver   | 根据异常类型，返回指定HTTP状态代码（在异常类上添加@ResponseStatus进行设置） |
| ExceptionHandlerExceptionResolver | 根据异常类型，选择对应@ResponseHandler注解方法进行处理       |

​		springMVC默认注册了三种异常处理解析器，并定义了它们的优先级，由高到底：

ExceptionHandlerExceptionResolver、ResponseStatusExceptionResolver、DefaultHandlerExceptionResolver

##### 3.6.1、@ResponseHandler

​		一般情况下，开发者都会自定义响应格式；因此一般使用@ResponseHandler对指定异常进行处理，搭配@ResponseBody响应固定错误内容

```java
	@ExceptionHandler(HttpRequestMethodNotSupportedException.class)
	@ResponseBody
	public ResponseEntity handleException(HttpRequestMethodNotSupportedException e) {
		log.error(e.getMessage(), e);
		return Result.error("不支持' " + e.getMethod() + "'请求");
	}
```

注意：

- 当多个@ExceptionHandler指定的异常存在继承关系时，ExceptionHandlerExceptionResolver会优先选择，当前继承关系最近的异常处理器
- 异常处理器本质上也是处理器，并且需要声明在控制器中，并只在当前控制器生效

#### 3.7、Model对象处理

​		处理器在处理请求时，都会提供一个Model对象，用于作为数据中间层，存储和访问数据（在前后端不分离的情况下，Model对象可以被视图层访问）

​		springMVC提供@ModelAttribute注解，用于开发者操作Model对象，并分为如下三个场景：

- @ModelAttribute注解在处理器参数上：

  ​		首先Model对象中，根据参数名获取对应匹配值，如果没有，则通过WebDataBinder进行参数绑定，并将其存入到Model中；如果有，则直接从model中获取

  ​		**一般用于请求参数绑定，将参数转化为JavaBean**

  ```java
  	@RequestMapping("/testA")
  	public String  test(@ModelAttribute("userA") User user) {
      }
  ```

- @ModelAttribute注解在非处理器方法上：

  ​		在当前控制器的每个处理器执行前，调用该方法并将其返回值存入到Model对象中

  ​		**一般用于初始化Model对象中的属性**

  ```java
  	@ModelAttribute("userA")
  	public User initModel(String id,String name) {
  		User user = new User();
  		user.setId(id);
  		user.setName(name);
  		return user;
  	}
  
  ```

- @ModelAtrribute注解在处理器方法上：

  ​		此时会将返回值存入Model对象中，并且会响应到当前请求路径的.jsp视图页面中

  ​		**在前后端分离的情况下，一般不使用**

注意：

- 处理器可以通过早期值匹配，直接获取Model对象

#### 3.8、控制器增强

​		springMVC提供@ControllerAdvice，定义增强版控制器，搭配@InitBinder、@ExceptionHandler、和ModelAttribut；实现对WebDataBinder初始化、异常、Model对象的全局处理

​		@ControllerAdvice本质上还是控制器，但不能搭配@RequestMapping定义处理器，映射处理请求;@ControllerAdvice默认作用所有@Controller，但是可以通过指定basePackages，来自定义作用范围

- 全局初始化WebDataBinder 

  ​		@ControllerAdvice和@InitBinder 搭配使用，全局初始化WebDataBinder对象

- 全局异常处理：

  ​		@ControllerAdvice和@ExecptionHandler搭配使用，全局处理控制层抛出的所有异常；

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

- 全局初始化Model对象

  @ControllerAdvice和@ModelAttribut搭配使用，全局处理所有请求Model对象；由于Model对象的通用性不强，一般不会全局处理，以免干扰过多不相干的映射处理器

### 4、springMVC拦截器

​		拦截器是web框架的必备功能，增强开发者对请求全局的控制处理，相对于Servelt api提出的过滤器，功能更加强大

#### 4.1、拦截器使用：

​	springMVC提供HandlerInterceptor接口，对所有可以映射的请求进行拦截处理：

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

- preHandle：

  ​		在处理器执行前调用，返回true，则将请求交给下一拦截器或处理器；返回false，则认为请求已被处理，不在进行任何后面处理；一般用于用户认证

- postHandle：

  ​		在处理器返回ModelAndView对象前调用，用于对ModelAndView对象进行额外处理；无论处理器是否返回Model、View对象，都会封装生成一个ModelAndView对象

  ​		一般用于对视图、Model的统一处理

- afterCompletion：

  ​		在处理器返回ModelAndView对象后调用；一般用于对控制层的统一异常处理、日志处理（**无论Handler执行是否发生异常，都会执行该方法**）

#### **4.2、拦截器配置：**

通过WebMvcConfigurer配置接口，添加过滤器，并配置过滤器拦截路径

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

#### 4.3、拦截器和过滤器的区别：

1、拦截器基于JAVA反射机制，过滤器是基于函数回调

2、过滤器依赖于Servlet容器，拦截器不依赖

3、拦截器只会对映射了控制器方法的请求进行处理；过滤器会对所有请求进行处理

4、拦截器可以访问springMVC处理请求所提供的各对象（handle、ModelAndView、Exception）；过滤器只能获取Servelt请求的相应对象

5、在一个请求中，只会调用一次过滤器；但对于拦截器，会在控制器中进行映射方法处理器的跳转，从而调用多次拦截器（请求每映射一次处理器，都会调用拦截器）

### 5、springMVC-Servlet组件：

​		Servlet提供三大组件：Servelt、Filter、Listener；它们实现了整个Servlet容器对于Web请求的处理

​		SpringMVC本质上就是代替编写繁琐的Servelt，但依然支持使用Servlet原生组件（一般情况下，DispatcherServlet会拦截所有请求，因此开发并不是编写额外的Servlet）

#### 5.1、Servlet组件配置

springMVC应用提供多种Servlet组件配置方式：

- XML原生配置：

  Servlet容器会默认读取项目下的WEB-INF/web.xml文件，来进行Servelt组件配置

  **Filter、Listener执行顺序：**根据在web.xml中的声明顺序决定

- Servelt3.0注解原生配置：

  ​		在Servelt3.0以后，为了减少xml配置的繁琐，提供了@WebFilter、@WebServlet、@WebListener来配置Servelt组件

  **Filter、Listener执行顺序：**根据@WebFilter、@WebListener所注解的类名自然排序（推荐使用Filter01作为前缀来控制）

- springMVC配置：

  springMVC提供**WebApplicationInitializer**、**AbstractAnnotationConfigDispatcherServletInitializer**接口，来获取servelt容器的上下文对象ServeltContext，配置Servelt组件

  **Filter、Listener执行顺序：**根据配置添加顺序决定

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

**三种配置优先级：**

​		由于Servelt容器，会优先读取WEB-INF/web.xml文件，然后再将配置好的ServeltContext交给springMVC处理，因此其优先级为：

XML配置    >  注解配置  >   springMVC配置

注意：

- 推荐单独使用一种进行配置，减少不必要的处理
- springMVC配置的Servlet会默认开启非懒加载和异步处理模式，

#### 5.2、springMVC内置过滤器：

​		springMVC提供一套基于Filter接口的过滤器抽象类，对Filter进行扩展，方便开发者在SpringMVC自定义复杂功能的过滤器：

- GenericFilterBean：将过滤器交给IOC容器管理，同时可以获取Servlet容器初始化参数
- OncePerRequestFilter：GenericFilterBean的子类，实现只对请求进行一次过滤（解决转发时，会导致request再次被过滤器处理的问题
- AbstractRequestLoggingFilter：OncePerRequestFilter的子类，提供额外beforeRequest、afterRequest方法，提供对过滤前后的编写相应代码（将整个过滤器代码分步化，增强过滤器全局处理请求的能力）

springMVC基于这些抽象类，提供了如下常用过滤器（并不会默认配置，需要开发者手动配置）

- CharacterEncodingFilter：

  ​		设置请求头的CharacterEncoding，默认使用utf-8；

  ​		springMVC在进行参数绑定时，对于url传参数据，会通过Servlet容器编码来进行解码；而非url传参数据，则使用CharacterEncoding进行解码；（url会被Servlet容器进行编码处理，因此使用Servlet容器默认编码来进行解码）

- CorsFilter：

  ​		实现CORS处理

- HiddenHttpMethodFilter：

  ​		可以将form表单的请求转化为DELETE、PUT（form表单只支持put、post）

- ShallowEtagHeaderFilter：

  ​		实现对HTTP缓存中Etag请求头的处理

### 6、RequestContextHolder

​		springMVC提供RequestContextHolder工具类，获取当前请求线程中的response、response对象

​		通过RequestAttributes实现ServletRequestAttributes接口，来获取内部的HttpServletRequest、HttpServletResponse：

```java
//通过RequestContextHolder来获取RequestAttributes
//两个方法在没有使用JSF的项目中是没有区别的
RequestAttributes reqAttributes = RequestContextHolder.currentRequestAttributes();
RequestAttributes reqAttributes = RequestContextHolder.getRequestAttributes();                          
//RequestAttributes向下转型为ServletRequestAttributes，然后获取其中的request、response对象
HttpServletRequest request = ((ServletRequestAttributes)reqAttributes).getRequest();
HttpServletResponse response = ((ServletRequestAttributes)reqAttributes).getResponse();
```

- **RequestContextHolder实现原理**

  ​		当请求被DispatcherServlet拦截后，在FrameworkServlet的doGet/doPost方法中，调用processRequest（request, response）来进行请求对象的预处理：

  ​		将当前请求的request、response对象封装到RequestAttributes中，使用RequestContextHolder的ThreadLoacl<RequestAttributes>属性进行保存，通过ThreadLocal技术，来实现多线程间的可见性

### 7、springMVC额外功能：

#### 7.1、URI处理：

​		springMVC提供UriComponentsBuilder工具类，来构建和编码URI

##### 7.1、构建URI

springMVC通过UriComponentsBuilder来方便开发者构建URI，url编码中默认使用utf-8，常用两种方式：

- 基于uri模块化配置，协议、host、path、quetry

```java
UriComponents uriComponents = UriComponentsBuilder.newInstance()
				.scheme("http")
				.host("localhost")
    			.port(8080)
				.path("/user/login")
				.queryParam("username", "yuanhuan")
				.queryParam("passwrod", "123456")
				.encode()
				.build();
		return uriComponents.expand().toUriString();
```

- 基于httpurl字符串，对于query可以使用占位符替代，然后在uriComponents中进行参数填充

```java
UriComponents uriComponents = UriComponentsBuilder
			.fromHttpUrl("http://localhost:8080/user/login")
			.queryParam("username", "{a}")
			.queryParam("passwrod", "{b}")
			.encode()
			.build();
	return uriComponents.expand("1","2").toUriString();
```

uriComponent在构建前后，都可以执行encode编码，但两种方式有本质的区别：

- 构建前调用：先对url模板进行编码，然后再对变量值进行编码，最后组装
- 构建后调用：先间url变量注入到url模板中，然后对完整的url进行编码

**前者会对URL变量进行严格编码，导致 ; 这样的路径合法，但具有保留含义的字符被转换，**

##### 7.2、URI模板工厂

springMVC提供DefaultUriBuilderFactory，来创建URIBuilderFactory，从而实现定义基础了URI模板，其构建的所有URIBuilder都在该baseURI上进行修改

```java
DefaultUriBuilderFactory urlBuilderFactory = new DefaultUriBuilderFactory("http://localhost:8080/user/login");

UriBuilder builder = urlBuilderFactory.builder();
return builder.queryParam("name","袁欢").build().toString();
```

注意：

- DefaultUriBuilderFactory还可以用于RestTemplate的构建，使其所有的uri都在baseURI基础上修改；如定义默认的协议、host、port和部分path
- DefaultUriBuilderFactory编码默认先处理url模板、然后处理变量值

##### 7.3、创建当前请求的URI

springMVC提供ServletUriComponentsBuilder，通过当前请求获取URI，并对其进行修改

```java
	UriComponents build = ServletUriComponentsBuilder.fromRequestUri(request)
        		.replaceQueryParam("name","袁欢")
				.build();
```

##### 7.4、创建控制器的URI

springMVC提供MvcUriComponentsBuilder，通过指定控制器类中的处理器，创建其URI

```java
UriComponents uriComponents = MvcUriComponentsBuilder
    .fromMethodName(UserController.class, "login", "袁欢").buildAndExpand();
	URI uri = uriComponents.encode().toUri();
```

注意：MvcUriComponentsBuilder基于ServletUriComponentsBuilder，其URI中的协议、Host、端口依赖于当前request的url；因此只能在Servlet请求处理中使用

#### 7.2、异步请求处理：

​		在Servlet3.0后，支持Servlet异步请求处理，避免线程阻塞，影响Servlet容器并发处理请求；springMVC通过**DeferredResult**、**Callable**作为处理器返回值，来整合编写Servlet异步请求：

```java
@RequestMapping("/testSync")
public DeferredResult<String> syncTask(){
	//定义异步结果集，定义异步处理超时时间
	DeferredResult<String> deferredResult = new DeferredResult<String>((long) 11000);
	Runnable runnable = new Runnable() {
		@Override
		public void run() {
			//异步处理代码逻辑，保存需要响应输出的结果
			deferredResult.setResult("hhhhSync");
		}
	};
    //指定异步线程
	Executors.newSingleThreadExecutor().submit(runnable);
	return deferredResult;
}
```

- **DeferredResult的工作过程：**

  1、SpringMVC调用request.startAsync()方法，从而使Servlet容器的工作线程释放，并使response处理打开状态

  2、SpringMVC将DeferredResult交给某个线程处理，得到Result结果，并进行处理输出到response中

  3、SpringMVC调用AsyncContext.complete方法，交给请求重新交给Servlet容器的工作线程

- **Callable和DeferredResult区别：**

  1、Callable不需要在处理器中指定线程执行，而是在处理器返回后，使用springMVC默认异步线程池处理

  2、Callable发生异常时，并不是在处理器方法中抛出，当依然能被全局异常处理器处理

- **springMVC异步线程池配置：**

  ​		springMVC提供多个AsyncTaskExecutor实现类，推荐使用ThreadPoolTaskExecutor，它是对java.util.concurrent.ThreadPoolExecutor的封装

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

注意:

- 异步请求需要保证处理请求的Servert和Filter都设置为可异步处理：asyncSupported=true
- springMVC提供AsyncHandlerInterceptor，来进行**被异步处理的请求**的拦截器配置（Filter开启异步支持，就能对被异步处理的请求生效）

#### 7.3、CORS：

​		springMVC提供了对CORS的内置支持，并且支持处理映射（HandleMapping）和处理器（Handle）两种级别的处理

##### 7.3.1、处理器级别

​		springMVC提供@CrossOrigin注解，来启用处理器级别的CORS处理：

```java
    @CrossOrigin
    @GetMapping("/{id}")
    public Account retrieve(@PathVariable Long id) {
        // ...
    }
```

@CrossOrigin属性：

| 属性名           | 作用                                                         |
| ---------------- | ------------------------------------------------------------ |
| origins          | 指定允许的跨域请求地址，默认为空，允许所有                   |
| allowedHeaders   | 指定允许的请求头，默认空，但CROS规范默认允许一些请求头如：Content-type |
| exposedHeaders   | 指定要暴露的请求头，默认为空                                 |
| methods          | 指定允许的http请求方法，默认为空，为当前控制器方法允许值     |
| allowCredentials | 允许请求发送cookie等安全证书，默认不启用                     |
| maxAge           | 预检请求有效期，默认-1 永不过期                              |

##### 7.3.2、处理映射级别

​		在WebMvcConfigurer中进行配置：

```java
@Configuration
public class MvcConfig implements WebMvcConfigurer {	
	@Override
	public void addCorsMappings(CorsRegistry registry) {
		registry.addMapping("/**")//拦截url-pattern 
			.allowedOrigins("*")	//允许的跨域请求，* 为允许所有(浏览器对于跨域请求，会自动添加origin请求头，value为当前项目的ip+端口)
			.allowedHeaders("*") //允许的请求头
			.allowedMethods("*")  //允许的HTTP请求方法
			.allowCredentials(true) //允许请求发送cookie等安全证书
			.maxAge(3600);//预检请求有效期 单位s
	}
```

注意：

- 处理器级别优先级更高
- spring内置的CORS在处理映射时处理，优先级小于用户自定义拦截器；当使用拦截器进行权限校验时，跨域预检请求并不会带token，导致被拦截，间接导致整个跨域处理失败；开发者无法修改CORS处理的优先级，因此推荐使用原生CorsFilter方式处理，使CORS优先于springMVC拦截器处理

#### 7.4、HTTP缓存：

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

### 8、springMVC配置：

- **springMVC配置类**

一般情况下，springMVC容器只需要扫描Controller注解的类

在配置类上声明@EnableWebMvc，springMVC会注入`DelegatingWebMvcConfiguration`；它为WebMvcConfigurer的实现类，为springMVC提供默认webMvc配置，并整合IOC容器中，所有实现WebMvcConfigurer接口Bean所定义的配置

```java
@EnableWebMvc
@ComponentScan(basePackages = { "com.yh.mvc" }, useDefaultFilters = false, includeFilters = {@Filter(type = FilterType.ANNOTATION, classes = { Controller.class }) })
@Configuration
public class MvcConfig {
}	
```

- **自定义配置SpringMvc**

通过实现WebMvcConfigurer接口（配置类必须添加@EnableWebMvc）；或者直接继承`DelegatingWebMvcConfiguration`（不需要配置类添加@EnableWebMvc），对springMVC中的内部组件进行配置，注入到IOC容器中生效

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

### 9、springMVC请求处理流程：

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

  1. 通过Handler参数解析器，完成控制器方法的参数注入（此时会对请求的request对象进行处理）
  2. 反射调用控制器方法
  3. 通过Handler返回值处理器，对控制器方法返回值进行处理（此时会对请求的response对象进行处理）
  4. 最后返回ModelAndView对象

- Handler执行后，DispatcherServlet进行整个处理结果进行进一步处理：（期间当出现异常，则调用springMVC异常处理器，来响应请求）

  1. 调用拦截器的postHandler方法，对请求ModelAndView对象进行处理
  2. 通过视图解析器，完成对MV对象的解析，找到相应视图
  3. 然后将MV对象中的Model数据，渲染到视图中
  4. 在响应请求前，调用拦截器的afterCompletion
  5. 最后响应请求

注意：

- 对应前后端分离项目，不会进行MV对象的视图解析，而时通过JSON进行数据交互；因此直接使用@ResponseBody，将控制器方法的返回值以json形式写入响应Body中，而Handler会返回一个null的MV对象

### 10、DispatcherServelt源码解析：

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

在RequestMappingHandlerAdapter类中，使用Handler进行请求处理,核心方法为invokeHandlerMethod

```java
@Nullable
	protected ModelAndView invokeHandlerMethod(HttpServletRequest request,
			HttpServletResponse response, HandlerMethod handlerMethod) throws Exception {
		
		//封装request、response对象
		ServletWebRequest webRequest = new ServletWebRequest(request, response);
		try {
            //获取当前处理器的modelFactory(用于model对象的数据绑定)
			WebDataBinderFactory binderFactory = getDataBinderFactory(handlerMethod);
			ModelFactory modelFactory = getModelFactory(handlerMethod, binderFactory);

            //初始化处理器可执行方法对象（HandlerMethod）
			ServletInvocableHandlerMethod invocableMethod = createInvocableHandlerMethod(handlerMethod);
            //封装参数解析器（HandlerMethodArgumentResolver）
			if (this.argumentResolvers != null) {
			invocableMethod.setHandlerMethodArgumentResolvers(this.argumentResolvers);
			}
            //封装返回值处理器（tHandlerMethodReturnValueHandler）
			if (this.returnValueHandlers != null) {
		invocableMethod.setHandlerMethodReturnValueHandlers(this.returnValueHandlers);
			}
            
            //封装参数绑定器工厂（BinderFactory）
			invocableMethod.setDataBinderFactory(binderFactory);
            //封装参数名发现器，用于解析需要的参数名（ParameterNameDiscoverer）
			invocableMethod.setParameterNameDiscoverer(this.parameterNameDiscoverer);

           	//创建modelAndView容器对象，获取request请求中的所有属性，进行封装（完成@ModelAttribute、@SessionAttribute的属性填充）
			ModelAndViewContainer mavContainer = new ModelAndViewContainer();
			mavContainer.addAllAttributes(RequestContextUtils.getInputFlashMap(request));
			modelFactory.initModel(webRequest, mavContainer, invocableMethod);
//如果进行重定向后，不忽略默认model				         mavContainer.setIgnoreDefaultModelOnRedirect(this.ignoreDefaultModelOnRedirect);

            //创建异步web请求,实现异步请求处理
			AsyncWebRequest asyncWebRequest = WebAsyncUtils.createAsyncWebRequest(request, response);
			asyncWebRequest.setTimeout(this.asyncRequestTimeout);

			WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);
			asyncManager.setTaskExecutor(this.taskExecutor);
			asyncManager.setAsyncWebRequest(asyncWebRequest);
			asyncManager.registerCallableInterceptors(this.callableInterceptors);
		asyncManager.registerDeferredResultInterceptors(this.deferredResultInterceptors);

			if (asyncManager.hasConcurrentResult()) {
				Object result = asyncManager.getConcurrentResult();
				mavContainer = (ModelAndViewContainer) asyncManager.getConcurrentResultContext()[0];
				asyncManager.clearConcurrentResult();
				LogFormatUtils.traceDebug(logger, traceOn -> {
					String formatted = LogFormatUtils.formatValue(result, !traceOn);
					return "Resume with async result [" + formatted + "]";
				});
				invocableMethod = invocableMethod.wrapConcurrentResult(result);
			}

            //通过异步请求执行处理器，并使用创建modelAndView容器对象封装结果
			invocableMethod.invokeAndHandle(webRequest, mavContainer);
			if (asyncManager.isConcurrentHandlingStarted()) {
				return null;
			}

			return getModelAndView(mavContainer, modelFactory, webRequest);
		}
		finally {
			webRequest.requestCompleted();
		}
	}
```

此时在ServletInvocableHandlerMethod类中，进行处理器方法的执行，核心方法为invokeAndHandle

```java
		//使用处理器方法处理请求
		Object returnValue = invokeForRequest(webRequest, mavContainer, providedArgs);
		setResponseStatus(webRequest);
```

然后在InvocableHandlerMethod#invokeForRequest中，先进行方法参数处理：

```java
		Object[] args = getMethodArgumentValues(request, mavContainer, providedArgs);
		if (logger.isTraceEnabled()) {
			logger.trace("Arguments: " + Arrays.toString(args));
		}
		return doInvoke(args);
```

然后更具不同类型的HandlerMethodArgumentResolver，来实现各种传输类型参数解析：

常用有：

| 解析器                             | 参数接受方式  |
| ---------------------------------- | ------------- |
| RequestParamMethodArgumentResolver | @RequestParam |
| PathVariableMethodArgumentResolver | @PathVariable |
| RequestResponseBodyMethodProcessor | @RequestBody  |

**具体参考org.springframework.web.servlet.mvc.method.annotation包下的类**

## RestClient：

​		springMvc提供RestTmeplate，编写REST客户端服务，以同步方式发送请求；提供更加方便快捷的API

### 1、RestTmplate实例化：

​		RestTmplate默认构造方法，底层使用java.net.HttpURLConnection进行HTTP请求，开发者可以通过ClientHttpRequestFactory接口参数，来通过构造方法来进行配置；springMVC提供如下HTTP客户端库的支持：

- HttpComponentsClientHttpRequestFactory ：用于配置Apache HttpComponents包
- Netty4ClientHttpRequestFactory：用于配置Netty4 包
- OkHttp3ClientHttpRequestFactory：用于配置OkHttp包
- SimpleClientHttpRequestFactory：默认使用，用于配置java.net包

### 2、RestTemplate常用方法：

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

- RestTempate方法的返回值有两种：javaBean、ResponseEntity，根据方法参数中的responseType来确定它们的泛型
- HttpEntity用于定义请求中的HTTP方法，URL，请求头和请求体，并使用消息转换器处理请求体
- MultiValueMap用于定义表单提交数据，并被消息转化器处理
- 如果没有使用HttpEntity对象进行body数据包装，则会使用消息转化器进行处理

### 3、RestTemplate常用场景：

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

## WebSocket：

​		随着HTML5的出现，webSocket协议被提出：用于在单个TCP连接上，创建客户端和服务器之间的全双工网络通信；

​		webSocket协议出现之前，使用轮询（polling）和Comet技术来实现客户端和服务端的实时通信：

- 轮询：

  ​		客户端通过定期向服务器发送请求，来尽可能实时获取服务端需要发送给它的消息；很明显，这种方式会导致不必要的请求，浪费流量带宽和服务器资源

- Comet：

  ​		分为长轮询和SSE长连接：

  - 长轮询：在轮询基础上，服务端添加额外处理：服务端在没有可发送的消息时，不进行空消息响应，而是让请求保持连接，直到有新消息响应或请求连接超时；这样能够减少轮询请求的次数，降低资源消耗；但服务器将请求挂起还是会浪费资源
  - SSE长连接：HTML5新增功能SSE（Server-Sent Events），实现客户端向服务器发送一个HTTP请求，保持长连接，服务器可以不断单向发送消息；相对于轮询和长轮询，有效解决了服务器无法主动发送消息的难题；但服务器的消息发送还是基于HTTP响应流，在高并发情况下，对服务器的性能要求非常高

**webSocket工作流程：**

- 浏览器发送建立WebSockt连接的http握手请求，使用upgrade=websocket、Connection: Upgrade两个请求头声明http协议升级为WebSockt
- 完成HTTP请求响应后，WebSocket连接建立，此时服务器和客户端通过TCP协议进行数据传输

**webSocket优点：**

- 由于WebSocket数据传输基于TCP协议，消息发送没有重复的头部数据，数据量小、性能开销小
- webSocket是全双工通信，通信双方可以实时发送、接收消息，在实时通信的应用实现上，更加方便

### 1、tomact-websocket：

​		javaEE7中，提供出JSR-356规范：Java WebSocket API；所有Servlet容器对其进行了实现，比如Tomcat从7.0.47开始支持该规范，提供tomact-websocket包，来编写基于Servlet容器实现的webSocket消息处理程序

#### 1.1、JSR-356规范：

- webSocket服务端由一系列Endpoint组成；Endpoint是一个java对象，用于处理WebSocket请求，类似于Servlet处理HTTP请求（但不同的是Endpoint对于每个webSocket请求，都会创建一个新的实例处理）
- Endpoint默认三个生命周期：onOpen（webSocket握手成功）、onColse（会话关闭）、onError（连接异常）
- 每个Endpoint实例创建后，都会有一个唯一会话（javax.websocket.Session）；在执行webSocket各个生命周期回调方法时，都会将当前会话交给Endpoint处理；session中提供了RemoteEndpoint，用于给另一个端口（客户端）发送消息，并分为两种：BasicRemote（以同步方式发送消息）、AsyncRemote（以异步方式发送消息）
- javax.websocket.WebSocketContainer管理应用中的所有Endpoint，类似于ServletContext
- javax.websocket.server.ServerApplicationConfig配置注册所有的Endpint

#### 1.2、Endpoint加载过程

​		Tomcat提供ServletContainerInitializer接口实现类org.apache.tomcat.websocket.server.WsSci，在Servlet容器初始化时，使用SPI机制处理所有@ServerEndpoint注解类、Endpoint子类和ServerApplicationConfig配置类，加载过程如下：

- 构建WebSocketContainer实例，并添加WsFilter过滤器，来拦截WebSocket请求
- 将扫描到的Endpint类（符合条件的注解类和使用ServerApplicationConfig配置的Endpoint子类），注册到WebSocketContainer上，用于处理WebSocket请求

#### 1.3、WebSocket请求处理过程

- 服务器接收到webSocket请求后，使用WsFilter判断当前请求头Upgrade=websocket是否存在；
- 如果存在，则根据请求路径查找到对应Endpoint类，并进行协议升级处理
- 添加webSocket响应头、构造Endpoint实例、构造WsHttpUpgradeHandler，来将协议升级为WebSocket
- 协议升级后，创建webSocket会话、为webSocket协议处理器（UparadeProcessor）输出流添加写监听器
- webSocket连接创建完成，回调Endpoint的onOpen方法
- 为webSocket协议处理器（UparadeProcessor）输入流添加读监听器，获取webSocket客户端发送的消息，回调Endpoint的onMessage方法处理

#### 1.4、注解式Endpoint的使用

​		Tomcat提供两种Endpoint编写方式：

1、编程式：继承javax.websocket.Endpoint，实现其方法

2、注解式：定义一个Java对象，添加Endpoint相关注解，然后使用ServerApplicationConfig进行配置

推荐使用注解式，来定义Endpoint：

- @ServerEndpoint：

  ​		用于定义webSocket服务端口类，通过属性url来定义端口监听地址

- @OnOpen

  ​		用于定义webSocket服务端口类中，webSocket连接创建的回调方法

- @OnClose

  ​		用于定义webSocket服务端口类中，webSocket连接关闭的回调方法

- @OnError

  ​		用于定义webSocket服务端口类中，webSocket连接传输错误的回调方法

- @OnMessage

  ​		用于定义webSocket服务端口类中，webSocket消息接收后的回调方法

```java
@Slf4j
@Component
@ServerEndpoint("/websocket/demo/{userId}")
public class WebSocketDemo {

	// 线程安全的Map，来保存该websocket连接创建的所有Endpoint实例的会话
	private static CopyOnWriteArraySet<Session> websocketSessionSet = new CopyOnWriteArraySet<>();


	@OnOpen 
	public void onOpen(Session session, @PathParam("userId") String userId) {
		websocketSessionSet.add(session);
		//将用户信息放入session中
		session.getUserProperties().put("userId", userId);
		log.info(userId + "-------webSocket连接开启，当前服务连接数为" + websocketSessionSet.size());
	}

	@OnClose
	public void onClose(Session session, @PathParam("userId") String userId) {
		websocketSessionSet.remove(session);
		log.info(userId + "-------webSocket连接关闭，当前服务连接数为" + websocketSessionSet.size());
	}

	@OnError
	public void onError(Session session, Throwable error, @PathParam("userId") String userId) {
		log.error(error.getMessage());
		try {
			if (session.isOpen()) {
				session.close();
				websocketSessionSet.remove(session);
				log.info(userId + "-------webSocket连接关闭，当前服务连接数为" + websocketSessionSet.size());
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	@OnMessage
	public void onMessage(String message, Session session, @PathParam("userId") String userId) {
		log.info(userId + ":" + message);
		sentMessage(userId + ":" + message, session);
	}


	//点对点发送
	public static void sentMessage(String message, Session session) {
		try {
			if (session.isOpen()) {
				session.getBasicRemote().sendText(message);
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

	//根据userid进行点对点推送
	public static void sentMessageByUserId(String message, String userId) {
		for (Session session : websocketSessionSet) {
			String sessionUserId = (String) session.getUserProperties().get(userId);
			if (Objects.equals(userId, sessionUserId)) {
				sentMessage(message, session);
			}
		}
	}

	//全局发送
	public static void sentMessageAll(String message) {
		for (Session session : websocketSessionSet) {
			sentMessage(message, session);
		}
	}
}
```

注意：

- websocketSessionSet静态变量由于需要被多个WebSocketDemo实例共用，因此需要保证它们的线程安全

#### 1.5、springBoot整合tomcat-websocket注意：

1、springBoot内置tomact，因此需要将所有Endpoint实例交给IOC容器管理，并注入到内置Tomcat中，spring-webSocket包中，提供了ServerEndpointExporter对象来实现：**将IOC容器中的Endpoint实例交给内置Tomcat中的WebSocketContainer管理**

```java
@Configuration
public class WebSocketConfig {
	@Bean
	public ServerEndpointExporter getServerEndpointExporter() {
		//管理webSocket服务的组件，交给springboot内置Tomcat
		return new ServerEndpointExporter();
	}	
}
```

**如果使用WAR包，应用通过外部Tomcat运行时，需要省略该配置**

2、每个WebSocket请求都会创建一个Endpoint实例，因此Endpoint作为IOC容器Bean，其作用域为prototype（原型，多实例）；所以Endpoint中无法进行依赖注入，只能进行手动注入

3、在编写Endpoint类的回调方法时，可以基于springMVC处理器参数注解，来对webSocket请求参数进行封装处理，注入到回调方法参数中

4、在使用内置tomcat管理Endpoint后，spring-test模块需要保证提供启动tomcat，否则无法执行测试方法

- 原因：

  ​		spring-test默认只提供tomcat运行环境，导致无法在springboot应用初始化时，将IOC容器管理的Endpoint交给tomcat中的webSocket容器处理

- 解决方法：

  ​		修改spring-tset模块中的web环境，通过@SpringBootTest的webEnvironment属性设置，提供四种web环境：

  - MOCK（默认值），模拟tomcat运行环境，并不会启动Tomcat，减少开销
    RANDOM_PORT，启动Tomcat，使用随机端口
    DEFINED_PORT，启动Tomcat，使用默认端口
    NONE，不提供web环境	

  ```java
  @RunWith(SpringRunner.class)
  @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
  public class MarkTest {
  }
  ```

### 2、webSocket客户端Demo

```java
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
Welcome<br/>
<input id="text" type="text" /><button onclick="send()">Send</button>    <button onclick="closeWebSocket()">Close</button>
<div id="message">
</div>

</body>
<script type="text/javascript">
    var websocket = null;

    //判断当前浏览器是否支持WebSocket
    if('WebSocket' in window){
        websocket = new WebSocket("ws://10.8.96.228:8080/test/websocket");
    }
    else{
        alert('Not support websocket')
    }

    //连接发生错误的回调方法
    websocket.onerror = function(){
        setMessageInnerHTML("error");
    };

    //连接成功建立的回调方法
    websocket.onopen = function(event){
        setMessageInnerHTML("open");
    }

    //接收到消息的回调方法
    websocket.onmessage = function(event){
        setMessageInnerHTML(event.data);
    }

    //连接关闭的回调方法
    websocket.onclose = function(){
        setMessageInnerHTML("close");
    }

    //监听窗口关闭事件，当窗口关闭时，主动去关闭websocket连接，防止连接还没断开就关闭窗口，server端会抛异常。
    window.onbeforeunload = function(){
        websocket.close();
    }

    //将消息显示在网页上
    function setMessageInnerHTML(innerHTML){
        document.getElementById('message').innerHTML += innerHTML + '<br/>';
    }

    //关闭连接
    function closeWebSocket(){
        websocket.close();
    }

    //发送消息
    function send(){
        var message = document.getElementById('text').value;
        websocket.send(message);
    }
</script>
</html>
```

### 3、spring-webSocket：

​		spring-webSocket用于提供编写webSocket消息服务，实现对tomcat-webscoket的封装

#### 3.1、WebSocket-API

##### 3.1.1、WebScoketHandler：

​		WebSocketHandler，用于编写webSocket消息处理程序；它有两个实现类：

- TextWebSocketHandler（消息内容类型为文本）

- BinaryWebSocketHandler（消息类型为二进制数据）

```java
//创建处理器
@Compent
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


//配置处理器
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Autowired
	private MyWebSocketHandler myWebSocketHandler;
    
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myWebSocketHandler, "/myHandler");
    }
}
```

##### 3.1.2、HandshakeInterceptor：

​		HandshakeInterceptor用于处理webSocket进行http握手前后的逻辑，从而阻止http握手、或设置webSocketSession中的属性

```java
//创建http握手拦截器
public class MyHandshakeInterceptor implements HandshakeInterceptor {
	//webSocket发送握手请求
    @Override
	public boolean beforeHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Map<String, Object> attributes) throws Exception {
		return false;
	}

    //webSocket完成握手请求
	@Override
	public void afterHandshake(ServerHttpRequest request, ServerHttpResponse response, WebSocketHandler wsHandler, Exception exception) {
	}
}

//配置http握手拦截器
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

	@Autowired
	private MyWebSocketHandler myWebSocketHandler;

	@Autowired
	private MyHandshakeInterceptor myHandshakeInterceptor;

	@Override
	public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
		registry.addHandler(myWebSocketHandler,"/myWebSocket")
		.addInterceptors(myHandshakeInterceptor);
	}
}
```

##### 3.1.3、ServletServerContainerFactoryBean

​		ServletServerContainerFactoryBean用于配置WebSocketContainer（webSocket容器）属性

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Bean
    public ServletServerContainerFactoryBean createWebSocketContainer() {
        ServletServerContainerFactoryBean container = new ServletServerContainerFactoryBean();
        //设置servlet容器允许的webSocket最大消息缓存大小、空闲时间
        container.setMaxTextMessageBufferSize(8192);
        container.setMaxBinaryMessageBufferSize(8192);
        container.setMaxSessionIdleTimeout(30 * 1000L);
        return container;
    }
}
```

##### 3.1.4、webSocket同源处理

​		webSocket和Http一样，会被浏览器进行同源限制；spring-webScoket同样可以基于CORS方式进行处理：

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Autowired
	private MyWebSocketHandler myWebSocketHandler;
    
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myWebSocketHandler, "/myHandler")
            	.setAllowedOrigins("*");
    }
}
```

#### 3.2、SockJS降级备用方案

​		spring-webSocket处理提供一个WebSocketAPI外，其重要功能是：基于这套API，内部实现对非WebSocket代替方案的支持

- webSocket协议缺点：

​		webSocket协议需要浏览器支持（IE10之前不支持）、客户端和服务器之间所有中间节点(网关、代理服务器)支持，同时还需要保证这些中间节点不会对webSocket长连接、空闲连接进行关闭处理；因此在复杂的网络环境中，webSocket协议没有很好的兼容性

- SockJS：

  SockJS是对webSocket协议的模拟，默认优先使用WebSocket，当WebSocket不可用时，则使用HTTP流和HTTP长轮询两种方案进行降级处理，实现浏览器和服务器的全双工通信；提供基于JavaSript语言的客户端库（sockjs-client）

spring-webSocket提供了对SockJS服务器的实现:

```java
//配置处理器
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    @Autowired
	private MyWebSocketHandler myWebSocketHandler;
    
    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(myWebSocketHandler, "/myHandler").withSockJS();
    }
}
```

#### 3.3、STOMP

​		weboscket协议定义了两种消息类型：文本、二进制；spring-websocket提供了STOMP作为消息传递协议，在websocket协议下，作为子协议进行消息传递

STOMP协议：

​		面向简单文本的消息传递协议，依靠于双向通信网络协议（如TCP、WebSocket）；以HTTP帧为模型，进行消息传递

STOMP协议优点：

- 无需开发者自定义消息传递协议和格式
- 可以整合消息队列对消息进行管理订阅和广播消息
- 通过STOMP目标表头，将消息路由转换到springMVC的控制器中，实现将客户端消息转交给springMVC控制层处理

**需要依赖于spring-messaging包**

## Spring WebFulx

​		spring5在Web模块，引入了反应式编程，也叫响应式编程；

### 1、反应式编程：

**1、命令式编程缺点：**

- 在常规的命令式编程中，线程会按照顺序一个一个执行代码编写的任务；但是当遇到某些特殊任务时，如IO任务（从某个DB或远端获取数据），该线程就会被阻塞，直到数据被完全读取，这是一种资源浪费；
- 命令式编程进行并发编程时，由于存在线程阻塞，因此导致在一定并发任务量下，线程数量更多，处理更加复杂

**2、反应式编程优点：**

​		反应式编程是基于函数式编程基础之上，描述数据将会流经的管道或流，实时对当前传入的数据进行处理；相对于命令式编程，虽然同一时间处理数据的能力有限，但其线程异步非阻塞，能够持续不断的利用资源，避免资源浪费

### 2、反应式流：

​		2013年，spring其下工程师提供出了反应式流（Reactive Streams）规范，提供对非阻塞、带被压机制的异步流处理标准

反应流和java流的异同点：

- Reactive Streams和Stream都是用于处理数据的函数式API

- Stream只能同步处理有限的数据集，并只支持集合类型数据；Reactive Streams支持异步处理任意大小（包括无限）的数据集，并通过回压避免多大的数据压垮消费者

#### 2.1、反应式流规范：

​		反应式流规范定义了4个接口：

- Publisher		数据生产发布者

  ```java
  public interface Publisher<T> {
      //数据订阅
      public void subscribe(Subscriber<? super T> s);
  }
  ```

- Subscriber       数据订阅消费者

  ```java
  public interface Subscriber<T> {
  	
      //订阅成功
      public void onSubscribe(Subscription s);
  
      //订阅数据发送
      public void onNext(T t);
  
      //订阅数据发送失败
      public void onError(Throwable t);
  
      //订阅数据已全部发送
      public void onComplete();
  }
  ```

- Subscription    数据订阅管理器

  ```java
  public interface Subscription {
  	
      //开始获取订阅数据，并设置能够接收的数据量
      public void request(long n);
  
     `//取消订阅
      public void cancel();
  }
  ```

- Processor         数据处理器，继承了Publish、Subscriber，定义整个数据生产、消费的处理过程

  ```java
  public interface Processor<T, R> extends Subscriber<T>, Publisher<R> {
  }
  ```

#### 2.2、Reactor

​		Reactor是一个用于在JVM平台上，对反应式流规范实现的反应式库，用于构建反应式的非阻塞异步应用

目前常见的Java反应式库

- RxJava
- JDK9中的Flow包：只是提供抽象层，通过SPI机制引入第三方实现
- Vertx内部响应式流驱动
- Reactor：由spring公司维护，spring所有项目的反应式编程都基于该框架

由于spring在Java语言的影响力，因此推荐使用Project Reactor作为反应式库

#### 2.3、反应式流图

### 3、Reactor的使用

​		引入Maven依赖：

```xml
<dependency>
   <groupId>io.projectreactor</groupId>
   <artifactId>reactor-core</artifactId>
</dependency>
```

#### 3.1、Flux和Mono

​		todo



JDK8提出了函数式API，

spring WebFlux，无阻塞、支持Reactive streams背压，可运行在Netty、Undertow 和 Servlet 3.1+ 容器等服务器上运行。