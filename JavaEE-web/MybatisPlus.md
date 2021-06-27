# MybatisPlus

MyBatis-Plus是myBatis的增强工具，在MyBaits的基础上只做增强不做改变，为简化开发、提高效率而生

## 1、特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

## 2、快速开始

mybatis-plus支持springboot的快速启动：

- 需要提供springboot父pom文件

```xml
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
    </dependency>
```

1、创建springboot项目，引入web模块、mybatis-plus模块（依赖关联mybatis、JDBC模块）、数据库驱动依赖

2、application.yml配置数据源

3、启动类配置Mapper扫描器

4、创建实体类

5、编写mapper类：

```java
public interface UserMapper extends BaseMapper<User> {

}
```

mapper类继承mubaits-plus中的BaseMapper<T>接口，从而直接提供该T实体类对应数据库表，单表的CRUD操作

## 3、mybatis配置

**mybatis配置基于sqlSessionFactory对象，而springBoot在进行mybatis-plus整合时，会自动使用MybatisSqlSessionFactoryBean对象进行sqlSessionFactory的Bean的创建：**

因此springboot在配置文件中，提供了如下常用配置：

| 配置                | 默认值                          | 作用                                                         |
| ------------------- | ------------------------------- | ------------------------------------------------------------ |
| configLocation      | null                            | 定义mybatis的配置文件位置                                    |
| mapperLocations     | ["classpath*:/mapper/**/*.xml"] | 配置mapper对于xml文件位置                                    |
| typeAliasesPackage  | null                            | 对指定包下的类，进行别名配置（方便xml编写，但需要保证所有类名没有重复） |
| typeHandlersPackage | null                            | 注册指定包下，所有的typeHandler（用于数据映射时的javaType-jdbcType的类型转换） |
| configuration       | null                            | 对于Mybatis原生属性进行配置                                  |

- configuration常用属性默认值：

  | 属性                     | 默认值  | 作用                                                         |
  | ------------------------ | ------- | ------------------------------------------------------------ |
  | mapUnderscoreToCamelCase | true    | 开启驼峰命名规则映射                                         |
  | aggressiveLazyLoading    | true    | 当懒加载对象中的任何懒属性被加载时，该对象全部加载           |
  | aggressiveLazyLoading    | PARTIAL | resultMap自动映射策略，PARTIAL为局部映射                     |
  | localCacheScope          | SESSION | 以session级别开启一级缓存（微服务项目时，需要关闭，修改为STATEMENT） |
  | cacheEnabled             | true    | 开启，但还需要在每个mapper中进行配置                         |
  | log-impl：               | null    | 指定日志实现类，进行sql日志打印：org.apache.ibatis.logging.stdout.StdOutImpl |

## 4、核心功能

### 1、CRUD方法

#### 1、mapper CRUD方法

​	mybatis-plus封装BaseMapper接口，提供如下方法：

- Insert：

  ```java
  // 插入一条记录
  int insert(T entity);
  ```

- Delete:

  ```java
  // 根据 wrapper 条件，删除记录
  int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
  // 删除（根据ID 批量删除）
  int deleteBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
  // 根据 ID 删除
  int deleteById(Serializable id);
  // 根据 columnMap 条件，删除记录
  int deleteByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
  ```

- Update:

  ```java
  // 根据 Wrapper 条件，更新entity记录
  int update(@Param(Constants.ENTITY) T entity, @Param(Constants.WRAPPER) Wrapper<T> updateWrapper);
  // 根据 ID 修改
  int updateById(@Param(Constants.ENTITY) T entity);
  ```

