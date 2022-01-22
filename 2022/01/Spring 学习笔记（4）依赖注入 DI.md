本篇文章主要对 Spring 框架中的核心功能之一**依赖注入 (DI，Dependency Injection)** 进行介绍，也是采用 **理论+实战** 的方式给大家阐述其中的原理以及明确需要注意的地方。

## 1. 依赖注入

依赖注入是实现控制反转的一种模式，主要是解决依赖性问题，它是将所依赖的传递给将使用的从属对象。我们将其拆分来看，首先说说什么是依赖，如下所示：

```java
class B{
    private A a; // B 类依赖 A 类
}
```

有两个类，即 A 类和 B 类，在 B 类中使用了 A 类对象，这时我们就说，当一个对象需要使用另一个对象的时候就产生了依赖。在之前的开发中，对于上面的情况，我们需要使用 `接口 引用名称 = new 实现类()` 的方式，但这存在一个问题，它们之间的耦合度太高了，彼此产生了依赖，不利于功能的拓展。而当我们引入**依赖注入**后，就对其进行了解耦，当我们需要使用某个接口的时候，不需要知道具体的实现类。

而对于注入，就是让容器去把符合依赖关系的对象通过 `bean` 属性或者构造函数的方式传递给需要的对象，分为**属性注入(Setter Injection)\**和\**构造器注入(Constructor Injection)**。这里先介绍用 setter 的方式进行注入，构造器的方法在文章的后面会讲到。

而 Bean 的配置形式可以是基于 xml 文件的方式，也可以是基于注解的方式。具体的讲，可以通过全类名（反射）、工厂方法（静态工厂方法和实例工厂方法）、FactoryBean 的形式进行配置。

## 2. DI 简单案例

为了演示依赖注入这一特点，这里的步骤一共有四步，如下所示：

- 创建 BookDao 接口和实现类
- 创建 BookService 接口和实现类
- 在 xml 配置文件中配置 dao 和 service
- 测试

###  创建 BookDao 接口和实现类

```java
// BookDao.java
public interface BookDao {
    public void save();
}
```

```java
// BookDaoImpl.java
public class BookDaoImpl implements BookDao {
    @Override
    public void save() {
        System.out.println("BookDao...save");
    }
}
```

###  创建 BookService 接口和实现类

```java
// BookService.java
public interface BookService {
	public void addBook();
}
```

```java
// BookServiceImpl.java
public class BookServiceImpl implements BookService {
    // 方式1：之前，接口 = new 实现类
    // private BookDao bookDao = new BookDaoImpl();

    private BookDao bookDao;

    public void setBookDao(BookDao bookDao) {
        this.bookDao = bookDao;
    }

    @Override
    public void addBook() {
        this.bookDao.save();
    }
}
```

可以看到，方式 1 是之前使用的方式，这种方式需要用到具体的实现类，增加了耦合性。而方式 2 通过 `接口 + setter方法` 的形式进行了解耦，这样就胡需要知道具体的实现类了。

### 在 xml 配置文件中配置 dao 和 service

我们还是在 `src` 下创建 `bean.xml` 配置文件，文件内容如下所示：

```xml
<!--创建 Dao-->
<bean id="bookDaoId" class="com.riotian.Dao.impl.BookDaoImpl"></bean>
<!--创建Service-->
<bean id="bookServiceId" class="com.riotian.Service.impl.BookServiceImpl">
    <property name="bookDao" ref="bookDaoId"></property>
</bean>
```

之前说过，每创建一个实例都需要一个 `<bean>` 标签，也就相当于**控制反转(IoC)**，其中`创建 service 实例`相当于替代了如下代码：

```java
BookService bookService = new BookServiceImpl();
```

而`创建 dao` 实例也相当于进行了**控制反转(IoC)**，也就相当于替代了如下代码：

```java
BookDao bookDao = new BookDaoImpl();
```

而 `<property>` 标签中的内容就像当于进行了**依赖注入(DI)**，也就是相当于替换了如下代码：

```java
bookService.setBookDao(bookDao);
```

这里对子标签 `<property>` 进行说明，该标签用于进行属性注入，其中 `name` 表示 `bean` 的属性名，是通过**BookServiceImpl.java 实现类**中的 `setter` 方法获得的，即 `bookDao`，而 `ref` 表示另一个 `bean` 的 `id` 值的引用，在这里引用的是 `dao` 的 `id` ，即 `bookDaoId` 。至此，对其属性就注入完成了，我们下面编写测试类。

### 测试类

```java
// TestDI.java
@Test
public void test3() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    BookService bookService = (BookService) app.getBean("bookServiceId");
    bookService.addBook();
}
```

实现方式和上一篇介绍 IoC 一样，也是先获取配置文件，然后获取 `bean` 并加载元数据，将其强转成 `BookService` 类型后调用 `addBook()` 方法即可，最后可以看到输出语句为 `BookDao...save`

需要注意的是，`getBean()` 的参数是配置文件中配置 `service` 的 `id` 。因为我们要先得到 service 层，通过 `service` 得到 `addBook()` 方法，在该方法中调用 `dao` 的 `save()` 方法，从而输出 `BookDao...save`

