# How to Configure PHP-FPM with NGINX

```Nginx``` ```Ubuntu``` ```UNIX/Linux```

PHP-FPM (FastCGI Process Manager) is an alternative to FastCGI implementation of PHP with some additional features useful for sites with high traffic. It is the preferred method of processing PHP pages with NGINX and is faster than traditional CGI based methods such as SUPHP or mod_php for running a PHP script. The main advantage of using PHP-FPM is that it uses a considerable amount of less memory and CPU as compared with any other methods of running PHP. The primary reason is that it demonizes PHP, thereby transforming it to a background process while providing a CLI script for managing PHP request.


# PHP-FPM NGINX Configuration Prerequisites


- You can open a SSH session to your Ubuntu 18.04 system using root or a sudo enabled user.
- You have already installed NGINX and PHP in your Ubuntu 18.04 system.

# NGINX PHP-FPM Configuration Steps


- Install PHP-FPM
- Configure PHP-FPM Pool
- Configure NGINX for PHP-FPM
- Test NGINX PHP-FPM Configuration

# 1. Install PHP-FPM


Nginx doesn’t know how to run a PHP script of its own. It needs a PHP module like PHP-FPM to efficiently manage PHP scripts. PHP-FPM, on the other hand, runs outside the NGINX environment by creating its own process. Therefore when a user requests a PHP page the nginx server will pass the request to PHP-FPM service using FastCGI. The installation of php-fpm in Ubuntu 18.04 depends on PHP and its version. Check the documentation of installed PHP before proceeding with installing FPM in your server. Assuming you have already installed the latest PHP 7.3, then you can install FPM using the following apt-get command.


```
# apt-get install php7.3-fpm

```


The FPM service will start automatically, once the installation is over. You can verify that using the following systemd command:


```
# systemctl status php7.3-fpm
● php7.3-fpm.service - The PHP 7.3 FastCGI Process Manager
   Loaded: loaded (/lib/systemd/system/php7.3-fpm.service; enabled; vendor preset: enabled)
   Active: active (running) since Sun 2019-02-17 06:29:31 UTC; 30s ago
     Docs: man:php-fpm7.3(8)
 Main PID: 32210 (php-fpm7.3)
   Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
    Tasks: 3 (limit: 1152)
   CGroup: /system.slice/php7.3-fpm.service
           ├─32210 php-fpm: master process (/etc/php/7.3/fpm/php-fpm.conf)
           ├─32235 php-fpm: pool www
           └─32236 php-fpm: pool www

```


# 2. Configure PHP-FPM Pool


The php-fpm service creates a default pool, the configuration (www.conf) for which can be found in /etc/php/7.3/fpm/pool.d folder. You can customize the default pool as per your requirements. But it is a standard practice to create separate pools to have better control over resource allocation to each FPM processes. Furthermore, segregating FPM pool will enable them to run independently by creating its own master process. That means each php application can be configured with its own cache settings using PHP-FPM. A change in one pool’s configuration does not require you to start or stop the rest of the FPM pools. Let us create an FPM pool for running a PHP application effectively through a separate user. To start with, create a new user who will have exclusive rights over this pool:


```
# groupadd wordpress_user
# useradd -g wordpress_user wordpress_user

```


Now navigate to the FPM configuration directory and create a configuration file using your favorite text editor like vi:


```
# cd /etc/php/7.3/fpm/pool.d
# vi wordpress_pool.conf
[wordpress_site]
user = wordpress_user
group = wordpress_user
listen = /var/run/php7.2-fpm-wordpress-site.sock
listen.owner = www-data
listen.group = www-data
php_admin_value[disable_functions] = exec,passthru,shell_exec,system
php_admin_flag[allow_url_fopen] = off
; Choose how the process manager will control the number of child processes. 
pm = dynamic 
pm.max_children = 75 
pm.start_servers = 10 
pm.min_spare_servers = 5 
pm.max_spare_servers = 20 
pm.process_idle_timeout = 10s

```


