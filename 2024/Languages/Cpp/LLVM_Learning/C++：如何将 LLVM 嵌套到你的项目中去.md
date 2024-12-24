
> Refrences
> 
> * [Embedding LLVM in your project](https://llvm.org/docs/CMake.html#embedding-llvm-in-your-project)


- IDE: Clion
- LLVM

```cmake
cmake_minimum_required(VERSION 3.9)  
project(clang_demo)  
  
find_package(LLVM REQUIRED CONFIG)  
  
message(STATUS "Found LLVM ${LLVM_PACKAGE_VERSION}")  
message(STATUS "Using LLVMConfig.cmake in: ${LLVM_DIR}")  
  
set(CMAKE_CXX_STANDARD 17)  
  
# Set your project compile flags.  
# E.g. if using the C++ header files  
# you will need to enable C++11 support  
# for your compiler.  
  
include_directories(${LLVM_INCLUDE_DIRS})  
add_definitions(${LLVM_DEFINITIONS})  
  
add_executable(clang_demo main.cpp)  
# Find the libraries that correspond to the LLVM components  
# that we wish to use  
llvm_map_components_to_libnames(llvm_libs support core irreader)  
  
# Link against LLVM libraries  
target_link_libraries(clang_demo ${llvm_libs})
```

```cpp
// main.cpp
#include "llvm/IR/LLVMContext.h"  
#include "llvm/IR/Module.h"  
  
using namespace llvm;  
  
static std::unique_ptr<LLVMContext> my_Context;  
static std::unique_ptr<Module> my_Module;  
  
static void InitializeModule() {  
    my_Context = std::make_unique<LLVMContext>();  
    my_Module = std::make_unique<Module>("Hello Modlue", *my_Context);  
}  
  
int main(int argc, char *argv[]) {  
    InitializeModule();  
    my_Module->print(outs(), nullptr);  
  
    return 0;  
}
```

**不使用 cmake 构建项目，可执行以下命令**

```bash
clang++ -g mai.cpp `llvm-config --cxxflags --ldflags --system-libs --libs core` -o hello_world
```

获取 LLVM 配置信息

```bash
llvm-config --cxxflags --ldflags --system-libs --libs core
```