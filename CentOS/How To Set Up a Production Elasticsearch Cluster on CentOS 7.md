# How To Set Up a Production Elasticsearch Cluster on CentOS 7

```Clustering``` ```Elasticsearch``` ```CentOS```

## Introduction


Elasticsearch is a popular open source search server that is used for real-time distributed search and analysis of data. When used for anything other than development, Elasticsearch should be deployed across multiple servers as a cluster, for the best performance, stability, and scalability.


This tutorial will show you how to install and configure a production Elasticsearch cluster on CentOS 7, in a cloud server environment.


Although manually setting up an Elasticsearch cluster is useful for learning, use of a configuration management tool is highly recommended with any cluster setup. If you want to use Ansible to deploy an Elasticsearch cluster, follow this tutorial: How To Use Ansible to Set Up a Production Elasticsearch Cluster.


# Prerequisites


You must have at least three CentOS 7 servers to complete this tutorial because an Elasticsearch cluster should have a minimum of 3 master-eligible nodes. If you want to have dedicated master and data nodes, you will need at least 3 servers for your master nodes plus additional servers for your data nodes.


If you would prefer to use Ubuntu instead, check out this tutorial: How To Set Up a Production Elasticsearch Cluster on Ubuntu 14.04


## Assumptions


This tutorial assumes that your servers are using a VPN like the one described here: How To Use Ansible and Tinc VPN to Secure Your Server Infrastructure. This will provide private network functionality regardless of the physical network that your servers are using.


If you are using a shared private network, such as DigitalOcean Private Networking, you must use a VPN to protect Elasticsearch from unauthorized access. Each server must be on the same private network because Elasticsearch doesn’t have security built into its HTTP interface. The private network must not be shared with any computers you don’t trust.


We will refer to your servers’ VPN IP addresses as vpn_ip. We will also assume that they all have a VPN interface that is named “tun0”, as described in the tutorial linked above.


# Install Java 8


Elasticsearch requires Java, so we will install that now. We will install a recent version of Oracle Java 8 because that is what Elasticsearch recommends. It should, however, work fine with OpenJDK, if you decide to go that route. Following the steps in this section means that you accept the Oracle Binary License Agreement for Java SE.


Complete this step on all of your Elasticsearch servers.


Change to your home directory and download the Oracle Java 8 (Update 73, the latest at the time of this writing) JDK RPM with these commands:


```
cd ~
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u73-b02/jdk-8u73-linux-x64.rpm"


```


Then install the RPM with this yum command (if you downloaded a different release, substitute the filename here):


```
sudo yum -y localinstall jdk-8u73-linux-x64.rpm


```


Now Java should be installed at /usr/java/jdk1.8.0_73/jre/bin/java, and linked from /usr/bin/java.


You may delete the archive file that you downloaded earlier:


```
rm ~/jdk-8u73-linux-x64.rpm


```


Now that Java 8 is installed, let’s install ElasticSearch.


# Install Elasticsearch


Elasticsearch can be installed with a package manager by adding Elastic’s package repository.


Run the following command to import the Elasticsearch public GPG key into rpm:


```
sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch


```


Create a new yum repository file for Elasticsearch. Note that this is a single command:


```
echo '[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=http://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1
' | sudo tee /etc/yum.repos.d/elasticsearch.repo


```


Install Elasticsearch with this command:


```
sudo yum -y install elasticsearch


```


Be sure to repeat this step on all of your Elasticsearch servers.


Elasticsearch is now installed but it needs to be configured before you can use it.


# Configure Elasticsearch Cluster


Now it’s time to edit the Elasticsearch configuration. Complete these steps on all of your Elasticsearch servers.


Open the Elasticsearch configuration file for editing:


```
sudo vi /etc/elasticsearch/elasticsearch.yml


```


The subsequent sections will explain how the configuration must be modified.


## Bind to VPN IP Address or Interface


You will want to restrict outside access to your Elasticsearch instance, so outsiders can’t access your data or shut down your Elasticsearch cluster through the HTTP API. In other words, you must configure Elasticsearch such that it only allows access to servers on your private network (VPN). To do this, we need to configure each node to bind to the VPN IP address, vpn_ip, or interface, “tun0”.


Find the line that specifies network.host, uncomment it, and replace its value with the respective server’s VPN IP address (e.g. 10.0.0.1 for node01) or interface name. Because our VPN interface is named “tun0” on all of our servers, we can configure all of our servers with the same line:


elasticsearch.yml — network.host
```
network.host: [_tun0_, _local_]

```


Note the addition of “_local_”, which configures Elasticsearch to also listen on all loopback devices. This will allow you to use the Elasticsearch HTTP API locally, from each server, by sending requests to localhost. If you do not include this, Elasticsearch will only respond to requests to the VPN IP address.



