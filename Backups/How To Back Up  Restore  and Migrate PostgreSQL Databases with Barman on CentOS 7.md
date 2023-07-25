# How To Back Up  Restore  and Migrate PostgreSQL Databases with Barman on CentOS 7

```Backups``` ```PostgreSQL``` ```CentOS```

## Introduction


PostgreSQL is an open-source database platform quite popular with web and mobile application developers for its ease of maintenance, cost effectiveness, and simple integration with other open-source technologies.


One critical task of maintaining a PostgreSQL environment is to back up its databases regularly. Backups form part of the Disaster Recovery (DR) process for any organization. This is important for a few reasons:


- Safeguarding against data loss due to failure of underlying infrastructure components like storage or the server itself
- Safeguarding against data corruption and unwanted or malicious loss of data
- Migrating production databases into development or test environments

Usually the responsibility of database backup and restoration falls on the shoulder of a DBA. In smaller organizations or startups, however, system administrators, DevOps engineers, or programmers often have to create their own database backends. So, it’s important for everyone using PostgreSQL to understand how backups work and how to restore from a backup.


In this tutorial you’ll set up the Barman backup server, make a backup from a primary database server, and restore to a standby server.


## A Brief Introduction to PostgreSQL Backup Methods


Before launching into your Barman setup, let’s take a moment to review the types of backups available for PostgreSQL, and their uses. (For an even broader overview of backup strategies, read our article about effective backups.)


PostgreSQL offers two types of backup methods:


- Logical backups
- Physical backups

Logical backups are like snapshots of a database. These are created using the pg_dump or pg_dumpall utility that ships with PostgreSQL. Logical backups:


- Back up individual databases or all databases
- Back up just the schemas, just the data, individual tables, or the whole database (schemas and data)
- Create the backup file in proprietary binary format or in plain SQL script
- Can be restored using the pg_restore utility which also ships with PostgreSQL
- Do not offer point-in-time recovery (PITR)

This means if you make a logical backup of your database(s) at 2:00 AM in the morning, when you restore from it, the restored database will be as it was at 2:00 AM. There is no way to stop the restore at a particular point in time, say at 1:30 AM. If you are restoring the backup at 10:00 AM, you have lost eight hours’ worth of data.


Physical backups are different from logical backups because they deal with binary format only and makes file-level backups. Physical backups:


- Offer point-in-time recovery
- Back up the contents of the PostgreSQL data directory and the WAL (Write Ahead Log) files
- Take larger amounts of disk space
- Use the PostgreSQL pg_start_backup and pg_stop_backup commands. However, these commands need to be scripted, which makes physical backups a more complex process
- Do not back up individual databases, schemas only, etc. It’s an all-or-nothing approach

WAL files contain lists of transactions (INSERT, UPDATE or DELETE) that happen to a database. The actual database files containing the data are located within the data directory. So when it comes to restoring to a point in time from a physical backup, PostgreSQL restores the contents of the data directory first, and then plays the transactions on top of it from the WAL files. This brings the databases to a consistent state in time.


How Barman Backups Work


Traditionally, PostgreSQL DBAs would write their own backup scripts and scheduled cron jobs to implement physical backups. Barman does this in a standardized way.


Barman or Backup and Recovery Manager is a free, open-source PostgreSQL backup tool from 2ndQuadrant - a professional Postgres solutions company. Barman was written in Python and offers a simple, intuitive method of physical backup and restoration for your PostgreSQL instance. Some benefits of using Barman are:


- It’s totally free
- It’s a well-maintained application and has professional support available from the vendor
- Frees up the DBA / Sysadmin from writing and testing complex scripts and cron jobs
- Can back up multiple PostgreSQL instances into one central location
- Can restore to the same PostgreSQL instance or a different instance
- Offers compression mechanisms to minimize network traffic and disk space

## Goals


In this tutorial we will create three DigitalOcean Droplets, install PostgreSQL 9.4 on two of these machines, and install Barman on the third.


One of the PostgreSQL servers will be our main database server: this is where we will create our production database. The second PostgreSQL instance will be empty and treated as a standby machine where we can restore from the backup.


The Barman server will communicate with the main database server and perform physical backups and WAL archiving.


We will then emulate a “disaster” by dropping a table from our live database.


Finally, we will restore the backed up PostgreSQL instance from the Barman server to the standby server.


## Prerequisites


