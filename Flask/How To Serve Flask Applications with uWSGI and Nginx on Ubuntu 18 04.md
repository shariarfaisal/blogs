# How To Serve Flask Applications with uWSGI and Nginx on Ubuntu 18 04

```Python``` ```Ubuntu``` ```Python Frameworks``` ```Nginx``` ```Ubuntu 18.04``` ```Flask```

## Introduction


In this guide, you will build a Python application using the Flask microframework on Ubuntu 18.04. The bulk of this article will be about how to set up the uWSGI application server and how to launch the application and configure Nginx to act as a front-end reverse proxy.


# Prerequisites


To complete this tutorial, you will need:


- 
A server with Ubuntu 18.04 installed and a non-root user with sudo privileges and a firewall enabled. Follow our initial server setup guide for guidance.

- 
Nginx installed, following Steps 1 and 2 of How To Install Nginx on Ubuntu 18.04.

- 
A domain name configured to point to your server. You can purchase one on Namecheap or get one for free on Freenom. You can learn how to point domains to DigitalOcean by following the relevant documentation on domains and DNS. Be sure to create the following DNS records:

An A record with your_domain pointing to your server’s public IP address.
An A record with www.your_domain pointing to your server’s public IP address.


- An A record with your_domain pointing to your server’s public IP address.
- An A record with www.your_domain pointing to your server’s public IP address.
- 
Familiarity with uWSGI, our application server, and the WSGI specification. This discussion of definitions and concepts goes over both in detail.


# Step 1 — Installing the Components from the Ubuntu Repositories


The first step is to install all of the pieces that you need from the Ubuntu repositories. You’ll install pip, the Python package manager, to manage your Python components. You’ll also get the Python development files necessary to build uWSGI.


First, update the local package index:


```
sudo apt update


```


Then install the packages that will allow you to build your Python environment. These will include python3-pip, along with a few more packages and development tools necessary for a robust programming environment:


```
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools


```


With these packages in place, you can move on to creating a virtual environment for your project.


# Step 2 — Creating a Python Virtual Environment


Next, set up a virtual environment in order to isolate your Flask application from the other Python files on the system.


Start by installing the python3-venv package, which will install the venv module:


```
sudo apt install python3-venv


```


Next, make a parent directory for your Flask project:


```
mkdir ~/myproject


```


Then move into the directory after you create it:


```
cd ~/myproject 


```


Create a virtual environment to store your Flask project’s Python requirements by running the following:


```
python3.6 -m venv myprojectenv


```


This will install a local copy of Python and pip into a directory called myprojectenv within your project directory.


Before installing applications within the virtual environment, you need to activate it:


```
source myprojectenv/bin/activate


```


Your prompt will change to indicate that you are now operating within the virtual environment.  It will read like the following: (myprojectenv) user@host:~/myproject$.


# Step 3 — Setting Up a Flask Application


Now that you are in your virtual environment, you can install Flask and uWSGI and get started on designing your application.


First, install wheel with the local instance of pip to ensure that your packages will install even if they are missing wheel archives:


```
pip install wheel


```



Note: Regardless of which version of Python you are using, when the virtual environment is activated, you should use the pip command (not pip3).

Next, install Flask and uWSGI:


```
pip install uwsgi flask


```


After installation is complete, you can begin using Flask.


## Creating a Sample App


Now that you have Flask available, you can create a simple application. As you may recall, Flask is a microframework and doesn’t include many of the tools that more full-featured frameworks might. Flask exists primarily as a module that you can import into your projects to assist you in initializing a web application.


While your application might be more complex, you’ll create your Flask app in a single file. You can create the file using your favorite text editor. For this example, we will use nano and name it myproject.py:


```
nano ~/myproject/myproject.py


```


The application code will live in this file. It will import Flask and instantiate a Flask object. You can use this to define the functions that should be run when a specific route is requested:


~/myproject/myproject.py
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')

```


This defines what content to present when the root domain is accessed. Save and close the file when you’re finished. If you’re using nano you can do this by pressing CTRL + X then Y and ENTER.


If you followed the initial server setup guide, you should have a UFW firewall enabled. To test the application, you need to allow access to port 5000:


```
sudo ufw allow 5000


```


Now test your Flask app:


```
python myproject.py


```


You will receive output like the following, including a helpful warning reminding you not to use this server setup in production:


```
Output* Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)

```


Visit your server’s IP address followed by :5000 in your web browser:


```
http://your_server_ip:5000

```


You should see something like the following:





When you are finished, hit CTRL + C in your terminal window to stop the Flask development server.


## Creating the WSGI Entry Point


Next, you’ll create a file that will serve as the entry point for your application. This will tell your uWSGI server how to interact with it.


First create and name the file wsgi.py:


```
nano ~/myproject/wsgi.py


```


In this file, import the Flask instance from your application and then run it:


~/myproject/wsgi.py
```
from myproject import app

if __name__ == "__main__":
    app.run()

