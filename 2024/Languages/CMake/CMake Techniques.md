#CMake

## 判断 CMake 编译环境

### 编译类型 `CMAKE_BUILD_TYPE`

可取值为：Debug, Release, RelWithDebInfo,  MinSizeRel 等预设值

```cmake
if (CMAKE_BUILD_TYPE MATCHES Debug)
    #do some thing
endif()
```

### 系统环境`CMAKE_SYSTEM_NAME`

代表当前系统的类型, 

值有ANDROID, APPLE, IOS, UNIX, WIN32, WINCE, WINDOWS_PHONE等

可以直接对这些值进行条件判断来确定

```cmake
if (UNIX)
    #cond1
elseif(WIN32)
    #cond2
endif()
```

### 编译工具环境

编译环境包括MSVC, MINGW, BORLAND, 等

可直接用if判断

msvc版本可通过MSVC10, MSVC11, MSVC12, MSVC14, MSVC60, MSVC70, MSVC71, MSVC80, MSVC90, MSVC_TOOLSET_VERSION, MSVC_VERSION等判断

## 编译器设置

### C/C++标准

```cmake
# 全局
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_C_STANDARD 11)
```

### 编译器flag

主要靠修改`CMAKE_CXX_FLAG_<BUILD_TYPE>`来进行修改, 也可以直接修改所有类型的, 或者通过判断编译类型来配置

```cmake
set(DEFINES "-Wunused-parameter")
set(CMAKE_CXX_FLAGS_DEBUG "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_C_FLAGS_DEBUG "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_EXE_LINKER_FLAGS_DEBUG "-fno-pie")

set(CMAKE_CXX_FLAGS_RELEASE "-pipe -O2 -std=gnu++11 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_C_FLAGS_RELEASE "-pipe -O2 -Wall -W -D_REENTRANT -fPIC ${DEFINES}")
set(CMAKE_EXE_LINKER_FLAGS_RELEASE "-fno-pie")
```

### 特殊的编译器设置

- sanitize (msvc目前只支持32位)

检查内存泄露和内存其他问题

```cmake
list(APPEND CMAKE_CXX_FLAGS_DEBUG -fsanitize=address -g)
```

配置后运行程序时发生内存问题程序会立马中断退出, 并且打印内存问题

- /utf-8 (msvc)

强制源码使用utf-8, 避免代码到了其他平台乱码

```cmake
list(APPEND CMAKE_CXX_FLAGS_DEBUG /utf-8)
```

- -DUNICODE (msvc)

决定windows下面使用标准字符api还是宽字符api

```cpp
#ifdef UNICODE
#define ShellExecute  ShellExecuteW
#else
#define ShellExecute  ShellExecuteA
#endif // !UNICODE
```

- /SUBSYSTEM:WINDOWS

更改软件启动入口, 默认是/SUBSYSTEM:CONSOLE, 会先启动命令行再启动其他ui, 而更改后命令行将不再启动, 所有打印信息也将会看不到, 只有通过x64dbg这种调试软件才能看到

- /OPT:REF /OPT:ICF

添加这两个参数release模式下编译也会生成pdb文件

## 根据模板生成文件

### 生成动态库的export头文件

一般来说Windows平台生成动态库的头文件要区分`__declspec(dllexport)`和`__declspec(dllimport)`，会专门用一个头文件进行定义，同时区分动态链接库和静态链接库，如果项目较复杂可以用cmake生成到build目录进行包含，利用``函数，下面是其的定义

```cmake
GENERATE_EXPORT_HEADER( LIBRARY_TARGET
          [BASE_NAME <base_name>]
          [EXPORT_MACRO_NAME <export_macro_name>]
          [EXPORT_FILE_NAME <export_file_name>]
          [DEPRECATED_MACRO_NAME <deprecated_macro_name>]
          [NO_EXPORT_MACRO_NAME <no_export_macro_name>]
          [INCLUDE_GUARD_NAME <include_guard_name>]
          [STATIC_DEFINE <static_define>]
          [NO_DEPRECATED_MACRO_NAME <no_deprecated_macro_name>]
          [DEFINE_NO_DEPRECATED]
          [PREFIX_NAME <prefix_name>]
          [CUSTOM_CONTENT_FROM_VARIABLE <variable>]
)

#例如

generate_export_header(${PROJECT_NAME}
            BASE_NAME ${MODULE_NAME}
            EXPORT_MACRO_NAME PROJECT_${MODULE_NAME_UPPER}_API
            EXPORT_FILE_NAME ${MODULE_NAME}_Export.hpp
            STATIC_DEFINE ${MODULE_NAME_UPPER}_BUILT_AS_STATIC
            )
```

在需要导出的类或函数加上`EXPORT_MACRO_NAME`就可以导出动态库了

