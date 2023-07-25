# How To Install a Fresh Percona Server or Replace MySQL

```MySQL``` ```Ubuntu``` ```CentOS``` ```Debian```

## Introduction


Percona Server is a drop-in replacement fork of the MySQL project. Percona aims to provide better performance, consistency, and scalability on all hardware. This tutorial will guide you through replacing a current MySQL or MariaDB installation with the latest Percona Server version, or installing Percona Server from scratch on a new Droplet.


# Benefits


Percona Server has a number of benefits over a basic MySQL installation:


- 
XtraDB: One of the key benefits of switching to Percona Server is XtraDB, a backwards compatible fork of the InnoDB engine with great improvements in performance and efficiency, allowing you to get better query throughput from your current hardware. As it is built on top of InnoDB, your current InnoDB tables will be loaded transparently through XtraDB without any migration processes.

- 
Stability and Consistency: Percona Server performs more uniformly under load, meaning your applications will be less liable to intermittent downtime or slowdowns.

- 
Metrics: Percona Server comes with a number of additional performance metrics built in, allowing you to discover exactly which users, tables, indexes, or queries are slowing you down. Getting great performance from your server becomes more scientific and less dependent on guesswork.

- 
PAM Authentication: Usually a feature reserved for MySQL Enterprise Edition, with Percona Server you can tie in your authentication scheme to your database access.

- 
Compatibility: As Percona server is a drop-in replacement for MySQL, you also get all of the usual benefits of MySQL’s widespread popularity and large community of users. This means any application designed for MySQL can safely use Percona Server without any changes.


# Prerequisites


- 
Debian, Ubuntu, or CentOS cloud server: Other distributions are not currently supported. Only CentOS versions 5 & 6 are supported; CentOS 7 is not supported at the time of writing.

- 
Either a fresh Droplet or an up-to-date MySQL/MariaDB installation: Percona Server can be installed either from scratch on a new Droplet or as a replacement for a current MySQL/MariaDB installation. Depending on which of these situations you have, some steps in this article may be relevant to only one case and will be marked with (New Only) or (Replacement Only).  Any unmarked sections or paragraphs should be used for both cases.

- 
Root access: All commands within this article should be executed as root.

- 
Memory: For a default installation you will need at least a 1GB Droplet, otherwise you may get installation failures due to insufficient memory for buffer pool allocation. If you have a 512MB Droplet with swap space allocated, you may also get a successful installation, albeit with subpar performance.

- 
Data Backup (Replacement Only): Before making any changes to your database server setup, ensure you have a backup of all of your current data. This tutorial will leave all of the data files in place, removing only the MySQL binary and associated tools – but it is always ideal to have a backup in the event something goes awry. We have a number of articles here on DigitalOcean that cover backing up your database files.

- 
Configuration Backup (Replacement Only): Likewise, it is recommended that you make a copy of your current MySQL/MariaDB configuration if you are replacing a current installation; this file can be found at /etc/mysql/my.cnf on Debian/Ubuntu systems and /etc/my.cnf on CentOS systems. On CentOS, the MariaDB package will remove the configuration file when uninstalled, so this step is especially important on these systems.


# Step One — Checking Versions (Replacement Only)


Percona Server versions are drop-in compatible with their equivalent MySQL versions only. i.e. MySQL 5.6 should be replaced with Percona Server 5.6 only. Attempting to use mismatched versions may lead to table corruption or prevent the server from starting.


To check which version you are currently running, first connect to MySQL with your current root password:


```
mysql -u root -p

```


Then find the current installed version:


```
SHOW VARIABLES LIKE "version";

```


This should identify if you need to install Percona Server 5.5 or 5.6. The one edge case is if you are running MariaDB 10.0, which should be replaced with Percona Server 5.6. If you are running a version of MySQL older than 5.5, you should first upgrade MySQL to 5.5 or greater before continuing.


# Step Two — Removing MySQL (Replacement Only)


Before we install Percona server, we will need to remove any MySQL or MariaDB packages that are currently installed, as you should not attempt to run both concurrently on the same data.


You should have a backup of your data and your configuration files before proceeding.


Before uninstalling MySQL, it is advised to stop the database server to prevent data corruption in the event the process isn’t stopped safely during the package removal:


```
service mysql stop

```


For Debian and Ubuntu based servers, the MySQL server and client packages need to be removed:


```
apt-get remove mysql-server mysql-client mysql-common
apt-get autoremove

```


For CentOS systems the default database is now MariaDB, which can be uninstalled as follows:


```
yum remove MariaDB-server MariaDB-client MariaDB-shared

```


For other variations, please consult your documentation for the uninstall procedure.


# Step Three — Installing Percona Server


Percona Server is likely not in your Linux distribution’s default repositories, as Percona manage their own repos to ensure updates are pushed to users as quickly as possible. Therefore, we will need to manually add the Percona APT or yum repositories before installation. Follow the instructions below for your server’s OS.


## Debian and Ubuntu (Apt)


The Debian and Ubuntu packages released by Percona are signed, meaning APT needs to be informed of the new signing key:


```
apt-key adv --keyserver keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A

```


Before we take the next step, make sure you know the distribution you are currently using. For Debian, this will be one of:


- squeeze
- wheezy

Likewise, for Ubuntu the supported distributions are:


- lucid
- precise
- saucy
- trusty

If you’re uncertain which distribution version you are using, you can execute the following command:


```
lsb_release -c

```


Once you are sure which distribution you are running, we can add the new Percona repositories by appending the following lines to the /etc/apt/sources.list file:


```
nano /etc/apt/sources.list

```


Add these lines at the bottom of the file, making sure to replace DIST with your distribution name (that is, you would replace DIST with wheezy or trusty, etc.):


```
deb http://repo.percona.com/apt DIST main
deb-src http://repo.percona.com/apt DIST main

```


Once you have saved the sources file, the Percona packages should next be pinned to ensure that the packages from Percona will always be prioritized over any packages from your distribution’s default repositories. To do this, we first create a new preference file for APT:


```
touch /etc/apt/preferences.d/00percona.pref

```


Now open this file at /etc/apt/preferences.d/00percona.pref with your chosen text editor (Vim, nano, etc.), add the following lines, and save:


```
Package: *
Pin: release o=Percona Development Team
Pin-Priority: 1001

```


Finally, once the sources are added and pinned, the package list can be updated and we can install the Percona Server package.


(New Only) For a fresh Droplet, it is advised you install the percona-server-server virtual package, which will install the version of Percona Server recommended by the Percona team:


```
apt-get update
apt-get install percona-server-server

```


(Replacement Only) Refer to the MySQL or MariaDB version you located earlier. For replacing version 5.5 use the percona-server-server-5.5 package and percona-server-server-5.6 for 5.6. MariaDB 10.0 should be replaced with Percona Server 5.6.


```
apt-get update
apt-get install percona-server-server-5.6

```


If this command completes without errors, Percona Server will be installed and successfully running. However, if you get errors during installation, ensure that you have sufficient free memory, as per the prerequisites section above. More information about any startup errors may be found in Percona Server’s log file at /var/log/mysqld.log.


(New Only) When installing on a fresh system, you may be asked during the installation process to set a root database user password. It is also recommended in this situation to run mysql_secure_installation to ensure no obvious security issues remain:


```
/usr/bin/mysql_secure_installation

```


## CentOS (Yum)


Only CentOS versions 5 and 6 are currently supported by Percona. At the time of writing, CentOS 7 is not supported.


The first step for CentOS systems is the installation of the Percona repository packages to yum.


For 64-bit CentOS systems:


```
yum install http://www.percona.com/downloads/percona-release/percona-release-0.0-1.x86_64.rpm

```


For 32-bit CentOS systems:


```
yum install http://www.percona.com/downloads/percona-release/percona-release-0.0-1.i386.rpm

```


Once this is complete, we can then install the correct version of the Percona Server package.


(New Only) When installing from scratch, it is recommended you use the Percona Server 5.6 package:


```
yum install Percona-Server-client-56 Percona-Server-server-56

```


(Replacement Only) When replacing a previous installation, use the version number found in the version check section above to choose the correct corresponding package of Percona-Server-server-55 or Percona-Server-server-56. MariaDB 10.0 should be replaced with Percona Server 5.6.


```
yum install Percona-Server-client-56 Percona-Server-server-56

```


You will be asked to accept the packages and then accept the package signing key –agree to both of these. After a short while the installation should complete without any errors. If you receive transaction errors during the installation, ensure that you have completely removed any MySQL/MariaDB packages before retrying.


With the package now installed, the final installation step is to start the server:


```
service mysql start

```


If you receive an error regarding the PID file, the server is failing to start. As mentioned in the prerequisites, this often occurs on low RAM servers where the memory limit prevents the XtraDB buffer pool allocation. If this is not the case, check /var/log/mysqld.log for any further error messages, making sure to check that you have properly matched versions between MariaDB and Percona Server.


Note: If you upgrade from MariaDB and receive an error like this:


```
Can't read dir of '/etc/my.cnf.d' (Errcode: 2 - No such file or directory)
Fatal error in defaults handling. Program aborted
Starting MySQL (Percona Server). ERROR! The server quit without updating PID file (/var/lib/mysql/percona-centos.pid).

```


You should be able to create the appropriate directory with the command mkdir /etc/my.cnf.d. Then try to start the server.


(New Only) When installing on a fresh system, your Percona Server root user will not have a password assigned and hence your databases will not be secure. It is therefore highly recommended that you use mysql_secure_installation to set up a new password and other security options. When asked to enter the current root password, just press Enter.


```
/usr/bin/mysql_secure_installation

```


# Step Four — Configuration


If you replaced an existing MySQL installation, you should have made a copy of your configuration file that can now be copied back. You can probably skip this section, although you may find the example settings useful.


However, if you are installing Percona Server on a fresh Droplet, you will need to add a configuration file, as Percona Server is currently running on default values that may not be optimal for your Droplet. Before we update the configuration for the first time, it is advised to stop Percona Server, as the PID file location may change. For future configuration changes, a simple restart should suffice after changes are made.


