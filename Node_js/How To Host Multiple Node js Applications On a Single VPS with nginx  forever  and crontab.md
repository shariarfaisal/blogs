# How To Host Multiple Node js Applications On a Single VPS with nginx  forever  and crontab

```Deployment``` ```Node.js``` ```Nginx```

## Requirements to Follow This Tutorial



You need to have nginx and Node.js installed, and there are already well written tutorials about these topics on DigitalOcean:


How to install nginx and How to install Node.js.


In addition, you should already own a domain, in order to map a running Node.js service to a domain name, instead of navigating to http://[your-vps-ip]:[port].


# Running Your Node.js Application with Forever



Forever is a simple command line tool for ensuring that a Node.js application runs continuously (i.e. forever). This means if your app encounters an error and crashes, forever will take care of this issue and restart it for you.


Simply install forever globally and forever can be used in a matter of seconds:


```
npm install forever -g

```


To start a script with forever you need to follow these steps:


Navigate to your Node.js application:


```
cd /path/to/your/node/app/

```


and run the server/main JavaScript file with forever:


```
forever start --spinSleepTime 10000 main.js

```


Where --spinSleepTime 10000 refers to the minimum uptime (in milliseconds) between launches of a crashing script. This command will work for almost all cases.


Now point your browser to http://[your-vps-ip]:[port] and see your app running.


# Map a Domain To Your Node.js Application



Now you’ll need to add a DNS record in your DigitalOcean control panel to map your domain name to your droplet (VPS).


The steps to follow are:<ol>
<li>Login at DigitalOcean.com</li>
<li>Click on the ‘DNS’ section in the left sidebar</li>
<li>Add a domain by clicking on the ‘Add Domain’ button, select your VPS of choice and enter the domain name you registered in the ‘Name’ field</li>
<li>Copy the Nameservers provided by DigitalOcean (e.g. NS1.DIGITALOCEAN.COM.) and add each one to the DNS records in the control panel of your domain registrar.</li></ol>


Note: changes will not be immediate, since DNS can take up to 24 hours to propagate.


# Map a Domain To a Service Running on Your VPS with nginx



In this section, you’ll learn how to set up a reverse proxy with nginx in a few simple steps.


First of all, create a file for your desired domain in /etc/nginx/conf.d/ with your favorite editor (I’ll use nano). The file should be named after the domain name, for consistency reasons.


```
nano /etc/nginx/conf.d/example.com.conf

```


Note: you can call the file whatever you want, the important part is the .conf extension.


In this file, you’ll want to copy the following code snippet and paste it in the file created before:


```
server {
    listen 80;

    server_name your-domain.com;

    location / {
        proxy_pass http://localhost:{YOUR_PORT};
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

```


Now simply replace your-domain.com with the domain you have registered and YOUR_PORT with the port your Node.js app is listening to on your VPS.


Note: to be able to reference multiple domains for one Node.js app (like www.example.com and example.com) you need to add the following code to the file /etc/nginx/nginx.conf in the http section:


```
server_names_hash_bucket_size 64;

```


If the DNS changes are propagated, you can point your web browser to your domain and you should see your application running, accessible from the internet.


# Restarting Your Node.js App at Reboot



Forever is good when it comes to keeping your application running when it crashes etc. but what happens when the VPS gets rebooted?


This is where a simple cronjob can prevent your application and your users from unexpected downtimes.


Create a file called starter.sh in your application’s home folder and copy the following code:


```
#!/bin/sh

if [ $(ps -e -o uid,cmd | grep $UID | grep node | grep -v grep | wc -l | tr -s "\n") -eq 0 ]
then
        export PATH=/usr/local/bin:$PATH
        forever start --sourceDir /path/to/your/node/app main.js >> /path/to/log.txt 2>&1
fi


```


where main.js should be replaced with your application’s main script.


This useful snippet has been taken from here


To start this script at each reboot you need to edit the crontab with this command:


```
crontab -e

```


and append the following code to this file


```
@reboot /path/to/starter.sh

```


Now set the absolute path to your starter.sh file.


Tip: Navigate where your starter.sh file is located and print the current directory with pwd.


Repeat the steps above for each of your domains/services.


<div class=“author”>Submitted by: <a href=“https://twitter.com/christian_fei”>Christian Fei</a></div>


