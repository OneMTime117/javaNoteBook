# web开发工具集框架

常用工具框架包括：

swagger、HibernateValidation、lombok、Jackson、logfj4/logback、HuTool

## 1、Swagger

### 1、基本概念：

​	**Swagger：**一个流行的API开发框架，以OAS（OpenAPI Specification OpenAPI规范）为基础，使用json/yaml来描述API接口，自动生成API接口文档（支持所有语言）

​	**OAS：**OpenApi规范，定于了REST API的描述格式，使用json/yaml来编写，方便开发人员阅读API接口，主要包含如下部分：

- openApi-版本号

  ```yaml
  # OpenAPI 规范版本号
  openapi: 3.0.0
  ```

- info-有关api元数据

  ```yaml
  # API 元数据信息
  info:
    title:  xx开放平台接口文档                    # 应用的名称
    description: |                          
      简短的描述信息，支持 markdown 语法。 | 表示换行，< 表示忽略换行。
    version: "1.0.0"                            # API 文档的版本信息
    termsOfService: 'http://swagger.io/terms/'  # 指向服务条款的 URL 地址
    contact:                                    # 所开放的 API 的联系人信息
      name: API Support                           # 人或组织的名称
      url: http://www.example.com/support         # 指向联系人信息的 URL 地址
      email: apiteam@swagger.io                   # 人或组织的 email 地址
    license:                                    # 所开放的 API 的证书信息。
      name: Apache 2.0
      url: 'http://www.apache.org/licenses/LICENSE-2.0.html'
  ```

- servers-接口文档服务

  ```yaml
  # 服务器连接信息
  servers:
    - url: https://development.gigantic-server.com/v1
      description: 开发服务器
    - url: https://staging.gigantic-server.com/v1
      description: 测试服务器
    - url: https://api.gigantic-server.com/v1
      description: 生产服务器
  ```

- paths-api的可用路径和操作	

  ```yaml
  # 对所提供的 API 有效的路径和操作
  paths:
    /pet/{petId}:
      get:
        tags:
          - pet
        summary: 简要总结，描述此路径内包含的所有操作。
        description: 详细说明，用于描述此路径包含的所有操作。
        operationId: getPetById      # 此操作的唯一标识符
        parameters:                 # 参数列表
          - name: petId
            in: path
            description: 路径参数，宠物 ID。
            required: true
            schema:
              type: integer
              format: int64
        responses:                  # 接口响应内容
          '200':
            description: 操作成功
            content:
              application/json:
                schema:				#成功，返回的数据对象
                  type: object
                  properties:
                    id:
                      type: integer
                      format: int64
          '400':
            description: 无效的 id
          '404':
            description: 找不到指定的资源
        security:             # 作用于此操作的安全机制
          - api_key: []         # 可以声明一个空数组来变相的移除顶层的安全声明
  ```

### 2、springfox-Swagger2

​	对于java语言的web开发，一般是基于springMVC来实现，因此通过引入**Springfox-Swagger2框架**；它是利用swagger2.0规范，基于springMVC和springboot项目，来自动生成JSON格式的描述文件。但是并不属于spring框架下的分支

- 导包：

  ```java
  <!-- swagger -->
  <dependency>
  	<groupId>io.springfox</groupId>
  	<artifactId>springfox-swagger2</artifactId>
  	<version>2.9.2</version>
  </dependency>
  ```

- 配置类:

  ```java
  /**
   * @author yuanhuan
   * @description: springfox-swagger2配置类，接口访问路径为：ip:port/xx/v2/api-docs
   *               如果引入springfox-swagger-UI,其接口UI地址为: ip:port/xx/swagger-ui.html
   *               如果引入knife4j,其接口UI地址为: ip:port/xx/doc.html
   * @date 2021/4/26 9:11
   */
  
  @Configuration
  @EnableSwagger2//开启swagger2功能
  public class swaggerConfig {
  
  	//自定义配置参数，用于外部控制swagger插件的开启
  	@Value(value="${swagger.enabled}")
  	Boolean swaggerEnabled;
  	@Bean
  	public Docket docket(){
  		Docket docket = new Docket(DocumentationType.SWAGGER_2)
  				//是否开启
  				.enable(swaggerEnabled)
  				//接口文档基本信息
  				.apiInfo(apiInfo())
  				//如果设置了多个docket实例，则需要指定分组名，默认为DEFAULT_GROUP_NAME
  //				.groupName(null)
  				//初始化api选择器的构建器
  				.select()
  				//需要扫描的api接口（默认any）
  				.apis(RequestHandlerSelectors.basePackage("com.yh.system.controller"))
  				//需要暴露的接口（默认any）
  				.paths(PathSelectors.any())
  				//构建选择器
  				.build()
  				//接口文档前缀basePath，需要和server.servlet.content-path保持一致
  				.pathMapping("/");
  		return docket;
  	}
  
  	private ApiInfo apiInfo(){
  		return new ApiInfoBuilder()
  				//接口文档标题
  				.title("Api Document for myProject-system")
  				//接口文档描述
  				.description("swagger文档")
  				//接口联系人
  				.contact(new Contact("yh","/","**"))
  				//版本
  				.version("1.0.0v")
  				.build();
  	}
  }
  ```

### 3、springfox-swagger-ui

​	单纯的根据swagger的json文档，并不能很好的使用，因此通过swagger UI 可以高效的使用swagger文档，方便前后端联调。只需要导包：

```xml
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
            <scope>compile</scope>
        </dependency>
```

### 4、knife4j

​	其前身为swagger-bootstrap-ui，是swagger-ui的一个ui皮肤项目，用于对其进行前端ui的增强处理，后提供了对于微服务、OAuth2.0项目的额外设计，使接口文档使用更加方便**（查看knife4j官网）**

​	其本身2.x版本基于springfox-swagger2实现，引入了springfox-swagger2，其配置和springfox-swagger2类似：

**将@EnableSwagger2替换为了@EnableSwagger2WebMvc，用于选择支持springMVC；也可以使用@EnableSwagger2WebFlux来支持spring5的webFlux**

```xml
        <!-- swagger增强文档 -->
        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.6</version>
        </dependency>
```

**其UI访问页面为：ip：port/xx/doc.html**

### 5、swagger2注解

​	swagger通过注解，来定义springMVC中controller层的接口信息，基于swagger-models包，常用注解如下：

#### 1、@Api

**请求类的说明**，注解在controller类上

| 属性     | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| value    | 没有意义                                                     |
| tags     | 生成文档标签列表（可以设置多个值，进行tags分组）             |
| consumes | 接口能接受的content-type                                     |
| produces | 接口返回的content-type                                       |
| position | 位置排序，默认0；数字越大，优先级越高（全为0时，根据字母排序） |
| hidden   | true，文档中隐藏接口                                         |

**tags分组，就是可以将一个controller的多个接口同时放在多个tags标签中，或者多个controller的接口放在一个tags标签内，方便接口的浏览**

#### 2、@ApiOperation

**请求方法的说明**，注解在controller类的方法上

| 属性              | 作用                             |
| ----------------- | -------------------------------- |
| value             | 接口描述                         |
| notes             | 接口备注说明                     |
| response          | 返回值数据类型                   |
| responseContainer | 返回值容器类型（list、set、map） |

**如果项目使用了特定的返回值对象（集合、泛型对象除外）进行封装，此时就需要使用response搭配responseContainer来指定返回值，否则不能正常识别返回的数据，从而无法生成返回值model的相关文档说明**

#### 3、@ApiImplicitParams、@ApiImplicitParam

**请求方法参数的说明，**如果不进行指定，会默认根据springMVC参数解析规则来生成参数说明；如果指定了，就会覆盖默认生成的;因此对于简单参数，可以不进行描述；如果进行描述，则一定要定义全部信息

| 属性      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| name      | 参数名                                                       |
| value     | 参数说明                                                     |
| required  | 是否必传，默认false                                          |
| paramType | query、header、path；对应@RequestParam、@RequestHeader、@PathVariable |
| dataType  | 数据类型，默认String，其他Ineger                             |

