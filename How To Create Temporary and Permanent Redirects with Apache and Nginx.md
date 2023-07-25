# How To Create Temporary and Permanent Redirects with Apache and Nginx

```Apache``` ```Nginx```

## Introduction


HTTP redirection, or URL redirection, is a technique of pointing one domain or address to another. There are many uses for redirection, and a few different kinds of redirection to consider. Redirects are used whenever a site needs people requesting one address to be directed to another address.


As you create content and administrate servers, you will often find the need to redirect traffic from one place to another. This guide will discuss the different use cases for these techniques, and how to accomplish them in Apache and Nginx, the two most common web servers.


# Prerequisites


- 
An Ubuntu 20.04 server set up by following the Ubuntu 20.04 initial server setup guide, including a sudo non-root user and a firewall.

- 
Apache or Nginx installed on your server, which you can do by following How To Install Apache on Ubuntu 20.04 or How To Install Nginx on Ubuntu 20.04. Both Apache and Nginx can be installed at the same time, and many stacks make use of both servers at once, but by default, they will conflict on the default HTTP/HTTPS ports 80 and 443, so if you are following this tutorial on a stock server configuration, you should only install one at a time.


# Step 1 – Reviewing Redirect Methods


There are many use cases for redirects. If you have established a web presence and would like to change your domain, it is best not to just abandon your old domain. Bookmarks to your site and links to your site located on other pages throughout the internet will break if your content disappears without any instructions to the browser about how to find its new location. Changing domains without redirecting will cause your site to lose traffic from previous visitors and lose all of the credibility you have worked to establish.


Often, it is helpful to register multiple variations of a name in order to benefit from users typing in addresses similar to your main domain. For example, if you have a domain called myspiders.com, you might also buy the domain names for myspiders.net and myspiders.org and redirect them both to your myspiders.com site. This allows you to catch users who might be trying to get to your site with the wrong address. It can also help prevent another site from using a similar domain and profiting off of your web presence.


Sometimes, it is necessary to change the names of pages that have already been published and received traffic on your site. Normally, this would lead to a 404 Not Found error or possibly another error depending on your security settings. These can be avoided by leading your visitors to another page that contains the correct content they were trying to access. There are a few different kinds of URL redirects, each of which mean something different to the client browser. The two most common types are 302 temporary redirects, and 301 permanent redirects.


## Temporary Redirects


Temporary redirects are useful if your web content for a certain URL temporarily needs to be served out of a different location. For example, if you are performing site maintenance, you may need to use a temporary redirect of all of the pages for your domain to an explanation page to inform your visitors that you will be back shortly.


Temporary redirects inform the browser that the content is temporarily located at a different location, but that they should continue to attempt to access the original URL.


## Permanent Redirects


Permanent redirects are useful when your content has been moved to a new location forever.


This is useful for when you need to change domains or when the URL needs to change for other reasons and the old location will no longer be used. This redirect informs the browser that it should no longer request the old URL and should update its information to point to the new URL.


## Forcing SSL


A common use for redirects is directing all site traffic to use SSL instead of standard HTTP.


Using redirects, it is possible to make all requests for http://www.mysite.com be redirected to https://www.mysite.com. Using LetsEncrypt to provide HTTPS will automatically generate a redirect configuration like this.


In the next part of this tutorial, you will learn some example configurations for redirects using the Apache web server. If you are using Nginx instead and you have no plans to use Apache, you can skip to Step 3.


# Step 2 – How to Redirect in Apache


Apache can redirect using a few different tools. Straightforward redirects can be implemented with tools from the mod_alias module, and more extensive redirects can be created with mod_rewrite.


## Using the Redirect Directive


In Apache, you can implement single-page redirects using the “Redirect” directive, which is included in the mod_alias module that is enabled by default. This directive takes at least two arguments: the old URL and the new URL.


Apache server blocks can be stored in the primary Apache settings file in /etc/apache2/apache2.conf, but are more maintainable if you create a new file in /etc/apache2/sites-available/ for each configuration. Apache’s convention is to create symbolic links (like shortcuts) from files in sites-available/ to another folder called sites-available/ as you decide to enable or disable them. By default, Apache is installed with a single site-specific configuration in /etc/apache2/sites-available/000-default.conf which is enabled and linked to /etc/apache2/sites-enabled/000-default.conf.


You can open that configuration using nano or your favorite text editor:


```
sudo nano /etc/apache2/sites-available/000-default.conf


```


By default, it contains a standard web server configuration that will listen on port 80 and look for an index.html file located in /var/www/html on your system.


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
# The ServerName directive sets the request scheme, hostname and port that
# the server uses to identify itself. This is used when creating
# redirection URLs. In the context of virtual hosts, the ServerName
# specifies what hostname must appear in the request's Host: header to
# match this virtual host. For the default virtual host (this file) this
# value is not decisive as it is used as a last resort host regardless.
# However, you must set it for any further virtual host explicitly.
#ServerName www.example.com

ServerAdmin webmaster@localhost
DocumentRoot /var/www/html
…
</VirtualHost>

```


If you instead needed to redirect requests for domain1.com to domain2.com, you could remove the existing directives from this server block and replace them with a permanent redirect:


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
	ServerName www.domain1.com
	Redirect / http://www.domain2.com
</VirtualHost>

<VirtualHost *:80>
	ServerName www.domain2.com
	. . .
	. . .
</VirtualHost>

```


