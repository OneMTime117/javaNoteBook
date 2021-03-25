# Idea操作指南

## 环境配置

**idea提供idea.properties文件，进行相关路径或参数的配置：**

**注意：路径需要使用正斜杠  /  **

- 个性化配置目录：idea.config.path（windows默认路径C:\Users\Administrator\AppData\Roaming\JetBrains\IntelliJIdea2020.3）
- 缓存和索引目录：idea.system.path（C:\Users\Administrator\AppData\Local\JetBrains\IntelliJIdea2020.3）

**idea64.exe.vmoptions文件用于修改JVM运行参数：(16G内存推荐设置)**

- -Xms512m
- -Xmx1500m
- -XX:ReservedCodeCacheSize=500m

## 缓存和索引

​	Idea提供缓存和索引来加快文件查询，从而加快各种查找、代码提示等操作的速度；但有时由于电脑死机等原因会发生错误，此时可以：

- File->Invalidate Caches/Restart       清除索引与缓存

  当然也可以直接手动删除缓存文件（idea.system.path配置的目录）

## 界面（view）

**通过view进行配置展示**

- Project窗口

  idea中，并没有工作空间这个概念，一个project项目就是要给windows窗口，其次以及就是Module模块，这种工程项目思想来源于微服务架构，因此一般情况下，即使要给项目也是通过构建Module来完成

  - 在project项目创建后，会自动生成.idea目录，当前project的配置文件目录

  - 创建Module模块后，会自动生成一个.iml文件，为当前Module的配置文件

  **这两个文件，会随着IDEA对项目的修改而发生变化，但一般情况下，不会进行版本控制**

- project structure窗口

  对整个project和module模块的进行配置和管理，常用包括：

  SDK、language level（用于指定编译时的语法检测范围，一般情况下，是和SDK对应）、对project和module模块进行管理和构建

## 编译（Build）

idea默认自动保存，但是并不会实时编译，需要进行手动编译：

- Build  ：只会编译修改的内容
- rebuild：则会重新编译整个项目

在build窗口，可以查看Build输出日志，并可以修改编译配置

## 版本控制（Git）

在Settings->Version Control进行配置，常用设置如下：

- 设置git的执行路径：一般情况下，idea安装后会自动找到git.exe路径
- commit 提交设置：
  - beforeCommit  >>> Optimize imports           提交前重新导包

- confirmation  确认设置
  - 当有新文件创建、文件被删除时，是否询问进行版本控制

## 实时代码模板（Live Template）

实时代码模板能够实现通过短小的前缀，直接自动生成整个相应代码，并可以通过获取上下文信息（类名、方法名等）来动态生成，并通过tab进行生成（可以使用自动提示，来选择相应模板enter确认）

Live Template编写：

**默认参数：**

- $END$   ，表示代码生成后，光标所在位置
- $SELECTION$，表示设置环绕实时代码模板，表示当前已经选取的内容

**模板生命周期：**

​	在模板内容下方，可以设置模板的生命周期，这样只有在指定语言/文件（java）、作用域（成员变量、方法内部...）中，才会自动提示或生成代码模板

​	以java为例，常用地方包含如下：

- declaration	成员变量（方法）定义区，即类里面，方法外面，**可以用于成员变量、方法和相关注解的定义**
- statement          方法内部的声明区，即在方法里面，可以用于**方法内部的代码生成**
- expression        表达式区，**在方法里面自动生成对某个变量后的赋值代码**

**配置:**

​	在模板内容左边可以设置**生成模板的确认快捷键、是否自动格式化代码、是否使用静态导包（如果为静态内部类，则直接导包为内部类，从而省略代码中外部类的书写）**

**变量参数和函数:**

​	每个模板可以自定义模板内容中，使用$$引入的变量，并且可以使用IDEA提供的内置函数，获取相关环境中的信息，或者提供一个自定义的默认值

## 文件代码模板（File and Code Templates）

