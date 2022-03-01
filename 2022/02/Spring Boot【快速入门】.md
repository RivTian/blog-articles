![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208115942.png)

## Spring Boot 概述

> **Build Anything with Spring Boot：** Spring Boot is the starting point for building all Spring-based applications. Spring Boot is designed to get you up and running as quickly as possible, with minimal upfront configuration of Spring.

上面是引自官网的一段话，大概是说： Spring Boot 是所有基于 Spring 开发的项目的起点。Spring Boot 的设计是为了让你尽可能快的跑起来 Spring 应用程序并且尽可能减少你的配置文件。

#### 什么是 Spring Boot

- 它使用 “习惯优于配置” （项目中存在大量的配置，此外还内置一个习惯性的配置，让你无须）的理念让你的项目快速运行起来。
- 它并不是什么新的框架，而是默认配置了很多框架的使用方式，就像 Maven 整合了所有的 jar 包一样，Spring Boot 整合了所有框架（引自：[Spring Boot(一)：入门篇——纯洁的微笑](http://www.ityouknow.com/springboot/2016/01/06/springboot(一)-入门篇.html)）

#### 使用 Spring Boot 有什么好处

回顾我们之前的 SSM 项目，搭建过程还是比较繁琐的，需要：

- 1）配置 web.xml，加载 Spring 和 Spring MVC
- 2）配置数据库连接、配置日志文件
- 3）配置家在配置文件的读取，开启注解
- 4）配置mapper文件
- **…..**

而使用 Spring Boot 来开发项目则只需要非常少的几个配置就可以搭建起来一个 Web 项目，并且利用 IDEA 可以自动生成生成，这简直是太爽了…

- 划重点：简单、快速、方便地搭建项目；对主流开发框架的无配置集成；极大提高了开发、部署效率。

## Spring Boot 快速搭建

**步骤①**：创建新模块，选择Spring Initializr，并配置模块相关基础信息

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208131204.png)

<font color="#ff0000"><b>特别关注</b></font>：点击Next时，IDEA 需要联网状态才可以进入到后面那一页，如果不能正常联网，就无法正确到达右面那个设置页了，会一直<font color="#ff0000"><b>联网</b></font>转转转

<font color="#ff0000"><b>特别关注</b></font>：选择java版本和你计算机上安装的JDK版本匹配即可，但是最低要求为JDK8或以上版本，推荐使用8或11

**步骤②**：选择当前模块需要使用的技术集

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208131246.png)

按照要求，左侧选择web，然后在中间选择Spring Web即可，选完右侧就出现了新的内容项，这就表示勾选成功了

> <font color="#ff0000"><b>关注</b></font>：此处选择的 Spring Boot 的版本使用默认的就可以了，需要说一点，Spring Boot 的版本升级速度很快，可能昨天创建工程的时候默认版本是2.5.4，今天再创建工程默认版本就变成2.5.5了，差别不大，无需过于纠结，回头可以到配置文件中修改对应的版本

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208131605.png)

项目建立后结构还是看上去挺清爽的，少了很多配置文件，我们来了解一下默认生成的有什么：

- `SpringbootApplication`： 一个带有 main() 方法的类，用于启动应用程序
- `SpringbootApplicationTests`：一个空的 Junit 测试了，它加载了一个使用 Spring Boot 字典配置功能的 Spring 应用程序上下文
- `application.properties`：一个空的 properties 文件，可以根据需要添加配置属性
- pom.xml： Maven 构建说明文件

> Tips：如何隐藏指定的文件或文件夹信息
>
> 看惯了使用 Maven 模式下创建的项目结构，Spring Boot 创建的结构还是特别别扭，处理方案无外乎两种，如果你对每一个文件/目录足够了解，没有用的完全可以删除掉，或者不删除，但是看着别扭，就设置文件为看不到就行了。删除不说了，直接Delete掉就好了。既然是在Idea下做隐藏功能，肯定隶属于Idea的设置，设置方式如下。
>
> **步骤①**：【Files】→【Settings】
>
> **步骤②**：【Editor】→【File Types】→【Ignored Files and Folders】
>
> **步骤③**： 输入要隐藏的名称，支持*号通配符，回车确认添加

**步骤③**：开发控制器类

