---
title: pgsql14+postgis Centos7.9
top: false
cover: false
toc: true
mathjax: true
date: 2024-06-06 16:31:15
password:
summary:
tags:
categories:
---
#在cenos7.9安装pgsql14并且配置postgis插件
在CentOS 7.9上安装PostgreSQL 14以及PostGIS插件通常涉及以下步骤：

### 安装PostgreSQL 14


## 例行 先关掉防火墙和selinux

1. **导入PostgreSQL 14的官方仓库**

   首先，你需要导入PostgreSQL的官方仓库。打开终端并运行以下命令：

   ```sh
   sudo yum install -y https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm
   ```

2. **安装PostgreSQL 14**

   一旦添加了仓库，你可以安装PostgreSQL 14：

   ```sh
   sudo yum install -y postgresql14-server
   ```

3. **初始化数据库**

   安装完成后，需要初始化数据库：

   ```sh
   sudo /usr/pgsql-14/bin/postgresql-14-setup initdb
   ```

4. **启动PostgreSQL服务并设置为开机启动**

   ```sh
   sudo systemctl enable postgresql-14
   sudo systemctl start postgresql-14
   ```

5. **切换到postgres用户并创建一个新的数据库角色（如果需要）**

   ```sh
   sudo su - postgres
   createuser <your_username>
   ```

6. **创建一个新的数据库（如果需要）**

   ```sh
   createdb <your_database_name>
   ```

7. **更改PostgreSQL用户的密码（推荐）**

   ```sh
   psql -c "alter user postgres with password 'your_secure_password';"
   ```

### 安装PostGIS

1. **安装EPEL仓库**

   PostGIS可能需要EPEL仓库中的一些依赖项，因此需要确保EPEL仓库已经安装：

   ```sh
   sudo yum install -y epel-release
   ```

2. **安装PostGIS**

   现在可以安装PostGIS扩展了：

   ```sh
   sudo yum install -y postgis31_14
   ```

   这将会安装PostGIS和它的依赖项。

3. **在你的数据库中启用PostGIS**

   你需要为你的数据库启用PostGIS扩展。首先，切换到`postgres`用户：

   ```sh
   sudo su - postgres
   ```

   然后连接到你的数据库：

   ```sh
   psql -d your_database_name
   ```

   在你的数据库中启用PostGIS扩展：

   ```sql
   CREATE EXTENSION postgis;
   ```

   如果你还需要其他相关功能，比如拓扑支持，你还可以添加：

   ```sql
   CREATE EXTENSION postgis_topology;
   ```

   退出psql：

   ```sh
   \q
   ```
##配置允许远程连接
cd /var/lib/pgsql/14/data
vim postgresql.conf

修改监听所有IP
# - Connection Settings -

listen_addresses = '*'          # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
vim pg_hba.conf

加上
# TYPE  DATABASE        USER            ADDRESS                 METHOD

host    all             all             0.0.0.0/0               md5
