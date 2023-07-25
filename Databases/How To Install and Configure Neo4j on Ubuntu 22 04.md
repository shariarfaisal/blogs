# How To Install and Configure Neo4j on Ubuntu 22 04

```Databases``` ```Ubuntu``` ```NoSQL``` ```Ubuntu 22.04```

## Introduction


Neo4j is a graph database that records relationships between data nodes, whereas traditional relational databases use rows and columns to store and structure data. Since each node stores references to all the other nodes that it is connected to, Neo4j can encode and query complex relationships with minimal overhead.


# Prerequisites


To follow this tutorial, you will need the following:


- One Ubuntu 22.04 server set up by following the Ubuntu 22.04 initial server setup guide, including a sudo-enabled non-root user and a firewall.

# Step 1 — Installing Neo4j


The official Ubuntu package repositories do not contain a copy of the Neo4j database engine. To install the upstream supported package from Neo4j you’ll need to add the GPG key from Neo4j to ensure package downloads are valid. Then you’ll add a new package source pointing to the Neo4j software repository, and finally install the package.


To get started, download and pipe the output of the following curl command to the gpg --dearmor command. This step will convert the downloaded key into a format that apt can use to verify packages:


```
curl -fsSL https://debian.neo4j.com/neotechnology.gpg.key |sudo gpg --dearmor -o /usr/share/keyrings/neo4j.gpg


```


Next, add the Neo4j 4.1 repository to your system’s APT sources:


```
echo "deb [signed-by=/usr/share/keyrings/neo4j.gpg] https://debian.neo4j.com stable 4.1" | sudo tee -a /etc/apt/sources.list.d/neo4j.list


```


The [signed-by=/usr/share/keyrings/neo4j.gpg] portion of the file instructs apt to use the key that you downloaded to verify repository and file information for neo4j packages.


The next step is to update your package lists, and then install the Neo4j package and all of its dependencies. This step will download and install a compatible Java package, so you can enter Y when the apt command prompts you to install all the dependencies:


```
sudo apt update


```


```
sudo apt install neo4j


```


Once the installation process is complete, Neo4j should be running. However, it is not set to start on a reboot of your system. So the last setup step is to enable it as a service and then start it:


```
sudo systemctl enable neo4j.service


```


Now start the service if it is not already running:


```
sudo systemctl start neo4j.service


```


After completing all of these steps, examine Neo4j’s status using the systemctl command:


```
sudo systemctl status neo4j.service


```


You should have output that is similar to the following:


```
Output● neo4j.service - Neo4j Graph Database
     Loaded: loaded (/lib/systemd/system/neo4j.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-04-29 15:01:36 UTC; 2s ago
   Main PID: 2053 (java)
      Tasks: 40 (limit: 38383)
     Memory: 658.1M
     CGroup: /system.slice/neo4j.service
. . .

```


There will be other verbose lines of output, but the important things to note are the highlighted enabled and running lines. Once you have Neo4j installed and running, you can move on to the next set of steps, which will guide you through connecting to Neo4j, configuring credentials, and inserting nodes into the database.


# Step 2 — Connecting to and Configuring Neo4j


Now that you have Neo4j installed and configured to run after any reboot, you can test connecting to the database, and configure administrator credentials.


To interact with Neo4j on the command line, use the cypher-shell utility. Invoke the utility like this:


```
cypher-shell


```


When you first invoke the shell, you will login using the default administrative neo4j user and neo4j password combination. Once you are authenticated, Neo4j will prompt you to change the administrator password:


```
cypher-shell promptusername: neo4j
password: *****
Password change required
new password: ********************
Connected to Neo4j 4.1.0 at neo4j://localhost:7687 as user neo4j.
Type :help for a list of available commands or :exit to exit the shell.
Note that Cypher queries must end with a semicolon.
neo4j@neo4j>

```


In this example, the highlighted ******************** is the masked version of the new password. Choose your own strong and memorable password and be sure to record it somewhere safe. Once you set the password you will be connected to the interactive neo4j@neo4j>  prompt where you can interact with Neo4j databases by inserting and querying nodes.



Note: The Community Edition of Neo4j supports running a single database at a time. Additionally, the Community version does not include the capability to assign roles and permissions to users so those steps are not included in this tutorial. For more information about various features that are supported by the Community Edition of Neo4j, consult the Neo4j Documentation here.

Now that you have set an administrator password and tested connecting to Neo4j, exit from the cypher-shell prompt by typing :exit:


```
:exit


```


Next you can optionally configure Neo4j to accept remote connections.


# Step 3 (Optional) — Configuring Neo4j for Remote Access


If you would like to incorporate Neo4j into a larger application or environment that uses multiple servers, then you will need to configure it to accept connections from other systems. In this step you will configure Neo4j to allow remote connections, and you will also add firewall rules to restrict which systems can connect to your Neo4j server.


By default Neo4j is configured to accept connections from localhost only (127.0.0.1 is the IP address for localhost). This configuration ensures that your Neo4j server is not exposed to the public Internet, and that only users with access to the local system can interact with Neo4j.


