# How To Configure Keyfile Authentication for MongoDB Replica Sets on Ubuntu 20 04

```Databases``` ```Security``` ```Ubuntu``` ```MongoDB``` ```NoSQL```

## Introduction


MongoDB, also known as Mongo, is an open-source document database used in many modern web applications. It is classified as a NoSQL database because it does not rely on the relational database model. Instead, it uses JSON-like documents with dynamic schemas. This means that, unlike relational databases, MongoDB does not require a predefined schema before you add data to a database.


When you’re working with multiple distributed MongoDB instances, as in the case of a replica set or a sharded database architecture, it’s important to ensure that the communications between them are secure. One way to do this is through keyfile authentication. This involves creating a special file that essentially functions as a shared password for each member in the cluster.


This tutorial outlines how to update an existing replica set to use keyfile authentication. The procedure involved in this guide will also ensure that the replica set doesn’t go through any downtime, so the data within the replica set will remain available for any clients or applications that need access to it.


# Prerequisites


To complete this tutorial, you will need:


- Three servers, each running Ubuntu 20.04. All three of these servers should have an administrative non-root user and a firewall configured with UFW. To set this up, follow our initial server setup guide for Ubuntu 20.04.
- MongoDB installed on each of your Ubuntu servers. Follow our tutorial on How To Install MongoDB on Ubuntu 20.04, making sure to complete each step on each of your servers.
- All three of your MongoDB installations configured as a replica set. Follow this tutorial on How To Configure a MongoDB Replica Set on Ubuntu 20.04 to set this up.
- SSH keys generated for each server. In addition, you should ensure that each server has the other two servers’ public keys added to its authorized_keys file. This is to ensure that each machine can communicate with one another over SSH, which will make it easier to distribute the keyfile to each of them in Step 2. To set these up, follow our guide on How To Set Up SSH Keys on Ubuntu 20.04.

Please note that, for clarity, this guide will follow the conventions established in the prerequisite replica set tutorial and refer to the three servers as mongo0, mongo1, and mongo2. It will also assume that you’ve completed Step 1 of that guide and configured each server’s hosts file so that the following hostnames will resolve to given server’s IP address:





Hostname
Resolves to




mongo0.replset.member
mongo0


mongo1.replset.member
mongo1


mongo2.replset.member
mongo2




There are a few instances in this guide in which you must run a command or update a file on only one of these servers. In such cases, this guide will default to using mongo0 in examples and will signify this by showing commands or file changes in a blue background, like this:


```



```


Any commands that must be run or file changes that must be made on multiple servers will have a standard gray background, like this:


```



```


# About Keyfile Authentication


In MongoDB, keyfile authentication relies on Salted Challenge Response Authentication Mechanism (SCRAM), the database system’s default authentication mechanism. SCRAM involves MongoDB reading and verifying credentials presented by a user against a combination of their username, password, and authentication database, all of which are known by the given MongoDB instance. This is the same mechanism used to authenticate users who supply a password when connecting to the database.


In keyfile authentication, the keyfile acts as a shared password for each member in the cluster. A keyfile must contain between 6 and 1024 characters. Keyfiles can only contain characters from the base64 set, and note that MongoDB strips whitespace characters when reading keys. Beginning in version 4.2 of Mongo, keyfiles use YAML format, allowing you to share multiple keys in a single keyfile.



Warning: The Community version of MongoDB comes with two authentication methods that can help keep your database secure, keyfile authentication and x.509 authentication. For production deployments that employ replication, the MongoDB documentation recommends using x.509 authentication, and it describes keyfiles as “bare-minimum forms of security” that are “best suited for testing or development environments.”
The process of obtaining and configuring x.509 certificates comes with a number of caveats and decisions that must be made on a case-by-case basis, meaning that this procedure is beyond the scope of a DigitalOcean tutorial. If you plan on using a replica set in a production environment, we strongly encourage you to review the official MongoDB documentation on x.509 authentication.
If you plan on using your replica set for testing or development, you can proceed with following this tutorial to add a layer of security to your cluster.

# Step 1 — Creating a User Administrator


When you enable authentication in MongoDB, it will also enable role-based access control for the replica set. Per the MongoDB documentation:



MongoDB uses Role-Based Access Control (RBAC) to govern access to a MongoDB system. A user is granted one or more roles that determine the user’s access to database resources and operations.

When access control is enabled on a MongoDB instance, it means that you won’t be able to access any of the resources on the system unless you’ve authenticated as a valid MongoDB user. Even then, you must authenticate as a user with the appropriate privileges to access a given resource.


