---
title: nvm
date: 2023-07-31 17:32:31
tags: [nvm]
categories: [tool]
description: nvm 安装
cover: images/nvm.jpg
comments: false
---


### 创建目录
``` bash
mkdir -p /usr/local/nvm
```
### 下载源码
``` bash
git clone https://github.com/nvm-sh/nvm.git /usr/local/nvm
```
### 切换目录
``` bash
cd /usr/local/nvm
```
### 安装
``` bash
./install.sh
```
### 添加淘宝镜像
``` bash
export NVM_NODEJS_ORG_MIRROR=https://npm.taobao.org/dist
```

### 重启配置文件不会操作也可以重启Linux
``` bash
source ~/.bashrc
#输入到bash xxx的时候需要根据操作系统决定 我用的deepin是.bashrc | Ubuntu应该是bash_profile
```
 
### 开始安装
``` bash
nvm install stable
```