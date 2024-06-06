---
title: geoserver install
top: false
cover: false
toc: true
mathjax: true
date: 2024-06-06 17:27:51
password:
summary:
tags:
categories:
---
# 安装部署geoserver
1、安装java11
sudo yum install java-11-openjdk -y
java --version

如果默认不是11，使用这个命令选择11
sudo alternatives --config java

vim /etc/profile
export JAVA_HOME=$(dirname $(dirname $(readlink $(readlink $(which java)))))
export PATH=$PATH:$JAVA_HOME/bin
source /etc/profile

2、官网下载geoserver的二进制文件压缩包
解压到/opt/geoserver

3、创建geoserver用户组和geoserver用户
groupadd geoserver
useradd -g geoserver geoserver

补充：
groupadd zhangsangroup   #创建用户组zhangsangroup
useradd -g zhangsan zhangsan  创建用户zhangsan并加入zhangsangroup组

useradd参数：

-u UID：指定 UID，这个 UID 必须是大于等于500，并没有其他用户占用的 UID
-g GID/GROUPNAME：指定默认组，可以是 GID 或者 GROUPNAME，同样也必须真实存在
-G GROUPS：指定额外组
-c COMMENT：指定用户的注释信息
-d PATH：指定用户的家目录

4、将文件夹授权给geoserver用户
chown -R geoserver:geoserver /opt/geoserver

5、配置systemctl、启动

vim /etc/systemd/system/geoserver.service

[Unit]
Description=GeoServer
After=network.target

[Service]
User=geoserver
Group=geoserver
Type=simple
ExecStart=/opt/geoserver/bin/startup.sh
ExecStop=/opt/geoserver/bin/shutdown.sh
Restart=on-failure
WorkingDirectory=/opt/geoserver

[Install]
WantedBy=multi-user.target


sudo systemctl daemon-reload
sudo systemctl start geoserver.service
sudo systemctl status geoserver.service
