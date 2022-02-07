## MyBatis 简介

MyBatis 是一个开源、轻量级的数据持久化框架，是 JDBC 和 Hibernate 的替代方案。MyBatis 内部封装了 JDBC，简化了加载驱动、创建连接、创建 statement 等繁杂的过程，开发者只需要关注 SQL 语句本身。

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220205155218.png)

- **持久层：** 可以将业务数据**存储到磁盘，具备长期存储能力**，只要磁盘不损坏，在断电或者其他情况下，重新开启系统仍然可以读取到这些数据。
- **优点：** 可以**使用巨大的磁盘空间**存储相当量的数据，并且很**廉价**
- **缺点：慢**（相对于内存而言）

### 为什么使用 MyBatis

在我们**传统的 JDBC 中**，我们除了需要自己提供 SQL 外，还必须操作 Connection、Statment、ResultSet，不仅如此，为了访问不同的表，不同字段的数据，我们需要些很多雷同模板化的代码，闲的**繁琐又枯燥**。

而我们在使用了 **MyBatis** 之后，**只需要提供 SQL 语句就好了**，其余的诸如：建立连接、操作 Statment、ResultSet，处理 JDBC 相关异常等等都可以交给 MyBatis 去处理，我们的**关注点于是可以就此集中在 SQL 语句上**，关注在增删改查这些操作层面上。

并且 MyBatis 支持使用简单的 XML 或注解来配置和映射原生信息，将接口和 Java 的 POJOs(Plain Old Java Objects,普通的 Java对象)映射成数据库中的记录。

## 搭建 MyBatis 环境

在实际开发之前，我们必须为 MyBatis 搭建适当的环境。下面介绍 MyBatis 的下载以及目录结构。

### MyBatis下载