#### 4、@ApiResponses、@ApiResponse

**请求方法响应码说明，**一般进行全局处理，说明响应code所对应的情况

#### 5、@ApiModel

**用于JavaBean作为model说明（requestBody和responseBody的内容），该注解只是对javaBean进行额外说明，但并不能控制是否生成JavaBean的属性说明**

| 属性        | 作用                 |
| ----------- | -------------------- |
| description | 作为入参时，对象描述 |

#### 6、@ApiModelProperty

**JavaBean中的属性含义说明**

| 属性     | 作用                                                         |
| -------- | ------------------------------------------------------------ |
| value    | 字段说明                                                     |
| required | 作为入参，是否必须传入，默认false                            |
| position | 位置排序，默认0；数字越大，优先级越高（全为0时，根据字母排序） |
| hidden   | 是否隐藏，默认false                                          |

**注意：**

- swagger对于属性含义文档生成，依赖于setter/getter方法，分别对于入参、响应的model文档
- swagger属性含义说明支持对象嵌套

### 6、swagger2注解使用实例

Model：

```java
@Data
@ApiModel(description = "演示dto")
public class DemoDTO {

	@ApiModelProperty(value = "名字", required = true)
	private String name;

	@ApiModelProperty(value = "密码", required = true)
	private String password;

	@ApiModelProperty(value = "时间", required = true)
	private LocalDateTime time;

	@ApiModelProperty("单个inner")
	private InnerDemoDTO inner;

	@ApiModelProperty("inner集合")
	private List<InnerDemoDTO> innerDemoDTOList;
}
```



```java
@ApiModel(description = "演示内部dto")
@Data
public class InnerDemoDTO {

	@ApiModelProperty("id")
	private String id;
}
```

Controller:

```java
@Api(tags = "swagger文档接口demo")
@RestController
@RequestMapping("/demo/swagger")
public class DemoController {

	@GetMapping("/demo1")
	@ApiOperation(value = "示例1" ,response = DemoDTO.class )
	@ApiImplicitParams({
			@ApiImplicitParam(name = "name", value = "姓名", required = true, paramType = "query" ),
			@ApiImplicitParam(name = "password", value = "密码", required = true, paramType = "query"),
			@ApiImplicitParam(name = "time", value = "时间", required = false, paramType = "query")
	})
	public Result demoResult(@RequestParam String name, @RequestParam String password, LocalDateTime time) {
		DemoDTO dto = new DemoDTO();
		dto.setName(name);
		dto.setPassword(password);
        dto.setTime(time);
		return Result.ok(dto);
	}

	@PostMapping("/demo2")
	@ApiOperation("示例2")
	public DemoDTO demoDemoDTO(@RequestBody DemoDTO dto){
		return dto;
	}
}
```

## 2、Bean Validation

### 1、基本概念

​	Bean Validation 是一个运行时的数据校验框架，如果验证失败，则直接返回错误信息。在JAVAEE6中，发布了Bean Validation的规范**validation-api**，即**JSR303**;

​	由于版权问题，JAVAEE8改为Jakarta EE 8，其对应的jar为：**jakarta.validation-api**，并且升级为**Bean Validation2**（即**JSR380**），支持更加灵活的验证

​	JAVAEE中只是定义了BeanValidation的规范，提供了相应的元数据模型和接口API，其**唯一实现是Hibernate Validator**

### 2、Hibernate Validator基本使用

1、导入Hibernate Validator最新版本：

```xml
<dependency>
     <groupId>org.hibernate.validator</groupId>
     <artifactId>hibernate-validator</artifactId>
     <version>6.1.6.Final</version>
</dependency>
```

2、在Bean中通过注解，定义校验的约束条件

```java
@Data
public class BeanValidatorDemoDTO {
	@NotNull(message="不能为null")
	private String nameA;
}
```

2、使用Validator校验器，进行数据校验

```java
//需要校验的dto		
BeanValidatorDemoDTO dto = new BeanValidatorDemoDTO();
//创建validator
Validator validator = Validation.buildDefaultValidatorFactory().getValidator();
//进行校验，获取校验结果（不通过的）
Set<ConstraintViolation<BeanValidatorDemoDTO>> validate = validator.validate(idDTO);
//打印不通过的结果消息
validate.forEach( action ->{
			System.out.println(action.getPropertyPath());//字段名
			System.out.println(action.getMessage());//message
			System.out.println(action.getInvalidValue());//字段值
});
```

**注意：**	

​	1、Validator实例是线程安全的，可以重复多次使用

​	2、ConstraintViolation<T>会保存所有验证错误的信息，如果为空，说明所有验证通过

### 3、Bean约束

#### 1、Bean约束的声明

##### 1.1、四种类型的Bean约束

- **字段约束(Field)**

  ```java
  @NotNull(message="不能为null")
  private String nameA;
  ```

  **Hibernate Validator基于反射实现，因此可以访问任何访问权限的字段，但不支持对静态字段的约束**

- **getter方法约束**

  ```java
  @NotNull(message="不能为null")
  public String getNameA(){
      return nameA;
  }
  ```

  **相对于字段约束，可以看做是其懒加载，即需要使用该字段时，进行校验。注意，避免和字段约束同时使用，这样会进行两次校验**

- **容器元素约束**

  ```java
  private List<@NotBlank(message = "不能为空字符串") String> parts 
  ```

  **可以对容器泛型进行Bean约束，从而校验容器中的元素，支持Iterable、List、Set、Map、Optional**

- **类约束**

  使用的比较少，约束注解支持很少，一般用于校验对象多个属性之间的相关性

##### 1.2、getter方法约束的继承

​	getter方法约束会随着类的继承，也一起传递，即使该方法在子类中被重写

##### 1.3、Bean约束的嵌套

​	通过@Valid注解，可以实现Bean之前的属性约束嵌套：

```java
@NotNull
@Valid
private BeanValidatorDemoDTO dto;
```

​	同理，通过该方式，也可以实现在容器约束中的嵌套：

```java
private List<@NotNull @Valid BeanValidatorDemoDTO> list;
```

#### 2、Bean约束的校验

​	Validator对象是Bean约束校验的关键，其实例化方式如下：

```java
ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
validator = factory.getValidator();
```

##### 2.1、校验方式

- Validator.validate(Object obj)

  对指定Bean进行校验

- Validator.validateProperty(Object obj,String propertyName)

  对指定Bean的perpertyName属性进行校验

- Validator.validateValue(Class clazz,String propertyName,Object value)

  指定Bean的perpertyName属性值，进行校验（用于测试Bean的约束是否正确）

##### 2.2、校验结果

​	当校验失败时，会返回对应数量的`ConstraintViolation`对象，使用Set保存。该对象保存了所校验bean的相关信息和失败约束：

| 方法                      | 作用                                     |
| ------------------------- | ---------------------------------------- |
| getMessage()              | 校验失败消息                             |
| getRootBean()             | 验证的最高级Bean对象                     |
| getRootBeanClass()        | 验证的最高级Bean类class对象              |
| getPropertyPath()         | 获取Bean验证的属性路径                   |
| getInvalidValue()         | 获取Bean验证的属性值                     |
| getConstraintDescriptor() | 获取Bean约束的元数据（即Bean的约束注解） |

### 5、Hibernate Validator约束注解

​	Hibernate Validator 6.0是Bean Validation2规范的实现，包含了Bean Validation2中所提供的所有约束注解，并提供了一些其他约束

#### 1、Bean Validation2 约束注解

在Bean Validation2规范中，并没有定义**类级别约束注解**

- @AssertFalse

  检查带注释的元素是否为false

    `Boolean`， `boolean`

- @AssertTrue

  同理@AssertFalse

- @DecimalMax(value=, inclusive=)

  检查带注解的元素是否小于等于指定值；当inclusive=false时，则为小于指定值；value为BigDecimal类型的字符串

  `BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @DecimalMin(value=, inclusive=)

  同理@DecimalMax

- @Digits(integer=, fraction=)

  检查带注释的值是否为最多由`integer`数字和`fraction`小数位组成的数字

   `BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @Email

  检查指定的字符序列是否为有效的电子邮件地址

  `CharSequence`