If you don’t create a user for your MongoDB system before enabling keyfile authentication (and, consequently, access control), you will not be locked out of your replica set. You can create a MongoDB user which you can use to authenticate to the set and, if necessary, create other users through Mongo’s localhost exception. This is a special exception MongoDB makes for configurations that have enabled access control but lack users. This exception only allows you to connect to the database on the localhost and then create a user in the admin database.


However, relying on the localhost exception to create a MongoDB user after enabling authentication means that your replica set will go through a period of downtime, since the replicas will not be able to authenticate their connection until after you create a user. This step outlines how to create a user before enabling authentication to ensure that your replica set remains available. This user will have permissions to create other users on the database, giving you the freedom to create other users with whatever permissions they need in the future. In MongoDB, a user with such permissions is known as a user administrator.


To begin, connect to the primary member of your replica set. If you aren’t sure which of your members is the primary, you can run the rs.status() method to identify it.


Run the following mongo command from the bash prompt of any of the Ubuntu servers hosting a MongoDB instance in your replica set. This command’s --eval option instructs the mongo operation to not open up the shell interface environment that appears when you run mongo by itself and instead run the command or method, wrapped in single quotes, that follows the --eval argument:


```
mongo --eval 'rs.status()'


```


rs.status()  returns a lot of information, but the relevant portion of the output is the "members" : array. In the context of MongoDB, an array is a collection of documents held between a pair of square brackets ([ and ]).


In the "members": array you’ll find a number of documents, each of which contains information about one of the members in your replica set. Within each of these member documents, find the "stateStr" field. The member whose "stateStr" value is "PRIMARY" is the primary member of your replica set. The following example shows a situation where mongo0 is the primary:


```
Output. . .
    "members" : [
        {
            "_id" : 0,
            "name" : "mongo0.replset.member:27017",
            "health" : 1,
            "state" : 1,
            "stateStr" : "PRIMARY",
. . .
        },
. . .

```


Once you know which of your replica set members is the primary, SSH into the server hosting that instance. For demonstration purposes, this guide will continue to use examples in which mongo0 is the primary:


```
ssh sammy@mongo0_ip_address


```


After logging into the server, connect to MongoDB by opening up the mongo shell environment:


```
mongo


```


When creating a user in MongoDB, you must create them within a specific database which will be used as their authentication database. The combination of the user’s name and their authentication database serve as a unique identifier for that user.


Certain administrative actions are only available to users whose authentication database is the admin database — a special privileged database included in every MongoDB installation — including the ability to create new users. Because the goal of this step is to create an user administrator that can create other users in the replica set, connect to the admin database so you can grant this user the appropriate privileges:


```
use admin


```


```
Outputswitched to db admin

```


MongoDB comes installed with a number of JavaScript-based shell methods you can use to manage your database. One of these, the db.createUser method, is used to create new users in the database in which the method is run.


Initiate the db.createUser method:


```
db.createUser(


```



Note: Mongo won’t register the db.createUser method as complete until you enter a closing parenthesis. Until you do, the prompt will change from a greater than sign (>) to an ellipsis (...).

This method requires you to specify a username and password for the user, as well as any roles you want the user to have. Recall that MongoDB stores its data in JSON-like documents; when you create a new user, all you’re doing is creating a document to hold the appropriate user data as individual fields.


As with objects in JSON, documents in MongoDB begin and end with curly braces ({ and }). Enter an opening curly brace to begin the user document:


```
{


```


Next, enter a user: field, with your desired username as the value in double quotes followed by a comma. The following example specifies the username UserAdminSammy, but you can enter whatever username you like:


```
user: "UserAdminSammy",


```


Next, enter a pwd field with the passwordPrompt() method as its value. When you execute the db.createUser method, the passwordPrompt() method will provide a prompt for you to enter your password. This is more secure than the alternative, which is to type out your password in cleartext as you did for your username.



Note: The passwordPrompt() method is only compatible with MongoDB versions 4.2 and newer. If you’re using an older version of Mongo, then you will have to write out your password in cleartext, similarly to how you wrote out your username:
pwd: "password",



Be sure to follow this field with a comma as well:


```
pwd: passwordPrompt(),


```


Then enter a roles field followed by an array detailing the roles you want your administrative user to have. In MongoDB, roles define what actions the user can perform on the resources that they have access to. You can define custom roles yourself, but Mongo also comes with a number of built-in roles that grant commonly-needed permissions.


Because you’re creating a user administrator, at a minimum you should grant them the built-in userAdminAnyDatabase role over the admin database. This will allow the user administrator to create and modify new users and roles. Because the administrative user has this role in the admin database, this will also grant it superuser access to the entire cluster:


```
roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]


```


Following that, enter a closing brace to signify the end of the document:


```
}


```


Then enter a closing parenthesis to close and execute the db.createUser method:


```
)


```


All together, here’s what your db.createUser method should look like:


```
> db.createUser(
... {
... user: "UserAdminSammy",
... pwd: passwordPrompt(),
... roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
... }
... )

```


If each line’s syntax is correct, the method will execute properly and you’ll be prompted to enter a password:


```
OutputEnter password:

```


Enter a strong password of your choosing. Then, you’ll receive a confirmation that the user was added:


```
OutputSuccessfully added user: {
    "user" : "UserAdminSammy",
    "roles" : [
        {
            "role" : "userAdminAnyDatabase",
            "db" : "admin"
        },
        "readWriteAnyDatabase"
    ]
}

```


With that, you’ve added a MongoDB user profile which you can use to manage other users and roles on your system. You can test this out by creating another user, as outlined in the remainder of this step.


Begin by authenticating as the user administrator you just created:


```
db.auth( "UserAdminSammy", passwordPrompt() )


```


db.auth() will return 1 if authentication was successful:


```
Output1

```



Note: In the future, if you want to authenticate as the user administrator when connecting to the cluster, you can do so directly from your server prompt with a command like the following:
mongo -u "UserAdminSammy" -p --authenticationDatabase "admin"


In this command, the -u option tells the shell that the following argument is the username which you want to authenticate as. The -p flag tells it to prompt you to enter a password, and the --authenticationDatabase option precedes the name of the user’s authentication database. If you enter an incorrect password or the username and authentication database do not match, you won’t be able to authenticate and you’ll have to try connecting again.
Also, be aware that in order for you to create new users in the replica set as the user administrator, you must be connected to the set’s primary member.

The procedure for adding another user is the same as it was for the user administrator. The following example creates a new user with the clusterAdmin role, which means they will be able to perform a number of operations related to replication and sharding. Within the context of MongoDB, a user with these privileges is known as a cluster administrator.


Having a dedicated user to perform specific functions like this is a good security practice, as it limits the number of privileged users you have on your system. After you enable keyfile authentication later in this tutorial, any client that wants to perform any of the operations allowed by the clusterAdmin role — such as any of the rs. methods, like rs.status() or rs.conf() — must first authenticate as the cluster administrator.


That said, you can provide whatever role you’d like to this user, and likewise provide them with a different name and authentication database. However, if you want the new user to function as a cluster administrator, then you must grant them the clusterAdmin role within the admin database.


In addition to creating a user to serve as the cluster administrator, the following method names the user ClusterAdminSammy and uses the passwordPrompt() method to prompt you to enter a password:


```
db.createUser(
{
user: "ClusterAdminSammy",
pwd: passwordPrompt(),
roles: [ { role: "clusterAdmin", db: "admin" } ]
}
)


```


Again, if you’re using a version of MongoDB that precedes version 4.2, then you will have to write out your password in cleartext instead of using the passwordPrompt() method.


If each line’s syntax is correct, the method will execute properly and you’ll be prompted to enter a password:


```
OutputEnter password:

```


Enter a strong password of your choosing. Then, you’ll receive a confirmation that the user was added:


```
OutputSuccessfully added user: {
    "user" : "ClusterAdminSammy",
    "roles" : [
        {
            "role" : "clusterAdmin",
            "db" : "admin"
        }
    ]
}

```


This output confirms that your user administrator is able to create new users and grant them roles. You can now close the MongoDB shell:


```
exit


```


Alternatively, you can close the shell by pressing CTRL + C.


At this point, if you have any clients or applications connected to your MongoDB cluster, it would be a good time to create one or more dedicated users with the appropriate roles which they can use to authenticate to the database. Otherwise, read on to learn how to generate a keyfile, distribute it among the members of your replica set, and then configure each one to require the replica set members to authenticate with the keyfile.


# Step 2 — Creating and Distributing an Authentication Keyfile


Before creating a keyfile, it can be helpful to create a directory on each server where you will store the keyfile in order to keep things organized. Run the following command, which creates a directory named mongo-security in the administrative Ubuntu user’s home directory, on each of your three servers:


```
mkdir ~/mongo-security


```


Then generate a keyfile on one of your servers. You can do this on any one of your servers but, for illustration purposes, this guide will generate the keyfile on mongo0.


Navigate to the mongo-security directory you just created:


```
cd ~/mongo-security/


```


Within that directory, create a keyfile with the following openssl command:


```
openssl rand -base64 768 > keyfile.txt


```