```


Save and close the file when you are finished.


# Step 4 — Configuring uWSGI


Your application is now written with an entry point established. You can now move on to configuring uWSGI.


## Testing uWSGI Serving


Before making more changes, it might be helpful to test that uWSGI can serve your application.


You can do this by passing the name of your entry point to uWSGI. This is constructed by the name of the module (minus the .py extension) plus the name of the callable within the application. In this case the entry point’s name is wsgi:app.


You’ll also specify the socket, so that it will be started on a publicly available interface, as well as the protocol, so that it will use HTTP instead of the uwsgi binary protocol. Use the same port number, 5000, that you opened earlier:


```
uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app


```


Visit your server’s IP address with :5000 appended to the end in your web browser again:


```
http://your_server_ip:5000

```


You should receive your application’s output again:





When you have confirmed that it’s functioning properly, press CTRL + C in your terminal window.


Now that you’re done with your virtual environment, you can deactivate it:


```
deactivate


```


Any Python commands will now use the system’s Python environment again.


## Creating a uWSGI Configuration File


You have tested and verified that uWSGI is able to serve your application, but ultimately you will want something more robust for long-term usage. You can create a uWSGI configuration file with the relevant options for this.


Place that file in your project directory and call it myproject.ini:


```
nano ~/myproject/myproject.ini


```


Inside, you’ll start off with the [uwsgi] header so that uWSGI knows to apply the settings.  You’ll specify two things: the module itself, by referring to the wsgi.py file minus the extension, and the callable within the file, app:


~/myproject/myproject.ini
```
[uwsgi]
module = wsgi:app

```


Next, tell uWSGI to start up in master mode and spawn five worker processes to serve actual requests:


~/myproject/myproject.ini
```
[uwsgi]
module = wsgi:app

master = true
processes = 5

```


When you were testing, you exposed uWSGI on a network port.  However, you’re going to be using Nginx to handle actual client connections, which will then pass requests to uWSGI. Since these components are operating on the same computer, a Unix socket is preferable because it is faster and more secure. Call the socket myproject.sock and place it in this directory.


Also change the permissions on the socket. This gives the Nginx group ownership of the uWSGI process later on, so make sure the group owner of the socket can read information from it and write to it. Additionally, clean up the socket when the process stops by adding the vacuum option:


~/myproject/myproject.ini
```
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

```


The last thing you’ll do is set the die-on-term option. This can help ensure that the init system and uWSGI have the same assumptions about what each process signal means. Setting this aligns the two system components, implementing the expected behavior:


~/myproject/myproject.ini
```
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true

```


You may have noticed that you did not specify a protocol like you did from the command line. That’s because by default, uWSGI speaks using the uwsgi protocol, a fast binary protocol designed to communicate with other servers. Nginx can speak this protocol natively, so it’s better to use this than to force communication by HTTP.


When you are finished, save and close the file.


# Step 5 — Creating a systemd Unit File


Next, create the systemd service unit file. Creating a systemd unit file will allow Ubuntu’s init system to automatically start uWSGI and serve the Flask application whenever the server boots.


Create a unit file ending in .service within the /etc/systemd/system directory to begin:


```
sudo nano /etc/systemd/system/myproject.service


```


Inside the file, start with the [Unit] section, which is used to specify metadata and dependencies. Describe your service here and tell the init system to only start after the networking target has been reached:


/etc/systemd/system/myproject.service
```
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

```


Next, open up the [Service] section. This will specify the user and group that you want the process to run under. Give your regular user account ownership of the process since it owns all of the relevant files. Also give group ownership to the www-data group so that Nginx can communicate easily with the uWSGI processes. Remember to replace the username here with your username:


/etc/systemd/system/myproject.service
```
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data

```


Next, map out the working directory and set the PATH environmental variable so that the init system knows that the executables for the process are located within your virtual environment. Also specify the command to start the service. Systemd requires that you give the full path to the uWSGI executable, which is installed within your virtual environment. You’ll pass the name of the .ini configuration file you created in your project directory.


Remember to replace the username and project paths with your own information:


/etc/systemd/system/myproject.service
```
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

```


Then add an [Install] section. This will tell systemd what to link this service to if you enable it to start at boot. You want this service to start when the regular multi-user system is up and running:


/etc/systemd/system/myproject.service
```
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myproject
Environment="PATH=/home/sammy/myproject/myprojectenv/bin"
ExecStart=/home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

[Install]
WantedBy=multi-user.target

```


With that, your systemd service file is complete. Save and close it now.


Now you can start the uWSGI service you created and enable it so that it starts at boot:


```
sudo systemctl start myproject
sudo systemctl enable myproject


```


Check the status:


```
sudo systemctl status myproject


