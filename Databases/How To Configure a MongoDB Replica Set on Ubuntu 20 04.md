# How To Configure a MongoDB Replica Set on Ubuntu 20 04

```Databases``` ```Security``` ```Ubuntu``` ```MongoDB``` ```High Availability``` ```NoSQL```

An earlier version of this tutorial was written by Justin Ellingwood.


## Introduction


MongoDB is a document-oriented database used in many modern web applications. It is classified as a NoSQL database because it does not rely on a traditional table-based relational database structure. Instead, it uses JSON-like documents with dynamic schemas. This means that, unlike relational databases, MongoDB does not require a predefined schema before you add data to a database.


When working with databases, it’s often useful to have multiple copies of your data. This provides redundancy in case one of the database servers fails and can improve a database’s availability and scalability, as well as reduce read latencies. The practice of synchronizing data across multiple separate databases is called replication. In MongoDB, a group of servers that maintain the same data set through replication are referred to as a replica set.


This tutorial provides a brief overview of how replication works in MongoDB and outlines how to configure and initiate a replica set with three members. In this example configuration, each member of the replica set will be a distinct MongoDB instance running on separate Ubuntu 20.04 servers.



Note: Be aware that the procedure outlined in this guide is intended to demonstrate how to get a replica set up and running quickly. Upon completing this tutorial you’ll have a functioning replica set, but it will not have any security features enabled. This setup is not recommended for production environments.
The Community version of MongoDB comes with two authentication methods that can help keep your database secure, keyfile authentication and x.509 authentication. For production deployments that employ replication, the MongoDB documentation recommends using x.509 authentication, and it describes keyfiles as “bare-minimum forms of security” that are “best suited for testing or development environments.” However, the process of obtaining and configuring x.509 certificates comes with a number of caveats and decisions that must be made on a case-by-case basis, which is beyond the scope of a DigitalOcean tutorial.
If you plan on using your replica set for testing or development, we strongly encourage you to follow our tutorial on How To Configure Keyfile Authentication for MongoDB Replica Sets on Ubuntu 20.04.

# Prerequisites


To complete this guide, you will need:


- Three servers, each running Ubuntu 20.04. All three of these servers should have a non-root administrative user and a firewall configured with UFW. To set this up, follow our initial server setup guide for Ubuntu 20.04.
- MongoDB installed on each of your Ubuntu servers. To this end, follow our tutorial on How To Install MongoDB on Ubuntu 20.04, making sure to complete each step on all three of your servers.

Please note that, for clarity, this guide will refer to the three servers as mongo0, mongo1, and mongo2. Any examples showing commands or file changes performed on mongo0 will have a blue background, like this:


```



```


Commands and file changes performed on mongo1 will have a pink background:


```



```


Example actions on mongo2 will have a green background:


```



```


Lastly, commands that must be run or file changes that must be made on every server will have a standard black background, like this:


```



```


# Understanding MongoDB Replica Sets


As mentioned in the introduction, MongoDB handles replication through an implementation called replica sets. Each running instance of MongoDB that’s part of a given replica set is referred to as one of its members. Every replica set must have one primary member and at least one secondary member.


The primary member is the main access point for transactions with the replica set and is the only member that can accept write operations. Each replica set can have only one primary member at a time, as replication happens by copying the primary’s oplog (short for “operations log”) and repeating the logged changes on the secondaries’ respective data sets. Multiple primaries accepting write operations would lead to data conflicts.


By default, applications will only query the primary member for both read and write operations. You can configure your setup to read from one or more of the secondary members, but since data is transferred asynchronously, reads from secondary nodes can result in old data being served. Thus, such a configuration isn’t ideal for every use case.


One feature that distinguishes MongoDB’s replica sets from other replication implementations is their automatic failover mechanism. In the event that the primary member becomes unavailable, an automated election process happens among the secondary nodes to choose a new primary. A replica set can have up to 50 members, but a maximum of 7 can vote in an election.


If the secondary member pool contains an even number of nodes, however, it could result in an inability to elect a new primary due to a voting impasse. This would necessitate the inclusion of a third type of member in the replica set: an arbiter. An arbiter is an optional member of a replica set that votes in situations like this to ensure that the set is able to reach a decision. Be aware, though, that arbiters do not have a copy of the data set and they’re barred from becoming the replica set’s primary. If a replica set has only one secondary member, then an arbiter is required.


There may be times when you don’t want all of your secondaries to follow the standard rules for secondary members of a replica set. MongoDB allows you to configure secondary members of a replica set to take on the following nonstandard roles:


