# Effective Java（高效java）

## 1、创建与销毁对象

### 1.1、使用静态工厂方法代替构造器

- **静态工厂方法相对于构造器的优势：**

  - 构造器没有方法名，只能通过参数列表进行区分；而静态工厂方法可以通过方法名准确有效的表达该方法所返回的对象类型

  - 可以在静态工厂方法内部添加额外逻辑：

    - **保证对象的单例**，实现单个对象的共享
    - **使返回的对象实例向上转型**，实现面向接口编程，有效减少开发者学习API的数量

    - **通过参数列表，返回同一父类的各种子实现类**
    - **通过字面量来实例化对象**，使用反射来实例化，在工厂中实现类的实例化与代码的解耦，从而在编写静态工厂方法时，并不需要实现类存在（JDBC中，DriverManager.getConnection（）就使用了这个）

- **静态工厂方法名的常用规范，基于JDK8中API的命名规范：**

  from：类型转换方法，通过单个参数获取对象实例

  of：聚合方法，将多个参数聚合为一个对象

  valueOf：类型转换方法，类似于from，一般用于和java数据类型有关的方法

  instance、getInstance：通过无参或有参方式获取当前对象实例

  create、newInstance：通过无参或有参方式获取当前对象实例，并保证每次方法调用都是一个新对象

  XXX/getXXX：获取指定对象类型实例

  newXXX：获取指定对象类型实例，并保证每次方法调用都是一个新对象

- **总结：**

  在一般情况下，优先考虑公共构造器，当对象实例构建满足静态工厂方法的某些优点时，再考虑使用静态工厂方法

### 1.2、遇到多个构造器参数时，考虑使用构建器

​	静态工厂方法和构造器有一个共同的局限性，不能很好的满足大量可选参数的对象构建。一般有两种方式来解决：

- 重叠构造器：（在很多框架中使用）

  提供一个满足全部参数的构造方法，然后从必要参数的构造器开始，逐层增加可选参数，并在内部统一调用使用全部参数的构造方法，没有的参数给与默认值：

  ```java
  public calss test{
      private String A；
     	private	String B;
      private String C;
      public test（String A，String B，String C）{
          this.A=A;
          this.B=B;
          this.C=C;
      }
      
      public test(String A){
          this.test(A,"","");
      }
      publci test(String B){
          this.test(A,B,"");
      }
  }
  ```

  **缺点：提高了类构造方法的编写难度、降低的可阅读性**

- javaBeans模式：（在MVC的model层使用）

  将创建无参对象，然后通过setter方法将每个参数传入对象中

  **优点：相对于重叠构造器，代码逻辑简单，阅读性强**

  **缺点：将对象的构造过程分到了多个setter方法中完成，在多线程环境下，就会有线程安全问题**

**以上两种方式各有利弊，因此在这种情况下，推荐开发者使用构建器来创建对象：**

提供一个Builder对象来封装目标对象所有的参数，然后提供一个方法来创建目标对象，并将参数传入

```java
public class TestBuilder {
	private final String A;
	private final String B;
	private final String C;

	private TestBuilder(Builder builder) {
		this.A = builder.A;
		this.B = builder.B;
		this.C = builder.C;
	}

	public static class Builder {
		private final String A;
		private String B;
		private String C;

		public Builder(String A) {
			this.A = A;
		}

		public Builder B(String B) {
			this.B = B;
			return this;
		}

		public Builder C(String C) {
			this.C = C;
			return this;
		}

		public TestBuilder build() {
			return new TestBuilder(this);
		}
	}
}
```

搭建使用注意：

- 目标类(外部类)，所有属性为final，从而保证只能被构造方法初始化；同理Builder中的必选属性为final，只在器构造方法中初始化
- 目标类构造方法为private，保证只能在内部调用实例化对象
- Builder（内部类）每个构建方法都返回当前this，实现链式方法调用
- Builder使用静态内部类，方便外部类构造方法直接获取内部类对象中的私有属性；同时符合类设计的单一职责原则（Builder是目标类的一部分，只是为目标类服务）

**构建器优缺点：**

- 优点：既保证了对象创建的安全性，有保证了代码的可读性和创建对象的方便；同时为每个可选参数的注入都提供一个方法调用，简单方便的实现数据校验
- 缺点：内部代码较多，并需要添加Builder对象带来额外开销

**综上所述：当类的构建有可选参数，并且参数总数大于5个时，推荐使用**

### 1.3、用私有构造器或枚举类型强化Singleton属性

​	Singleton（单例类）最常用的实现方式有两种：

