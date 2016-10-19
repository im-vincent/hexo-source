title: rman多路径备份
date: 2015-10-15 10:46:59
tags: [oracle]
---
#### 建立备份目录

```
cd /backup
mkdir data rman logs arch
```
#### 配置rman

```
rman target /

CONFIGURE DATAFILE BACKUP COPIES FOR DEVICE TYPE DISK TO 2;
CONFIGURE ARCHIVELOG BACKUP COPIES FOR DEVICE TYPE DISK TO 2;
```

#### 配置rman_backup_twoaddr.sh

```
#!/bin/ksh
export LANG=en_US
BACKUP_DATE=`date +%d`
RMAN_LOG_FILE=/opt/backup/logs/rmanlog`date +%Y%m%d`.out
TODAY=`date`
USER=`id|cut -d "(" -f2|cut -d ")" -f1`
echo "-----------------$TODAY-------------------">$RMAN_LOG_FILE
ORACLE_HOME=/opt/ora10g/db_1
export ORACLE_HOME
RMAN=$ORACLE_HOME/bin/rman
export RMAN
ORACLE_SID=zmedi
export ORACLE_SID
ORACLE_USER=oracle
export ORACLE_USER

echo "ORACLE_SID: $ORACLE_SID">>$RMAN_LOG_FILE
echo "ORACLE_HOME:$ORACLE_HOME">>$RMAN_LOG_FILE
echo "ORACLE_USER:$ORACLE_USER">>$RMAN_LOG_FILE
echo "==========================================">>$RMAN_LOG_FILE
echo "BACKUP DATABASE BEGIN......">>$RMAN_LOG_FILE
echo "                   ">>$RMAN_LOG_FILE
chmod 666 $RMAN_LOG_FILE

WEEK_DAILY=`date +%a`
case  "$WEEK_DAILY" in
       "Mon")
            BAK_LEVEL=2
            ;;
       "Tue")
            BAK_LEVEL=2
            ;;
       "Wed")
            BAK_LEVEL=2
            ;;
       "Thu")
            BAK_LEVEL=1
            ;;
       "Fri")
            BAK_LEVEL=2
            ;;
       "Sat")
            BAK_LEVEL=2
            ;;
       "Sun")
            BAK_LEVEL=0
            ;;
       "*")
            BAK_LEVEL=error
esac

export BAK_LEVEL=$BAK_LEVEL
echo "Today is : $WEEK_DAILY  incremental level= $BAK_LEVEL">>$RMAN_LOG_FILE

RUN_STR="
BAK_LEVEL=$BAK_LEVEL
export BAK_LEVEL
ORACLE_HOME=$ORACLE_HOME
export ORACLE_HOME
ORACLE_SID=$ORACLE_SID
export ORACLE_SID
$RMAN nocatalog TARGET / msglog $RMAN_LOG_FILE append <<EOF
run
{
allocate channel c1 type disk MAXPIECESIZE=64G;
allocate channel c2 type disk MAXPIECESIZE=64G;
allocate channel c3 type disk MAXPIECESIZE=64G;
allocate channel c4 type disk MAXPIECESIZE=64G;
backup as compressed backupset incremental level= $BAK_LEVEL  skip inaccessible  
Database format='/opt/backup/data/lev"$BAK_LEVEL"_%d_%U','/opt/nfsbackup/246zgwdb/rmanbackup/data/lev"$BAK_LEVEL"_%d_%U' tag='lev"$BAK_LEVEL"'
plus archivelog format='/opt/backup/data/arch_%d_%U','/opt/nfsbackup/246zgwdb/rmanbackup/data/arch_%d_%U' delete input;
release channel c1;
release channel c2;
release channel c3;
release channel c4;
}
allocate channel for maintenance device type disk;
report obsolete;
delete noprompt obsolete;
crosscheck backup;
delete noprompt expired backup;
list backup summary;
release channel;
EOF
"
 # Initiate the command string

if [ "$CUSER" = "root" ]
then
    echo "Root Command String: $RUN_STR" >> $RMAN_LOG_FILE    
    su - $ORACLE_USER -c "$RUN_STR" >> $RMAN_LOG_FILE
    RSTAT=$?
else
    echo "User Command String: $RUN_STR" >> $RMAN_LOG_FILE    
    /bin/sh -c "$RUN_STR" >> $RMAN_LOG_FILE
    RSTAT=$?
fi

# ---------------------------------------------------------------------------
# Log the completion of this script.
# ---------------------------------------------------------------------------

if [ "$RSTAT" = "0" ]
then
    LOGMSG="ended successfully"
else
    LOGMSG="ended in error"
fi
echo >> $RMAN_LOG_FILE
echo Script $0 >> $RMAN_LOG_FILE
echo ==== $LOGMSG on `date` ==== >> $RMAN_LOG_FILE
echo >> $RMAN_LOG_FILE
exit $RSTAT



```
