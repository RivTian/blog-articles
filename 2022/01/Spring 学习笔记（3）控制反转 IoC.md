本篇文章主要对 Spring 框架中的核心功能之一**控制反转 (IOC，Inversion of Control)**进行介绍，采用**理论 + 实战**的方式给大家阐述其中的原理以及明确需要注意的地方。

## 控制反转

通俗的讲，控制反转是一种思想，也是工程设计的一种原则，通过控制反转可以将对象的一部分控制权转移到容器（例如 Spring 容器）中，由容器帮我们控制创建对象，我们只需要在上下文中进行配置即可，配置的过程其实就是创建对象的过程。

其思想是：**反转资源获取的方向**。传统的资源查找的方向是要求组件向容器发起请求查找资源，同时容器返回资源。而采用 IoC 之后，容器会主动的将资源推送给它所管理的组件，组件所要做的就是选择一种合适的方式来接收资源。

而 DI（Denpendency Injection）是 IoC 的另一种表述方式，即**组件以一些预先定义好的方式（例如 setter 方法）接收来自容器的资源注入**。

> 此外还可以用构造函数的方式注入

### 这种实现方式有什么好处呢？

首先，在传统的程序设计中，我们想要使用某个对象就需要 new 一个对象，创建对象的实例，这是我们主动创建的过程，而引入 IoC 之后，我们将这个创建对象的过程交给容器去做，由容器去控制对象的创建，当然也可以注入依赖对象，而这个过程是对象被动的接受依赖对象的。这里的容器就是我们正在学习的 Spring，它可以创建对象的实例，当我们需要对象时，直接在 Spring 容器中获得即可，直接去拿就好了。

### 如何拿到呢？

这时就需要用到上下文配置文件了，也就是我们常说的`.xml`配置文件，我们只需要在配置文件中将指定的实现类的路径名（全限定名）配置到 xml 文件中即可。

你看，通过这种方式可以很好的将创建对象的过程与我们主业务进行很好的分离，更好的降低耦合度，因为我们让 IoC 容器帮助我们去找对象并将相应的依赖注入到对象中，这时我们就不需要再主动去创建对象了。简单点说，就是以后需要写 new 的地方，我们都可以交给 IoC 容器去做，在这里也就是让 Spring 去做就可以了。

当然，这个地方配合**依赖注入**(DI，Dependency Injection) 来讲可能会好一点，不过没关系，博主将通过一个简单的小例子来让你明白控制反转的具体操作时什么样子的。

## IoC 简单案例

在进行编码之前，我们需要一些准备工作，如环境配置说明、导入相应 jar 包、创建目标类等等，下面我们一步一步来做。

### 1.1 环境说明

以下声明了所有使用的编码环境，已经确定能够在该环境下进行所有的编码测试等，关于 jar 包，我们用到的时候在说。

- JDK：11
- 软件：IDEA 2021.2.1
- 操作系统：Windows 10

### 1.2 导入相应 jar 包

这里的 jar 暂时用到的是 4+1，即 4 个核心(core、beans、beans、expression)和 1 个依赖(commons.logging)，这些包你可以在官网下载。如下所示：

- spring-core-3.2.0.RELEASE.jar
- spring-beans-3.2.0.RELEASE.jar
- spring-beans-3.2.0.RELEASE.jar
- spring-expression-3.2.0.RELEASE.jar
- com.springsource.org.apache.commons.logging-1.1.1.jar

### 1.3 利用 Maven 快速配置 Spring 开发环境

详细方法：[Click Here](https://www.cnblogs.com/RioTian/p/15829108.html)

### 1.4 编写目标接口与实现类

我们在这里创建一个 UserService 接口，然后再创建它的一个实现类 UserServiceImpl，类里面由一个添加用户的方法 save()，然后方法里面输出一条语句即可。如下所示：

```Java
// UserService.java
public interface UserService {
    public void save();
}
```

```java
// UserServiceImpl.java
public class UserServiceimpl implements UserService {

    public void save() {
        System.out.println("ioc...save");
    }
}
```

### 1.5 编写测试类

然后我们再写一个测试类，用于对目标类进行测试，如下所示：

```java
// SpringTest.java
public class SpringTest {
    @Test
    public void test1() {
        // 之前开发时使用的方式: new
        UserService userService = new UserServiceimpl();
        userService.save();
    }
}
```

这时运行测试类就得到了输出语句。你可以看到，在测试类中，我们使用以前的方法 new 了一个对象，然后调用了 addUser() 方法，最后将输出语句打印。这显然不是我们目的，我们要实现的是用 Spring 进行 IoC 后，得到同样的效果。

### 1.6 编写配置文件

下面就开始创建一个配置文件，我们暂且将其命名为 `bean.xml`(开发中一般将其命名为 `applicationContext.xml`)。配置文件一般放置于 `src` 目录下，如下所示：

```xml
<!-- bean.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	   xsi:schemaLocation="http://www.springframework.org/schema/beans
       				       http://www.springframework.org/schema/beans/spring-beans.xsd">
	
    <!-- 配置service-->
	<bean id="userServiceId" class="com.my.a_ioc.UserServiceImpl"></bean>

</beans>
```

在 `<beans>` 标签中的像网址一样的链接表示 schema 约束(模式约束)，由于我们要使用到 `<bean>` 标签，该标签可以帮助我们**配置需要创建的对象**，所以这里应该将其添加进来。接下来你可以看到在 `<bean>` 标签中有两个属性，其中 `id` 是用于从 Spring 容器中获得实例的，`class` 表示创建实例的全限定名，即**包名+类名**。

`bean.xml` 配置文件写好以后，我们需要再编写一个测试类去测试它，这个测试类将演示如何从 Spring 容器中获取实例，如下所示：

```java
@Test
public void Test2() {
    //1、获得容器
    String xmlPath = "src/main/resources/bean.xml";
    // File Not Find？
    //ApplicationContext app = new ClassPathXmlApplicationContext(xmlPath);
    ApplicationContext app = new ClassPathXmlApplicationContext("bean.xml");
    //2、获得内容，不需要自己new，直接从spring容器中获取: getBean 方法
    UserService userService = (UserService) app.getBean("userService");
    userService.save();
}
```

首先要指定`bean.xml`文件的路径，这里可以右击配置文件，选择`Copy Qualified Name`即可，然后将`com`之前的部分删掉。这里通过`ClassPathXmlApplicationContext`加载`bean.xml`文件，返回一个`ApplicationContext`对象，此过程实例化一个 IoC 容器。

然后通过`getBean`方法获取配置的`bean`，参数`userServiceId`与之前在`bean.xml`文件中的`id`是相同的。由于其返回的类型是`Object`，所以在这里我们将其强转成`UserService`类型的，最后调用`addUser()`方法即可。输出结果与之前实现的方法相同。

## 2.总结

至此，一个简单的 IoC 示例就完成了。让我们再梳理一下 IoC 容器是如何工作的。

首先是配置文件的准备，在配置文件中声明`bean`，为`bean`配置元数据，例如：`id`、`class`，然后 IoC 容器会对配置文件中的元数据进行解析，得到元数据后进行实例化、配置及组装`bean`操作，最后在测试类中实例化容器，获取需要的`bean`。主要的过程是对`bean`的定义以及元数据的配置。

如果你认为文章有错误或者描述不准确的话，请与我联系，谢谢！。另外，你还可以参考下面的链接进一步加深对 IoC 的理解。

## 参考

- [【第二章】IoC 之 2.1 IoC 基础 —— 跟我学 Spring3](https://jinnianshilongnian.iteye.com/blog/1413846)