- 直接提供一个实例化的静态不可变对象

  ```java
  public class Singleton{
      public static final Singleton singleton = new Singleton();
      private Singleton(){}
  }
  ```

- 通过静态工厂方法返回一个唯一不变的对象

  ```java
  public class Singleton{
      private static final Singleton singleton = new Singleton();
      private Singleton(){}
      public static Singleton newInstance(){
          return singleton;
      }
  }
  ```

**两者对比：**

- 相同点：两者都是使用私有构造器，来限制对象的实例化只能在内部完成，避免对象在外部被创建
- 不同点：后者通过静态工厂方法，从而实现对singleton对象的额外处理，通过可以对其进行改造，使singleton的创建变为懒加载（通过双重锁定来实现线程安全）

**通过枚举，可以由第三种方式来实现Singleton：**

​	对于以上两种方式，都存在一个问题：**都只是通过私有构造器来实现单例，无法避免使用反射和反序列化来创建一个新对象**

​	我们可以利用枚举，通过JVM来保证单例和线程安全：

```java
public enum Singleton {
    INSTANCE;
}
```

需要实现singleton的类直接继承Enum，并提供一个实例对象INSTANCE，在保证私有构造方法同时，提供序列化机制、并且通过JVM保证除自身提供的实例对象外，不允许创建其他实例，避免反射创建新实例

因此，**单个元素的枚举类型是实现单例的最好方式**

### 1.4、通过私有构造器强化不可实例化的能力

​	对于一些工具类，只需要提供静态方法、静态域，并不需要提供实例对象，而编译器会默认添加一个无参构造器，因此我们可以提供手写一个私有构造器，来确保类不能被实例化，方便API的使用

​	副作用：该类无法被继承子类

### 1.5、优先考虑依赖注入来引用资源

​	对于一个类需要引用其他资源时，这个资源是不确定的，存在多种实例，此时就要考虑使用依赖注入，来保证资源使用的灵活性,并确保该类每个实例，都有一个确定的资源：

​	通过构造方法，将需要引用的资源注入，并完成资源校验过程;

```java
public class Test{
    private final String data;
    
    public Test(String data){
        check(data);
        this.data=data;
    } 
}
```

### 1.6、避免创建不必要的对象

1、String对象的创建直接使用 String s =”123“，而不是使用String s= new String("123")，因此String构造方法实际上创建了2个对象

2、对于基础数据类型包装类，同样不要使用构造器Interger（String s）进行创建，而是使用直接赋值获取Interger .valueOf(String s)，保证不可变对象的重用，即包装类可以使用常量池

3、保证不会变对象的重用，如String.matcher（）方法，内部每次调用都会创建一个新的Pattern实例，如果需要使用同一个正则表达式，则应该显式创建Pattern实例，使用该对象的matcher方法来匹配

4、优先使用基础数据类型，不要进行无意识的装箱操作，因此每一次装箱在常量池外都会创建一个新的包装类

### 1.7、消除过期的对象引用

### 1.8、避免使用终结方法和清除方法

​	java提供finalizer（终结方法）、cleaner（清除方法，JDK9提供）来标记对象进行垃圾回收，但是他们都是不可预测的，并且存在性能问题，无法保证对象被立即回收

### 1.9、try-with-resources优先于try-finally

​	java类库中，存在需要需要手动关闭资源的close方法，如InputStream、java.sql.Connection等；在JDK7之前，我们使用try-catch-finally来保证及时方法发生异常，也能够正常调用close方法来关闭资源。但这种方式存在一些问题：

- 代码繁琐,在使用close方法时，同样需要进行try-catch保证资源关闭时的异常捕获，在存在多个资源使用时，就会有很多嵌套，减低代码的可阅读性
- 在try、和finally中都会对资源进行异常捕获，这样就会导致catch的异常信息被fianlly中的异常信息抹除，让系统的调试变的复杂

​	在JDK7后，引入了try-with-resources语法糖，直接在try（）中创建资源，然后在{ }中编写资源使用的逻辑代码。但是需要**保证资源实现AutoCloseable接口**

```java
try(BufferedReader br = new BufferedReader(new FileReader("D://xxx.txt"))){
    br.readLine();
}catch(Exception){
    xx
}
```

​	可以发现try-with-resources直接省略了对资源finally中的close方法调用，使代码变得非常简洁。同时，当readLine（）方法和隐藏的close方法都发生异常时，会保留第一个异常信息，避免被其他后续的异常覆盖。从而保证开发者能够快速找到导致资源使用异常的信息

## 2、对于所有对象都通用的方法