- Priority 0 Replication Members: There are some situations where the election of certain set members to the primary position could have a negative impact on your application’s performance. For instance, if you are replicating data to a remote datacenter or a certain secondary member’s hardware is inadequate for it to function as the main access point for the set, setting its priority to 0 can ensure that this member will not become a primary but can continue copying data.
- Hidden Replication Members: Some situations require you to keep one set of members accessible and visible to your clients while hiding background members which have separate purposes and shouldn’t be used for read operations. As an example, you may need a secondary member to be the base for analytics work, which would benefit from an up-to-date dataset but would cause a strain on working members. By setting this member to hidden, it will not interfere with the general operations of the replica set. Hidden members must be set to a priority of 0 to avoid becoming the primary member, but they can vote in elections.
- Delayed Replication Members: By setting the delay option for a secondary member, you can control how long the secondary waits to perform each action it copies from the primary’s oplog. This is useful if you would like to safeguard against accidental deletions or recover from destructive operations. For instance, if you delay a secondary by a half-day, it would not immediately perform accidental operations on its own set of data and could be used to revert changes. Delayed members cannot become primary members, but can vote in elections. In most situations, they should also be hidden to prevent application processes from reading data that is out-of-date.

# Step 1 — Configuring DNS Resolution


When it comes time to initialize your replica set in Step 4, you’ll need to provide an address where each replica set member can be reached by the other two in the set. The MongoDB documentation recommends against using IP addresses when configuring a replica set, since IP addresses can change unexpectedly. Instead, MongoDB recommends using logical DNS hostnames when configuring replica sets.


One way to do this is to configure subdomains for each replication member. Although configuring subdomains would be ideal for a production environment or another long-term solution, this tutorial will outline how to configure DNS resolution by editing each server’s respective hosts files.


hosts is a special file that allows you to assign human-readable hostnames to numerical IP addresses. This means that if the IP address of any of your servers ever changes, you’ll only have to update the hosts file on the three servers instead of reconfiguring the replica set.


On Linux and other Unix-like systems, hosts is stored in the /etc/ directory. On each of your three servers, edit the file with your preferred text editor. Here, we’ll use nano:


```
sudo nano /etc/hosts


```


After the first few lines which configure the localhost, add an entry for each member of the replica set. These entries take the form of an IP address followed by the human-readable name of your choice, as in this example:


/etc/hosts
```
IP_address   any_hostname

```


You can configure your servers to use whatever hostname you’d like, but it can be helpful to make each hostname descriptive. In examples throughout this guide, the three servers will use these hostnames:


- mongo0.replset.member
- mongo1.replset.member
- mongo2.replset.member

Using these hostnames, your /etc/hosts files would look similar to the following highlighted lines:


/etc/hosts
```
. . .
127.0.0.1 localhost

203.0.113.0 mongo0.replset.member
203.0.113.1 mongo1.replset.member
203.0.113.2 mongo2.replset.member
. . .

```



If you don’t know your servers’ IP addresses offhand, you can run the following curl command on each server to retrieve them. icanhazip.com is a website that shows the IP address of whatever computer is used to access it. By providing its URL as an argument to the curl command, the command will print the IP address of the server from which you run it to standard output:
curl -4 icanhazip.com


If you’re using DigitalOcean Droplets, you can also find your servers’ IP addresses in the Control Panel.

The new lines you add here should be identical on each of the three hosts in your set. Save and close the file on each of your servers. If you used nano to edit these files, do so by pressing CTRL + X, Y, and then ENTER.


After editing, saving, and closing the hosts file on each of your servers, you’ll have finished configuring DNS resolution for your replica set. You can now move on to updating each server’s firewall rules to allow them to communicate with one another.


# Step 2 — Updating Each Server’s Firewall Configurations with UFW


Assuming you followed the prerequisite initial server setup guide you will have set up a firewall on each of the servers on which you’ve installed MongoDB and enabled access for the OpenSSH UFW profile. This is an important security measure, as these firewalls currently block connections to any port on your servers, save for ssh connections that present keys which align with those in each server’s respective authorized_keys file.


However, these firewalls will also block the MongoDB instances on each server from communicating with one another, preventing you from initiating the replica set. To correct this, you’ll need to add new firewall rules to allow each server access to the port on the other two servers on which MongoDB is listening for connections.


On mongo0, run the following ufw command to allow mongo1 access to port 27017 on mongo0:


```
sudo ufw allow from mongo1_server_ip to any port 27017


```


Be sure to change mogno1_server_ip to reflect your mongo1 server’s actual IP address. Note that ufw commands will not work with hostnames configured in the hosts file, so be sure to use your servers’ actual IP addresses in this command and the following ones. Also, if you’ve updated the Mongo instance on this server to use a non-default port, be sure to change 27017 to reflect the port that your MongoDB instance is actually using.


Then add another firewall rule to give mongo2 access to the same port:


```
sudo ufw allow from mongo2_server_ip to any port 27017


```


Next, update the firewall rules for your other two servers. Run the following commands on mongo1, making sure to change the IP addresses to reflect those of mongo0 and mongo2, respectively:


```
sudo ufw allow from mongo0_server_ip to any port 27017
sudo ufw allow from mongo2_server_ip to any port 27017


```


Lastly, run these two commands on mongo2. Again, be sure that you enter the correct IP addresses for each server:


```
sudo ufw allow from mongo0_server_ip to any port 27017
sudo ufw allow from mongo1_server_ip to any port 27017


```


After adding these UFW rules, each of your three MongoDB servers will be allowed to access the port used by MongoDB on the other two servers. However, you won’t be able to test this yet, since the Mongo instance on each server is currently blocking any external connections. After you enable replication by updating each MongoDB instance’s configuration file in the next step, you will be able to perform this test.


# Step 3 — Enabling Replication in Each Server’s MongoDB Configuration File


At this point, you’ve edited your servers’ /etc/hosts files to configure hostnames which will resolve to each one’s IP address. You’ve also opened up each of your servers’ firewalls to allow the other two servers access to the default MongoDB port, 27107. Now you’re ready to begin configuring the MongoDB installation on each server to enable replication.


This step outlines how to do this by editing MongoDB’s configuration file, /etc/mongod.conf. You must complete every procedure in this step on each server, but for demonstration purposes we will use mongo0 in examples.


On mongo0, open the MongoDB configuration file in your preferred text editor:


```
sudo nano /etc/mongod.conf


```


Even though you’ve opened up each server’s firewall to allow the other servers access to port 27017, MongoDB is currently bound to 127.0.0.1, the local loopback network interface. This means that MongoDB is only able to accept connections that originate on the server where it’s installed.


To allow remote connections, you must bind MongoDB to your servers’ publicly-routable IP addresses in addition to 127.0.0.1. This way, your MongoDB installation will be able to listen to connections made to your MongoDB server from remote machines.


Find the network interfaces section. It will look like this by default:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1
. . .

```


Append a comma to the line beginning with bindIp: followed by mongo0’s hostname or public IP address. This example uses the hostname configured in Step 1:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,mongo0.replset.member
. . .

```


Next, find the line that reads #replication: towards the bottom of the file. It will look like this:


/etc/mongod.conf
```
. . .
#replication:
. . . 

```


