# How To Configure Apache Content Caching on CentOS 7

```Apache``` ```Caching``` ```Server Optimization``` ```CentOS```

## What is Caching?


Caching is a method of improving server performance by allowing commonly requested content to be temporarily stored in a way that allows for faster access.  This speeds up processing and delivery by cutting out some resource intensive operations.


By creating effective caching rules, content that is suitable for caching will be stored to improve response times, conserve resources, and minimize load.  Apache provides a variety of caches suitable for speeding up different types of operations.  In this guide, we will be discussing how to configure Apache 2.4 on CentOS 7 using its various caching modules.


To learn more about developing general caching strategies, check out this article.


# An Introduction to Caching in Apache


Apache can cache content with varying levels of sophistication and scalability.  The project divides these into three groups according to the method in which the content is cached.  The general breakdown is:


- File Caching: The most basic caching strategy, this simply opens files or file descriptors when the server starts and keeps them available to speed up access.
- Key-Value Caching: Mainly used for SSL and authentication caching, key-value caching uses a shared object model that can store items which are costly to compute repeatedly.
- Standard HTTP caching: The most flexible and generally useful caching mechanism, this three-state system can store responses and validate them when they expire.  This can be configured for performance or flexibility depending on your specific needs.

A quick look at the above descriptions may reveal that the above methods have some overlap, but also that it may be helpful to use more than one strategy at the same time.  For instance, using a key-value store for your SSL sessions and enabling a standard HTTP cache for responses could allow you to take significant load off of your data sources and speed up many content delivery operations for your clients.


Now that you have a broad understanding of each of Apache’s caching mechanisms, let’s look at these systems in more detail.


# File Caching


## General Overview


- Primary modules involved: mod_file_cache
- Main use cases: storing either file contents or file descriptors when the server starts.  These are static representations that cannot reliably be changed until the server is restarted.
- Features: simple, improves performance of slow filesystems
- Drawbacks: experimental feature, does not respond to updates on the filesystem, must be used sparingly to fit within operating system’s limitations, can only be used on static files

## The Details


The mod_file_cache module is mainly used to speed up file access on servers with slow filesystems.  It provides a choice of two configuration directives, both of which aim to accelerate the process of serving static files by performing some of the work when the server is started rather than when the files are requested.


The CacheFile directive is used to specify the path to files on disk that you would like to accelerate access to.  When Apache is started, Apache will open the static files that were specified and cache the file handle, avoiding the need to open the file when it is requested.  The number of files that can be opened in this way is subject to the limitations set by your operating system.


The MMapFile directive also opens files when Apache is first started.  However, MMapFile caches the file’s contents in memory rather than just the file handler.  This allows for faster performance for those pages, but it has some serious limitations.  It maintains no record of the amount of memory it has used, so it is possible to run out of memory.  Also note that child processes will copy any of the allocated memory, which can result in faster resource depletion than you initially may anticipate.  Only use this directive sparingly.


These directives are evaluated only when Apache starts.  This means that you cannot rely on Apache to pick up changes made after it has started.  Only use these on static files that will not change for the lifetime of the Apache session.  Depending on how the files are modified, the server may be notified of changes, but this is not expected behavior and will not always work correctly.  If changes must be made to files passed to these directives, restart Apache after the changes have been made.


## How To Enable File Caching


File caching is provided by the mod_file_cache module.  To use this functionality, you’ll need to enable the module.


When running CentOS 7, the module will be installed when you install Apache, but the default configuration does not load the module.  To load the module, we’ll create a simple file in our /etc/httpd/conf.modules.d directory to load the module.  We’ll call this file 00-cache.conf:


```
sudo nano /etc/httpd/conf.modules.d/00-cache.conf


```


Inside, we need to use the LoadModule directive to enable the functionality we need.  Add the following line to the file:


/etc/httpd/conf.modules.d/00-cache.conf
```
LoadModule file_cache_module modules/mod_file_cache.so

```


Save and close the file when you’re finished.


Afterwards, you should edit the main configuration file to set up your file caching directives.  Open the file by typing:


```
sudo nano /etc/httpd/conf/httpd.conf


```


To set up file handle caching, use the CacheFile directive.  This directive takes a list of file paths, separated by spaces, like this:


/etc/httpd/conf/httpd.conf
```
CacheFile /var/www/html/index.html /var/www/html/somefile.index

```


When the server is restarted, Apache will open the files listed and store their file handles in the cache for faster access.


