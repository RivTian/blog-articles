#CLion 

* [Clion+Qt+msvc的配置使用，调试器显示Qt特有类型变量的值(如Qstring等)](http://t.csdnimg.cn/v0LZU)
* [Clion配置Qt+MSVC/MinGW环境](http://t.csdnimg.cn/iQJRH)
* [推荐的 Qt CMake 项目结构模板讲解](https://www.helywin.com/posts/20181216114140/)

> Build with CLion 2023.3 EAP (Nova) & Qt 5.15.2 & MSVC 2019 v142

![](https://miro.medium.com/v2/resize:fit:720/format:webp/1*zTVjtXBz5Hh7BMO1J9PmMA.png)

# 安裝

## 安裝 CLion

CLion 可以免費使用 30 天，如果學校有提供 E-mail 的話，可以到 JetBrains 官網啟用教育版授權，便可以免費使用一年（如果 E-mail 沒有被註銷，授權到期後可以重複啟用）。

## 安裝 MSVC 2019 v142

安裝 Visual Studio 的時候，選擇安裝*『使用 c + + 的桌面開發』*，其中就包含了 MSVC 元件，亦可自己到『個別元件**』**分頁單獨選擇要安裝的 MSVC 元件。

* [Windows Tools | How To Install VS Microsoft C++ Build Tools on Windows](https://www.cnblogs.com/RioTian/p/17730245.html)

## 安裝 Qt

安裝 Qt 時，需要將 *MSVC 2019 64-bit* 選項打勾，安裝 MSVC 需要的元件。

# 設定編譯環境

## 設定編譯器

安裝完所有程式後，開啟 CLion，到 *Settings > Build, Execution, Deployment > Toolchains* 設定編譯工具。

![設定位於 Visual Studio 裡的 MSVC 編譯器的位置<br/>VC142 with Clang](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231114064406_image-20231114142702264.png)

* [Windows 关于 MSVC 搭配 Clang 的方法](https://www.cnblogs.com/RioTian/p/17758813.html)



## 選擇編譯版本

預設專案已經包含編譯 Debug 版本的設定，如果要編譯 Release 版本，可至 *Settings > Build, Execution, Deployment > CMake* 新增 Release 版本。

![在 Profiles 裡面按一下加號，就會自動產生 Release Profile](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231114064538_image-20231114142920934.png)

接著只要在 CLion 視窗右上方切換 Profile 就可以編譯 Release 版本了。

![選擇 Release Profile 並編譯專案](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231114064505_image-20231114143013859.png)



# 建立 Qt Widgets Executable 專案

建立專案時，選擇 *Qt Widgets Executable*，並且將 *Qt CMake prefix path* 填入位於 Qt 資料夾下的 MSVC 資料夾。

![建立 Qt Widgets Executable 專案](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231114064618_image-20231114143050862.png)

專案建立完成後，可以先執行看看範例程式是否能成功執行。

![預設範例程式，內含一個按鈕](https://miro.medium.com/v2/resize:fit:582/format:webp/1*kjM_ASHZt2JpHCWM_pU2AQ.png)

# 微調 CMakeLists.txt

## 取消顯示命令提示字元

用預設的 CMakeLists.txt 編譯程式後所產生的執行檔，透過手動點兩下執行時，除了顯示 Qt UI 的界面以外，還會另外跳出一個命令提示字元視窗，這是因為編譯出來的程式沒有指定 WIN32 的緣故，因此需手動修改。

在 CMakeLists.txt 裡找到 *add_executable* 命令並加入 *WIN32*：

```cmake
add_executable(${PROJECT_NAME} WIN32 main.cpp mainwindow.cpp mainwindow.h mainwindow.ui)
```

## Qt UI 顯示樣式老舊

當程式執行時，界面是傳統 Windows 樣式的話，表示缺少 *qwindowsvistastyle.dll*，該檔案可以在 *.\Qt\5.15.2\msvc2019_64\plugins\styles* 底下找到，可自行複製到與執行檔同目錄下的 *plugins/styles* 底下，或修改 CMakeLists.txt，使程式編譯完成後，自動複製到該目錄下。

在 CMakeLists.txt 加入以下程式：

> *判斷 qwindowsvistastyle.dll 是否存在，並將其複製至指定位置。*

```cmake
if (EXISTS  "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll")
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E make_directory
            "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E copy
            "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll"
            "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
endif ()
```

![套用 Windows Vista Style 後，滑鼠遊標停留在按鈕上時，應該要有視覺回饋。](https://miro.medium.com/v2/resize:fit:574/format:webp/1*cQVse-3BROFG7u1hyVYyWQ.png)



## 最終的 CMakeLists.txt

> *add_executable: Line 14*
> *複製 qwindowsvistastyle.dll: Line 52 ~ 60*
>
> CMakeLists.txt from [Weikeup ](https://gitlab.com/weikeup)

* [GitLab CMakeLists.txt for Qt UI](https://gitlab.com/-/snippets/2111766?source=post_page-----599784958c0a--------------------------------)

* [个人在搭建项目使用的 CMakeLists.txt 项目](https://files.cnblogs.com/files/RioTian/V3ServerShow.zip?t=1699944180&download=true)


```cmake
cmake_minimum_required(VERSION 3.5)

project(V3ServerShow VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# set(CMAKE_PREFIX_PATH "D:/Soft/Language/QT/5.15.2/msvc2019_64") # Do not use before importing the package
set(CMAKE_TOOLCHAIN_FILE "D:/Soft/Language/vcpkg/scripts/buildsystems/vcpkg.cmake")

install(
        FILES "${CMAKE_CURRENT_BINARY_DIR}/cpprestsdk-config.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/${CPPREST_EXPORT_DIR}
)

find_package(QT NAMES Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)
find_package(cpprestsdk REQUIRED)
# find_package(CURL REQUIRED) # Test import 3rd_party

set(PROJECT_SOURCES
        main.cpp
        view/render.cpp
        view/render.h
        view/render.ui
        global/def.h
        global/type.h
        http/types.hpp
)

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(V3ServerShow
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET V3ServerShow APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(V3ServerShow SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(V3ServerShow
            ${PROJECT_SOURCES}
            logo.qrc
        )
    endif()
endif()

target_link_libraries(V3ServerShow PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)
target_link_libraries(V3ServerShow PRIVATE cpprestsdk::cpprest cpprestsdk::cpprestsdk_zlib_internal cpprestsdk::cpprestsdk_brotli_internal)
# target_link_libraries(V3ServerShow PRIVATE CURL::libcurl) # Test import 3rd_party



# Qt for iOS sets MACOSX_BUNDLE_GUI_IDENTIFIER automatically since Qt 6.1.
# If you are developing for iOS or macOS you should consider setting an
# explicit, fixed bundle identifier manually though.
if(${QT_VERSION} VERSION_LESS 6.1.0)
  set(BUNDLE_ID_OPTION MACOSX_BUNDLE_GUI_IDENTIFIER com.example.V3ServerShow)
endif()
set_target_properties(V3ServerShow PROPERTIES
    ${BUNDLE_ID_OPTION}
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

include(GNUInstallDirs)
install(TARGETS V3ServerShow
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(V3ServerShow)
endif()

if (WIN32)
    set(DEBUG_SUFFIX)
    if (CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB ${REQUIRED_LIBS})
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt${QT_VERSION}${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)

    if (EXISTS "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/styles/qwindowsvistastyle${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/styles/")
    endif ()
endif ()
```

# 建立 & 編輯 UI

## 建立 UI 檔案

在專案上面點右鍵，並依序選擇 *New > Qt UI Class*，輸入名稱後，需將 *Parent class* 設定為 *QMainWindow*。

![Parent class 需設為 QMainWindow](https://miro.medium.com/v2/resize:fit:720/format:webp/1*-vf8lR3Plm0pBJ0KUpEVgQ.png)

建立 UI 檔後，先在名為 *MainWindow* 的 Widget 裡面新增一個空的 Widget，並設定 *Class* 為 *QWidget*，這樣才能在 Qt Designer 裡面新增其他元件。

```ui
<widget class="QMainWindow" name="MainWindow">
    <!-- 略 -->
    <widget class="QWidget"></widget>
</widget>
```

## 編輯 UI

使用 Qt Designer 開啟 UI 檔，即可編輯界面，接著存檔後，重新編譯程式就會產生新的 UI 標頭檔。

> Qt Designer 位於 C:\Qt\5.15.2\msvc2019_64\bin\designer.exe

透過設定 External Tools 可以用更方便的方式開啟 Qt Designer：

首先至 *Settings > External Tools*，點選加號新增外部工具，在 *Program* 填入 Qt Designer 的執行檔位置， *Arguments* 填入 *$FileName$*，*Working directory* 填入 *$FileDir$*，並按下 OK 即可。

![新增 External Tools](https://miro.medium.com/v2/resize:fit:720/format:webp/1*kvsgoGK6nlh7mJD0t1QKEA.png)



接著只要對 UI 檔按右鍵，並選擇 *External Tools > Qt Designer*，就可以用 Qt Designer 開啟該檔案。

![External Tools > Qt Designer](https://miro.medium.com/v2/resize:fit:700/1*4LT6RrRPOLs8Z9aLii7mJA.png)



## 顯示 UI

在編輯完 UI 後，將程式重新編譯，使其產生 UI 的標頭檔 (*.h)後，接著修改 *main* 方法，讓界面顯示出來：

```cpp
int main(int argc, char *argv[]) {
    QApplication a(argc, argv);
    MainWindow m;
    m.show();
    return QApplication::exec();
}
```

## 啟用 UI 縮放

如果系統顯示有設定縮放的話，可能會造成 Qt Designer 拉出來的界面跟實際執行的界面比例不同，這時候在環境變數新增 *QT_AUTO_SCREEN_SCALE_FACTOR = 1* 就能解決。

在 *main* 方法的第一行新增以下程式即可：

```cpp
qputenv("QT_AUTO_SCREEN_SCALE_FACTOR", "1");
```

# Include OpenCV

Install OpenCV & Add the following code in CMakeLists.txt

```cmake
set(OPENCV_INCLUDE_DIRS "C:/opencv/build/include")
set(OPENCV_LIB "C:/opencv/build/x64/vc15/lib/opencv_world3410${DEBUG_SUFFIX}.lib")

include_directories(${OPENCV_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${OPENCV_LIB})
```

完