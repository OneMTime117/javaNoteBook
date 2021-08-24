# HTML5和CSS3

## 1、web页面历史

### 1、W3C的建立

- 1989年，博纳斯-李 在欧洲粒子物理实验室写出了全世界第一个web页面
- 1990年，博纳斯-李  以HTML为基础，发明了最原始的web浏览器

- 1994年，博纳斯-李 牵头创建了万维网组织（W3C），提供了web页面的W3C开发规范（如HTML2规范），避免网页在不同浏览器中的不同；网景公司开发Navigator浏览器

### 2、第一次浏览器战争

- 1995年，网景公司推出JavaScript脚本语言，并嵌入到Navigator2.0浏览器中运行
- 1996年，微软发布JScript（是JavaScript的逆向工程的实现），并内置如IE3中，从此，第一次浏览器战争开启
- 1997年，欧洲计算机制造商协会（European Computer Manufacturers Association）推出ECMAScript规范，来标准化浏览器的脚本语言，JavaScript、JScript就是其规范的实现
- 1998年，ECMAScript2规范发布，并通过ISO成为国际标准ISO/IEC 16262
- 1999年，ECMAScript3规范发布，成为浏览器主流脚本语言规范，并且第一次浏览器战争结束，微软因为Window系统的捆绑销售，赢得生理，直到2002年，占有率达到96%

### 3、动态页面的发展

​		JavaScript诞生之除，用于实现对前端样式的动态变化，所有页面都还是纯静态的，为了让web页面更加生动，最早期使用CGI、applet等技术，来实现动态页面，但有很多缺点：

**CGI：需要使用C语言编写，效率低、依赖于服务器系统**

**applet：在浏览器端运行，并且不能处理复杂的java代码**

因此为了简化动态页面的开发，PHP、JSP、ASP.NET动态技术相继诞生：

- 1995年，拉斯马斯·勒德尔夫借鉴吸收了C、JAVA、Perl等流行计算机语言的特定，开发了易于程序员学习的PHP脚本语言，来实现开发者开始编写动态页面

- 1996年，微软随着IIS3.0（Internet Information Services,Window上的网络服务组件）发布，提供了ASP技术，服务端通过脚本语言，实现动态生成HTML页面

  2002年，微软推出ASP.NET，使用.NET语言提供更加方便的WEB应用构建框架，从而也替换了ASP技术

- 1999年,SUN公司创建JSP技术，以JAVA语言为技术，通过响应客户端请求，来动态生成HTML页面；同时后面推出了一系列MVC框架，使后端代码非常臃肿，前端开发重度的绑定在后端环境中，大大影响了前后端的开发效率

### 4、AJAX的出现，进入web2.0

- 1999年，W3C发布HTML4，同年微软推出实现web页面异步数据传输功能的浏览器控件ActiveX，被其他浏览器厂商模仿实现XMLHttpRequest相关API（用于在不重新加载页面的情况下，更新网页），这就是后来的AJAX（Asynchronous JavaScript and XML，通过javaScript执行异步网络请求）

- 2004，Goolge发布了Gmail web应用，内部使用了大量的AJAX技术，因此慢慢流行起来，让Web页面进入了一个新时代-SPA（Single Page Application 单页面应用）

  **此后，前后端分离，JavaScript通过Ajax接口进行数据交换，实现页面的动态变化**

### 5、第二次浏览器战争

​		微软在第一次浏览器大战中胜利后，迅速垄断浏览器市场。但IE并不遵循W3C后续标准，导致出现了前端兼容性问题

- 2004年，前网景公司所创建的Mozilla社区发布了Firefox浏览器，获取巨大成功，直到2008年，市场份额达到25%以上，而IE跌至65%以下

期间以Firefox和Opera为首的W3C阵营与IE开始对抗，进一步导致了浏览器的差异性问题越来越多；为了解决浏览器兼容性问题，Dojo、jQuery、YUI、ExtJS、MooTools前端框架相继出现，最终JQuery成为了前端的标配技术

## 2、W3C规范

​		在W3C规范中，将一个页面分为三个部分组成：

1、页面结构：通过**HTML语言**描述页面的结构

2、内容表现：通过**CSS语言**控制页面元素的样式

3、行为：通过**JavaScript语言**响应用户操作	

## 3、HTML5

### 1、HTML5简介

​		HTML（Hypertext Markup Language），超文本标记语言，使用标签的形式来标识页面中的不同组成部分，从而交给浏览器额外处理；超文本表示超链接，可以实现一个页面到另外一个页面的跳转

​		2014年10月，W3C发布了HTML5最新修订版，基于1999年发布的上个版本HTML4.0.1，时隔15年，html技术迎来了全面的更新：

**新特性：**

1、绘画canvas元素：基于JavaScript进行图像绘制

2、多媒体 video、audio元素：实现页面中的视频、音频播放

3、更好的支持本地离线存储

4、新的表单元素

5、新的特殊内容元素

#### 1.2、HTML5兼容性

​		现代浏览器都支持HTML5，但由于Windows系统版本原因，对于Windows XP等老系统内置还是使用的IE9以下版本（IE6-8)；而HTML5只支持IE9及以上，因此对于HTML5所有新元素，需要进行额外IE兼容性处理

