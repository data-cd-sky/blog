---
title: flink-deploy
top: false
cover: false
toc: true
mathjax: true
date: 2024-08-05 14:30:53
password:
summary:
tags:
categories:
---
参考：https://nightlies.apache.org/flink/flink-docs-master/zh/docs/try-flink/local_installation/
官方文档

此时暂时使用率不高，主要用spark处理批数据，先docker compose启动一个容器做测试
version: '3'
services:
  jobmanager:
    image: flink:1.13.2
    ports:
      - "8081:8081" # Flink Web Dashboard
    command: jobmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager

  taskmanager:
    image: flink:1.13.2
    depends_on:
      - jobmanager
    command: taskmanager
    environment:
      - JOB_MANAGER_RPC_ADDRESS=jobmanager