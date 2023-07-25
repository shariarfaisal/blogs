# How To Install and Secure Redis on CentOS 8

```Databases``` ```Redis``` ```NoSQL``` ```Firewall``` ```CentOS 8``` ```CentOS```

## Introduction


Redis is an open-source, in-memory key-value data store which excels at caching. A non-relational database, Redis is known for its flexibility, performance, scalability, and wide language support.


Redis was designed for use by trusted clients in a trusted environment, and has no robust security features of its own. Redis does, however, have a few security features like a basic unencrypted password as well as command renaming and disabling. This tutorial provides instructions on how to install Redis and configure these security features. It also covers a few other settings that can boost the security of a standalone Redis installation on CentOS 8.


Note that this guide does not address situations where the Redis server and the client applications are on different hosts or in different data centers. Installations where Redis traffic has to traverse an insecure or untrusted network will require a different set of configurations, such as setting up an SSL proxy or a VPN between the Redis machines.


# Prerequisites


To complete this tutorial, you will need a server running CentOS 8. This server should have a non-root user with administrative privileges and a firewall configured with firewalld. To set this up, follow our Initial Server Setup guide for CentOS 8.


# Step 1 — Installing and Starting Redis


You can install Redis with the DNF package manager. The following command will install Redis, its dependencies, and nano, a user-friendly text editor. You don’t have to install nano, but we’ll use it in examples throughout this guide:


```
sudo dnf install redis nano


```


This command will prompt you to confirm that you want to install the selected packages. Press y then ENTER to do so:


```
Output. . .

Total download size: 1.5 M
Installed size: 5.4 M
Is this ok [y/N]: y

```


Following this, there is one important configuration change to make in the Redis configuration file, which was generated automatically during the installation.


Open this file with your preferred text editor. Here we’ll use nano:


```
sudo nano /etc/redis/redis.conf


```


Inside the file, find the supervised directive. This directive allows you to declare an init system to manage Redis as a service, providing you with more control over its operation. The supervised directive is set to no by default. Since you are running CentOS, which uses the systemd init system, change this to systemd:


/etc/redis/redis.conf
```
. . .

# If you run Redis from upstart or systemd, Redis can interact with your
# supervision tree. Options:
#   supervised no      - no supervision interaction
#   supervised upstart - signal upstart by putting Redis into SIGSTOP mode
#   supervised systemd - signal systemd by writing READY=1 to $NOTIFY_SOCKET
#   supervised auto    - detect upstart or systemd method based on
#                        UPSTART_JOB or NOTIFY_SOCKET environment variables
# Note: these supervision methods only signal "process is ready."
#       They do not enable continuous liveness pings back to your supervisor.
supervised systemd

. . .

```


That’s the only change you need to make to the Redis configuration file at this point, so save and close it when you are finished. If you used nano to edit the file, do so by pressing CTRL + X, Y, then ENTER.


After editing the file, start the Redis service:


```
sudo systemctl start redis.service


```


If you’d like Redis to start on boot, you can enable it with the enable command:


```
sudo systemctl enable redis


```


Notice that this command doesn’t include the .service suffix after the unit file name. You can usually leave this suffix off of systemctl commands, as it’s typically implied when interacting with systemd.


You can check Redis’s status by running the following:


```
sudo systemctl status redis


```


```
Output● redis.service - Redis persistent key-value database
   Loaded: loaded (/usr/lib/systemd/system/redis.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/redis.service.d
           └─limit.conf
   Active: active (running) since Wed 2020-09-30 20:05:24 UTC; 13s ago
 Main PID: 13734 (redis-server)
    Tasks: 4 (limit: 11489)
   Memory: 6.6M
   CGroup: /system.slice/redis.service
           └─13734 /usr/bin/redis-server 127.0.0.1:6379

```


Once you’ve confirmed that Redis is indeed running, you can test its functionality with this command:


```
redis-cli ping


```


This should print PONG as the response:


```
OutputPONG

```


If this is the case, it means you now have Redis running on your server and you can begin configuring it to enhance its security.


# Step 2 — Configuring Redis and Securing it with a Firewall


An effective way to safeguard Redis is to secure the server it’s running on. You can do this by ensuring that Redis is bound only to either localhost or to a private IP address and also that the server has a firewall up and running.


However, if you chose to set up Redis using another tutorial, then you may have updated the configuration file to allow connections from anywhere. This is not as secure as binding to localhost or a private IP.


To remedy this, open the Redis configuration file again with your preferred text editor:


```
sudo nano /etc/redis.conf


```


Locate the line beginning with bind and make sure it’s uncommented:


/etc/redis.conf
```
. . .
bind 127.0.0.1

```


If you need to bind Redis to another IP address (as in cases where you will be accessing Redis from a separate host) we strongly encourage you to bind it to a private IP address. Binding to a public IP address increases the exposure of your Redis interface to outside parties:


/etc/redis.conf
```
. . .
bind your_private_ip

```


After confirming that the bind directive isn’t commented out, you can save and close the file.