这里的 ApplicationContext 代表 IoC 容器，实际上它是一个接口。在 Spring IoC 容器读取 Bean 配置 Bean 实例之前，需要对容器本身进行初始化。只有容器进行初始化后，才可以从 IoC 容器里获取 Bean 实例并使用。

Spring 提供了两种类型的 IoC 容器实现：

- BeanFactory：IoC 容器的基本实现；
- ApplicationContext：提供了更多的高级功能，是 BeanFactory 的子接口。

BeanFactory 是 Spring 框架的基础设施，面向 Spring 本身；ApplicationContext 面向使用 Spring 框架的开发者，几乎所有的应用场景都会使用 ApplicationContext 而非底层的 BeanFactory。但无论使用哪种方式，其配置文件都是相同的。

ApplicationContext 主要有两个实现类：

- ClassPathXmlApplicationContext：从类路径下加载配置文件。
- FileSystemXmlApplicationContext：从文件系统中加载配置文件。

此外，ConfigurableApplicationContext 扩展于 ApplicationContext，新增加了两个主要方法：refresh() 和 close()，让 ApplicationContext 具有启动、刷新和关闭上下文的能力。WebApplicationContext 是专门为 WEB 应用准备的，它允许从相对于 WEB 根目录的路径中完成初始化工作。它们之间的关系如下图所示：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220121212804.png)

---

需要注意的是：**ApplicationContext  有一个作用范围 Scope**

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220121104108.png)

* 当 scope 的取值为 **singleton** 时

  Bean 的实例化个数：1 个

  Bean 实例化时机： 当 Spring 核心文件被加载时，实例化配置的 Bean 实例

  Bean 的生命周期

  * 对象创建：当应用加载，创建容器时，对象就被创建了
  * 对象运行：只要容器在，对象一直活着
  * 对象销毁：当应用卸载，销毁容器时，对象被销毁

* 当 scope 的取值为 **prototype** 时

  Bean 的实例化个数：多个

  Bean 实例化时机： 当调用 getBean() 方法时实例化Bean

  Bean 的生命周期

  * 对象创建：当使用对象时，创建新的对象实例
  * 对象运行：只要容器在，对象一直活着
  * 对象销毁：当对象长时间不用时，被 Java 的垃圾回收期回收了

---

在获取 Bean 的时候，除了可以使用 getBean(String) 方法之外，还可以使用 getBean(Class) 方法。如果使用后者，则需要确保当前 Bean 在 IoC 容器中是唯一的。如果不是唯一的，那么会出现如下错误：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置 bean-->
    <!--scope默认为singleton-->
    <bean id="helloWorld" class="com.riotian.demo.HelloWorld" >
        <property name="name" value="Spring"></property>
        <property name="age" value="123"></property>
    </bean>

    <bean id="helloWorld2" class="com.riotian.demo.HelloWorld" >
        <property name="name" value="Spring"></property>
        <property name="age" value="123"></property>
    </bean>
</beans>
```

```
Exception in thread "main" org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'com.riotian.demo.HelloWorld' available: expected single matching bean but found 2: helloWorld,helloWorld2
```

## 3. 注入的方式

### setter 方法注入

如前面所提到的，使用 setter 方法注入，就是在 xml 文件中引入 property 属性，如下所示：

```xml
<property name="name" value="Spring"></property>
```

### 构造方法注入

通过构造方法注入 Bean 的属性值或依赖对象，它保证了 Bean 实例在实例化后就可以使用。该方式通过`<constructor-arg>`元素里声明属性。但需要注意的是，该元素里没有 name 属性。下面通过一个示例，展示其使用方法。

```java
// Car.java
public class Car {
    private String brand;
    private String corp;
    private double price;
    private int maxSpeed;

    public Car(String brand, String corp, double price) {
        this.brand = brand;
        this.corp = corp;
        this.price = price;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", corp='" + corp + '\'' +
                ", price=" + price +
                ", maxSpeed=" + maxSpeed +
                '}';
    }
}
```

```xml
<!--通过构造方法配置 Bean 的属性-->
<bean id="car" class="com.riotian.pojo.Car">
	<constructor-arg value="Audi"></constructor-arg>
	<constructor-arg value="ShangHai"></constructor-arg>
	<constructor-arg value="300000"></constructor-arg>
</bean>
```

```java
// Test.java
@Test
public void testConstructBean() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    Car car = (Car) app.getBean("car");
    System.out.println(car);
}
```

输出结果如下：

```java
Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=0}
```

如果我们改造一下，再添加一个构造器，同时在 xml 文件中再配置一个 Bean，看看是什么效果。

```java
// Car.java
public class Car {
    private String brand;
    private String corp;
    private double price;
    private int maxSpeed;

    public Car(String brand, String corp, double price) {
        this.brand = brand;
        this.corp = corp;
        this.price = price;
    }

