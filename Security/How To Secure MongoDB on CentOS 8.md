# How To Secure MongoDB on CentOS 8

```Databases``` ```Security``` ```MongoDB``` ```NoSQL``` ```CentOS``` ```CentOS 8```

An earlier version of this tutorial was written by Melissa Anderson.


## Introduction


MongoDB, also known as Mongo, is an open-source document database used in many modern web applications.  It is classified as a NoSQL database because it does not rely on a traditional table-based relational database structure.  Instead, it uses JSON-like documents with dynamic schemas.


MongoDB doesn’t have authentication enabled by default, meaning that any user with access to the server where the database is installed can add and delete data without restriction.  In order to secure this vulnerability, this tutorial will walk you through creating an administrative user and enabling authentication.  You’ll then test to confirm that only this administrative user has access to the database.


# Prerequisites


To complete this tutorial, you will need the following:


- To complete this tutorial, you will need a server running CentOS 8.  This server should have a non-root user with administrative privileges and a firewall configured with firewalld.  To set this up, follow our Initial Server Setup guide for CentOS 8.
- MongoDB installed on your server.  This tutorial was validated using MongoDB version 4.4, though it should generally work for older versions of MongoDB as well.  To install Mongo on your server, follow our tutorial on How To Install MongoDB on CentOS 8.

# Step 1 — Adding an Administrative User


Since the release of version 3.0, the MongoDB daemon is configured to only accept connections from the local Unix socket, and it is not automatically open to the wider Internet. However, authentication is still disabled by default. This means that any users that have access to the server where MongoDB is installed also have complete access to the databases.


As a first step to securing this vulnerability, you will create an administrative user.  Later, you’ll enable authentication and connect as this administrative user to access the database.


To add an administrative user, you must first connect to the Mongo shell. Because authentication is disabled you can do so with the mongo command, without any other options:


```
mongo


```


There will be some output above the Mongo shell prompt.  Because you haven’t yet enabled authentication, this will include a warning that access control isn’t enabled for the database and that read and write access to data and and the database’s configuration are unrestricted:


```
OutputMongoDB shell version v4.4.1
 . . . 
---
The server generated these startup warnings when booting: 
        2020-10-02T14:38:03.448+00:00: ***** SERVER RESTARTED *****
        2020-10-02T14:38:04.209+00:00: Access control is not enabled for the database. Read and write access to data and configuration is unrestricted
---
 . . .
> 

```


The second warning will disappear after you enable authentication, but for now it means anyone who can access your CentOS server could also take control over your database.



Note: If you installed MongoDB by following a tutorial other than the one linked in the prerequisites, you may have another warning in the prompt that reads:
Output. . .
        2020-10-02T14:38:04.633+00:00: /sys/kernel/mm/transparent_hugepage/enabled is 'always'. We suggest setting it to 'never'
. . .

This warning refers to Transparent Huge Pages (THP), a Linux memory management system enabled on CentOS 8 by default. THP uses extra large memory pages to reduce the impact of Translation Lookaside Buffer lookups on machines with large amounts of memory. However, as this warning indicates, this system can have a negative impact on database performance and the MongoDB documentation recommends that you disable it.
If you’d like details on how to disable Transparent Huge Pages, please see Step 2 of our guide on How To Install MongoDB on CentOS 8.

To illustrate, run Mongo’s show dbs command:


```
show dbs


```


This command returns a list of every database on the server. However, when authentication is enabled, the list changes based on the Mongo user’s role, or what level of access it has to certain databases. Because authentication is disabled, though, it will return every database currently on the system without restrictions:


```
Outputadmin   0.000GB
config  0.000GB
local   0.000GB

```


In this example output, only the default databases appear. However, if you have any databases holding sensitive data on your system, any user could find them with this command.


As part of mitigating this vulnerability, this step is focused on adding an administrative user. To do this, you must first connect to the admin database. This is where information about users, like their usernames, passwords, and roles, are stored:


```
use admin


```


```
Outputswitched to db admin

```


MongoDB comes installed with a number of JavaScript-based shell methods you can use to manage your database.  One of these, the db.createUser method, is used to create new users on the database on which the method is run.


Initiate the db.createUser method:


```
db.createUser(


```


This method requires you to specify a username and password for the user, as well as any roles you want the user to have.  Recall that MongoDB stores its data in JSON-like documents.  When you create a new user, all you’re doing is creating a document to hold the appropriate user data as individual fields.


As with objects in JSON, documents in MongoDB begin and end with curly braces ({ and }).  To begin adding a user, enter an opening curly brace:



Note: Mongo won’t register the db.createUser method as complete until you enter a closing parenthesis.  Until you do, the prompt will change from a greater than sign (>) to an ellipsis (...).

```
{


```


Next, enter a user: field, with your desired username as the value in double quotes followed by a comma.  The following example specifies the username AdminSammy, but you can enter whatever username you like:


```
user: "AdminSammy",


```


Next, enter a pwd field with the passwordPrompt() method as its value.  When you execute the db.createUser method, the passwordPrompt() method will provide a prompt for you to enter your password.  This is more secure than the alternative, which is to type out your password in cleartext as you did for your username.



Note: The passwordPrompt() method is only compatible with MongoDB versions 4.2 and newer.  If you’re using an older version of Mongo, then you will have to write out your password in cleartext, similarly to how you wrote out your username:
pwd: "password",



Be sure to follow this field with a comma as well:


```
pwd: passwordPrompt(),


```


Then enter the roles you want your administrative user to have.  Because you’re creating an administrative user, at a minimum you should grant them the userAdminAnyDatabase role over the admin database.  This will allow the administrative user to create and modify new users and roles.  Because the administrative user has this role in the admin database, this will also grant it superuser access to the entire cluster.


