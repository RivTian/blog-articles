Boost是一个功能强大、构造精巧、跨平台、开源并且完全免费的C++程序库，有着“C++‘准’标准库”的美誉，值得每位C++程序员学习使用。

## 1 安装Boost

### 使用Homebrew安装

1. 下载安装[HomeBrew](http://brew.sh/)
2. `brew install boost`
3. 留意运行日志会显示头文件目录 `/usr/local/Cellar/boost/1.82.0_1/include`, lib目录`/usr/local/Cellar/boost/1.82.0_1/lib`

 MBP 14 M2 Pro 
 `/opt/homebrew/Cellar/boost/1.82.0_1`  
 

## 2. 在XCode项目中使用Boost

1. 新建一个Command Line Tool项目
2. 在Build Setings - Header Search Paths 增加头文件目录
3. 替换main.cpp中代码，运行！输入任意数字回车可看到结果。

```cpp
#include <iostream>
#include <boost/lambda/lambda.hpp>

int main(int argc, const char * argv[]) {
    
    printf("Please input any number:");
    using namespace boost::lambda;
    typedef std::istream_iterator<int> in;
    
    std::for_each(
                  in(std::cin), in(), std::cout << (_1 * 3) << " " );
    
    return 0;
}

```

## 3. 在XCode项目中使用Boost Lib库

Boost的很多功能都是直接在hpp头文件里实现的，比如上面的lambda例子不用导入任何lib就可以运行了。但也有一部分需要依赖指定lib库才能使用。比如下面这个正则表达式的例子:

```cpp
#include <iostream>
#include <boost/regex.hpp>

int main(int argc, const char * argv[]) {
    std::string   str = "2013-08-15";
    boost::regex  rex("(?<year>[0-9]{4}).*(?<month>[0-9]{2}).*(?<day>[0-9]{2})");
    boost::smatch res;
    
    std::string::const_iterator begin = str.begin();
    std::string::const_iterator end   = str.end();
    
    if (boost::regex_search(begin, end, res, rex))
    {
        std::cout << "Day:   " << res ["day"] << std::endl
        << "Month: " << res ["month"] << std::endl
        << "Year:  " << res ["year"] << std::endl;
    }
}
```

### 3.1 使用静态库

在Build Setings - Other linker flags `/usr/local/boost_1_63_0/stage/lib/libboost_regex.a`

使用命令行编译相当于

```bash
c++ -I /usr/local/boost_1_63_0 main.cpp -o main /usr/local/boost_1_63_0/stage/lib/libboost_regex.a
./main
```

如果这里直接使用 boost_regex, Xcode默认加载动态库。实际运用中可以考虑将目录中的动态库删除，只保留静态库，并在Build Setings - Library Search Paths 增加lib文件目录。

### 3.2 使用动态库

1. 在Build Setings - Library Search Paths 增加lib文件目录
2. 将lib文件目录中的 libboost_regex.dylib文件拖入项目
3. 确保在Build Phases - Link Bindary With Libraries中已经有该库
4. 在Build Phases - Copy Files, 复制 libboost_regex.dylib 到Products Directory

使用命令行编译相当于

```bash
c++ -I /usr/local/boost_1_63_0 main.cpp -o main -L/usr/local/boost_1_63_0/stage/lib/ -lboost_regex
cp /usr/local/boost_1_63_0/stage/lib/libboost_regex.dylib ./
./main
```