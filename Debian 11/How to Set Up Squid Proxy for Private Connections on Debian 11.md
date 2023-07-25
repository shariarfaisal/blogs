# How to Set Up Squid Proxy for Private Connections on Debian 11

```Debian``` ```Debian 11``` ```Security```

## Introduction


Proxy servers are a type of server application that functions as a gateway between an end user and an internet resource. Through a proxy server, an end user is able to control and monitor their web traffic for a wide variety of purposes, including privacy, security, and caching. For example, you can use a proxy server to make web requests from a different IP address than your own. You can also use a proxy server to research how the web is served differently from one jurisdiction to the next, or avoid some methods of surveillance or web traffic throttling.


Squid is a stable, popular, open-source HTTP proxy. In this tutorial, you will be installing and configuring Squid to provide an HTTP proxy on a Debian 11server.


# Prerequisites


To complete this guide, you will need:


- A Debian 11 server and a non-root user with sudo privileges. You can learn more about how to set up a user with these privileges in our Initial Server Setup with Debian 11 guide.

You will use the domain name your_domain in this tutorial, but you should substitute this with your own domain name, or IP address.


# Step 1 — Installing Squid Proxy


Squid has many use cases beyond routing an individual user’s outbound traffic. In the context of large-scale server deployments, it can be used as a distributed caching mechanism, a load balancer, or another component of a routing stack. However, some methods of horizontally scaling server traffic that would typically have involved a proxy server have been surpassed in popularity by containerization frameworks such as Kubernetes, which distribute more components of an application. At the same time, using proxy servers to redirect web requests as an individual user has become increasingly popular for protecting your privacy. This is helpful to keep in mind when working with open-source proxy servers which may appear to have many dozens of features in a lower-priority maintenance mode. The use cases for a proxy have changed over time, but the fundamental technology has not.


Begin by running the following commands as a non-root user to update your package listings and install Squid Proxy:


```
sudo apt update
sudo apt install squid


```


Squid will automatically set up a background service and start after being installed. You can check that the service is running properly:


```
systemctl status squid.service


```


```
Output● squid.service - Squid Web Proxy Server
     Loaded: loaded (/lib/systemd/system/squid.service; enabled; vendor preset: enabled)
     Active: active (running) since Wed 2021-12-15 21:45:15 UTC; 2min 11s ago

```


By default, Squid does not allow any clients to connect to it from outside of this server. In order to enable that, you’ll need to make some changes to its configuration file, which is stored in /etc/squid/squid.conf. Open it in nano or your favorite text editor:


```
sudo nano /etc/squid/squid.conf


```


Be advised that Squid’s default configuration file is very, very long, and contains a massive number of options that have been temporarily disabled by putting a # at the start of the line they’re on, also called being commented out. You will most likely want to search through the file to find the lines you want to edit. In nano, this is done by pressing Ctrl+W, entering your search term, pressing Enter, and then repeatedly pressing Alt+W to find the next instance of that term if needed.


Begin by navigating to the line containing the phrase http_access deny all. You should see a block of text explaining Squid’s default access rules:


/etc/squid/squid.conf
```
. . . 
#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#
include /etc/squid/conf.d/*
# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
#http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
http_access deny all
. . . 

```


From this, you can see the current behavior – localhost is allowed; other connections are not. Note that these rules are parsed sequentially, so it’s a good idea to keep the deny all rule at the bottom of this configuration block. You could change that rule to allow all, enabling anyone to connect to your proxy server, but you probably don’t want to do that. Instead, you can add a line above http_access allow localhost that includes your own IP address, like so:


/etc/squid/squid.conf
```
#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#
include /etc/squid/conf.d/*
# Example rule allowing access from your local networks.
acl localnet src your_ip_address
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
#http_access allow localnet
http_access allow localhost

```


- acl means an Access Control List, a common term for permissions policies
- localnet in this case is the name of your ACL.
- src is where the request would originate from under this ACL, i.e., your IP address.

If you don’t know your local IP address, it’s quickest to go to a site like What’s my IP which can tell you where you accessed it from. After making that change, save and close the file. If you are using nano, press Ctrl+X, and then when prompted, Y and then Enter.


At this point, you could restart Squid and connect to it, but there’s more you can do in order to secure it first.


# Step 2 — Securing Squid


Most proxies, and most client-side apps that connect to proxies (e.g., web browsers) support multiple methods of authentication. These can include shared keys, or separate authentication servers, but most commonly entail regular username-password pairs. Squid allows you to create username-password pairs using built-in Linux functionality, as an additional or an alternative step to restricting access to your proxy by IP address. To do that, you’ll create a file called /etc/squid/passwords and point Squid’s configuration to it.


