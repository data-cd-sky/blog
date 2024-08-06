---
title: java 8&11 install
top: false
cover: false
toc: true
mathjax: true
date: 2024-07-10 12:12:12
password:
summary:
tags:
categories:
---
以下是在CentOS 7.9上安装Java 8和Java 11的步骤
# CentOS 7.9 安装 Java 8 和 Java 11

在CentOS 7.9上安装Java的步骤如下。我们将展示如何使用yum安装OpenJDK版本的Java 8和Java 11。

## 安装Java 8

1. **更新包索引**

   ```bash
   sudo yum update
   ```

2. **安装OpenJDK 8**

   ```bash
   sudo yum install java-1.8.0-openjdk
   ```

3. **验证安装**

   检查Java版本以确认安装成功。

   ```bash
   java -version
   ```

   输出应该显示Java 8的版本信息。

## 安装Java 11

1. **更新包索引**

   ```bash
   sudo yum update
   ```

2. **安装OpenJDK 11**

   ```bash
   sudo yum install java-11-openjdk
   ```

3. **验证安装**

   检查Java版本以确认安装成功。

   ```bash
   java -version
   ```

   输出应该显示Java 11的版本信息。

## 设置默认Java版本

如果您的系统上安装了多个Java版本，您可能需要设置默认的Java版本。

1. **查看所有已安装的Java版本**

   ```bash
   alternatives --config java
   ```

   这将列出所有安装的Java版本及其路径。

2. **选择默认的Java版本**

   使用上一步骤中显示的选择编号来设置默认的Java版本。

   ```bash
   sudo alternatives --config java
   ```

   输入对应的编号，然后按回车键。

## 配置环境变量

为了确保Java环境变量正确设置，您可以将`JAVA_HOME`和`PATH`环境变量添加到您的用户或系统环境中。

1. **编辑用户的bash配置文件**

   ```bash
   vi ~/.bash_profile
   ```

   或者，编辑全局环境变量文件：

   ```bash
   sudo vi /etc/profile
   ```

2. **添加`JAVA_HOME`和`PATH`变量**

   对于Java 8，添加：

   ```bash
   export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk
   export PATH=$PATH:$JAVA_HOME/bin
   ```

   对于Java 11，添加：

   ```bash
   export JAVA_HOME=/usr/lib/jvm/java-11-openjdk
   export PATH=$PATH:$JAVA_HOME/bin
   ```

3. **使更改生效**

   ```bash
   source ~/.bash_profile
   ```

   或者，如果编辑的是`/etc/profile`，则需要重新登录或重启系统。