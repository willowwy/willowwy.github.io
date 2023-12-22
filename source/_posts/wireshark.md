---
title: 可抓包wireshark安装Linux
date: 2023-07-31 17:24:51
tags: [wireshark]
categories: [Tools]
description: 可抓包wireshark安装
cover: images/wireshark.jpg
comments: false
---

# wireshark

图形界面安装的大部分wireshark都是不能抓取数据包的，需要root权限，所以需要安装命令行版本的wireshark

## 安装
``` bash
$ sudo apt-get install wireshark # 安装wireshark
$ sudo dpkg-reconfigure wireshark-common # 选择yes
$ sudo usermod -a -G wireshark $USER # 将当前用户加入wireshark组
$ sudo reboot # 重启
```
