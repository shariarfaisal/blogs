# How To Configure a Galera Cluster with MySQL on Ubuntu 18 04 Servers

```Databases``` ```MySQL``` ```Ubuntu 18.04``` ```Clustering``` ```High Availability``` ```Open Source```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Clustering adds high availability to your database by distributing changes to different servers. In the event that one of the instances fails, others are quickly available to continue serving.


Clusters come in two general configurations, active-passive and active-active. In active-passive clusters, all writes are done on a single active server and then copied to one or more passive servers that are poised to take over only in the event of an active server failure. Some active-passive clusters also allow SELECT operations on passive nodes. In an active-active cluster, every node is read-write and a change made to one is replicated to all.


MySQL is an open source relational database management system that is a popular choice for SQL databases. Galera is a database clustering solution that enables you to set up multi-master clusters using synchronous replication. Galera automatically handles keeping the data on different nodes in sync while allowing you to send read and write queries to any of the nodes in the cluster. You can learn more about Galera at the official documentation page.


In this guide, you will configure an active-active MySQL Galera cluster. For demonstration purposes, you will configure and test three Ubuntu 18.04 Droplets that will act as nodes in the cluster. This amount of nodes is the smallest configurable cluster.


# Prerequisites


To follow along, you will need a DigitalOcean account, in addition to the following:


- Three Ubuntu 18.04 Droplets with private networking enabled, each with a non-root user with sudo privileges.

For setting up private networking on the three Droplets, follow our Private Networking Quickstart guide.
For assistance setting up a non-root user with sudo privileges, follow our Initial Server Setup with Ubuntu 18.04 tutorial.


- For setting up private networking on the three Droplets, follow our Private Networking Quickstart guide.
- For assistance setting up a non-root user with sudo privileges, follow our Initial Server Setup with Ubuntu 18.04 tutorial.

While the steps in this tutorial have been written for and tested against DigitalOcean Droplets, much of them will also be applicable to non-DigitalOcean servers with private networking enabled.


# Step 1 — Adding the MySQL Repositories to All Servers


In this step, you will add the relevant MySQL and Galera package repositories to each of your three servers so that you will be able to install the right version of MySQL and Galera used in this tutorial.



Note: Codership, the company behind Galera Cluster, maintains the Galera repository, but be aware that not all external repositories are reliable. Be sure to install only from trusted sources.

In this tutorial, you will use MySQL version 5.7. You’ll start by adding the external Ubuntu repository maintained by the Galera project to all three of your servers.


Once the repositories are updated on all three servers, you will be ready to install MySQL along with Galera.


First, on all three of your servers, add the Galera repository key with the apt-key command, which the APT package manager will use to verify that the package is authentic:


```
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv BC19DDBA


```


After a few seconds, you will receive the following output:


```
OutputExecuting: /tmp/apt-key-gpghome.RG5cTZjQo0/gpg.1.sh --keyserver keyserver.ubuntu.com --recv BC19DDBA
gpg: key D669017EBC19DDBA: public key "Codership Oy <info@galeracluster.com>" imported
gpg: Total number processed: 1
gpg:               imported: 1

```


Once you have the trusted key in each server’s database, you can add the repositories. To do so, create a new file called galera.list within the /etc/apt/sources.list.d/ directory on each server:


```
sudo nano /etc/apt/sources.list.d/galera.list


```


In the text editor, add the following lines, which will make the appropriate repositories available to the APT package manager:


/etc/apt/sources.list.d/galera.list
```
deb http://releases.galeracluster.com/mysql-wsrep-5.7/ubuntu bionic main
deb http://releases.galeracluster.com/galera-3/ubuntu bionic main

```


Save and close the files on each server (press CTRL + X, Y, then ENTER).


The Codership repositories are now available to all three of your servers. However, it’s important that you instruct apt to prefer Codership’s repositories over others to ensure that it installs the patched versions of the software needed to create a Galera cluster. To do this, create another new file called galera.pref within the /etc/apt/preferences.d/ directory of each server:


```
sudo nano /etc/apt/preferences.d/galera.pref


```


Add the following lines to the text editor:


/etc/apt/preferences.d/galera.pref
```
# Prefer Codership repository
Package: *
Pin: origin releases.galeracluster.com
Pin-Priority: 1001

```


