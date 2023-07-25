# How To Use Ansible to Set Up a Production Elasticsearch Cluster

```Ubuntu``` ```Elasticsearch``` ```Clustering``` ```CentOS```

## Introduction


In this tutorial, we’ll show you how to use Ansible, a configuration management tool, to install a production Elasticsearch cluster on Ubuntu 14.04 or CentOS 7 in a cloud server environment. We will build upon the How To Use Ansible and Tinc VPN to Secure Your Server Infrastructure tutorial to ensure that your Elasticsearch nodes will be secure from computers outside of your own network.


Elasticsearch is a popular open source search server that is used for real-time distributed search and analysis of data. When used for anything other than development, Elasticsearch should be deployed across multiple servers as a cluster, for the best performance, stability, and scalability.


# Prerequisites


You must have at least three Ubuntu 14.04 or CentOS 7 servers, with private networking, to complete this tutorial because an Elasticsearch cluster should have a minimum of 3 master-eligible nodes. If you want to have dedicated master and data nodes, you will need at least 3 servers for master nodes plus additional servers for any data nodes. Also note that, if you plan on using the default Elasticsearch heap size of 2 GB, your servers should be allocated at least 4 GB of memory.


After obtaining your servers, configure them to use a mesh VPN with this tutorial: How To Use Ansible and Tinc VPN to Secure Your Server Infrastructure. Make sure that each server has a unique Ansible inventory hostname.


If you are using a private network, such as DigitalOcean Private Networking, your servers will be able to communicate securely with other servers on the same account or team within the same region. This is particularly important when using Elasticsearch, as it doesn’t have security built into its HTTP interface.


## Assumptions


We will assume that all of the servers that you want to use as Elasticsearch nodes have a VPN interface that is named “tun0”, as described in the tutorial linked above. If they don’t, and you would rather have your ES nodes listen on a different interface, you will have to make the appropriate changes in site.yml file of the Playbook.


We will also assume that your Playbook is located in a directory called ansible-tinc in the home directory of your local computer.


# Download the ansible-elasticsearch Playbook


Elastic provides an Ansible role that can be used to easily set up an Elasticsearch cluster. To use it, we simply need to add it to our ansible-tinc playbook and define a few host groups and assign the appropriate roles to the groups. Again, if you haven’t already followed the prerequisite VPN tutorial, it can be found here.


First, change to the directory that your Tinc Ansible Playbook is in:


```
cd ~/ansible-tinc


```


Then clone the ansible-elasticsearch role, which is available on Elastic’s GitHub account, to the Playbook’s roles directory:


```
cd roles
git clone https://github.com/elastic/ansible-elasticsearch


```


Rename the role to “elasticsearch”:


```
mv ansible-elasticsearch elasticsearch


```


# Update site.yml


Let’s edit the master Playbook file, site.yml, to map three different Elasticsearch roles to three different Ansible host groups. This will allow us to create dedicated master, dedicated data, and master-eligible/data Elasticsearch nodes by simply adding hosts to the appropriate groups.


Change back to the Ansible Playbook’s directory:


```
cd ~/ansible-playbook


```


In your favorite editor, edit a new file called elasticsearch.yml. We’ll use vi:


```
vi site.yml


```


## Map Elasticsearch Dedicated Master Role to Group


At the bottom of the file, map the dedicated master elasticsearch role to the elasticsearch_master_nodes group by adding these lines:


site.yml — Dedicated master nodes
```
- hosts: elasticsearch_master_nodes
  roles:
    - { role: elasticsearch, es_instance_name: "node1", es_config: { discovery.zen.ping.unicast.hosts: "node01, node02, node03", network.host: "_tun0_, _local_", cluster.name: "production", discovery.zen.ping.multicast.enabled: false,  http.port: 9200, transport.tcp.port: 9300, node.data: false, node.master: true, bootstrap.mlockall: true } }
  vars:
    es_major_version: "2.x"
    es_version: "2.2.1"
    es_heap_size: "2g"
    es_cluster_name: "production"

```


This role will create dedicated master nodes because it configures the nodes with these values: node.master: true and node.data: false.