    public Car(String brand, String corp, int maxSpeed) {
        this.brand = brand;
        this.corp = corp;
        this.maxSpeed = maxSpeed;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", corp='" + corp + '\'' +
                ", price=" + price +
                ", maxSpeed=" + maxSpeed +
                '}';
    }
}
```

```xml
<!--通过构造方法配置 Bean 的属性-->
<bean id="car" class="com.riotian.pojo.Car">
	<constructor-arg value="Audi" index="0"></constructor-arg>
	<constructor-arg value="ShangHai" index="1"></constructor-arg>
	<constructor-arg value="300000" index="2"></constructor-arg>
</bean>

<bean id="car2" class="com.example.springdemo.beans.Car">
	<constructor-arg value="BMW"></constructor-arg>
	<constructor-arg value="Beijing"></constructor-arg>
	<constructor-arg value="240"></constructor-arg>
</bean>
```

```java
// Test.java
@Test
public void testConstructBean() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    Car car = (Car) app.getBean("car");
    Car car2 = (Car) app.getBean("car2");
    System.out.println(car);
    System.out.println(car2);
}
```

输出结果如下：

```java
Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=0}
Car{brand='BMW', corp='Beijing', price=240.0, maxSpeed=0}
```

可以看到，IoC 容器在给两个构造器赋值的时候，产生了歧义。我们原本想把 id=“car2” 中的 240 这个值赋值给 maxSpeed，可是现在这个值也都赋值给了 price。

这有点类似于方法的重载，我们可以通过指定参数的类型来进行赋值。如下所示：

```xml
<!--通过构造方法配置 Bean 的属性-->
<bean id="car" class="com.riotian.pojo.Car">
	<constructor-arg value="Audi" index="0"></constructor-arg>
	<constructor-arg value="ShangHai" index="1"></constructor-arg>
	<constructor-arg value="300000" type="double"></constructor-arg>
</bean>

<bean id="car2" class="com.example.springdemo.beans.Car">
	<constructor-arg value="BMW" type="java.lang.String"></constructor-arg>
	<constructor-arg value="Beijing" type="java.lang.String"></constructor-arg>
	<constructor-arg value="240" type="int"></constructor-arg>
</bean>
```

输出结果如下：

```java
Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=0}
Car{brand='BMW', corp='Beijing', price=0.0, maxSpeed=240}
```

可以看到，输出了我们想要的结果。因此，在使用构造器注入属性值时，可以指定参数的位置和参数的类型，以区分重载的构造器。

### 注入的细节

可以看到，我们在 xml 中通过使用 value=“240” 的方式，将一个字符串类型的 240 赋值给了 int 类型的 maxSpeed。对于这样的字面值，即使用字符串表示的值，可以通过`value`元素标签或 value 属性注入。如下所示：

```xml
<bean id="car2" class="com.riotian.pojo.Car">
	<constructor-arg value="BMW" type="java.lang.String"></constructor-arg>
	<constructor-arg value="Beijing" type="java.lang.String"></constructor-arg>
	<constructor-arg type="int">
		<value>240</value>
	</constructor-arg>
</bean>
```

基本数据类型及其封装类型、String 等类型都可以采取字面值植入的方式。如果字面值包含特殊字符，那么可以使用`<![CDATA[]]>`把字面值包裹起来。如下所示：

```xml
<bean id="car2" class="com.riotian.pojo.Car">
	<constructor-arg value="BMW" type="java.lang.String"></constructor-arg>
	<constructor-arg type="java.lang.String">
		<value><![CDATA[<Beijing^>]]></value>
	</constructor-arg>
	<constructor-arg type="int">
		<value>240</value>
	</constructor-arg>
</bean>
```

输出结果如下：

```java
Car{brand='BMW', corp='<Beijing^>', price=0.0, maxSpeed=240}
```

### 引入其他的 Bean

组成应用程序的 Bean 经常需要相互协作，以完成应用程序的功能。要使 Bean 能够相互访问，就必须在 Bean 的配置文件中指定对 Bean 的引用。

在 Bean 的配置文件中，可以通过`<ref>`元素或 ref 属性为 Bean 的属性或构造器参数指定对 Bean 的引用。也可以在属性或构造器里包含 Bean 的声明，这样的 Bean 被称为**内部 Bean**。如下所示：

```java
// People.java
public class People {
    private String name;
    private int age;

    private Car car;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "People{" +
                "name='" + name + '\'' +
                ", age=" + age +
                ", car=" + car +
                '}';
    }
}
```

```xml
<bean id="car2" class="com.riotian.pojo.Car">
    <constructor-arg value="BMW" type="java.lang.String"></constructor-arg>
    <!--<constructor-arg value="BeiJing" type="java.lang.String"></constructor-arg>-->
    <constructor-arg type="java.lang.String">
        <value><![CDATA[<Beijing^>]]></value>
    </constructor-arg>
    <!--<constructor-arg value="240" type="int"></constructor-arg>-->
    <constructor-arg type="int">
        <value>240</value>
    </constructor-arg>
