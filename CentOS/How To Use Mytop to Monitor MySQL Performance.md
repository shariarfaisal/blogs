# How To Use Mytop to Monitor MySQL Performance

```MySQL``` ```Monitoring``` ```Server Optimization``` ```CentOS```

## Introduction


Mytop is an open source, command line tool used for monitoring MySQL performance. It was inspired by the Linux system monitoring tool named top and is similar to it in look and feel. Mytop connects to a MySQL server and periodically runs the show processlist and show global status commands. It then summarizes the information in a useful format. Using mytop, we can monitor (in real-time) MySQL threads, queries, and uptime as well as see which user is running queries on which database, which are the slow queries, and more. All this information can be used to optimize the MySQL server performance.


In this tutorial, we will discuss how to install, configure, and use mytop.


# Prerequisites


Before you get started with this tutorial, you should have the following:


- CentOS 7 64-bit Droplet (works with CentOS 6 as well)
- Non-root user with sudo privileges. To setup a user of this type, follow the Initial Server Setup with CentOS 7 tutorial. All commands will be run as this user.
- MySQL server running on the Droplet. To install MySQL, please follow Step #2 of the How To Install Linux, Apache, MySQL, PHP (LAMP) stack on CentOS article.

# Step 1 — Installing Mytop


Let us install the packages required for mytop.


First, we need to install the EPEL (Extra Packages for Enterprise Linux) yum repository on the server. EPEL is a Fedora Special Interest Group that creates, maintains, and manages a high quality set of open source add-on software packages for Enterprise Linux. Run the following command to install and enable the EPEL repository on your server:


On CentOS 7:


```
sudo rpm -ivh http://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-5.noarch.rpm


```


On CentOS 6:


```
sudo rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm


```


Before proceeding, verify that the EPEL repo is enabled using:


```
sudo yum repolist


```


If enabled, you will see the following repo listed in the output:


```
epel/x86_64                                                            Extra Packages for Enterprise Linux 7 - x86_64

```


Next, let us protect the base packages from EPEL using the yum plugin protectbase.


```
sudo yum install yum-plugin-protectbase.noarch -y


```


The purpose of the protectbase plugin is to protect certain yum repositories from updates from other repositories. Packages in the protected repositories will not be updated or overridden by packages in non-protected repositories even if the non-protected repository has a later version.


Now we are ready to install mytop package. Run the following command to install it:


```
sudo yum install mytop -y


```


This will install the mytop package as well as all its dependencies, mostly perl modules.


# Step 2 — Configuring Mytop


Before using mytop, create a customized configuration file for mytop named .mytop. Run the command:


```
sudo nano /root/.mytop


```


and add the following content in the file and save and exit.


/root/.mytop
```
host=localhost
db=mysql
delay=5
port=3306
socket=
batchmode=0
color=1
idle=1

```


This configuration file will be used when you run mytop directly as root and when you run it with the sudo command in front of it as a non-root sudo user.


You can make changes to this configuration file depending on your needs. For example, the delay option specifies the amount of time in seconds between display refreshes. If you wish to refresh the mytop display every 3 seconds, you could edit the file  /root/.mytop using


```
sudo nano /root/.mytop


```


and change the following:


/root/.mytop
```
delay=3

```


The idle parameter specifies whether to allow idle (sleeping) threads to appear in the list in mytop display screen. The default is to show idle threads. If idle threads are omitted, the default sorting order is reversed so that the longest running queries appear at the top of the list. If you wish to do this, edit the /root/.mytop file and change the following:


/root/.mytop
```
idle=0

```


You can refer the manual pages of mytop for information on all the parameters in the configuration file — it contains a description of each parameter. To access the manual page, use the command:


```
man mytop


```


You can type q to quit the manual.


# Step 3 — Connecting to Mytop


In this section, we will discuss how to connect to mytop and use it to view MySQL queries.


Mytop requires credentials to access the database, which can be provided via a prompt, on the command line, or stored in the configuration file. For better security, we will use the --prompt option to mytop, which asks for the password each time.
Let us connect to mytop using:


```
sudo mytop --prompt


```


and enter the MySQL root password at the prompt. You can also use several command line arguments with the mytop command. Please refer the manual page for the complete list. For example, if you want to use a different mysql user such as sammy to connect to mytop, run the command:


```
sudo mytop -u sammy --prompt


```


To connect and monitor only a specific database, you can use the command:


```
sudo mytop -d databasename --prompt


```


To quit mytop and return to your shell prompt, type q.


# Step 4 — Viewing and Interpreting the Mytop Display


In this section, we will see how to interpret mytop display and the different features offered by the tool.


Once we connect to mytop using mytop --prompt we will be taken to the thread view. It will show something similar to:


```
Output of mytopMySQL on localhost (5.5.41-MariaDB)                    up 0+00:05:52 [01:33:15]
 Queries: 148  qps:    0 Slow:     0.0         Se/In/Up/De(%):    09/00/00/00 
             qps now:    2 Slow qps: 0.0  Threads:    6 (   5/   0) 67/00/00/00 
 Key Efficiency: 2.0%  Bps in/out:  14.7/320.7k   Now in/out: 192.5/731.8k

      Id      User         Host/IP         DB      Time    Cmd Query or State
       --      ----         -------         --      ----    --- ----------
        2      root       localhost      mysql         0  Query show full processlist                          
       16      root       localhost                    0  Sleep
       17      root       localhost     testdb         0  Query SELECT * FROM dept_emp
       18      root       localhost     testdb         0  Query SELECT * FROM dept_emp
       19      root       localhost     testdb         0  Query SELECT * FROM dept_emp
       20      root       localhost     testdb         0  Query SELECT * FROM dept_emp

```


