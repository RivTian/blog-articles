
#3rd_Party 

## 常用简写：

```cpp
namespace logging = boost::log;
namespace src = boost::log::sources;
namespace expr = boost::log::expressions;
namespace sinks = boost::log::sinks;
namespace attrs = boost::log::attributes;
namespace keywords = boost::log::keywords;
```

## 要点：

1. 结构图要牢记在心；
    
2. `trivial`头文件可用于一般的控制台输出，日志等级被定义在改头文件；
    
3. 全局日志等级过滤使用`logging::core::get()->set_filter();`
    
4. 如果不仅仅需要简单的控制台输出，这时候就要添加`sink`，使用`logging::add_file_log`可添加文件sink后端，格式如下：

```cpp
logging::add_file_log
(   
    keywords::file_name="sample_%N.log",        //文件名格式
    keywords::rotation_size=10*1024*1024,       //超过此大小自动建立新文件
    keywords::time_based_rotation=sinks::file::rotation_at_time_point(0,0,0),   //每隔指定时间重建新文件
    keywords::format="[%TimeStamp%]:%Message%"      //日志消息格式
);
```

除了这种语法外，也可以建立`sinks::text_file_backend`后端，用之初始化一个sink。 file rotation的选项有很多，可以指定日期、时间间隔、文件大小，甚至可以指定自己的谓词。 此外，可以设置在建立新文件后首先执行(`sink->locked_backend()->set_open_handler(xx)`)，以及rotation之前最后执行的语句。 可以综合管理这些日志文件，限制输出文件夹，日志总大小限制，以及最低磁盘空间限制。如下：

```cpp
sink->locked_backend()->set_file_collector(sinks::file::make_collector(
        keywords::target="logs",                //目标文件夹
        keywords::max_size=16*1024*1024,        //所有日志加起来的最大大小,
        keywords::min_free_space=100*1024*1024  //最低磁盘空间限制
        ));
```

如果多个后端制定了相同的存放文件夹，限制取其中最严格的，另外，注意避免不同后端日志的命名冲突问题。 注意使用`sink->locked_backend()->scan_for_files()` 来扫描其他实例建立的日志文件。

以上都是使用程序本身使用单日志文件的情况，如果需要根据请求的不同将消息分发到不同的文件，可以使用`sinks::text_mutlifile_backend`。该后端常用于多线程调试。

5. 除了文件后端，更常用的是文本流后端`sinks::text_ostream_backend`，与别的后端不同，文本流后端可以添加多个输出对象，这些对象由于都在同一个`sink`中，所以输出格式是一样的，这种做法的性能比添加文件后端更高，但是会失去对文件的控制能力。
    
6. 对于大规模应用程序，为了方便查看记录，各模块的日志应该相互独立，因此一个 logger 一般是不够的，我们需要自己建立 logger。logger 的建立方法很简单，new一个 `src::logger` 就可以了…
    

如果真的只需要一个logger，除了使用前面提到的trivial中的宏以外，也可以用`BOOST_LOG_INLINE_GLOBAL_LOGGER_DEFAULT(my_logger,src::logger_mt)`，自己定义属于自己的全局logger。然后在需要的时候使用`src::logger_mt& lg=my_logger::get()`获得logger的单例引用（这里最好是线程安全的logger，显然）。

logger的使用方法：

```cpp
logging::record rec=log.open_record();
if(rec)
{
	logging::record_ostream strm(rec);
	strm<<"Hello,World!";
	strm.flush();
	lg.push_record(boost::move(rec));
}
```

以上可以简化成一个宏：`BOOST_LOG(lg)<<"Hello World"`

7. `attribute`是log record的附加信息，不同于一般的消息记录，属性可以被单独拿出来处理，作为某种过滤条件，或者其他使用。属性分为全局属性，特定线程属性和特定源的属性。
    

常用的属性，如时间戳、计数器，`boost.log`都已经有实现好的版本，直接使用`logging::add_common_attributes()` 可以一次性获得`LineID`, `TimeStamp`, `ProcessID`, `ThreadID` 这些常用属性（单线程程序没有线程ID）。

可以自己注册安全等级，定义相关枚举，然后使用其作为`src::serverity_logger<>`的模板参数初始化，就得到了自定义`Severity`属性的logger。这种logger可以使用`BOOST_LOG_SEV(logger,serverity)`来记录。

特定范围的属性可以用来做一些特殊日志，比如需要评估性能的地方可以使用`BOOST_LOG_SCOPED_THREAD_ATTR("Timeline",attrs::timer());`注册一个时间线标签。

此外，可以给属性定义占位符，相当于注册为关键字，方便在流中使用。关于属性的函数大多定义在`expr`命名空间里面。

8. 格式化输出消息，前文有`set_formatter`的用法。格式化消息可以使用stl格式`expr::stream<<xxx`，也可以使用`boost::format`格式，即`expr::format("%1%)%xxx`这种。
    

为了取得最大灵活性，可以自定义一个`formatter`。

9. 过滤子，如前文，可以使用`sink.set_filter`来设置，可以使用的过滤条件包括布尔表达式和lambda表达式等，如果想要确定有没有某个属性，可使用`expr::has_attr()`。
    
10. 宽字符。使用windows的话，这块算是最恶心的部分了
    

### 辅助函数

`boost.log`定义了大量辅助函数实现常用功能。

