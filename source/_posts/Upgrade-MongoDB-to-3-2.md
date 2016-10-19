title: Upgrade MongoDB to 3.2
date: 2016-01-12 15:52:29
tags: [mongodb]
---


## mongodb 3.2 Upgrade Config Servers to Replica Set
参考文档 https://docs.mongodb.org/manual/tutorial/upgrade-config-servers-to-replica-set/
本实验默认使用wiredTiger进行演示,mmapv1有所不同，具体请参考官方文档

### 安装演示环境

```
安装gcc
[root@5434b45f63a1 src]# yum install gcc python-devel tar

使用pip安装mtools
[root@5434b45f63a1 src]# curl -O https://bootstrap.pypa.io/get-pip.py
[root@5434b45f63a1 src]# python get-pip.py
[root@5434b45f63a1 src]# pip install mtools pymongo

安装mongodb 3.2
[root@5434b45f63a1 src]# curl -OL https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel62-3.2.1.tgz
[root@5434b45f63a1 src]#tar xvfz mongodb-linux-x86_64-rhel62-3.2.1.tgz
[root@5434b45f63a1 src]# mv mongodb-linux-x86_64-rhel62-3.2.1 /usr/local/
[root@5434b45f63a1 src]# cd /usr/local/
[root@5434b45f63a1 local]# ln -s mongodb-linux-x86_64-rhel62-3.2.1/ mongodb

```
### 使用mtools建立分片环境

```
[root@5434b45f63a1 ~]# mlaunch --replicaset 3 --sharded 3 --config 3 --mongos 1

[root@5434b45f63a1 ~]# mlaunch list

PROCESS          PORT     STATUS     PID

mongos           27017    running    876

config server    27027    running    591
config server    27028    running    605
config server    27029    running    625

shard01
    mongod       27018    running    384
    mongod       27019    running    407
    mongod       27020    running    430

shard02
    mongod       27021    running    453
    mongod       27022    running    476
    mongod       27023    running    499

shard03
    mongod       27024    running    522
    mongod       27025    running    545
    mongod       27026    running    568
    
```

### 升级config至副本集
1.	关闭balancer

	```
	sh.stopBalancer()
	sh.getBalancerState()
	
	```

2.  配置config服务

	```
	--configdb 5434b45f63a1:27027,5434b45f63a1:27028,5434b45f63a1:27029
	5434b45f63a1:27027 为第一台config
	
	rs.initiate( {
   		_id: "csReplSet",
   		version: 1,
   		configsvr: true,
   		members: [ { _id: 0, host: "5434b45f63a1:27027" } ]
		} )
		
	重启config
	mongod --configsvr --replSet csReplSet --configsvrMode=sccc --storageEngine wiredTiger --port 27027 --dbpath <path>
	
	```
	
3.  添加config服务
	
	```
	Do not add existing config servers to the replica set.
	Use new dbpaths for the new instances.
	
	如果是mmapv1 需要加3个新的config
	如果是WiredTiger 需要添加2个新的config
	
	rs.add( { host: <host:port>, priority: 0, votes: 0 } )
	
	```
4.  关闭原来的config
5.  设置replica set config权重
	
	```
	var cfg = rs.conf();

	cfg.members[0].priority = 1;
	cfg.members[1].priority = 1;
	cfg.members[2].priority = 1;
	cfg.members[3].priority = 1;
	cfg.members[0].votes = 1;
	cfg.members[1].votes = 1;
	cfg.members[2].votes = 1;
	cfg.members[3].votes = 1;

	rs.reconfig(cfg);
	
	```
6. 切换第一台config 	the server started with --configsvrMode=sccc.

	```
	rs.stepDown()
	
	```
7.	开启第一台config

	```
	mongod --configsvr --replSet csReplSet --storageEngine wiredTiger --port <port> --dbpath 	<path>
	
	```
8.	重启mongos

	```
	mongos --configdb csReplSet/<rsconfigsver1:port1>,<rsconfigsver2:port2>,<rsconfigsver3:port3>

	```