Save and close that file, then run the following command on each server in order to include package manifests from the new repositories:


```
sudo apt update


```


Now that you have successfully added the package repository on all three of your servers, you’re ready to install MySQL in the next section.


# Step 2 — Installing MySQL on All Servers


In this step, you will install the MySQL package on your three servers.


Run the following command on all three servers to install a version of MySQL patched to work with Galera, as well as the Galera package.


```
sudo apt install galera-3 mysql-wsrep-5.7


```


You will be asked to confirm whether you would like to proceed with the installation. Enter Y to continue with the installation. During the installation, you will also be asked to set a password for the MySQL administrative user. Set a strong password and press ENTER to continue.


Once MySQL is installed, you will disable the default AppArmor profile to ensure that Galera functions properly, as per the official Galera documentation. AppArmor is a kernel module for Linux that provides access control functionality for services through security profiles.


Disable AppArmor by executing the following on each server:


```
sudo ln -s /etc/apparmor.d/usr.sbin.mysqld /etc/apparmor.d/disable/


```


This command adds a symbolic link of the MySQL profile to the disable directory, which disables the profile on boot.


Then, run the following command to remove the MySQL definition that has already been loaded in the kernel.


```
sudo apparmor_parser -R /etc/apparmor.d/usr.sbin.mysqld


```


Once you have installed MySQL and disabled the AppArmor profile on your first server, repeat these steps for your other two servers.


Now that you have installed MySQL successfully on each of the three servers, you can proceed to the configuration step in the next section.


# Step 3 — Configuring the First Node


In this step you will configure your first node. Each node in the cluster needs to have a nearly identical configuration. Because of this, you will do all of the configuration on your first machine, and then copy it to the other nodes.


By default, MySQL is configured to check the /etc/mysql/conf.d directory to get additional configuration settings from files ending in .cnf. On your first server, create a file in this directory with all of your cluster-specific directives:


```
sudo nano /etc/mysql/conf.d/galera.cnf


```


Add the following configuration into the file. The configuration specifies different cluster options, details about the current server and the other servers in the cluster, and replication-related settings. Note that the IP addresses in the configuration are the private addresses of your respective servers; replace the highlighted lines with the appropriate IP addresses.


/etc/mysql/conf.d/galera.cnf
```
[mysqld]
binlog_format=ROW
default-storage-engine=innodb
innodb_autoinc_lock_mode=2
bind-address=0.0.0.0

# Galera Provider Configuration
wsrep_on=ON
wsrep_provider=/usr/lib/galera/libgalera_smm.so

# Galera Cluster Configuration
wsrep_cluster_name="test_cluster"
wsrep_cluster_address="gcomm://First_Node_IP,Second_Node_IP,Third_Node_IP"

# Galera Synchronization Configuration
wsrep_sst_method=rsync

# Galera Node Configuration
wsrep_node_address="This_Node_IP"
wsrep_node_name="This_Node_Name"

```


- The first section modifies or re-asserts MySQL settings that will allow the cluster to function correctly. For example, Galera won’t work with MyISAM or similar non-transactional storage engines, and mysqld must not be bound to the IP address for localhost. You can learn about the settings in more detail on the Galera Cluster system configuration page.
- The “Galera Provider Configuration” section  configures the MySQL components that provide a WriteSet replication API. This means Galera in your case, since Galera is a wsrep (WriteSet Replication) provider. You specify the general parameters to configure the initial replication environment. This doesn’t require any customization, but you can learn more about Galera configuration options in the documentation.
- The “Galera Cluster Configuration” section defines the cluster, identifying the cluster members by IP address or resolvable domain name and creating a name for the cluster to ensure that members join the correct group. You can change the wsrep_cluster_name to something more meaningful than test_cluster or leave it as-is, but you must update wsrep_cluster_address with the private IP addresses of your three servers.
- The Galera Synchronization Configuration section defines how the cluster will communicate and synchronize data between members. This is used only for the state transfer that happens when a node comes online. For your initial setup, you are using rsync, because it’s commonly available and does what you’ll need for now.
- The Galera Node Configuration section clarifies the IP address and the name of the current server. This is helpful when trying to diagnose problems in logs and for referencing each server in multiple ways. The wsrep_node_address must match the address of the machine you’re on, but you can choose any name you want in order to help you identify the node in log files.