- @Future

  带注释的日期是否是将来的日期

  支持JDK8中所有时间类型

- @Past

  带注释的日期是否是过去的日期

  支持JDK8中所有时间类型

- @Max(value=)

  检查带注释的值小于或等于最大值

    `BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @Min(value=)

  同理@Max

- @Size(min=, max=)

  检查带注释的元素的大小是否介于`min`和之间`max`（包括）

  `CharSequence`，`Collection`，`Map`和数组

- @Null

  检查注释的值是 `null`

  任何对象类型

- @NotNull

  同理@Null

- @NotEmpty

  检查带注释的元素是否不为null或为大小为空

  `CharSequence`，`Collection`，`Map`和数组

- @NotBlank

  带注释的字符序列不为null，并且修剪的长度大于0，即会去掉两端空格

  `CharSequence`

- @Negative

  检查元素是否严格为负，零值被视为无效

  `BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @Positive

  同理@Negative，检查元素是否严格为正，零值被视为无效

- @NegativeOrZero

  检查元素是负数或者0

  `BigDecimal`，`BigInteger`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @PositiveOrZero

  同理@NegativeOrZero，检查元素是正数或者0

- @Pattern(regex=, flags=)

  带注释的字符串是否与正则表达式匹配；regex为正则表达式；flags为匹配行为

  `CharSequence`

**注意：Hibernate Validator在实现上，对部分约束进行了提供了额外数据类型的支持:**

- 对于所有数字校验（如@DecimalMax、@Max、@Digits、@Negative等），支持`CharSequence`类型(用于表示表示的数字)、`Number`、`javax.money.MonetaryAmount`
- 对于所有时间校验，支持`ReadablePartial`

#### 2、Hibernate Validator额外约束注解

​	HibernateValidator自身还提供了一些额外自定义约束，大部分约束都是针对于特殊情况和特殊java对象的校验来设计的，比如：判断信用卡、货币单位、java.time.Duration、条形码类型等

​	其中最常用的额外约束为：

- @Range(min=, max=)

  检查带注释的值是否介于（包括）指定的最小值和最大值之间

  `BigDecimal`，`BigInteger`，`CharSequence`，`byte`，`short`，`int`，`long`和原始类型的相应的包装

- @URL(protocol=, host=, port=, regexp=, flags=)

  带注释的字符序列是否为有效的URL

  `CharSequence`

- @Length（min= ，max= ）

  检查字符序列是否介于（包括）指定的最小值和最大值之间

  `CharSequence`

- @CreditCardNumber(ignoreNonDigitCharacters=)

  检查带注释的字符序列是否通过了Luhn校验和测试（Luhn算法用于校验银行卡号的有效性）;ignoreNonDigitCharacters忽略非数字字符，默认为false

  `CharSequence`

### 6、方法约束

​	在Bean Validation 1.1开始，约束不再仅限于JavaBean，还可以作用于任何Java类型的方法参数和返回值上

#### 1、方法约束的优点

​	1、相对于Bean约束，不需要手动调用检查，减少不必要的代码编写和维护

​	2、将约束条件注解在方法上，方便声明方法调用时，参数和返回值的必要满足条件，减少文档编写

#### 2、方法约束的声明

```java
    public void rentCar(
            @NotNull Customer customer,
            @NotNull @Future Date startDate,
            @Min(1) int durationInDays) {
        //...
    }
```

**注意：**

**不支持静态方法的约束**

##### 1.1、级联验证

**通过@Valid能够实现方法约束和Bean约束的级联验证**

```java
	@Valid
	public customer rentCar(@Valid Customer customer) {
        	//...
    }
```

此时方法参数、返回值会根据对应的Bean约束进行校验

##### 1.2、方法约束的继承

1、对于方法返回值约束，会继承到子类方法中，并与子类方法上的范围值约束共同作用

**方法返回值约束不能在继承关系中被弱化**

```java
public interface Car{  
    @NotNull
    public List<String> getName(){
        //...
    }
}

public class CarImpl{ 
    @NotEmpty
    public List<String> getName(){
        //...
    }
}
```

​	此时子类对象进行方法调用时，对其方法会进行@NotNull、@NotEmpty的约束校验

2、对于方法参数约束，可以继承到子类方法，但是子类方法不能继续使用额外约束；并且并且超类本身方法参数未指定约束，也不能在子类方法中使用；

**方法参数约束不能在继承关系中被增强**

**注意：**

​	当存在子类继承两个接口，并且接口方法签名一致时；如果子类进行该方法的重写，就需要保证两个接口的方法都没有声明方法参数约束

#### 3、方法约束的校验

​	对于方法约束，提供额外的一个校验器对象`ExecutableValidator`,该对象是基于Validator实例获取而来

```java
ValidatorFactory validatorFactory = Validation.buildDefaultValidatorFactory();
ExecutableValidator executableValidator = validatorFactory.getValidator().forExecutables();
```

​	其使用和Validator类似，对于校验失败的约束，返回一个`Set<ContraintVilation>`结果集,包括如下方法：

- ExecutableValidator#validateParameters(Object obj,Method method,Object[] parameterValues)

  对象实例、方法对象、方法参数值数组，从而验证方法参数的约束

- ExecutableValidator#validateReturnValue(Object obj,Method method,returnValue)

  对象实例、方法对象、方法返回值，从而验证方法返回值的约束

- ExecutableValidator#validateConstructorReturnValue()、ExecutableValidator#validateConstructorReturnValue()

  在方法校验中，使用反射来实现对方法信息的获取，因此方法约束的校验需要分为普通方法和构造方法

此时，ConsatraintViolation校验结果也会提供返回对应信息的方法：

ConstraintViolation#getExecutableParameters()		返回验证的方法参数数组

ConstraintViolation#getExecutableReturnValue()		返回验证的方法返回值对象

#### 4、方法参数交叉约束

​	对于方法参数约束，可以使用多个参数直接的自定义交叉校验：

通过`@ParameterScriptAssert`注解实现，需要使用到兼容 JSR 223规范的JVM脚本语言，比如：

```java
	//实现校验 luggage集合大小小于  passengers集合大小的2倍
	@ParameterScriptAssert(lang = "javascript", script = "luggage.size() <= passengers.size() * 2")
    public void load(List<Person> passengers, List<PieceOfLuggage> luggage) {
        //...
    }
```

### 7、自定义约束错误信息

Bean  Validation规范中，默认在约束注解中提供message属性，进行自定义消息，并且可以通过{}获取资源文件中的消息参数，同时**会优先获取约束注解中的属性值**

```java
@Lenght(min=1,max=2,message="name长度在{min}和{max}之前")
private String name;
```

- Hibernate Validator中，提供了对EL表达式的支持，错误消息定义更加灵活

```java
@Lenght(min=1,max=2,message="name长度大于${min>0?min:0}")
private String name;
```

#### 自定义消息资源

​	Hibernate Validator支持自定义消息插值的方式，并加载额外消息参数资源

默认情况下，使用`ResourceBundleMessageInterpolator`对象进行消息资源插入，并借助`PlatformResourceBundleLocator`进行消息资源加载解析，其解析规则如下：

1、如果消息没有占位符，则不进行处理，直接输出

2、有占位符，则进行资源文件加载解析：

​	1、从用户自己的`classpath`里面去找名为资源文件(userResourceBundleLocator),默认为ValidationMessages，可以自定义声明资源名（包名.文件名）

​	2、加载贡献的资源包(contributorResourceBundleLocator)

​	3、使用defaultResourceBundle加载默认资源包(由Hibernate Validator包提供，并且支持国际化)

3、根据占位符，进行KV匹配，拼接消息

**进行资源加载时，都会根据当前Loacl来定位国际化资源**

通过在classpath路径下，自定义资源文件，就可以实现自定义消息资源，但需要保证文件名满足国际化，比如中国地区为：**ValidationMessages_zh_CN.properties**

### 8、自定义约束注解

实例：校验性别字段只能为0,1，并且抛出错误为性别只能为男、女

分为三步：

