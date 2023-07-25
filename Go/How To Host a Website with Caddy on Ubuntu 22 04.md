# How To Host a Website with Caddy on Ubuntu 22 04

```Go``` ```Ubuntu``` ```Ubuntu 22.04``` ```Let's Encrypt```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Caddy is a web server designed around simplicity and security that comes with a number of features that are useful for hosting websites. For example, it can automatically obtain and manage TLS certificates from Let’s Encrypt to enable HTTPS, and includes support for HTTP/2. HTTPS is a system for securing traffic between your users and your server, and is quickly becoming a basic expectation of any website running in production — without it, Chrome and Firefox will warn that your website is “Not Secure” if users try to submit login information.


In this tutorial, you’ll build Caddy from source using xcaddy, a custom Caddy builder tool, and use it to host a website secured with HTTPS. This entails compiling it, configuring it using a Caddyfile and installing a plugin. In the end, your domain will serve static pages, while being secured with free TLS certificates from Let’s Encrypt.


# Prerequisites


- An Ubuntu 22.04 server with root privileges, with at least 2 GB RAM and a secondary, non-root account. You can set this up by following our Initial Server Setup Guide for Ubuntu 22.04. For this tutorial, the non-root user is sammy.
- The Go language toolchain installed on your server. Follow Steps 1 and 2 in How To Install Go on Ubuntu 20.04 to set up the latest Go version (Caddy needs Go version 17 or higher).
- A fully registered domain name. This tutorial will use your_domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.
- An A DNS record with your_domain pointing to your server’s public IP address and a CNAME DNS record with www.your_domain pointing to @. You can follow this introduction to DigitalOcean DNS for details on how to add them.
- A personal access token (API key) with read and write permissions for your DigitalOcean account. Visit How to Create a Personal Access Token to create one.

# Step 1 — Building Caddy


In this step, you’ll build Caddy from source with the ability to later add plugins, all without changing Caddy’s source code. You’ll use xcaddy to download and build Caddy and its plugins according to your needs.


Visit the xcaddy releases page and copy the link of the latest release for the linux_amd64 platform. Before downloading it, navigate to /tmp by running the following command:


```
cd /tmp


```


Then, download the latest release using wget:


```
wget https://github.com/caddyserver/xcaddy/releases/download/v0.3.1/xcaddy_0.3.1_linux_amd64.tar.gz


```


Once downloaded, extract only the binary:


```
tar xvf xcaddy_0.3.1_linux_amd64.tar.gz xcaddy


```


Finally, move the xcaddy executable to /usr/bin, making it accessible system wide:


```
sudo mv xcaddy /usr/bin


```


Now that xcaddy is installed, you’ll build Caddy. For that purpose, create a separate directory for storing it:


```
mkdir ~/caddy


```


Navigate to it by running the following command:


```
cd ~/caddy


```


To build the latest version of Caddy without any third party plugins, run the following command:


```
xcaddy build


```


This command will take some time to finish, but it will print an output similar to this:


```
Output2022/08/10 15:55:18 [INFO] Temporary folder: /tmp/buildenv_2022-08-10-1555.834895411
2022/08/10 15:55:18 [INFO] Writing main module: /tmp/buildenv_2022-08-10-1555.834895411/main.go
package main

import (
        caddycmd "github.com/caddyserver/caddy/v2/cmd"

        // plug in Caddy modules here
        _ "github.com/caddyserver/caddy/v2/modules/standard"
)

func main() {
        caddycmd.Main()
}
2022/08/10 15:55:18 [INFO] Initializing Go module
2022/08/10 15:55:18 [INFO] exec (timeout=10s): /usr/local/go/bin/go mod init caddy
go: creating new go.mod: module caddy
go: to add module requirements and sums:
        go mod tidy
2022/08/10 15:55:18 [INFO] Pinning versions
2022/08/10 15:55:18 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v github.com/caddyserver/caddy/v2
go: downloading github.com/caddyserver/caddy v1.0.5
...
2022/08/10 15:55:49 [INFO] Build environment ready
2022/08/10 15:55:49 [INFO] Building Caddy
2022/08/10 15:55:49 [INFO] exec (timeout=0s): /usr/local/go/bin/go mod tidy
...
2022/08/10 15:55:57 [INFO] exec (timeout=0s): /usr/local/go/bin/go build -o /home/sammy/caddy/caddy -ldflags -w -s -trimpath
2022/08/10 15:58:48 [INFO] Build complete: ./caddy
2022/08/10 15:58:48 [INFO] Cleaning up temporary folder: /tmp/buildenv_2022-08-10-1555.834895411

```