### 2.1、覆盖equals时请遵守通用约定

​	Obejct为所有类提供equals方法：

```java
  public boolean equals(Object obj) {
        return (this == obj);
    }
```

该方法保证了**每个类的实例都是唯一的，不存在逻辑相等**

​	但在实际需求过程中，需要逻辑相等的时候，比如比较值类（String、Integer）是否相等，此时就需要覆盖equals，而在覆盖equals方法时，需要遵守通用约定：

- 自反性：非null对象x，x.equals(x)必须返回true
- 对称性：非null对象x，y；x.equals(y)和y.equals(x)的结果一致
- 传递性：非null对象x、y、z；x.equals(y)为true、y.equals(z)为true，则x.equals(z)必须为true
- 一致性：x.equals(y)多次调用的结果一致，其方法不会对x、y对象进行修改
- 非空性：对于任何非null对象x，x.equals（null）必须返回false

**实现高质量的equals方法编写：**

1、使用==操作符，检查参数是否为该对象的引用，是，则直接返回true，从而提高判断效率

2、使用instanceof操作符，检查参数是否和该对象为同一个类型，避免进行类型转换时报错，同时还可以避免显式对参数进行null的检查

3、将参数进行强转，从而进行接下来的逻辑相等判断（Object原方法，入参参数为Object类型）

4、对类中每个”关键域“进行匹配检查，如果全部匹配则返回true：

一般情况下，这些关键域就是基本数据类型和引用数据类型的比较：

- 对于非float、double的基础类型，可以直接使用==；
- 而float、double则使用静态Float.compare(float，float)方法，因此浮点数存在-0.0、NaN常量值；而Float.equals（Object ）方法需要进行自动装箱，影响效率

- 对于引用对象类型，则调用其equals方法

以String类的equals重写为例：

```java
   public boolean equals(Object anObject) {
        if (this == anObject) {   //第一步
            return true;
        }
        if (anObject instanceof String) {	//第二步
            String anotherString = (String)anObject;  //第三步
            int n = value.length;	//第四步，先比较长度
            if (n == anotherString.value.length) {//再比较每个字符是否相同
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
```



**注意：**

- 不要修改equals方法参数列表，这样就不是覆盖，而是重写了

- 覆盖equals时，总要覆盖hashCode，原因看下一张

### 2.2、覆盖equals时总要覆盖hashCode

​	Object类提供hashCode（）方法，并遵循如下规范：

```java
  public native int hashCode();
```

- 只有对象的equals方法比较操作的信息（关键域）没有修改，则hashCode方法返回值永远不会改变
- 如果两个对象equals方法比较相等，则hashCode方法返回值一定相等
- 如果两个对象equals方法比较不相等，hashCode方法返回值也可能相等，但为了减少hash冲突，程序员应该降低冲突率

**HashCode设计的意义：**

​	用于配合基于散列（HASH）的集合一起运行，包括HashSet、HashMap等；在集合中，的确可以通过equals方法逐一比较来获取指定数据，但是运行效率会变成平方级别。而HashCode方法就是通过获取散列（hash）值，在hash表中直接找到对应的数据。

**Object对象HashCode方法的底层算法规则：**

​	通过一定规则，对对象的相关信息（对象的存储地址、对象的字段信息等）进行算法计算，最后获取到一个int值，也因此所有对象的HashCode并不是唯一的，可能出现Hash冲突	

当我们覆盖equals方法后，则第二条规范就会失效，因此HashCode也就失去的它本身的意义：**通过比较hash值来代替equals方法，来确定对象应该存放在当前集合中的位置**

**实现高质量的HashCode方法编写：**

1、由于需要保证equals方法相等的两对象，它们的HashCode一定要相等，因此HashCode的计算方式必须与对象的关键域相关，并且保证对象关键域不改变时，HashCode结果值永远不变

2、关键域各种类型的计算方式：

- 基础数据类型：使用Type.hachCode（f）
- 引用数据类型：递归调用f.hashCode(),如果对象为null，则返回一个常量如0
- 数组：则按顺序计算每个Hash值，然后进行Hash值合并

3、Hash值合并公式：

```java
int result = 31*result +C
```

result为合并的结果值，c为当前需要合并的Hash值

- 为什么不是单纯的加法：

  如果是单纯加法，那么元素的顺序就不会成为Hash值区分的因素，如数组[1,2,3] 和[2,1,3],就会导致他们的Hash值相同，但实际上equlas方法返回false