### 生成程序版本信息

程序版本信息有许多，包括版本号、git分支、git提交号、sdk版本、构建日期等等，Windows下还有文件版本信息（属性里面的内容）

#### 版本信息模板

编写模板，和生成代码

```cpp
//Global.hpp.in
#include "PROJECT_global.hpp"
#include "BuildTime.hpp"

namespace PROJECT {

#define PROJECT_APP_NAME "@PROJECT_APP_NAME@"
#define PROJECT_APP_ORG "@PROJECT_APP_ORG@"
#define PROJECT_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define PROJECT_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define PROJECT_VERSION_PATCH @PROJECT_VERSION_PATCH@
#define PROJECT_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
#define PROJECT_VERSION_BETA @PROJECT_VERSION_BETA@
#define PROJECT_VERSION "@PROJECT_VERSION_MAJOR@.@PROJECT_VERSION_MINOR@.@PROJECT_VERSION_PATCH@.@PROJECT_VERSION_TWEAK@"
#define PROJECT_VERSION_STATUS "@PROJECT_VERSION_STATUS@"
#define PROJECT_VERSION_COMMIT "@PROJECT_VERSION_COMMIT@"
#define PROJECT_VERSION_BRANCH "@PROJECT_VERSION_BRANCH@"
#define PROJECT_VERSION_DATE BUILD_TIME
#define PROJECT_QT_VERSION "@PROJECT_QT_VERSION@"
#define PROJECT_QT_PLATFORM "@PROJECT_QT_PLATFORM@"
}
```

```cmake
set(PROJECT_APP_NAME "Project")
set(PROJECT_APP_ORG "Company")
set(PROJECT_VERSION ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}.${PROJECT_VERSION_PATCH}.${PROJECT_VERSION_TWEAK})
set(PROJECT_VERSION_BETA 1)
set(PROJECT_VERSION_STATUS ${GIT_STATUS})
set(PROJECT_VERSION_COMMIT ${GIT_COMMIT})
set(PROJECT_VERSION_BRANCH ${GIT_BRANCH})
set(PROJECT_VERSION_DATE ${GIT_DATE})
set(PROJECT_QT_VERSION ${QT_VERSION})
set(PROJECT_QT_PLATFORM ${QT_PLATFORM})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/Global.hpp.in ${CMAKE_CURRENT_BINARY_DIR}/gen/Global.hpp @ONLY)
```

#### 从文本文件获取版本号

```cmake
file(READ ${CMAKE_SOURCE_DIR}/VERSION VERSION_STR)
string(REPLACE "." ";" VERSION_STR_LIST ${VERSION_STR})
list(LENGTH VERSION_STR_LIST LEN)
if (NOT LEN EQUAL 4)
    message(FATAL_ERROR "version number not in correct format")
endif ()
list(GET VERSION_STR_LIST 0 PROJECT_VERSION_MAJOR)
list(GET VERSION_STR_LIST 1 PROJECT_VERSION_MINOR)
list(GET VERSION_STR_LIST 2 PROJECT_VERSION_PATCH)
list(GET VERSION_STR_LIST 3 PROJECT_VERSION_TWEAK)
message(VERSION:${PROJECT_VERSION_MAJOR})
```

利用cmake读写文件可以实现buildVersion累加

#### 获取git信息

```cmake
include(FindGit)
if (${GIT_FOUND})
    unset(GIT_VERSION_NUM CACHE)
    unset(GIT_BRANCH CACHE)
    message("Found Git executable, version: ${GIT_VERSION_STRING}")
    execute_process(
            COMMAND ${GIT_EXECUTABLE} describe --tags --exact-match --abbrev=0
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_TAG
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE GIT_VERSION_RESULT
    )
    if (NOT ${GIT_VERSION_RESULT} EQUAL 0)
        set(GIT_STATUS "u")
        set(GIT_TAG "notag")
    else ()
        set(GIT_VERSION_STATUS "")
    endif ()

    execute_process(
            COMMAND ${GIT_EXECUTABLE} log -1 --format=%H
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_COMMIT
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE GIT_COMMIT_RESULT
    )
    if (NOT GIT_COMMIT_RESULT EQUAL 0)
        message(FATAL_ERROR "cannot find git commit number")
    else ()
        message("git commit: ${GIT_COMMIT}")
    endif ()

    execute_process(
            COMMAND ${GIT_EXECUTABLE} rev-parse --abbrev-ref HEAD
            WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}
            OUTPUT_VARIABLE GIT_BRANCH
            OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE GIT_BRANCH_RESULT
    )
    if (NOT GIT_BRANCH_RESULT EQUAL 0)
        message(WARNING "cannot find git branch name")
    else ()
        message("git branch: ${GIT_BRANCH}")
    endif ()

    set(GIT_STATUS ${GIT_STATUS} CACHE INTERNAL "git version status")
    set(GIT_COMMIT ${GIT_COMMIT} CACHE INTERNAL "git commit number")
    set(GIT_BRANCH ${GIT_BRANCH} CACHE INTERNAL "git branch name")
else ()
    MESSAGE(WARNING "cannot find Git executable, skip git version check")
    set(GIT_COMMIT "unknown")
    set(GIT_BRANCH "unknown")
endif ()
```

