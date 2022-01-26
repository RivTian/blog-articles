本文介绍 Spring 中 数据库使用以及 JdbcTemplate 的使用方式。

## 传统 JDBC 回顾

JDBC 我们一定不陌生，刚开始学习的时候，我们写过很多很多重复的模板代码：

```java
public Student getOne(int id) {

    String sql = "select id,name from student where id = ?";
    Student student = null;
    // 声明 JDBC 变量
    Connection con = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        // 注册驱动程序
        Class.forName("com.myql.jdbc.Driver");
        // 获取连接
        con = DriverManager.getConnection("jdbc://mysql://localhost:" +
                "3306/student", "root", "root");
        // 预编译SQL
        ps = con.prepareStatement(sql);
        // 设置参数
        ps.setInt(1, id);
        // 执行SQL
        rs = ps.executeQuery();
        // 组装结果集返回 POJO
        if (rs.next()) {
            student = new Student();
            student.setId(rs.getInt(1));
            student.setName(rs.getString(1));
        }
    } catch (ClassNotFoundException | SQLException e) {
        e.printStackTrace();
    } finally {
        // 关闭数据库连接资源
        try {
            if (rs != null && !rs.isClosed()) {
                rs.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (ps != null && !ps.isClosed()) {
                ps.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
        try {
            if (con != null && con.isClosed()) {
                con.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    return student;
}
```

现在光是看着就头大，并且我还把它完整的写了出来..

同时这还仅仅是一个 JDBC 的方法，并且最主要的代码只有`ps = con.prepareStatement(sql);`这么一句，而且有很多模板化的代码，包括建立连接以及关闭连接..我们必须想办法解决一下！

### 优化传统的 JDBC

#### 第一步：创建 DBUtil 类

我想第一步我们可以把重复的模板代码提出来创建一个 `DBUtil` 数据库工具类：

```java
package util;

import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class DBUtil {
    static String ip = "127.0.0.1";
    static int port = 3306;
    static String database = "student";
    static String encoding = "UTF-8";
    static String loginName = "root";
    static String password = "root";

    static {
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    public static Connection getConnection() throws SQLException {
        String url = String.format("jdbc:mysql://%s:%d/%s?characterEncoding=%s&useSSL=false", ip, port, database, encoding);
        return DriverManager.getConnection(url, loginName, password);
    }
}
```

这样我们就可以把上面的代码变成这样：

```java
public Student getOne(int id) {

    String sql = "select id,name from student where id = ?";
    Student student = null;
    // 声明 JDBC 变量
    Connection con = null;
    PreparedStatement ps = null;
    ResultSet rs = null;
    try {
        // 获取连接
        con = DBUtil.getConnection();
        // 预编译SQL
        ps = con.prepareStatement(sql);
        // 设置参数
        ps.setInt(1, id);
        // 执行SQL
        rs = ps.executeQuery();
        // 组装结果集返回 POJO
        ....
    } catch (SQLException e) {
        e.printStackTrace();
    } finally {
        // 关闭数据库连接资源
        ....
    }
    return student;
}
```

也只是少写了一句注册驱动程序少处理了一个异常而已，并没有什么大的变化，必须再优化一下

#### 第二步：使用 try-catch 语句自动关闭资源

