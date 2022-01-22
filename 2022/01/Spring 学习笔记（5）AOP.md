本文介绍 Spring 中 AOP 的原理及使用方式。

## 引入

在介绍 AOP 之前，假设我们有一个需求：在实现加减乘除功能的同时，满足：

- 在程序执行期间追踪正在发生的活动；
- 希望计算器只能处理正数运算。

有如下代码：

```java
public interface ArithmeticCalculator {
    int add(int i, int j);
    int sub(int i, int j);
    int mul(int i, int j);
    int div(int i, int j);
}
```

```java
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
    public static final Logger LOGGER = LoggerFactory.getLogger(ArithmeticCalculator.class);

    @Override
    public int add(int i, int j) {
        LOGGER.info("The method add begins with [{}, {}]", i, j);
        int result = i + j;
        LOGGER.info("The method add ends with {}", result);
        return result;
    }

    @Override
    public int sub(int i, int j) {
        LOGGER.info("The method sub begins with [{}, {}]", i, j);
        int result = i - j;
        LOGGER.info("The method sub ends with {}", result);
        return result;
    }

    @Override
    public int mul(int i, int j) {
        LOGGER.info("The method mul begins with [{}, {}]", i, j);
        int result = i * j;
        LOGGER.info("The method mul ends with {}", result);
        return result;
    }

    @Override
    public int div(int i, int j) {
        LOGGER.info("The method div begins with [{}, {}]", i, j);
        int result = i / j;
        LOGGER.info("The method div ends with {}", result);
        return result;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        ArithmeticCalculator arithmeticCalculator = new ArithmeticCalculatorImpl();
        int result = arithmeticCalculator.add(1, 2);
        System.out.println(result);

        result = arithmeticCalculator.sub(4, 2);
        System.out.println(result);
    }
}
```

输出结果如下：

```
23:01:49.551 [main] INFO com.example.springdemo.aop.helloworld.ArithmeticCalculator - The method add begins with [1, 2]
23:01:49.553 [main] INFO com.example.springdemo.aop.helloworld.ArithmeticCalculator - The method add ends with 3
3
23:01:49.553 [main] INFO com.example.springdemo.aop.helloworld.ArithmeticCalculator - The method sub begins with [4, 2]
23:01:49.553 [main] INFO com.example.springdemo.aop.helloworld.ArithmeticCalculator - The method sub ends with 2
2
```

可以看到，在上面的实现类中，「打印日志」的代码是十分相似的，并且当「打印日志」的代码与具体的「业务代码」放到一起时，「业务代码」的功能就显得不那么清晰了。不仅冗余的日志代码很多，而且上面的方法不是一种很好的实现方式。即：

- 代码混乱：越来越多的非业务需求（日志和验证等）加入后，原有的业务方法急剧膨胀。每个方法在处理核心逻辑的同时还必须兼顾其他多个关注点。
- 代码分散：以日志需求为例，只是为了满足这个单一的需求，就不得不在多个模块（方法）中多次重复写相同的日志代码。如果日志需求发生变化，必须修改所有模块。

针对以上问题，可以使用基于动态代理的方式，完成 AOP 的功能。代理设计模式的原理是：**使用一个代理将对象包装起来**，然后用该代理对象取代原始对象。任何对原始对象的调用都需要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。

### 使用动态代理实现

```java
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {
    public static final Logger LOGGER = LoggerFactory.getLogger(ArithmeticCalculator.class);

    @Override
    public int add(int i, int j) {
        int result = i + j;
        return result;
    }

    @Override
    public int sub(int i, int j) {
        int result = i - j;
        return result;
    }

    @Override
    public int mul(int i, int j) {
        int result = i * j;
        return result;
    }

    @Override
    public int div(int i, int j) {
        int result = i / j;
        return result;
    }
}
```

