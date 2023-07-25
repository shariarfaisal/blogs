# How To Configure a Clustered Web Server with Varnish and Nginx on Ubuntu 13 10

```Ubuntu``` ```Nginx``` ```Scaling```

# Introduction



## About clustered web servers



A clustered web server is a technique used within web hosting to distribute the load across multiple machines or ‘nodes’. The aim of this technique is to remove single points of failure and increase website availability and uptime. It is typical that web clusters will utilize multiple backend and frontend nodes.


Clustering doesn’t have to be expensive and it’s extremely easy to get started with – this guide will demonstrate how to create a round robin two node clustered web server with Nginx and Varnish.


## About Varnish



Varnish is a HTTP accelerator; in other words a caching server. It allows us to speed up websites by directing HTTP requests static copy of the website maintained and produced by Varnish.


## About Nginx



Nginx is a lightweight, high performance HTTP server that will serve as the backend service to Varnish. It will not directly serve websites to visitors; however, it will respond to requests from Varnish whenever cache is required to be built.


# Setup



To perform the steps in this tutorial, you will need three droplets, all of which can be the minimal 512mb instance.


I recommend naming the hostnames of the instances as following:


- 
varnish

- 
nginx01

- 
nginx02


Of course you may add as many “nginx0x” as you wish, but for this tutorial I’ll be sticking with 2.


Upon initial SSH into the three newly created instances execute the following command:


```
sudo apt-get update

```


# Step One - Install Nginx



Nginx is the software that will be responsible for serving our website to Varnish.


skip this step for your varnish instance.You must install this on the nginx01 and nginx02 instances, that means repeat this process on each nginx0x server you wish to use.


It is recomended that Nginx is installed from source in order to ensure we get the most up to date version.


Nginx has two major dependencies: PCRE (Perl compatible regular expression library) and zlib (A compression library). At the time of writing this guide, the latest version are:


Nginx: 1.4.4


PCRE: 	8.34


zlib: 1.2.8


We must now download the source of the above ready to be extracted and built; enter each of the following commands seperatly:


```
wget http://nginx.org/download/nginx-1.4.4.tar.gz

wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.34.tar.gz

wget http://zlib.net/zlib-1.2.8.tar.gz

tar -zxvf nginx-1.4.4.tar.gz

tar -zxvf pcre-8.34.tar.gz

tar -zxvf zlib-1.2.8.tar.gz

```


Before we go on to build Nginx, we must first obtain a program called “Make” and a compiler for C++ source code ‘g++’, this will be responsible for carrying out all the commands required to build Nginx on our instances. You may obtain it through apt-get:


```
sudo apt-get install make g++

```


At this stage we can now go ahead and build Nginx/ First change directories into the extracted Nginx folder you just created:


```
cd nginx-1.4.4

```


Next we must configure the build options for our particular instance:


```
./configure --with-pcre=../pcre-8.34 --with-zlib=../zlib-1.2.8

```


Then we can go on to create the Nginx binaries:


```
make

```


Finally, we can install Nginx to our system:


```
sudo make install

```


# Step Two - Install Varnish



Varnish will be responsible for serving our website to a visitor.


You must only install this on the varnish instance.


First we need to obtain the GPG Key varnish provides for us to access their repository. We can download it by running the command:


```
wget http://repo.varnish-cache.org/debian/GPG-key.txt

```


Then install the key:


```
sudo apt-key add GPG-key.txt

```


We then need to add the Varnish repository list to our instances sources list:


```
echo "deb http://repo.varnish-cache.org/ubuntu/ precise varnish-3.0" | sudo tee -a /etc/apt/sources.list

```


Then ensure apt-get is aware of Varnish packages:


```
sudo apt-get update

```


Finally, install Varnish:


```
sudo apt-get install varnish

```


At this stage, we are ready to configure both Nginx and Varnish to serve a website to the outside world!


# Step Three - Configure Nginx



