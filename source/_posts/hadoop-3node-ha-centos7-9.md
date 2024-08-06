---
title: hadoop-3node-ha-centos7.9
top: false
cover: false
toc: true
mathjax: true
date: 2024-07-16 14:39:37
password:
summary:
tags:
categories:
---

# Hadoop三节点高可用集群配置指南

本指南将帮助您在CentOS 7上配置Hadoop三节点高可用集群。我们假设您已经安装了Hadoop，并且已经准备好三台机器作为节点。

## 前提条件
- 三台运行CentOS 7的服务器，分别命名为`node1`, `node2`, `node3`。
- 每台服务器上都已经安装了Java和Hadoop。
- 每台服务器的`/etc/hosts`文件已配置好所有节点的IP地址和主机名。
- 关闭防火墙，selinux,配置好时间同步
- 三台服务器之间的SSH免密登录已经设置好。

## 免密步骤
ssh-keygen -t rsa -b 4096
cat ~/.ssh/id_rsa.pub
vim ~/.ssh/authorized_keys
放入三台的id_rsa.pub
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
测试免密登录
ssh adc-hadoop-1...-3

## ZooKeeper安装与配置

Hadoop高可用性依赖于ZooKeeper集群。

1. **在所有节点上安装ZooKeeper**

https://blog.csdn.net/networken/article/details/116407042

3. **配置ZooKeeper集群**

   在每个节点的`/etc/zookeeper/conf/zoo.cfg`文件中进行配置。

   ```conf
   tickTime=2000
   dataDir=/var/lib/zookeeper
   clientPort=2181
   initLimit=5
   syncLimit=2
   server.1=adc-hadoop-1:2888:3888
   server.2=adc-hadoop-2:2888:3888
   server.3=adc-hadoop-3:2888:3888
   ```

   在`/var/lib/zookeeper`目录下创建一个文件名为`myid`的文件，文件内容为该节点对应的编号（例如，`node1`的`myid`文件中就写入`1`）。

   ```bash
   echo "1" > /var/lib/zookeeper/myid # 在node1上
   # 对应地，在node2上写入2，在node3上写入3
   ```

   然后重启ZooKeeper服务。

   ```bash
   sudo service zookeeper-server restart
   ```

## Hadoop配置

1. **配置`core-site.xml`**

   在所有节点的`$HADOOP_CONF_DIR/core-site.xml`文件中添加以下配置：

   ```xml
   <configuration>
     <property>
       <name>fs.defaultFS</name>
       <value>hdfs://mycluster</value>
     </property>
     <property>
       <name>ha.zookeeper.quorum</name>
       <value>node1:2181,node2:2181,node3:2181</value>
     </property>
   </configuration>
   ```

2. **配置`hdfs-site.xml`**

   在所有节点的`$HADOOP_CONF_DIR/hdfs-site.xml`文件中添加以下配置：

   ```xml
   <configuration>
     <property>
       <name>dfs.replication</name>
       <value>3</value>
     </property>
     <property>
       <name>dfs.nameservices</name>
       <value>mycluster</value>
     </property>
     <property>
       <name>dfs.ha.namenodes.mycluster</name>
       <value>nn1,nn2</value>
     </property>
     <property>
       <name>dfs.namenode.rpc-address.mycluster.nn1</name>
       <value>node1:8020</value>
     </property>
     <property>
       <name>dfs.namenode.rpc-address.mycluster.nn2</name>
       <value>node2:8020</value>
     </property>
     <property>
       <name>dfs.namenode.http-address.mycluster.nn1</name>
       <value>node1:50070</value>
     </property>
     <property>
       <name>dfs.namenode.http-address.mycluster.nn2</name>
       <value>node2:50070</value>
     </property>
     <property>
       <name>dfs.namenode.shared.edits.dir</name>
       <value>qjournal://node1:8485;node2:8485;node3:8485/mycluster</value>
     </property>
     <property>
       <name>dfs.journalnode.edits.dir</name>
       <value>/mnt/journal</value>
     </property>
     <property>
       <name>dfs.ha.automatic-failover.enabled</name>
       <value>true</value>
     </property>
   </configuration>
   ```

3. **配置`yarn-site.xml`**

   在所有节点的`$HADOOP_CONF_DIR/yarn-site.xml`文件中添加以下配置：

   ```xml
   <configuration>
     <property>
       <name>yarn.resourcemanager.ha.enabled</name>
       <value>true</value>
     </property>
     <property>
       <name>yarn.resourcemanager.cluster-id</name>
       <value>mycluster</value>
     </property>
     <property>
       <name>yarn.resourcemanager.ha.rm-ids</name>
       <value>rm1,rm2</value>
     </property>
     <property>
       <name>yarn.resourcemanager.hostname.rm1</name>
       <value>node1</value>
     </property>
     <property>
       <name>yarn.resourcemanager.hostname.rm2</name>
       <value>node2</value>
     </property>
     <!-- 配置其他YARN相关属性 -->
   </configuration>
   ```

## 初始化JournalNode

在所有节点上启动JournalNode服务。

```bash
hadoop-daemon.sh start journalnode
```

## 格式化NameNode

在`node1`和`node2`上分别格式化NameNode。

```bash
hdfs namenode -format
```

## 启动Hadoop集群

1. **启动NameNode和DataNode**

   在`node1`和`node2`上启动NameNode。

   ```bash
   start-dfs.sh
   ```

2. **启动ResourceManager和NodeManager**

   在所有节点上启动YARN服务。

   ```bash
   start-yarn.sh
   ```

## 验证集群状态

使用Web界面或`hdfs haadmin`命令来验证集群的状态。

```bash
hdfs haadmin -getServiceState nn1
hdfs haadmin -getServiceState nn2
```

现在，您应该有一个运行中的Hadoop高可用集群。
