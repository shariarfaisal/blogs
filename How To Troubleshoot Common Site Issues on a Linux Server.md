# How To Troubleshoot Common Site Issues on a Linux Server

```Apache``` ```Linux Basics``` ```Nginx```

## Introduction


Everyone has problems with their web server or site at one time or another. Learning where to look when you come across a problem and which components are the likely culprits will help you fix these problems as quickly and robustly as possible.


In this guide, you’ll gain an understanding of how to troubleshoot these issues so that you can get your site back up and running.


# What Types of Problems are Typical?


The majority of problems that you’ll encounter when trying to get your site up and running fall into a predictable spectrum.


We will go over these in more depth in the sections below, but for now, here’s a checklist of items to look into:


- Is your web server installed?
- Is the web server running?
- Is the syntax of your web server configuration files correct?
- Are the ports you configured open (not blocked by a firewall)?
- Are your DNS settings directing you to the correct place?
- Does the document root point to the location of your files?
- Is your web server serving the correct index files?
- Are the permissions and ownership of the file and directory structures correct?
- Are you restricting access through your configuration files?
- If you have a database backend, is it running?
- Can your site connect to the database successfully?

These are some of the common problems that administrators come across when a site is not working correctly. The exact issue can usually be narrowed down by taking a look at the different components’ log files and by referencing the error pages shown in your browser.


Below, we’ll go through each of these scenarios so that you can make sure your services are configured correctly.


# Check the Logs


Before blindly trying to track down a problem, try to check the logs of your web server and any related components. These will usually be in /var/log in a subdirectory specific to the service.


For instance, if you have an Apache server running on an Ubuntu server, by default the logs will be kept in /var/log/apache2. Check the files in this directory to see what kind of error messages are being generated. If you have a database backend that is giving you trouble, that will likely keep its logs in /var/log as well.


Other things to check are whether the processes themselves leave you error messages when the services are started. If you attempt to visit a web page and get an error, the error page can contain clues too (although not as specific as the lines in the log files).


Use a search engine to try to find relevant information that can point you in the right direction. In many cases, it may be helpful to paste a snippet of your logs directly into a search engine to find other examples of the same issue. The steps below can help you troubleshoot further.


# Is your Web Server Installed?


The first thing you will probably need to serve your sites correctly is a web server. In some cases, your web pages may be being served directly by a Docker container or some other application, and you won’t actually need to install a dedicated web server, but most deployments will still include at least one.


Most people will have installed a server before getting to this point, but there are some situations where you may have actually accidentally uninstalled the server when performing other package operations.


If you are on an Ubuntu or Debian system and need to install the Apache web server, you can type:


```
sudo apt-get update
sudo apt-get install apache2


```


On these systems, the Apache process is called apache2.


If you are running Ubuntu or Debian and want the Nginx web server, you can instead type:


```
sudo apt-get update
sudo apt-get install nginx


```


On these systems, the Nginx process is called nginx.


If you are running RHEL, Rocky Linux, or Fedora and need to use the Apache web server, you can type:


```
sudo dnf install httpd


```


On these systems, the Apache process is called httpd.


If you are running RHEL, Rocky Linux, or Fedora and want to use Nginx, you can type this. Again, remove the “sudo” if you are logged in as root:


```
sudo dnf install nginx


```


On these systems, the Nginx process is called nginx. Unlike on Ubuntu, Nginx is not started automatically after being installed on these RPM-based distributions. Read on to learn how to start it.


# Is your Web Server Running?


Now that you are sure your server is installed, is it running?


There are plenty of ways of finding out if the service is running or not. One method that is fairly cross-platform is to use the netstat command.


Running netstat with the -plunt flags will tell you all of the processes that are using ports on the server. To learn more about running netstat, you can refer to How To Use Top, Netstat, Du, & Other Tools to Monitor Server Resources. You can then grep the output of netstat for the name of the process you are looking for:


```
sudo netstat -plunt | grep nginx


```


```
Outputtcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      15686/nginx: master
tcp6       0      0 :::80                   :::*                    LISTEN      15686/nginx: master

```


You can change nginx to the name of the web server process on your server. If you see a line like the one above, it means your process is up and running. If you don’t get any output back, it means you queried for the wrong process or that your web server is not running.


If your web server isn’t running, you can start it using your Linux distribution’s init system. Most software that’s designed to run in the background will register itself with the init system after being installed, so that you can start and stop it programmatically. Most distributions also now use the same init system, systemd, which provides the systemctl command.


For instance, you could start the nginx service by typing:


```
sudo systemctl start nginx


```


If your web server starts, you can check with netstat again to verify that everything is correct.


# Is the Syntax of your Web Server Configuration File Correct?


If your web server was unable to start, this is usually an indication that your configuration files need some attention. Both Apache and Nginx require strict adherence to their syntax in order for their configuration to be parsed correctly.


The configuration files for system services are usually located within a subdirectory of the /etc/ directory named after the process itself.


