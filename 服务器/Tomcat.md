# Tomcat

## 1、Servlet历史 

​	20世纪90年代，互联网和浏览器飞速发展，基于浏览器的B/S模式开始火爆。起初web服务器只能返回静态资源，无法实现动态请求

#### CGI

​	Common Gateway Interface通用网关接口，**用于实现服务器对请求的动态处理**，由C、shell编写，并且不能移植。其作用过程：

1. WEB服务器接收一个用户请求；
2. WEB服务器将请求转交给CGI程序处理；
3. CGI程序将处理结果返回给WEB服务器；
4. WEB服务器把结果送回用户；

对于JAVA语言，实现web开发，也就需要类似CGI程序，因此就提供了Servlet：

#### applet

​	java进行web开发，起初是通过applet（java小型程序），来让浏览器解析web服务器响应回来的java代码，从而实现动态请求；而applet就是安装在浏览器的插件，但是，其性能非常低，每个请求都需要在浏览器端创建一个线程执行，并且响应传输的java代码不能太多和太复杂

#### Servlet

​	在applet之后，提供出Servlet（server applet），即实现在web服务器端执行java代码。其处理过程如下：

1. WEB服务器接收一个用户请求；
2. WEB服务器将请求转交给WEB服务器关联的Servlet容器；
3. Servlet容器找到对应的Servlet并执行这个Servlet；
4. Servlet容器将处理结果返回给WEB服务器；
5. WEB服务器把结果送回用户；

#### Jsp

​	随着Servlet的发展，SUN公司发现Servlet编程非常繁琐：

1、每个Servlet都有近似的代码，存在大量冗余

2、需要在Servlet中编写前端代码（Http页面），而这种编写方式非常不直观，不利于前端页面的开发

​	在Servlet1.1中，提出了JSP （java server page），实现在静态文件中插入编写后台代码，从而方便后台代码的编写，当着同样存在一些问题：

1、前后端代码都会编写在JSP文件中，导致代码非常混乱

#### MVC思想

​	在Servlet1.2中，提出了MVC思想：

1、Servlet（C）：Serlvet完成Controller功能，用于接收请求和部分简单代码处理

2、Model（M）：Serlvet将请求数据交给Model进一步处理，Model处理完请求数据后，会返回封装好的model数据，封装到Servlet的响应对象中，同时将请求重定向到指定了JSP页面，进行JSP页面渲染

3、JSP（V）：通过JSP标签来在指定model数据的渲染方式

这种方式将整个动态请求处理过程分为了3步，大大减少了JSP中的后端代码，并方便后端代码的复用

## 2、Servlet规范

### 1、定义

- 广义：

  ​	基于java技术，搭配容器托管，实现对http请求的动态处理；是javaweb开发的一个重要组件（另外两个JSP/JSF，因前后端分离而被淘汰）

- 狭义：

  ​	一个JAVAEE规范，定义了http服务器调用和管理动态资源规则，以及使用java进行动态资源开发过程

### 2、Servlet容器

​	Servlet容器（也称之为Servlet引擎）用于管理Servlet生命周期，提供基于请求/响应发送模型的网络服务，来解析MIME请求和格式化MIME响应。通常Servlet容器会嵌入在以java开发的web服务器中使用 （Tomcat），也可以基于组件的形式外置扩展。

​	请求处理过程如图：

![](../Typora图片/Servlet容器请求处理过程.png)

Servlet容器会将请求封装为ServletApi中的HttpServletRequest对象，然后基于Servlet实例，调用器init（）、service（）方法对请求进行处理，输出HttpServletResponse对象，最后交给Servlet容器保证为http响应返回给客户端

### 3、xml.web文件

​	xml.web文件为web应用的部署描述文件（Deployment Descriptor），用于交给Servlet容器进行相关配置和部署，常用包括：

- ServletContext初始化参数
- session配置
- servlet、Filter、Listenner声明、映射、顺序和初始化参数

