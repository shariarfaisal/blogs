# How To Deploy Web2py Python Applications with uWSGI and Nginx on Ubuntu 14 04

```Python``` ```Ubuntu``` ```Deployment``` ```Python Frameworks``` ```Nginx```

## Introduction


The web2py framework is a powerful and easy-to-use tool for quickly developing full-featured Python web applications.  With web2py, you can easily develop and manage your applications through the use of an administrative web UI.


In this guide, we will demonstrate how to deploy a web2py application on Ubuntu 14.04.  We will be using the uWSGI application server to interface with the application with multiple worker processes.  In front of uWSGI, we will set up Nginx in a reverse proxy configuration to handle the actual client connections.  This is a much more robust deployment strategy than using the web2py server or uWSGI alone.


# Prerequisites and Goals


In order to complete this guide, you should have a fresh Ubuntu 14.04 server instance with a non-root user with sudo privileges configured.  You can learn how to set this up by running through our initial server setup guide.


We will be downloading the web2py framework and testing it to make sure the default application environment functions correctly.  Afterwards, we will download and install the uWSGI application container to serve as an interface between requests and the web2py Python code.  We will set up Nginx in front of this so that it can handle client connections and proxy requests to uWSGI.  We will configure each of our components to start at boot to minimize the need for administrative intervention.


# Download the web2py Framework


Our first step will be to download the actual web2py framework.  This is maintained in a git repository on GitHub, so the best way to download it is with git itself.


We can download and install git from the default Ubuntu repositories by typing:


```
sudo apt-get update
sudo apt-get install git

```


Once git is installed, we can clone the repository to our user’s home directory.  We can name the application whatever we’d like.  In our example, we are using the name myapp for simplicity.  We need to add the --recursive flag because the database abstraction layer is handled as its own git submodule:


```
git clone --recursive https://github.com/web2py/web2py.git ~/myapp

```


The web2py framework will be downloaded to a directory called myapp within your home directory.


We can test out the default application by moving into the directory:


```
cd ~/myapp

```


The administrative interface must be secured by SSL, so we can make a simple self-signed certificate to test this out.  Create the server key and certificate by typing:


```
openssl req -x509 -new -newkey rsa:4096 -days 3652 -nodes -keyout myapp.key -out myapp.crt

```


You will have to fill out some information for the certificate you are generating.  The only part that actually matters in this circumstance is the Common Name field, which should reference your server’s domain name or IP address:


```
. . .

Country Name (2 letter code) [AU]:US
State or Province Name (full name) [Some-State]:New York
Locality Name (eg, city) []:New York City
Organization Name (eg, company) [Internet Widgits Pty Ltd]:DigitalOcean
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:server_domain_or_IP
Email Address []:admin@email.com

```


When you are finished, an SSL key and certificate should be in your application directory.  These will be called myapp.key and myapp.crt respectively.


With that complete, we can start up the web2py web interface to test it out.  To do this, we can type:


```
python web2py.py -k myapp.key -c myapp.crt -i 0.0.0.0 -p 8000

```


You will be asked to select a password for the administrative interface.


Now, you can visit your application in your web browser by navigating to:


```
https://server_domain_or_IP:8000

```


Make sure you use https instead of http in the above address.  You will be warned that your browser does not recognize the SSL certificate:





This is expected since we have signed our own certificate.  Click on the “Advanced” link or whatever other link your browser gives you and then proceed to the site as planned.  You will see the web2py interface:





By clicking on the “Administrative Interface” button on the far right, you should be able to input the password you selected when running the server and get to the administrative site:





This gives you access to the actual code that is running your applications, allowing you to edit and tweak the files from within the interface itself.


When you are finished looking around, type CTRL-C back in your terminal window.  We have tested our application and demonstrated that it can be accessed on the web when the web2py development server is running.


# Install and Configure uWSGI


Now that we have the web2py application operational, we can configure uWSGI.  uWSGI is an application server that can communicate with applications over a standard interface called WSGI.  To learn more about this, read this section of our guide on setting up uWSGI and Nginx on Ubuntu 14.04.


## Installing uWSGI


Unlike the guide linked above, in this tutorial, we’ll be installing uWSGI globally.  Before we can install uWSGI, we will need to install pip, the Python package manager, and the Python development files that uWSGI relies on.  We can install these directly from Ubuntu’s repositories:


```
sudo apt-get install python-pip python-dev

```


Now we can install uWSGI globally with pip by typing:


```
sudo pip install uwsgi

```


The uWSGI application container server interfaces with Python applications using the WSGI inteface specification.  The web2py framework includes a file designed to provide this interface within its handlers directory.  To use the file, we need to move it out of the directory and into the main project directory:


