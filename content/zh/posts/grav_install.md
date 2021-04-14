---
title: "Grav 安装和使用"
date: 2020-01-15T11:09:16+08:00
draft: false
---


## Grav 安装

### 简介
Grav 是一个简单、可扩展的纯文件 CMS 平台，解压即用，无需安装，我最看重的两个特点是不需要数据库，支持Markdown，方便部署

### 安装

- 官网 [https://getgrav.org](https://getgrav.org/downloads)

- 选择包括后台管理的 （grav core + Admin plugin ）下载
[下载地址](https://getgrav.org/download/core/grav-admin/latest)

- 解压到 Web 目录下 默认为 grav-admin 目录

- 拷贝配置（webserver-configs/nginx.conf） 到本地 nginx配置目录

- 修改nginx.conf配置  
  root 为网站目录  
  servername 为 网站域名  
  fastcgi_pass 为php配置地址

- 然后输入地址访问  
   http://xxxx.com/grav-admin 

- 界面会检查相关目录权限初始化  
  如果没有错误的话，会出现创建管理员界面

- 创建管理员成功后，登陆管理员后台 

- 点击个人头像，修改语言为中文，刷新后可以使用了

```shell
wget https://getgrav.org/download/core/grav-admin/1.6.19 
unzip 1.6.19
cd grav-admin
cp webserver-configs/nginx.conf /usr/local/nginx/conf/nginx.conf
chmod 777 cache backup logs images assets tmp user -R

```
### 配置

