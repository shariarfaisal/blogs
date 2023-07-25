# How To Use WP Super Cache and Jetpack Photon To Optimize WordPress Performance on Ubuntu 14 04

```Ubuntu``` ```WordPress``` ```Server Optimization``` ```Caching``` ```Nginx```

## Introduction


In this tutorial, we will teach you how to optimize WordPress performance by using the WP Super Cache and Jetpack Photon plugins, and Nginx as the web server. With this setup, your WordPress site can greatly increase its concurrent user capacity by taking advantage of caching techniques that the aforementioned plugins provide.


WP Super Cache works by caching your WordPress pages as static HTML pages so that page requests, for an already cached page, do not need to be processed by the WordPress PHP scripts. Typically, most visitors of your site will view cached versions of the WordPress pages, so your server will have more processing power to serve an increased number of users. The WP Super Cache plugin is developed by Donncha O Caoimh.


Jetpack Photon is an image acceleration service that works by caching and serving your WordPress images via its own Content Delivery Network (CDN). Photon is one of the modules included in the Jetpack plugin, which is developed by the Jetpack Team of Automattic.


# Prerequisites


To follow this tutorial, you will need a WordPress server that uses Nginx as its web server. If you do not have that, you may use these tutorials to create one:


- How To Install Linux, nginx, MySQL, PHP (LEMP) stack on Ubuntu 14.04
- How To Install WordPress with Nginx on Ubuntu 14.04

## Plugin Requirements or Limitations


WP Super Cache does not work with plugins that use query arguments because it does not work if you pass query arguments to Nginx. Also, because of this, you must not use the WordPress Default Permalink settings (which uses WordPress page numbers as arguments).


Jetpack Photon Limitations:


- You must connect your site to WordPress.com to enable Jetpack, which requires a free WordPress.com account
- Your WordPress site must listen on port 80 (Photon will not work with HTTPS-only sites)
- Once an gif, jpg, or png image is cached, it cannot be updated. The only workaround is to re-upload a renamed image to your site.
- Images that take too long to copy to the Photon CDN (more than 10 seconds) must be renamed and re-uploaded

If you do not want to use Photon, feel free to skip that section of the tutorial.


Now that we have the prerequisites out of the way, let’s start installing WP Super Cache!


# Install and Configure WP Super Cache Plugin


The first step to installing the WP Super Cache Plugin is to download it from wordpress.org to your home directory:


```
cd ~; wget http://downloads.wordpress.org/plugin/wp-super-cache.1.4.zip

```


If you do not have the unzip package installed, do it now:


```
sudo apt-get install unzip

```


Then unzip the WP Super Cache plugin to your WordPress plugins directory (replace the highlighted path with your own, if you installed WordPress somewhere else):


```
cd /var/www/html/wp-content/plugins
unzip ~/wp-super-cache.1.4.zip

```


Next, we will change the group ownership of the plugin:


```
sudo chgrp -R www-data wp-super-cache

```


And we will allow the plugin to write to the wp-content directory and the wp-config.php file:


```
chmod g+w /var/www/html/wp-content
chmod g+w /var/www/html/wp-config.php

```


Now that the WordPress files are set up properly, let’s activate the plugin.


## Activate WP Super Cache Plugin


