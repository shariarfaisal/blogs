# How to Install MySQL 5 6 from Official Yum Repositories

```MySQL``` ```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.
The following DigitalOcean tutorial may be of immediate interest, as it outlines installing MySQL on a CentOS 7 server:

How To Install MySQL on CentOS 7


Submitted by: Morgan Tocker, MySQL Community Manager @ Oracle


## Introduction



In October 2013, the MySQL development team officially launched support for yum repositories.  This means that you can now ensure that you have the latest and greatest version of MySQL installed directly from the source!


In this guide we will install MySQL 5.6 on a fresh installation of Centos 6, and then apply a couple of extra touches to optimize configuration for DigitalOcean.


If you are not familiar with the changes to MySQL 5.6, I recommend getting started with What’s new in MySQL 5.6.  There’s a number of very useful features.


# Selecting a Server


For today I will be choosing the $40/month server, since it includes 4G of RAM and a 60GB SSD, but you are free to choose a different size depending on the demands of your application.  MySQL can install on systems with very small amounts of RAM, as well as scale up to over 100GB.  If you chose a different server size, please just make sure to adjust the configuration advice recommended below:


<img src=“https://assets.digitalocean.com/articles/centos_mysql/select-size.png”>


The official repositories support both 32-bit and 64-bit Enterprise Linux 6, as well as Fedora 18 & 19.  CentOS is the __C__ommunity __Ent__erprise __O__perating __S__ystem and works fine for this purpose:


<img src=“https://assets.digitalocean.com/articles/centos_mysql/centos-6.png”>


# Installing MySQL


The yum repository file needs to be downloaded from MySQL’s developer website.  Once installed, a simple yum update will make sure you are running on the latest point release of MySQL 5.6, including security updates.  Yum also ensures that any dependencies are also installed, which makes the installation process just a bit simpler.


To get started, go to http://dev.mysql.com/downloads/repo/, and click on download link for Red Hat Enterprise Linux 6 / Oracle Linux 6:


<img src=“https://assets.digitalocean.com/articles/centos_mysql/download-el6.png”>


Then copy the link under ‘No thanks, just start my download’:


<img src=“https://assets.digitalocean.com/articles/centos_mysql/no-thanks-link.png”>


Log into your server and then download this file. Below is an example URL-- you may want to double-check there is not a later version of the repo available:


```
wget http://dev.mysql.com/get/mysql-community-release-el6-3.noarch.rpm/from/http://repo.mysql.com/

```


Install the repository from the local file:


```
sudo yum localinstall mysql-community-release-el6-*.noarch.rpm

```


You now have the official repository installed on your server, but with no software installed yet.  The repository includes the MySQL Server, the MySQL Workbench administration tool and the ODBC driver.  Let’s install the MySQL Server:


```
sudo yum install mysql-community-server

```


Start MySQL:


```
sudo service mysqld start

```


Configure MySQL to start automatically on reboot:


```
sudo chkconfig mysqld on
chkconfig --list mysqld

```


That’s it.  You’re all set!


# Applying Finishing Touches



The MySQL development team put in a lot of effort to make sure that MySQL is better tuned out of the box, and 5.6 requires very little configuration.  Having said that, there will be a couple of things you will want to tweak, which can be added to /etc/my.cnf under the [mysqld] heading group:


```
sudo vim /etc/my.cnf

```


- 
It is recommended to set innodb_buffer_pool_size to 50-80% of system memory.  In the case of the server I selected, 50% of 4GB = 2GB.  This allows MySQL to cache more data (the default is only 128M) and can improve performance significantly.

- 
MySQL defaults to having very small transaction logs, which is a feature used to provide crash recovery. In development small log files can be useful to save space, but in production you will want to increase these to allow more writes to queue up in the background. The setting is innodb_log_file_size and recommended values range between 128M and 4G.

- 
When using SSDs, sequential IO is no faster than random IO. This means we can tell MySQL it can disable one of the optimizations it does and save a little CPU. The setting is innodb_flush_neighbors=0.

- 
By default, MySQL is exceptionally cautious to not lose any data even if there is a power loss.  This comes at a performance cost, and in many cloud environments users chose to allow a few seconds of data loss on power failure instead. The setting to change is innodb_flush_log_at_trx_commit=2.

- 
UTF-8 is a better default for storing international characters in MySQL.  You can change to it by setting character-set-server=utf8mb4 and collation-server=utf8mb4_general_ci.  Individual databases, tables and columns can still overwrite this if necessary.

- 
Many syadmins recommend setting the timezone of servers to GMT, which can be done with timezone=GMT.

- 
To ensure compatibility with older versions, MySQL defaults to allowing incorrect and out of range values.  I recommend that you enable the new stricter SQL_MODE options as long as your application(s) do not rely on this legacy MySQL behaviour.


*All together, these are the changes:


```
innodb_buffer_pool_size=2G
innodb_log_file_size=256M
innodb_flush_neighbors=0
innodb_flush_log_at_trx_commit=2

# Default to UTF-8 for text columns
character-set-server=utf8mb4
collation-server=utf8mb4_general_ci

# Set the default timezone to GMT.
# This is a common recommended practice, but can also remove this line
# If your date/time values appear incorrect.
timezone=GMT

# This configuration item is optional:
# It makes MySQL more strict and rejects invalid values.
# You may need to remove this line for legacy applications.
sql-mode="STRICT_TRANS_TABLES,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_AUTO_VALUE_ON_ZERO,NO_ENGINE_SUBSTITUTION,NO_ZERO_DATE,NO_ZERO_IN_DATE,ONLY_FULL_GROUP_BY"

```


# Further Reading



This concludes the basic installation of MySQL 5.6 using the official yum repos.  The MySQL team is interested in hearing your feedback.  If you encounter any issues or simply have a suggestion, please visit http://forums.mysql.com/list.php?11.


I also have a more advanced version of what to configure in MySQL 5.6 after installation available on my blog.  But rest assured, if you have followed the steps in this article, you are 95% of the way there!


