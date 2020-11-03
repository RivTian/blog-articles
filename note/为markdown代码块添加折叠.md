> 引言：
>
> 当我们在文章某段代码中写太多内容时，进行适当的内容折叠是非常有必要的。

请尝试点击一下Code👇

<details>
<summary>Code</summary>
<pre><code class="language-cpp">int found(int a[],int left,int right,int x) {
	while (left < right) {
		int mid = (right + left) >> 1;
		if (a[mid] < x) left = mid + 1;
		else right = mid;
	}
	return left;
}
</code></pre>
</details>


点开上面的Code就出现折叠的代码是不是很有意思？

其实实现这个功能的原理很简单、利用HTML的知识短短几行即可完成。

```HTML
<details>
<summary>Code</summary>
<pre><code class="language-cpp">
被折叠的代码块或者文章内容,内部不可以有空行
</code></pre>
</details>
```

利用上方的HTML样式即可轻松完成代码在markdown中实现折叠

核心是利用 `datails` 标签描述文档或文档某个部分的细节。

==与\<summary>标签== 配合使用可以为 details 定义标题。标题是可见的，用户点击标题时，会显示出 details。

> 好了，理解本质以后再看看最开始的 Code中的所有代码吧
>
> ```HTML
> <details>
> <summary>Code</summary>
> <pre><code class="language-cpp">int found(int a[],int left,int right,int x) {
> 	while (left < right) {
> 		int mid = (right + left) >> 1;
> 		if (a[mid] < x) left = mid + 1;
> 		else right = mid;
> 	}
> 	return left;
> }
> </code></pre>
> </details>
> ```
>
> 注意事项：
>
> 对于 <>的特殊符号应该用 `&lt;` `&gt;` 代替，不然会显示错误。(只需要修改 < 替换为 `&lt;` 即可）
>
> \<code>必须紧贴代码开头（避免多出首行），\</code>需单独一行（避免少了尾行）

![动态演示图来自网络](https://gitee.com//riotian/blogimage/raw/master/img/20201016201226.gif)

**参考：**

https://www.w3school.com.cn/tags/tag_details.asp