Warning: Because Elasticsearch doesn’t have any built-in security, it is very important that you do not set this to any IP address that is accessible to any servers that you do not control or trust. Do not bind Elasticsearch to a public or shared private network IP address!

## Set Cluster Name


Next, set the name of your cluster, which will allow your Elasticsearch nodes to join and form the cluster. You will want to use a descriptive name that is unique (within your network).


Find the line that specifies cluster.name, uncomment it, and replace its value with the your desired cluster name. In this tutorial, we will name our cluster “production”:


elasticsearch.yml — cluster.name
```
cluster.name: production

```


## Set Node Name


Next, we will set the name of each node. This should be a descriptive name that is unique within the cluster.


Find the line that specifies node.name, uncomment it, and replace its value with your desired node name. In this tutorial, we will set each node name to the hostname of server by using the ${HOSTNAME} environment variable:


elasticsearch.yml — node.name
```
node.name: ${HOSTNAME}

```


If you prefer, you may name your nodes manually, but make sure that you specify unique names. You may also leave node.name commented out, if you don’t mind having your nodes named randomly.


## Set Discovery Hosts


Next, you will need to configure an initial list of nodes that will be contacted to discover and form a cluster. This is necessary in a unicast network.


Find the line that specifies discovery.zen.ping.unicast.hosts and uncomment it. Replace its value with an array of strings of the VPN IP addresses or hostnames (that resolve to the VPN IP addresses) of all of the other nodes.


For example, if you have three servers node01, node02, and node03 with respective VPN IP addresses of 10.0.0.1, 10.0.0.2, and 10.0.0.3, you could use this line:


elasticsearch.yml — hosts by IP address
```
discovery.zen.ping.unicast.hosts: ["10.0.0.1", "10.0.0.2", "10.0.0.3"]

```


Alternatively, if all of your servers are configured with name-based resolution of their VPN IP addresses (via DNS or /etc/hosts), you could use this line:


elasticsearch.yml — hosts by name
```
discovery.zen.ping.unicast.hosts: ["node01", "node02", "node03"]

```



Note: The Ansible Playbook in the prerequisite VPN tutorial automatically creates /etc/hosts entries on each server that resolve each VPN server’s inventory hostname (specified in the Ansible hosts file) to its VPN IP address.

## Save and Exit


Your servers are now configured to form a basic Elasticsearch cluster. There are more settings that you will want to update, but we’ll get to those after we verify that the cluster is working.


Save and exit elasticsearch.yml.


## Start Elasticsearch


Now start Elasticsearch:


```
sudo systemctl start elasticsearch


```


Then run this command to start Elasticsearch on boot up:


```
sudo systemctl enable elasticsearch


```


Be sure to repeat these steps (Configure Elasticsearch Cluster) on all of your Elasticsearch servers.


# Check Cluster State


If everything was configured correctly, your Elasticsearch cluster should be up and running. Before moving on, let’s verify that it’s working properly. You can do so by querying Elasticsearch from any of the Elasticsearch nodes.


From any of your Elasticsearch servers, run this command to print the state of the cluster:


```
curl -XGET 'http://localhost:9200/_cluster/state?pretty'


```


You should see output that indicates that a cluster named “production” is running. It should also indicate that all of the nodes you configured are members:


```
Cluster State:{
  "cluster_name" : "production",
  "version" : 36,
  "state_uuid" : "MIkS5sk7TQCl31beb45kfQ",
  "master_node" : "k6k2UObVQ0S-IFoRLmDcvA",
  "blocks" : { },
  "nodes" : {
    "Jx_YC2sTQY6ayACU43_i3Q" : {
      "name" : "node02",
      "transport_address" : "10.0.0.2:9300",
      "attributes" : { }
    },
    "k6k2UObVQ0S-IFoRLmDcvA" : {
      "name" : "node01",
      "transport_address" : "10.0.0.1:9300",
      "attributes" : { }
    },
    "kQgZZUXATkSpduZxNwHfYQ" : {
      "name" : "node03",
      "transport_address" : "10.0.0.3:9300",
      "attributes" : { }
    }
  },
...

```


If you see output that is similar to this, your Elasticsearch cluster is running! If any of your nodes are missing, review the configuration for the node(s) in question before moving on.


Next, we’ll go over some configuration settings that you should consider for your Elasticsearch cluster.


# Enable Memory Locking


Elastic recommends to avoid swapping the Elasticsearch process at all costs, due to its negative effects on performance and stability. One way avoid excessive swapping is to configure Elasticsearch to lock the memory that it needs.


Complete this step on all of your Elasticsearch servers.


