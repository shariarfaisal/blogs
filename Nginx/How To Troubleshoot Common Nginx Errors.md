# How To Troubleshoot Common Nginx Errors

```Nginx```

## Introduction


There are several methods you can use to troubleshoot Nginx errors. Keep in mind that these methods of troubleshooting are meant as a starting point, and further investigation is often required to diagnose the root cause of an issue. As you go through this tutorial, the errors themselves will provide key information leading you to a solution.


This tutorial will review the following commands that are commonly used to troubleshoot Nginx across most Linux distributions:


- sudo cat /var/log/nginx/error.log: This is used to print a log with a list of errors and details about them.
- sudo nginx -t: This is used to check for syntax errors in your configuration file.
- systemctl status nginx: This is used to check if your Nginx service is active or inactive.

You will learn more about these commands and how to use them for troubleshooting various Nginx errors.


# Troubleshooting with the Error Log


When you receive an error from Nginx, it’s not always clear what the issue might be. For this reason, one error could be connected to a larger issue or a separate issue altogether. It depends on your specific circumstance the setup may vary. To get a full overview of the Nginx errors happening, run the following command to receive a running list:


```
sudo cat /var/log/nginx/error.log


```


You must run this command as a privileged user. We recommend using a sudo-enabled user rather than the root user. The cat command stands for concatenate, which is used to read the contents of a file and print it in the output of your terminal. In this case, cat is reading and printing the contents of the /var/log/nginx/error.log file. When you run this command, your output will return a list of errors. Keep in mind that if there are no errors, your prompt will remain blank. Here is an example of an error log:


```
2022/11/28 23:58:22 [emerg] 168641#168641: invalid host in "[::]443" of the "listen" directive in /etc/nginx/sites-enabled/test.do-community.com:12
2022/11/28 23:59:44 [emerg] 168664#168664: invalid number of arguments in "root" directive in /etc/nginx/sites-enabled/test.do-community.com:4
2022/11/29 00:00:19 [emerg] 168701#168701: "server" directive is not allowed here in /etc/nginx/sites-enabled/test.do-community.com:6

```


This error log output provides key information about the specific error you are encountering. The first part of this log item details the date and time the error occurred and the type of error message. In this case, this is the [emerg] or emergency type of message. The final component of a log item consists of the error message itself and, where applicable, what file and specific line it can be found.


Overall, consulting your Nginx error log is helpful if you want more context on the error(s) you may be receiving.


# Checking for Syntax Errors


One of the most common errors with Nginx has to do with syntax in your configuration file. Whether it’s missing characters or an incorrect syntax structure, if the syntax is incorrect, it will not work. This is because the configuration file consists of various directives and must be declared correctly, or they will be invalid. To check if you have any syntax errors, run the following command:


```
sudo nginx -t


```


This command should be run by a privileged user, we recommend a sudo-enabled user as opposed to the root user. Additionally, the t flag signifies that this will test the file before actually running it. If your syntax is correct, you will receive the following output:


```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If not, you will receive an error message similar to the examples we share about the error log. Here is what your output may return if there’s a syntax error:


```
[emerg] invalid host in "[::]443" of the "listen" directive in /etc/nginx/sites-enabled/test.do-community.com:12
nginx: configuration file /etc/nginx/nginx.conf test failed

```


We recommend always running this syntax command to verify that there’s nothing missing or invalid in your configuration file. Additionally, you should always run sudo systemctl reload nginx after making any configuration changes. This will restart Nginx and apply any changes you’ve made.


# Troubleshooting with systemctl status nginx


Another option when troubleshooting Nginx errors is to verify that this service is active and working on your system. It’s possible that the installation was incomplete, or perhaps the service has not been turned on. You can check whether your Nginx service is active or not with the following status check via the systemd init system:


```
systemctl status nginx


