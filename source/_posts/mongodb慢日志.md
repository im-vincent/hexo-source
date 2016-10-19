title: mongodb慢日志
date: 2015-10-08 10:57:27
tags: [mongodb]
---



[官方文档](http://docs.mongodb.org/manual/tutorial/manage-the-database-profiler/ "Title")


> http://blog.csdn.net/mchdba/article/details/8950632
> 参考博客 



### Profiling Levels

The following profiling levels are available:

- 0 - the profiler is off, does not collect any data. mongod always writes operations longer than the slowOpThresholdMs threshold to its log.
- 1 - collects profiling data for slow operations only. By default slow operations are those slower than 100 milliseconds.
You can modify the threshold for “slow” operations with the slowOpThresholdMs runtime option or the setParameter command. See the Specify the Threshold for Slow Operations section for more information.
- 2 - collects profiling data for all database operations.


### Enable Database Profiling and Set the Profiling Level
```
db.setProfilingLevel(2) #全部开启

db.setProfilingLevel(1,20) #超过20ms的记录
```

### Check Profiling Level
```
db.getProfilingStatus()
db.getProfilingLevel()

```

### Example Profiler Data Queries
```
db.system.profile.find().limit(10).sort( { ts : -1 } ).pretty()
db.system.profile.find( { op: { $ne : 'command' } } ).pretty()
db.system.profile.find( { ns : 'mydb.test' } ).pretty()
db.system.profile.find( { millis : { $gt : 5 } } ).pretty()

查看执行时间大于100毫秒的执行操作，并倒序排列,并取前5行
db.system.profile.find({millis:{$gt:100}}).sort({$natural:-1}).limit(5);
```


### Change Size of system.profile Collection on the Primary
```
db.setProfilingLevel(0)

db.system.profile.drop()

db.createCollection( "system.profile", { capped: true, size:4000000 } )

db.setProfilingLevel(1)
```