For example, you could get to the main configuration directory of Apache on Ubuntu by typing:


```
cd /etc/apache2


```


The Apache configuration directory on RHEL, Rocky, and Fedora also reflects the RHEL name for that process:


```
cd /etc/httpd


```


The configuration will be spread out among many different files. When trying and failing to start a service, it will usually produce errors pointing to the configuration file and the line where the problem was first found. You can start investigating that file.


Each of these web servers also provide a way of validating the configuration syntax of your files.


If you are using Apache, you can use the apache2ctl or apachectl command to check your configuration files for syntax errors:


```
apache2ctl configtest


```


```
OutputAH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1. Set the 'ServerName' directive globally to suppress this message
Syntax OK

```


Syntax OK essentially means that there are no major errors preventing the server from running, and every message printed prior to that is a minor error or a warning. In this case, the Could not reliably determine the server's fully qualified domain name reflects an out-of-the-box Apache setup on a server that has not yet been configured with a domain name, but which should still be accessible by its IP address.


If you have an Nginx web server, you can run a similar test by typing:


```
sudo nginx -t


```


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If you remove a semicolon at the end of a configuration line in /etc/nginx/nginx.conf (a common error for Nginx configurations), you would get a message like this:


```
sudo nginx -t 


```


```
Outputnginx: [emerg] invalid number of arguments in "tcp_nopush" directive in /etc/nginx/nginx.conf:18
nginx: configuration file /etc/nginx/nginx.conf test failed

```


There is an invalid number of arguments because Nginx looks for a semicolon to end statements. If it doesn’t find one, it drops down to the next line and interprets that as further arguments for the last line.


You can run these tests in order to find syntax problems in your files. Fix the problems that it references until you can get the files to pass the test.


# Are the Ports you Configured Open?


Generally, web servers run on port 80 for HTTP web traffic and use port 443 for HTTPS traffic encrypted with TLS/SSL. In order for users to reach your site, these ports must be accessible.


You can test whether your server has its port open by running netcat from your local machine.


You’ll need to use your remote server’s IP address and tell it what port to check, like this:


```
nc -z 111.111.111.111 80


```


This will check whether port 80 is open on the server at 111.111.111.111. If it is open, the command will return right away. If it is not open, the command will continuously try to form a connection, unsuccessfully. You can stop this process by pressing CTRL+C in the terminal window.


If your web ports are not accessible, you should look at your firewall configuration. You may need to open up port 80 or port 443.


# Are your DNS Settings Directing you to the Correct Place?


If you can reach your site by its IP address, but not through a domain name that you’ve set up, you may need to take a look at your DNS settings.


In order for visitors to reach your site through its domain name, you should have an “A” or “AAAA” record pointing to your server’s IP address in the DNS settings. You can query for your domain’s “A” record by using the host command on a local or:


```
host -t A example.com


```


```
Outputexample.com has address 93.184.216.119

```


The line that is returned to you should match the IP address of your server. If you need to check an “AAAA” record (for IPv6 connections), you can type:


```
host -t AAAA example.com


```


```
Outputexample.com has IPv6 address 2606:2800:220:6d:26bf:1447:1097:aa7

```


Keep in mind that any changes you make to your DNS records can take some time to propagate, depending on your domain name registrar. It can sometimes be helpful to use a site like ​​https://www.whatsmydns.net/ to check when your DNS changes have come into effect globally (usually half an hour or so). You may receive inconsistent results to these queries after a change since your request will often hit different servers that are not all up-to-date yet.


If you are using DigitalOcean, you can learn how to configure DNS settings for your domain here.


## Make sure your Configuration Files Also Handle your Domain Correctly


If your DNS settings are correct, you may also want to check your Apache virtual host files or the Nginx server block files to make sure they are configured to respond to requests for your domain.


In Apache, your virtual host file might look like this:


/etc/apache2/sites-enabled/000-default.conf
```
&lt;VirtualHost *:80&gt;
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html
. . .

```


This virtual host is configured to respond to any requests on port 80 for the domain example.com.


A similar chunk in Nginx might look something like this:


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .

```


These two blocks are configured to respond to the same default types of requests.


# Does the Document Root Point to the Location of your Files?


Another consideration is whether your web server is pointed at the correct file location.


Each virtual server in Apache or server block in Nginx is configured to point to a specific port or local directory. If this is configured incorrectly, the server will throw an error message when you try to access the page.


In Apache, the document root is configured through the DocumentRoot directive:


/etc/apache2/sites-enabled/default
```
&lt;VirtualHost *:80&gt;
    ServerName example.com
    ServerAlias www.example.com
    ServerAdmin admin@example.com
    DocumentRoot /var/www/html
. . .

```


This line tells Apache that it should look for the files for this domain in the /var/www/html directory. If your files are kept elsewhere, you’ll have to modify this line to point to the correct location.


In Nginx, the root directive configures the same thing:


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .

```


