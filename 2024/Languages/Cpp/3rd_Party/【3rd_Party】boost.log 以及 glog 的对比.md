#3rd_Party 

日志能方便地诊断程序原因、统计程序运行数据，是大型软件系统必不可少的组件之一。本文将从设计上和功能上对比 C++ 语言常见的两款日志库： `boost::log` 和 `google-glog` 。

## 设计

boost::log 的设计主要有日志器（ Logger ）、日志核心（ Logging core ）、 Sink 前后端（ frontend, backend ）组成。日志文本以及日志环境由日志器（ Logger ）负责搜集，日志核心负责处理日志数据（例如全局过滤、将日志记录传递给 Sink ）， Sink 前端分为同步、异步以及不考虑线程同步问题的版本，它们负责将日志记录传递给 Sink 后端处理。 Sink 后端负责把日志记录格式化并输出到不同的介质中（例如日志文件、报警以及统计源中）。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_230821080159_Logging%20Core.png)

glog 的设计比较简单，核心类由 LogMessage 组成，它负责初始化日志信息（源代码文件名、源代码行号、输出方法，相当于 boost::log 中的 Logger 功能）、将日志信息传递给不同的 Sink （相当于 boost::log 中的 Logging core ）。 LogFileObject 为默认文件输出对象，每个 severity 拥有一个此对象，它负责将日志信息格式化并写入文件。 Sink 由用户自定义，负责接收并处理日志消息。它的核心类图如下所示。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_230821080246_Logging%20Core2.png)

## 功能

### 严重性分级

boost::log 的默认分级包括 trace, debug, info, warning, error, fatal ，支持自定义分级。而 glog 仅仅支持 INFO, WARNING, ERROR, FATAL ，如果想要扩展严重性分级，需要使用 VLOG 宏，通过全局变量或者命令行来过滤。如果不使用自定义 Sink ， glog 默认是以严重性等级将日志分散保存到不同文件中，如果需要将自定义等级（ VLOG ）输出到文件，需要修改库源码（参考 LogMessage::SendToLog ）。如果给 glog 的日志级别设置为 FATAL ， glog 在写完日志之后会自动终止程序。

### 日志过滤

glog 通过设置 stderrthreshold 全局变量或者命令行参数来实现大于等于某个级别的日志记录到 stderr 以及 log 文件中。而 boost::log 支持两层过滤，可以通过 core::set_filter 设置全局过滤器，也能通过 sink::set_filter 设置 sink 过滤器。配合 [lambda 表达式](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/expressions.html) 或者函数对象， boost::log 可以生成无论多么复杂的过滤器。（例如只把某个级别的日志写入文件，或者把统计日志、 trace 日志分别重定向到不同的 sink 中，然后输出到不同的介质）。

### 线程安全