​		html5shiv.js，用于解决IE6-8中不支持HTML5新元素问题

### 2、基本结构元素

HTML5页面，由一系列结构标签元素组成，而HTML元素可以进行嵌套和设置HTML属性来提供附加信息

标签书写注意：

- 标签分为单体标签（空内容元素）或者成对标签，W3C规范推荐单体标签需要闭合，如<br />,但浏览器支持不闭合的单体标签；成对标签默认就是闭合的
- 标签不区分大小写，但是W3C推荐使用小写
- 标签属性不区分大小写，但是W3C推荐使用小写

HTML基本页面结构：

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>这是我的学习网站</title>
</head>
<body>
    
</body>
</html>
```

**相关标签说明：**

#### 1、!DOCTYPE html

​		必须在HTML5文档第一行，声明当前文档为HTML文件

#### 2、html

​		定义整个HTML文档，是所有HTML5元素的父元素

#### 3、head

​		定于HTML页面头部，是所有HTML5头部元素父元素，头部元素有：

```html
<title>文档标题</title>
```

##### 1、title

​		定义HTML页面标题（浏览器工具栏标题、收藏夹标题、搜索引擎结果页面标题）

##### 2、base

​		定义HTML页面的基本URL，用于对HTML元素中的相对URL进行解析；默认使用当前HTML页面的URL

##### 3、link

​		在HTML页面中引入外部资源，通常用于引入css样式

```html
<link rel="stylesheet" type="text/css" href="mystyle.css">
```

| 属性 | 作用                         |
| ---- | ---------------------------- |
| rel  | 链接文档和当前HTML文档的关系 |
| type | 链接文档的MIME类型           |
| href | 超链接地址                   |

##### 4、style

​		定义HTML页面内部css样式

```html
<style type="text/css">
	body {background-color:yellow}
	p {color:blue}
</style>
```

##### 5、meta

​		定义HTML页面元数据，用于被浏览器解析，进行一系列配置，常用属性有name、content、charset、http-equiv

- name、content用来进行元数据K-V定义,如描述(description)，关键词(keywords)

```html
<mate name="keywords" content="htmls,前端">
```

- charset用于指定网页使用的字符集
- http-equiv搭配content属性，定义网页访问时的http响应头，或模拟一个http响应头

##### 6、script

​		编写浏览器脚本或加载浏览器脚本文件，如JavaScript

```html
<!-- 编写JS代码 -->
<script>
	document.write("Hello World")
</script>
<!-- 加载JS文件 -->
<script type="text/javascript" src="//test.js"></script>
```

#### 4、body

​		定义HTML页面身体，是所有HTML内容元素的父元素

### 3、Body内容元素

#### 1、文档类元素

##### 1、h1-h6

​		定义HTML标题，从大到小，为标题1-标题6

注意：

- 浏览器会自动对标题上下添加空行
- 标题字体会自动加粗，但注意标题的作用是让用户快速浏览页面，并为搜索引擎提供内容索引，并不是为了加粗、加大字体

##### 2、hr

​		定义HTML水平线，一般用于分割文档内容

##### 3、p

​		定义HTML段落

注意：

- 浏览器会自动对段落上下添加空行
- p为块级元素，但只能放文字、图片和表单，不能放标题、列表、表格和段落

##### 4、br

​		定义HTML换行，一般搭配p标签一起使用

##### 5、文本格式化元素

| 元素名 | 描述     | 样式                  |
| ------ | -------- | --------------------- |
| b      | 加粗文本 | <b>你好</b>           |
| i      | 斜体字   | <i>你好</i>           |
| em     | 着重文本 | <em>你好</em>         |
| ins    | 插入字   | <ins>你好</ins>       |
| del    | 删除字   | <del>你好</del>       |
| small  | 小号字   | <small>你好</small>   |
| strong | 加重字   | <strong>你好</strong> |

##### 6、计算机输出元素

| 元素名 | 描述                     | 样式              |
| ------ | ------------------------ | ----------------- |
| code   | 定义文本为计算机代码     | <code>你好</code> |
| kbd    | 定义文本为键盘码         | <code>你好</code> |
| samp   | 定义文本为计算机代码样本 | <code>你好</code> |
| var    | 定义文本为变量           | <code>你好</code> |
| pre    | 定义预格式文本           | <code>你好</code> |

可以看出，这些元素内容样式一致，但对于浏览器来说，它们的含义不同，这样就可以通过自定义样式将它们区分开来，进行特殊处理

##### 7、引文、引用和定义元素

| 元素名     | 描述                 | 样式                          |
| ---------- | -------------------- | ----------------------------- |
| abbr       | 定义文本为缩写       | <abbr>你好</abbr>             |
| address    | 定义文本为地址       | <address>你好</address>       |
| bdo        | 定义文字方向         | <bdo>你好</bdo>               |
| blockquote | 定义文本为长的引用   | <blockquote>你好</blockquote> |
| q          | 定义文本为短的引用   | <q>你好</q>                   |
| cite       | 定义文本为引用、引证 | <cite>你好</cite>             |
| dfn        | 定义为项目           | <dfn>你好</dfn>               |

#### 2、超链接 a

​		HTML超链接，可以实现一个HTML文档跳转到另外一个新的HTML文档，通过a标签来实现

默认情况下：

- 未访问过的超链接为蓝色字体、带下划线
- 访问过的超链接为紫色字体、带下划线
- 点击超链接时，为红色字体、带下划线

a标签的使用：

```html
<!-- 超链接 --> 
<a href="http://www.w3school.com.cn/index.html" target="_blank">这是一个a标签</a>