When you are satisfied with your cluster configuration file, copy the contents into your clipboard, then save and close the file.


Now that you have configured your first node successfully, you can move on to configuring the remaining nodes in the next section.


# Step 4 — Configuring the Remaining Nodes


In this step, you will configure the remaining two nodes. On your second node, open the configuration file:


```
sudo nano /etc/mysql/conf.d/galera.cnf


```


Paste in the configuration you copied from the first node, then update the Galera Node Configuration to use the IP address or resolvable domain name for the specific node you’re setting up. Finally, update its name, which you can set to whatever helps you identify the node in your log files:


/etc/mysql/conf.d/galera.cnf
```
. . .
# Galera Node Configuration
wsrep_node_address="This_Node_IP"
wsrep_node_name="This_Node_Name"
. . .


```


Save and exit the file.


Once you have completed these steps, repeat them on the third node.


You’re almost ready to bring up the cluster, but before you do, make sure that the appropriate ports are open in your firewall.


# Step 5 — Opening the Firewall on Every Server


In this step, you will configure your firewall so that the ports required for inter-node communication are open. On every server, check the status of the firewall by running:


```
sudo ufw status


```


In this case, only SSH is allowed through:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)

```


Since only SSH traffic is permitted in this case, you’ll need to add rules for MySQL and Galera traffic. If you tried to start the cluster, it would fail because of these firewall rules.


Galera can make use of four ports:


- 3306 For MySQL client connections and State Snapshot Transfer that use the mysqldump method.
- 4567 For Galera Cluster replication traffic. Multicast replication uses both UDP transport and TCP on this port.
- 4568 For Incremental State Transfer.
- 4444 For all other State Snapshot Transfer.

In this example, you’ll open all four ports while you do your setup. Once you’ve confirmed that replication is working, you’d want to close any ports you’re not actually using and restrict traffic to just servers in the cluster.


Open the ports with the following commands:


```
sudo ufw allow 3306,4567,4568,4444/tcp
sudo ufw allow 4567/udp


```



Note: Depending on what else is running on your servers, you might want to restrict access right away. The UFW Essentials: Common Firewall Rules and Commands guide can help with this.

After you have configured your firewall on the first node, create the same firewall settings on the second and third node.


Now that you have configured the firewalls successfully, you’re ready to start the cluster in the next step.


# Step 6 — Starting the Cluster


In this step, you will start your MySQL Galera cluster. But first, you will enable the MySQL systemd service, so that MySQL will start automatically whenever the server is rebooted.


## Enable MySQL to Start on Boot on All Three Servers


Use the following command on all three servers to enable the MySQL systemd service:


```
sudo systemctl enable mysql


```


You will see the following output, which shows that the service has been linked successfully to the startup services list:


```
OutputCreated symlink /etc/systemd/system/multi-user.target.wants/mysql.service → /lib/systemd/system/mysql.service.

```


Now that you’ve enabled mysql to start on boot on all of the servers, you’re ready to proceed to bring the cluster up.


## Bring Up the First Node


To bring up the first node, you’ll need to use a special startup script. The way you’ve configured your cluster, each node that comes online will try to connect to at least one other node specified in its galera.cnf file to get its initial state. Without using the mysqld_bootstrap script that allows systemd to pass the --wsrep-new-cluster parameter,  a normal systemctl start mysql would fail because there are no nodes running for the first node to connect with.


Run the following on your first server:


```
sudo mysqld_bootstrap


```


This command will not display any output on successful execution. When this script succeeds, the node is registered as part of the cluster, and you can see it with the following command:


```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"


```


After entering your password, you will see the following output, indicating that there is one node in the cluster:


```
Output+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 1     |
+--------------------+-------+


```


On the remaining nodes, you can start mysql normally. They will search for any member of the cluster list that is online, and when they find one, they will join the cluster.


## Bring Up the Second Node


Now you can bring up the second node. Start mysql:


```
sudo systemctl start mysql


```


No output will be displayed on successful execution. You will see your cluster size increase as each node comes online:


```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"


