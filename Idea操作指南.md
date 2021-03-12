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

- 设置git的执行路径：一般情况下，idea安装后回自动找到git.exe路径
- 



## 配置settings

- Appearance&Behavior（外观、行为）

1、界面外观主题：

Appearance（外观）

- Editor（编辑）

1、代码编辑区字体设置：
Editor -> Font

2、控制台字体设置：

Editor->Color scheme ->Console Font

3、文件编码：全局编码、项目文件编码、properties文件编码

Editor->File Encodings

4、







## 快捷键

编码必备：

| 快捷键                  | 功能                         |
| ----------------------- | ---------------------------- |
| Ctrl + F                | 查找                         |
| Ctrl + R                | 替换                         |
| Ctrl + Y                | 删除选中行                   |
| Ctrl + D                | 复制当前选中行，并插入到下面 |
| Ctrl + /                | 注释                         |
| Ctrl + Shift + /        | 代码块注释                   |
| Ctrl + Alt + L          | 格式化代码                   |
| Ctrl + Alt + O          | 重新导包                     |
| Ctrl + Alt + 左右方向键 | 回到上一个操作点             |
| ALT + Enter             | 代码补全                     |
| Ctrl + Shift + U        | 全大小写轮换                 |

类、方法信息查看：

| 快捷键            | 功能                                     |
| ----------------- | ---------------------------------------- |
| Ctrl + N          | 通过类名查找类                           |
| Ctrl + Shift + F  | 全局查找                                 |
| Ctrl + ALT + 左键 | 进入接口方法的实现方法（接口类的实现类） |
| Ctrl + H          | 查看类的继承关系（Hierarchy窗口）        |
| ALT + 7           | 查看类结构（Structure窗口）              |

