# How to Connect to a Terminal from Your Browser Using Python WebSSH

```Linux Basics``` ```Nginx``` ```Python```

# Introduction


Ordinarily, you connect to an SSH server using a command line app in a terminal, or terminal emulator software that includes an SSH client. Some tools, like Python’s WebSSH, make it possible to connect over SSH and run a terminal directly in your web browser.


This can be useful in a number of situations. It is particularly helpful for giving live presentations or demos, when it would be challenging to share a regular terminal window in a way that makes visual sense. It can also be helpful in educational settings when granting access to command line novices, as it avoids them needing to install software on their machines (especially on Windows, where the default options come with caveats). Finally, Python’s WebSSH in particular is very portable and requires no dependencies other than Python to get up and running. Other web-based terminal stacks can be much more complicated, and specific to Linux.


In this tutorial, you will set up WebSSH and connect over SSH in your browser. You will then optionally secure it with an SSL certificate and run it behind an Nginx reverse proxy for a production deployment.


## Prerequisites


- 
A Windows, Mac, or Linux environment with a running SSH service. It can be useful to run WebSSH locally, but if you don’t have an SSH service running on a local machine, you can use a remote Linux server by following our initial server setup guide for Ubuntu 22.04.

- 
The Python programming language installed along with pip, its package manager. You can install Python and pip on Ubuntu by following Step 1 of this tutorial.

- 
Optionally, to enable HTTPS in the browser, you will need SSL certificates and your own domain name. You can obtain them by following How To Use Certbot Standalone Mode to Retrieve Let’s Encrypt SSL Certificates. If you do not have your own domain name, you can use an IP address for the first two steps of this tutorial.


# Step 1 – Installing WebSSH


If you’ve already installed Python and pip, you should be able to install Python packages from PyPI, the Python software repository. WebSSH is designed to be installed and run directly from the command line, so you won’t need to set up another virtual environment as discussed in the How To Install Python 3 and Set Up a Programming Environment. Virtual environments are more useful when working on your own projects, not when installing system-wide tools.


Use pip install to install the WebSSH package:


```
sudo pip3 install webssh


```


```
Output…
Successfully built webssh
Installing collected packages: tornado, pycparser, cffi, pynacl, paramiko, webssh
Successfully installed cffi-1.15.1 paramiko-2.11.0 pycparser-2.21 pynacl-1.5.0 tornado-6.2 webssh-1.6.0

```


Installing via sudo will install the wssh command globally. You can verify this by using which wssh:


```
which wssh


```


```
Output/usr/local/bin/wssh

```


You’ve now installed WebSSH. In the next step, you’ll run and connect to it. First, though, you’ll need to add a firewall rule. WebSSH runs on port 8888 by default. If you are using ufw as a firewall, allow that port through ufw:


```
sudo ufw allow 8888


```


You’ll revisit that firewall rule later in this tutorial.


# Step 2 – Running and Connecting to WebSSH


If you’re running WebSSH on a local machine, you can run wssh by itself with no additional arguments to start. If you’re running WebSSH on a remote server, you’ll need to add the --fbidhttp=False flag to allow remote connections over regular HTTP. This is not secure if you are connecting over an unprotected network, but it is useful for a demo, and you will secure WebSSH later in this tutorial.


```
wssh --fbidhttp=False


```


Now you can connect to WebSSH and log in. Navigate to your_domain:8888 in a web browser (using localhost in place of your_domain if you are running locally). You will see the WebSSH login page:





Provide your regular SSH credentials. If you followed DigitalOcean’s initial server setup guide, you will be using key-based authentication rather than a password. That means you should only need to specify the Hostname of the server you’re connecting to, your Username for the server, and your SSH key, which should be located in the .ssh/ folder within your local home directory (usually named id_rsa).



Note: As you might guess from having to manually specify a Hostname, WebSSH can also be used to connect to servers other than the one it is running on. For this tutorial, it is being run on the same server you are connecting to.

Click the Connect button, and you should be greeted with your default terminal welcome prompt:





At this point, you can use your terminal as normal, exactly as if you’d connected over SSH. Multiple users can also connect through the same WebSSH instance simultaneously. If you are running WebSSH on a local machine solely for the purpose of streaming or capturing video, this may be all you need. You can enter Ctrl+C in the terminal that you launched WebSSH from (not the WebSSH terminal) to stop the WebSSH server when finished.


