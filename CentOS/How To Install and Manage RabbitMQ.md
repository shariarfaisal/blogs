# How To Install and Manage RabbitMQ

```Ubuntu``` ```Messaging``` ```CentOS``` ```Debian```

## Introduction



Putting things off for a while instead of immediately doing them can be considered lazy. In fact, most of the time it probably is. However, there are times when it’s absolutely the right thing to do. Occasionally, one needs to delay a time-consuming job for a while; it needs to be queued for future execution so that something more important can be dealt with. For this to happen, you need a broker: someone who will accept messages (e.g. jobs, tasks) from various senders (i.e. a web application), queue them up, and distribute them to the relevant parties (i.e. workers) to make use of them - all asynchronously and on demand.


In this DigitalOcean article, we aim to introduce you to the RabbitMQ project: an open-source message-broker application stack which implements the Advanced Message Queuing Protocol (AMQP) to handle the entirety of the scenario we explained above.


# Messaging, Message Brokers and Queues



Messaging is a way of exchanging certain data between processes, applications, and servers (virtual and physical). These messages exchanged, helping with certain engineering needs, can consist of anything from plain text messages to blobs of binary data serving to address different needs. For this to work, an interface managed by a third party program (a middleware) is needed… welcome Message Brokers.


Message Brokers are usually application stacks with dedicated pieces covering the each stage of the exchange setup. From accepting a message to queuing it and delivering it to the requesting party, brokers handle the duty which would normally be much more cumbersome with non-dedicated solutions or simple hacks such as using a database, cron jobs, etc. They simply work by dealing with queues which technically constitute infinite buffers, to put messages and pop-and-deliver them later on to be processed either automatically or by polling.


## Why use them?



These message brooking solutions act like a middleman for various services (e.g. your web application). They can be used to greatly reduce loads and delivery times by web application servers since tasks, which would normally take quite bit of time to process, can be delegated for a third party whose sole job is to perform them (e.g. workers). They also come in handy when a more “guaranteed” persistence is needed to pass information along from one place to another.


## When to use them?



All put together, the core functionality explained expands to cover a multitude of areas, including-but-not-limited-to:


- 
Allowing web servers to respond to requests quickly instead of being forced to perform resource-heavy procedures on the spot

- 
Distributing a message to multiple recipients for consumption (e.g. processing)

- 
Letting offline parties (i.e. a disconnected user) fetch data at a later time instead of having it lost permanently

- 
Introducing fully asynchronous functionality to the backend systems

- 
Ordering and prioritising tasks

- 
Balancing loads between workers

- 
Greatly increase reliability and uptime of your application

- 
and much more


# RabbitMQ



RabbitMQ is one of the more popular message broker solutions in the market, offered with an open-source license (Mozilla Public License v1.1) as an implementation of Advanced Message Queuing Protocol. Developed using the Erlang language, it is actually relatively easy to use and get started. It was first published in early 2007 and has since seen an active development with its latest release being version 3.2.2 (December 2013).


## How does it work?



RabbitMQ works by offering an interface, connecting message senders (Publishers) with receivers (Consumers) through an exchange (Broker) which distributes the data to relevant lists (Message Queues).


```
APPLICATION       EXCHANGE        TASK LIST        WORKER
   [DATA] -------> [DATA] ---> [D]+[D][D][D] --->  [DATA]
 Publisher        EXCHANGE          Queue         Consumer 

```


## How is it different than the others?



RabbitMQ, unlike some other solutions, is a fully-fledged application stack (i.e. a message broker). It gives you all the tools you need to work with, instead of acting like a framework for you to implement your own. Being extremely popular, it is really easy to get going using RabbitMQ and to find answers to your questions online.


# Advanced Message Queuing Protocol (AMQP) in Brief



AMQP is a widely accepted open-source standard for distributing and transferring messages from a source to a destination. As a protocol and standard, it sets a common ground for various applications and message broker middlewares to interoperate without encountering issues caused by individually set design decisions.


# Installing RabbitMQ



RabbitMQ packages are distributed both with CentOS / RHEL & Ubuntu / Debian based systems. However, they are - like with most applications - outdated. The recommended way to get RabbitMQ on your system is therefore to download the package online and install manually.


