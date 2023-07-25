# How To Install Apache Kafka on Debian 10

```Apache``` ```Messaging``` ```Debian 10``` ```Open Source```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Apache Kafka is a popular distributed message broker designed to handle large volumes of real-time data. A Kafka cluster is highly scalable and fault-tolerant, and also has a much higher throughput compared to other message brokers such as ActiveMQ and RabbitMQ. Though it is generally used as a publish/subscribe messaging system, a lot of organizations also use it for log aggregation because it offers persistent storage for published messages.


A publish/subscribe messaging system allows one or more producers to publish messages without considering the number of consumers or how they will process the messages. Subscribed clients are notified automatically about updates and the creation of new messages. This system is more efficient and scalable than systems where clients poll periodically to determine if new messages are available.


In this tutorial, you will install and configure Apache Kafka 2.1.1 securely on a Debian 10 server, then test your setup by producing and consuming a Hello World message. You will then optionally install KafkaT to monitor Kafka and set up a Kafka multi-node cluster.


# Prerequisites


To follow along, you will need:


- One Debian 10 server with at least 4GB of RAM and a non-root user with sudo privileges. Follow the steps specified in our Initial Server Setup guide for Debian 10 if you do not have a non-root user set up.
- OpenJDK 11 installed on your server. To install this version, follow the instructions in How To Install Java with Apt on Debian 10 on installing specific versions of OpenJDK. Kafka is written in Java, so it requires a JVM.


Note: Installations without 4GB of RAM may cause the Kafka service to fail, with the Java virtual machine (JVM) throwing an Out Of Memory exception during startup.

# Step 1 — Creating a User for Kafka


Since Kafka can handle requests over a network, it is a best practice to create a dedicated user for it. This minimizes damage to your Debian machine should the Kafka server be compromised. You will create the dedicated user kafka in this step.


Logged in as your non-root sudo user, create a user called kafka with the useradd command:


```
sudo useradd kafka -m


```


The -m flag ensures that a home directory will be created for the user. This home directory, /home/kafka, will act as your workspace directory for executing commands later on.


Set the password using passwd:


```
sudo passwd kafka


```


Enter the password you wish to use for this user.


Next, add the kafka user to the sudo group with the adduser command, so that it has the privileges required to install Kafka’s dependencies:


```
sudo adduser kafka sudo


```


Your kafka user is now ready. Log into this account using su:


```
su -l kafka


```


Now that you’ve created the Kafka-specific user, you can move on to downloading and extracting the Kafka binaries.


# Step 2 — Downloading and Extracting the Kafka Binaries


In this step, you will download and extract the Kafka binaries into dedicated folders in your kafka user’s home directory.


To start, create a directory in /home/kafka called Downloads to store your downloads:


```
mkdir ~/Downloads


```


Next, install curl using apt-get so that you’ll be able to download remote files:


```
sudo apt-get update && sudo apt-get install curl


```


When prompted, type Y to confirm the curl download.


Once curl is installed, use it to download the Kafka binaries:


```
curl "https://archive.apache.org/dist/kafka/2.1.1/kafka_2.11-2.1.1.tgz" -o ~/Downloads/kafka.tgz


```


Create a directory called kafka and change to this directory. This will be the base directory of the Kafka installation:


```
mkdir ~/kafka && cd ~/kafka


```


Extract the archive you downloaded using the tar command:


```
tar -xvzf ~/Downloads/kafka.tgz --strip 1


```


You specified the --strip 1 flag to ensure that the archive’s contents are extracted in ~/kafka/ itself and not in another directory inside of it, such as ~/kafka/kafka_2.12-2.1.1/.


Now that you’ve downloaded and extracted the binaries successfully, you can move on to configuring Kafka to allow for topic deletion.


# Step 3 — Configuring the Kafka Server


Kafka’s default behavior will not allow us to delete a topic, the category, group, or feed name to which messages can be published. To modify this, you will edit the configuration file.


Kafka’s configuration options are specified in server.properties. Open this file with nano or your favorite editor:


```
nano ~/kafka/config/server.properties


```


Let’s add a setting that will allow us to delete Kafka topics. Add the following highlighted line to the bottom of the file:


~/kafka/config/server.properties
```
...
group.initial.rebalance.delay.ms

delete.topic.enable = true

```


Save the file, and exit nano. Now that you’ve configured Kafka, you can create systemd unit files for running and enabling Kafka on startup.


