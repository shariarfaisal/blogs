# How To Install MariaDB on CentOS 8

```Databases``` ```MySQL``` ```CentOS``` ```CentOS 8``` ```MariaDB```

## Introduction


MariaDB is an open-source database management system, commonly used as an alternative for the MySQL portion of the popular LAMP (Linux, Apache, MySQL, PHP/Python/Perl) stack. It is intended to be a drop-in replacement for MySQL.


In this tutorial, we will explain how to install the latest version of MariaDB on a CentOS 8 server. If you’re wondering about MySQL vs. MariaDB, MariaDB is the preferred package and should work seamlessly in place of MySQL. If you specifically need MySQL, see the How to Install MySQL on CentOS 8 guide.


# Prerequisites


To follow this tutorial, you will need a CentOS 8 server with a non-root sudo-enabled user. You can learn more about how to set up a user with these privileges in the Initial Server Setup with CentOS 8 guide.


# Step 1 — Installing MariaDB


First, use dnf to install the MariaDB package:


```
sudo dnf install mariadb-server


```


You will be asked to confirm the action. Press y then ENTER to proceed.


Once the installation is complete, start the service with systemctl:


```
sudo systemctl start mariadb


```


Then check the status of the service:


```
sudo systemctl status mariadb


```


```
Output● mariadb.service - MariaDB 10.3 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; disabled; vendor preset: disabled)
   Active: active (running) since Fri 2020-04-03 17:32:46 UTC; 52min ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 4567 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 30 (limit: 5059)
   Memory: 77.1M
   CGroup: /system.slice/mariadb.service
           └─4567 /usr/libexec/mysqld --basedir=/usr

. . .

Apr 03 17:32:46 centos8-mariadb systemd[1]: Started MariaDB 10.3 database server.

```


If MariaDB has successfully started, the output should show active (running) and the final line should look something like:


```
OutputApr 03 17:32:46 centos8-mariadb systemd[1]: Started MariaDB 10.3 database server..

```


Next, let’s take a moment to ensure that MariaDB starts at boot, using the systemctl enable command:


```
sudo systemctl enable mariadb


```


```
OutputCreated symlink /etc/systemd/system/mysql.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/mysqld.service → /usr/lib/systemd/system/mariadb.service.
Created symlink /etc/systemd/system/multi-user.target.wants/mariadb.service → /usr/lib/systemd/system/mariadb.service.

```


We now have MariaDB running and configured to run at startup. Next, we’ll turn our attention to securing our installation.


# Step 2 — Securing the MariaDB Server


MariaDB includes a security script to change some of the less secure default options for things like remote root logins and sample users. Use this command to run the security script:


```
sudo mysql_secure_installation


```


The script provides a detailed explanation for every step. The first step asks for the root password, which hasn’t been set so we’ll press ENTER as it recommends. Next, we’ll be prompted to set that root password. Keep in mind that this is for the root database user, not the root user for your CentOS server itself.


Type Y then ENTER to enter a password for the root database user, then follow the prompts.


After updating the password, we will accept all the security suggestions that follow by pressing y and then ENTER. This will remove anonymous users, disallow remote root login, remove the test database, and reload the privilege tables.


Now that we’ve secured the installation, we’ll verify it’s working by connecting to the database.


# Step 3 — Testing the Installation


We can verify our installation and get information about it by connecting with the mysqladmin tool, a client that lets you run administrative commands. Use the following command to connect to MariaDB as root (-u root), prompt for a password (-p), and return the version.


```
mysqladmin -u root -p version


```


You should see output similar to this:


```
Outputmysqladmin  Ver 9.1 Distrib 10.3.17-MariaDB, for Linux on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version		10.3.17-MariaDB
Protocol version	10
Connection		Localhost via UNIX socket
UNIX socket		/var/lib/mysql/mysql.sock
Uptime:			6 min 5 sec

Threads: 7  Questions: 16  Slow queries: 0  Opens: 17  Flush tables: 1  Open tables: 11  Queries per second avg: 0.043

```


This indicates the installation has been successful.


# Conclusion


In this guide you installed MariaDB to act as an SQL server. During the installation process you also secured the server. Optionally, you also created a separate password-authenticated administrative user.


Now that you have a running and secure MariaDB server, here some examples of next steps that you can take to work with the server:


- You may want to import and export databases
- You could incorporate MariaDB into a larger software stack, such as the LAMP stack: How To Install Linux, Apache, MariaDB, PHP (LAMP stack) on CentOS 7
- You may need to update your firewalld firewall to allow for external database traffic

