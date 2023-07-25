# How To Set Up uWSGI and Nginx to Serve Python Apps on CentOS 7

```Python``` ```Python Frameworks``` ```Nginx``` ```CentOS```

## Introduction


In this guide, we will be setting up a simple WSGI application served by uWSGI.  We will use the Nginx web server as a reverse proxy to the application server to provide more robust connection handling.  We will be installing and configuring these components on a CentOS 7 server.


# Definitions and Concepts


## Clarifying Some Terms


Before we jump in, we should address some confusing terminology associated with the interrelated concepts we will be dealing with.  These three separate terms that appear interchangeable, but actually have distinct meanings:


- WSGI: A Python spec that defines a standard interface for communication between an application or framework and an application/web server.  This was created in order to simplify and standardize communication between these components for consistency and interchangeability.  This basically defines an API interface that can be used over other protocols.
- uWSGI: An application server container that aims to provide a full stack for developing and deploying web applications and services.  The main component is an application server that can handle apps of different languages.  It communicates with the application using the methods defined by the WSGI spec, and with other web servers over a variety of other protocols.  This is the piece that translates requests from a conventional web server into a format that the application can process.
- uwsgi: A fast, binary protocol implemented by the uWSGI server to communicate with a more full-featured web server.  This is a wire protocol, not a transport protocol.  It is the preferred way to speak to web servers that are proxying requests to uWSGI.

## WSGI Application Requirements


The WSGI spec defines the interface between the web server and application portions of the stack.  In this context, “web server” refers to the uWSGI server, which is responsible for translating client requests to the application using the WSGI spec.  This simplifies communication and creates loosely coupled components so that you can easily swap out either side without much trouble.


The web server (uWSGI) must have the ability to send requests to the application by triggering a defined “callable”.  The callable is simply an entry point into the application where the web server can call a function with some parameters.  The expected parameters are a dictionary of environmental variables and a callable provided by the web server (uWSGI) component.


In response, the application returns an iterable that will be used to generate the body of the client response.  It will also call the web server component callable that it received as a parameter.  The first parameter when triggering the web server callable will be the HTTP status code and the second will be a list of tuples, each of which define a response header and value to send back to the client.


With the “web server” component of this interaction provided by uWSGI in this instance, we will only need to make sure our applications have the qualities described above.  We will also set up Nginx to handle actual client requests and proxy them to the uWSGI server.


# Install the Components


To get started, we will need to install the necessary components on our CentOS 7 server.  We can mainly do this using yum and pip.


First, we need to install the EPEL repository so that we have access to a wider range of packages.  We can do that easily in a single yum command by typing:


```
sudo yum install epel-release

```


Now, we can install our components.  We need to get the Python development libraries and headers, the pip Python package manager, and the Nginx web server and reverse proxy.  We will also need a compiler to build the uWSGI binary momentarily:


```
sudo yum install python-pip python-devel nginx gcc

```


Once the package installation is complete, you will have access to the pip Python package manager.  We can use this to install the virtualenv package, which we will use to isolate our application’s Python environment from any others that may exist on the system:


```
sudo pip install virtualenv

```


Once this is complete, we can begin to create the general structure for our application.  We will create the virtual environment discussed above and will install the uWSGI application server within this environment.


# Set up an App Directory and a Virtualenv


We will start by creating a folder for our app.  This can hold a nested folder containing the actual application code in a more complete application.  For our purposes, this directory will simply hold our virtual environment and our WSGI entry point:


```
mkdir ~/myapp/

```


Next, move into the directory so that we can set up the environment for our application:


```
cd ~/myapp

```


Create a virtual environment with the virtualenv command.  We will call this myappenv for simplicity:


```
virtualenv myappenv

```


A new Python environment will be set up under a directory called myappenv.  We can activate this environment by typing:


```
source myappenv/bin/activate

```


Your prompt should change to indicate that you are now operating within the virtual environment.  It will look something like this:


```
(myappenv)username@host:~/my_app$

```


If you wish to leave this environment at any time, you can simply type:


```
deactivate

```


If you have deactivated your environment, re-activate it again to continue with the guide.


With this environment active, any Python packages installed will be contained within this directory hierarchy.  They will not interfere with the system’s Python environment.  With this in mind, we can now install the uWSGI server into our environment using pip.  The package for this is called uwsgi (this is still the uWSGI server and not the uwsgi protocol):


```
pip install uwsgi

```


You can verify that it is now available by typing:


```
uwsgi --version

```


If it returns a version number, the uWSGI server is available for use.


# Create a WSGI Application


Next, we will create an incredibly simple WSGI application using the WSGI specification requirements we discussed earlier.  To reiterate, the application component that we must provide should have the following properties:


- It must provide an interface through a callable (a function or other language construct that can be called)
- The callable must take as parameters a dictionary containing environmental variable-like key-value pairs and a callable that is accessible on the server (uWSGI).
- The application’s callable should return an iterable that will produce the body to send the client.
- The application should call the web server’s callable with the HTTP status and request headers.

We will write our application in a file called wsgi.py in our application directory:


```
nano ~/myapp/wsgi.py

```


Inside of this file, we will create the simplest WSGI compliant application we can.  As with all Python code, be sure to pay attention to the indentation:


```
def application(environ, start_response):
    start_response('200 OK', [('Content-Type', 'text/html')])
    return ["<h1 style='color:blue'>Hello There!</h1>"]

```


The above code constitutes a complete WSGI application.  By default, uWSGI will look for a callable called application, which is why we called our function application.  As you can see, it takes two parameters.


The first we called environ because it will be an environmental variable-like key-value dictionary.  The second is called start_response and is the name the app will use internally to refer to the web server (uWSGI) callable that is sent in.  Both of these parameter names were simply selected because of their use in the examples in the PEP 333 spec that defines WSGI interactions.


Our application must take this information and do two things.  First, it must call the callable it received with an HTTP status code and any headers it wants to send back.  In this case, we are sending a “200 OK” response and setting the Content-Type header to text/html.


Secondly, it needs to return with an iterable to use as the response body.  Here, we’ve just used a list containing a single string of HTML.  Strings are iterable as well, but inside of a list, uWSGI will be able to process the entire string with one iteration.


In a real world scenario, this file would likely be used as a link to the rest of your application code.  For instance, Django projects include a wsgi.py file by default that translates requests from the web server (uWSGI) to the application (Django).  The simplified WSGI interface stays the same regardless of how complex the actual application code is.  This is one of the strengths of the interface.


Save and close the file when you are finished.


To test out the code, we can start up uWSGI.  We will tell it to use HTTP for the time being and to listen on port 8080.  We will pass it the name of the script (suffix removed):


```
uwsgi --socket 0.0.0.0:8080 --protocol=http -w wsgi

```


Now, if you visit your server’s IP address or domain name in your web browser followed by :8080, you should see the first level header text we passed as the body in our wsgi.py file:





Stop the server with CTRL-C when you have verified that this works.


We’re done with designing our actual application at this point.  You can deactivate our virtual environment if you desire:


```
deactivate

```


# Configure a uWSGI Config File


In the above example, we manually started the uWSGI server and passed it some parameters on the command line.  We can avoid this by creating a configuration file.  The uWSGI server can read configurations in a variety of formats, but we will use the .ini format for simplicity.


To continue with the naming we’ve been using thus far, we’ll call the file myapp.ini and place it in our application folder:


```
nano ~/myapp/myapp.ini

```


Inside, we need to establish a section called [uwsgi].  This section is where all of our configuration items will live.  We’ll start by identifying our application.  The uWSGI server needs to know where the application’s callable is.  We can give the file and the function within:


```
[uwsgi]
module = wsgi:application

```


We want to mark the initial uwsgi process as a master and then spawn a number of worker processes.  We will start with five workers:


```
[uwsgi]
module = wsgi:application

master = true
processes = 5

```


We are actually going to change the protocol that uWSGI uses to speak with the outside world.  When we were testing our application, we specified --protocol=http so that we could see it from a web browser.  Since we will be configuring Nginx as a reverse proxy in front of uWSGI, we can change this.  Nginx implements a uwsgi proxying mechanism, which is a fast binary protocol that uWSGI can use to talk with other servers.  The uwsgi protocol is actually uWSGI’s default protocol, so simply by omitting a protocol specification, it will fall back to uwsgi.


Since we are designing this config for use with Nginx, we’re also going to change from using a network port and use a Unix socket instead.  This is more secure and faster.


