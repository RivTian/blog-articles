> **原作者为 [RioTian@cnblogs](https://www.cnblogs.com/RioTian/)， 本作品采用 [CC 4.0 BY](http://creativecommons.org/licenses/by/4.0/) 进行许可，转载请注明出处。**

在学 Java 连接数据库时老师提到过 JSP，刚好这个学期有 JSP 的课程，现做一些基础以及环境配置的介绍。

## 什么是 JSP？

- 动态网页技术标准
- 能把 Java 代码嵌入静态的页面中
- JSP 编译器把 JSP 文件编译成 Java 写的 `Servlet(服务器端程序)`，再交由 Java 编译器

```jsp
<html>
    <title>Hello world!</title>
    <body>
        <%=out.println("Hello world!")%>
    </body>
</html>
```

## 环境配置

1. 语言环境：JDK
2. 网页服务器：Tomcat，[安装教程](https://blog.csdn.net/ThinkWon/article/details/102622905)
3. IDE：IntelliJ IDEA / Eclipse

前两点非常简单不再赘述。

老师竟然说 JSP 没什么好用的编辑器只能在 Eclipse 中写（黑人问号

明明 IDEA 就很好....

**这里记录一下 IDEA 如何配置 JSP 的开发环境**

## IntelliJ IDEA Configuration

- 新建一个 Java Enterprise 项目，下面 Application Server 选择**新建**，

  选择 Tomcat Server，设置 Tomcat Home (安装目录)

> **这里需要注意** IDEA 可能会没有权限访问 Tomcat 的目录，导致无法读取 Tomcat，需要手动访问一次该目录提权：
>
> * Windows：资源管理器直接访问，会提示需要管理员权限，点继续就 OK 了
>
> * Linux：chmod 777

* 下面 Additional Libraries and Frameworks 选择 Web Application，

  点 Next，改名创建，得到如下图所示结构的项目

<img src="https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917213111.png" alt="image-20210917213110401" style="zoom:50%;" />





* 打开 index.jsp

  打开右上角 Edit Configuration

  选择 Application server

  URL 一般是：`http://localhost:8080/demo_war_exploded/`

  其中 demo 为项目名字，_war_exploded 为自动生成的后缀，需要保留

* 打开 Deployment，界面应如下图

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917213209.png)

​		 如果这个 war_exploded 没有出现在里面的话点击右边加号自己加进去

- 简单写一下 index.jsp

  比如下面实现了一个简单的 Hello world 的页面

```jsp
<html>
    <title>Hello world!</title>
    <body>
        <%=out.println("Hello world!")%>
    </body>
</html>
```

- 点击右上角 Run，开始运行

---

> update：21.9.22

切回至“Server”，在“Open Browser”设置里，

建议勾选上“After Launch”，意思为当启动TomCat后，自动打开浏览器并运行项目，此时同样可以在别的浏览器手动输入地址进行访问。

建议将“`On 'Update' action`”和“`On frame deactivation`”设置为“`Update classes and resources`”，这样当修改项目内容时会自动更新字节码文件和资源文件，避免反复重启TomCat服务器：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210922090258.png)

## Question & Solution

运行之后会大概率出现以下问题，不要问我怎么知道的…

---

$$
QAQ
$$

---

* 端口超界，不能为 - 1

原因：Tomcat 安装的默认 Shutdown 端口为 - 1，需要修改一下

解决：打开 Tomcat 安装目录下的 server.xml，类似如下路径

```sh
C:\Program Files\apache-tomcat-8.5.71-windows-x64\apache-tomcat-8.5.71\conf\server.xml
```

修改 `shutdown port` 为任一未被占用的端口 (1024 - 65535)，如图中 `8005` 位置

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917214252.png)

* 端口 8080 已被占用

原因：Tomcat 运行中并占用了 8080 端口

解决：打开刚才的 Edit Configuration，修改 Tomcat Server Settings 中的 HTTP port 为任一未被占用的端口 (1024 - 65535)，如图中”8088”，同时别忘修改上面的 URL 的端口

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917213519.png)

* output 输出乱码

原因：Tomcat 根据你的操作系统默认语言使用了 GBK 编码

解决：打开 Tomcat 安装目录下的 **logging.properties**，类似如下路径

```
C:\Program Files\apache-tomcat-8.5.71-windows-x64\apache-tomcat-8.5.71\conf\logging.properties
```

修改大约第 47 行的

```
java.util.logging.ConsoleHandler.encoding = UTF-8
```

为

```
java.util.logging.ConsoleHandler.encoding = GBK
```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917213816.png)

- 打开网页 404

  原因：URL 写错了，定位不到文件

  解决：Edit Configuration 中检查一下 URL 是否与端口设置一致，是否正确定位了文件路径

- 输出一堆红色字

  正常现象，开发者选色鬼才

- 网页调试的方法

  运行之后修改了代码，怎么才能重新看到修改后的网页呢？

  无需终止 Tomcat 重新运行，只需在左下这个位置点击 Deploy，然后刷新网页就可以啦

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210917213934.png)

