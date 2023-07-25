# How To Set Up an Apache Active-Passive Cluster Using Pacemaker on CentOS 7

```Clustering``` ```Load Balancing``` ```CentOS```

## Introduction


High availability is an important topic nowadays because service outages can be very costly. It’s prudent to take measures which will keep your your website or web application running in case of an outage. With the Pacemaker stack, you can configure a high availability cluster.


Pacemaker is a cluster resource manager. It manages all cluster services (resources) and uses the messaging and membership capabilities of the underlying cluster engine. We will use Corosync as our cluster engine. Resources have a resource agent, which is a external program that abstracts the service.


In an active-passive cluster, all services run on a primary system. If the primary system fails, all services get moved to the backup system. An active-passive cluster makes it possible to do maintenance work without interruption.


In this tutorial, you will learn how to build a high availability Apache active-passive cluster. The web cluster will get addressed by its virtual IP address and will automatically fail over if a node fails.


Your users will access your web application by the virtual IP address, which is managed by Pacemaker. The Apache service and the virtual IP are always located on the same host. When this host fails, they get migrated to the second host and your users will not notice the outage.


## Prerequisites


Before you get started with this tutorial, you will need the following:


- 
Two CentOS 7 Droplets, which will be the cluster nodes. We’ll refer to these as webnode01 (IP address: your_first_server_ip) and webnode02 (IP address: your_second_server_ip).

- 
A user on both servers with root privileges. You can set this up by following this Initial Server Setup with CentOS 7 tutorial.


You’ll have to run some commands on both servers, and some commands on only one.


# Step 1 — Configuring Name Resolution


First, we need to make sure that both hosts can resolve the hostname of the two cluster nodes. To accomplish that, we’ll add entries to the /etc/hosts file. Follow this step on both webnode01 and webnode02.


Open /etc/hosts with nano or your favorite text editor.


```
sudo nano /etc/hosts


```


Add the following entries to the end of the file.


/etc/hosts
```
your_first_server_ip webnode01.example.com webnode01
your_second_server_ip webnode02.example.com webnode02

```


Save and close the file.


# Step 2 — Installing Apache


In this section, we will install the Apache web server. You have to complete this step on both hosts.


First, install Apache.


```
sudo yum install httpd


```


The Apache resource agent uses the Apache server status page for checking the health of the Apache service. You have to activate the status page by creating the file /etc/httpd/conf.d/status.conf.


```
sudo nano /etc/httpd/conf.d/status.conf


```


Paste the following directive in this file. These directives allow the access to the status page from localhost but not from any other host.


/etc/httpd/conf.d/status.conf
```
<Location /server-status>
   SetHandler server-status
   Order Deny,Allow
   Deny from all
   Allow from 127.0.0.1
</Location>

```


Save and close the file.


# Step 3 — Installing Pacemaker


Now we will install the Pacemaker stack. You have to complete this step on both hosts.


Install the Pacemaker stack and the pcs cluster shell. We’ll use the latter later to configure the cluster.


```
sudo yum install pacemaker pcs


```


Now we have to start the pcs daemon, which is used for synchronizing the Corosync configuration across the nodes.


```
sudo systemctl start pcsd.service


```


In order that the daemon gets started after every reboot, we will also enable the service.


```
sudo systemctl enable pcsd.service


```


After you have installed these packages, there will be a new user on your system called hacluster. After the installation, remote login is disabled for this user. For tasks like synchronizing the configuration or starting services on other nodes, we have to set the same password for this user.


```
sudo passwd hacluster


```


# Step 4 — Configuring Pacemaker


Next, we’ll allow cluster traffic in FirewallD to allow our hosts to communicate.


First, check if FirewallD is running.


```
sudo firewall-cmd --state


```


If it’s not running, start it.


```
sudo systemctl start firewalld.service


```


You’ll need to do this on both hosts. Once it’s running, add the high-availability service to FirewallD.


```
sudo firewall-cmd --permanent --add-service=high-availability


```


After this change, you need to reload FirewallD.


```
sudo firewall-cmd --reload


```