glog 仅支持同步日志。它的写入代码可以参考 LogMessage::Flush ，在写入日志的时候使用了互斥锁，因此性能会受 I/O 速度影响。有两个支持异步日志的非官方 glog 库，他们分别是 [g2log](http://www.codeproject.com/Articles/288827/g-log-An-efficient-asynchronous-logger-using-Cplus) 和 [g3log](https://github.com/KjellKod/g3log) ，均采用 C++11 编写。 boost::log 支持 [同步和异步 sink](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/sink_frontends.html) ，同步 sink 在将日志传递给 backend 时会加 [互斥锁](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/sink_frontends.html#log.detailed.sink_frontends.sync) 。 boost::log 的同步异步 sink 切换起来非常方便，只需要修改类型名就可以了。

### 自定义 sink 功能

boost::log 和 glog 都支持自定义 sink ，可以实现一条日志信息复制分流到多个 sink 进行处理，也可以在 sink 中以不同的格式输出日志。但是 glog 的 sink 没有过滤器，能获取到的信息也有限。而 boost 将日志抽象成 record 对象，不仅仅包含了日志的文本，还可以包含更丰富的自定义属性。

### 自定义属性功能

有时程序需要记录的信息往往不仅仅包含一条消息，还需要包含执行环境的一些属性（例如网络对端的 IP 地址）。 boost::log 提供了 [属性集](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/attributes.html) 功能，属性集不仅仅包括常用的数据（例如计数器、时间、计时器、线程 ID 等信息）还支持自定义属性。用户可以将程序的任意上下文放入日志记录对象中，然后在 sink 中进行处理。而 glog 不支持此功能，它的 sink 能处理的数据都以函数参数的形式提供给回调函数。

### 自定义日志器（ Logger ）

glog 支持为不同严重级别的日志设定日志器，可以通过这点来将不同日志级别的日志分散到不同的文件中。 boost::log 也支持自定义日志源，但它不是用来过滤级别的（因为过滤功能用 sink 的 filtering 就够了），它的日志源可以包含 [特定环境](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/sources.html#log.detailed.sources.channel_logger) 的信息（例如在网络连接 network_connection 中的日志源可以携带远程 IP 地址这个属性，这样从那个日志源发出的每一条日志信息都包含此属性）。

### 滚动日志

glog 通过设置 FLAGS_max_log_size 这个全局变量来实现日志尺寸大小。 boost::log 则支持不同的 sink 按照文件大小、时间等参数来 [旋转](http://boost-log.sourceforge.net/libs/log/doc/html/log/detailed/sink_backends.html#log.detailed.sink_backends.text_file.file_rotation) 。如果想要 glog 支持按照时间旋转日志，需要 [改动](http://blog.csdn.net/vv_demon/article/details/9378915) 源代码。

### 宽字符支持

boost::log 支持 [宽字符](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/tutorial/wide_char.html) 日志， sink 可以通过设置 locale 进行必要的编码转换。而 glog 仅仅支持 const char* 类型的日志消息。

### 参数配置

glog 的参数配置（例如缓冲日志时间、最大日志大小、日志文件夹）大多以全局变量或者命令行参数的形式，并且它们控制的是全局范围的参数。 boost::log 可以针对不同 sink 设置不同参数。

### 格式化

除非自定义 sink ，默认情况下 glog 写入文件的格式是固定的（参考 LogFileObject::Write ），而 boost::log 如果想要设置不同的格式，只需要传递一个 formatter 对象给 sink 即可。

### 调试支持

glog 提供 DLOG 等宏，这些宏在没有定义 NDEBUG 时有效。 boost::log 则需要程序员自己条件输出。通过不同的 sink/filter 可以实现 glog 同样的功能。

### 条件日志

glog 支持 LOG_IF ， LOG_EVERY_N 等条件日志宏。 boost::log 没有提供类似的宏，但是可以通过 filter 实现。

### 失败信号处理器

在 Linux 操作系统中， glog 提供 google::InstallFailureSignalHandler() 函数，当程序收到诸如 SIGSEGV 等会导致进程终止的信号时， glog 的信号处理器可以输出 stack trace 信息，方便调试。 glog 原生是为 Linux 操作系统准备的，因此对 Windows 的支持很有限，这个功能也不支持。 boost::log 没有实现这个功能。

### 文档

boost::log 的文档很全面，例程也很丰富， Google 上的资料也很多。 glog 官方只有 [一篇](https://google-glog.googlecode.com/svn/trunk/doc/glog.html) 简介，细节问题则分散在开源社区的 issue 中（例如 [Rolling File Log](https://code.google.com/p/google-glog/issues/detail?id=22) ），搜索引擎上的资料也少。

### 其他

- boost::log 提供很多使用小工具，例如支持日志排序、输出操作、 binary dump 操作（ logging::dump(packet.data(), packet.size() 对于输出内存块很方便）。
    
- boost::log 支持从 [配置文件](http://www.boost.org/doc/libs/1_59_0/libs/log/doc/html/log/detailed/utilities.html#log.detailed.utilities.setup.settings_file) 中读取日志配置信息，可以通过修改配置文件来修改日志过滤器、格式等配置。
    
- 对于不同的输出方式， boost::log 提供了一些现成的 [sink backend](http://boost-log.sourceforge.net/libs/log/doc/html/log/detailed/sink_backends.html) ，例如输出到 Windows 事件日志、调试器、 Linux syslog 接口、文本文件等。
    
- glog 提供了一些类似单元测试的宏，例如 CHECK_EQ, CHECK_NE 等，可以有条件地终止程序， glog 还提供了类似 static_assert 的 GOOGLE_GLOG_COMPILE_ASSERT 。
    

## 总结

glog ：小巧精简，源代码简洁易懂，便于学习。缺点是不太灵活，扩充功能需要修改库源码。文档资料少。 boost::log ：功能丰富，可扩展性强，文档全面。现代化的 C++ 设计，泛型编程思维，值得研究。搜索引擎中的资料也多。缺点是上手慢，体积大。一旦掌握之后，用起来将会得心应手，游刃有余。

如果需要丰富的功能（如输出到网络 /XML 、统计功能、更多的个性化设置），建议选择 boost::log 。如果需要快速上手，拿来即用，对功能没有太高的要求，可以选择 glog 。