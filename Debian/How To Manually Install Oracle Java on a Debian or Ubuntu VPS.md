# How To Manually Install Oracle Java on a Debian or Ubuntu VPS

```Ubuntu``` ```Java``` ```Debian```


Status: Deprecated
This article is deprecated and no longer maintained.
Reason
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.
See Instead
This article may still be useful as a reference, but may not follow best practices or work on this or other Ubuntu releases. We strongly recommend using a recent article written for the version of Ubuntu you are using.

How To Install Java with Apt-Get on Ubuntu 16.04
How To Install Java with Apt-Get on Debian 8

If you are currently operating a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:

How to upgrade from Ubuntu 12.04 to Ubuntu 14.04.
How to upgrade from Ubuntu 14.04 to Ubuntu 16.04
How to migrate server data to a supported version


## Introduction


Java is a programming technology originally developed by Sun Microsystems and later acquired by Oracle. Oracle Java is a proprietary implementation for Java that is free to download and use for commercial use, but not to redistribute, therefore it is not included in a officially maintained repository.


There are many reasons why you would want to install Oracle Java over OpenJDK. In this tutorial, we will not discuss the differences between the above mentioned implementations.


# Assumptions


This tutorial assumes that you have an account with DigitalOcean, as well as a Droplet running Debian 7 or Ubuntu 12.04 or above. You will need root privileges (via sudo) to complete the tutorial.


You will need to know whether you are running a 32 bit or a 64 bit OS:


```
uname -m

```


- 
x86_64: 64 bit kernel

- 
i686: 32 bit kernel


# Downloading Oracle Java JDK


Using your web browser, go to the Oracle Java SE (Standard Edition) website and decide which version you want to install:


- 
JDK: Java Development Kit. Includes a complete JRE plus tools for developing, debugging, and monitoring Java applications.

- 
Server JRE: Java Runtime Environment. For deploying Java applications on servers. Includes tools for JVM monitoring and tools commonly required for server applications.


In this tutorial we will be installing the JDK Java SE Development Kit 8 x64 bits. Accept the license and copy the download link into your clipboard. Remember to choose the right tar.gz (64 or 32 bits). Use wget to download the archive into your server:


```
    wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u5-b13/jdk-8u5-linux-x64.tar.gz

```


Oracle does not allow downloads without accepting their license, therefore we needed to modify the header of our request. Alternatively, you can just download the compressed file using your browser and manually upload it using a SFTP/FTP client.


Always get the latest version from Oracle’s website and modify the commands from this tutorial accordingly to your downloaded file.


# Installing Oracle JDK


In this section, you will need sudo privileges:


```
    sudo su

```


The /opt directory is reserved for all the software and add-on packages that are not part of the default installation. Create a directory for your JDK installation:


```
    mkdir /opt/jdk

```


and extract java into the /opt/jdk directory:


```
    tar -zxf jdk-8u5-linux-x64.tar.gz -C /opt/jdk

```


Verify that the file has been extracted into the /opt/jdk directory.


```
    ls /opt/jdk

```


# Setting Oracle JDK as the default JVM


In our case, the java executable is located under /opt/jdk/jdk1.8.0_05/bin/java . To set it as the default JVM in your machine run:


```
    update-alternatives --install /usr/bin/java java /opt/jdk/jdk1.8.0_05/bin/java 100

```


and


```
    update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk1.8.0_05/bin/javac 100

```


# Verify your installation


Verify that java has been successfully configured by running:


```
    update-alternatives --display java

```


and


```
    update-alternatives --display javac

```


The output should look like this:


```
    java - auto mode
    link currently points to /opt/jdk/jdk1.8.0_05/bin/java
    /opt/jdk/jdk1.8.0_05/bin/java - priority 100
    Current 'best' version is '/opt/jdk/jdk1.8.0_05/bin/java'.

    javac - auto mode
    link currently points to /opt/jdk/jdk1.8.0_05/bin/javac
    /opt/jdk/jdk1.8.0_05/bin/javac - priority 100
    Current 'best' version is '/opt/jdk/jdk1.8.0_05/bin/javac'.

```


Another easy way to check your installation is:


```
    java -version

```


The output should look like this:


```
    java version "1.8.0_05"
    Java(TM) SE Runtime Environment (build 1.8.0_05-b13)
    Java HotSpot(TM) 64-Bit Server VM (build 25.5-b02, mixed mode)

```


# (Optional) Updating Java


To update Java, simply download an updated version from Oracle’s website and extract it under the /opt/jdk directory, then set it up as the default JVM with a higher priority number (in this case 110):


```
    update-alternatives --install /usr/bin/java java /opt/jdk/jdk.new.version/bin/java 110
    update-alternatives --install /usr/bin/javac javac /opt/jdk/jdk.new.version/bin/javac 110

```


You can keep the old version or delete it:


```
    update-alternatives --remove java /opt/jdk/jdk.old.version/bin/java
    update-alternatives --remove javac /opt/jdk/jdk.old.version/bin/javac
    
    rm -rf /opt/jdk/jdk.old.version

```


The installation procedure documented above is confirmed to work on a Debian server, but can also be applied to an Ubuntu server. If you encounter any problem after following all the steps, please post a comment below.


