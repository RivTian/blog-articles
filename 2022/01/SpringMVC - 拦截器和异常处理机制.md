**SpringMVC 拦截器**类似于 **Servlet** 开发中的过滤器 Filter，用于对处理器进行预处理和后处理。（责任链模式，AOP思想）

如何使用拦截器

1. 创建 拦截器实现 HandlerInterceptor 接口

   ```java
   public class MyInterceptor1 implements HandlerInterceptor {
       //在目标方法执行之前 执行
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           System.out.println("preHandle...");
           return true; // 返回值 false:后面方法都不执行 true:后面方法都执行
       }
   
       //在目标方法执行之后 视图对象返回之前执行
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
           System.out.println("postHandle...");
       }
   
       //整个流程都执行完毕后 执行
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
           System.out.println("afterCompletion...");
       }
   }
   ```

2. 在`spring-mvc.xml`中配置拦截器

   ```xml
   <mvc:interceptors>
       <mvc:interceptor>
           <!-- 对哪些资源执行拦截操作-->
           <mvc:mapping path="/**/"/> <!-- 对所有方法都执行 -->
           <bean class="com.riotian.interceptor.MyInterceptor2"/>
       </mvc:interceptor>
       <mvc:interceptor>
           <!-- 对哪些资源执行拦截操作-->
           <mvc:mapping path="/**/"/> <!-- 对所有方法都执行 -->
           <bean class="com.riotian.interceptor.MyInterceptor1"/>
       </mvc:interceptor>
   </mvc:interceptors>
   ```

3. 测试拦截器的拦截效果
   执行`http://localhost:8080/Spring05/target`

   ```java
   @Controller
   public class TargetController {
   
       @RequestMapping("/target")
       public ModelAndView show() {
           System.out.println("目标资源执行......");
           ModelAndView modelAndView = new ModelAndView();
           modelAndView.addObject("name","RioTian");
           modelAndView.setViewName("index");
           return modelAndView;
       }
   }
   ```

4. 可以看到打印了日志

   ```java
   preHandle...
   目标资源执行....
   postHandle...
   afterCompletion...
   ```

有了拦截器，我们就可以在preHandler中进行拦截跳转操作

```java
// 目标方法执行之前执行
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
    System.out.println("preHandle...");
    String param = request.getParameter("param");
    if("yes".equals(param)){
        return true;
    } else {
        request.getRequestDispatcher("/error.jsp").forward(request,response);
        return false; // fasle: 后面都不执行 true: 代表放行
    }
}
```

在postHandle中，可以进行参数的增加修改

```java
public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    System.out.println("postHandle...");
    modelAndView.addObject("name", "riotian");
}
```

#### 多个拦截器

多个拦截器，会先顺序执行 `preHandle` ，再反顺序调用 `postHandle` 和 `afterCompletion`。

```
preHandle1...
preHandle2...
目标资源执行...
postHandle2...
postHandle1...
afterCompletion2...
afterCompletion1...
```

### SpringMVC异常处理机制

可以用来进行统一的异常处理。
新建处理类继承自 `HandlerExceptionResolver`

```java
public class MyExceptionResolver implements HandlerExceptionResolver {
    /*
    参数Exception：异常对象
    返回值ModelAndView：跳转到错误视图信息
 */
    public ModelAndView resolveException(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Object o, Exception e) {
        ModelAndView modelAndView = new ModelAndView();

        if(e instanceof MyException){
            modelAndView.addObject("info","自定义异常");
        }else if(e instanceof ClassCastException){
            modelAndView.addObject("info","类转换异常");
        }

        modelAndView.setViewName("error");

        return modelAndView;
    }
}
```

在`spring-mvc.xml`中配置自定义异常处理器

```xml
<bean class="com.riotian.resolver.MyExceptionResolver"/>
```

如果出现Crash，就可以进行统一处理了。
