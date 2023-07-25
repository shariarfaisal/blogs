# How To Upgrade Nginx In-Place Without Dropping Client Connections

```Nginx```

## Introduction


Nginx is a powerful web server and reverse proxy that is used to serve many of the most popular sites in the world. In this guide, we’ll demonstrate how to upgrade the Nginx executable in place, without losing client connections.


# Prerequisites


Before beginning this guide, you should have a non-root user on your server, configured with sudo privileges. You will also need to have Nginx installed.


You can follow our initial server setup guide for Ubuntu 22.04  and then install Nginx on that server.


# How the Upgrade Works


Nginx works by spawning a master process when the service starts. The master service, in turn, spawns one or more worker processes that handle the actual client connections. Nginx is designed to perform certain actions when it receives specific low-level signals from the system. Using these signals provides you with the opportunity to upgrade Nginx or its configuration in-place, without losing client connections.


Nginx’s provided installation and upgrade scripts are designed to send these signals when starting, stopping, and restarting Nginx. However, sending these signals manually allows you to audit the upgrade and revert quickly if there are problems. This will also provide an option to upgrade gracefully if you have installed Nginx from source or are not relying on your package manager to configure the service.


The following signals will be used:


- USR2: This spawns a new set of master/worker processes without affecting the old set.
- WINCH: This tells the Nginx master process to gracefully stop its associated worker instances.
- HUP: This tells an Nginx master process to re-read its configuration files and replace worker processes with those adhering to the new configuration. If an old and new master are running, sending this to the old master will spawn workers using their original configuration.
- QUIT: This shuts down a master and its workers gracefully.
- TERM: This initiates a fast shutdown of the master and its workers.
- KILL: This immediately kills a master and its workers without any cleanup.

# Finding Nginx Process PIDs


In order to send signals to the various processes, we need to know the PID for the target process. There are two ways to find this.


First, you can use the ps utility and then grep for Nginx among the results. This allows you to see the master and worker processes:


```
ps aux | grep nginx


```


```
outputroot       16653  0.0  0.2 119160  2172 ?        Ss   21:48   0:00 nginx: master process /usr/sbin/nginx
nginx      16654  0.0  0.9 151820  8156 ?        S    21:48   0:00 nginx: worker process
sammy      16688  0.0  0.1 221928  1164 pts/0    S+   21:48   0:00 grep --color=auto nginx

```


The second, highlighted column contains the PIDs for the selected processes. The last column clarifies that the first result is an Nginx master process.


Another way to find the PID for the master Nginx process is to print out the contents of the /run/nginx.pid file:


```
cat /run/nginx.pid


```


```
output16653

```


If there are two Nginx master processes running, the old one will be moved to /run/nginx.pid.oldbin.


# Spawn a New Nginx Master/Workers Set


The first step to gracefully updating is to actually update your Nginx package and/or binaries. Do this using whatever method is appropriate for your Nginx installation, whether through a package manager or a source installation.


After the new binary is in place, you can spawn a second set of master/worker processes that use the new executable.


You can do this by sending the USR2 signal directly to the PID number you queried (make sure to substitute the PID of your own Nginx master process here):


```
sudo kill -s USR2 16653


```


Or, you can read and substitute the value stored in your PID file directly into the command, like this:


```
sudo kill -s USR2 `cat /run/nginx.pid`


```


If you check your running processes, you will see that you now have two sets of Nginx masters and workers:


```
ps aux | grep nginx


```


```
outputroot       16653  0.0  0.2 119160  2172 ?        Ss   21:48   0:00 nginx: master process /usr/sbin/nginx
nginx      16654  0.0  0.9 151820  8156 ?        S    21:48   0:00 nginx: worker process
root       16699  0.0  1.5 119164 12732 ?        S    21:54   0:00 nginx: master process /usr/sbin/nginx
nginx      16700  0.0  0.9 151804  8008 ?        S    21:54   0:00 nginx: worker process
sammy      16726  0.0  0.1 221928  1148 pts/0    R+   21:55   0:00 grep --color=auto nginx

```


You can also see that the original /run/nginx.pid file has been moved to /run/nginx.pid.oldbin and the newer master process’s PID has been written to /run/nginx.pid:


```
tail -n +1 /run/nginx.pid*


```


```
output==> /run/nginx.pid <==
16699

==> /run/nginx.pid.oldbin <==
16653

```


You can now send signals to either of the master processes using the PIDs contained in these files.