</bean>
```

```java
@Test
public void testBeanRef() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    People peoson = (People) app.getBean("people");
    System.out.println(peoson);
}
```

输出结果如下：

```java
People{name='RioTian', age=21, car=Car{brand='Tian', corp='NanChang', price=200000.0, maxSpeed=0}}
```

当然，也可以写成如下形式：

```xml
<bean id="people" class="com.riotian.pojo.People">
    <property name="name" value="RioTian"/>
    <property name="age" value="21"/>
    <!--  引用上面的car2  -->
    <!--<property name="car" ref="car2"/>-->
    <!--<property name="car">-->
    <!--    <ref bean="car2"/>-->
    <!--</property>-->
    <property name="car">
        <bean class="com.riotian.pojo.Car">
            <constructor-arg value="Tian"></constructor-arg>
            <constructor-arg value="NanChang"/>
            <constructor-arg value="200000" type="double"/>
        </bean>
    </property>
</bean>
```

```java
People{name='RioTian', age=21, car=Car{brand='Tian', corp='NanChang', price=200000.0, maxSpeed=0}}
```

这里不需要在 Bean 中指定 id 了，因为内部 Bean 不能被外部使用，所以这里的 id 就没有意义了。

### null 值和级联属性

可以使用专用的`<null/>`元素标签为 Bean 的字符串或其他对象类型的属性注入 null 值。同时，Spring 也支持级联属性的配置。

```xml
<bean id="person2" class="com.riotian.pojo.People">
	<constructor-arg value="Jerry"></constructor-arg>
	<constructor-arg value="25"></constructor-arg>
	<constructor-arg><null/></constructor-arg>
</bean>
```

输出结果如下：

```
Person{name='Jerry', age=25, car=null}
```

对于级联属性，如下所示：

```xml
<bean id="person2" class="com.riotian.pojo.People">
	<constructor-arg value="Jerry"></constructor-arg>
	<constructor-arg value="25"></constructor-arg>
	<constructor-arg ref="car"></constructor-arg>
	<property name="car.maxSpeed" value="250"></property>
</bean>
```

输出结果如下：

```
Person{name='Jerry', age=25, car=Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=250}}
```

需要注意的是，属性需要先初始化后才能为级联属性赋值，否则会有异常。这是区别于 Structs2 的一点。

### 集合属性

在 Spring 中可以通过一组内置的 xml 标签（`<list>`、`<set>`、`<map>`）来配置集合属性。List 类型和数组类型的属性，可以使用`<List>`，Set 类型的属性需要使用`<Set>`。对于 Map 类型，需要使用多个`<entry>`作为子标签，每个条目包含一个键和一个值。

```xml
<bean id="person3" class="com.riotian.pojo.People">
	<property name="name" value="Mike"></property>
	<property name="age" value="25"></property>
	<property name="cars">
		<list>
			<ref bean="car"></ref>
			<ref bean="car2"></ref>
		</list>
	</property>
</bean>
```

```java
@Test
public void testCollectionAttributes() {
    ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
    Person person = (Person) context.getBean("person3");

    System.out.println(person);
}
```

输出结果如下：

```
Person{name='Mike', age=25, cars=[Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=250}, Car{brand='BMW', corp='<ShangHai^>', price=0.0, maxSpeed=240}]}
```

对于 Map 类型的赋值，如下所示：

```java
@Test
public void testMapAttributes() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    NewPerson newPerson = (NewPerson) app.getBean("newPerson");
    System.out.println(newPerson);
}
```

输出结果如下所示：

```
NewPerson{name='Rose', age=28, cars={AA=Car{brand='Audi', corp='ShangHai', price=300000.0, maxSpeed=0}, BB=Car{brand='BMW', corp='<Beijing^>', price=0.0, maxSpeed=240}}}
```

## 4. 自动装配

Spring IoC 容器可以自动装配 Bean，只需在`<bean>`的 autowire 属性里指定自动装配的模式，有以下几种方式：

- byType：根据类型自动装配。如果 IoC 容器中有多个与目标 Bean 类型一致的 Bean，在这种情况下，Spring 将无法判断哪个 Bean 最适合该属性，所以不能执行自动装配。
- byName：根据名称自动装配。必须将目标 Bean 的名称和属性名设置相同。
- constructor：通过构造器自动装配。当 Bean 中存在多个构造器时，此方式将会很复杂，**不推荐使用**。

以往的实现方式：

```xml
<bean id="address" class="com.riotian.springdemo.autowire.Address"
	p:city="Beijing" p:street="ChaoYang">
</bean>

<bean id="car" class="com.riotian.springdemo.autowire.Car"
	p:brand="Audi" p:price="400000">
</bean>

<bean id="person" class="com.riotian.springdemo.autowire.Person"
	p:name="Tom" p:address-ref="address" p:car-ref="car"></bean>
```

采用自动装配的方式：

```xml
<bean id="person" class="com.riotian.springdemo.autowire.Person"
	p:name="Tom" autowire="byName">