Log in to your WordPress site as your administrator user, and go to Dashboard (http://example.com/wp-admin/). Activate the WP Super Cache plugin, then go into its settings window, by following these steps:


1. Click on Plugins (left bar)
2. Click on Activate directly beneath WP Super Cache
3. Click on WP Super Cache Settings

## Enable Caching


Now we will enable caching and configure WP Super Cache with some reasonable settings:


1. Click the Advanced tab
2. Check Cache hits to this website for quick access.
3. Select Use mod_rewrite to serve cache files.

This configures WP Super Cache to cache files that are accessed, and the mod_rewrite setting leaves it up to Nginx to serve the cached files. We are not actually going to use mod_rewrite because it is an Apache plugin, and we are using Nginx as our web server, but we will need to update our Nginx server block configuration so that Nginx appropriately serves the cached files. We will get to that after we tweak a few more WP Super Cache settings (note: the following settings are optional):


1. Check Compress pages so they’re served more quickly to visitors.
2. Check Don’t cache pages for known users.
3. Check Cache rebuild.
4. Check Extra homepage checks.

Next, you need to save your settings by clicking the Update Status button, which should be below the settings you just changed:





WP Super Cache is now configured to cache your WordPress pages. We still need to configure Nginx to serve the cached files, but let’s look at a few other things in the WP Super Cache settings window.


## Warnings About Mod Rewrite and Garbage Collection


At this point, you will see some warning banners at the top of the WP Super Cache configuration window. There will be two warnings about Mod Rewrite rules (here is the first one):





You may ignore this because we are going to use Nginx instead of Apache.


Next, you will see a warning about Garbage Collection settings:





This warning can be removed by dismissing it (i.e. click the “Dismiss” button) or by configuring garbage collection. To configure garbage collection, go to the Expiry Time & Garbage Collection section in the Advanced tab, then configure it to your liking, then click the Change Expiration button.


## Viewing Cache Contents


You can see the list of all of the cached pages by going to the Contents tab of the WP Super Cache settings. Here you will see the “Cache stats”, which shows how many files are cached (and which files are cached). You may also delete the current cache from here.


WP Super Cache only caches pages visited by users who aren’t logged in, haven’t left a comment, or haven’t viewed a password protected post. So if you are wondering why pages that you are visiting aren’t being cached, try viewing your WordPress site in private browsing mode. Also, Nginx is not yet configured to serve cached files, so you will not see any improvements in access times.


## Additional WP Super Cache Configuration


In addition to the settings discussed above, there are many others that you might find to be useful or interesting. We will briefly go over the CDN and Preloading tabs.


Using a CDN – Skip if you are going to use Jetpack Photon


If you use a CDN, be sure to enable CDN support in the CDN tab. All the settings that you need to offload your static assets are located here.


Preloading Cache


In the Preload tab, you can configure WP Super Cache to automatically cache pages. This can be configured to preload your entire site or a fixed number of your recent posts on a time interval that you specify. Preloading pages takes system resources (CPU to retrieve pages, and disk space to store the static pages), so keep that in consideration when deciding if you want to enable it.


# Configure Nginx To Serve Cached Files


Now that your WordPress site is caching pages with WP Super Cache, you must configure Nginx to serve the cached files. Edit the Nginx server block configuration:


```
sudo vi /etc/nginx/sites-enabled/wordpress

```


If you followed the prerequisite tutorials, place the following configuration lines directly beneath the server_name line:


```
	set $cache_uri $request_uri;

	# POST requests and urls with a query string should always go to PHP
	if ($request_method = POST) {
		set $cache_uri 'null cache';
	}   
	if ($query_string != "") {
		set $cache_uri 'null cache';
	}   

	# Don't cache uris containing the following segments
	if ($request_uri ~* "(/wp-admin/|/xmlrpc.php|/wp-(app|cron|login|register|mail).php|wp-.*.php|/feed/|index.php|wp-comments-popup.php|wp-links-opml.php|wp-locations.php|sitemap(_index)?.xml|[a-z0-9_-]+-sitemap([0-9]+)?.xml)") {
		set $cache_uri 'null cache';
	}   

	# Don't use the cache for logged in users or recent commenters
	if ($http_cookie ~* "comment_author|wordpress_[a-f0-9]+|wp-postpass|wordpress_logged_in") {
		set $cache_uri 'null cache';
	}

	# Use cached or actual file if they exists, otherwise pass request to WordPress
	location / {
		try_files /wp-content/cache/supercache/$http_host/$cache_uri/index.html $uri $uri/ /index.php ;
	}    

```


Then delete the lines that follow until location ~ \.php$ {.


Restart Nginx to put the configuration changes into effect:


```
sudo service nginx restart

```


Now your WordPress site’s pages will be cached via WP Super Cache! If you want to also cache your images, using Jetpack Photon, continue on to the next section.


# Install and Enable Jetpack Photon


Download the Jetpack plugin to your home directory:


```
cd ~; wget http://downloads.wordpress.org/plugin/jetpack.latest-stable.zip

```


Then unzip the Jetpack archive in your WordPress plugins directory:


```
cd /var/www/html/wp-content/plugins
unzip ~/jetpack.latest-stable.zip
sudo chgrp -R www-data jetpack

```


Jetpack comes with several modules other than Photon, many of which are enabled by default. If you want to use the other Jetpack modules, in addition to Jetpack, skip the following edit, and activate the Photon module through the Jetpack plugin settings on your WordPress administrator dashboard. Otherwise, we can disable the other modules by adding a few lines of code to the plugin’s PHP files.


Open wp-config.php for editing:


```
vi /var/www/html/wp-config.php

```


Go to the end of the file and add the following lines of code:


```
function change_default_modules() {
    return array( 'photon' );  // activate these modules by default
}
add_filter( 'jetpack_get_default_modules', 'change_default_modules' );

function activate_specific_jetpack_modules( $modules ) {
        $active_modules = array( 'photon' );  // enable these modules
        $modules = array_intersect_key( $modules, array_flip( $active_modules ) );  // deactivate other modules
        return $modules;
}
add_filter( 'jetpack_get_available_modules', 'activate_specific_jetpack_modules' );

```


Save and quit. Now when you activate the Jetpack plugin, it will only load the Photon module and disable the use of all of the other Jetpack modules.


# Activate Jetpack Plugin


Now log in to your WordPress site as your administrator user, and go to Dashboard (http://example.com/wp-admin/). Activate the Jetpack plugin, then go into its settings, by following these steps:


1. Click on Plugins (left bar)
2. Click on Activate directly beneath Jetpack
3. Click Connect to WordPress.com, in the green banner near the top of Plugins window
4. Enter your WordPress.com login and click Authorize Jetpack




Now all of the images on your WordPress site (.png, .jpg, .gif) will be served from Jetpack’s Photon CDN. Here are a few ways your server will be affected:


- Less bandwidth consumption: Your server will use less outgoing bandwidth because the Photon CDN, which is provided by WordPress.com, will serve your site’s images
- Less resource consumption: It will consume less CPU and memory because it no longer serves images to users, and mostly only static pages
- More user capacity: It will be able to handle more concurrent users because it is using less resources per request

That’s it! The Photon CDN will cache and serve your images as they are requested. Note that you can disable Photon in the Jetpack plugin settings at any time, if you decide that you do not want to use it.


# Performance Comparison


To show you an idea of the potential performance benefit of this setup, we set up two 1 CPU / 1GB RAM VPSs (one without WP Super Cache, one with it) and we used Apache JMeter to perform a load test against them (multiple users accessing 5 WordPress pages over 10 seconds in a loop).


The non-cached server was able to handle about 3 simulated users per second before showing performance issues due to CPU utilization.


The cached server, with WP Super Cache installed, was able to serve over 50 simulated users per second (millions a day) without showing any performance degradation–in fact, it returned the requests more quickly because the requested pages were cached!


A tutorial on how to use Apache JMeter to perform your own load testing is available here: How To Use Apache JMeter To Perform Load Testing on a Web Server


# Conclusion


Now that you have WP Super Cache and Jetpack Photon installed, you should be able to serve many more users than before. You may want to play with the WP Super Cache settings until you feel like you have a configuration that best suits your needs.


Feel free to post questions or your own performance comparisons!


