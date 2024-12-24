
#Qt 

> 参考博客：
> 
> * https://blog.csdn.net/hp_cpp/article/details/80338116 从文件中读取json
> * https://www.cnblogs.com/ybqjymy/p/17264853.html
> * https://www.jb51.net/article/260149.htm
> * https://blog.csdn.net/cpp_learner/article/details/118421096

之前介绍了许多 C++ 的 Json 第三方库，

下面介绍一下 Json 在 Qt 中的使用。

从Qt 5.0开始提供了对 Json 的支持，我们可以直接使用Qt提供的 Json 类进行数据的组织和解析。相关的类常用的主要有四个，具体如下：

|     Json类      |                             介绍                             |
| :-------------: | :----------------------------------------------------------: |
| `QJsonDocument` | 它封装了一个完整的JSON文档，并且可以从UTF-8编码的基于文本的表示以及Qt自己的二进制格式读取和写入该文档。 |
|  `QJsonArray`   | JSON数组是一个值列表。可以通过从数组中插入和删除QJsonValue来操作该列表。 |
|  `QJsonObject`  | JSON对象是键值对的列表，其中键是唯一的字符串，值由QJsonValue表示。 |
|  `QJsonValue`   |                该类封装了JSON支持的数据类型。                |

## 一、QJsonValue

在Qt中 `QJsonValue` 可以封装的基础数据类型有六种（和Json支持的类型一致），分别为：

* 布尔类型：`QJsonValue::Bool`
* 浮点类型（包括整形）：`QJsonValue::Double`
* 字符串类型： `QJsonValue::String`
* Json数组类型： `QJsonValue::Array`
* Json对象类型：`QJsonValue::Object`
* 空值类型： `QJsonValue::Null`

这个类型可以通过 `QJsonValue` 的构造函数被封装为一个类对象:

```cpp
// Json对象
QJsonValue(const QJsonObject &o);
// Json数组
QJsonValue(const QJsonArray &a);
// 字符串
QJsonValue(const char *s);
QJsonValue(QLatin1String s);
QJsonValue(const QString &s);
// 整形 and 浮点型
QJsonValue(qint64 v);
QJsonValue(int v);
QJsonValue(double v);
// 布尔类型
QJsonValue(bool b);
// 空值类型
QJsonValue(QJsonValue::Type type = Null);
```

如果我们得到一个 `QJsonValue` 对象，如何判断内部封装的到底是什么类型的数据呢？这时候就需要调用相关的判断函数了，具体如下：

```cpp
// 是否是Json数组
bool isArray() const;
// 是否是Json对象
bool isObject() const;
// 是否是布尔类型
bool isBool() const;
// 是否是浮点类型(整形也是通过该函数判断)
bool isDouble() const;
// 是否是空值类型
bool isNull() const;
// 是否是字符串类型
bool isString() const;
// 是否是未定义类型(无法识别的类型)
bool isUndefined() const;
```

通过判断函数得到对象内部数据的实际类型之后，如果有需求就可以再次将其转换为对应的基础数据类型，对应的API函数如下：

```cpp
// 转换为Json数组
QJsonArray toArray(const QJsonArray &defaultValue) const;
QJsonArray toArray() const;
// 转换为布尔类型
bool toBool(bool defaultValue = false) const;
// 转换为浮点类型
double toDouble(double defaultValue = 0) const;
// 转换为整形
int toInt(int defaultValue = 0) const;
// 转换为Json对象
QJsonObject toObject(const QJsonObject &defaultValue) const;
QJsonObject toObject() const;
// 转换为字符串类型
QString toString() const;
QString toString(const QString &defaultValue) const;
```

## 二、QJsonObject

`QJsonObject` 封装了Json中的对象，在里边可以存储多个键值对，为了方便操作，键值为字符串类型，值为`QJsonValue` 类型。

关于这个类的使用类似于C++中的STL类，仔细阅读API文档即可熟练上手使用，下面介绍一些常用API函数:

* 如何创建空的Json对象

  ```cpp
  QJsonObject::QJsonObject();	// 构造空对象
  ```

* 将键值对添加到空对象中

  ```cpp
  iterator QJsonObject::insert(const QString &key, const QJsonValue &value);
  ```

* 获取对象中键值对个数

  ```cpp
  int QJsonObject::count() const;
  int QJsonObject::size() const;
  int QJsonObject::length() const;
  ```

