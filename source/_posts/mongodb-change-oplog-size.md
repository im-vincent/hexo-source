title: mongodb change oplog size
date: 2015-11-11 10:35:30
tags: [mongodb]
---

```
use local
db = db.getSiblingDB('local')
db.temp.drop()
db.temp.save( db.oplog.rs.find( { }, { ts: 1, h: 1 } ).sort( {$natural : -1} ).limit(1).next() )
db.temp.find()
db = db.getSiblingDB('local')
db.oplog.rs.drop()
db.runCommand( { create: "oplog.rs", capped: true, size: (2 * 1024 * 1024 * 1024) } )
db.oplog.rs.save( db.temp.findOne() )
db.oplog.rs.find()

```