```java
//Rest模式
//@RestController 注解： 该注解是 @Controller 和 @ResponseBody 注解的合体版
@RestController
@RequestMapping("/books")
public class BookController {
    @GetMapping
    public String getById(){
        System.out.println("springboot is running...");
        return "springboot is running...";
    }
}
```

**步骤④：**利用 IDEA 启动 Spring Boot

我们回到 `SpringbootApplication` 这个类中，然后右键点击运行：

- **注意**：我们之所以在上面的项目中没有手动的去配置 Tomcat 服务器，是因为 Spring Boot 内置了 Tomcat

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208131750.png)

​	使用带main方法的java程序的运行形式来运行程序，运行完毕后，控制台输出上述信息。

​	不难看出，运行的信息中包含了8080的端口，Tomcat这种熟悉的字样，难道这里启动了Tomcat服务器？是的，这里已经启动了。那服务器没有配置，哪里来的呢？后面再说。现在你就可以通过浏览器访问请求的路径，测试功能是否工作正常了

访问路径：	http://localhost:8080/books

![Seg-Static](https://cdn.jsdelivr.net/gh/Kanna-jiahe/blogimage/img/20210419185017.png)



## 解析 Spring Boot 项目

是不是感觉很神奇？目前的效果其实依赖的底层逻辑还是很复杂的，但是从开发者角度来看，目前只有两个文件展现到了开发者面前

### 解析 pom.xml 文件

这是maven的配置文件，描述了当前工程构建时相应的配置信息

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.6.3</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.riotian</groupId>
    <artifactId>SpringBoot_01_quickstart</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringBoot_01_quickstart</name>
    <description>SpringBoot_01_quickstart</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

我们可以看到一个比较陌生一些的标签 `<parent>` ，这个标签是在配置 Spring Boot 的父级依赖：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.6.3</version>
    <relativePath/> <!-- lookup parent from repository -->
</parent>
```

有了这个，当前的项目才是 Spring Boot 项目，spring-boot-starter-parent 是一个特殊的 starter ，它用来提供相关的 Maven 默认依赖，**使用它之后，常用的包依赖就可以省去 version 标签。**

关于具体 Spring Boot 提供了哪些 jar 包的依赖，我们可以到本地 Maven 仓库下来查看

### 应用入口类

Spring Boot 项目通常有一个名为 **Application** 的入口类，入口类里有一个 main 方法， **这个 main 方法其实就是一个标准的 Java 应用的入口方法。**

**@SpringBootApplication** 是 Spring Boot 的核心注解，它是一个组合注解，该注解组合了：**@Configuration、@EnableAutoConfiguration、@ComponentScan；** 若不是用 @SpringBootApplication 注解也可以使用这三个注解代替。

- 其中，**@EnableAutoConfiguration 让 Spring Boot 根据类路径中的 jar 包依赖为当前项目进行自动配置**，例如，添加了 spring-boot-starter-web 依赖，会自动添加 Tomcat 和 Spring MVC 的依赖，那么 Spring Boot 会对 Tomcat 和 Spring MVC 进行自动配置。
- **Spring Boot 还会自动扫描 @SpringBootApplication 所在类的同级包以及下级包里的 Bean** ，所以入口类建议就配置在 grounpID + arctifactID 组合的包名下（这里为 cn.wmyskxz.springboot 包）



通过上面的实践，我们不难发现，Spring Boot程序简直太好写了，几乎什么都没写，功能就有了，这也是Spring Boot技术为什么现在这么火的原因，和 Spirng 程序相比，Spring Boot程序在开发的过程中各个层面均具有优势

| **类配置文件**          | **Spring**   | **Spring Boot** |
| ----------------------- | ------------ | --------------- |
| pom 文件中的坐标        | **手工添加** | **勾选添加**    |
| web3.0配置类            | **手工制作** | **无**          |
| Spring/Spring MVC配置类 | **手工制作** | **无**          |
| 控制器                  | **手工制作** | **手工制作**    |

一句话总结一下就是<font color="#ff0000"><b>能少写就少写</b></font>，<font color="#ff0000"><b>能不写就不写</b></font>，这就是Spring Boot技术给我们带来的好处，行了，现在你就可以动手做一做Spring Boot程序了，看看效果如何，是否真的帮助你简化开发了

### Spring Boot 的配置文件

> Spring Boot 的全局配置文件的作用是对一些默认配置的配置值进行修改。

Spring Boot 使用一个全局的配置文件 `application.properties` 或 `application.yml`，放置在【src/main/resources】目录或者类路径的 /config 下。

Spring Boot 不仅支持常规的 properties 配置文件，还支持 yaml 语言的配置文件。yaml 是以数据为中心的语言，在配置数据的时候具有面向对象的特征。

简单实例一下

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208132513.png)

我们同样的将 Tomcat 默认端口设置为 8080 ，并将默认的访问路径从 “`/`” 修改为 “`/hello`” 时，使用 properties 文件和 yml 文件的区别如上图。

- 注意： yml 需要在 “`:`” 后加一个空格，幸好 IDEA 很好地支持了 yml 文件的格式有良好的代码提示
- 在此，我们可以自己配置多个属性

我们直接把 .properties 后缀的文件删掉，使用 .yml 文件来进行简单的配置，然后使用 @Value 来获取配置属性：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208133011.png)

重启 Spring Boot ，输入地址：[localhost:8080/books](http://localhost:8080/books) 能看到正确的结果：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208132920.png)

**注意：** 我们并没有在 yml 文件中注明属性的类型，而是在使用的时候定义的。

你也可以在配置文件中使用当前配置，仍然可以得到正确的结果

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208133234.png)

 这样写配置文件繁琐而且可能会造成类的臃肿，因为有许许多多的 @Value 注解。

针对 @Value 多的情况来说，我们可以 **封装配置信息**、以此来解决问题

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208140251.png)

> 关于这里出现的 `Spring Boot Configuration Annotation Processor not configured` 
>
> 在 pom.xml 中引入依赖后刷新即可解决
>
> ```xml
> <dependency>
>     <groupId>org.springframework.boot</groupId>
>     <artifactId>spring-boot-configuration-processor</artifactId>
>     <optional>true</optional>
> </dependency>
> ```



我们可以把配置信息封装成一个类，首先在我们的 name 和 age 前加一个 student 前缀，然后新建一个 StudentProperties 的类用来封装这些信息，并用上两个注解：

- @Component：表明当前类是一个 Java Bean
- @ConfigurationProperties(prefix = “student”)：表示获取前缀为 sutdent 的配置信息
- @Date，@NoArgsConstructor，@AllArgsConstructor 是 LomBok 插件的注解，可以自动生成 Set 和 Get 以及构造方法

这样我们就可以在控制器中使用，重启得到正确信息：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208140446.png)

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208140457.png)

## Spring Boot 热部署

在目前的 Spring Boot 项目中，当发生了任何修改之后我们都需要重新启动才能够正确的得到效果，这样会略显麻烦，Spring Boot 提供了热部署的方式，当发现任何类发生了改变，就会通过 JVM 类加载的方式，加载最新的类到虚拟机中，这样就不需要重新启动也能看到修改后的效果了。

* 做法很简单，修改 pomxml 即可！

我们往 pom.xml 中添加一个依赖就可以了：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional> <!-- 这个需要为 true 热部署才有效 -->
</dependency>
```

重新启动 Spring Boot ，然后修改任意代码，就能观察到控制台的自动重启现象：

> 关于如何在 IDEA 中配置热部署：[传送门](https://blog.csdn.net/xusheng_Mr/article/details/78771746)

## Spring Boot 中使用 .JSP

上面已经完成了 Spring Boot 项目的简单搭建，我们仅仅需要进行一些简单的设置，写一个 HelloController 就能够直接运行了，不要太简单…接下来我们再深入了解一下 Spring Boot 的使用。

### Spring Boot 支持 JSP

Spring Boot 的默认视图支持是  [Thymeleaf](#)  模板引擎，但是这个我们不熟悉啊，我们还是想要使用 JSP 怎么办呢？

- 第一步：修改 pom.xml 增加对 JSP 文件的支持

  ```xml
  <!-- servlet依赖. -->
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <scope>provided</scope>
  </dependency>
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>jstl</artifactId>
  </dependency>
  
  <!-- tomcat的支持.-->
  <dependency>
      <groupId>org.apache.tomcat.embed</groupId>
      <artifactId>tomcat-embed-jasper</artifactId>
      <scope>provided</scope>
  </dependency>
  ```

- 第二步：配置试图重定向 JSP 文件的位置

  修改 application.yml 文件，将我们的 JSP 文件重定向到 /WEB-INF/views/ 目录下：

  ![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208160424.png)

- 第三步：编写 HelloController

  ```java
  @Controller
  public class HelloController {
  
      public String hello(Model model) {
          model.addAttribute("now", DateFormat.getDateTimeInstance().format(new Date()));
          return "hello";
      }
  
  }
  ```

* 第四步：新建 hello.jsp 文件

  在【src/main】目录下依次创建 webapp、WEB-INF、views 目录，并创建一个 hello.jsp 文件：

  ```jsp
  <%@ page contentType="text/html;charset=UTF-8" language="java" pageEncoding="UTF-8" %>
  <html>
      <head>
          <title>index.jsp</title>
      </head>
      <body>
      <h2>Hi, JSP ,现在时间是 ${now}</h2>
      </body>
  </html>
  ```

* 第五步：查阅结果

  如果部署了热部署功能，只需要等待控制台重启信息完成之后再刷新网页就可以看到正确效果了：

  ![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208163352.png)

### 关于运行项目访问时出现的 404 解决方案

错误原因：在集成JSP的时候我们要用插件来运行，不然就会报错。

解决办法：

首先确认 application.properties/application.yaml 中配置未写错

```properties
spring.mvc.view.prefix=/WEB-INF/views/
spring.mvc.view.suffix=.jsp
------------------------------------
server:
  port: 8080
spring:
  mvc:
    view:
      prefix: /WEB-INF/views/
      suffix: .jsp
```

<font color="#ff0000">Clean</font> 并刷新 <font color="#ff0000">maven</font>

运行要使用`spring-boot:run`【重点】，重启并访问`localhost:8080/hello`

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208163826.png)

## 集成 MyBatis

- 第一步：修改 pom.xml 增加对 MySql 和 MyBatis 的支持

  ```xml
  <!-- mybatis -->
  <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.1.1</version>
  </dependency>
  <!-- mysql -->
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
  </dependency>
  ```

- 第二步：新增数据库链接参数

  ```yaml
  # application.yaml
  server:
    port: 8080
  spring:
    mvc:
      view:
        prefix: /WEB-INF/views/
        suffix: .jsp
    datasource:
      url: jdbc:mysql://localhost:3306/javat?useSSL=false&characterEncoding=UTF-8
      username: root
      password: kokoro
      driver-class-name: com.mysql.jdbc.Driver
    jpa:
      hibernate:
        ddl-auto: update # 新增数据库链接必备的参数
  ```

- 第三步：创建 User 实体类和 UserMapper 映射类

  在【com.riotian】下新建一个【domain】包，然后在其下创建一个 User 类：

  ```java
  @Data
  @AllArgsConstructor
  @NoArgsConstructor
  public class User {
      private int id;
      private String username;
      private String password;
  }
  ```

  在【com.riotian】下新建一个【mapper】包，然后在其下创建一个 UserMapper 接口：

  ```java
  @Mapper
  public interface UserMapper {
  
      @Select("select * from user")
      List<User> findAll();
  
  }
  ```

* 第四步：编写 UserController

  在【com.riotian.Controller】包中创建一个 UserController：

  ```java
  @Controller
  public class UserController {
  
      @Autowired
      UserMapper userMapper;
  
      @RequestMapping("/listUser")
      public String listUser(Model model) {
          List<User> userList = userMapper.findAll();
          model.addAttribute("userList", userList);
          return "listUser";
      }
  
  }
  ```

* 第五步：编写 listUser.jsp 文件

  简化一下 JSP 的文件，仅显示两个字段的数据：

  ```jsp
  <%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
  <%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c"%>
  <table align='center' border='1' cellspacing='0'>
      <tr>
          <td>id</td>
          <td>username</td>
  
      </tr>
      <c:forEach items="${userList}" var="user">
          <tr>
              <td>${user.id}</td>
              <td>${user.username}</td>
          </tr>
      </c:forEach>
  </table>
  ```

* 第六步：重启服务器运行

  因为往 pom.xml 中新增加了依赖的包，所以自动重启服务器没有作用，我们需要手动重启一次，然后在地址输入：[localhost:8080/listUser](http://localhost:8080/listUser) 查看效果：

  ![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220208171456.png)

## 参考文章

* Fate.gitee.io-Spring Boot合集
* how2j.cn-Spring Boot 系列教程