​	在创建文件后，可以指定文件生成相应代码，idea已经默认提供了最基础的代码，比如class文件，会生产相应class声明和package

​	利用这个方式，我们可以定义文件创建时，需要自动生成的代码，并和Live Tmplates类似，使用一系列函数、变量来动态生成

## 后缀补全（Postfix Completion）

​	类似于实时代码模板，但却是通过一个变量.后缀key 来进行触发，并且更加方便快捷

比如if模板,可以直接使用b.if 来完成

## Debug调试

​	在Debugger配置中，设置传输类型：Shared memory（相对于socket更快

Debug常用快捷键：

| 快捷键   | 介绍                                                         |
| -------- | ------------------------------------------------------------ |
| F7       | 下一步，如果是方法，则进入方法体                             |
| F8       | 下一步，不进入方法体(不会进入JDK中的源代码)                  |
| F9       | 跳到下一个断点，如果没有则恢复程序运行                       |
| Shift+F7 | 当前行有多个方法时，可以进行选择进入那个方法内部（包括JDK源码） |
| Shift+F8 | 跳出当前方法，返回上一级代码调试                             |

**Debug界面按钮（上面除外）：**

- **Evaluate Expression表达式调试框（Alt + F8)**：用于对指定对象进行表达式计算，并得出相应结果

-  **Show Execution Point（Alt+F10）**：将光标跳转到当前执行的断点行
- **Force Step Into (Alt + Shift + F7)**：强制进入当前方法体（包括JDK源码）
- **Drop Frame** ：退回到断点所在处，并且回退之前所执行的下一级代码（相对于Shift+F8，可以有效的重新调试当前行方法，并且跳出多级方法的进入）
- **Mute breakpoints**：使所有断点失效
- **Run to Cursor (Alt + F9)**：将程序运行到光标所在处，不需要打断点
- **View Breakpoints (Ctrl + Shift + F8)**：查看所有断点

在View Breakpoints界面中，我们可以设置该断点生效的条件，即满足条件，程序才会停留在此断点处，通常使用在遍历较大集合的循环中，方便进入指定条件下的循环体（当进行一步步来到当前断点时，并不会直接使条件生效，因为断点的作用是让程序停到该点，而此时程序已经进入debug进步的调试方式，因此需要通过F9放行来让程序运行到改断点满足的条件处）

通过鼠标右键，可以直接编辑当前断点的满足条件

## Settings常用配置

1、代码补全提示不区分大小写（Editor.General.Code Completion）

2、代码检查等级（代码编辑器的右上角）

​	设置为All Problem，可以提示一些API调用的冗余

3、自动导包、自动去掉无用的包（Editor.General.Auto Import）

4、代码编辑区字体设置（Editor.Font）

5、控制台字体设置（Editor->Color scheme ->Console Font）

6、文件编码：全局编码、项目文件编码、properties文件编码（Editor->File Encodings）

## 快捷键

- 当前行/光标操作

  | 快捷键                  | 功能                                       |
  | ----------------------- | ------------------------------------------ |
  | Ctrl + Y                | 删除当前行（推荐使用Ctrl + X，行剪切）     |
  | Ctrl + X                | 剪切当前行                                 |
  | Ctrl + C                | 复制当前行                                 |
  | Ctrl + D                | 复制并在下方粘贴当前行                     |
  | Ctrl + Q（长按Ctrl）    | 显示当前光标所在变量 / 类名 / 方法名的文档 |
  | Ctrl + W                | 选中光标所在的单词/或整行                  |
  | Ctrl + Shift + W        | 取消选中的单词/或整行                      |
  | Ctrl + 左右方向键       | 移动光标到下/上个单词                      |
  | Ctrl + Alt + 左右方向键 | 移动光标到下/上个操作点（可以跨文件）      |
  | Ctrl + 上下方向键       | 类似于使用鼠标滚轮，光标位置不变           |
  | Ctrl + 括号             | 移动光标到当前代码所在的花括号开始/结束处  |