1、自定义约束注解

 ```java
//注释三要素
@Documented
@Target({ METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE })
@Retention(RUNTIME)
//约束对应验证器,可以不指定
@Constraint(validatedBy = {SexConstraintValidator.class})
//约束目标（属性元素，方法参数（用于交差参数校验））
@SupportedValidationTarget(ValidationTarget.ANNOTATED_ELEMENT)
//支持可重复
@Repeatable(Sex.List.class)
//使用组合注解时，声明忽略每个注解的错误报告
//@ReportAsSingleViolation
public @interface Sex {

	// 三个必备的基本属性
	String message() default "性别只能为 0 未知,1 男,2 女";
	Class<?>[] groups() default {};
	Class<? extends Payload>[] payload() default {};

	// 重复注解
	@Target({METHOD, FIELD, ANNOTATION_TYPE, CONSTRUCTOR, PARAMETER, TYPE_USE})
	@Retention(RUNTIME)
	@Documented
	public @interface List {
		Sex[] value();
	}
}
 ```

2、实现对应的校验器

```java
public class SexConstraintValidator implements ConstraintValidator<Sex,String> {
	@Override
	public void initialize(Sex constraintAnnotation) {
	}

	@Override
	public boolean isValid(String value, ConstraintValidatorContext context) {
		HibernateConstraintValidatorContext hibernateContext = context.unwrap(HibernateConstraintValidatorContext.class);
		//用于添加消息参数，进行message占位符参数匹配
//		hibernateContext.addMessageParameter("key","value");
		if (value == null) {
			return true;//为null，则不进行校验
		}

		SexEnum[] values = SexEnum.values();
		for (SexEnum sexEnum : values) {
			if (Objects.equals(sexEnum.getCode(),false)){
				return true;
			}
		}
		return false;
	}
}
```

3、在消息资源中，定义约束注解默认的校验错误信息

​	对于message属性，default进行{}占位符声明，并在ValidationMessages.properties文件中添加KV对

### 9、其他

​	Hibernate Validator还有一些其他不常用的功能：

- ValueExtractor：

  ​	value提取器，用于从而自定义的容器中提取值，实现泛型对象中的元素校验

- 分组约束：

  ​	通过约束注解中的groups属性，来定义一个class对象；将具有相同class对象的约束放在一组，其他则放在default组，此时在进行指定对象的校验时，就可以额外声明组的calss，来指定一组约束进行校验（默认选择default组）

- ValidationProvider：

  ​	Validator实例由ValidatorFactory生成，其配置的关键就在于ValidationProvider，默认情况下，使用org.hibernate.validator.HibernateValidator

- MessageInterpolator：

  ​	用于定义错误消息插值方式

- 快速结束：

  ​	Validator进行多个约束校验时，只有出现校验错误，则返回结果：

  需要手动指定Validator的提供者，Hibernate Validator，从而来设置Hibernate Validator特定属性：

  ```
  ValidatorFactory validatorFactory = Validation.byProvider(HibernateValidator.class)
  				.configure()
  				.failFast(true)
  				.buildValidatorFactory();
  ```

### 10、Hibernate Validator搭配springMVC使用

​	在spring中，其spring-context模块提供了validation数据校验功能，并且基于Bean Validation规范，对其实（Hibernate Validator）进行整合：

注意：在springBoot2.3之后，validation模块需要单独手动引入，提供两种方式：

```xml
        <!-- 1、springBoot提供的starter -->
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- 2、直接手动引入hibernateValidate -->
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.1.6.Final</version>
        </dependency>
```

第二种更加方便，可以手动指定使用的hibernateValidate版本

#### 1、配置spring中的校验器

- spring提供`LocalValidatorFactoryBean`来注册内部所使用的校验器Bean（用于springMVC的controller层中Bean约束校验）

  `LocalValidatorFactoryBean`默认会优先获取spring容器中的资源`ResourceBundle`

  ```java
  @Configuration
  public class HibernateValidatorConfig {
  
      //定义Contorller中，Bean约束的校验器（用于支持hibernateValidator的Bean约束）
  	@Bean
  	public LocalValidatorFactoryBean localValidatorFactoryBean(){
  		LocalValidatorFactoryBean factoryBean = new LocalValidatorFactoryBean();
          //校验器提供者
  		factoryBean.setProviderClass(HibernateValidator.class);
          //消息插值器(使用默认值)
  		MessageInterpolatorFactory interpolatorFactory = new MessageInterpolatorFactory();
  		factoryBean.setMessageInterpolator(interpolatorFactory.getObject());
  		return factoryBean;
  	}
  }
  ```

- 配置spring容器中，Bean进行方法约束的校验器：(一般情况下，和controller层的约束校验使用同一个，可以实现controller中的方法简单参数约束校验)

  ```java
  //	定义方法校验的后置处理，指定使用的校验处理器（用于支持hibernateValidator的方法约束）
  	@Bean
  	public MethodValidationPostProcessor methodValidationPostProcessor(){
  		MethodValidationPostProcessor methodValidationPostProcessor = new MethodValidationPostProcessor();
  	methodValidationPostProcessor.setValidatorFactory(localValidatorFactoryBean());
  		return methodValidationPostProcessor;
  	}
  ```

**注意：**

​	在springboot中，默认提供了`LocalValidatorFactoryBean`和`MethodValidationPostProcessor`两个Bean的注入，在不需要改变校验器配置的情况下，不需要额外声明注册（在`ValidationAutoConfiguration`中）

#### 2、spring的消息国际化配置

​	在spring中，Hibernate的消息资源依赖于spring的消息国际化配置：

通过`ResourceBundleMessageSource`来声明国际化资源配置：

```java
	//国际化消息资源配置(springboot必须指定BeanName为messageSource，这样自动配置的Bean才会失效)
	@Bean("messageSource)
	public ResourceBundleMessageSource resourceBundleMessageSource(){
		ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();		messageSource.setBasename("ValidationMessages");
		messageSource.setCacheMillis(3);//资源文件缓存时间 3分钟
		messageSource.setDefaultEncoding("UTF-8");
		messageSource.setUseCodeAsDefaultMessage(true);
		return messageSource;
	}
```

**注意：**

​	在springboot中，默认定义了国际化资源Bean，在`MessageSourceAutoConfiguration`自动配置类中，定义了如下规则：

​	1、默认加载配置文件中，前缀为spring.messages属性，如 spring.messages.name=yh,则存在name=yh该条消息

​	2、获取resources目录下，资源文件为**messages.properties**文件（可以通过spring.messages.basename=xxx 进行修改；并支持国际化资源命名），basename为 包名.文件名

**消息插入方式：**

​	spring整合HibernateValidator，消息插入规则如下：

1、默认不指定消息插值器时，则只使用spring提供的资源文件

2、当使用HibernateValidator的默认插值器时，则只能使用HibernateValidator的默认资源，无法加载自定义资源（springboot默认使用该方式）

​	为了同时满足两种资源的使用，则在自定义资源时，手动加载HibernateValidator的默认资源：

```java
messageSource.setBasenames("message.ValidationMessages","org.hibernate.validator.ValidationMessages");//自定义资源、HibernateValidator默认资源
```

#### 3、@Validated注解使用

springMVC提供@Validated注解实现方法约束校验，注解可以作用在方法参数和类上，用于开启当前bean的方法约束校验（基于Bean后置处理器`MethodValidationPostProcessor`，实现Aop环绕通知，在方法执行前后，进行方法约束校验）

**@Validated和@Valid的异同点：**

- 相同点：

  1、它们都可以实现JSR-303规范

- 不同点：

  1、@Validated可以作用在方法参数、方法和类上；@Valid可以作用在方法参数、方法和属性上

  2、@Validated可注解在类上，用于开启spring容器中Bean的方法约束校验

  3、@Valid只能作用在Controller中，搭配springMVC一系列参数处理器，来实现请求入参的约束校验

  4、@Validated还能够实现分组校验

**@Validated的分组校验：**

​	在复杂业务场景中，多个请求存在相同入参数据，但校验参数的数据却不相同，为了更好的实现Bean的复用，可以通过分组来实现每个接口需要校验的属性：**（实际上也可以通过定义多个DTO入参对象，来解决这个问题，但这样代码过于冗余）**

