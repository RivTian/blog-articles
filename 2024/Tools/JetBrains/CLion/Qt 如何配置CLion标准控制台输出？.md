
#CLion

CMake 相关问题：

即CMakeLists.txt文件中，在`add_executable`添加了WIN32。即当使用了WIN32标识后，就去掉了控制台，那么自然就没有信息打印出来了。

```cmake
# for example
add_executable(${PROJECT_NAME} WIN32
    ${_SRC_FILES}
    ${_PLATFORM_SRC_FILES}
    ${_UI_FILES}
    ${_RES_FILES}
    ${_WEB_FILES}
)

set_target_properties(V3ServerShow PROPERTIES
    ${BUNDLE_ID_OPTION}
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    # WIN32_EXECUTABLE TRUE # 避免存在 win32
)
```

另外某些人可能有用的写法

需要在运行环境参数中加入如下语句：

```bash
QT_ASSUME_STDERR_HAS_CONSOLE=1
```

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231115053453_%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20231115133440.png)

使用此方法，CLion在调试器启动时显示`QDebug`和QML的`console.*()`。