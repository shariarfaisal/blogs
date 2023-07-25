# How To Set Up a Help Desk System with OTRS on CentOS 7

```Miscellaneous``` ```CentOS```

## Introduction


OTRS is an Open source Ticket Request System. It provides a single point of contact for users, customers, IT personnel, IT services, and any external organizations. The program is written in Perl, supports a variety of databases (MySQL, PostgreSQL, etc), and can integrate with LDAP directories.


In this tutorial, you will learn how to install and set up OTRS on your CentOS server.


# Prerequisites


To follow this tutorial, you will need:


- 
One CentOS 7 Droplet with a sudo non-root user, which you can set up by following this initial CentOS server setup article.

- 
4 GB of swap space, which you can set up by following this swap tutorial.


# Step 1 — Installing MariaDB


In this step, we’ll install the prerequisite programs for OTRS.


First, enable the EPEL (Extra Packages for Enterprise Linux) repository.


```
sudo yum install epel-release


```


Then update your system.


```
sudo yum update


```


In this tutorial, we’ll use MySQL for our database, so install MariaDB (which is a fork of MySQL).


```
sudo yum install mariadb-server mariadb


```


You will need to change the default MySQL settings in order to make it suitable for OTRS. Open its configuration file using vi or your favorite text editor.


```
sudo vi /etc/my.cnf


```


Add the following lines under the [mysqld] section, which specify the sizes of a few files.


/etc/my.cnf
```
[mysqld]
max_allowed_packet = 20M
query_cache_size = 32M
innodb_log_file_size = 256M
datadir=/var/lib/mysql
. . .

```


Then save and close the file. Make sure you do this before you start MySQL for the first time.


Now, start MariaDB.


```
sudo systemctl start mariadb.service


```


Next, secure the MySQL database.


```
sudo mysql_secure_installation


```


You will be asked a few questions. You can accept the default values for all of the questions by just pressing ENTER for each, except for setting the new root password. Make a note of your root user password because you will need it later in this tutorial.


Now we have everything we need to install the OTRS application.


# Step 2 — Installing OTRS


We will install OTRS using the pre-built RPM package for CentOS. First, we need to download the latest RPM from their official repository. You can browse the repository directory to determine the latest version.


```
wget http://ftp.otrs.org/pub/otrs/RPMS/rhel/7/otrs-5.0.7-01.noarch.rpm


```


Next, install OTRS.


```
sudo yum install otrs-5.0.7-01.noarch.rpm


```


Because OTRS is written in Perl, it uses a number of Perl modules. We can check for missing modules by using the CheckModules.pl script included with OTRS.


```
sudo /opt/otrs/bin/otrs.CheckModules.pl


```


You’ll see output like this.


Output
```
  o Apache::DBI......................ok (v1.12)
  o Apache2::Reload..................FAILED! Not all prerequisites for this module correctly installed. 
. . .
  o XML::LibXSLT.....................ok (v1.80)
  o XML::Parser......................ok (v2.41)
  o YAML::XS.........................Not installed! Use: 'yum install "perl(YAML::XS)"' (required - Very important)

```


Some modules are only needed for optional functionality, such as communication with other databases or handling mail with Chinese character sets. You can install the missing modules with the yum commands provided in the output. Feel free to go through them manually, or use the command below.


```
sudo yum install "perl(Apache2::Reload)" "perl(Crypt::Eksblowfish::Bcrypt)" "perl(Encode::HanExtra)" "perl(JSON::XS)" "perl(Mail::IMAPClient)" "perl(ModPerl::Util)" "perl(Text::CSV_XS)" "perl(YAML::XS)"


```


Whenever you’re done installing modules, you can rerun the script to make sure that all the required modules have been installed.


# Step 3 — Сonfiguring OTRS


In this step, we’ll configure OTRS’s database and mail settings.


First, we need to restart Apache to load the configuration changes for OTRS.


```
sudo systemctl restart httpd.service


```


Now you can access the installer’s web page. Open http://your_server_ip/otrs/installer.pl in your favorite web browser. On the first screen, you will see a welcome screen with information about the OTRS offices. Click Next. The next screen will have the license, which you can accept by clicking Accept license and continue after reading.


