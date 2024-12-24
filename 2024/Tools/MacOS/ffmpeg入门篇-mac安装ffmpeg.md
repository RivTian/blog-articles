
> http://www.manks.top/ffmpeg-install-mac.html

今天我们来看一下如何在mac上安装ffmpeg。

在mac上我们有3种方法可以安装ffmpeg。

第一种我们在[ffmpeg安装](http://www.manks.top/ffmpeg-install-static.html)一文中已经提到过了，直接下载静态库；

第二种是编译安装，不仅要安装xcode，还要安装很多的依赖库，还是让我们的mac省省心吧，忽略；

第三种就是我们今天要说的，通过Homebrew安装。

如果在此之前你通过 Homebrew 已经安装过 ffmpeg，可以执行命令 brew uninstall ffmpeg 先进行卸载。  

安装之前，我们先看下 Homebrew 的版本，这取决于我们采用哪种方式安装。终端执行 brew -v

```bash
➜ brew -v
Homebrew 4.1.12
```

如果你的电脑显示 command not found，请先执行下面的命令安装 Homebrew。

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
请注意，由于 Homebrew 的版本不同，我们接下来又有两种不同的操作方法。

##### 1、Homebrew 版本小于2.0

此时可以先看下 Homebrew 支持哪些配置选项，然后选择安装我们需要的options即可。

```bash
» brew options ffmpeg 
--with-chromaprint
 Enable the Chromaprint audio fingerprinting library
--with-fdk-aac
 Enable the Fraunhofer FDK AAC library
--with-fontconfig
 Build with fontconfig support
  ..............................
```

安装的时候，类似下面这样，你可以自行选择要安装哪些配置选项，这里建议大家把上面所有的option都加上

```bash
brew install ffmpeg --with-chromaprint --with-fdk-aac --with-xxx ......
```

##### 2、Homebrew 版本大于2.0

如果你的 Homebrew 版本大于2.0，通过 brew options ffmpeg，你会发现终端没有输出任何 options。这是怎么回事呢？

> ffmpeg官方说了 “Since v2.0, Homebrew does not offer options for its core formulae anymore. Users who want to build ffmpeg with additional libraries (including non-free ones) need to use so-called taps from third party repositories. These repositories are not maintained by Homebrew.”，大概意思是说从Homebrew2.0 开始，Homebrew 不再为其核心公式提供配置选项。所以想要扩展其他库的小伙伴需要选择第三方的存储库 homebrew-ffmpeg。当然你也可以直接选择 brew install ffmpeg，不带任何扩展库，但是这样安装的结果缺少很多编解码库，稍微复杂的命令都执行不了。  

首先执行 brew tap 命令  

```bash
brew tap homebrew-ffmpeg/ffmpeg
```

然后再看下这个仓库支持的 options   

```bash
brew options homebrew-ffmpeg/ffmpeg/ffmpeg
```

最后 install 的时候同样把这些 options 加上，类似下面这样

```bash
brew install homebrew-ffmpeg/ffmpeg/ffmpeg --with-chromaprint --with-fdk-aac --with-xxx ......
```

安装需要一定的时间，安装完之后直接在终端测试 ffmpeg 是否安装成功

```bash
~ » ffmpeg -version
ffmpeg version 4.1 Copyright (c) 2000-2018 the FFmpeg developers
  built with Apple LLVM version 10.0.0 (clang-1000.10.44.4)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/4.1 --enable-shared --enable-pthreads --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags='-I/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/include -I/Library/Java/JavaVirtualMachines/jdk1.8.0_251.jdk/Contents/Home/include/darwin' --host-ldflags= --enable-ffplay --enable-gpl --enable-libmp3lame --enable-libopus --enable-libsnappy --enable-libtheora --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libxvid --enable-lzma --enable-chromaprint --enable-frei0r --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libfdk-aac --enable-libfontconfig --enable-libfreetype --enable-libgme --enable-libgsm --enable-libmodplug --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-libopenh264 --enable-librsvg --enable-librtmp --enable-librubberband --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtesseract --enable-libtwolame --enable-libvidstab --enable-libwavpack --enable-libwebp --enable-libzmq --enable-opencl --enable-openssl --enable-videotoolbox --enable-libopenjpeg --disable-decoder=jpeg2000 --extra-cflags=-I/usr/local/Cellar/openjpeg/2.3.0/include/openjpeg-2.3 --enable-nonfree
  libavutil      56. 22.100 / 56. 22.100
  libavcodec     58. 35.100 / 58. 35.100
  libavformat    58. 20.100 / 58. 20.100
  libavdevice    58.  5.100 / 58.  5.100
  libavfilter     7. 40.101 /  7. 40.101
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  3.100 /  5.  3.100
  libswresample   3.  3.100 /  3.  3.100
  libpostproc    55.  3.100 / 55.  3.100
```

如果你用的仓库是 homebrew-ffmpeg/ffmpeg/ffmpeg，版本应该也是4.x的，唯一的不同可能多了几个options。

但是无论上面哪种情况，你都应该至少指定配置选项option，因为我们后面针对ffmpeg的操作会有些复杂。