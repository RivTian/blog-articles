本文较为详细地介绍了在Windows系统下，如何配置Sublime Text的C++编译运行环境。目前实现了了可以在Sublime Text按下快捷键后，调出CMD或者终端来运行C/C++程序，从而解决了Sublime Text无法接收输入的问题。
另外本文也介绍了一些Sublime Text的使用和用户配置备份的方法，具有一定的参考作用。

## 1.使用背景

现在搞ACM-ICPC的比赛，经常写算法程序。说一说使用过的IDE吧：

* Visual Studio 2013~2019

    配合`Visual Assist`插件真的非常好用，没有之一！无论是写程序或者调试代码都能非常优雅地完成！当然我只用到了这个它的一小部分，不过我感觉这种企业级别的IDE绝对是写工程的不二选择。可惜在比赛时并不会提供这么优秀的IDE，而且想 `VS` 这种大型IDE是很吃机器配置的。在老式台式机上打开项目的速度几乎崩溃。

* CodeBlock 和 Dev

    比赛举办方通常提供的IDE，一个开源的轻量级的IDE。体积不大，功能也比较齐全，不过个人稍微觉得有几个缺点：界面颜值较低，虽然有字体颜色等几项可调节，但两个IDE给我的感觉就是不好看（老颜狗了

    关于下载速度的话，由于我电脑上常年有科学方法所以没有感到下载速度慢

