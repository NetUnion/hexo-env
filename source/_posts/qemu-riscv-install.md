---
title: 使用 QEMU 搭建 RISC-V 开发环境
tags:
  - riscv
  - PLCT
description: 本文比较详细的描述了如何在 Archlinux 下安装 QEMU 并搭建 RISC-V 开发环境
date: 2022-10-12 17:42:51
---


RISC-V 是近年来新兴起的一种开源的精简指令集处理器架构，QEMU 则是一款开源的计算机模拟器，可以帮助我们快速的搭建一个 RISC-V 的开发环境。

## 安装 QEMU

参考 [Archlinux Wiki](https://wiki.archlinux.org/title/QEMU)，我们需要安装 `qemu-emulators-full` 和 `qemu-base` 软件包。

```bash
sudo pacman -S qemu-base qemu-emulators-full
```

## 安装 RISC-V

### 获取 RISC-V 镜像

Debian 团队和 Ubuntu 团队都提供了对应系统的 RISC-V 镜像，笔者这里采用了 Debian 的镜像。

```shell
mkdir debian-rv64 && cd debian-rv64
wget "https://gitlab.com/api/v4/projects/giomasce%2Fdqib/jobs/artifacts/master/download?job=convert_riscv64-virt" -O debian-rv64.zip
unzip debian-rv64.zip && rm debian-rv64.zip
cd artifacts
```

### 获取 OpenSBI 和 U-Boot 二进制文件

OpenSBI 用于将 RISC-V 平台在 M-Mode 和 S-Mode 中切换，对于 RISC-V 系统的具体的启动模式和流程本文不再赘述，读者可参考 [An Introduction to RISC-V Boot flow: Overview, Blob vs Blobfree standards](https://crvf2019.github.io/pdf/43.pdf) 这篇文章。OpenSBI 已经包含在了 `qemu-emulators-full` 软件包中，位于 `/usr/share/qemu/opensbi-riscv64-generic-fw_dynamic.bin`。

U-Boot 则是一款开源的启动引导器（Bootloader），可用于多种系统架构的引导，AUR 中已经提供了相关的二进制包 `u-boot-qemu-bin`，安装完成后，可以在 `/usr/share/u-boot-qemu-bin/qemu-riscv64_smode` 中找到 `uboot.elf`，将该文件拷贝到当前目录中。

```bash
sudo cp /usr/share/u-boot-qemu-bin/qemu-riscv64_smode/uboot.elf .
```

### 创建备份镜像

笔者建议给当前的系统创建一个 Overlay 镜像以便于在系统出错时快速恢复初始状态。

```bash
qemu-img create -o backing_file=image.qcow2,backing_fmt=qcow2 -f qcow2 overlay.qcow2
```

### 启动虚拟机

可以通过下列命令启动虚拟机，建议保存为 `.sh` 文件。

```bash
qemu-system-riscv64 \
    -machine virt \
    -cpu rv64 \
    -m 8G \
    -device virtio-blk-device,drive=hd \
    -drive file=image.qcow2,if=none,id=hd \
    -device virtio-net-device,netdev=net \
    -netdev user,id=net,hostfwd=tcp::62222-:22 \
    -bios /usr/share/qemu/opensbi-riscv64-generic-fw_dynamic.bin \
    -kernel ./uboot.elf \
    -object rng-random,filename=/dev/urandom,id=rng \
    -device virtio-rng-device,rng=rng \
    -append "root=LABEL=rootfs console=ttyS0" \
    -nographic
```

这里我们给虚拟机分配了 8G 内存，并转发了 22 端口到宿主机的 62222 端口以便于 SSH 访问。这个系统镜像可以使用 root/root 或者 debian/debian 登录。

至此，RISC-V 的开发环境便搭建完成了。

{% asset_img uname.png 查看内核版本 %}

### 安装 neofetch

apt 安装即可

```bash
apt update
apt install neofetch
```

{% asset_img neofetch.png neofetch 界面 %}