- 为什么是乘以31：

  31是一个奇素数，如果乘数是偶数，溢出就会导致有效信息丢失，因为*2相当于位移运算。而素数能够是Hash表中数据的均匀分布，减少hash冲突。而31在100以内的奇素数中，大小刚好，减少hash冲突和超出int范围；同时有个关键点：    31*i=（i<<5）-i  可以是该公式运算简化为位运算和减法，提高性能。但在现代化虚拟机中，可以对乘法自动优化

4、Hash值的重用

​	对于不可变对象，Hash值是可以重用的，这样有效减少的HashCode的运算开销

**目前IDE和相关插件可以自动化重写equals和HashCode方法**

### 2.3、始终要覆盖toString

​	Object为所有类提供toString（）方法

```java
   public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }
```

​	用于获取类的全类名@HashCode的16进制，这样的表达方式并不能完整表现当前对象的信息，因此推荐开发者所有子类都需要覆盖toString方法

​	覆盖toString方法的好处：

- 在println、+、assert方法内部，都自动调用的参数的toString方法，这样就能更加简便的打印对象信息
- 在覆盖toString方法时，可以方便对打印信息进行格式化处理，方便查看

### 2.4、谨慎地覆盖clone

​	Object为所有类提供了clone（）方法

```java
    protected native Object clone() throws CloneNotSupportedException;
```

​	它是一个本地方法，必须搭配Cloneable接口使用，用于提供对对象的逐域拷贝：

- Cloneable接口：

  ​	它并没有提供任何方法，但在底层实现了对Object.clone()方法的开启，对于没有继承Cloneable接口的类，其对象调用clone（）方法时，会抛出CloneNotSupportedException异常（不支持clone），因此Cloneable接口就是开启Object.clone（）方法的使用

- clone（）方法：

  ​	该方法可以创建一个对象，提供java语言之外的一种构建对象的方式，其拷贝规范为：

  ```java
  x.clone（）！=x				//两者不为同一对象
  x.clone（）.getClass==x.getClass()	//两者为同一个类型
  x.clone().equals(x)			//不强求，但通常是相等的
  ```

- clone（）方法的缺陷：

  ​		类的域中存在非final类对象（引用可变）时，通过原对象得到的克隆对象，它们两者的该域值会使用同一个对象实例，因此它们对自身域的操作都会互相影响，导致失去了clone（）方法本身的意义
  
  ​		为了避免这种情况发生，则需要对**clone（）方法进行重写：**
  
  ​		重写clone方法，将其转换为public，保证递归克隆类中每一个域及其子级的不可变实例，并将其放入克隆对象中；但需要保证该域非final，因此final不能够重写赋值
  
  **注意：**
  
  ​		对于hashMap对象，他内部使用了hash桶数组来保存相同hash值的数据，此时单纯clonehashMap并不能进行深度clone，导致原对象和clone对象直接存在不确定行为

- clone（）重写的替代

  ​		通过对clone（）方法的重写，能进行对象的深度clone，但是对于复杂对象相对复杂，需要手动对部分域进行修正，因此clone方法并不太好使用

  ​		**在开发中，推荐使用拷贝构造器和拷贝工厂来完成指定复杂对象的clone**

### 2.5、考虑实现Comparable接口

​		JDK提供Comparable接口，来定义一个类的比较方式：

```java
public interface Comparable<T>{
    public int compareTo(T o);
}
```

- Comparable接口实现要求：

1、通过泛型来保证，只能进行相同类型对象的比较

2、必须判断<、= 和 >三种场景，分别返回-1、0、1

3、比较需要有传递性、对称性、自反性

## 3、类和接口

### 1、使类和成员可访问性最小化

​		一套API设计的是否合理，关键在于封装的隐藏性，隐藏内部数据和实现细节，解除系统各模块中的耦合关系，方便开发这快速理解、开发和运维              

​		Java中体提供访问控制机制，来决定类、接口、成员变量的可访问性，实现信息隐藏：

- 对于顶层（即非内部类）类和接口，只有两种访问等级：包私有（default）、公有（public）

- 当某个类或接口只在一个类的内部使用时，应该考虑将其写为私有内部类，降低其可访问性
- 类成员有四种访问等级：私有（private）、包私有（default）、受保护（protected）、公有（public）

除公有静态final域外，共有类都不应该有共有域，并且要确保公有静态final域对象是不可变的

### 2、应该在公有类中使用访问方法来代替使用公有域

​		在公有类中，不应该暴露数据域，否则会导致无法对数据域的访问进行修改和限制，降低数据域访问的灵活性；因此推荐使用getter、setter方法来代替使用公有域

​		对于不可变域（final），由于数据域无法被修改，因此允许暴露，无需额外使用访问方法

