# How To Configure Logging and Log Rotation in Nginx on an Ubuntu VPS

```Linux Basics``` ```Ubuntu``` ```Logging``` ```Nginx``` ```Server Optimization```

## Introduction


To save yourself some trouble with your web server, you can configure logging. Logging information on your server gives you access to the data that will help you troubleshoot and assess situations as they arise.


In this tutorial, you will examine Nginx’s logging capabilities and discover how to configure these tools to best serve your needs. You will use an Ubuntu 22.04 virtual private server as an example, but any modern distribution should function similarly.


# Prerequisites


To follow this tutorial, you will need:


- One Ubuntu 22.04 server with a non-root sudo-enabled user with a firewall. Follow our Initial Server Setup to get started.
- Nginx installed on the server. Follow our How to Install Nginx on Ubuntu 22.04 tutorial to get it installed.

With Nginx running on your Ubuntu 22.04 server, you’re ready to begin.


# Understanding the Error_log Directive


Nginx uses a few different directives to control system logging. The one included in the core module is called error_log.


## error_log Syntax


The error_log directive is used to handle logging general error messages. If you are familiar with Apache, this is very similar to Apache’s ErrorLog directive.


The error_log directive applies the following syntax:


/etc/nginx/nginx.conf
```
error_log log_file log_level

```


The log_file specifies the file where the logs will be written. The log_level specifies the lowest level of logging that you would like to record.


## Logging Levels


The error_log directive can be configured to log more or less information as required. The level of logging can be any one of the following:


- emerg: Emergency situations where the system is in an unusable state.
- alert: Severe situations where action is needed promptly.
- crit: Important problems that need to be addressed.
- error: An error has occurred and something was unsuccessful.
- warn: Something out of the ordinary happened, but is not a cause for concern.
- notice: Something normal, but worth noting what has happened.
- info: An informational message that might be nice to know.
- debug: Debugging information that can be useful to pinpoint where a problem is occurring.

The levels higher on the list are considered a higher priority. If you specify a level, the log captures that level, and any level higher than the specified level.


For example, if you specify error, the log will capture messages labeled error, crit, alert, and emerg.


An example of this directive in use is inside the main configuration file. Use your preferred text editor to access the following configuration file. This example uses nano:


```
sudo nano /etc/nginx/nginx.conf


```


Scroll down the file to the # Logging Settings section and notice the following directives:


/etc/nginx/nginx.conf
```
. . .
##
# Logging Settings
##

access_log /var/log/nginx/access.log;
error_log /var/log/nginx/error.log;
. . .

```


If you do not want the error_log to log anything, you must send the output into /dev/null:


/etc/nginx/nginx.conf
```
. . .
error_log /dev/null crit;
. . .

```


The other logging directive, access_log, will be discussed in the following section.


# Understanding HttpLogModule Logging Directives


While the error_log directive is part of the core module, the access_log directive is part of the HttpLogModule. This provides the ability to customize the logs.


There are a few other directives included with this module that assist in configuring custom logs.


## log_format Directive


The log_format directive is used to describe the format of a log entry using plain text and variables.


There is one format that comes predefined with Nginx called combined. This is a common format used by many servers.


The following is an example of the combined format if it was not defined internally and needed to be specified with the log_format directive:


/etc/nginx/nginx.conf
```
log_format combined '$remote_addr - $remote_user [$time_local]  '
		    '"$request" $status $body_bytes_sent '
		    '"$http_referer" "$http_user_agent"';

```


This definition spans multiple lines until it finds the semi-colon (;).


The lines beginning with a dollar sign ($) indicate variables, while the characters like -, [, and ] are interpreted literally.


The general syntax of the directive is:


/etc/nginx/nginx.conf
```
log_format format_name string_describing_formatting;

```


You can use variables supported by the core module to formulate your logging strings.


# Understanding the access_log Directive


The access_log directive uses similar syntax to the error_log directive, but is more flexible. It is used to configure custom logging.


The access_log directive uses the following syntax:


/etc/nginx/nginx.conf
```
access_log /path/to/log/location [ format_of_log buffer_size ];

```


The default value for access_log is the combined format mentioned in the log_format section. You can use any format defined by a log_format definition.


The buffer size is the maximum size of data that Nginx will hold before writing it all to the log. You can also specify compression of the log file by adding gzip into the definition:


/etc/nginx/nginx.conf
```
access_log /path/to/log/location format_of_log gzip;

```


Unlike the error_log directive, if you do not want logging, you can turn it off by updating it in the configuration file:


/etc/nginx/nginx.conf
```
. . .
##
# Logging Settings
##

access_log off;
error_log /var/log/nginx/error.log;

. . .

```


It is not necessary to write to /dev/null in this case.


# Managing a Log Rotation


As log files grow, it becomes necessary to manage the logging mechanisms to avoid filling up your disk space. Log rotation is the process of switching out log files and possibly archiving old files for a set amount of time.


Nginx does not provide tools to manage log files, but it does include mechanisms to assist with log rotation.


## Manual Log Rotation


To manually rotate your logs, you can create a script to rotate them. For example, move the current log to a new file for archiving. A common scheme is to name the most recent log file with a suffix of .0, and then name older files with .1, and so on:


```
mv /path/to/access.log /path/to/access.log.0


```


The command that actually rotates the logs is kill -USR1 /var/run/nginx.pid. This does not kill the Nginx process, but instead sends it a signal causing it to reload its log files. This will cause new requests to be logged to the refreshed log file:


```
kill -USR1 `cat /var/run/nginx.pid`


```


The /var/run/nginx.pid file is where Nginx stores the master process’s PID. It is specified at the top of the /etc/nginx/nginx.conf configuration file with the line that begins with pid:


```
sudo nano /etc/nginx/nginx.conf


```


/etc/nginx/nginx.conf
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;
...

```


After the rotation, execute sleep 1 to allow the process to complete the transfer. You can then zip the old files or do whatever post-rotation processes you like:


```
sleep 1
[ post-rotation processing of old log file ]


```


## Log Rotation with logrotate


The logrotate application is a program used to rotate logs. It is installed on Ubuntu by default, and Nginx on Ubuntu comes with a custom logrotate script.


Use your preferred text editor to access the rotation script. This example uses nano:


```
sudo nano /etc/logrotate.d/nginx


```


The first line of the file specifies the location that the subsequent lines will apply to. Keep this in mind if you switch the location of logging in the Nginx configuration files.


The rest of the file specifies that the logs will be rotated daily and that 52 older copies will be preserved.


Notice that the postrotate section contains a command similar to the manual rotation mechanisms previously employed:


/etc/logrotate.d/nginx
```
. . .
postrotate
	[ ! -f /var/run/nginx.pid ] || kill -USR1 `cat /var/run/nginx.pid`
endscript
. . .

```


This section tells Nginx to reload the log files once the rotation is complete.


# Conclusion


Proper log configuration and management can save you time and energy in the event of a problem with your server. Having access to the information that will help you diagnose a problem can be the difference between a trivial fix and a persistent headache.


It is important to keep an eye on server logs in order to maintain a functional site and ensure that you are not exposing sensitive information. This guide serves only as an introduction to your experience with logging. You can learn more general tips in our tutorial on How To Troubleshoot Common Nginx Errors.


