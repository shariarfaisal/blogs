# How To Install and Use TimescaleDB on CentOS 7

```PostgreSQL``` ```CentOS```

The author selected the Computer History Museum to receive a donation as part of the Write for DOnations program.


## Introduction


Many applications, such as monitoring systems and data collection systems, accumulate data for further analysis. These analyses often look at the way a piece of data or a system changes over time. In these instances, data is represented as a time series, with every data point accompanied by a timestamp. An example would look like this:


```
2019-11-01 09:00:00    server.cpu.1    0.9
2019-11-01 09:00:00    server.cpu.15   0.8
2019-11-01 09:01:00    server.cpu.1    0.9
2019-11-01 09:01:00    server.cpu.15   0.8
...

```


The relevance of time series data has recently grown thanks to the new deployments of the Internet of Things (IoT) and Industrial Internet of Things. There are more and more devices that collect various time series information: fitness trackers, smart watches, home weather stations, and various sensors, to name a few. These devices collect a lot of information, and all this data must be stored somewhere.


Classic relational databases are most often used to store data, but they don’t always fit when it comes to the huge data volumes of time series. When you need to process a large amount of time series data, relational databases can be too slow. Because of this, specially optimized databases, called NoSQL databases, have been created to avoid the problems of relational databases.


TimescaleDB is an open-source database optimized for storing time series data. It is implemented as an extension of PostgreSQL and combines the ease-of-use of relational databases and the speed of NoSQL databases. As a result, it allows you to use PostgreSQL for both storing business data and time series data in one place.


By following this tutorial, you’ll set up TimescaleDB on CentOS 7, configure it, and learn how to work with it. You’ll run through creating time series databases and making simple queries. Finally, you’ll see how to remove unnecessary data.


# Prerequisites


To follow this tutorial, you will need:


- One CentOS 7 server set up by following our Initial Server Setup with CentOS 7 guide, including a non-root user with sudo privileges and a firewall set up with firewalld. To set up firewalld, follow the “Configuring a Basic Firewall” section of the Additional Recommended Steps for New CentOS 7 Servers tutorial.
- PostgreSQL installed on your server. Follow our How To Install and Use PostgreSQL on CentOS 7 guide to install and configure it.

# Step 1 — Installing TimescaleDB


TimescaleDB is not available in CentOS default package repositories, so in this step you will install it from the TimescaleDB’s third-party repository.


First, create a new repository file:


```
sudo vi /etc/yum.repos.d/timescaledb.repo


```


Enter insert mode by pressing i and paste the following configuration into the file:


/etc/yum.repos.d/timescaledb.repo
```
[timescale_timescaledb]
name=timescale_timescaledb
baseurl=https://packagecloud.io/timescale/timescaledb/el/7/$basearch
repo_gpgcheck=1
gpgcheck=0
enabled=1
gpgkey=https://packagecloud.io/timescale/timescaledb/gpgkey
sslverify=1
sslcacert=/etc/pki/tls/certs/ca-bundle.crt
metadata_expire=300

```


When you’re finished, press ESC to leave insert mode, then :wq and ENTER to save and exit the file. To learn more about the text editor vi and its successor vim, check out our Installing and Using the Vim Text Editor on a Cloud Server tutorial.


You can now proceed with the installation. This tutorial uses PostgreSQL version 11; if you are using a different version of PostgreSQL (9.6 or 11, for example), replace the value in the following command and run it:


```
sudo yum install -y timescaledb-postgresql-11


```


TimescaleDB is now installed and ready to be used. Next, you will turn it on and adjust some of the settings associated with it in the PostgreSQL configuration file to optimize the database.


# Step 2 — Configuring TimescaleDB


The TimescaleDB module works fine with the default PostgreSQL configuration settings, but to improve performance and make better use of processor, memory, and disk resources, developers of TimescaleDB suggest configuring some individual parameters. This can be done automatically with the timescaledb-tune tool or by manually editing your server’s postgresql.conf file.