To follow this tutorial, you will need to create three DigitalOcean Droplets (or your own Linux servers), each with at least 2 GB of RAM and 2 CPU cores. We won’t go into the details of creating a Droplet; you can find more information here.


All three servers should have the same OS (CentOS 7 x64 bit).


We will name the machines as follows:


- main-db-server (we will denote its IP address as main-db-server-ip)
- standby-db-server (we will denote its IP address as standby-db-server-ip)
- barman-backup-server (we will denote its IP address as barman-backup-server-ip)

The actual IP addresses of the machines can be found from the DigitalOcean control panel.


You should also set up a sudo user on each server and use that for general access. Most of the commands will be executed as two different users (postgres and barman), but you will need a sudo user on each server as well so you can switch to those accounts. To understand how sudo privileges work, see this DigitalOcean tutorial about enabling sudo access.



Note: This tutorial will be use the default Barman installation directory as the backup location. In CentOS, this location is: /var/lib/barman/. 2ndQuadrant recommends it’s best to keep the default path. In real-life use cases, depending on the size of your databases and the number of instances being backed up, you should check that there is enough space in the file system hosting this directory.


Warning: You should not run any commands, queries, or configurations from this tutorial on a production server. This tutorial will involve changing configurations and restarting PostgreSQL instances. Doing so in a live environment without proper planning and authorization would mean an outage for your application.

# Step 1 — Installing PostgreSQL Database Servers


We will first set up our database environment by installing PostgreSQL 9.4 on the main-db-server and the standby-db-server.


Please complete the PostgreSQL installation steps from this LEPP stack tutorial. From this tutorial, you will need to:


- Follow the section Step One — Installing PostgreSQL
- Follow the section Step Two — Configuring PostgreSQL

In Step Two — Configuring PostgreSQL, instead of making changes to the pg_hba.conf file to allow access to the database for a web server, add this line so the Barman server can connect, using the barman-backup-server IP address, followed by /32:


```
host	all		all		barman-backup-server-ip/32		trust


```


This configures PostgreSQL to accept any connection coming from the Barman server.


The rest of the instructions in that section can be followed as they are.



Note: Installing PostgreSQL will create an operating system user called postgres on the database server. This account does not have a password; you’ll switch to it from your sudo user.

Make sure you have installed PostgreSQL on both the main-db-server and the standby-db-server, and that you have allowed access on both of them from the barman-backup-server.


Next we’ll add some sample data to the main database server.


# Step 2 — Creating PostgreSQL Database and Tables


Once PostgreSQL is installed and configured on both the machines, we’ll add some sample data to the main-db-server to simulate a production environment.


On the main-db-server, switch to the user postgres:


```
sudo su - postgres


```


Start the psql utility to access the database server:


```
psql


```


From the psql prompt, run the following commands to create a database and switch to it:


```
CREATE DATABASE mytestdb;
\connect mytestdb;


```


An output message will tell you that you are now connected to database mytestdb as user postgres.


Next, add two tables in the database:


```
CREATE TABLE mytesttable1 (id integer NULL);
CREATE TABLE mytesttable2 (id integer NULL);


```


These are named mytesttable1 and mytesttable2.


Quit the client tool by typing \q and pressing ENTER.


# Step 3 — Installing Barman


Now we’ll install Barman on the backup server, which will both control and store our backups.


Complete this step on the barman-backup-server.


To do this, you will first need to install the following repositories:


- Extra Packages for Enterprise Linux (EPEL) repository
- PostgreSQL Global Development Group RPM repository

Run the following command to install EPEL:


```
sudo yum -y install epel-release


```


Run these commands to install the PostgreSQL repo:


```
sudo wget http://yum.postgresql.org/9.4/redhat/rhel-7Server-x86_64/pgdg-centos94-9.4-1.noarch.rpm
sudo rpm -ivh pgdg-centos94-9.4-1.noarch.rpm


```


Finally, run this command to install Barman:


```
sudo yum -y install barman


```



Note: Installing Barman will create an operating system user called barman. This account does not have a password; you can switch to this user from your sudo user account.

Barman is installed! Now, let’s make sure the servers can connect to each other securely.


# Step 4 — Configuring SSH Connectivity Between Servers


In this section, we’ll establish SSH keys for a secure passwordless connection between the main-db-server and the barman-backup-server, and vice versa.


Likewise, we’ll establish SSH keys between the standby-db-server and the barman-backup-server, and vice versa.