​	1、定义分组标识类（一般为接口）

```java
public class ValidationGroups {
    public interface Update {
    }
    
    public interface Insert {
    }

    public interface select {
    }

    public interface Delete{

    }
}
```

​	2、DTO定义分组约束

```java
@Data
public class GroupValidatorDemoDTO {

	@NotBlank(groups = {ValidationGroups.Update.class, ValidationGroups.Delete.class, ValidationGroups.Select.class})
	private String id;

	@NotBlank(groups = {ValidationGroups.Insert.class, ValidationGroups.Update.class})
	private String name;
}
```

​	3、controller各方法定义校验组别

```java
	@ApiOperation("分组校验")
	@PostMapping("/group/insert")
	public Result groupInsert(@Validated({ValidationGroups.Insert.class})
	                              @RequestBody GroupValidatorDemoDTO dto){
		return Result.ok();
	}
```

#### 4、springMVC全局异常，封装校验错误信息

​	springMVC基于HibernateValidator有两种校验方式，Bean约束校验和方法约束校验，分别对应了用Bean进行参数接收和使用简单类型进行参数接收,而它们校验失败的异常如下：

| 约束方式 | 参数接收方式                | 异常                            |
| -------- | --------------------------- | ------------------------------- |
| Bean约束 | @ReqeustBody                | MethodArgumentNotValidException |
| Bean约束 | @ModelAttribute             | BindException                   |
| 方法约束 | @RequestParam/@PathVariable | ConstraintViolationException    |

简单错误信息封装：**不暴露字段信息，错误字段依赖于message的声明，因此在使用约束时，需要保证message信息完整**

```java
	// BeanValidation参数校验异常
	/*
	 * @RequestBody Bean约束校验异常
	 *
	 */
	@ExceptionHandler(MethodArgumentNotValidException.class)
	public Result handleMethodArgumentNotValid(MethodArgumentNotValidException e) {
		String message = e.getBindingResult().getAllErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage).collect(Collectors.joining(","));
		return Result.error(message);
	}

	/*
	 * @ModelAttribute Bean约束校验异常
	 */
	@ExceptionHandler(BindException.class)
	public Result handleBind(BindException e) {
		String message = e.getBindingResult().getAllErrors().stream().map(DefaultMessageSourceResolvable::getDefaultMessage).collect(Collectors.joining(","));
		return Result.error(message);
	}

	/*
	 * @RequestParam/@PathVariable 方法参数约束校验异常
	 */
	@ExceptionHandler(ConstraintViolationException.class)
	public Result handleConstraintViolation(ConstraintViolationException e) {
		String message = e.getConstraintViolations().stream().map(ConstraintViolation::getMessage).collect(Collectors.joining(","));
		return Result.error(message);
	}
```

#### 5、springMVC整合HibernateValidator使用原理

根据springMVC参数接受方式，HibernateValidator的校验位置也相应不同：

- @ReqeustBody

  在RequestResponseBodyMethodProcessor#resolveArgument方法中处理：

  ```java
  WebDataBinder binder = binderFactory.createBinder(webRequest, arg, name);
  			if (arg != null) {
  				validateIfApplicable(binder, parameter);
  				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
  					throw new MethodArgumentNotValidException(parameter, binder.getBindingResult());
  				}
  			}
  ```

  会将得到的参数交给webDataBinder管理，绑定到指定的javaBean，并进行Validator校验；并将结果报错到bindingResult中，如果有错误，则抛出MethodArgumentNotValidException异常

- @ModelAttribute

  在ModelAttributeMethodProcessor#resolveArgument方法中处理：

  ```java
  		if (bindingResult == null) {
  			// Bean property binding and validation;
  			// skipped in case of binding failure on construction.
  			WebDataBinder binder = binderFactory.createBinder(webRequest, attribute, name);
  			if (binder.getTarget() != null) {
  				if (!mavContainer.isBindingDisabled(name)) {
  					bindRequestParameters(binder, webRequest);
  				}
  				validateIfApplicable(binder, parameter);
  				if (binder.getBindingResult().hasErrors() && isBindExceptionRequired(binder, parameter)) {
  					throw new BindException(binder.getBindingResult());
  				}
  			}
  			// Value type adaptation, also covering java.util.Optional
  			if (!parameter.getParameterType().isInstance(attribute)) {
  				attribute = binder.convertIfNecessary(binder.getTarget(), parameter.getParameterType(), parameter);
  			}
  			bindingResult = binder.getBindingResult();
  		}
  ```

  同样使用webDataBinder进行处理，如果校验失败，则抛出BindException异常

- @RequestParam

  @RequestParam参数使用的方法约束校验，通过spring的Bean后置处理器进行切面增强，使用环绕通知，来对Bean的方法参数进行校验

  在MethodValidationInterceptor#invoke，进行处理器方法的参数校验

  ```java
  	try {
  			result = execVal.validateParameters(
  					invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
  		}
  		catch (IllegalArgumentException ex) {
  			// Probably a generic type mismatch between interface and impl as reported in SPR-12237 / HV-1011
  			// Let's try to find the bridged method on the implementation class...
  			methodToValidate = BridgeMethodResolver.findBridgedMethod(
  					ClassUtils.getMostSpecificMethod(invocation.getMethod(), invocation.getThis().getClass()));
  			result = execVal.validateParameters(
  					invocation.getThis(), methodToValidate, invocation.getArguments(), groups);
  		}
  		if (!result.isEmpty()) {
  			throw new ConstraintViolationException(result);
  		}
  
  ```

  当校验结果不为空是，则抛出ConstraintViolationException异常

### 约束注解使用注意：

1、所有约束注解对`null`是免疫的，即如果校验值为null，则不进行校验逻辑（@NotNull、@Null、@NotEmpty、@NotBlank除外）；因此大部分情况下，需要组合@NotNull使用

2、在Hibernate Vlidator中，@Email 、@NotEmpty等一系列注解优先于Bean Validation规范被提出，在Bean Validation2发布后，搜入了这些注解，导致有两套相同注解包，最后Hibernate Vlidator在后续版本中将自己的注解进行了@Deprecated过期处理；因此推荐使用Bean Validation2 规范包中的注解

## 3、Lombok

​	lombok并不是一个框架，而是java使用工具包（插件），可以帮助开发者简化POJO的编写，通过一些注解，来减少冗余样板式代码的声明；

### 1、lombok原理

​	Lombok实现了JSR269 规范，即**插件化注解处理（Pluggable Annotation Processing）**,在JDK6中，该规范被得到支持实现。使得代码在编译期间，根据指定注解，进行额外的代码处理，实现编译期的代码插入。

​	JSR269规范的准备步骤：

1、自定义一个注解，并且声明周期为编译期：@Retention(RetentionPolicy.SOURCE)

2、自定义一个AnnotationProcessor，继承javax.annotation.processing.AbstractProcessor，并重写process方法

3、在自定义的AnnotationProcessor类上，添加javax.annotation.processing.SupportedAnnotationTypes注解，定义其所处理的注解类型；添加javax.annotation.processing.SupportedSourceVersion定义源代码编译版本；添加javax.annotation.processing.SupportedOptions注解，定义编译参数

​	此时就可以实现，在编译器对源代码进行处理，而Lombok编译期的处理流程如下：

在源代码进行编译处理后，lombok会根据自己的抽象语法树进行相应注解匹配，如果有使用了lombok的注解，则其注解处理器就会根据对应的语法树，生成相应代码的编译数据，写入到class文件中

### 2、Lombok常用注解：

#### @Getter/@Setter

​	注释在类和字段上，并提供value属性，来定义方法访问权限级别，默认为AccessLevel.PUBLIC

​	当注释在类时，相当于为所有非static字段添加@Getter/Setter注解（当然也可以使用在非static字段上）

**注意：**

- 对于Boolean包装类型，即使字段名使用is开头，其getter、setter方法名也会添加get/set前缀（而IDE自动生成的，会去掉is，然后使用get、set前缀）
- 对于boolean类型，则和IDE自动生成方式一致，getter方法直接使用isXXX；settter方法去掉is，添加set前缀

