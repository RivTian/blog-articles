本篇文章主要对 Spring 框架进行整体介绍，包括其核心功能模块与体系结构，让大家对该框架有个大体的认识。

## 1. 前景提要

如果你之前学过 Servlet 的话，那么一定会对 [MVC](https://zh.wikipedia.org/wiki/MVC) 分层概念有所了解，我们一般在做项目的时候，会将与数据库操作（比如增、删、改、查）相关的部分放在 Dao 层，将登录、注册等功能放在 Service 层，而将用户获取表单数据、调用业务逻辑、分发转向等操作放在 Web 层（也叫表示层，具体的说，应该是在表示层下的控制层）。Servlet 的主要功能是交互式地浏览和交互数据，生成动态的 Web 内容，因此其可以响应任何类型的请求。

一个清晰的结构图如下：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220121144534.png)

## 2. Spring 介绍

这时候你可能会问了：介绍 Servlet 干嘛？这和 Spring 有什么关系吗？其实，我这里是为了引出 Spring 框架，接下来会通过结构图的形式让你知道 Spring 处在什么位置。如下所示：

你可以看到，不管是 SSH 框架还是 SSM 框架，底层都需要 Spring 的支持，所以说，学好 Spring 极其重要。

Spring 其实是一种容器框架，基于**控制反转** (IoC，Inversion of Control) 与**依赖注入** (DI，Dependency Injection) 进行实现，我们暂且将这两个名词理解为：通过 IoC 和 DI 操作将**对象**交付给 Spring，让 Spring 容器来控制**对象**，也就是我们平时 new 的**对象**不在程序中 new 了，而是让 Spring 帮助我们去做，我们只需要在配置文件中进行配置就可以了（当然可以在程序中标记注解），后面几篇文章会详细讲到。

## 3. Spring 体系结构

Spring 框架是一个分层的架构，每个层次有许多模块，在这里我们先主要关注 **Core Container**、**AOP**、**Aspects**、**JDBC**、**Transactions** 这几个部分，如下所示：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220121144643.png)

**Core Container** 部分是 Spring 的核心容器，其中 **Beans** 用于管理各种 bean，**Core** 是容器的核心，这两个部分是 Spring 的基本内容，包括控制反转与依赖注入。**Context** 指的是上下文，是 Spring 的配置文件。**SpEl** 模块提供了强大的表达式语言(就如同 JSP 中的EL表达式一样)，可以在程序运行时查询和操作对象。**AOP** 是切面编程的部分，而 **Aspect** 是一种 AOP 框架。

在 Data Access/Integration 中的 **JDBC** 用于 JDBCTemplate 的数据库开发，**Transactions** 用于事务的管理。当然，Web 层还有用于 Web 的开发。主要是这么多内容，这些内容都会在以后的文章中进行介绍，也会编写例子来加深对 Spring 的理解。

## 4. Spring 特点

说完了 Spring 的结构，我们来看一下它有哪些优点。

1. Spring 相当与一个容器，也称之为工厂，它可以用于创建对象以及维护对象之间的依赖关系，Spring 通过管理这些对象可以简化我们的开发，降低程序的耦合性。
2. Spring 支持 AOP 编程，也就是面向切面编程。这个切面就相当于在程序中的某个地方进行了拦截，拦截后会执行其它操作，等到操作执行完之后就回过头来继续执行原来的程序，用到地方最多的是对账号登录的权限进行拦截，判断有没有权限，如果有权限就放行，没有权限就执行另外的操作。
3. 以前我们在操作数据库的事务的时候需要自己手动编写代码，而引入 Spring 后，我们只需在 Spring 的配置文件中进行配置就可以实现事务的管理了。
4. 以前我们在测试程序的时候需要单独写一个测试类，而 Spring 支持 Junit4，可以通过注解的方式测试 Spring 程序。
5. Spring 可以很方便的集成其它框架，从上面第二张图可以看出，Spring 可以与 Structs、Hibernate、MyBatis 等框架进行整合。
6. 以前我们在实现对数据库操作的时候可以通过 JDBC 实现，这种 API 的方式很是繁琐，而 Spring 很好的实现了 API 的封装，从而使得这些 API 的使用难度大大降低。

## 5. 初探 Spring