Once finished, you’ll have the caddy executable available in the current folder. Move it to /usr/bin to install it:


```
sudo mv caddy /usr/bin


```


You can try running caddy to check that it’s installed correctly:


```
caddy version


```


The output will contain the version of Caddy you’ve just compiled:


```
Outputv2.6.1 h1:EDqo59TyYWhXQnfde93Mmv4FJfYe00dO60zMiEt+pzo=

```


You have now built and executed Caddy. In the next step, you’ll install Caddy as a service so that it starts automatically at boot, and then you’ll adjust ownership and permissions settings to ensure the server’s security.


# Step 2 — Installing Caddy


Now that you’ve verified you’re able to build and run Caddy, you can configure a systemd service so that Caddy will be launched automatically on system startup. To understand more about systemd, visit our Systemd Essentials tutorial.


Caddy requires its own user and group in order to run as a systemd service. Create the group with the following command:


```
sudo groupadd --system caddy


```


Then, create a new user called caddy that belongs to the caddy group:


```
sudo useradd --system \
    --gid caddy \
    --create-home \
    --home-dir /var/lib/caddy \
    --shell /usr/sbin/nologin \
    --comment "Caddy web server" \
    caddy


```


The new caddy user will have its own home directory created. Because its shell is set to nologin, it won’t be possible to log in as caddy.


Change ownership of the Caddy binary to the root user:


```
sudo chown root:root /usr/bin/caddy


```


This change will prevent other accounts from modifying the executable. However, while the root user will own Caddy, you are advised to execute it using other, non-root accounts present on the system, as will be the case with the systemd service. Running commands from a non-root account ensures that the attacker won’t be able to modify the binary or execute commands as root in the event that Caddy (or another program) is compromised.


Next, set the binary file’s permissions to 755, which will give root full read/write/execute permissions for the file, while other users will only be able to read and execute it:


```
sudo chmod 755 /usr/bin/caddy


```


You have now finished setting up the Caddy binary and are ready to start Caddy configuration.


Create a directory where you’ll store Caddy’s configuration files:


```
sudo mkdir /etc/caddy


```


Then set the user and group permissions for it:


```
sudo chown -R root:caddy /etc/caddy


```


Setting the user as root and the group as caddy ensures that Caddy will have read and write access to the folder (via the caddy group) and that only the superuser account will have the same rights to read and modify.


In a later step, you’ll enable automatic TLS certificate provisioning from Let’s Encrypt. In preparation for that, make a directory to store any TLS certificates that Caddy will obtain and give it the same ownership rules as the /etc/caddy directory:


```
sudo mkdir /etc/ssl/caddy
sudo chown -R root:caddy /etc/ssl/caddy


```


Caddy must be able to write certificates to this directory and read from it in order to encrypt requests. For this reason, modify the permissions for the /etc/ssl/caddy directory so that it’s only accessible by root and caddy:


```
sudo chmod 0770 /etc/ssl/caddy


```


Next create a directory to store the files that Caddy will host:


```
sudo mkdir /var/www


```


Then set the directory’s owner and group to caddy:


```
sudo chown caddy:caddy /var/www


```


To install the Caddy service, download the systemd unit file from the Caddy GitHub repository to /etc/systemd/system by running:


```
sudo sh -c 'curl https://raw.githubusercontent.com/caddyserver/dist/master/init/caddy.service > /etc/systemd/system/caddy.service'


```


You’ll receive a response like this:


```
Output % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1030  100  1030    0     0   4698      0 --:--:-- --:--:-- --:--:--  4703
​```

Modify the service file's permissions so it can only be modified by its owner, `root`:

​```command
sudo chmod 644 /etc/systemd/system/caddy.service

```


Then, reload systemd to detect the Caddy service:


```
sudo systemctl daemon-reload


```


Check whether systemd has detected the Caddy service by running systemctl status:


```
sudo systemctl status caddy


```


You’ll receive an output similar to:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: https://caddyserver.com/docs/

```


If you have the same output, then the new service was detected by systemd.


As part of the initial server setup, you enabled ufw and allowed SSH connections. For Caddy to serve HTTP and HTTPS traffic from your server, you’ll need to allow them in ufw by running the following command:


```
sudo ufw allow proto tcp from any to any port 80,443