* VS Code 

    VS Code同VS一样出自微软，曾经我写过 [Windows VS Code 配置 C/C++ 开发环境教程](https://www.cnblogs.com/RioTian/p/13812319.html) ，再看发布时间发现已经使用了接近一年了（是目前使用时间最长的IDE软件），但作为一个追求简洁和敲代码敲个爽的蒟蒻决定放弃 VS Code（尽管各方面已经被我调教的很好了）

## 2. 下载安装

[官网下载](http://www.sublimetext.com/)。Sublime Text是需要激活的，当然你不激活也可以正常使用，只是它会时不时提醒你而已。有能力的话，还是可以支持一下这么优秀的一款软件~

### 2.1 细节配置

**字体、主题风格等设置**

当需要更改主题时，直接可以通过`Preferences —> Color Scheme`来设置，主界面上只能改变字体的大小。若需要改变字体和字体大小，可以先`Preferences —> Browse Packages`，找到`Default`文件夹，然后找到`Preferences.sublime-settings`这个文件，用Sublime Text 3打开这个文件，这个文件保存了一些常用的设置，比如字体、主题风格、是否显示行号、智能提示延迟时间等，可以根据自己的需要自行设置。

**打开（关闭）侧边栏、右边缩略图等常用面板**

默认情况下Sublime Text 3是没有打开侧边栏文件浏览器的，可以通过`View`来打开和关闭侧边栏，默认情况下Sublime Text 3右边是有文件的缩略图的，可以通过`View`来打开和关闭缩略图。

**快捷键寻找文件和已定义的函数**

在Sublime Text 3中可以非常快速地切换到想找的文件，只需要通过`Ctrl+P`打开切换面板即可。然后输入想找的文件名称就可以快速找切换到该文件了。如果想要找函数，可以通过输入`@+函数名`可以快速切换到定义该函数的文件。

### 2.2 插件安装

> 这里记录了我最常用的几个插件

#### Package Control

*必装的插件，有了它可以很方便的安装和管理其他的插件。*

1. 在Sublime Text中使用快捷键`ctrl+``调出`console`或者点击：`View`->`Show Console`.然后把以下代码(适用于Sublime Text 3)复制进去:

    ```json
     import urllib.request,os,hashlib; h = '6f4c264a24d933ce70df5dedcf1dcaee' + 'ebe013ee18cced0ef93d5f746d80ef60'; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( 'http://packagecontrol.io/' + pf.replace(' ', '%20')).read(); dh = hashlib.sha256(by).hexdigest(); print('Error validating download (got %s instead of %s), please try manual install' % (dh, h)) if dh != h else open(os.path.join( ipp, pf), 'wb' ).write(by)
    ```

    如果安装失败请打开全局代理或者手动安装

    手动下载方法：

    - 点击 Preferences *>* Browse Packages…menu
    - 在packages文件夹的上层目录有一个Installed Packages目录
    - 点击链接下载 [Package Control.sublime-package](https://link.zhihu.com/?target=https%3A//packagecontrol.io/Package%20Control.sublime-package) 并且复制到 Installed Packages目录
    - 重启软件

2. 重启 Sublime Text 

3. 在`Perferences->package settings`中看到`package control`这一项，则安装成功。

4. `Ctrl+Shift+p`调出命令面板，选择`Install Package`并回车，在弹出的界面输入你想要安装的插件名字，回车安装即可。

5. 这里给出一些我用到的插件：

    * **ConvertToUTF8** ： 解决中文在sublime中乱码的问题

        如果出现乱码，只要在`File`里面找到`Encoding`并选择合适的编码模式即可，快捷键`Ctrl+Shift+C`。

    * **AStyleFormatter**：Sublime Text 3下的C/C++代码整理工具，好像还支持java

        设置在文件保存时的自动格式化：`Preferences —> Packge Settings ->sublimeAStyleFormatter ->(setting default) ` 

        修改 `autoformat_on_save` 的值为 `true` 即可

        关于格式化的样式也可以在这里修改，具体选择的中文解释：Here

    * **EasyClangComplete**：C++极简极速快速补全的插件

    * **SyncedSideBarBg**： 解决sidebar颜色和sublime背景不统一的问题

    * **SideBarEnhancements**： 侧边栏增强

6. 备注

    - 打开`sidebar`的方法：快捷键`Ctrl+k,Ctrl+b`或者`View`->`SideBar`->`Show SideBar`
    - 要使用`SideBarEnhancements`需要在`File`->`Open Folder`中打开一个文件夹，才能在sidebar中进行新建，删除等操作。
    - 插件的设置可以在：`Perferences->package settings`进行如用户配置，快捷键配置等设置



## 3. 配置C++编译环境

1. 下载编译器（已有的可以跳过此步)：`MinGW`或者`tdm gcc`二选一即可。推荐用`tdm gcc`。
    安装地址：[minGW](http://sourceforge.net/projects/mingw/)、[tdm-gcc](https://sourceforge.net/projects/tdm-gcc/?source=directory)。鉴于外国网站网速感人，下面丢两个我离线下载到百度网盘的版本,如失效可留言：[minGW 网盘地址](http://pan.baidu.com/s/1i4mC841)、[tdm gcc网盘地址](http://pan.baidu.com/s/1c1qQ0dq)。最新版本请以第1,2条下载地址的为准。

2. `tdm gcc`的安装过程非常简单，下载下来直接next,next,next就可以了。`MinGW`的安装过程稍稍麻烦一些。把之前的minGW安装器安装完之后打开，如图勾选

    ![勾选](https://raw.githubusercontent.com/RivTian/Blogimg/main/img/20210513171129.png)

    ，然后在`Installation`->`Apply Changes`。

3. 设置环境变量（win10环境）：`此电脑`->`属性`->`高级系统设置`->`环境变量`->`系统变量中的path`->添加你的编译器路径下的`bin目录`。比如我的minGW安装路径是：`C:\MinGW`,则添加`C:\MinGW\bin`。我的`tdm gcc`安装路径是: `C:\TDM-GCC-64`，则添加`C:\TDM-GCC-64\bin`。按你安装的编译器是什么来选择，这里注意查看这个bin目录下是否有`g++.exe,gcc.exe`等，有则是正常的路径。

4. 测试g++/gcc环境：在cmd中输入`g++ -v`

    ![gcc 验证](https://raw.githubusercontent.com/RivTian/Blogimg/main/img/20210513171218.png)

5. 在Sublime Text中编写cpp程序测试：
    在Sublime Text新建文件`demo.cpp`并按`ctrl+b`编译输出。测试成功如图所示

    ![运行测试](https://raw.githubusercontent.com/RivTian/Blogimg/main/img/20210513171321.png)

    （注：你可以在`tool`->`build-system`中选择编译系统。）

### 3.1 Sublime build System配置

很快你会发现，按上面的方法写出来的程序只能输出，不能输入。不像我们在IDE上面编写的程序那样，执行的时候会弹出CMD来接受输入，那么这时候我们需要自定义一个`Sublime build System`

1. 首先花一点点时间来了解一些基础知识：

    - `sublime build system`目录:

        ```json
         {
             "folders":
             [
                 {
                     "follow_symlinks": true,
                     "path": "."
                 }
             ],
             "build_systems":
             [
                 {
                     // 在选定 Tools | Build System | Automatic 时使用。Sublime Text使用这个 选择器自动为活动试图选择构建系统
                     "selector": "source.python",
                     // 在运行``cmd``前会切换到该目录。运行结束后会切换到原来的目录。
                     "working_dir":"D:/Developer/pyenv/python27",
                     // 输出``cmd``的编码。必须是合法的Python编码，缺省为``UTF-8``
                     "encoding":"UTF-8",
                     // 在环境变量被传递给``cmd``前，将他们封装成词典。
                     "env": {"PYTHONPATH":"."},
                     // 如果该选项为``true`` ，``cmd``则可以通过shell运行
                     "shell":false,
                     // 该选项可以在调用``cmd``前替换当前进程的’ PATH 。原来的’ PATH 将在运行后恢复。使用这个选项可以在不修改系统设置的前提下将目录添加到’ PATH 中
                     "path":"./Scripts;%PATH%",
                     // For Mac OS X and Linux and Unix
                     //"path":"/Users/user/work/myvirtualenv/bin:$PATH",
                     "name": "Run virtualenv python",
                     // 包括命令及其参数数组。如果不指定绝对路径，外部程序会在你系统的:const:PATH 环境变量中搜索。
                     "cmd": ["python.exe", "-u", "$file"],
                     // ``file_regex``选项用Perl的正则表达式来捕获构建系统的错误输出，主要包括四部分内容，分别是 file name*, line number, column number and error message. Sublime Text 在匹配模式中使用分组的方式捕获信息。file name 和 *line number*域是必须的
                     "file_regex": "^[ ]*File \"(...*?)\", line ([0-9]*)"
                     // variants 用来替代主构建系统的备选。如果构建系统的选择器与激活的文件匹配，变量的``名称``则 会出现在 Command Palette 中。
                     // line_regex: 当``file_regex``与该行不匹配，如果``line_regex``存在，并且确实与当前行匹配， 则遍历整个缓冲区，直到与``file regex``匹配的行出现，并用这两个匹配决定最终要跳转的文件 或行
        
                     // $file_path    当前文件所在路径, 比如 C:\Files.
                     // $file   当前文件的完整路径, 比如 C:\Files\Chapter1.txt.
                     // $file_name  当前文件的文件名, 比如 Chapter1.txt.
                     // $file_extension 当前文件的扩展名, 比如 txt.
                     // $file_base_name 当前文件仅包含文件名的部分, 比如 Document.
                     // $packages   Packages 文件夹的完整路径.
                     // $project    当前项目文件的完整路径.
                     // $project_path   当前项目文件的路径.
                     // $project_name   当前项目文件的名称.
                     // $project_extension  当前项目文件的扩展部分.
                     // $project_base_name  当前项目仅包括名的部分.
                 }
             ]
         }
        ```

    - g++编译命令：
        以`Test.cpp`为例，命令: `g++ Test.cpp`。生成默认为a.exe的文件，这个过程包含了编译和链接。再说下`-o`命令，`-o`命令表示输出的意思，gcc/g++命令是非常灵活的，你不指定输出的文件名的时候默认生成的是.exe文件。你要输出Test.exe的话可以用：`g++ -o Test.exe Test.cpp`。-o命令是输出的意思，这样就输出了`Test.exe`。命令:`-Wall`：输出所有警告信息

    - cmd命令
        `/c`:当执行完相应的命令(在命令提示符中)后自动退出命令提示符;`pause`:暂停命令,执行时会在命令行窗口显示“请按任意键继续. . .”并等待你按键。;当路径中的文件名/文件夹名字**含有空格**时，需要用`""`将这个文件名/文件夹名字括起来。

2. 开始配置：(这里的介绍来自 **Echo 学长**)
    `Sublime Text 3`->`Tools`->`Build System`->`New Build System`,输入以下内容：

    ```json
     {
          "encoding": "utf-8",
          "working_dir": "$file_path",
          "shell_cmd": "g++ -Wall \"${file}\" -o \"${file_path}/${file_base_name}\"",
          "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
          "selector": "source.c++",
    
          "variants":
          [
              {   
              "name": "Run",
                  "shell_cmd": "g++ -Wall \"${file}\" -o \"${file_base_name}\" && start cmd /c \"\"${file_path}/${file_base_name}\" & pause\""
              }
          ]
      }
    ```

    解释一下

    ```json
      "shell_cmd": "g++ -Wall "{file}\" -o \"{file_base_name}" && start cmd /c ""{file_path}/{file_base_name}" & pause"
    ```

    以`d:test.cpp`为例，这里相当于在cmd中执行了`g++ -Wall "D:\blank and blank\test and test.cpp" -o "D:\blank and blank/test and test"`，这句话的意思就是：**用g++编译器编译D:\blank and blank\test and test.cpp文件，并在译D:\blank and blank\目录下生成test and test文件**。之前都是直接从网上复制下来别人写的，结果都没有解决**文件名和文件路径中**包含空格的问题。我这里使用了**转义字符\和”**来把文件名和文件路径用双引号括起来，解决了空格的问题。

    ```
      start cmd /c \"\"${file_path}/${file_base_name}\" & pause\"
    ```

    类似地，这句话的意思就是：**启动cmd并输入命令”D:\blank and blank/test and test” & pause**,运行结果如图：

    ![运行结果](https://img-qjf.oss-cn-beijing.aliyuncs.com/%E8%BF%90%E8%A1%8C%E7%BB%93%E6%9E%9C.png)

    你可以在Sublime Text中的console中看到你在`ctrl+b`编译之后的执行的命令
    ![st console](https://img-qjf.oss-cn-beijing.aliyuncs.com/st_console.png)

3. 保存配置文件：
    按`ctrl+s`保存配置文件，文件名`xx.sublime-bulid`,这里后缀名一定要是`sublime-bulid`,然后你就可以在`Preference`->`Browse Package`->`user`文件夹中找到你保存的`xx.sublime-bulid`文件了。接下来在`tool`->`build-system`中选择xx（你刚刚保存的文件的文件名）。然后你就可以直接在Sublime Text中按`ctrl+b`或者`ctrl+shift+b`来编译运行了，程序都是以cmd的形式运行的。

## 4. 开启C++11

开启c++11只需要在刚刚的配置文件中加入`-std = c++11`，完整的配置如下:

```json
  {
    "encoding": "utf-8",
    "working_dir": "$file_path",
    "shell_cmd": "g++ -Wall -std=c++11 \"${file}\" -o \"${file_path}/${file_base_name}\"",
    "file_regex": "^(..[^:]*):([0-9]+):?([0-9]+)?:? (.*)$",
    "selector": "source.c++",

    "variants":
    [
        {   
        "name": "Run",
            "shell_cmd": "g++ -Wall -std=c++11 \"${file}\" -o \"${file_base_name}\" && start cmd /c \"\"${file_path}/${file_base_name}\" & pause\""
        }
    ]
}
```



## 5.Snippets

点击`Tools->New Snippet`之后，会新建一个文件，内容如下：

```xml
<snippet>
    <content><![CDATA[
Hello, ${1:this} is a ${2:snippet}. //这里输入你想要键入的代码～
]]></content>
    <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
    <!-- <tabTrigger>hello</tabTrigger> --> //这里把hello换成你想要使用的快捷键。
    <!-- Optional: Set a scope to limit where the snippet will trigger -->
    <!-- <scope>source.python</scope> --> //这里选择起作用的文件类型
</snippet>
```

设置完毕之后，`Ctrl+S`保存，默认会保存在User文件夹下，为了方便管理，不妨新建一个Snippet文件夹，后缀名为`.sublime-snippet`。保存好之后，就可以使用啦～ 用我自己的一个Snippet文件举例：

> 同样保存配置，文件名格式：`xx.init.sublime-snippet`。保存后也是可以在`Preference`->`Browse Package`->`user`文件夹中找到。
>
> Sublime Text自带的snippet可以在`sublime text安装目录`->`Packages`中找到。
> 以`C++`为例：
> `C:\Program Files\Sublime Text 3\Packages\c++.sublime-package`。
> 把后缀名改成：`c++.zip`即可用压缩文件打开。里面就有Sublime自带的`snippet`，用sublime text打开修改即可。修改完毕之后记得把`.zip`改回`.sublime-package`。

```xml
<snippet>
    <content>
<![CDATA[
#include<bits/stdc++.h>
using namespace std;
using ll = long long;
void solve() {
    $1
}
int main() {
    ios::sync_with_stdio(false), cin.tie(nullptr);
    solve();
    return 0;
}
]]>
    </content>
    <!-- Optional: Set a tabTrigger to define how to trigger the snippet -->
    <tabTrigger>qq</tabTrigger>
    <description>单组输入</description> //描述信息，可选
    <!-- Optional: Set a scope to limit where the snippet will trigger -->
    <scope>source.c, source.c++</scope>
</snippet>
```

这样在我敲下 `qq` 和 `Tab` 以后会自动将`qq`转换成我预先设定的代码。

> 关于代码里的 `$1` 代表转换代码以后输入光标所在位置，同样的可以设置  `$2` 为在按下 Tab 以后跳转的位置

## 5.备份

配置到现在，Sublime也算用的顺手了，要是换一台电脑都得这么捣鼓一下，肯定得疯。所以下面介绍一下如何同步自己的Sublime配置——只要备份`Packages\User`文件夹即可，里面的`sublime-settings`文件都保存了你的所有设置，更换电脑之后，只要恢复过去，打开Sublime的时候会自动检测，下载并安装你需要的包。

Windows下备份文件夹：

`C:\Users\yourusername\AppData\Roaming\Sublime Text 3\Packages\User`

## 6.后记

VS Code 很香，但作为一个追求简洁和敲代码敲个爽的蒟蒻决定放弃 VS Code（尽管效率真的很高）开始捡回 Sublime Text，不管不顾的决定把它改造成一个超级适应我的的IDE，所以，走你～～

## 7. 参考

* [ECHO 学长：Sublime Text 配置C语言开发环境](https://blog.qiujinfeng.com)
* 关于[快捷键修改](https://www.jianshu.com/p/d895f2238753)与[对照](https://blog.csdn.net/a772304419/article/details/79343374)

