# Deploying MongoDB With Redundancy

```Security``` ```MongoDB``` ```Deployment``` ```Backups```

No matter what precautions you or your cloud provider take to prevent them, computers are always at risk of hardware failure. An important part of managing any computer system, not just a MongoDB installation, is to make regular backups of your important information. By taking and storing backups of your data, you can restore your application to working order if your database server crashes and your original data is lost.


Just as you should regularly back up your MongoDB data, it’s equally important that you store those backups in a separate location from the server hosting your database. If you were to store your backups in the same data center as your database, both the database and your backups would be unavailable if the data center were to experience a failure and you wouldn’t be able to use the backups to get your application back online.


Replication is a practice that’s similar to making backups: where making a backup involves taking a point-in-time snapshot of all the data held in a database, replication involves constantly synchronizing data across multiple separate databases. It’s often useful to have multiple replicas of your data, as this provides redundancy in case one of the database servers fails and can also improve a database’s availability and scalability, as well as reduce read latencies. In MongoDB, a group of servers that maintain the same data set through replication are referred to as a replica set.


The official documentation recommends that any Mongo database used in a production environment be deployed as a replica set, since MongoDB replica sets employ a feature known as automatic failover. This means that if the primary fails and is unable to communicate with the secondary members for a predetermined amount of time, the secondary members will automatically elect a new primary member, thereby ensuring that your data remains available to your application or the clients that depend on it.


## Related Resources


- The Importance of Offsite Backups
- How To Back Up, Restore, and Migrate a MongoDB Database on Ubuntu 20.04
- How To Configure a MongoDB Replica Set on Ubuntu 20.04

