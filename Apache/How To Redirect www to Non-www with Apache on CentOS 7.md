# How To Redirect www to Non-www with Apache on CentOS 7

```Apache``` ```CentOS```

## Introduction


Many web developers need to allow their users to access their website or application via both the www subdomain and the root (non-www) domain. That is, users should have the same experience when visiting www.my-website.com and my-website.com. While there are many ways to set this up, the most SEO-friendly solution is to choose which domain you prefer—the subdomain or the root domain—and have the web server redirect users who visit the other one to the preferred domain.


There are many kinds of HTTP redirects, but in this scenario, it’s best to use a 301 redirect, which tells clients, “The website you have requested has permanently moved to another URL. Go there instead.” Once the browser receives the HTTP 301 response code from the server, it sends a second request to the new URL given by the server and the user is presented with the website, probably never noticing they were redirected.


Why not configure your web server to just serve the same website for requests to both domain names? That may seem easier, but it does not confer the SEO advantages of the 301 redirect. A permanent redirect tells search engine crawlers that there is one canonical location for your website, and this improves the search rankings of that one URL.


In this tutorial, you will configure a 301 redirect using Apache on CentOS 7. If you are running Nginx instead of Apache, see this tutorial instead: How To Redirect www to Non-www with Nginx on CentOS 7
.


# Prerequisites


To complete this tutorial, you first need:


- Superuser privileges on the server that is running Apache. If you don’t already have that set up, follow our Initial Server Setup with CentOS 7 guide.
- Apache installed and configured to serve your website. Follow this tutorial to do that: How to Install the Apache Web Server on CentOS 7.
- A registered domain name. If you don’t have one yet, you can get a free one from Freenom. You may use whatever DNS provider you like (including your registrar) to host your domain’s records—just make sure to point your registrar to your provider’s nameservers. If you opt to use DigitalOcean DNS, this article from our documentation shows how to do that.

Let’s get started by configuring your DNS records.


# Step 1 — Configuring DNS Records


First, you need to point both www.my-website.com and my-website.com to your server running Apache. (The rest of the tutorial assumes your domain is my-website.com. Replace that with your own domain wherever you see it below.) You will do this by creating a DNS A record for each name that points to your Apache server’s IP address.


Open your DNS provider’s web console. This tutorial uses DigitalOcean DNS.


In the Add a domain form, enter your registered domain name in the text field and click Add Domain. This will bring up the new domain’s page, where you can view, add, and delete records for the domain.


Under Create new record, type “@” in the HOSTNAME text field. This is a special character that indicates you are adding a record for the root domain name, a record for just plain my-website.com. In the WILL DIRECT TO text field, enter the public IPv4 address of your server, and click Create Record. (No need to change the TTL.)


For your second DNS record, you could use a CNAME record instead of an A record. A CNAME record is an alias that points to another name instead of an IP address. You could create a CNAME that directs www.my-website.com to my-website.com, and any HTTP request for the www subdomain would find its way to your server since you just created the A record for the root domain. But to keep things simple, just create another A record like the first one, entering “www” in the HOSTNAME field and the server’s public IP address in the WILL DIRECT TO field.


When you have created both records, it should look something like this:





With the two records in place, web requests for both my-website.com and www.my-website.com should reach your Apache server. Now let’s configure the server.


# Step 2 — Configuring the Redirect in Apache


The Apache Web Server provides two modules to help you configure redirects: mod_alias and mod_rewrite. Although mod_rewrite is more powerful, mod_alias is more straightforward to understand and use. If you needed to redirect requests containing specific query strings, for example, or HTTP headers, you would need to use mod_rewrite. Many choose mod_rewrite for its regular expression matching capabilities, which mod_alias lacks. But for the simple case of redirecting all requests for www.my-website.com to my-website.com, mod_alias will do. (Apache itself recommends choosing mod_alias when possible, saying that choosing mod_rewrite when it’s unnecessary “leads to configurations which are confusing, fragile, and hard to maintain”.)


The module should be enabled by default on CentOS 7, but to double check, run this command:


```
httpd -M | grep alias_module


```


If alias_module (shared) appears in the output, the module is already enabled. If not, enable it by appending this line to /etc/httpd/conf.modules.d/00-base.conf:


```
echo “LoadModule alias_module modules/mod_alias.so” | sudo tee -a /etc/httpd/conf.modules.d/00-base.conf


```


With mod_alias enabled, you may use the directives Redirect, RedirectMatch, and others listed in the mod_alias docs in your Apache configuration.


Now let’s configure your VirtualHosts.


As stated in the Prerequisites, you should already have your website configured in Apache. It may be configured in the main Apache configuration file (/etc/httpd/conf/httpd.conf) or perhaps in its own file (e.g., /etc/httpd/conf.d/my-website.com.conf). If you used the Apache Install Guide linked in the Prerequisites to configure your VirtualHosts, it may be in a file like /etc/httpd/sites-available/my-website.com.conf. Wherever your main site is configured, open that file in vi or your favorite editor (yum install nano, if you prefer):


```
sudo vi /etc/httpd/conf/httpd.conf


```