In this configuration, Nginx looks for files for this domain in the /usr/share/nginx/html directory.


# Is your Web Server Serving the Correct Index Files?


If your document root is correct and your index pages are not being served correctly when you go to your site or a directory location on your site, you may have your indexes configured incorrectly.


Depending on the complexity of your web applications, many web servers will still default to serving index files. This is usually an index.html file or an index.php file depending on your configuration.


In Apache, you may find a line in your virtual host file that configures the index order that will be used for specific directories explicitly, like this:


/etc/apache2/sites-enabled/default
```
&lt;Directory /var/www/html&gt;
    DirectoryIndex index.html index.php
&lt;/Directory&gt;

```


This means that when the directory is being served, Apache will look for a file called index.html first, and try to serve index.php as a backup if the first file is not found.


You can set the order that will be used to serve index files for the entire server by editing the mods-enabled/dir.conf file, which will set the defaults for the server. If your server is not serving an index file, make sure you have an index file in your directory that matches one of the options in your file.


In Nginx, the directive that does this is is called index and it is used like this:


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;
    root /usr/share/nginx/html;
    index index.html index.htm;
    server_name example.com www.example.com;
. . .

```


# Are the Permissions and Ownership Set Correctly?


In order for the web server to correctly serve files, it must be able to read the files and have access to the directories where they are kept. This can be controlled through the file and directory permissions and ownership.


To read files, the directories containing the content must be readable and executable by the user account associated with the web server. On Ubuntu and Debian, Apache and Nginx run as the user www-data which is a member of the www-data group.


On RHEL, Rocky, and Fedora, Apache runs under a user called apache which belongs to the apache group. Nginx runs under a user called nginx which is a part of the nginx group.


With this in mind, you can look at the files and folders that you are hosting:


```
ls -l /path/to/web/root


```


The directories should be readable and executable by the web user or group, and the files should be readable in order to read content. In order to upload, write or modify content, the directories must additionally be writeable and the files need to be writable as well.


To modify the ownership of a file, you can use chown:


```
sudo chown user_owner:group_owner /path/to/file


```


This can also be done to a directory. You can change the ownership of a directory and all of the files under it by passing the -R flag:


```
sudo chown -R user_owner:group_owner /path/to/file


```


To learn more about permissions, refer to An Introduction to Linux Permissions.


# Are you Restricting Access through your Configuration Files?


Your web server settings may also be configured to deny access from the files you are trying to serve.


In Apache, this would be configured in the virtual host file for that site, or through an .htaccess file located in the directory itself.


Within these files, it is possible to restrict access in a few different ways. Directories can be restricted like this in Apache 2.4+:


/etc/apache2/sites-enabled/default
```
&lt;Directory /usr/share&gt;
    AllowOverride None
    Require all denied
&lt;/Directory&gt;

```


In Nginx, these restrictions will take the form of deny directives and will be located in your server blocks or main config files:


/etc/nginx/sites-enabled/default
```
location /usr/share {
    deny all;
}

```


# If you have a Database Backend, is it Running?


If your site relies on a database backend like MySQL, PostgresSQL, MongoDB, etc. you need to make sure that it is up and available.


You can do that in the same way that you checked that the web server was running. Again, you can search through running processes with netstat and grep:


```
sudo netstat -plunt | grep mysql


```


```
Outputtcp        0      0 127.0.0.1:3306        0.0.0.0:*         LISTEN      3356/mysqld

```


As you can see, the service is running on this machine. Make sure you know the name that your service runs under when searching for it.


An alternative is to search for the port that your service runs on. Look at the documentation for your database to find the default port that it runs on (MySQL defaults to 3356), or check your configuration files.


# If you have a Database Backend, can your Site Connect Successfully?


The next step to take if you are troubleshooting an issue with a database backend is to see if you can connect correctly. This usually means checking the files that your site reads to find out the database information.


For instance, for a WordPress site, the database connection settings are stored in a file called wp-config.php. You need to check that the DB_NAME, DB_USER, and DB_PASSWORD are correct in order for your site to connect to the database.


You can test whether the file has the correct information by trying to connect to the database manually on the command line. Most databases will support similar syntax to MySQL:


```
mysql -u DB_USER_value -pDB_PASSWORD_value DB_NAME_value


```


If you cannot connect using the values you found in the file, you may need to modify the access permissions of your databases.


# If all else Fails, Check the Logs Again


Checking the logs should actually be your first step, but it is also a good last step prior to asking for more help.


If you have reached the end of your ability to troubleshoot on your own and need some help, you will get more relevant help faster by providing log files and error messages. Experienced administrators will probably have a good idea of what is happening if you give them the pieces of information that they need.


# Conclusion


Hopefully these troubleshooting tips will have helped you track down and fix some of the more common issues that administrators face when trying to get their sites up and running.


If you have any additional tips for things to check and ways of problem solving, please share them with other users in the comments.


