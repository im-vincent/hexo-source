title: install zabbix from packages
date: 2015-12-09 15:32:17
tags: [linux,zabbix]
---


## Installation from packages
https://www.zabbix.com/documentation/2.4/manual/installation/install_from_packages

本来准备用tar包去安装的，结果弄了一个上午都没弄好。网上写的大部分都有问题。

索性就看官方文档了，发现还是官方文档比较好一点。

有几个地方需要记录下

```
#默认安装完mysql是没有启动的
service mysqld start

#mysql建立数据库和赋予对应权限
mysql -u root -p
create database zabbix character set utf8;
grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
flush privileges;

#php时区调整
/etc/httpd/conf.d/zabbix.conf
Asia/Shanghai
 
```