```


You should receive output like the following:


```
Output● myproject.service - uWSGI instance to serve myproject
   Loaded: loaded (/etc/systemd/system/myproject.service; enabled; vendor preset
   Active: active (running) since Mon 2021-10-25 22:34:52 UTC; 14s ago
 Main PID: 9391 (uwsgi)
    Tasks: 6 (limit: 1151)
   CGroup: /system.slice/myproject.service
           ├─9391 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i
           ├─9410 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i
           ├─9411 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i
           ├─9412 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i
           ├─9413 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i
           └─9414 /home/sammy/myproject/myprojectenv/bin/uwsgi --ini myproject.i

```


If you receive any errors, be sure to resolve them before continuing with the tutorial.


# Step 6 — Configuring Nginx to Proxy Requests


Your uWSGI application server should now be up and running, waiting for requests on the socket file in the project directory. Now you can configure Nginx to pass web requests to that socket using the uwsgi protocol.


Begin by creating a new server block configuration file in Nginx’s sites-available directory.  Name it myproject to stay consistent with the rest of the guide:


```
sudo nano /etc/nginx/sites-available/myproject


```


Open up a server block and tell Nginx to listen on the default port 80. Also tell it to use this block for requests for your server’s domain name:


/etc/nginx/sites-available/myproject
```
server {
    listen 80;
    server_name your_domain www.your_domain;
}

```


Next, add a location block that matches every request. Within this block, you’ll include the uwsgi_params file that specifies some general uWSGI parameters that need to be set. Then you’ll pass the requests to the socket you defined using the uwsgi_pass directive:


/etc/nginx/sites-available/myproject
```
server {
    listen 80;
    server_name your_domain www.your_domain;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/sammy/myproject/myproject.sock;
    }
}

```


Save and close the file when you’re finished.


To enable the Nginx server block configuration you’ve just created, link the file to the sites-enabled directory:


```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled


```


With the file in that directory, you can test for syntax errors by running the following:


```
sudo nginx -t


```


If this returns without indicating any issues, restart the Nginx process to read the new configuration:


```
sudo systemctl restart nginx


```


Now adjust the firewall again. You no longer need access through port 5000, so you can remove that rule:


```
sudo ufw delete allow 5000


```


After, you’ll allow access to the Nginx server:


```
sudo ufw allow 'Nginx Full'


```


You should now be able to navigate to your server’s domain name in your web browser:


```
http://your_domain

```


You should see your application output:





If you encounter any errors, try checking the following:


- sudo less /var/log/nginx/error.log: checks the Nginx error logs.
- sudo less /var/log/nginx/access.log: checks the Nginx access logs.
- sudo journalctl -u nginx: checks the Nginx process logs.
- sudo journalctl -u myproject: checks your Flask app’s uWSGI logs.

# Step 7 — Securing the Application


To ensure that traffic to your server remains secure, you should get an SSL certificate for your domain. There are multiple ways to do this, including getting a free certificate from Let’s Encrypt, generating a self-signed certificate, or buying one from another provider and configuring Nginx to use it by following Steps 2 through 6 of  How to Create a Self-signed SSL Certificate for Nginx in Ubuntu 18.04. We will demonstrate with option one for the sake of expediency. For the full tutorial, check out How To Secure Nginx with Let’s Encrypt on Ubuntu 18.04.


First, add the Certbot Ubuntu repository:


```
sudo add-apt-repository ppa:certbot/certbot


```


You’ll need to press ENTER to accept.


Next, install Certbot’s Nginx package with apt:


```
sudo apt install python-certbot-nginx


```


Certbot provides a variety of ways to obtain SSL certificates through plugins. The Nginx plugin will take care of reconfiguring Nginx and reloading the configuration whenever necessary. To use this plugin, run the following:


```
sudo certbot --nginx -d your_domain -d www.your_domain


```


This runs certbot with the --nginx plugin, using -d to specify the names you’d like the certificate to be valid for.


If this is your first time running certbot, you will be prompted to enter an email address and agree to the terms of service. After doing so, certbot will communicate with the Let’s Encrypt server, then run a challenge to verify that you control the domain you’re requesting a certificate for.


If that’s successful, certbot will ask how you’d like to configure your HTTPS settings:


```
OutputPlease choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):

```


Select your choice then hit ENTER. The configuration will be updated, and Nginx will reload to pick up the new settings. certbot will wrap up with a message telling you the process was successful and where your certificates are stored:


```
OutputIMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2022-01-24. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le


```


If you followed the Nginx installation instructions in the prerequisites, you will no longer need the redundant HTTP profile allowance:


```
sudo ufw delete allow 'Nginx HTTP'


```


To verify the configuration, navigate once again to your domain, using https://:


```
https://your_domain

```


You should see your application output once again, along with your browser’s security indicator, which should indicate that the site is secured.


# Conclusion


In this guide, you created and secured a simple Flask application within a Python virtual environment. You created a WSGI entry point so that any WSGI-capable application server can interface with it, and then configured the uWSGI app server to provide this function. After, you created a systemd service file to automatically launch the application server on boot. You also created an Nginx server block that passes web client traffic to the application server, relaying external requests, and secured traffic to your server with Let’s Encrypt.


Flask is an extremely flexible framework meant to provide your applications with functionality without being too restrictive about structure and design. You can use the general stack described in this guide to serve the flask applications that you want to design.


