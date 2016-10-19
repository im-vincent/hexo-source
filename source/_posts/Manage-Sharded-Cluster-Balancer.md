title: Manage Sharded Cluster Balancer
date: 2015-11-17 15:18:06
tags: [mongodb]
---

https://docs.mongodb.org/manual/tutorial/manage-sharded-cluster-balancer/

#### Check the Balancer State

The following command checks if the balancer is enabled (i.e. that the balancer is allowed to run). The command does not check if the balancer is active (i.e. if it is actively balancing chunks).

To see if the balancer is enabled in your cluster, issue the following command, which returns a boolean:

```
sh.getBalancerState()
```

#### Check the Balancer Lock

To see if the balancer process is active in your cluster, do the following:

Connect to any mongos in the cluster using the mongo shell.
Issue the following command to switch to the Config Database:
use config
Use the following query to return the balancer lock:

```
db.locks.find( { _id : "balancer" } ).pretty()
```

#### Schedule the Balancing Window
```
use config
sh.setBalancerState( true )
db.settings.update(
   { _id: "balancer" },
   { $set: { activeWindow : { start: "23:00", stop: "6:00" } } },
   { upsert: true }
)

```

#### Remove a Balancing Window Schedule

If you have set the balancing window and wish to remove the schedule so that the balancer is always running, issue the following sequence of operations:

```
use config
db.settings.update({ _id : "balancer" }, { $unset : { activeWindow : true } })
```