Edit the Elasticsearch configuration:


```
sudo vi /etc/elasticsearch/elasticsearch.yml


```


Find the line that specifies bootstrap.mlockall and uncomment it:


elasticsearch.yml — bootstrap.mlockall
```
bootstrap.mlockall: true

```


Save and exit.


Next, open the /etc/sysconfig/elasticsearch file for editing:


```
sudo vi /etc/sysconfig/elasticsearch


```


First, find ES_HEAP_SIZE, uncomment it, and set it to about 50% of your available memory. For example, if you have about 4 GB free, you should set this to 2 GB (2g):


/etc/default/elasticsearch — ES_HEAP_SIZE
```
ES_HEAP_SIZE=2g

```


Next, find and uncomment MAX_LOCKED_MEMORY=unlimited. It should look like this when you’re done:


/etc/default/elasticsearch — MAX_LOCKED_MEMORY
```
MAX_LOCKED_MEMORY=unlimited

```


Save and exit.


The last file to edit is the Elasticsearch systemd unit file. Open it up for editing:


```
sudo vi /usr/lib/systemd/system/elasticsearch.service


```


Find and uncomment LimitMEMLOCK=infinity. It should look like this when you’re done:


/usr/lib/systemd/system/elasticsearch.service — LimitMEMLOCK
```
LimitMEMLOCK=infinity

```


Save and exit.


Now reload the systemctl daemon and restart Elasticsearch to put the changes into place:


```
sudo systemctl daemon-reload
sudo systemctl restart elasticsearch


```


Be sure to repeat this step on all of your Elasticsearch servers.


## Verify Mlockall Status


To verify that mlockall is working on all of your Elasticsearch nodes, run this command from any node:


```
curl http://localhost:9200/_nodes/process?pretty


```


Each node should have a line that says "mlockall" : true, which indicates that memory locking is enabled and working:


```
Nodes process output:...
  "nodes" : {
    "kQgZZUXATkSpduZxNwHfYQ" : {
      "name" : "es03",
      "transport_address" : "10.0.0.3:9300",
      "host" : "10.0.0.3",
      "ip" : "10.0.0.3",
      "version" : "2.2.0",
      "build" : "8ff36d1",
      "http_address" : "10.0.0.3:9200",
      "process" : {
        "refresh_interval_in_millis" : 1000,
        "id" : 1650,
        "mlockall" : true
      }
...

```


If mlockall is false for any of your nodes, review the node’s settings and restart Elasticsearch. A common reason for Elasticsearch failing to start is that ES_HEAP_SIZE is set too high.


# Configure Open File Descriptor Limit (Optional)


By default, your Elasticsearch node should have an “Open File Descriptor Limit” of 64k. This section will show you how to verify this and, if you want to, increase it.


## How to Verify Maximum Open Files


First, find the process ID (PID) of your Elasticsearch process. An easy way to do this is to use the ps command to list all of the processes that belong to the elasticsearch user:


```
ps -u elasticsearch


```


You should see output that looks like this. The number in the first column is the PID of your Elasticsearch (java) process:


```
Output:  PID TTY          TIME CMD
11708 ?        00:00:10 java

```


Then run this command to show the open file limits for the Elasticsearch process (replace the highlighted number with your own PID from the previous step):


```
cat /proc/11708/limits | grep 'Max open files'


```


```
OutputMax open files            65535                65535                files

```


The numbers in the second and third columns indicate the soft and hard limits, respectively, as 64k (65535). This is OK for many setups, but you may want to increase this setting.


## How to Increase Max File Descriptor Limits


To increase the maximum number of open file descriptors in Elasticsearch, you just need to change a single setting.


Open the /usr/lib/systemd/system/elasticsearch.service file for editing:


```
sudo vi /usr/lib/systemd/system/elasticsearch.service


```


Find LimitNOFILE and set it to the limit you desire. For example, if you want a limit of 128k descriptors, change it to 131070:


/usr/lib/systemd/system/elasticsearch.service — LimitNOFILE
```
LimitNOFILE=131070

```


Save and exit.


Now reload the systemctl daemon and restart Elasticsearch to put the changes into place:


```
sudo systemctl daemon-reload
sudo systemctl restart elasticsearch


```


Then follow the previous subsection to verify that the limits have been increased.


Be sure to repeat this step on any of your Elasticsearch servers that require higher file descriptor limits.


# Configure Dedicated Master and Data Nodes (Optional)


There are two common types of Elasticsearch nodes: master and data. Master nodes perform cluster-wide actions, such as managing indices and determining which data nodes should store particular data shards. Data nodes hold shards of your indexed documents, and handle CRUD, search, and aggregation operations. As a general rule, data nodes consume a significant amount of CPU, memory, and I/O.