# Step 4 — Creating Systemd Unit Files and Starting the Kafka Server


In this section, you will create systemd unit files for the Kafka service. This will help you perform common service actions such as starting, stopping, and restarting Kafka in a manner consistent with other Linux services.


ZooKeeper is a service that Kafka uses to manage its cluster state and configurations. It is commonly used in distributed systems as an integral component. In this tutorial, you will use Zookeeper to manage these aspects of Kafka. If you would like to know more about it, visit the official ZooKeeper docs.


First, create the unit file for zookeeper:


```
sudo nano /etc/systemd/system/zookeeper.service


```


Enter the following unit definition into the file:


/etc/systemd/system/zookeeper.service
```
[Unit]
Requires=network.target remote-fs.target
After=network.target remote-fs.target

[Service]
Type=simple
User=kafka
ExecStart=/home/kafka/kafka/bin/zookeeper-server-start.sh /home/kafka/kafka/config/zookeeper.properties
ExecStop=/home/kafka/kafka/bin/zookeeper-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target

```


The [Unit] section specifies that ZooKeeper requires networking and for the filesystem to be ready before it can start.


The [Service] section specifies that systemd should use the zookeeper-server-start.sh and zookeeper-server-stop.sh shell files for starting and stopping the service. It also specifies that ZooKeeper should be restarted automatically if it exits abnormally.


Next, create the systemd service file for kafka:


```
sudo nano /etc/systemd/system/kafka.service


```


Enter the following unit definition into the file:


/etc/systemd/system/kafka.service
```
[Unit]
Requires=zookeeper.service
After=zookeeper.service

[Service]
Type=simple
User=kafka
ExecStart=/bin/sh -c '/home/kafka/kafka/bin/kafka-server-start.sh /home/kafka/kafka/config/server.properties > /home/kafka/kafka/kafka.log 2>&1'
ExecStop=/home/kafka/kafka/bin/kafka-server-stop.sh
Restart=on-abnormal

[Install]
WantedBy=multi-user.target

```


The [Unit] section specifies that this unit file depends on zookeeper.service. This will ensure that zookeeper gets started automatically when the kafka service starts.


The [Service] section specifies that systemd should use the kafka-server-start.sh and kafka-server-stop.sh shell files for starting and stopping the service. It also specifies that Kafka should be restarted automatically if it exits abnormally.


Now that the units have been defined, start Kafka with the following command:


```
sudo systemctl start kafka


```


To ensure that the server has started successfully, check the journal logs for the kafka unit:


```
sudo journalctl -u kafka


```


You will see output similar to the following:


```
OutputMar 23 13:31:48 kafka systemd[1]: Started kafka.service.

```


You now have a Kafka server listening on port 9092, which is the default port for Kafka.


You have started the kafka service, but if you were to reboot your server, it would not yet be started automatically. To enable kafka on server boot, run:


```
sudo systemctl enable kafka


```


Now that you’ve started and enabled the services, it’s time to check the installation.


# Step 5 — Testing the Installation


Let’s publish and consume a Hello World message to make sure the Kafka server is behaving correctly. Publishing messages in Kafka requires:


- A producer, which enables the publication of records and data to topics.
- A consumer, which reads messages and data from topics.

First, create a topic named TutorialTopic by typing:


```
~/kafka/bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic TutorialTopic


```


You can create a producer from the command line using the kafka-console-producer.sh script. It expects the Kafka server’s hostname, port, and a topic name as arguments.


Publish the string Hello, World to the TutorialTopic topic by typing:


```
echo "Hello, World" | ~/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic TutorialTopic > /dev/null


```


The --broker-list flag determines the list of message brokers to send the message to, in this case localhost:9092. --topic designates the topic as TutorialTopic.


Next, you can create a Kafka consumer using the kafka-console-consumer.sh script. It expects the ZooKeeper server’s hostname and port and a topic name as arguments.


The following command consumes messages from TutorialTopic. Note the use of the --from-beginning flag, which allows the consumption of messages that were published before the consumer was started:


```
~/kafka/bin/kafka-console-consumer.sh --bootstrap-server `localhost:9092` --topic TutorialTopic --from-beginning


```


--bootstrap-server provides a list of ingresses into the Kafka cluster. In this case, you are using localhost:9092.


You will see Hello, World in your terminal:


```
OutputHello, World

```