If you are running on a remote server, you will not want to use WebSSH in production behind an unsecured HTTP connection. Although you would still be protected by your SSH service’s authentication mechanism, using an SSH connection over HTTP poses a significant security risk, and will likely allow others to steal your SSH credentials. In the next steps, you’ll secure your WebSSH instance so that it is no less safe than a regular SSH connection.


# Step 3 – (Optional) Securing WebSSH with an SSL Certificate


To complete this step, you should have already obtained your own domain name and SSL certificates. One way to do that is by using LetsEncrypt in standalone mode.


When LetsEncrypt retrieves certificates, by default, it stores them in /etc/letsencrypt/live/your_domain. Check to make sure that you have them:


```
sudo ls /etc/letsencrypt/live/your_domain


```


```
OutputREADME  cert.pem  chain.pem  fullchain.pem  privkey.pem

```


To run WebSSH with HTTPS support, you’ll need to provide a path to a cert, and a path to a key. These are fullchain.pem and privkey.pem. By default, WebSSH provides HTTPS access on port 4433, so open that port in your firewall as well:


```
sudo ufw allow 4433


```


Next, you’ll need to allow


Then, run WebSSH with the paths to your cert and your key:


```
sudo wssh --certfile='/etc/letsencrypt/live/your_domain/fullchain.pem' --keyfile='/etc/letsencrypt/live/your_domain/privkey.pem'


```


In a web browser, navigate to https://your_domain:4433, and you should see the same interface as in the prior step, now with HTTPS support. This is now enough setup for a safe production configuration. However, you’re still running the wssh app directly from your terminal, and you’re accessing it in the browser from an unusual port. In the last two steps of this tutorial, you will remove both of these limitations.


# Step 4 – (Optional) Running WebSSH Behind an Nginx Reverse Proxy


Putting a web server such as Nginx in front of other web-facing applications can improve performance and make it much more straightforward to secure a site. You’ll install Nginx and configure it to reverse proxy requests to WebSSH, meaning it will take care of handling requests from your users to WebSSH and back again.


Refresh your package list, then install Nginx using apt:


```
sudo apt update nginx
sudo apt install nginx


```


If you are using a ufw firewall, you should make some changes to your firewall configuration at this point, to enable access to the default HTTP/HTTPS ports, 80 and 443. ufw has a stock configuration called “Nginx Full” which provides access to both of these ports:


```
sudo ufw allow “Nginx Full”


```


Nginx allows you to add per-site configurations to individual files in a subdirectory called sites-available/. Using nano or your favorite text editor, create a new Nginx configuration at /etc/nginx/sites-available/webssh:


```
sudo nano /etc/nginx/sites-available/webssh


```


Paste the following into the new configuration file, being sure to replace your_domain with your domain name.


/etc/nginx/sites-available/webssh
```
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name your_domain www.your_domain
    root /var/www/html;

    access_log /var/log/nginx/webssh.access.log;
    error_log /var/log/nginx/webssh.error.log;

    location / {
        proxy_pass http://127.0.0.1:8888;
        proxy_http_version 1.1;
        proxy_read_timeout 300;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Real-PORT $remote_port;
    }

    listen 443 ssl;
    # RSA certificate
    ssl_certificate /etc/letsencrypt/live/your_domain/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your_domain/privkey.pem;

    # Redirect non-https traffic to https
    if ($scheme != "https") {
        return 301 https://$host$request_uri;
    }
}

```


You can read this configuration as having three main “blocks” to it. The first block, coming before the location / line, contains a boilerplate Nginx configuration for serving a website on the default HTTP port, 80. The location / block contains a configuration for proxying incoming connections to WebSSH, running on port 8888 internally, while preserving SSL. The configuration at the end of the file, after the location / block, loads your LetsEncrypt SSL keypairs and redirects HTTP connections to HTTPS.


Save and close the file. If you are using nano, press Ctrl+X, then when prompted, Y and then Enter.


Next, you’ll need to activate this new configuration. Nginx’s convention is to create symbolic links (like shortcuts) from files in sites-available/ to another folder called sites-enabled/ as you decide to enable or disable them. Using full paths for clarity, make that link:


```
sudo ln -s /etc/nginx/sites-available/webssh /etc/nginx/sites-enabled/webssh


```