#### @ToString

​	注释在类上，用于重写该类的toString（）方法；默认情况下，会按顺序打印每个字段名称和值；

- toString打印字段的选择：

  **默认情况下，只会打印所有非静态字段**

  1、可以通过@ToString.Exclude注解，排除指定字段；

  2、设置@ToString注解中的属性onlyExplicitlyIncluded = true，则可以使用@ToString.Include注解，指定需要打印的所有字段;同时@ToString.Include还提供rank属性来指定打印顺序（数字越大，优先级越高，默认为0）

- 父类字段的打印：

  @ToString体用callSuper属性，默认为false，不进行父类字段的打印；当设置为true后，就会先输出父类的toString方法（注意，对于Object父类，并不会进行打印）

#### @EqualsAndHashCode

​	用于对所有非静态、非瞬态字段，进equals（）、hashCode（）方法处理；并且也可以使用@EqualsAndHashCode.Include/@EqualsAndHashCode.Exclude来指定字段或删除部分字段

​	**注意：**

​	当存在父类继承时，该功能会存在一些问题（当超类没有提供有价值的equals、hashCode方法），因此不推荐使用

- hashCode算法：

  ```java
      @Override public int hashCode() {
        final int PRIME = 59;
        int result = 1;
        result = (result*PRIME) + super.hashCode();
        result = (result*PRIME) + this.width;
        result = (result*PRIME) + this.height;
        return result;
      }
  ```

#### @NoAragsConstructor、@RequiredArgsConstructor、@AllArgsConstuctor

​	用于生成类的构造函数，不处理静态字段：

@NoAragsConstructor：生成无参构造函数，并需要保证所有final字段被初始化（否则编译错误）；使用force = true属性，可以自动完成所有final字段的初始化（0，null，false）

@RequiredArgsConstructor：生成所有final字段的构造函数

@AllArgsConstructor：生成所有非静态字段的构造函数

### @Data

​	@Data捆绑了@ToString、@EqualsAndHashCode、@Getter/Setter和@RequiredArgsConstructor注解，方便lombok对于POJO的模板化的注解使用（即，POJO类一般都需要添加以上注解）

​	但是@Data并没有提供绑定注解的相关属性设置，如果需要改变有关绑定注解的属性，可以进行额外显示声明

### @Builder

​	@Builder会自动为类创建构建器，做成如下操作：
1、创建一个内部静态类 FooBuilder，并提供一个静态方法获取该类对象

2、在构建器类中，对非静态的字段进行构建器方法创建

3、提供构建器的toString方法

### @Slf4j

​	用于简化类中静态、不可变日志对象的创建，目前，所有日志框架都支持SLF4J日志门面，因此推荐使用@Slf4j进行创建：

```java
private static final org.slf4j.Logger log =org.slf4j.LoggerFactory.getLogger(LogExample.class);
```

## 4、jackson

### 1、JSON

​	JSON是JavaScript Object Notation，用于描述javaScript中的对象，普遍用于前后端的数据传输

1、优点:

- 只允许使用UTF-8编码，避免了编码格式转换
- 格式简单，可读性强
- 浏览器内置JSON支持，直接可以交给JavaScript处理

2、数据类型：

- 对象，即KV键值对：{“key”：value}
- 数组：[1,2,3]
- 字符串: "abc"
- 数字（整数、小数）：12.23
- 布尔值：true/false
- 空值：null

### 2、JAX-RS、JSR 353

​	在JSON未出来前，所有的web数据通讯都是用xml进行传输，这也是AJAX的来由（异步的javaScript和xml）；而JAX-RS就是在JAVA EE6引入，用于RESTful web服务的API搭建（类似于springMVC），提供了一系列注解和api，将javaBean封装为web资源（即xml数据），进行请求响应

​	在JSON替代xml后，就需要提供一个技术规范，将javaBean封装为json数据。在JAVAEE 7中，提供了JSR 353规范，用于处理JSON，其中由Oracle公司开发的JSONP就是实现了该规范

### 3、常用json库比较

​	目前有四种常用json库：

- Json-lib：

  ​	出现最早，性能差，依赖很多其他包，对于泛型集合的转换存在问题，基本被淘汰

- Gson

  ​	Google公司开发，满足各种java复杂对象的解析，不需要额外依赖包

- FastJson

  ​	阿里开发，高性能JSON处理器，转换速度最快，但是对于复杂类型Bean转换为Json存在问题，需要手动定制引用

- JackSon

  ​	使用最为广泛，被springMVC使用，api灵活，可进行模块化的定制扩展，也因此有许多相关依赖包。对于大文件的解析速度更快

### 4、Jackson使用

​	Jackson目前常用2.x版本，其核心模块依赖包括：

- jackson-core 核心包，提供流式API来进行JSON解析和生成
- jackson-databind 数据绑定包，提供**数据绑定**和**树模型**两种API方式，进行JSON解析和生成（底层都是依赖于流式API完成）
- jackson-annotations 注解包，提供相关jackson注解，来定义javaBean的json序列化和反序列化配置

#### 1、三种API的使用

##### 1、流式API

​	由于是最底层的API，因此速度最快，但使用起来比较麻烦，主要是通过JsonParser、JsonGenerator来实现JSON对象的生成：

```java
JsonFactory jsonFactory = new JsonFactory();
		//创建json生成器,并指定json数据输出位置
		StringWriter stringWriter = new StringWriter();
		JsonGenerator generator = jsonFactory.createGenerator(stringWriter);

		//开始进行json生成
		generator.writeStartObject();
		//json键值对
		generator.writeStringField("name", "yh");
		//json数组
		generator.writeFieldName("array");
		generator.writeStartArray();
		generator.writeString("msg1");
		generator.writeString("msg2");
		generator.writeEndArray();
		//json对象
		generator.writeFieldName("object");
		generator.writeStartObject();
		generator.writeNumberField("id", 1234);
		generator.writeStringField("name", "ddd");
		generator.writeEndObject();
		//接受JSON对象生成，将json数据输出(解析为json字符串)
		generator.writeEndObject();
		generator.close();
		System.out.println(stringWriter.toString());

		//创建json解析器，解析json字符串
		String jsonStr = "{\"name\":\"YH\",\"id\":\"12345\"}";
		JsonParser parser = jsonFactory.createParser(jsonStr);
		//递归循环JSON中的每个元素（包括{、}、[、]等）
		while (parser.nextToken() != JsonToken.END_OBJECT) {
			System.out.println(parser.getText());
		}
```

##### 2、数据绑定

​	Jackson提供com.fasterxml.jackson.databind.ObjectMapper对象，来完成所有数据绑定，实现javaBean和JSON的转换

```java
		ObjectMapper objectMapper = new ObjectMapper();
		//读取json字符串，转换为JavaBean(支持常用集合List、Map)
		String jsonStr = "{\"name\":\"YH\",\"id\":\"12345\"}";
		HashMap hashMap = objectMapper.readValue(jsonStr, HashMap.class);

		//将javaBean解析为json字符串
		String valueStr = objectMapper.writeValueAsString(hashMap);
		System.out.println(valueStr);
```

​	注意：

- ObjectMapper线程安全，并可以重复使用

##### 3、树模型API

​	Jackson提供树模型api，用于优化解析获取较大json中的某个字段的场景。这样可以避免对整个json字符串进行解析，减少不必要的性能损失

```java
		// json字符串转化为jsonNode（json节点，每个节点都有自己对应的值）
		String json = "{\"name\":\"YH\",\"id\":\"54321\"}";
		JsonNode readTree = objectMapper.readTree(json);
		//获取当前树节点下的字段数据
		JsonNode jsonNode = readTree.get("name");
		String asText = jsonNode.asText();
		System.out.println(asText);
```

注意：

- 由于树模型使用繁琐，一般只用于解析大型json字符串；而不会用于JSON字符串的生成

#### 2、Jackson常用注解

**常用注解：**