In this tutorial, you will use the timescaledb-tune tool. It reads the postgresql.conf file and interactively suggests making changes.


Run the following command to start the configuration wizard:


```
sudo timescaledb-tune --pg-config=/usr/pgsql-11/bin/pg_config


```


First, you will be asked to confirm the path to the PostgreSQL configuration file:


```
OutputUsing postgresql.conf at this path:
/var/lib/pgsql/11/data/postgresql.conf

Is this correct? [(y)es/(n)o]:

```


The utility automatically detects the path to the configuration file, so confirm this by entering y:


```
Output...
Is this correct? [(y)es/(n)o]: y
Writing backup to:
/tmp/timescaledb_tune.backup201912191633

```


Next, enable the TimescaleDB module by typing y at the next prompt and pressing ENTER:


```
Outputshared_preload_libraries needs to be updated
Current:
#shared_preload_libraries = ''
Recommended:
shared_preload_libraries = 'timescaledb'
Is this okay? [(y)es/(n)o]:  y
success: shared_preload_libraries will be updated

```


Based on the characteristics of your server and the PostgreSQL version, you will then be offered to tune your settings. Press y to start the tuning process:


```
OutputTune memory/parallelism/WAL and other settings? [(y)es/(n)o]:  y
Recommendations based on 7.64 GB of available memory and 4 CPUs for PostgreSQL 11

Memory settings recommendations
Current:
shared_buffers = 128MB
#effective_cache_size = 4GB
#maintenance_work_mem = 64MB
#work_mem = 4MB
Recommended:
shared_buffers = 1955MB
effective_cache_size = 5865MB
maintenance_work_mem = 1001121kB
work_mem = 5005kB
Is this okay? [(y)es/(s)kip/(q)uit]:

```


timescaledb-tune will automatically detect the server’s available memory and calculate recommended values for the shared_buffers, effective_cache_size, maintenance_work_mem, and work_mem settings. If you’d like to learn more about how this is done, check out the GitHub page for timescaledb-tune.


If these settings look OK, enter y:


```
Output...
Is this okay? [(y)es/(s)kip/(q)uit]:  y
success: memory settings will be updated

```


At this point, if your server has multiple CPUs, you will find the recommendations for parallelism settings. However if you have one CPU, timescaledb-tune will send you directly to the WAL settings.


Those with multiple CPUs will encounter recommendations like this:


```
OutputParallelism settings recommendations
Current:
missing: timescaledb.max_background_workers
#max_worker_processes = 8
#max_parallel_workers_per_gather = 2
#max_parallel_workers = 8
Recommended:
timescaledb.max_background_workers = 8
max_worker_processes = 15
max_parallel_workers_per_gather = 2
max_parallel_workers = 4
Is this okay? [(y)es/(s)kip/(q)uit]:

```


These settings regulate the number of workers that process requests and background tasks. You can learn more about these settings from the TimescaleDB and PostgreSQL documentation.


Type y then ENTER to accept these settings:


```
Output...
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: parallelism settings will be updated

```


Next, you will find recommendations for Write Ahead Log (WAL):


```
OutputWAL settings recommendations
Current:
#wal_buffers = -1
#min_wal_size = 80MB
#max_wal_size = 1GB
Recommended:
wal_buffers = 16MB
min_wal_size = 4GB
max_wal_size = 8GB
Is this okay? [(y)es/(s)kip/(q)uit]:

```


WAL preserves data integrity, but the default settings can cause inefficient I/O that slows down write performance. Type and enter y to optimize these settings:


```
Output...
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: WAL settings will be updated

```


You’ll now find some miscellaneous recommendations:


```
OutputMiscellaneous settings recommendations
Current:
#default_statistics_target = 100
#random_page_cost = 4.0
#checkpoint_completion_target = 0.5
#max_locks_per_transaction = 64
#autovacuum_max_workers = 3
#autovacuum_naptime = 1min
#effective_io_concurrency = 1
Recommended:
default_statistics_target = 500
random_page_cost = 1.1
checkpoint_completion_target = 0.9
max_locks_per_transaction = 64
autovacuum_max_workers = 10
autovacuum_naptime = 10
effective_io_concurrency = 200
Is this okay? [(y)es/(s)kip/(q)uit]:

```