自动资源关闭是 JDK 7 中新引入的特性，不了解的同学可以去看一下三颗心大佬写的[Java 7新特性详解](https://www.jianshu.com/p/6bc2e4c82f6b) 

于是代码可以进一步优化成这样：

```java
public Student getOne(int id) {

    String sql = "select id,name from student where id = ?";
    Student student = null;
    // 将 JDBC 声明变量包含在 try(..) 里将自动关闭资源
    try (Connection con = DBUtil.getConnection(); PreparedStatement ps = con.prepareStatement(sql)) {

        // 设置参数
        ps.setInt(1, id);
        // 执行SQL
        ResultSet rs = ps.executeQuery();
        // 组装结果集返回 POJO
        if (rs.next()) {
            student = new Student();
            student.setId(rs.getInt(1));
            student.setName(rs.getString(1));
        }
    } catch (SQLException e) {
        e.printStackTrace();
    }
    return student;
}
```

这样看着好太多了，但仍然不太满意，因为我们最核心的代码也就只是执行 SQL 语句并拿到返回集，再来再来

#### 再进一步改进 DBUtil 类：

在 `DBUtil` 类中新增一个方法，用来直接返回结果集：

```java
public static ResultSet getResultSet(String sql, Object[] objects) throws SQLException {

    ResultSet rs = null;
    try (Connection con = getConnection(); PreparedStatement ps = con.prepareStatement(sql)) {

        // 根据传递进来的参数，设置 SQL 占位符的值
        for (int i = 0; i < objects.length; i++) {
            ps.setObject(i + 1, objects[i]);
        }
        // 执行 SQL 语句并接受结果集
        rs = ps.executeQuery();
    }
    // 返回结果集
    return rs;
}
```

这样我们就可以把我们最开始的代码优化成这样了：

```java
public Student getOne(int id) {

    String sql = "select id,name from student where id = ?";
    Object[] objects = {id};
    Student student = null;
    try (ResultSet rs = DBUtil.getResultSet(sql, objects);) {
        student.setId(rs.getInt(1));
        student.setName(rs.getString(1));
    } catch (SQLException e) {
        // 处理异常
        e.printStackTrace();
    }
    return student;
}
```

wooh!看着爽多了，但美中不足的就是没有把 try-catch 语句去掉，我们也可以不进行异常处理直接把 SQLException 抛出去：

```java
public Student getOne(int id) throws SQLException {

    String sql = "select id,name from student where id = ?";
    Object[] objects = {id};
    Student student = null;
    try (ResultSet rs = DBUtil.getResultSet(sql, objects);) {
        student.setId(rs.getInt(1));
        student.setName(rs.getString(1));
    }
    return student;
}
```

其实上面的版本已经够好了，这样做只是有些强迫症。

> 我们自己定义的 `DBUtil` 工具已经很实用了，因为是从模板化的代码中抽离出来的，所以我们可以一直使用

## Spring 中的 JDBC

要想使用 Spring 中的 JDBC 模块，就必须引入相应的 jar 文件：

- **需要引入的 jar 包：**

- spring-jdbc-4.3.16.RELEASE.jar

- spring-tx-4.3.16.RELEASE.jar

- Maven 坐标引用

  ```xml
  <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.32</version>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.0.5.RELEASE</version>
  </dependency>
  <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>5.0.5.RELEASE</version>
  </dependency>
  ```

好在 IDEA 在创建 Spring 项目的时候已经为我们自动部署好了，接下来我们来实际在 Spring 中使用一下 JDBC：

#### 配置数据库资源

就像我们创建 `DBUtil` 类，将其中连接的信息封装在里面一样，我们需要将这些数据库资源配置起来

- **配置方式：**
- 使用简单数据库配置
- 使用第三方数据库连接池

我们可以使用 Spring 内置的类来配置，但大部分时候我们都会使用第三方数据库连接池来进行配置，由于使用第三方的类，一般采用 XML 文件配置的方式，我们这里也使用 XML 文件配置的形式：

##### 使用简单数据库配置

首先我们来试试 Spring 的内置类 `org.springframework.jdbc.datasource.SimpleDriverDataSource`：

```xml
<bean id="dateSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
    <property name="username" value="root"/>
    <property name="password" value="root"/>
    <property name="driverClass" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc://mysql://locolhost:3306/student"/>
</bean>
```

我们来测试一下，先把我们的 JDBC 操作类写成这个样子：

```java
@Component("jdbc")
public class JDBCtest {

    @Autowired
    private DataSource dataSource;

    public User getOne(int stuID) throws SQLException {
        // id username password email
        String sql = "select id, username from user where id = " + stuID;
        User user = new User();
        Connection conn = dataSource.getConnection();
        Statement st = conn.createStatement();
        ResultSet rs = st.executeQuery(sql);
        if (rs.next()) {
            user.setId(rs.getLong("id"));
            user.setUsername(rs.getString("username"));
        }
        return user;
    }
}
```

然后编写测试类：

```java
@Test
public void test() throws SQLException {
    ApplicationContext context =
            new ClassPathXmlApplicationContext("applicationContext.xml");
    JDBCtest jdbc = (JDBCtest) context.getBean("jdbc");
    User user = jdbc.getOne(1);
    System.out.println(user.getId());
    System.out.println(user.getUsername());
}
```

成功取出数据库中的数据：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220126112817.png)

