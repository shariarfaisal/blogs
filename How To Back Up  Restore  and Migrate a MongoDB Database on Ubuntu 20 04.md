# How To Back Up  Restore  and Migrate a MongoDB Database on Ubuntu 20 04

```Ubuntu``` ```Backups``` ```MongoDB``` ```Data Analysis```

The author selected the COVID-19 Relief Fund to receive a donation as part of the Write for DOnations program.


# Introduction


MongoDB is one of the most popular NoSQL database engines. It is famous for being scalable, robust, reliable, and easy to use. database engines. It is famous for being scalable, robust, reliable, and easy to use. In this article, you will back up, restore, and migrate a sample MongoDB database.


Importing and exporting a database means dealing with data in a human-readable format that is compatible with other software products. In contrast, MongoDB’s backup and restore operations create or use MongoDB-specific binary data, which preserves not only the consistency and integrity of your data but also its specific MongoDB attributes. Thus, for migration, it’s usually preferable to use backup and restore as long as the source and target systems are compatible.


# Prerequisites


Before following this tutorial, please make sure you complete the following prerequisites:


- An Ubuntu 20.04 server with a sudo non-root user and a firewall, which you can set up with the Ubuntu 20.04 initial server setup guide.
- MongoDB installed and configured using the How To Install MongoDB on Ubuntu 20.04 tutorial.
- Example MongoDB database imported using the instructions in How To Import and Export a MongoDB Database.

Except otherwise noted, all of the commands that require root privileges in this tutorial should be run as a non-root user with sudo privileges.


# Step 1 — Using JSON and BSON in MongoDB


Before continuing further with this article, some basic understanding of the matter is needed. If you have experience with other NoSQL database systems such as Redis, you may find some similarities when working with MongoDB.


MongoDB uses JSON and BSON (binary JSON) formats for storing its information. JSON is the human-readable format that is perfect for exporting and, eventually, importing your data. You can further manage your exported data with any tool that supports JSON, including a simple text editor.


An example .json document looks like this:


Example of JSON Format
```
{"address":[
    {"building":"1007", "street":"Park Ave"},
    {"building":"1008", "street":"New Ave"},
]}

```


JSON is convenient to work with, but it does not support all the data types available in BSON. This means that there will be the so-called ‘loss of fidelity’ of the information if you use JSON. For backing up and restoring, it’s better to use the binary BSON.


Second, you don’t have to worry about explicitly creating a MongoDB database. If the database you specify for import doesn’t already exist, it is automatically created. Even better is the case with the collections’ (database tables) structure. In contrast to other database engines, in MongoDB, the structure is again automatically created upon the first document (database row) insert.


Third, in MongoDB, reading or inserting large amounts of data, such as this article’s tasks, can be resource-intensive and consume much of your CPU, memory, and disk space. This is critical considering that MongoDB is frequently used for large databases and Big Data. The simplest solution to this problem is to run the exports and backups during the night or non-peak hours.


Fourth, information consistency could be problematic if you have a busy MongoDB server where the information changes during the database export or backup process. One possible solution for this problem is replication, which you may consider when you advance in the MongoDB topic.


While you can use the import and export functions to backup and restore your data, there are better ways to ensure the full integrity of your MongoDB databases. To backup your data, you should use the command mongodump. For restoring, use mongorestore. Let’s see how they work.


# Step 2 — Using mongodump to Back Up a MongoDB Database


Let’s cover backing up your MongoDB database first.


An essential argument to mongodump is --db, which specifies the name of the database you want to back up. If you don’t specify a database name, mongodump backs up all of your databases. The second important argument is --out, which defines the directory into which the data will be dumped. For example, let’s back up the newdb database and storing it in the /var/backups/mongobackups directory. Ideally, we’ll have each of our backups in a directory with the current date like /var/backups/mongobackups/10-29-20.


First create that directory /var/backups/mongobackups:


```
sudo mkdir /var/backups/mongobackups


```


Then run mongodump:


```
sudo mongodump --db newdb --out /var/backups/mongobackups/$(date +'%m-%d-%y')


```


You will see an output like this:


```
Output2020-10-29T19:22:36.886+0000    writing newdb.restaurants to
2020-10-29T19:22:36.969+0000    done dumping newdb.restaurants (25359 documents)

```