The script will continue to run, waiting for more messages to be published to the topic. Feel free to open a new terminal and start a producer to publish a few more messages. You should be able to see them all in the consumer’s output. If you’d like to learn more about how to use Kafka, see the official Kafka documentation.


When you are done testing, press CTRL+C to stop the consumer script. Now that you have tested the installation, you can move on to installing KafkaT in order to better administer your Kafka cluster.


# Step 6 — Installing KafkaT (Optional)


KafkaT is a tool from Airbnb that makes it easier for you to view details about your Kafka cluster and perform certain administrative tasks from the command line. Because it is a Ruby gem, you will need Ruby to use it. You will also need the build-essential package to be able to build the other gems it depends on. Install them using apt:


```
sudo apt install ruby ruby-dev build-essential


```


You can now install KafkaT using the gem command:


```
sudo CFLAGS=-Wno-error=format-overflow gem install kafkat


```


The CFLAGS=-Wno-error=format-overflow option disables format overflow warnings and is required for the ZooKeeper gem, which is a dependency of KafkaT.


KafkaT uses .kafkatcfg as the configuration file to determine the installation and log directories of your Kafka server. It should also have an entry pointing KafkaT to your ZooKeeper instance.


Create a new file called .kafkatcfg:


```
nano ~/.kafkatcfg


```


Add the following lines to specify the required information about your Kafka server and Zookeeper instance:


~/.kafkatcfg
```
{
  "kafka_path": "~/kafka",
  "log_path": "/tmp/kafka-logs",
  "zk_path": "localhost:2181"
}

```


You are now ready to use KafkaT. For a start, here’s how you would use it to view details about all Kafka partitions:


```
kafkat partitions


```


You will see the following output:


```
OutputTopic                 Partition   Leader      Replicas        ISRs    
TutorialTopic         0             0         [0]             [0]
__consumer_offsets	  0		        0		  [0]			  [0]
...

```


This output shows TutorialTopic, as well as __consumer_offsets, an internal topic used by Kafka for storing client-related information. You can safely ignore lines starting with __consumer_offsets.


To learn more about KafkaT, refer to its GitHub repository.


Now that you have installed KafkaT, you can optionally set up Kafka on a cluster of Debian 10 servers to make a multi-node cluster.


# Step 7 — Setting Up a Multi-Node Cluster (Optional)


If you want to create a multi-broker cluster using more Debian 10 servers, repeat Step 1, Step 4, and Step 5 on each of the new machines. Additionally, make the following changes in the ~/kafka/config/server.properties file for each:


- 
Change the value of the broker.id property such that it is unique throughout the cluster. This property uniquely identifies each server in the cluster and can have any string as its value. For example, "server1", "server2", etc., would be useful as identifiers.

- 
Change the value of the zookeeper.connect property such that all nodes point to the same ZooKeeper instance. This property specifies the ZooKeeper instance’s address and follows the <HOSTNAME/IP_ADDRESS>:<PORT> format. For this tutorial, you would use your_first_server_IP:2181, replacing your_first_server_IP with the IP address of the Debian 10 server you already set up.


If you want to have multiple ZooKeeper instances for your cluster, the value of the zookeeper.connect property on each node should be an identical, comma-separated string listing the IP addresses and port numbers of all the ZooKeeper instances.



Note: If you have a firewall activated on the Debian 10 server with Zookeeper installed, make sure to open up port 2181 to allow for incoming requests from the other nodes in the cluster.

# Step 8 — Restricting the Kafka User


Now that all of the installations are done, you can remove the kafka user’s admin privileges. Before you do so, log out and log back in as any other non-root sudo user. If you are still running the same shell session you started this tutorial with, simply type exit.


Remove the kafka user from the sudo group:


```
sudo deluser kafka sudo


```


To further improve your Kafka server’s security, lock the kafka user’s password using the passwd command. This makes sure that nobody can directly log into the server using this account:


```
sudo passwd kafka -l


```


At this point, only root or a sudo user can log in as kafka by typing in the following command:


```
sudo su - kafka


```


In the future, if you want to unlock it, use passwd with the -u option:


```
sudo passwd kafka -u


```


You have now successfully restricted the kafka user’s admin privileges.


# Conclusion


You now have Apache Kafka running securely on your Debian server. You can make use of it in your projects by creating Kafka producers and consumers using Kafka clients, which are available for most programming languages. To learn more about Kafka, you can also consult the Apache Kafka documentation.


