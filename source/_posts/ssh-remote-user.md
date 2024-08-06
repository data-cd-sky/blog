---
title: ssh remote user
top: false
cover: false
toc: true
mathjax: true
date: 2024-08-06 09:06:05
password:
summary:
tags:
categories:
---
sudo useradd app
sudo passwd app
sudo usermod -aG maintain app
sudo vim /etc/sudoers
app      ALL=(ALL) NOPASSWD: ALL
sudo -su app
sudo usermod -aG docker $USER
sudo mkdir /home/app
sudo chown app:app /home/app
sudo chmod 700 /home/app