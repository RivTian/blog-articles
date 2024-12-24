
#Hash #Software

可直接拖放、复制粘贴、添加文件或文件夹的方式来批量计算Hash，操作简便、体积小、免费。这篇来介绍他的汉化和其它一些功能设置。

[Here](https://blog.csdn.net/NDASH/article/details/110440995)

### 高级功能-命令行选项

| / file <文件名\| 文件夹\| 通配符>     | 指定要哈希的文件名，文件夹或通配符。                                                                                                                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| / files <文件名> <文件名> <文件名> …  | 指定要哈希的多个文件名，文件夹或通配符。                                                                                                                                                                                   |
| /文件夹<文件夹>                    | 指定一个文件夹及其所有子文件夹。                                                                                                                                                                                       |
| / wildcard <全路径通配符> <子文件夹深度> | 指定具有完整路径的通配符（例如：c：\ folder \ *。exe）和要扫描的子文件夹的深度。对于参数：0 =无子文件夹，1 =一级子文件夹，2 =二级子文件夹，依此类推… 1000 =无限数量的子文件夹。                                                                                               |
| / virustotal <文件名>           | 计算指定文件的哈希，然后在VirusTotal网站中将其打开。                                                                                                                                                                        |
|                              | 允许您打开/关闭指定的哈希类型（0 =关闭，1 =打开）。例如： HashMyFiles.exe / MD5 1 / SHA1 1 / SHA256 0                                                                                                                           |
| / stext <文件名>                | 将哈希列表保存到常规文本文件中。                                                                                                                                                                                       |
| / stab <文件名>                 | 将哈希列表保存到制表符分隔的文本文件中。                                                                                                                                                                                   |
| / stabular <文件名>             | 将哈希列表保存到表格文本文件中。                                                                                                                                                                                       |
| / shtml <文件名>                | 将哈希列表保存到HTML文件（水平）中。                                                                                                                                                                                   |
| / sverhtml <文件名>             | 将哈希列表保存到HTML文件（垂直）中。                                                                                                                                                                                   |
| / sxml <文件名>                 | 将哈希列表保存到XML文件。                                                                                                                                                                                         |
| / scomma <文件名>               | 将哈希列表保存到以逗号分隔的文件中。                                                                                                                                                                                     |
| / sort <列>                   | 此命令行选项可与其他保存选项一起使用，以按所需列进行排序。如果未指定此选项，则列表将根据您在用户界面中进行的最后排序进行排序。参数可以指定列索引（第一列为0，第二列为1，依此类推）或列名，例如“ Filename”和“ Identical”。如果要按降序排序，可以指定“〜”前缀字符（例如：“〜Identical”）。如果要按多列排序，可以在命令行中输入多个/ sort。             |
| / nosort                     | 当您指定此命令行选项时，列表将被保存而不会进行任何排序。                                                                                                                                                                           |
| /保存直接                        | 将哈希列表保存在SaveDirect模式下。与其他保存命令行选项（/ scomma，/ stab，/ sxml等）一起使用时，使用SaveDirect模式时，哈希行直接保存到磁盘上，而无需将它们加载到内存中第一。这意味着，只要您有足够的磁盘空间来存储保存的文件，就可以将具有大量哈希的列表保存到磁盘中，而不会出现任何内存问题。此模式的缺点：无法使用/ sort命令行选项根据选择的列对行进行排序。 |

示例：

```sh
HashMyFiles.exe / file“ c：\ temp \ *。zip” / shtml“ c：\ temp \ 1.html”
HashMyFiles.exe / file“ d：\ temp \ myfile.zip” / stab“ d：\ temp \ myfile.txt“
HashMyFiles.exe / file” d：\ my files“
HashMyFiles.exe / files” c：\ temp \ *。zip“” c：\ temp \ 1234.exe“” c：\ temp \ Hello .exe“ / shtml” c：\ temp \ 1.html“
HashMyFiles.exe /文件夹” c：\ temp“ / shtml” c：\ temp \ 1.html“
HashMyFiles.exe /文件夹” c：\ temp“ / shtml“ c：\ temp \ 1.html” / sort“相同” / sort“文件名”
HashMyFiles.exe / folder“ c：\ temp” / shtml“ c：\ temp \ 1.html” /
sort〜1 HashMyFiles。 exe /通配符“ c：\ temp \ *。zip” 1 / shtml“ c：\ temp \ 1。 html“
HashMyFiles.exe / SaveDirect / folder” c：\ temp“ / scomma” c：\ temp \ 1.csv“
```

