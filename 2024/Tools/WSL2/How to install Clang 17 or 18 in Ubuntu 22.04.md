#WSL2

This simple tutorial shows how to install the latest Clang compiler 17 and/or 18 in Ubuntu 20.04, Ubuntu 22.04, and Ubuntu 23.10.

Ubuntu includes several versions of Clang in its system repositories. But, it rarely builds newer releases into Ubuntu stable repositories.

You can easily install Clang 10, 11, 12, 13, 14, and 15 by running `sudo apt install clang-xx` (replace xx with major version number) command in terminal.

For the most recent 18 and 17, they are also easy to install via **the official apt repository**.

### Step 1: Download the Automatic installation script

The official [Clang repository](https://apt.llvm.org/), so far supports Ubuntu 18.04, Ubuntu 20.04, Ubuntu 22.04, Ubuntu 23.04, and Ubuntu 23.10. It has a script to make adding repository and installing Clang as easy as few Linux commands.

1. First, open terminal. When terminal opens, run command to **download the official installation script**:
    
```sh
wget https://apt.llvm.org/llvm.sh
```
    
    You may also use the script in Debian stable, though you may need to install `wget` first.
    
2. After downloading the script, **add executable permission** by running command:
    
```sh
chmod u+x llvm.sh
```
    

### Step 2: Use the script to install Clang

The script automate the process of adding the official apt repository, updating package cache, and installing specific Clang version into your system.

All this can be done by running a single command. For example, **install Clang-18**:

```sh
sudo ./llvm.sh 18
```

![](file://D:\Coding\GitLab\personal-docs\Docs\Tools\asserts\image-20240123084337145.png?lastModify=1706162574)

_Replace `18` with `17` for installing Clang-17, or even `19` if it’s already released when you see this tutorial_

During the process, it will ask to hit Enter to confirm adding the apt repository. Then, you may just wait until the process done.

### Step 3: Verify

If everything’s done successfully, just run `clang-xx --version` and/or `locate clang-xx` to verify.

![](file://D:\Coding\GitLab\personal-docs\Docs\Tools\asserts\image-20240123084639619.png?lastModify=1706162574)

### Uninstall

To remove the repository added by the script, just open terminal (Ctrl+Alt+T) and run command to remove the corresponding source file:

```sh
sudo rm /etc/apt/sources.list.d/archive_uri-http_apt_llvm_org_*.list
```

And, remove the repository key file via command:

```sh
sudo rm /etc/apt/trusted.gpg.d/apt.llvm.org.asc
```

Or, launch “Software & Updates” and remove source line and key from “Other Software” and “Authentication” tabs.

![Only Sample for user](file://D:\Coding\GitLab\personal-docs\Docs\Tools\asserts\remove-llvm-repo-600x340.webp?lastModify=1706162574)

To remove Clang packages (replace `18` accordingly), just run command:

```sh
sudo apt remove --autoremove clang-18 lldb-18 lld-18 clangd-18
```