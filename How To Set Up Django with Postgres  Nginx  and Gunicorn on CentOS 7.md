# How To Set Up Django with Postgres  Nginx  and Gunicorn on CentOS 7

```Python``` ```Deployment``` ```Django``` ```Nginx``` ```Python Frameworks``` ```CentOS```

## Introduction


Django is a powerful web framework that can help you get your Python application or website off the ground.  Django includes a simplified development server for testing your code locally, but for anything even slightly production related, a more secure and powerful web server is required.


In this guide, we will demonstrate how to install and configure some components on CentOS 7 to support and serve Django applications.  We will be setting up a PostgreSQL database instead of using the default SQLite database.  We will configure the Gunicorn application server to interface with our applications.  We will then set up Nginx to reverse proxy to Gunicorn, giving us access to its security and performance features to serve our apps.


# Prerequisites and Goals


In order to complete this guide, you should have a fresh CentOS 7 server instance with a non-root user with sudo privileges configured.  You can learn how to set this up by running through our initial server setup guide.


We will be installing Django within a virtual environment.  Installing Django into an environment specific to your project will allow your projects and their requirements to be handled separately.


Once we have our database and application up and running, we will install and configure the Gunicorn application server.  This will serve as an interface to our application, translating client requests in HTTP to Python calls that our application can process.  We will then set up Nginx in front of Gunicorn to take advantage of its high performance connection handling mechanisms and its easy-to-implement security features.


Let’s get started.


# Install the Packages from EPEL and the CentOS Repositories


To begin the process, we’ll download and install all of the items we need from the CentOS repositories.  We will also need to use the EPEL repository, which contains extra packages that aren’t included in the main CentOS repositories.  Later we will use the Python package manager pip to install some additional components.


First, enable the EPEL repository so that we can get the components we need:


```
sudo yum install epel-release

```


With the new repository available, we can install all of the pieces we need in one command:


```
sudo yum install python-pip python-devel postgresql-server postgresql-devel postgresql-contrib gcc nginx 

```


This will install pip, the Python package manager.  It will also install the PostgreSQL database system and some libraries and other files we will need to interact with it and build off of it.  We included the GCC compiler so that pip can build software and we installed Nginx to use as a reverse proxy for our installation.


# Set Up PostgreSQL for Django


We’re going to jump right in and set PostgreSQL up for our installation.


## Configure and Start PostgreSQL


To start off, we need to initialize the PostgreSQL database.  We can do that by typing:


```
sudo postgresql-setup initdb

```


After the database has been initialized, we can start the PostgreSQL service by typing:


```
sudo systemctl start postgresql

```


With the database started, we actually need to adjust the values in one of the configuration files that has been populated.  Use your editor and the sudo command to open the file now:


```
sudo nano /var/lib/pgsql/data/pg_hba.conf

```


This file is responsible for configuring authentication methods for the database system.  Currently, it is configured to allow connections only when the system user matches the database user.  This is okay for local maintenance tasks, but our Django instance will have another user configured with a password.


We can configure this by modifying the two host lines at the bottom of the file.  Change the last column (the authentication method) to md5.  This will allow password authentication:


```
. . .

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
#host    all             all             127.0.0.1/32            ident
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
#host    all             all             ::1/128                 ident
host    all             all             ::1/128                 md5

```


When you are finished, save and close the file.


With our new configuration changes, we need to restart the service.  We will also enable PostgreSQL so that it starts automatically at boot:


```
sudo systemctl restart postgresql
sudo systemctl enable postgresql

```


## Create the PostgreSQL Database and User


Now that we have PostgreSQL up and running the way we want it to, we can create a database and database user for our Django application.


To work with Postgres locally, it is best to change to the postgres system user temporarily.  Do that now by typing:


```
sudo su - postgres

```


When operating as the postgres user, you can log right into a PostgreSQL interactive session with no further authentication.  This is due to the line we didn’t change in the pg_hba.conf file:


```
psql

```


You will be given a PostgreSQL prompt where we can set up our requirements.


First, create a database for your project:


```
CREATE DATABASE myproject;

```


Every command must end with a semi-colon, so check that your command ends with one if you are experiencing issues.


Next, create a database user for our project.  Make sure to select a secure password:


```
CREATE USER myprojectuser WITH PASSWORD 'password';

```


Now, we can give our new user access to administer our new database:


```
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;

```


When you are finished, exit out of the PostgreSQL prompt by typing:


```
\q

```


Now, exit out of the postgres user’s shell session to get back to your normal user’s shell session by typing:


```
exit

```


# Create a Python Virtual Environment for your Project


