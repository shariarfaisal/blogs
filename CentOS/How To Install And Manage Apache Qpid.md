# How To Install And Manage Apache Qpid

```Ubuntu``` ```Messaging``` ```CentOS``` ```Debian```

## Introduction



When it comes to sending and receiving messages between applications and processes, there are many solutions you can choose from. They all have their pros and cons, therefore the best thing to do is to cross-check your requirements and match them against various available solutions.


Apache Qpid is one of the open-source messaging systems that implements the Advanced Message Queuing Protocol (AMQP) to help you solve your needs of advanced messaging between different elements of your deployment stack.


# Messaging, Message Brokers and Queues



Messaging is a way of exchanging certain data between processes, applications, and servers (virtual and physical). These messages exchanged, helping with certain engineering needs, can consist of anything from plain text messages to blobs of binary data serving to address different needs. For this to work, an interface managed by a third party program (a middleware) is needed - welcome Message Brokers.


Message Brokers are usually application stacks with dedicated pieces covering each stage of the exchange setup. From accepting a message to queuing it and delivering it to the requesting party, brokers handle the duty which would normally be much harder or cumbersome to do with non-dedicated solutions or simple hacks such as using a database, cron jobs, etc. They simply work by dealing with queues which technically constitute infinite buffers, to put messages and pop-and-deliver them later on to be processed either automatically or by polling.


## Why use them?



These message brooking solutions act like a middleman for various services (e.g. your web application). They can be used to greatly reduce loads and delivery times by web application servers since tasks, which would normally take quite bit of time to process, can be delegated for a third party whose sole job is to perform them (i.e. workers). They also come in handy when a more “guaranteed” persistence is needed to pass information along from one place to another.


## When to use them?



All put together, the core functionality explained expands to cover a multitude of areas, including-but-not-limited-to:


- 
Allowing web servers to respond to requests quickly instead of being forced to perform resource-heavy procedures on the spot

- 
Distributing a message to multiple recipients for consumption (i.e. processing)

- 
Letting offline parties (e.g. a disconnected user) fetch data at a later time instead of having it lost permanently

- 
Introducing fully asynchronous functionality to the backend systems

- 
Ordering and prioritising tasks

- 
Balancing loads between workers

- 
Greatly increase reliability and uptime of your application

- 
and much more.


# Apache Qpid



Apache Software Foundation has several solutions when it comes to messaging and one of them is Apache Qpid: The foundation’s implementation of AMQP. Unlike some more basic applications which are aimed at helping developers to craft their own solutions, Qpid, similar to RabbitMQ, offers a nice toolset capable of queuing, security and transaction management, clustering, persistence via a pluggable layer and more. Its API, by default, supports multiple programming languages and it comes with both C++ (for Perl, Python, Ruby, .NET etc.) and Java (JMS API) brokers. Alongside RabbitMQ, Qpid is probably the most popular choice.


## How is it different than the others?



Message brokers which are fully fledged differ only slightly on the surface. However, a deeper look into internals reveal the truth behind how things work. The following are the features which make Apache Qpid stand out compared to others:


- 
Client failover detection and automatic healing by connecting to a different broker

- 
Easy clustering by replicating queues across different servers

- 
Error handling by default in clusters

- 
Easy persistence via a pluggable architecture to offer high-availability

- 
and more.


# Advanced Message Queuing Protocol (AMQP) in Brief



AMQP is a widely accepted open-source standard for distributing and transferring messages from a source to a destination. As a protocol and standard, it sets a common ground for various applications and message broker middlewares to interoperate without encountering issues caused by individually set design decisions.


# Installing Apache Qpid



Getting started with Apache Qpid means installing two different sets of tools:


- 
An implementation of Qpid Broker depending on programming language of your choice (e.g. C++ broker for Python or Java Broker for Java)

- 
Qpid Client libraries (e.g. Qpid Python)


Note: We will be performing our installations and the actions listed here on a fresh and newly created droplet for various reasons. If you are actively serving clients and might have modified your system, to not to break anything working and to not to run in to issues, you are highly advised to try the following instructions on a new system.


## Installing on CentOS 6 / RHEL Based Systems



Let’s update our droplet:


```
yum -y update

```


And then let’s run the following to get Qpid C++ Server and its tools (including Python bindings):


```
yum install -y qpid-cpp-server qpid-tools    

```


If you require, continue installing Qpid’s language bindings for others such as Ruby:


```
yum install -y ruby-qpid

```


## Installing on Ubuntu 13 / Debian 7 Based Systems



The process for downloading and installing Apache Qpid on Ubuntu and Debian will be similar to CentOS.


Let’s begin with updating our system’s default application toolset:


```
apt-get    update 
apt-get -y upgrade

```


And then let’s run the following to get Qpid C++ Server and its tools:


```
apt-get install -y qpidd qpid-tools
apt-get install -y libqpidmessaging2-dev python-qpid ruby-qpid

```


Note: During the installation process you will be prompted to enter a password of your choice for the Qpid daemon administrator.


# Managing Apache Qpid



## Managing on CentOS / RHEL Based Systems



To start, stop, restart, and check the application status, use the following:


```
# To start the service:
/sbin/service qpidd start

# To stop the service:
/sbin/service qpidd stop

# To restart the service:
/sbin/service qpidd restart
    
# To check the status:
/sbin/service qpidd status

# To force reload:
/sbin/service qpidd force-reload

```


## Managing on Ubuntu / Debian Based Systems



To start, stop, restart, and check the application status on Ubuntu and Debian, use the following:


```
# To start the service:
service qpidd start

# To stop the service:
service qpidd stop

# To restart the service:
service qpidd restart
    
# To check the status:
service qpidd status

# To force reload:
service qpidd force-reload

```


And that’s it! You now have your own Apache Qpid message broker working on your droplet.


To learn more about Qpid and its vast array of configuration options, check out its documentation for C++ Implementation and Java Implementation.


# Working with Apache Qpid



Following our installation of Qpid along with its Python language bindings, let’s look into a simple Qpid example to understand basics of working with it.


Create a (sample) hello_world.py file using nano:


```
nano hello_world.py

```


Paste the below self-explanatory module:


```
# Import the modules we need
from qpid.messaging import *

broker     = "localhost:5672" 
address    = "amq.topic" 
connection = Connection(broker)

try:
    connection.open()

    # Define the session
    session = connection.session()

    # Define a sender *and* a receiver
    sender   = session.sender(address)
    receiver = session.receiver(address)

    # Send a simple "Hello world!" message to the queue
    sender.send(Message("Hello world!"));

    # Fetch the next message in the queue
    message = receiver.fetch()

    # Output the message
    print message.content

    # Check with the server
    session.acknowledge()

except MessagingError, err:
    print err

finally:
    connection.close()

```


Press CTRL+X and confirm with Y to save and exit.


When you run the above script, you should see our message (i.e. Hello world!) as the output now.


```
python hello_world.py
# Hello world!

```


If you run into an issue, be sure that qpid is running. You can start it using the commands above.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