If, instead, you wish to map a few files directly into memory, you can use the MMapFile directive.  Its syntax is basically the same as the last directive, in that it simply takes a list of file paths:


/etc/httpd/conf/httpd.conf
```
MMapFile /var/www/html/index.html /var/www/html/somefile.index

```


In practice, there would be no reason to configure both CacheFile and MMapFile for the same set of files, but you could use both on different sets of files.


When you are finished, you can save and close the files.  Check the configuration file syntax by typing:


```
sudo apachectl configtest


```


If the last line reads Syntax OK, you can safely restart your Apache instance:


```
sudo systemctl restart httpd


```


Apache will restart, caching the file contents or handlers depending on the directives you used.


# Key-Value Caching


## General Overview


- Primary modules involved: mod_socache_dbm, mod_socache_dc, mod_socache_memcache, mod_socache_shmcb
- Supporting modules involved: mod_authn_socache, mod_ssl
- Main use cases: storing SSL sessions or authentication details, SSL stapling
- Features: shared object cache to store complex resources, can assist in SSL session caching and stapling, flexible backends
- Drawbacks: has no validation mechanisms, need to configure separate software for more performant/flexible backends, some bugs in code

## The Details


Key-value caching is more complex than file caching and has more focused benefits.  Also known as a shared object cache, Apache’s key-value cache is mainly used to avoid repeating expensive operations involved with setting up a client’s access to content, as opposed to the content itself.  Specifically, it can be used to cache authentication details, SSL sessions, and to provide SSL stapling.



Note
Currently, there are some issues with every shared object cache provider.  References to the issues will be outlined below.  Take these into consideration when evaluating whether to enable this feature.

The actual caching is accomplished through the use of one of the shared object caching provider modules.  These are:


- mod_socache_dbm: This backend uses the simple dbm database engine, which is a file-based key-value store that makes use of hashing and fixed-size buckets.  This provider suffers from some memory leaks, so for most cases it is recommended to use mod_socache_shmcb instead.
- mod_socache_dc: This provider uses the distcache session caching software.  This project has not been updated since 2004 and is not even packaged for some distributions, so use with a healthy dose of caution.
- mod_socache_memcache: This uses the memcache distributed memory object cache for storing items.  This is the best option for a distributed cache among multiple servers.  Currently, it does not properly expire entries, but a patch was committed to the trunk of Apache’s version control that fixes the issue.
- mod_socache_shmcb: Currently, this is the best option for key-value caching.  This caches to a cyclic buffer in shared memory, which will remove entries as it becomes full.  It currently chokes on entries over 11k in size.

Along with the above provider modules, additional modules will be required depending on the objects being cached.  For instance, to cache SSL sessions or to configure SSL stapling, mod_ssl must be enabled, which will provide the SSLSessionCache and SSLStaplingCache directives respectively.  Similarly, to set up authentication caching, the mod_authn_socache module must be enabled so that the AuthnCacheSOCache directive can be set.


## How To Enable Key-Value Caching


With the above bugs and caveats in mind, if you still wish to configure this type of caching in Apache, follow along below.


The method used to set up the key-value cache will depend on what it will be used for and what provider you are using.  We’ll go over the basics of both authentication caching and SSL session caching below.


Currently, there is a bug with authentication caching that prevents passing arguments to the cache provider.  So any providers that do not provide default settings to fall back on will have issues.


##$ Authentication Caching


Authentication caching is useful if you are using an expensive authentication method, such as LDAP or database authentication.  These types of operations can have a significant impact on performance if the backend must be hit every time an authentication request is made.


Setting up caching involves modifying your existing authentication configuration (we will not cover how to set up authentication in this guide).  The modifications themselves will be much the same regardless of the backend authentication method.  We’ll use mod_socache_shmcb for our demonstration.  This module is already enabled in our /etc/httpd/conf.modules.d/00-base.conf file.


Open your main Apache configuration file so that you can specify this shared cache backend for use with authentication:


```
sudo nano /etc/httpd/conf/httpd.conf


```


Inside, towards the top of the file, add the AuthnCacheSOCache directive.  Specify that shmcb should be used as the provider.  If the bug discussed earlier preventing option passing is fixed by the time you read this, you can specify a location and size for the cache.  The number is in bytes, so the commented example will result in a 512 kilobyte cache:


/etc/httpd/conf/httpd.conf
```
AuthnCacheSOCache shmcb

# If the bug preventing passed arguments to the provider gets fixed,
# you can customize the location and size like this
#AuthnCacheSOCache shmcb:${APACHE_RUN_DIR}/auth_cache(512000)

```