### 3、使类的可变性最小化

​		不可类相对于可变类，在使用、设计和实现上更加简单、安全

- 不可变类实现规则：
  - 不提供任何设置对象状态的方法（设值方法）
  - 类不能够被继承，即添加final关键字
  - 声明所有域都是final的，通过JDK机制来现在对象状态不可变
  - 声明所有域private，防止直接修改被域引用的可变对象
  - 确保类所有方法，都无法访问可变对象域

- 不可变类对象的优缺点：

  优点：

  - 不可变类本质上是线程安全的，因此推荐使用不可变类对象作为常量，被共享

  缺点：

  - 每个不可变类对象都表示一个唯一的值，因此每个值都需要一个单独的对象，而为了操作这个对象，就需要创建多个中间值，如String，因此JDK提供了对应的操作对象StringBuilder，来解决这个问题（注意StringBuilder并不能保证线程安全）

  **对于某些类，将其设计为不可变性是不存在的，但我们应该尽量降低其可变性，从而更好的分析该对象的行为，降低出错的可能性**

### 4、复合优先于继承

​		继承打破了类的封装性。子类依赖于超类进行功能实现，当超类发生改变时，我们就需要维护子类，否则可能出现子类存在未知方法，将父类进行方法重写；并且在使用上，需要更加详细的文档说明。

​		复合，在当前类中新增一个私有域，来保存需要代码复用对象，让其变为当前类中的组件，进行使用；此时代码复用类不会被当前类的变化所影响，从而让当前类更加稳定。

- 在什么时候使用继承：

  ​		对于两个类A和B，只有两者存在is-a关系时，才考虑使用继承来实现两者的代码复用，并且是否允许将父类的细节暴露给子类

### 5、使用继承时，必须提供文档说明

​		使用继承存在一定的风险，因此必须提供文档说明，精确描述父类中每个方法被覆盖时，所带来的影响、需要提供实现的功能和该方法何时被调用

​		因此，设计父类时，应该精心挑选protected方法，用于子类对其重写，提供改变父类行为的入口；

继承使用时注意：

- 父类构造器决不能调用可被覆盖的方法，原因：

  ```java
  public class Super{
      public Super(){
          A();
      }
      public void A(){
          
      }
  }
  
  public class Sub(){
      private final String name;
      
      public Sub(){
          name="Sub";
      }
      
      @Override
      public void A(){
          System.out.println(name);
      }
  }
  
  public static void main(String [] args){
      Sub sub = new Sub;
      sub.A();
  }
  ```

  当执行main方法时，在Sub被加载时，方法表中提供被重写的A方法，而执行父类构造方法时，就会调用重写的A方法，打印name，但此时name还没有初始化，因此打印null

### 6、接口优于抽象类

​		JAVA提供两个机制：接口和抽象类，来实现类的多态。但在JDK8推出默认方法（default method）后，由于JAVA只支持单继承，因此在大部分情况下，接口优于抽象类

### 7、尽量避免利用默认方法，为接口新增方法

​		虽然JDK8提供的默认方法，避免了现有接口新增方法时，其实现类在编译时不会保存；但是我们不易控制在运行时，默认方法给接口实现类带来的影响；因此在接口新增方法时，应该主动对每个实现类进行方法重写，控制新增方法在实现类中的表现

### 8、接口只用于定义类型

​		有一种接口叫常量接口，接口只用于声明一系列常量；而这种使用方式是对接口的不良使用，接口只是用于定义类型；当接口默认方法或接口实现类统一使用某个常量时，才会在接口中定义常量（接口中，成员变量默认为static final）

### 9、静态内部类优于非静态内部类

​		静态内部类实际上就是一个普通类，只是作为内部类嵌套其中，在使用上没有本质的区别；但静态内部类能直接访问外围类的私有属性，大部分情况下：

**如果内部类不需要访问外围类实例时，则直接使用静态内部类定义**

### 10、单个源文件只写一个顶级类

​		在java中，允许一个java文件中，声明多个顶级class（但public修饰的只有一个）；但这种写法并没有实际意义，并且当两个java文件中，同时定义相同名的class时，将会编译报错；

​		因此一个源文件中，只能写一个顶级类，需要将两个class定义在一起时，则选择静态内部类代替

## 4、泛型

### 1、不要使用原生态类型

​		在JDK5推出了泛型后，原生态类型就没有了实际使用意义（除了兼容JDK5之前的代码）；因为使用泛型可以有效在编译期，检查容器元素类型不同的错误，保证了代码的安全性和可描述性（告诉开发者，当前容器是用于存储何种类型）

