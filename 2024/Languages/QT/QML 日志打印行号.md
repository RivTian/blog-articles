
在qml中我们调试打印信息使用console.log()方式去打印信息，但是在控制台上并不能显示该条信息具体打印的位置以及是哪个文件。如果我们项目的文件非常多，那么很难定位。那么使用Qt日志重定向功能很好的解决这个问题。

```cpp
// main.cpp 
#include <QFile>
#include <QMutex>
#include <QDateTime>
 
QtMessageHandler gDefaultHandler = NULL;
void myMessageOutput(QtMsgType type, const QMessageLogContext &context, const QString &msg)
{
    QDateTime time = QDateTime::currentDateTime();
    QString strTime = time.toString("MM-dd hh:mm:ss");
    QString strMessage = QString("[%1 File:%2  Line:%3  Function:%4]:  %5")
            .arg(strTime).arg(context.file).arg(context.line).arg(context.function).arg(msg);
 
    QString strMsg("");
    switch(type)
    {
    case QtDebugMsg:
        strMsg = QString("Debug");
        break;
    case QtInfoMsg:
        strMsg = QString("Info");
        break;
    case QtWarningMsg:
        strMsg = QString("Warning:");
        break;
    case QtCriticalMsg:
        strMsg = QString("Critical:");
        break;
    case QtFatalMsg:
        strMsg = QString("Fatal:");
        break;
    default:
        strMsg = QString("Err");
        break;
    }
 
    strMessage = strMsg+strMessage;
 
#if 0
    // 加锁
    static QMutex mutex;
    mutex.lock();
 
 
    // 输出信息至文件中（读写、追加形式）
    QFile file("D:\\log.txt");
    file.open(QIODevice::ReadWrite | QIODevice::Append);
    QTextStream stream(&file);
    stream << strMessage << "\r\n";
    file.flush();
    file.close();
 
    // 解锁
    mutex.unlock();
#endif
    //用系统原来的函数完成原来的功能. 比如输出到调试窗
    if(gDefaultHandler)
    {
        gDefaultHandler(type,context,strMessage);
    }
 
}

```

```cpp
gDefaultHandler = qInstallMessageHandler(myMessageOutput);
// 在main函数中调用：
```

这样不论是qml还是cpp文件的打印，都可以输出你想要的信息。