Take note of this command’s arguments:


- rand: instructs OpenSSL to generate pseudo-random bytes of data
- -base64: specifies that the command should use base64 encoding to represent the pseudo-random data as printable text. This is important because, as mentioned previously, MongoDB keyfiles can only contain characters in the base64 set
- 768: the number of bytes the command should generate. In base64 encoding, three binary bytes of data are represented as four characters. Because MongoDB keyfiles can have a maximum of 1024 characters, 768 is the maximum number of bytes you can generate for a valid keyfile

Following this command’s 768 argument is a greater-than sign (>). This redirects the command’s output into a new file named keyfile.txt which will serve as your keyfile. Feel free to name the keyfile something other than keyfile.txt if you’d like, but be sure to change the filename whenever it appears in later commands.


Next, modify the keyfile’s permissions so that only the owner has read access:


```
chmod 400 keyfile.txt


```


Following this, distribute the keyfile to the other two servers hosting the MongoDB instances in your replica set. Assuming you followed the prerequisite guide on How To Set Up SSH Keys, you can do so with the scp command:


```
scp keyfile.txt sammy@mongo1.replset.member:/home/sammy/mongo-security
scp keyfile.txt sammy@mongo2.replset.member:/home/sammy/mongo-security


```


Notice that each of these commands copies the keyfile directly to the ~/mongo-security/ directories you created previously on mongo1 and mongo2. Be sure to change sammy to the name of the administrative Ubuntu user profile you created on each server.


Next, change the file’s owner to the mongodb user profile. This is a special user that was created when you installed MongoDB, and it’s used to run the mongod service. This user must have access to the keyfile in order for MongoDB to use it for authentication.


Run the following command on each of your servers to change the keyfile’s owner to the mongodb user account:


```
sudo chown mongodb:mongodb ~/mongo-security/keyfile.txt


```


After changing the keyfiles’ owner on each server, you’re ready to reconfigure each of your MongoDB instances to enforce keyfile authentication.


# Step 3 — Enabling Keyfile Authentication


Now that you’ve generated a keyfile and distributed it to each of the servers in your replica set, you can update the MongoDB configuration file on each server to enforce keyfile authentication.


In order to avoid any downtime while configuring the members of your replica set to require authentication, this step involves reconfiguring the secondary members of the set first. Then, you’ll direct your primary member to step down and become a secondary member. This will cause the secondary members to hold an election to select a new primary, keeping your cluster available to whatever clients or applications need access to it. You’ll then reconfigure the former primary node to enable authentication.


On each of your servers hosting a secondary member of your replica set, open up MongoDB’s configuration file with your preferred text editor:


```
sudo nano /etc/mongod.conf


```


Within the file, find the security section. It will look like this by default:


/etc/mongod.conf
```
. . .
#security:
. . .

```