- Select:

  ```java
  // 根据 ID 查询
  T selectById(Serializable id);
  // 根据 Wrapper 条件，查询一条记录
  T selectOne(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  
  // 查询（根据ID 批量查询）
  List<T> selectBatchIds(@Param(Constants.COLLECTION) Collection<? extends Serializable> idList);
  // 根据 Wrapper 条件，查询全部记录
  List<T> selectList(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  // 查询（根据 columnMap 条件）
  List<T> selectByMap(@Param(Constants.COLUMN_MAP) Map<String, Object> columnMap);
  // 根据 Wrapper 条件，查询全部记录
  List<Map<String, Object>> selectMaps(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  // 根据 Wrapper 条件，查询全部记录。注意： 只返回第一个字段的值
  List<Object> selectObjs(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  
  // 根据 Wrapper 条件，查询全部记录（并翻页）
  IPage<T> selectPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  // 根据 Wrapper 条件，查询全部记录（并翻页）
  IPage<Map<String, Object>> selectMapsPage(IPage<T> page, @Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  // 根据 Wrapper 条件，查询总记录数
  Integer selectCount(@Param(Constants.WRAPPER) Wrapper<T> queryWrapper);
  ```

#### 2、service CRUD方法

​	**mybatisPlus默认实现的一套Service接口，内部通过mybatis底层（使用SqlHelper进行了封装）和BaseMapper方法完成**

service 方法为了和mapper方法进行区分，方法命名改为save（新增）、remove（删除）、get（单行查询）、list（多行查询）、page（分页查询）、count（总数查询）前缀开头

- Save：

  ```java
  // 插入一条记录（选择字段，策略插入）
  boolean save(T entity);
  // 插入（批量）
  boolean saveBatch(Collection<T> entityList);
  // 插入（批量）
  boolean saveBatch(Collection<T> entityList, int batchSize);
  ```

- SaveOrUpdate:

  ```java
  // TableId 注解存在更新记录，否插入一条记录
  boolean saveOrUpdate(T entity);
  // 先根据 Wrapper 条件尝试更新，sql无法执行，则使用saveOrUpdate(T)方法插入一条记录
  boolean saveOrUpdate(T entity, Wrapper<T> updateWrapper);
  // 批量修改插入
  boolean saveOrUpdateBatch(Collection<T> entityList);
  // 批量修改插入
  boolean saveOrUpdateBatch(Collection<T> entityList, int batchSize);
  ```

- Remove:

  ```java
  // 根据 Wrapper 条件，删除记录
  boolean remove(Wrapper<T> queryWrapper);
  // 根据 ID 删除
  boolean removeById(Serializable id);
  // 根据 columnMap 条件，删除记录
  boolean removeByMap(Map<String, Object> columnMap);
  // 删除（根据ID 批量删除）
  boolean removeByIds(Collection<? extends Serializable> idList);
  ```

- Get：

  ```java
  // 根据 ID 查询
  T getById(Serializable id);
  // 根据 Wrapper，查询一条记录
  T getOne(Wrapper<T> queryWrapper);
  // 根据 Wrapper，查询一条记录，指定是否抛出异常
  T getOne(Wrapper<T> queryWrapper, boolean throwEx);
  // 根据 Wrapper，查询一条记录
  Map<String, Object> getMap(Wrapper<T> queryWrapper);
  // 根据 Wrapper，查询一条记录
  <V> V getObj(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
  ```

- List：

  ```java
  // 查询所有
  List<T> list();
  // 查询列表
  List<T> list(Wrapper<T> queryWrapper);
  // 查询（根据ID 批量查询）
  Collection<T> listByIds(Collection<? extends Serializable> idList);
  // 查询（根据 columnMap 条件）
  Collection<T> listByMap(Map<String, Object> columnMap);
  // 查询所有列表
  List<Map<String, Object>> listMaps();
  // 查询列表
  List<Map<String, Object>> listMaps(Wrapper<T> queryWrapper);
  // 查询全部记录
  List<Object> listObjs();
  // 查询全部记录
  <V> List<V> listObjs(Function<? super Object, V> mapper);
  // 根据 Wrapper 条件，查询全部记录
  List<Object> listObjs(Wrapper<T> queryWrapper);
  // 根据 Wrapper 条件，查询全部记录
  <V> List<V> listObjs(Wrapper<T> queryWrapper, Function<? super Object, V> mapper);
  ```

- Page：

  ```java
  // 无条件分页查询
  IPage<T> page(IPage<T> page);
  // 条件分页查询
  IPage<T> page(IPage<T> page, Wrapper<T> queryWrapper);
  // 无条件分页查询
  IPage<Map<String, Object>> pageMaps(IPage<T> page);
  // 条件分页查询
  IPage<Map<String, Object>> pageMaps(IPage<T> page, Wrapper<T> queryWrapper);
  ```