```java
public class ArithmeticCalculatorLoggingProxy {

    // 要代理的对象
    private ArithmeticCalculator target;

    public ArithmeticCalculatorLoggingProxy(ArithmeticCalculator target) {
        this.target = target;
    }

    public ArithmeticCalculator getLoggingProxy() {
        ArithmeticCalculator proxy = null;

        // 代理对象与普通对象不同，普通对象直接 new 出来即可，此时 JVM 有默认的类加载器，
        // 而现在的对象是自己代理出来的，需要确定代理对象由哪个类加载器进行加载
        ClassLoader loader = target.getClass().getClassLoader();
        // 代理对象的类型，即其中有哪些方法
        Class[] interfaces = new Class[]{ArithmeticCalculator.class};
        // 当调用代理对象的方法时，所执行的代码就在 InvocationHandler 中
        InvocationHandler handler = new InvocationHandler() {
            /**
             * @param proxy 正在返回的那个代理对象，一般情况下在 invoke 方法中都不使用该对象
             * @param method 正在被调用的方法
             * @param args 调用方法时传入的参数
             * @return
             * @throws Throwable
             */
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                String methodName = method.getName();
                // 日志
                System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
                // 执行方法
                Object result = method.invoke(target, args);
                // 日志
                System.out.println("The method " + methodName + " ends with " + result);
                return result;
            }
        };
        proxy = (ArithmeticCalculator) Proxy.newProxyInstance(loader, interfaces, handler);

        return proxy;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        ArithmeticCalculator target = new ArithmeticCalculatorImpl();

        ArithmeticCalculator proxy = new ArithmeticCalculatorLoggingProxy(target).getLoggingProxy();

        int result = proxy.add(1, 2);
        System.out.println(result);

        proxy.sub(4, 2);
        System.out.println(result);
    }
}
```

输出结果如下所示：

```
The method add begins with [1, 2]
The method add ends with 3
3
The method sub begins with [4, 2]
The method sub ends with 2
3
```

从结果中可以看到，「业务代码」是简洁的，我们使用动态代理的方式，实现了打印日志的功能。但是这种方式在实际开发中较为繁琐，需要写较多代码。而我们用得最多的就是通过 AOP 的方式进行实现。

## AOP

AOP（Aspect-Oriented Programming, **面向切面编程**）是一种新的方法论，是对传统 OOP（Object-Oriented Programming, 面向对象编程）的补充。

AOP 的主要编程对象是**切面**，而**切面模块化横切关注点**。

在使用 AOP 编程时，仍然需要定义公共功能，但可以明确的定义这个功能在哪里，以什么方式应用，并且不需要修改受影响的类。这样的话，**横切关注点就被模块化到特殊的对象（切面）里了**。其好处在于：

- 每个事物的逻辑位于一个位置，代码不分散，便于维护和升级。
- 业务模块更简洁，只包含核心业务代码。

与 AOP 相关的的术语如下：

- 切面（Aspect）：**横切关注点（跨越应用程序多个模块的功能）被模块化的特殊对象**。

- 通知（Advice）：**切面必须要完成的工作**。

- 目标（Target）：**被通知的对象**。

- 代理（Proxy）：**向目标对象应用通知之后所创建的对象**。

- 连接点（JoinPoint）：

  程序执行的某个特定位置

  。例如，类中某个方法调用前、调用后、方法抛出异常后等。

  - 连接点由两个信息确定：方法表示的程序执行点和相对点表示的方位。例如，ArithmeticCalculator#add()方法执行前的连接点，执行点为 ArithmeticCalculator#add()；**方位**为该方法执行前的位置。

- 切点（PointCut）：每个类都有多个连接点。例如，ArithmeticCalculator 的所有方法实际上都是连接点，即

  连接点是程序类中客观存在的事务

  。AOP 通过切点定位到特定的连接点。

  - 例如，连接点相当于数据库中的记录，切点相当于查询条件。
  - 切点和连接点不是一对一的关系，一个切点匹配多个连接点。切点通过 org.springframework.aop.Pointcut 接口进行描述，它使用类和方法作为连接点的查询条件。

代码如下所示：

```java
public interface ArithmeticCalculator {

    int add(int i, int j);

    int sub(int i, int j);

    int mul(int i, int j);

    int div(int i, int j);
}
```

