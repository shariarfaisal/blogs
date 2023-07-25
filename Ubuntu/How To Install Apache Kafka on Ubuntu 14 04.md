# How To Install Apache Kafka on Ubuntu 14 04

```Apache``` ```Ubuntu``` ```Java``` ```Messaging```

## Introduction


Apache Kafka is a popular distributed message broker designed to handle large volumes of real-time data efficiently. A Kafka cluster is not only highly scalable and fault-tolerant, but it also has a much higher throughput compared to other message brokers such as ActiveMQ and RabbitMQ. Though it is generally used as a pub/sub messaging system, a lot of organizations also use it for log aggregation because it offers persistent storage for published messages.


In this tutorial, you will learn how to install and use Apache Kafka 0.8.2.1 on Ubuntu 14.04.


# Prerequisites


To follow along, you will need:


- Ubuntu 14.04 Droplet
- At least 4GB of swap space

# Step 1 — Create a User for Kafka


As Kafka can handle requests over a network, you should create a dedicated user for it. This minimizes damage to your Ubuntu machine should the Kafka server be comprised.



Note: After setting up Apache Kafka, it is recommended that you create a different non-root user to perform other tasks on this server.

As root, create a user called kafka using the useradd command:


```
useradd kafka -m


```


Set its password using passwd:


```
passwd kafka


```


Add it to the sudo group so that it has the privileges required to install Kafka’s dependencies. This can be done using the adduser command:


```
adduser kafka sudo


```


Your Kafka user is now ready. Log into it using su:


```
su - kafka


```


# Step 2 — Install Java


Before installing additional packages, update the list of available packages so you are installing the latest versions available in the repository:


```
sudo apt-get update


```


As Apache Kafka needs a Java runtime environment, use apt-get to install the default-jre package:


```
sudo apt-get install default-jre


```


# Step 3 — Install ZooKeeper


Apache ZooKeeper is an open source service built to coordinate and synchronize configuration information of nodes that belong to a distributed system. A Kafka cluster depends on ZooKeeper to perform—among other things—operations such as detecting failed nodes and electing leaders.


Since the ZooKeeper package is available in Ubuntu’s default repositories, install it using apt-get.


```
sudo apt-get install zookeeperd


```


After the installation completes, ZooKeeper will be started as a daemon automatically. By default, it will listen on port 2181.


To make sure that it is working, connect to it via Telnet:


```
telnet localhost 2181


```


At the Telnet prompt, type in ruok and press ENTER.


If everything’s fine, ZooKeeper will say imok and end the Telnet session.


# Step 4 — Download and Extract Kafka Binaries


Now that Java and ZooKeeper are installed, it is time to download and extract Kafka.


To start, create a directory called Downloads to store all your downloads.


```
mkdir -p ~/Downloads


```


Use wget to download the Kafka binaries.


```
wget "http://mirror.cc.columbia.edu/pub/software/apache/kafka/0.8.2.1/kafka_2.11-0.8.2.1.tgz" -O ~/Downloads/kafka.tgz


```


Create a directory called kafka and change to this directory. This will be the base directory of the Kafka installation.


```
mkdir -p ~/kafka && cd ~/kafka


```


Extract the archive you downloaded using the tar command.


```
tar -xvzf ~/Downloads/kafka.tgz --strip 1


```


# Step 5 — Configure the Kafka Server


The next step is to configure the Kakfa server.


Open server.properties using vi:


```
vi ~/kafka/config/server.properties


```


By default, Kafka doesn’t allow you to delete topics. To be able to delete topics, add the following line at the end of the file:


~/kafka/config/server.properties
```
delete.topic.enable = true

```


Save the file, and exit vi.


# Step 6 — Start the Kafka Server


Run the kafka-server-start.sh script using nohup to start the Kafka server (also called Kafka broker) as a background process that is independent of your shell session.


```
nohup ~/kafka/bin/kafka-server-start.sh ~/kafka/config/server.properties > ~/kafka/kafka.log 2>&1 &


```


Wait for a few seconds for it to start. You can be sure that the server has started successfully when you see the following messages in ~/kafka/kafka.log:


excerpt from ~/kafka/kafka.log
```

...

[2015-07-29 06:02:41,736] INFO New leader is 0 (kafka.server.ZookeeperLeaderElector$LeaderChangeListener)
[2015-07-29 06:02:41,776] INFO [Kafka Server 0], started (kafka.server.KafkaServer)

```


You now have a Kafka server which is listening on port 9092.


# Step 7 — Test the Installation


Let us now publish and consume a “Hello World” message to make sure that the Kafka server is behaving correctly.


To publish messages, you should create a Kafka producer. You can easily create one from the command line using the kafka-console-producer.sh script. It expects the Kafka server’s hostname and port, along with a topic name as its arguments.


Publish the string “Hello, World” to a topic called TutorialTopic by typing in the following:


```
echo "Hello, World" | ~/kafka/bin/kafka-console-producer.sh --broker-list localhost:9092 --topic TutorialTopic > /dev/null


```


As the topic doesn’t exist, Kafka will create it automatically.


To consume messages, you can create a Kafka consumer using the kafka-console-consumer.sh script. It expects the ZooKeeper server’s hostname and port, along with a topic name as its arguments.


The following command consumes messages from the topic we published to. Note the use of the --from-beginning flag, which is present because we want to consume a message that was published before the consumer was started.


```
~/kafka/bin/kafka-console-consumer.sh --zookeeper localhost:2181 --topic TutorialTopic --from-beginning


```


If there are no configuration issues, you should see Hello, World in the output now.


The script will continue to run, waiting for more messages to be published to the topic. Feel free to open a new terminal and start a producer to publish a few more messages. You should be able to see them all in the consumer’s output instantly.


When you are done testing, press CTRL+C to stop the consumer script.


# Step 8 — Install KafkaT (Optional)


KafkaT is a handy little tool from Airbnb which makes it easier for you to view details about your Kafka cluster and also perform a few administrative tasks from the command line. As it is a Ruby gem, you will need Ruby to use it. You will also need the build-essential package to be able to build the other gems it depends on. Install them using apt-get:


```
sudo apt-get install ruby ruby-dev build-essential


```


You can now install KafkaT using the gem command:


```
sudo gem install kafkat --source https://rubygems.org --no-ri --no-rdoc


```


Use vi to create a new file called .kafkatcfg.


```
vi ~/.kafkatcfg


```


This is a configuration file which KafkaT uses to determine the installation and log directories of your Kafka server. It should also point KafkaT to your ZooKeeper instance. Accordingly, add the following lines to it:


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


You should see the following output:


output of kafkat partitions
```
Topic		    Partition	Leader		Replicas		ISRs	
TutorialTopic	0		      0		      [0]			[0]

```


To learn more about KafkaT, refer to its GitHub repository.


# Step 9 — Set Up a Multi-Node Cluster (Optional)


If you want to create a multi-broker cluster using more Ubuntu 14.04 machines, you should repeat Step 1, Step 3, Step 4 and Step 5 on each of the new machines. Additionally, you should make the following changes in the server.properties file in each of them:


- the value of the broker.id property should be changed such that it is unique throughout the cluster
- the value of the zookeeper.connect property should be changed such that all nodes point to the same ZooKeeper instance

If you want to have multiple ZooKeeper instances for your cluster, the value of the zookeeper.connect property on each node should be an identical, comma-separated string listing the IP addresses and port numbers of all the ZooKeeper instances.


# Step 10 — Restrict the Kafka User


Now that all installations are done, you can remove the kafka user’s admin privileges. Before you do so, log out and log back in as any other non-root sudo user. If you are still running the same shell session you started this tutorial with, simply type exit.


To remove the kafka user’s admin privileges, remove it from the sudo group.


```
sudo deluser kafka sudo


```


To further improve your Kafka server’s security, lock the kafka user’s password using the passwd command. This makes sure that nobody can directly log into it.


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


# Conclusion


You now have a secure Apache Kafka running on your Ubuntu server. You can easily make use of it in your projects by creating Kafka producers and consumers using Kafka clients which are available for most programming languages. To learn more about Kafka, do go through its documentation.


