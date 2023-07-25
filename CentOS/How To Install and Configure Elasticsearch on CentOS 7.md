# How To Install and Configure Elasticsearch on CentOS 7

```Miscellaneous``` ```CentOS```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Elasticsearch is a platform for the distributed search and analysis of data in real time. Its popularity is due to its ease of use, powerful features, and scalability.


Elasticsearch supports RESTful operations. This means that you can use HTTP methods (GET, POST, PUT, DELETE, etc.) in combination with an HTTP URI (/collection/entry) to manipulate your data. The intuitive RESTful approach is both developer and user friendly, which is one of the reasons for Elasticsearch’s popularity.


Elasticsearch is free and open-source software with a solid company behind it — Elastic. This combination makes it suitable for many use cases, from personal testing to corporate integration.


This article will introduce you to Elasticsearch and show you how to install, configure, and start using it.


# Prerequisites


To follow this tutorial you will need the following:


- A server running CentOS 7 with a minimum of 1GB of memory and a non-root sudo user. For detailed instructions, check out our Initial Server Setup Guide for CentOS 7
- wget installed on your server

# Step 1 — Installing Java on CentOS 7


Elasticsearch is written in the Java programming language. Your first task, then, is to install a Java Runtime Environment (JRE) on your server. You will use the native CentOS OpenJDK package for the JRE. This JRE is free, well-supported, and automatically managed through the CentOS Yum installation manager.


Install the latest version of OpenJDK 8:


```
sudo yum install java-1.8.0-openjdk.x86_64


```


Now verify your installation:


```
java -version


```


The command will create an output like this:


```
Outputopenjdk version "1.8.0_262"
OpenJDK Runtime Environment (build 1.8.0_262-b10)
OpenJDK 64-Bit Server VM (build 25.262-b10, mixed mode)

```


When you advance in using Elasticsearch and you start looking for better Java performance and compatibility, you may opt to install Oracle’s proprietary Java (Oracle JDK 8). For more information, reference our article on How To Install Java on CentOS and Fedora.


# Step 2 — Downloading and Installing Elasticsearch on CentOS 7


You can download Elasticsearch directly from elastic.co in zip, tar.gz, deb, or rpm packages. For CentOS, it’s best to use the native rpm package, which will install everything you need to run Elasticsearch.


At the time of this writing, the latest Elasticsearch version is 7.9.2.


From a working directory of your choosing, download the program:


```
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.9.2-x86_64.rpm


```


Then install it using the rpm command:


```
sudo rpm -ivh elasticsearch-7.9.2-x86_64.rpm


```


Elasticsearch will install in /usr/share/elasticsearch/, with its configuration files placed in /etc/elasticsearch and its init script added in /etc/init.d/elasticsearch.


To make sure Elasticsearch starts and stops automatically with the server, add its init script to the default runlevels:


```
sudo systemctl enable elasticsearch.service


```


With Elasticsearch installed, you will now configure a few important settings.


# Step 3 — Configuring Elasticsearch on CentOS 7


Now that you have installed Elasticsearch and its Java dependency, it is time to configure Elasticsearch.


The Elasticsearch configuration files are in the /etc/elasticsearch directory. The ones we’ll review and edit are:


- 
elasticsearch.yml — Configures the Elasticsearch server settings. This is where most options are stored, which is why we are mostly interested in this file.

- 
jvm.options — Provides configuration for the JVM such as memory settings.


The first variables to customize on any Elasticsearch server are node.name and cluster.name in elasticsearch.yml. Let’s do that now.


As their names suggest, node.name specifies the name of the server (node) and the cluster to which the latter is associated. If you don’t customize these variables, a node.name will be assigned automatically in respect to the server hostname. The cluster.name will be automatically set to the name of the default cluster.


The cluster.name value is used by the auto-discovery feature of Elasticsearch to automatically discover and associate Elasticsearch nodes to a cluster. Thus, if you don’t change the default value, you might have unwanted nodes, found on the same network, in your cluster.


Let’s start editing the main elasticsearch.yml configuration file.


Open it using nano or your preferred text editor:


```
sudo nano /etc/elasticsearch/elasticsearch.yml


```


Remove the # character at the beginning of the lines for node.name and cluster.name to uncomment them, and then change their values. Your first configuration changes in the /etc/elasticsearch/elasticsearch.yml file will look like this:


/etc/elasticsearch/elasticsearch.yml
```
...
node.name: "My First Node"
cluster.name: mycluster1
...

```


The networking settings are also found in elasticsearch.yml. By default, Elasticsearch will listen on localhost on port 9200 so that only clients from the same server can connect. You should leave these settings unchanged from a security point of view, because the open source and free edition of Elasticsearch doesn’t offer authentication features.


Another important setting is the node.roles property. You can set this to master-eligible (simply master in the configuration), data or ingest.