* 通过key得到value

  ```cpp
  QJsonValue QJsonObject::value(const QString &key) const;    // utf8
  QJsonValue QJsonObject::value(QLatin1String key) const;	    // 字符串不支持中文
  QJsonValue QJsonObject::operator[](const QString &key) const;
  QJsonValue QJsonObject::operator[](QLatin1String key) const;
  ```

* 删除键值对

  ```cpp
  void QJsonObject::remove(const QString &key);
  QJsonValue QJsonObject::take(const QString &key);	// 返回key对应的value值
  ```

* 通过key进行查找

  ```cpp
  iterator QJsonObject::find(const QString &key);
  bool QJsonObject::contains(const QString &key) const;
  ```

* 遍历，方式有三种：

  * 使用相关的迭代器函数

  * 使用 [] 的方式遍历, 类似于遍历数组, []中是键值

  * 先得到对象中所有的键值, 在遍历键值列表, 通过key得到value值

    ```cpp
    QStringList QJsonObject::keys() const;
    ```

    

## 三、QJsonArray

`QJsonArray` 封装了Json中的数组，在里边可以存储多个元素，为了方便操作，所有的元素类统一为 `QJsonValue` 类型。关于这个类的使用类似于C++中的STL类，仔细阅读API文档即可熟练上手使用，下面介绍一些常用API函数:

* 创建空的Json数组

  ```cpp
  QJsonArray::QJsonArray();
  ```

* 添加数据

  ```cpp
  void QJsonArray::append(const QJsonValue &value);	// 在尾部追加
  void QJsonArray::insert(int i, const QJsonValue &value); // 插入到 i 的位置之前
  iterator QJsonArray::insert(iterator before, const QJsonValue &value);
  void QJsonArray::prepend(const QJsonValue &value); // 添加到数组头部
  void QJsonArray::push_back(const QJsonValue &value); // 添加到尾部
  void QJsonArray::push_front(const QJsonValue &value); // 添加到头部
  ```

* 计算数组元素的个数

  ```cpp
  int QJsonArray::count() const;
  int QJsonArray::size() const;
  ```

* 从数组中取出某一个元素的值

  ```cpp
  QJsonValue QJsonArray::at(int i) const;
  QJsonValue QJsonArray::first() const; // 头部元素
  QJsonValue QJsonArray::last() const; // 尾部元素
  QJsonValueRef QJsonArray::operator[](int i);
  ```

* 删除数组中的某一个元素

  ```cpp
  iterator QJsonArray::erase(iterator it);    // 基于迭代器删除
  void QJsonArray::pop_back();           // 删除尾部
  void QJsonArray::pop_front();          // 删除头部
  void QJsonArray::removeAt(int i);      // 删除i位置的元素
  void QJsonArray::removeFirst();        // 删除头部
  void QJsonArray::removeLast();         // 删除尾部
  QJsonValue QJsonArray::takeAt(int i);  // 删除i位置的原始, 并返回删除的元素的值
  ```

* Json 数组的遍历，常用的方式有两种：

  * 可以使用迭代器进行遍历（和使用迭代器遍历STL容器一样）
  * 可以使用数组的方式遍历

## 四、QJsonDocument

它封装了一个完整的JSON文档，并且可以从 $UTF-8$ 编码的基于文本的表示以及Qt自己的二进制格式读取和写入该文档。

`QJsonObject` 和 `QJsonArray` 这两个对象中的数据是不能直接转换为字符串类型的，如果要进行数据传输或者数据的持久化，操作的都是字符串类型而不是 `QJsonObject` 或者 `QJsonArray` 类型，我们需要通过一个Json文档类进行二者之间的转换。

下面依次介绍一下这两个转换流程应该如何操作:

* **QJsonObject** 或者 **QJsonArray** ===> **字符串**

  1. 创建 `QJsonDocument` 对象

     ```cpp
     QJsonDocument::QJsonDocument(const QJsonObject &object);
     QJsonDocument::QJsonDocument(const QJsonArray &array);
     ```

     可以看出，通过构造函数就可以将实例化之后的 **QJsonObject 或者 QJsonArray **转换为`QJsonDocument` 对象了。

  2. 将文件对象中的数据进行序列化

     ```cpp
     // 二进制格式的json字符串
     QByteArray QJsonDocument::toBinaryData() const;	 
     // 文本格式
     QByteArray QJsonDocument::toJson(JsonFormat format = Indented) const;
     ```

     通过调用 `toxxx()` 方法就可以得到文本格式或者二进制格式的 Json 字符串了。

  3. 使用得到的字符串进行数据传输, 或者磁盘文件持久化