Save and close the file when you are finished.


Next, open your virtual host configuration page that has authentication configured.  We’ll assume you’re using a virtual host config called site.conf located within the /etc/httpd/conf.d directory, but you should modify it to reflect your environment:


```
sudo nano /etc/httpd/conf.d/site.conf


```


A basic virtual host set up with authentication may look something like this:


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    <Directory /var/www/html/private>
        AuthType Basic
        AuthName "Restricted Files"
        AuthUserFile /etc/httpd/.htpasswd
        AuthBasicProvider file
        Require valid-user
    </Directory>
</VirtualHost>

```


In the location where you’ve configured authentication, modify the block to add caching.  Specifically, you need to add the AuthnCacheProvideFor to tell it which authentication sources to cache, add a cache timeout with AuthnCacheTimeout, and add socache to the AuthBasicProvider list ahead of your conventional authentication method.  The results will look something like this:


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    <Directory /var/www/html/private>
        AuthType Basic
        AuthName "Restricted Files"
        AuthUserFile /etc/apache/.htpasswd
        AuthBasicProvider socache file
        AuthnCacheProvideFor file
        AuthnCacheTimeout 300
        Require valid-user
    </Directory>
</VirtualHost>

```


The above example is for file authentication, which probably won’t benefit from caching very much.  However, the implmentation should be very similar when using other authentiation methods.  The only substantial difference would be where the “file” specification is in the above example, the other authentication method would be used instead.


Save and close the file.  Check your changes for syntax errors by typing:


```
sudo apachectl configtest


```


If no syntax errors are found, restart Apache to implement your caching changes:


```
sudo systemctl restart httpd


```


##$ SSL Session Caching


The handshake that must be performed to establish an SSL connection carries significant overhead.  As such, caching the session data to avoid this initialization step for further requests can potentially skirt this penalty.  The shared object cache is a perfect place for this.


If you have SSL already configured for your Apache server, mod_ssl will be enabled (if not, use yum to install the mod_ssl module).  On CentOS 7, this means that an ssl.conf file will be available in the /etc/httpd/conf.d directory.  This actually already sets up caching.  Inside, you will see some lines like this:


/etc/httpd/conf.d/ssl.conf
```
. . .

SSLSessionCache         shmcb:/run/httpd/sslcache(512000)
SSLSessionCacheTimeout  300

. . .

```


This is actually enough to set up session caching.  To test this, you can use OpenSSL’s connection client.  Type:


```
openssl s_client -connect 127.0.0.1:443 -reconnect -no_ticket | grep Session-ID


```


If the session ID is the same in all of the results, your session cache is working correctly.  Press CTRL-C to exit back to the terminal.


# Standard HTTP Caching


## General Overview


- Primary modules involved: mod_cache
- Supporting modules involved: mod_cache_disk, mod_cache_socache
- Main use cases: Caching general content
- Features: Can correctly interpret HTTP caching headers, can revalidate stale entries, can be deployed for maximum speed or flexibility depending on your needs
- Drawbacks: Can leak sensitive data if incorrectly configured, must use additional modules to correctly set the caching policy

## The Details


The HTTP protocol encourages and provides the mechanisms for caching responses all along the content delivery path.  Any computer that touches the content can potentially cache each item for a certain amount of time depending on the caching policies set forth at the content’s origins and the computer’s own caching rules.


The Apache HTTP caching mechanism caches responses according to the HTTP caching policies it sees.  This is a general purpose caching system that adheres to the same rules that any intermediary server would follow that has a hand in the delivery.  This makes this system very flexible and powerful and allows you to leverage the headers that you should already be setting on your content (we’ll cover how to do this below).


Apache’s HTTP cache is also known as a “three state” cache.  This is because the content it has stored can be in one of three states.  It can be fresh, meaning it is allowed to be served to clients with no further checking, it can be stale, meaning that the TTL on the content has expired, or it can be non-existent if the content is not found in the cache.


If the content becomes stale, at the next request, the cache can revalidate it by checking the content at the origin.  If it hasn’t changed, it can reset the freshness date and serve the current content.  Otherwise, it fetches the changed content and stores that for the length of time allowed by its caching policy.


##$ Module Overview


The HTTP caching logic is available through the mod_cache module.  The actual caching is done with one of the caching providers.  Typically, the cache is stored on disk using the mod_cache_disk module, but shared object caching is also available through the mod_cache_socache module.


