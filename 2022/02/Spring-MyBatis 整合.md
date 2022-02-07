## 整合 MyBatis

### 1.导入依赖

版本根据自己配置选择

```xml
<!-- mybatis -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.6</version>
</dependency>
<!-- mysql -->
<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.47</version>
</dependency>
<!-- spring -->
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-webmvc</artifactId>
    <version>5.3.1</version>
</dependency>
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>5.3.1</version>
</dependency>
<!-- aop -->
<dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.5</version>
</dependency>
<!-- mybatis-spring -->
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>2.0.2</version>
</dependency>
<!-- junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
</dependency>
```

**配置Maven静态资源过滤**

```xml
<build>
   <resources>
       <resource>
           <directory>src/main/resources</directory>
           <includes>
                <include>**/*.properties</include>
                <include>**/*.xml</include>
           </includes>
       </resource>
       <resource>
           <directory>src/main/java</directory>
           <includes>
               <include>**/*.properties</include>
               <include>**/*.xml</include>
                </includes>
           <filtering>true</filtering>
        </resource>
  </resources>
</build>
```

### 2.Mybatis项目搭建

#### 2.1 实体类

```java
public class User {
    private int id; 
    private String name;  
    private String pwd;   
    
    /* get and set */
}
```

#### 2.2 编写接口

```java
public interface UserMapper {
    public List<User> selectUser();
}
```

#### 2.3 Mapper.xml文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="Dao.UserMapper">
    <select id="selectUser" resultType="User">
       select * from mybatistest.user
    </select>
</mapper>
```

#### 2.4 配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>
    <typeAliases>
        <package name="pojo"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url"
                          value="jdbc:mysql://localhost:3306/mybatistest?useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="148729"/>
            </dataSource>
        </environment>
    </environments>

        <mappers>
            <mapper resource="Dao/UserMapper.xml"/>
        </mappers>
</configuration>
```

#### 2.5 测试

```java
@Test
public void selectUser() throws Exception {

    String resource = "mybatis.xml";
    InputStream inputStream = Resources.getResourceAsStream(resource);
    SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
    SqlSession sqlSession = sqlSessionFactory.openSession();

    UserMapper mapper = sqlSession.getMapper(UserMapper.class);

    List<User> userList = mapper.selectUser();
    for (User user: userList){
        System.out.println(user);
    }

    sqlSession.close();
}
```

**测试成功**

### 3.Mybatis-spring

**什么是 MyBatis-Spring？**

MyBatis-Spring 会帮助你将 MyBatis 代码无缝地整合到 Spring 中。

MyBatis-Spring 需要以下版本：

| MyBatis-Spring | MyBatis | Spring 框架 | Spring Batch | Java    |
| -------------- | ------- | ----------- | ------------ | ------- |
| 2.0            | 3.5+    | 5.0+        | 4.0+         | Java 8+ |
| 1.3            | 3.4+    | 3.2.2+      | 2.1+         | Java 6+ |

要和 Spring 一起使用 MyBatis，需要在 Spring 应用上下文中定义至少两样东西：**一个 SqlSessionFactory** 和至少**一个数据映射器类**即**DataSource**。

在 MyBatis-Spring 中，可使用SqlSessionFactoryBean来创建 SqlSessionFactory。要配置这个工厂 bean，只需要把下面代码放在 Spring 的 XML 配置文件中：

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource" />
</bean>
```

注意：SqlSessionFactory需要一个 DataSource（数据源。这可以是任意的 DataSource，只需要和配置其它 Spring 数据库连接一样配置它就可以了。

在基础的 MyBatis 用法中，是通过 SqlSessionFactoryBuilder 来创建 SqlSessionFactory 的。

**而在 MyBatis-Spring 中，则使用 SqlSessionFactoryBean 来创建。**

**SqlSessionTemplate 是 MyBatis-Spring 的核心**。作为 SqlSession 的一个实现，这意味着可以使用它无缝代替你代码中已经在使用的 SqlSession。

模板可以参与到 Spring 的事务管理中，并且由于其是线程安全的，可以供多个映射器类使用，你应该总是用 SqlSessionTemplate 来替换 MyBatis 默认的 DefaultSqlSession 实现。在同一应用程序中的不同类之间混杂使用可能会引起数据一致性的问题。

可以使用 **SqlSessionFactory 作为构造方法的参数**来**创建 SqlSessionTemplate 对象**。

```xml
<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```

现在，这个 bean 就可以直接注入到你的 DAO bean 中了。你需要在你的 bean 中添加一个 SqlSession 属性，就像下面这样：

```java
public class UserDaoImpl implements UserDao {