```


The output will be:


```
OutputRule added
Rule added (v6)

```


Use ufw status to verify your changes:


```
sudo ufw status


```


You’ll receive the following output:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80,443/tcp                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
80,443/tcp (v6)            ALLOW       Anywhere (v6)

```


Your installation of Caddy is now complete, though it isn’t configured to serve anything. In the next step, you’ll configure Caddy to serve files from the /var/www directory.


# Step 3 — Configuring Caddy


In this section, you’ll write basic Caddy configuration for serving static files from your server.


Create a basic HTML file called index.html in /var/www:


```
sudo nano /var/www/index.html


```


Add the following lines:


/var/www/index.html
```
<!DOCTYPE html>
<html>
<head>
<title>Hello from Caddy!</title>
</head>
<body>
<h1 style="font-family: sans-serif">This page is being served via Caddy</h1>
</body>
</html>

```


When shown in a web browser, this file will display a heading with the text This page is being served via Caddy. Save and close the file.


Caddy reads its configuration from a file called Caddyfile, stored under /etc/caddy. Create and open the file for editing:


```
sudo nano /etc/caddy/Caddyfile


```


Add the following lines:


/etc/caddy/Caddyfile
```
http:// {
    root * /var/www
    encode gzip
    file_server
}


```


This basic Caddy config declares that all HTTP traffic to your server should be served with files (file_server) from /var/www (which is marked as root) and compressed using gzip to reduce page loading times on the client side.


Caddy has different directives for many use cases. For example, the log directive could be useful for logging all HTTP requests that occur. You can review more options at the official documentation page for directives.


When you are done, save and close the file.


To test that everything is working correctly, start the Caddy service:


```
sudo systemctl start caddy


```


Next, run systemctl status to find information about the status of the Caddy service:


```
sudo systemctl status caddy


```


You’ll receive the following:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-08-10 15:02:41 UTC; 2s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 5443 (caddy)
      Tasks: 7 (limit: 1119)
     Memory: 7.5M
        CPU: 30ms
     CGroup: /system.slice/caddy.service
             └─5443 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

```


You can now visit your server’s IP in a web browser. Your sample web page will display:





You have now configured Caddy to serve static files from your server. In the next step, you’ll extend Caddy’s functionality through plugins.


# Step 4 — Enabling Automatic TLS with Let’s Encrypt


Plugins can change and extend Caddy’s behavior. Generally, they offer more config directives, according to your use case. In this step, you’ll enable automatic Let’s Encrypt certificate provisioning and renewal, using TXT DNS records for verification. To verify using TXT DNS records, you’ll install the official plugin to interface with the DigitalOcean DNS API.


To add a plugin, you need to recompile Caddy using xcaddy, specifying the repositories of plugins that should be available. Run the following command to compile Caddy with support for DigitalOcean DNS:


```
xcaddy build --with github.com/caddy-dns/digitalocean@master


```


The output will be similar to this:


```
Output...
2022/08/10 15:03:24 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v github.com/caddy-dns/digitalocean@master github.com/caddyserver/caddy/v2
go: downloading github.com/caddy-dns/digitalocean v0.0.0-20220527005842-9c71e343246b
go: downloading github.com/libdns/digitalocean v0.0.0-20220518195853-a541bc8aa80f
go: downloading github.com/digitalocean/godo v1.41.0
go: downloading github.com/google/go-querystring v1.0.0
go: downloading golang.org/x/oauth2 v0.0.0-20210514164344-f6687ab2804c
go: downloading google.golang.org/appengine v1.6.6
go: added github.com/caddy-dns/digitalocean v0.0.0-20220527005842-9c71e343246b
go: added github.com/digitalocean/godo v1.41.0
go: added github.com/libdns/digitalocean v0.0.0-20220518195853-a541bc8aa80f
2022/08/10 15:03:33 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v
go: downloading github.com/antlr/antlr4 v0.0.0-20200503195918-621b933c7a7f
go: downloading github.com/Masterminds/semver v1.4.2
go: downloading github.com/cenkalti/backoff v2.2.1+incompatible
go: downloading github.com/cpuguy83/go-md2man v1.0.10
go: downloading github.com/antlr/antlr4/runtime/Go/antlr v0.0.0-20220804214150-8b0cc382067f
go: downloading github.com/antlr/antlr4 v4.10.1+incompatible
go: upgraded github.com/antlr/antlr4/runtime/Go/antlr v0.0.0-20220418222510-f25a4f6275ed => v0.0.0-20220804214150-8b0cc382067f
go: upgraded golang.org/x/oauth2 v0.0.0-20210514164344-f6687ab2804c => v0.0.0-20211104180415-d3ed0bb246c8
go: upgraded google.golang.org/appengine v1.6.6 => v1.6.7
2022/08/10 15:03:39 [INFO] Build environment ready
2022/08/10 15:03:39 [INFO] Building Caddy
2022/08/10 15:03:39 [INFO] exec (timeout=0s): /usr/local/go/bin/go mod tidy
2022/08/10 15:03:40 [INFO] exec (timeout=0s): /usr/local/go/bin/go build -o /home/sammy/caddy/caddy -ldflags -w -s -trimpath
2022/08/10 15:03:56 [INFO] Build complete: ./caddy
2022/08/10 15:03:56 [INFO] Cleaning up temporary folder: /tmp/buildenv_2022-08-10-1503.1377463227