The mod_cache_disk module caches on disk, so it can be useful if you are proxying content from a remote location, generating it from a dynamic process, or just trying to speed things up by caching on a faster disk than your content typically resides on.  This is the most well-tested provider and should probably be your first choice in most cases.  The cache is not cleaned automatically, so a tool called htcacheclean must be run occasionally to slim down the cache.  This can be run manually, set up as a regular cron job, or run as a daemon.


The mod_cache_socache module caches to one of the shared object providers (the same ones discussed in the last section).  This can potentially have better performance than mod_cache_disk (depending on which shared cache provider is selected).  However, it is much newer and relies on the shared object providers, which have the bugs discussed earlier.  Comprehensive testing is recommended before implementing the mod_cache_socache option.


##$ HTTP Cache Placement


Apache’s HTTP cache can be deployed in two different configurations depending on your needs.


If the CacheQuickHandler is set to “on”, the cache will be checked very early in the request handling process.  If content is found, it will be served directly without any further handling.  This means that it is incredibly quick, but it also means that it does not allow for processes like authentication for content.  If there is content in your cache that normally requires authentication or access control, it will be accessible to anyone without authentication if the CacheQuickHandler is set to “on”.


Basically, this emulates a separate cache in front of your web server.  If your web server needs to do any kind of conditional checking, authentication, or authorization, this will not happen.  Apache will not even evaluate directives within <Location> or <Directory> blocks.  Note that CacheQuickHandler is set to “on” by default!


If the CacheQuickHandler is set to “off”, the cache will be checked significantly later in the request processing sequence.  Think of this configuration as placing the cache between your Apache processing logic and your actual content.  This will allow the conventional processing directives to be run prior to retrieving content from the cache.  Setting this to “off” trades a bit of speed for the ability to process requests more deeply.


## How To Configure Standard HTTP Caching


In order to enable caching, you’ll need to enable the mod_cache module as well as one of its caching providers.  As we stated above, mod_cache_disk is well tested, so we will rely on that.


##$ Setting Up htcacheclean to Automatically Manage the Cache


On a CentOS 7 system, the htcacheclean utility, which is used to pare down the cache as it grows, is installed during the httpd installation.  A systemd unit file called htcacheclean.service is included by default.


If you plan on setting up caching, it is a good idea to configure this service to run automatically.  The service file actually daemonizes the service, running the cleaning operation at a configurable interval.  However, by default, there is not a mechanism for starting it when Apache starts.


To set this up, we’ll make a directory called httpd.service.requires within the /etc/systemd/system directory.  This can be used to specify dependencies of the httpd.service unit file that starts Apache:


```
sudo mkdir -p /etc/systemd/system/httpd.service.requires


```


Afterwards, we can link the htcacheclean.service unit file into this directory.  This will cause the htcacheclean service to start and clean the cache at regular intervals when Apache is started:


```
sudo ln -s /usr/lib/systemd/system/htcacheclean.service /etc/systemd/system/httpd.service.requires

```


You can configure the htcacheclean options, including the cleaning interval, by editing the htcacheclean file in the /etc/sysconfig directory:


```
sudo nano /etc/sysconfig/htcacheclean

```


Here, you can modify the cleaning interval, the cache root, the maximum cache size, and any other options for the utility:


/etc/sysconfig/htcacheclean
```
INTERVAL=15
CACHE_ROOT=/var/cache/httpd/proxy
LIMIT=100M
OPTIONS=

```



Note
Keep in mind that if you change the value of CACHE_ROOT, you will need to adjust the value the CacheRoot directive that we will be setting in our Apache configuration.

When you are finished, save and close the file.


Restart Apache to start the htcacheclean to automatically clean the cache:


```
sudo systemctl restart httpd

```


##$ Modifying the Global Configuration


Most of the configuration for caching will take place within individual virtual host definitions or location blocks.  However, there are also some global configuration items that should be used to set up some general attributes.  Open your main Apache config file to configure these items:


```
sudo nano /etc/httpd/conf/httpd.conf


```


We need to add the CacheRoot directory to point to the path that should be used to store our cached items.  This should always match the CACHE_ROOT value found in your /etc/sysconfig/htcacheclean file so that the cache can be managed correctly.  We will also set the CacheDirLevels and CacheDirLength directives, which both contribute towards defining how the cache directory structure will be built:


/etc/httpd/conf/httpd.conf
```
CacheRoot /var/cache/httpd/proxy
CacheDirLevels 2
CacheDirLength 1

```


