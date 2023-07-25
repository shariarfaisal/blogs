# How to Set Up Dante Proxy for Private Connections on Debian 11

```Debian``` ```Debian 11``` ```Security```

## Introduction


Proxy servers are a type of server application that functions as a gateway between an end user and an internet resource. Through a proxy server, an end user is able to control and monitor their web traffic for a wide variety of purposes, including privacy, security, and caching. For example, you can use a proxy server to make web requests from a different IP address than your own. You can also use a proxy server to research how the web is served differently from one jurisdiction to the next, or avoid some methods of surveillance or web traffic throttling.


Dante is a stable, popular, open-source SOCKS proxy. In this tutorial, you will be installing and configuring Dante to provide a SOCKS proxy on a Debian 11 server.


# Prerequisites


To complete this guide, you will need:


- A Debian 11 server and a non-root user with sudo privileges. You can learn more about how to set up a user with these privileges in our Initial Server Setup with Debian 11 guide.

You will use the domain name your_domain in this tutorial, but you should substitute this with your own domain name, or IP address.


# Step 1 — Installing Dante


Dante is an open-source SOCKS proxy server. SOCKS is a less widely used protocol, but it is more efficient for some peer-to-peer applications, and is preferred over HTTP for some kinds of traffic. Begin by running the following commands as a non-root user to update your package listings and install Dante:


```
sudo apt update
sudo apt install dante-server


```


Dante will also automatically set up a background service and start after being installed. However, it is designed to gracefully quit with an error message the first time it runs, because it ships with all of its features disabled. You can verify this by using the systemctl command:


```
systemctl status danted.service


```


```
Output● danted.service - SOCKS (v4 and v5) proxy daemon (danted)
     Loaded: loaded (/lib/systemd/system/danted.service; enabled; vendor preset: enabled)
     Active: failed (Result: exit-code) since Wed 2021-12-15 21:48:22 UTC; 1min 45s ago
       Docs: man:danted(8)
             man:danted.conf(5)
   Main PID: 14496 (code=exited, status=1/FAILURE)

Dec 15 21:48:21 proxies systemd[1]: Starting SOCKS (v4 and v5) proxy daemon (danted)...
Dec 15 21:48:22 proxies systemd[1]: Started SOCKS (v4 and v5) proxy daemon (danted).
Dec 15 21:48:22 proxies danted[14496]: Dec 15 21:48:22 (1639604902.102601) danted[14496]: warning: checkconfig(): no socks authentication methods enabled.  This means all socks requests will be blocked after negotiation. Perhaps this is not intended?

```


To successfully start Dante’s services, you’ll need to enable them in the config file.


Dante’s config file is provided, by default, in /etc/danted.conf. If you open this file using nano or your favorite text editor, you will see a long list of configuration options, all of them disabled. You could try to navigate through this file and enable some options line-by-line, but in practice it will be more efficient and more readable to delete this file and replace it from scratch. Don’t worry about doing this. You can always review Dante’s default configuration by navigating to its online manual, and you could even redownload the package manually from Ubuntu’s package listing to reobtain the stock configuration file if you ever wanted. In the meantime, go ahead and delete it:


```
sudo rm /etc/danted.conf


```


Now you can replace it with something more concise. Opening a file with a text editor will automatically create the file if it doesn’t exist, so by using nano or your favorite text editor, you should now get an empty configuration file:


```
sudo nano /etc/danted.conf


```


Add the following contents:


/etc/danted.conf
```
logoutput: syslog
user.privileged: root
user.unprivileged: nobody

# The listening network interface or address.
internal: 0.0.0.0 port=1080

# The proxying network interface or address.
external: eth0

# socks-rules determine what is proxied through the external interface.
socksmethod: username

# client-rules determine who can connect to the internal interface.
clientmethod: none

client pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

socks pass {
    from: 0.0.0.0/0 to: 0.0.0.0/0
}

```


You now have a usable SOCKS server configuration, running on port 1080, which is a common convention for SOCKS. You can also break down the rest of this configuration file line-by-line:


- logoutput refers to how Dante will log connections, in this case using regular system logging
- user.privileged allows dante to have root permissions for checking permissions
- user.unprivileged does not grant the server any permissions for running as an unprivileged user, as this is unnecessary when not granting more granular permissions
- internal connection details specify the port that the service is running on and which IP addresses can connect
- external connection details specify the network interface used for outbound connections, eth0 by default on most servers

The rest of the configuration details deal with authentication methods, which are discussed in the next section. Don’t forget to open port 1080 in your firewall if you’re using ufw:


```
sudo ufw allow 1080


```


At this point, you could restart Dante and connect to it, but you would have a SOCKS server that’s open to the entire world, which you probably don’t want, so you’ll learn how to secure it first.


# Step 2 — Securing Dante


If you followed this tutorial so far, Dante will be making use of regular Linux user accounts for authentication. This is helpful, but the password used for that connection will be sent over plain text, so it’s important to create a dedicated SOCKS user that won’t have any other login privileges. To do that, you’ll use useradd with flags that won’t assign a login shell to the user, then set a password:


```
sudo useradd -r -s /bin/false your_dante_user
sudo passwd your_dante_user


```


You’ll also want to avoid logging into this account over an unsecured wireless connection or sharing the server too widely. Otherwise, malicious actors can and will make repeated efforts to log in.


Dante supports other authentication methods, but many clients (i.e., applications) that will connect to SOCKS proxies only support basic username and password authentication, so you may want to leave that part as-is. What you can do as an alternative is to restrict access to only specific IP addresses. This isn’t the most sophisticated option, but given the combination of technologies in use here, it’s a sensible one. You may have already learned how to restrict access to specific IP addresses with ufw from our prerequisite tutorials , but you can also do it within Dante directly. Edit your /etc/danted.conf:


```
sudo nano /etc/danted.conf


```


/etc/danted.conf
```
…
client pass {
    from: your_ip_address/0 to: 0.0.0.0/0
}

```


In order to support multiple IP addresses, you can use CIDR notation, or just add another client pass {} configuration block:


/etc/danted.conf
```
client pass {
    from: your_ip_address/0 to: 0.0.0.0/0
}

client pass {
    from: another_ip_address/0 to: 0.0.0.0/0
}

```


After that, you can finally restart Dante with your configuration changes.


```
sudo systemctl restart danted.service


```


This time, when you check the service status, you should see it running without any errors:


```
systemctl status danted.service


```


```
Output● danted.service - SOCKS (v4 and v5) proxy daemon (danted)
     Loaded: loaded (/lib/systemd/system/danted.service; enabled; vendor preset: enable>
     Active: active (running) since Thu 2021-12-16 18:06:26 UTC; 24h ago

```


In the next step, you’ll connect to your proxy at last.


# Step 3 — Connecting through Dante


In order to demonstrate your Dante server, you’ll use a command line program called curl, which is popular for making different types of web requests. In general, if you want to verify whether a given connection should be working in a browser under ideal circumstances, you should always test first with curl. You’ll be using curl on your local machine in order to do this – it’s installed by default on all modern Windows, Mac, and Linux environments, so you can open any local shell to run this command:


```
curl -v -x socks5://your_dante_user:your_dante_password@your_server_ip:1080 http://www.google.com/


```


```
Output*   Trying 68.183.159.74:1080...
* SOCKS5 connect to IPv4 142.251.33.68:80 (locally resolved)
* SOCKS5 request granted.
* Connected to 68.183.159.74 (68.183.159.74) port 1080 (#0)
> GET / HTTP/1.1
> Host: www.google.com
…

```


The credentials that you used for curl should now work anywhere else you might want to use your new proxy server.


# Conclusion


In this tutorial, you learned to deploy a popular, open-source API endpoint for proxying traffic with little to no overhead. Many applications have built-in proxy support (often at the OS level) going back decades, making this proxy stack highly reusable.


Next, you may want to learn how to deploy Squid, an HTTP proxy which can run alongside Dante for proxying different types of web traffic.


Because one of the most common use cases for proxy servers is proxying traffic to and from different global regions, you may want to review how to use Ansible to automate server deployments next, in case you find yourself wanting to duplicate this configuration in other data centers.