- Count：

  ```java
  // 查询总记录数
  int count();
  // 根据 Wrapper 条件，查询总记录数
  int count(Wrapper<T> queryWrapper);
  ```

- Chain：链式查询

  ```java
  // 链式查询 普通
  QueryChainWrapper<T> query();
  // 链式查询 lambda 式
  LambdaQueryChainWrapper<T> lambdaQuery(); 
  
  // 示例：
  query().eq("column", value).one();
  lambdaQuery().eq(Entity::getId, value).list();
  ```

#### 3、注意细节：

- **entity指定更新字段时，不包含主键id（@TableId）**
- 需要保证主键唯一，避免使用复合主键，并且主键类型可序列化（实现Serializable接口）
- Get方法中，结果集如果是多个，则加上限制条件 wrapper.last("LIMIT 1")，随机取一条；对于getOne，则可以指定是否抛出异常，默认抛出异常
- 如果条件入参为空，则不会加入生成的sql中

### 2、条件构造器

mybatis-plus提供条件构造器，来简化生成sql的where条件，配合BaseMapper、IService接口方法使用

- 注意：sql拼接，是否存在sql注入风险

- **AbstractWrapper**

是所有条件构造器的父类，提供了如下条件构造方法：

| 方法        | 对应sql条件关键字       | 备注                     |
| ----------- | ----------------------- | ------------------------ |
| eq          | =                       | 等于                     |
| ne          | <>                      | 不等于                   |
| gt          | >                       | 大于                     |
| ge          | >=                      | 大于等于                 |
| lt          | <                       | 小于                     |
| le          | <=                      | 小于等于                 |
| between     | BETWEEN                 | 两者之间                 |
| notBetwen   | NOT BETWEEN             | 不在两者之间             |
| like        | LIKE  '%Value%'         | 模糊查询                 |
| notLike     | NOT  LIKE  '%Value%'    | 模糊查询不满足的         |
| likeLeft    | LIKE  '%Value'          | 左模糊查询               |
| likeRight   | LIKE   'Value%'         | 右模糊查询               |
| isNull      | IS    NULL              | 为null                   |
| isNotNull   | IS    NOT   NULL        | 不为null                 |
| in          | IN                      | 满足如下集合             |
| inSql       | IN （SQL语句）          | in 子查询sql拼接   |
| notInSql    | NOT    IN   (SQL语句)   | not in 子查询sql拼接 |
| groupBy     | GROUP BY 字段, ...      | 分组                     |
| orderByAsc  | ORDER BY 字段, ... ASC  | 升序排序                 |
| orderByDesc | ORDER BY 字段, ... DESC | 降序排序                 |
| having      | HAVING ( SQL语句 )    | 分组后的条件查询sql拼接  |
| or          | OR拼接/OR嵌套           | 无参则OR拼接前后两个条件，有参则嵌套参数对于条件 |
| and | AND嵌套 | 嵌套参数中的条件 |
| apply | sql拼接 | and开头，可以使用{index}，来进行动态参数绑定 |
| exists | EXISTS（SQL语句） | exists子查询sql拼接 |
| notExists | NOT  EXISTS（SQL语句） | not   exists 子查询sql拼接 |
| nested | 正常嵌套 | 嵌套参数中的条件，但括号外没有关键字 |
| last | sql拼接到最后 | 多次执行时，之后最后一次生效 |

- **QueryWrapper/LambdaQueryWrapper**

  继承AbstractWrapper，用于Mybatis-plus查询、删除类型方法的入参，可以设置需要查询的字段

  ```java
  //查询字段sql
  select(String... sqlSelect)
  // 定义需要查询的字段，如果predicate返回false，则该字段不进行查询
  select(Predicate<TableFieldInfo> predicate)
  select(Class<T> entityClass, Predicate<TableFieldInfo> predicate)
  ```

  - select（Predicate<TableFieldInfo> predicate)示例

  ```java
  LambdaQueryWrapper<SysUser> select = eq.select(SysUser.class,sysUser -> {
      //此时会过滤，字段名为username的字段
  	return !Objects.equals(sysUser.getColumn(),"username");
  });
  ```