The above FPM configuration options and their values are described below.


- [wordpress_site]: The name of the pool and must be unique across all pool names.
- user and group: The user and group under which the pool will run.
- listen: The name of the socket file for this pool.
- listen.owner and listen.group: Must match to the user and group on which NGINX is running. In our case it is www-data.
- php_admin_value: Allows to set custom php configuration values.
- php_admin_flag: Allows to set PHP boolean flags.
- pm: The process manager settings and the value is Dynamic means the number of child processes are set dynamically based on the following directives.
- pm.max_children: The maximum number of children that can be alive at the same time.
- pm.start_servers: The number of children created on startup.
- pm.min_spare_servers: The minimum number of children in ‘idle’ state (waiting to process). If the number of idle processes is less than this number then some children will be created.
- pm.max_spare_servers: The maximum number of children in idle state (waiting to process). If the number of idle processes is greater than this number then some children will be killed.
- pm.process_idle_timeout: The desired maximum number of idle server processes. Used only when pm value is set to dynamic.
Apart from above settings, it is also possible to pass few system environmental variable to php-fpm service using something like env['PHP_FOO'] = $bar. For example, adding the following options in the above configuration file will set the hostname and temporary folder location to the PHP environment.

```
...
...
env[HOSTNAME] = $HOSTNAME
env[TMP] = /tmp
...
...

```


Also, the process managers settings in the above pool configuration file are set to dynamic. Choose a setting that best suits your requirement. The other configuration options for process manager are:-   Static: A fixed number of PHP processes will be maintained.


- ondemand: No children are created at startup. Children will be forked when new requests are received in the server.

Once you are done with creating the above configuration file, restart fpm service to apply new settings:


```
# systemctl start php7.3-fpm

```


The FPM pool will be created immediately to serve php pages. Remember, you can create a separate systemd service by specifying the above FPM configuration file thereby enabling you to start/stop this pool without affecting other pools.


# 3. Configure NGINX for PHP-FPM


Now create an NGINX server block that will make use of the above FPM pool. To do that, edit your NGINX configuration file and pass the path of pool’s socket file using the option fastcgi_pass inside location block for php.


```
server {
         listen       80;
         server_name  example.journaldev.com;
         root         /var/www/html/wordpress;

         access_log /var/log/nginx/example.journaldev.com-access.log;
         error_log  /var/log/nginx/example.journaldev.com-error.log error;
         index index.html index.htm index.php;

         location / {
                      try_files $uri $uri/ /index.php$is_args$args;
         }

         location ~ \.php$ {
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php7.2-fpm-wordpress-site.sock;
            fastcgi_index index.php;
            include fastcgi.conf;
    }
}

```


NGINX Server Block
Make sure the above configuration setting is syntactically correct and restart NGINX.


```
# nginx-t
# systemctl restart nginx

```


# 4. Test PHP-FPM NGINX Configuration


To test if the above NGINX configuration file is indeed using the newly created FPM pool, create a php info file inside the web root. I have used /var/www/html/wordpress as a web root in the above NGINX configuration file. Adjust this value according to your environment.


```
# cd /var/www/html/wordpress
# echo "<?php echo phpinfo();?>" > info.php

```


Once you are done with creating the PHP info page, point your favorite web browser to it. You will notice that the value of $_SERVER['USER'] and $_SERVER['HOME'] variable are pointing to wordpress_user and /home/wordpress_user respectively that we set in the FPM configuration file previously and thus confirms that the NGINX is serving the php pages using our desired FPM pool.


NGINX PHP-FPM Testing
# Summary


In this article, we learned how to install php-fpm and configure separate pools for different users and applications. We also learned how to configure an NGINX server block to connect to a PHP-FPM service. PHP-FPM provides reliability, security, scalability, and speed along with a lot of performance tuning options. You can now split the default PHP-FPM pool into multiple resource pools to serve different applications. This will not only enhance your server security but also enable you to allocate server resources optimally!