​       对于web应用程序的结构目录中，都会存在一个特殊的目录：**WEB-INF**

​	WEB-INF,该目录中的内容不能通过Servlet容器直接提供给客户端访问，只能交给Servlet、Servlet容器进行处理，一般用于存放web.xml文件、classes目录（编译后的class文件）和lib目录（外部jar包）

### 4、Servlet

#### 1、Servlet生命周期

​	Servlet的生命周期由容器管理，因此Servlet不能够用代码控制其生命

Servlet生命周期分为如下几步：

**1、加载和实例化：**

​	Servlet的加载和实例化发生在该Servlet的第一次请求时，因此常常说Serlvet容器启动后，第一次访问会较慢，但通过web.xml配置文件中的<load-on-startup> 1 <load-on-startup>标签，可以定义Servlet容器启动时，该Servlet的加载的优先级，当为0，则不进行加载

**2、初始化：**

​	Servlet初始化会紧跟在实例化后进行，即执行Servlet的init方法

**3、请求处理：**

​	初始化后，Servlet就能够进行请求处理，执行service方法；在httpServlet抽象类中，也提供doGet、doPost方法进行请求处理

**4、销毁：**

​	Servlet容器会在合适的时候对Servlet实例进行销毁，而该策略取决于容器；当容器关闭时，Servlet对象一定会被销毁，调用destroy方法

#### 2、Servlet接口相关对象和方法

Servlet在Servlet规范中，继承关系如下：

![](../Typora图片/Servlet接口UML图.jpg)

**Servlet接口：**

​	Servlet对象，定义了Servlet生命周期中所使用的方法

```java
public interface Servlet {
    //初始化方法
    void init(ServletConfig var1) throws ServletException;

    //获取servletConfig
    ServletConfig getServletConfig();

    //处理请求方法
    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    //获取Servlet相关信息
    String getServletInfo();

    //毁灭方法
    void destroy();
}
```

**ServletConfig接口：**

​	Servlet配置对象，在Servlet初始化期间，用于Servlet容器向Servlet传输数据

```java
public interface ServletConfig {

	//获取Servlet实例名
    public String getServletName();

	//获取ServletContext
    public ServletContext getServletContext();

	//获取Servle初始化参数  
    public String getInitParameter(String name);

	//获取Servlet初始化参数名集合
    public Enumeration<String> getInitParameterNames();
}
```

**GenericServlet抽象类：**

​	在初始化前，将ServletConfig作为成员变量保存，并通过ServletConfig对象实现相关接口方法

**HttpServlet抽象类：**  

​	基于Http协议通讯规范，将service请求处理方法，根据请求方法，划分为多个方法，等待被重写

#### 3、Servlet请求处理

​	当Servlet初始化后，Servlet容器就可以使用它来处理客户端请求。客户端请求会被封装为ServletRequest对象，并将最后需要返回的响应封装为ServletResponse对象，因此Servlet的service方法，其实就是对两个对象的处理

​	当在Http请求下，Servlet容器所提供给Servlet操作的对象分别为：HttpServletRequest和HttpServletResponse

- **请求并发处理：**

  ​	Servlet实例本身是线程不安全的。Servlet容器会通过一个工作线程池，来并发调用同一个Servlet实例的service方法。因此需要开发者以代码的形式，保证Servlet的线程安全：

  1、不能在Servlet中使用全局变量

  2、使用同步块synchronized，保证线程安全（这样会严重影响性能）