- **UpdateWrapper/LambdaUpdateWrapper**

  继承AbstractWrapper，用于Mybatis-plus修改类型方法的入参，可以设置更新字段和值

  ```java
  //更新的K-V
  set(String column, Object val)
  set(boolean condition, String column, Object val)
  //更新sql 
  setSql(String sql)
  ```

### 3、分页

- 配置分页插件

  ```java
  	@Bean
  	public MybatisPlusInterceptor mybatisPlusInterceptor() {
  				//添加mybatisPlus分页插件，通过mapper层代码里的拦截器实现
  		MybatisPlusInterceptor mybatisPlusInterceptor = new MybatisPlusInterceptor();
  		PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor(DbType.MYSQL);
  		paginationInnerInterceptor.setMaxLimit(500l);//null,每页查询最大值,防止恶意传参，size非常大
  		paginationInnerInterceptor.setOverflow(true);//默认false,当前页大于最大页时，返回第一页数据
  		paginationInnerInterceptor.setOptimizeJoin(true);//默认true,leftJoin优化
  		mybatisPlusInterceptor.addInnerInterceptor(paginationInnerInterceptor);
  		return mybatisPlusInterceptor;
  	}
  ```

- 自定义分页mapper

  ```java
  IPage<UserInfoDTO> pageUserInfoDTO(Page<?> page, @Param("dto")QueryUserInfoDTO dto);
  ```

  注意：**要保证page参数在第一个，并且对于实体类型，需要添加@Param注解，使用前缀**

- page、Ipage类

  page继承Ipage为分页参数和返回值封装，提供分页所需的所有属性：

  | 属性                     | 默认值 | 作用                                                |
  | ------------------------ | ------ | --------------------------------------------------- |
  | List<T> records          |        | 数据                                                |
  | long total               |        | 总数                                                |
  | long size                | 10     | 大小                                                |
  | long current             | 1      | 当前页数                                            |
  | List<OrderItem> orders   |        | 排序字段和顺序                                      |
  | boolean optimizeCountSql | true   | count语句优化（去掉一些不必要的操作，排序、左连接） |
  | boolean isSearchCount    | true   | 是否进行总数查询（false则不返回total）              |
  | boolean hitCount         | false  | 记录是否命中mybatis缓存                             |
  | long  maxLimit           | null   | 单页分页条数限制                                    |

- 分页优化：

  **mybatis进行配置分页后，会自动添加JsqlParserCountOptimize分页优化类**

  分页查询分为两步，查询总数、查询真实分页数据；而分页优化则是针对于查询总数来实现，有如下特点：

  - 优化order by：
    - 如果sql中包含**group by**，则不进行优化
  - 优化left join：
    - 如果主表连接为一对多的查询，则需要手动关闭分页优化（mybatis-Plus无法识别，当前查询是否为一对多），否则会导致分页失效，**推荐使用page参数，来设置关闭optimizeCountSql**
    - left join 连接后，存在条件过滤，则不进行优化
  - 优化count(1)：将count（1）替换正常查询的select 字段，有效避免先查询所有数据，再统计数据条数
    - select字段中包含#{}、${}参数替换时，由于无法确定真实select和from所在位置，因此无法进行替换，则不进行优化
    - select 字段中包含distinct，会导致数据变少，因此不进行优化
    - select 字段中包含group by，这样会导致count（1）作为group by分组后的聚合函数，改变sql含义，不进行优化

### 4、主键生成器

​	数据库主键生成策略常用了四种

- 自增（需要数据库支持主键自动递增）
- Sequence序列（通过insert语句的触发，自动生成一个递增/递减的序列，作为主键，如Oracle）
- UUID
- 雪花算法

#### @IdFiled

​	通过对实体类Id属性配置@IdFiled注解，来定义主键在insert时的生成策略：

@IdFiled属性：

| 属性  | 作用                               |
| ----- | ---------------------------------- |
| value | 用于指定当前id所映射的数据库字段名 |
| type  | 主键类型，IdType枚举               |

