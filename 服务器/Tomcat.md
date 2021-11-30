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

### 3、web.xml文件

​	web.xml文件为web应用的部署描述文件（Deployment Descriptor），用于交给Servlet容器进行相关配置和部署，常用包括：

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

​		响应对象,封装了服务器返回客户端的所有信息,在HTTP协议中,`将请求转为了 `HttpServletRequest`对象`，封装响应头和响应的消息体

​		Response是响应对象的实现类，其接口继承包括：HttpServletResponse》》ServletResponse

#### 1、缓冲

​		Servlet容器提供服务器缓冲功能，通过Response对象api实现

```java
//获取缓冲区大小
public int getBufferSize()  
//设置缓冲区大小    
public void setBufferSize(int size)
//刷新提交缓冲区数据    
public void flushBuffer()
//响应数据是否提交
public boolean isCommitted()
//清除缓冲区所有数据
public void reset()
//清除缓冲区所有数据,但不包括响应头
public void resetBuffer()
```

缓冲区大小设置,必须在ServletOutputStream/Writer写入数据前,否则会抛出IllegalStateException异常

如果缓冲区数据达到最大值,则会将数据输出到客户端,并认为响应已被提交;此时如果调用reset\resetBuffer方法,则会抛出IllegalStateException异常

#### 2、响应头

​		HttpServletResponse提供api进行HTTP响应头操作

```java
//设置响应头(如果响应头存在，则覆盖)
public void setHeader(String name, String value)
//添加响应头（如果响应头存在，则添加额外一个同名响应头）
public void addHeader(String name, String value)
```

​		相同响应头数据，会通过set保存value值，最后通过逗号分割展示

除了默认的String类型的value值外，还提供date、int类型数据的header值设置

#### 3、非阻塞IO

​	Servelt 容器在进行异步请求和升级处理时，支持非阻塞写；在进行http响应数据的写入时，ServletOutputStream提供一系列方法，来写入http响应数据

```java
boolean isReady();//判断数据是否可以被无阻塞写入

void setWriteListener(WriteListener listener)；//设置数据流写入监听器，进行方法回调
```

通过WriteListener `来为`ServletOutputStream`的数据读取提供一系列回调方法：

```java
onWritePossible()；//当数据可以被写入时，进行回调
 
onError(Throwable t)；//数据写入发生错误时，进行回调
```

#### 4、重定向和错误

​		HttpServletResponse提供api，实现重定向和错误响应

- sendRedirect

  ​		将设置了header和内容体的响应重定向到另外一个url

  url支持相对路径(在当前url的基础上拼接),但是必须要保证该url路径有效

- sendError

  ​		将设置了header和内容体以响应错误的方式返回客户端

#### 5、国际化

​		HttpServletResponse提供api，设置local和字符集

```java
//设置响应头-ContentType，告诉浏览器响应内容的类型（MEMI类型）和编码格式
//同时设置Servlet容器保存数据的编码格式 
public void setContentType(String type) 
//设置Servlet容器保存数据的编码格式 
public void setCharacterEncoding(String charset)
//设置locale
public void setLocale(Locale locale)
```

在没有调用setContentType、setCharacterEncoding方法时，设置locale可以改变字符集

#### 6、结束响应对象

​		在响应关闭时，缓冲区数据会立即将数据提交给客户端，如下情况时，响应会关闭：

- servlet的service方法执行完成（完成正常响应流程）
- 调用sendError、sendRedirect方法，执行完成
- AsyncContext.complete方法调用（完成异步处理）

#### 7、Response对象生命周期

​	每个Response对象都只能在Servlet的service方法，或过滤器的doFilter方法的作用域内有效；除非通过调用request.startAsync方法，实现请求异步处理；此时，Response对象一直有效，直到执行AsyncContext.complete方法

### 8、Filter

​		过滤器Filter为Java组件，允许在资源请求和响应过程中，修改header和负载；是一种代码重用技术，常用场景包括：

- 验证过滤器
- 日志记录
- 数据预处理（图像转化、数据压缩、加密、转化）
- 缓存过滤器

#### 1、生命周期

​		Servlet提供Filter接口，来定义创建过滤器，并在web.xml部署描述符中进行配置，过滤指定映射的Servlet

```java
//初始化方法
public default void init(FilterConfig filterConfig) throws ServletException {}

//过滤处理方法
public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

//销毁方法
public default void destroy() {}
```

​		Filter整个生命周期为：

- Servlet容器初始化后，创建Filter实例，调用init初始化方法

- 请求传入Servlet容器后，交给过滤器链处理
- 调用第一个Filter的doFilter方法，将请求、响应对象包装为ServletRequest、ServletResponse，对其进行相应处理
- 当执行FilterChain.doFilter()方法后，将请求交给下一个Filter，直到最后一个后，目标为最终的Web资源
- 然后按之前相反顺序向上跳出doFilter（）方法，最后返回给客户端响应

- Servlet容器关闭时，调用destroy销毁方法

**处理请求的serivce方法和过滤器链都允许在同一个线程中**



# Http协议