在 MyBatis 的官方网站 [http://mybatis.org](http://mybatis.org/)，可以下载到最新版本的 MyBatis，本教程使用版本为 MyBatis 3.5.5。

如果您打不开网站或下载进度较慢，可以通过 https://github.com/mybatis/mybatis-3/releases 网址下载。

> 官方还提供了文档： [戳这里](http://www.mybatis.org/mybatis-3/zh/index.html)，虽然感觉写得一般，但还是有一些参考价值…唉，别当教程看，当字典看！

下载好 MyBatis 的包解压后，可以得到以下的文件目录：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220205155608.png)

其中 mybatis-3.5.9.jar 包就是 MyBatis 的项目工程包，【lib】文件夹下就是 MyBatis 项目需要依赖的第三方包，pdf 文件是它英文版的说明，不要英文也可以戳上面的链接。

之后在使用 MyBatis 框架时，需要把它的核心包和依赖包引入到应用程序中。如果是 Web 应用，只需将核心包和依赖包复制到 /WEB-INF/lib 目录中。

如果您使用的 Maven，那么 pom.xml 文件内容如下（根据自己的版本修改相应的内容）。

```xml
<dependencies>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.6</version>
    </dependency>

    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.46</version>
    </dependency>

</dependencies>
```

## 第一个MyBatis程序

在创建 MyBatis 项目之前，首先创建 t_user 数据表，SQL 语句如下。

```sql
DROP TABLE IF EXISTS `t_user`;
CREATE TABLE `t_user`  (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  `password` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 3 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
```

创建 MyBatis 程序的步骤为：

1. 配置 MyBatis 环境
2. 编写MyBatis核心配置文件
3. 创建实体类
4. 创建DAO接口
5. 创建SQL映射文件
6. 编写测试类

下面介绍如何使用 IDEA 创建一个简单的 MyBatis 程序。

1. 创建 Web 应用，部署 jar 包（建议 Maven导入，使用 Maven 创建 Web 项目的方法：[Click Here](https://www.cnblogs.com/RioTian/p/15646257.html)）

   在 IDEA 中创建 Web 项目 MyBatis-quick，并将下载的 MyBatis 的核心 jar 包、依赖 jar 包以及 MySQL 数据库的驱动 jar 包复制到 /WEB-INF/lib 目录中。

2. 创建日志文件

   MyBatis 默认使用 log4j 输出日志信息，如果开发者需要查看控制台输出的 SQL 语句，可以在 classpath 路径下配置其日志文件。在 MyBatis-quick 的 src 目录下创建 log4j.properties 文件，其内容如下：

   ```properties
   ### direct log messages to stdout ###
   log4j.appender.stdout=org.apache.log4j.ConsoleAppender
   log4j.appender.stdout.Target=System.out
   log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
   log4j.appender.stdout.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
   
   ### direct messages to file mylog.log ###
   log4j.appender.file=org.apache.log4j.FileAppender
   log4j.appender.file.File=c:/mylog.log
   log4j.appender.file.layout=org.apache.log4j.PatternLayout
   log4j.appender.file.layout.ConversionPattern=%d{ABSOLUTE} %5p %c{1}:%L - %m%n
   
   ### set log levels - for more verbose logging change 'info' to 'debug' ###
   
   log4j.rootLogger=debug, stdout
   
   ```

   在日志文件中配置了全局的日志配置、MyBatis 的日志配置和控制台输出，其中 MyBatis 的日志配置用于将 com.riotian 包下所有类的日志记录级别设置为 DEBUG。该配置文件内容不需要开发者全部手写，可以从 [MyBatis 使用手册中的 Logging 小节](https://mybatis.org/mybatis-3/zh/logging.html)复制，然后进行简单修改。

3. 创建持久化类

   在 src 目录下创建一个名为 com.riotian.domainn 的包，在该包中创建持久化类 User。注意，在类中声明的属性与数据表 User 的字段一致。

   ```java
   package com.riotian.domain;
   
   public class User {
   
       private int id;
       private String username;
       private String password;
   
       /* get and set */
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", username='" + username + '\'' +
                   ", password='" + password + '\'' +
                   '}';
       }
   }
   
   ```
   
4. 创建映射文件

   在 resources 目录下创建 【com/riotian/mapper】包，在该包下创建映射文件 UserMapper.xml

   UserMapper.xml 文件内容如下:

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="userMapper">
   
       <!--删除操作-->
       <delete id="delete" parameterType="int">
           delete from user where id=#{abc}
       </delete>
   
       <!--修改操作-->
       <update id="update" parameterType="com.riotian.domain.User">
           update user set username=#{username},password=#{password} where id=#{id}
       </update>
   
       <!--插入操作-->
       <insert id="save" parameterType="com.riotian.domain.User">
           insert into user values(#{id},#{username},#{password})
       </insert>
   
       <!--查询操作-->
       <select id="findAll" resultType="user">
           select * from user
       </select>
   
       <!--根据id进行查询-->
       <select id="findById" resultType="user" parameterType="int">
           select * from user where id=#{id}
       </select>
   
   </mapper>
   ```

   上述代码中，\<mapper> 元素是配置文件的根元素，它包含了 namespace 属性，该属性值通常设置为“包名+SQL映射文件名”，用于指定唯一的命名空间。

   子元素 \<select>、\<insert> 中的信息用于执行查询、添加操作。在定义的 SQL 语句中，“#{}”表示一个占位符，相当于“?”，而“#{name}”表示该占位符待接收参数的名称为 name。

5. 创建配置文件

   MyBatis 核心配置文件主要用于配置数据库连接和 MyBatis 运行时所需的各种特性，包含了设置和影响 MyBatis 行为的属性。

   在 src 目录下创建 MyBatis 的核心配置文件 mybatis-config.xml，在该文件中配置了数据库环境和映射文件的位置，具体内容如下：

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
   
       <!--通过properties标签加载外部properties文件-->
       <properties resource="jdbc.properties"></properties>
   
       <!--自定义别名-->
       <typeAliases>
           <typeAlias type="com.riotian.domain.User" alias="user"></typeAlias>
       </typeAliases>
   
       <!--数据源环境-->
       <environments default="developement">
           <environment id="developement">
               <transactionManager type="JDBC"></transactionManager>
               <dataSource type="POOLED">
                   <property name="driver" value="${jdbc.driver}"/>
                   <property name="url" value="${jdbc.url}"/>
                   <property name="username" value="${jdbc.username}"/>
                   <property name="password" value="${jdbc.password}"/>
               </dataSource>
           </environment>
       </environments>
       
   
       <!--加载映射文件-->
       <mappers>
           <mapper resource="com/riotian/mapper/UserMapper.xml"></mapper>
       </mappers>
   
   
   </configuration>
   ```

   上述映射文件和配置文件都不需要读者完全手动编写，都可以从 MyBatis 使用手册中复制，然后做简单修改。

   为了方便管理以后各框架集成所需的配置文件，可以在项目工程下新建 Source Folder 类型的 resources 目录，并在此目录下添加 MyBatis 的核心配置文件，默认文件名为 " configuration.xml"。但需要注意的是，为了方便在框架集成时更好地区分各个配置文件，我们一般将此文件名命名“mybatis-config.xml”，该文件用于配置数据库连接信息和 MyBatis 的参数。

6. 创建测试类

   在 test 目录下创建一个名为 Test 的包，在该包中创建 MyBatisTest 测试类。在测试类中首先使用输入流读取配置文件，然后根据配置信息构建 SqlSessionFactory 对象。

   接下来通过 SqlSessionFactory 对象创建 SqlSession 对象，并使用 SqlSession 对象的方法执行数据库操作。 MyBatisTest 测试类的代码如下：

   ```java
   package Test;
   
   import com.riotian.domain.User;
   import org.apache.ibatis.io.Resources;
   import org.apache.ibatis.session.SqlSession;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.apache.ibatis.session.SqlSessionFactoryBuilder;
   import org.junit.Test;
   
   import java.io.IOException;
   import java.io.InputStream;
   import java.util.List;
   
   public class test {
   
       @Test
       //查询一个对象
       public void test5() throws IOException {
           //获得核心配置文件
           InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
           //获得session工厂对象
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           //获得session回话对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           //执行操作  参数：namespace+id
           User user = sqlSession.selectOne("userMapper.findById", 1);
           //打印数据
           System.out.println(user);
           //释放资源
           sqlSession.close();
       }
   
       @Test
       //删除操作
       public void test4() throws IOException {
   
           //获得核心配置文件
           InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
           //获得session工厂对象
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           //获得session回话对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           //执行操作  参数：namespace+id
           sqlSession.delete("userMapper.delete",2);
   
           //mybatis执行更新操作  提交事务
           sqlSession.commit();
   
           //释放资源
           sqlSession.close();
       }
   
       @Test
       //修改操作
       public void test3() throws IOException {
           //模拟user对象
           User user = new User();
           user.setId(2);
           user.setUsername("lucy");
           user.setPassword("123");
   
           //获得核心配置文件
           InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
           //获得session工厂对象
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           //获得session回话对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           //执行操作  参数：namespace+id
           sqlSession.update("userMapper.update",user);
   
           //mybatis执行更新操作  提交事务
           sqlSession.commit();
   
           //释放资源
           sqlSession.close();
       }
   
       @Test
       //插入操作
       public void test2() throws IOException {
   
           //模拟user对象
           User user = new User();
           user.setUsername("xxx");
           user.setPassword("abc");
   
           //获得核心配置文件
           InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
           //获得session工厂对象
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           //获得session回话对象
           SqlSession sqlSession = sqlSessionFactory.openSession(true);
           //执行操作  参数：namespace+id
           sqlSession.insert("userMapper.save",user);
   
           //mybatis执行更新操作  提交事务
           //sqlSession.commit();
   
           //释放资源
           sqlSession.close();
       }
   
       @Test
       //查询操作
       public void testFindId() throws IOException {
           //获得核心配置文件
           InputStream resourceAsStream = Resources.getResourceAsStream("sqlMapConfig.xml");
           //获得session工厂对象
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           //获得session回话对象
           SqlSession sqlSession = sqlSessionFactory.openSession();
           // 执行操作 参数 namespace + id
           List<User> userList = sqlSession.selectList("userMapper.findAll");
           // 打印数据
           System.out.println(userList);
           // 释放资源
           sqlSession.close();
       }
   
   }
   ```

   

## MyBatis CRUD 操作小结

增删改查映射配置与API

```java
// 查询数据
List<User> userList = sqlSession.selectList("userMapper.findAll");
```

```xml
<select id="findAll" resultType="com.itheima.domain.User">
select * from User
</select>
```

```java
// 添加数据
sqlSession.insert("userMapper.add", user)
```

```xml
<insert id="add" parameterType="com.itheima.domain.User">
insert into user values(#{id},#{username},#{password})
</insert>
```


```java
// 修改数据
sqlSession.update("userMapper.update", user);
```

```xml
<update id="update" parameterType="com.itheima.domain.User">
update user set username=#{username},password=#{password} where id=#{id}
</update>
```

```java
// 删除数据
sqlSession.delete("userMapper.delete",3);
```

```xml
<delete id="delete" parameterType="java.lang.Integer">
delete from user where id=#{id}
</delete>
```

此外还有一些常用的总结：

- **关于 parameterType：** 就是用来在 SQL 映射文件中指定输入参数类型的，可以指定为基本数据类型（如 int、float 等）、包装数据类型（如 String、Interger 等）以及用户自己编写的 JavaBean 封装类
- **关于 resultType：** 在加载 SQL 配置，并绑定指定输入参数和运行 SQL 之后，会得到数据库返回的响应结果，此时使用 resultType 就是用来指定数据库返回的信息对应的 Java 的数据类型。
- **关于 “#{}” ：** 在传统的 JDBC 的编程中，占位符用 “?” 来表示，然后再加载 SQL 之前按照 “?” 的位置设置参数。而 “#{}” 在 MyBatis 中也代表一种占位符，该符号接受输入参数，在大括号中编写参数名称来接受对应参数。当 “#{}” 接受简单类型时可以用 `value` 或者其他任意名称来获取。
- **关于 “${}” ：** 在 SQL 配置中，有时候需要拼接 SQL 语句（例如模糊查询时），用 “#{}” 是无法达到目的的。在 MyBatis 中，“\${}” 代表一个 “拼接符号” ，可以在原有 SQL 语句上拼接新的符合 SQL 语法的语句。使用 “\${}” 拼接符号拼接 SQL ，会引起 SQL 注入，所以一般不建议使用 “${}”。
- **MyBatis 使用场景：** 通过上面的入门程序，不难看出在进行 MyBatis 开发时，我们的大部分精力都放在了 SQL 映射文件上。 **MyBatis 的特点就是以 SQL 语句为核心的不完全的 ORM（关系型映射）框架。**与 Hibernate 相比，Hibernate 的学习成本比较高，而 SQL 语句并不需要开发人员完成，只需要调用相关 API 即可。这对于开发效率是一个优势，但是缺点是没办法对 SQL 语句进行优化和修改。而 MyBatis 虽然需要开发人员自己配置 SQL 语句，MyBatis 来实现映射关系，但是这样的项目可以适应经常变化的项目需求。**所以使用 MyBatis 的场景是：对 SQL 优化要求比较高，或是项目需求或业务经常变动。**

## **MyBatis核心配置文件概述**

MyBatis 的配置文件用于配置数据库相关的设置，比如读取哪个数据库，事物管理方式，映射等。
以下的代码是MaBatis的XML配置文件的层次结构，**需要注意的是这些层次结构不能够出现颠倒，否则在解析时会出现异常。**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>  <!--配置-->
    <porperties/><!--属性-->
    <settings/><!--设置 -->
    <typeAliases/><!--类型命名 -->
    <TypeHandlers/><!-- 类型处理器-->
    <objectFactory/><!--对象工厂-->
    <plugins/><!--插件-->
    <environments><!--配置环境-->
      <environment><!--环境变量-->
        <transactionManager/><!--事物管理器-->
        <datasource><!--数据源-->
      </environment><!--环境变量-->
    </environments><!--配置环境-->
    <databaseIdProcider><!--数据库厂商标识-->
    <mappers/><!--映射器-->
</configuration>  <!--配置-->
```

---

## 参考文章

- 《Java EE 互联网轻量级框架整合开发》
- 《Spring MVC + MyBatis开发从入门到项目实战》
- [How2j-MyBatis 系列教程](http://how2j.cn/k/mybatis/mybatis-tutorial/1087.html)
- 全能的百度和万能的大脑
- [三颗心脏的博客](https://www.wmyskxz.com/2018/04/15/mybatis-1-kuai-su-ru-men/#CRUD-%E6%93%8D%E4%BD%9C)
