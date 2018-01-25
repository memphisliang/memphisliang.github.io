---
layout: post
title: 在Debian8上编译安装Python3
---

# 在 Debian 8 上编译安装 Python 3

在本篇博文中，会教大家如何在Debian系统上编译安装Python 3，在过程中还会简单解释一些编译常识。

从源码编译安装一个软件在Linux系统中是十分常见的一件事，无论是在Debian（官方仓库中的软件太旧），CentOS（软件不全），还是ArchLinux（软件更新太快，想用旧版本）中都需要编译软件，所以了解如何编译软件对使用Linux十分有用。

## 编译前准备

在任何一个Linux系统中编译一个软件，都需要一些基本开发软件的支持，如gcc，g++编译器，链接器等，还需要待编译软件所需要的支持库。

在编译Python之前，我们需要安装`libssl-dev`和`build-essential`两个库，`build-essential`库提供了gcc等编译软件，而`libssl-dev`库是编译Python所需的库，如果没有这个库，Python 3的编译会失败。

```
# apt-get install build-essential libssl-dev
```

有时除了安装必须的库之外，还有一些额外的库会被建议安装，这些库不是软件必须的，但安装这些库会让软件提供更强大的功能或效率，所以建议大家安装。

```
# apt-get install make zlib1g-dev libbz2-dev libsqlite3-dev
```

上面这些库是编译 Python 3 之前建议建议大家安装的，除了可以让Python提供更多的功能之外，有时还会避免以后使用`pip`等工具安装其他第三方库时出现一些莫名其妙的错误。

## configure设置

安装完必须的库之后就是配置编译选项了，软件提供的源码中通常包含一个`configure`脚本，我们可以用它来配置一些编译选项，如软件安装的位置，优化选项等。

```
$ ./configure --help
```

可以用上面的命令来参看脚本提供的配置选项，编译软件中最常用的编译选项是`--prefix`选项，可以用这个选项来决定软件安装的位置。

```
$ ./configure --prefix=/usr/local/python3
```

在这里我们把Python安装在`/usr/local/python3`这个目录中，这样会方便我们以后管理这些自己编译的软件。

如果不使用`--prefix`选项，软件通常会自动安装在`/usr/local`目录下，并按文件类型分布在`/usr/local/bin`和`/usr/local/lib`等多个目录中，之后我们要删除这个软件就会十分困难。

使用`--prefix`选项把软件的所有文件安装在一个单独的目录中，这里是`/usr/local/python3`，之后我们想删除这个软件，只需要直接删除`python3`这个目录就可以了，大大方便了我们之后的管理。

## 安装

配置完就是编译了，我们使用`make`命令来编译软件，它会自动把源码编译成可执行文件，注意，这个命令不需要root权限，因为它只是生成可执行文件，但还没有安装到系统中。

```
$ make
```

然后是安装，因为要存取`/usr/local`目录，所以需要root权限。

```
# make install
```

## 修改PATH环境变量

经过以上的编译安装，软件已经在我们的系统中了，但我们还需要把软件路径添加到环境变量$PATH中，不然系统就搜索不到这个命令。

在`~/.bashrc`文件的最后添加以下代码，以使每次登陆设置都会生效。

```
PATH="/usr/local/python3/bin:$PATH"
```

注意： 路径中是`/usr/local/python3/bin`，而不是`/usr/bin/python3`，因为`python3`这个可执行文件是在`bin`目录下的，如果设置为`/usr/local/python3`，它只会搜索`python3`目录，而不会搜索`python3`目录下的`bin`目录，就会找不到命令。

```
$ python3 -V
```

使用这个命令来测试是否成功，这个命令应该会输出当前Python的版本信息。

## 删除软件

之后，如果我们想要删除python3，只需要删除`/usr/local/python3`这个安装时设置的目录，然后在把$PATH中的路径删除就可以了。

或者我们可能需要重编译软件，就只需要覆盖这个目录就好。
