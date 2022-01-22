> 《Spring 学习笔记》系列文章是博主在学习过 Spring 后对其进行总结的入门系列博文，适合初入 Spring 的小白，如果你最近正在学习或者打算学习 Spring 的话，不妨随着本系列教程一起加深对 Spring 框架的学习吧。

## 1.导读

最近正在学习 Spring 框架，看了一些视频和笔记，将 Spring 的大部分内容了解了一遍。之前博主没有接触过 Java 框架，至于 Web 层的 Struts 、dao 层的 Hibernate 也没有直接去学习，反而是直奔 SSM ，即 Spring、SpringMVC、MyBatis。

因为 SSM 是目前主流的框架，许多企业都会用到这一容器框架，就 Spring 而言，它能够贯穿于表示层、业务逻辑层以及持久化层…先不说这么多，这点到具体篇幅里去讲。

《Spring 学习笔记之文章导读》是为了对这一系列做一个大纲总览，包括前期的 Spring 核心技术 IoC 与 DI，中期的 AOP 与 JdbcTemplate，以及后期的事务管理、整合 Junit、整合 web 这三个部分，每个部分又分为许多章节，为了方便查看，我将所有的内容以“目录树”的形式给出，如下所示。

## 2.大纲

**一、前期部分**

1. Spring 框架介绍
2. IoC
3. DI
4. Spring 核心 API
5. 使用 xml 配置 Bean
6. 使用注解配置 Bean

**二、中期部分**

1. AOP
   - AOP介绍
   - 手动代理
   - AOP通知类型
   - 利用 Spring 实现代理
   - 利用 AOP 实现代理
   - AspectJ
2. JdbcTemplate
   - 介绍
   - JdbcTemplate 结合 API
   - JdbcTemplate 结合 DBCP
   - JdbcTemplate 结合 C3P0
   - JdbcTemplate 结合 JdbcDaoSupport
   - JdbcTemplate 结合 properties
3. 事务管理
   - 介绍
   - 手动管理事务
   - 利用工厂 Bean 生成代理
   - 基于 xml 配置 AOP
   - 基于注解配置 AOP

**三、后期部分**

1. 整合 Junit
2. 整合 Web

## 3.总结

内容看起来可能不是很多，但每个部分对应的具体细节倒是不少。虽说是学习笔记，但这也是博主在学习过程中自己整理的知识点，毕竟刚学没多久，只是想通过写文章的方式锻炼自己总结问题的同时，也能够培养写作能力。这些文章难免会有遗漏或者错误之处，如果你发现了可以在下方留言，或者给我私信，我会和你一起学习、一起讨论。Peace