​		当我们需要创建一个未知参数类型的容器时，可以使用List<Object> 或List<?>,但两者有本质的区别：

- List<Object>

  ​		它是一个参数化泛型，可以存放任何类型对象，因为Object是所有类的父类；但是在存放时，会自动将元素强转为Object，当我们取出了，就需要手动强转还原，因此我们需要知道每个实际元素的类型，才能避免不出现转化异常的同时，使用原有类型功能

- List<?>

  ​		它是一个通配符类型的泛型，可以存放一种未知的参数类型，但区别于前者，它必须保证所有元素是同一种类型，从而实现泛型的安全性；

  注意：JDK大部分容器对于元素的操作方法的参数都是参数化泛型的，因此不能使用，如：

  ```java
      public boolean add(E e) {
          ensureCapacityInternal(size + 1);  // Increments modCount!!
          elementData[size++] = e;
          return true;
      }
  ```

唯一使用原生态类型场景：

​		当使用instanceof关键词判断对象类型时，由于泛型在允许在运行过程中被改变，因此编译器禁止在instatnceof判断中，使用参数化泛型；而此时使用通配符类型的泛型和原生态类型的作用是一样的，因此此时使用泛型是多余的，推荐使用原生态类型

​		同样，当进行原生态类型的判断后，应该马上将目标对象强转为通配符类型的泛型，保证目标对象使用的安全性：

```java
if(obj instanceof List){
    List<?> list = (List<?>)obj;
}
```

### 2、消除非受检警告

​		1、在使用泛型编程中，由于开发人员使用不规范，编译器常常会出现非受检警告，比如，在创建泛型变量时，

必须指定实例化时的泛型，如果使用原生态类型，会出现非受检转化警告

```java
//此时会抛出非受检警告
@SuppressWarnings("unchecked")
List<String> list = new ArrayList();

List<String> list = new ArrayList<String>();
```

在JDK7开始，引入了菱形操作符，这样编译器会自动根据变量的泛型，来推测出菱形操作符中的参数类型，从而不会提出非受检警告

​		2、在泛型方法中，我们需要将**一个数组转化为泛型参数类型数组**时，编译器就会提出非受检转化警告，但是为方法的通用性，这种转化是无法避免，因此就可以使用@SuppressWarnings("unchecked")来消除非受检警告，并提示其他开发者理解该代码这样设计的原因

```java
	public <T> T[] copy(String [] strings) {
		@SuppressWarnings("unchecked")
		T[] objects1 = (T[]) strings;
		return objects1;
	}
```

**注意：**

@SuppressWarnings("unchecked")注解可以放在某个语句上、方法上、类上，一般情况下，我们要尽可能降低它的作用范围

### 3、List优于数组

​		对于泛型List和数组，他们都是用于存储一种类型的集合，但有本质上的区别：

- 数组的类型是具体的，且为协变的（容器元素类型可以向上转型）

- 泛型List时非具体的，只在编译时具体化，来进行编译性检查，运行时则被擦除，从而支持非泛型，并且不可变

  1、因此，在编译期List就可以抛出类型转化异常，而数组由于支持协变，因此只能在运行时抛出

  2、对于泛型数组，由于在运行时类型被擦除，因此程序无法判断容器元素类型，因此编译器会提示未受检的转换警告

由于数组的以上缺点，List优于数组

### 4、优先考虑泛型

​		在进行类的设计时，优先考虑泛型，使类可以操作不同类型元素，更加安全和更加容易

```java
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	public static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(E e) {
		elements[size++] = e;
	}
}
```

​		对于element成员变量，通过在构造方法中使用Object创建数组后，在强制转换，原因是：

数组类型是具体的，不能使用泛型标识符代替，即这样是错误的：

```java
E [] element = new E[10];
```

- 泛型标识符本身没有实际意义，但JDK使用规范中，有明确规定：

| 泛型标识符 | 含义                                      |
| ---------- | ----------------------------------------- |
| E          | element集合元素                           |
| T          | type   java类型                           |
| K          | key                                       |
| V          | value                                     |
| N          | number    java数值类型                    |
| ？         | 不确定java类型                            |
| S、U、V    | 2nd、3rd、4th（用于指定区分多个Java类型） |

- 泛型类中，元素容器为什么使用数组：

  java本身是不支持List的，比如ArrayList就是通过数组实现，因此如果在不能够使用List的情况下，我们可以选择数组来实现，并且可以提供效率

### 5、优先使用泛型方法

