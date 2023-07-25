# How To Deploy a Local Django App to a VPS

```Ubuntu``` ```Django``` ```Nginx``` ```Debian```

# Prerequisites



This tutorial assumes that you have already set up your virtual private server with your operating system of choice (Debian 7 was used for this tutorial; Ubuntu will also work). If you have not already done so, you can follow this tutorial. Before you begin, make sure your cloud server is properly configured to host Django applications with a database server, web server, and virtualenv already installed. If you have not already done this, please follow steps 1 - 6 about setting up a server for Django.


# Step One: Update Packages



Before doing anything, it is always good practice to ensure all of your packages managed through apt, or whatever your package manager of choice is, are up to date. You can do this by connecting to your VPS via SSH and running the following commands:


```
sudo apt-get update

```


```
sudo apt-get upgrade

```


The first command downloads any updates for packages managed through apt-get. The second command installs the updates that were downloaded. After running the above commands, if there are updates to install you will likely be prompted to indicate whether or not you want to install these updates. If this happens, just type “y” and then hit “enter” when prompted.


# Step Two: Set Up Your Virtualenv



If you completed the prerequisites, this should already be set up and you can skip this step.


Now we need to set up our virtualenv where our project files and Python packages will live. If you don’t use virtualenv, then simply create the directory where your Django project will live and move to step three.


To create your virtualenv run the following command. Remember to replace the path with the desired path of your project project on the virtual private server:


```
virtualenv /opt/myproject

```


Now that you have your virtualenv set up, you may activate your virtualenv and install Django and any other Python packages you may need using pip. Below is an example of how to activate your virtualenv and use pip to install Django:


```
source /opt/myproject/bin/activate

```


```
pip install django

```


Now we’re ready to create a database for our project!


# Step Three: Create a Database



This tutorial assumes you use PostgreSQL as your database server. If not, you will need to check out documentation on how to create a database for your database server of choice.


To create a database with PostgreSQL start by running the following command:


```
sudo su - postgres

```


Your terminal prompt should now say “postgres@yourserver”. If so, run this command to create your database, making sure to replace “mydb” with your desired database name:


```
createdb mydb

```


Now create your database user with the following command:


```
createuser -P

```


You will now be met with a series of six prompts. The first will ask you for the name of the new user (use whatever name you would like). The next two prompts are for your password and confirmation of password for the new user. For the last three prompts, simply enter “n” and hit “enter.” This ensures your new user only has access to what you give it access to and nothing else. Now activate the PostgreSQL command line interface like so:


```
psql

```


Finally, grant this new user access to your new database with the following command:


```
GRANT ALL PRIVILEGES ON DATABASE mydb TO myuser;

```


You are now set with a database and a user to access that database. Next, we can work on configuring our web server to serve up our static files!


# Step Four: Configure Your VPS


We need to create a new configuration file for our site. This tutorial assumes you are using NGINX as your cloud server. If this is not the case, you will need to check the documentation for your web server of choice in order to complete this step.


For NGINX, run the following command to create and edit your site’s web server configuration file, making sure to replace “myproject” at the end of the command with the name of your project:


```
sudo nano /etc/nginx/sites-available/myproject

```


Now enter the following lines of code into the open editor:


```
server {
    server_name yourdomainorip.com;

    access_log off;

    location /static/ {
        alias /opt/myenv/static/;
    }

    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header X-Forwarded-Host $server_name;
        proxy_set_header X-Real-IP $remote_addr;
        add_header P3P 'CP="ALL DSP COR PSAa PSDa OUR NOR ONL UNI COM NAV"';
    }
}

```


Save and exit the file. The above configuration has set NGINX to serve anything requested at yourdomainorip.com/static/ from the static directory we set for our Django project. Anything requested at yourdomainorip.com will proxy to localhost on port 8001, which is where we will tell Gunicorn (or your app server of choice) to run. The other lines ensure that the hostname and IP address of the request get passed on to Gunicorn. Without this, the IP address of every request becomes 127.0.0.1 and the hostname is just your VPS hostname.


Now we need to set up a symbolic link in the /etc/nginx/sites-enabled directory that points to this configuration file. That is how NGINX knows this site is active. Change directories to /etc/nginx/sites-enabled like this:


```
cd /etc/nginx/sites-enabled

```


Once there, run this command:


```
sudo ln -s ../sites-available/myproject

```


Now restart NGINX with the command below and you should be set:


```
sudo service nginx restart

```


You may see the following error upon restart:


```
server_names_hash, you should increase server_names_hash_bucket_size: 32

```


You can resolve this by editing ’ /etc/nginx/nginx.conf ’


Open the file and uncomment the following line:


```
server_names_hash_bucket_size 64;


```


Now let’s get our project files pushed up to our droplet!


# Step Five: Move Local Django Project to Droplet



We have several options here: FTP, SFTP, SCP, Git, SVN, etc. We will go over using Git to transfer your local project files to your virtual private server.


Find the directory where you set up your virtualenv or where you want your project to live. Change into this directory with the following command:


```
cd /opt/myproject

```


Once there, create a new directory where your project files will live. You can do this with the following command:


```
mkdir myproject

```


It may seem redundant to have two directories with the same name; however, it makes it so that your virtualenv name and project name are the same.


Now change into the new directory with the following command:


```
cd myproject

```


If your project is already in a Git repo, simply make sure your code is all committed and pushed. You can check if this is the case by running the following command locally on your computer in a terminal (for Mac) or a command prompt (for PC):


```
git status

```


If you see no files in the output then you should be good to go. Now SSH to your droplet and install Git with the following command:


```
sudo apt-get install git

```


Make sure to answer yes to any prompts by entering “y” and hitting “enter.” Once Git is installed, use it to pull your project files into your project directory with the following command:


```
git clone https://webaddressforyourrepo.com/path/to/repo .

```


If you use Github or Bitbucket for Git hosting there is a clone button you can use to get this command. Be sure to add the “.” at the end. If we don’t do this, then Git will create a directory with the repo name inside your project directory, which you don’t want.


If you don’t use Git then use FTP or another transfer protocol to transfer your files to the project directory created in the steps above.


Now all that’s left is setting up your app server!


# Step Six: Install and Configure App Server



If you completed the prerequisites, this should already be set up and you can skip this step.


Now we need to install our app server and make sure it listens on port 8001 for requests to our Django app. We will use Gunicorn in this example. To install Gunicorn, first activate your virtualenv:


```
source /opt/myproject/bin/activate

```


Once your virtualenv is active, run the following command to install Gunicorn:


```
pip install gunicorn

```


Now that Gunicorn is installed, bind requests for your domain or ip to port 8001:


```
gunicorn_django --bind yourdomainorip.com:8001

```


Now you can hit “ctrl + z” and then type “bg” to background the process (if you would like). More advanced configuration and setup of Gunicorn can be found in step nine of this tutorial.


Now you’re ready for the final step!


# Step Seven: Configure Your App



The final step is to configure your app for production. All of the changes we need to make are in your “settings.py” file for your Django project.  Open this file with the following command:


```
sudo nano /opt/myproject/myproject/settings.py

```


The path to your settings file may differ depending on how your project is set up. Modify the path in the command above accordingly.


With your settings file open, change the DEBUG settings to False:


```
DEBUG = False

```


This will make it so that errors will show up to users as 404 or 500 error pages, rather than giving them a stack trace with debug information.


Now edit your database settings to look like the following, using your database name, user, and password instead of the ones shown below:


```
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.psycopg2', # Add 'postgresql_psycopg2', 'mysql', 'sqlite3' or 'oracle'.
        'NAME': 'mydb',                      # Or path to database file if using sqlite3.
        # The following settings are not used with sqlite3:
        'USER': 'myuser',
        'PASSWORD': 'password',
        'HOST': '',                      # Empty for localhost through domain sockets or '127.0.0.1' for localhost through TCP.
        'PORT': '',                      # Set to empty string for default.
    }
}

```


Now edit your static files settings:


```
STATIC_ROOT = '/opt/myproject/static/'

STATIC_URL = '/static/'

```


Save and exit the file. Now all we need to do is collect our static files. Change into the directory where your “manage.py” script is and run the following command:


```
python manage.py collectstatic

```


This command will collect all the static files into the directory we set in our settings.py file above.


And that’s it! You’ve now got your app deployed to production and ready to go.