* **字符串** ===> **QJsonObject** 或者 **QJsonArray**

  一般情况下，通过网络通信或者读磁盘文件就会得到一个Json格式的字符串，如果想要得到相关的原始数据就需要对字符串中的数据进行解析，具体解析流程如下：

  1. 将得到的 Json 格式字符串通过 `QJsonDocument` 类的静态函数转换为 `QJsonDocument` 类对象

     ```cpp
     [static] QJsonDocument QJsonDocument::fromBinaryData(const QByteArray &data, DataValidation validation = Validate);
     // 参数文件格式的json字符串
     [static] QJsonDocument QJsonDocument::fromJson(const QByteArray &json, QJsonParseError *error = Q_NULLPTR);
     ```

  2. 将文档对象转换为json数组/对象

     ```cpp
     // 判断文档对象中存储的数据是不是数组
     bool QJsonDocument::isArray() const;
     // 判断文档对象中存储的数据是不是json对象
     bool QJsonDocument::isObject() const
         
     // 文档对象中的数据转换为json对象
     QJsonObject QJsonDocument::object() const;
     // 文档对象中的数据转换为json数组
     QJsonArray QJsonDocument::array() const;
     ```

  3. 通过调用QJsonArray , QJsonObject 类提供的 API 读出存储在对象中的数据。

  关于Qt中Json数据对象以及字符串之间的转换的操作流程是固定的，我们在编码过程中只需要按照上述模板处理即可，相关的操作是没有太多的技术含量可言的。

## 五、Example

### 5.1 Write to File

```cpp
void writeJson()
{
    QJsonObject obj;
    obj.insert("Name", "Ace");
    obj.insert("Sex", "man");
    obj.insert("Age", 20);

    QJsonObject subObj;
    subObj.insert("Father", "Gol·D·Roger");
    subObj.insert("Mother", "Portgas·D·Rouge");
    QJsonArray array;
    array.append("Sabo");
    array.append("Monkey D. Luffy");
    subObj.insert("Brother", array);
    obj.insert("Family", subObj);
    obj.insert("IsAlive", false);
    obj.insert("Comment", "yyds");

    QJsonDocument doc(obj);
    QByteArray json = doc.toJson();

    QFile file("d:\\ace.json");
    file.open(QFile::WriteOnly);
    file.write(json);
    file.close();
}
```

### 5.2 Read from File

```cpp
void MainWindow::readJson()
{
    QFile file("d:\\ace.json");
    file.open(QFile::ReadOnly);
    QByteArray json = file.readAll();
    file.close();

    QJsonDocument doc = QJsonDocument::fromJson(json);
    if(doc.isObject())
    {
        QJsonObject obj = doc.object();
        QStringList keys = obj.keys();
        for(int i=0; i<keys.size(); ++i)
        {
            QString key = keys.at(i);
            QJsonValue value = obj.value(key);
            if(value.isBool())
            {
                qDebug() << key << ":" << value.toBool();
            }
            if(value.isString())
            {
                qDebug() << key << ":" << value.toString();
            }
            if(value.isDouble())
            {
                qDebug() << key << ":" << value.toInt();
            }
            if(value.isObject())
            {
                qDebug()<< key << ":";
                // 直接处理内部键值对, 不再进行类型判断的演示
                QJsonObject subObj = value.toObject();
                QStringList ls = subObj.keys();
                for(int i=0; i<ls.size(); ++i)
                {
                    QJsonValue subVal = subObj.value(ls.at(i));
                    if(subVal.isString())
                    {
                        qDebug() << "   " << ls.at(i) << ":" << subVal.toString();
                    }
                    if(subVal.isArray())
                    {
                        QJsonArray array = subVal.toArray();
                        qDebug() << "   " << ls.at(i) << ":";
                        for(int j=0; j<array.size(); ++j)
                        {
                            // 因为知道数组内部全部为字符串, 不再对元素类型进行判断
                            qDebug() << "       " << array[j].toString();
                        }
                    }
                }
            }
        }
    }
}
```

一般情况下，对于Json字符串的解析函数都是有针对性的，因为需求不同设计的Json格式就会有所不同，所以不要试图写出一个通用的Json解析函数，这样只会使函数变得臃肿而且不易于维护，每个Json格式对应一个相应的解析函数即可。

上面的例子中为了给大家演示Qt中Json类相关API函数的使用将解析步骤写的复杂了，因为在解析的时候我们是知道Json对象中的所有key值的，可以直接通过key值将对应的value值取出来，因此上面程序中的一些判断和循环其实是可以省去的。