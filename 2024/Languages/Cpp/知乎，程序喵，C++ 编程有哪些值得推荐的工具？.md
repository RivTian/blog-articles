
本人只会Linux平台C++开发，介绍下自己常用的C++相关工具：

-   **IDE：**VSCode（画图会使用VSCode里的drawio插件）、Clion（付费，但是这钱花的也值）、XCode（苹果平台的）、emacs。
-   **代码调试工具：**gdb、lldb、valgrind。
-   **构建系统**：CMake、Bazel、Ninja。
-   **静态代码检测工具**：[cppcheck](https://www.zhihu.com/search?q=cppcheck&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)、Clang-Tidy、PC-lint、[SonarQube+sonar-cxx](https://www.zhihu.com/search?q=SonarQube%2Bsonar-cxx&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)、Facebook的infer、Clang Static Analyzer。
-   **[内存泄漏](https://www.zhihu.com/search?q=%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)检测工具**：valgrind、ASan、mtrace、ccmalloc、debug_new。
-   **profiling工具**：gperftools、perf、intel VTune、AMD CodeAnalyst、[gnu prof](https://www.zhihu.com/search?q=gnu%20prof&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)、Quantify。

关于性能方面的工具完全可以看这个网站：

* https://www.brendangregg.com/linuxperf.html

里面很多相当有用的图片，比如：

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/20230509074139.png)

-   **网络I/O**：dstat、**[tcpdump](https://www.zhihu.com/search?q=tcpdump&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)**（推荐）、sar
-   **磁盘I/O**：**iostat**（推荐）、dstat、sar
-   **[文件系统](https://www.zhihu.com/search?q=%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)空间**：**df**
-   **内存容量**：free、**vmstat**（推荐）、sar
-   **进程内存分布**：**pmap**
-   **CPU负载**：uptime、**top**
-   **[系统调用](https://www.zhihu.com/search?q=%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)追踪**：**strace**（推荐）
-   **网络吞吐量**：**iftop**、nethogs、sar
-   **[网络延迟](https://www.zhihu.com/search?q=%E7%BD%91%E7%BB%9C%E5%BB%B6%E8%BF%9F&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)**：**ping**
-   **CPU使用率**：**pidstat**（推荐）、vmstat、[mpstat](https://www.zhihu.com/search?q=mpstat&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A1992218543%7D)、top、sar、time
-   **上下文切换**：**pidstat**（推荐）、vmstat、perf