```java
@Component("arithmeticCalculator")
public class ArithmeticCalculatorImpl implements ArithmeticCalculator {

    @Override
    public int add(int i, int j) {
        int result = i + j;
        return result;
    }

    @Override
    public int sub(int i, int j) {
        int result = i - j;
        return result;
    }

    @Override
    public int mul(int i, int j) {
        int result = i * j;
        return result;
    }

    @Override
    public int div(int i, int j) {
        int result = i / j;
        return result;
    }
}
```

```java
/**
 * 把该类声明为切面
 * 1. 需要把该类放入到 IoC 容器中；
 * 2. 然后添加一个切面注解
 *
 * @date 2021/07/28
 */
@Aspect
@Component
public class LoggingAspect {

    // 前置通知：在目标方法开始之前执行
    @Before("execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.add(int, int))")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("The method " + methodName + "begins with " + args);
    }
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 配置自动扫描的包 -->
    <context:component-scan base-package="com.example.springdemo.aop.impl"/>

    <!-- 使 AspectJ 注解起作用：自动为匹配的类生成代理对象 -->
    <aop:aspectj-autoproxy/>
</beans>
```

```java
public class Main {
    public static void main(String[] args) {
        // 创建 Spring 的 IoC 容器
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContextAop.xml");
        // 从容器中获取 bean 的实例
        ArithmeticCalculator arithmeticCalculator = (ArithmeticCalculator) context.getBean("arithmeticCalculator");
        // 使用 bean
        int result = arithmeticCalculator.add(1, 2);
        System.out.println("add result: " + result);

        result = arithmeticCalculator.div(9 , 3);
        System.out.println("div result: " + result);
    }
}
```

输出结果如下：

```
The method addbegins with [1, 2]
add result: 3
div result: 3
```

如果想要让 ArithmeticCalculator 接口中的所有方法都设置前置通知的话，则可以将`@Before`注解中的`add`方法用通配符`*`替换，如下所示：

```java
@Before("execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))")
```

输出结果如下所示：

```
The method addbegins with [1, 2]
add result: 3
The method divbegins with [9, 3]
div result: 3
```

其中，AspectJ 支持 5 中类型的通知注解：

- @Before：前置通知，在目标方法执行前执行。
- @After：后置通知，在目标方法执行后（无论是否发生异常）执行。
  - 在后置通知中，还不能访问目标方法执行的结果。
- @AfterReturning：返回通知，在目标方法返回结果之后执行。
- @AfterThrowing：异常通知，在目标方法抛出异常之后执行。
- @Around：环绕通知，围绕着目标方法执行。

针对**切入点表达式**，如果将其修改为`@Before("execution(* ArithmeticCalculator.add(..))")`，则其中`*`代表匹配任意修饰符及任意返回值，参数列表中的`..`表示匹配任意数量的参数。也就是说，切入点表达式是根据方法的签名来匹配各种方法的。

此外，可以在通知方法中声明一个类型为 JoinPoint 的参数，然后就可以访问方法的名称或者参数值了。

下面将剩余的其他通知类型进行说明：

