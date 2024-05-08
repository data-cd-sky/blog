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

## 前置
关闭防火墙
关闭防火墙（firewalld）
停止 firewalld 服务：

sudo systemctl stop firewalld
禁用 firewalld 服务在启动时自动启动：

sudo systemctl disable firewalld
检查 firewalld 的状态以确保它已停止并且被禁用：

sudo systemctl status firewalld

关闭 SELinux
临时关闭 SELinux（直到下次重启）：

sudo setenforce 0
永久关闭 SELinux（需要重启生效）：

编辑 SELinux 配置文件：

sudo nano /etc/selinux/config
找到这一行：

SELINUX=enforcing
将其修改为：

SELINUX=disabled
保存并关闭文件。

配置hostname
#编辑/etc/hosts文件
vi /etc/hosts
10.32.9.227 adc-mysql-1
10.32.9.228 adc-mysql-2
10.32.11.176 adc-mysql-3
重启网卡
service network restart
查看hostname是否修改
hostname

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

检查是否存在mysql用户
id mysql

创建数据目录sudo，并授权给mysql用户
sudo mkdir -p /data/mysql/data
sudo mkdir -p /data/mysql/tmp
sudo mkdir -p /data/mysql/logs
sudo mkdir -p /data/mysql/binlog
sudo chown -R mysql:mysql /data/mysql
sudo chmod 777 /data
sudo chmod -R 750 /data/mysql

配置my.cnf
配置如下：
sudo vim /etc/my.cnf
'''
[mysqld]
datadir=/data/mysql/data​
socket=/data/mysql/mysql.sock​
tmpdir=/data/mysql/tmp​
log-error=/data/mysql/logs/mysqld.log​
pid-file=/data/mysql/mysqld.pid​
#是否打开慢查询sql日志​
general_log=0​
general_log_file=/data/mysql/logs/general.log​
slow_query_log=1​
slow_query_log_file=/data/mysql/logs/mysql-slow.log​
long_query_time=5​
character-set-server=utf8mb4​
collation-server=utf8mb4_general_ci​
default-storage-engine=INNODB​
init_connect='SET NAMES utf8mb4'​
transaction_isolation=READ-COMMITTED​
binlog_format=ROW​
binlog_expire_logs_seconds=604800
log_bin=/data/mysql/binlog/mysql-bin​
log_bin_index=/data/mysql/binlog/mysql-bin.index​
relay_log=/data/mysql/logs/relay-log​
relay_log_index=/data/mysql/logs/relay-log.index​

#MySQL连接闲置超过一定时间后(单位：秒)将会被强行关闭​
##MySQL默认的wait_timeout 值为8个小时, interactive_timeout参数需要同时配置才能生效​
interactive_timeout=1800​
wait_timeout=1800​
#最大连接数​
max_connections=4096​ 
#最大错误连接数​
max_connect_errors = 10​
#是否对sql语句大小写敏感，1表示不敏感​
lower_case_table_names=1​
#时区​
default_time_zone = "+8:00"​ ​ 
#默认使用“mysql_native_password”插件认证​
default_authentication_plugin=mysql_native_password​
#INNODB​
innodb_flush_method = O_DIRECT​
innodb_log_file_size = 512M​
innodb_log_files_in_group = 3​
innodb_flush_log_at_trx_commit = 0​
innodb_strict_mode = ON​
innodb_data_file_path = ibdata1:256M;ibdata2:16M:autoextend​
innodb_checksum_algorithm = strict_crc32​
innodb_lock_wait_timeout = 5​
sql_mode='STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION'​

#mgr​
server_id=1
gtid_mode=ON​
enforce_gtid_consistency=ON​
master_info_repository=TABLE​
relay_log_info_repository=TABLE​
relay_log_recovery=ON​
binlog_checksum=NONE​
log_slave_updates=ON​
plugin_load_add='group_replication.so'​
transaction_write_set_extraction=XXHASH64​
loose-group_replication_group_name="6789b6d7-de75-11ec-a59e-fa163e7035c7"​
loose-group_replication_start_on_boot=off​
loose-group_replication_local_address= "10.32.9.227:33061"​
loose-group_replication_group_seeds= "10.32.9.227:33061,10.32.9.228:33061,10.32.11.176:33061"​ loose-group_replication_bootstrap_group=off
loose-group_replication_ip_whitelist="10.32.9.227,10.32.9.228,10.32.11.176"​
[client]​
socket=/data/mysql/mysql.sock​
default-character-set=utf8mb4
'''

初始化MySQL数据库
sudo mysqld --initialize
sudo cat /data/mysql/logs/mysqld.log
sudo grep 'temporary password' /data/mysql/logs/mysqld.log
把初始化好的文件也从root转给mysql用户
sudo chown -R mysql:mysql /data/mysql

启动MySQL服务
systemctl start mysqld
查看MySQL服务状态
systemctl status mysqld

登录mysql更改临时密码，创建管理员账号
sudo mysql -uroot -p

修改mysql root 用户密码
alter user 'root'@'localhost' identified by 'root' ;
修改其他ip地址可以连接本机mysql
use mysql;
update user set host='%' where user = 'root';
flush privileges;
alter user 'root'@'%' identified with mysql_native_password by 'root';
flush privileges;