All of these different parameters are aimed at increasing performance. For example, SSDs can process many concurrent requests, so the best value for the effective_io_concurrency might be in the hundreds. You can find more info about these options in the PostgreSQL documentation.


Press y then ENTER to continue.


```
Output...
Is this okay? [(y)es/(s)kip/(q)uit]: y
success: miscellaneous settings will be updated
Saving changes to: /var/lib/pgsql/11/data/postgresql.conf

```


As a result, you will get a ready-made configuration file at /var/lib/pgsql/11/data/postgresql.conf.



Note: If you are doing the installation from scratch, you could also run the initial command with the --quiet and --yes flags, which will automatically apply all the recommendations and will make changes to the postgresql.conf configuration file:
sudo timescaledb-tune --pg-config=/usr/pgsql-11/bin/pg_config --quiet --yes



In order for the configuration changes to take effect, you must restart the PostgreSQL service:


```
sudo systemctl restart postgresql-11.service


```


Now the database is running with optimal parameters and is ready to work with the time series data. In the next steps, you’ll try out working with this data: creating new databases and hypertables and performing operations.


# Step 3 — Creating a New Database and Hypertable


With your TimescaleDB setup optimized, you are ready to work with time series data. TimescaleDB is implemented as an extension of PostgreSQL, so operations with time series data are not much different from relational data operations. At the same time, the database allows you to freely combine data from time series and relational tables in the future.


First, you will create a new database and turn on the TimescaleDB extension for it. Log in to your PostgreSQL database:


```
sudo -u postgres psql


```


Now create a new database and connect to it. This tutorial will name the database timeseries:


```
CREATE DATABASE timeseries;
\c timeseries


```


You can find additional information about working with the PostgreSQL database in our How To Create, Remove & Manage Tables in PostgreSQL on a Cloud Server tutorial.


Finally, enable the TimescaleDB extension:


```
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;


```


You will see the following output:


```
OutputWARNING:
WELCOME TO
 _____ _                               _     ____________
|_   _(_)                             | |    |  _  \ ___ \
  | |  _ _ __ ___   ___  ___  ___ __ _| | ___| | | | |_/ /
  | | | |  _ ` _ \ / _ \/ __|/ __/ _` | |/ _ \ | | | ___ \
  | | | | | | | | |  __/\__ \ (_| (_| | |  __/ |/ /| |_/ /
  |_| |_|_| |_| |_|\___||___/\___\__,_|_|\___|___/ \____/
               Running version 1.5.1
For more information on TimescaleDB, please visit the following links:

 1. Getting started: https://docs.timescale.com/getting-started
 2. API reference documentation: https://docs.timescale.com/api
 3. How TimescaleDB is designed: https://docs.timescale.com/introduction/architecture

Note: TimescaleDB collects anonymous reports to better understand and assist our users.
For more information and how to disable, please see our docs https://docs.timescaledb.com/using-timescaledb/telemetry.

CREATE EXTENSION

```


The primary point of interaction with your timeseries data are hypertables, an abstraction of many individual tables holding the data, called chunks.


To create a hypertable, start with a regular SQL table and then convert it into a hypertable via the function create_hypertable.


Make a table that will store data for tracking temperature and humidity across a collection of devices over time:


```
CREATE TABLE conditions (
  time        TIMESTAMP WITH TIME ZONE NOT NULL,
  device_id   TEXT,
  temperature  NUMERIC,
  humidity     NUMERIC
);


```


This command will create a table called conditions with four columns. The first column will store the timestamp, which includes the time zone and cannot be empty. Next, you will use the time column to transform your table into a hypertable that is partitioned by time:


```
SELECT create_hypertable('conditions', 'time');


```