Save and close the file. If you are using nano, press “Ctrl+X”, then when prompted, “Y” and then Enter.


This redirect instructs the browser to direct all requests for www.domain1.com to www.domain2.com. This is only for a single page, not for the entire site.


By default, the Redirect directive establishes a 302, or temporary, redirect. If you would like to create a permanent redirect, you can do so in either of the following two ways:


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
	ServerName www.domain1.com
Redirect 301 /oldlocation http://www.domain2.com/newlocation
</VirtualHost>

```


/etc/apache2/sites-available/000-default.conf
```
<VirtualHost *:80>
	ServerName www.domain1.com
Redirect permanent /oldlocation http://www.domain2.com/newlocation
</VirtualHost>

```


## Using the RedirectMatch Directive


To redirect more than a single page, you can use the RedirectMatch directive, which allows you to specify directory matching patterns using regular expressions.


This will allow you to redirect entire directories instead of just single files.


RedirectMatch matches patterns in parenthesis and then references the matched text in the redirect using “$1”, where 1 is the first group of text. Subsequent groups are given numbers sequentially.


For example, if you wanted to match every request for something within the /images directory to a subdomain named images.example.com, you could use the following:


/etc/apache2/sites-available/default
```
<VirtualHost *:80>
…
RedirectMatch ^/images/(.*)$ http://images.example.com/$1
…
</VirtualHost>

```


As with the Redirect directive, you can specify the type of redirect by adding the redirect code before the URL location rules.


In order for your changes to take effect, you’ll need to restart Apache. On a modern Ubuntu server, you can do this using systemctl:


```
sudo systemctl restart apache2


```


## Using mod_rewrite to Redirect


The most flexible, but complicated way to create redirect rules is with the module called mod_rewrite. For information on working with mod_rewrite, refer to How to Rewrite URLs with mod_rewrite for Apache.


In the next part of this tutorial, you will learn some example configurations for redirects using the Nginx web server. If you already installed and configured Apache, you will need to have port 80 free before following the configuration steps for Nginx on the same server. You can accomplish this by temporarily uninstalling Apache with apt remove, which will retain your configuration files should you later want to reinstall it, or by changing Apache’s configuration to listen for connections on a port other than 80, which is used by Apache, Nginx, and all HTTP connections by default.


# Step 3 – How to Redirect in Nginx


Redirects in Nginx are a core feature and make use of common Nginx directives. Most of the time, you can implement redirects by creating a new server block for each URL.


Nginx server blocks can be stored in the primary Nginx settings file in /etc/nginx/nginx.conf, but are more maintainable if you create a new file in /etc/nginx/sites-available/ for each configuration. Nginx’s convention is to create symbolic links (like shortcuts) from files in sites-available/ to another folder called sites-available/ as you decide to enable or disable them. By default, Nginx is installed with a single site-specific configuration in /etc/nginx/sites-available/default which is enabled and linked to /etc/nginx/sites-enabled/default.


You can open that configuration using nano or your favorite text editor:


```
sudo nano /etc/nginx/sites-available/default


```


By default, it contains a standard web server configuration that will listen on port 80 and look for an index.html file located in /var/www/html on your system.


/etc/nginx/sites-available/default
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;
…
        root /var/www/html;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;
…
}

```


If you instead needed to redirect requests for domain1.com to domain2.com, you could remove the existing directives from this server block and replace them with a permanent redirect:


/etc/nginx/sites-available/default
```
server {
	listen 80;
	server_name domain1.com;
	return 301 $scheme://domain2.com$request_uri;
}

```


Save and close the file. If you are using nano, press “Ctrl+X”, then when prompted, “Y” and then Enter.


The return directive executes a URL substitution and then returns the status code given to it and the redirection URL.


In this case, it uses the $scheme variable to use whatever scheme was used in the original request (http or https). It then returns the 301 permanent redirect code and the newly formed URL.


In order to process a folder redirect to a separate subdomain, you can perform an operation similar to the Apache folder redirection using the rewrite directive. This directive, when placed in a server block, will issue a temporary redirect for requests inside the images directory to the subdomain images.example.com:


/etc/nginx/sites-available/default
```
server {
…
rewrite ^/images/(.*)$ http://images.example.com/$1 redirect;
…
}

```


For a permanent redirect, you could change redirect to permanent at the end of the statement.


In order for your changes to take effect, you’ll need to restart Nginx. On a modern Ubuntu server, you can do this using systemctl:


```
sudo systemctl restart nginx


```


You can expand on this configuration in many other ways. Using these fundamentals of Nginx in order to produce your desired server configuration can keep your stack very lean, as well as helping to concisely document and handle any potential edge cases around your server traffic.


# Conclusion


In this tutorial, you learned to configure both temporary and permanent redirects for popular web servers. Be sure to use the correct redirection type, as an improper use of temporary redirects can hurt your search ranking.


Using redirects correctly will allow you to leverage your current web presence while allowing you to modify your site structure as necessary. If you would like to learn more about configuring redirects in Nginx, you can read How To Create Temporary and Permanent Redirects with Nginx