<!-- 引用锚点 -->
<a href="#hhh">跳转到hhh</a>
<a name="hhh">hhh1</a>
<h1 id="hhh">hhh2</h1>
```

注意：

- a标签和其他内容标签都可以来定义锚点，通过id/name属性来声明锚点名（**推荐使用name，根据规范**）

- a标签使用#引用描点，没有匹配值时，则默认定位到HTML页面顶部，因此一般直接使用#，用于定位到HTML页面顶部

a标签常用属性：

| 属性名 | 描述                     |
| ------ | ------------------------ |
| href   | 超文本引用/定位到锚点    |
| target | 引用的HTML文档在何处显示 |
| name   | 定义锚点                 |

target常用特殊值：

| 特殊值 | 作用                                 |
| ------ | ------------------------------------ |
| _blank | 新窗口打开超链接HTML文档             |
| _self  | 当前页面打开超链接HTML文档（默认值） |

#### 3、图像 img

​		HTML5中，图像通过img标签进行定义

img标签的使用：

```html
<img src="url" alt="图像无法加载" height=400 width=400>
```

img常用属性：

| 属性名        | 描述                                     |
| ------------- | ---------------------------------------- |
| src           | 定义图像源，即url地址                    |
| alt           | 当图像无法正确加载时，使用该默认文字描述 |
| height、width | 定义高度、宽度，单位默认为像素           |

注意：

- 一般情况下，都需要指定图像的高度和宽度，否则页面加载时，会使用图片默认大小，影响HTML页面布局
- 当高宽只修改其中之一时，会进行等比缩放

#### 4、表格 table

​		表格通过table标签定义，每个表格有若干行，由**tr标签**定义（table row）；每一行有若干个单元格，由**td标签**定义（table data）；单元格中的内容包括文本、图像、列表、表单、表格。

​		table标签的使用：

```html
    <table >
        <tr>
            <th>属性</th>
            <th>描述</th>
        </tr>
        <tr>
            <td>name</td>
            <td>姓名</td>
        </tr>
        <tr>
            <td>class</td>
            <td>类名</td>
        </tr>
    </table>
```

table元素下的其他子标签：

| 标签名  | 描述       |
| ------- | ---------- |
| th      | 表格的表头 |
| tr      | 表格的行   |
| td      | 表格单元   |
| caption | 表格标题   |

**跨行、跨列的单元格:**

在th、td使用rowspan来定义当前单元格所占行数；使用colspan定义当前单元格所占列数；

如果一个单元格占有两行，那么下一行的单元格将减少一个；同理一个单元格占有两列，那该行单元格会减少一列

#### 5、列表

​		HTML5中，列表分为无序列表、有序列表和自定义列表

- 无序列表ul

  ​		通过**ul标签**（unordered list）来定义无序列表，使用粗体黑点进行标记；**li子标签**（list item）定义列表项

  ```html
      <ul>
          <li>name</li>
          <li>sex</li>
      </ul>
  ```

- 有序列表ol

  ​		通过**ol标签**（ordered list）来定义有序列表，使用数组进行标记；同理使用**li子标签**定义列表项

  ```java
      <ol>
          <li>name</li>
          <li>sex</li>
      </ol>
  ```

- 自定义列表dl

  ​		通过**dl标签**（define list）来自定义列表，包含**dt子标签**定义列表项（define term）和**dd子标签**定义列表项描述（dfine describe）

  ```java
   <dl>
          <dt>name</dt>
          <dd>姓名</dd>
          <dt>sex</dt>
          <dd>性别</dd>
   </dl>
  ```

**列表之间是可以相互嵌套的，而每一级的标识符也有所不同**

#### 6、音频 audio

​		通过audio标签，在H5中引入外部音频文件，默认情况下，所有音频的播放是不给用户控制的，因此一	般搭配controls属性使用

```html
<audio src="./source/audio.mp3" controls></audio>
```

​		1、audio常用属性：

- autoplay：

  ​		在网页打开时，可以自动播放（但大部分浏览器提示用户体验，并不会自动播放）

- loop：

  ​		音乐循环播放

- src：

  ​		引入单个音频文件

  2、audio存在浏览器兼容问题，因此出现audio无法生效，而通过audio标签中的文本内容，就可以进行相应提示

```html
<audio src="./source/audio.mp3" controls>当前音频文件无法加载，请使用其他浏览器</audio>
```

​		3、通过source子标签，引入多个音频文件,默认使用第一个，当浏览器无法使用该音频格式文件时，则使用第二个，这样能够很好的解决浏览器的音频格式不兼容问题

```java
<audio src="./source/audio.mp3" controls>
	当前音频文件无法加载，请使用其他浏览器
	<source src="./source/audio.mp3">	
	<source src="./source/audio.ogg">
