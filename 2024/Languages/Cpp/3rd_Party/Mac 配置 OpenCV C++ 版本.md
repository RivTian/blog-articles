
今天紀錄一下如何在 Mac 上安裝 OpenCV for C++ 開發環境

使用 Brew 安装，pkgconfig 检测，2023.5.17

Mac x86 ( Intel ) , Mac M1 ( Apple silicon ) 和 Ubuntu 也適用  
此筆記用 OpenCV 4.7.0_4 版本做範例

## 1. 安装 cmake 与 pkg-config

如果您的 **Mac** 沒有 cmake, pkg-config 請先使用 brew 安裝 ( [brew 官網](https://brew.sh/) )

```bash
brew install cmake pkg-config
```

如果您是 **Ubuntu** 使用者參考以下指令

```bash
sudo apt-get update  
sudo apt-get install -y cmake build-essential git pkg-config
```

後面步驟不管是 MacOS / Ubuntu 都是相同

## **2. 由 OpenCV 官方 GitHub clone Source Code**

( 官方 github : https://github.com/opencv/ )

選一個你會放置 OpenCV 程式的資料夾位置  
然後 clone OpenCV source code

以下用 4.5.4 為例 :

```bash
cd 某个文件夹下
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

另外很多功能是在 opencv_contrib 這 Repository 中, 建議一起安裝 ( 上 2 行 )

## 3. Checkout 成 4.5.4 版

```bash
cd opencv && git checkout 4.5.4  
cd ../  
cd opencv_contrib && git checkout 4.5.4  
cd ../
```

## 4. 建立 build folder

mkdir build_opencv_4.5.4

![](https://miro.medium.com/v2/resize:fit:1060/format:webp/1*1EhhAJGO05uqqRYpjh0Xaw.png)

然後記得進入該資料夾 ( [感謝網友修正](https://medium.com/@michael31703/seachaos%E6%82%A8%E5%A5%BD-%E5%9C%A8%E7%AC%AC4%E6%AD%A5%E9%A9%9F%E6%98%AF%E4%B8%8D%E6%98%AF%E5%B0%91%E4%BA%86cd-build-opencv-4-3-0%E5%91%A2-%E5%A6%82%E6%9E%9C%E6%88%91%E4%B8%8D%E5%9C%A8build-opencv-4-3-0%E7%9B%AE%E9%8C%84%E4%B8%8B%E5%9F%B7%E8%A1%8Ccmake%E6%9C%83%E6%9C%89%E5%95%8F%E9%A1%8C-249c5714f6b2) )

```bash
cd ./build_opencv_4.5.4
```

## 5. 使用 cmake

```bash
cmake CMAKE_BUILD_TYPE=Release \  
  -DBUILD_EXAMPLES=ON \  
  -DOPENCV_GENERATE_PKGCONFIG=ON \  
  -DCMAKE_INSTALL_PREFIX=/usr/local/ \  
  -DOPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules \  
  ../opencv
```

## 6. Make install

```bash
make -j12  
make install
```

此部分和安裝其他軟體差不多，編譯上會花一點時間

**_Ubuntu 使用者 :_**  
視情況可能需要 sudo 指令 ，例如以下錯誤

```bash
-- Install configuration: "Release"  
CMake Error at cmake_install.cmake:41 (file):  
  file cannot create directory: /usr/local/share/licenses/opencv4.  
  Maybe need administrative privileges.
```

改用使用 sudo make install 應可解決

## 7. 測試

可以到 opencv 的官方範例檔案中進行編譯測試例如以下指令 :

```bash
cd ../opencv/samples/cpp  
g++ -std=c++14 -ggdb opencv_version.cpp -o /tmp/opencv_version `pkg-config --cflags --libs opencv4`
```

編譯 C++ 通過就可以執行程式看看:

```bash
/tmp/opencv_version
```

![OpenCV 範例程式執行成功](https://miro.medium.com/v2/resize:fit:360/format:webp/1*w4QqztUhd1EmlRpr2xd-rw.png)

# Brew 安装

```bash
brew install opencv
pkg-config --cflags --libs opencv
```

CLion里面去配置 创建一个新项目，修改cmakeLists

```cpp
cmake_minimum_required(VERSION 3.25)  
project(demo)  
  
set(CMAKE_CXX_STANDARD 17)  
  
find_package(OpenCV REQUIRED)  
  
include_directories(${OpenCV_INCLUDE_DIRS})  
  
  
add_executable(demo main.cpp)  
target_link_libraries(demo ${OpenCV_LIBS})
```

### 代码测试

```cpp
#include <iostream>  
#include <opencv2/opencv.hpp>  
#include <string>  
  
using namespace cv;  
using namespace std;  
  
void ImageHold(string str) {  
	Mat image = imread(str);  
	  
	imshow("test_opencv", image);  
	waitKey(0);  
}  
  
int main() {  
	std::cout << "Hello, World!" << std::endl;  
	string path = "/Users/koshkaaaa/Documents/File/Fav/";  
	string str = "6F23E4AF-A2BC-4E2E-958E-C24C3F1AEF0E_1_105_c.jpeg";  
	ImageHold(path + str);  
	return 0;  
}
```

以上為簡單 OpenCV C++ 安裝筆記