</bean>
```

输出结果如下：

```
Person{name='Tom', address=null, car=Car{brand='Audi', price=400000.0}}
```

这里使用了`autowire="byName"`，它会根据 Bean 的名字和当前 Bean 的 setter 风格的属性名进行自动装配。而`byType`根据 Bean 的类型和当前 Bean 的属性的类型进行自动装配。但是，如果 IoC 容器中有一个以上的类型匹配的 Bean，则会抛出异常。

自动装配也会存在一些缺点：

- 在 Bean 的配置文件中设置 autowire 属性进行自动装配时，将会装配 Bean 的所有属性。然而，如果只想装配个别属性，那么 autowire 属性就不那么灵活了；
- autowire 要么根据类型自动装配，要么根据名称自动装配，两者不能兼而有之；
- **一般情况下，实际项目中很少用到自动装配的功能。**

## 5. Bean 之间的关系

### 继承关系

Bean 之间在配置的时候，存在依赖或继承关系。下面是使用以前的方式创建两个 Bean：

```xml
<bean id="address" class="com.example.springdemo.autowire.Address"
	p:city="Beijing" p:street="ChaoYang">
</bean>

<bean id="address2" class="com.example.springdemo.autowire.Address"
	p:city="Beijing" p:street="WangJing">
</bean>
```

你可以看到，两个 Bean 的 city 和 class 属性值都是一样的，为了复用属性值，我们可以使用 Bean 的 parent 属性指定当前 Bean 所继承另外一个 Bean 的配置，如下所示：

```xml
<bean id="address" class="com.example.springdemo.autowire.Address"
	p:city="Beijing^" p:street="ChaoYang">
</bean>

<bean id="address2" p:street="WangJing" parent="address">
</bean>
```

需要注意的是：

- Spring 允许继承 Bean 的配置，被继承的 Bean 称为父 Bean。继承这个父 Bean 的 Bean 称为子 Bean；
- 子 Bean 可以从父 Bean 中继承配置，包括 Bean 的属性配置；
- 子 Bean 也可以覆盖从父 Bean 继承过来的配置；
- **父 Bean 可以作为配置模板，也可以作为 Bean 实例。若只想把父 Bean 作为模板，可以设置`＜bean＞`的 abstract 属性为 true，这样 Spring 将不会实例化这个 Bean；**
- **并不是`＜bean＞`元素里的所有属性都会被继承。比如：autowire、abstract 等；**
- **可以忽略父 Bean 的 class 属性，让子 Bean 指定自己的类，而共享相同的属性配置。但此时 abstract 必须设为 true。**

### 依赖关系

Spring 允许用户通过`depends-on`属性设定当前 Bean 的前置依赖 Bean，前置依赖的 Bean 会在当前 Bean 实例化之前创建完成。如果需要依赖于多个 Bean，则可以使用逗号、空格的方式配置 Bean 的名称。

在配置 Person 的时候，如果想实现必须有一个关联的 Car，即 Person 这个 Bean 依赖于 Car 这个 Bean，那么可以使用`depends-on="car3"`。在指定`depends-on="car3"`之后，如果`car3`这个 Bean 没有创建，则 IoC 在初始化的时候就会报错。如下所示：

```xml
<!-- 
	<bean id="car3" class="om.riotian.pojo.Car"
		p:brand="TOYOTA" p:price="200000">
    </bean> 
-->
<bean id="person" class="com.riotian.springdemo.autowire.Person"
	p:name="Tom" p:address-ref="address" depends-on="car3">
</bean>
```

如果没有创建`car3`这个 Bean，则报错如下：

```code
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'person' defined in class path resource [beans_relation.xml]: 'person' depends on missing bean 'car3'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'car3' available
```

### Bean 的作用域

默认情况下，在配置文件所配置的 Bean 都是单例的。如下所示：

```xml
<bean id="car" class="com.riotian.pojo.Car">
	<property name="brand" value="Audi"></property>
	<property name="price" value="100000"></property>
</bean>
```

```java
@Test
public void testBeanScope() {
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    Car car = (Car) app.getBean("car");
    Car car2 = (Car) app.getBean("car");
    System.out.println( car == car2 );
}
```

输出结果如下：

```
true
```

可以在 xml 文件中，配置 Bean 的 scope，如下所示：

```xml
<bean id="car"
	class="com.riotian.pojo.Car"
	scope="prototype">
	<property name="brand" value="Audi"></property>
	<property name="price" value="100000"></property>
</bean>
```

此时再运行测试方法，其结果就变为 false 了。这是因为每次创建的 Bean 都是原生的 Bean，即这两个 Bean 都不是单例的。

需要注意的是：默认情况下，Bean 的创建过程是单例的（singleton），即在整个容器的生命周期内只创建一个 Bean。因此当前类中的无参构造器是会执行的。而如果指定`scope=prototype`，那么当创建 IoC 容器的时候，无参的构造器是不会执行的。而当获取 Bean 时，每次都会执行无参的构造器。

## 6. IoC 容器中 Bean 的生命周期

Spring IoC 容器可以管理 Bean 的生命周期，Spring 允许在 Bean 生命周期的特定点执行定制的任务。其管理过程如下：

- 通过构造器或工厂方法创建 Bean 实例；
- 为 Bean 的属性设置值以及对其他 Bean 的引用；
- 调用 Bean 的初始化方法；
- 此时的 Bean 可以使用了；
- 当容器关闭时，调用 Bean 的销毁方法。
  - 在 Bean 的声明里设置`init-method`和`destroy-method`属性，可以为 Bean 指定初始化和销毁方法。

代码如下所示：

```java
// NewCar.java
public class NewCar {

