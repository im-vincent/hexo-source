title: chunks碎片处理
date: 2015-10-08 11:26:33
tags: [mongodb]
---

> http://docs.mongodb.org/manual/reference/method/db.collection.getShardDistribution/

## 查询chunk碎片
db.collection.getShardDistribution()


```
Shard shard-a at shard-a/MyMachine.local:30000,MyMachine.local:30001,MyMachine.local:30002
data : 38.14Mb docs : 1000003 chunks : 2
estimated data per chunk : 19.07Mb
estimated docs per chunk : 500001

Shard shard-b at shard-b/MyMachine.local:30100,MyMachine.local:30101,MyMachine.local:30102
data : 38.14Mb docs : 999999 chunks : 3
estimated data per chunk : 12.71Mb
estimated docs per chunk : 333333

Totals
data : 76.29Mb docs : 2000002 chunks : 5
Shard shard-a contains 50% data, 50% docs in cluster, avg obj size on shard : 40b
Shard shard-b contains 49.99% data, 49.99% docs in cluster, avg obj size on shard : 40b
```


## 合并chunk减少碎片
> https://github.com/snmaynard/mongo-scripts

### MergeChunks.js

This script allows you to merge your empty chunks across your entire cluster. Run the script on a mongos instance attached to 2.6 mongods. The script contains a function that takes the following arguments,

mergeChunks(dbName, collection, keyPattern, sleepValue = 0)

- dbName: Set this to the database name
- collection: Set this to the collection you want to merge chunks in
- keyPattern: Set this to the key pattern, ie { "_id" : "hashed" }
- sleepValue: How long to sleep between each chunk (for backoff) - optional

### SplitChunks

This script allows you to split your chunks on a single shard. Useful when a shard contains significantly more data than others. The script is built using ruby on mongoid as it needs to connect to both a mongos and a mongod. The script contains a function that takes the following arguments,

split_chunks(db, collection, key_pattern, shard, primary_mongod, split_min = 10, mongos = "localhost:27017")

- db: The database name to split chunks in
- collection: The collection to split chunks in
- key_pattern: The key pattern used to shard the collection, i.e. {"_id" => 1}
- shard: The id of the shard to split chunks on (check sh.status())
- primary_mongod: The address and port of the mongod i.e. "mongo.example.com:27017"
- split_min: The minimum number of splits required in a chunk to actually split it - optional
- mongos: The address and port of the mongos - optional
