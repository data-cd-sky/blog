---
title: db-migrate
top: false
cover: false
toc: true
mathjax: true
date: 2024-05-08 14:30:32
password:
summary:
tags: db
categories: db
---
# 数据库迁移

## MGR单主集群，通过sql创建用户
创建一个data管理员用户
创建一个map用户，管理3dServer
CREATE DATABASE auth;
CREATE DATABASE process;
CREATE DATABASE 3dServer;
CREATE DATABASE label;

CREATE USER 'data'@'%' IDENTIFIED BY 'data';
GRANT ALL PRIVILEGES ON *.* TO 'data'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;

CREATE USER 'map'@'%' IDENTIFIED BY 'map';
GRANT ALL PRIVILEGES ON `3dServer`.* TO 'map'@'%';
FLUSH PRIVILEGES;

-- 如果创建错误用户可以删除
DROP USER '3dServer'@'%';
FLUSH PRIVILEGES;