By default, every Elasticsearch node is configured to be a “master-eligible” data node, which means they store data (and perform resource-intensive operations) and have the potential to be elected as a master node. For a small cluster, this is usually fine; a large Elasticsearch cluster, however, should be configured with dedicated master nodes so that the master node’s stability can’t be compromised by intensive data node work.


## How to Configure Dedicated Master Nodes


Before configuring dedicated master nodes, ensure that your cluster will have at least 3 master-eligible nodes. This is important to avoid a split-brain situation, which can cause inconsistencies in your data in the event of a network failure.


To configure a dedicated master node, edit the node’s Elasticsearch configuration:


```
sudo vi /etc/elasticsearch/elasticsearch.yml


```


Add the two following lines:


elasticsearch.yml — dedicated master
```
node.master: true 
node.data: false

```


The first line, node.master: true, specifies that the node is master-eligible and is actually the default setting. The second line, node.data: false, restricts the node from becoming a data node.


Save and exit.


Now restart the Elasticsearch node to put the change into effect:


```
sudo systemctl restart elasticsearch


```


Be sure to repeat this step on your other dedicated master nodes.


You can query the cluster to see which nodes are configured as dedicated master nodes with this command: curl -XGET 'http://localhost:9200/_cluster/state?pretty'. Any node with data: false and master: true are dedicated master nodes.


## How to Configure Dedicated Data Nodes


To configure a dedicated data node—a data node that is not master-eligible—edit the node’s Elasticsearch configuration:


```
sudo vi /etc/elasticsearch/elasticsearch.yml


```


Add the two following lines:


elasticsearch.yml — dedicated data
```
node.master: false 
node.data: true

```


The first line, node.master: false, specifies that the node is not master-eligible. The second line, node.data: true, is the default setting which allows the node to be a data node.


Save and exit.


Now restart the Elasticsearch node to put the change into effect:


```
sudo systemctl restart elasticsearch


```


Be sure to repeat this step on your other dedicated data nodes.


You can query the cluster to see which nodes are configured as dedicated data nodes with this command: curl -XGET 'http://localhost:9200/_cluster/state?pretty'. Any node that lists master: false and does not list data: false are dedicated data nodes.


## Configure Minimum Master Nodes


When running an Elasticsearch cluster, it is important to set the minimum number of master-eligible nodes that need to be running for the cluster to function normally, which is sometimes referred to as quorum. This is to ensure data consistency in the event that one or more nodes lose connectivity to the rest of the cluster, preventing what is known as a “split-brain” situation.


To calculate the number of minimum master nodes your cluster should have, calculate n / 2 + 1, where n is the total number of “master-eligible” nodes in your healthy cluster, then round the result down to the nearest integer. For example, for a 3-node cluster, the quorum is 2.



Note: Be sure to include all master-eligible nodes in your quorum calculation, including any data nodes that are master-eligible (default setting).

The minimum master nodes setting can be set dynamically, through the Elasticsearch HTTP API. To do so, run this command on any node (replace the highlighted number with your quorum):


```
curl -XPUT localhost:9200/_cluster/settings?pretty -d '{
    "persistent" : {
        "discovery.zen.minimum_master_nodes" : 2
    }
}'


```


```
Output:{
  "acknowledged" : true,
  "persistent" : {
    "discovery" : {
      "zen" : {
        "minimum_master_nodes" : "2"
      }
    }
  },
  "transient" : { }
}

```



Note: This command is a “persistent” setting, meaning the minimum master nodes setting will survive full cluster restarts and override the Elasticsearch configuration file. Also, this setting can be specified as discovery.zen.minimum_master_nodes: 2 in /etc/elasticsearch.yml if you have not already set it dynamically.

If you want to check this setting later, you can run this command:


```
curl -XGET localhost:9200/_cluster/settings?pretty


```


# How To Access Elasticsearch


You may access the Elasticsearch HTTP API by sending requests to the VPN IP address any of the nodes or, as demonstrated in the tutorial, by sending requests to localhost from one of the nodes.


Your Elasticsearch cluster is accessible to client servers via the VPN IP address of any of the nodes, which means that the client servers must also be part of the VPN.


If you have other software that needs to connect to your cluster, such as Kibana or Logstash, you can typically configure the connection by providing your application with the VPN IP addresses of one or more of the Elasticsearch nodes.


# Conclusion


Your Elasticsearch cluster should be running in a healthy state, and configured with some basic optimizations!


Elasticsearch has many other configuration options that weren’t covered here, such as index, shard, and replication settings. It is recommended that you revisit your configuration later, along with the official documentation, to ensure that your cluster is configured to meet your needs.