    private String brand;

    public NewCar() {
        System.out.println("Car's constructor...");
    }

    public void setBrand(String brand) {
        System.out.println("setBrand...");
        this.brand = brand;
    }

    public void init() {
        System.out.println("init...");
    }

    public void destroy() {
        System.out.println("destroy...");
    }

    @Override
    public String toString() {
        return "NewCar{" +
                "brand='" + brand + '\'' +
                '}';
    }
}
```

```xml
<bean id="car5" class="com.riotian.pojo.NewCar" init-method="init" destroy-method="destroy">
    <property name="brand" value="Audi"/>
</bean>
```

```java
@Test
public void testBeanCycle() {
    ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    NewCar car = (NewCar) app.getBean("car5");
    System.out.println(car);

    // 关闭 IoC 容器
    app.close();
}
```

输出结果如下：

```java
Car's constructor...
setBrand...
init...
NewCar{brand='Audi'}
destroy...
```

### Bean 的后置处理器

Bean 的后置处理器允许在调用初始化方法的前后，对 Bean 进行额外的处理。Bean 后置处理器对 IoC 容器里所有的 Bean 实例进行逐一处理，而非单一实例。典型的应用是：检查 Bean 属性的正确性或根据特定的标准更改 Bean 的属性。

对 Bean 的后置处理器而言，需要实现接口 BeanPostProcessor。在初始化方法被调用前后，Spring 把每个 Bean 实例分别传递给上述接口的两个方法，即：

- Object postProcessBeforeInitialization(Object bean, String beanName)
- Object postProcessAfterInitialization(Object bean, String beanName)

在添加了 Bean 的后置处理器之后，Bean 的声明周期如下所示：

- 通过构造器或工厂方法创建 Bean 实例；
- 为 Bean 的属性设置值以及对其他 Bean 的引用；
- **将 Bean 实例传递给 Bean 后置处理器的 postProcessBeforeInitialization 方法**；
- 调用 Bean 的初始化方法；
- **将 Bean 实例传递给 Bean 后置处理器的 postProcessAfterInitialization 方法**
- 此时的 Bean 可以使用了；
- 当容器关闭时，调用 Bean 的销毁方法。
  - 在 Bean 的声明里设置`init-method`和`destroy-method`属性，可以为 Bean 指定初始化和销毁方法。

代码如下所示：

```java
public class MyBeanPostProcessor implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization: " + bean + ", " + beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization: " + bean + ", " + beanName);
        return bean;
    }
}
```

```xml
<!-- 配置 Bean 的后置处理器: 不需要配置 id，IoC 容器会自动识别它是一个 BeanPostProcessor-->
<bean class="com.riotian.Demo.MyBeanPostProcessor"></bean>
```

```java
@Test
public void testBeanCycle() {
    ClassPathXmlApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    NewCar car = (NewCar) app.getBean("car5");
    System.out.println(car);

    // 关闭 IoC 容器
    app.close();
}
```

输出结果如下所示：

```java
Car's constructor...
setBrand...
postProcessBeforeInitialization: NewCar{brand='Audi'}, car5
init...
postProcessAfterInitialization: NewCar{brand='Audi'}, car5
NewCar{brand='Audi'}
destroy...
```

在 MyBeanPostProcessor 类中，postProcessBeforeInitialization 方法和 postProcessAfterInitialization 方法中的 bean 表示 Bean 实例本身；beanName 表示在 IoC 容器中配置的 Bean 的名字，也就是 id；方法的返回值实际上返回给用户的那个 Bean。因此，可以在其返回之前，修改返回的 Bean，甚至返回一个新的 Bean。

## 7. 配置 Bean 的方式

### 静态工厂方法

代码如下：

```java
public class Car {
    private String brand;
    private double price;

    public Car() {
        System.out.println("Car's constructor...");
    }

    public Car(String brand, double price) {
        this.brand = brand;
        this.price = price;
    }

    public void setBrand(String brand) {
        System.out.println("setBrand...");
        this.brand = brand;
    }

    @Override
    public String toString() {
        return "Car{" +
                "brand='" + brand + '\'' +
                ", price=" + price +
                '}';
    }
}
```

```java
/**
 * 静态工厂方法：直接调用某一个类的静态方法就可以返回 Bean 的实例
 */
public class StaticCarFactory {

    private static Map<String, Car> cars = new HashMap<>();

    static {
        cars.put("Audi", new Car("Audi", 300000));
        cars.put("Ford", new Car("Ford", 200000));
    }