First, you’ll need to install some utilities from the Apache project in order to have access to a password generator that Squid likes.


```
sudo apt install apache2-utils


```


This package provides the htpasswd command, which you can use in order to generate a password for a new Squid user. Squid’s usernames won’t overlap with system usernames in any way, so you can use the same name you’ve logged in with if you want. You’ll be prompted to add a password as well:


```
sudo htpasswd -c /etc/squid/passwords your_squid_username


```


This will store your username along with a hash of your new password in /etc/squid/passwords, which will be used as an authentication source by Squid. You can cat the file afterward to see what that looks like:


```
sudo cat /etc/squid/passwords


```


```
Outputsammy:$apr1$Dgl.Mtnd$vdqLYjBGdtoWA47w4q1Td.

```


After verifying that your username and password have been stored, you can update Squid’s configuration to use your new /etc/squid/passwords file. Using nano or your favorite text editor, reopen the Squid configuration file and add the following highlighted lines:


```
sudo nano /etc/squid/squid.conf


```


/etc/squid/squid.conf
```
…
#
# INSERT YOUR OWN RULE(S) HERE TO ALLOW ACCESS FROM YOUR CLIENTS
#
include /etc/squid/conf.d/*
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/passwords
auth_param basic realm proxy
acl authenticated proxy_auth REQUIRED
# Example rule allowing access from your local networks.
acl localnet src your_ip_address
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
#http_access allow localnet
http_access allow localhost
http_access allow authenticated
# And finally deny all other access to this proxy
http_access deny all
…

```


These additional directives tell Squid to check in your new passwords file for password hashes that can be parsed using the basic_ncsa_auth mechanism, and to require authentication for access to your proxy. You can review Squid’s documentation for more information on this or other authentication methods. After that, you can finally restart Squid with your configuration changes. This might take a moment to complete.


```
sudo systemctl restart squid.service


```


And don’t forget to open port 3128 in your firewall if you’re using ufw:


```
sudo ufw allow 3128


```


In the next step, you’ll connect to your proxy at last.


# Step 3 — Connecting through Squid


In order to demonstrate your Squid server, you’ll use a command line program called curl, which is popular for making different types of web requests. In general, if you want to verify whether a given connection should be working in a browser under ideal circumstances, you should always test first with curl. You’ll be using curl on your local machine in order to do this – it’s installed by default on all modern Windows, Mac, and Linux environments, so you can open any local shell to run this command:


```
curl -v -x http://your_squid_username:your_squid_password@your_server_ip:3128 http://www.google.com/


```


The -x argument passes a proxy server to curl, and in this case you’re using the http:// protocol, specifying your username and password to this server, and then connecting to a known-working website like google.com. If the command was successful, you should see the following output:


```
Output*   Trying 138.197.103.77...
* TCP_NODELAY set
* Connected to 138.197.103.77 (138.197.103.77) port 3128 (#0)
* Proxy auth using Basic with user 'sammy'
> GET http://www.google.com/ HTTP/1.1

```


It is also possible to access https:// websites with your Squid proxy without making any further configuration changes. These make use of a separate proxy directive called CONNECT in order to preserve SSL between the client and the server:


```
curl -v -x http://your_squid_username:your_squid_password@your_server_ip:3128 https://www.google.com/


```


```
Output*   Trying 138.197.103.77...
* TCP_NODELAY set
* Connected to 138.197.103.77 (138.197.103.77) port 3128 (#0)
* allocate connect buffer!
* Establish HTTP proxy tunnel to www.google.com:443
* Proxy auth using Basic with user 'sammy'
> CONNECT www.google.com:443 HTTP/1.1
> Host: www.google.com:443
> Proxy-Authorization: Basic c2FtbXk6c2FtbXk=
> User-Agent: curl/7.55.1
> Proxy-Connection: Keep-Alive
>
< HTTP/1.1 200 Connection established
<
* Proxy replied OK to CONNECT request
* CONNECT phase completed!

```


The credentials that you used for curl should now work anywhere else you might want to use your new proxy server.


# Conclusion


In this tutorial, you learned to deploy a popular, open-source API endpoint for proxying traffic with little to no overhead. Many applications have built-in proxy support (often at the OS level) going back decades, making this proxy stack highly reusable.


Next, you may want to learn how to deploy Dante, a SOCKS proxy which can run alongside Squid for proxying different types of web traffic.


Because one of the most common use cases for proxy servers is proxying traffic to and from different global regions, you may want to review how to use Ansible to automate server deployments next, in case you find yourself wanting to duplicate this configuration in other data centers.