</audio>
```

#### 7、视频 video

​		通过video标签，在H5中引入外部视频文件，默认情况下，所有视频的播放是不给用户控制的，因此一般搭配controls属性使用

​		和source一样，支持source、loop、autoplay属性，来实现对应功能

### 4、区块元素

#### 1、块级元素和内联元素

​		HTML中，大部分内容元素根据浏览器的显示方式，分类定义为**块级元素**或**内联元素**

- 块级元素 block level：

  1、浏览器在显示时，会在内容前后另起新行，独占一行

  2、能设置宽高，当不设置宽度时，默认为父级容器的100%

  3、能够包含内联元素和块级元素

- 内联（行内）元素 inline ：

  1、浏览器在显示时，不会在内容前后另起新行

  2、不能设置宽高，默认宽度为内容宽度

  3、元素内部只能定义文本内容，不能进行元素嵌套

  根据元素中的内容定义，我们可以把元素分为文本元素和容器元素：

  | 文本元素        | 容器元素   |
  | --------------- | ---------- |
  | hr、p、br       | h1-h6      |
  | a               | table、tr  |
  | img             | ul、ol、dl |
  | th、td、caption |            |
  | li、dt、dd      |            |
  | 文本格式化元素  |            |

  **根据这些分类，所有的容器元素都为块级元素；所有的文本元素，除去p以外，都是内联元素，而p属于块级元素**
  
  对于引用元素，如果单独占一行时，则为块级元素，如长引用blockquote

#### 2、div和span

​		div元素，为块级元素，用于作为组合其他HTML元素的容器，它本身没有特殊含义，一般用于定义HTML文档布局，通过CSS使用设置其内容块样式

​		span元素，为内联元素，用于作为文本的容器，它本身没有特殊含义，一般用于搭配CSS定义其文本样式

- 除这两种之外，HTML5额外来提供带有语义功能的容器元素，但由于存在浏览器兼容问题，不怎么被使用：

  | 元素    | 语义         |
  | ------- | ------------ |
  | header  | 头部         |
  | nav     | 导航         |
  | section | 独立区域     |
  | aside   | 侧边栏       |
  | article | 独立文章     |
  | footer  | 网页底部     |
  | main    | 表示网页主体 |

  



### 5、表单form

​		表单是一个包含表单元素的区域，允许用于在其中使用文本域、下拉列表、单选框、复选框等元素

- 输入元素input

  ​		用于接受用户输入，通过属性type，支持多种类型，name作为K，value作为V，进行数据保存，常用如下：

  | type属性值 | 描述                                                         |
  | ---------- | ------------------------------------------------------------ |
  | text       | 文本域，size定义输入框大小（默认20），value定义文本域中的默认值 |
  | password   | 密码字段，默认隐藏用户输入，使用星号或圆点代替，size定义输入框大小（默认20），value定义文本域中的默认值 |
  | radio      | 单选按钮，内部使用value来指定该按钮锁表示的值；通过checked属性来进行默认勾选（出现多个name相同的按钮同时指定，则默认勾选最后一个） |
  | checkbox   | 多选框，内部使用value来指定选项表示的值；通过checked属性来进行默认勾选 |
  | submit     | 提交按钮，通过value指定提交按钮的文本；点击后会将整个表单收集的K-V内容，转移到form表单中action属性指定的地方 |
  | button     | 按钮，通过value指定提交按钮的文本                            |

  ```html
  		name: <input type="text" name="name" value="yh"><br>
          password:<input type="password" name="password" value="123456"><br>
          <input type="radio" name="sex" value="男" checked>男<br>
          <input type="radio" name="sex" value="女" >女<br>
          <input type="checkbox" name="hobby" vlaue="game">游戏<br>
          <input type="checkbox" name="hobby" vlaue="food" checked >食物<br>
  ```

- 下拉列表元素select

  ​		select元素定义了一个下拉列表，并通过option元素来定义下拉列表中的选项，并且内容使用value来指定该选项所表示的值，同时可以使用selected来定义下拉列表的默认值（不指定，则默认为第一个option）

  ```HTML
  <select name="cars">
        <option value="BMW"> BMW </option>
        <option value="audi" selected > audi </option>
  </select>
  ```

- 表单元素form

  ​		form元素定义表单，并指定表单中数据的提交方式：
  
  | 属性    | 描述                                                         |
  | ------- | ------------------------------------------------------------ |
  | action  | 表单提交地址                                                 |
  | enctype | 表单数据的编码格式（只适用Post请求）：<br />application/x-www-form-urlencoded<br/>multipart/form-data<br/>text/plain |
  | method  | 表单数据发送方式（get请求只支持application/x-www-form-urlencoded） |
  
  ```html
  <form action="url" method="get" enctype="application/x-www-form-urlencoded" >
    First name: <input type="text" name="fname"><br>
    Last name: <input type="text" name="lname"><br>
    <input type="submit" value="提交">
  </form>
  ```

### 6、内联框架元素

​		H5通过内联框架元素iframe来实现，在当前页面中，引入另外一个H5页面

```html
<iframe src="url" width=800 height=600></iframe>
```

​		内联框架元素为行内元素，基本不会使用，因为内联的网页不利于被搜索引擎搜索到；并且它在不同浏览器中的兼容性不太一样

### 7、HTML注意

#### 1、HTML5实体（转义字符）

​		在HTML中，有些特殊符号不能直接书写，如连续的空格、大于号、小于号

- 语法：

  **&实体名；**

- 常用实体：

| 实体名 | 含义                                   |
| ------ | -------------------------------------- |
| nbsp   | 空格（Non-breaking Space不间断的空格） |
| gt     | 大于号（great than）                   |
| lt     | 小于号（less than）                    |
| copy   | 版权符号                               |

#### 2、HTML5中的资源引用

​		当静态资源在同一个目录下时，可以通过相对路径进行引用，而相对路径的表示方式有两种：

- ./：表示当前文件所在目录，其可以省略
- ../：表示当前文件所在目录的上一级，可以重复使用，来表示多级上级目录

#### 3、浏览器对于元素错误嵌套的自动优化

- 当两个元素无法进行嵌套时，浏览器并不会保存，而是在解析时，会将外部元素看做为未结束或未开始，自动补全，从而取消其嵌套关系

```html
<p><h1>哈哈</h1></p>
会被优化为：
<p></p>
<h1>哈哈</h1>
<p></p>
```

- 当一个元素放在它父类元素外部时，浏览器会将其转移到内部，并按声明顺序定义其位置

```html
<body></body>
<h1>name</h1>
会优化为：
<body>
    <h1>name</h1>
