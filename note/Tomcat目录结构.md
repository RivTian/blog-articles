> 前排提醒：本文不涉及如何安装 Tomcat，仅分析Tomcat目录结构。
>
> Tomcat 版本：apache-tomcat-8.5.71

Tomcat 服务器是一个免费的开放源代码的 Web 应用服务器，属于轻量级应用服务器，在中小型系统和并发访问用户不是很多的场合下被普遍使用，是开发和调试 JSP 程序的首选。十分有必要了解Tomcat目录结构。

## 总结概括：

### 一级目录

> bin ——Tomcat执行脚本目录
>  conf ——Tomcat配置文件
>  lib ——Tomcat运行需要的库文件（JARS）
>  logs ——Tomcat执行时的LOG文件
>  temp ——Tomcat临时文件存放目录
>  webapps ——Tomcat的主要Web发布目录（存放我们自己的JSP,SERVLET,类）
>  work ——Tomcat的工作目录，Tomcat将翻译JSP文件到的Java文件和class文件放在这里。

### 二级目录（仅列出一级目录下几个重要的文件）

(1) bin目录下的文件

* catalina.sh 用于启动和关闭tomcat服务器
* configtest.sh 用于检查配置文件
* startup.sh 启动Tomcat脚本
* shutdown.sh 关闭Tomcat脚本

 (2) conf目录下的文件

* server.xml Tomcat 的全局配置文件
* web.xml 为不同的Tomcat配置的web应用设置缺省值的文件
* tomcat-users.xml Tomcat用户认证的配置文件

 (3) lib目录下的文件

* 包含被Tomcat使用的各种各样的jar文件。

 (4) logs目录下的文件

* localhost_access_log.2013-09-18 .txt 访问日志
* localhost.2013-09-18.log 错误和其它日志
*  manager.2013-09-18.log 管理日志
*  catalina.2013-09-18.log Tomcat启动或关闭日志文件

 (5) webapps目录下的文件

* 含Web应用的程序（JSP、Servlet和JavaBean等） 

 (6) work目录下的文件

* 由Tomcat自动生成，这是Tomcat放置它运行期间的中间( intermediate)文件(诸如编译的JSP文件)地方。

  如果当Tomcat运行时，你删除了这个目录那么将不能够执行包含JSP的页面。

---

$$
QAQ
$$

---

## 目录结构

解压Tomcat后的目录结构如下图

![解压Tomcat后的目录结构图](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165137.png)

各目录及文件说明

| 目录及文件               | 说明                                                         |
| :----------------------- | :----------------------------------------------------------- |
| bin                      | 用于存放 Tomcat的启动、停止等批处理脚本和Shell脚本           |
| bin/startup. bat         | 用于在 Windows下启动 Tomcat                                  |
| bin/startup.sh           | 用于在 Linux下启动 Tomcat                                    |
| bin/shutdown. bat        | 用于在 Windows下停止 Tomcat                                  |
| bin/shutdown.sh          | 用于在 Linux下停止 Tomcat                                    |
| conf                     | 用于存放 Tomcat的相关配置文件                                |
| conf/Catalina            | 用于存储针对每个虚拟机的 Context 配置                        |
| conf/context.xml         | 用于定义所有Web应用均需要加载的 Context 配置，如果Web应用指定了自己的context.xml，那么该文件的配置将被覆盖 |
| conf/catalina.properties | Tomcat环境变量配置                                           |
| conf/catalina.policy     | 当 Tomcat在安全模式下运行时，此文件为默认的安全策略配置      |
| conf/logging.properties  | Tomcat日志配置文件，可通过该文件修改 Tomcat日志级别以及日志路径等 |
| conf/server.xml          | Tomcat服务器核心配置文件，用于配置 Tomcat的链接器、监听端口、处理请求的虚拟主机等。可以说，Tomcat主要根据该文件的配置信息创建服务器实例 |
| conf/tomcat-users.xml    | 用于定义 Tomcat默认用户及角色映射信息，Tomcat的 Manager模块即用该文件中定义的用户进行安全认证 |
| conf/web.xml             | Tomcat中所有应用默认的部署描述文件，主要定义了基础 Servlet和MIME映射。如果应用中不包含 Web. xml，那么 Tomcat将使用此文件初始化部署描述，反之，Tomcat会在启动时将默认部署描述与自定义配置进行合并 |
| lib                      | Tomcat服务器依赖库目录，包含 Tomcat服务器运行环境依赖lar包   |
| logs                     | Tomcat默认的日志存放路径                                     |
| webapps                  | Tomcat默认的Web应用部署目录                                  |
| work                     | 存放Web应用JSP代码生成和编译后产生的class文件目录            |
| temp                     | 存放tomcat在运行过程中产生的临时文件                         |

## bin目录

用于存放 Tomcat的启动、停止等批处理脚本和Shell脚本

![bin 目录](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165223.png)

## conf目录

用于存放 Tomcat的相关配置文件

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165255.png)

## lib目录

Tomcat服务器依赖库目录，包含 Tomcat服务器运行环境依赖lar包

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165313.png)

## webapps目录

Tomcat默认的Web应用部署目录

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165338.png)

## temp目录

存放tomcat在运行过程中产生的临时文件

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20210925165354.png)



## work 目录

Tomcat的工作目录，Tomcat将翻译JSP文件到的Java文件和class文件放在这里。