| 注解                  | 作用                                                         |
| --------------------- | ------------------------------------------------------------ |
| @JsonProperty         | 序列化、反序列化时，java对象属性名映射json的字段名           |
| @JsonIgnore           | 忽略该字段的json处理                                         |
| @JsonIgnoreProperties | 注解在类上，忽略多个字段的json处理                           |
| @JsonInclude          | 注解在字段或类上，定义序列化条件（always、non_default、non_empty、non_null） |
| @JsonFormat           | 时间类型字段的序列化和反序列化的格式转换                     |

#### 3、Jackson的序列化和反序列化

##### 1、序列化：

1、创建JsonGenerator生成器，并加载序列化配置信息；

2、获取序列化类的class对象；

3、根据class对象，创建对应的jsonSerializer序列化器：

- 扫描class对象的所有Field字段（包含static、final字段），添加到property列表中
- 扫描class对象中的所有get方法，找到对应的property，当做其get方法;如果没有找到，则创建一个新的property，并作为它的get方法
- 处理class对象的类、字段注解，标记到property中
- 遍历property列表，对应处理相关注解在property所做的标记（如，@JsonIgnore会在property中标记为ignore，此时就会将该property移除）

3、JsonGenerator使用流式API，调用JsonSerializer的序列化方法，进行JSON对象的生成

**结论：**

​	**Jackson序列化依赖于get方法，即使某个get方法没有对应的属性，也会被序列化**

##### 2、反序列化

1、创建JsonParser解析器，并加载反序列化配置信息

2、通过指定的class对象，获取相关信息

3、通过JsonParser解析json字符串，获取所有kv键值对，进行java对象的数据绑定：

​	1、通过无参构造方法，创建java对象实例，并获取该对象中所有可反序列化的property列表

​	2、获取当前解析的KV键值对，根据key找到对应的java对象属性（没有，则跳过该KV对的数据绑定）

​	3、基于java对象反射，将value注入到java对象的属性中

**结论：**

​	**Jackson反序列化依赖于无参构造器和JAVA反射,并会对于带有get/set方法的私有属性进行反序列化**

**注意：**	

​	对于一个javaBean，Jackson默认只会对所有public字段，进行序列化和反序列；private字段需要添加get/set方法，来控制序列化和反序列化的权限

#### 4、Jackson的模块引入

​	Jackson以模块的形式进行更新，来升级支持各种常用java库数据类型，常用模块入下：

- jackson-module-parameter-names：

  ​	用于支持JDK8中，通过获取构造函数和构造器方法中的参数，来定义JSON反序列化时，和类字段名的映射关系，从而无需使用@JsonProperty注解来改变映射

- jackson-datatype-jsr310：

  ​	用于支持JDK8中的日期/时间类型

- jackson-datatype-jdk8：

  ​	用于支持JDK8中，除日期/时间外，新增的Optional包装类

#### 5、ObjectMapper

​	Jackson对外，基于ObjectMapper来实现JSON的解析操作，开发者可以通过配置该对象的相关属性，来改变JSON序列化和反序列化的规则：

```java
ObjectMapper objectMapper = new ObjectMapper();
objectMapper.setTimeZone(TimeZone.getTimeZone("GMT+8:00"));
//指定date类型的序列化与反序列化处理
objectMapper.setDateFormat(new SimpleDateFormat(DEFAULT_DATE_TIME_FORMAT));
//Jdk8时间日期类模块插入
JavaTimeModule javaTimeModule = new JavaTimeModule();
objectMapper.registerModule(javaTimeModule);

//序列化参数配置
objectMapper.configure(SerializationFeature.WRITE_ENUMS_USING_TO_STRING, true);
objectMapper.setSerializationInclusion(JsonInclude.Include.ALWAYS);
```

## 5、junit4	

​		**java单元测试是最小的功能单元测试代码，即针对单个java方法的测试（java的最小单位为方法）**

### main方法进行测试的缺点：

- 只能有一个main（）方法，不能把所有测试代码按一个个方法分开
- 无法打印出测试结果和预期结果

### 单元测试的优点：

- 可以打印测试报告
- 可以进行自动测试，进行执行结果的判断
- 可以在一个类中，书写多个测试方法，并且每个单元测试都可以独立执行

### JUnit4框架，java最常用的测试框架：

1、JUnit4框架有标准规定，测试类必须在src/test/java的目录中；maven项目就满足该目录结果

并且，**测试类不能使用Test命名，否则会出现导入的org.junit.Test类名冲突**

2、编写一个测试方法，只需要使用@Test注解，并且保证方法满足以下条件：

- 无返回值
- 无参数
- public

```java
@Test
public void Test(){
    
}
```

2、对单元测试进行断言（Assertion），即判定程序的执行结果是否符合预期，从而通过测试

使用assertEquals（obj，obj）、、、等方法来对结果进行断言

3、@Before、@After、@BeforeClass、@AfterClass使用，用于指定单元测试执行前后的代码（如资源初始化和释放）

**测试类执行单元测试的过程：**

- 初始化类的静态变量，执行一次@BeforeClass方法（该方法必须满足：无返回值、无参数、静态方法、public）
- 初始化类的成员变量，执行测试类的构造方法
- 执行@Before方法（和@Test方法一致）；然后执行@Test方法；最后执行@After方法
- 当执行测试类时，会自动从上到下执行所有@Test方法，因此相应也会多次执行初始化类的成员变量，执行测试类的构造方法、@Before和@After方法
- 最后在执行一次@AfterClass方法

**在执行测试类时，每一次执行@Test方法前，都是重新创建了一个测试类实例，因此测试类中的非静态成员变量，不能在每个@Test中共享，而是每次都为初始值**

**因此在多个@Test方法需要使用某个资源时，为了减少资源初始化和释放时间，一般将其初始化代码和释放代码放在@BeforeClass方法和@AfterClass中**

4、异常测试和超时测试；

@Test注解提供expected属性，用于指定异常类的Class对象，当未抛出该异常类型时，则测试失败；反之测试成功

@Test注解提供timeout属性，用于指定测试方法执行时间，单位为毫秒；当测试方法执行时间超出该值时，测试失败

5、忽略测试：

@Ignore，注解在已被@Test注解的方法上，用于在执行测试类时，忽略被注解的@test方法

6、使用不同运行器进行测试：

@RunWith（Calss），指定该测试类单元测试使用的运行器，如springboot测试类注解为@RunWith(SpringRunner.class)

不同的运行器，单元测试的规则和注解也有所不同，Junit默认运行器为 org.junit.runner.Runner

## 6、logback

推荐使用Slf4j+Logback,相对于Log4j有更多的优点、特性和性能

### **1、日志级别：**

| 日志级别 | 描述                                               |
| -------- | -------------------------------------------------- |
| OFF      | 关闭：最高级别，不输出日志。                       |
| FATAL    | 致命：输出非常严重的可能会导致应用程序终止的错误。 |
| ERROR    | 错误：输出错误，但应用还能继续运行。               |
| WARN     | 警告：输出可能潜在的危险状况。                     |
| INFO     | 信息：输出应用运行过程的详细信息。                 |
| DEBUG    | 调试：输出更细致的对调试应用有用的信息。           |
| TRACE    | 跟踪：输出更细致的程序运行轨迹。                   |
| ALL      | 所有：输出所有级别信息。                           |

一般情况下，使用 ERROR、WARN、INFO 、DEBUG四种级别

### 2、logback框架配置

1、java -Dlogback.configurationFile= /path/mylogback.xml  ;通过指定java执行命令，来加载自定义的logback配置文件

2、当没有指定logback.configurationFile参数时，默认在calsspath下按顺序查找：

- logback.groovy
- logback-test.xml
- logback.xml
- JDK6.0以上，自动查找com.qos.logback.classic.spi.**Configurator**接口的第一个实现类

 3、都没有找到时，使用**ch.qos.logback.classic.BasicConfigurator**类默认logback配置，将日志在控制台输出

logback.xml常用配置：

- configuration  配置