If you’ve followed the prerequisite Initial Server Setup tutorial and installed firewalld on your server, and you do not plan to connect to Redis from another host, then you do not need to add any extra firewall rules for Redis. After all, any incoming traffic will be dropped by default unless explicitly allowed by the firewall rules. Since a default standalone installation of Redis server is listening only on the loopback interface (127.0.0.1 or localhost), there should be no concern for incoming traffic on its default port.


If, however, you do plan to access Redis from another host, you will need to make some changes to your firewalld configuration using the firewall-cmd command. Again, you should only allow access to your Redis server from your hosts by using their private IP addresses in order to limit the number of hosts your service is exposed to.


To begin, add a dedicated Redis zone to your firewalld policy:


```
sudo firewall-cmd --permanent --new-zone=redis


```


Then specify which port you’d like to have open. Redis uses port 6379 by default:


```
sudo firewall-cmd --permanent --zone=redis --add-port=6379/tcp


```


Next, specify any private IP addresses which should be allowed to pass through the firewall and access Redis:


```
sudo firewall-cmd --permanent --zone=redis --add-source=client_server_private_IP


```


After running those commands, reload the firewall to implement the new rules:


```
sudo firewall-cmd --reload


```


Under this configuration, when the firewall encounters a packet from your client’s IP address, it will apply the rules in the dedicated Redis zone to that connection. All other connections will be processed by the default public zone. The services in the default zone apply to every connection, not just those that don’t match explicitly, so you don’t need to add other services (e.g. SSH) to the Redis zone because those rules will be applied to that connection automatically.


Keep in mind that using any firewall tool will work, whether you use firewalld, ufw, or iptables. What’s important is that the firewall is up and running so that unknown individuals cannot access your server. In the next step, you will configure Redis to only be accessible with a strong password.


# Step 3 — Configuring a Redis Password


Configuring a Redis password enables one of its built-in security features — the auth command — which requires clients to authenticate before being allowed access to the database. Like the bind setting, the password is configured directly in Redis’s configuration file, /etc/redis.conf. Reopen that file:


```
sudo nano /etc/redis.conf


```


Scroll to the SECURITY section and look for a commented directive that reads:


/etc/redis.conf
```
. . .
# requirepass foobared

```


Uncomment it by removing the #, and change foobared to a very strong password of your choosing.



Note: Rather than make up a password yourself, you may use a tool like apg or pwgen to generate one. If you don’t want to install an application just to generate a password, though, you may use the command below. This command echoes a string value and pipes it into the following sha256sum command, which will display the string’s SHA256 checksum.
Be aware that entering this command as written will generate the same password every time. To create a unique password, change the string in quotes to any other word or phrase:
echo "digital-ocean" | sha256sum


Though the generated password will not be pronounceable, it will be very strong and long, which is exactly the type of password required for Redis. After copying and pasting the output of that command as the new value for requirepass, it should read:
/etc/redis.conf
. . .
requirepass password_copied_from_output

Alternatively, if you prefer a shorter password, you could instead use the output of the following command. Again, change the word in quotes so it will not generate the same password as this command:
echo "digital-ocean" | sha1sum



After setting the password, save and close the file then restart Redis:


```
sudo systemctl restart redis


```


To test that the password works, open the Redis client:


```
redis-cli


```


The following is a sequence of commands used to test whether the Redis password works. The first command tries to set a key to a value before authentication:


```
set key1 10


```


That won’t work as you have not yet authenticated, so Redis returns an error:


```
Output(error) NOAUTH Authentication required.

```


The following command authenticates with the password specified in the Redis configuration file:


```
auth your_redis_password


```


Redis will acknowledge that you have been authenticated:


```
OutputOK

```


After that, running the previous command again should be successful:


```
set key1 10


```


```
OutputOK

```


The get key1 command queries Redis for the value of the new key:


```
get key1


```


```
Output"10"

```


This last command exits redis-cli. You may also use exit:


```
quit


```


It should now be very difficult for unauthorized users to access your Redis installation. Be aware that if you’re already using the Redis client and then restart Redis, you’ll need to re-authenticate. Also, please note that without SSL or a VPN, the unencrypted password will still be visible to outside parties if you’re connecting to Redis remotely.


Next, this guide will go over renaming Redis commands to further protect Redis from malicious actors.


# Step 4 — Renaming Dangerous Commands


The other security feature built into Redis allows you to rename or completely disable certain commands that are considered dangerous. When run by unauthorized users, such commands can be used to reconfigure, destroy, or otherwise wipe your data. Some of the commands that are considered to be dangerous include:


- FLUSHDB
- FLUSHALL
- KEYS
- PEXPIRE
- DEL
- CONFIG
- SHUTDOWN
- BGREWRITEAOF
- BGSAVE
- SAVE
- SPOP
- SREM
- RENAME
- DEBUG

This is not a comprehensive list, but renaming or disabling all of the commands in this list can help to improve your data store’s security. Whether you should disable or rename a given command will depend on your specific needs. If you know you will never use a command that can be abused, then you may disable it. Otherwise, you should rename it instead.


