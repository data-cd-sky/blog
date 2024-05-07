---
title: db-mysql
top: false
cover: false
toc: true
mathjax: true
date: 2024-05-06 16:34:45
password:
summary:
tags: db
categories: db
---
# 搭建高可用mysql-三节点


## 机器环境：
```
cat /etc/*release 
lscpu
free -h
df -h
```
查看当前机器系统、cpu核数、内存、磁盘空间

1、adc-mysql-1:centos7.9，4C，7.7G（swap 8G），92G，10.32.9.227
2、adc-mysql-2:centos7.9，4C，7.7G（swap 8G），92G，10.32.9.228
3、adc-mysql-3:centos7.9，4C，7.6G（swap 0），92G，10.32.11.176

## 各节点安装 MySQL
1、mysql版本：8.0.30

添加yum源
安装mysql
配置my.cnf
要在CentOS 7.9上安装MySQL 8.0.30，您需要按照以下步骤操作：

import秘钥，此处需要带年份，2024的秘钥没有，所以使用的2023
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023

下载MySQL Yum仓库配置包：
此处参考网上文档，使用7-5
wget https://dev.mysql.com/get/mysql80-community-release-el7-5.noarch.rpm
安装MySQL Yum仓库配置包：

sudo rpm -ivh mysql80-community-release-el7-5.noarch.rpm
这个命令将添加MySQL仓库到您的仓库列表中。

查看可用的MySQL版本：
yum list mysql-community-server --showduplicates
这将列出所有可用的MySQL社区服务器版本。

选择特定版本进行安装：
sudo yum install mysql-community-server-8.0.30

启动MySQL服务：
安装完成后，启动MySQL服务：

sudo systemctl start mysqld
安全设置MySQL：
运行mysql_secure_installation脚本来设置root密码和调整安全设置：

sudo mysql_secure_installation
检查MySQL服务状态：
检查MySQL服务是否成功启动：

sudo systemctl status mysqld