idType枚举值：

| 值          | 作用                                                       |
| ----------- | ---------------------------------------------------------- |
| INPUT       | insert前自行set主键值（可以根据数据库序列策略，来进行set） |
| AUTO        | 数据库ID自增                                               |
| NONE        | 默认值，使用全局设置（全局默认为ASSIGN_ID）                |
| ASSIGN_ID   | 默认使用雪花算法分配id（IdentifierGenerator.nextId）       |
| ASSIGN_UUID | 默认使用UUID分配id（IdentifierGenerator.nextUUID）         |

#### 使用ASSIGN_ID、ASSIGN_UUID进行自定义主键

- 自定义IdentifierGenerator：

  ```java
  //自定义identifierGenerator,id生成器
  public class MybatisPlusIdGenerator implements IdentifierGenerator {
  	@Override
  	public Number nextId(Object entity) {
  		return null;
  	}
  
  	@Override
  	public String nextUUID(Object entity) {
  		return null;
  	}
  }
  
  ```

- 进行id生成器的Bean注入：

  ```java
  	//注入自定义的id生成器,一般情况下，使用默认的雪花算法和UUID就可以
  	@Bean
  	public IdentifierGenerator identifierGenerator(){
  		return new MybatisPlusIdGenerator();
  	}
  ```

#### 雪花算法和UUID

​	在分布式系统中，需要保证主键ID：

1、全局唯一，不能重复

2、趋势递增，在Mysql的InnoDB引擎中，使用聚合索引，因此需要通过有序性，来保证数据库插入性能，并且通过有序的ID，能够保证对数据进行排序或增量同步

3、包含时间戳，能够有效获取分布式id的生成时间

- UUID：

  UUID能够保证全局唯一性，但是无序，导致入库性能差

- 雪花算法：

  相对于UUID，雪花算法有效解决的其无序性缺点，能实现每秒生成26万个自增可排序的ID

  其长度为64bit的long类型，1bit表示符号位、41bit表示时间戳毫秒数、10bit表示工作机器id、12big序列号

  由于毫秒数在高位、自增序列在低位，因此能够保证id按照时间趋势递增

  缺点：由于雪花算法依赖于机器时间，在分布式系统中，不能够保证所有机器时间完全同步，因此可能会出现不是全局递增的情况

## 5、扩展功能

### 1、逻辑删除

​	mybait-plus可以通过指定一个字段来进行逻辑删除，但需要注意其限制：

1、只对自动注入的sql生效，即使用mybatis-plus提供的API方法

2、CRUD的实现方式：

- 插入：不进行操作

- 查询：追加where条件过滤已删除数据，并且使用wrapper/entity作为参数条件时，会忽略该逻辑删除字段

- 更新：追加where条件防止更新已删除数据，并且使用wrapper/entity作为参数条件时，会忽略该逻辑删除字段

- 删除：转变为更新

  **对于逻辑删除字段，支持Integer、Boolean、LocalDateTime（null为未删除、now（）为已删除）**

**注意：**逻辑删除是为了避免数据被恶意/或不小心删除，方便恢复数据；如果需要对已删除的数据进行展示，则严格意义上只是作为一个状态类型字段使用，因此不能用逻辑删除

#### 逻辑配置

​	1、使用配置文件全局配置：

```yaml
mybatis-plus:
  global-config:
    db-config:
      logic-delete-field: flag  # 全局逻辑删除的实体字段名
      logic-delete-value: 1 # 逻辑已删除值(默认为 1)
      logic-not-delete-value: 0 # 逻辑未删除值(默认为 0)
```

​	2、在对应的实体类字段上添加逻辑删除标识

```java
@TableLogic
private Integer deleted;
```

通过@TableLogic进行进行局部的逻辑删除：

​	1、只有表字段中带有@TableLogic注解，才能够使用逻辑删除

​	2、通过@TableLogic属性，来局部设置当前表的逻辑删除值/未删除值	

| 属性   | 作用         |
| ------ | ------------ |
| value  | 逻辑未删除值 |
| delval | 删除值       |

### 2、通用枚举

