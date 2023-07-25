# How To Find Broken Links on Your Website Using Wget on Debian 7

```Miscellaneous``` ```Debian```

## Introduction


How many times have you clicked a HTML link on a webpage only to get a 404 Not Found error? Broken links exist because webpages sometimes get moved or deleted over time. It is the job of the webmaster to find those broken links before the human web visitors or the search engine robots do. Delay in correcting the problem results in bad user experience and possible penalty for search engine page ranking.


If your website contains more than a few pages, manually checking each individual link becomes too labor intensive, but there are numerous tools that automate that task. You can use web-based apps, like ones provided by Google Webmaster Tools and the World Wide Web Consortium (W3C), but they generally lack more advanced features. If you run WordPress, you could use a plugin, but some shared web hosting companies ban them because they run on the same server as the website, and link checking is resource-intensive.


Another option is to use a Linux-based program on a separate machine. These include general web crawlers that also uncover broken links (like wget) and custom-built link checkers (like linkchecker and klinkstatus). They are highly customizable and minimize any negative impact on the response time of your target website.


This tutorial explains how to use wget to find all of the broken links on a website so you can correct them.


## Prerequisites


To follow this tutorial, you will need:


- 
Two Debian 7 Droplets, one generic machine to run wget from (generic-1) and one which hosts your website (webserver-1).

- 
A sudo non-root user on both generic-1 and webserver-1. Click here for instructions.

- 
webserver-1 needs to have the LAMP stack installed. Click here for instructions.

- 
Optionally, the webserver can have its own registered domain name. If so, use your domain name wherever you see your_server_ip. Click here for instructions.


Although this tutorial is written for Debian 7, the wget examples should also run on other modern Linux distributions. You may need to install wget on other distributions where it is not included by default.


# Step 1 — Creating a Sample Webpage


First, we’ll add a sample webpage with multiple missing links.


Log into webserver-1. Open a new file called spiderdemo.html for editing using nano or your favorite text editor.


```
sudo nano /var/www/spiderdemo.html

```


Paste the following into the file. This is a very simple webpage that includes two broken links, one internal (add in your server IP where it’s highlighted below) and one external.


```
<html>
<head> <title>Hello World!</title> </head>
<body>

<p>
<a href="http://your_server_ip/badlink1">Internal missing link</a>.
<a href="https://www.digitalocean.com/thisdoesntexist">External missing link</a>.
</p>

</body>
</html>

```


Save and close the file.


Next, change the file owner and group of spiderdemo.html to the default webserver user, www-data.


```
sudo chown www-data:www-data /var/www/spiderdemo.html

```


Finally, change the file permissions of the new HTML file.


```
sudo chmod 664  /var/www/spiderdemo.html

```


You can now see the example page at http://your_server_ip/spiderdemo.html.


# Step 2 — Running wget


wget is a general-purpose website downloader which can also be used as a web crawler. In this step, we’ll configure wget to report whether each link points to an existing page or is broken without downloading the page.


Note: Only check links on a website which you own. Link checking on a website incurs significant computing overhead, so these activities may be interpreted as spamming.


Log into generic-1 and run the following wget command. Explanations of each flag are below; you can modify this command for your use case.


```
wget --spider -r -nd -nv -H -l 1 -w 2 -o run1.log  http://your_server_ip/spiderdemo.html

```


The following are the basic flags you’ll need:


- 
--spider stops wget from downloading the page.

- 
-r makes wget recursively follow each link on the page.

- 
-nd, short for --no-directories, prevents wget from creating a hierarchy of directories on your server (even when it is configured to spider only).

- 
-nv, short for --no-verbose, stops wget from outputting extra information that is unnecessary for identifying broken links.


The following are optional parameters which you can use to customize your search:


- 
-H, short for --span-hosts,makes wget crawl to subdomains and domains other than the primary one (i.e. external sites).

- 
-l 1 is short for --level. By default, wget crawls up to five levels deep from the initial URL, but here we set it to one. You may need to play with this parameter depending on the organization of your website.

- 
-w 2, short for --wait, instructs wget to wait 2 seconds between requests to avoid bombarding the server, minimizing any performance impact.

- 
-o run1.log saves wget’s output to a file called run1.log instead of displaying it in your terminal.


After you run the above wget command, extract the broken links from the output file using the following command.


```
grep -B1 'broken link!' run1.log

```


The -B1 parameter specifies that, for every matching line, wget displays one additional line of leading context before the matching line. This preceding line contains the URL of the broken link. Below is sample output from the above grep command.


```
http://your_server_ip/badlink1:
Remote file does not exist -- broken link!!!
https://www.digitalocean.com/thisdoesntexist:
Remote file does not exist -- broken link!!!

```


# Step 3 — Finding Referrer URLs


Step 2 reports the broken links but doesn’t identify the referrer webpages, i.e., the pages on your site that contain those links. In this step, we’ll find the referrer webpages.


A convenient way to identify the referrer URL is by examining the webserver’s access log. Log in to webserver-1 and search the Apache logs for the broken link.


```
sudo grep Wget /var/log/apache2/access.log | grep "HEAD /badlink1"    

```


The first grep in the above command finds all access requests by wget to the webserver.  Each access request includes the User Agent string, which identifies the software agent responsible for generating the web request. The User Agent*identifier for wget is Wget/1.13.4 (linux-gnu).


The second grep searches for the partial URL of the broken link (/badlink1). The partial URL used is that part of the URL that follows the domain.


Sample output from the grep command chain is as follows:


```
111.111.111.111 - - [10/Apr/2015:17:26:12 -0800] "HEAD /badlink1 HTTP/1.1" 404 417 "http://your_server_ip/spiderdemo.html" "Wget/1.13.4 (linux-gnu)"   

```


The referrer URL is the second last item on the line: http://your_server_ip/spiderdemo.html.


## Conclusion


This tutorial explains how to use the wget tool to find the broken links on a website, and how to find the referrer pages which contain those links. You can now make corrections by updating or removing any broken links.