Note: We will be performing our installations and perform the actions listed here on a fresh and newly created VPS due to various reasons. If you are actively serving clients and might have modified your system, in order to not break anything working and to not to run into issues, you are highly advised to try the following instructions on a new system.


## Installing on CentOS 6 / RHEL Based Systems



Before installing RabbitMQ, we need to get its main dependencies such as Erlang. However, first and foremost we should update our system and its default applications.


Run the following to update our droplet:


```
yum -y update

```


And let’s use the below commands to get Erlang on our system:


```
# Add and enable relevant application repositories:
# Note: We are also enabling third party remi package repositories.
wget http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
wget http://rpms.famillecollet.com/enterprise/remi-release-6.rpm
sudo rpm -Uvh remi-release-6*.rpm epel-release-6*.rpm

# Finally, download and install Erlang:
yum install -y erlang

```


Once we have Erlang, we can continue with installing RabbitMQ:


```
# Download the latest RabbitMQ package using wget:
wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.2.2/rabbitmq-server-3.2.2-1.noarch.rpm

# Add the necessary keys for verification:
rpm --import http://www.rabbitmq.com/rabbitmq-signing-key-public.asc

# Install the .RPM package using YUM:
yum install rabbitmq-server-3.2.2-1.noarch.rpm

```


## Installing on Ubuntu 13 / Debian 7 Based Systems



The process for downloading and installing RabbitMQ on Ubuntu and Debian will be similar to CentOS due to our desire of having a more recent version.


Let’s begin with updating our system’s default application toolset:


```
apt-get    update 
apt-get -y upgrade

```


Enable RabbitMQ application repository:


```
echo "deb http://www.rabbitmq.com/debian/ testing main" >> /etc/apt/sources.list

```


Add the verification key for the package:


```
curl http://www.rabbitmq.com/rabbitmq-signing-key-public.asc | sudo apt-key add -

```


Update the sources with our new addition from above:


```
apt-get update

```


And finally, download and install RabbitMQ:


```
sudo apt-get install rabbitmq-server

```


In order to manage the maximum amount of connections upon launch, open up and edit the following configuration file using nano:


```
sudo nano /etc/default/rabbitmq-server

```


Uncomment the limit line (i.e. remove #) before saving and exit by pressing CTRL+X followed with Y.


# Managing RabbitMQ



As we have mentioned before, RabbitMQ is very simple to get started with. Using the instructions below for your system, you can quickly manage its process and have it running at the system start-up (i.e. boot).


## Enabling the Management Console



RabbitMQ Management Console is one of the available plugins that lets you monitor the [RabbitMQ] server process through a web-based graphical user interface (GUI).


Using this console you can:


- 
Manage exchanges, queues, bindings, users

- 
Monitor queues, message rates, connections

- 
Send and receive messages

- 
Monitor Erlang processes, memory usage

- 
And much more


To enable RabbitMQ Management Console, run the following:


```
sudo rabbitmq-plugins enable rabbitmq_management

```


Once you’ve enabled the console, it can be accessed using your favourite web browser by visiting: http://[your droplet's IP]:15672/.


The default username and password are both set “guest” for the log in.


Note: If you enable this console after running the service, you will need to restart it for the changes to come into effect. See the relevant management section below for your operating system to be able to do it.


## Managing on CentOS / RHEL Based Systems



Upon installing the application, RabbitMQ is not set to start at system boot by default.


To have RabbitMQ start as a daemon by default, run the following:


```
chkconfig rabbitmq-server on

```


To start, stop, restart and check the application status, use the following:


```
# To start the service:
/sbin/service rabbitmq-server start

# To stop the service:
/sbin/service rabbitmq-server stop

# To restart the service:
/sbin/service rabbitmq-server restart
    
# To check the status:
/sbin/service rabbitmq-server status

```


## Managing on Ubuntu / Debian Based Systems



To start, stop, restart and check the application status on Ubuntu and Debian, use the following:


```
# To start the service:
service rabbitmq-server start

# To stop the service:
service rabbitmq-server stop

# To restart the service:
service rabbitmq-server restart
    
# To check the status:
service rabbitmq-server status

```


And that’s it! You now have your own message queue working on your virtual server.


# Configuring RabbitMQ



RabbitMQ by default runs with its standard configuration. In general, it does not require much tempering with for most needs as long as everything runs smoothly.


To learn about configuring it for custom needs, check out its documentation for Configuration.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


