title: 'library cache: mutex X'
date: 2016-05-30 14:56:42
tags: [oracle]
---

##### library cache: mutex X

问题描述

1. nmon查看cpu负载过高
2. top查看进程，发现有个进程cpu使用率100%。运行时间超过4000秒

```shell
  PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
 3258 oracle    20   0 32.3g 627m 610m R 97.3  0.5   5019:37 oracle
```

解决方案

```
sqlplus "/ as sysdba"
oradebug setospid X
oradebug unlimit
oradebug Event 10046 trace name context forever, level 12
--Generate trace for 20 minutes
oradebug Event 10046 trace name context off
oradebug tracefile_name
--使用10046进行分析

col SID format a5
col SERIAL# format a5
col STATUS format a10
col EVENT format a30
select s.sid,
       s.serial#,
       s.status,
       s.event,
       s.p1,
       s.p2,
       s.p3
  from v$session s
 where s.process = '3258';
  SID SERIA STATUS     EVENT                                  P1         P2         P3
----- ----- ---------- ------------------------------ ---------- ---------- ----------
  298     5 ACTIVE     library cache: mutex X         3793251071          0         95
--查看sql相关信息

```

其他查询mutex脚本

[mutex](https://github.com/im-vincent/oracle/tree/master/script/mutex)