```
mv ~/myapp/handlers/wsgihandler.py ~/myapp

```


With the WSGI handler in the main project directory, we can check that uWSGI is able to serve the application by typing:


```
uwsgi --http :8000 --chdir ~/myapp -w wsgihandler:application

```


This should start up the application again on port 8000.  This time, since we aren’t using the SSL certificate and key, it will be served over plain HTTP instead of HTTPS.  You can test this in your browser again using the http protocol.  You will be unable to test the admin interface because web2py disables this when encryption is not available.


When you are finished, type CTRL-C in your terminal window to stop the server.


## Creating a uWSGI Configuration File


Now that we know that uWSGI can serve the application, we can create a uWSGI configuration file with our application’s information.


Create a directory at /etc/uwsgi/sites to store our configurations and then move into that directory:


```
sudo mkdir -p /etc/uwsgi/sites
cd /etc/uwsgi/sites

```


We will call our configuration file myapp.ini:


```
sudo nano myapp.ini

```


In the configuration file, we need to start with a [uwsgi] header under which all of our configuration directives will be placed.  After the header, we will indicate the directory path of our application and tell it the module to execute.  This will be the same information we used on the command line earlier.  You do not need to modify the module line:


```
[uwsgi]
chdir = /home/user/myapp
module = wsgihandler:application

```


Next, we need to specify that we want uWSGI to operate in master mode.  We want to spawn five worker processes:


```
[uwsgi]
chdir = /home/user/myapp
module = wsgihandler:application

master = true
processes = 5

```


Next, we need to specify how we want uWSGI to get connections.  In our test of the uWSGI server, we accepted normal HTTP connections.  However, since we will be configuring Nginx as a reverse proxy in front of uWSGI, we have other options.  Nginx can proxy using the uwsgi protocol, a fast binary protocol designed by uWSGI for communicating with other servers.  We will communicate using this protocol, which is the default if we do not specify another protocol.


Since we are communicating with Nginx using the uwsgi protocol, we don’t need a network port.  Instead, we will use a Unix socket, which is more secure and faster.  We will place this in our application directory.  We need to modify the permissions so that the group can read and write to the socket.  In a moment, we will be giving the Nginx process group ownership of the socket so that uWSGI and Nginx can communicate through the socket:


```
[uwsgi]
chdir = /home/user/myapp
module = wsgihandler:application

master = true
processes = 5

socket = /home/user/myapp/myapp.sock
chmod-socket = 660
vacuum = true

```


The vacuum directive above will clean up the socket file when the uWSGI process ends.


Our uWSGI configuration file is now complete.  Save and close the file.


## Create a uWSGI Upstart File


We have created a configuration file for uWSGI, but we still have not set up our application server to start automatically at boot.  To implement this functionality, we can create a simple Upstart file.  We will tell it to run uWSGI in “Emperor mode”, which allows the application server to read any number of configurations and start the server for each.


Create a file in the /etc/init directory where the Upstart process looks for its configuration files:


```
sudo nano /etc/init/uwsgi.conf

```


We will start by giving our service file a description and indicating which runlevels we want to start it automatically on.  The conventional multi-user runlevels are 2, 3, 4, and 5.  We will have Upstart stop the service when the server goes to other runlevels (like during shutdown, reboot, or single-user mode):


```
description "uWSGI application server in Emperor mode"

start on runlevel [2345]
stop on runlevel [!2345]

```


Next, we need to specify the user and group to run the process under.  We will be using our normal user account since it owns all of our project files.  For our group, we need to allow the www-data group ownership, which is the group that Nginx operates under.  This will allow the web server to freely communicate with uWSGI since our uWSGI config gave the socket group read and write privileges.


Afterwards, we simply need to specify the command to run to start uWSGI.  We just need to use the --emperor flag and pass it the directory containing our configuration file:


```
description "uWSGI application server in Emperor mode"

start on runlevel [2345]
stop on runlevel [!2345]

setuid user
setgid www-data

exec /usr/local/bin/uwsgi --emperor /etc/uwsgi/sites

```


This will start a uWSGI application server to handle our web2py site.  Emperor mode allows us to easily add configuration files in this directory for other projects.  They will be handled automatically.


We are now finished with our Upstart script.  Save and close the file.  At this point, we cannot start the uWSGI service because we have not yet installed Nginx.  This means that the group we told our script to run under is not available yet.


# Install and Configure Nginx as a Reverse Proxy


With uWSGI configured and ready to go, we can now install and configure Nginx as our reverse proxy.  This can be downloaded from Ubuntu’s default repositories:


```
sudo apt-get install nginx

```