`logging::add_console_log()`可以直接添加控制台sink，返回`boost::shared_ptr<sinks::synchronous_sink<sinks::text_ostream_backend>>`，可利用返回值设置过滤器或其他属性。

`logging::add_file_log()`，前文已经提过，可以直接得到文件sink…

最后`logging::add_common_attributes`可以获得常用属性，前文也有详解。

### 示例

完全封装`boost.log`的工程量比较大，不过对于小规模工程，使用一个初始化函数就足够了，但是这会导致整个项目与`boost`深度耦合。

```cpp
/*
 *  FILE: boost.log.init.hpp
 *  INSTRUCTION: 这个文件用于初始化boost.log
 *  DETAIL: 这里重新定义了日志级别和对应的文本内容. 
 *          文件大小限制等信息被硬编码在cpp文件中，如有需求可以修改。
 */
#ifndef BOOST_LOG_INIT_TAIRAN_HPP
#define BOOST_LOG_INIT_TAIRAN_HPP

#include <iostream>
#include <boost/date_time/posix_time/posix_time.hpp>
#include <boost/log/support/date_time.hpp>
#include <boost/log/common.hpp>
#include <boost/log/expressions.hpp>
#include <boost/log/expressions/keyword.hpp>

#include <boost/log/attributes.hpp>
#include <boost/log/attributes/timer.hpp>
#include <boost/log/sources/logger.hpp>
#include <boost/log/sources/severity_logger.hpp>
#include <boost/log/sinks/sync_frontend.hpp>
#include <boost/log/sinks/text_file_backend.hpp>

#include <boost/log/utility/setup/file.hpp>
#include <boost/log/utility/setup/console.hpp>
#include <boost/log/utility/setup/common_attributes.hpp>

#include <boost/log/attributes/named_scope.hpp>

namespace logging = boost::log;
namespace attrs = boost::log::attributes;
namespace src = boost::log::sources;
namespace sinks = boost::log::sinks;
namespace expr = boost::log::expressions;
namespace keywords = boost::log::keywords;

enum SeverityLevel
{
    Log_Info,
    Log_Notice,
    Log_Debug,
    Log_Warning,
    Log_Error,
    Log_Fatal
};

// The formatting logic for the severity level
template< typename CharT, typename TraitsT >
inline std::basic_ostream< CharT, TraitsT >& operator<< (
    std::basic_ostream< CharT, TraitsT >& strm, SeverityLevel lvl)
{
    static const char* const str[] =
    {
        "Info",
        "Notice",
        "Debug",
        "Warning",
        "Error",
        "Fatal"
    };
    if (static_cast< std::size_t >(lvl) < (sizeof(str) / sizeof(*str)))
        strm << str[lvl];
    else
        strm << static_cast< int >(lvl);
    return strm;
}

BOOST_LOG_ATTRIBUTE_KEYWORD(log_severity, "Severity", SeverityLevel)
BOOST_LOG_ATTRIBUTE_KEYWORD(log_timestamp, "TimeStamp", boost::posix_time::ptime)
BOOST_LOG_ATTRIBUTE_KEYWORD(log_uptime, "Uptime", attrs::timer::value_type)
BOOST_LOG_ATTRIBUTE_KEYWORD(log_scope, "Scope", attrs::named_scope::value_type)

void g_InitLog();

#endif
```

实现：

```cpp
#include "boost.log.init.hpp"

void g_InitLog()
{
    logging::formatter formatter=
        expr::stream
        <<"["<<expr::format_date_time(log_timestamp,"%H:%M:%S")
        <<"]"<<expr::if_(expr::has_attr(log_uptime))
        [
            expr::stream<<" ["<<format_date_time(log_uptime,"%O:%M:%S")<<"]"
        ]

    <<expr::if_(expr::has_attr(log_scope))
        [
            expr::stream<<"["<<expr::format_named_scope(log_scope,keywords::format = "%n")<<"]"
        ]
    <<"<"<<log_severity<<">"<<expr::message;

    logging::add_common_attributes();

    auto console_sink=logging::add_console_log();
    auto file_sink=logging::add_file_log
        (
        keywords::file_name="%Y-%m-%d_%N.log",      //文件名
        keywords::rotation_size=10*1024*1024,       //单个文件限制大小
        keywords::time_based_rotation=sinks::file::rotation_at_time_point(0,0,0)    //每天重建
        );

    file_sink->locked_backend()->set_file_collector(sinks::file::make_collector(
        keywords::target="logs",        //文件夹名
        keywords::max_size=50*1024*1024,    //文件夹所占最大空间
        keywords::min_free_space=100*1024*1024  //磁盘最小预留空间
        ));

    file_sink->set_filter(log_severity>=Log_Warning);   //日志级别过滤

    file_sink->locked_backend()->scan_for_files();

    console_sink->set_formatter(formatter);
    file_sink->set_formatter(formatter);
    file_sink->locked_backend()->auto_flush(true);

    logging::core::get()->add_global_attribute("Scope",attrs::named_scope());
    logging::core::get()->add_sink(console_sink);
    logging::core::get()->add_sink(file_sink);
}
```

使用：

调用`g_initlog()`初始化一次以后，在任意位置声明logger，使用宏`BOOST_LOG_SEV(logger,SeverityLevel)<<"..."`来写入日志即可。