​		和泛型类一样，泛型方法能很好的支持对多种类型的相同处理，提高代码的可复用性，同时提供类型安全性，消除非受检警告

```java
public static <E> Set<E> union(Set<E> s1,Set<E> s2){
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

### 6、利用有限通配符来提升API的灵活性

​		泛型虽然提供了类、方法对多种类型的支持，但由于泛型是不可便的，因此无法使用类的多态特性，从而降低了API的灵活性；

​		对于以上问题，JDK提供了有限通配符来解决，通过**对输入参数进行有限通配符处理**，从而实现将**子类容器（生产者）中元素**向上转型放入内部泛型E容器中；或者将内部泛型E容器中的元素向上转型放入一个**超类容器（消费者）**中

```java
public class Stack<E> {
	private E[] elements;
	private int size = 0;
	public static final int DEFAULT_INITIAL_CAPACITY = 16;

	public Stack() {
		elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
	}

	public void push(E e) {
		elements[size++] = e;
	}
    
    public void pushAll(Collection<? extends E> src){
        //参数容器元素向上转型，放入elements
        for (E e :src){
            push(e);
        }
    }
    
  	public void popAll(Collection<? super E> dst){
        //element容器元素向上转型,放入dst
		for (E element : elements) {
			dst.add(element);
		}
	}
    
}
```

​		**对于泛型方法的返回值，不应该使用有限通配符来进行处理，因为返回值的向上转型应该交给调用者决定**

- **使用有限通配符修饰泛型参数**

  ```JAVA
  <T extends Comparable<? super T>>
  ```

  该泛型参数表示，类型必须继承Comparable<? super T>>接口，实现对泛型参数本身的限制

  **推荐使用Comparable<? super T>>代替Comparable< T >：**

  Comparable是消费者，其接口实现方法，对于泛型T的父类参数，依然可以向上转型进行处理，因此使用<? super T>通配符，提高API灵活性

### 7、谨慎同时使用泛型和可变参数

​		两者都是在JDK5中提供，但并不能很好的搭配使用：

1、可变参数的作用和实质：

**可变参数的作用是实现方法同类型参数个数的动态接受，并可以直接使用数组进行入参，其底层的实现细节就是使用一个数组**

2、两者同时使用，带来的问题：

- 对于泛型可变参数，其内部也是通过泛型数组来实现，虽然java不支持创建泛型数组，但是泛型可变参数的声明却是合法的。因此java语言设计者认为，泛型可变参数在实践中有存在的必要，如JDK中的Array.asList(T... a)、Collections.addAll(Collection<? super T> C,T... elements)等方法

- 由于方法使用泛型数组作为入参，编译器就发出heap pollution警告：

  **当使用原生态类型参数变量指向泛型类型对象时，就有可能产生堆污染（heap pollution），即指定泛型集合中，引入不同类型数据**

​		当泛型数组向上转型为Object数组时，此时泛型的安全性就会被破坏，存在发生堆污染的可能：

```java
	public  void add(List<String>... lists) {
		Object[] objects=lists;
		ArrayList<Integer> integers = new ArrayList<>();
		integers.add(1);
        //此时，lists产生堆污染，存储了非List<String>类型数据
		objects[0]=integers;
        //此时隐式进行了Integer->String的转化，抛出ClassCastException异常
		String s = lists[0].get(0);
	}
```

如果将List<String>... 泛型可变参数，替换为List<List<String>>时，泛型在编译期就可以抛出转化异常，避免的向上转型

3、@SafeVarargs注解

​		在JDK7中，增加了@SafeVarargs注解，来消除泛型可变参数方法的堆污染警告；在这之前，则需要在每个带方法调用处添加@SuppressWarnings("unchecked")注解

​		@SafeVarargs是让方法设计者自己承诺，方法不会存在堆污染操作；如果需要方法绝对安全，则要满足两个条件：

- 方法没有在可变参数数组中保存其他值
- 方法没有将可变参数数组交给不被信任代码处理

**注意：**

运行一个方法访问一个泛型可变参数数组是不安全的：

```java
	public static <T> T[] toArray(T... args){
		return args;
	}

	public static <T> T[] pickTwo(T a,T b){
		return toArray(a,b);
	}
