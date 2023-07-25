# How To Scale Web Applications on Ubuntu 12 10

```Ubuntu``` ```Scaling``` ```Server Optimization```

##  Scaling a Web Application on Ubuntu


Scaling web applications is one of the most exciting things for a web administrator to have to do. Scaling is the process by which a system administrator utilizes multiple servers to serve a single web application.


Most scaling involves separating your web server and your database, and adding redundant systems to each aspect. 


This article will walk you through the steps to take an application from a single server to two, by adding a redundant web-server.


The two servers (which will be referred to as "Server A" and "Server B") will be your web servers and will use nginx for load balancing.


In all of the examples in this tutorial, the following server to IP map will apply:


Server A: 1.1.1.1


Server B: 2.2.2.2


Server A and B will be load balanced by using a program called nginx. Nginx can function by a webserver itself, but in our case we will 
only be using it for a load-balancer between two servers running apache.


# Step 1 - Configure Nginx on Server A


The following steps will result in Server A and Server B sharing the load from website traffic.


The first thing we are going to do is install nginx on Server A, to do our load balancing.


```
 
sudo apt-get install nginx php5-fpm

```


Once it is installed, we need to configure it a bit. We need to edit /etc/nginx/sites-enabled/default and tell nginx the IP addresses and port numbers where our website will actually be hosted.


Go ahead and open that file:


```
sudo nano /etc/nginx/sites-enabled/default
```


We can do that with an upstream block. An example upstream block is shown here and explained line by line below. 
In the following set of examples,


```
upstream nodes {
	ip_hash; 
        server 1.1.1.1:8080 max_fails=3 fail_timeout=30s;
	server 2.2.2.2:8080 max_fails=3 fail_timeout=30s; 
}

```


The first line defines an upstream block and names it "nodes" and the last line closes that block.


You can create as many upstream blocks as you like, but they must be uniquely named.


The two "server" lines are the important ones; they define the IP addresses and port numbers that our actual web servers are listening on.


Keep in mind that this IP address can be that of the same server that we are running nginx on.


Whether or not that is the case, it is recommended that you use a port other than 80.


A port other than the default HTTP port is recommended because you want to make it hard for end-users to accidentally stumble upon any of the individual servers used for web-server load-balancing.


A firewall may also be used as a preventative measure, as all web connections to any of the servers in your upstream originate from the IP address of the server running nginx. Steps to enhance security on your web servers will be explored later in this article.


The next thing we have to do is configure nginx so it will respond to and forward requests for a particular hostname. We can accomplish both of these with a virtualhost block that includes a proxy_pass line.


See below for an example and an explanation.


```
server {
        listen   1.1.1.1:80;

        root /path/to/document/root/;
        index index.html index.htm;

        server_name domain.tld www.domain.tld;

        location ~ \.php$ {
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }

        location / {
                proxy_pass http://nodes;
        }
}

```


There are a few key pieces to this configuration, including the "listen" line, the "server_name" line and the "location" block.


Make sure to edit the document root to point to your site


The first two are standard configuration elements, specifying the IP address and port that our web server is listening on respectively, but it is the last element, the "location" block, that allows us to load balance our servers.


Since Server A is going to serve as both the endpoint that users will connect to and as one of the load-balanced servers, we need to make a second virtualhost block, listening on a non-standard port for incoming connections.


```
server {
        listen   127.0.0.1:8080; 

         root /path/to/document/root/;
        index index.html index.htm index.php;

        server_name domain.tld www.domain.tld;

        location ~ \.php$ {
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }
}

```


Once you're done, reload nginx:


```
sudo service nginx reload
```


# Step 2 - Configure nginx on Server B


We need to set up a similar virtualhost block on Server B so it will also respond to requests for our domain. It will look very similar to the second server block we have on Server A.


```
server {
        listen   2.2.2.2:8080; 

        root /path/to/document/root/;
        index index.html index.htm index.php;

        server_name domain.tld www.domain.tld;

        location ~ \.php$ {
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_pass unix:/var/run/php5-fpm.sock;
                fastcgi_index index.php;
                include fastcgi_params;
        }
}

```


Reload nginx on the second server as well:


```
sudo service nginx reload
```


That is the only configuration that we need to do on this server!


One of the drawbacks when dealing with load-balanced web servers is the possibility of data being out of sync between the servers.


A solution to this problem might be employing a git repository to sync to each server, which will be the subject of a future tutorial.


You should now have a working load-balanced configuration. As always, any feedback in the comments is welcome!


By Jason Kurtz