```xml
 <!-- scan 配置文件改变后，是否重新加载
      scanPeriod 检测配置文件是否发送改变的时间周期，默认为毫秒，可以自定义单位
      debug 是否打印logback内部日志（一般不需要）
  -->
<configuration scan="ture" scanPeriod="60 seconds" debug="false">
</configuration>
```

- property  全局变量

```xml
<!-- 定义参数常量，用于在日志组件中使用，达到全局变量的作用-->
<property name="LOG_FILE_PATTERN"  value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level ${PID:- } --- [%15thread]  %-50logger{49} : %msg%n"/>
```

- appender  日志输出器

```xml
<!-- 定义日志在文件中输出的组件
使用多个组件来定义不同等级的日志文件输出，一般使用四个等级：DUBUG、INFO、warn、ERROR-->

<!--输出控制台-->
<appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
         <pattern>${LOG_CONSOLE_PATTERN}</pattern>
    </encoder>
</appender>

 <!--输出到文件,info-->
<appender name="infoFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <!-- 滚动策略,每天生成一个文件，指定文件名、文件保存时间(一般情况下，不推荐直接使用file标签，将日志输出到一个文件中，达到指定大小后再切分) -->
	<rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
		<fileNamePattern>
        	${LOG_FILE_PATH}/{PROJECT_NAME}/info/{PROJECT_NAME}_info_%d.log}			         </fileNamePattern>
        <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
    </rollingPolicy>
        <encoder>
            <pattern>${LOG_FILE_PATTERN}</pattern>
        </encoder>
        <!-- ThresholdFilter过滤打印指定级别日志, levelFilter过滤打印该级别及以上日志-->
     <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
         	<
     </filter>
    </appender>
```

- root 定义全局日志

```xml
<!-- 定义全局logger，当前类没有找到对应logger时，使用root全局设置输出 -->
<root level="info">
     <appender-ref ref="consoleAppender"/>
</root>
```

- logger 定义日志

```xml
<!-- 定义单个logger，name用于匹配类的包名或指定的loggerName,匹配则使用当前logger进行日志输出，否则使用全局logger
logger默认是继承使用root中的Appender组件;为了放在日志重复输出，添加additivity="false",不继承root中的Appender组件-->
<logger name="testLogger" level="debug" additivity="false">
    <appender-ref ref="debugAppender"/>
</logger>
```

### 3、logback启动原理

- 使用slf4j接口的好处：基本所有日志框架，都可以作为其实现层，因此我们可以面向该接口进行编程，通过切换slf4j的实现，来简单完整实现底层日志框架的切换

- logback启动过程：

  调用slf4接口的方法：LoggerFactory.getLogger(Test.class)；此时slf4j调用org.slf4j.StaticLoggerBinder进行初始化，就会扫描项目中所有`org.slf4j.StaticLoggerBinder`的具体实现；

  而logback-classic中就定义了StaticLoggerBinder的实现方法，此时就会加载logback.xml

### 4、logback日志使用要点

- 定义logger变量

```java
private static final Logger logger = LoggerFactory.getLogger(XX.class);
要确保一个类中，只有一个logger对象（静态成员常量），从而减少对象的内存占用
```

- 在生产环境中，应该禁止debug级别日志的输出

  由于logger会继承root的输出源，因此一般需要添加additivity="true"，防止日志重复输出

- error日志信息输出时，应该携带异常信息，帮助分析异常原因

```java
		try {
			int a= 1/0;
		} catch (Exception e) {
			log.error("test发生异常，异常信息："+e.getMessage(),e);
		}
```

- 减少日志打印对应用程序性能的损耗

  虽然我们在生产环境下，会提高日志级别的打印，减少不必要的日志打印；但是日志中的字符串拼接还是会执行，这样会白白浪费性能，因此需要以下手段来进行这样情况的发生：

  1、使用条件判断，判断当前logger对象的需要输出的日志级别，再对日志进行打印后者忽略

  ```java
  	if (log.isErrorEnabled()) {
  		log.error("test发生异常，异常信息："+e.getMessage(),e);
  	}
  ```

  2、使用占位符，来减少字符串拼接时，String对象的创建；使用占位符时，只有需要打印该字符串时，才会进拼接（有些日志框架不支持占位符,logback支持）

  ```java
  	int id=100;
  	String user="yh"; 
  	log.debug("错误，id：{},user:{}",id, user);
  ```

- 日志文件命名：推荐使用projectName_logName_logType.log

### 5、springboot默认配置和推荐配置

**1、springboot整合logback时，提供logback默认配置：**

​	全局日志级别为INFO，不进行日志文件输出（通过logging.logback包下的base.xml、console-appender.xml、defaults.xml、file-appender.xml 配置文件提供实现）

可以通过springboot主配置文件，来对默认配置进行修改：

- logging.level.* 可以改变指定包名下的日志级别，创建单独的logger

- logging.level.root 可以改变全局日志级别

- logging.file  设置日志文件生成路径（可以是相对路径或绝对路径）

- logging.path 设置日志文件目录（在项目根目录下生成）

  logging.file优先级大于logging.path，默认情况下，日志文件大于10MB时，会产生新文件

当需要自定义logback配置时，springBoot推荐使用logback-spring.xml命名，这样可以实现对spring-profile功能的支持：

在logback-spring.xml内部，使用**springProfile标签**对root、logger标签进行多环境配置

**2、一般情况下，推荐自定义logback配置文件，基于阿里规范，日志配置推荐如下：**

```xml
<?xml version="1.0" encoding="UTF-8" ?>

<configuration scan="true" scanPriod="60 seconds" debug="false">
    <!--引入springboot提供了关键字解析器-->
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>

    <property name="LOG_FILE_MAX_HISTORY" value="30 day"/>
    <property name="LOG_FILE_PATH" value="D://log"/>
    <property name="PROJECT_NAME" value="system"/>
    <!--时间、日志级别、PID、线程、日志名(日志创建使用的class全类名)、msg-->
    <property name="LOG_CONSOLE_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} %clr(%-5level){green} %clr(${PID:- }){magenta} --- [%15thread]  %clr(%-50logger{49}){cyan} : %msg%n"/>
    <property name="LOG_FILE_PATTERN"
              value="%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level ${PID:- } --- [%15thread]  %-50logger{49} : %msg%n"/>

    <!--输出控制台-->
    <appender name="consoleAppender" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>${LOG_CONSOLE_PATTERN}</pattern>
        </encoder>
    </appender>

    <!--输出到文件,info-->
    <appender name="infoFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 滚动策略，log.filePathTimeBasedRollingPolicy,每天生成一个文件，指定文件名、文件保存时间 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${PROJECT_NAME}/info/${PROJECT_NAME}_info_%d.log}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_FILE_PATTERN}</pattern>
        </encoder>
        <!-- 过滤打印指定级别日志 -->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
    </appender>

    <!--输出到文件,warn-->
    <appender name="warnFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${PROJECT_NAME}/warn/${PROJECT_NAME}_warn_%d.log}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_FILE_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>WARN</level>
        </filter>
    </appender>

    <!--输出到文件,error-->
    <appender name="errorFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${PROJECT_NAME}/error/${PROJECT_NAME}_error_%d.log}</fileNamePattern>
            <maxHistory>${LOG_FILE_MAX_HISTORY}</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${LOG_FILE_PATTERN}</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
    </appender>

    <!--全局日志级别为info,当springboot中指定额外的logger时，默认继承root所有组件-->
    <springProfile name="dev,test">
        <root level="info">
            <appender-ref ref="consoleAppender"/>
        </root>
    </springProfile>

    <springProfile name="prod">
        <root level="info">
            <appender-ref ref="consoleAppender"/>
            <appender-ref ref="infoFileAppender"/>
            <appender-ref ref="warnFileAppender"/>
            <appender-ref ref="errorFileAppender"/>
        </root>
    </springProfile>

</configuration>
```

springboot默认配置和开发者自定义配置可以同时存在，但自定义配置可以覆盖默认配置：

​		一般情况下，我们只修改springboot默认配置中的logging.level.*，用于创建新的logger，来进行局部的日志打印；而新的logger会继承自定义的全局root日志组件，进行相同方式的日志输出

## 7、HuTool

见官方文档