​	让mybaits更加优雅的使用枚举属性，即通过枚举来接受JSON参数，并且进行数据库的字段映射

这样可以有效避免手动对固定类型字段进行翻译：

#### 1、配置：

- 定义通用枚举：

  ```java
  @AllArgsConstructor
  public enum  SexEnum implements IEnum<String> {
  
  	NONE("0","未知"),
  	MAN("1","男"),
  	WOMAN("2","女");
  
  	private String code;
  	private String msg;
  
  	//作为数据库字段的数据
  	@Override
  	public String getValue() {
  		return this.code;
  	}
      
      //重写toString方法，用于json数据返回
      @Override
  	public String toString() {
  		return this.msg;
  	}
  }
  ```

- 开启枚举类包扫描，注入到spring容器中，交给mybatis-plus处理，此时会使用mybatis-plus定义好的MybatisEnumTypeHandle进行javaType和jdbcType的类型转换

  ```java
  mybatis-plus:
  	typeEnumsPackage: 
  ```

- 配置Jackson枚举序列化方式，使用toString

  ```java
  //mybaits枚举序列化时，返回其code值(即枚举序列化使用toString方法)
  objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING,true);
  ```

#### 2、springMVC接收枚举参数：

- springMVC默认提供StringToEnumConverterFactory，对于将String转换为枚举，实现url、表单传参：

​	**根据枚举实例名进行匹配**，可以通过自定义ConverterFacotry来修改匹配策略

- 使用jackson对枚举进行反序列化，从而支持body传参，默认：

  1、根据枚举实例名进行匹配

  2、根据枚举第一个属性值进行匹配

### 3、类型处理器

​	mybatis中，提供类型处理器，用于JavaType与JdbcType之间的转换，但是由于mybatisPlus提供了许多封装好的service、mapper方法，因此不能以mybatis正常方式进行配置。mybatisPlus通过@TableName、@TableField注解，来进行全局的类型处理器声明：

首先需要了解@TableName、@TableField的常用属性：

#### @TableName

@tableName常用属性

| 属性          | 作用                                   |
| ------------- | -------------------------------------- |
| value         | 表名（默认驼峰模式匹配）               |
| schema        | schema模式（部分数据库）               |
| autoResultMap | 默认false，是否自动构建resultMap并使用 |

- autoResultMap:

  **开启后，mybatis会自动构建一个resultMap，用于在进行映射时设置typehandle，进行类型转换**；本质上，在不开启autoResultMap时，mybaits会使用resultType进行映射，此时就无法使用mybatis设置的自定义类型转换器，而是使用默认的类型转换器

#### @TableField

@TableField常用属性：

| 属性  | 作用                                 |
| ----- | ------------------------------------ |
| value | 定义数据库字段名（默认驼峰模式匹配） |
| exist | 默认ture，是否作为数据库字段         |
| fill  | 字段填充策略                         |

**对于entity中非映射成员变量，也需要添加@TableField（exist=false），否则在进行单表CRUD操作时，会默认将其拼接到sql中**

#### 使用和配置：

```java
@TableName(value = "sys_user", autoResultMap = true)
public class SysUser extends BaseEntity {
	xxxx
 
    //用于将String类型的json数据，转换为对象
    @TableField(typeHandler = JacksonTypeHandler.class)
	private BaseDto baseDato;
}
```

### 4、自动填充

​	通过定义填充处理器，来指定某些字段在执行时的填充策略，就常见的使用就是，定义数据createdBy、createdDate等字段：

