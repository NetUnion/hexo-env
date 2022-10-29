---
title: 为 Debian RISC-V 打包 neofetch
tags:
  - riscv
  - PLCT
description: 本文以 neofetch 为例，介绍了 Debian RISC-V 的软件包打包流程
date: 2022-10-12 20:21:20
---


neofetch 是 Linux 下的一款用于查看系统配置信息的小工具，本文以此为例，介绍 Debian deb 软件包的打包流程。

## 前期准备工作

### 安装工具包

要处理 deb 软件包，要使用到 `dpkg` 和其他一些工具，可通过如下命令安装。

```shell
sudo apt-get install build-essential fakeroot dpkg-dev git devscripts debhelper
```

### 添加源码仓库源并获取源码

已经在 debian 官方软件源内的软件都可以通过 `apt source <pkg name>` 获取到源码，但要在 `/etc/apt/source.list` 中添加对应的地址。

```text
deb-src http://deb.debian.org/debian sid main
```

接着，更新软件源并获取源码（`apt source` 可以不加 `sudo`）

```shell
mkdir src && cd src && mkdir neofetch && cd neofetch
sudo apt update
apt source neofetch
```

源码包中通常会含有三个文件，拓展名分别是 `debian.tar.xz`、`dsc` 和 `orig.tar.xz`，分别是 debian 化的源码包、软件包的描述文件和原始的代码。如果正确安装了 `dpkg`，源码包会自动解压到 `<pkg name>-<version>` 目录中。否则就要用 `dpkg-source -x neofetch_7.1.0-4.dsc` 命令解压。

### 编译源码

通常来讲，为新架构移植软件时通常需要对现有代码进行修改，不过本文仅仅是做一个演示，而且 neofetch 的源码在 RISC-V 可以直接编译通过，这里笔者只改一下版本号。

首先，要更新 changelog，使用 `dch -v <version>-<revision>` 指定版本（如果报错提示需要邮箱，使用 `export EMAIL=<your email>` 设置环境变量），`dch` 会自动设置好对应的格式。这里笔者将版本号改为了 `7.1.0-4-david` 并简单地写了 `Update Version for david`。

接着，在 `neofetch-7.1.0` 目录下使用 `dpkg-buildpackage -us -uc -b` 打包软件包，这会在上级目录下生成几个文件，当前的文件夹结构如下

```text
.
├── neofetch-7.1.0-4
├── neofetch_7.1.0-4-david_all.deb
├── neofetch_7.1.0-4-david_riscv64.buildinfo
└── neofetch_7.1.0-4-david_riscv64.changes

1 directory, 3 files
```

### 安装新的 deb

要安装新的本地 deb 文件，只需要 `sudo apt install ./neofetch_7.1.0-4-david_all.deb` 即可，使用 `apt policy neofetch` 查看版本信息如图。

{% asset_img apt.png apt policy neofetch %}