Be sure to update the highlighted hostnames in the discovery.zen.ping.unicast.hosts variable to match the Ansible inventory hostnames (or VPN IP addresses) of a few of your Elasticsearch servers. This will allow these nodes to discover the Elasticsearch cluster. In the example, we are using node01, node02, and node03 because those were the hostnames used in the prerequisite VPN tutorial. Also, if your VPN interface is named something other than “tun0”, update the network.host variable accordingly.


If you want to use a different version of Elasticsearch, update es_version. Note that this configuration won’t work for versions prior to 2.2 because older versions do not accept comma-delimited lists for the network.host variable.


Update es_heap_size to a value that is roughly half of the free memory on your dedicated master servers. For example, if your server has about 4 GB free, set the heap size to “2g”.


Now any hosts that belong to the elasticsearch_master_nodes Ansible host group will be configured as dedicated master Elasticsearch nodes.


## Map Elasticsearch Master/Data Role to Group


At the bottom of the file, map the master-eligible and data elasticsearch role to the elasticsearch_master_data_nodes group by adding these lines:


site.yml — Master-eligible/data nodes
```
- hosts: elasticsearch_master_data_nodes
  roles:
    - { role: elasticsearch, es_instance_name: "node1", es_config: { discovery.zen.ping.unicast.hosts: "node01, node02, node03", network.host: "_tun0_, _local_", cluster.name: "production", discovery.zen.ping.multicast.enabled: false, http.port: 9200, transport.tcp.port: 9300, node.data: true, node.master: true, bootstrap.mlockall: true } }
  vars:
    es_major_version: "2.x"
    es_version: "2.2.1"
    es_heap_size: "2g"
    es_cluster_name: "production"

```


This role will create data nodes that are master-eligible because it configures the nodes with these values: node.master: true and node.data: true.


Be sure to update the highlighted hostnames in the discovery.zen.ping.unicast.hosts variable to match the Ansible inventory hostnames (or VPN IP addresses) of a few of your Elasticsearch servers. Also, if your VPN interface is named something other than “tun0”, update the network.host variable accordingly.


Set es_version to the same value that you used for the dedicated master role.


Update es_heap_size to a value that is roughly half of the free memory on your master-eligible/data servers.


Now any hosts that belong to the elasticsearch_master_data_nodes Ansible host group will be configured as data nodes that are master-eligible.


## Map Elasticsearch Dedicated Data Role to Group


At the bottom of the file, map the dedicated data elasticsearch role to the elasticsearch_data_nodes group by adding these lines:


site.yml — Dedicated data nodes
```
- hosts: elasticsearch_data_nodes
  roles:
    - { role: elasticsearch, es_instance_name: "node1", es_config: { discovery.zen.ping.unicast.hosts: "node01, node02, node03", network.host: "_tun0_, _local_", cluster.name: "production", discovery.zen.ping.multicast.enabled: false, http.port: 9200, transport.tcp.port: 9300, node.data: true, node.master: false, bootstrap.mlockall: true } }
  vars:
    es_major_version: "2.x"
    es_version: "2.2.1"
    es_heap_size: "2g"
    es_cluster_name: "production"

```


This role will create dedicated data nodes because it configures the nodes with these values: node.master: false and node.data: true.


Be sure to update the highlighted hostnames in the discovery.zen.ping.unicast.hosts variable to match the Ansible inventory hostnames (or VPN IP addresses) of a few of your Elasticsearch servers. Also, if your VPN interface is named something other than “tun0”, update the network.host variable accordingly.


Set es_version to the same value that you used in the previous roles.


Update es_heap_size to a value that is roughly half of the free memory on your dedicated data servers.


Now any hosts that belong to the elasticsearch_data_nodes Ansible host group will be configured as dedicated data Elasticsearch nodes.


## Save and Exit


Now that you’ve defined the three roles, and mapped them to host groups, you can save and exit site.yml.


Feel free to add more Elasticsearch roles and host group mappings later.


# Update Host Inventory File


Now that the new Elasticsearch roles have been mapped to host groups, you can create different types of Elasticsearch nodes by simply adding the hosts to the appropriate host groups.


Edit the Ansible hosts inventory file:


```
vi hosts


```


If you followed the prerequisite tutorial, your file should look something like this (with your server hostnames and IP addresses):