Look for any ServerAlias directives set in the VirtualHost. If you find a line with ServerAlias set to www.my-website.com, remove that line. (Or, if that line contains many aliases in a comma-separated list, remove only www.my-website.com from the list.) You need to remove this alias because you are going to create a separate VirtualHost for the subdomain containing nothing but the ServerName and the Redirect. The main VirtualHost for the site will no longer serve requests for www.my-website.com.


Now create a VirtualHost in a separate file (e.g. /etc/httpd/conf.d/www.my-website.com.conf):


```
sudo vi /etc/httpd/conf.d/www.my-website.com.conf 


```


Paste the following contents into the file, replacing my-website.com with your own domain name:


/etc/httpd/conf.d/www.my-website.com.conf
```
<VirtualHost *:80>
    ServerName www.my-website.com
    Redirect permanent / http://my-website.com/
</VirtualHost>

```


Save and exit when you are finished. If you created this file in /etc/httpd/sites-available, as per our Apache Install Guide, create a symlink to the file in /etc/httpd/sites-enabled/:


```
sudo ln -s /etc/httpd/sites-available/www.my-website.com.conf /etc/httpd/sites-enabled/


```


This new VirtualHost configures Apache to send a 301 redirect back to any clients requesting www.my-website.com, and directs them to visit my-website.com instead. The redirect preserves the request URI, so that a request to http://www.my-website.com/login.php will be redirected to http://my-website.com/login.php.



Note: if your site’s main VirtualHost contains a ServerAlias with a wildcard subdomain—*.my-website.com—you should consider removing it and creating a new VirtualHost like the one you just created for every subdomain you want to redirect. If you don’t want to redirect all subdomains and need some of them to keep being served by the main VirtualHost, it is best to explicitly name each subdomain as an alias, especially now that you have one subdomain whose requests you don’t want inadvertently matched to the main VirtualHost. (You can name each subdomain on its own ServerAlias line, or name them all as a comma-separated list in one ServerAlias line.)
If you must keep the ServerAlias for *.my-website.com, you need to make sure Apache loads the new www VirtualHost before the main one, because if the main one is loaded first, Apache will use it to handle requests for www.my-website.com since that name matches the wildcard alias. Run the following command to see which VirtualHost will be loaded first once you restart Apache:
httpd -S


Look for the lines containing namevhost my-website.com and namevhost www.my-website.com. If the www line appears first, you’re all set. If the VirtualHost for the root domain appears first, however, there are a few ways to make Apache load the www one first:

If your main VirtualHost is contained in a file (e.g. /etc/httpd/conf/httpd.conf) that uses the Include or IncludeOptional directive to include the directory that contains your new www VirtualHost, just move that Include line above the main VirtualHost within the file.
If your main VirtualHost and your new www VirtualHost files are contained in the same directory (e.g., /etc/httpd/conf.d/), you can force Apache to load the www one first by renaming the files and prepending some numbers to the file names. Prepend 01- to the www VirtualHost’s file name (e.g., /etc/httpd/conf.d/01-www.my-website.com.conf) and 02- to the main VirtualHost’s file name (e.g., /etc/httpd/conf.d/02-my-website.com.conf).

Run httpd -S again to ensure that the www VirtualHost appears first.

When you are ready, restart Apache:


```
sudo systemctl restart httpd


```


Before visiting www.my-website.com in your browser, make a request using curl on either your server or your local machine (if curl is installed locally):


```
curl -IL http://www.my-website.com


```


The -I flag tells curl to show only the headers from the server response. The -L flag tells curl to obey any redirects from the server by automatically making a second request, this time to the URL given in the Location header (just as a web browser would do). Since you have configured the 301 redirect, curl should make two requests, and you should see just the headers of the two responses:


```
OutputHTTP/1.1 301 Moved Permanently
Date: Tue, 03 Jan 2023 19:24:44 GMT
Server: Apache/2.4.53
Location: http://my-website.com/
Content-Type: text/html; charset=iso-8859-1

HTTP/1.1 200 OK
Date: Tue, 03 Jan 2023 19:24:44 GMT
Server: Apache/2.4.53
Last-Modified: Thu, 01 Dec 2022 22:10:57 GMT
ETag: "39-5eecb7ed6bfc9"
Accept-Ranges: bytes
Content-Length: 57
Content-Type: text/html; charset=UTF-8

```


In the 301 (Moved Permanently) response to the original request to http://www.my-website.com, notice the second to last header: Location: http://my-website.com. The second response is from curl’s followup request to the URL given in that Location header, and if your website is healthy, the server should have responded with 200 (OK).


Finally, visit http://www.my-website.com in your web browser. Blink, and you’ll miss the redirect. Your website should appear as usual, but look again in your address bar and notice that the “www” is missing from the URL. Most users will never notice this, and so they will have the same experience as if they had requested http://my-website.com.


# Conclusion


In this tutorial, you added two DNS records for your website and configured Apache to redirect a secondary domain to your preferred domain. Now your website is reachable via both domains. Maybe it already was before you read this tutorial; perhaps you were serving it directly from both domain names. But with just four more lines of Apache configuration, you have improved your website’s standing in the eyes of the search engines—and thereby exposed it to more users across the internet.


Curious about the more powerful mod_rewrite module? Check out our tutorial How to Rewrite URLs with mod_rewrite for Apache on Ubuntu 22.04.


