
#CXX17

> References
> 
> - [现代C++学习——实现多类型存储std::variant](https://zhuanlan.zhihu.com/p/597987861)
>     
> - [如何优雅的使用 std::variant 与 std::optional](https://zhuanlan.zhihu.com/p/366537214)
>     

std::variant 是 [C++17](https://kheresy.wordpress.com/2017/04/15/changes-between-c14-and-c17-dis/) 中，一個新加入標準函式庫的 template 容器；他的概念基本上是和 union（[參考](http://en.cppreference.com/w/cpp/language/union)）一樣，是一個可以用來儲存多種型別資料的容器。

比如說：

```cpp
std::variant<int, double> v;
```

就代表 v 這個变量，可以用來儲存 int 或 double 的資料，variant 內部自己會去記錄相關的資訊。

而和 union 不同的地方，variant 也是 type-safe 的，再加上有許多函式可以搭配使用，所以在使用上應該算是相對安全；另外也由於他是標準函式庫的 template class，在使用時不需要另外去宣告一個新的型別。

---

### 基本使用

如果要使用 variant 的話，程式必須先 include <\variant> 這個 header 文件；而之後呢，則就是以 template 的形式，把要允許的型別指定好，就可以用了。

下面是一個簡單的例子：

```cpp
std::variant<int, double, std::string> x, y;

// assign value
x = 1;
y = "1.0";

// overwrite value
x = 2.0;
```

這邊是宣告了 x、y 兩個變數，透過 variant 讓他們可以儲存 int、double 或 string 的資料。

而接下來，則是讓 x 去記錄一個 int 的數字 1、並讓 y 去記錄一個字串 1.0。

之後，則是把 x 改為一個 double 的數字 2.0；而這個時候，本來 x 記錄的 int 的數字 1 就會消失了。

實際上 variant 在儲存資料的時候，內部還會有一個索引值，來記錄目前是儲存哪一種類型的資料，而透過他的 index() 這個函式，也就可以知道目前是使用第幾種型別了。

例如，在上面的程式執行完後，再繼續執行下面的程式碼：

```cpp
// check index
std::cout << "x - " << x.index() << std::endl;
std::cout << "y - " << y.index() << std::endl;
```

這樣就會得到 x 的 index 是 1、y 的 index 是 2 的結果了。

---

### 讀取資料

當要讀取 variant 的資料的時候，需要透過 std::get<>() 這個 template 函式來在編譯階段決定讀取的型別。他有兩種方法可以指定，一個給一個數字當 index，或是直接告訴他是要用哪個型別。

下面就是簡單的範例：

```cpp
// read value
double d = std::get<double>(x);
std::string s = std::get<2>(y);
```

像上面在讀取 y 的資料時候，是告訴系統是要把 y 當成第 2 號的資料型別來讀取，也就是 std::string 了。

而如果是指定錯誤的型別的話，std::get<>() 則是會丟出一個例外狀況，沒處理的話就會讓程式當掉。下面就是這樣的例子：

```cpp
// error type
try
{
	int i = std::get<int>(x);
}
catch (std::bad_variant_access e)
{
	std::cerr << e.what() << std::endl;
}
```

由於的 x 內部是儲存 double 的資料，但是這邊卻試著把他當 int 讀，所以在執行後就會丟出 std::bad_variant_access 這個例外狀況了。

而如果不想用 try-catch 來處理例外狀況的話，則可以使用 std::get_if<>() 這個函式，來取得值的指標；而如果型別不符合的話，則是會得到一個 nullptr。下面就是一個簡單的例子：

```cpp
// use get_if
int* i = std::get_if<int>(&x);
if (i == nullptr)
{
	std::cout << "wrong type" << std::endl;
}
else
{
	std::cout << "value is " << *i << std::endl;
}
```

---

### 透過 visit() 來自動處理型別

如果只是透過上面提到的get<>() 來做存取，那其實 Heresy 個人會覺得用 variant 的意義感覺不算很大。

個人覺得 variant 要好用，還要搭配 std::visit() 這個函式（[參考](http://en.cppreference.com/w/cpp/utility/variant/visit)）來使用。

visit() 基本上是一個用來處理 variant 型別的函式，讓開發者不用自己根據所有可能、一種一種去切換；在使用時，需要給他一個可以處理所有可能型別的可呼叫（callable）物件、來進行操作。

比如說，這邊要可以比較快輸出上面的 x 和 y 的話，可以定義一個 SOutput 如下：

```cpp
struct SOutput
{
	void operator()(const int& i)
	{
		std::cout << i << std::endl;
	}
 
	void operator()(const double& d)
	{
		std::cout << d << std::endl;
	}
 
	void operator()(const std::string& s)
	{
		std::cout << s << std::endl;
	}
};
```

可以看到，這邊有針對所有有用到型別（int、double 和 string ）都去定義對應的 function call operator。之後要使用的時候，則只要呼叫：

```cpp
std::visit(SOutput(), y);
```

這樣編譯器就會找到對應的函式來執行了！

由於這部分會在編譯階段做檢查，所以這邊的 SOutput 要確定有針對所有可能的型別，都撰寫對應的函式，如果有缺的話，在編譯階段就不會過了！這也是一種相對安全的程式寫法。

而這邊也可以透過 template 的方式，來減少重複的程式碼；像是上面的 SOutput 就可以改寫成：

```cpp
struct STOutput
{
	template<typename TYPE>
	void operator()(const TYPE& v)
	{
		std::cout << v << std::endl;
	}
};
```

如此一來，只要寫一個函式，就可以對應所有狀況了～

而如果搭配 [C++14](https://kheresy.wordpress.com/2014/08/20/c14-is-done/) 的 Generic Lambda，則可以更簡單地寫成：

```cpp
std::visit(
	[](const auto& v) {std::cout << v << std::endl; },
	x);
```

這樣應該就算是相當方便了～

---

不過，如果有要透過不同的型別，做不同的處理，就不能這麼方便的 Generic Lambda 了…基本上，這邊就得回到前面，自己去定義一個 callable object，然後針對需求，各自去實作對應的函式。

下面就是一個簡單的例子：

```cpp
 struct STwice  
 {  
     template<typename TYPE>  
     void operator()(TYPE& v)  
     {  
         v *= 2;  
     }  
    
     template<>  
     void operator()(std::string& s)  
     {  
         s += s;  
     }  
 };
```

在這個 STwice 裡，如果型別 `std::string` 的話，他會把字串重複兩次；而如果是其他的型別的話，則是會透過 template 處理、直接乘二。

透過這樣的寫法，就可以根據不同的型別，做不同的處理了。

---

如果不想另外定義一個 struct 的話，其實在 cppreference 有提供一個使用多個 lambda 來組合的例子（[參考](http://en.cppreference.com/w/cpp/utility/variant/visit)）；他的概念應該是使用 [parameter pack](https://kheresy.wordpress.com/2017/05/05/parameter-pack-in-c11/) 的多重繼承的方法來做的，但是他的語法在 msvc 無法正確編譯…

而他用的語法…恩，Heresy 也看不懂（應該是 User-defined deduction guides、[參考](http://en.cppreference.com/w/cpp/language/class_template_argument_deduction)）。 orz

不過，如果真的想要組合多個 lambda 的話，可以參考 lambda_util::compose() 這個實作（[gist](https://gist.github.com/cdacamar/584c6d43a9cca1ccffec3b36ad5dfe3f)），這份程式在 MSVC2017 是可以正確運作的。

而如果把它直接拿來用的話，前面的 STwice 就可以變成下面這樣：

```cpp
 std::visit(  
     lambda_util::compose(  
         [](auto& v) { v *= 2; },  
         [](std::string& s) { s += s; }  
     ), y);
```

基本上，算是好寫一點了。

---

這邊針對 std::variant 的介紹大概就先到這邊了。實際上，他還有一些其他函式可以用，不過這邊就先跳過了。

完整的範例程式，可以參考放在 GitHub 上的檔案：[https://github.com/KHeresy/misc/blob/master/std_variant.cpp](https://github.com/KHeresy/misc/blob/master/std_variant.cpp)。

不過，由於 C++17 是相對新、還沒完全定案的標準，所以編譯器要相當新的版本才能支援；以 MSVC 來說，就是需要 VisualStudio 2017 才能支援，而 gcc 的 libstdc++ 則是要到 7.0 以後才支援。

而 Boost 雖然也有提供 Variant（[網頁](http://www.boost.org/doc/html/variant.html)）這個函式庫，但是實際上他的語法和 C++17 的似乎是略有不同；像 Boost 的版本的 visit() 就變成是 apply_visitor()，也沒有 get_if<>() 這個函式（似乎是直接用 get<>()）…

所以以現階段來說，個人是覺得還不是很適合直接正式使用吧。

---

另外，在 Heresy 來看，std::variant 一個可能可以拿來實用的地方，就是透過它來讓不同的資料可以放在同一個容器內、批次處理。

下面就是一個簡單的範例：

```cpp
 // vector  
 using var_t = std::variant<int, double, std::string>;  
 std::vector<var_t> vData = { 1, 2.0, "hi" };  
 for (var_t& v : vData)  
 {  
     std::visit(STwice(), v);  
     std::visit(SOutput(), v);  
 }
```

要做這樣的事，以往大多是要用比較複雜的繼承、抽象化來解決的；而現在有了 variant，在某些狀況下應該是可以更簡單就可以做到同樣的事了！

而相較於使用繼承會把程式分散在個別的類別中，這邊的特色是，針對不同型別的處理的程式會都集中在一起，某方面來說算是各有優缺點了。

這部分可以參考《[New Tools for a More Functional C++](https://www.slideshare.net/SumantTambe/new-tools-for-a-more-functional-c-80294483?ref=http://cpptruths.blogspot.tw/2017/09/new-tools-for-more-functional-c.html)》這份投影片。而實際上，Heresy 也是因為看了這份投影片，才來研究 variant 的。