This is to ensure PostgreSQL (on both database servers) and Barman can “talk” to each other during backups and restores.


For this tutorial you will need to make sure:


- User postgres can connect remotely from the main-db-server to the barman-backup-server
- User postgres can connect remotely from the standby-db-server to the barman-backup-server
- User barman can connect remotely from the barman-backup-server to the main-db-server
- User barman can connect remotely from the barman-backup-server to the standby-db-server

We will not go into the details of how SSH works. There’s a very good article on DigitalOcean about SSH essentials which you can refer to.


All the commands you’ll need are included here, though.


We’ll show you how to do this once for setting up the connection for the user postgres to connect from the main-db-server to the barman-backup-server.


From the main-db-server, switch to user postgres if it’s not already the current user:


```
sudo su - postgres


```


Run the following command to generate an SSH key pair:


```
ssh-keygen -t rsa


```


Accept the default location and name for the key files by pressing ENTER.


Press ENTER twice to create the private key without any passphrase.


Once the keys are generated, there will be a .ssh directory created under the postgres user’s home directory, with the keys in it.


You will now need to copy the SSH public key to the authorized_keys file under the barman user’s .ssh directory on the barman-backup-server.



Note: Unfortunately you can’t use the ssh-copy-id barman@barman-backup-server-ip command here. That’s because this command will ask for the barman user’s password, which is not set by default. You will therefore need to copy the public key contents manually.

Run the following command to output the postgres user’s public key contents:


```
cat ~/.ssh/id_rsa.pub


```


Copy the contents of the output.


Switch to the console connected to the barman-backup-server server and switch to the user barman:


```
sudo su - barman


```


Run the following commands to create a .ssh directory, set its permissions, copy the public key contents to the authorized_keys file, and finally make that file readable and writable:


```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
echo "public_key_string" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys


```


Make sure you put the long public key string starting with ssh-rsa between the quotation marks, instead of public_key_string.


You’ve copied the key to the remote server.


Now, to test the connection, switch back to the main-db-server and test the connectivity from there:


```
ssh barman@barman-backup-server-ip


```


After the initial warning about the authenticity of the remote server not being known and you accepting the prompt, a connection should be established from the main-db-server server to the barman-backup-server. If successful, log out of the session by executing the exit command.


```
exit


```


You need to set up SSH key connections three more times. You can skip making the .ssh directory if it’s already made (although this isn’t necessary).


- Run the same commands again, this time from the standby-db-server to the barman-backup-server
- Run them a third time, this time originating from the barman user on the barman-backup-server, and going to the postgres user on the main-db-server
- Finally, run the commands to copy the key from the barman user on the barman-backup-server to the postgres user on the standby-db-server

Make sure you test the connection each way so that you can accept the initial warning about the new connection.


From standby-db-server:


```
ssh barman@barman-backup-server-ip


```


From barman-backup-server:


```
ssh postgres@main-db-server-ip


```


From barman-backup-server:


```
ssh postgres@standby-db-server-ip


```



Note: Ensuring SSH connectivity between all three servers is a requirement for backups to work.

# Step 5 — Configuring Barman for Backups


You will now configure Barman to back up your main PostgreSQL server.


The main configuration file for BARMAN is /etc/barman.conf. The file contains a section for global parameters, and separate sections for each server that you want to back up. The default file contains a section for a sample PostgreSQL server called main, which is commented out. You can use it as a guide to set up other servers you want to back up.



A semicolon (;) at the beginning of a line means that line is commented out. Just like with most Linux-based applications, a commented-out configuration parameter for Barman means the system will use the default value unless you uncomment it and enter a different value.
One such parameter is the configuration_files_directory, which has a default value of /etc/barman.d. What this means is, when enabled, Barman will use the .conf files in that directory for different Postgres servers’ backup configurations. If you find the main file is getting too lengthy, feel free to make separate files for each server you want to back up.
For the sake of simplicity in this tutorial, we will put everything in the default configuration file.

Open /etc/barman.conf in a text editor as your sudo user (user barman has only read access to it):


```
sudo vi /etc/barman.conf


```


The global parameters are defined under the [barman] section. Under this section, make the following changes. The finished values are shown below the bullet points:


- Uncomment the line for compression and keep the default value of gzip. This means the PostgreSQL WAL files - when copied under the backup directory - will be saved in gzip compressed form
- Uncomment the line for reuse_backup and keep the default value of link. When creating full backups of the PostgreSQL server, Barman will try to save space in the backup directory by creating file-level incremental backups. This uses rsync and hard links. Creating incremental full backups has the same benefit of any data de-duplication method: savings in time and disk space
- Uncomment the line for immediate_checkpoint and set its value to true. This parameter setting ensures that when Barman starts a full backup, it will request PostgreSQL to perform a CHECKPOINT. Checkpointing ensures any modified data in PostgreSQL’s memory cache are written to data files. From a backup perspective, this can add some value because BARMAN would be able to back up the latest data changes
- Uncomment the line for basebackup_retry_times and set a value of 3. When creating a full backup, Barman will try to connect to the PostgreSQL server three times if the copy operation fails for some reason
- Uncomment the line for basebackup_retry_sleep and keep the default value of 30.  There will be a 30-second delay between each retry
- Uncomment the line for last_backup_maximum_age and set its value to 1 DAYS

The new settings should look like this exactly:


Excerpts from /etc/barman.conf
```
[barman]
barman_home = /var/lib/barman

. . .

barman_user = barman
log_file = /var/log/barman/barman.log
compression = gzip
reuse_backup = link

. . .

immediate_checkpoint = true

. . .

basebackup_retry_times = 3
basebackup_retry_sleep = 30
last_backup_maximum_age = 1 DAYS

```


What we are doing here is this:


- Keeping the default backup location
- Specifying that backup space should be saved. WAL logs will be compressed and base backups will use incremental data copying
- Barman will retry three times if the full backup fails halfway through for some reason
- The age of the last full backup for a PostgreSQL server should not be older than 1 day

At the end of the file, add a new section. Its header should say [main-db-server] in square brackets. (If you want to back up more database servers with Barman, you can make a block like this for each server and use a unique header name for each.)


This section contains the connection information for the database server, and a few unique backup settings.


Add these parameters in the new block:


Excerpt from /etc/barman.conf
```
[main-db-server]
description = "Main DB Server"
ssh_command = ssh postgres@main-db-server-ip
conninfo = host=main-db-server-ip user=postgres
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
wal_retention_policy = main

```


The retention_policy settings mean that Barman will overwrite older full backup files and WAL logs automatically, while keeping enough backups for a recovery window of 7 days. That means we can restore the entire database server to any point in time in the last seven days. For a production system, you should probably set this value higher so you have older backups on hand.


You’ll need to use the IP address of the main-db-server in the ssh_command and conninfo parameters. Otherwise, you can copy the above settings exactly.


The final version of the modified file should look like this, minus all the comments and unmodified settings:


Excerpts from /etc/barman.conf
```
[barman]
barman_home = /var/lib/barman

. . .

barman_user = barman
log_file = /var/log/barman/barman.log
compression = gzip
reuse_backup = link

. . .

immediate_checkpoint = true

. . .

basebackup_retry_times = 3
basebackup_retry_sleep = 30
last_backup_maximum_age = 1 DAYS

. . .

[main-db-server]
description = "Main DB Server"
ssh_command = ssh postgres@main-db-server-ip
conninfo = host=main-db-server-ip user=postgres
retention_policy_mode = auto
retention_policy = RECOVERY WINDOW OF 7 days
wal_retention_policy = main

```


Save and close the file.


Next, we’ll make sure our main-db-server is configured to make backups.


# Step 6 — Configuring the postgresql.conf File


There is one last configuration to be made on the main-db-server, to switch on backup (or archive) mode.


First, we need to locate the value of the incoming backup directory from the barman-backup-server. On the Barman server, switch to the user barman:


```
sudo su - barman


```


Run this command to locate the incoming backup directory:


```
barman show-server main-db-server | grep incoming_wals_directory


```


This should output something like this:


```
barman show-server command outputincoming_wals_directory: /var/lib/barman/main-db-server/incoming

```


Note down the value of incoming_wals_directory; in this example, it’s /var/lib/barman/main-db-server/incoming.


Now switch to the main-db-server console.


Switch to the user postgres if it’s not the current user already.


Open the postgresql.conf file in a text editor:


```
vi $PGDATA/postgresql.conf


```


Make the following changes to the file:


- Uncomment the wal_level parameter and set its value to archive instead of minimal
- Uncomment the archive_mode parameter and set its value to on instead of off
- Uncomment the archive_command parameter and set its value to 'rsync -a %p barman@barman-backup-server-ip:/var/lib/barman/main-db-server/incoming/%f' instead of ''. Use the IP address of the Barman server. If you got a different value for incoming_wals_directory, use that one instead

Excerpts from postgresql.conf
```
wal_level = archive                     # minimal, archive, hot_standby, or logical

. . .

archive_mode = on               # allows archiving to be done

. . .

archive_command = 'rsync -a %p barman@barman-backup-server-ip:/var/lib/barman/main-db-server/incoming/%f'                # command to use to archive a logfile segment


```


Switch back to your sudo user.


Restart PostgreSQL:


```
sudo systemctl restart postgresql-9.4.service


```



Note: If you are configuring an existing production PostgreSQL instance, there’s a good chance these three parameters will be set already. You will then have to add/modify only the archive_command parameter so PostgreSQL sends its WAL files to the backup server.

# Step 7 — Testing Barman


It’s now time to check if Barman has all the configurations set correctly and can connect to the main-db-server.


On the barman-backup-server, switch to the user barman if it’s not the current user. Run the following command to test the connection to your main database server:


```
barman check main-db-server


```


Note that if you entered a different name between the square brackets for the server block in the /etc/barman.conf file in Step 5, you should use that name instead.


If everything is okay, the output should look like this:


```
barman check command outputServer main-db-server:
        PostgreSQL: OK
        archive_mode: OK
        wal_level: OK
        archive_command: OK
        continuous archiving: OK
        directories: OK
        retention policy settings: OK
        backup maximum age: FAILED (interval provided: 1 day, latest backup age: No available backups)
        compression settings: OK
        minimum redundancy requirements: OK (have 0 backups, expected at least 0)
        ssh: OK (PostgreSQL server)
        not in recovery: OK

```


Don’t worry about the backup maximum age FAILED state. This is happening because we have configured Barman so that the latest backup should not be older than 1 day. There is no backup made yet, so the check fails.


If any of the other parameters are in a FAILED state, you should investigate further and fix the issue before proceeding.


There can be multiple reasons for a check to fail: for example, Barman not being able to log into the Postgres instance, Postgres not being configured for WAL archiving, SSH not working between the servers, etc. Whatever the cause, it needs to be fixed before backups can happen. Run through the previous steps and make sure all the connections work.


To get a list of PostgreSQL servers configured with Barman, run this command:


```
barman list-server


```


Right now it should just show:


Output
```
main-db-server - Main DB Server

```


# Step 8 — Creating the First Backup


Now that you have Barman ready, let’s create a backup manually.


Run the following command as the barman user on the barman-backup-server to make your first backup:


```
barman backup main-db-server


```


Again, the main-db-server value is what you entered as the head of the server block in the /etc/barman.conf file in Step 5.


This will initiate a full backup of the PostgreSQL data directory. Since our instance has only one small database with two tables, it should finish very quickly.


Output
```
Starting backup for server main-db-server in /var/lib/barman/main-db-server/base/20151111T051954
Backup start at xlog location: 0/2000028 (000000010000000000000002, 00000028)
Copying files.
Copy done.
Asking PostgreSQL server to finalize the backup.
Backup size: 26.9 MiB. Actual size on disk: 26.9 MiB (-0.00% deduplication ratio).
Backup end at xlog location: 0/20000B8 (000000010000000000000002, 000000B8)
Backup completed
Processing xlog segments for main-db-server
        Older than first backup. Trashing file 000000010000000000000001 from server main-db-server
        000000010000000000000002
        000000010000000000000002.00000028.backup

```


## Backup File Location


So where does the backup get saved? To find the answer, list the contents of the /var/lib/barman directory:


```
ls -l /var/lib/barman


```


There will be one directory there: main-db-server. That’s the server Barman is currently configured to back up, and its backups live there. (If you configure Barman to back up other servers, there will be one directory created per server.) Under the main-db-server directory, there will be three sub-directories:


- base: This is where the base backup files are saved
- incoming: PostgreSQL sends its completed WAL files to this directory for archiving
- wals: Barman copies the contents of the incoming directory to the wals directory

During a restoration, Barman will recover contents from the base directory into the target server’s data directory. It will then use files from the wals directory to apply transaction changes and bring the target server to a consistent state.


## Listing Backups


There is a specific Barman command to list all the backups for a server. That command is barman list-backup. Run the following command to see what it returns for our main-db-server:


```
barman list-backup main-db-server

```


```
Outputmain-db-server 20151111T051954 - Wed Nov 11 05:19:46 2015 - Size: 26.9 MiB - WAL Size: 0 B

```


- The first part of the output is the name of the server. In this case, main-db-server
- The second part - a long alphanumeric value - is the backup ID for the backup. A backup ID is used to uniquely identify any backup Barman makes. In this case, it’s 20151111T051954. You will need the backup ID for the next steps
- The third piece of information tells you when the backup was made
- The fourth part is the size of the base backup (26.9 MB in this case)
- The fifth and final part of the string gives the size of the the WAL archive backed up

To see more details about the backup, execute this command using the name of the server, and the backup ID (20151111T051954 in our example) from the previous command:


```
barman show-backup main-db-server backup-id


```


A detailed set of information will be shown:


```
OutputBackup 20151111T051954:
  Server Name            : main-db-server
  Status                 : DONE
  PostgreSQL Version     : 90405
  PGDATA directory       : /var/lib/pgsql/9.4/data

  Base backup information:
    Disk usage           : 26.9 MiB (26.9 MiB with WALs)
    Incremental size     : 26.9 MiB (-0.00%)
    Timeline             : 1
    Begin WAL            : 000000010000000000000002
    End WAL              : 000000010000000000000002
    WAL number           : 1
    WAL compression ratio: 99.84%
    Begin time           : 2015-11-11 05:19:44.438072-05:00
    End time             : 2015-11-11 05:19:46.839589-05:00
    Begin Offset         : 40
    End Offset           : 184
    Begin XLOG           : 0/2000028
    End XLOG             : 0/20000B8

  WAL information:
    No of files          : 0
    Disk usage           : 0 B
    Last available       : 000000010000000000000002

  Catalog information:
    Retention Policy     : VALID
    Previous Backup      : - (this is the oldest base backup)
    Next Backup          : - (this is the latest base backup)

```


To drill down more to see what files go into the backup, run this command:


```
barman list-files main-db-server backup-id


```


This will give a list of the base backup and WAL log files required to restore from that particular backup.


# Step 9 — Scheduling Backups


Ideally your backups should happen automatically on a schedule.


In this step we’ll automate our backups, and we’ll tell Barman to perform maintenance on the backups so files older than the retention policy are deleted. To enable scheduling, run this command as the barman user on the barman-backup-server (switch to this user if necessary):


```
crontab -e


```


This will open a crontab file for the user barman. Edit the file, add these lines, then save and exit:


cron
```
30 23 * * * /usr/bin/barman backup main-db-server
* * * * * /usr/bin/barman cron

```


The first command will run a full backup of the main-db-server every night at 11:30 PM. (If you used a different name for the server in the /etc/barman.conf file, use that name instead.)


The second command will run every minute and perform maintenance operations on both WAL files and base backup files.


# Step 10 — Simulating a “Disaster”


You will now see how you can restore from the backup you just created. To test the restoration, let’s first simulate a “disaster” scenario where you have lost some data.


We’re dropping a table here. Don’t do this on a production database!


Go back to the main-db-server console and switch to the user postgres if it’s not already the current user.


Start the psql utility:


```
psql


```


From the psql prompt, execute the following command to switch the database context to mytestdb:


```
\connect mytestdb;


```


Next, list the tables in the database:


```
\dt


```


The output will show the tables you created at the beginning of this tutorial:


```
Output            List of relations
 Schema |     Name     | Type  |  Owner
--------+--------------+-------+----------
 public | mytesttable1 | table | postgres
 public | mytesttable2 | table | postgres

```


Now, run this command to drop one of the tables:


```
drop table mytesttable2;


```


If you now execute the \dt command again:


```
\dt


```


You will see that only mytesttable1 remains.


This is the type of data loss situation where you would want to restore from a backup. In this case, you will restore the backup to a separate server: the standby-db-server.


# Step 11 — Restoring or Migrating to a Remote Server


You can follow this section to restore a backup, or to migrate your latest PostgreSQL backup to a new server.


Go to the standby-db-server.


First, stop the PostgreSQL service as the sudo user. (The restart will choke if you try to run the restoration while the service is running.)


```
sudo systemctl stop postgresql-9.4.service


```


Once the service stops, go to the barman-backup-server. Switch to the user barman if it’s not already the current user.


Let’s locate the details for the latest backup:


```
barman show-backup main-db-server latest


```


Output
```
Backup 20160114T173552:
  Server Name            : main-db-server
  Status                 : DONE
  PostgreSQL Version     : 90405
  PGDATA directory       : /var/lib/pgsql/9.4/data

  Base backup information:

. . .

    Begin time           : 2016-01-14 17:35:53.164222-05:00
    End time             : 2016-01-14 17:35:55.054673-05:00

```


From the output, note down the backup ID printed on the first line (20160114T173552 above). If the latest backup has the data you want, you can use latest as the backup ID.


Also check when the backup was made, from the Begin time field (2016-01-14 17:35:53.164222-05:00 above).


Next, run this command to restore the specified backup from the barman-backup-server to the standby-db-server:


```
barman recover --target-time "Begin time"  --remote-ssh-command "ssh postgres@standby-db-server-ip"   main-db-server   backup-id   /var/lib/pgsql/9.4/data


```


There are quite a few options, arguments, and variables here, so let’s explain them.


- --target-time "Begin time": Use the begin time from the show-backup command
- --remote-ssh-command "ssh postgres@standby-db-server-ip": Use the IP address of the standby-db-server
- main-db-server: Use the name of the database server from your /etc/barman.conf file
- backup-id: Use the backup ID from the show-backup command, or use latest if that’s the one you want
- /var/lib/pgsql/9.4/data: The path where you want the backup to be restored. This path will become the new data directory for Postgres on the standby server. Here, we have chosen the default data directory for Postgres in CentOS. For real-life use cases, choose the appropriate path

For a successful restore operation, you should receive output like this:


Output from Barman Recovery
```
Starting remote restore for server  main-db-server using backup backup-id
Destination directory: /var/lib/pgsql/9.4/data
Doing PITR. Recovery target time: Begin time
Copying the base backup.
Copying required WAL segments.
Generating recovery.conf
Identify dangerous settings in destination directory.

IMPORTANT
These settings have been modified to prevent data losses

postgresql.conf line 207: archive_command = false

Your PostgreSQL server has been successfully prepared for recovery!

```


Now switch to the standby-db-server console again. As the sudo user, start the PostgreSQL service:


```
sudo systemctl start postgresql-9.4.service


```


That should be it!


Let’s verify that our database is up. Switch to user postgres and start the psql utility:


```
sudo su - postgres
psql


```


Switch the database context to mytestdb and list the tables in it:


```
\connect mytestdb;
\dt


```


Output
```
            List of relations
 Schema |     Name     | Type  |  Owner   
--------+--------------+-------+----------
 public | mytesttable1 | table | postgres
 public | mytesttable2 | table | postgres
(2 rows)

```


The list should show two tables in the database. In other words, you have just recovered the dropped table.


Depending on your larger recovery strategy, you may now want to fail over to the standby-db-server, or you may want to check that the restored database is working, and then run through this section again to restore to the main-db-server.


To restore to any other server, just make sure you’ve installed PostgreSQL and made the appropriate connections to the Barman server, and then follow this section using the IP address of your target recovery server.


# Conclusion


In this tutorial we have seen how to install and configure Barman to back up a PostgreSQL server. We have also learned how to restore or migrate from these backups.


With careful consideration, Barman can become the central repository for all your PostgresSQL databases. It offers a robust backup mechanism and a simple command set. However, creating backups is only half the story. You should always validate your backups by restoring them to a different location. This exercise should be done periodically.


Some questions for fitting Barman into your backup strategy:


- How many PostgreSQL instances will be backed up?
- Is there enough disk space on the Barman server for hosting all the backups for a specified retention period? How can you monitor the server for space usage?
- Should all the backups for different servers start at the same time or can they be staggered throughout off-peak period? Starting backups of all servers at the same time can put unnecessary strain on the Barman server and the network
- Is the network speed between the Barman server and Postgres servers reliable?

Another point to be mindful of is that Barman cannot backup and restore individual databases. It works on the file system level and uses an all-or-nothing approach. During a backup, the whole instance with all its data files are backed up; when restoring, all those files are restored. Similarly, you can’t  do schema-only or data-only backups with Barman.


We therefore recommend you design your backup strategy so it makes use of both logical backups with pg_dump or pg_dumpall and physical backups with Barman. That way, if you need to restore individual databases quickly, you can use pg_dump backups. For point-in-time recovery, use Barman backups.