先看下面的代码：

```java
// HelloWorld.java
public class HelloWorld {
    private String name;
    private int age;

    public void setName(String name) {
        this.name = name;
    }
    
    public void setAge(int age) {
        this.age = age;
    }

    public void hello() {
        System.out.println("hello: " + name);
    }
}
```

```java
// Main.java
public class Main {
    public static void main(String[] args) {
        HelloWorld helloWorld = new HelloWorld();
        helloWorld.setName("Carol");
        helloWorld.setAge(123);

        helloWorld.hello();
    }
}
```

上面的代码很简单，在 Main 这个类中，首先创建了一个 HelloWorld 对象，然后对 name 和 age 进行了属性填充，最后调用其 hello 方法将内容输出。

然而，引入 Spring 以后，「创建对象」和「填充属性」的工作可以交给 Spring 来做。那么该如何操作呢？

在项目工程的 `src/main/resources` 目录下，创建一个 Spring 配置文件，即 applicationContext.xml。其内容如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--配置 bean-->
    <bean id="helloWorld" class="com.example.springdemo.beans.HelloWorld" >
        <property name="name" value="Spring"></property>
        <property name="age" value="123"></property>
    </bean>
</beans>
```

其中，`class`需要指定当前类的全限定名，Spring 通过反射的方式来创建 HelloWorld 对象；`id`通常为首字母小写的类名，用来标识这个对象的同时，引用这个 Bean。Bean 的定义中可以有零个或多个`<property></property>`。`<property></property>`被称为**元素**，元素中有多个**属性**，即`name`和`value`。每个 property 元素对应 JavaBean 中相应的 setter 方法。

此时，可以改写 main 方法，如下所示：

```java
public class Main {
    public static void main(String[] args) {
        // 创建 Spring 的 IoC 容器对象
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

        // 从 IoC 容器中获取 Bean 实例
        HelloWorld helloWorld = (HelloWorld) context.getBean("helloWorld");

        helloWorld.hello();
    }
}
```

输出内容如下：

```java
hello: Spring, age: 123
```

可以看到，Spring 容器自动为属性进行了赋值。为了看到 Spring 在创建 IoC 容器对象的时候做了哪些事情，我们在 main 方法中将后两行代码注释掉，然后在 HelloWorld 类中添加一个无参的构造器，同时在 setter 方法中加入输出语句，如下所示：

```java
public class HelloWorld {
    private String name;
    private int age;

    public void setName(String name) {
        System.out.println("setName: " + name);
        this.name = name;
    }

    public void setAge(int age) {
        System.out.println("setAge: " + age);
        this.age = age;
    }

    public void hello() {
        System.out.println("hello: " + name + ", age: " + age);
    }

    public HelloWorld() {
        System.out.println("HelloWorld's Constructor...");
    }
}
```

运行程序后，输出内容如下：

```java
HelloWorld's Constructor...
setName: Spring
setAge: 123
```

可以看到，Spring 在创建 IoC 容器的时候，就执行了无参构造器，并调用了 setter 方法。

需要注意的是：由于在 xml 中使用全类名的方式，通过反射来创建对象。因此需要当前类提供无参的构造器，如果没有提供的话，那么在运行程序的时候，会出现如下错误：

```java
Instantiation of bean failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [com.example.springdemo.beans.HelloWorld]: No default constructor found; nested exception is java.lang.NoSuchMethodException: com.example.springdemo.beans.HelloWorld.<init>()
```

## 6. 总结

这是《Spring 学习笔记》的第一篇文章，主要围绕 Spring 是什么？Spring 可以做什么？以及 Spring 的特点进行介绍，通过引入了结构图可以很清晰的看到 Spring 的内部结构特点，便于自己的理解。当然了，博主只是简单的对这部分内容进行了说明，如果感兴趣的话，可以查看下方的官方文档进行阅读。

此外，如果有任何错误或者描述不正确的地方，欢迎以评论或邮箱的方式联系我。

## 参考

- [Spring 官方文档](https://docs.spring.io/spring/docs/4.3.24.RELEASE/spring-framework-reference/html/overview.html)
- [廖雪峰 Spring 教程](https://www.liaoxuefeng.com/wiki/1252599548343744/1266263217140032) 