- 使用@TableField指定对于字段的填充处理器填充策略：

  ```java
  @Data
  public class BaseEntity {
  	/**
  	 * uuid
  	 */
  	@TableId(value = "id", type = IdType.ASSIGN_UUID)
  	private String id;
  
  	/**
  	 * 创建人
  	 */
  	@TableField(value = "created_by" ,fill = FieldFill.INSERT)
  	private String createdBy;
  
  	/**
  	 * 最后修改人
  	 */
  	@TableField(value = "last_modified_by" ,fill = FieldFill.INSERT_UPDATE)
  	private String lastModifiedBy;
  
  	/**
  	 * 创建时间
  	 */
  	@TableField(value = "created_date" ,fill = FieldFill.INSERT)
  	private LocalDateTime createdDate;
  
  	/**
  	 * 最后修改时间
  	 */
  	@TableField(value = "last_modified_date",fill = FieldFill.INSERT_UPDATE)
  	private LocalDateTime lastModifiedDate;
  
  ```

  FieldFill提供如下生成策略：

  | 策略          | 描述                   |
  | ------------- | ---------------------- |
  | DEFAULT       | 默认值，不处理         |
  | INSERT        | 插入时，填充字段       |
  | UPDATE        | 更新时，填充字段       |
  | INSERT_UPDATE | 插入和更新时，填充字段 |

- 定义填充处理器，用于元数据entity的字段填充：

  ```java
  	//mybatisPlus填充处理器，填充BaseEntity中的字段
  	@Bean
  	public MetaObjectHandler metaObjectHandler() {
  		return new MetaObjectHandler() {
  			@Override
  			public void insertFill(MetaObject metaObject) {
  				//新增时，添加createdDate字段数据
  				this.strictInsertFill(metaObject, "createdDate", LocalDateTime.class, LocalDateTime.now());
  			}
  
  			@Override
  			public void updateFill(MetaObject metaObject) {
  				//修改时，添加lastModifiedDate字段数据
  				this.strictUpdateFill(metaObject, "lastModifiedDate", LocalDateTime.class, LocalDateTime.now());
  			}
  		};
  	}
  ```

### 6、sql注入器

​	mybatisPlus所有CRUD操作都是基于sql注入器来完成的，本质上就是对mybatis的增强，流程如下：

- mybatis扫描注册mapper类，使用sqlSession对其进行动态代理，创建MapperProxy
- MapperProxy调用方法，使用executor完成相应操作：
  - 获取方法签名、sql命令,sql命令来判断为那种sql操作（CRUD），通过方法签名中的返回值类型，来选择sqlSession需要执行的方法
  - 通过方法签名获取statement信息，并使用sqlsession获取configuration，对statement进行处理
  - 在MybatisConfiguration中，提供了mappedStatements用于保存当前mapper的所有注入器方法的mappedStatements
  - 之后就完成mybatis的后续操作

而mybatisPlus的sql注入器，就可以完成MybatisConfiguration.mappedStatements的创建

#### 1、自定义sql注入器

1、自定义父类mapper接口，继承BaseMapper

```java
public interface MyBaseMapper<T> extends BaseMapper<T> {
	//自定义CRUD方法
	int deleteAll();
}
```

2、自定义mapper方法的MappedStatement的生成类

```java
public class DeleteAll extends AbstractMethod  {

	@Override
	public MappedStatement injectMappedStatement(Class<?> mapperClass, Class<?> modelClass, TableInfo tableInfo) {
		SqlMethod sqlMethod = SqlMethod.LOGIC_DELETE;
		String methodName ="deleteAll";//对应配置的mapper方法名
		String sql;
		SqlSource sqlSource;
		if (tableInfo.isWithLogicDelete()) {//逻辑删除
			//只拼接表名和逻辑删除条件判断
			sql = String.format(sqlMethod.getSql(), tableInfo.getTableName(), this.sqlLogicSet(tableInfo),"","");
			sqlSource = this.languageDriver.createSqlSource(this.configuration, sql, modelClass);
			return this.addUpdateMappedStatement(mapperClass, modelClass, this.getMethod(sqlMethod), sqlSource);
		} else {//非逻辑删除
			sqlMethod = SqlMethod.DELETE;
			//只拼接表名
			sql = String.format(sqlMethod.getSql(), tableInfo.getTableName(),"","");
			sqlSource = this.languageDriver.createSqlSource(this.configuration, sql, modelClass);
			return this.addDeleteMappedStatement(mapperClass, methodName, sqlSource);
		}
	}

```