```

在编译器处理pickTwo方法时，由于a，b为非参数化泛型，为了内部实现toArray的泛型可变参数，编译器将会使用Object数组来保存a，b，从而确保支持任意具体实例的数据，因此toArray方法返回时，内部会进行一次Object[ ]->T[ ]的转化，导致抛出ClassCastException异常

**综上所述，由于泛型和可变参数结合时，存在一些问题，因此推荐使用List代替数组（4.4）**

### 8、优先考虑类型安全的异构容器

​		泛型保证了容器类型转化的安全性，但是也约束了容器的灵活性，无法同时存储多种类型；但是我们基于当前JDK所提供了容器，可以实现一个类型安全的异构容器：

- 类型安全，容器中的元素类型是安全的，能被编译器检验的
- 异构，基于map，但是key为不同类型

```java
public class Favorites{
    private Map<Class<?>,Object> favorites=new HashMap<>();
    
    public <T> void putFavorites(Class<T> type,T instance){
        favorites.put(type,instance);
    }
    
    public <T> void getFavorites(Class<T> type){
        return type.cast(favorites.get(type));
    }
}
```

## 5、枚举和注解

### 1、使用enum枚举代替常量集

通过enum枚举代替常量集，具有以下优点：

- 能够提高代码可读性，方便对一组常量数据进行管理
- enum枚举支持switch语句，适合对特定常量添加额外行为
- enum由java语言保证其单例，搭配开发者设置多个final成员变量，更加灵活的对枚举常量与其他数据做关联

### 2、用实例域代替序数

枚举都有一个ordinal方法，返回每个枚举成员所声明的顺序；如果需要为每个枚举成员添加一个序列数时，并要利用ordinal方法实现，原因：

ordinal方法得到的序列数会收到枚举成员声明顺序的影响，因此不容易受控制，导致出现糟糕的代码

可以通过添加一个final成员变量，来手动设置每个枚举成员的序列数

### 3、用EnumSet操作多个常量集合

EnumSet用于存储多个Enum实例，从而方便对多个常量进行操作

### 4、使用EnumMap实现枚举为key的映射关系

```java
EnumMap<YesOrNotEnum,String> enumMap = new EnumMap<>(YesOrNotEnum.class);
enumMap.put(YES,"sss");
```

### 5、用接口实现扩展枚举

​		枚举类本身继承enum，因此无法通过基础扩展枚举，只能使用接口扩展

### 6、注解优于命名模式

​		在注解没有出来前，有许多工具插件的使用，都需要通过合理的命名来实现；如JUNIT3，测试类方法必须带有test；在JDK5推出注解后，就避免了这种问题的发生

### 7、始终使用@Override注解

​		所有覆写方法都需要添加@Override，避免开发者覆写方法编写错误，导致发生不必要的意外

### 8、不要使用注解代替标记接口

​		标记接口表示没有方法声明的接口。相对于注解，它能够在编译过程中，实现错误代码的捕捉；如，Serializable标记接口表明类可以序列化，当使用ObjectOutputStream.writeObject（）方法输出对象时，编译器会校验当前对象是否可以序列化，否则就会抛出错误

## 6、Lambda和Stream

### 1、Lambda优先于匿名类

​		JDK8中，推出Lambda语法糖来代替匿名内部类，简化代码的书写方式

注意：

- Lambda表达式可以简化方法的入参类型，编译器会自动进行类型推导
- 如果转化为Lambda表达式后，代码还是有多行，则需要考虑是否对当前接口进行重构（接口方法职责过重）

### 2、方法引用优先于Lambda

​		方法引用是对Lambda进一步简化，来消灭一些简单代码的书写，如getter、setter、toString

在JDK8中，也对八种包装类方法进行的增强，如Integer.sum（int1,int2）方法，从而方便使用方法引用

### 3、坚持使用标准的函数接口

​		JDK8中，提供了一系列函数接口，作为通用对象，来代替单方法接口，并使用Lambda或方法引用简化书写，如Runnable；从而避免开发者，自己编写相关接口

### 4、不要滥用Stream

​		虽然StreamAPI能够简化代码，但是也不能将所有操作一次性放入Sream中进行操作，否则会导致代码可读写严重降低

### 5、优先选择Stream中无副作用的函数

​		Stream方法分为中间操作和终止操作，每个方法都有自己对应的作用，不要单纯只是利用Stream的写法，还需要考虑Stream-api范式：

**将一组计算进行分级，每一级结果靠近为上级结果的纯函数（函数结果只依赖于入参，不更新任何状态值）**

因此就需要注意如下：

- forEeach应该只用于遍历集合，不应该在内部进行额外计算，改变其他变量状态值

### 6、Stream要优先使用Collection作为返回类型

​		Stream不支持Iterable接口，无法进行foreach遍历，因此在API设计时，应该优先返回Collection；这样既可进行foreach遍历，而Collection又提供重新转化为Stream的方法

### 7、谨慎使用Stream并行

​		