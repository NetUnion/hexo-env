---
title: 搭建 Deepin RISCV QEMU 环境
tags:
  - deepin
  - riscv
description: 本文简述了如何构建 Deepin risc-v 的 qemu 镜像
date: 2022-10-27 22:06:47
---


笔者最近开始了 Deepin risc-v 的软件包修复工作，俗话说，“工欲善其事，必先利其器”。在开始工作前，有必要准备好趁手的开发环境，本文简述了如何基于 PLCT 实验室的 Tarsier 项目提供的环境构建 Deepin risc-v 的 qemu 镜像。

## 下载 `rootfs.dde.ext4`

这是 Tarsier 项目的 deepin 小组提供的 rootfs，我们要基于此制作镜像。首先下载它：

```shell
wget https://mirror.iscas.ac.cn/deepin-riscv/deepin-stage1/rootfs.dde.ext4
```

## 新建 QEMU 镜像并分区、格式化

由于上文的镜像太小而且未正常分区，在 qemu 中使用会造成读写问题，就需要新建一个镜像：

```shell
qemu-img create -f raw deepin.raw 8G
```

下面，挂载这个新镜像并分区、格式化

```shell
sudo losetup -P /dev/loop0 deepin.raw
sudo fdisk /dev/loop0
sudo mkfs.ext4 /dev/loop0p1
```

注意，在 fdisk 交互式操作界面中，先按 `g` 新建 gpt 表，再按 `n` 新建分区。

最后，挂载新分区

```shell
sudo mkdir /mnt/deepin
sudo mount /dev/loop0p1 /mnt/deepin
```

## 拷贝文件到新镜像

挂载老镜像

```shell
sudo mkdir /mnt/deepin-old
sudo mount rootfs.dde.ext4 /mnt/deepin-old
```

接着，需要修改 `/mnt/deepin/etc/shadow` 以清除 root 密码：将该文件第一行修改为：

```text
root::19292:0:99999:7:::
```

并修改 `/mnt/deepin/etc/apt/source.list` 添加软件源：`deb [trusted=yes] https://mirror.iscas.ac.cn/deepin-riscv/deepin-stage1/ beige main`

拷贝所有文件到新镜像中：

```shell
sudo cp -r /mnt/deepin-old/* /mnt/deepin
```

解除两个镜像的挂载

```shell
sudo umount /dev/loop0p1
sudo umount rootfs.dde.ext4
sudo losetup -D
```

## 准备内核并启动镜像

由于该镜像中并不包含内核，需要一个内核来引导启动，笔者使用了 oerv 项目的内核，读者也可自行从 ubuntu 等发行版中提取 riscv 架构的内核。

```shell
wget https://repo.openeuler.org/openEuler-preview/RISC-V/Image/fw_payload_oe.elf
```

使用以下脚本启动

```shell
qemu-system-riscv64 \
  -nographic -machine virt \
  -smp 8 -m 8G \
  -device virtio-vga \
  -kernel fw.elf \
  -drive file=deepin.raw,if=none,id=hd0 \
  -object rng-random,filename=/dev/urandom,id=rng0 \
  -device virtio-rng-device,rng=rng0 \
  -device virtio-blk-device,drive=hd0 \
  -device virtio-net-device,netdev=usernet \
  -netdev user,id=usernet \
  -bios none \
  -device qemu-xhci -usb -device usb-kbd -device usb-tablet \
  -append "root=/dev/vda1 rw console=ttyS0"
```

## 之后……

由于之前清除了密码，先设置一个密码

```shell
passwd root
```

更新软件源

```shell
apt update && apt install neofetch
```

{% asset_img deepin-neofetch.png deepin neofetch %}