Ansible hosts inventory — Original file
```
[vpn]
node01 vpn_ip=10.0.0.1 ansible_host=45.55.41.106
node02 vpn_ip=10.0.0.2 ansible_host=159.203.104.93
node03 vpn_ip=10.0.0.3 ansible_host=159.203.104.127
node04 vpn_ip=10.0.0.4 ansible_host=159.203.104.129

[removevpn]

```


Now add three groups that correspond to the mappings that we defined in site.yml.


Ansible hosts inventory — Elasticsearch groups
```
[elasticsearch_master_nodes]

[elasticsearch_master_data_nodes]

[elasticsearch_data_nodes]

```


Now distribute your Elasticsearch hosts among the new host groups, depending on the types of Elasticsearch nodes that you want your cluster to consist of. For example, if you want three dedicated master nodes and a single dedicated data node, your inventory file would look something like this:


Ansible hosts inventory — Complete example
```
[vpn]
node01 vpn_ip=10.0.0.1 ansible_host=45.55.41.106
node02 vpn_ip=10.0.0.2 ansible_host=159.203.104.93
node03 vpn_ip=10.0.0.3 ansible_host=159.203.104.127
node04 vpn_ip=10.0.0.4 ansible_host=159.203.104.129

[removevpn]

[elasticsearch_master_nodes]
node01
node02
node03

[elasticsearch_master_data_nodes]

[elasticsearch_data_nodes]
node04

```



Note: Each Elasticsearch node must also be defined in the [vpn] host group so that all nodes can communicate with each other over the VPN. Also, any server that needs to connect to the Elasticsearch cluster must also be defined in the [vpn] host group.

Once your inventory file reflects your desired Elasticsearch (and VPN) setup, save and exit.


# Create Elasticsearch Cluster


Now that site.yml and hosts are set up, you are ready to create your Elasticsearch cluster by running the Playbook.


Run the Playbook with this command:


```
ansible-playbook site.yml


```


After the Playbook completes its run, your Elasticsearch cluster should be up and running. The next step is to verify that everything is working properly.


# Verify Elasticsearch Cluster Status


From any of your Elasticsearch servers, run this command to print the state of the cluster:


```
curl -XGET 'http://localhost:9200/_cluster/state?pretty'


```


You should see output that indicates that a cluster named “production” is running. It should also indicate that all of the nodes you configured are members:


```
Cluster State:{
  "cluster_name" : "production",
  "version" : 8,
  "state_uuid" : "SgTyn0vNTTu2rdKPrc6tkQ",
  "master_node" : "OzqMzte9RYWSXS6OkGhveA",
  "blocks" : { },
  "nodes" : {
    "OzqMzte9RYWSXS6OkGhveA" : {
      "name" : "node02-node1",
      "transport_address" : "10.0.0.2:9300",
      "attributes" : {
        "data" : "false",
        "master" : "true"
      }
    },
    "7bohaaYVTeeOHvSgBFp-2g" : {
      "name" : "node04-node1",
      "transport_address" : "10.0.0.4:9300",
      "attributes" : {
        "master" : "false"
      }
    },
    "cBat9IgPQwKU_DPF8L3Y1g" : {
      "name" : "node03-node1",
      "transport_address" : "10.0.0.3:9300",
      "attributes" : {
        "master" : "false"
      }
    },
...

```


If you see output that is similar to this, your Elasticsearch cluster is running! If some of your nodes are missing, review your Ansible hosts inventory to make sure that your host groups are defined properly.


## Troubleshooting


If you get curl: (7) Failed to connect to localhost port 9200: Connection refused, Elasticsearch isn’t running on that server. This is usually caused by Elasticsearch configuration errors in the site.yml file, such as incorrect network.host or discovery.zen.ping.unicast.hosts entries. In addition to reviewing that file, also check the Elasticsearch logs on your servers (/var/log/elasticsearch/node01-node1/production.log) for clues.


If you want to see an example of the Playbook produced by following this tutorial, check out this GitHub repository. This should help you see what a working site.yml and hosts file looks like.


# Conclusion


Your Elasticsearch cluster should be running in a healthy state, and configured with some basic optimizations!


Elasticsearch has many other configuration options that weren’t covered here, such as index, shard, and replication settings. It is recommended that you revisit your configuration later, along with the official documentation, to ensure that your cluster is configured to meet your needs.