```java
@Aspect
@Component
public class LoggingAspect {

    // 前置通知：在目标方法开始之前执行
    @Before("execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("beforeMethod: The method " + methodName + " begins with " + args);
    }

    // 在后置通知中，还不能访问目标方法执行的结果
    @After("execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))")
    public void afterMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("afterMethod: The method " + methodName + " ends with " + args);
    }

    // 返回通知：在目标方法返回结果之后执行
    // 返回通知是可以访问到方法的返回值的
    @AfterReturning(value = "execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))", returning = "result")
    public void afterReturningMethod(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("afterReturningMethod: The method " + methodName + " ends with " + result);
    }


    // 异常通知
    @AfterThrowing(value = "execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))", throwing = "exception")
    public void afterThrowingMethod(JoinPoint joinPoint, Exception exception) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("afterThrowingMethod: The method " + methodName + " occurs exception: " + exception);
    }

    // 环绕通知需要携带 ProceedingJoinPoint 类型的参数
    // 环绕通知类似于动态代理的全过程：ProceedingJoinPoint 类型的参数可以决定是否执行目标方法
    // 且环绕通知必须有返回值，返回值即为目标方法的返回值
    @Around(value = "execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))")
    public Object aroundMethod(ProceedingJoinPoint proceedingJoinPoint) {

        Object result = null;
        String methodName = proceedingJoinPoint.getSignature().getName();

        // 执行目标方法
        try {
            // 前置通知
            System.out.println("The method " + methodName + " begins with" + Arrays.asList(proceedingJoinPoint.getArgs()));
            result = proceedingJoinPoint.proceed();
            // 返回通知
            System.out.println("The method " + methodName + " ends with result: " + result);
        } catch (Throwable throwable) {
            // 异常通知
            System.out.println("The method " + methodName + " occurs exception: " + throwable);
            throw new RuntimeException(throwable);
        }
        // 后置通知
        System.out.println("The method " + methodName + " ends");

        return result;
    }
}
```

对于**环绕通知**，虽然它的功能是最强的，即类似于动态代理的方式，但是并不代表是经常使用的。

### 切面的优先级

可以在切面类上添加一个`@Order()`注解，值越小，则优先级越高。

此外，还可以复用切入点表达式，直接新建一个方法，然后使用`@Pointcut()`注解即可，如下所示：

```java
@Order(1)
@Aspect
@Component
public class LoggingAspect {

    // 定义一个方法，用于声明切入点表达式
    // 一般情况下，该方法中不需要再加入其他代码
    @Pointcut("execution(public int com.example.springdemo.aop.impl.ArithmeticCalculator.*(int, int))")
    public void declareJoinPointExpression() {

    }

    // 前置通知：在目标方法开始之前执行
    @Before("declareJoinPointExpression()")
    public void beforeMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("beforeMethod: The method " + methodName + " begins with " + args);
    }

    // 在后置通知中，还不能访问目标方法执行的结果
    @After("declareJoinPointExpression()")
    public void afterMethod(JoinPoint joinPoint) {
        String methodName = joinPoint.getSignature().getName();
        List<Object> args = Arrays.asList(joinPoint.getArgs());
        System.out.println("afterMethod: The method " + methodName + " ends with " + args);
    }

    // 返回通知：在目标方法返回结果之后执行
    // 返回通知是可以访问到方法的返回值的
    @AfterReturning(value = "declareJoinPointExpression()", returning = "result")
    public void afterReturningMethod(JoinPoint joinPoint, Object result) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("afterReturningMethod: The method " + methodName + " ends with " + result);
    }


    // 异常通知
    @AfterThrowing(value = "declareJoinPointExpression()", throwing = "exception")
    public void afterThrowingMethod(JoinPoint joinPoint, Exception exception) {
        String methodName = joinPoint.getSignature().getName();
        System.out.println("afterThrowingMethod: The method " + methodName + " occurs exception: " + exception);
    }

    // 环绕通知需要携带 ProceedingJoinPoint 类型的参数
    // 环绕通知类似于动态代理的全过程：ProceedingJoinPoint 类型的参数可以决定是否执行目标方法
    // 且环绕通知必须有返回值，返回值即为目标方法的返回值
    @Around("declareJoinPointExpression()")
    public Object aroundMethod(ProceedingJoinPoint proceedingJoinPoint) {

        Object result = null;
        String methodName = proceedingJoinPoint.getSignature().getName();

        // 执行目标方法
        try {
            // 前置通知
            System.out.println("The method " + methodName + " begins with" + Arrays.asList(proceedingJoinPoint.getArgs()));
            result = proceedingJoinPoint.proceed();
            // 返回通知
            System.out.println("The method " + methodName + " ends with result: " + result);
        } catch (Throwable throwable) {
            // 异常通知
            System.out.println("The method " + methodName + " occurs exception: " + throwable);
            throw new RuntimeException(throwable);
        }
        // 后置通知
        System.out.println("The method " + methodName + " ends");

        return result;
    }
}
```