Now that we have our database ready, we can begin getting the rest of our project requirements ready.  We will be installing our Python requirements within a virtual environment for easier management.


To do this, we first need access to the virtualenv command.  We can install this with pip:


```
sudo pip install virtualenv

```


With virtualenv installed, we can start forming our project.  Create a directory where you wish to keep your project and move into the directory afterwards:


```
mkdir ~/myproject
cd ~/myproject

```


Within the project directory, create a Python virtual environment by typing:


```
virtualenv myprojectenv

```


This will create a directory called myprojectenv within your myproject directory.  Inside, it will install a local version of Python and a local version of pip.  We can use this to install and configure an isolated Python environment for our project.


Before we install our project’s Python requirements, we need to activate the virtual environment.  You can do that by typing:


```
source myprojectenv/bin/activate

```


Your prompt should change to indicate that you are now operating within a Python virtual environment.  It will look something like this: (myprojectenv)user@host:~/myproject$.


With your virtual environment active, install Django, Gunicorn, and the psycopg2 PostgreSQL adaptor with the local instance of pip:


```
pip install django gunicorn psycopg2

```


# Create and Configure a New Django Project


With our Python components installed, we can create the actual Django project files.


## Create the Django Project


Since we already have a project directory, we will tell Django to install the files here.  It will create a second level directory with the actual code, which is normal, and place a management script in this directory.  The key to this is the dot at the end that tells Django to create the files in the current directory:


```
django-admin.py startproject myproject .

```


## Adjust the Project Settings


The first thing we should do with our newly created project files is adjust the settings.  Open the settings file in your text editor:


```
nano myproject/settings.py

```


Start by finding the section that configures database access.  It will start with DATABASES.  The configuration in the file is for a SQLite database.  We already created a PostgreSQL database for our project, so we need to adjust the settings.


Change the settings with your PostgreSQL database information.  We tell Django to use the psycopg2 adaptor we installed with pip.  We need to give the database name, the database username, the database username’s password, and then specify that the database is located on the local computer.  You can leave the PORT setting as an empty string:


```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject',
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}


```


Next, move down to the bottom of the file and add a setting indicating where the static files should be placed.  This is necessary so that Nginx can handle requests for these items.  The following line tells Django to place them in a directory called static in the base project directory:


```
STATIC_ROOT = os.path.join(BASE_DIR, "static/")

```


Save and close the file when you are finished.


## Complete Initial Project Setup


Now, we can migrate the initial database schema to our PostgreSQL database using the management script:


```
cd ~/myproject
./manage.py makemigrations
./manage.py migrate

```


Create an administrative user for the project by typing:


```
./manage.py createsuperuser

```


You will have to select a username, provide an email address, and choose and confirm a password.


We can collect all of the static content into the directory location we configured by typing:


```
./manage.py collectstatic

```


You will have to confirm the operation.  The static files will then be placed in a directory called static within your project directory.


Finally, you can test your project by starting up the Django development server with this command:


```
./manage.py runserver 0.0.0.0:8000

```


In your web browser, visit your server’s domain name or IP address followed by :8000:


```
http://server_domain_or_IP:8000

```


You should see the default Django index page:





If you append /admin to the end of the URL in the address bar, you will be prompted for the administrative username and password you created with the createsuperuser command:





After authenticating, you can access the default Django admin interface:





When you are finished exploring, hit CTRL-C in the terminal window to shut down the development server.


## Testing Gunicorn’s Ability to Serve the Project


The last thing we want to do before leaving our virtual environment is test Gunicorn to make sure that it can serve the application.  We can do this easily by typing:


```
cd ~/myproject
gunicorn --bind 0.0.0.0:8000 myproject.wsgi:application

```


This will start Gunicorn on the same interface that the Django development server was running on.  You can go back and test the app again.  Note that the admin interface will not have any of the styling applied since Gunicorn does not know about the static content responsible for this.


We passed Gunicorn a module by specifying the relative directory path to Django’s wsgi.py file, which is the entry point to our application, using Python’s module syntax.  Inside of this file, a function called application is defined, which is used to communicate with the application.  To learn more about the WSGI specification, click here.


When you are finished testing, hit CTRL-C in the terminal window to stop Gunicorn.


We’re now finished configuring our Django application.  We can back out of our virtual environment by typing:


```
deactivate

```


# Create a Gunicorn Systemd Service File


We have tested that Gunicorn can interact with our Django application, but we should implement a more robust way of starting and stopping the application server.  To accomplish this, we’ll make a Systemd service file.


