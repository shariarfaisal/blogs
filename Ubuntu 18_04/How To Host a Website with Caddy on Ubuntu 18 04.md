# How To Host a Website with Caddy on Ubuntu 18 04

```Ubuntu 18.04``` ```Go``` ```Let's Encrypt```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Caddy is a web server designed around simplicity and security that comes with a number of features that are useful for hosting websites. For example, it can automatically obtain and manage TLS certificates from Let’s Encrypt to enable HTTPS, and includes support for HTTP/2. HTTPS is a system for securing traffic between your users and your server, and is quickly becoming a basic expectation of any website running in production — without it, Chrome and Firefox will warn that your website is “Not Secure” if users try to submit login information.


In this tutorial, you’ll build Caddy from source using xcaddy, a custom Caddy builder tool, and use it to host a website secured with HTTPS. This entails compiling it, configuring it using a Caddyfile and installing plugin. In the end, your domain will serve static pages, while being secured with free TLS certificates from Let’s Encrypt.


# Prerequisites


- An Ubuntu 18.04 server with root privileges, and a secondary, non-root account. You can set this up by following our Initial Server Setup Guide for Ubuntu 18.04. For this tutorial the non-root user is sammy.
- A fully registered domain name. This tutorial will use your_domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice.
- An A DNS record with your_domain pointing to your server’s public IP address. You can follow this introduction to DigitalOcean DNS for details on how to add them.
- The Go language toolchain installed on your server, versions 1.14 and onwards. Follow our guide on How To Install Go and Set Up a Local Programming Environment on Ubuntu 18.04 to set up the latest Go. You don’t need to create any example projects.
- A personal access token (API key) with read and write permissions for your DigitalOcean account. Visit How to Create a Personal Access Token to create one.

# Step 1 — Building Caddy


In this step, you’ll build Caddy from source with the ability to later add plugins, all without changing Caddy’s source code. You’ll accomplish this using xcaddy, which will download and build Caddy and its plugins for you, according to your needs.


Head over to its releases page and copy the link of the latest release for the linux_amd64 platform. Before downloading it, navigate to /tmp by running the following command:


```
cd /tmp


```


Then, download the latest release using wget:


```
wget https://github.com/caddyserver/xcaddy/releases/download/v0.1.8/xcaddy_0.1.8_linux_amd64.tar.gz


```


Once its downloaded, extract only the binary:


```
tar xvf xcaddy_0.1.8_linux_amd64.tar.gz xcaddy


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


To build the latest version of Caddy, without any third party plugins, run the following command:


```
xcaddy build


```


This command will take some time to finish, and its output will look similar to this:


```
Output2021/02/23 21:12:07 [INFO] Temporary folder: /tmp/buildenv_2021-02-23-2112.542119152
2021/02/23 21:12:07 [INFO] Writing main module: /tmp/buildenv_2021-02-23-2112.542119152/main.go
2021/02/23 21:12:07 [INFO] Initializing Go module
2021/02/23 21:12:07 [INFO] exec (timeout=10s): /usr/local/go/bin/go mod init caddy
go: creating new go.mod: module caddy
go: to add module requirements and sums:
        go mod tidy
2021/02/23 21:12:07 [INFO] Pinning versions
2021/02/23 21:12:07 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v github.com/caddyserver/caddy/v2
...
2021/02/23 21:12:34 [INFO] Build environment ready
2021/02/23 21:12:34 [INFO] Building Caddy
2021/02/23 21:12:34 [INFO] exec (timeout=0s): /usr/local/go/bin/go mod tidy
...
2021/02/23 21:12:51 [INFO] exec (timeout=0s): /usr/local/go/bin/go build -o /home/sammy/caddy/caddy -ldflags -w -s -trimpath
2021/02/23 21:14:26 [INFO] Build complete: ./caddy
2021/02/23 21:14:26 [INFO] Cleaning up temporary folder: /tmp/buildenv_2021-02-23-2112.542119152

```


Once it finishes, you’ll have the caddy executable available in the current folder. Move it to /usr/bin to install it:


```
sudo mv caddy /usr/bin


```


You can try running caddy to check that it’s installed correctly:


```
caddy version


```


The output will contain the version of Caddy you’ve just compiled:


```
Outputv2.3.0 h1:fnrqJLa3G5vfxcxmOH/+kJOcunPLhSBnjgIvjXV/QTA=

```


You have now built and executed Caddy. In the next step, you’ll install Caddy as a service so that it starts automatically at boot, and then you’ll adjust its ownership and permissions settings to ensure the server’s security.


# Step 2 — Installing Caddy


Now that you’ve verified you’re able to build and run Caddy, it’s time to configure a systemd service so that Caddy can be launched automatically on system startup. To understand more about systemd, visit our Systemd Essentials tutorial.


Caddy requires its own user and group in order to run as a systemd service. First, create the group by running the following command:


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


The new caddy user will have its own home directory created. It won’t be possible to log in as caddy, as it’s shell is set to nologin.


Change ownership of the Caddy binary over to the root user:


```
sudo chown root:root /usr/bin/caddy

