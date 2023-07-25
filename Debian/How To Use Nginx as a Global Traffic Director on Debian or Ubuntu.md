# How To Use Nginx as a Global Traffic Director on Debian or Ubuntu

```Ubuntu``` ```Nginx``` ```Debian```

## Introduction


As your customer base grows, so does the distance between your server(s) and your customers. We all know that if your server load increases – you scale. But what to do when distance is the problem?


The solution is simple: install server(s) in geographical locations closer to your customer base and direct them based on their location. But how do we do this easily while being cost effective?


In this guide, we’ll configure Nginx to detect and redirect customers to a subdomain that points to a more geographically appropriate server.


# Prerequisites


To complete this guide, you will need a user with sudo privileges.
You also need to know how to create servers in different regions.


## Assumptions


This article makes a few assumptions for ease of readability.


- Your domain is www.example.com
- Your primary server is in the US
- You want to install a GTD for Europe and Asia
- Your server IPs are as follows:
US: 1.1.1.1
EU: 1.1.1.2
AS: 1.1.1.3

# Subdomains and DNS configuration


Choosing subdomains is all up to you. For this tutorial, let’s use eu.example.com for Europe and as.example.com for Asia.


For each of those subdomains, add an A record in your DNS configuration with the IP of the server for that region:


- eu.example.com - 1.1.1.2
- as.example.com - 1.1.1.3

It should look something like this:





# Install Nginx and GeoIP


To have Nginx with a GeoIP module, you have two options: 1) use a precompiled package (only -full and -extra have GeoIP module) or 2) compile your nginx with the --with-http_geoip_module configuration parameter – In this case, you also need geoip-dev libraries.


Let’s use the already available repository packages.


```
sudo apt-get update
sudo apt-get install nginx-full geoip-database

```


Now both Nginx and GeoIP binaries are available. But there is one more thing: GeoIP’s city database includes region information. You need to download and install it manually.


```
wget -N http://geolite.maxmind.com/download/geoip/database/GeoLiteCity.dat.gz
gunzip GeoLiteCity.dat.gz
mv GeoLiteCity.dat /usr/share/GeoIP/

```


# Configure Nginx and your virtual host


Here we are going to tell Nginx where the GeoIP database files are. And in our virtual host, we are going to configure how Nginx will respond to requests based on their geoip information.


Open nginx.conf (default /etc/nginx/nginx.conf) with your preferred editor. Add the line geoip_city /usr/share/GeoIP/GeoLiteCity.dat;. Your nginx.conf will look like this:


```
http {
  geoip_city /usr/share/GeoIP/GeoLiteCity.dat;
  ...
}

```


Save it.


Now let’s edit your virtual host (default /etc/nginx/sites-available/default). Inside we need to create a map and add the subdomains to the server_name directive.


The map in Nginx allows us to set a variable $closest_server based on the value of $geoip_city_continent_code. You can read more about the Map module on Nginx’s documentation.


```
map $geoip_city_continent_code $closest_server {
  default www.example.com;
  EU      eu.example.com;
  AS      as.example.com;
}

```


Next we add the location-based subdomains to the $server_name directive:


```
server {
  server_name example.com
              www.example.com
              eu.example.com
              as.example.com;
  ...
}

```


The last part of the process is to make a condition in your virtual host that redirects visitors to the closest server. Add the following condition to your configuration:


```
server {
  ...

  if ($closest_server != $host) {
    rewrite ^ $scheme://$closest_server$request_uri break;
  }

  ...
}

```


After you’re done with all the changes your virtual host file would look like this:


```
map $geoip_city_continent_code $closest_server {
  default www.example.com;
  EU      eu.example.com;
  AS      as.example.com;
}

server {
  server_name example.com
              www.example.com
              eu.example.com
              as.example.com;

  if ($closest_server != $host) {
    rewrite ^ $scheme://$closest_server$request_uri break;
  }

  ...
}

```


Repeat this step for each server you want to configure. That way all of your servers will act as a traffic directors.


# Run a few tests


After completing all of these steps, the final one is to test what you have done. When working with Nginx, always test new configurations before applying.


Nginx provides an option to test its configuration files, without affecting currently running Nginx. You can do that by running one of these commands:


nginx -t, service nginx configtest or /etc/init.d/nginx configtest


If everything is good - Reload your Nginx configuration:


nginx -s reload, service nginx reload or /etc/init.d/nginx reload


To see your traffic director in action. Open a browser and visit www.example.com:


If you visit the site using a proxy in Europe, you should be redirected to eu.example.com:


If you visit the site using a proxy in Asia, you should be redirected to as.example.com:


And from now on your global visitors will be immediately redirected to a server closest to them, improving their experience on your website.


<div class=“author”>Submitted by: <a href=“https://github.com/SobanVuex”>Alex Soban</a></div>


