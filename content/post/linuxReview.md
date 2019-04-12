---
title : "对用过的Linux(Centos7.0+)指令的总结"
date: 2019-04-11T01:37:56+08:00
lastmod: 2019-04-11T01:37:56+08:00
draft: false
tags: ["Linux", "Centos"]
categories: ["Linux related"]
author: "yang zhang"
---
###开启防火墙对指定Port外界访问的权限
例如开启数据库port3306的port
```bash
!/bin/bash
/sbin/iptables -I INPUT -p tcp --dport 3306 -j ACCEPT
service iptables save
service iptables restart
```

###重装mariadb
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

### 

