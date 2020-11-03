在定义类的过程中，无论是显式创建类的构造方法，还是向类中添加实例方法，都要求将 self 参数作为方法的第一个参数。例如，定义一个 Person 类：

```python
class Person:
    def __init__(self):
        print("正在执行构造方法")
    # 定义一个study()实例方法
    def study(self):
        print(self,"正在学Python")
zhangsan = Person()
zhangsan.study()
lisi = Person()
lisi.study()
```

那么，self 到底扮演着什么样的角色呢？接下来将对 self 参数做详细的介绍。

事实上，Python 只是规定，无论是构造方法还是实例方法，最少要包含一个参数，并没有规定该参数的具体名称。之所以将其命名为 self，只是程序员之间约定俗成的一种习惯，遵守这个约定，可以使我们编写的代码具有更好的可读性（大家一看到 self，就知道它的作用）。

那么，self 参数的具体作用是什么呢？打个比方，如果把类比作造房子的图纸，那么类实例化后的对象是真正可以住的房子。根据一张图纸（类），我们可以设计出成千上万的房子（类对象），每个房子长相都是类似的（都有相同的类变量和类方法），但它们都有各自的主人，那么如何对它们进行区分呢？

当然是通过 self 参数，它就相当于每个房子的门钥匙，可以保证每个房子的主人仅能进入自己的房子（每个类对象只能调用自己的类变量和类方法）。

> 如果你接触过其他面向对象的编程语言（例如 C++），其实 Python 类方法中的 self 参数就相当于 C++ 中的 this 指针。

也就是说，同一个类可以产生多个对象，当某个对象调用类方法时，该对象会把自身的引用作为第一个参数自动传给该方法，换句话说，Python 会自动绑定类方法的第一个参数指向调用该方法的对象。如此，Python解释器就能知道到底要操作哪个对象的方法了。

因此，程序在调用实例方法和构造方法时，不需要手动为第一个参数传值。例如，更改前面的 Person 类，如下所示：

```python
class Person:
    def __init__(self):
        print("正在执行构造方法")
    # 定义一个study()实例方法
    def study(self):
        print(self,"正在学Python")
zhangsan = Person()
zhangsan.study()
lisi = Person()
lisi.study()
```

上面代码中，study() 中的 self 代表该方法的调用者，即谁调用该方法，那么 self 就代表谁。因此，该程序的运行结果为：

```python
正在执行构造方法
<__main__.Person object at 0x0000021ADD7D21D0> 正在学Python
正在执行构造方法
<__main__.Person object at 0x0000021ADD7D2E48> 正在学Python
```


另外，对于构造函数中的 self 参数，其代表的是当前正在初始化的类对象。举个例子：

```
class Person:
    name = "xxx"
    def __init__(self,name):
        self.name=name

zhangsan = Person("zhangsan")
print(zhangsan.name)
lisi = Person("lisi")
print(lisi.name)
```

运行结果为：

```
zhangsan
lisi
```

可以看到，zhangsan 在进行初始化时，调用的构造函数中 self 代表的是 zhangsan；而 lisi 在进行初始化时，调用的构造函数中 self 代表的是 lisi。

值得一提的是，除了类对象可以直接调用类方法，还有一种函数调用的方式，例如：

```python
class Person:
    def who(self):
        print(self)
zhangsan = Person()
#第一种方式
zhangsan.who()
#第二种方式
who = zhangsan.who
who()#通过 who 变量调用zhangsan对象中的 who() 方法
```

运行结果为：

```python
<__main__.Person object at 0x0000025C26F021D0>
<__main__.Person object at 0x0000025C26F021D0>
```

显然，无论采用哪种方法，self 所表示的都是实际调用该方法的对象。

> 总之，无论是类中的构造函数还是普通的类方法，实际调用它们的谁，则第一个参数 self 就代表谁。