## SpringMVC的数据响应方式

- 页面跳转
  - 直接返回字符串
  - 通过ModelAndView对象返回
- 回写数据
  - 直接返回字符串
  - 返回对象或集合

### 页面跳转

方式一、返回带有前缀的字符串:
转发：`forward:/WEB-INF/views/index.jsp`
重定向： `redirect:/index.jsp`

方式二、返回ModelAndView对象

```java
@RequestMapping("/quick2")
public ModelAndView save2(){
    /**
     * Model:模型 用作封装数据
     * View: 视图 用作展示数据
     */
    ModelAndView modelAndView = new ModelAndView();
    //设置模型数据 (会放到request域当中)
    modelAndView.addObject("username","riotian");
    //设置视图名称
    modelAndView.setViewName("success");
    return modelAndView;
}
```

### 回写数据

一、直接回写字符串

```java
@RequestMapping(value = "/quick6")
public void save6(HttpServletResponse response) throws IOException {
    response.getWriter().write("<h1>HelloWorld</h1>");
    //return "success";
}
```

二、回写对象和集合

```java
@RequestMapping(value = "/quick7")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行视图跳转，直接进行数据响应
public String save7() {
    //return "success";
    return "hello RioTian07";
}

@RequestMapping(value = "/quick6")
public void save6(HttpServletResponse response) throws IOException {
    response.getWriter().write("<h1>HelloWorld</h1>");
    //return "success";
}
```

回写 JSON 格式字符串需要添加依赖的坐标

```xml
<!-- 导入 JSON 转换工具 -->
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-core</artifactId>
    <version>2.9.9</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.9.9.3</version>
</dependency>
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-annotations</artifactId>
    <version>2.9.9</version>
</dependency>
```

使用 JSON 包

```java
// 利用转换工具返回 Json 字符串
@RequestMapping(value = "/quick9")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public String save9() throws JsonProcessingException {
    User user = new User();
    user.setUsername("Lisi");
    user.setAge(30);
    // 使用 Json 转换工具将对象转化为 JSON 格式字符串返回, 需要导包
    ObjectMapper objectMapper = new ObjectMapper();
    String json = objectMapper.writeValueAsString(user);

    //return "{\"username\":\"zhangsan\", \"age\":\"18\"}";
    return json;
}

// 返回 Json 字符串
@RequestMapping(value = "/quick8")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public String save8() {
    //return "success";
    return "{\"username\":\"zhangsan\", \"age\":\"18\"}";
}
```

更简单的方式
需要先在`spring-mvc.xml`中配置

```xml
<!--配置处理器映射器-->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <bean class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter"></bean>
        </list>
    </property>
</bean>
```

然后，进行使用

```java
// 利用转换工具返回 Json 字符串
@RequestMapping(value = "/quick10")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public User save10() throws JsonProcessingException {
    User user = new User();
    user.setUsername("Lisi");
    user.setAge(30);
    return user;
}
```

### VC的注解驱动

在SpringMVC的各个组件中，处理器映射器、处理器适配器、视图解析器成为SpringMVC的三大组件。
使用`<mvc:annitation-driven>`自动加载RequestMappingHandlerMapping (处理映射器)和RequestMappingHandlerAdapter (处理适配器)，可以在`Spring-xml.xml`配置文件中使用`mvc:annotation-driven`替代注解处理器和适配器的配置。
同时`<mvc:annotation-driven/>`默认底层就会集成jackson进行对象或集合的json格式字符串的转换。

```xml
<!--MVC 注册驱动, 有了 MVC 驱动可以不用处理器映射器-->
<mvc:annotation-driven/>
<!-- 如果找不到就交给原始容器找 -->
<mvc:default-servlet-handler/>
```

## 获得请求参数

### 获得基本类型参数

```java
@RequestMapping(value = "/quick11")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public void save11(String username, int age) {
    // http://localhost:8080/Spring03/user/quick11?username=Lisi&age=30
    System.out.println(username);
    System.out.println(age);
}
// 控制台输出
// Lisi
// 30
```

### 获得POJO类型参数

```java
@RequestMapping(value = "/quick12")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public void save12(User user) {
    // http://localhost:8080/Spring03/user/quick12?username=Lisi&age=30
    System.out.println(user);
}
// 控制台输出
// User{username='Lisi', age=30}
```

### 获得数组类型参数

```java
@RequestMapping(value = "/quick13")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public void save13(String[] strs) {
    // http://localhost:8080/Spring03/user/quick13?strs=bb&strs=aa
    System.out.println(Arrays.asList(strs));
}
// 控制台输出
// [bb, aa]
```

### 获得集合类型参数

```java
// 创建一个 VO 对象
public class VO {
    private List<User> userList;

    public List<User> getUserList() {
        return userList;
    }

    public void setUserList(List<User> userList) {
        this.userList = userList;
    }

    @Override
    public String toString() {
        return "VO{" +
                "userList=" + userList +
                '}';
    }
}
// 编写方法
@RequestMapping(value = "/quick14")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public void save14(VO vo) {
    // http://localhost:8080/Spring03/user/quick14
    System.out.println(vo);
}
// 控制台日志
// VO{userList=null}
```