We will specify our own user’s name to run the uwsgi server and own the socket file.  We will create a directory under /run to place the socket file so that both uWSGI and Nginx can access it. We’ll call the socket itself myapp.sock.  We will change the permissions to “664” so that Nginx can write to it (we will be starting uWSGI with the www-data group that Nginx uses.  We’ll also add the vacuum option, which will remove the socket when the process stops:


```
[uwsgi]
module = wsgi:application

master = true
processes = 5

uid = user
socket = /run/uwsgi/myapp.sock
chown-socket = user:nginx
chmod-socket = 660
vacuum = true

```


We need one final option since we will be creating a systemd file to start our application at boot.  Systemd and uWSGI have different ideas about what the SIGTERM signal should do to an application.  To sort out this discrepancy so that the processes can be handled as expected with Systemd, we just need to add an option called die-on-term so that uWSGI will kill the process instead of reloading it:


```
[uwsgi]
module = wsgi:application

master = true
processes = 5

uid = user
socket = /run/uwsgi/myapp.sock
chown-socket = user:nginx
chmod-socket = 660
vacuum = true

die-on-term = true

```


Save and close the file when you are finished.  This configuration file is now set to be used with an Upstart script.


# Create a Systemd Unit File to Manage the App


We can launch a uWSGI instance at boot so that our application is always available.  To do this, we can create a systemd unit file.  We will place this in the /etc/systemd/system directory which is the best place for user-created unit files.  We’ll be calling the unit file uwsgi.service:


```
sudo nano /etc/systemd/system/uwsgi.service

```


First, we start out with the [Unit] section, where we can pace our metadata.  The only thing we will be putting here is a description of our service:


```
[Unit]
Description=uWSGI instance to serve myapp

```


Next, we will open up the [Service] section.  Because we are using a virtual environment, our service start commands will be more complex than they traditionally would be.  We will use an ExecStartPre command to make sure our socket directory is created and owned by the correct parties.  This will be allowed to fail (by putting an - after the equal sign) in case they are already set up.  This will be passed into a single call to bash.


For the actual ExecStart command which will start uWSGI, we will also pass the actual commands to bash.  This allows us to execute a few different commands since only a single command (bash in this case) can be run by this directive.  We will use this to change to our application directory, activate the virtual environment, and start uWSGI with the .ini file we created:


```
[Unit]
Description=uWSGI instance to serve myapp

[Service]
ExecStartPre=-/usr/bin/bash -c 'mkdir -p /run/uwsgi; chown user:nginx /run/uwsgi'
ExecStart=/usr/bin/bash -c 'cd /home/user/myapp; source myappenv/bin/activate; uwsgi --ini myapp.ini'

```


Now, all that is left to do is formulate the [Install] section.  This will determine what happens when we enable the unit.  Basically, it specifies which states the unit should autostart.  We want to specify that when enabled, this unit should start whenever the server is in multi-user mode:


```
[Unit]
Description=uWSGI instance to serve myapp

[Service]
ExecStartPre=-/usr/bin/bash -c 'mkdir -p /run/uwsgi; chown user:nginx /run/uwsgi'
ExecStart=/usr/bin/bash -c 'cd /home/user/myapp; source myappenv/bin/activate; uwsgi --ini myapp.ini'

[Install]
WantedBy=multi-user.target

```


Once you have the above configuration written out, save and close the file.


Now, we can start the service by typing:


```
sudo systemctl start uwsgi

```


Check that it started without issues by typing:


```
systemctl status uwsgi

```


If there were no errors, enable the service so that it starts at boot by typing:


```
sudo systemctl enable uwsgi

```


You can stop the service at any time by typing:


```
sudo systemctl stop uwsgi

```


# Configure Nginx to Proxy to uWSGI


At this point, we have a WSGI app and have verified that uWSGI can read and serve it.  We have created a configuration file and Systemd unit file.  Our uWSGI process will listen on a socket and communicate using the uwsgi protocol.


We’re now at the point where we can work on configuring Nginx as a reverse proxy.  Nginx has the ability to proxy using the uwsgi protocol for communicating with uWSGI.  This is a faster protocol than HTTP and will perform better.


The Nginx configuration that we will be setting up is extremely simple.  We will be modifying the existing nginx.conf file and adding a new server block.  Open the file with sudo for editing:


```
sudo nano /etc/nginx/nginx.conf

```


Before the default server block, we will add our own server block:


```
http {

    . . .

    include /etc/nginx/conf.d/*.conf;

    server {
    }

    server {
        listen 80 default_server;
        server_name localhost;

        . . .

```


The block we created will hold the configuration for our uWSGI proxy.  The rest of the configuration items below are placed within this block.  The server block should listen on port 80 and respond to your server’s domain name or IP address:


```
server {
    listen 80;
    server_name server_domain_or_IP;
}

```


Afterwards, we can open up a single location block that will handle all requests.  Within this block, we will include the uwsgi parameters found in the /etc/nginx/uwsgi_params file, and we will pass the traffic to the socket where uWSGI is listening:


```
server {
    listen 80;
    server_name server_domain_or_IP;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/run/uwsgi/myapp.sock;
    }
}

```


That is actually all we need for a simple application.  There are some improvements that could be made for a more complete application.  For instance, we might define a number of upstream uWSGI servers outside of this block and then pass them to that.  We might include some more uWSGI parameters.  We might also handle any static files from Nginx directly and pass only dynamic requests to the uWSGI instance.


We do not need any of those features in our three-line app though, so we can save and close the file.


You can test to make sure that your Nginx configuration is valid by typing:


```
sudo nginx -t

```


If this returns without any errors, start the service by typing:


```
sudo systemctl start nginx

```


Start Nginx at boot by enabling the service:


```
sudo systemctl enable nginx

```


You should be able to go to your server’s domain name or IP address (without a port number) and see the application you configured:





# Conclusion


If you’ve made it this far, you’ve created a simple WSGI application and have some insight into how more complex applications would need to be designed.  We have installed the uWSGI application container/server into a purpose-made virtual environment to serve our application.  We’ve made a configuration file and a Systemd unit file to automate this process.  In front of the uWSGI server, we’ve set up an Nginx reverse proxy that can speak to the uWSGI process using the uwsgi wire protocol.


You can easily see how this can be expanded when setting up an actual production environment.  For instance, uWSGI has the ability to manage multiple applications using something called “emperor mode”.  You can expand the Nginx configuration to load balance between uWSGI instances, or to handle static files for your application.  When serving multiple applications, it may be in your best interest to install uWSGI globally instead of in a virtual environment, depending on your needs.  The components are all fairly flexible, so you should be able to tweak their configuration to accommodate many different scenarios.