3、创建sql注入器类，实现`ISqlInjector`接口（如果是继承了BaseMapper，则直接继承DefaultSqlInjector，重写它的方法，保证将BaseMapper中的mapper方法一起注册）

 ```java
public class MySqlInjector extends DefaultSqlInjector {
	@Override
	public List<AbstractMethod> getMethodList(Class<?> mapperClass) {
		//注入BaseMapper的sql
		List<AbstractMethod> methodList = super.getMethodList(mapperClass);
		//注入自定义sql
		methodList.add(new DeleteAll());
		return methodList;
	}
}
 ```

4、配置sql注入器

```java
	//添加mybatisPlus自定义sql注入器（由于sql注入器只能有一个，因此需要继承DefaultSqlInjector,保证原有mapperCRUD方法的sql注入）
	@Bean
	public ISqlInjector iSqlInjector(){
		return new MySqlInjector();
	}
```

#### 2、mybatis默认可装配mapper方法

继承指定类，编写sql注入的生成器，重写Predicate默认值（指定实体类中起作用的字段）：

| mapper方法                                | 作用                     |
| ----------------------------------------- | ------------------------ |
| alwaysUpdateSomeColumnById(T entity)      | 更新指定字段             |
| insertBatchSomeColumn(List<T> entityList) | 批量插入指定字段         |
| deleteByIdWithFill(T entity)              | 根据指定字段条件进行删除 |

## 6、插件（拦截器）

​	mybatisPlus本身提供了很多插件：

| 插件拦截器类                     | 作用               |
| -------------------------------- | ------------------ |
| **PaginationInnerInterceptor**   | 自动分页           |
| TenantLineInnerInterceptor       | 多租户             |
| DynamicTableNameInnerInterceptor | 动态表名           |
| OptimisticLockerInnerInterceptor | 乐观锁             |
| IllegalSQLInnerInterceptor       | sql性能规范        |
| **BlockAttackInnerInterceptor**  | 防止全表更新与删除 |

在引入插件时，需要保证拦截器的顺序，推荐：

- 多租户,动态表名
- 分页，乐观锁
- sql性能规范,防止全表更新与删除

**对sql进行单次改造的优先；不对sql进行改造的放在最后，从而保证互不影响**

### @InterceptorIgnore

mybatisPlus提供@InterceptorIgnore注解，作用于Mapper接口方法上，来忽略某些内置拦截器的拦截：

| 属性名           | 作用                                   |
| ---------------- | -------------------------------------- |
| tenantLine       | 行级租户                               |
| dynamicTableName | 动态表名                               |
| blockAttack      | 攻击 SQL 阻断解析器,防止全表更新与删除 |
| illegalSql       | 垃圾SQL拦截                            |

**这些属性默认为false，如果设置为ture，则忽略该方法的对应拦截**

## 7、关于sql注入问题：

​	对于已有BaseMapper中的方法，其模板已经被myabtisPlus固定，放在**SqlMethod**枚举中，然后通过mapper接口类型确定modelClass，从而获取实体类中的mybatisPlus注解、字段数据，进行sql拼接

​	因此对于多余大部分拼接的sql字符串，都是通过实体类固定死的，不会存在sql注入问题，但mybatisPlus通过条件构造器，还提供了一些更灵活的方法，需要我们注意sql注入问题：

- AbstractWrapper.apply（String applySql, Object... params）方法，如果applySql作为参数交给用户控制时，就会存在sql注入问题，当然mybatisPlus提供了占位符的写法避免这个问题，如：

  ```java
  apply("date_format(dateColumn,'%Y-%m-%d') = {0}", "2008-08-08")--->date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")
  ```

- AbstractWrapper.lastSql（String lastSql）、AbstractWrapper.exists（）、insql（）等直接进行sql拼接时，如果sql中使用了用户传递的参数，则需要注意sql注入问题


## 8、其他

​	mybatisPlus还可以通过引入其他的依赖包，进行增强处理：

- **sql分析打印**

  该功能依赖 `p6spy` 组件，完美的输出打印 SQL 及执行时长

- **多数据源处理**

  需要根据实际情况，进行配置使用

- **代码生成器（AutoGenerator） **

  AutoGenerator 是 MyBatis-Plus 的代码生成器，通过 AutoGenerator 可以快速生成 Entity、Mapper、Mapper XML、Service、Controller 等各个模块的代码，极大的提升了开发效率