    /**
     * 静态工厂方法
     *
     * @param name
     * @return
     */
    public static Car getCar(String name) {
        return cars.get(name);
    }
}
```

```xml
 <!-- 通过静态工厂方法配置 Bean，注意不是配置静态工厂方法实例，而是配置 Bean 实例 -->
<bean id="car1" class="com.example.springdemo.factory.StaticCarFactory"
		factory-method="getCar">
	<constructor-arg value="Audi"></constructor-arg>
</bean>
```

```java
public class Main {
    @Test
    public void testStaticFactory() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans_factory.xml");
        Car car1 = (Car) context.getBean("car1");
        System.out.println(car1);
    }
}
```

输出结果如下：

```java
Car{brand='Audi', price=300000.0}
```

需要注意的是，xml 中的 class 属性需要指向静态工厂方法的全类名；factory-method 需要指向静态工厂方法的名字。如果工厂方法需要传入参数，则使用 constructor-arg 配置参数。

### 实例工厂方法

代码如下所示：

```java
/**
 * 实例工厂方法：先需要创建工厂本身，再调用工厂的实例方法来返回 Bean 的实例
 */
public class InstanceCarFactory {
    private Map<String, Car> cars = null;

    public InstanceCarFactory() {
        cars = new HashMap<>();
        cars.put("Audi", new Car("Audi", 400000));
        cars.put("Ford", new Car("Ford", 500000));
    }

    public Car getCar(String brand) {
        return cars.get(brand);
    }
}
```

```xml
<!-- 配置工厂的实例 -->
<bean id="carFactory" class="com.example.springdemo.factory.InstanceCarFactory"></bean>

<!-- 通过实例工厂方法来配置 Bean -->
<bean id="car2" factory-bean="carFactory" factory-method="getCar">
	<constructor-arg value="Ford"></constructor-arg>
</bean>
```

```java
public class Main {
    @Test
    public void testInstanceFactory() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans_factory.xml");
        Car car = (Car) context.getBean("car2");
        System.out.println(car);
    }
}
```

输出结果如下：

```java
Car{brand='Ford', price=500000.0}
```

需要注意的是，factory-bean 需要指向实例工厂方法的 Bean。

### FactoryBean

代码如下：

```java
public class CarFactoryBean implements FactoryBean<Car> {
    private String brand;

    public void setBrand(String brand) {
        this.brand = brand;
    }

    /**
     * 返回 Bean 的对象
     *
     * @return
     * @throws Exception
     */
    @Override
    public Car getObject() throws Exception {
        return new Car(brand, 500000);
    }

