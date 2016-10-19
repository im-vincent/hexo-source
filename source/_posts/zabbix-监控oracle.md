title: zabbix 监控oracle
date: 2016-01-14 15:18:20
tags: [linux,zabbix,oracle]
---

## 安装zabbix客户端

```
rpm -ivh http://repo.zabbix.com/zabbix/2.4/rhel/6/x86_64/zabbix-release-2.4-1.el6.noarch.rpm
yum install zabbix-agent -y
chkconfig zabbix-agent on
service zabbix-agent start
```

## 安装cx_oracle
上传oracle客户端介质
oracle-instantclient12.1-basic.x86_64
oracle-instantclient12.1-devel.x86_64

```
yum install python-devel -y
yum install oracle-instantclient12.1* -y
```
```
vi .bash_profile
export PATH
export ORACLE_HOME=/usr/lib/oracle/12.1/client64
export TNS_ADMIN=$ORACLE_HOME/network/admin
export PATH=$PATH:$ORACLE_HOME/bin:$ORACLE_HOME/lib:/lib:/usr/lib:$ORACLE_HOME/rdbms/lib
export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib:/usr/lib:$ORACLE_HOME/rdbms/lib
export CLASSPATH=$ORACLE_HOME/JRE:$ORACLE_HOME/jlib:$ORACLE_HOME/rdbms/jlib
PATH=$PATH:$ORACLE_HOME/bin:$ORACLE_HOME/lib:/lib:/usr/lib:$ORACLE_HOME/rdbms/lib
```
这个比较重要，不然后期会报错

```
echo /usr/lib/oracle/12.1/client64/lib >> /etc/ld.so.conf
ldconfig
```
安装cx_oracle

```
pip install cx_oracle
```

## 配置zabbix agent 检查oracle

```
mkdir /etc/zabbix/scripts -p
echo EnableRemoteCommands=1 >> /etc/zabbix/zabbix_agentd.conf
填写zabbix server地址
sed -i 's/127.0.0.1/8.8.8.8/' /etc/zabbix/zabbix_agentd.conf
上传pyora.py至/etc/zabbix/scripts
上传userparameter_oracle.conf至/etc/zabbix/zabbix_agentd.d
service zabbix-agent start
```

## oracle server建立zabbix用户

```
CREATE USER ZABBIX IDENTIFIED BY zabbix;
GRANT CONNECT TO ZABBIX;
GRANT RESOURCE TO ZABBIX;
ALTER USER ZABBIX DEFAULT ROLE ALL;
GRANT SELECT ANY TABLE TO ZABBIX;
GRANT CREATE SESSION TO ZABBIX;
GRANT SELECT ANY DICTIONARY TO ZABBIX;
GRANT UNLIMITED TABLESPACE TO ZABBIX;
GRANT SELECT ANY DICTIONARY TO ZABBIX;
GRANT SELECT ON V_$SESSION TO ZABBIX;
GRANT SELECT ON V_$SYSTEM_EVENT TO ZABBIX;
GRANT SELECT ON V_$EVENT_NAME TO ZABBIX;
GRANT SELECT ON V_$RECOVERY_FILE_DEST TO ZABBIX;
```

## zabbix server 设置
Configuration => Templates => Import => Template Oracle Auto Discovery.xml

设置环境变量

```
{$ADDRESS}
{$DATABASE}
{$PASSWORD}
{$USERNAME}
```

