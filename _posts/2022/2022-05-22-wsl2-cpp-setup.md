---
layout: post
title: c++ 开发环境搭建 (wsl2)
tags: [cpp]
author: Zhong Ling Xiao
---







# **[ 目录 ]**

1. 安装 wsl2
2. Ubuntu-20.04 软件源更新
3. 安装 zsh
4. 安装 oh-my-zsh
5. c++ (cmake) 开发环境搭建



------



# **1. 适用于 Linux 的 Windows 子系统 (wsl)**

[https://docs.microsoft.com/zh-cn/windows/wsl/](https://docs.microsoft.com/zh-cn/windows/wsl/)

## **1.1 安装**

```
$ wsl --list --online
```

```
以下是可安装的有效分发的列表。
请使用“wsl --install -d <分发>”安装。

NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
Ubuntu-16.04    Ubuntu 16.04 LTS
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS
```



安装 Ubuntu 20.04 LTS 版本

```bash
$ wsl --install -d Ubuntu-20.04
```



## **1.2 卸载**

检查已经安装的 wsl2 系统

```bash
$ wsl --list
```

```
适用于 Linux 的 Windows 子系统分发版:
Ubuntu-20.04
```



卸载 Ubuntu-20.04

```bash
$ wsl --unregister Ubuntu-20.04
```



------



# **2. Ubuntu-20.04 软件源更新**

默认情况下Ubuntu 的apt下载源是国外的仓库，我们可以将它的默认下载地址切换到国内环境下，例如阿里云的镜像地址。



## **2.1 备份**

```bash
$ sudo cp /etc/apt/sources.list /etc/apt/sources.list.bakup
```



## **2.2 修改**

```bash
$ sudo vim /etc/apt/sources.list
```

将 `sources.list` 文件内容替换成下面的

```
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```



## **2.3 更新**

```bash
$ sudo apt-get update
```



## **2.4 升级**

```bash
$ sudo apt-get upgrade
```



------

# **3. 安装 zsh**

[zsh](https://www.zsh.org/) 是 Z shell 的简称。用于交互式登录的 shell 及脚本编写的命令解释器，可以拓展一些功能丰富的第三方插件和主题，提高 shell 使用效率。



## **3.1 安装**

```bash
$ sudo apt-get install zsh
```



## **3.2 检查**

```bash
$ cat /etc/shells
```

```
# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
/bin/zsh
/usr/bin/zsh
```



## **3.3 设置为系统默认 shell**

```bash
$ chsh -s /bin/zsh
```



## **3.4 首次设置 zsh** 

我们需要重新开一个 Shell Session，第一次运行 zsh 时会进入如下的配置引导页面：

```
This is the Z Shell configuration function for new users,
zsh-newuser-install.
You are seeing this message because you have no zsh startup files
(the files .zshenv, .zprofile, .zshrc, .zlogin in the directory
~).  This function can help you with a few settings that should
make your use of the shell easier.

You can:

(q)  Quit and do nothing.  The function will be run again next time.

(0)  Exit, creating the file ~/.zshrc containing just a comment.
     That will prevent this function being run again.

(1)  Continue to the main menu.

--- Type one of the keys in parentheses ---
```

这里我们选择 `1` ，开始进行配置：

```
Attempting to extract information from manual pages...
Please pick one of the following options:

(1)  Configure settings for history, i.e. command lines remembered
     and saved by the shell.  (Recommended.)

(2)  Configure the new completion system.  (Recommended.)

(3)  Configure how keys behave when editing command lines.  (Recommended.)

(4)  Pick some of the more common shell options.  These are simple "on"
     or "off" switches controlling the shell's features.

(0)  Exit, creating a blank ~/.zshrc file.

(a)  Abort all settings and start from scratch.  Note this will overwrite
     any settings from zsh-newuser-install already in the startup file.
     It will not alter any of your other settings, however.

(q)  Quit and do nothing else.  The function will be run again next time.
--- Type one of the keys in parentheses ---
```

这里我们选择 `1`



------

# **4. 安装 oh-my-zsh**

由于 zsh 配置较为复杂，推荐使用配置管理工具来配置 zsh。下面介绍使用 [oh-my-zsh](https://ohmyz.sh/) 来修改 zsh 的主题和安装常用的插件。



## **4.1 安装**

```bash
$ wget https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh
```



给 `install.sh` 添加权限

```bash
$ chmod +x install.sh
```



执行`install.sh`

```bash
$ ./install.sh
```



我们发现速度非常慢，需要更换国内镜像。这里我们打开 `install.sh` 进行编辑

```bash
$ vim install.sh
```



找到如下部分：

```sh
# Default settings
ZSH=${ZSH:-~/.oh-my-zsh}
REPO=${REPO:-ohmyzsh/ohmyzsh}
REMOTE=${REMOTE:-https://github.com/${REPO}.git}
BRANCH=${BRANCH:-master}
```



将中间两行改为：

```sh
REPO=${REPO:-mirrors/oh-my-zsh}
REMOTE=${REMOTE:-https://gitee.com/${REPO}.git}
```



最后保存退出即可：`:wq`



## **4.2 配置 zsh**

```bash
$ vim ~/.zshrc
```



**主题修改**

```
ZSH_THEME="robbyrussell"
```

其中可以将 robbyrussell 改为 [Theme](https://github.com/ohmyzsh/ohmyzsh/wiki/Themes) 中的任意一种，比如 `ZSH_THEME="agnoster"` 。



**插件配置**

```
plugins=(git)
```

修改为

```
plugins=(git zsh-autosuggestions)
```



## **4.3 安装 autosuggestion 插件**

上面我们为 zsh 配置了 autosuggestions 插件，但这个插件并不是 zsh 自带的插件，需要下载安装。

```bash
$ git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```



安装完成后，重新更新配置。

```bash
$ source .zshrc
```



在编辑指令的时候，对于之前使用过的指令，按 `→` 即可快速补全。



------



# **5. c++ (cmake) 开发环境搭建**

我们需要在 Windows 环境中安装 VS Code。从官网下载最新版本，直接安装：[https://code.visualstudio.com/](https://link.zhihu.com/?target=https%3A//code.visualstudio.com/)

## **5.1 基础软件安装**

*build-essential*

构建基础包（`build-essential`）实际上是属于 Debian 的。在它里面其实并不是一个软件。它包含了创建一个 Debian 包（`.deb`）所需的软件包列表。这些软件包包括 `libc`、`gcc`、`g++`、`make` 等。构建基础包包含这些所需的软件包作为依赖，所以当你安装它时，你只需一个命令就能安装所有这些软件包。

```bash
$ sudo apt-get install build-essential
```



*GDB*

全称 GNU symbolic debugger，是 Linux 下常用的程序调试器。

```bash
$ sudo apt-get install gdb
```



*CMake*

是一种优秀的跨平台的构建系统，类似于 Makefile，是专为 C/C++ 开发的一套构建系统。

```bash
$ sudo apt install cmake
```



*Remote-WSL*

1. 在 Windows 环境中安装 VS Code。[官网](https://code.visualstudio.com/)下载最新版本并安装
2. 在 VS Code 的插件市场，搜索 `remote-wsl` 插件，然后安装



## **5.2 测试**

创建一个文件夹 test ，然后进入文件夹。

```bash
$ mkdir test
$ cd test
```



在文件夹中启动 vscode

```bash
$ code .
```



新建 main.cpp 文件，完成代码

```cpp
#include <iostream>

using namespace std;
int main() {
    cout << "Hell World!" << endl;
}
```



新建 CMakeLists.txt 文件，完成代码

```
cmake_minimum_required (VERSION 3.10)
project (test)
add_executable(test main.cpp)
```



构建应用程序

```bash
$ cmake .
$ make
```



测试应用程序

```bash
$ ./test
```

```
Hell World!
```

