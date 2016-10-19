title: 'CRS-0184: Cannot communicate with the CRS daemon'
date: 2015-09-25 15:28:00
tags: [oracle]
---
- crs_stat-t/crsctl status res -t
CRS-0184: Cannot communicate with the CRSdaemon.

- crsctlcheck crs
CRS-4638: Oracle High Availability Servicesis online
CRS-4535: Cannot communicate with ClusterReady Services
CRS-4529: Cluster Synchronization Servicesis online
CRS-4533: Event Manager is online

- 检查message
zjbmi01
Aug 25 09:23:08 zjbmi01 kernel: pcieport 0000:00:03.0: em3: Link is down
Aug 25 09:23:08 zjbmi01 kernel: pcieport 0000:00:03.1: em1: Link is down
Aug 25 09:23:08 zjbmi01 kernel: pcieport 0000:00:03.1: em2: Link is down
zjbmi02
Aug 25 09:23:08 zjbmi02 kernel: pcieport 0000:00:03.0: em3: Link is down
Aug 25 09:23:08 zjbmi02 kernel: pcieport 0000:00:03.1: em2: Link is down
Aug 25 09:23:08 zjbmi02 kernel: pcieport 0000:00:03.1: em1: Link is down

- 检查crs日志
2015-08-25 09:23:09.938: [    AGFW][2849957632]{0:3:1071} Received state change for ora.net1.network zjbmi01 1 [old state = ONLINE, new state = OFFLINE]
2015-08-25 09:23:09.938: [    AGFW][2849957632]{0:3:1071} Agfw Proxy Server sending message to PE, Contents = [MIDTo:2|OpID:3|FromA:{Invalid|Node:0|Process:0|Type:0}|ToA:{Invalid|Node:-1|Process:-1|Type:-1}|MIDFrom:0|Type:4|Pri2|Id:163360:Ver:2]
2015-08-25 09:23:09.938: [    AGFW][2849957632]{0:3:1071} Agfw Proxy Server replying to the message: RESOURCE_STATUS[Proxy] ID 20481:3076255
2015-08-25 09:23:09.938: [   CRSPE][2839451392]{0:3:1071} State change received from zjbmi01 for ora.net1.network zjbmi01 1
2015-08-25 09:23:09.939: [   CRSPE][2839451392]{0:3:1071} Processing PE command id=7647. Description: [Resource State Change (ora.net1.network zjbmi01 1) : 0x7fdf380f5830]
2015-08-25 09:23:09.939: [   CRSPE][2839451392]{0:3:1071} RI [ora.net1.network zjbmi01 1] new external state [OFFLINE] old value: [ONLINE] on zjbmi01 label = []
2015-08-25 09:23:09.939: [    CRSD][2839451392]{0:3:1071} {0:3:1071} Resource Resource Instance ID[ora.net1.network zjbmi01 1]. Values:
确定网络问题被驱逐

- 检查asm_disk是否正常
```
$sqlplus/ as sysasm
SQL>col name format a20
SQL>col path format a20
SQL>select name,path,header_status,mount_status,state from v$asm_disk;
$asmcmd
mount -a
```
重新挂载后一切正常

- reboot os
init 6
