# How To Use PostgreSQL with your Django Application on CentOS 7

```Python``` ```PostgreSQL``` ```Django``` ```Python Frameworks``` ```CentOS```

## Introduction


Django is a flexible framework for quickly creating Python applications.  By default, Django applications are configured to store data into a lightweight SQLite database file.  While this works well under some loads, a more traditional DBMS can improve performance in production.


In this guide, we’ll demonstrate how to install and configure PostgreSQL to use with your Django applications.  We will install the necessary software, create database credentials for our application, and then start and configure a new Django project to use this backend.


# Prerequisites


To get started, you will need a clean CentOS 7 server instance with a non-root user set up.  The non-root user must be configured with sudo privileges.  Learn how to set this up by following our initial server setup guide.


When you are ready to continue, read on.


# Install the Components from the CentOS and EPEL Repositories


Our first step will be install all of the pieces that we need from the repositories.  We will install pip, the Python package manager, in order to install and manage our Python components.  We will also install the database software and the associated libraries required to interact with them.


Some of the software we need is in the EPEL repository, which contains extra packages.  We can enable this repository easily by tying:


```
sudo yum install epel-release

```


With EPEL enabled, we can install the necessary components by typing:


```
sudo yum install python-pip python-devel gcc postgresql-server postgresql-devel postgresql-contrib

```


# Perform Initial PostgreSQL Configuration


After the installation, you need to initialize the PostgreSQL database by typing:


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


This file is responsible for configuring authentication methods for the database system. Currently, it is configured to allow connections only when the system user matches the database user. This is okay for local maintenance tasks, but our Django instance will have another user configured with a password.


We can configure this by modifying the two host lines at the bottom of the file. Change the last column (the authentication method) to md5. This will allow password authentication:


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


# Create a Database and Database User


By default, Postgres uses an authentication scheme called “peer authentication” for local connections.  We could see this for the local entry in the pg_hba.conf file we edited. Basically, this means that if the user’s operating system username matches a valid Postgres username, that user can login with no further authentication.


During the Postgres installation, an operating system user named postgres was created to correspond to the postgres PostgreSQL administrative user. We need to change to this user to perform administrative tasks:


```
sudo su - postgres

```


You should now be in a shell session for the postgres user. Log into a Postgres session by typing:


```
psql

```


First, we will create a database for our Django project. Each project should have its own isolated database for security reasons. We will call our database myproject in this guide, but it’s always better to select something more descriptive:


```
CREATE DATABASE myproject;

```


Remember to end all commands at an SQL prompt with a semicolon.


Next, we will create a database user which we will use to connect to and interact with the database. Set the password to something strong and secure:


```
CREATE USER myprojectuser WITH PASSWORD 'password';

```


Afterwards, we’ll modify a few of the connection parameters for the user we just created. This will speed up database operations so that the correct values do not have to be queried and set each time a connection is established.


We are setting the default encoding to UTF-8, which Django expects. We are also setting the default transaction isolation scheme to “read committed”, which blocks reads from uncommitted transactions. Lastly, we are setting the timezone. By default, our Django projects will be set to use UTC:


```
ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
ALTER ROLE myprojectuser SET timezone TO 'UTC';

```


Now, all we need to do is give our database user access rights to the database we created:


```
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;

```


Exit the SQL prompt to get back to the postgres user’s shell session:


```
\q

```


Exit out of the postgres user’s shell session to get back to your regular user’s shell session:


```
exit

```


# Install Django within a Virtual Environment


Now that our database is set up, we can install Django.  For better flexibility, we will install Django and all of its dependencies within a Python virtual environment.


You can get the virtualenv package that allows you to create these environments by typing:


```
sudo pip install virtualenv

```


Make a directory to hold your Django project.  Move into the directory afterwards:


```
mkdir ~/myproject
cd ~/myproject

```


We can create a virtual environment to store our Django project’s Python requirements by typing:


```
virtualenv myprojectenv

```


This will install a local copy of Python and pip into a directory called myprojectenv within your project directory.


Before we install applications within the virtual environment, we need to activate it. You can do so by typing:


```
source myprojectenv/bin/activate

```


Your prompt will change to indicate that you are now operating within the virtual environment. It will look something like this (myprojectenv)user@host:~/myproject$.


Once your virtual environment is active, you can install Django with pip.  We will also install the psycopg2 package that will allow us to use the database we configured:


```
pip install django psycopg2

```


We can now start a Django project within our myproject directory.  This will create a child directory of the same name to hold the code itself, and will create a management script within the current directory.  Make sure to add the dot at the end of the command so that this is set up correctly:


```
django-admin.py startproject myproject .

```


# Configure the Django Database Settings


Now that we have a project, we need to configure it to use the database we created.


Open the main Django project settings file located within the child project directory:


```
nano ~/myproject/myproject/settings.py

```


Towards the bottom of the file, you will see a DATABASES section that looks like this:


```
. . .

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

. . .

```


This is currently configured to use SQLite as a database.  We need to change this so that our PostgreSQL database is used instead.


First, change the engine so that it points to the postgresql_psycopg2 backend instead of the sqlite3 backend.  For the NAME, use the name of your database (myproject in our example).  We also need to add login credentials.  We need the username, password, and host to connect to.  We’ll add and leave blank the port option so that the default is selected:


```
. . .

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

. . .

```


When you are finished, save and close the file.


# Migrate the Database and Test your Project


Now that the Django settings are configured, we can migrate our data structures to our database and test out the server.


We can begin by creating and applying migrations to our database.  Since we don’t have any actual data yet, this will simply set up the initial database structure:


```
cd ~/myproject
python manage.py makemigrations
python manage.py migrate

```


After creating the database structure, we can create an administrative account by typing:


```
python manage.py createsuperuser

```


You will be asked to select a username, provide an email address, and choose and confirm a password for the account.


Once you have an admin account set up, you can test that your database is performing correctly by starting up the Django development server:


```
python manage.py runserver 0.0.0.0:8000

```


In your web browser, visit your server’s domain name or IP address followed by :8000 to reach default Django root page:


```
http://server_domain_or_IP:8000

```


You should see the default index page:





Append /admin to the end of the URL and you should be able to access the login screen to the admin interface:





Enter the username and password you just created using the createsuperuser command.  You will then be taken to the admin interface:





When you’re done investigating, you can stop the development server by hitting CTRL-C in your terminal window.


By accessing the admin interface, we have confirmed that our database has stored our user account information and that it can be appropriately accessed.


# Conclusion


In this guide, we’ve demonstrated how to install and configure PostgreSQL as the backend database for a Django project.  While SQLite can easily handle the load during development and light production use, most projects benefit from implementing a more full-featured DBMS.