- **请求异步处理：**

  ​	在web应用中，常常会由于某个请求处理时间过长(如JDBC连接超时），而导致该线程一直阻塞，这样会严重影响Servlet容器并发处理请求的。因此在Servlet3.0中，引入了异步请求处理功能，

### 5、ServletContext

​	ServletContext（Servlet上下文），Servlet容器会为每个web项目创建一个该对象。它全局唯一，所有servlet都共享该对象，因此也被称为**全局应用程序共享对象**

**作用：**

1、作为全局对象，能够实现在不同动态资源（Servlet）间传递和共享数据

```java
public Object getAttribute(String name);

public Enumeration<String> getAttributeNames();

public void setAttribute(String name, Object object);

public void removeAttribute(String name);
```

2、可以读取和配置全局初始化参数

```java
public boolean setInitParameter(String name, String value);

public Enumeration<String> getInitParameterNames();

public String getInitParameter(String name);
```

3、获取web应用下的资源文件

```java
public URL getResource(String path) throws MalformedURLException;

public InputStream getResourceAsStream(String path);
```

4、控制Servlet容器配置和部署

- 对Servlet、Filter、Listenner组件的管理
- 对Servket容器中，请求、响应的编码格式、session超时时间进行管理

**在Servlet容器中，使用ApplicationContext类对ServletContext接口进行实现**

### 6、ServletRequest

​	请求对象，用于封装客户端请求的所有信息。在HTTP协议中，将请求转为了 `HttpServletRequest`对象，封装http请求头和消息体消息

​	Request是请求对象的实现类，其接口继承包括：HttpServletRequest》》ServletRequest

#### 1、HTTP协议参数

​	Sevlet容器会将RUI和postBody中的参数，以K-V键值对的形式保存到`HttpServletRequest`对象中，并且同一个key可以对应多个value，在`ServletRequest`接口中，提供如下方法，来访问这些参数：

```java
//获取所有参数key
public Enumeration<String> getParameterNames();

//获取key对应的value数组
public String[] getParameterValues(String na
 public String getParameter(String name);me);

//获取所有的key-value参数
public Map<String, String[]> getParameterMap();

//获取key对应value数组的第一个值
public String getParameter(String name);
```

- **注意：**

  ​	对于HTTP1.1所定义的路径参数，必须通过getRequestURI/getPathInfo获取uri，然后手动解析内部传参

##### 1.1、post表单传参

- K-V键值对

​	必须保证content-type=application/x-www-form-urlencoded，并当调用用参数方法时，Servelt内部才会懒加载，读取请求输入流，进行表单中的参数绑定；之后Post请求的输入流也就无法使用

#### 2、文件上传

必须保证content-type=multipart/form-data,然后通过`HttpServletRequest`接口中的如下方法，进行文件访问

```java
//获取所有文件
public Collection<Part> getParts() throws IOException,
            ServletException;

//根据文件名获取指定文件
public Part getPart(String name) throws IOException,
            ServletException;

```

#### 3、属性

​	Request中的属性并不是由Http协议提供，而是通过Servelt容器进行设置，用于Servelt之间的数据传递。在`ServletRequest`接口中提供如下方法，进行操作：

```java
//获取请求中的所有属性名
public Enumeration<String> getAttributeNames();

//获取请求中的指定key的属性值
public Object getAttribute(String name);

//设置当前请求中的属性
public void setAttribute(String name, Object o);
```

#### 4、请求头

​	`HttpServletRequest`接口提供如下方法，来获取http请求中的头信息，Http请求头允许多个请求头name重复

```java
//获取所有请求头name
public Enumeration<String> getHeaderNames();

//获取指定请求头对应值的数组
public Enumeration<String> getHeaders(String name);

//获取指定请求头对应的第一个值
public String getHeader(String name);
```

​	Http请求头都会以String形式进行封装，HttpServletRequest接口提供如下方法，支持String对Date、int类型的转换(转换失败，则抛出异常)

```java
public int getIntHeader(String name);

public long getDateHeader(String name);
```

#### 5、请求路径传参

​	Servlet容器将请求路径分为了三个部分：

- ContextPath，Servlet容器的上下文路径，默认为/

- ServletPath，用于进行Servlet匹配映射的路径

- PathInfo，ContextPath、ServletPath除外后的路径，如果没有，则为null或者为/

  因此：requestURI=contextPath+servletPath+pathInfo

注意：

​	**对于.xx文件名结尾的路径，pathInfo为null，ServletPath就是该文件在项目中的路径**

在`HttpServletRequest`接口中，提供如下方法，来访问该三种路径和完整的URI和URL:

```java
public String getRequestURI();

public StringBuffer getRequestURL();

public String getContextPath();

public String getServletPath();

public String getPathInfo();
```

#### 6、非阻塞IO

​	Servelt 容器在进行异步请求和升级处理时，使用非阻塞IO来提高web容器并发量；在进行http请求数据的读取时，ServletInputStream提供一系列方法，来读取http请求数据

```java
public ServletInputStream getInputStream()    通过request获取http请求输入流

boolean isFinished();//判断是否数据读取完成

boolean isReady();//判断数据是否可以被无阻塞读取

void setReadListener(ReadListener listener)；//设置数据流读取监听器，进行方法回调
```

通过`ReadListener `来为`ServletInputStream`的数据读取提供一系列回调方法：

```java
onDataAvailable()；//当数据可以被读取时，进行回调

onAllDataRead()；//当数据读取完时，进行回调
 
onError(Throwable t)；//数据读取发生错误时，进行回调
```

#### 7、cookie

​	request通过getCookies方法，来获取请求中的cokie数组，它有一个个KV组成，服务器可以进行set并返回给浏览器，其中服务器会通过cokie中的sessionId来判断用户。

​	一般情况下，浏览器会将cookie设置为httpOnly cookie，防止暴露给客户端脚本，减少XSS风险

#### 8、SSL属性

​	对于https请求，会携带SSL属性数据，在Servlet容器中，会以java.security.cert.X509Certificate 类型的对象数 进行封装，交给开发者进行操作，并存放在Request对象的attribute属性中，key为：javax.servlet.request.X509Certificate

#### 9、国际化

​	http可以通过Accept-Language头和HTTP/1.1规范中的相关机制，来表示客户端的首选语言环境，并在Request中，以getLocale方法进行获取

#### 10、请求数据编码

​	对于get请求参数数据，会经过Servlet容器处理，封装到request对象的Parameter中，并且Servlet容器默认使用ISO-8859-1编码，将二进制数据进行编码，因此此时直接调用getParameter会导致中文乱码，只能对字符串进行ISO-8869-1解码，然后再进行UTF-8的编码

​	对于Post请求参数，并不会被Servlet容器预处理，而是在读取请求被处理时，根据content-type头进行选择处理，如果不进行指定，则默认使用ISO-8859-1进行post数据获取；为了避免无法解析，Request对象提供setCharacterEncoding（String enc）方法来指定解析数据的编码格式，**在post数据读取前执行**

#### 11、Request对象生命周期

​	每个Request对象都只能在Servlet的service方法，或过滤器的doFilter方法的作用域内有效；除非通过调用request.startAsync方法，实现请求异步处理；此时，request对象一直有效，直到执行AsyncContext.complete方法

### 7、ServletResponse

​	响应对象，封装服务器返回给客户端的所有信息，在HTTP协议中，包含http头信息和消息体

​	`Response`是响应对象的实现类，其接口继承包括：HttpServletResponse》》ServletResponse

#### 1、缓冲

​	Servlet容器提供缓冲功能，来提供响应数据的传输效率，并提供一系列对ServletResponse缓冲区的操作：

```java
//设置缓冲器大小
public void setBufferSize(int size);

//获取缓冲器大小
public int getBufferSize();

//刷新缓冲器
public void flushBuffer() throws IOException;

//清空缓冲器
public void resetBuffer();

//判断响应是否返回给客户端
public boolean isCommitted();

//清空所有响应数据（缓冲器和响应头）
public void reset();
```

​	对于reset、resetBuffer，必须保证响应没哟狯给客户端，否则会抛出IllegalStateException

#### 2、响应头、非阻塞IO

​	`HttpServletResponse`接口提供一系列方法来进行HTTP响应头的操作，整个API和请求头一致

​	`ServletOutputStream` 接口和请求输入流类似，支持非阻塞IO

#### 3、重定向和错误

```java
public void sendRedirect(String location) throws IOException;

public void sendError(int status, String message) throws IOException 
```

1、`HttpServletResponse`提供sendRedirect方法，将响应返回给客户端，并在响应头中指定一个url，让客户端再次发起请求

2、`HttpServletResponse`提供sendError方法，根据状态码来告诉客户端重定向到Servelt容器在web.xml中的指定错误页面，并将response响应体中的数据以text/html形式展示

**http重定向实现原理：**

​	1、请求响应状态码设置为302

​	2、添加location响应头

#### 4、国际化和字符编码

和request一样，`HttpServeltResponse`提供setContentType、setLocale来设置http协议中的响应头ContentType和ContentLanguage

#### 5、Response的生命周期

​	每个Request对象都只能在Servlet的service方法，或过滤器的doFilter方法的作用域内有效；除非通过调用request.startAsync方法，实现请求异步处理；此时，request对象一直有效，直到执行AsyncContext.complete方法

​	当Response对象销毁前，容器会flush响应缓冲区的数据，将其返回给客户端

### 8、过滤器Filter

#### 1、过滤器的原理和作用

​	过滤器是一种代码重用技术，全局对所有请求和响应在被Servlet处理前，进行额外处理，如改变Http请求、响应内容、header信息

Serlvet提供`Filter`接口，来定义过滤器：

```java
public interface Filter {
    void init(FilterConfig var1) throws ServletException;

    void doFilter(ServletRequest var1, ServletResponse var2, FilterChain var3) throws IOException, ServletException;

    void destroy();
}
```

​	过滤器组件示例：

- 权限验证过滤器（Token）
- 日志记录过滤器
- 数据处理过滤器（加密、压缩、格式转换）
- 缓存过滤器
- MIME-类型链过滤器

#### 2、过滤器生命周期

​	Servlet容器来实例化Servlet前，需要初始化所有Filter，调用它们的init方法，并获取web.xml中filterConfig配置信息（初始化参数）；每个过滤器有且只有一个实例，但其线程安全，可以被每个请求处理的线程调用

​	当指定多个过滤器时，Servlet容器会将它们按顺序组成一个过滤器链，调用第一个过滤器的doFilter方法，传入ServletRequest、ServletResponse和FilterChain对象引用，如果满足条件，则调用FilterChain.doFilter方法，执行下一个过滤器的doFilter，循环往复，直到最后一个过滤器，然后访问目标web资源

​	在容器关闭前，会调用Filter的destroy方法，释放过滤器资源

#### 3、过滤器配置

在web.xml中，提供<filter>标签进行过滤器定义,它有三个子标签，来定义过滤器

- filter-name：过滤器名（保证唯一）
- filter-class：过滤器对应的类
- init-params：过滤器初始化参数

在web.xml中，提供<filter-mapping>标签进行过滤器的配置，它有三个子标签：

- filter-name：引入定义好的Filter

- url-pattern：指定映射url，进行拦截处理

- servlet-name：指定拦截的Servlet

  url-pattern、servlet-name可以同时指定多个，并且servlet-name匹配对应的过滤器永远为最后一个（允许一个过滤器被多次匹配，同样也会按照顺序多次处理请求）

**在Servlet3.1中，提供@WebFilter注解简化Filter的配置**

#### 4、过滤器和RequestDispatcher

​	在Servlet2.4后，过滤器配置支持识别请求是否被RequestDispatcher分发，从而进行更加细致化的匹配拦截

在web.xml的filter-mapping标签中，使用dispatcher子标签值进行约束：

1、REQUEST：

​	表示请求直接来自于客户端，也是其默认值

2、FORWARD：

​	表示请求被请求分派器的forward（）方法调用处理

3、INCLUDE：

​	表示请求被请求分派器的include（）方法调用处理

4、ERROR：

​	表示请求被错误页面机制处理

5、ASYNC：

​	表示请求被异步处理

dispatcher可以指定多个， 形成并集

### 9、会话Session

​	由于Http是一种无状态协议，无法将请求与指定客户端绑定，因此在服务器和浏览器之间就创建一种额外的机制，会话；**它是对Http协议的扩展，实现服务器对用户整个会话的追踪和记录**；在Servlet规范中，提供HttpSession接口进行实现

#### 1、会话跟踪机制原理

​	会话机制的实现，通过客户端的Cookie和服务端的Session共同完成

在Http响应头中，通过Set-Cookie来传输服务端传递给客户端的Cookie信息，Servlet容器默认创建一个Cookie：

```java
Set-Cookie: JSESSIONID=07CA5FA20FB2B28A3AF3A385A2D2693D; Path=/; HttpOnly
```

- JSEEIONID 为cookie的标准name，用于浏览器回传cookie时，服务端通过sessionId与其对应，进行session绑定
- path 表示当前域下，哪些请求可以访问传递cookie， /表示任何请求
- HttpOnly 表示cookie不能在浏览器中，被js脚本获取

在后续客户端的请求发送时，都会自动通过cookie请求头，来传递cookie中的name-value键值对；服务端通过解析name=JSEEIONID的cookie值，来绑定会话，实现会话跟踪

#### 2、Cookie属性和使用

​	除了cookie本身的K-V键值对、HttpOnly、Path外，还提供其他属性，来定义cookie在浏览器中的作用范围：

- int maxAge：告诉浏览器，该cookie的失效时间
- boolean secure：告诉浏览器，该cookie是否必须使用安全协议传输，如SSL

- String domain：告诉浏览器，允许访问该cookie的域名

​	Cookie可以指定多个，通过其KV来保存客户端信息，除了实现会话追踪外，还能进行不同Servlet之间的数据交换，Cookie包含如下特征：

- 不可跨越
- 使用Unicode编码传输，支持中文

#### 3、会话的生命周期

- 创建：

  ​	当开发者手动调用getSession（）、getSession（true），就会返回一个session（如果没有，就会创建），并保持在服务端，此时Servlet容器就会自动创建一个name=JSEEIONID的cookie，返回sessionId，从而实现会话追踪

- 销毁：

  ​	在Http协议中，客户端并没有显示的终止信号，来提示服务端关闭会话，因此会话超时时终止会话的唯一机制。Servlet容器定义了一个默认的会话超时时间（30分钟），通过`HttpSession`接口中的相关方法，可以获取和自定义：

  ```java
  //获取最后一次访问时间
  public long getLastAccessedTime();
  //设置会话超时时间
  public void setMaxInactiveInterval(int interval);
  //获取会话超时时间
  public int getMaxInactiveInterval();
  ```

  当超时时间=-1时，会话永不过期，直到Servlet容器关闭

#### 4、URL重写

​	由于cookie可能会被浏览器禁用，此时就无法实现session的绑定；因此Servlet提供另外一种方式，来完成SessionId的回传。但相对于cookie，需要开发者手动进行传递，并且会暴露会话标识

URL重写的实现方式：在URL后面添加 ;jseeionid=xxxx

```java
http://www.myserver.com/catalog/index.html;jsessionid=1234
```

#### 5、Session属性

​	`HttpSession`接口提供一系列方法，实现session会话内的数据交换

```java
//获取指定属性值
public Object getAttribute(String name);

//获取所有属性名
public Enumeration<String> getAttributeNames();

//设置属性
public void setAttribute(String name, Object value);

//删除属性
public void removeAttribute(String name);
```

​	Session本身通过Servlet容器来保证线程安全，因此多个请求线程访问同一个属性时，不需要额外处理；**但是对于属性中的对象，对其进行的操作，并不是线程安全的；因此对于该对象的成员变量修改，必须由开发者添加线程同步机制**

如：

```java
	UserInfoDTO user = (UserInfoDTO)session.getAttribute("user");
		synchronized (user){
			user.setUsername("YH");
		}	
```

### 分派请求

# Http协议

转发与重定向