</body>
```

4、HTML元素可以设置一个id属性，但必须保证同一个页面中，不出现重复的id，重复则按顺序第一个优先生效

## 4、CSS3

### 1、基本概念

​		CSS（Cascading Style Sheets）,层叠样式表，用于定义如何显示HTML元素;CSS3是最新的CSS标准，将之前已经最新的规范，分为系列模块，通过各个模块进行更新，因此理论上CSS3并不是第三个版本，而是所有默认最新版本合集的统称，并向下兼容；从2001.5草案发布，至今一直在更新中

- CSS基本语法：

  CSS有选择器和多条声明组成，选择器为需要改变样式的HTML元素，而声明是一个键值对，来定义样式属性

  推荐书写方式：每行定义一个声明，可读性更强

  ```css
  h1
  {
      color:red;
      text-align:center;
  }
  ```


### 2、创建方式

​		css由三种创建方式：

- 外部样式表

  ​		当样式需要应用在多个HTML页面时，外部样式表是最理想的选择，通过link标签进行外部样式表的引入，使用一个单独的css后缀文件创建css

  ​		由于多个html页面可以引用通过外部样式表，因此利用浏览器静态资源缓存，可以提高网页加载速度

  ```html
  <link rel="stylesheet"  type="text/css" href="./text/myStyle.css">
  ```

- 内部样式表

  ​		在单个页面需要特殊样式时，可以在head元素下的style子元素中，定义创建css

  ```html
  <head>
      <style>
      	.h1
          {
              color:red;
          }
      </style>
  </head>
  ```

- 内联样式

  ​		css支持直接在某个HTML元素中，使用style属性设置样式；但这种方式会导致样式与HTML元素混杂在一起，不能实现代码重用，因此**不推荐使用**

  ```css
  <h1 style="color: red;">
  	你是谁
  </h1>
  ```

多重样式的优先级：

​		CSS允许同时多重方式定义样式，如在外部样式表和内部样式表中同时定义某个class值得样式

一般情况下，三种方式的优先级如下：

- 内联样式表>内部样式表=外部样式表>元素浏览器默认样式

- 内部样式表和外部样式表的顺序则根据link和style元素定义顺序决定，因此为了防止这种歧义的发生，推荐将所有link的声明放在style之前，保证内部样式表>外部样式表

### 3、常用基本样式

#### 1、背景 background

用于设置元素文本的背景，有如下属性

| 属性名                | 作用                                                         | 默认值                   |
| --------------------- | ------------------------------------------------------------ | ------------------------ |
| background-color      | 设置背景颜色,可以使用color_name(颜色名称，如red)、hex_number（十六进制颜色值，如#ff0000）和rgb_number(rgb代码颜色值，如rgb（255,0,0）) | transparent，透明        |
| background-image      | 设置背景图像，通过url（'URL'）引入                           | nono，没有背景图像       |
| background-repeat     | 设置背景图像平铺方式；repeat-x水平方向重复，repeat-y垂直方向重复，no-repeat只显示一次 | repeat，水平垂直方向重复 |
| background-position   | 设置背景图像初始位置，定义x，y两个方向，可以使用百分比、像素和关键词（top、bottom、left、right、center）；如果只指定一个关键词，那第二个默认为center | 0% 0%                    |
| background-attachment | 设置背景图像是否固定，或者随着页面的其余部分滚动；scroll（随着页面滚动），fixed（不随页面滚动），local（随元素内容滚动）， | scroll                   |
| background-size       | 设置背景图像大小，可以使用百分比和像素                       | 100% 100%                |
| background-origin     | 设置背景图像存放在元素背景的相对位置区域，基于盒子模型，分为border-box（外边框）、centent-box（内容边框）、padding-box（内边框） | border-box               |
| background-clip       | 基于盒子模型，对整个背景进行裁剪，指定背景填充区域，分为border-box（包含边框）、centent-box（只有内容区域）、padding-box（不包含边框） | border-box               |

CSS3新属性

- background-image	在原有基础上，通过逗号分割，可以设置多张背景图片，用于平铺
- background-size      之前图片只能使用原有大小，在CSS3中则使用该属性定义图片大小

- background-origin     基于盒子模型，实现根据细腻化的**背景图像的相对位置**
- background-clip          基于盒子模型，实现更加细腻化的**背景裁剪填充**

注意：

**1、大部分background-xx属性值都可以设置inherit，用于继承父类元素对应样式**

**2、background属性可以实现复合属性的简写，但必须按一定顺序声明空格分割，并且可以省略跳过**

background顺序为：

color > image > repeat > attachment >position

```css
div.div1
        {
            border: 10px dotted black;
            padding: 35px;
            background: yellow url("source/TestingGame.png") repeat-x scroll 100px 100px;
        }