To change the network socket that Neo4j uses from localhost to one that other systems can use, you will need to edit the /etc/neo4j/neo4j.conf file. Open the configuration file in your preferred editor and find the dbms.default_listen_address setting. The following example uses nano to edit the file:


```
sudo nano /etc/neo4j/neo4j.conf


```


Locate the commented out #dbms.default_listen_address=0.0.0.0 line and uncomment it by removing the leading # comment character.


/etc/neo4j/neo4j.conf
```
. . .
#*****************************************************************
# Network connector configuration
#*****************************************************************

# With default configuration Neo4j only accepts local connections.
# To accept non-local connections, uncomment this line:
dbms.default_listen_address=0.0.0.0
. . .

```


By default, the value 0.0.0.0 will bind Neo4j to all available IPv4 interfaces on your system, including localhost. If you would like to limit Neo4j to a particular IP address, for example a private network IP that your servers use for a datapath, specify the IP address that is assigned to your server’s private network interface here.


You can also configure Neo4j to use IPv6 interfaces. As with IPv4, you can set the default_listen_address value to a specific IPv6 address that you will use to communicate with Neo4j. If you want to limit Neo4j to only use the local IPv6 address for your server, specify ::1, which corresponds to localhost using IPv6 notation.


When you are finished configuring the default IP address that Neo4j will use for connections, save and close neo4j.conf. If you’re using nano, you can do so by pressing CTRL+X, followed by Y and then ENTER.



Note: If you configure Neo4j with an IPv6 address, you will not be able to connect to Neo4j with  cypher-shell using the IPv6 address directly. Instead, you will need to either configure a DNS name that resolves to the IPv6 address, or add an entry in the remote system’s /etc/hosts file that maps the address to a name. You will then be able to use the DNS or hosts file name to connect to Neo4j using IPv6 from your remote system.
For example, a Neo4j server with an IPv6 address like 2001:db8::1 would require the remote connecting system to have an /etc/hosts entry like the following, substituting a name in place of the highlighted your_hostname:
/etc/hosts
. . .
2001:db8::1 your_hostname

You would then connect to the server from the remote system using the name that you specified like this:
cypher-shell -a 'neo4j://your_hostname:7687'


If you restrict Neo4j to use the IPv6 localhost address of ::1, then you can connect to it locally on the Neo4j server itself using the preconfigured ip6-localhost name from your /etc/hosts file like this:
cypher-shell -a 'neo4j://ip6-localhost:7687'


Once you invoke cypher-shell with the connection URI, you will be prompted for your username and password as usual.

Now that you have configured Neo4j to allow remote connections, it is important to limit remote access so that only trusted systems can connect to it. To restrict remote access to Neo4j, you can use Ubuntu’s default UFW firewall. If you followed the prerequisite Initial Server Setup with Ubuntu 22.04 tutorial then UFW is already installed and ready for use on your server.


Neo4j creates two network sockets in a default installation, one on port 7474 for the built-in HTTP interface, and the main bolt protocol on port 7687. Neo4j recommends not using the HTTP port in production, so create firewall rules for port 7687 only.


To configure the firewall to allow a trusted remote host access to the bolt interface using IPv4, type the following command:


UFW IPv4 Single Host Example
```
sudo ufw allow from 203.0.113.1 to any port 7687 proto tcp


```


Substitute the IP address of the trusted remote system that you will use to access Neo4j in place of the highlighted 203.0.113.1 value.


If you want to allow an entire network range access, like a private management or datapath network, use a rule like this one:


UFW IPv4 Network Example
```
sudo ufw allow from 192.0.2.0/24 to any port 7687 proto tcp


```


Again, substitute the network that you would like to have access to Neo4j in place of the highlighted 192.0.2.0/24 network.


If you would like to allow hosts to access Neo4j remotely using IPv6, add a rule like the following:


UFW IPv6 Single Host Example
```
sudo ufw allow from 2001:DB8::1/128 to any port 7687 proto tcp


```


Substitute your trusted system’s IPv6 address in place of the highlighted 2001:DB8::1/128 address.


As with IPv4, you can also allow a range of IPv6 addresses access to your Neo4j server. To do so create a UFW rule like the following:


UFW IPv6 Single Host Example
```
sudo ufw allow from 2001:DB8::/32 to any port 7687 proto tcp


```


Again substitute in your trusted network range in place of the highlighted 2001:DB8::/32 network range.


Once you have created the appropriate UFW rule or rules for your network configuration and trusted hosts or networks, enable UFW to make the rules take effect:


```
sudo ufw reload


```


You can examine the currently loaded UFW rules using the ufw status command. Run it to ensure that the addresses or networks that you specified can access Neo4j on port 7687:


```
sudo ufw status


```


You should have output that is similar to the following:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW       Anywhere
22/tcp (v6)                ALLOW       Anywhere (v6)
7687/tcp                   ALLOW       203.0.113.1

```


You now have a Neo4j server that is configured to allow access on port 7687 to a trusted remote server or network. In the next section of this tutorial you will learn about adding nodes to the database, and how to define relationships between them.


# Step 4 — Using Neo4j


To start using Neo4j, let’s add some example nodes and then define relationships between them. Connect to Neo4j using cypher-shell.


```
cypher-shell


