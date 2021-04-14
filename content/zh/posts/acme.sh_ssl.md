---
title: "acme.sh自动部署更新SSL证书"
date: 2020-01-29T11:09:16+08:00
tags:
- ssl
categories:
- "技术"
series:
- 温故而知新
draft: false
toc: true
---

自从letsencrypt可以获取免费ssl之后，全民https，但是有很多局限，比如每三个月需要更新证书，本地需要webserver等一些不便的方式，所以我们需要一切自动化。 
<!--more--> 
acme.sh 是一个自动申请 https 证书的脚本，实现了 acme 协议, 可以从 [letsencrypt](https://letsencrypt.org/) 生成免费的证书。  

## 安装
一般有两种方式,默认情况下安装到 ```~/.acme.sh/``` 目录下  

1. 脚本安装  
```bash
curl  https://get.acme.sh | sh
```

2. 源代码安装  
```bash
git clone https://github.com/Neilpang/acme.sh.git /data/acme.sh
/data/acme.sh/acme.sh --install
```
安装程序会执行三个操作
   - 创建和复制acme.sh到你的主目录```（$HOME）~/.acme.sh/```。所有证书也将被放置在这个文件夹中。
   - 创建别名acme.sh=~/.acme.sh/acme.sh。
   - 创建每月 cron 任务，根据需要检查和更新证书。
## 生成证书

一般有两种方式验证: http 和 dns 验证

1. http 方式需要在你的网站根目录下放置一个文件, 来验证你的域名所有权，完成验证，然后就可以生成证书了。
2. dns 方式, 在域名上添加一条 txt 解析记录, 验证域名所有权。

我一般使用的是第一种方式,简单些, 并且现在的脚本已经自动化了，只需要指定域名, 并指定域名所在的网站根目录. acme.sh 会全自动的生成验证文件, 并放到网站的根目录, 然后自动完成验证. 最后自动删除验证文件. 

```bash
./acme.sh --issue -d test.xxxx.cn --nginx --webroot /www/test
```
等待执行完成，一切比较顺利的情况下，结束后如果显示成功，则证书申请成功。

安装成功后输出以下日志  
```base
[14:57:08 CST 2020] Your cert is in  /root/.acme.sh/**********.cer
[14:57:08 CST 2020] Your cert key is in  /root/.acme.sh/**********.key
[14:57:08 CST 2020] The intermediate CA cert is in  /root/.acme.sh/**********.cer
[14:57:08 CST 2020] And the full chain certs is there:  /root/.acme.sh/**********fullchain.cer
```

```bash
[14:43:01 CST 2020] Please check log file for more details: /root/.acme.sh/acme.sh.log
```
但是也有可能出现上面的错误。就需要查看日志文件和debug参数了,如果没时间去排除错误情况的话，直接使用下面的方式(独立)。

```bash
./acme.sh --issue -d test.xxxx.cn --standalone
```
独立验证方式需要确保80端口没有被占用，自动启用服务，自动验证后关闭服务。

## 安装证书
默认情况下 证书都安装在```/root/.acme.sh/test.xxxx.cn```目录下,一般不直接使用这个目录，通过命令去安装证书文件到指定目录下。

```bash
acme.sh --installcert -d test.xxxx.cn --key-file /data/nginx/ssl_cert/test.xxxx.cn/test.xxxx.cn.key --fullchain-file /data/nginx/ssl_cert/test.xxxx.cn.cer --reloadcmd "systemctl force-reload nginx.service"
```

key文件路径为 ```/data/nginx/ssl_cert/test.xxxx.cn/test.xxxx.cn.key ```  
完整链证书文件路径为 ```/data/nginx/ssl_cert/test.xxxx.cn.cer```

## 更新配置

如果原来是有配置 SSL 证书的，所以只需要把对应的证书路径进行替换即可。

```conf
# /etc/nginx/conf.d/xxx.conf
ssl_certificate     /data/nginx/ssl_cert/test.xxxx.cn.cer;
ssl_certificate_key /data/nginx/ssl_cert/test.xxxx.cn/test.xxxx.cn.key;
```
完整的关键配置信息参考
```conf
server {
    ....
    listen 443 ssl;
    server_name test.xxxx.cn;
    ssl_certificate /data/nginx/ssl_cert/test.xxxx.cn.cer;
    ssl_certificate_key /data/nginx/ssl_cert/test.xxxx.cn/test.xxxx.cn.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    ....
}
; http 自动跳转 https（可选）
server {
  listen         80;
  server_name    test.xxxx.cn;
  return         rewrite ^(.*) https://$host$1 permanent;
}
```

重启 Nginx。
```bash
//centos6 
service nginx restart

//centos7
systemctl force-reload nginx
```

## 更新证书

crontab 中已经自动加上了acme的脚本 ，可以参考

## 更多

```bash
# 升级 acme.sh 到最新版
acme.sh --upgrade

```

## 参考文章
- [acme.sh](https://github.com/acmesh-official/acme.sh)
- [acme.sh 中文说明](https://github.com/acmesh-official/acme.sh/wiki/说明)