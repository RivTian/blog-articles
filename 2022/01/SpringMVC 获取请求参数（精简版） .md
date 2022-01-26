## SpringMVC获取请求数据：

DispatcherServlet底层中获取请求数据并传递给单元方法使用。

DispatcherServlet会根据请求动态调用对应的单元方法处理，而请求先被DispatcherServlet接收，意味着DispatcherServlet中拥有此次请求相关的请求数据，那么就需要DispatcherServlet将请求数据传递给处理调用的单元方法，需要在单元方法中声明对应的形参接收请求数据。

## 紧耦方式获取请求数据：

使用：在单元方法中声明request形参，直接接收此次请求的request对象来获取请求数据；

缺点：需要使用request对象获取请求数据，需要类型转换或者对象的封装。

```java
@RequestMapping("demoRequest")
public String demoRequest(HttpServletRequest request){
    //获取请求数据
        String uname=req.getParameter("uname");
        int age= Integer.parseInt(req.getParameter("age"));
    //处理请求数据
        System.out.println("紧耦方式是，使用request对象获取:"+uname+age);
    //响应处理结果
    return "requestc.jsp";
}
```

## 解耦方式获取请求数据：

**方式一**：使用单元方法形参直接获取对应的请求数据；形参名和请求数据的键名一致；

好处是不用自己获取请求数据，会自动进行类型转换；

本质：DispatcherServlet底层会使用单元方法的形参名为键名获取此次的请求数据并赋值给单元方法使用；

**注意**：如果请求中不存在对应的请求数据，会将null值传递给单元方法，所以如果涉及到类型转换会出现转换异常；

```java
@RequestMapping("demoParam")
public String demoParam(String uname,int age){
    //处理请求
    System.out.println("解耦方式一使用形参获取:"+uname+age);
    //响应处理结果
    return "param.jsp";
}
```

## 解耦方式二：使用形参别名获取请求数据

方式一存在的**问题**：使用单元方法的形参名和请求数据的键名一致可以获取请求数据，那么如果形参名和请求数据的键名不一致，会造成请求数据无法获取，就需要我们修改形参名让它与键名一致，但是不管怎么修改都会涉及到调用代码的修改，此时使用形参别名比较方便，别名的值和请求数据的键名一致；

使用：在单元方法的形参声明前使用@RequestParam（配置信息），声明别名；

本质：如果我们使用单元方法的别名，则DispatcherServlet底层会使用别名作为键名获取请求数据

@RequestParam注解的使用：

- value属性：值为形参的别名；
- defaultValue属性：设置形参的默认值，当请求中没有对应的请求数据时，则将默认值传递给单元方法的形参；
- required属性：设置根据别名必须能够获取到对应的请求数据，如果请求中没有则报400异常，不能和defaultValue属性同时使用；

```java
@RequestMapping("demoParam2")
public String demoParam2(@RequestParam(value="uname",required = true) String uname2,@RequestParam(value="age") int age2){
    //处理请求
    System.out.println("解耦方式二使用形参别名获取:"+uname2+age2);
    //响应处理结果
    return "param2.jsp";
}
```

## 解耦方式三：使用对象获取请求数据

使用：在单元方法上声明实体类类型的形参，来获取请求数据，DispatcherServlet会将请求数据封装到形参类型的对象中，然后将对象传递给单元方法；实体类的属性名必须和请求数据的键名一致；

```java
@RequestMapping("demoParam4")
public String demoParam4(String uname,User user){
    //处理请求数据
    System.out.println(user+":"+uname);
    return "param4.jsp";
}
```

## restful风格的请求数据获取：

传统的请求数据风格：localhost:8080/Sprint03/as?username=zhangsan&pwd=123

问题：传统方式会将请求数据以键值对的形式发送给后台使用，后台根据键名获取即可，但是键名是声明在前台代码的，造成后台代码和前台之间有了之间的联系，如果需要修改键名会造成后台的代码也需要修改；

解决：在请求中不声明键名，直接发送数据给后台，但是请求数据必须按照指定的规则来发送。

实现：使用restful规则

要求请求数据要直接在请求路径中附带，不使用问号拼接，但是数据在请求路径中的顺序必须按照指定的顺序声明，不能随便声明，因为后台会按照顺序获取路径中的请求数据；

**eg：** localhost:8080/project/as/zhangsan/123

SpringMVC获取restful路径中的数据：

- 在单元方法的别名中使用{键名}的方式进行参数占位；
- 在单元方法的形参声明前使用注解@PathVariable("占位的键名")将数据赋值给形参；

```java
@RequestMapping("demoRest/{un}/{pw}")
public String demoRest(@PathVariable("un") String username, @PathVariable("pw") String pwd){
    //处理请求数据
    System.out.println(uname+":"+pwd);
    return "rest.jsp";
}
```