```

#### 2、文本属性

- color 文本颜色

  ​		同background-color一样，有三种定义方式，十六进制值、RGB值、颜色名；默认为black（黑色）

- text-align 文本对齐方式

  ​		center居中，left左对齐，right右对齐，justify每行宽度相等，左右边距对齐；默认为left（左对齐）

- text-decoration  文本修饰（主要用于添加删除线）	

  ​		overline上划线，line-through中间线（删除线），underline下划线；默认为none

- text-transform  文本大小写转换

  ​		uppercase全大写，lowercase全小写，capitalize每个单词首字母大写

- text-indent  文本首行缩进

  ​		指定缩进的大小，默认为0

- white-space   设置元素中空白的处理方式

  在进行文本内容书写时，对于空白非空格内容，可以进行额外的处理；但一般情况下，这样会影响我们元素内容的书写方式，因此只会使用normal和nowrap，用于是否限制内容一行展示

  | 属性值   | 描述                                                         |
  | -------- | ------------------------------------------------------------ |
  | normal   | 空白会被浏览器忽略，可以正常不使用< br>换行                  |
  | pre      | 空白会被浏览器保留，即在HTML标签中，每一行内容在截止前的空白，都会在HTML页面的体现 |
  | nowrap   | 空白被忽略，文本不会换行，直到使用< br>元素                  |
  | pre-wrap | 保留空白，且能正常不使用< br>换行                            |
  | pre-line | 空白被忽略，但能正常不使用< br>换行                          |
  |          |                                                              |

- 文字间距：

  | 属性名         | 描述                                                       |
  | -------------- | ---------------------------------------------------------- |
  | letter-spacing | 字符间距                                                   |
  | line-height    | 行高                                                       |
  | word-spacing   | 字间距（不常用，但可以对所有子元素起作用，可以设置为负数） |

  letter-spacing和word-spacing的区别：

  - letter-spacing：增加字符间的空白，默认值normal为0，即字符默认间距，并且无法设置为负数
  - word-spacing：增加或减少单词间的空白，默认值normal为0，即单词的默认间距，可以将其设置为负数，来减少间距

  因此注意，word-spacing是用于控制单词间的空白，因此如果两个字符间有空格，则就会使当前的空格变大或变小

- 文本方向

  | 属性名       | 描述                                                         |
  | ------------ | ------------------------------------------------------------ |
  | direction    | 文本方向，默认为ltr（left to right从左往右），trl（从右往左） |
  | unicode-bidi | 允许文本是否被重写，以支持不同语言对于direction的使用，当修改为trl时，就还需unicode-bidi属性值为bidi-override，才能实现文本重新书写和排序，否则文本只是会向右对其 |

  ```css
       direction: rtl;
       unicode-bidi: bidi-override;
  ```

**CSS3新增文本属性**

- text-shadow  文本阴影

​		指定文本水平阴影偏移量（必填）、垂直阴影偏移量（必填）、模糊距离（默认0）和阴影颜色（默认文本颜色）

```CSS
text-shadow: 2px 2px  black;
```

- text-overflow	文本溢出处理

  指定文本溢出当前元素大小时，处理方式：

  - clip，默认值，对文本进行修剪
  - ellipsis，使用省略号代替修剪的文本
  - 字符串，使用自定义字符串来代替修剪的文本（慎用，只能在火狐上生效）

```css
        div.div2
        {
            border: red solid 2px;
            width: 300px;
            height: 20px;
            text-overflow: '111';
            white-space:nowrap; 
            overflow:hidden; 
        }
