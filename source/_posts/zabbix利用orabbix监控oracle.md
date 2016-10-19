title: zabbix利用orabbix监控oracle
date: 2015-12-14 17:22:52
tags:
---
下载orabbix

```
wget http://www.vdisk.cn/Orabbix/orabbix-1.2.3.zip
```

* 确保安装jdk环境，java version查看,没有则通过yum来安装JAVA：yum install java

```
midir -p /opt/orabbix
cd /opt/orabbix
unzip orabbix-1.2.3.zip
chmod -R a+x orabbix/
cp /opt/orabbix/conf/config.props.sample /opt/orabbix/conf/config.props

```
* 编辑orabbix配置文件

```
vi confi/config.props

ZabbixServerList=ZabbixServer1
ZabbixServer1.Address=8.8.8.8 #这里是zabbix server地址
ZabbixServer1.Port=10051

OrabbixDaemon.PidFile=./logs/orabbix.pid
OrabbixDaemon.Sleep=300
OrabbixDaemon.MaxThreadNumber=100

DatabaseList=ora29a
DatabaseList.MaxActive=10
DatabaseList.MaxWait=100
DatabaseList.MaxIdle=1

ora29a.Url=jdbc:oracle:thin:@1.2.3.4:1521:ora29a
ora29a.User=zabbix
ora29a.Password=zabbix
ora29a.MaxActive=10
ora29a.MaxWait=100
ora29a.MaxIdle=1
ora29a.QueryListFile=./conf/query.props

```

* 建立oracle user

```
CREATE USER ZABBIX
     IDENTIFIEDBY zabbix     <Password>;
     GRANT CONNECT TO ZABBIX;
     GRANTRESOURCE TO ZABBIX;
     ALTERUSER ZABBIX DEFAULT ROLE ALL;
     GRANT SELECT ANY TABLE TO ZABBIX;
     GRANT CREATE SESSION TO ZABBIX;
     GRANTSELECT ANY DICTIONARY TO ZABBIX;
     GRANTUNLIMITED TABLESPACE TO ZABBIX;
     GRANTSELECT ANY DICTIONARY TO ZABBIX;
```

* Oracle 11g添加如下命令

```
exec dbms_network_acl_admin.create_acl(acl => 'resolve.xml',description =>'resolve acl', principal =>'ZABBIX', is_grant => true, privilege =>'resolve');
exec dbms_network_acl_admin.assign_acl(acl=> 'resolve.xml', host =>'*');
commit;
```

* 创建执行文件（直接cp即可）

```
[root@oracle orabbix]# cp /opt/orabbix/init.d/orabbix/etc/init.d/orabbix
7.保存退出，启动orabbix服务(确保有执行权限)
/etc/init.d/orabbix start
Orabbix服务加入随系统启动：
chkconfig --add orabbix
chkconfig --level 345 orabbix on
```
* zabbix server 导入模板

