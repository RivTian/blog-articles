
#CMake 

warning C4819：该文件包含不能在当前代码页(936)中表示的字符。请将该文件保存为 Unicode 格式以防止数据丢失

```cmake
if (win32)
	add_complie_options(/W4)
	add_complie_options(/wd4819)
endif()
```