##### 使用第三方数据库连接池

上面配置的这个简单的数据源一般用于测试，因为它不是一个数据库连接池，知识一个很简单的数据库连接的应用。在更多的时候，我们需要使用第三方的数据库连接，比如使用 C3P0数据库连接池：

```xml
<!--1、加载jdbc.properties-->
<context:property-placeholder location="classpath:jdbc.properties"/>

<!--2、配置数据源对象-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"/>
    <property name="jdbcUrl" value="${jdbc.url}"/>
    <property name="user" value="${jdbc.username}"/>
    <property name="password" value="${jdbc.password}"/>
    <property name="initialPoolSize" value="3"></property>
    <property name="maxPoolSize" value="10"></property>
    <property name="maxStatements" value="100"></property>
    <property name="acquireIncrement" value="2"></property>
</bean>

<!--3、配置JdbcTemplate对象-->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

跟上面的测试差不多，不同的是需要引入相关支持 C3P0 数据库连接池的 jar 包而已。

#### Jdbc Template

Spring 中提供了一个 Jdbc Template 类，它自己已经封装了一个 DataSource 类型的变量，我们可以直接使用：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <bean id="dataSrouce" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"/>
        <property name="url" value="jdbc:mysql://localhost:3306/javat?useSSL=false"/>
        <property name="username" value="root"/>
        <property name="password" value="root"/>
    </bean>

    <context:component-scan base-package="jdbc" />

    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <property name="dataSource" ref="dataSrouce"/>
    </bean>

</beans>
```

我们来改写一下 JDBC 操作的类：

```java
@Component("jdbc")
public class JDBCtest {

    @Autowired
    private JdbcTemplate jdbcTemplate;

    public User getOne(int stuID) throws SQLException {
        // id username password email
        String sql = "SELECT id, username FROM user WHERE id = ?";
        User user = jdbcTemplate.queryForObject(sql, new RowMapper<User>() {
            @Override
            public User mapRow(ResultSet resultSet, int i) throws SQLException {
                User stu = new User();
                stu.setId(resultSet.getLong("id"));
                stu.setUsername(resultSet.getString("username"));
                return stu;
            }
        }, stuID);
        return user;
    }
}
```

测试类不变，运行可以获得正确的结果：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20220126113733.png)

但是好像并没有简单多少的样子，那我们来看看其他 CRUD 的例子：

```java
    /**
     * 增加一条数据
     *
     * @param user
     */
    public void add(User user) {
        this.jdbcTemplate.update("INSERT INTO student(id,username,password,email) VALUES(?,?,?,?)",user.getId(),user.getUsername(),user.getPassword(),user.getEmail());
    }

    /**
     * 更新 username
     *
     * @param user
     */
    public void update(User user) {
        this.jdbcTemplate.update("update user set name = ? where id = ?", user.getUsername(),user.getId());
    }

    /**
     * 删除一条数据
     *
     * @param id
     */
    public void delete(int id) {
        this.jdbcTemplate.update("delete from user from id = ?", id);
    }
```

现在应该简单多了吧，返回集合的话只需要稍微改写一下上面的 getOne() 方法就可以了

---

#### 参考资料：

- 《Java EE 互联网轻量级框架整合开发》
- 《Spring 实战》
- 全能的百度和万能的大脑
- [JDBC的封装](https://tobing.top/2019/11/04/JDBC%E7%9A%84%E5%B0%81%E8%A3%85/)
- [Spring之JdbcTemplate使用](https://www.cnblogs.com/antLaddie/p/12859647.html)

> 扩展阅读：[官方文档](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/JdbcTemplate.html) 、 [Spring 中 JdbcTemplate 实现增删改查](https://liuyanzhao.com/5689.html)、[① 彻底理解数据库事务](http://www.hollischuang.com/archives/898)、[② Spring事务管理详解](https://blog.csdn.net/donggua3694857/article/details/69858827)、[③ Spring 事务管理（详解+实例）](http://www.mamicode.com/info-detail-1248286.html)、[④ 全面分析 Spring 的编程式事务管理及声明式事务管理](https://www.ibm.com/developerworks/cn/education/opensource/os-cn-spring-trans/)

