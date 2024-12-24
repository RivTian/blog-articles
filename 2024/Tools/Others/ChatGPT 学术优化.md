项目地址：[https://github.com/binary-husky/chatgpt_academic](https://github.com/binary-husky/chatgpt_academic)

API申请地址：[https://platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys)

## 直接运行 Mac

### 1、下载项目

```
j Github
git clone https://github.com/binary-husky/chatgpt_academic.git
cd chatgpt_academic
```

### 2. 配置API_KEY和代理设置

在`config.py`中，配置 海外Proxy 和 OpenAI API KEY，说明如下

```
1. 如果你在国内，需要设置海外代理才能够顺利使用 OpenAI API，设置方法请仔细阅读config.py（1.修改其中的USE_PROXY为True; 2.按照说明修改其中的proxies）。
2. 配置 OpenAI API KEY。你需要在 OpenAI 官网上注册并获取 API KEY。一旦你拿到了 API KEY，在 config.py 文件里配置好即可。
3. 与代理网络有关的issue（网络超时、代理不起作用）汇总到 https://github.com/binary-husky/chatgpt_academic/issues/1
```

![](https://cdn.jsdelivr.net/gh/RivTian/Blogimg/img/image-20230414120112590.png)

> P.S. 程序运行时会优先检查是否存在名为`config_private.py`的私密配置文件，并用其中的配置覆盖`config.py`的同名配置。因此，如果您能理解我们的配置读取逻辑，我们强烈建议您在`config.py`旁边创建一个名为`config_private.py`的新配置文件，并把`config.py`中的配置转移（复制）到`config_private.py`中。`config_private.py`不受git管控，可以让您的隐私信息更加安全。

### 3、安装依赖

--javascripttypescriptbashsqljsonhtmlcssccppjavarubypythongorustmarkdown

```
# （选择一）推荐
python -m pip install -r requirements.txt   
​
# （选择二）如果您使用anaconda，步骤也是类似的：
# （选择二.1）conda create -n gptac_venv python=3.11
# （选择二.2）conda activate gptac_venv
# （选择二.3）python -m pip install -r requirements.txt
​
# 备注：使用官方pip源或者阿里pip源，其他pip源（如一些大学的pip）有可能出问题，临时换源方法： 
# python -m pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple/
```

### 4、运行

--javascripttypescriptbashsqljsonhtmlcssccppjavarubypythongorustmarkdown

```
j chatgpt_academic
conda activate gptac_venv
python main.py
```

### 5. 测试实验性功能

--javascripttypescriptbashsqljsonhtmlcssccppjavarubypythongorustmarkdown

```
- 测试C++项目头文件分析
    input区域 输入 `./crazy_functions/test_project/cpp/libJPG` ， 然后点击 "[实验] 解析整个C++项目（input输入项目根路径）"
- 测试给Latex项目写摘要
    input区域 输入 `./crazy_functions/test_project/latex/attention` ， 然后点击 "[实验] 读tex论文写摘要（input输入项目根路径）"
- 测试Python项目分析
    input区域 输入 `./crazy_functions/test_project/python/dqn` ， 然后点击 "[实验] 解析整个py项目（input输入项目根路径）"
- 测试自我代码解读
    点击 "[实验] 请解析并解构此项目本身"
- 测试实验功能模板函数（要求gpt回答历史上的今天发生了什么），您可以根据此函数为模板，实现更复杂的功能
    点击 "[实验] 实验功能函数模板"
```

**其他部署方式请参考项目原文**