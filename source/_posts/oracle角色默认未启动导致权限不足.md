title: oracle角色默认未启动导致权限不足
date: 2016-05-27 13:38:46
tags: [oracle]
---
### 问题描述
查询用户有connect，resource角色，但是无法建立或删除表

### 解决方案

```
SQL> select * from user_role_privs;
USERNAME   GRANTED_RO ADMIN_OPTION DEFAULT_ROLE OS_GRANTED
---------- ---------- ------------ ------------ ----------
TEST       CONNECT    NO           YES          NO
TEST       RESOURCE   NO           NO           NO
首先查看用户角色权限。有RESOURCE权限，但是被关闭了

SQL> select * from session_privs;
PRIVILEGE
----------------------------------------
CREATE SESSION
UNLIMITED TABLESPACE
ANALYZE ANY
使用session_privs再次确认无权限。

SQL> create table t as select * from user_users;
create table t as select * from user_users
ORA-01031: insufficient privileges
无权限

SQL> set role resource;
Role set
激活权限

SQL> create table t as select * from user_users;
Table created
激活后权限就有了。

SQL> conn dba/dba
SQL> alter user test default role all;
SQL> select * from user_role_privs;
USERNAME   GRANTED_RO ADMIN_OPTION DEFAULT_ROLE OS_GRANTED
---------- ---------- ------------ ------------ ----------
TEST       CONNECT    NO           YES          NO
TEST       RESOURCE   NO           YES          NO
解决完成
```