You can get back to this view if you are in another view by typing t.


The above display screen is broken into two parts. The top four lines comprises the header which can be toggled on or off by pressing SHIFT-H. The header contains summary information about your MySQL server.


- 
The first line identifies the hostname of the server and the version of MySQL it is running. The right-hand side shows the uptime of the MySQL server process in days+hours:minutes:seconds format as well as the current time.

- 
The second line displays the total number of queries the server has processed (148 in our case), the average number of queries per second, the number of slow queries, and the percentage of Select, Insert, Update, and Delete queries.

- 
The third line shows real-time values since last mytop refresh. The normal refresh (delay) time for mytop is 5 seconds, so if 100 queries were run in the last 5 seconds since the refresh, then the qps now number would be 20. The first field is the number of queries per second (qps now:    2). The second value is the number of slow queries per second. The Threads:    6 (   5/   0) segment indicates there are total 6 connected threads, 5 are active (one is sleeping), and there are 0 threads in the thread cache. The last field in the third line shows the query percentages, like in the previous line, but since last mytop refresh.

- 
The fourth line displays key buffer efficiency (how often keys are read from the buffer rather than disk) and the number of bytes that MySQL has sent and received, both overall and in the last mytop cycle. Key Efficiency: 2.0% shows 2% of keys are read from the buffer, not from disk. Bps in/out:  14.7/320.7k shows that since startup, MySQL has averaged 14.7kbps of inbound traffic and 320.7kbps for outbound traffic. Now in/out shows the traffic again, but since last mytop refresh.


The second part of the display lists current MySQL threads, sorted according to their idle time (least idle first). You can reverse the sort order by pressing O if needed. The thread id, username, host from which the user is connecting, database to which the user is connected, number of seconds of idle time, the command the thread is executing (or the state of the thread), and first part of the query info are all displayed here. If the thread is in a Query state(i.e. Cmd displays Query) then the next column Query or State will show the first part of the query that is being run. If the command state is Sleep or Idle then the Query or State column will usually be blank. In our example output above, thread with id 2 is actually mytop running the show processlist query to collect information. The thread with id 16 is sleeping (not processing a query, but still connected). The thread with id 17 is running a SELECT query on testdb database.


Now that we have understood the basic display of mytop, we will see how to use it to collect more information on the MySQL threads and queries. Let us take a look at the following mytop display:


```
[secondary_output Output of mytop]
MySQL on localhost (5.5.41-MariaDB)                    up 0+00:13:10 [23:54:45]
 Queries: 2.8k   qps:    4 Slow:    51.0         Se/In/Up/De(%):    45/00/00/00 
             qps now:   17 Slow qps: 0.0  Threads:   52 (  51/   0) 96/00/00/00 
 Key Efficiency: 100.0%  Bps in/out: 215.4/ 7.6M   Now in/out:  2.0k/16.2M

      Id      User         Host/IP         DB      Time    Cmd Query or State
       --      ----         -------         --      ----    --- ----------
       34      root       localhost     testdb         0  Query show full processlist
     1241      root       localhost                    1  Sleep
     1242      root       localhost     testdb         1  Query SELECT * FROM dept_emp
     1243      root       localhost     testdb         1  Query SELECT * FROM dept_emp
     1244      root       localhost     testdb         1  Query SELECT * FROM dept_emp
     1245      root       localhost     testdb         1  Query SELECT * FROM dept_emp
     1246      root       localhost     testdb         1  Query SELECT * FROM dept_emp
     1247      root       localhost     testdb         1  Query SELECT * FROM dept_emp

```


In the mytop thread view (default view) shown above, the queries are truncated. To view the entire query, you can press F, and it will ask:


```
Full query for which thread id:

```


Enter the thread id for the query you want to see. For example, enter 1244. Then it will show the following:


```
Thread 1244 was executing following query:

SELECT * FROM dept_emp WHERE ...

-- paused. press any key to resume or (e) to explain --

```


We can type e to explain the query. This will explain the query that is being run so that we can figure out if the query is optimized. EXPLAIN is one of the most powerful tools for understanding and optimizing troublesome MySQL queries. For example:


```
EXPLAIN SELECT * FROM dept_emp:

*** row 1 ***
          table:  dept_emp
           type:  ALL
  possible_keys:  NULL
            key:  NULL
        key_len:  NULL
            ref:  NULL
           rows:  332289
          Extra:  NULL
-- paused. press any key to resume --

```


You can press any key to exit this mode or type t to go back to the default thread view.


Another useful view available in mytop is the command view. To access command view, type c. It will look similar to the following:


```
           Command      Total  Pct  |  Last  Pct
           -------      -----  ---  |  ----  ---
            select       1782  55%  |   100   8%
       show status        723  22%  |   533  45%
  show processlist        708  22%  |   532  45%
         change db          2   0%  |     0   0%
    show variables          1   0%  |     0   0%
       Compression          0   0%  |     0   0%


```


The Command column shows the type of command or query being run. The Total column stands for the total number of that type of command run since the server started, and the Pct column shows the same in percentage. On the other side of the vertical line we have the Last column which tells us the number of that type of command run since the last refresh of mytop. This information gives us insight into what the MySQL server is doing in the short term and the long term.


We have discussed some of the important and useful features of mytop in this tutorial. There are many others available. To view a complete list of options, you can press the ? key while mytop is running.


# Conclusion


You should now have a good understanding of how to use mytop to monitor your MySQL server. It is also a starting point to finding problem SQL queries and optimizing them, thus increasing the overall performance of the server. You can get more information on how to optimize MySQL queries and tables on your server in this tutorial.