We don’t need to modify the confgurigation of Nginx too much, it’s defaults will be fine for this guide. However I recomend we modify the “Welcome to nginx” page we see to specify which VPS is serving the webpage to Varnish.


Navigate to the root html directory where the Nginx welcome page is located:


```
cd /usr/local/nginx/html/

```


Now edit index.html:


```
vim index.html

```


Modify the file to match the following:


nginx01:


```
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>I am nginx01</p>

```


nginx02:


```
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>I am nginx02</p>

```


Now we can start Nginx (Note: If this command produces no output, it has been executed successfully):


```
sudo /usr/local/nginx/sbin/nginx

```


# Step Four - Configure Varnish



First you must setup Varnish to run on port 80. To do that, you must modify the default Varnish configuration file. First change directories to where this file is located:


```
cd /etc/default

```


Then we must open the varnish file:


```
sudo vim varnish

```


Locate the following block within the file:


```
## Alternative 2, Configuration with VCL
#
# Listen on port 6081, administration on localhost:6082, and forward to
# one content server selected by the vcl file, based on the request.  Use a 1GB
# fixed-size cache file.
#
	DAEMON_OPTS="-a :6081 \
         -T localhost:6082 \
         -f /etc/varnish/default.vcl \
         -S /etc/varnish/secret \
         -s malloc,256m"

```


Modify it to match the following:


```
## Alternative 2, Configuration with VCL
#
# Listen on port 6081, administration on localhost:6082, and forward to
# one content server selected by the vcl file, based on the request.  Use a 1GB
# fixed-size cache file.
#
	DAEMON_OPTS="-a :80 \
         -T localhost:6082 \
         -f /etc/varnish/default.vcl \
         -S /etc/varnish/secret \
         -s malloc,256m"

```


Next we need to configure our load balancer. Change directories to where our Varnish configuration script is located:


```
cd /etc/varnish

```


Then open the default.vcl file:


```
sudo vim default.vcl

```


You must remove the backend default block within this file which looks like the following:


```
backend default {
	.host = "127.0.0.1";
	.port = "8080";
}

```


Replace it with the following. Ensure you change the .host respectivly for nginx01 and nginx02 to your public (or private if your instance has this feature) DigitalOcean IP:


```
# define our first nginx server
backend nginx01 {
	.host = "192.168.0.100";
	.port = "80";
}

# define our second nginx server
backend nginx02 {
	.host = "192.168.0.101";
	.port = "80";
}

# configure the load balancer
director nginx round-robin {
	{ .backend = nginx01; }
	{ .backend = nginx02; }
}

# When a request is made set the backend to the round-robin director named nginx
sub vcl_recv {
	set req.backend = nginx;
}

```


# Step Five - Test Availability



Let’s check to see if we can access our website through our Varnish server. Locate the public IP of the varnish instance you started and browser to it through a web browser. If you see the following text, everything is working!


```
Welcome to nginx!

If you see this page, the nginx web server is successfully installed and working. Further configuration is required.

I am nginx01

For online documentation and support please refer to nginx.org.
Commercial support is available at nginx.com.

Thank you for using nginx.

```


You can test to see if the site stays online by shutting down Nginx on the server that Nginx reports to be serving from. In my case this was nginx01, to shut down nginx you can execute the following:


```
/usr/local/nginx/sbin -s stop

```


Try your Varnish public IP again. You may still see the VPS you just switched off reported as the active server; this is because Varnish is holding the cache. Once this cache expires you will see nginx02 is serving the content.


To force Varnish to clear its cache, restart the service:


```
sudo service varnish restart

```


## Conclusion



At this stage, you now have a fully configured Varnish load balanced round robin cluster. I recomend you follow other tutorials on configuring your Nginx servers further: How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 12.04


<div class=“author”>Article Submitted By: <a href =“http://jacob.uk.com”>Jacob Clark</a></div>