If you want to learn more about FirewallD, you can read this guide about how to configure FirewallD on CentOS 7.


Now that our two hosts can talk to each other, we can set up the authentication between the two nodes by running this command on one host (in our case, webnode01).


```
sudo pcs cluster auth webnode01 webnode02
Username: hacluster


```


You should see the following output:


Output
```
webnode01: Authorized
webnode02: Authorized

```


Next, we’ll generate and synchronize the Corosync configuration on the same host. Here, we’ll name the cluster webcluster, but you can call it whatever you like.


```
sudo pcs cluster setup --name webcluster webnode01 webnode02


```


You’ll see the following output:


Output
```
Shutting down pacemaker/corosync services...
Redirecting to /bin/systemctl stop  pacemaker.service
Redirecting to /bin/systemctl stop  corosync.service
Killing any remaining services...
Removing all cluster configuration files...
webnode01: Succeeded
webnode02: Succeeded

```


The corosync configuration is now created and distributed across all nodes. The configuration is stored in the file /etc/corosync/corosync.conf.


# Step 5 — Starting the Cluster


The cluster can be started by running the following command on webnode01.


```
sudo pcs cluster start --all


```


To ensure that Pacemaker and corosync starts at boot, we have to enable the services on both hosts.


```
sudo systemctl enable corosync.service
sudo systemctl enable pacemaker.service


```


We can now check the status of the cluster by running the following command on either host.


```
sudo pcs status


```


Check that both hosts are marked as online in the output.


Output
```
. . .

Online: [ webnode01 webnode02 ]

Full list of resources:


PCSD Status:
  webnode01: Online
  webnode02: Online

Daemon Status:
  corosync: active/enabled
  pacemaker: active/enabled
  pcsd: active/enabled

```



Note: After the first setup, it can take some time before the nodes are marked as online.

# Step 6 — Disabling STONITH and Ignoring Quorum


## What is STONITH?


You will see a warning in the output of pcs status that no STONITH devices are configured and STONITH is not disabled:


Warning
```
. . .
WARNING: no stonith devices and stonith-enabled is not false
. . .

```


What does this mean and why should you care?


When the cluster resource manager cannot determine the state of a node or of a resource on a node, fencing is used to bring the cluster to a known state again.


Resource level fencing ensures mainly that there is no data corruption in case of an outage by configuring a resource. You can use resource level fencing, for instance, with DRBD (Distributed Replicated Block Device) to mark the disk on a node as outdated when the communication link goes down.


Node level fencing ensures that a node does not run any resources. This is done by resetting the node and the Pacemaker implementation of it is called STONITH (which stands for “shoot the other node in the head”). Pacemaker supports a great variety of fencing devices, e.g. an uninterruptible power supply or management interface cards for servers.


Because the node level fencing configuration depends heavily on your environment, we will disable it for this tutorial.


```
sudo pcs property set stonith-enabled=false


```



Note: If you plan to use Pacemaker in a production environment, you should plan a STONITH implementation depending on your environment and keep it enabled.

## What is Quorum?


A cluster has quorum when more than half of the nodes are online. Pacemaker’s default behavior is to stop all resources if the cluster does not have quorum. However, this does not make sense in a two-node cluster; the cluster will lose quorum if one node fails.


For this tutorial, we will tell Pacemaker to ignore quorum by setting the no-quorum-policy:


```
sudo pcs property set no-quorum-policy=ignore


```


# Step 7 — Configuring the Virtual IP address


From now on, we will interact with the cluster via the pcs shell, so all commands need only be executed on one host; it doesn’t matter which one.


The Pacemaker cluster is now up and running and we can add the first resource to it, which is the virtual IP address. To do this, we will configure the ocf:heartbeat:IPaddr2 resource agent, but first, let’s cover some terminology.


Every resource agent name has either three or two fields that are separated by a colon:


- 
The first field is the resource class, which is the standard the resource agent conforms to. It also tells Pacemaker where to find the script. The IPaddr2 resource agent conforms to the OCF (Open Cluster Framework) standard.

- 
The second field depends on the standard. OCF resources use the second field for the OCF namespace.

