# Install MongoDB on Linux

```Database``` ```MongoDB```

Sometime back I wrote a post on how to install MongoDB on Mac OS X. However most of the development usually happens on Unix/Linux machines. So today we will look into how to install MongoDB on linux system.


# Install MongoDB on Linux


 Current version of MongoDB is 3.4.7 and I will be installing 64-bit version through command line. The steps to install MongoDB on Linux are very simple, just follow the below terminal commands to download and install it.


1. Download and extract the MongoDB binaries

```
root@dev [/home/journal]# mkdir mongodb
root@dev [/home/journal]# cd mongodb/
root@dev [/home/journal/mongodb]# curl -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-3.4.7.tgz
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 82.7M  100 82.7M    0     0  1704k      0  0:00:49  0:00:49 --:--:-- 1334k
root@dev [/home/journal/mongodb]# tar xvf mongodb-linux-x86_64-3.4.7.tgz
 mongodb-linux-x86_64-3.4.7/README
 mongodb-linux-x86_64-3.4.7/THIRD-PARTY-NOTICES
 mongodb-linux-x86_64-3.4.7/MPL-2
 mongodb-linux-x86_64-3.4.7/GNU-AGPL-3.0
 mongodb-linux-x86_64-3.4.7/bin/mongodump
 mongodb-linux-x86_64-3.4.7/bin/mongorestore
 mongodb-linux-x86_64-3.4.7/bin/mongoexport
 mongodb-linux-x86_64-3.4.7/bin/mongoimport
 mongodb-linux-x86_64-3.4.7/bin/mongostat
 mongodb-linux-x86_64-3.4.7/bin/mongotop
 mongodb-linux-x86_64-3.4.7/bin/bsondump
 mongodb-linux-x86_64-3.4.7/bin/mongofiles
 mongodb-linux-x86_64-3.4.7/bin/mongooplog
 mongodb-linux-x86_64-3.4.7/bin/mongoreplay
 mongodb-linux-x86_64-3.4.7/bin/mongoperf
 mongodb-linux-x86_64-3.4.7/bin/mongod
 mongodb-linux-x86_64-3.4.7/bin/mongos
 mongodb-linux-x86_64-3.4.7/bin/mongo

```


1. Add MongoDB bin directory to PATH variable

```
root@dev [/home/journal/mongodb]# mv mongodb-linux-x86_64-3.4.7 mongodb
root@dev [/home/journal/mongodb]# cd mongodb
root@dev [/home/journal/mongodb/mongodb]# echo $PATH
/usr/local/jdk/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/bin:/usr/X11R6/bin:/root/bin
root@dev [/home/journal/mongodb/mongodb]# export PATH=$PATH:/home/journal/mongodb/mongodb/bin

```


1. Create directory for MongoDB files and start it

```
root@dev [/home/journal/mongodb/mongodb]# mkdir data
root@dev [/home/journal/mongodb/mongodb]# cd bin
root@dev [/home/journal/mongodb/mongodb/bin]# ./mongod --dbpath /home/journal/mongodb/mongodb/data &
[1] 30387
root@dev [/home/journal/mongodb/mongodb/bin]# 2014-08-04T13:56:05.916+0000 [initandlisten] MongoDB starting : pid=30387 port=27017 dbpath=/home/journal/mongodb/mongodb/data 64-bit host=dev.journaldev.com
2014-08-04T13:56:05.917+0000 [initandlisten] db version v3.4.7
2014-08-04T13:56:05.917+0000 [initandlisten] git version: 255f67a66f9603c59380b2a389e386910bbb52cb
2014-08-04T13:56:05.917+0000 [initandlisten] build info: Linux build12.nj1.10gen.cc 2.6.32-431.3.1.el6.x86_64 #1 SMP Fri Jan 3 21:39:27 UTC 2014 x86_64 BOOST_LIB_VERSION=1_49
2014-08-04T13:56:05.917+0000 [initandlisten] allocator: tcmalloc
2014-08-04T13:56:05.917+0000 [initandlisten] options: { storage: { dbPath: "/home/journal/mongodb/mongodb/data" } }
2014-08-04T13:56:05.922+0000 [initandlisten] journal dir=/home/journal/mongodb/mongodb/data/journal
2014-08-04T13:56:05.922+0000 [initandlisten] recover : no journal files present, no recovery needed
2014-08-04T13:56:06.064+0000 [FileAllocator] allocating new datafile /home/journal/mongodb/mongodb/data/local.ns, filling with zeroes...
2014-08-04T13:56:06.064+0000 [FileAllocator] creating directory /home/journal/mongodb/mongodb/data/_tmp
2014-08-04T13:56:06.067+0000 [FileAllocator] done allocating datafile /home/journal/mongodb/mongodb/data/local.ns, size: 16MB,  took 0.001 secs
2014-08-04T13:56:06.069+0000 [FileAllocator] allocating new datafile /home/journal/mongodb/mongodb/data/local.0, filling with zeroes...
2014-08-04T13:56:06.070+0000 [FileAllocator] done allocating datafile /home/journal/mongodb/mongodb/data/local.0, size: 64MB,  took 0.001 secs
2014-08-04T13:56:06.071+0000 [initandlisten] build index on: local.startup_log properties: { v: 1, key: { _id: 1 }, name: "_id_", ns: "local.startup_log" }
2014-08-04T13:56:06.071+0000 [initandlisten] 	 added index to empty collection
2014-08-04T13:56:06.071+0000 [initandlisten] waiting for connections on port 27017

```


1. Use “ps” command to confirm MongoDB is running

```
root@dev [/home/journal/mongodb/mongodb/bin]#
root@dev [/home/journal/mongodb/mongodb/bin]# ps -eaf | grep mongo
root      7199 28009  0 14:09 pts/0    00:00:00 grep mongo
root     30387 28009  0 13:56 pts/0    00:00:02 ./mongod --dbpath /home/journal/mongodb/mongodb/data
root@dev [/home/journal/mongodb/mongodb/bin]#

```


That’s it, our MongoDB is installed on linux machine and running fine. However, you might want to export the PATH through your user profile i.e .bash_profile or .profile, so that it’s not gone once you quit the terminal.


## Execute MongoDB commands


Now let’s connect to the MongoDB and run some mongodb commands to make sure it’s running fine.


```
root@dev [~]# cd /home/journal/mongodb/mongodb/bin/
root@dev [/home/journal/mongodb/mongodb/bin]# ./mongo
MongoDB shell version: 3.4.7
connecting to: test
> show dbs
admin  (empty)
local  0.078GB
> use journaldev
switched to db journaldev
> db.names.save({"id":123,"name":"Pankaj"})
WriteResult({ "nInserted" : 1 })
> db.names.find()
{ "_id" : ObjectId("53df918adbef24e88560fa5b"), "id" : 123, "name" : "Pankaj" }
> db.datas.save({})
WriteResult({ "nInserted" : 1 })
> show collections
datas
names
system.indexes
> show dbs
admin       (empty)
journaldev  0.078GB
local       0.078GB
> exit
bye
root@dev [/home/journal/mongodb/mongodb/bin]# 

```


As you can see that everything seems to be smooth and I am able to save and retrieve data from MongoDB database. If you quit the terminal from which the mongodb was started, it will be stopped. Use nohup command to start it, so that it won’t stop even after you close the terminal. MongoDB Download Page