```
service mysql stop

```


Below is a sample configuration file for a 1GB droplet, with a relatively small buffer pool for compatibility. On Debian/Ubuntu this file should be written to /etc/mysql/my.cnf. On CentOS, this file should be written to /etc/my.cnf. The file contains in-line comments to explain what the different sections are doing.


```
# Generated by Percona Configuration Wizard (http://tools.percona.com/) version REL5-20120208
[mysql]

# CLIENT #
# Configure default options for clients
port                           = 3306

[mysqld]

# GENERAL #
# Choose user for execution, default storage engine and location of the PID file
user                           = mysql
default-storage-engine         = InnoDB
pid-file                       = /var/lib/mysql/mysql.pid

# MyISAM #
# Setup MyISAM options with a minimal config, as InnoDB is our default engine
key-buffer-size                = 32M
myisam-recover                 = FORCE,BACKUP

# SAFETY #
# Enforce limits and safety checks
max-allowed-packet             = 16M
max-connect-errors             = 1000000
innodb                         = FORCE

# DATA STORAGE #
# Select location for database files
datadir                        = /var/lib/mysql/

# BINARY LOGGING #
# Enable and setup the binary log
log-bin                        = /var/lib/mysql/mysql-bin
expire-logs-days               = 14
sync-binlog                    = 1

# CACHES AND LIMITS #
# Configure reasonable default limits throughout Percona Server
tmp-table-size                 = 32M
max-heap-table-size            = 32M
query-cache-type               = 0
query-cache-size               = 0
max-connections                = 500
thread-cache-size              = 50
open-files-limit               = 65535
table-definition-cache         = 1024
table-open-cache               = 2048

# INNODB #
# Setup InnoDB/XtraDB engine a 300MB buffer pool and 32MB log file size
innodb-flush-method            = O_DIRECT
innodb-log-files-in-group      = 2
innodb-log-file-size           = 32M
innodb-flush-log-at-trx-commit = 1
innodb-file-per-table          = 1
innodb-buffer-pool-size        = 300M

# LOGGING #
# Setup log file locations for error log and slow log
# Slow log may be disabled on production setups to prevent extra IO
log-error                      = /var/lib/mysql/mysql-error.log
log-queries-not-using-indexes  = 1
slow-query-log                 = 1
slow-query-log-file            = /var/lib/mysql/mysql-slow.log

```


For larger Droplets, or as a starting point for your customized configuration file, you can use the Percona Configuration Wizard to create a suitable base for your configuration file.


Once you have saved the file, you can restart Percona Server:


```
service mysql restart

```


If the server fails to start with this config, try reducing innodb-buffer-pool-size to a smaller value and repeating the above command.


# Step Five — Checking Your Installation


Now that we have Percona Server installed and running, we can ensure that everything has gone as planned by running a few final checks. First, connect to the database using the mysql client, logging in with your database root user password:


```
mysql -u root -p

```


```
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 45
Server version: 5.5.38-35.2 Percona Server (GPL), Release 35.2, Revision 674

Copyright (c) 2009-2014 Percona LLC and/or its affiliates
Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 

```


Immediately we can see that the server version in the connection text is now specifying Percona Server. By using the SHOW VARIABLES command, we can dig into further details about the specific version that has been installed:


```
SHOW VARIABLES LIKE "version%";

```


```
+-------------------------+--------------------------------------------------+
| Variable_name           | Value                                            |
+-------------------------+--------------------------------------------------+
| version                 | 5.5.38-35.2                                      |
| version_comment         | Percona Server (GPL), Release 35.2, Revision 674 |
| version_compile_machine | x86_64                                           |
| version_compile_os      | debian-linux-gnu                                 |
+-------------------------+--------------------------------------------------+
4 rows in set (0.00 sec)

```


The precise values and versions may vary in your installation, but the key point is that we are now have Percona Server running rather than MySQL.


Next, we can check that we are taking advantage of XtraDB for any InnoDB based tables:


```
SHOW STORAGE ENGINES\G

```


The result will show this block among many others:


```

...

*************************** 8. row ***************************
      Engine: InnoDB
     Support: DEFAULT
     Comment: Percona-XtraDB, Supports transactions, row-level locking, and foreign keys
Transactions: YES
          XA: YES
  Savepoints: YES
  
...

9 rows in set (0.00 sec)


```


The comment field within the response shows that the XtraDB engine has been loaded as the engine for InnoDB based tables. As a final check, it is advised that you ensure all of your databases and tables are being read properly in the new server.


If all of these checks were passed, you now have Percona Server running successfully. However, if any of these checks were not successful, please ensure you have properly completed all prior steps of this tutorial, paying particular attention to matching MySQL version numbers to Percona Server version numbers.


## Next Steps


As Percona is drop-in compatible with MySQL, all of the DigitalOcean tutorials covering MySQL can safely be used with Percona Server. When connecting to the database, your applications will still function identically, although hopefully now with the performance boosts from both Percona Server and XtraDB.


MySQL commands will work exactly as before.