Once Nginx is installed, we can go ahead and modify the server block configuration.  We will use the default server block as a base, since it has most of what we need:


```
sudo nano /etc/nginx/sites-available/default

```


The web2py application detects whether you are connecting with plain HTTP or with SSL encryption.  Because of this, our file will actually contain one server block for each.  We will start with the server block that is configured to operate on port 80.


Change the server_name to reference the domain name or IP address where your site should be accessible.  Afterwards, we will create a location {} block that will match requests for static content.  Basically, we want to use regular expressions to match requests ending with /static/ with a preceding path component.  We want to map these requests to the applications directory within our web2py project.  Make sure you reference your user’s home directory and app name:


```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.html index.htm;

    server_name server_domain_or_IP;

    location ~* /(\w+)/static/ {
        root /home/user/myapp/applications/;
    }

    . . .

```


Afterwards, we need to adjust the location / {} block to pass requests to our uWSGI socket.  We just need to include the uWSGI parameter file packaged with Nginx and pass the requests to the socket we configured (in our uWSGI .ini file):


```
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.html index.htm;

    server_name server_domain_or_IP;

    location ~* /(\w+)/static/ {
        root /home/user/myapp/applications/;
    }

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/user/myapp/myapp.sock;
    }
}

```


This will be all that we need for our first server block.


At the bottom of the file, there is a commented out section that has most of the directives needed to serve content with SSL.  You can identify this block because it has listen 443; as the first directive.  Uncomment this block to begin configuring it.


To start, change the server_name again to match your server’s domain name or IP address.  We can then jump to the ssl_certificate and ssl_certificate_key directives.  We will be placing the SSL certificates we generated into a directory at /etc/nginx/ssl momentarily, so specify the path to the files at that location:


```
server {
    listen 443;
    server_name server_domain_or_IP;

    root html;
    index index.html index.htm;

    ssl on;
    ssl_certificate /etc/nginx/ssl/myapp.crt;
    ssl_certificate_key /etc/nginx/ssl/myapp.key;

    . . .

```


In the ssl_protocols list, remove SSLv3, as has been found to have vulnerabilities inherent in the protocol itself.


We can then jump to the location / {} block and put the same uWSGI proxying information we did in the last server block:


```
server {
    listen 443;
    server_name server_domain_or_IP;

    root html;
    index index.html index.htm;

    ssl on;
    ssl_certificate /etc/nginx/ssl/myapp.crt;
    ssl_certificate_key /etc/nginx/ssl/myapp.key;

    ssl_session_timeout 5m;

    #ssl_protocols SSLv3 TLSv1 TLSv1.1 TLSv1.2;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers "HIGH:!aNULL:!MD5 or HIGH:!aNULL:!MD5:!3DES";
    ssl_prefer_server_ciphers on;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/user/myapp/myapp.sock;
    }
}

```


You should now have two server blocks configured in this file.  Save and close it when you are finished.


# Final Steps


Next, we need to move the SSL certificates to the directory we specified.  Create the directory first:


```
sudo mkdir -p /etc/nginx/ssl

```


Now, move the certificate and key you created to that directory.  If you have an SSL certificate signed by a commercial certificate authority, you can substitute the certificate and corresponding key here in order to avoid untrusted SSL certificate warnings for your visitors:


```
sudo mv ~/myapp/myapp.crt /etc/nginx/ssl
sudo mv ~/myapp/myapp.key /etc/nginx/ssl

```


Modify the permissions so non-root users cannot access that directory:


```
sudo chmod 700 /etc/nginx/ssl

```


Now, check your Nginx configuration file for syntax errors:


```
sudo nginx -t

```


If no syntax errors are reported, we can go ahead and restart the Nginx:


```
sudo service nginx restart

```


We can also start our uWSGI service:


```
sudo service uwsgi start

```


The last thing we need to do is copy our application’s parameters file so that it will be read correctly when serving connections on port 443.  This contains the password we configured for the administrative interface.  We just need to copy it to a new name that indicates port 443 instead of port 8000:


```
cp ~/myapp/parameters_8000.py ~/myapp/parameters_443.py

```


With that, you should be able to access your server using your server’s domain name or IP address.  Use https if you wish to sign into the administrative interface.


# Conclusion


In this guide, we’ve set up a sample web2py project to practice deployment.  We’ve configured uWSGI to act as an interface between our application and client requests.  We then set up Nginx in front of uWSGI to allow for SSL connections and to efficiently process client requests.


The web2py project simplifies the development of sites and web applications by providing a workable web interface from the very beginning.  By leveraging the general tool chain described in this article, you can easily serve the applications you create from a single server.


