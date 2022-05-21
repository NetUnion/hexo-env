---
title: 在 IntelliJ IDEA 中使用 Maven 安装依赖并生成 JAR 文件
date: 2022-05-21 17:10:27
tags: [Java, Maven, IntelliJ IDEA]
description: 笔者最近在学习 Java，在对 Java 的基础语言有了一定的了解后，就想着写一些小项目并打包成可执行的 JAR 文件。如何构建包含依赖的 JAR 文件呢？
---

笔者最近在学习 Java，在对 Java 的基础语言有了一定的了解后，就想着写一些小项目并打包成可执行的 JAR 文件。在了解了 Maven 和 Gradle 这两种构建工具后，笔者选择了教程较多也比较成熟的 Maven。

## Maven 项目创建

使用 IDEA 可以很方便的创建和管理 Maven 项目，在 2022.1 版本中，新建项目页面可以直接选择语言和构建工具了。

{% asset_img new_proj.png 新建项目界面 %}

这样，IDEA 就会自动为我们创建好 `pom.xml` 和目录结构了。

## 依赖管理

Maven 的依赖管理如果在 `pom.xml` 中手写的话比较麻烦，我们可以直接用 IDEA 管理，在 IDE 的最下面可以找到 “依赖项” 按钮，在这个界面中，先在左侧选择要管理依赖的模块，就可以在右侧看到该模块中已有的依赖了。要添加新的依赖，直接在搜索框中搜索，然后点击添加即可。

{% asset_img ylx.png 依赖项管理 %}

{% asset_img ss.png 搜索并添加依赖项 %}

添加后，记得点击 “加载 Maven 变更” 按钮应用更改。

{% asset_img bg.png 应用更改 %}

## 打包包含依赖项的 JAR 包

写完了程序，如果项目中包含了第三方依赖，为了方便分发，我们要将依赖也一起包进 JAR 中，这时就可以使用 `maven-assembly-plugin` 插件了。

首先要将这个插件作为依赖项加入到 pom 中去，这里直接用 IDEA 添加或者手动添加均可。

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-assembly-plugin</artifactId>
        <version>3.3.0</version>
    </dependency>
    [...]
</dependencies>
```

然后在 `build` 块中应用插件，并配置用 `package` 命令执行插件以及 JAR 相关配置

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-assembly-plugin</artifactId>
            <version>3.3.0</version>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
                <archive>
                    <manifest>
                        <mainClass>Main</mainClass> <!--这里填入要执行的主类--> 
                    </manifest>
                </archive>
            </configuration>
            <executions>
                <execution>
                    <id>make-assembly</id>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

然后执行 `mvn package` 命令或者在 IDEA 的 Maven 管理界面中双击 package 按钮即可，JAR 会生成在 target 文件夹中，分别为带有依赖的和不带依赖的。

{% asset_img maven.png Maven 管理界面 %}
