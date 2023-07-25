# How To Optimize Your Tomcat Installation on Ubuntu 14 04

```Security``` ```Ubuntu``` ```Server Optimization``` ```Java``` ```Nginx```

## Introduction


Tomcat is a popular implementation of the Java Servlet and JavaServer Pages technologies. It is released by the Apache Software Foundation under the popular Apache open source license. Its powerful features, favorable license, and great community makes it one of the best and most preferred Java servlets.


Tomcat almost always requires additional fine-tuning after its installation. Read this article to learn how to optimize your Tomcat installation so that it runs securely and efficiently.


This article continues the subject of running Tomcat on Ubuntu 14.04, and it is assumed that you have previously read How To Install Apache Tomcat 7 on Ubuntu 14.04 via Apt-Get.


# Prerequisites


This guide has been tested on Ubuntu 14.04. The described installation and configuration would be similar on other OS or OS versions, but the commands and location of configuration files may vary.


For this tutorial, you will need:


- Ubuntu 14.04 Droplet
- A non-root sudo user (see Initial Server Setup with Ubuntu 14.04)
- Tomcat installed and configured per the instructions in How To Install Apache Tomcat 7 on Ubuntu 14.04 via Apt-Get

All the commands in this tutorial should be run as a non-root user. If root access is required for the command, it will be preceded by sudo.


# Serving Requests on the Standard HTTP Port


As you have probably already noticed, Tomcat listens on TCP port 8080 by default. This default port comes mainly because of the fact that Tomcat runs under the unprivileged user tomcat7. In Linux, only privileged users like root are allowed to listen on ports below 1024 unless otherwise configured. Thus, you cannot simply change Tomcat’s listener port to 80 (HTTP).


So, the first task of optimizing your Tomcat installation is solving the above problem and making sure your Tomcat web applications are available on the standard HTTP port.


The simplest way (but not necessarily the best way) to resolve this is by creating a firewall (iptables) — forwarding from TCP port 80 to TCP port 8080. This can be done with the iptables command:


```
sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080


```


To make this iptables rule permanent check the article How To Set Up a Firewall Using IPTables on Ubuntu 14.04 in the part Saving your Iptables Configuration.


To remove this iptables rule, you can simply replace the -A flag for adding rules with the -D flag for removing rules in the above command like this:


```
sudo iptables -t nat -D PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080


```


Such a simple traffic forwarding is not optimal from a security or a performance point of view. Instead, a good practice is to add a web server, such as Nginx, in front of Tomcat. The reason for this is that Tomcat is just a Java servlet with basic functions of a web server while Nginx is a typical, powerful, fully functional web server. Here are some important benefits of using Nginx as a front end server:


- Nginx is more secure than Tomcat and could efficiently protect it from various attacks. In case of urgent security updates, it’s much easier, faster, and safer to update the frontend Nginx web server than to worry about downtime and compatibility issues associated with Tomcat upgrades.
- Nginx more efficiently serves HTTP and HTTPS traffic with better support for static content, caching, and SSL.
- Nginx is easily configured to listen on any port, including 80 and 443.

If are convinced of the above benefits, then first make sure that you have removed the previous iptables rule and then install Nginx with the command:


```
sudo apt-get install nginx


```


After that, edit Nginx’s default server block configuration (/etc/nginx/sites-enabled/default) with your favorite editor like this:


```
sudo nano /etc/nginx/sites-enabled/default


```


Look for the location / part, which specifies how all requests should be served and make sure it looks like this:


/etc/nginx/sites-enabled/default
```
location / {
	proxy_pass http://127.0.0.1:8080/;
}

```


The above proxy_pass directive means that all request should be forwarded to the local IP 127.0.0.1 on TCP port 8080 where Tomcat listens. Close the file, and restart Nginx with the command:


```
sudo service nginx restart


```


After that, try accessing Tomcat by connecting to your Droplet’s IP at the standard HTTP port in your browser. The URL should look like http://your_droplet's_ip. If everything works fine, Tomcat’s default page should be opened. If not, make sure that you have removed the iptables rule and that Tomcat has been properly installed as per the prerequisites of this article.


# Securing Tomcat


Securing Tomcat is probably the most important task which is often neglected. Luckily, in just a few steps you can have a fairly secure Tomcat setup. To follow this part of the article you should have Nginx installed and configured in front of Tomcat as previously described.


## Removing the Administrative Web Applications


The usual trade-off between functionality and security is valid for Tomcat too. To increase security, you can remove the default web manager and host-manager applications. This will be inconvenient because you will have to do all the administration, including web application deployments, from the command line.


Removing Tomcat’s web admin tools is good for security because you don’t have to worry that someone may misuse them. This good security practice is commonly applied for production sites.


The administrative web applications are contained in Ubuntu’s package tomcat7-admin. Thus, to remove them run the command:


```
sudo apt-get remove tomcat7-admin


```


## Restricting Access to the Administrative Web Applications


If you haven’t removed the administrative web applications, as recommended in the previous part, then we can at least restrict the access to them. Their URLs should be http://your_servlet_ip/manager/ and http://your_servlet_ip/host-manager/. If you see a 404 Not Found error at these URLs, then it means they have already been removed, and you don’t have to do anything. Still, you can read the following instructions to learn how to proceed with other sensitive resources you may wish to protect.