## 配置MGR
打开my.cnf增加配置如下：
```
#mgr
server_id=3
gtid_mode=ON
enforce_gtid_consistency=ON
master_info_repository=TABLE
relay_log_info_repository=TABLE
relay_log_recovery=ON
log_bin=binlog
binlog_checksum=NONE
binlog_format=ROW
log_slave_updates=ON

plugin_load_add='group_replication.so'
transaction_write_set_extraction=XXHASH64
loose-group_replication_group_name="6e51636d-8e7c-8253-0b4b-1029b2e4f6f9"
loose-group_replication_start_on_boot=off
loose-group_replication_local_address= "10.32.11.176:33061"
loose-group_replication_group_seeds= "10.32.9.227:33061,10.32.9.228:33061,10.32.11.176:33061"
loose-group_replication_bootstrap_group=off
loose-group_replication_ip_whitelist="10.32.9.227,10.32.9.228,10.32.11.176"
```

增加mgr复制用户（三台机器都要操作）
SET SQL_LOG_BIN=0;
CREATE USER mgruser@'%' IDENTIFIED BY 'mgruser';
GRANT REPLICATION SLAVE ON *.* TO mgruser@'%';
FLUSH PRIVILEGES;
SET SQL_LOG_BIN=1;
CHANGE MASTER TO MASTER_USER='mgruser', MASTER_PASSWORD='mgruser' FOR CHANNEL 'group_replication_recovery';

show plugins;
install PLUGIN group_replication SONAME 'group_replication.so';
查看是否下载成功，使用show plugins;语句查看


## 启动mgr单主模式启动一主两从

主节点操作
SET GLOBAL group_replication_bootstrap_group=ON;
START GROUP_REPLICATION;
SET GLOBAL group_replication_bootstrap_group=OFF;

从节点操作
START GROUP_REPLICATION;

查看此时MGR组信息，MEMBER_STATE状态全部是ONLINE才是成功。不然就需要查看日志信息。
SELECT * FROM performance_schema.replication_group_members;


四、测试数据读写
主库上操作
1. 创建数据库
create database mgr;
查看从库上是否有数据库mgr
2. 创建表
use mgr;
CREATE TABLE `user` (
  `id` bigint NOT NULL ,
  `account` varchar(30)  NOT NULL ,
  `name` varchar(50) NOT NULL ,
  PRIMARY KEY (`id`) USING BTREE,
  UNIQUE KEY `UN_ACCOUNT` (`account`) USING BTREE
) 
从库上查看表信息
 3. 插入数据
INSERT INTO user VALUES (1, 'zhangsan', '张三');
INSERT INTO user VALUES (2, 'lisi', '李四');
从库查看信息如下
 4. 删除数据
delete from user where id = 1;
从库user表信息更新
5. 从库只有查询的权力，没有更改的权限
6. 主库服务停掉，查看从库状态变化
在node1 上执行
stop group_replication;
node1 状态变成OFFLINE下线状态
 在node2 上查看组信息，发现node3现在的MEMBER_ROLE为PRIMARY
 重新启动node1

start group_replication;
node1 重新加入到组中，但是此时是从节点

参考：https://blog.csdn.net/axibazZ/article/details/127103865

## 配置mysql router
五、MGR整合MySQL Router实现读写分离

1. 下载安装MySQL Router

这里我安装在node2节点上

下载
wget https://dev.mysql.com/get/Downloads/MySQL-Router/mysql-router-8.0.23-el7-x86_64.tar.gz
解压
tar -zxvf mysql-router-8.0.23-el7-x86_64.tar.gz
重命名
mv mysql-router-8.0.23-el7-x86_64 mysql-router-8.0
将mysql-router的目录添加到环境变量PATH中
echo "export PATH=$PATH:/opt/apps/mysql-router-8.0/bin/" >> /etc/profile
source /etc/profile

使用yum install 安装 yum install mysqlrouter

验证是否安装成功
mysqlrouter -V
2. 修改MySQL Router配置

在mysql-router-8.0目录下

创建日志和数据目录
mkdir logs data
在/etc 目录下创建mysqlrouter.cnf文件，内容如下
[DEFAULT]
logging_folder = /var/log/mysqlrouter
runtime_folder = /run/mysqlrouter
config_folder = /etc/mysqlrouter

connect_timeout=30
read_timeout=30

[logger]
level = INFO

# If no plugin is configured which starts a service, keepalive
# will make sure MySQL Router will not immediately exit. It is
# safe to remove once Router is configured.
[keepalive]
interval = 60

[routing:primary]
bind_address = 0.0.0.0
bind_port = 3307
max_connections = 1024
destinations = 10.32.9.227:3306,10.32.9.228:3306,10.32.11.176:3306
routing_strategy = first-available

[routing:secondary]
bind_address = 0.0.0.0
bind_port = 3308
max_connections = 1024
destinations = 10.32.9.228:3306,10.32.11.176:3306
routing_strategy = round-robin


修改权限(使用yum install则不需要)
chown -R mysql:mysql /opt/apps/mysql-router-8.0/
chown -R mysql:mysql /etc/mysqlrouter.conf
3. 启动MySQL Router
sudo systemctl start mysqlrouter(使用yum install)
mysqlrouter --config=/etc/mysqlrouter.conf &（安装包安装）
4. 验证是否正确
在node1 上连接 228 的 3307 端口查看现在的主库为node1，并且其他节点连接129的7001也均为node1。
在node1 上连接 228 的 3308 端口，显示连接的为node3
在node2 上连接 228 的 3308 端口，显示连接的为node2
在node3上连接 129 的 3308 端口，显示连接的为node3
5、验证连接3307写入

DELIMITER $$

CREATE PROCEDURE InsertMassiveData()
BEGIN
  DECLARE i INT DEFAULT 1;
  WHILE i <= 10000 DO
    INSERT INTO user (id, account, name) VALUES (i, CONCAT('user', i), CONCAT('Name', i));
    SET i = i + 1;
  END WHILE;
END$$

DELIMITER ;

CALL InsertMassiveData();

select count(*) from mgr.user;
结果为10000；

至此，MySQL MGR集群安装完毕。