```


This will prevent other accounts from modifying the executable. However, even while the root user will own Caddy, it’s advised to execute it only using other, non-root accounts present on the system — as will be the case with the systemd service. This makes sure that in the event of Caddy (or another program) being compromised, the attacker won’t be able to modify the binary, or execute commands as root.


Next, set the binary file’s permissions to 755 — this gives root full read/write/execute permissions for the file, while other users will only be able to read and execute it:


```
sudo chmod 755 /usr/bin/caddy

```


You have now finished setting up the Caddy binary, and are ready to start writing Caddy configuration.


Create a directory where you’ll store Caddy’s configuration files by running the following command:


```
sudo mkdir /etc/caddy

```


Then, set the correct user and group permissions for it:


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


Next, create a directory to store the files that Caddy will host:


```
sudo mkdir /var/www

```


Then, set the directory’s owner and group to caddy:


```
sudo chown caddy:caddy /var/www


```


Caddy reads its configuration from a file called Caddyfile, stored under /etc/caddy. Create the file on disk by running:


```
sudo touch /etc/caddy/Caddyfile

```


To install the Caddy service, download the systemd unit file from the Caddy GitHub repository to /etc/systemd/system by running:


```
sudo sh -c 'curl https://raw.githubusercontent.com/caddyserver/dist/master/init/caddy.service > /etc/systemd/system/caddy.service'

```


Modify the service file’s permissions so it can only be modified by its owner, root:


```
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


You’ll see output similar to:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
       Docs: https://caddyserver.com/docs/

```


If you see this same output, then the new service was correctly detected by systemd.


As part of the initial server setup prerequisite, you enabled ufw, the uncomplicated firewall, and allowed SSH connections. For Caddy to be able to serve HTTP and HTTPS traffic from your server, you’ll need to allow them in ufw by running the following command:


```
sudo ufw allow proto tcp from any to any port 80,443


```


The output will be:


```
OutputRule added
Rule added (v6)

```


Use ufw status to check whether your changes worked:


```
sudo ufw status

```


You’ll see the following output:


```
OutputStatus: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
80,443/tcp                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
80,443/tcp (v6)            ALLOW       Anywhere (v6)

```


Your installation of Caddy is now complete, but it isn’t configured to serve anything. In the next step, you’ll configure Caddy to serve files from the /var/www directory.


# Step 3 — Configuring Caddy


In this section, you’ll write basic Caddy configuration for serving static files from your server.


Start by creating a basic HTML file in /var/www, called index.html:


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


Open the Caddyfile configuration file you created earlier for editing:


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


This is a basic Caddy config, and declares that all HTTP traffic to your server should be served with files (file_server) from /var/www (which is marked as root) and compressed using gzip to reduce page loading times on the client side.


When you are done, save and close the file.


Caddy has a huge number of different directives for many use cases. For example, the php_fastcgi directive could be useful for enabling PHP.


To test that everything is working correctly, start the Caddy service:


```
sudo systemctl start caddy

```


Next, run systemctl status to find information about the status of the Caddy service:


```
sudo systemctl status caddy

```


You’ll see the following:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-12-08 11:19:47 UTC; 34s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 4631 (caddy)
      Tasks: 6 (limit: 1137)
     Memory: 10.4M
     CGroup: /system.slice/caddy.service
             └─4631 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile

Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: USER=caddy
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: INVOCATION_ID=45713fb36abe48ecaf4aa72a12542658
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: JOURNAL_STREAM=9:33053
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.425965,"msg":"using provided configuration","config_file":"/etc/caddy/Caddyfile","config_adapter":""}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.4281814,"logger":"admin","msg":"admin endpoint started","address":"tcp/localhost:2019","enforce_origin":false,"origins":["127.0.0.1:2019","localhost:2019","[::1]:2019"]}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.4285417,"logger":"http","msg":"server is listening only on the HTTP port, so no automatic HTTPS will be applied to this server","server_name":"srv0","http_port":80}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.430661,"msg":"autosaved config","file":"/var/lib/caddy/.config/caddy/autosave.json"}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.430849,"msg":"serving initial configuration"}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.4311824,"logger":"tls.cache.maintenance","msg":"started background certificate maintenance","cache":"0xc00021f0a0"}
Dec 08 11:19:47 ubuntu-s-1vcpu-1gb-fra1-01 caddy[4631]: {"level":"info","ts":1607426387.4313655,"logger":"tls","msg":"cleaned up storage units"}

```


You can now browse to your server’s IP in a web browser. Your sample web page will display:





You have now configured Caddy to serve static files from your server. In the next step, you’ll extend Caddy’s functionality through the use of plugins.


# Step 4 — Enabling Automatic TLS with Let’s Encrypt


Plugins offer a way of changing and extending Caddy’s behavior. Generally, they offer more config directives for you to use, according to your use case. In this section, you’ll enable automatic Let’s Encrypt certificate provisioning and renewal, using TXT DNS records for verification. To verify using TXT DNS records, you’ll install the official plugin for interfacing with the DigitalOcean DNS API.