如果没有问题此时应该可以正确运行啦，**开始在 IDE 中愉快的写 JSP 吧（雾**

---

$$
QAQ
$$

---

## 基本语法

因为了解不深，详细的就不多说了。

除了特别的这几个，其他的都和 HTML 与 Java 语法规则差不多。

### 脚本

```jsp
<%
	//在这里编写你的Java代码
%>
```

任何文本、HTML 标签、JSP 元素必须写在脚本程序的外面

### 声明

```jsp
<%!
    //Java
%>
```

用于声明变量、方法，要写函数只能在这里面

必须先声明才可以使用

### 表达式

```jsp
<%= //Java %>
```

表达式里面的代码可以不写分号

### 注释

```jsp
<%-- 该部分注释在网页中不会被显示 --%> 

<!-- 该部分注释在网页源代码中会被显示 --> 
```

### 指令

```jsp
<%@ page 页面属性 %>
<%@ include 包含文件 %>
<%@ taglib 标签 %>
```

### 模板

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8" %>
<!--导入的Java包-->
<html>
<head>
    <title>网页标题</title>
</head>
<body>

<%!
    //定义变量
%>

<%!
    //定义函数
%>

<%
    //脚本
%>

<!-- 在这里定义其他网页上展示的组件 -->

</body>
</html>
```

### 示例：计算 N 的阶乘

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Calc</title>
</head>
<body>
<div style="text-align: center;"/>
<h1>
  Calculate N!
</h1>
<!--*.jsp-->
<form action="index.jsp"/>
<input type="number" name="num"/><br/><br/>
<input type="submit"/><br/><br/>
<form/>

<%!
    private int n = -1;
%>
<%!
    public int fac(int n) {
        if (n < 0) {
            return n;
        } else if (n == 0) {
            return 1;
        } else {
            int product = 1;
            for (int i = 1; i <= n; i++) {
                product *= i;
                if (product < 0) {
                    break;
                }
            }
            return product;
        }
    }
%>
<%
    try {
        int n = fac(Integer.parseInt(request.getParameter("num")));
        if (n < 0) {
            out.print("<h2>invalid</h2>");
        } else {
            out.print("<h2>"+n+"</h2>");
        }
    } catch (Exception e) {
        //ignore java.lang.NumberFormatException: null
    }
%>

</body>
</html>
```

### 示例：数据库查询系统

- 依赖包：`mysql-connector-java-5.1.*.jar`
- 可能需要将 jar 包拷贝到项目路径下的 WEB-INF/lib 中（不存在就新建）

```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@ page import="java.sql.*" %>
<%@ page import="java.io.*" %>
<html>
<head>
    <title>Database Query System</title>
</head>
<body>
<div style="text-align: center;">
    <h1>Database Query System</h1>
    <h2>powered by bipy</h2><br/>
    <form method="post">
        <textarea name="SQL_statement" rows=10 cols=100></textarea><br/>
        <input type="submit" name="Submit"/>
        <input type="reset" name="Clear"/>
    </form>
</div>
<br/>
</body>
<%!
    private Statement statement;
    private String sql_input;
    private Connection connection;
    private boolean status = false;
%>
<%!
    //判断SQL语句类型，并返回结果（影响个数/报错/结果集）
    public Object process(String sql) {
        try {
            if (sql.toLowerCase().matches("^(update|create|use|drop|delete|insert).*$")) {
                return statement.executeUpdate(sql);
            } else if (sql.toLowerCase().matches("^(show|select).*$")) {
                return statement.executeQuery(sql);
            } else {
                return "Not Supported";
            }
        } catch (SQLException e) {
            e.printStackTrace();
            return "SQL syntax error";
        }
    }
%>
<%
    try {
        //避免重新建立连接而丢弃前面步骤
        if (!status) {
            Class.forName("com.mysql.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost", "root", "123");
            statement = connection.createStatement();
            status = true;
        }
    } catch (Exception e) {
        e.printStackTrace();
    }
%>
<div style="text-align: center">
    <%
    	//结果输出
        sql_input = request.getParameter("SQL_statement");
        if (sql_input != null) {
            Object rt = process(sql_input);
            if (rt instanceof String) {
                out.print("<h2>" + rt.toString() + "</h2>");
            } else if (rt instanceof Integer) {
                out.print("<h2>" + "Query OK. " + rt + " effected." + "</h2>");
            } else {
                ResultSet rs = (ResultSet) rt;
                ResultSetMetaData rsmd = rs.getMetaData();
                out.println("<table align=\"center\" border=\"1\">");
                out.println("<tr>");
                for (int i = 1; i <= rsmd.getColumnCount(); ++i)
                    out.println("<th>" + rsmd.getColumnName(i) + "</th>");
                out.println("<tr>");
                while (rs.next()) {
                    out.println("<tr>");
                    for (int i = 1; i <= rsmd.getColumnCount(); ++i)
                        out.println("<td>" + rs.getString(i) + "</td>");
                    out.println("</tr>");
                }
                out.println("</table>");
            }
        }
    %>
</div>
</html>
```
