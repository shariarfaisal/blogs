# How To Set Up Nginx Server Blocks  Virtual Hosts  on Rocky Linux 9

```Nginx``` ```Rocky Linux``` ```Rocky Linux 9```

## Introduction


When using the Nginx web server, server blocks (similar to virtual hosts in Apache) can be used to encapsulate configuration details and host more than one domain on a single server.


In this guide, you’ll learn how to configure server blocks in Nginx on a Rocky Linux 9 server.


## Prerequisites


You should be using a non-root user with sudo privileges throughout this tutorial. If you do not have a user like this configured, you can create one by following our Rocky Linux 9 initial server setup guide.


You will also need to have Nginx installed on your server. You can install it following How To Install Nginx on Rocky Linux 9:


# Step 1 — Setting Up New Document Root Directories


For demonstration purposes, this tutorial will cover setting up two domains with an Nginx server. The domain names used in this guide are example.com and test.com. If you have your own domains already, you can use those instead.



Note: For more information on registering a new domain with DigitalOcean, please see our Domains and DNS product documentation.

If you do not have two domain names to configure, you can use placeholder names for now. You will still be able to test your configuration.


By default, Nginx on Rocky Linux 9 has one server block enabled. It is configured to serve documents out of a directory at /usr/share/nginx/html. While this works well for a single site, you need additional directories to serve multiple sites. You can consider the /usr/share/nginx/html directory the default directory that will be served if the client request doesn’t match any of your other sites.


You can create a directory structure within /usr/share/nginx for each of your sites. The actual web content will be placed in an html directory within these site-specific directories. This gives you some additional flexibility to create other directories associated with your sites as  necessary.


Create these directories for each of your sites. The -p flag tells mkdir to create any necessary parent directories along the way:


```
sudo mkdir -p /usr/share/nginx/example.com/html
sudo mkdir -p /usr/share/nginx/test.com/html


```


Now that you have your directories, you can reassign ownership of the web directories to your normal user account. This will let you write to them without sudo permissions.


You can use the $USER environment variable to assign ownership to the account that you are currently signed in as (make sure you’re not logged in as root). You should assign the group permission of these directories to nginx, an account that is automatically created for Nginx on Rocky Linux which will allow the web server itself to create new files if needed by your web applications. The chown command allows you to do both at once:


```
sudo chown -R $USER:nginx /usr/share/nginx/example.com/html
sudo chown -R $USER:nginx /usr/share/nginx/test.com/html


```


Finally, use the chmod command to ensure that both your user and the nginx group have full (7) permissions, whereas other users have read-only (5) permissions. To learn more about Linux permissions, refer to Linux Permissions Basics.


```
sudo chmod -R 775 /usr/share/nginx


```


Your directory structure is now configured and you can move on.


# Step 2 — Creating Sample Pages for Each Site


Now that you have your directory structure set up, create a default page for each of your sites so that you will have something to display.


Using nano or your favorite text editor, create an index.html file in your first domain:


```
nano /usr/share/nginx/example.com/html/index.html


```


Inside the file, create a barebones web landing page that indicates what site you are currently accessing. It will look like this:


/usr/share/nginx/example.com/html/index.html
```
<html>
    <head>
        <title>Welcome to Example.com!</title>
    </head>
    <body>
        <h1>Success! The example.com server block is working!</h1>
    </body>
</html>

```


Save and close the file when you are finished. If you are using nano, press Ctrl+X, then when prompted, Y and then Enter.


Since the file for your second site is going to be the same for demonstration purposes, you can copy it over to your second document root like this:


```
cp /usr/share/nginx/example.com/html/index.html /usr/share/nginx/test.com/html/


```


Now, you can open the new file in nano or your favorite text editor:


```
nano /usr/share/nginx/test.com/html/index.html


```


Modify it so that it refers to your second domain:


/usr/share/nginx/test.com/html/index.html
```
<html>
    <head>
        <title>Welcome to Test.com!</title>
    </head>
    <body>
        <h1>Success!  The test.com server block is working!</h1>
    </body>
</html>

```


Save and close this file when you are finished. You now have some pages to display to visitors of your two domains.


# Step 3 — Creating Server Block Files for Each Domain


Now that you have content to serve, you need to create the server blocks that will tell Nginx how to do this.


By default, Nginx on Rocky Linux contains one “default” server block inside of its main configuration file, nginx.conf. That block, inside of that file, looks like this:


/etc/nginx/nginx.conf
```
…
   server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
…

```


You can use this as a template for your own configurations, which you can create in separate files. Begin by designing your first domain’s server block, which you can then copy over for your second domain and make any necessary modifications.


## Creating the First Server Block File