Note: You may notice that the official plugin we’re using in this step is marked as deprecated. That’s because it’s in the process of being replaced with a set of newer, modular libraries based on libdns.
However, the digitalocean library is currently in a developmental state with some open issues. Until the issues are resolved, we’ll use the earlier plugin, which works correctly despite being labeled obsolete.

To add a plugin, you’ll need to recompile Caddy using xcaddy, but specifying the repositories of plugins that should be available. Run the following command to compile Caddy with support for DigitalOcean DNS:


```
xcaddy build --with github.com/caddy-dns/lego-deprecated


```


The output will be similar to this:


```
Output2021/02/23 21:18:46 [INFO] Temporary folder: /tmp/buildenv_2021-02-23-2118.769615504
2021/02/23 21:18:46 [INFO] Writing main module: /tmp/buildenv_2021-02-23-2118.769615504/main.go
2021/02/23 21:18:46 [INFO] Initializing Go module
2021/02/23 21:18:46 [INFO] exec (timeout=10s): /usr/local/go/bin/go mod init caddy
go: creating new go.mod: module caddy
go: to add module requirements and sums:
        go mod tidy
2021/02/23 21:18:46 [INFO] Pinning versions
2021/02/23 21:18:46 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v github.com/caddyserver/caddy/v2
go get: added github.com/caddyserver/caddy/v2 v2.3.0
2021/02/23 21:18:49 [INFO] exec (timeout=0s): /usr/local/go/bin/go get -d -v github.com/caddy-dns/lego-deprecated
...
2021/02/23 21:19:17 [INFO] Build environment ready
2021/02/23 21:19:17 [INFO] Building Caddy
2021/02/23 21:19:17 [INFO] exec (timeout=0s): /usr/local/go/bin/go mod tidy
...
2021/02/23 21:19:20 [INFO] exec (timeout=0s): /usr/local/go/bin/go build -o /home/sammy/caddy/caddy -ldflags -w -s -trimpath
2021/02/23 21:20:09 [INFO] Build complete: ./caddy
2021/02/23 21:20:09 [INFO] Cleaning up temporary folder: /tmp/buildenv_2021-02-23-2118.769615504

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


Next, you’ll configure Caddy to work with DigitalOcean’s API to set DNS records. Caddy needs to read your API token as an environment variable to configure DigitalOcean’s DNS, so you’ll edit its systemd unit file:


```
sudo nano /etc/systemd/system/caddy.service

```


Add the highlighted line in the [Service] section, replacing your_token_here with your API token:


/etc/systemd/system/caddy.service
```
...
[Service]
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


Save and close this file, then reload the systemd daemon as you did earlier to ensure the configuration is updated:


```
sudo systemctl daemon-reload

```


Run systemctl restart to check that your configuration changes were OK:


```
sudo systemctl restart caddy

```


Then, run systemctl status to see if it ran correctly:


```
sudo systemctl status caddy


```


The output will look like:


```
Output● caddy.service - Caddy
     Loaded: loaded (/etc/systemd/system/caddy.service; disabled; vendor preset: enabled)
     Active: active (running) since Tue 2020-12-08 13:39:17 UTC; 1s ago
       Docs: https://caddyserver.com/docs/
   Main PID: 5440 (caddy)
      Tasks: 5 (limit: 1137)
     Memory: 9.8M
     CGroup: /system.slice/caddy.service
             └─5440 /usr/bin/caddy run --environ --config /etc/caddy/Caddyfile
...

```


You’ll need to make a couple of slight changes to your Caddyfile, so open it up for editing:


```
sudo nano /etc/caddy/Caddyfile

```


Add the highlighted lines to the Caddyfile, making sure to replace your_domain with your domain (instead of just http://) and adding the tls block, specifying that DigitalOcean DNS should be used:


/etc/caddy/Caddyfile
```
your_domain {
    root * /var/www
    encode gzip
    file_server
    
    tls {
        dns lego_deprecated digitalocean
    }
}

```


Using a domain rather than just a protocol specifier for the hostname will cause Caddy to serve requests over HTTPS. The tls directive configures Caddy’s behavior when using TLS, and the dns subdirective specifies that Caddy should use the DigitalOcean DNS system, instead of HTTP.


With this, your website is ready to be deployed. Restart Caddy with systemctl and then enable it, so it runs on boot:


```
sudo systemctl restart caddy
sudo systemctl enable caddy

```


If you browse to your domain, you’ll automatically be redirected to HTTPS, with the same message shown.


Your installation of Caddy is now complete and secured, and you can further customize according to your use case.


# Conclusion


You now have Caddy installed and configured on your server, serving static pages at your desired domain, secured with free Let’s Encrypt TLS certificates.


A good next step would be to find a way of being notified when new versions of Caddy are released. For example, you could use the Atom feed for Caddy releases, or a dedicated service such as dependencies.io.


You can explore Caddy’s documentation for more information on configuring Caddy.


