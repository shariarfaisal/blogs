# How To Install Cassandra and Run a Multi-Node Cluster on Ubuntu 22 04

```Apache``` ```Databases``` ```NoSQL``` ```Ubuntu 22.04```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Apache Cassandra is an open-source, masterless, and distributed NoSQL database system. Cassandra is suited for mission-critical applications and multi-node setups because it’s scalable, elastic, and fault-tolerant. Cassandra database management works through a node system, and nodes are held within a cluster.


In this tutorial, you’ll install Cassandra and run a multi-node cluster on Ubuntu 22.04.


# Prerequisites


To complete this tutorial, you’ll need the following:


- At least 2 Ubuntu 22.04 servers configured using the Ubuntu 22.04 initial server setup guide with a sudo non-root user, firewall, and at least 2 GB of RAM in the same datacenter region. Cassandra will not run if deployed on a server with only 1 GB of RAM. See the official hardware requirements guide for Cassandra for more information. It is recommended, but not required, that all the nodes in your cluster have the same or similar specifications.
- Java runtime installed on your machine. You can install OpenJDK 8, OpenJDK 11, Oracle Java Standard Edition 8, or Oracle Java Standard Edition 11 runtime using Step 1 of How To Install Java with Apt on Ubuntu 22.04.
- Cassandra installed on each server using Steps 1 and 2 of How To Install Cassandra and Run a Single-Node Cluster on Ubuntu 22.04.


Note: Be sure to reboot the servers after completing the prerequisite steps and before starting Step 1 of this article.

# Step 1 — Configuring the Firewall to Allow Cassandra Traffic


For a multi-node cluster to function, all member nodes must be able to communicate, which means the firewall must be configured to allow Cassandra traffic. In this step, you will configure the firewall to allow that traffic.


For each node, you will need to allow traffic through the following network ports from other nodes in the cluster:


- 
7000 is the TCP port for commands and data.

- 
9042 is the TCP port for the native transport server. The Cassandra command line utility, cqlsh, will connect to the cluster through this port.


Best security practice dictates that for all nodes within a datacenter region, all communication should be via the internal network interfaces, not the Internet-facing network interfaces. For a DigitalOcean server, the network interface of interest is the private network interface. For each node in your cluster, you’ll need the IP address of that interface.


You can get that information from your DigitalOcean dashboard (from the Networking tab of each Droplet). You can also use the following command to extract the same information through the command line:


```
hostname -I | cut -d' ' -f3


```


The I option to the hostname command causes it to output all the IPv4 addresses associated with the server in a single line, with each address separated by a single space (except the loopback address 127.0.0.1). That output is then piped (|) to the cut command.


The d option tells the cut command how to separate or delimit the received output. In this case, they are separated by a single space.


The f3 option tells the cut command to output the third field, which is the IP address of the private network interface that you want.


Executed on each node, you should now have the IP addresses of the private network interface of all the nodes. The next step is to modify the firewall rules on each node using those IP addresses.


For this tutorial, the first node will be called node1, and the second node will be called node2. Where the prompts refer to node1-internal-ip-address or node2-internal-ip-address, replace that section with the IP addresses you just extracted.


On node1, execute the following command:


```
sudo ufw allow from node2-internal-ip-address to node1-internal-ip-address proto tcp port 7000,9042


```


On node2, reverse the IP addresses like so:


```
sudo ufw allow from node1-internal-ip-address to node2-internal-ip-address proto tcp port 7000,9042


```


Repeat the command for as many nodes in your cluster, only changing the sequence of IP addresses. If you have N nodes in your cluster, you will need to run N - 1 of that command on each node.


The rules take effect immediately, so you don’t need to reload the firewall. You can view the firewall rules on each node with the following command:


```
sudo ufw status numbered


```


The output will show the rule(s) you just added:


```
OutputStatus: active

     To                         Action      From
     --                         ------      ----
[ 1] OpenSSH                    ALLOW IN    Anywhere                  
[ 2] 10.124.0.3 7000,9042/tcp  ALLOW IN    10.124.0.2                
[ 3] OpenSSH (v6)               ALLOW IN    Anywhere (v6)

```


With the firewall rules in place, you should be able to ping one node from the other. Use this command in the terminal for one of your nodes to send three packets to the other node:


```
ping -c 3 internal-ip-address-of-other-node


```


If the packets were transmitted across the firewall, the output should be like so:


```
OutputPING internal-ip-address-of-other-node (internal-ip-address-of-other-node) 56(84) bytes of data.
64 bytes from internal-ip-address-of-other-node: icmp_seq=1 ttl=64 time=0.043 ms
64 bytes from internal-ip-address-of-other-node: icmp_seq=2 ttl=64 time=0.061 ms
64 bytes from internal-ip-address-of-other-node: icmp_seq=3 ttl=64 time=0.066 ms

--- internal-ip-address-of-other-node ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2036ms
rtt min/avg/max/mdev = 0.043/0.056/0.066/0.009 ms

```


If the ping failed, review your firewall rules to set them up again. If you cannot ping one node from the other, you won’t be able to set up your multi-node Cassandra cluster.


With the firewall configured, you can now start setting up Cassandra directly.


# Step 2 — Deleting Cassandra’s Pre-Installed Data


You now have a single-node cluster on each server that will become part of your multi-node cluster. In this step, you’ll set up the nodes to act as one and to function as a multi-node Cassandra cluster.


The commands in this step must be repeated on each node participating in the cluster, so open as many shell terminals as you have nodes in the cluster.


The first command you’ll run on each node will stop the Cassandra daemon:


```
sudo systemctl stop cassandra


```


Then remove (rm) the default dataset that came with each installation of Cassandra:


```
sudo rm -rf /var/lib/cassandra/*


```


The r option recursively deletes all the files and folders under the target directory. The f option says to never prompt the user for input.


When that is completed on all the nodes that are to be part of your cluster, they are now ready to be configured as members of the cluster. You’ll do that in the next step.


# Step 3 — Configuring the Cassandra Cluster


In this step, you’ll make the necessary changes to Cassandra’s configuration file in all the nodes that will be part of the cluster. For fault tolerance and larger productions, you may have multiple seed nodes, but this tutorial will only use one seed node.


The /etc/cassandra/cassandra.yaml configuration file contains many directives and is very well commented. Starting with your designated seed node (node1 in this tutorial), open the configuration file using:


```
sudo nano /etc/cassandra/cassandra.yaml


```


You will modify only the following directives in the file to set up your multi-node Cassandra cluster:


/etc/cassandra/cassandra.yaml
```
...
cluster_name: 'CassandraDOCluster'
...
seed_provider:
  - class_name: org.apache.cassandra.locator.SimpleSeedProvider
    parameters:
         - seeds: "node1-internal-ip-address"
...
listen_address: "targetnode-internal-ip-address"
...
rpc_address: "targetnode-internal-ip-address"
...
endpoint_snitch: GossipingPropertyFileSnitch
...
auto_bootstrap: false

```


Update cluster_name with the name of your cluster. This example uses CassandraDOCluster.


Under the seed_provider section is a comma-delimited list, called - seeds, of the internal IP addresses for your cluster’s seed node(s). In the config file for every node, the - seeds section will have the IP address for the seed node. Treat your cluster’s seed node as unique because that’s the one you’ll be starting Cassandra on first.


The listen_address and rpc_address default to localhost, but both need to be changed to the internal IP address of the target node. For the file on node1, you will put the same node1-internal-ip-address in all three places. For the file on node2, you will put the node1-internal-ip-address under seeds and use the node2-internal-ip-address for the listen_address and rpc_address. Do this for all nodes on your cluster.


The endpoint_snitch gives the name of a snitch class that will be used for locating nodes and routing requests within your Cassandra cluster. By default, it is set to SimpleSnitch, which will only work for a Cassandra cluster within a single datacenter. For production deployments, GossipingPropertyFileSnitch is recommended.