```


If your service is running, it will read as active (running in your output:


```
 nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enable>
     Active: active (running) since Tue 2022-11-29 16:37:49 UTC; 29s ago
       Docs: man:nginx(8)
    Process: 2679 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_proce>
    Process: 2680 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (c>
   Main PID: 2787 (nginx)
      Tasks: 2 (limit: 1116)
     Memory: 3.3M
        CPU: 32ms
     CGroup: /system.slice/nginx.service
             ├─2787 "nginx: master process /usr/sbin/nginx -g daemon on; master>
             └─2790 "nginx: worker process"

```


If your service is not running, it will read as inactive (dead) in the output:


```
○ nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; preset: enable>
     Active: inactive (dead) since Tue 2022-11-29 16:42:27 UTC; 1s ago
   Duration: 4min 38.006s
       Docs: man:nginx(8)
    Process: 2679 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_proce>
    Process: 2680 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (c>
    Process: 2915 ExecStop=/sbin/start-stop-daemon --quiet --stop --retry QUIT/>
   Main PID: 2787 (code=exited, status=0/SUCCESS)
        CPU: 39ms

```


If this happens, it’s possible that you may need to restart your Nginx service again. You can do this with the following command:


```
sudo systemctl restart nginx


```


After, you can run systemctl status nginx to verify your service is active again. If you’re interested in learning more about these basic management commands, read our tutorial, where we discuss how to manage the Nginx process.


# Other Troubleshooting Tips


Even though we only reviewed three methods for troubleshooting Nginx errors, here are a couple more examples that are specific to firewall and configuration settings.


## Adjusting Firewall Settings


When you set up Nginx, your server defaults to port 80 HTTP traffic. If you don’t have this port open to receive these requests, then your website will not work properly. Depending on your distribution, adjusting your firewall settings may vary. This example will use the Uncomplicated Firewall (ufw). To confirm whether your port is open, you can check the status with the following:


```
sudo ufw status


```


Again, you need a user with privileges, and a sudo-enabled user is recommended rather than the root user. If your output returns the following, it means you do have the appropriate port 80 open, specifically the Nginx HTTP profile is listed:


```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx HTTP                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx HTTP (v6)            ALLOW       Anywhere (v6)

```


If you need to open additional ports, such as 443 to allow for HTTPS traffic, then you can add the rule to the list:


```
sudo ufw allow 'Nginx HTTPS'


```


Alternatively, you can add both rules with the single profile 'Nginx Full':


```
sudo ufw allow 'Nginx Full'


```


To verify your ports are open, run sudo ufw status again, and if they’re listed then you’re ready to go. You can also go check your web browser to ensure your server is up and running successfully.


## Checking the Configuration File


If you’re using an Nginx web server, you likely have a server block set up, especially if you followed our tutorial on How To Install Nginx. To serve the HTML content you desire, you should have a configuration block set up for your site. Here is an example of a server block with the configuration details for your domain:


/etc/nginx/sites-available/your_domain
```
server {
       listen 80;
         listen [::]:80;

       root /var/www/your_domain/html;
        index index.html index.htm index.nginx-debian.html;

        server_name your_domain www.your_domain;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


When you’re adding to or updating the configuration file, always remember to save when you’re done. If you’re using the nano text editor, you can do this by pressing CTRL + X, Y, then ENTER. The primary way to troubleshoot any issues with your configuration file is to run the syntax check sudo nginx -t mentioned earlier, and enable those changes by restarting Nginx with sudo systemctl restart nginx.


You can always evaluate your configuration file by opening it with your preferred text editor, such as in the following:


```
sudo nano /etc/nginx/sites-available/your_domain


```


Remember that any directives you have or add to this file must be exact, or you will receive an error that something is invalid.


# Conclusion


This tutorial provided a quick reference guide on how to troubleshoot common errors you might encounter with Nginx. As you recall, these commands provide a first step in diagnosing the issue, but you may need to investigate the error further. If you’re ever uncertain, though, you can always begin by checking the Nginx error log for detailed entries about each error. If you’re interested in a more comprehensive explanation of some common Nginx errors, you can read the following tutorials:


- Common Nginx Syntax Errors
- Common Nginx Connection Errors
- Nginx SSL Certificate and HTTPS Redirect Errors