```

text-overflow必须搭配overflow：nowrap和white-space：hidden两个属性使用，前者定义文本不能超过块级元素边框范围，后者定义文字必须在一行，否则及时块级元素定义了高度，也还是会继续展示，只是由于元素边框的大小限制，而被隐藏

- word-break 	单词换行规则（CJK中日韩除外）
  - normal		浏览器默认换行规则
  - break-all        允许单词内换行
  - keep-all         不允许在单词内换行（只能在空格和字符连接处）

- word-wrap     用于处理长单词的换行

  - normal		浏览器默认（只能在断字处换行）
  - break-word        可以在长单词内换行

  和word-break有所不同，前者是定义换行规则，当我们不允许单词内换行时，在单词过长大于元素宽度时，就会出现内容溢出的情况，因此避免这样的情况产生；但我们使用word-break不允许单词内换行时，一般都需要添加word-wrap：break-word，避免长单词溢出

#### 3、字体 Fonts

- font-family		字体系列

  ​		可以设置多个，根据顺序作为备选；当字体系列名超过一个单词时，必须使用引号

  ```java
  p
  {
  	font-family: 'Courier New', Courier, monospace;
  }
  ```

  ​		一般情况下，编译器代码提示，会将多个字体作为一组进行提示，说明这些字体系列是相互互补来完成所有字符的展示。

  ​		**字体被分为通用字体和具体名称的特殊字体，每个浏览器都提供有其本身的字体设置**

- font-style        字体样式

  ​		有三个属性值：

  - normal（默认值，正常）
  - italic（斜体）
  - oblique（倾斜体），模仿斜体，当字体库不支持斜体字时，使用

- font-size        字体大小

  ​		根据使用的单位，可以将字体分为绝对大小和相对大小，如px为指定像素大小；em为浏览器默认大小比例，1em则为1：1的文字大小，一般为16px；

- font-weight        字体粗细

  ​		字符粗细是一个笼统描述，基于字体本身决定，它定义了100-900的9个区间，其中有两个固定值和两个相对值：

  - normal（默认值，标准），为400
  - bold（粗体），为700
  - bolder （更粗），是相对于父类字体的下一个更粗区间
  - lighter （更细），是相对于父类字体的上一个较细区间

  注意：并不是所有字体都有9个区间，如果没有该区间，则区上一个较细的区间

- font-variant         以字体大小来控制单词的大小写展示
  - normal（默认值。正常）
  - small-caps，让小写单词使用的较小的字体大小

- font        和background一样，简写属性，用于定于字体所有属性，顺序如下：

  font-style > font-variant > font-weight >font-size/line-height > font-family

  其中font-size和font-family必须手动指定

  ```css
  p.ex1
  {
      font:15px arial,sans-serif;
  }
   
  p.ex2
  {
      font:italic bold 12px/30px Georgia, serif;
  }
  
  ```

### 4、盒子模型和样式

​		所有HTML元素可以作为一个盒子，而CSS就可以对这个盒子属性进行操作，CSS盒子模型（box model）包括：

![](.\图片\css盒子模型.jpg)

- margin 外边距 元素边框外的区域，透明，默认为0
- border 边框
- padding  边框和内容之间的区域
- content  内容，其大小通过width和height属性定义

#### 1、盒子模型的兼容性

​		在早期，IE浏览器并不支持W3C规范。在IE5、6中，width和height属性值为content+padding+border；

#### 2、边框  border

- border-style    边框样式

  边框样式属性值有：

  | 属性值 | 作用                           |
  | ------ | ------------------------------ |
  | nono   | 无边框                         |
  | dotted | 点线边框                       |
  | dashed | 虚线边框                       |
  | solid  | 实线边框                       |
  | double | 双边框，两边框距离就是边框宽度 |
  | groove | 3D沟槽边框（H5）               |
  | ridge  | 3D脊边框（H5）                 |
  | inset  | 3D嵌入边框（H5）               |
  | outset | 3D突出边框（H5）               |

- border-width     边框宽度

  可以使用CSS长度值，还提供3个关键字，它们具体的长度由浏览器决定

  | 属性值 | 作用               |
  | ------ | ------------------ |
  | thick  | 粗边框             |
  | medium | 中等边框（默认值） |
  | thin   | 细边框             |

- border-color    边框颜色

  可以设置为transparent透明

- border     简写属性，其顺序为：

  border-width > border-style > border-color

  其中border-style是必须的

边框属性可以细分为上下左右四种类型，属性名添加left、right、top、bottom，如：

border-right 、border-top-color、border-bottom-width等

也可以直接在一个没有方向的属性中（**border除外**）同时设置4个属性值，使用逗号分割，上、右、下、左顺时针依次指定:

border-color: red ,green ,black, white,其中包含两种特殊情况：

- 只有2个属性值时，分别声明上、左右两种
- 只有3个属性值是，分别声明上、左右、下三种

**CSS3新增样式：**

- border-radius   圆角，指定圆角半径值
- box-shadow    边框阴影，和text-shadow一致
- border-image   设置边框内图像

#### 3、轮廓 outline

​		用于对边框外部区域添加样式，但不受到元素大小的限制，完全依赖于边框；因此如果两个元素相邻，并且不指定外边距时，后一个元素的轮廓会影响到前一个元素样式

​		outline相关属性的使用和border一致，包括如下：

| 属性          | 描述                          |
| ------------- | ----------------------------- |
| outline-style | 轮廓样式                      |
| outline-color | 轮廓颜色                      |
| outline-width | 轮廓宽度                      |
| outline       | 轮廓简写属性（style是必须的） |

#### 4、外边距 margin

​		外边距margin用于定义元素之间的区域（即两个元素边框间的距离），没有颜色是完全透明的，并且可以单独改变上、下、左、右的外边距，和border方式一致

​		**外边距合并：**

当两个相邻元素同时指定margin时，在表现上，它们会合并外边距使用最大的那个，但实际上，相当于两个外边距区域重合，同时满足对外边距的需求

#### 5、内间距（填充）padding

​		内间距paading用于定于元素边框和元素内容之间的区域，并且该区域会收到元素背景样式的影响

​		padding同理，其相关属性使用和一致

### 5、选择器分类

​		当选择器为某个HTML元素时，则定义的样式会全局改变所有HTML元素，一般并不会这样使用；基本上会使用通过HTML元素的id、class属性，将CSS选择器分为：

#### 1、通用选择器

​		改变所有HTML元素样式

```css
*
{
  color:red;
}
```

#### 2、元素选择器

​		css的基本使用方式，指定某种HTML元素的样式

```css
  h1
{
      color:red;
}
```

#### 3、id选择器

​		在HTML中，元素的id是唯一的，因此CSS样式也只会改变对应的HTML元素，其前缀为#

```css
#hh
{
  color:red;
    text-align:center;
}
```

#### 4、class选择器

​		class选择器区别于id选择器，用于描述一组HTML元素的样式，因为一个class值可以在多个元素中使用，其前缀为.

```css
.hh
{
	color：red;
    text-align:center;
}
```

#### 5、复合选择器		

class选择器还可以指定某类元素中，指定class值的元素的样式，如class=hh的所有p元素

```css
p.hh
{
    color：red;
    text-align:center;
}
```

注意：

​		HTML元素中，属性id和class不用使用数字，部分浏览器无法识别

#### 6、选择器的分组和嵌套

- 选择器分组，即并集选择器，用于将具有相同样式的选择器合并

  ```css
  h1,h2,p
  {
      color:green;
  }
  ```

- 选择器嵌套，即交集选择器，用于对满足多种选择器匹配条件的HTML元素设置样式

  ```css
  .a.b.c
  {
       color:green;
  }
  ```

  复合选择器就属于交集选择器

  由于id选择器就决定了HTML元素的唯一性，因此一般不使用再交集选择器中

#### 7、关系选择器

​		HTML元素之间的关系有：

- 父元素

  ​		a是b的父元素，则a直接包含b

- 子元素

  ​		a是b的子元素，则b直接包含a

- 祖先元素

  ​		a是b的祖先元素，则a直接或间接包含b

- 后代元素

  ​		a是b的后代元素，则b直接或间接包含b

- 兄弟元素

  ​		a是b的兄弟元素，则a和b具有相同的父元素

关系选择器，就是通过元素之间的关系，来选择匹配对应元素：

- 子元素选择器

```css
div > p			  div元素下一级的所有p元素
```

- 后代元素选择器

```java
div p             div元素内的所有的p元素
```

- 兄弟选择器

```css
div + p             div元素同级别的下一个p元素
div ~ p				div元素所有同级别的p元素
```

#### 8、属性选择器

​		除了class、id属性外，HTML元素还可以添加其他属性，通过属性选择器，就可以选择带有额外属性的元素：

```css
div[title]    所有带有title属性的div元素
div[title=abc]    所有带有title属性、且属性值为abc的div元素
div[title^=abc]    所有带有title属性、且属性值以abc开头的div元素
div[title$=abc]    所有带有title属性、且属性值以abc结尾的div元素
div[title*=abc]    所有带有title属性、且属性值包含abc的div元素
```

### 6、伪类和伪元素

#### 1、伪类

​		html元素上的一系列不真实存在的属性，用于描述一个元素的特殊状态，搭配css使用，定义元素处于当前状态时的样式：

常用伪类：

- 获取指定位置的元素

| 伪类名             | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| ：first-child      | 第一个的子元素，且为指定类型                                 |
| ：last-child       | 最后一个子元素，且为指定类型                                 |
| ：nth-child（n）   | 第n个元素，且为指定类型，特殊值有：<br />n表示所有元素<br />2n（even）表示偶数位元素<br />2n+1（odd）表示奇数位元素 |
| ：first-of-type    | 作为子元素，父元素中的第一个该类型                           |
| ：last-of-type     | 作为子元素，父元素中的最后一个该类型                         |
| ：nth-of-type（n） | 作为子元素，父元素中的第n个该类型，特殊值参考nth-child（n）  |

**xxx-child和xxx-of-type的区别：**

前者是对所有子元素进行排序，获取指定位的元素；后者是对所有该类型的子元素排序，因此后者不会受到其他子元素类型的影响

- 

