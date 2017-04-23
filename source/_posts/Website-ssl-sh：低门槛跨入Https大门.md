---
title: Website-ssl.sh：低门槛跨入Https大门
date: 2016-09-03 06:27:33
tags: 
- Https
categories: 
- 网络
---

## 转载
### 一  前言
##### 网站`https化`已是大势所趋，我的个人blog也是老早之前就想搞的，总是没时间。 今天休假，在家折腾了一下，`baidufe.com`的图标终于变成`绿色的小锁`了，美美哒！
##### [jerry Qu](https://imququ.com/post/letsencrypt-certificate.html)大神研究的很深，我把自己的操作步骤整合了一下，做成一个小工具，也许对大家有用！ https://github.com/zxlie/website-ssl.sh
<!-- more -->
### 二 使用方式
 **1、下载**
```
# 下载工具
curl -so website-ssl.sh https://raw.githubusercontent.com/zxlie/website-ssl.sh/master/website-ssl.sh
chmod 0755 website-ssl.sh
```
没错，就这么下载了就能用了！当然，`github`源文件的下载，你也可以用你熟悉的任何方式！
注意：此工具会使用到`openssl`命令，请务必保证你的机器上已安装此工具！

**2、 配置**

** 2.1 wsl.cnf.sh的配置**
首次执行`website-ssl.sh`的时候，工具会自动在当前目录下创建配置文件：`wsl.cnf.sh`

```
sh website-ssl.sh
```
得到结果：

```
您的配置文件「wsl.cnf.sh」配置不正确或还未进行配置，请检查！
```
你可用任意编辑工具打开wsl.cnf.sh文件，针对头部的如下几个配置项进行`按需配置`：

```
# ************************ 配置区域 START ******************************
# 你的ssl主目录位置
ssl_dir="/home/work/www/ssl"
# nginx中配置的，给 Let's Encrypt 验证用的
challenges_dir="/home/work/www/challenges/"
# 按照你的需求进行配置，多个域名用空格分开
websites="your-website.com www.your-website.com"
# ************************ 配置区域 END ********************************
```
 **2.2 nginx conf文件的配置**

本工具是用`Let's Encrypt`来实现的`https`，所以证书的申请需要一个域名验证的过程； 也就是需要对目标站点的`Nginx`增加一个`location`，形如：

```
# CA认证
location ^~ /.well-known/acme-challenge/ {
    # 注：这里的$challenges_dir请替换成你自己的真实目录，如：/home/work/www/challenges/
    alias $challenges_dir;
    try_files $uri =404;
}
```
**3 使用**

```
# 直接执行脚本，获取帮助信息
sh website-ssl.sh
```
结果

```
 网站ssl自动化工具（v1.0）使用方法:

    usage: sh website-ssl.sh -v | csr | pem | nginx | renew | crontab | upgrade
    -v        查看工具的版本号
    csr       根据域名配置生成csr证书文件（For pem）
    pem       生成 Let's Encrypt 认可的pem证书文件
    nginx     获取nginx配置文件Demo
    renew     更新pem证书文件
    crontab   自动更新pem证书文件的crontab任务
    upgrade   升级「website-ssl.sh」工具到最新版
```
**4 实际使用案例**

*** step 1: *** *** 创建`pem`文件

```
sh website-ssl.sh pem
```
 **注** ***：这一步会自动为我们创建`domain`文件和`ssl-encrypt.pem`文件***

***step2:*** ***获取`nginx`配置的Demo***

```
sh website-ssl.sh nginx
```
 **注** ***：如果知道怎么配置nginx,这一步可忽略***
*** step3:配置自己的`nginx.conf`文件***

核心就是配置一下这个：

```
server {
    listen 443 ssl;
    server_name  your-website.com;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers EECDH+CHACHA20:EECDH+CHACHA20-draft:EECDH+AES128:RSA+AES128:EECDH+AES256:RSA+AES
    ssl_prefer_server_ciphers on;
    ssl_certificate /home/work/www/ssl/ssl-encrypt.pem;
    ssl_certificate_key /home/work/www/ssl/domain.key;
    ssl_session_timeout 5m;

    ...
}
```