The master-eligible role is responsible for the cluster’s health and stability. In large deployments with a lot of cluster nodes, it’s recommended to have more than one dedicated node with a master role only. Typically, a dedicated master node will neither store data nor create indices. Thus, there will be no chance of being overloaded, by which the cluster health could be endangered.


The data role defines the nodes that will store the data. Even if a data node is overloaded, the cluster health shouldn’t be affected seriously, provided there are other nodes to take the additional load.


Lastly, the ingest role allows a node to accept and process data streams. In larger setups, there should be dedicated ingest nodes in order to avoid possible overload on the master and data nodes.



Note: one node may have one or more roles allowing scalability, redundancy and high-availability of the Elasticsearch setup. By default, all of these roles are assigned to the node. This is suitable for a single-node Elasticsearch, as in the example scenario described in this article. Therefore, you don’t have to change the role. Still, if you want to change the role, such as dedicating a node as a master, you can do it by changing /etc/elasticsearch/elasticsearch.yml like this:
/etc/elasticsearch/elasticsearch.yml
...
node.roles: [ master ]
...


Another setting to consider changing is path.data. This determines the path where data is stored, and the default path is /var/lib/elasticsearch. In a production environment it’s recommended that you use a dedicated partition and mount point for storing Elasticsearch data. In the best case, this dedicated partition will be a separate storage media that will provide better performance and data isolation. You can specify a different path.data path by uncommenting the path.data line and changing its value like this:


/etc/elasticsearch/elasticsearch.yml
```
...
path.data: /media/different_media
...

```


Now that you have made all your changes, save and close elasticsearch.yml.


You must also edit your configurations in jvm.options.


Recall that Elasticsearch is run by a JVM, i.e. essentially it’s a Java application. So just as any Java application it has JVM settings that can be configured in the file /etc/elasticsearch/jvm.options. Two of the most important settings, especially in regards to performance, are Xms and Xmx, which define the minimum (Xms) and maximum (Xmx) memory allocation.


By default, both are set to 1GB, but that is almost never optimal. Not only that, but if your server only has 1GB of RAM, you won’t be able to start Elasticsearch with the default settings. This is because the operating system takes at least 100MB so it will not be possible to dedicate 1GB to Elasticsearch.


Unfortunately, there is no universal formula for calculating the memory settings. Naturally, the more memory you allocate, the better your performance, but make sure that there is enough memory left for the rest of the processes on the server. For example, if your machine has 1GB of RAM, you could set both Xms and Xmx to 512MB, thus allowing another 512MB for the rest of the processes. Note that usually both Xms and Xmx are set to the same value in order to avoid the performance penalty of the JVM garbage collection.


If your server only has 1GB of RAM, you must edit this setting.


Open jvm.options:


```
sudo nano /etc/elasticsearch/jvm.options


```


Now change the Xms and Xmx values to 512MB:


/etc/elasticsearch/jvm.options
```
...
-Xms512m
-Xmx512m
...

```


Save and exit the file.


Now start Elasticsearch for the first time:


```
sudo service elasticsearch start


```


Allow at least 10 seconds for Elasticsearch to start before you attempt to use it. Otherwise, you may get a connection error.



Note: You should know that not all Elasticsearch settings are set and kept in configuration files. Instead, some settings are set via its API, like index.number_of_shards and index.number_of_replicas. The first determines into how many pieces (shards) the index will split. The second defines the number of replicas that will be distributed across the cluster. Having more shards improves the indexing performance, while having more replicas makes searching faster.
Assuming that you are still exploring and testing Elasticsearch on a single node, you can play with these settings and alter them by executing the following curl command:
curl -XPUT -H 'Content-Type: application/json' 'http://localhost:9200/_all/_settings?preserve_existing=true' -d '{
  "index.number_of_replicas" : "0",
  "index.number_of_shards" : "1"
}'



With Elasticsearch installed and configured, you will now secure and test the server.


# Step 4 — (Optional) Securing Elasticsearch on CentOS 7


Elasticsearch has no built-in security and anyone who can access the HTTP API can control it. This section is not a comprehensive guide to securing Elasticsearch. Take whatever measures are necessary to prevent unauthorized access to it and the server/virtual machine on which it is running.


By default, Elasticsearch is configured to listen only on the localhost network interface, i.e. remote connections are not possible. You should leave this setting unchanged unless you have taken one or both of the following measures:


- You have limited the access to TCP port 9200 only to trusted hosts with iptables.
- You have created a vpn between your trusted hosts and you are going to expose Elasticsearch on one of the vpn’s virtual interfaces.

Only once you have done the above should you consider allowing Elasticseach to listen on other network interfaces besides localhost. Such a change might be considered when you need to connect to Elasticsearch from another host, for example.


To change the network exposure, open the file elasticsearch.yml:


```
sudo nano /etc/elasticsearch/elasticsearch.yml


```


In this file find the line that contains network.host, uncomment it by removing the # character at the beginning of the line, and then change the value to the IP address of the secured network interface. The line will look something like this:


/etc/elasticsearch/elasticsearch.yml
```
...
network.host: 10.0.0.1
...

```



Warning: Because Elasticsearch doesn’t have any built-in security, it is very important that you do not set this to any IP address that is accessible to any servers that you do not control or trust. Do not bind Elasticsearch to a public or shared private network IP address.

Also, for additional security you can disable scripts that are used to evaluate custom expressions. By crafting a custom malicious expression, an attacker might be able to compromise your environment.


To disable custom expressions, add the following line at the end of the /etc/elasticsearch/elasticsearch.yml file:


```
...
[label /etc/elasticsearch/elasticsearch.yml]
script.allowed_types: none
...

```


For the above changes to take effect, you will have to restart Elasticsearch.


Restart Elasticsearch now:


```
sudo service elasticsearch restart


```


In this step you took some measures to secure your Elasticsearch server. Now you are ready to test the application.


# Step 5 — Testing Elasticsearch on CentOS 7


By now, Elasticsearch should be running on port 9200. You can test this using curl, the command-line tool for client-side URL transfers.


To test the service, make a GET request like this:


```
curl -X GET 'http://localhost:9200'


```


You will see the following response:


```
Output{
  "name" : "My First Node",
  "cluster_name" : "mycluster1",
  "cluster_uuid" : "R23U2F87Q_CdkEI2zGhLGw",
  "version" : {
    "number" : "7.9.2",
    "build_flavor" : "default",
    "build_type" : "rpm",
    "build_hash" : "d34da0ea4a966c4e49417f2da2f244e3e97b4e6e",
    "build_date" : "2020-09-23T00:45:33.626720Z",
    "build_snapshot" : false,
    "lucene_version" : "8.6.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

```


If you see a similar response, Elasticsearch is working properly. If not, recheck the installation instructions and allow some time for Elasticsearch to fully start.


Your Elasticsearch server is now operational. In the next step you will add and retrieve some data from the application.


# Step 6 — Using Elasticsearch on CentOS 7


In this step you will add some data to Elasticsearch and then make some manual queries.


Use curl to add your first entry:


```
curl -H 'Content-Type: application/json' -X POST 'http://localhost:9200/tutorial/helloworld/1' -d '{ "message": "Hello World!" }'


```


You will see the following output:


```
Output{"_index":"tutorial","_type":"helloworld","_id":"1","_version":3,"result":"updated","_shards":{"total":2,"successful":1,"failed":0},"_seq_no":2,"_primary_term":4

```


Using curl, you sent an HTTP POST request to the Elasticseach server. The URI of the request was /tutorial/helloworld/1. Let’s take a closer look at those parameters:


- tutorial is the index of the data in Elasticsearch.
- helloworld is the type.
- 1 is the id of our entry under the above index and type.

Note that it’s also required to set the content type of all POST requests to JSON with the argument -H 'Content-Type: application/json'. If you do not do this Elasticsearch will reject your request.


Now retrieve your first entry using an HTTP GET request:


```
curl -X GET 'http://localhost:9200/tutorial/helloworld/1'


```


The result will look like this:


```
Output{"_index":"tutorial","_type":"helloworld","_id":"1","_version":3,"_seq_no":2,"_primary_term":4,"found":true,"_source":{ "message": "Hello World!" }}

```


To modify an existing entry you can use an HTTP PUT request like this:


```
curl -H 'Content-Type: application/json' -X PUT 'localhost:9200/tutorial/helloworld/1?pretty' -d '
{
  "message": "Hello People!"
}'


```


Elasticsearch will acknowledge successful modification like this:


```
Output{
  "_index" : "tutorial",
  "_type" : "helloworld",
  "_id" : "1",
  "_version" : 2,
  "result" : "updated",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}

```


In the above example you modified the message of the first entry to "Hello People!". With that, the version number increased to 2.


To make the output of your GET operations more human-readable, you can also “prettify” your results by adding the pretty argument:


```
curl -X GET 'http://localhost:9200/tutorial/helloworld/1?pretty'


```


Now the response will output in a more readable format:


```
Output{
  "_index" : "tutorial",
  "_type" : "helloworld",
  "_id" : "1",
  "_version" : 2,
  "_seq_no" : 1,
  "_primary_term" : 1,
  "found" : true,
  "_source" : {
    "message" : "Hello People!"
  }
}

```


This is how you can add and query data in Elasticsearch. To learn about the other operations you can check the Elasticsearch API documentation.


# Conclusion


In this tutorial you installed, configured, and began using Elasticsearch on CentOS 7. Once you are comfortable with manual queries, your next task will be to start using the service from your applications.


