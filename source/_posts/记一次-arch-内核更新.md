---
title: 记一次 Arch 内核更新
date: 2022-03-22 01:37:46
tags: [Arch, Linux]
description: 今天尝试着把 Arch 的内核改成了 mainline 版本，但是开机时卡住了，究其原因，是显卡驱动的问题。
---

## 切换到 Mainline 内核

Mainline 内核要比 Arch 默认采用的 Stable 内核更新些，但是 Arch 的官方仓库并不提供现成的二进制文件。

要切换到 Mainline，我们有两种选择：

1. 从 AUR 下载源码并编译
2. 添加第三方软件库

第一种方式其实就是从 AUR 获取 PKGBUILD 文件，然后从 kernel.org 获取源码并自行编译，这样一来，大量的时间就耗费在了下载和编译上，如果没有特别的需求的话，笔者不建议选择这个方式。

第二种方式就简单很多了，AUR 上 `linux-mainline` 软件包的维护者为我们提供了现成的二进制文件，我们只需要将他的仓库添加到 `pacman` 的配置文件中即可。`archlinuxcn` 仓库也提供了这个软件包，笔者也更推荐后者（原因见下）。

在 `/etc/pacman.conf` 的末尾添加以下内容

```text
[miffe]
Server = https://arch.miffe.org/$arch/
```

然后添加该第三方仓库的密钥

```bash
pacman-key --recv-keys 313F5ABD
pacman-key --lsign-key 313F5ABD
```

最后更新仓库并安装二进制软件包

```bash
sudo pacman -Syu
sudo pacman -S linux-mainline linux-mainline-headers
```

别忘了重建 Grub 引导项目

```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```

## 进不了桌面环境了

安装完内核后，重启，发现开机卡在了 `/dev/nvme0p3: clean blocks` 上。但此时系统是正常启动了的，按下 `ctrl+alt+F2` 可进入新的 TTY。

在查了一些资料后，笔者发现原来在 [Arch Wiki](https://wiki.archlinux.org/title/NVIDIA_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#%E5%AE%9A%E5%88%B6%E5%86%85%E6%A0%B8) 中对于这个问题早有解答。

> 如果你使用的是一个定制的内核，可以通过 DKMS 来自动编译 Nvidia 内核模块。
> 安装 nvidia-dkms 软件包。 Nvidia 内核模块会在每次 Nvidia 或者内核更新的时候自动重新编译，这个功能由 DKMS pacman钩子实现。

简单来讲，就是 Nvidia 的内核模块和当前内核版本不匹配的过。理论上，每次更新内核时 `nvidia-dkms` 会自动帮我们更新内核模块，但由于种种原因，这次更新时这个玩意并没有生效……所以我们重新安装它来生成对应的内核模块。

```bash
sudo pacman -S nvidia-dkms
```

重启后，即可正常进入桌面环境。

## 卸载旧内核

在卸载旧内核时，笔者又遇到了问题：`nvidia` 包依赖于 `linux`，这就导致无法把稳定版内核删掉……虽然可以通过保留稳定版内核的方式暂时解决这个问题，不过如果稳定版内核更新了，nvidia-dkms 是否会重新生成内核模块导致无法再启动主线内核呢？笔者尚不清楚这个问题的答案，也决定不冒这个险。

经过一通查找，笔者发现在 `archlinuxcn` 第三方仓库中有 `nvidia-mainline` 这个软件包是依赖于 `linux-mainline` 的，解决了无法删除稳定版内核的问题。而且该仓库中也提供 `linux-mainline` 包，笔者就将之前的 `miffe` 源删除了，并替换成了这个，具体的导入过程可参考[这里](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)
