# How To Configure Nginx to Use Custom Error Pages on Ubuntu 22 04

```Nginx``` ```Ubuntu 22.04```

## Introduction


Nginx is a high performance web server capable of serving content with flexibility and power. When designing your web pages, it is often helpful to customize every piece of content that your users will see. This includes error pages for when they request content that is not available. In this guide, you’ll configure Nginx to use custom error pages on Ubuntu 22.04.


# Prerequisites


To get started on with this guide, you will need:


- A non-root user with sudo privileges. You can set up a user of this type by following along with the initial server setup guide for Ubuntu 22.04.
- Nginx installed on your system, following Steps 1 and 2 of this guide on how to install Nginx on Ubuntu 22.04.

When you have completed the above steps, continue with this guide.


# Creating Your Custom Error Pages


The custom error pages in this tutorial are for demonstration purposes, but the exact content of your custom error pages can be different to your liking.


Put your custom error pages in the /usr/share/nginx/html directory where Nginx sets its default document root. You’ll make a page for 404 errors called custom_404.html and one for general 500-level errors called custom_50x.html. You can use the following lines if you are just testing. Otherwise, place your own custom error content in these locations.


First, create the HTML file for your custom 404 page using nano or your preferred text editor:


```
sudo nano /usr/share/nginx/html/custom_404.html


```


Insert your custom error into your created file:


```
<h1 style='color:red'>Error 404: Not found :-(</h1>
<p>I have no idea where that file is, sorry. Are you sure you typed in the correct URL?</p>

```


Save and exit the file. If you’re using nano, hit CTRL+O to save then hit CTRL+X to exit.


Next, create the HTML file for your custom general 500-level error page:


```
sudo nano /usr/share/nginx/html/custom_50x.html


```


Insert your custom error into your created file:


```
<h1>Oops! Something went wrong...</h1>
<p>We seem to be having some technical difficulties. Hang tight.</p>

```


Save and exit the file once you’ve inserted your custom error content.


Now you have two custom error pages that you can serve when client requests result in different errors.


# Configuring Nginx to Use your Error Pages


Now, you need to tell Nginx that it should be utilizing these pages whenever the corresponding error conditions occur. Open the server block file in the /etc/nginx/sites-enabled directory that you wish to configure. The default server block file called default will be used here, but you should adjust your own server blocks if you’re using a non-default file:


```
sudo nano /etc/nginx/sites-enabled/default


```


Next you can point Nginx to your custom error pages.


## Directing 404 Errors to the Custom 404 Page


Use the error_page directive so that when a 404 error occurs (when a requested file is not found), the custom page you created is served. Create a location block for the file, where you are able to ensure that the root matches your file system location and that the file is only accessible through internal Nginx redirects (not requestable directly by clients):


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;


    . . .

    error_page 404 /custom_404.html;
    location = /custom_404.html {
        root /usr/share/nginx/html;
        internal;
    }
}

```


Usually, you don’t have to set the root in the new location block since it matches the root in the server block. However, you are being explicit here so that your error pages are served even if you move your regular web content and the associated document root to a different location.


## Directing 500 Level Errors to the Custom 50x Page


Next, add the directives to make sure that when Nginx encounters 500-level errors (server-related problems), it will serve the other custom page you made. This will follow the exact same formula you used in the last section. This time you’ll set multiple 500-level errors to all use the custom_50x.html page.


At the bottom, you’ll also add a dummy FastCGI pass so that you can test your 500-level error page. This is a dummy because it is expected to throw an error, because the backend does not actually exist. Requesting a page here will allow you to test that 500-level errors serve your custom page.


Edit your file as follows:


/etc/nginx/sites-enabled/default
```
server {
    listen 80 default_server;


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


Save and close the file when you are finished.


# Restarting Nginx and Testing your Pages


Test your configuration file’s syntax by typing:


```
sudo nginx -t


```


If any errors were reported, fix them before continuing. When no syntax errors are returned, restart Nginx by typing:


```
sudo systemctl restart nginx


```


Now, when you go to your server’s domain or IP address and request a non-existent file, you should see the 404 page you set up:


```
http://server_domain_or_IP/thiswillerror

```





When you go to the location you set up for the FastCGI pass, you will receive a 502 Bad Gateway error with your custom 500-level page:


```
http://server_domain_or_IP/testing

```





You can now go back and remove the fake FastCGI pass location from your Nginx config.


# Conclusion


You should now be serving custom error pages for your site. This is an easy way to personalize your users’ experience even when they are experiencing problems. One suggestion for these pages is to include links to locations where they can go to get help or more information. If you do this, make sure that the link destinations are accessible even when the associated errors are occurring.