The CacheDirLevels and CacheDirLength both contribute towards defining how the cache directory structure will be built.  An md5 hash of the URL being served will be created as the key used to store the data.  The data will be organized into directories derived from the beginning characters of each hash.  CacheDirLevels specifies the number of subdirectories to create and CacheDirLength specifies how many characters to use as the name of each directory.  So a hash of b1946ac92492d2347c6235b4d2611184 with the default values shown above would be filed in a directory structure of b/1/946ac92492d2347c6235b4d2611184.  Usually, you won’t need to modify these values, but it’s good to know what they’re used for.


Some other values you can set in this file are CacheMaxFileSize and CacheMinFileSize which set the ranges of file sizes in bytes that Apache will commit to the cache, as well as CacheReadSize and CacheReadTime, which allows you to wait and buffer content before sending to the client.  This can be useful if the content resides somewhere other than this server.


##$ Modifying the Virtual Server


Most of the configuration for caching will happen on a more granular level, either in the virtual host definition or in a specific location block.


Open one of your virtual host files to follow along.  We’ll assume you’re using a file called site.conf within your /etc/httpd/conf.d directory in this guide:


```
sudo nano /etc/httpd/conf.d/site.conf


```


In the virtual host block, outside of any location block, we can begin configuring some of the caching properties.  In this guide, we’ll assume that we want to turn the CacheQuickHandler off so that more processing is done.  This allows us up more complete caching rules.


We will also take this opportunity to configure cache locking.  This is a system of file locks that Apache will use when it is checking in with the content origin to see whether content is still valid.  During the time when this query is being satisfied, if additional requests for the same content come in, it would result in additional requests to the backend resource, which could cause load spikes.


Setting a cache lock for a resource during validation tells Apache that the resource is currently being refreshed.  During this time, the stale resource can be served with a warning header indicating its state.  We’ll set this up with a cache lock directory in the /tmp folder.  We’ll allow a maximum of 5 seconds for a lock to be considered valid.  These examples are taken directly from Apache’s documentation, so they should work well for our purposes.


We will also tell Apache to ignore the Set-Cookie headers and not store them in the cache.  Doing so will prevent Apache from accidentally leaking user-specific cookies out to other parties.  The Set-Cookie header will be stripped before the headers are cached.


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    CacheQuickHandler off

    CacheLock on
    CacheLockPath /tmp/mod_cache-lock
    CacheLockMaxAge 5

    CacheIgnoreHeaders Set-Cookie
</VirtualHost>

```


We still need to actually enable caching for this virtual host.  We can do this with the CacheEnable directive.  If this is set in a virtual host block, we would need to provide the caching method (disk or socache) as well as the requested URIs that should be cached.  For example, to cache all responses, this could be set to CacheEnable disk /, but if you only wanted to cache responses under the /public URI, you could set this to CacheEnable disk /public.


We will take a different approach by enabling our cache within a specific location block.  Doing so means we don’t have to provide a URI path to the CacheEnable command.  Any URI that would be served from that location will be cached.  We will also turn on the CacheHeader directive so that our response headers will indicate whether the cache was used to serve the request or not.


Another directive we’ll set is CacheDefaultExpire so that we can set an expiration (in seconds) if neither the Expires nor the Last-Modified headers are set on the content.  Similarly, we’ll set CacheMaxExpire to cap the amount of time items will be saved.  We’ll set the CacheLastModifiedFactor so that Apache can create an expiration date if it has a Last-Modified date, but no expiration.  The factor is multiplied by the time since modification to set a reasonable expiration.


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    CacheQuickHandler off

    CacheLock on
    CacheLockPath /tmp/mod_cache-lock
    CacheLockMaxAge 5

    CacheIgnoreHeaders Set-Cookie

    <Location />
        CacheEnable disk
        CacheHeader on

        CacheDefaultExpire 600
        CacheMaxExpire 86400
        CacheLastModifiedFactor 0.5
    </Location>
</VirtualHost>

```


Save and close your file when you’ve configured everything that you need.


Check your entire configuration for syntax errors by typing:


```
sudo apachectl configtest


```


If no errors are reported, restart your service by typing:


```
sudo systemctl restart httpd


```


# Setting Expires and Caching Headers on Content


In the above configuration, we configured HTTP caching, which relies on HTTP headers.  However, none of the content we’re serving actually has the Expires or Cache-Control headers needed to make intelligent caching decisions.  To set these headers, we need to take advantage of a few more modules.


The mod_expires module can set both the Expires header and the max-age option in the Cache-Control header.  The mod_headers module can be used to add more specific Cache-Control options to tune the caching policy further.  Both of these modules are enabled by default in the CentOS 7 Apache package.


