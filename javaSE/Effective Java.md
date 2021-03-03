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
}catch(Exception)
```

​	可以发现try-with-resources直接省略了对资源finally中的close方法调用，使代码变得非常简洁。同时，当readLine（）方法和隐藏的close方法都发生异常时，会保留第一个异常信息，静止其他后续的异常。从而保证开发者能够快速找到导致资源使用异常的信息

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

当我们覆盖equals方法后，则第二条规范就会失效，O