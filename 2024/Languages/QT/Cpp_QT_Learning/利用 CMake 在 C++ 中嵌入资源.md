
在程序中嵌入资源是很常见的需求，资源可以是 GLSL Shader、LICENSE/EULA 文本、预定义公钥、图标图片等等。特定的平台和SDK里，会有特定的接口和工具来导入管理这些资源，比如 VC 的rc、QT 的 qrc，或者直接使用文件读写、dlopen/dlsym 来加载外部资源。对于C\C++而言，可以直接将资源转换成字符串字面量或数组，嵌入到程序中使用，本文即讲解如何在 cmake 构建中方便的嵌入各种资源。

嵌入文本资源可以在源文件中写字面量，但是字符转义和格式调整会比较麻烦，二进制资源可以使用 `xxd` / `hexdump` 工具来转成数组，但这增加额外依赖。其实利用 CMake 本身的指令语法，就可以 实现上述的资源嵌入。主要思路如下：

1.  将资源文件纳入工程中，利用 cmake 获取文件大小、文件名以及文件内容；
2.  利用 `string(MAKE_C_IDENTIFIER...)` 转换文件名为合法的 C 符号，后面将会使用该 符号作为资源字节数组名；
3.  利用 `string(REGEX...)` 转换文件内容为 HEX 编码，并导出到生成的 C 文件中；

再配合 [CMake 中使用 protobuf/protobuf-c](http://tisyang.github.io/post/2020-10-05-cmake-protobuf/) 文中的 `add_custom_target` 方法，可以实现只有资源文件修改后才会触发构建转换。

以示例工程为例，工程有3个文件，分别是

-   `a_demo_assets.assets`，项目资源文件，其内容是 GPLv2 License 文本；
-   `main.c`，项目主程序源码，内容是打印资源文件大小和内容；
-   `CMakeLists.txt`，构建脚本；

`main.c` 源码内容如下，其中 `a_demo_assets.assets.h` 是资源生成的头文件，文件名与 资源文件名相关，头文件就包含两个常量定义 `A_DEMO_ASSETS_ASSETS__SIZE` 和 `A_DEMO_ASSETS_ASSETS__DATA`，分别为资源字节大小和资源字节数组。

需要注意的是，转换的资源字节数组最后会多一个 `0x00` 字节，用来保证文本资源可以满足 C 风格字符串末尾截断。

```c
#include <stdio.h> 
#include "a_demo_assets.assets.h"

int main(int argc, char **argv){
	printf("a_demo_assets size=%u\n", A_DEMO_ASSETS_ASSETS__SIZE); 
	printf("a_demo_assets data=\n");
	fwrite(A_DEMO_ASSETS_ASSETS__DATA, A_DEMO_ASSETS_ASSETS__SIZE, 1, stdout); 
	return 0;
}
```

`CMakeLists.txt` 内容如下，这里是实现的核心地方，细节可以参见注释。

```cmake
cmake_minimum_required(VERSION 3.0)  
project(demo_cmake_embed)  
  
# 需要生成嵌入的文件  
file (GLOB EMBED_FILES  
"${CMAKE_CURRENT_SOURCE_DIR}/*.assets"  
)  
# 输出文件目录  
set (GEN_EMBED_OUTPUT_HDR_DIR  
"${CMAKE_CURRENT_BINARY_DIR}/gen_inc")  
set (GEN_EMBED_OUTPUT_SRC_DIR  
"${CMAKE_CURRENT_BINARY_DIR}/gen_src")  
file(MAKE_DIRECTORY ${GEN_EMBED_OUTPUT_HDR_DIR})  
file(MAKE_DIRECTORY ${GEN_EMBED_OUTPUT_SRC_DIR})  
  
# 依次处理文件  
foreach(input_src ${EMBED_FILES})  
# 配置输出文件名  
file(SIZE ${input_src} embed_file_size)  
get_filename_component(embed_file ${input_src} NAME)  
set(gen_embed_file "${GEN_EMBED_OUTPUT_SRC_DIR}/${embed_file}.c")  
set(gen_embed_file_header "${GEN_EMBED_OUTPUT_HDR_DIR}/${embed_file}.h")  
# 清空输出文件  
file(WRITE ${gen_embed_file} "")  
file(WRITE ${gen_embed_file_header} "")  
# for c compatibility  
string(MAKE_C_IDENTIFIER ${embed_file} token)  
# to upper case  
string(TOUPPER ${token} token)  
# read hex data from file  
file(READ ${input_src} filedata HEX)  
# convert hex data for C compatibility  
string(REGEX REPLACE "([0-9a-f][0-9a-f])" "0x\\1," filedata ${filedata})  
# append data to output file  
file(APPEND ${gen_embed_file}  
"const unsigned char ${token}__DATA[] = {\n${filedata}0x00\n};\n"  
"const unsigned long ${token}__SIZE = ${embed_file_size};\n")  
file(APPEND ${gen_embed_file_header}  
"extern const unsigned char ${token}__DATA[];\n"  
"extern const unsigned long ${token}__SIZE;\n")  
# 加入到生成文件列表  
list(APPEND GEN_EMBED_FILES  
${gen_embed_file}  
${gen_embed_file_header}  
)  
endforeach()  
  
add_custom_target(  
embed_gen_files  
DEPENDS ${GEN_EMBED_FILES}  
)  
include_directories(${GEN_EMBED_OUTPUT_HDR_DIR})  
  
add_executable(${PROJECT_NAME} main.c ${GEN_EMBED_FILES})  
add_dependencies(${PROJECT_NAME} embed_gen_files)  
#target_link_libraries(${PROJECT_NAME} m)
```