  private SqlSession sqlSession;
  public void setSqlSession(SqlSession sqlSession) {
    this.sqlSession = sqlSession;
  }
  public User getUser(String userId) {
    return sqlSession.getMapper...;
  }
}
```

整合之后，转换为XML注入：

```xml
<bean id="userDao" class="org.mybatis.spring.sample.dao.UserDaoImpl">
  <property name="sqlSession" ref="sqlSession" />
</bean>
```

### 4.整合方式一

1. **创建beans.xml**

2. **配置数据源替换mybaits的数据源**

   ```xml
    <!--配置数据源：数据源有非常多，可以使用第三方的，也可使使用Spring的-->
   <bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
           <property name="driverClassName" value="com.mysql.jdbc.Driver"/>
           <property name="url" value="jdbc:mysql://localhost:3306/mybatistest?useSSL=true&amp;useUnicode=true&amp;characterEncoding=utf8"/>
           <property name="username" value="root"/>
           <property name="password" value="root"/>
    </bean>
   ```

3. **配置SqlSessionFactory，关联MyBatis**

   ```xml
   <!--配置SqlSessionFactory-->
   <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <property name="dataSource" ref="dataSource"/>
        <!--关联Mybatis-->
        <property name="configLocation" value="classpath:mybatis.xml"/>
        <property name="mapperLocations" value="classpath:Dao/*.xml"/>
   </bean>
   ```

4. **注册sqlSessionTemplate，关联sqlSessionFactory**

   ```xml
   <!--注册sqlSessionTemplate , 关联sqlSessionFactory-->
   <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
       <!--利用构造器注入-->
       <constructor-arg index="0" ref="sqlSessionFactory"/>
   </bean>
   ```

5. **增加Dao接口的实现类；私有化sqlSessionTemplate，用set方法注入sqlSession**

   ```java
   public class UserDaoImpl implements UserMapper{
       private SqlSessionTemplate sqlSession;
       public void setSqlSession(SqlSessionTemplate sqlSession) {
           this.sqlSession = sqlSession;
       }
       public List<User> selectUser() {
           UserMapper mapper = sqlSession.getMapper(UserMapper.class);
           return mapper.selectUser();
       }
   }
   ```

6. **注册bean实现**

   ```xml
   <bean id="userDao" class="Dao.UserDaoImpl">
        <property name="sqlSession" ref="sqlSession"/>
   </bean>
   ```

7. **测试**

   ```java
   @Test
   public void test2(){
       ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
       UserMapper mapper = (UserMapper) context.getBean("userDao");
       List<User> user = mapper.selectUser();
       System.out.println(user);
   }
   ```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220207191526.png)

**注意：**

这时候可能会出错，`Error creating bean with name 'sqlSessionFactory' defined in class path resource [beans.xml]: Invocation of init method failed;`

原因可能为`mybatis.xml`中的`<mappers>映射语句`没删除，删除就行。

### 5.整合方式二

Dao继承Support类 , 直接利用 getSqlSession() 获得 , 然后直接注入SqlSessionFactory。

比起方式1 , 不需要管理SqlSessionTemplate , 而且对事务的支持更加友好

1. **我们上面写的UserDaoImpl修改一下**

   ```java
   public class UserDaoImpl extends SqlSessionDaoSupport implements UserMapper {
       public List<User> selectUser() {
           UserMapper mapper = getSqlSession().getMapper(UserMapper.class);
           return mapper.selectUser();
       }
   }
   ```

2. **修改bean的配置**

   ```xml
   <bean id="userDao" class="Dao.UserDaoImpl">
        <property name="sqlSessionFactory" ref="sqlSessionFactory" />
   </bean>
   ```

3. **测试**

   ```java
   @Test
   public void test2(){
        ApplicationContext context = new ClassPathXmlApplicationContext("beans.xml");
        UserMapper mapper = (UserMapper) context.getBean("userDao");
        List<User> user = mapper.selectUser();
        System.out.println(user);
   }
   ```

   **结果相同**

   **整合到spring以后可以完全不要mybatis的配置文件，除了这些方式可以实现整合之外，我们还可以使用注解来实现**

## 声明式事物

### 1.事务回顾

- 事务在项目开发过程非常重要，涉及到数据的一致性的问题，不容马虎！
- 事务管理是企业级应用程序开发中必备技术，用来确保数据的完整性和一致性。

事务就是把一系列的动作当成一个独立的工作单元，这些动作要么全部完成，要么全部不起作用。

**事务四个属性ACID**

1. 原子性（atomicity）

   事务是原子性操作，由一系列动作组成，事务的原子性确保动作要么全部完成，要么完全不起作用

2. 一致性（consistency）

   一旦所有事务动作完成，事务就要被提交。数据和资源处于一种满足业务规则的一致性状态中

3. 隔离性（isolation）

   可能多个事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏

4. 持久性（durability）

   事务一旦完成，无论系统发生什么错误，结果都不会受到影响。通常情况下，事务的结果被写到持久化存储器中

没有进行事务的管理；我们想让他们都成功才成功，有一个失败，就都失败，我们就应该需要**事务**

以前我们都需要自己手动管理事务，十分麻烦。但是Spring给我们提供了**事务管理**，我们只需要配置即可；

### 2.Spring中的事务管理

Spring在不同的事务管理API之上定义了一个抽象层，使得开发人员不必了解底层的事务管理API就可以使用Spring的事务管理机制。Spring支持编程式事务管理和声明式的事务管理。

**编程式事务管理**

- 将事务管理代码嵌到业务方法中来控制事务的提交和回滚
- 缺点：必须在每个事务操作业务逻辑中包含额外的事务管理代码

**声明式事务管理**

- 一般情况下比编程式事务好用。
- 将事务管理代码从业务方法中分离出来，以声明的方式来实现事务管理。
- 将事务管理作为横切关注点，通过aop方法模块化。Spring中通过Spring AOP框架支持声明式事务管理。

**使用Spring管理事务，注意头文件的约束导入 : tx**

```xml
xmlns:tx="http://www.springframework.org/schema/tx"

http://www.springframework.org/schema/tx
http://www.springframework.org/schema/tx/spring-tx.xsd">
```

**事务管理器**

- 无论使用Spring的哪种事务管理策略（编程式或者声明式）事务管理器都是必须的。
- 就是 Spring的核心事务管理抽象，管理封装了一组独立于技术的方法。

**JDBC事务**

```xml
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource" />
</bean>
```

**配置好事务管理器后我们需要去配置事务的通知，举例：**

```xml
<!--配置事务通知-->
<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <!--配置哪些方法使用什么样的事务,配置事务的传播特性-->
        <tx:method name="add" propagation="REQUIRED"/>
        <tx:method name="get" read-only="true"/>
        <tx:method name="*" propagation="REQUIRED"/>
    </tx:attributes>
</tx:advice>
```

**spring事务传播特性：**

事务传播行为就是多个事务方法相互调用时，事务如何在这些方法间传播。spring支持7种事务传播行为：

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作

Spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。

**配置AOP**

**切记一定：导入aop的头文件！**

```xml
<aop:config>
    <aop:pointcut id="txPointcut" expression="execution(* com.kuang.dao.*.*(..))"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointcut"/>
</aop:config>
```