```



Note: If you configured Neo4j to allow remote access in [Step 3 (Optional) — Configuring Neo4j for Remote Access](step-3-optional-configuring-neo4j-for-remote-access], connect using a URI that corresponds to the address of your Neo4j server. For example if your Neo4j server’s IP is 203.0.113.1, then connect to it like this from your remote system:
cypher-shell -a 'neo4j://203.0.113.1:7687'


You will be prompted for your username and password as usual.
If you are using IPv6, ensure that you have an /etc/hosts entry with a name as described in Step 3. Then connect to your Neo4j server from your remote system with a cypher-shell command like this:
cypher-shell -a 'neo4j://your_hostname:7687'


Again, ensure that the highlighted your_hostname maps to your Neo4j server’s IPv6  address in your remote system’s /etc/hosts file.

Once you have logged into Neo4j with your username and password, you can query and add nodes and relationships to the database.


To get started, add a Great White shark node to Neo4j. The following command will create a node of type Shark, with a name Great White.


```
CREATE (:Shark {name: 'Great White'});


```


After each command you will receive output that is similar to the following:


```
Output0 rows available after 3 ms, consumed after another 0 ms
Added 1 nodes, Set 1 properties, Added 1 labels

```



Note: A full explanation of each of the following cypher queries is beyond the scope of this tutorial. For details about the syntax of the cypher query language, refer to the The Neo4j Cypher Manual .

Next, add some more sharks, and relate them using a relationship called FRIEND. Neo4j allows you to relate nodes with arbitrarily named relationships, so FRIEND can be whatever label for a relationship that you would like to use.


In the following example you’ll add three sharks, and link them together using a relationship called FRIEND:


```
CREATE
(:Shark {name: 'Hammerhead'})-[:FRIEND]->
(:Shark {name: 'Sammy'})-[:FRIEND]->
(:Shark {name: 'Megalodon'});


```


You should receive output that indicates the three new sharks were added to the database:


```
Output. . .
Added 3 nodes, Created 2 relationships, Set 3 properties, Added 3 labels

```


Neo4j allows you to relate nodes using arbitrary names for relationships, so in addition to their existing FRIEND relation, Sammy and Megalodon can also be related using a taxonomic rank.


Sammy and Megalodon share a common order of Lamniformes. Since relationships can have properties just like nodes, create an ORDER relationship with a name property that is set to Lamniformes to help describe one of Sammy and Megalodon’s relationships:


```
MATCH (a:Shark),(b:Shark)
WHERE a.name = 'Sammy' AND b.name = 'Megalodon'
CREATE (a)-[r:ORDER { name: 'Lamniformes' }]->(b)
RETURN type(r), r.name;


```


After adding that relationship, you should have output like the following:


```
Output+-------------------------+
| type(r) | r.name        |
+-------------------------+
| "ORDER" | "Lamniformes" |
+-------------------------+

1 row available after 2 ms, consumed after another 7 ms
Created 1 relationships, Set 1 properties

```


Next, add a SUPERORDER relationship between Sammy and Hammerhead based on their taxonomic superorder, which is Selachimorpha. Again, the relationship is given a name property, which is set to Selachimorpha:


```
MATCH (a:Shark),(b:Shark)
WHERE a.name = 'Sammy' AND b.name = 'Hammerhead'
CREATE (a)-[r:SUPERORDER { name: 'Selachimorpha'}]->(b)
RETURN type(r), r.name;


```


Again you will receive output that indicates the type of the relationship, along with the name that was added to describe the relation:


```
Output+--------------------------------+
| type(r)      | r.name          |
+--------------------------------+
| "SUPERORDER" | "Selachimorpha" |
+--------------------------------+

1 row available after 2 ms, consumed after another 8 ms
Created 1 relationships, Set 1 properties

```


Finally, with all these nodes and relationships defined and stored in Neo4j, examine the data using the following query:


```
MATCH (a)-[r]->(b)
RETURN a.name,r,b.name
ORDER BY r;


```


You should receive output like the following:


```
Output+---------------------------------------------------------------------+
| a.name       | r                                     | b.name       |
+---------------------------------------------------------------------+
| "Hammerhead" | [:FRIEND]                             | "Sammy"      |
| "Sammy"      | [:FRIEND]                             | "Megalodon"  |
| "Sammy"      | [:ORDER {name: "Lamniformes"}]        | "Megalodon"  |
| "Sammy"      | [:SUPERORDER {name: "Selachimorpha"}] | "Hammerhead" |
+---------------------------------------------------------------------+

4 rows available after 72 ms, consumed after another 1 ms

```


The output includes the FRIEND relationships that were defined between Hammerhead, Sammy, and Megalodon, as well as the ORDER and SUPERORDER taxonomic relationships.


When you are finished adding and exploring nodes and relationships to your Neo4j database, type the :exit command to leave the cypher-shell.


# Conclusion


You have now installed, configured, and added data to Neo4j on your server. You also optionally configured Neo4j to accept connections from remote systems and secured it using UFW.


If you want to learn more about using Neo4j and the cypher query language, consult the official
Neo4j Documentation.