- 
The third field is the name of the resource agent.


Resources can have meta-attributes and instance attributes. Meta-attributes do not depend on the resource type; instance attributes are resource agent-specific. The only required instance attribute of this resource agent is ip (the virtual IP address), but for the sake of explicitness we will also set cidr_netmask (the subnetmask in CIDR notation).


Resource operations are actions the cluster can perform on a resource (e.g. start, stop, monitor). They are indicated by the keyword op. We will add the monitor operation with an interval of 20 seconds so that the cluster checks every 20 seconds if the resource is still healthy. What’s considered healthy depends on the resource agent.


First, we will create the virtual IP address resource. Here, we’ll use 127.0.0.2 as our virtual IP and Cluster_VIP for the name of the resource.


```
sudo pcs resource create Cluster_VIP ocf:heartbeat:IPaddr2 ip=127.0.0.2 cidr_netmask=24 op monitor interval=20s


```


Next, check the status of the resource.


```
sudo pcs status


```


Look for the following line in the output:


Output
```
...
Full list of resources:

 Cluster_VIP	(ocf::heartbeat:IPaddr2):	Started webnode01
...

```


The virtual IP address is active on the host webnode01.


# Step 8 — Adding the Apache Resource


Now we can add the second resource to the cluster, which will the Apache service. The resource agent of the service is ocf:heartbeat:apache.


We will name the resource WebServer and set the instance attributes configfile (the location of the Apache configuration file) and statusurl (the URL of the Apache server status page). We will choose a monitor interval of 20 seconds again.


```
sudo pcs resource create WebServer ocf:heartbeat:apache configfile=/etc/httpd/conf/httpd.conf statusurl="http://127.0.0.1/server-status" op monitor interval=20s


```


We can query the status of the resource like before.


```
sudo pcs status


```


You should see WebServer in the output running on webnode02.


Output
```
...
Full list of resources:

 Cluster_VIP	(ocf::heartbeat:IPaddr2):	Started webnode01
 WebServer	(ocf::heartbeat:apache):	Started webnode02
...

```


As you can see, the resources run on different hosts. We did not yet tell Pacemaker that these resources must run on the same host, so they are evenly distributed across the nodes.



Note: You can restart the Apache resource by running sudo pcs resource restart WebServer (e.g. if you change the Apache configuration). Make sure not to use systemctl to manage the Apache service.

# Step 9 — Configuring Colocation Constraints


Almost every decision in a Pacemaker cluster, like choosing where a resource should run, is done by comparing scores. Scores are calculated per resource, and the cluster resource manager chooses the node with the highest score for a particular resource. (If a node has a negative score for a resource, the resource cannot run on that node.)


We can manipulate the decisions of the cluster with constraints. Constraints have a score. If a constraint has a score lower than INFINITY, it is only a recommendation. A score of INFINITY means it is a must.


We want to ensure that both resources are run on the same host, so we will define a colocation constraint with a score of INFINITY.


```
sudo pcs constraint colocation add WebServer Cluster_VIP INFINITY


```


The order of the resources in the constraint definition is important. Here, we specify that the Apache resource (WebServer) must run on the same hosts the virtual IP (Cluster_VIP) is active on. This also means that WebSite is not permitted to run anywhere if Cluster_VIP is not active.


It is also possible to define in which order the resources should run by creating ordering constraints or to prefer certain hosts for some resources by creating location constraints.


Verify that both resources run on the same host.


```
sudo pcs status


```


Output
```
...
Full list of resources:

 Cluster_VIP	(ocf::heartbeat:IPaddr2):	Started webnode01
 WebServer	(ocf::heartbeat:apache):	Started webnode01
...

```


Both resources are now on webnode01.


# Conclusion


You have set up an Apache two node active-passive cluster which is accessible by the virtual IP address. You can now configure Apache further, but make sure to synchronize the configuration across the hosts. You can write a custom script for this (e.g. with rsync) or you can use something like csync2.


If you want to distribute the files of your web application among the hosts, you can set up a DRBD volume and integrate it with Pacemaker.


