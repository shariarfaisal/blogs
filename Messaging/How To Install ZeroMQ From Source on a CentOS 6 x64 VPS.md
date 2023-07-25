# How To Install ZeroMQ From Source on a CentOS 6 x64 VPS

```Messaging``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.

## Introduction



There are numerous choices available for application level messaging implementations, each with its own special benefits over the others. One thing is surely certain, however, and that is for many situations a full featured solution and protocol implementation (such as Advanced Message Queuing Protocol) can be an over-kill.


For these situations, a lean and truly high-performant messaging library which allows you to craft the exact system you need has tons of benefits. One of these libraries - and perhaps the go-to solution of its kind - is ZeroMQ.


ZeroMQ is an asynchronous messaging system toolkit (i.e. a library). It doesn’t follow conventions, nor does it set itself defining a new protocol. Despite the heavyweight champions of the world, this superbly efficient messaging component focuses on handling tasks as efficiently and as powerfully as they can be handled without being an extra abundant layer when it’s not needed.


In this DigitalOcean article, we are going to learn about setting up the latest version of ZeroMQ from source, which should allow you to start implementing efficient lightweight messaging into your application stack.


# ZeroMQ



If you have past experience with other messaging systems such as RabbitMQ, it might be a little bit challenging to understand the position of ZeroMQ due to some wide range of irrelevant comparisons all over the internet. These two are completely different tools aimed to solve different kinds of problems.


ZeroMQ, as we have mentioned at the beginning, is a library (i.e. toolkit). Although it might appear as a lower level solution compared to others, it brings everything necessary to quickly implement custom messaging solutions with its ease of use and large range of different programming language bindings.


What this translates to is the need of downloading and setting up ZeroMQ library, followed by the additional files for your programming language of choice to get started building a ZeroMQ application. In our tutorial, to get the latest versions and have stable installation, we are going to install ZeroMQ from source in a few simple steps.


# Installing From Source



Building applications on Unix systems can appear scary to some, but it is generally easier than you think. Although it should be noted that there are other tools to achieve the same task, we will be using GNU make to build ZeroMQ. GNU make is one of the most widespread utilities as it’s been built into Unix systems since its introduction in the late 70s.


## Why Build From Source?



Many system administrators choose to build software from source as it can help to solve problems caused by deb/rpm (pre-made) packages. It also allows you to customize the installation process, to have multiple versions of the same application on a single system, and to use the desired one without worrying about pre-built binaries (compiled files).


# Taking Care of ZeroMQ Dependencies and Preparing the System



With the more recent builds of both operating systems and ZeroMQ application itself, installation process became more and more simpler. Nonetheless, we need to make a few preparations before starting the build process.


## Update the Default Operating System Tools



To ensure that we have the latest version of default system tools, let’s begin with running a base update on our system:


```
yum -y update

```


## Enable Additional Application Repository (EPEL)



To be able to download certain tools necessary for building and using ZeroMQ  and many others, we need to enable EPEL: Extra Packages for Enterprise Linux. This will allow us to download and install many packages that are not available by default with ease.


Run the following commands to enable EPEL:


```
# If you are on a 64-bit CentOS / RHEL based system: 
wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm

# If you are on a 32-bit CentOS / RHEL based system:
wget http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm

```


## Download Additional Tools for Building From Source



ZeroMQ’s build process - as previously mentioned - requires a few additional tools. Upon enabling EPEL, we can easily download them using the default package manager YUM.


Run the following to get the tools:


```
yum install -y uuid-devel
yum install -y pkgconfig
yum install -y libtool
yum install -y gcc-c++

```


# Downloading and Installing ZeroMQ From Source



After covering all necessary applications, we can begin the installation process for ZeroMQ.


The latest available version for ZeroMQ is 4.0.3 released on 2013-11-24.


Let’s begin with downloading the application source:


```
wget http://download.zeromq.org/zeromq-4.0.3.tar.gz

```


Extract the contents of the tar archive and enter the directory:


```
tar xzvf zeromq-4.0.3.tar.gz
cd zeromq-4.0.3

```


Configure the application build procedure:


```
./configure

```


Build the program using the Makefile:


```
make

```


Install the application:


```
make install

```


Update the system library cache:


```
echo /usr/local/lib > /etc/ld.so.conf.d/local.conf
ldconfig    

```


And that’s it! You now have ZeroMQ messaging library set up on your system, which can be used to to create messaging applications.


# Getting ZeroMQ Language Bindings



## Python Bindings: PyZMQ



It is possible to download and build Python bindings for ZeroMQ (PyZMQ) using the Python package manager pip.


Download and install PyZQM with pip:


```
pip install pyzmq

```


If you would like to learn about setting up Python 2.7.x and 3.x on CentOS along with common Python tools including pip, check out our article How To Set Up Python on CentOS.


## Ruby Bindings: zmq Gem



ZeroMQ Ruby bindings are available as a Ruby Gem called zmq.


For default ZeroMQ installations, run the following to get zmq:


```
gem install zmq

```


For non-default ZeroMQ installations, use the following:


```
gem install zmq -- --with-zmq-dir=/path/to/zeromq

```


## Other Programming Language Bindings for ZeroMQ



For all other ZeroMQ bindings - including but not limited to PHP, C#, Erlang, Haskell, Java, Lua and more - visit ZeroMQ Community Wiki.


<div class=“author”>Submitted by: <a
href=“https://twitter.com/ostezer”>O.S. Tezer</a></div>


