
#3rd_Party #Cpp 

## Unit Test 和 gtest 介绍

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231011063900_Test.png)

**单元测试**（ `Unit Test` ，模块测试）是开发者编写的一小段代码，用于检验被测代码的一个很小的、很明确的功能是否正确，通过编写单元测试可以在编码阶段发现程序编码错误，甚至是程序设计错误。

单元测试不但可以增加开发者对于所完成代码的自信，同时，好的单元测试用例往往可以在 **回归测试** 的过程中，很好地保证之前所发生的修改没有破坏已有的程序逻辑。因此，单元测试不但不会成为开发者的负担，反而可以在保证开发质量的情况下，加速迭代开发的过程。

**GoogleTest**是一个跨平台的(`Liunx`、`Mac OS X`、`Windows`、`Cygwin`、`Windows CE` and `Symbian`) `C++` 单元测试框架，**GoogleTest**由 `google` 公司发布, 且遵循 `New BSD License`（可用作商业用途）的开源项目, 为当前比较主流的 `C++` 单元测试框架，目前所在公司也在使用。

## gtest 安装、导入项目（Linux系统）

### 下载源码

我本地使用的系统参数：

```bash
bash-4.2$ uname -a
Linux yejy 3.10.0-514.el7.x86_64 #1 SMP Tue Nov 22 16:42:41 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

**gtest github** 地址：

> - https://github.com/google/googletest

下载源码：

```bash
bash-4.2$ git clone https://github.com/google/googletest
```



### 导入项目

#### 简单测试

下载源码后，接着就是将其导入到我们的项目中使用，如果你只是想简单测试一下，可以直接编译 gtest 源码，生成相应的静态库，将库和头文件拷贝到系统的头文件和库中，然后就可以直接写代码进行测试了，步骤如下：

```bash
bash-4.2$ cd googletest
bash-4.2$ cmake
bash-4.2$ make
bash-4.2$ cp libgtest*.a  /usr/lib 
bash-4.2$ cp –a include/gtest /usr/include
```

写一个简单的测试程序：

```cpp
#include<gtest/gtest.h>

int add(int a, int b){
    return a+b;
}

TEST(testCase, test0){
    EXPECT_EQ(add(4,3), 7); // 断言检测两参数是否相等
}

