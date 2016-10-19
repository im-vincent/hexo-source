title: oracle数据库坏块处理
date: 2016-01-18 16:26:57
tags: [oracle]
---

### oracle数据库坏块处理

常规情况遇到坏块最好使用rman去修复省事实力

```
select * from v$database_block_corruption;
查看坏块

select tablespace_name, segment_type, owner, segment_name
  From dba_extents
 Where file_id = 19
   and 318333 between block_id and block_id + blocks - 1;
查看坏块对应的对象

RMAN> recover corruption list; 
RMAN> recover datafile xxx  block xxx
使用rman修复坏块
```


这次遇到的问题

1. 数据库无备份
2. 坏块所在对象为一个测试用户

开始想直接把用户删掉，重新导入。但是试验后发现，v$database_block_corruption的信息并不能删除

在网上查询，使用expdp/impdp可以修复坏块，但是会丢数据。本来也是测试数据无所谓了。 

expdp

```
DUMPFILE="bmi_tianjin.dmp"
LOGFILE="exp_bmi_tianjin.log"
DIRECTORY=DUMP_DIR
COMPRESSION=ALL
CONTENT=ALL
SCHEMAS=('BMI_TIANJIN')
```

impdp

```
DUMPFILE="bmi_tianjin.dmp"
LOGFILE="imp_bmi_tianjin.log"
DIRECTORY=DUMP_DIR
STREAMS_CONFIGURATION=n
TABLE_EXISTS_ACTION=TRUNCATE
SKIP_UNUSABLE_INDEXES=y
CONTENT=ALL
PARTITION_OPTIONS=none
```

rman

```
run
{
allocate channel c1 type disk maxpiecesize 64g;
allocate channel c2 type disk maxpiecesize 64g;
allocate channel c3 type disk maxpiecesize 64g;
allocate channel c4 type disk maxpiecesize 64g;
backup validate database;
release channel c1;
release channel c2;
release channel c3;
release channel c4;
}
个人感觉只校验那个有问题的数据文件也可以，没有实验
```
