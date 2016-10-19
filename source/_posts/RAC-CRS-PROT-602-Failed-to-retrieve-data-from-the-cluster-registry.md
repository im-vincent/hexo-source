title: 'RAC CRS PROT-602: Failed to retrieve data from the cluster registry'
date: 2015-10-12 10:58:07
tags: [oracle]
---

最近巡检发现crs检测发现有点异常，但是业务没有收到影响，记录下

###ocrcheck###
```
PROT-602: Failed to retrieve data from the cluster registry
PROC-26: Error while accessing the physical storage
```

###crsctl query css votedisk###
```
##  STATE    File Universal Id                File Name Disk group
--  -----    -----------------                --------- ---------
 1. ONLINE   4288e5de20214f1bbfaae80747c8df2c (/dev/mapper/ocr_01) [OCR]
 2. ONLINE   44a9e2ae3f064fafbf78ddd425d8a399 (/dev/mapper/ocr_02) [OCR]
 3. ONLINE   5914cd18dd654f77bf1b893b5b9d73c4 (/dev/mapper/ocr_03) [OCR]
Located 3 voting disk(s).
```


###crsctl status res -t###
```
--------------------------------------------------------------------------------
NAME           TARGET  STATE        SERVER                   STATE_DETAILS       
--------------------------------------------------------------------------------
Local Resources
--------------------------------------------------------------------------------
ora.ARCH.dg
               ONLINE  ONLINE       XXXX01                                      
               ONLINE  ONLINE       XXXX02                                      
ora.DATA.dg
               ONLINE  ONLINE       XXXX01                                      
               ONLINE  ONLINE       XXXX02                                      
ora.LISTENER.lsnr
               ONLINE  ONLINE       XXXX01                                      
               ONLINE  ONLINE       XXXX02                                      
ora.OCR.dg
               OFFLINE OFFLINE      XXXX01                                      
               ONLINE  ONLINE       XXXX02                                      
ora.asm
               ONLINE  ONLINE       XXXX01                  Started             
               ONLINE  ONLINE       XXXX02                  Started             
ora.gsd
               OFFLINE OFFLINE      XXXX01                                      
               OFFLINE OFFLINE      XXXX02                                      
ora.net1.network
               ONLINE  ONLINE       XXXX01                                      
               ONLINE  ONLINE       XXXX02                                      
ora.ons
               ONLINE  ONLINE       XXXX01                                      
               ONLINE  ONLINE       XXXX02       

```

```

检查asm磁盘状态
su - grid

sqlplus / as sysasm
col name format a15
col path format a17
select name,path,header_status,mount_status,state from v$asm_disk;


挂载被dismount磁盘
asmcmd

mount -a

检查asm日志
sqlplus / as sysasm

SQL> show parameter dump

NAME                                 TYPE        VALUE
------------------------------------ ----------- ------------------------------
background_core_dump                 string      partial
background_dump_dest                 string      /u01/app/grid/diag/asm/+asm/+A
                                                 SM2/trace  <---asm实例日志
core_dump_dest                       string      /u01/app/grid/diag/asm/+asm/+A
                                                 SM2/cdump
max_dump_file_size                   string      unlimited
shadow_core_dump                     string      partial
user_dump_dest                       string      /u01/app/grid/diag/asm/+asm/+A
                                                 SM2/trace


tail -100 /u01/app/grid/diag/asm/+asm/+ASM2/trace
WARNING: Waited 15 secs for write IO to PST disk 0 in group 2.
WARNING: Waited 15 secs for write IO to PST disk 0 in group 2.
WARNING: Waited 15 secs for write IO to PST disk 0 in group 3.
WARNING: Waited 15 secs for write IO to PST disk 1 in group 3.
WARNING: Waited 15 secs for write IO to PST disk 2 in group 3.
WARNING: Waited 15 secs for write IO to PST disk 0 in group 3.
WARNING: Waited 15 secs for write IO to PST disk 1 in group 3.
WARNING: Waited 15 secs for write IO to PST disk 2 in group 3.
Fri Dec 04 01:48:17 2015
NOTE: process _b000_+asm1 (99207) initiating offline of disk 0.3916016564 (OCR_0000) with mask 0x7e in group 3
NOTE: process _b000_+asm1 (99207) initiating offline of disk 1.3916016563 (OCR_0001) with mask 0x7e in group 3
NOTE: process _b000_+asm1 (99207) initiating offline of disk 2.3916016562 (OCR_0002) with mask 0x7e in group 3
NOTE: checking PST: grp = 3
GMON checking disk modes for group 3 at 22 for pid 27, osid 99207
ERROR: no read quorum in group: required 2, found 0 disks
NOTE: checking PST for grp 3 done.
NOTE: initiating PST update: grp = 3, dsk = 0/0xe969abb4, mask = 0x6a, op = clear
NOTE: initiating PST update: grp = 3, dsk = 1/0xe969abb3, mask = 0x6a, op = clear
NOTE: initiating PST update: grp = 3, dsk = 2/0xe969abb2, mask = 0x6a, op = clear
GMON updating disk modes for group 3 at 23 for pid 27, osid 99207
ERROR: no read quorum in group: required 2, found 0 disks
Fri Dec 04 01:48:17 2015
NOTE: cache dismounting (not clean) group 3/0xD0095B7D (OCR)
WARNING: Offline for disk OCR_0000 in mode 0x7f failed.
WARNING: Offline for disk OCR_0001 in mode 0x7f failed.
WARNING: Offline for disk OCR_0002 in mode 0x7f failed.
NOTE: messaging CKPT to quiesce pins Unix process pid: 99209, image: oracle@zjbmi01 (B001)
Fri Dec 04 01:48:17 2015
NOTE: halting all I/Os to diskgroup 3 (OCR)
Fri Dec 04 01:48:18 2015
NOTE: LGWR doing non-clean dismount of group 3 (OCR)
NOTE: LGWR sync ABA=13.88 last written ABA 13.88
Fri Dec 04 01:48:18 2015
NOTE: No asm libraries found in the system
Fri Dec 04 01:48:18 2015
kjbdomdet send to inst 2
detach from dom 3, sending detach message to inst 2
Fri Dec 04 01:48:18 2015
List of instances:
 1 2
Dirty detach reconfiguration started (new ddet inc 1, cluster inc 8)
 Global Resource Directory partially frozen for dirty detach
* dirty detach - domain 3 invalid = TRUE
 770 GCS resources traversed, 0 cancelled
Dirty Detach Reconfiguration complete
Fri Dec 04 01:48:18 2015
WARNING: dirty detached from domain 3
NOTE: cache dismounted group 3/0xD0095B7D (OCR)
SQL> alter diskgroup OCR dismount force /* ASM SERVER:3490274173 */
```

bug 相关信息

```

redhat 'multipathd' and disk checker recognises failed disks later then Oracle ASM
https://access.redhat.com/solutions/1241173

oracle bug 17274537
http://blog.csdn.net/wuweilong/article/details/41042145

```
