# How To Configure Nginx to Use Custom Error Pages on Ubuntu 14 04

```Ubuntu``` ```Nginx```

## Introduction


Nginx is a high performance web server capable of serving content with flexibility and power.  When designing your web pages, it is often helpful to customize every piece of content that your users will see.  This includes error pages for when they request content that is not available.  In this guide, we’ll demonstrate how to configure Nginx to use custom error pages on Ubuntu 14.04.


# Prerequisites


To get started on with this guide, you will need a non-root user with sudo privileges.  You can set up a user of this type by following along with our initial set up guide for Ubuntu 14.04.


You will also need to have Nginx installed on your system.  Learn how to set this up by following this guide.


When you have completed the above steps, continue with this guide.


# Creating Your Custom Error Pages


We will create a few custom error pages for demonstration purposes, but your custom pages will obviously be different.


We will put our custom error pages in the /usr/share/nginx/html directory where Ubuntu’s Nginx sets its default document root.  We’ll make a page for 404 errors called custom_404.html and one for general 500-level errors called custom_50x.html.  You can use the following lines if you are just testing.  Otherwise, put your own content in these locations:


```
echo "<h1 style='color:red'>Error 404: Not found :-(</h1>" | sudo tee /usr/share/nginx/html/custom_404.html
echo "<p>I have no idea where that file is, sorry.  Are you sure you typed in the correct URL?</p>" | sudo tee -a /usr/share/nginx/html/custom_404.html
echo "<h1>Oops! Something went wrong...</h1>" | sudo tee /usr/share/nginx/html/custom_50x.html
echo "<p>We seem to be having some technical difficulties. Hang tight.</p>" | sudo tee -a /usr/share/nginx/html/custom_50x.html


```


We now have two custom error pages that we can serve when client requests result in different errors.


# Configuring Nginx to Use your Error Pages


Now, we just need to tell Nginx that it should be utilizing these pages whenever the correct error conditions occur.  Open the server block file in the /etc/nginx/sites-enabled directory that you wish to configure.  We will use the default server block file called default, but you should adjust your own server blocks if you’re using a non-default file:


```
sudo nano /etc/nginx/sites-enabled/default


```


We can now point Nginx to our custom error pages.


## Direct 404 Errors to the Custom 404 Page


Use the error_page directive so that when a 404 error occurs (when a requested file is not found), the custom page you created is served.  We will create a location block for the file, where we are able to ensure that the root matches our file system location and that the file is only accessible through internal Nginx redirects (not requestable directly by clients):


/etc/nginx/sites-enabled/default
```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        . . .

        error_page 404 /custom_404.html;
        location = /custom_404.html {
                root /usr/share/nginx/html;
                internal;
        }
}

```


Usually, we would not have to set the root in the new location block since it matches the root in the server block.  However, we are being explicit here so that our error pages are served even if we move our regular web content and the associated document root to a different location.


## Direct 500 Level Errors to the Custom 50x Page


Next, we can add the directives to make sure that when Nginx encounters 500-level errors (server-related problems), it will serve the other custom page we made.  This will follow the exact same formula we used in the last section.  This time we set multiple 500-level errors to all use the custom_50x.html page:


/etc/nginx/sites-enabled/default
```
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        . . .

        error_page 404 /custom_404.html;
        location = /custom_404.html {
                root /usr/share/nginx/html;
                internal;
        }

        error_page 500 502 503 504 /custom_50x.html;
        location = /custom_50x.html {
                root /usr/share/nginx/html;
                internal;
        }

        location /testing {
                fastcgi_pass unix:/does/not/exist;
        }
}

```


At the bottom, we also added a dummy FastCGI pass so that we can test our 500-level error page.  This will not work correctly since the backend does not exist.  Requesting a page here will allow us to test that 500-level errors serve our custom page.


Save and close the file when you are finished.


# Restarting Nginx and Testing your Pages


Test your configuration file’s syntax by typing:


```
sudo nginx -t


```


If any errors were reported, fix them before continuing.  When no syntax errors are returned, restart Nginx by typing:


```
sudo service nginx restart


```


Now, when you go to your server’s domain or IP address and request a non-existent file, you should see the 404 page we set up:


```
http://server_domain_or_IP/thiswillerror

```





When you go to the location we set up for the FastCGI pass, we will receive a 502 Bad Gateway error with our custom 500-level page:


```
http://server_domain_or_IP/testing

```





You can now go back and remove the fake FastCGI pass location from your Nginx config.


# Conclusion


You should now be serving custom error pages for your site.  This is an easy way to personalize your users’ experience even when they are experiencing problems.  One suggestion for these pages is to include links to locations where they can go to get help or more information.  If you do this, make sure that the link destinations are accessible even when the associated errors are occurring.