At this point, Nginx is accepting connections on port 80 so that you can access all web applications at http://your_servlet_ip from everywhere. Similarly, Tomcat listens on port 8080 globally, i.e. http://your_servlet_ip:8080, where you can find the same applications. To improve the security, we will restrict the resources available on port 80 via Nginx. We will also make Tomcat and its exposed port 8080 available only locally to the server and Nginx.


Open the default server block configuration file /etc/nginx/sites-enabled/default:


```
sudo nano /etc/nginx/sites-enabled/default


```


After the server_name directive but above the default root location (location /) add the following and replace your_local_ip with your local computer’s IP address:


/etc/nginx/sites-enabled/default
```
...
location /manager/ {
	allow your_local_ip;
	deny all;
	proxy_pass http://127.0.0.1:8080/manager/;
}
...

```


You should apply the same restriction for the host-manager application by adding another configuration block in which manager is replaced by host-manager like this (again, replace your_local_ip with your local IP address):


/etc/nginx/sites-enabled/default
```
...
location /host-manager/ {
	allow your_local_ip;
	deny all;
	proxy_pass http://127.0.0.1:8080/host-manager/;
}
...

```


Once you restart Nginx, access to the manager and host-manager web contexts will be limited only to your local IP address:


```
sudo service nginx restart


```


You can test it by opening in your browser http://your_servlet_ip/manager/ and http://your_servlet_ip/host-manager/. The applications should be available, but if you try to access the same URLs using a public proxy or a different computer, then you should see a 403 Forbidden error.


Furthermore, as an extra measure you can also remove Tomcat’s documentation and examples with the command:


```
sudo apt-get remove tomcat7-docs tomcat7-examples


```


Please note that Tomcat still listens for external connections on TCP port 8080. Thus, Nginx and its security measures can be easily bypassed. To resolve this problem configure Tomcat to listen on the local interface 127.0.0.1 only. For this purpose open the file /etc/tomcat7/server.xml with your favorite editor:


```
sudo nano /etc/tomcat7/server.xml


```


Add address="127.0.0.1" in the Connector configuration part like this:


/etc/tomcat7/server.xml
```
...
<Connector address="127.0.0.1" port="8080" protocol="HTTP/1.1"
	connectionTimeout="20000"
    URIEncoding="UTF-8"
    redirectPort="8443" />
...

```


After that restart Tomcat for the new setting to take effect:


```
sudo service tomcat7 restart


```


Following the above steps would ensure that you have a good, basic level of security for Tomcat.


# Fine Tuning the JVM settings


Naturally, the universal Java Virtual Machine (JVM) fine-tuning principles are applicable to Tomcat too. While the JVM tuning is a whole science of itself, there are some basic, good practices which anyone can easily apply:


- The maximum heap size,Xmx, is the maximum memory Tomcat can use. It should be set to a value which leaves enough free memory for the Droplet itself to run and any other services you may have on the Droplet. For example, if your Droplet has 2 GB of RAM, then it might be safe to allocate 1GB of RAM to xmx. However, please bear in mind that the actual memory Tomcat uses will be a little bit higher than the size of Xmx.
- The minimal heap size,Xms, is the amount of memory allocated at startup. It should be equal to the xmx value in most cases. Thus, you will avoid having the costly memory allocation process running because the size of the allocated memory will be constant all the time.
- The memory where classes are stored permanently, MaxPermSize, should allow Tomcat to load your applications’ classes and leave spare memory from the Xmx value for the instantiation of these classes. If you are not sure how much memory your applications’ classes require, then you could set the MaxPermSize to half the size of Xmx as a start — 512 MB in our example.

On Ubuntu 14.04 you can customize Tomcat’s JVM options by editing the file /etc/default/tomcat7. So, to apply the above tips please open this file with your favorite editor:


```
sudo nano /etc/default/tomcat7


```


If you have followed Tomcat’s installation instructions from the prerequisites you should find the following line:


/etc/default/tomcat7
```
...
JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom -Djava.awt.headless=true -Xmx512m -XX:MaxPermSize=256m -XX:+UseConcMarkSweepGC"
...

```


Provided your Droplet has 2 GB of RAM and you want to allocate around 1 GB to Tomcat, this line should be changed to:


/etc/default/tomcat7
```
...
JAVA_OPTS="-Djava.security.egd=file:/dev/./urandom -Djava.awt.headless=true -Xms1024m -Xmx1024m -XX:MaxPermSize=512m -XX:+UseConcMarkSweepGC"
...

```


For this setting to take effect, you have to restart Tomcat:


```
sudo service tomcat7 restart


```


The above JVM configuration is a good start, but you should monitor Tomcat’s log (/var/log/tomcat7/catalina.out) for problems, especially after restarting Tomcat or doing deployments. To monitor the log use the tail command like this:


```
sudo tail -f /var/log/tomcat7/catalina.out


```


If you are new to tail, you have to press the key combination Ctrl-C on your keyboard to stop tailing the log.


Search for errors like OutOfMemoryError. Such an error would indicate that you have to adapt the JVM settings and more specifically increase the Xmx size.


# Conclusion


That’s it! Now you have secured and optimized Tomcat in just a few easy-to-follow steps. These basic optimizations are recommended, not only for production, but even for test and development environments which are exposed to the Internet.


