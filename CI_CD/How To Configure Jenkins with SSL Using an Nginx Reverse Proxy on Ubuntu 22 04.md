# How To Configure Jenkins with SSL Using an Nginx Reverse Proxy on Ubuntu 22 04

```CI/CD``` ```Deployment``` ```Let's Encrypt``` ```Ubuntu 22.04``` ```Ubuntu```

## Introduction


By default, Jenkins comes with its own built-in Winstone web server listening on port 8080, which is convenient for getting started. It’s also a good idea, however, to secure Jenkins with SSL to protect passwords and sensitive data transmitted through the web interface.


In this tutorial, you will configure Nginx as a reverse proxy to direct client requests to Jenkins.


# Prerequisites


To begin, you’ll need the following:


- One Ubuntu 22.04 server configured with a non-root sudo-enabled user and firewall, following the Ubuntu 22.04 initial server setup guide.
- Jenkins installed, following the steps in How to Install Jenkins on Ubuntu 22.04
- Nginx installed, following the steps in How to Install Nginx on Ubuntu 22.04
- An SSL certificate for a domain provided by Let’s Encrypt. Follow How to Secure Nginx with Let’s Encrypt on Ubuntu 22.04 to obtain this certificate. Note that you will need a registered domain name that you own or control. This tutorial will use the domain name example.com throughout.

# Step 1 — Configuring Nginx


In the prerequisite tutorial How to Secure Nginx with Let’s Encrypt on Ubuntu 22.04, you configured Nginx to use SSL in the /etc/nginx/sites-available/example.com file. Open this file to add your reverse proxy settings:


```
sudo nano /etc/nginx/sites-available/example.com


```


In the server block with the SSL configuration settings, add Jenkins-specific access and error logs:


/etc/nginx/sites-available/example.com
```
. . . 
server {
        . . .
        # SSL Configuration
        #
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        access_log            /var/log/nginx/jenkins.access.log;
        error_log             /var/log/nginx/jenkins.error.log;
        . . .
        }

```


Next configure the proxy settings. Since all requests are being sent to Jenkins, comment out the default try_files line, which would otherwise return a 404 error before the request reaches Jenkins:


/etc/nginx/sites-available/example.com
```
. . .
           location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;        }
. . . 

```


Let’s now add the proxy settings, which include:


- proxy_params: The /etc/nginx/proxy_params file is supplied by Nginx and ensures that important information, including the hostname, the protocol of the client request, and the client IP address, is retained and available in the log files.
- proxy_pass: This sets the protocol and address of the proxied server, which in this case will be the Jenkins server accessed via localhost on port 8080.
- proxy_read_timeout: This enables an increase from Nginx’s 60 second default to the Jenkins-recommended 90 second value.
- proxy_redirect: This ensures that responses are correctly rewritten to include the proper host name.

Be sure to substitute your SSL-secured domain name for example.com in the proxy_redirect line below:


/etc/nginx/sites-available/example.com
```
Location /  
. . .
           location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.
                # try_files $uri $uri/ =404;
                include /etc/nginx/proxy_params;
                proxy_pass          http://localhost:8080;
                proxy_read_timeout  90s;
                # Fix potential "It appears that your reverse proxy setup is broken" error.
                proxy_redirect      http://localhost:8080 https://example.com;

```


Once you’ve made these changes, save the file and exit the editor. We’ll hold off on restarting Nginx until after we’ve configured Jenkins, but we can test our configuration now:


```
sudo nginx -t


```


If all is well, the command will return:


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If not, fix any reported errors until the test passes.



Note:
If you misconfigure the proxy_pass (by adding a trailing slash, for example), you will get something similar to the following in your Jenkins Configuration page.

If you see this error, double-check your proxy_pass and proxy_redirect settings in the Nginx configuration.

# Step 2 — Configuring Jenkins


For Jenkins to work with Nginx, you will need to update the Jenkins configuration so that the Jenkins server listens only on the localhost interface rather than on all interfaces (0.0.0.0). If Jenkins listens on all interfaces, it’s potentially accessible on its original, unencrypted port (8080).


Let’s modify the /etc/default/jenkins configuration file to make these adjustments:


```
sudo nano /etc/default/jenkins


```


Locate the JENKINS_ARGS line and add --httpListenAddress=127.0.0.1 to the existing arguments:


/etc/default/jenkins
```
. . .
JENKINS_ARGS="--webroot=/var/cache/$NAME/war --httpPort=$HTTP_PORT --httpListenAddress=127.0.0.1"

```


Save and exit the file.


To use the new configuration settings, restart Jenkins:


```
sudo systemctl restart jenkins


```


Since systemctl doesn’t display output, check the status:


```
sudo systemctl status jenkins


```


You should see the active (exited) status in the Active line:


```
Output● jenkins.service - Jenkins Continuous Integration Server
     Loaded: loaded (/lib/systemd/system/jenkins.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 16:35:49 UTC; 2s ago
   Main PID: 89751 (java)
      Tasks: 44 (limit: 4665)
     Memory: 358.9M
        CPU: 20.195s
     CGroup: /system.slice/jenkins.service
             └─89751 /usr/bin/java -Djava.awt.headless=true -jar /usr/share/java/jenkins.war --webroot=/var/cache/jenkins/war --httpPort=8080

```


Restart Nginx:


```
sudo systemctl restart nginx


```


Check the status:


```
sudo systemctl status nginx


```


```
Output● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 16:36:17 UTC; 7s ago
       Docs: man:nginx(8)
    Process: 89866 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
    Process: 89869 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exited, status=0/SUCCESS)
   Main PID: 89870 (nginx)
      Tasks: 3 (limit: 4665)
     Memory: 4.1M
        CPU: 51ms
     CGroup: /system.slice/nginx.service
             ├─89870 "nginx: master process /usr/sbin/nginx -g daemon on; master_process on;"

```


With both servers restarted, you should be able to visit the domain using either HTTP or HTTPS. HTTP requests will be redirected automatically to HTTPS, and the Jenkins site will be served securely.


# Step 3 — Testing the Configuration


Now that you have enabled encryption, you can test the configuration by resetting the administrative password. Let’s start by visiting the site via HTTP to verify that you can reach Jenkins and are redirected to HTTPS.


In your web browser, enter http://example.com, substituting your domain for example.com. After you press ENTER, the URL should start with https and the location bar should indicate that the connection is secure.


You can enter the administrative username you created in How To Install Jenkins on Ubuntu 22.04 in the User field, and the password that you selected in the Password field.


Once logged in, you can change the password to be sure it’s secure.


Click on your username in the upper-right-hand corner of the screen. On the main profile page, select Configure from the list on the left side of the page:





This will take you to a new page, where you can enter and confirm a new password:





Confirm the new password by clicking Save. You can now use the Jenkins web interface securely.


## Conclusion


In this tutorial, you configured Nginx as a reverse proxy to Jenkins’ built-in web server to secure your credentials and other information transmitted via the web interface. Now that Jenkins is secure, you can learn how to set up a continuous integration pipeline to automatically test code changes.  Other resources to consider if you are new to Jenkins are the Jenkins project’s “Creating your first Pipeline” tutorial or the library of community-contributed plugins.