On the next screen, you will be prompted to select a database type. The defaults (MySQL and Create a new database for OTRS) are fine, so click Next to proceed.





Then you’ll have to enter the MySQL credentials you chose in a previous step. Click Check database settings to make sure it works.





The installer will generate credentials for the new database. There is no need to remember this generated password, so click Next to proceed.





The database will be created and you will see the successful result. Click Next.


Next you have to provide some required system settings:


- System FQDN: A fully qualified domain name. You can set up your own host name, or you can just use your server’s IP address here.
- AdminEmail: The e-mail address of your system administrator. Emails about errors with OTRS will go here.
- Organization: Your organization’s name.

Leave all other options at their default values.





In order to be able to receive e-mails from users, you have to configure an incoming mail account.


Provide the necessary credentials in the Configure Inbound Mail section. For example, if you use Google as your mail provider, you can create an app password and enter the following information:


- Inbound mail type: IMAPS
- Inbound mail host: imap.gmail.com
- Inbound mail user: your_email_address
- Inbound mail password: your_app_password

To check the configuration, press the corresponding button. After a few seconds you will see the message: “Mail check successful.” Click OK to proceed to final screen.





The installation is complete! As a result, you will see the page with a link to the admin panel and the credentials of the superuser.


Make sure you write down the generated password for the root@localhost user and the start page URL.


The only thing left after a successful installation is to start the OTRS daemon and activate its cronjob.


```
sudo su - otrs -c "/opt/otrs/bin/otrs.Daemon.pl start"
sudo su - otrs -c "/opt/otrs/bin/Cron.sh start"


```


# Step 4 — Securing OTRS


At the moment, we have a fully functional application, but it’s not secure to use the superuser account with OTRS. Instead, we’ll create new agents.


In OTRS, agents are users who have rights to the various functions of the system. In our example, we will use single agent who has access to the all functions of the system.


First of all, we have to log in as root@localhost to create new agents. Open the link which we received at the end of the installation. Enter root@localhost for the username and the password you copied at the end of step 3, then click Login.


You will see the main dashboard. It contains several widgets which show different information about tickets, statistics, news, etc. You can freely rearrange them by dragging or switch their visibility in settings.





First we have to create a new agent. To do this, follow the link by clicking on the red message in the top of the screen, then click the Add agent button. This will bring you to a screen with a lot of fields. Fortunately, most of the default options are fine. You can simply fill in the first name, last name, username, password, and email fields.


Next, you need to change group relations for the new agent. Because our agent will also be the administrator, we will give it full read and write access to all groups. To do this, click the checkmark next to RW all the way on the right, under Change Group Relations for Agent.


Finally, click Submit. Now you can log out and log back in again using the newly created account.  You can customize your agent’s preferences by clicking on the gear in the top left corner of the screen. There you can change your password, choose the interface language, setup notifications, setup the favorite queues, change interface skin, etc.


Once you save your settings, you are ready to accept tickets from customers.


# Step 5 — Handling Tickets


Let’s go over how to deal with tickets. Customers have two ways to forward new tickets to OTRS: via the customer front-end or by sending an email.


The customer front end is located at http://your_server_ip/otrs/customer.pl. You can create a customer account there and submit a ticket using the GUI.


You can also create new ticket by sending an email to the address specified during installation. By default, all tickets received by mail are stored in one queue and have normal priority. All customer tickets can be viewed in the customer web interface regardless of how they were sent.


All new tickets created using the customer front-end, will immediately appear on the agent’s dashboard. Tickets sent by mail may not immediately appear on the dashboard because OTRS checks for them every 10 minutes.


On the agent dashboard, you can see the information on all currently actual tickets: their status (new, opened, escalated, etc.), their age (the time elapsed from the moment when ticket was received), and subject.





You can click on the ticket number (in the Ticket # column) to view its details. The agent can also take actions on the ticket here, like changing its priority or state, moving it to another queue, closing it, adding a note, and so on.


# Conclusion


In this tutorial, we have learned how to set up and use a simple help desk service using OTRS. You can learn more about OTRS by reading the OTRS Admin Manual.