In addition, the following example also grants the administrative user the readWriteAnyDatabase role.  This grants the administrative user the ability to read and modify data on any database in the cluster except for the config and local databases, which are mostly for internal use:


```
roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]


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
... user: "AdminSammy",
... pwd: passwordPrompt(),
... roles: [ { role: "userAdminAnyDatabase", db: "admin" }, "readWriteAnyDatabase" ]
... }
... )

```


If each line’s syntax is correct, the method will execute properly and you’ll be prompted to enter a password:


```
OutputEnter password: 

```


Enter a strong password of your choosing.  Then, you’ll receive a confirmation that the user was added:


```
OutputSuccessfully added user: {
	"user" : "AdminSammy",
	"roles" : [
		{
			"role" : "userAdminAnyDatabase",
			"db" : "admin"
		},
		"readWriteAnyDatabase"
	]
}

```


Following that, you can exit the MongoDB client:


```
exit


```


At this point, your user will be allowed to enter credentials.  However, they will not be required to do so until you enable authentication and restart the MongoDB daemon.


# Step 2 —  Enabling Authentication


To enable authentication, you must edit mongod.conf, MongoDB’s configuration file.  Once you enable it and restart the Mongo service, users will still be able to connect to the database without authenticating.  However, they won’t be able to read or modify any data until they provide a correct username and password.


Open the configuration file with your preferred text editor.  Here, we’ll use nano:


```
sudo nano /etc/mongod.conf


```


Scroll down to find the commented-out security section:


/etc/mongod.conf
```
. . .
#security:

#operationProfiling:

. . .

```


Uncomment this line by removing the pound sign (#):


/etc/mongod.conf
```
. . .
security:

#operationProfiling:

. . .

```


Then add the authorization parameter and set it to "enabled".  When you’re done, the lines should look like this:


/etc/mongod.conf
```
. . .
security:
  authorization: "enabled"
. . . 

```


Note that the security: line has no spaces at the beginning, while the authorization: line is indented with two spaces.


After adding these lines, save and close the file.  If you used nano to open the file, do so by pressing CTRL + X, Y, then ENTER.


Then restart the daemon to put these new changes into effect:


```
sudo systemctl restart mongod


```


Next, check the service’s status to make sure that it restarted correctly:


```
sudo systemctl status mongod


```


If the restart command was successful, you’ll receive output that indicates that the mongod service is active and was recently started:


```
Output● mongod.service - MongoDB Database Server
   Loaded: loaded (/usr/lib/systemd/system/mongod.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-10-02 14:57:31 UTC; 7s ago
     Docs: https://docs.mongodb.org/manual
  Process: 15667 ExecStart=/usr/bin/mongod $OPTIONS (code=exited, status=0/SUCCESS)
  Process: 15665 ExecStartPre=/usr/bin/chmod 0755 /var/run/mongodb (code=exited, status=0/SUCCESS)
  Process: 15662 ExecStartPre=/usr/bin/chown mongod:mongod /var/run/mongodb (code=exited, status=0/SUCCESS)
  Process: 15659 ExecStartPre=/usr/bin/mkdir -p /var/run/mongodb (code=exited, status=0/SUCCESS)
 Main PID: 15669 (mongod)
   Memory: 159.2M
   CGroup: /system.slice/mongod.service
           └─15669 /usr/bin/mongod -f /etc/mongod.conf

```


Having verified the daemon is back up and running, you can test that the authentication setting you added works as expected.


# Step 3 —  Testing Authentication Settings


To begin testing that the authentication requirements you added in the previous step are working correctly, start by connecting without specifying any credentials to verify that your actions are indeed restricted:


```
mongo 


```


Now that you’ve enabled authentication, none of the warnings you encountered previously will appear:


```
OutputMongoDB shell version v4.4.1
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("d57987e5-24dc-427e-89b7-9ab362ebac6b") }
MongoDB server version: 4.4.1
> 

```


Confirm whether your access is restricted by running the show dbs command again:


```
show dbs


```


Recall from Step 1 that there are at least a few default databases on your server. However, in this case the command won’t have any output because you haven’t authenticated as a privileged user.


Because this command doesn’t return any information, it’s safe to say the authentication setting is working as expected.  You also won’t be able to create users or perform other privileged tasks without first authenticating.


Go ahead and exit the MongoDB shell:



Note: Instead of running the following exit command as you did previously in Step 1, an alternative way to close the shell is to just press CTRL + C.

```
exit


```


Next, make sure that your administrative user is able to authenticate properly by running the following mongo command to connect as this user.  This command includes the -u flag, which precedes the name of the user you want to connect as.  Be sure to replace AdminSammy with your own administrative user’s username.  It also includes the -p flag, which will prompt you for the user’s password, and specifies admin as the authentication database where the specified username was created:


```
mongo -u AdminSammy -p --authenticationDatabase admin


```


Enter the user’s password when prompted, and then you’ll be dropped into the shell.  Once there, try issuing the show dbs command again:


```
show dbs


```


This time, because you’ve authenticated properly, the command will successfully return a list of all the databases currently on the server:


```
Outputadmin   0.000GB
config  0.000GB
local   0.000GB

```


This confirms that authentication was enabled successfully.


# Conclusion


By completing this guide, you’ve set up an administrative MongoDB user which you can employ to create and modify new users and roles, and otherwise manage your MongoDB instance.  You also configured your MongoDB instance to require that users authenticate with a valid username and password before they can interact with any data.


For more information on how to manage MongoDB users, check out the official documentation on the subject.  You may also be interested in learning more about how authentication works on MongoDB.


Also, if you plan to interact with your MongoDB instance remotely, you can follow our guide on How To Configure Remote Access for MongoDB on CentOS 8.


