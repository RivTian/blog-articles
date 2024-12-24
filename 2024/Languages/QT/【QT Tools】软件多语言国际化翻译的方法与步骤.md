
#Qt 

在Qt的项目开发过程中，有时软件要翻译成多语言版本，这就涉及到国际化方面的操作。虽然Qt对这方面集成了很多工具，操作起来比较方便，本文还是总结一下国际化的方法和步骤，用以备忘和参考。

我们通常在写程序时，为了正常显示中文，一般会直接用到类`QTextCodec`和`tr()`函数，其实这只是一种临时的做法，方便我们快速完成程序、显示效果。当要真正发布一个程序时，最好的方式是`在程序中使用英文字符串，再使用国际化工具进行翻译`。

## 1.软件国际化的方法

在Qt中，所有的输入部件和文本绘制方式对对所有支持的语言都提供了内置支持，Qt内置的字体引擎可以在同一时间正确而精细地绘制不同的文本。在Qt项目开发过程中，我们可以使用`Qt Linguist`工具来完成应用程序的翻译工作。

在Qt中编写代码时，要对需要显示的字符串调用`tr()`函数，完成代码编辑后，对这个应用程序的翻译主要分三个步骤：

> CMD: 此处代指 Qt Creator [工具] -> [外部] -> [Qt 语言家]

- 在`CMD`中运行`lupdate`工具，从源代码中提取要翻译的文本，这里会生成一个`.ts`文件，此文件格式是`xml`格式；
- 在`Qt Linguist`中打开`.ts`文件，并完成翻译工作；
- 在`CMD`中运动`lrelease`工具，从`.ts`文件中生成一个`.qm`文件，它是一个二进制文件，用于程序运行时使用。

其中，`.ts`文件和`.qm`文件都是平台无关的 。



## 2.具体实例实现步骤

- 新建Qt Gui应用，项目名称为`myI18N`，类名为`MainWindow`，基类保持`QMainWindow`不变。
- 建立完项目后，点击`mainwindow.ui`文件进入设计模式，先添加一个`&File`菜单，再为其添加一个`&New`子菜单并设置快捷键为`Ctrl+N`，然后往界面上拖入一个`Push Button` 。
- 下面我们再使用代码添加几个标签，打开`mainwindow.cpp`文件，添加头文件`#include <QLabel>`，然后在构造函数中添加代码：

```cpp
label = new QLabel(this);
label->setText(tr("helllo QT"));
label->move(100,50);
label2 = new QLabel(this);
label2->setText(tr("password","mainwindow"));
label2->move(100,80);
label3 = new QLabel(this);
int id = 123;
QString name = "wusanlang";
label3->setText(tr("ID is %1, Name is %2").arg(id).arg(name));
label3->resize(180,20);
label3->move(100,120);
```

这里值得注意的是`tr()`函数是不止一个参数的，其原型为：

```cpp
QString QObject::tr(const char *sourceText, const char *disambiguation = Q_NULLPTR, int n = -1)
```

第一个参数`sourceText`就是要显示的字符串，`tr()`函数会返回`sourceText`的译文；

第二个参数`disambiguation`是消除歧义字符串，比如这里的`password`，如果一个程序中需要输入多个不同的密码，那么在没有上下文的情况下，就很难确定这个`password`到底指哪个密码。这个参数一般使用类名或者部件名，比如这里使用了`mainwindow`，就说明这个`password`是在`mainwindow`上的；

第三个参数n表明是否使用了复数，因为英文单词中复数一般要在单词末尾加“s”，比如“1 message”，复数时为“2 messages”。遇到这种情况，就可以使用这个参数，它可以根据数值来判断是否需要添加“s”。

* 程序运行结果如下图所示。

![](https://images.cnblogs.com/cnblogs_com/RioTian/2326543/o_231010021110_%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16969038582443.png)

源程序编写完成后，我们在`.pro`文件中指定生成`.ts`文件的名字，即在文件中添加语句：

```properties
TRANSLATIONS = myI18N_zh_CN.ts    //文件名可以随意，但要注意区分
```

* 在`CMD`中执行`lupdate.exe`程序，生成`.ts`文件。
* 这时使用`Qt Linguist` 打开`.ts`文件进行翻译 ，翻译完成后保存。
* 翻译完成后，使用`lrelease.exe`程序，生成`.qm`文件。

- 在项目程序中，添加`.qm`文件来改变程序的语言。

在使用`qm`文件时，要用到`QTranslator`类，其使用方法为：

```cpp
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);         //全局对象指针是宏qApp

    QTranslator translator;
    translator.load("../myI18N_zh_CN.qm");
    a.installTranslator(&translator);

    MainWindow w;
    w.show();

    return a.exec();
}
```

本测试项目中，我自己制作了，通过点击按钮来实现中英文的切换，其按钮槽函数为实现如下：

```cpp
void MainWindow::on_pushButton_clicked()
{
    QTranslator translator;
    if(m_bTranslator)    //翻译成中文
    {
        translator.load("../myI18N_zh_CN.qm");
        qApp->installTranslator(&translator);
        ui->retranslateUi(this);
        //自己添加非ui控件的动态翻译
        label->setText(QApplication::translate("MainWindow", "helllo QT", Q_NULLPTR));
        label2->setText(QApplication::translate("MainWindow", "password", Q_NULLPTR));
        label3->setText(QApplication::translate("MainWindow", "ID is %1, Name is %2", Q_NULLPTR));
    }
    else
    {
        qApp->removeTranslator(&translator);
        ui->retranslateUi(this);
        //自己添加的非ui控件的动态翻译
        label->setText(QApplication::translate("MainWindow", "hello QT", Q_NULLPTR));
        label2->setText(QApplication::translate("MainWindow", "password", Q_NULLPTR));
        label3->setText(QApplication::translate("MainWindow", "ID is %1, Name is %2", Q_NULLPTR));
    }
    m_bTranslator = !m_bTranslator;
}
```

Qt国际化翻译并不难，本文主要总结归纳一下，以便备忘和后期参考。