Create and open a Systemd service file for Gunicorn with sudo privileges in your text editor:


```
sudo nano /etc/systemd/system/gunicorn.service

```


Start with the [Unit] section, which is used to specify metadata and dependencies.  We’ll put a description of our service here and tell the init system to only start this after the networking target has been reached:


```
[Unit]
Description=gunicorn daemon
After=network.target

```


Next, we’ll open up the [Service] section.  We’ll specify the user and group that we want to process to run under.  We will give our regular user account ownership of the process since it owns all of the relevant files.  We’ll give the Nginx user group ownership so that it can communicate easily with Gunicorn.


We’ll then map out the working directory and specify the command to use to start the service.  In this case, we’ll have to specify the full path to the Gunicorn executable, which is installed within our virtual environment.  We will bind it to a Unix socket within the project directory since Nginx is installed on the same computer.  This is safer and faster than using a network port.  We can also specify any optional Gunicorn tweaks here.  For example, we specified 3 worker processes in this case:


```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=user
Group=nginx
WorkingDirectory=/home/user/myproject
ExecStart=/home/user/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/user/myproject/myproject.sock myproject.wsgi:application

```


Finally, we’ll add an [Install] section.  This will tell Systemd what to link this service to if we enable it to start at boot.  We want this service to start when the regular multi-user system is up and running:


```
[Unit]
Description=gunicorn daemon
After=network.target

[Service]
User=user
Group=nginx
WorkingDirectory=/home/user/myproject
ExecStart=/home/user/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/user/myproject/myproject.sock myproject.wsgi:application

[Install]
WantedBy=multi-user.target

```


With that, our Systemd service file is complete.  Save and close it now.


We can now start the Gunicorn service we created and enable it so that it starts at boot:


```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn

```


# Configure Nginx to Proxy Pass to Gunicorn


Now that Gunicorn is set up, we need to configure Nginx to pass traffic to the process.


## Modify the Nginx Configuration File


We can go ahead and modify the server block configuration by editing the main Nginx configuration file:


```
sudo nano /etc/nginx/nginx.conf

```


Inside, open up a new server block just above the server block that is already present:


```
http {
    . . .

    include /etc/nginx/conf.d/*.conf;

    server {
    }

    server {
        listen 80 default_server;

        . . .

```


We will put all of the configuration for our Django application inside of this new block.  We will start by specifying that this block should listen on the normal port 80 and that it should respond to our server’s domain name or IP address:


```
server {
    listen 80;
    server_name server_domain_or_IP;
}

```


Next, we will tell Nginx to ignore any problems with finding a favicon.  We will also tell it where to find the static assets that we collected in our ~/myproject/static directory.  All of these files have a standard URI prefix of “/static”, so we can create a location block to match those requests:


```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }
}

```


Finally, we’ll create a location / {} block to match all other requests.  Inside of this location, we’ll set some standard proxying HTTP headers so that Gunicorn can have some information about the remote client connection.  We will then pass the traffic to the socket we specified in our Gunicorn Systemd unit file:


```
server {
    listen 80;
    server_name server_domain_or_IP;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/user/myproject;
    }

    location / {
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_pass http://unix:/home/user/myproject/myproject.sock;
    }
}

```


Save and close the file when you are finished.


## Adjust Group Membership and Permissions


The nginx user must have access to our application directory so that it can serve static files, access the socket files, etc.  CentOS locks down each user’s home directory very restrictively, so we will add the nginx user to our user’s group so that we can then open up the minimum permissions necessary to get this to function.


Add the nginx user to your group with the following command.  Substitute your own username for the user in the command:


```
sudo usermod -a -G user nginx

```


Now, we can give our user group execute permissions on our home directory.  This will allow the Nginx process to enter and access content within:


```
chmod 710 /home/user

```


With the permissions set up, we can test our Nginx configuration file for syntax errors:


```
sudo nginx -t

```


If no errors are present, restart the Nginx service by typing:


```
sudo systemctl start nginx

```


Tell the init system to start the Nginx server at boot by typing:


```
sudo systemctl enable nginx

```


You should now have access to your Django application in your browser over your server’s domain name or IP address without specifying a port.


# Conclusion


In this guide, we’ve set up a Django project in its own virtual environment.  We’ve configured Gunicorn to translate client requests so that Django can handle them.  Afterwards, we set up Nginx to act as a reverse proxy to handle client connections and serve the correct project depending on the client request.


Django makes creating projects and applications simple by providing many of the common pieces, allowing you to focus on the unique elements.  By leveraging the general tool chain described in this article, you can easily serve the applications you create from a single server.