Create your first server block config file inside of the /etc/nginx/conf.d directory. The main Nginx config file includes the line include /etc/nginx/conf.d/*.conf; by default, which means it will check files matching that pattern for additional server blocks.


Use your favorite text editor with sudo privileges to create a config file for your first domain:


```
sudo nano /etc/nginx/conf.d/example.com.conf


```


Start by pasting in this barebones server block:


/etc/nginx/conf.d/example.com.conf
```
server {
        listen 80;
        listen [::]:80;

        root /usr/share/nginx/html;
        index index.html index.htm index.nginx-debian.html;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Firstly after that, you should review the listen directives. Only one of your server blocks on the server can have the default_server option enabled. This specifies which block should serve a request if the server_name requested does not match any of the available server blocks. This shouldn’t happen very frequently in real world scenarios since visitors will be accessing your site through your domain name.


You can choose to designate one of your sites as the “default” by including the default_server option in the listen directive, or you can leave the default server block enabled in nginx.conf, which will serve the content of the /usr/share/nginx/html directory if the requested host cannot be found.


In this guide, you’ll leave the “default” server block in place to serve non-matching requests, so your new example.com configuration will not contain default_server:


/etc/nginx/conf.d/example.com.conf
```
server {
        listen 80;
        listen [::]:80;

        . . .
}

```


The next thing to adjust is the document root, specified by the root directive. Point it to the site’s document root that you created:


/etc/nginx/conf.d/example.com.conf
```
server {
        listen 80;
        listen [::]:80;

        root /usr/share/nginx/example.com/html;

}

```


Next, you need to modify the server_name to match requests for your first domain. You can additionally add any aliases that you need to match. In this tutorial, you will add an example.com and an www.example.com alias to demonstrate.


When you are finished, your file will look like this:


/etc/nginx/conf.d/example.com.conf
```
server {
        listen 80;
        listen [::]:80;

        root /usr/share/nginx/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


That is all you need for a configuration. Save and close the file to exit.


## Creating the Second Server Block File


Now that you have your initial server block configuration, you can use that as a basis for your second file. Copy it over to create a new file:


```
sudo cp /etc/nginx/conf.d/example.com.conf /etc/nginx/conf.d/test.com.conf


```


Open the new file with sudo privileges in your preferred editor:


```
sudo nano /etc/nginx/conf.d/test.com.conf


```


Again, make sure that you do not use the default_server option for the listen directive in this file if you’ve already used it elsewhere. Adjust the root directive to point to your second domain’s document root and adjust the server_name to match your second site’s domain name (make sure to include any aliases).


When you are finished, your file will likely look like this:


/etc/nginx/conf.d/test.com.conf
```
server {
        listen 80;
        listen [::]:80;

        root /usr/share/nginx/test.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name test.com www.test.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Save and close the file. In the next step, you’ll reload Nginx to reflect your changes.


# Step 4 — Enabling your Server Blocks and Restarting Nginx


You now have three server blocks enabled, which are configured to respond based on their listen directive and the server_name (you can read more about how Nginx processes these directives here):


- example.com: Will respond to requests for example.com and www.example.com
- test.com: Will respond to requests for test.com and www.test.com
- default: Will respond to any requests on port 80 that do not match the other two blocks.

Save and close the file when you are finished. Next, test to make sure that there are no syntax errors in any of your Nginx files:


```
sudo nginx -t


```


If no problems were found, restart Nginx to enable your changes:


```
sudo systemctl restart nginx


```


Nginx should now be serving both of your domain names.


# Step 5 — Modifying Your Local Hosts File for Testing (Optional)


If you have not been using domain names that you own and which actually point to this server’s IP address, and have instead been using placeholder values, you can modify your local computer’s hosts file to let you temporarily test your Nginx server block configuration.


This will not allow other visitors to view your site correctly, but it will give you the ability to reach each site independently and test your configuration. This works by intercepting requests that would usually go to DNS to resolve domain names. Instead, you can set your hosts file to automatically hardcode certain domains to certain remote addresses.



Note: Make sure you are operating on your local computer during these steps and not a remote server. You will need to have root access, be a member of the administrative group, or otherwise be able to edit system files to do this.

If you are working in a Mac or Linux environment, your hosts file is located at /etc/hosts:


```
sudo nano /etc/hosts


```


If you are on Windows, your hosts file is located at C:\Windows\System32\drivers\etc\hosts.


You need to know your server’s public IP address and the domains you want to route to the server. Assuming that your server’s public IP address is 203.0.113.5, the lines you add to your file would look like this:


/etc/hosts
```
127.0.0.1   localhost
. . .

203.0.113.5 example.com www.example.com
203.0.113.5 test.com www.test.com

```


This will intercept any requests for example.com and test.com and send them to your server, which is what you’ll need if you don’t actually own the domains that you are testing.


Save and close the file when you are finished.


# Step 6 — Testing Your Results


Now that you are all set up, you should test that your server blocks are functioning correctly. You can do that by visiting the domains in your web browser:


```
http://example.com

```


You should see a page that looks like this:





If you visit your second domain name, you should see a slightly different site:


```
http://test.com

```





If both of these sites work, you have successfully configured two independent server blocks with Nginx.


At this point, if you adjusted your hosts file on your local computer for testing, you’ll probably want to remove the lines you added.


If you need domain name access to your server for a public-facing site, you will probably want to purchase a domain name for each of your sites.


# Conclusion


You should now have the ability to create server blocks for each domain you wish to host from the same server. There aren’t any real limits on the number of server blocks you can create, so long as your hardware can handle the traffic.


Next, you may want to learn How to Set Up Password Authentication with Nginx.