#### 构建日期

构建日期用cmake生成问题在于编译的时候不一定会重新跑cmake，所以需要用到cmake执行cmake脚本的命令

更新时间cmake：

```cmake
message("start updating build time")
if (NOT DEFINED BUILD_TIME_HEADER)
    message(FATAL_ERROR "Need to pass `-DBUILD_TIME_HEADER:PATH=" /path/to/header"` to this script")
endif ()
unset(DATE CACHE)
string(TIMESTAMP DATE "%b %d %Y %a %H:%M:%S")
set(DATE "${DATE}")
message("compile date: ${DATE}")
file(WRITE ${BUILD_TIME_HEADER}
        "//this file is automatically generated by UpdateBuildTime.cmake\n"
        "#ifndef BUILD_TIME_HEADER\n"
        "#define BUILD_TIME_HEADER\n"
        "\n"
        "#define BUILD_TIME \"${DATE}\"\n"
        "\n"
        "#endif //BUILD_TIME_HEADER\n"
        )
message("Write build date to file ${BUILD_TIME_HEADER}")
```

在构建时增加`BuildTime`目标依赖

```cmake
add_custom_target(BuildTime
        COMMAND ${CMAKE_COMMAND}
        -DBUILD_TIME_HEADER:PATH="${BUILD_TIME_HEADER}"
        -P "${CMAKE_SOURCE_DIR}/cmake/UpdateBuildTime.cmake")
if (CMAKE_BUILD_TYPE MATCHES Release)
    add_dependencies(${PROJECT_NAME} BuildTime)
endif ()
```

增加条件的原因是防止调试编译时过于频繁的更新编译时间

### 生成Windows下文件信息

将信息编译到exe或者dll中， 同样利用`confugure_file`

```cpp
#if defined(UNDER_CE)
#include <winbase.h>
#else
#include <winver.h>
#endif
VS_VERSION_INFO VERSIONINFO
        FILEVERSION @RC_FILE_VERSION_NUM@
        PRODUCTVERSION @RC_PRODUCT_VERSION_NUM@
        FILEFLAGSMASK 0x3fL
#ifdef _DEBUG
FILEFLAGS 0x1L
#else
FILEFLAGS 0x0L
#endif
FILEOS VOS__WINDOWS32
FILETYPE VFT_DLL
FILESUBTYPE 0x0L
BEGIN
        BLOCK "StringFileInfo"
BEGIN
        BLOCK "040904b0"
BEGIN
        VALUE "Comments", @RC_COMMENTS@
        VALUE "CompanyName", @RC_COMPANY_NAME@
        VALUE "FileDescription", @RC_FILE_DESCRIPTION@
        VALUE "FileVersion", @RC_FILE_VERSION@
        VALUE "InternalName", @RC_INTERNAL_NAME@
        VALUE "LegalCopyright", @RC_LEGAL_COPYRIGHT@
        VALUE "OriginalFilename", @RC_ORIGINAL_FILE_NAME@
        VALUE "ProductName", @RC_PRODUCT_NAME@
        VALUE "ProductVersion", @RC_PRODUCT_VERSION@
        VALUE "PrivateBuild", @RC_PRIVATE_BUILD@
        VALUE "LegalTrademarks", @RC_LEGAL_TRADEMARKS@
        END
END
        BLOCK "VarFileInfo"
BEGIN
        VALUE "Translation", 0x804, 1300
END
        END
```

编译时和源代码一起编译就可以

## 导入第三方库

```cmake
function(set_library_target NAMESPACE LIB_NAME DEBUG_LIB_FILE_NAME RELEASE_LIB_FILE_NAME INCLUDE_DIR)
    add_library(${NAMESPACE}::${LIB_NAME} STATIC IMPORTED)
    set_target_properties(${NAMESPACE}::${LIB_NAME} PROPERTIES
            IMPORTED_CONFIGURATIONS "RELEASE;DEBUG"
            IMPORTED_LOCATION_RELEASE "${RELEASE_LIB_FILE_NAME}"
            IMPORTED_LOCATION_DEBUG "${DEBUG_LIB_FILE_NAME}"
            INTERFACE_INCLUDE_DIRECTORIES "${INCLUDE_DIR}"
            )
    set(${NAMESPACE}_${LIB_NAME}_FOUND 1)
endfunction()
```