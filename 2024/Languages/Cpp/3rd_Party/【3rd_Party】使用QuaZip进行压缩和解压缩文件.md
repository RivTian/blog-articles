
> * [QuaZIP使用记录](https://blog.csdn.net/kenfan1647/article/details/116619267)
> * [官方文档](https://quazip.sourceforge.net/annotated.html)

#Cpp #3rd_Party 

## 一、QuaZIP是什么

QuaZIP is a simple C++ wrapper over Gilles Vollant’s ZIP/UNZIP package that can be used to access ZIP archives. It uses the Qt toolkit.



- 官方主页：http://quazip.sourceforge.net/
- github地址：https://github.com/stachenov/quazip
- souceforge下载地址：https://sourceforge.net/projects/quazip/

### 二、如何编译

#### 2.1、使用CMake编译

首先要知道的一件事：quazip是对zlib的封装，需要依赖zlib库的。直接使用CMake编译的话会遇到zlib库的路径问题：

##### 遇到问题：

CMake提示：Could NOT find ZLIB (missing: ZLIB_LIBRARY ZLIB_INCLUDE_DIR)

##### 解决方案：

注意到 *https://github.com/stachenov/quazip* 下载的压缩包里面，有个文件 **NEWS.txt**：

```markdown
QuaZip changes

* 2021-11-13 1.2
        * Add CMake option to disable installation (Sebastian Rettenberger)
        * Qt's internal zlib can be used now (QUAZIP_USE_QT_ZLIB, OFF by
          default)
        * Make tests optional (QUAZIP_ENABLE_TESTS, OFF by default)
          (Gleb Ignatev)
        * Fix: QuaZip::close() can be safely called multiple times now
        * Fix: -lz added to pkgconfig

```

其中 Qt’s internal zlib can be used now (QUAZIP_USE_QT_ZLIB, OFF bydefault)，意思是说，支持直接使用QT内置的zlib。

* A、修改 CMakeLists.txt 中的 option(QUAZIP_USE_QT_ZLIB “” **OFF**)，为option(QUAZIP_USE_QT_ZLIB “” **ON**)
* B、再使用CMake（带gui界面的）根据需要配置一下，配置完毕会生成 QuaZip.sln
* C、编译QuaZip.sln，结束后就可以得到 **quazip1-qt5.dll**

#### 2.2、手动新建quazip工程

- A、下载zlib，地址：http://www.zlib.net/
- B、编译zlib，直接cmake编译即可
- C、新建quazip的vs工程（qt），把quazip压缩包的文件都搬到新建的工程，然后把zlib以第三方库的形式配置到quazip工程
- D、编译quazip.sln工程，最后得到 **quazip.dll**

C/C++ > 常规 > 附加包含目录：./zlib-1.2.11/include/

C/C++ > 预处理 > 预处理定义：QUAZIP_BUILD

链接器 > 常规 > 附加库目录： ./zlib-1.2.11/lib/x86/

链接器 > 输入 > 附加依赖项：zlibd.lib




需要注意的是，quazip.dll 需要依赖 zlib.dll，所以在用到的地方需要把zlib.dll一并带上，而使用 quazip1-qt5.dll，则不需要。



## 三、Sample

```cpp
#include <QtCore/QCoreApplication>
#include <QDir>
#include <QFile>
#include <QFileInfo>
#include <QDateTime>
#include <QDebug>
#include <QElapsedTimer>
#include <QJsonDocument>
#include <QJsonObject>
#include "JlCompress.h"

int main(int argc, char* argv[])
{
	QCoreApplication a(argc, argv);

	QString zipName = QString("%1%2").arg("./download/").arg("package.zip");
	QString unzipDir = "./unzip_dir";
	QDir dir(unzipDir);
	if (!dir.exists())
	{
		bool okey = dir.mkdir(unzipDir);
		//bool okey = dir.mkpath(unzipDir);
		QString tip = QString("mkpath %1,%2").arg(unzipDir).arg(okey);
		qDebug() << tip;
	}

	QElapsedTimer timer;
	timer.start();
	QStringList files = {};
	QFileInfo fileInfo(zipName);
	if (fileInfo.exists())
	{
		files = JlCompress::extractDir(zipName, unzipDir);
	}
	qDebug() << "files count:" << files.size();
	qDebug() << "the operation(JlCompress::extractDir) took" << timer.elapsed() << "ms";

	QVariantMap NameAndPathMap;
	// 这里解压出来后的文件是按照文件名排序的
	foreach(QString fileItem, files)
	{
		QString fileName = QFileInfo(fileItem).fileName();
		qint64 fileSize = QFileInfo(fileItem).size();
		qDebug() << "Name:" << fileName << "Size:" << fileSize << "\tPath:" << fileItem;
		//
		NameAndPathMap.insert(fileName, fileItem);
	}

	//save to file
	QJsonObject root = QJsonObject::fromVariantMap(NameAndPathMap);
	QString content = QString(QJsonDocument(root).toJson(QJsonDocument::Indented/*Compact*/));
	//
	QFile writer("./content.json");
	if (writer.open(QIODevice::WriteOnly))
	{
		writer.write(content.toUtf8());
		writer.close();
	}
	qDebug() << "over!!!";
	return a.exec();
}
```

### 四、踩过的坑

编译了zlib后，在环境变量里面都添加了 **ZLIB_LIBRARY** 和 **ZLIB_INCLUDE_DIR**，但是在CMake 配置quazip时，都没有生效。
至于网上有些使用cmake命令构建，我没过试过，例如：

```bash
cmake .. -DZLIB_INCLUDE_DIR=E:\zlib\zlib-1.2.11 -DZLIB_LIBRARY=E:\zlib\zlib-1.2.11\zlibstaticd.lib
```

有兴趣有时间的童鞋可以尝试一下。