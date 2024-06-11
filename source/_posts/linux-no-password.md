---
title: linux-no_password
top: false
cover: false
toc: true
mathjax: true
date: 2024-05-10 17:32:22
password:
summary:
tags: linux
categories: linux
---
# 实现服务器登录免密

## 在服务器 A 上生成一个 SSH 密钥对（有则跳过）：

ssh-keygen -t rsa
默认情况下，这会在 ~/.ssh/ 目录下创建两个文件：id_rsa（私钥）和 id_rsa.pub（公钥）。

## 将公钥复制到服务器 B：
cat ~/.ssh/id_rsa.pub
然后复制输出的内容（公钥），登录到服务器 B，创建或编辑 ~/.ssh/authorized_keys 文件，并将复制的公钥粘贴到文件的末尾：

## 在服务器 B 上
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo '粘贴公钥内容' >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys


免密要注意 /home和~/ 755 别给777

以上齐活