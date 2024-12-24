原文：

> [https://www.qt.io/blog/introduction-to-the-qml-cmake-api](https://www.qt.io/blog/introduction-to-the-qml-cmake-api)

当 Qt 6 迁移到 CMake 时，我们希望为 QML 项目提供更好的体验。在最初的Qt 6.0发布版中，我们只提供了一些技术预览API，这并没有比 qmake 自 Qt 5.15 以来提供的 API 做得更多。

现在，随着即将发布的 Qt 6.2，我们已经完成了 CMake API。在这篇博客文章中，我们将说明为什么需要为 QML 添加构建系统支持，并展示如何实现两个常用用例。

## 为什么我们需要CMake的支持?

在讨论创建 QML 应用程序和库的新方法之前，我们首先应该了解到目前为止它们是如何完成的:

- 只有自定义 c++ 被放到实际的QML名称空间中;无论是通过 5.15 以来的声明式类型注册，还是仍然使用手动 qml*Register 调用。
- QML 文件通常简单地放在单个文件夹中。如果它们相互引用，人们通常依赖于隐式导入( QML 文档可以在其他 QML 文档中使用类型，只要它们在同一个文件夹中)。大多数人都忽略了 qmldir 文件，除非他们有很强的理由使用它们(很可能是因为他们需要一个 QML 单例)。然后，应用程序只需通过 QQmlApplicationEngine 或 QQuickWindow 加载主 QML 文件。
- 图像、着色器或字体等附加资源通常被放入资源系统中。然后，QML 文件将通过qrc 路径引用它们。因此，您不能再使用 QML(场景)中的 QML 文件或 QtCreator 中的 QML 设计器。
- 要使用 qmlcachegen，需要创建一个包含所有 QML 文件的 QRC 文件，然后通过指定 `QT += qtquickcompiler.prf` 启用 qmlcachegen 这只会在您从资源文件系统加载相应文件时生效，但是，从物理文件系统加载文件时，它们仍将在运行时编译。

上面已经列出了一些由迄今为止使用的相当特别的方法引起的问题。 通常，我们的工具很难对 QML 项目进行推理。 输入 CMake 和 qt6_add_qml_module。 有了这个新引入的函数，构建系统就知道 QML 项目的所有部分，并且可以根据需要将其传递给我们的工具。 让我们看几个例子！

## 一个基本的 QML 应用程序……
我们从一个小小的“Hello, World!”开始 例子。 我们在同一目录中有一个 QML 文件 main.qml

```qml
import QtQuick  
  
Window {  
	id: root  
	visible: true  
	Text {  
		text: "Hello, world!"  
		anchors.centerIn: parent  
	}  
}
```

和一个实例化它的 main.cpp 文件：

```cpp
#include <QGuiApplication>  
#include <QQmlApplicationEngine>  
  
int main(int argc, char *argv[])  
{  
	QGuiApplication app(argc, argv);  
	  
	QQmlApplicationEngine engine;  
	const QUrl url(u"qrc:/hello/main.qml"_qs);  
	engine.load(url);  
	  
	return app.exec();  
}
```


除了 URL 中的“hello”外，大部分都很简单。我们一会儿再回到这个问题上。但首先，让我们看看构建项目所需的CMakeLists.txt文件:

首先，我们做一些通用的设置。

```cmake
cmake_minimum_required(VERSION 3.18)  
  
project(hello VERSION 1.0 LANGUAGES CXX)  
  
set(CMAKE_AUTOMOC ON)  
set(CMAKE_CXX_STANDARD 17)  
set(CMAKE_CXX_STANDARD_REQUIRED ON)  
  
find_package(Qt6 6.2 COMPONENTS Quick Gui REQUIRED)  
  
qt_add_executable(
	myapp  
	main.cpp  
)  
  
target_link_libraries(myapp PRIVATE Qt6::Gui Qt6::Quick)
```

我们添加了 Quick 模块，并告诉 CMake 为我们的可执行文件创建一个目标，接下来会发生有趣的部分：

```cmake
qt_add_qml_module(
	myapp  
	URI hello  
	VERSION 1.0  
	QML_FILES main.qml
	)
```

在这里，我们终于看到了 `qt_add_qml_module` 的作用。 我们将可执行文件的目标、URI、模块版本和 QML 文件列表传递给它。 反过来，它确保 myapp 成为 QML 模块。 除此之外，这会将 QML 文件放入资源文件系统中的 qrc:/${URI} 中。 这现在可能看起来很奇怪，实际上在这篇博文的结尾看起来仍然很奇怪：您必须等待即将发布的关于 QML 模块的博文，我们将详细解释为什么需要这样做。 在这一点上，您可能会想知道这种奇怪是否真的值得。 毕竟在 Qt 5 中使用 cmake 已经可以将文件放入资源系统，而无需太多仪式！

请放心，我们已经获得了两件事：第一，`qt_add_qml_module` 确保 qmlcachegen 运行。 其次，它创建一个 myapp_qmllint 目标，该目标对 QML_FILES 中的文件运行 qmlint。 阅读我们[关于工具的博客](https://www.qt.io/blog/whats-new-in-qml-tooling-in-qt-6.2?hsLang=en)文章，了解为什么要运行 qmllint。

## 变得更加先进

让我们扩展这个示例，使其更有趣。我们添加了一个新的 QML 文件，FramedImage.qml 和 img 子目录中的图像。

```qml
import QtQuick  
  
Rectangle {  
	border.width: 2  
	border.color: "black"  
	Image {  
		source: Qt.resolvedUrl("img/world.png")  
		anchors.centerIn: parent  
		sourceSize.width: parent.width  
		sourceSize.height: parent.height  
	}  
}
```

在 main.qml 中使用:

```qml
import QtQuick  
  
Window {  
	id: root  
	visible: true  
	Text {  
		text: "Hello, world!"  
		anchors.centerIn: parent  
		color: "white"  
		z: 2  
	}  
	FramedImage { anchors.fill: parent }  
}
```

更新我们的 CMakeList.txt:

```cmake
qt_add_qml_module(
	myapp  
	URI hello  
	VERSION 1.0  
	QML_FILES  
	main.qml  
	FramedImage.qml  
	RESOURCES  
	img/world.png  
)
```

您可能已经预料到 QML_FILES 现在也会列出 FramedImage.qml。 更有趣的变化是 RESOURCES 条目。 通过在那里添加引用的资源，它们会自动添加到与 QML 文件相同的根路径下的应用程序 - 也在资源文件系统中。 无需重复 qt_add_resources 中的路径。 并且通过保持资源系统中的路径与 source 和 build 目录中的路径一致，我们确保始终找到图像： 由于它是相对于 FramedImage.qml 解析的，因此它指的是资源文件系统中的图像 如果我们从那里加载 main，或者如果我们使用 qml 工具检查它，则加载到实际文件系统中的那个。

## 创建一个 qml 库

现在，让我们开始第二个例子，我们创建了一个向 QML 公开 C++ 类型的库。 我们这个例子的目录结构看起来像：

```text
├── CMakeLists.txt  
	└── example  
		└── mylib  
			├── CMakeLists.txt  
			├── mytype.cpp  
			└── mytype.h
```

mytype.h 声明一个类并使用 Qt 5.15 中引入的声明性注册宏将其公开给引擎：

```cpp
#include <QObject>  
#include <QtQml/qqmlregistration.h>  
  
class MyType : public QObject  
{  
	Q_OBJECT  
	QML_ELEMENT  
	Q_PROPERTY(int answer READ answer CONSTANT)  
public:  
	int answer() const;  
};
```

顶层 CMakeLists.txt 文件进行一些基本设置，然后使用 add_subdirectory 将其包含在 mylib 中。 我们使用与 QML 模块的 URI 相对应的子目录结构，但将点替换为斜杠 - 这与引擎在导入路径中搜索模块时使用的逻辑相同。 通过遵循该约定，我们有助于工具。

```cmake
qt6_add_qml_module(
	mylib  
	URI example.mylib  
	VERSION 1.0  
	SOURCES  
	mytype.h mytype.cpp  
)
```

首先，这次我们要添加 C++ 类型。 为此，我们使用 SOURCES 参数指定它们。 此外，这次我们还没有为 mylib 创建任何目标。 如果 qt6_add_qml_module 的 target 参数不存在，它会自动创建一个库目标，这就是我们在这种情况下想要的。

如果您构建项目，您会注意到我们不仅构建了库，还构建了一个 QML 插件——即使我们没有编写自定义 QQmlEngineExtensionPlugin。 那么插件有什么作用呢？ 就其本身而言，不是很多。 mylib 库本身已经包含向引擎注册类型的代码。 但是，这仅在我们可以链接库的情况下才有用。 为了使模块在 qml 工具加载的 QML 文件中可用，我们需要一些可以加载的插件。 然后插件负责实际链接库，并确保类型得到注册。 应该注意的是，自动插件生成只有在模块除了注册类型之外不做任何事情时才可能。 如果它需要做一些更高级的事情，比如在 initializeEngine 中注册一个图像提供者，你仍然需要手动编写插件。 qt6_add_qml_module 通过 NO_GENERATE_PLUGIN_SOURCE 对此提供支持。 我们将在稍后的博客文章中更详细地介绍这一点。

另外，您还记得我们提到遵循目录布局约定有助于工具吗？ 该布局反映在我们的构建目录中，插件就位。 这意味着您可以将构建目录的路径传递给 qml 工具（通过 -I 标志），它会找到插件。 反过来，这可以帮助您测试纯 C++ 库模块上的更改，因为您不需要执行任何进一步的安装步骤。

在结束之前，让我们向模块添加一个 QML 文件：在 lib 子文件夹中，我们添加一个 Mistake.qml 文件：

```qml
import example.mylib  
  
MyType{  
	answer: 43  
}
```

并调整 qt6_add_qml_module 调用：

```cmake
qt6_add_qml_module(
	mylib  
	URI example.mylib  
	VERSION 1.0  
	SOURCES  
	mytype.h mytype.cpp  
	QML_FILES  
	Mistake.qml  
)
```

顾名思义（Mistake），我们在这里犯了一个错误： answer 实际上是一个只读属性。 我们为什么要这样做？ 当然，为了炫耀承诺的 qmllint 集成：CMake 将为我们创建一个 qmllint 目标，一旦我们运行它，qmllint 就会警告我们这个问题：

(只有当您使用了dev分支最近的qmllint时)

```shell
$> cmake --build . --target mylib_qmllint  
...  
Warning: Mistake.qml:4:13: Cannot assign to read-only property answer  
	answer: 43
```

## 展望

这是关于新 CMake API 的系列文章中的第一篇博文。 我们介绍了 qt6_add_qml_module，并展示了它如何处理一些基本用例。 我们已经看到 CMake 通过处理重复性任务来帮助你，比如手动编写 QML 插件，以及它如何帮助我们的工具故事。

在本博客系列的下一部分中，我们将深入研究 QML 模块，并查看一些更高级的用例：包含多个 QML 库的应用程序和安装 QML 模块库。

如果您想了解有关它提供的所有选项的更多信息，还可以查看 [qt6_add_qml_module 的参考文档](https://doc-snapshots.qt.io/qt6-dev/qt-add-qml-module.html)。