```


Once the compilation is finished, move the resulting binary to /usr/bin by running:


```
sudo mv caddy /usr/bin


```


Then, set appropriate permissions:


```
sudo chown root:root /usr/bin/caddy
sudo chmod 755 /usr/bin/caddy


```


Next, you’ll configure Caddy to work with DigitalOcean’s API to set DNS records. Caddy needs to read your API token as an environment variable to configure DigitalOcean’s DNS, so you’ll edit its systemd unit file. Open the file for editing:


```
sudo nano /etc/systemd/system/caddy.service


```


Add the highlighted line in the [Service] section, replacing your_token_here with your API token:


/etc/systemd/system/caddy.service
```
...
[Service]
Type=notify
User=caddy
Group=caddy
Environment=DO_AUTH_TOKEN=your_token_here
ExecStart=/usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
ExecReload=/usr/bin/caddy reload --config /etc/caddy/Caddyfile
TimeoutStopSec=5s
LimitNOFILE=1048576
LimitNPROC=512
PrivateTmp=true
ProtectSystem=full
AmbientCapabilities=CAP_NET_BIND_SERVICE
...

```


Save and close this file, then reload the systemd daemon to ensure the configuration is updated:


```
sudo systemctl daemon-reload


```


Run systemctl restart to check your configuration changes:


```
sudo systemctl restart caddy


```


Then run systemctl status to check if it ran correctly:


```
sudo systemctl status caddy


```


You will receive the following output:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: active (running) since Wed 2022-08-10 15:06:01 UTC; 2s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 5620 (caddy)
      Tasks: 7 (limit: 1119)
     Memory: 7.5M
        CPU: 37ms
     CGroup: /system.slice/caddy.service
             └─5620 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

```


Next, you’ll need to make slight changes to your Caddyfile, so open it for editing:


```
sudo nano /etc/caddy/Caddyfile


```


Add the highlighted lines to the Caddyfile, replacing your_domain with your domain (instead of http://) and adding the tls block with the DigitalOcean DNS specified:


/etc/caddy/Caddyfile
```
your_domain {
    root * /var/www
    encode gzip
    file_server
    
    tls {
        dns digitalocean {env.DO_AUTH_TOKEN}
    }
}

```


Using a domain rather than a protocol specifier for the hostname will cause Caddy to serve requests over HTTPS. The tls directive configures Caddy’s behavior when using TLS and you order it to use the digitalocean plugin for setting DNS records and requesting the certificates. The dns subdirective specifies that Caddy should use the DigitalOcean DNS system instead of HTTP.


Save and close the file.


With this, your website is ready to be deployed. Restart Caddy with systemctl and then enable it so it runs on boot:


```
sudo systemctl restart caddy
sudo systemctl enable caddy


```


When you visit your domain, you’ll be redirected to HTTPS automatically with the same This page is being served via Caddy message.


Your installation of Caddy is now complete and secured, and you can further customize according to your use case.


# Conclusion


You now have Caddy installed and configured on your server, serving static pages at your desired domain and secured with free Let’s Encrypt TLS certificates.


A good next step would be to set up notifications for when new versions of Caddy are released. For example, you could use the Atom feed for Caddy releases or a dedicated service such as dependencies.io. You can explore Caddy’s documentation for more information on configuring Caddy.


You can also learn more about using the Go language with our series on How To Code in Go.