The auto_bootstrap directive is not in the configuration file, so it will need to be added and set to false. It is optional if you’re adding nodes to an existing cluster but required when you’re initializing a new cluster (one with no data).


When you’re finished modifying the file, save and close it. Repeat this step for all the servers you want to include in the cluster, ensuring that the list of seed node(s) is the same and that the listen_address and rpc_address match the internal IP address of the target node.



Note: The Cassandra product documentation states that each node must be restarted after modifying the file, but that was not necessary for the nodes used in this tutorial.

After all the nodes have been properly configured, you will next restart the Cassandra daemon on all the nodes.


# Step 4 — Restarting Cassandra


With all the nodes configured, you can restart the Cassandra daemon on each node, starting with the seed node.


First, run the following command in the terminal for the seed node:


```
sudo systemctl start cassandra


```


Verify that the daemon is active:


```
sudo systemctl status cassandra


```


You will see an output like this:


```
Output● cassandra.service - LSB: distributed storage system for structured data
     Loaded: loaded (/etc/init.d/cassandra; generated)
     Active: active (running) since Sat 2022-07-09 22:43:19 UTC; 22h ago
       Docs: man:systemd-sysv-generator(8)
      Tasks: 70 (limit: 2327)
     Memory: 1.2G
        CPU: 44min 311ms
     CGroup: /system.slice/cassandra.service
             └─18800 /usr/bin/java -ea -da:net.openhft... -XX:+UseThreadPriorities -XX:+HeapDumpOnOutOfMemoryError -Xss256k -XX:+A>

Jul 09 22:43:19 cassa-1 systemd[1]: Starting LSB: distributed storage system for structured data...
Jul 09 22:43:19 cassa-1 systemd[1]: Started LSB: distributed storage system for structured data.

```


Using the same commands, restart the daemon on the other node(s) and verify that the daemon is running on each node.


In this step, you restarted your Cassandra nodes. In the next and final step, you’ll check the status of the cluster and connect to it.


# Step 5 — Connecting to Your Multi-Node Cassandra Cluster


You’ve now completed all the steps necessary to make the nodes into a multi-node cluster. In this step, you will connect to the cluster.


First, verify that the nodes are communicating:


```
sudo nodetool status


```


The output should be:


```
OutputDatacenter: dc1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load        Tokens  Owns (effective)  Host ID                               Rack 
UN  10.124.0.3  991.64 KiB  256     100.0%            9ab882d9-b408-4e75-bd00-79f278e81277  rack1
UN  10.124.0.2  413.57 KiB  256     100.0%            92fc1d95-cf4e-4a68-b1cf-d7e2507fc003  rack1

```


If you can see all the nodes you configured, you successfully set up a multi-node Cassandra cluster.


Next, connect to the cluster using cqlsh. When you use cqlsh, you can specify the IP address of any node in the cluster:


```
cqlsh server-internal-ip-address 9042


```


9042 is the TCP port that cqlsh will use to connect to the cluster.


You will see it connect:


```
OutputConnected to CassandraDOCluster at 10.124.0.2:9042
[cqlsh 6.0.0 | Cassandra 4.0.4 | CQL spec 3.4.5 | Native protocol v5]
Use HELP for help.
cqlsh>

```


You can also query the cluster to see cluster information:


```
describe cluster


```


The output will be like so:


```
OutputCluster: CassandraDOCluster
Partitioner: Murmur3Partitioner
Snitch: DynamicEndpointSnitch

```


Type exit to quit:


```
exit


```


You can now connect to your multi-node cluster.


# Conclusion


You now have a multi-node Cassandra cluster running on Ubuntu 22.04. More information about Cassandra is available at the project’s website. For more on seed nodes, see “What are seeds?” in the FAQ. If you need to troubleshoot the cluster, check the log files in the /var/log/cassandra directory.


