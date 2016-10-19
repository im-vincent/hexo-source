title: 编译oracle失效对象
date: 2015-12-17 16:47:28
tags: [oracle]
---

编译oracle 失效对象

* 执行@$ORACLE_HOME/rdbms/admin/utlrp.sql脚本编译数据库失效对象。

```
sqlplus / as sysdba
@$ORACLE_HOME/rdbms/admin/utlrp.sql
```
* ORACLE提供了自动编译的接口dbms_utility.compile_schema(user,false); 调用这个过程就会编译所有失效的过程、函数、触发器、包

```
exec dbms_utility.compile_schema( 'SCOTT' )
```
