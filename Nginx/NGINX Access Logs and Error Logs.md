# NGINX Access Logs and Error Logs

```Nginx```

Logs are very useful to monitor activities of any application apart from providing you with valuable information while you troubleshoot it. Like any other application, NGINX also records events like visitors to your site, issues it encountered and more to log files. This information enables you to take preemptive measures in case you notice some serious discrepancies in the log events. This article will guide you in details about how to configure NGINX logging so that you have a better insight into its activities. 


# Prerequisite


- You have already installed NGINX by following our tutorial from here.

# Logs in NGINX


By default, NGINX writes its events in two types of logs - the error log and the access log. In most of the popular Linux distro like Ubuntu, CentOS or Debian, both the access and error log can be found in /var/log/nginx, assuming you have already enabled the access and error logs in the core NGINX configuration file. Let us find out more about NGINX access log, error log and how to enable them if you have not done it earlier.


# What is NGINX access log?


The NGINX logs the activities of all the visitors to your site in the access logs. Here you can find which files are accessed, how NGINX responded to a request, what browser a client is using, IP address of clients and more. It is possible to use the information from the access log to analyze the traffic to find sites usages over time. Further, by monitoring the access logs properly, one can find out if a user is sending some unusual request for finding flaws in the deployed web application.


# What is NGINX error log?


On the other hand, if NGINX faces any glitches then it will record the event to the error log. This may happen if there is some error in the configuration file. Therefore if NGINX is unable to start or abruptly stopped running then you should check the error logs to find more details. You may also find few warnings in the error log but it does not indicate that a problem has occurred but the event may pose a serious issue in the near future.


# How to enable NGINX access log?


In general, the access log can be enabled with access_log directive either in http or in server section. The first argument log_file is mandatory whereas the second argument log_format is optional. If you don’t specify any format then logs will be written in default combined format.


```
access_log log_file log_format;

```


The access log is enabled by default in the http context of core NGINX configuration file. That means access log of all the virtual host will be recorded in the same file.


```
http {
      ...
      ...
      access_log  /var/log/nginx/access.log;
      ...
      ...
}

```


It is always better to segregate the access logs of all the virtual hosts by recording them in a separate file. To do that, you need to override the access_log directive that is defined in the http section with another access_log directive in the server context.


```

http {
      ...
      ...
      access_log  /var/log/nginx/access.log;
    
         server {
                  listen 80; 
                  server_name domain1.com
                  access_log  /var/log/nginx/domain1.access.log;
                  ...
                  ...
                }
}

```


Reload NGINX to apply the new settings. To view the access logs for the domain domain1.com in the file /var/log/nginx/domain1.access.log, use the following tail command in the terminal.


```
# tail -f /var/log/nginx/domain1.access.log

```


# Apply Custom Format in Access Log


The default log format used to record an event in the access log is combined log format. You can override the default behavior by creating your own custom log format and then specify the name of the custom format in the access_log directive. The following example defines a custom log format by extending the predefined combined format with the value of gzip compression ratio of the response. The format is then applied by indicating the log format with the access_log directive.


```
http {
            log_format custom '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" "$gzip_ratio"';

            server {
                    gzip on;
                    ...
                    access_log /var/log/nginx/domain1.access.log custom;
                    ...
            }
}

```


Once you have applied above log format in your environment, reload NGINX. Now tail the access log to find the gzip ratio at the end of the log event.


```
# tail -f /var/log/nginx/domain1.access.log
47.29.201.179 - - [28/Feb/2019:13:17:10 +0000] "GET /?p=1 HTTP/2.0" 200 5316 "https://domain1.com/?p=1" "Mozilla/5.0 (Windows NT 6.1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/72.0.3626.119 Safari/537.36" "2.75"

```


# How to enable NGINX error log?


The error_log directive sets up error logging to file or stderr, or syslog by specifying minimal severity level of error messages to be logged. The syntax of error_log directive is:


```
error_log log_file log_level;

```


The first argument log_file defines the path of the log file and the second argument log_level defines the severity level of the log event to be recorded. If you don’t specify the log_level then by default, only log events with a severity level of error are recorded. For example, the following example sets the severity level of error messages to be logged to crit. Further, the error_log directive in the http context implies that the error log for all the virtual host will be available in a single file.


```
http {
       	  ...
	  error_log  /var/log/nginx/error_log  crit;
	  ...
}

```


It is also possible to record error logs for all the virtual host separately by overriding the error_log directive in the server context. The following example exactly does that by overriding error_log directive in the server context.


```
http {
       ...
       ...
       error_log  /var/log/nginx/error_log;
       server {
	        	listen 80;
		        server_name domain1.com;
       		        error_log  /var/log/nginx/domain1.error_log  warn;
                        ...
	   }
       server {
	        	listen 80;
		        server_name domain2.com;
      		        error_log  /var/log/nginx/domain2.error_log  debug;
                        ...
	   }
}

```


All the examples described above records the log events to a file. You can also configure the error_log directive for sending the log events to a syslog server. The following error_log directive sends the error logs to syslog server with an IP address of 192.168.10.11 in debug format.


```
error_log syslog:server=192.168.10.11 debug;

```


In some situation, you might want to disable the error log. To do that, set the log file name to /dev/null.


```
error_log /dev/null;

```


# Nginx Error Log Severity Levels


There are many types of log levels that are associated with a log event and with a different priority. All the log levels are listed below. In the following log levels, debug has top priority and includes the rest of the levels too. For example, if you specify error as a log level, then it will also capture log events those are labeled as crit, alert and emergency.


1. emerg: Emergency messages when your system may be unstable.
2. alert: Alert messages of serious issues.
3. crit: Critical issues that need to be taken care of immediately.
4. error: An error has occured. Something went wrong while processing a page.
5. warn: A warning messages that you should look into it.
6. notice: A simple log notice that you can ignore.
7. info: Just an information messages that you might want to know.
8. debug: Debugging information used to pinpoint the location of error.

# Summary


The access and error logs in NGINX will not only keep a tab on users activity but also save your time and effort in the process of debugging. Further, you can also customize the access log if you need more information at your disposal. It is always better to enable access and error logs because these two files contain all the clues for better maintenance of the NGINX server.


