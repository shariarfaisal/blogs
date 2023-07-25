# How To Configure Content Caching Using Apache Modules On A VPS

```Apache``` ```Caching```

## What is Web Caching


Web caching is a way of improving server performance by allowing commonly requested content to be stored in an easier to access location.  This allows the visitor to access the content faster instead of having to fetch the same data multiple times.


By effectively creating caching rules, content that is suitable for caching will be stored to conserve resources, while highly-dynamic content will be served normally.  In this guide, we will be discussing how to configure Apache using its caching modules.


Note: This guide was written with Apache 2.2 in mind.  Changes to Apache 2.4 have lead to the replacement of some of the modules discussed in this guide.  Due to this consideration, not all of the steps recommended below will work on Apache 2.4 installations.


# An Introduction to Caching in Apache


Apache has a number of different methods of caching content that is frequently accessed.  The two most common modules that enable this functionality are called "mod_cache" and "mod_file_cache".


## The mod_file_cache Module


The mod_file_cache module is the simpler of the two caching mechanisms.  It works by caching content that:


- Is requested frequently
- Changes very infrequently

If these two requirements are met, then mod_file_cache may be useful.  It works by performing some of the file access operations on commonly used files when the server is started.


## The mod_cache Module


The mod_cache module provides HTTP-aware caching schemes.  This means that the files will be cached according to an instruction specifying how long a page can be considered "fresh".


It performs these operations using either the "mod_mem_cache" module or the "mod_disk_cache" modules.  These are more complex caching models than mod_file_cache and are more useful in most circumstances.


# Using mod_file_cache with Apache


The mod_file_cache module is useful to cache files that will not change for the life of the current Apache instance.  The techniques used with this module will cause any subsequent changes to not be applied until the server is restarted.


These caching mechanisms can only be used with normal files, so no dynamically generated content or files generated by special content handlers will work here.


The module provides two directives that are used to accomplish caching in different ways.


## MMapFile


MMapFile is a directive used to create a list of files and then map those files into memory.  This is done only at server start up, so it is essential that none of the files set to use this type of caching are changed.


You can set up this type of caching in the server configuration file.  This is done by specifying files to be cached in memory in a space-separated list:


```
MMapFile /var/www/index.html /var/www/otherfile.html var/www/static-image.jpg
```


These files will be held in memory and served from there when the resource is requested.  If any of the files are changed, you need to restart the server.


## CacheFile


This directive works by opening handles to the files listed.  It maintains a table of these open file descriptors and uses it to cut time it takes to open these files.


Again, changes to the file during operation of the server will not be recognized by the cache.  The original contents will continue to be served until the server is restarted.


This directive is used by specifying a space-separated list of files that should be cached with this method:


```
CacheFile /this/file.html that/file.html another/file/to/server.html
```


This will cause these files to be cached on server start.


# Using mod_cache with Apache


The mod_cache module is a more flexible and powerful caching module.  It functions by implementing HTTP-aware caching of commonly accessed files.


While all caching mechanisms rely on serving files in some persistent state, mod_cache can handle changing content by configuring how long a file is valid for caching.


The module relies on two other modules to do the majority of the cache implementation.  These are "mod_disk_cache" and "mod_mem_cache".


The difference between these two is where the cache is kept, either on disk or in memory respectively.  These are stored and retrieved using URI based keys.  This is important to note as you can improve the caching of your site by turning on canonical naming.


This can be accomplished by putting this directive in the server configuration or virtual host definition:


```
UseCanonicalName On
```


# How to Configure Caching


We will examine some common configuration directives and how they affect the functionality of the caching mechanisms.


If you look in the "/etc/apache2/mods-available" directory, you can see some of the default configuration files for these modules.


## Configuring mod_mem_cache


Let's look at the mod_mem_cache configuration:


```
sudo nano /etc/apache2/mods-available/mem_cache.conf
```


```
<IfModule mod_mem_cache.c>
	CacheEnable mem /
	MCacheSize 4096
	MCacheMaxObjectCount 100
	MCacheMinObjectSize 1
	MCacheMaxObjectSize 2048
</IfModule>
```