    /**
     * 返回 Bean 的类型
     *
     * @return
     */
    @Override
    public Class<?> getObjectType() {
        return Car.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
```

```xml
<bean id="car" class="com.example.springdemo.factoryBean.CarFactoryBean">
	<property name="brand" value="BMW"></property>
</bean>
```

```java
public class Main {
    @Test
    public void testFactoryBean() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans_factoryBean.xml");
        Car car = (Car) context.getBean("car");

        System.out.println(car);
    }
}
```

输出结果如下：

```java
Car{brand='BMW', price=500000.0}
```

如果想要自定义一个 xxxFactoryBean，则需要实现 FactoryBean 接口。在 xml 文件中，通过 FactoryBean 来配置 Bean 实例。其中，class 指向 FactoryBean 的全类名，property 用于配置 FactoryBean 的属性，但实际返回的实例是 FactoryBean 中 getObject() 方法所返回的实例。

## 8. 基于注解的方式配置 Bean

如果想要将一个 Bean 加上注解，然后放到 IoC 容器的话，则需要用到组件扫描（Component scanning）的功能。Spring 能够从 classpath 下自动扫描，侦测和实例化具有特定注解的组件。这些组件包括：

- @Component：基本注解，标识了一个受 Spring 管理的组件；
- @Repository：标识持久层组件；
- @Service：标识业务层组件；
- @Controller：标识表现层组件。

对于扫描到的组件，Spring 有默认的命名策略。它会使用非限定类名的第一个小写字母。也可以在注解中通过 value 属性值来标识组件的名称。

在组件类上使用了特定的注解之后，还需要在 Spring 的配置文件中声明`<context:component-scan>`，其中：

- base-package 属性指定一个需要扫描的基类包，Spring 容器将会扫描这个基类包中的类及其子包中的所有类。
- 当需要扫描多个包时，可以使用逗号分隔。
- 如果希望扫描特定的类而非基类包下的所有类，可以使用 resource-pattern 属性过滤特定的类。
- `<context:include-filter>`：表示子节点想要包含的目标类。
- `<context:exclude-filter>`：表示子节点想要排除在外的目标类。
- `<context:component-scan>`下可以拥有若干个`<context:include-filter>`和`<context:exclude-filter>`。

代码如下所示：

```java
package com.example.springdemo.annotation;

import org.springframework.stereotype.Component;

@Component
public class TestObject {
}
```

```java
package com.example.springdemo.annotation.service;

import org.springframework.stereotype.Service;

@Service
public class UserService {
    public void add() {
        System.out.println("UserService add()...");
    }
}

```

```java
package com.example.springdemo.annotation.repository;

public interface UserRepository {
    void save();
}
```

```java
package com.example.springdemo.annotation.repository;

import org.springframework.stereotype.Repository;

@Repository("userRepository")
public class UserRepositoryImpl implements UserRepository {
    @Override
    public void save() {
        System.out.println("UserRepository save()...");
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <context:component-scan base-package="com.example.springdemo.annotation"/>
</beans>
```

```java
public class Main {
    @Test
    public void testAnnotation() {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans_annotation.xml");

        TestObject to = (TestObject) context.getBean("testObject");
        System.out.println(to);

        UserController userController = (UserController) context.getBean("userController");
        System.out.println(userController);

        UserService userService = (UserService) context.getBean("userService");
        System.out.println(userService);

        UserRepository userRepository = (UserRepository) context.getBean("userRepository");
        System.out.println(userRepository);
    }
}
```

输出结果如下：

```java
com.example.springdemo.annotation.TestObject@47af7f3d
com.example.springdemo.annotation.controller.UserController@7c729a55
com.example.springdemo.annotation.service.UserService@3bb9a3ff
com.example.springdemo.annotation.repository.UserRepositoryImpl@661972b0
```

此外，可以通过 resource-pattern 指定扫描的资源，如下所示：

```xml
<context:component-scan base-package="com.example.springdemo.annotation"
	resource-pattern="repository/*.class"></context:component-scan>
```

context:exclude-filter 的使用：

```xml
<context:component-scan base-package="com.example.springdemo.annotation">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

需要注意的是，expression 中需要填写的是某个注解的包名。

context:include-filter 的使用：

```xml
<context:component-scan base-package="com.example.springdemo.annotation"
                            use-default-filters="false">
	<context:include-filter type="annotation" expression="org.springframework.stereotype.Repository"/>
</context:component-scan>
```

需要说明的是，当使用 context:include-filter 时，需要指定`use-default-filters="false"`，只有这样才能仅扫描到所包含的组件。

此外，在开发中使用最多的就是组件装配。`<context:component-scan>`元素会自动注册 AutowiredAnnotationBeanPostProcessor 实例，该实例可以自动装配具有 @Autowired、@Resource、@Inject 的注解属性。如下所示：

```java
@Controller
public class UserController {

    @Autowired
    private UserService userService;

    public void execute() {
        System.out.println("UserController execute()...");
        userService.add();
    }
}
```

```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    public void add() {
        System.out.println("UserService add()...");
        userRepository.save();
    }
}
```

输出结果如下：

```java
com.example.springdemo.annotation.controller.UserController@6b81ce95
UserController execute()...
UserService add()...
UserRepository save()...
```

这个 @Autowired 注解会自动装配具有兼容类型的单个 Bena 属性。它可以作用于构造器、普通字段、具有参数的方法上面。

默认情况下，所有使用 @Autowired 注解的属性都需要被设置。当 Spring 找不到匹配的 Bean 装配属性时，则会抛出异常。如果某一属性允许不被设置，可以通过设置 @Autowired 注解的 required 属性为 false。

默认情况下，当 IoC 容器里存在多个类型兼容的 Bean 时，通过类型的自动装配将无法工作。此时可以在 @Qualifier 注解里提供 Bean 的名称。Spring 允许对方法的入参标注 @Qualifiter，以指定注入 Bean 的名称。

如果我们在 UserRepositoryImpl 类中引用了一个没有被设置为 Bean 的 TestObject 类时，那么此时会抛出如下异常：

```java
public class TestObject {

}
```

```java
@Repository("userRepository")
public class UserRepositoryImpl implements UserRepository {

    @Autowired
    private TestObject testObject;

    @Override
    public void save() {
        System.out.println("UserRepository save()...");
        System.out.println(testObject);
    }
}
```

```java
No qualifying bean of type 'com.example.springdemo.annotation.TestObject' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```

此时，可以在 TestObject 的 @Autowired 注解上添加 required = false，那么程序运行后，testObject 的值就为 null。

## 9. 泛型依赖注入

Spring 4.x 中可以为子类注入子类对应的泛型类型的成员变量的引用。如下所示：

```java
public class User {
}
```

```java
public class BaseRepository<T> {
}
```

```java
public class BaseService<T> {

    @Autowired
    protected BaseRepository<T> repository;

    public void add() {
        System.out.println("add()...");
        System.out.println(repository);
    }
}
```

```java
@Repository
public class UserRepository extends BaseRepository<User> {
}
```

```java
@Service
public class UserService extends BaseService<User>{
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans-4.2.xsd http://www.springframework.org/schema/context
       http://www.springframework.org/schema/context/spring-context-4.2.xsd">

    <context:component-scan base-package="com.example.springdemo.generic.di"/>
</beans>
```

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("beans_generic_di.xml");
        UserService userService = (UserService) context.getBean("userService");

        userService.add();
    }
}
```

输出结果如下所示：

```java
add()...
com.example.springdemo.generic.di.UserRepository@23f7d05d
```

