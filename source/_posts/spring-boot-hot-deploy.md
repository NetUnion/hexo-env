---
title: Spring Boot 的热部署配置
date: 2022-07-04 13:17:51
tags: [Java, Spring Boot, IntelliJ IDEA]
description: 在编写 Spring Boot 项目时，我们通常会希望能够做到「不停机」的部署模式，本文说明了如何配置 pom.xml 和 Idea IDE 以实现热部署。
---

## 添加 devtools 依赖并配置启用

Spring 为开发者提供了一个名为 spring-boot-devtools 的模块来使 Spring Boot 项目支持热部署，我们首先将这一模块加入到 pom.xml 的依赖项中。

```xml
<project>
    ...
    <dependencies>
        ...
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <version>2.7.0</version>
        </dependency>
    </dependencies>
    ...
</project>
```

配置并启用插件:

```xml
<project>
    ...
    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>2.7.0</version>
                <configuration>
                    <fork>true</fork>
                </configuration>
            </plugin>
        </plugins>
    </build>
    ...
</project>
```

## 设置 Idea 运行时自动编译

首先启用 Idea 的自动编译功能：在**文件**-**设置**-**构建、 执行、 部署**-**编译器**中启用「自动构建项目」。

{% asset_img settings.png 启用自动构建 %}

但仅仅这样设置的话，Idea 仍然不能在项目运行时自动构建项目，我们需要在**文件**-**设置**-**高级设置**中启用「即使开发的应用程序当前正在运行，也允许自动 make 启动」。这里的设置和网络上很多老版本的教程不同，不是在注册表中而是在高级设置中开启了。

{% asset_img advset.png 高级设置选项 %}