```
These directives are only read if the mod_mem_cache module is loaded.  This can be done by typing the following:

```


```
sudo a2enmod mem_cache
sudo service apache2 restart
```


```
This will enable mod_mem_cache and also mod_cache.

```


```
CacheEnable mem /
```


The "CacheEnable mem /" line tells apache to create a memory cache for contents stored under "/" (which is everything).


```
MCacheSize 4096
MCacheMaxObjectCount 100
```


The next few lines describe the total size of the cache and the kinds of objects that will be stored.  The "MCacheSize" directive and the "MCacheMaxOjectCount" directive both describe the maximum size of the cache, first in terms of memory usage, and then in terms of the maximum amount of objects.


```
MCacheMinObjectSize 1
MCacheMaxObjectSize 2048
```


The next two lines describe the kinds of data that will be cached, in terms of memory usage.  The default values specify that files between 1 byte and 2 kilobytes will be considered for caching.


## Configuring mod_disk_cache


We can learn about a different set of directives by examining the mod_disk_cache configuration file:


```
sudo nano /etc/apache2/mods-available/disk_cache.conf
```


```
<IfModule mod_disk_cache.c>
	CacheRoot /var/cache/apache2/mod_disk_cache
	#CacheEnable disk /
	CacheDirLevels 5
	CacheDirLength 3
</IfModule>
```


This configuration is loaded if you enable the mod_disk_cache module, which can be done by typing:


```
sudo a2enmod disk_cache
sudo service apache2 restart
```


This command will also enable mod_cache in order to work properly.



```
CacheRoot /var/cache/apache2/mod_disk_cache
#CacheEnable disk /
```


The "CacheRoot" directive specifies where the cached content will be kept.  The "CacheEnable disk /" directive is disabled by default.  It is suggested that you enable this on a virtual host basis to get a better idea of what will be cached.


```
CacheDirLevels 5
CacheDirLength 3
```


The other two directives determine the caching structure within the cache root.  Each cached element is hashed by the its URL, and then the hash is used as a filename and directory path.


The CacheDirLevel decides how many directories to create from the hash string and the CacheDirLength decides how many characters are in each directory name.


For example, if you have a file that hashes to "abcdefghijklmnopqrstuvwxyz", then a CacheDirLevel of 2 and a CacheDirLength of 4 would lead to this file being stored in:


```
[path_of_cache_root]/abcd/efgh/ijklmnopqrstuv
```


Caching that is stored on disk can become large depending on the expiration dates of the content.  Apache includes a tool called "htcacheclean" to pare the cache down to a configured size.  This is outside the scope of this guide.


# Using CacheLock to Avoid Overwhelming the Backend


A problem can arise on a busy server when a cached resource expires.


The cache that needs to be refreshed will have to refetch the file from the normal file resource.  During this time, if there are more requests for the file.  This can create a huge spike in requests to the backend server as the cached version is being refreshed.


To avoid this situation, it is possible to enable a lock file that indicates that the resource is being recached and that subsequent requests should not go to the backend, because the issue is being addressed.


This lock can prevent apache from trying to cache the same resource multiple times when first caching.  It also will serve the stale resource until the refreshed cache is complete.


Three directives are used to control CacheLock:


```
CacheLock [ On | Off ]
CacheLockMaxAge [time_in_seconds]
CacheLockPath [/path/to/lock/directory]
```


The first directive turns on the feature and the third directive establishes the directory where resource locks will be created.


The second directive, CacheLockMaxAge, is used to establish the longest time in seconds that a lock file will be considered valid.  This is important in case there is a failure or an abnormal delay in refreshing a resource.


# Conclusion


Caching in Apache can be simple or involved depending on your needs.  While any kind of caching can improve your site performance, it is important to test your configurations to ensure that they are operating correctly.


It is also essential that you are familiar with the repercussions of improperly configured caching.  It is sometimes necessary to re-evaluate your security practices after implementing caching to ensure that private resources are not accidentally being cached for public consumption.


The apache user documentation has plenty of information about how to configure caching if you get stuck.  Even if you have a handle on the configuration, it is a helpful reference and good resource.


By Justin Ellingwood