Uncomment this line by removing the pound sign (#). Then, on the next line, add a keyFile: directive followed by the full path to the keyfile you created in the previous step:


/etc/mongod.conf
```
. . .
security:
  keyFile: /home/sammy/mongo-security/keyfile.txt
. . .

```


Note that there are two spaces at the beginning of this new line. These are necessary for the configuration file to be read correctly. When you enter this line in your own configuration files, make sure that the path you provide reflects the actual path of the keyfile on each server.


Below the keyFile directive, add a transitionToAuth directive with a value of true. When set to true, this configuration option allows the MongoDB instance to accept both authenticated and non-authenticated connections. This is useful when reconfiguring a replica set to enforce authentication, as it will ensure that your data remains available as you restart each member of the set:


/etc/mongod.conf
```
. . .
security:
  keyFile: /home/sammy/mongo-security/keyfile.txt
  transitionToAuth: true
. . .

```


Again, make sure that you include two blank spaces before the transitionToAuth directive.


After making those changes, save and close the file. If you used nano to edit it, you can do so by pressing CTRL + X, Y, and then ENTER.


Then restart the mongod service on both of the secondary instances’ servers to immediately put these changes into effect:


```
sudo systemctl restart mongod


```


With that, you’ve configured keyfile authentication for the secondary members of your replica set. At this point, both authenticated and non-authenticated users can access these members without restriction.


Next, you’ll repeat this procedure on the primary member. Before doing so, though, you must step down the member so it’s no longer the primary. To do this, open up the MongoDB shell on the server hosting the primary member. For illustration purposes, this guide will again assume this is mongo0:


```
mongo


```


From the prompt, run the rs.stepDown() method. This will instruct the primary to become a secondary member, and will cause the current secondary members to hold an election to determine which will serve as the new primary:


```
rs.stepDown()


```


If the method returns "ok" : 1 in the output, it means the primary member successfully stepped down to become a secondary:


```
Output{
    "ok" : 1,
    "$clusterTime" : {
        "clusterTime" : Timestamp(1614795467, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    },
    "operationTime" : Timestamp(1614795467, 1)
}

```


After stepping down the primary, you can close the Mongo shell:


```
exit


```


Next, open up the MongoDB configuration file on this server:


```
sudo nano /etc/mongod.conf


```


Find the security section and uncomment the security header by removing the pound sign. Then add the same keyFile and transitionToAuth directives you added to the other MongoDB instances. After making these changes, the security section will look like this:


/etc/mongod.conf
```
. . .
security:
  keyFile: /home/sammy/mongo-security/keyfile.txt
  transitionToAuth: true
. . .

```


Again, make sure that the file path after the keyFile directive reflects the keyfile’s actual location on this server.


When finished, save and close the file. Then restart the mongod process:


```
sudo systemctl restart mongod


```


Following that, all of your MongoDB instances are able to accept both authenticated and non-authenticated connections. In the final step of this guide, you’ll configure your instances to require users to authenticate before performing privileged actions.


# Step 4 — Restarting Each Member Without transitionToAuth to Enforce Authentication


At this point, each of your MongoDB instances are configured with the transitionToAuth set to true. This means that even though you’ve enabled each server to use the keyfile you created to authenticate connections internally, they’re still able to accept non-authenticated connections.


To change this and require each member to enforce authentication, reopen the mongod.conf file on each server:


```
sudo nano /etc/mongod.conf


```


Find the security section and disable the transitionToAuth directive. You can do this by commenting the line out by prepending it with a pound sign:


/etc/mongod.conf
```
. . .
security:
  keyFile: /home/sammy/mongo-security/keyfile.txt
  #transitionToAuth: true
. . .

```


After disabling the transitionToAuth directive in each instance’s configuration file, save and close each file.


Then, restart the mongod service on each server:


```
sudo systemctl restart mongod


```


Following that, each of the MongoDB instances in your replica set will require you to authenticate to perform privileged actions.


To test this, try running a MongoDB method that works when invoked by an authenticated user that has the appropriate privileges. Try running the following command from any of your Ubuntu servers’ prompts:


```
mongo --eval 'rs.status()'


```


Even though you ran this method successfully in Step 1, the rs.status() method can now only be run by a user that has been granted the clusterAdmin or clusterManager roles since you’ve enabled keyfile authentication. Regardless of whether you run this command on a server hosting the primary member or one of the secondary members, it will not work because you have not authenticated:


```
Output. . .
MongoDB server version: 4.4.4
{
    "operationTime" : Timestamp(1616184183, 1),
    "ok" : 0,
    "errmsg" : "command replSetGetStatus requires authentication",
    "code" : 13,
    "codeName" : "Unauthorized",
    "$clusterTime" : {
        "clusterTime" : Timestamp(1616184183, 1),
        "signature" : {
            "hash" : BinData(0,"huJUmB/lrrxpx9YfnONM4mayJwo="),
            "keyId" : NumberLong("6941116945081040899")
        }
    }
}

```


Recall that, after enabling access control, all of the cluster administration methods (including rs. methods like rs.status()) will only work when invoked by an authenticated user that has been granted the appropriate cluster management roles. If you’ve created a cluster administrator — as outlined in Step 1 — and authenticate as that user, then this method will work as expected:


```
mongo -u "ClusterAdminSammy" -p --authenticationDatabase "admin" --eval 'rs.status()'


```


After entering the user’s password when prompted, you will see the rs.status() method’s output:


```
Output. . .
MongoDB server version: 4.4.4
{
    "set" : "shard2",
    "date" : ISODate("2021-03-19T20:21:45.528Z"),
    "myState" : 2,
    "term" : NumberLong(4),
    "syncSourceHost" : "mongo1.replset.member:27017",
    "syncSourceId" : 2,
    "heartbeatIntervalMillis" : NumberLong(2000),
    "majorityVoteCount" : 2,
. . .

```


This confirms that the replica set is enforcing authentication, and that you’re able to authenticate successfully.


# Conclusion


By completing this tutorial, you created a keyfile with OpenSSL and then configured a MongoDB replica set to require its members to use it for internal authentication. You also created a user administrator which will allow you to manage users and roles in the future. Throughout all of this, your replica set will not have gone through any downtime and your data will have remained available to your clients and applications.


If you’d like to learn more about MongoDB, we encourage you to check out our entire library of MongoDB content.