- 文件/文件内容操作

  | 快捷键            | 功能                                                         |
  | ----------------- | ------------------------------------------------------------ |
  | Ctrl + F          | 文件内容查找                                                 |
  | Ctrl + Shift + F  | 全局文件内容查找（可以指定模块）                             |
  | Ctrl + R          | 文件内容替换                                                 |
  | Ctrl + /          | 注释                                                         |
  | Ctrl + Shift + /  | 多行注释（内部新增行使，不会自动生成*，需要手动修改第一行为 /**） |
  | Ctrl + Z          | 撤销                                                         |
  | Ctrl + Shift + Z  | 取消撤销                                                     |
  | Ctrl + Shift + U  | 大小写轮换                                                   |
  | Alt + Enter       | 代码补全修复                                                 |
  | Ctrl + ALT + 左键 | 进入接口实现类（实现方法）                                   |
  | Ctrl + 左键       | 通过方法跳转到其调用地点                                     |
  | Ctrl + Alt + L    | 格式化代码                                                   |
  | Ctrl + Alt + O    | 优化导包（无法确认的，需要使用Alt + Enter进行代码修复）      |
  | Ctrl + Shift + J  | 自动将下一行合并到当前行末尾（可以用于去掉空白行，全选后执行，然后格式化） |
  | Ctrl + Alt + T    | 对选择内容进行环绕处理（try-catch、if、while。。。）         |

- 窗口操作

  | 快捷键           | 功能                              |
  | ---------------- | --------------------------------- |
  | Ctrl + E         | 显示最近打开的文件记录列表        |
  | Ctrl + N         | 通过**类名** 查找类文件           |
  | Ctrl + Shift + N | 通过路径查找文件                  |
  | Shift + 左键     | 关闭指定文件窗口                  |
  | Ctrl + H         | 查看类的继承关系（Hierarchy窗口） |
  | ALT + 7          | 查看类结构（Structure窗口）       |

## 常用代码的模板、补全：

- 打印

  使用后缀补全,对指定对象进行打印

  ```java
  //s.sout
  System.out.println(s);
  //s.souts
  System.out.println("s = " + s);
  //s.serr
  System.err.println(s);
  ```

- for循环

  使用后缀补全，对数组、集合、数字进行循环遍历

  ```java
  //s.for,foreach语法
  for (String s1 : s) {
  }
  
  //s.fori
  for (int i = 0; i < s.length; i++) {
  }
  
  //s.forr
  for (int i = s.length - 1; i >= 0; i--) {
  }
  
  //10.for
  for (int i = 0; i < 10; i++) {			
  }
  ```

- if

  使用后缀补全，对boolean表达式进行if判断

  ```java
  //Objects.equals("1","2").if
  if (Objects.equals("1","2")) {
  }
  
  //Objects.equals("1","2").not
  !Objects.equals("1","2")
  
  //Objects.equals("1","2").else
  if (Objects.equals("1", "2")) {
  }
  ```

  使用后缀补全，对对象进行是否为null的if判断

  ```java
  s.notnull
  if (s != null) {			
  }
  s.null
  if (s == null) {
  }
  ```

- while、switch

  使用后缀补全、对boolean表达式进行while语句判断

  ```java
  Iterator<String> iterator = listStr.iterator();
  //iterator.hashNext.while
  while (iterator.hasNext()) {
  }
  ```

  使用后缀补全、对基本数据类型、String、枚举进行switch处理

  ```java
  //s.switch
  switch (s) {
  }
  ```

- 数组获取stream对象

  ```java
  //s.stream
  Arrays.stream(s);
  ```

- xml文件中，生成
- javaDoc

​	**使用实时模板生成：**

​		cdoc：class doc

​		mdoc：method doc

- main方法

​	**使用实时模板生成：**

​		main：main方法

​		mains：springBoot的main方法

- XML中生成<![CDATA[ ]]>标签

  **使用实时模板生成：**

  ​	cd：在xml文件中生成<![CDATA[ ]]>标签



