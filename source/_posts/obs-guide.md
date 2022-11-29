---
title: OBS 简易使用指南
tags:
  - PLCT
description: 本文简述了 OBS 自动构建系统的使用方法（以 WebUI 为主）
date: 2022-11-29 16:32:59
---


## 什么是 OBS

OBS(Open Build Service) 是一个用于自动化源代码构建的通用系统，能为各种操作系统和硬件架构创建映像和安装包，也是软件包源代码托管平台。OBS 提供网页管理界面，也可在终端环境下可使用 `osc` 命令行工具进行相关操作。关于命令行操作，笔者也尚未实践，暂且按下不表。

## OBS WebUI 的使用实例

OBS 的使用和 Git 很是类似，在注册账号后，每个人会有一个 home project，用户既可以在这个项目下添加软件包，也可新建子项目（subproject）或者其他项目。

在有了一个 project 后，我们就可以为其添加 Repositories 了，这里的 Repositories 主要是约定了软件包的构建方法和架构，用户可以从发行版添加 repo，也可从其他项目中添加。

{% asset_img 211c0c192bb0f67ff01e664c52f91f4df312f71a3b8515b79de0c2f959f19bd0.png Repositories 页面 %}

### 软件包的不同状态

对于某个特定的软件包可能有如下几种状态：

- `success`：构建成功
- `failed`：构建失败
- `building`：正在构建
- `unresolvable`：软件有未解决的依赖项，这里的依赖项既包括本项目中成功构建的软件包，也包括引入的其他仓库中的软件包。
- `blocked`：有时某一软件包所依赖的一些软件包可能正处于 rebuild 状态（比如有版本更新），这时该软件包就会被 blocked，要等其他软件包构建成功后才能继续构建
- `broken`：通常是构建描述文件丢失（注：缺少源码包而有 dsc 文件时通常会显示为 building 状态）
- 其他：通常为开始构建前的一些等待状态

{% asset_img 67e01e6d1548fef13a8f9c1d3b5da3cbdfdcde374dd5574e86f466d1f4f17a2b.png 构建状态 %}

### 修包

当某个软件包处于 `failed` 或者 `unresolvable` 状态时，用户就需要修复这一软件包或者修复其依赖（可能是构建失败或者缺失软件包）。

对于 `failed` 的情况，就需要给对应的软件包打 “补丁”，关于这部分内容，读者可参考 Gui-Yue 同学的[文章](https://www.cnblogs.com/Gui-Yue/articles/16913125.html)。

对于 `unresolvable` 而且是依赖缺失的情况，就需要用户自行新建软件包

{% asset_img f5bfd8161b4c286896e70af6cd4d77e244d314d1ca9fe8d5f66d01b719f3c4b0.png 新建软件包 %}

建好后，只需要将对应的源码和/或构建描述文件上传即可，OBS 会自动开始构建

{% asset_img 5b4190b4e891eb1fa4acf34036852d5aca595a9a9a439475d6a2f34dc7919efb.png 上传源码 %}

当然，对于 Deepin 的 RISC-V 发行版来说，用户通常可以在 Deepin 的官方仓库找到对应的源码包，而对于其他发行版或者新软件包而言，就需要用户自行创建符合 Debian 或特定发行版规范的源码包和描述文件了。