Like the authentication password, renaming or disabling commands is configured in the SECURITY section of the /etc/redis.conf file. To enable or disable Redis commands, open the configuration file for editing one more time:


```
sudo nano /etc/redis.conf


```



NOTE: These are examples. You should choose to disable or rename the commands that make sense for you. You can learn more about Redis’s commands and determine how they might be misused at redis.io/commands.

To disable or kill a command, rename it to an empty string, like this:


/etc/redis.conf
```
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""

```


To rename a command, give it another name like in the examples below. Renamed commands should be difficult for others to guess, but easy for you to remember:


/etc/redis.conf
```
# It is also possible to completely kill a command by renaming it into
# an empty string:
#
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command DEBUG ""
rename-command SHUTDOWN SHUTDOWN_MENOT
rename-command CONFIG ASC12_CONFIG

```


Save your changes and close the file. Then apply the changes by restarting Redis:


```
sudo systemctl restart redis.service


```


To test your new commands, enter the Redis command line:


```
redis-cli


```


Authenticate yourself using the password you defined earlier:


```
auth your_redis_password


```


```
OutputOK

```


Assuming that you renamed the CONFIG command to ASC12_CONFIG, attempting to use the config command will fail:


```
config get requirepass


```


```
Output(error) ERR unknown command 'config'

```


Calling the renamed command instead will be successful. Note that Redis commands are not case-sensitive:


```
asc12_config get requirepass


```


```
Output1) "requirepass"
2) "your_redis_password"

```


Finally, you can exit from redis-cli:


```
exit


```



Warning: Regarding renaming commands, there’s a cautionary statement at the end of the SECURITY section in the /etc/redis.conf file, which reads:
/etc/redis.conf
. . .

# Please note that changing the name of commands that are logged into the
# AOF file or transmitted to slaves may cause problems.

. . .

This means if the renamed command is not in the AOF file, or if it is but the AOF file has not been transmitted to replicas, then there should be no problem. Keep that in mind as you’re renaming commands. The best time to rename a command is when you’re not using AOF persistence or right after installation (that is, before your Redis-using application has been deployed).
When you’re using AOF and dealing with Redis replication, consider this answer from the project’s GitHub issue page. The following is a reply to the author’s question:

The commands are logged to the AOF and replicated to the slave the same way they are sent, so if you try to replay the AOF on an instance that doesn’t have the same renaming, you may face inconsistencies as the command cannot be executed (same for slaves).

The best way to handle renaming in cases like that is to make sure that renamed commands are applied to the primary instance as well as every secondary instance in a Redis installation.

# Step 5 — Setting Data Directory Ownership and File Permissions


This step will go through a couple of ownership and permissions changes you may need to make to improve the security profile of your Redis installation. This involves making sure that only the user that needs to access Redis has permission to read its data. That user is, by default, the redis user.


You can verify this by grep-ing for the Redis data directory in a long listing of its parent directory. This command and its output are given below:


```
ls -l /var/lib | grep redis


```


```
Outputdrwxr-x---. 2 redis          redis            22 Sep 30 20:15 redis

```


This output indicates that the Redis data directory is owned by the redis user, with secondary access granted to the redis group. This ownership setting is secure as are the folder’s permissions which, using octal notation, are set to 750.


If your Redis data directory has insecure permissions (for example, it’s world-readable) you can ensure that only the Redis user and group have access to the folder and its contents by running the chmod command. The following example changes this folder’s the permissions setting to 770:


```
sudo chmod 770 /var/lib/redis


```


The other permission you may need to change is that of the Redis configuration file. By default, it has a file permission of 640 and is owned by root, with secondary ownership by the root group:


```
ls -l /etc/redis.conf


```


```
Output-rw-r-----. 1 redis root 62344 Sep 30 20:14 /etc/redis.conf

```


That permission (640) means the Redis configuration file is readable only by the redis user and the root group. Because the configuration file contains the unencrypted password you configured in Step 4, redis.conf should be owned by the redis user, with secondary ownership by the redis group. To set this, run the following command:


```
sudo chown redis:redis /etc/redis.conf


```


Then change the permissions so that only the owner of the file can read and write to it:


```
sudo chmod 600 /etc/redis.conf


```


You may verify the new ownership and permissions by running the previous ls commands again:


```
ls -l /var/lib | grep redis


```


```
Outputtotal 40
drwxrwx---. 2 redis          redis            22 Sep 30 20:15 redis

```


```
ls -l /etc/redis.conf


```


```
Outputtotal 40
-rw-------. 1 redis redis 62344 Sep 30 20:14 /etc/redis.conf

```


Finally, restart Redis to reflect these changes:


```
sudo systemctl restart redis


```


With that, your Redis installation has been secured.


# Conclusion


Keep in mind that once someone is logged in to your server, it’s very easy to circumvent the Redis-specific security features you’ve put in place. This is why the most important security feature covered in this tutorial is the firewall, as that prevents unknown users from logging into your server in the first place.


If you’re attempting to secure Redis communication across an untrusted network you’ll have to employ an SSL proxy, as recommended by Redis developers in the official Redis security guide.