Note that in the above directory path, you used date +'%m-%d-%y' to automatically get the current date. This will allow you to have backups inside the directory like /var/backups/10-29-20/, which makes it especially convenient for automating backups.


At this point, you have a complete backup of the newdb database in the directory /var/backups/mongobackups/10-29-20/newdb/. This backup has everything to restore the newdb properly and preserve its so-called “fidelity.”


As a general rule, you should make regular backups, preferably when the server is least loaded. Thus, you can set the mongodump command as a cron job so that it runs regularly, such as every day at 03:03 AM.


To accomplish this open crontab, cron’s editor:


```
sudo crontab -e


```


When you run sudo crontab, you will be editing the cron jobs for the root user. This is recommended because if you set the crons for your user, they might not execute properly, especially if your sudo profile requires password verification.


Inside the crontab prompt, insert the following mongodump command:


crontab
```
3 3 * * * mongodump --out /var/backups/mongobackups/$(date +'\%m-\%d-\%y')


```


In the above command, we omit the --db argument on purpose because you will typically want to have all of your databases backed up. Also, the special character % has to be escaped to comply with the cron syntax.


Depending on your MongoDB database sizes, you may soon run out of disk space with too many backups. That’s why it’s also recommended to clean the old backups regularly or to compress them.


For example, to delete all the backups older than seven days, you can use the following bash command:


```
find /var/backups/mongobackups/ -mtime +7 -exec rm -rf {} \;


```


Similar to the previous mongodump command, you can also add this as a cron job. It should run just before you start the next backup; for the 03:03 AM job, this deletion will run at at 03:01 AM. Open crontab again:


```
sudo crontab -e


```


Insert the following line:


crontab
```
1 3 * * * find /var/backups/mongobackups/ -mtime +7 -exec rm -rf {} \;


```


Save and close the file.


Completing all the tasks in this step will ensure a proper backup solution for your MongoDB databases. Next, you will restore the database.


# Step 3 — Using mongorestore to Restore and Migrate a MongoDB Database


When you restore your MongoDB database from a previous backup, you have the exact copy of your MongoDB information taken at a particular time, including all the indexes and data types, which is especially useful when you want to migrate your MongoDB databases. For restoring MongoDB, we’ll use the command mongorestore, which works with the binary backups that mongodump produces.


Let’s continue our examples with the newdb database and see how we can restore it from the previously taken backup. We’ll first specify the name of the database with the --nsInclude argument. We’ll be using newdb.* to restore all collections. To restore a single collection such as restaurants, use newdb.restaurants instead.


Then, using --drop, we’ll make sure that the target database is first dropped so that the backup is restored in a clean database. As a final argument we’ll specify the directory of the last backup, which will look something like this: /var/backups/mongobackups/10-29-20/newdb/.


Once you have a timestamped backup, you can restore it using this command (updating the date to match yours):


```
sudo mongorestore --db newdb --drop /var/backups/mongobackups/10-29-20/newdb/


```


You will see an output like this:


```
Output2020-10-29T19:25:45.825+0000    the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
2020-10-29T19:25:45.826+0000    building a list of collections to restore from /var/backups/mongobackups/10-29-20/newdb dir
2020-10-29T19:25:45.829+0000    reading metadata for newdb.restaurants from /var/backups/mongobackups/10-29-20/newdb/restaurants.metadata.json
2020-10-29T19:25:45.834+0000    restoring newdb.restaurants from /var/backups/mongobackups/10-29-20/newdb/restaurants.bson
2020-10-29T19:25:46.130+0000    no indexes to restore
2020-10-29T19:25:46.130+0000    finished restoring newdb.restaurants (25359 documents)
2020-10-29T19:25:46.130+0000    done

```


In the above case, we are restoring the data on the same server where we created the backup. If you wish to migrate the data to another server and use the same technique, you should copy the backup directory, which is /var/backups/mongobackups/10-29-20/newdb/ in our case, to the other server.


# Conclusion


You have now performed some essential tasks related to backing up, restoring, and migrating your MongoDB databases. No production MongoDB server should ever run without a reliable backup strategy, such as the one described here.


You can find more tutorials on how to configure and use MongoDB in these DigitalOcean community articles. We also encourage you to check out the official MongoDB documentation, as it’s a great resource on the possibilities that MongoDB provides.