This command calls the create_hypertable() function, which creates a TimescaleDB hypertable from a PostgreSQL table, replacing the latter.


You will receive the following output:


```
Output    create_hypertable
-------------------------
 (1,public,conditions,t)
(1 row)

```


In this step, you created a new hypertable to store timeseries data. Now you can populate it with data by writing to the hypertable, then run through the process of deleting it.


# Step 4 — Writing and Deleting Data


In this step, you will insert data using standard SQL commands and import large sets of data from external sources. This will show you the relational database aspects of TimescaleDB.


First, try out the basic commands. Data can be inserted into the hypertable using the standard INSERT SQL command. Insert some sample temperature and humidity data for the theoretical device weather-pro-000000 using the following command:


```
INSERT INTO conditions(time, device_id, temperature, humidity)
  VALUES (NOW(), 'weather-pro-000000', 84.1, 84.1);


```


You will receive the following output:


```
OutputINSERT 0 1

```


You can also insert multiple rows of data at once. Try the following:


```
INSERT INTO conditions
  VALUES
    (NOW(), 'weather-pro-000002', 71.0, 51.0),
    (NOW(), 'weather-pro-000003', 70.5, 50.5),
    (NOW(), 'weather-pro-000004', 70.0, 50.2);


```


You will receive the following:


```
OutputINSERT 0 3

```


You can also specify that the INSERT command will return some or all of the inserted data using the RETURNING statement:


```
INSERT INTO conditions
  VALUES (NOW(), 'weather-pro-000002', 70.1, 50.1) RETURNING *;


```


You will see the following output:


```
Output             time              |     device_id      | temperature | humidity
-------------------------------+--------------------+-------------+----------
 2019-09-15 14:14:01.576651+00 | weather-pro-000002 |        70.1 |     50.1
(1 row)

```


If you want to delete data from the hypertable, use the standard DELETE SQL command. Run the following to delete whatever data has a temperature higher than 80 or a humidity higher than 50:


```
DELETE FROM conditions WHERE temperature > 80;
DELETE FROM conditions WHERE humidity > 50;


```


After the delete operation, it is recommended to use the VACUUM command, which will reclaim space still used by data that had been deleted.


```
VACUUM conditions;


```


You can find more info about VACUUM command in the PostgreSQL documentation.


These commands are fine for small-scale data entry, but since time series data often generates huge datasets from multiple devices simultaneously, it’s essential also to know how to insert hundreds or thousands of rows at a time. If you have prepared data from external sources in a structured form, for example in csv format, this task can be accomplished quickly.


To test this out, you will use a sample dataset that represents temperature and humidity data from a variety of locations. It was created by TimescaleDB developers to allow you to test out their database. You can check out more info about sample datasets in the TimescaleDB documentation.


Let’s see how you can import data from the weather_small sample dataset into your database. First, quit Postgresql:


```
\q


```


Then download the dataset and extract it:


```
cd /tmp
curl https://timescaledata.blob.core.windows.net/datasets/weather_small.tar.gz -o weather_small.tar.gz
tar -xvzf weather_small.tar.gz


```


Next, import the temperature and humidity data into your database:


```
sudo -u postgres psql -d timeseries -c "\COPY conditions FROM weather_small_conditions.csv CSV"


```


This connects to the timeseries database and executes the \COPY command that copies the data from the chosen file into the conditions hypertable. It will run for a few seconds.


When the data has been entered into your table, you will receive the following output:


```
OutputCOPY 1000000

```


In this step, you added data to the hypertable manually and in batches. Next, continue on to performing queries.


# Step 5 — Querying Data


Now that your table contains data, you can perform various queries to analyze it.


To get started, log in to the database:


```
sudo -u postgres psql -d timeseries


```


As mentioned before, to work with hypertables you can use standard SQL commands. For example, to show the last 10 entries from the conditions hypertable, run the following command:


```
SELECT * FROM conditions LIMIT 10;


```


You will see the following output:


```
Output          time          |     device_id      |    temperature     | humidity
------------------------+--------------------+--------------------+----------
 2016-11-15 12:00:00+00 | weather-pro-000000 |               39.9 |     49.9
 2016-11-15 12:00:00+00 | weather-pro-000001 |               32.4 |     49.8
 2016-11-15 12:00:00+00 | weather-pro-000002 | 39.800000000000004 |     50.2
 2016-11-15 12:00:00+00 | weather-pro-000003 | 36.800000000000004 |     49.8
 2016-11-15 12:00:00+00 | weather-pro-000004 |               71.8 |     50.1
 2016-11-15 12:00:00+00 | weather-pro-000005 |               71.8 |     49.9
 2016-11-15 12:00:00+00 | weather-pro-000006 |                 37 |     49.8
 2016-11-15 12:00:00+00 | weather-pro-000007 |                 72 |       50
 2016-11-15 12:00:00+00 | weather-pro-000008 |               31.3 |       50
 2016-11-15 12:00:00+00 | weather-pro-000009 |               84.4 |     87.8
(10 rows)

```


This command lets you see what data is in the database. Since the database contains a million records, you used LIMIT 10 to limit the output to 10 entries.


To see the most recent entries, sort the data array by time in descending order:


```
SELECT * FROM conditions ORDER BY time DESC LIMIT 20;


```


This will output the top 20 most recent entries.


You can also add a filter. For example, to see entries from the weather-pro-000000 device, run the following:


```
SELECT * FROM conditions WHERE device_id = 'weather-pro-000000' ORDER BY time DESC LIMIT 10;


```


In this case, you will see the 10 most recent temperature and humidity datapoints recorded by the weather-pro-000000 device.


In addition to standard SQL commands, TimescaleDB also provides a number of special functions that are useful for timeseries data analysis. For example, to find the median of the temperature values, you can use the following query with the percentile_cont function:


```
SELECT percentile_cont(0.5)
  WITHIN GROUP (ORDER BY temperature)
  FROM conditions
  WHERE device_id = 'weather-pro-000000';


```


You will see the following output:


```
Output percentile_cont
-----------------
            40.5
(1 row)


```


In this way, you’ll see the median temperature for the entire observation period where the weather-pro-00000 sensor is located.


To show the latest values from each of the sensors, you can use the last function:


```
select device_id, last(temperature, time)
  FROM conditions
  GROUP BY device_id;


```


In the output you will see a list of all the sensors and relevant latest values.


To get the first values use the first function.


The following example is more complex. It will show the hourly average, minimum, and maximum temperatures for the chosen sensor within the last 24 hours:


```
SELECT time_bucket('1 hour', time) "hour",
trunc(avg(temperature), 2) avg_temp,
trunc(min(temperature), 2) min_temp,
trunc(max(temperature), 2) max_temp
FROM conditions
WHERE device_id = 'weather-pro-000000'
GROUP BY "hour" ORDER BY "hour" DESC LIMIT 24;


```


Here you used the time_bucket function, which acts as a more powerful version of the PostgreSQL date_trunc function. As a result, you will see which periods of the day the temperature rises or decreases:


```
Output          hour          | avg_temp | min_temp | max_temp
------------------------+----------+----------+----------
 2016-11-16 21:00:00+00 |    42.00 |    42.00 |    42.00
 2016-11-16 20:00:00+00 |    41.92 |    41.69 |    42.00
 2016-11-16 19:00:00+00 |    41.07 |    40.59 |    41.59
 2016-11-16 18:00:00+00 |    40.11 |    39.79 |    40.59
 2016-11-16 17:00:00+00 |    39.46 |    38.99 |    39.79
 2016-11-16 16:00:00+00 |    38.54 |    38.19 |    38.99
 2016-11-16 15:00:00+00 |    37.56 |    37.09 |    38.09
 2016-11-16 14:00:00+00 |    36.62 |    36.39 |    37.09
 2016-11-16 13:00:00+00 |    35.59 |    34.79 |    36.29
 2016-11-16 12:00:00+00 |    34.59 |    34.19 |    34.79
 2016-11-16 11:00:00+00 |    33.94 |    33.49 |    34.19
 2016-11-16 10:00:00+00 |    33.27 |    32.79 |    33.39
 2016-11-16 09:00:00+00 |    33.37 |    32.69 |    34.09
 2016-11-16 08:00:00+00 |    34.94 |    34.19 |    35.49
 2016-11-16 07:00:00+00 |    36.12 |    35.49 |    36.69
 2016-11-16 06:00:00+00 |    37.02 |    36.69 |    37.49
 2016-11-16 05:00:00+00 |    38.05 |    37.49 |    38.39
 2016-11-16 04:00:00+00 |    38.71 |    38.39 |    39.19
 2016-11-16 03:00:00+00 |    39.72 |    39.19 |    40.19
 2016-11-16 02:00:00+00 |    40.67 |    40.29 |    40.99
 2016-11-16 01:00:00+00 |    41.63 |    40.99 |    42.00
 2016-11-16 00:00:00+00 |    42.00 |    42.00 |    42.00
 2016-11-15 23:00:00+00 |    42.00 |    42.00 |    42.00
 2016-11-15 22:00:00+00 |    42.00 |    42.00 |    42.00
(24 rows)

```


You can find more useful functions in the TimescaleDB documentation.


Now you know how to handle your data. Next, you will go through how to delete unnecessary data and how to compress data.


# Step 6 — Configuring Data Compression and Deletion


As data accumulates, it will take up more and more space on your hard drive. To save space, the latest version of TimescaleDB provides a data compression feature. This feature doesn’t require tweaking any file system settings, and can be used to quickly make your database more efficient. For more information on how this compression works, take a look at this Compression article from TimescaleDB.


First, enable the compression of your hypertable:


```
ALTER TABLE conditions SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'device_id'
);


```


You will receive the following data:


```
OutputNOTICE:  adding index _compressed_hypertable_2_device_id__ts_meta_sequence_num_idx ON _timescaledb_internal._compressed_hypertable_2 USING BTREE(device_id, _ts_meta_sequence_num)
ALTER TABLE

```



Note: You can also set up TimescaleDB to compress data over the specified time period. For example, you could run:
SELECT add_compress_chunks_policy('conditions', INTERVAL '7 days');


In this example, the data will be automatically compressed after a week.

You can see the statistics on the compressed data with the command:


```
SELECT *
FROM timescaledb_information.compressed_chunk_stats;


```


You will then see a list of chunks with their statuses: compression status and how much space is taken up by uncompressed and compressed data in bytes.


If you don’t have the need to store data for a long period of time, you can delete out-of-date data to free up even more space. There is a special drop_chunks function for this. It allows you to delete chunks with data older than the specified time:


```
SELECT drop_chunks(interval '24 hours', 'conditions');


```


This query will drop all chunks from the hypertable conditions that only include data older than a day ago.


You will receive the following output:


```
Output              drop_chunks
----------------------------------------
 _timescaledb_internal._hyper_1_2_chunk
(1 row)

```


To automatically delete old data, you can configure a cron task. See our tutorial to learn more about how to use cron to automate various system tasks.


Exit from the database:


```
\q


```


Next, edit your crontab with the following command, which should be run from the shell:


```
crontab -e


```


Now add the following line to the end of the file:


crontab
```
...

0 1 * * * /usr/bin/psql -h localhost -p 5432 -U postgres -d postgres -c "SELECT drop_chunks(interval '24 hours', 'conditions');" >/dev/null 2>&1

```


This job will delete obsolete data that is older than one day at 1:00 AM every day.


# Conclusion


You’ve now set up TimescaleDB on your CentOS server. You also tried out creating hypertables, inserting data into it, querying the data, compressing, and deleting unnecessary records. With these examples, you’ll be able to take advantage of TimescaleDB’s key benefits over traditional relational database management systems for storing time-series data, including:


- Higher data ingest rates
- Quicker query performance
- Time-oriented features

