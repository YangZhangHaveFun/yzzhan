---
title : "对用过的Linux(Centos7.0+)指令的总结"
date: 2019-04-11T01:37:56+08:00
lastmod: 2019-05-08T01:37:56+08:00
draft: false
tags: ["Linux", "Centos"]
categories: ["Linux related"]
author: "yang zhang"
---
### 开启防火墙对指定Port外界访问的权限
例如开启数据库port3306的port
```bash
!/bin/bash
/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
service iptables save
service iptables restart
```
如果iptables save不能用则使用以下命令
```bash
# It is possible to go back to a more classic iptables setup. First, stop and mask the firewalld service:
systemctl stop firewalld
systemctl mask firewalld
# Then, install the iptables-services package:
yum install iptables-services
# Enable the service at boot-time:
systemctl enable iptables
# Managing the service
systemctl [stop|start|restart] iptables
# Saving your firewall rules can be done as follows:
service iptables save
# check your listenning ports with:
netstat -nat |grep :3306
```

### 重装mariadb
重装database server的时候, 任何残留的mysql文件都会导致新安装的mariadb.service不能正常启动.
```bash
# 首先检查数据库的状态
systemctl status mariadb.service
# 如果在运行则先停止服务
systemctl stop mariadb.service
# 然后检查所有的安装文件和相应的依赖包, 然后全部删掉
rpm -qa | grep 'maria*'
yum -y remove maria*
rpm -qa | grep 'maria*'
# 先把配置文件删掉
rm /etc/my.cnf
# 然后检查遗留文件
find / -name mysql
find / -name mariadb
# Manually delete every single file and corresponding folder, here only shows part of code as example. 
rm -rf /var/log/mariadb/*
rm -d /var/log/mariadb
# 检查一遍有没有剩余
find / -name mysql
find / -name mariadb
# 重启一哈
sudo reboot
# 安装
yum -y install mariadb mariadb-server
systemctl status mariadb
systemctl enable mariadb
mysql_secure_installation
```

### linux系统下生成秘钥并通知本地系统
```bash
ssh-keygen -t rsa -C "yzzhan@student.unimelb.edu.au"
# identification has been saved in /home/root/.ssh/id_rsa.
# public key has been saved in /home/root/.ssh/id_rsa.pub

ssh-add ~/.ssh/id_rsa

```

### 对ssh用法的总结
先来一段关于ssh的定义
> SSH is a software package that enables secure system administration and file transfers over insecure networks.

#### 基本用法
一般最简单ssh的用法就是直接登录远程服务器,然后输入密码进入服务器
```bash
ssh USER@IP_OR_DOMAINNAME
```
或者通过把自己的公钥放到服务器,用自己的私钥来访问远程服务器. 这需要私钥存放在~/.ssh目录下
```bash
ssh -i id_rsa USER@IP_OR_DOMAINNAME
```
如果觉得麻烦可以把自己的公钥放在服务器的~/.ssh/authorized_keys文件里.这样就可以直接登录了.
> An authorized key in SSH is a public key used for granting login access to users. The authentication mechanism is called public key authentication.

#### 高级用法
这些是ssh的一些基本用法. 下面开始介绍ssh的高级用法 ssh Tunnel-Local and Remote Port Forwarding

##### Local Port Forwarding 本地端口映射
对于一个服务器来说,22端口一般都会开放,用来做ssh远程连接.但是对于数据库的端口一般是不对外开放的.那我们如何在数据库端口不对外开放的情况下连接数据库呢.这里拿couchdb做例子对应的外部访问端口是5984.
这里提供的方法是通过ssh建立连接,然后把服务器的5984端口映射到本机上,再通过本机访问
```bash
ssh -L 9000:localhost:5984
```
这时通过访问 http://localhost:9000 就可以访问到远程服务器的couchdb服务器了,并且绕过了服务器的防火墙.

这里对这行shell command做一些解释, -L 表示要进行的操作是local port forwarding. 9000是本地的目标端口. `localhost:5984`是指以服务器的角度看的localhost:5984, 换言之就是服务器的数据库以为请求是localhost:5984.
> If we however say something like 9000:localhost:5984, it means localhost from the server’s perspective, not localhost on your machine. This means forward my local port 9000 to port 5984 on the server, because when you’re on the server, localhost means the server itself.

##### Remote Port Forwarding 远程端口映射
这个功能相对于上个功能有一点鸡肋,主要的作用在于本地无法访问外网,但是可以连到一个内网的计算机,这个计算机可以连接外网.如果你想把自己的某个端口暴露给外界,则可以使用Remote Port Forwarding的方法.
```bash
ssh -R 9000:localhost:5984 user@example.com
```
-R 代表remote port forwarding, 9000代表远程服务器要监听的端口. localhost就代表本地的localhost,5984是本地想要开放的端口. 但是,默认的配置是关闭remote port forwarding,需要在`/etc/ssh/sshd_config`这个文件里配置
```bash
GatewayPorts yes
```
然后重启ssh service. 之后别的计算机可以通过user@example.com:9000来访问本地的5984端口.

需要注意的是执行以上ssh命令会同时登陆服务器, 但其实并不需要,我们可以加上以下参数避免同时登陆.
```bash
ssh -nNT -R 9000:localhost:5984 user@example.com
```