```


You will see the following output indicating that the second node has joined the cluster and that there are two nodes in total.


```
Output+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 2     |
+--------------------+-------+


```


## Bring Up the Third Node


It’s now time to bring up the third node. Start mysql:


```
sudo systemctl start mysql


```


Run the following command to find the cluster size:


```
mysql -u root -p -e "SHOW STATUS LIKE 'wsrep_cluster_size'"


```


You will see the following output, which indicates that the third node has joined the cluster and that the total number of nodes in the cluster is three.


```
Output+--------------------+-------+
| Variable_name      | Value |
+--------------------+-------+
| wsrep_cluster_size | 3     |
+--------------------+-------+

```


At this point, the entire cluster is online and communicating successfully. Next, you can ensure the working setup by testing replication in the following section.


# Step 7 — Testing Replication


You’ve gone through the steps up to this point so that your cluster can perform replication from any node to any other node, known as active-active replication. In this step, you will test and see if the replication is working as expected.


## Write to the First Node


You’ll start by making database changes on your first node. The following commands will create a database called playground and a table inside of this database called equipment.


```
mysql -u root -p -e 'CREATE DATABASE playground;
CREATE TABLE playground.equipment ( id INT NOT NULL AUTO_INCREMENT, type VARCHAR(50), quant INT, color VARCHAR(25), PRIMARY KEY(id));
INSERT INTO playground.equipment (type, quant, color) VALUES ("slide", 2, "blue");'


```


In the previous command, the CREATE DATABASE statement creates a database named playground. The CREATE statement creates a table named equipment inside the playground database having an auto-incrementing identifier column called id and other columns. The type column, quant column, and color column are defined to store the type, quantity, and color of the equipment respectively. The INSERT statement inserts an entry of type slide, quantity 2 and color blue.


You now have one value in your table.


## Read and Write on the Second Node


Next, look at the second node to verify that replication is working:


```
mysql -u root -p -e 'SELECT * FROM playground.equipment;'


```


The data you entered on the first node will be visible here on the second, proving that replication is working:


```
Output+----+-------+-------+-------+
| id | type  | quant | color |
+----+-------+-------+-------+
|  1 | slide |     2 | blue  |
+----+-------+-------+-------+

```


From this same node, write data to the cluster:


```
mysql -u root -p -e 'INSERT INTO playground.equipment (type, quant, color) VALUES ("swing", 10, "yellow");'


```


## Read and Write on the Third Node


From the third node, you can read all of this data by querying the table again:


```
mysql -u root -p -e 'SELECT * FROM playground.equipment;'


```


You will see the following output showing the two rows:


```
Output   +----+-------+-------+--------+
   | id | type  | quant | color  |
   +----+-------+-------+--------+
   |  1 | slide |     2 | blue   |
   |  2 | swing |    10 | yellow |
   +----+-------+-------+--------+

```


Again, you can add another value from this node:


```
mysql -u root -p -e 'INSERT INTO playground.equipment (type, quant, color) VALUES ("seesaw", 3, "green");'


```


## Read on the First Node


Back on the first node, you can verify that your data is available everywhere:


```
mysql -u root -p -e 'SELECT * FROM playground.equipment;'


```


You will see the following output, which indicates that the rows are available on the first node.


```
Output   +----+--------+-------+--------+
   | id | type   | quant | color  |
   +----+--------+-------+--------+
   |  1 | slide  |     2 | blue   |
   |  2 | swing  |    10 | yellow |
   |  3 | seesaw |     3 | green  |
   +----+--------+-------+--------+

```


You’ve now verified successfully that you can write to all of the nodes and that replication is being performed properly.


# Conclusion


At this point, you have a working three-node Galera test cluster configured. If you plan on using a Galera cluster in a production situation, it’s recommended that you begin with no fewer than five nodes.


Before production use, you may want to take a look at some of the other state snapshot transfer (sst) agents like xtrabackup, which allows you to set up new nodes quickly and without large interruptions to your active nodes. This does not affect the actual replication, but is a concern when nodes are being initialized.


You might also be interested in other clustering solutions for MySQL, in which case you can check out our How To Create a Multi-Node MySQL Cluster on Ubuntu 18.04 tutorial. If you are looking to try out a managed database solution, see our DigitalOcean Managed Databases documentation.