At this point, both master/worker sets are operational and capable of serving client requests. The first set is using the original Nginx executable and configuration and the second set is using the newer versions. They can continue to operate side-by-side, but for consistency, we should start to transition to the new set.


# Shut Down the First Master’s Workers


In order to begin the transition to the new set, the first thing to do is stop the original master’s worker processes. The original workers will finish up handling all of their current connections and then exit.


Stop the original set’s workers by issuing the WINCH signal to their master process:


```
sudo kill -s WINCH `cat /run/nginx.pid.oldbin`


```


This will let the new master’s workers handle new client connections alone. The old master process will still be running, but with no workers:


```
ps aux | grep nginx


```


```
outputroot       16653  0.0  0.2 119160  2172 ?        Ss   21:48   0:00 nginx: master process /usr/sbin/nginx
root       16699  0.0  1.5 119164 12732 ?        S    21:54   0:00 nginx: master process /usr/sbin/nginx
nginx      16700  0.0  0.9 151804  8008 ?        S    21:54   0:00 nginx: worker process
sammy      16755  0.0  0.1 221928  1196 pts/0    R+   21:56   0:00 grep --color=auto nginx

```


This lets you audit the new workers as they accept connections in isolation.


# Evaluate the Outcome and Take the Next Steps


You should test and audit your system at this point to make sure that there are no signs of problems. You can leave your configuration in this state for as long as you wish to ensure that the new Nginx executable is bug-free and able to handle your traffic.


Your next step will depend entirely on whether you encounter problems.


## If Your Upgrade was Successful


If you experienced no issues with your new set’s workers, you can safely shut down the old master process. To do this, send the old master the QUIT signal:


```
sudo kill -s QUIT `cat /run/nginx.pid.oldbin`


```


The old master process will exit gracefully, leaving only your new set of Nginx master/workers. At this point, you’ve successfully performed an in-place binary update of Nginx without interrupting client connections.


## If Your Upgrade was Unsuccessful


If your new set of workers seem to be having problems, you can transition back to the old configuration and binary. This is possible as long as you have not QUIT the older master process.


The best way to do this is to restart your old master’s workers by sending it the HUP signal. Usually, when you send an Nginx master the HUP signal, it will re-read its configuration files and start new workers. However, when the target is an older master, it will spawn new workers using its original, working configuration:


```
sudo kill -s HUP `cat /run/nginx.pid.oldbin`


```


You now should be back to having two sets of master/worker processes:


```
ps aux | grep nginx


```


```
outputroot       16653  0.0  0.2 119160  2172 ?        Ss   21:48   0:00 nginx: master process /usr/sbin/nginx
nginx      16654  0.0  0.9 151820  8156 ?        S    21:48   0:00 nginx: worker process
root       16699  0.0  1.5 119164 12732 ?        S    21:54   0:00 nginx: master process /usr/sbin/nginx
nginx      16700  0.0  0.9 151804  8008 ?        S    21:54   0:00 nginx: worker process
sammy      16726  0.0  0.1 221928  1148 pts/0    R+   21:55   0:00 grep --color=auto nginx

```


The newest workers are associated with the old master. Both worker sets will be accepting client connections at this point. Now, stop the newer, buggy master process and its workers by sending the QUIT signal:


```
sudo kill -s QUIT `cat /run/nginx.pid`


```


You should be back to your old master and workers:


```
ps aux | grep nginx


```


```
outputroot       16653  0.0  0.2 119160  2172 ?        Ss   21:48   0:00 nginx: master process /usr/sbin/nginx
nginx      16654  0.0  0.9 151820  8156 ?        S    21:48   0:00 nginx: worker process
sammy      16688  0.0  0.1 221928  1164 pts/0    S+   21:48   0:00 grep --color=auto nginx

```


The original master will regain the /run/nginx.pid file for its PID.


If this does not work for any reason, you can try just sending the new master server the TERM signal, which should initiate a shutdown. This should stop the new master and any workers while automatically kicking over the old master to start its worker processes. If there are serious problems and the buggy workers are not exiting, you can send each of them a KILL signal to clean up. This should be a last resort, however, as it will cut off connections.


After transitioning back to the old binary, remember that you still have the new version installed on your system. You should remove the buggy version and roll back to your previous version so that Nginx will run without issues on reboot.


# Conclusion


By now, you should be able to seamlessly transition your machines from one Nginx binary to another. Nginx’s ability to handle two master/workers sets while maintaining information about their relationships provides us with the ability to upgrade server software without taking the server machines offline.


