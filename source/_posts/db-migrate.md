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

## 配置旧数据库主机到新数据库主机免密
参考另一篇文章 linux-no_password


## 两边创建sql文件夹
cd ~/
mkdir -p sql
cd sql
## 在sql目录执行 数据库服务器a导出原数据库的表
mysqldump -uroot -proot -h 127.0.0.1 sse count_info_history > count_info_history.sql
## scp 将sql复制过去
scp *.sql xxx@xxx.xx.xx.xx:/home/xxx/sql
## 在服务器b上导入到新数据库
cd ~/sql/
mysql -uUsername -pPassword new_database < table_name.sql

## 迁移原则：
1、在不产生新数据时迁移；
2、先迁移使用频次更高的；以便有问题可以更早发现
3、逐步迁移，迁移前，先识别出相关项目；各后端做确认；



# 迁移pgsql数据；
1、dump备份数据库：
pg_dump -U username -W -F c -b -v -f "/path/to/your/backup/filename.backup" dbname

2、配置免密 然后传输
scp

3、在新库恢复数据：
createdb -U username newdbname
pg_restore -U username -d newdbname -v "/path/to/your/backup/filename.backup"
