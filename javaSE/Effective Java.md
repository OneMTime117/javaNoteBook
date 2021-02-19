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

2、对于基础数据类型包装类