int main(int argc, char **argv)
{
  testing::InitGoogleTest(&argc, argv); // 初始化，所有测试都是这里启动的
  return RUN_ALL_TESTS(); // 运行所有测试用例
}
```



编译代码，当然你可以用 `make` 或者 `cmake` 编译都可以，具体输出：

```bash
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from testCase
[ RUN      ] testCase.test0
[       OK ] testCase.test0 (0 ms)
[----------] 1 test from testCase (0 ms total)
```



#### 工业生产

上面这种测试方法比较特殊，等于是把 `gtest` 库和 `gnu c` 库一样使用了，正常工作项目中，肯定不会这样用的。

正确的做法是 **以第三方库的形式直接将源码引入进项目**。可能有人就会说了，为什么一定要将源代码引入其中，而不先编译出静态库，然后导入其中呢，这样编译自己项目的时候不就不用再重新编译了吗？ 这里主要是考虑 **跨平台**，编译环境会有多种，需要多次编译，因此需要源码导入，同宿主项目一起编译。

我比较熟悉的编译工具是 **cmake**, 工作中使用的也是这个，该工具也是跨平台的，在编译大型跨平台项目时，很有优势，那这边就大致讲一下引入步骤，如果你对 `cmake` 很熟悉，那这边就很轻松了。

首先看一下引入后的代码结构，如下图：

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231011064123_unit_test.png)

重点是这个文件 **unit_test/CMakeLists.txt** ：

```cmake
file(GLOB SRC_FILES ./*.cpp)
file(GLOB HEADER_FILES ./*.h)

# 将给定目录添加到编译器用于搜索包含文件的目录中。相对路径被解释为相对于当前源目录。
# 相当于gcc命令的-I，告诉编译器到该目录中查找头文件
include_directories(${CMAKE_SOURCE_DIR}/src)
if(ENABLE_TEST)
    include_directories(
        ${CMAKE_SOURCE_DIR}/3rdlib/googletest/googlemock/include
        ${CMAKE_SOURCE_DIR}/3rdlib/googletest/googletest/include
        )
endif()

# 生成可执行文件 posix_thread_test.exx 
add_executable(posix_thread_test.exx 
            ${SRC_FILES}
            )

# 引入 gtest 库，posixthread 为源代码库
target_link_libraries(posix_thread_test.exx 
            gtest
            posixthread   
)

target_install(posix_thread_test.exx)
```



导入项目，主要就是看 **unit_test/CMakeLists.txt** 这个文件了，其他基本变化不大，如果你熟悉 `cmake` 很容易就能看懂。 至于图中的源码，是最近在封装 **Posix-thread** 时写的，源码大部分引用了陈硕老师的 `muduo` 网络库中的线程相关代码。

## gtest 具体使用

介绍一下断言，断言主要用来做一些逻辑判断，主要有以下两类接口:

> - ASSERT_XXX()： 如果断言失败，则测试处理终止。
> - EXPECT_XXX()： 非致命性失败，允许继续处理。

|                Test                 |                 Fatal                  |                NonFatal                |
| :---------------------------------: | :------------------------------------: | :------------------------------------: |
|           condition 为真            |         ASSERT_TRUE(condition)         |         EXPECT_TRUE(condition)         |
|           condition 为假            |        ASSERT_FALSE(condition)         |        EXPECT_FALSE(condition)         |
|                Equal                |          ASSERT_EQ(arg1,arg2)          |          EXPECT_EQ(arg1,arg2)          |
|              Not Equal              |          ASSERT_NE(arg1,arg2)          |          EXPECT_NE(arg1,arg2)          |
|              Less Than              |          ASSERT_LT(arg1,arg2)          |          EXPECT_LT(arg1,arg2)          |
|         Less Than or Equal          |          ASSERT_LE(arg1,arg2)          |          EXPECT_LE(arg1,arg2)          |
|            Greater Than             |          ASSERT_GT(arg1,arg2)          |          EXPECT_GT(arg1,arg2)          |
|        Greater Than or Equal        |          ASSERT_GE(arg1,arg2)          |          EXPECT_GE(arg1,arg2)          |
|           C String Equal            |        ASSERT_STREQ(str1,str2)         |        EXPECT_STREQ(str1,str2)         |
|         C String Not Equal          |        ASSERT_STRNE(str1,str2)         |        EXPECT_STRNE(str1,str2)         |
|         C String Case Equal         |      ASSERT_STRCASEEQ(str1,str2)       |      EXPECT_STRCASEEQ(str1,str2)       |
|       C String Case Not Equal       |      ASSERT_STRCASENE(str1,str2)       |      EXPECT_STRCASENE(str1,str2)       |
|   Verify that exception is thrown   | ASSERT_THROW(statement,exception_type) | EXPECT_THROW(statement,exception_type) |
|   Verify that exception is thrown   |      ASSERT_ANY_THROW(statement)       |      EXPECT_ANY_THROW(statement)       |
| Verify that exception is NOT thrown |       ASSERT_NO_THROW(statement)       |       EXPECT_NO_THROW(statement)       |

测试代码如下：

```cpp
#include <gtest/gtest.h>
#include <posix_thread.h>

void threadFunc()
{
  std::cout << "tid= "<< PosixThread::CurrentThread::tid() << std::endl;
}

TEST(PosixThreadTest, CreateThread)
{
  std::cout << "pid= " << ::getpid() << "   tid= " <<PosixThread::CurrentThread::tid() << std::endl;

  PosixThread::Thread t1(threadFunc);
  t1.start();
  ASSERT_TRUE(t1.started());
  EXPECT_FALSE(t1.started()); // 故意失败
  ASSERT_FALSE(t1.started()); // 故意失败
  std::cout << "t1.tid: " << t1.tid() << std::endl;
  std::cout << "thread name: " << t1.name().c_str() << std::endl;

  t1.join();

  std::cout << "CreateThread end !\n"
            << std::endl;
}

TEST(AtomicTest, AtomicInt64)
{
  std::cout << "pid= " << ::getpid() << "  tid= " <<PosixThread::CurrentThread::tid() << std::endl;

  PosixThread::AtomicInt64 a0;
  ASSERT_EQ(a0.get(), 0);
  ASSERT_EQ(a0.getAndAdd(1), 0);
  ASSERT_EQ(a0.get(), 1);
  ASSERT_EQ(a0.addAndGet(2), 3);
  ASSERT_EQ(a0.get(), 3);
  ASSERT_EQ(a0.incrementAndGet(), 4);
  ASSERT_EQ(a0.get(), 4);
  a0.increment();
  ASSERT_EQ(a0.get(), 5);
  ASSERT_EQ(a0.addAndGet(-3), 2);
  ASSERT_EQ(a0.getAndSet(100), 2);
  ASSERT_EQ(a0.get(), 100);
}
```



执行结果：

```bash
bash-4.2$ ./output/bin/posix_thread_test.exx 
[==========] Running 2 tests from 2 test cases.
[----------] Global test environment set-up.
[----------] 1 test from PosixThreadTest
[ RUN      ] PosixThreadTest.CreateThread
pid= 5297   tid= 5297
tid= 5298
/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:16: Failure
Value of: t1.started()
  Actual: true
Expected: false
/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:17: Failure
Value of: t1.started()
  Actual: true
Expected: false
[  FAILED  ] PosixThreadTest.CreateThread (0 ms)
[----------] 1 test from PosixThreadTest (0 ms total)

[----------] 1 test from AtomicTest
[ RUN      ] AtomicTest.AtomicInt64
pid= 5297  tid= 5297
[       OK ] AtomicTest.AtomicInt64 (0 ms)
[----------] 1 test from AtomicTest (0 ms total)

[----------] Global test environment tear-down
[==========] 2 tests from 2 test cases ran. (0 ms total)
[  PASSED  ] 1 test.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] PosixThreadTest.CreateThread

 1 FAILED TEST
```



从执行结果，我们可以很清楚的知道测试用例具体执行到哪一步，如果失败了，我们可以看到具体是哪一行代码出问题了，程序预期结果是什么，但是实际结果又是什么，输出十分详细。

我们还可以将测试结果导出到 `xml` 文件，通过参数：`--gtest_output` 实现。

```bash
bash-4.2$ ./output/bin/posix_thread_test.exx --gtest_output="xml:./test.xml"
bash-4.2$ cat test.xml 
<?xml version="1.0" encoding="UTF-8"?>
<testsuites tests="2" failures="1" disabled="0" errors="0" timestamp="2019-01-04T21:36:40" time="0" name="AllTests">
  <testsuite name="PosixThreadTest" tests="1" failures="1" disabled="0" errors="0" time="0">
    <testcase name="CreateThread" status="run" time="0" classname="PosixThreadTest">
      <failure message="/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:16&#x0A;Value of: t1.started()&#x0A;  Actual: true&#x0A;Expected: false" type=""><![CDATA[/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:16
Value of: t1.started()
  Actual: true
Expected: false]]></failure>
      <failure message="/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:17&#x0A;Value of: t1.started()&#x0A;  Actual: true&#x0A;Expected: false" type=""><![CDATA[/home/willy/myshare/thread-pool/unit_test/thread_test.cpp:17
Value of: t1.started()
  Actual: true
Expected: false]]></failure>
    </testcase>
  </testsuite>
  <testsuite name="AtomicTest" tests="1" failures="0" disabled="0" errors="0" time="0">
    <testcase name="AtomicInt64" status="run" time="0" classname="AtomicTest" />
  </testsuite>
</testsuites>
```



此外，在运行可执行目标程序时，可以使用 `--gtest_filter` 来指定要执行的测试用例，支持字符串正则匹配，主要如下几种常用情况：

```cpp
./output/bin/posix_thread_test.exx 没有指定filter，运行所有测试；
./output/bin/posix_thread_test.exx --gtest_filter=* 指定filter为*，运行所有测试；
./output/bin/posix_thread_test.exx --gtest_filter=PosixThreadTest.* 运行测试用例FooTest的所有测试；
./output/bin/posix_thread_test.exx --gtest_filter=*Null*:*Thread* 运行所有全名；
./output/bin/posix_thread_test.exx --gtest_filter=PosixThreadTest.*-PosixThreadTest.CreateThread 
运行测试用例FooTest的所有测试，但不包括PosixThreadTest.CreateThread。
```



**gtest** 还有很多方便你测试的功能，包括 **事件机制**, **参数化**, **死亡测试**, **运行参数**等，我们点到为止，如果想继续深入，可以参考这位博主的 gtest 系列, 很详细：
[玩转Google开源C++单元测试框架Google Test系列](http://www.cnblogs.com/coderzh/archive/2009/03/31/1426758.html)

## googlemock 使用

googlemock，是用于编写和使用C++ 模拟类的框架，在我们工作中，主要用来模拟应用程序的一部分，在单元测试用例编写过程中，**常常需要编写模拟对象来隔离被测试单元的“下游”或“上游”程序逻辑或环境，从而达到对需要测试的部分进行隔离测试的目的**，它可以帮助我们获得更好的系统设计并编写更好的测试。googlemock 同样遵循 New BSD License（可用作商业用途）的开源项目。

在开发过程中，经常出现各联调模块间，进度不一的情况；测试环境非常不稳定，易导致测试失败，导致达不到单元测试的目的，模仿对象提供了解决这些问题的方法：模仿对象符合实际对象的接口，但只包含用来“欺骗”测试对象并跟踪其行为的必要代码。因此，其实现往往比实际实现类简单很多。

官方教程：

> - https://github.com/google/googletest/blob/master/googlemock/docs/ForDummies.md

官方的 `Tutorial` 讲的很详细，我在`github`上也找了一个使用例子，很简洁，但是能很好的说明问题，大致代码如下：
mail_service.h文件：

```cpp
#ifndef MAIL_SERVICE_HPP
#define MAIL_SERVICE_HPP

/** \brief Mail service. This represents one of the collaborators of the SUT. 
 * \author David Stutz
 */
// 邮件服务
class MailService
{
public:
    /** \brief Send a mial.
     * \param[in] message message to send
     */
    virtual void send(std::string message) = 0;
    
};

#endif /* MAIL_SERVICE_HPP */
```



order.h文件：

```cpp
#ifndef ORDER_HPP
#define ORDER_HPP

#include <string>
#include <memory>
#include "warehouse.h"
#include "mail_service.h"

/** \brief An order of a product with quantity. */
// 订单
class Order
{
public:
    /** \brief Constructor.
     * \param[in] quantity quantity requested
     * \param[in] product product name requested
     */
    Order(int quantity, std::string product)
    {
        this->quantity = quantity;
        this->product = product;
    }
    
    /** \brief Set the mail service to use. 
     * \param[in] mailService the mail service to attach
     */
    // 设置邮件服务
    void setMailService(std::shared_ptr<MailService> mailService)
    {
        this->mailService = mailService;
    }
    
    /** \brief Fill the order given the warehouse. 
     * \param[in] warehouse the warehouse to use
     * \return whether the operation was successful
     */
     // 判断产品是否有库存，发送邮件通知
    bool fill(Warehouse &warehouse)
    {
        if (warehouse.hasInventory(quantity, product))
        {
            // ...
            warehouse.remove(quantity, product);
            this->mailService->send("Order filled.");
            
            return true;
        }
        else
        {
            // ...
            this->mailService->send("Order not filled.");
            
            return false;
        }
    }
    
private:
    
    /** \brief Product name. */
    std::string product;
    
    /** \brief Quantity requested. */
    int quantity;
    
    /** \brief Mail service to use. */
    std::shared_ptr<MailService> mailService;
};

#endif /* ORDER_HPP */
```



warehouse.h文件：

```cpp
#ifndef WAREHOUSE_HPP
#define WAREHOUSE_HPP

#include <string>

/** \brief Warehouse interface. This interface is one of the collaborators of our SUT.
 * \author David Stutz
 */
class Warehouse
{
public:
    /** \brief Check whether the product in the given quantity is on stock.
     * \param[in] quantity quantity requested
     * \param[in] product product name
     * \return whether the warehouse has the product on stock for the given quantity
     */
     // 是否有库存
    virtual bool hasInventory(int quantity, std::string product) const = 0;
    
    /** \brief Remove the given quantity of the product from the warehouse.
     * \param[in] quantity quantity to remove
     * \param[in] product product name to remove
     */
     // 从库存中删除
    virtual void remove(int quantity, std::string product) = 0;
    
};

#endif /* WAREHOUSE_HPP */
```



主要场景就是处理产品订单，其中库存Warehouse类和邮件服务MailService类，我们只声明一下虚基类，不实现，然后通过模拟对象的方式`mock`一下Warehouse和MailService，来达到订单类接口测试的正常开展，具体测试代码：

```cpp
#include <gmock/gmock.h>
#include "lib/mail_service.h"
#include "lib/order.h"
#include "lib/warehouse.h"

using ::testing::Return;
using ::testing::_; // Matcher for parameters

class MockWarehouse : public Warehouse
{
public:
    
    // see https://github.com/google/googletest/blob/master/googlemock/docs/ForDummies.md
    MOCK_CONST_METHOD2(hasInventory, bool(int, std::string));
    MOCK_METHOD2(remove, void(int, std::string));
};

class MockMailService : public MailService
{
public:
    MockMailService()
    {
        
    }
    
    MOCK_METHOD1(send, void(std::string));
};

TEST(OrderTest, Fill)
{
    MockWarehouse warehouse;
    std::shared_ptr<MockMailService> mailService = std::make_shared<MockMailService>();
    
    Order order(50, "Talisker");
    order.setMailService(mailService);
    
    EXPECT_CALL(warehouse, hasInventory(50, "Talisker"))
        .Times(1)
        .WillOnce(Return(true));

    EXPECT_CALL(warehouse, remove(50, "Talisker"))
        .Times(1);
    
    EXPECT_CALL(*mailService, send(_)) // Not making assumptions on the message send ...
        .Times(1);
    
    ASSERT_TRUE(order.fill(warehouse));
}


int main(int argc, char **argv)
{
  testing::InitGoogleMock(&argc, argv);

  // Runs all tests using Google Test.
  return RUN_ALL_TESTS();
}
```



测试结果：

```bash
bash-4.2$ ./output/bin/order.exx 
[==========] Running 1 test from 1 test case.
[----------] Global test environment set-up.
[----------] 1 test from OrderTest
[ RUN      ] OrderTest.Fill
[       OK ] OrderTest.Fill (0 ms)
[----------] 1 test from OrderTest (0 ms total)

[----------] Global test environment tear-down
[==========] 1 test from 1 test case ran. (1 ms total)
[  PASSED  ] 1 test.
```



其中main函数和gtest差不多，只是初始化的是googlemock，我们着重了解的是几个宏的含义:

**MOCK_METHOD**

```cpp
MOCK_METHOD#1(#2, #3(#4) )

MOCK_CONST_METHOD2(hasInventory, bool(int, std::string));
MOCK_METHOD2(remove, void(int, std::string));
```



**其中#1表示你要mock的方法共有几个参数，#2是你要mock的方法名称，#3表示这个方法的返回值类型，#4是这个方法具体的参数。**

**EXPECT_CALL**

```cpp
using ::testing::Return;
EXPECT_CALL(warehouse, hasInventory(50, "Talisker"))
        .Times(1)
        .WillOnce(Return(true));
```



**设定期望对象被访问的方式及其响应，其中warehouse为对象，希望hasInventory在传递参数为(50, “Talisker”)时，被调用且仅被调用一次，第一次返回true。**

**ON_CALL**

```cpp
ON_CALL(#1, #2(#3)).WillByDefault(Return(#4));

ON_CALL(foo, GetSize())                         
  .WillByDefault(Return(1));
  // ... other default actions ...
```



**其中#1表示mock对象，#2表示个方法名称，#3表示方法的参数，#4表示参数为#1, \#2，#3情况下返回结果。**

`ON_CALL`和`EXPECT_CALL`的区别? `ON_CALL`定义了调用mock方法时会发生什么，但并不意味着对被调用方法的任何期望。 `EXPECT_CALL`不仅定义了行为，还设置了对给定次数（以及在指定顺序时按给定顺序）使用给定参数调用方法的期望。

GoogleMock 为开发者设定 Mock 类行为，跟踪程序运行过程及结果，提供了丰富的支持。但与此同时，应用程序也应该尽量降低应用代码间的耦合度，使得单元测试可以很容易对被测试单元进行隔离。(尽量做到高内聚，低耦合)

## 总结

Googletest 与 GoogleMock，很好的简化了我们的C++单元测试工作，本篇文章对此做了一个总结，让自己对gtest有了一个系统的认识。**测试并不只是测试工程师的责任，对于开发工程师，为了保证发布给测试环节的代码具有足够好的质量（ Quality ），为所编写的功能代码编写适量的单元测试是十分必要的。**

**如果还想更加深入的了解，可查阅官方文档**：

> - https://github.com/google/googletest/tree/master/googlemock/docs

## 参考链接

> https://www.ibm.com/developerworks/cn/linux/l-cn-cppunittest/?mhq=gtest&mhsrc=ibmsearch_a
> https://blog.csdn.net/russell_tao/article/details/7344739
> http://www.cnblogs.com/coderzh/archive/2009/03/31/1426758.html
> https://github.com/davidstutz/googlemock-example