```java
// 方式二
// 使用@RequestBody
// ajax.jsp
<head>
    <title>ajax.jsp</title>
    <script src="${pageContext.request.contextPath}/js/jquery-3.3.1.js"></script>
    <script>
      // 1. 准备数据
      var userList = new Array();
      userList.push({username:"zhangsan", age:"18"});
      userList.push({username:"lisi", age:"28"});

      // 2. 发送 Ajax 请求
      $.ajax({
            type:"POST",
            url:"${pageContext.request.contextPath}/user/quick15",
            data:JSON.stringify(userList),
            contentType:"application/json;charset=utf-8",
      })
    </script>
</head>
<body></body>

// 编写方法接口
@RequestMapping(value = "/quick15")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
public void save15(@RequestBody List<User> userList) {
    // http://localhost:8080/Spring03/ajax.jsp
    System.out.println(userList);
}
```

### 开启静态资源访问

```xml
<mvc:resources mapping="/js/**" location="/js/"/>
<mvc:resources mapping="/img/**" location="/img/"/>

<!-- 或者直接使用 -->
<mvc:default-servlet-handler/>
```

### 请求数据乱码问题

post 可能会出现乱码问题，在`web.xml`中配置过滤器

```xml
<!-- 配置全局过滤的filter-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### 参数绑定注解@RequestParam

当请求的参数名称与Controller的业务方法参数名称不一致时，就可以通过@RequestParam注解进行绑定。

- value:指定请求参数名称
- required: 再此指定的请求参数是否必须包含，默认是true，提交时如果没有此参数则报错。
- defaultValue:当没有指定请求参数时，则使用指定的默认值赋值

```java
@RequestMapping(value = "/quick16")
@ResponseBody // 告知SpringMVC 在访问这个方法时不进行页面跳转
// RequestParam("name") 显示绑定 name 和 username.
// 并且默认情况下要带（name or username），不然报错. 改 required = false 即可
// defaultValue 表示不带参数时 username = defaultValue
public void save16(@RequestParam(value = "name",required = false, defaultValue = "RioTian") String username) {
    // http://localhost:8080/Spring03/user/quick16?name=zhangsan
    System.out.println(username);
}
```



### **SpringMVC中获取Restful风格的参数(从请求路径中获取参数 )**

**Restful** 是一种软件**架构风格、设计风格**，而**不是标准**，只是提供了一组设计原则和约束条件。主要用于客户端和服务器交互类的软件，基于这个风格设计的软件可以更简洁，更有层次，更易于实现缓存机制等。

**Restful风格**的请求是使用 **“url+请求方式”** 表示一次请求目的的，HTTP 协议里面四个表示操作方式的动词如下：

* GET：用于获取资源
* POST：用于新建资源
* PUT：用于更新资源
* DELETE：用于删除资源

例如：

- /user/1 `GET`: 得到id=1的user
- /user/1 `DELETE`: 删除id=1的user
- /user/1 `PUT`: 更新id=1的user
- /user `POST`:新增user

上述 url 地址/user/1 中的 $1$ 就是要获得的请求参数，在SpringMVC中可以使用占位符进行参数绑定。地址 /user/1 可以写成 /user/{id}，占位符 {id} 对应的就是 $1$ 的值。在业务方法中我们可以使用`@PathVariable` 注解进行占位符的匹配获取工作。

```java
// Restful 风格的参数获取
@RequestMapping(value = "/quick17/{username}") // {username} 相当于占位符， 搭配 PathVariable 获取
@ResponseBody
public void save17(@PathVariable(value = "username") String username) {
    // http://localhost:8080/Spring03/user/quick17/RioTian
    System.out.println(username);
}
```

## 自定义类型转换器

有些类型需要需要自定义转换器，比如日期类型自定义格式的数据，就需要自定义转换器。

```java
// 转换器相关
@RequestMapping(value = "/quick18")
@ResponseBody
public void save17(Date date) {
    // http://localhost:8080/Spring03/user/quick18?date=2022-01-24
    System.out.println(date);
}
```

步骤:

1. 定义转换器类实现Converter接口

   ```java
   public class DateConverter implements Converter<String/*转入前*/, Date/*转入后*/> {
   	public Date convert(String dataStr) {
   	    //将日期字符串转换成日期对象，返回
   	    SimpleDateFormat format = new SimpleDateFormat("yyyyMMdd");
   	    try {
   	        return format.parse(dataStr);
   	    } catch (ParseException e) {
   	        e.printStackTrace();
   	    }
   	    return null;
   	}
   }
   ```

2. 在`spring-mvc.xml`配置文件中声明转换器

   ```xml
       声明转换器
       <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
           <property name="converters">
               <list>
                   <bean class="com.riotian.converter.DateConverter"></bean>
               </list>
           </property>
       </bean>
   ```

3. 在`spring-mvc.xml`配置文件中的`<annotation-driven>`中引用转换器

   ```xml
   <mvc:annotation-driven conversion-service="conversionService"/>
   ```

   

## 获取请求头

### @RequestHeader

- value:名称
- required:是否必须携带此请求头

```java
@RequestMapping("/quick20")
@ResponseBody
public void save20(@RequestHeader(value = "user-agent") String userAgent) throws IOException {
    System.out.println("userAgent:" + userAgent);
}
```

### @CookieValue

可以获得指定Cookie的值

- value:cookie的名称
- required:是否必须携带此cookie

此时可以去 F12 中查看 Cookie 中是否带有个JSESSIONID的Cookie

```java
@RequestMapping(value="/quick21")
@ResponseBody
public void save21(@CookieValue(value = "JSESSIONID") String jsessionId) throws IOException {
    System.out.println(jsessionId);
}
```

