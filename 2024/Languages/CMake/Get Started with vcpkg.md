#vcpkg

## Prerequisites

- Git
- g++ >= 6
- Package: zip, unzip

### Install vcpkg

Installing vcpkg is a two-step process: first, clone the repo, then run the bootstrapping script to produce the vcpkg binary. The repo can be cloned anywhere, and will include the vcpkg binary after bootstrapping as well as any libraries that are installed from the command line. It is recommended to clone vcpkg as a submodule to an existing project if possible for greater flexibility.

<span style="font-size: 20px;"><b>Step 1: Clone the vcpkg repo</b></span>


```sh
git clone https://github.com/Microsoft/vcpkg.git
```

Make sure you are in the directory you want the tool installed to before doing this.

<span style="font-size: 20px;"><b>Step 2: Run the bootstrap script to build vcpkg</b></span>

- Need package: zip, unzip 

```sh
yay -S zip unzip
./vcpkg/bootstrap-vcpkg.sh
```

Install libraries for your project

```sh
vcpkg install [packages to install]
```

if install fail because of proxy，please run the code, relative record: [Here](obsidian://open?vault=Obsidian_Notes&file=Tools%2FWSL2%2FWSL2%20%E8%AE%BE%E7%BD%AE%E4%B8%80%E9%94%AE%E4%BB%A3%E7%90%86)

```sh
cd 
source ~/.proxyrc
```
## Using vcpkg with CMake

In order to use vcpkg with CMake outside of an IDE, you can use the toolchain file:

```sh
cmake -B [build directory] -S . -DCMAKE_TOOLCHAIN_FILE=[path to vcpkg]/scripts/buildsystems/vcpkg.cmake
```

Then build with:

```sh
cmake --build [build directory]
```

With CMake, you will need to find_package() to reference the libraries in your Cmakelists.txt files.

Browse the [vcpkg documentation](https://vcpkg.io/en/docs/README.html) to learn more about how to enable different workflows.