We can go straight to modifying our virtual host file again to begin configuring these items:


```
sudo nano /etc/httpd/conf.d/site.conf


```


The mod_expires module provides just three directives.  The ExpiresActive turns expiration processing on in a certain context by setting it to “on”.  The other two directives are very similar to each other.  The ExpiresDefault directive sets the default expiration time, and the ExpiresByType sets the expiration time according to the MIME type of the content.  Both of these will set the Expires and the Cache-Control “max-age” to the correct values.


These two settings can take two different syntaxes.  The first is simply “A” or “M” followed by a number of seconds.  This sets the expiration in relation to the last time the content was “accessed” or “modified” respectively.  For example, these both would expire content 30 seconds after it was accessed.


```
ExpiresDefault A30
ExpireByType text/html A30

```


The other syntax allows for more verbose configuration.  It allows you to use units other than seconds that are easier for humans to calculate.  It also uses the full word “access” or “modification”.  The entire expiration configuration should be kept in quotes, like this:


```
ExpiresDefault "modification plus 2 weeks 3 days 1 hour"
ExpiresByType text/html "modification plus 2 weeks 3 days 1 hour"

```


For our purposes, we’ll just set a default expiration.  We will start by setting it to 5 minutes so that if we make a mistake while getting familiar, it won’t be stored on our clients’ computers for an extremely long time.  When we’re more confident in our ability to select policies appropriate for our content, we can adjust this to something more aggressive:


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    CacheQuickHandler off

    CacheLock on
    CacheLockPath /tmp/mod_cache-lock
    CacheLockMaxAge 5

    CacheIgnoreHeaders Set-Cookie

    <Location />
        CacheEnable disk
        CacheHeader on

        CacheDefaultExpire 600
        CacheMaxExpire 86400
        CacheLastModifiedFactor 0.5

        ExpiresActive on
        ExpiresDefault "access plus 5 minutes"
    </Location>
</VirtualHost>

```


This will set our Expires header to five minutes in the future and set Cache-Control max-age=300.  In order to refine our caching policy further, we can use the Header directive.  We can use the merge option to add additional Cache-Control options.  You can call this multiple times and add whichever additional policies you’d like.  Check out this guide to get an idea about the caching policies you’d like to set for your content.  For our example, we’ll just set “public” so that other caches can be sure that they’re allowed to store copies.


To set ETags for static content on our site (to use for validation), we can use the FileETag directive.  This will work for static content.  For dynamically generated content, you’re application will be responsible for correctly generating ETags.


We can use the directive to set the attributes that Apache will use to calculate the Etag.  This can be INode, MTime, Size, or All depending on if we want to modify the ETag whenever the file’s inode changes, its modification time changes, its size changes, or all of the above.  You can provide more than one value, and you can modify the inherited setting in child contexts by preceding the new settings with a + or -.  For our purposes, we’ll just use “all” so that all changes are registered:


/etc/httpd/conf.d/site.conf
```
<VirtualHost *:80>
    ServerName server_domain_or_IP
    DocumentRoot /var/www/html

    CacheQuickHandler off

    CacheLock on
    CacheLockPath /tmp/mod_cache-lock
    CacheLockMaxAge 5

    CacheIgnoreHeaders Set-Cookie

    <Location />
        CacheEnable disk
        CacheHeader on

        CacheDefaultExpire 600
        CacheMaxExpire 86400
        CacheLastModifiedFactor 0.5

        ExpiresActive on
        ExpiresDefault "access plus 5 minutes"

        Header merge Cache-Control public
        FileETag All
    </Location>
</VirtualHost>

```


This will add “public” (separated by a comma) to whatever value Cache-Control already has and will include an ETag for our static content.


When you are finished, save and close the file.  Check the syntax of your changes by typing:


```
sudo apachectl configtest


```


If no errors were found, restart your service to implement your caching policies:


```
sudo systemctl restart httpd


```


# Conclusion


Configuring caching with Apache can seem like a daunting job due to how many options there are.  Luckily, it is easy to start simple and then grow as you require more complexity.  Most administrators will not require each of the caching types.


When configuring caching, keep in mind the specific problems that you’re trying to solve to avoid getting lost in the different implementation choices.  Most users will benefit from at least setting up headers.  If you are proxying or generating content, setting an HTTP cache may be helpful.  Shared object caching is useful for specific tasks like storing SSL sessions or authentication details if you are using a backend provider.  File caching can probably be limited to those with slow systems.