By default, Nginx includes another configuration file at /etc/nginx/sites-available/default, linked to /etc/nginx/sites-enabled/default, which also serves its default index page. You’ll want to disable that rule by removing it from /sites-enabled, because it conflicts with your new WebSSH configuration:


```
sudo rm /etc/nginx/sites-enabled/default


```



Note: The Nginx configuration in this tutorial is designed to serve a single application, WebSSH. You could expand this Nginx configuration to serve multiple applications on the same server by following the Nginx documentation.

Next, run nginx -t to verify your configuration before restarting Nginx:


```
sudo nginx -t


```


```
Outputnginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


Now you can restart your Nginx service, so it will reflect your new configuration:


```
sudo systemctl restart nginx


```


Finally, you can remove the firewall rules you created earlier for accessing WebSSH directly, since all traffic will now be handled by Nginx over the standard HTTP/HTTPS ports:


```
sudo ufw delete allow 8888
sudo ufw delete allow 4433


```


Restart webssh on the command line:


```
wssh


```


You do not need to provide the cert and key paths this time, since Nginx is handling that. Then navigate to your_domain in a web browser.


Notice that the WebSSH is now being served over HTTPS via Nginx without needing to specify a port. At this point, you have automated everything except launching wssh itself. You will do that in the final step.


# Step 5 – (Optional) Creating a Systemd Service for WebSSH


Deploying server-side applications that do not automatically run in the background can be unintuitive at first, since you’d need to start them directly from the command line every time. The solution to this is to set up your own background service.


To do this, you’ll create a unit file that can be used by your server’s init system. On nearly all modern Linux distros, the init system is called Systemd, and you can interact with it by using the systemctl command.


If WebSSH is still running in your terminal, press Ctrl+C to stop it. Then, using nano or your favorite text editor, open a new file called /etc/systemd/system/webssh.service:


```
sudo nano /etc/systemd/system/webssh.service


```


Your unit file needs, at minimum, a [Unit] section, a [Service] section, and an [Install] section:


/etc/systemd/system/webssh.service
```
[Unit]
Description=WebSSH terminal interface
After=network.target

[Service]
User=www-data
Group=www-data
ExecStart=wssh

[Install]
WantedBy=multi-user.target

```


This file can be broken down as follows:


- 
The [Unit] section contains a plaintext description of your new service, as well as an After hook that specifies when it should be run at system startup, in this case after your server’s networking interfaces have come up.

- 
The [Service] section specifies which command should actually be run, as well as which user should be running it. In this case, www-data is the default Nginx user on a Ubuntu server, and wssh is the command itself.

- 
The [Install] section contains only the WantedBy=multi-user.target line, which works together with the After line in the [Unit] section to ensure that the service is started when the server is ready to accept user logins.


Save and close the file. You can now start your new WebSSH service, and enable it to run on boot automatically:


```
sudo systemctl start webssh
sudo systemctl enable webssh


```


Use systemctl status webssh to verify that it started successfully. You should receive similar output to when you first ran the command in a terminal.


```
sudo systemctl status webssh


```


```
Output● webssh.service - WebSSH terminal interface
     Loaded: loaded (/etc/systemd/system/webssh.service; enabled; vendor preset: enabled)
     Active: active (running) since Thu 2022-08-11 22:08:25 UTC; 2s ago
   Main PID: 15678 (wssh)
      Tasks: 1 (limit: 1119)
     Memory: 20.2M
        CPU: 300ms
     CGroup: /system.slice/webssh.service
             └─15678 /usr/bin/python3 /usr/local/bin/wssh

Aug 11 22:08:25 webssh22 systemd[1]: Started WebSSH terminal interface.
Aug 11 22:08:26 webssh22 wssh[15678]: [I 220811 22:08:26 settings:125] WarningPolicy
Aug 11 22:08:26 webssh22 wssh[15678]: [I 220811 22:08:26 main:38] Listening on :8888 (http)

```


You can now reload https://your_domain in your browser, and you should once again get the WebSSH interface. From now on, WebSSH and Nginx will automatically restart with your server and run in the background.


# Conclusion


In this tutorial, you installed WebSSH, a portable solution for providing a command line interface in a web browser. You improved your deployment by adding SSL, then by adding an Nginx reverse proxy, and finally by creating a system service for WebSSH. This is a good model for deploying small server-side web applications in general, and particularly important for SSH, which relies on key pairs for security.


Next, you may want to learn about other connection options for SSH.


