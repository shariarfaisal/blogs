# Installing and Configuring Graphite and Statsd on an Ubuntu 12 04 VPS

```Apache``` ```Ubuntu``` ```Monitoring``` ```Node.js```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## Introduction


Graphite and statsd can be very valuable tools used to visualize data.  Statsd collects data about running processes and graphite is a graphing library that can be used to create graphs.


We will be setting these tools up on an Ubuntu 12.04 VPS instance.


# Installing Graphite


Graphite has many dependencies that we will need to fulfill.  We also want to install the web server packages that will allow us to access the web interface.


First, update and upgrade your packages with apt-get:


```
sudo apt-get update
sudo apt-get upgrade
```


Install the needed packages:


```
sudo apt-get install --assume-yes apache2 apache2-mpm-worker apache2-utils apache2.2-bin apache2.2-common libapr1 libaprutil1 libaprutil1-dbd-sqlite3 build-essential python3.2 python-dev libpython3.2 python3-minimal libapache2-mod-wsgi libaprutil1-ldap memcached python-cairo-dev python-django python-ldap python-memcache python-pysqlite2 sqlite3 erlang-os-mon erlang-snmp rabbitmq-server bzr expect libapache2-mod-python python-setuptools
```


We will use python-setuptool's "easy_install" utility to install a few more important python components:


```
sudo easy_install django-tagging zope.interface twisted txamqp
```


Next, we will acquire the Graphite components from the project's website.


Carbon is the data aggregator, Graphite web is the web component, and whisper is the database library:


```
cd ~
wget https://launchpad.net/graphite/0.9/0.9.10/+download/graphite-web-0.9.10.tar.gz
wget https://launchpad.net/graphite/0.9/0.9.10/+download/carbon-0.9.10.tar.gz
wget https://launchpad.net/graphite/0.9/0.9.10/+download/whisper-0.9.10.tar.gz
```


Use tar to extract the archives:


```
find *.tar.gz -exec tar -zxvf '{}' \;
```


Move into the whisper directory and install it with provided setup file:


```
cd whisper*
sudo python setup.py install
```


Now, move to the carbon directory and do the same:


```
cd ../carbon*
sudo python setup.py install
```


Finally, move to the Graphite directory and check dependencies.  You should receive a message indicating that your dependencies are all correctly installed.:


```
cd ../graphite*
sudo python check-dependencies.py
```


```
All necessary dependencies are met.
All optional dependencies are met.
```


You are now safe to install Graphite:


```
sudo python setup.py install
```


# Configuring Graphite


Next, we will configure the software that we have just installed.


We will move to the Graphite configuration directory copy or create some files that we will use for our applications:


```
cd /opt/graphite/conf
sudo cp carbon.conf.example carbon.conf
```


```
Create a file to handle our polling settings:

```


```
sudo nano storage-schemas.conf
```


Copy the following data into the file:


```
[stats]
priority = 110
pattern = .*
retentions = 10:2160,60:10080,600:262974
```


Save and close the file.


Next, we will configure the Graphite database. Go to the Graphite webapp directory and run the database script:


```
cd /opt/graphite/webapp/graphite/
sudo python manage.py syncdb
```


You will be prompted to create a superuser account.  Type "yes" to continue and then follow the prompts.


Copy the local settings example file to the production version while you are in this directory:


```
sudo cp local_settings.py.example local_settings.py
```


## Configuring Apache


Next, we will configure Apache to serve our web content.


Begin by copying some more example configuration files:


```
sudo cp ~/graphite*/examples/example-graphite-vhost.conf /etc/apache2/sites-available/default
sudo cp /opt/graphite/conf/graphite.wsgi.example /opt/graphite/conf/graphite.wsgi
```


Give the Apache web user ownership of Graphite's storage directory so that it can write data properly:


```
sudo chown -R www-data:www-data /opt/graphite/storage
```


Create a directory for our WSGI data:


```
sudo mkdir -p /etc/httpd/wsgi
```


We need to edit the Apache configuration file we copied earlier:


```
sudo nano /etc/apache2/sites-available/default
```


Change the WSGISocketPrefix to reflect the directory we just created.  Also, change the ServerName property to reflect your domain name or IP address:


```
...
...
WSGISocketPrefix /etc/httpd/wsgi

<VirtualHost *:80>
	ServerName Your.Domain.Name.Here
	...
	...
```


Save and close the file.


Restart Apache to implement our changes:


```
sudo service apache2 restart
```


# Installing and Configuring Statsd


To use statsd, we need to install node.js.  To do this, we will install "python-software-properties", which contains a utility to add PPAs:


```
sudo apt-get install python-software-properties
```


We can now add the appropriate PPA:


```
sudo apt-add-repository ppa:chris-lea/node.js
```


Press "enter" to accept the repository change.  Now update the apt package list again and install node.js:


```
sudo apt-get update
sudo apt-get install nodejs
```


We will now install statsd to feed Graphite data from our VPS.


First, install git so that we can download the project files:


```
sudo apt-get install git
```


Now clone the git repository into our optional software directory:


```
cd /opt
sudo git clone git://github.com/etsy/statsd.git
```


Create the configuration file for statsd within the new statsd directory:


```
sudo nano /opt/statsd/localConfig.js
```


Copy and paste the following information into the file:


```
{
	graphitePort: 2003,
	graphiteHost: "127.0.0.1",
	port: 8125
}
```


# Starting the Services


Start the carbon data aggregator so that it will begin sending data:


```
sudo /opt/graphite/bin/carbon-cache.py start
```


Next, move to the statsd directory and run the files by using the node.js command:


```
cd /opt/statsd
node ./stats.js ./localConfig.js
```


```
19 Jul 21:15:34 - reading config file: ./localConfig.js
19 Jul 21:15:34 - server is up
```


You should receive a message informing you that the statsd server is up and running.


# Viewing the Results


To see the graph data, open a web browser and navigate to your domain or IP address:


```
Your.Domain.Name.Here
```


You will see a blank box in the middle.


![](https://assets.digitalocean.com/tutorial_images/DtyajrM.png?1)
You will also see a navigation menu on the left.  This is what we will address first


![](https://assets.digitalocean.com/tutorial_images/ZpOD4wo.png?1)
Click on the "Graphite" box and you should have a number of different options.  If you click through the file hierarchy, you will see many sets of data.


![](https://assets.digitalocean.com/tutorial_images/NIVYDrI.png?1)
Choose one and click it.  It will appear that no data has been graphed.  We will need to change the scale to see any information.


![](https://assets.digitalocean.com/tutorial_images/CWXSv7E.png?1)
On the Graphite Composer, choose the second icon from the left to adjust the date range.  Choose a range that only captures a few hours on either side of the current time.


![](https://assets.digitalocean.com/tutorial_images/dntgBGC.png?1)
Next Click on "Graph Options", mouse over "Line Mode" and select "Connected Line" in order to connect the data points.


![](https://assets.digitalocean.com/tutorial_images/l1R2cBf.png?1)
![](https://assets.digitalocean.com/tutorial_images/4aA8H87.png?1)
You might also want to select "Auto-Refresh" to have the graph update automatically as it receives data.


Be aware that every new data set you select adds that data to the graph.  It does not switch to that data set.  Click on "Graph Data" to remove data sets as necessary.  You can also click on the data category on the left again to de-select it.


By Justin Ellingwood