Uncomment this line by removing the pound sign (#). Then add a replSetName directive below this line followed by a name which MongoDB will use to identify the replica set:


/etc/mongod.conf
```
. . .
replication:
  replSetName: "rs0"
. . . 

```


In this example, the replSetName directive’s value is "rs0". You can provide whatever name you’d like here, but it can be helpful to use a descriptive name. Keep in mind, though, that each server’s mongod.conf file must have the same name after the replSetName directive in order for each of their MongoDB instances to become members of the same replica set.


Note that there are two spaces before the replSetName directive and that the name is wrapped in quotation marks ("), both of which are necessary for this configuration to be read properly.


After updating these two sections of the file, net and replication, they will look like this:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,mongo0.replset.member
. . .
replication:
  replSetName: "rs0"
. . . 

```


Save and close the file. Then make these same changes to the /etc/mongod.conf files on mongo1 and mongo2. After doing so, these updated sections will look like this in mongo1’s configuration file:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,mongo1.replset.member
. . .
replication:
  replSetName: "rs0"
. . . 

```


And here’s how these sections will look in mongo2’s configuration file:


/etc/mongod.conf
```
. . .
# network interfaces
net:
  port: 27017
  bindIp: 127.0.0.1,mongo2.replset.member
. . .
replication:
  replSetName: "rs0"
. . . 

```


To reiterate, the IP address or hostname you add to each server’s bindIp directive must be that of the server whose mongod.conf file you’re editing.


After making these changes to each server’s mongod.conf file, save and close each file. Then, restart the mongod service on each server by issuing the following command:


```
sudo systemctl restart mongod


```


With that, you’ve enabled replication for each server’s MongoDB instance.



Note: At this point, you can use the nc command to test whether the firewall rules you added in Step 2 are correct. nc, short for netcat, is a utility used to establish network connections with TCP or UDP. It’s useful for testing in cases like this because it allows you to specify both an IP address and a port number when making a connection.
The following example nc command includes the -z option, which limits the utility to only scan for a listening daemon on the target server without sending it any data. Recall from the prerequisite installation tutorial that MongoDB is running as a service daemon, making this option useful for testing connectivity. It also includes the v option which increases the command’s verbosity, causing it to return more information than it would otherwise.
This example nc command shows an attempt to reach mongo1 from mongo0:
nc -zv mongo1.replset.member 27017


The following output indicates that mongo0 is able to reach mongo1 on the port used by MongoDB:
OutputConnection to mongo1.replset.member 27017 port [tcp/*] succeeded!

You can test the connection between each pair of servers by repeating this command on each server and specifying the appropriate hostnames or IP addresses.

After editing each server’s mongod.conf file to enable replication and restarting the mongod service, you’re ready to initiate the replica set and add each Mongo instance as a member.


# Step 4 — Starting the Replica Set and Adding Members


Now that you’ve configured each of your three MongoDB installations, you can open up a MongoDB shell to initiate replication and add each as a member.


For demonstration purposes, the examples in this step will use the MongoDB instance on mongo0 to initiate the replica set. However, you can initiate replication from any server whose mongod.conf file has been appropriately configured.


On mongo0, open up the MongoDB shell:


```
mongo


```


From the prompt, you can initiate a replica set from the mongo shell by running the rs.initiate() method. However, running this method by itself would only initiate replication for the machine on which you run the method, and you’d then need to add your other Mongo instances by issuing an rs.add() method for each member.


Recall that MongoDB stores its data in JSON-like structures known as documents. Because you’ve already edited the mongod.conf file on each of your servers to configure the three Mongo instances for replication, you can instead include a document that holds each member’s configuration details within the rs.initiate method. This will allow you to start the replica set and add each member at once, rather than having to run multiple separate methods.


To do this, begin an rs.initiate() method by typing the following and pressing ENTER:


```
rs.initiate(


```


Mongo won’t register the rs.initiate method as complete until you enter a closing parenthesis. Until you do, the prompt will change from a greater than sign (>) to an ellipsis (...).


As with objects in JSON, documents in MongoDB begin and end with curly braces ({ and }). To begin adding the replica set’s configuration document, enter an opening curly brace:


```
{


```


MongoDB documents are composed of any number of field-and-value pairs that take the form of field: value. The first field-and-value pair of this particular document must be an _id: field that provides a name to identify the replica set; this field’s value must be the same as the replSetName directive you set in your mongod.conf files, which is "rs0" in our examples.


Enter this field-and-value pair, following it with a comma, and then press ENTER to begin a new line:


```
_id: "rs0",


```


Next, add a members: field. Instead of a single value, though, follow this members: field with an array containing multiple documents, each of which represent a replica set member to add. In MongoDB documents, arrays are always placed within a pair of square brackets ([ and ]).


Add the members: field followed by an opening square bracket to begin the array, and then press ENTER to move to the next line:


```
members: [


```


Now add a document with two field-and-value pairs, separated by a comma, to represent the first member of the replica set. The first of this document’s fields is another _id: field which accepts an integer used to identify the member internally. The second is a host: field, which must be followed by a string containing a hostname that will resolve to an address where the member Mongo instance can be reached:


```
{ _id: 0, host: "mongo0.replset.member" },


```



Note: If any of your Mongo instances are running on a port other than MongoDB’s default — 27017 — you must follow the hostname with a colon (:) and then the port number, as in this example:
{ _id: 0, host: "mongo0.replset.member:27018" },



After entering the first one, enter additional documents for the other members of your replica set. Make sure to separate each document with a comma:


```
{ _id: 1, host: "mongo1.replset.member" },
{ _id: 2, host: "mongo2.replset.member" }


```


Next, end the array by entering a closing square bracket:


```
]


```


Lastly, end the configuration document with a closing curly brace, and then close the method with a closing parenthesis:


```
})


```


All together, the rs.initiate() method will look like this:


```
> rs.initiate(
... {
... _id: "rs0",
... members: [
... { _id: 0, host: "mongo0.replset.member" },
... { _id: 1, host: "mongo1.replset.member" },
... { _id: 2, host: "mongo2.replset.member" }
... ]
... })

```


Assuming that you entered all the details correctly, once you press ENTER after typing the closing parenthesis the method will run and initiate the replica set. If the method returns "ok" : 1 in the output, it means that the replica set was started correctly:


```
Output{
    "ok" : 1,
    "$clusterTime" : {
        "clusterTime" : Timestamp(1612389071, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    },
    "operationTime" : Timestamp(1612389071, 1)
}

```


If the replica set was initiated as expected, you’ll notice that the MongoDB client’s prompt will change from just a greater-than sign (>) to the following:


```



```


MongoDB comes installed with a few built-in methods which you can use to manage and retrieve information about your replica set. Of these, the rs.help() method can be particularly helpful as it returns a list of these replica set methods and descriptions of what they do:


```
rs.help()


```


```
Output    rs.status()                                { replSetGetStatus : 1 } checks repl set status
    rs.initiate()                              { replSetInitiate : null } initiates set with default settings
    rs.initiate(cfg)                           { replSetInitiate : cfg } initiates set with configuration cfg
    rs.conf()                                  get the current configuration object from local.system.replset
    rs.reconfig(cfg)                           updates the configuration of a running replica set with cfg (disconnects)
    rs.add(hostportstr)                        add a new member to the set with default attributes (disconnects)
    rs.add(membercfgobj)                       add a new member to the set with extra attributes (disconnects)
    rs.addArb(hostportstr)                     add a new member which is arbiterOnly:true (disconnects)
    rs.stepDown([stepdownSecs, catchUpSecs])   step down as primary (disconnects)
    rs.syncFrom(hostportstr)                   make a secondary sync from the given member
    rs.freeze(secs)                            make a node ineligible to become primary for the time specified
    rs.remove(hostportstr)                     remove a host from the replica set (disconnects)
    rs.secondaryOk()                               allow queries on secondary nodes

    rs.printReplicationInfo()                  check oplog size and time range
    rs.printSecondaryReplicationInfo()             check replica set members and replication lag
    db.isMaster()                              check who is primary
    db.hello()                              check who is primary

    reconfiguration helpers disconnect from the database so the shell will display
    an error, even if the command succeeds.

```


After running rs.help() or another one of these methods, you may see the client prompt change again to the following:


```



```


This means that the MongoDB instance that you’re connected to was elected to serve as the primary set member.


Be aware that if you have additional nodes that you’d like to add to the replica set in the future, you can do so with the rs.add() method after configuring them as you did the current replica set members in the previous steps:


```
rs.add( "mongo3.replset.member" )


```


You can now close the MongoDB client by pressing CTRL + C or by running the exit command:


```
exit


```


Your replica set is now up and running, and you can begin integrating it with your application.



Warning: When you opened up the MongoDB prompt to initiate the replica set, you may have noticed a warning message like this:
. . .
        2021-02-03T21:45:48.379+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
. . .

This message indicates that you haven’t yet enabled access control for your database. Per the MongoDB documentation:

MongoDB uses Role-Based Access Control (RBAC) to govern access to a MongoDB system. A user is granted one or more roles that determine the user’s access to database resources and operations.

Because access control hasn’t been enabled on any of your MongoDB instances, anyone with access to any of the three servers in the replica set could also gain access to the Mongo instance on that server. This poses an important security risk, since this means they could also gain access to your application data.
One way to remove this warning and add a layer of security to your replica set is by configuring keyfile authentication. As mentioned in the introduction, though, the MongoDB documentation describes keyfiles as “bare-minimum forms of security” that are “best suited for testing or development environments.”
Be aware that, for production deployments, the MongoDB documentation instead recommends using x.509 certificates for internal member authentication. The process of obtaining and configuring x.509 certificates comes with a number of caveats and decisions that must be made on a case-by-case basis, which is beyond the scope of this tutorial.
If you plan on using your replica set for testing or development, we strongly encourage you to follow our tutorial on How To Configure Keyfile Authentication for MongoDB Replica Sets on Ubuntu 20.04.

# Conclusion


Database replication has found wide use as a strategy to improve performance, availability, and data security, to the point where it’s recommended that any database used in a production environment has some form of replication enabled. Replicas are also versatile, and can take on many different roles in a data architecture, like reporting or disaster recovery. The automatic failover feature found in MongoDB’s replica sets make them particularly valuable for helping to ensure that your data remains highly available in the event of an outage.


If you’d like to learn more about MongoDB, we encourage you to check out our entire collection of MongoDB tutorials.


