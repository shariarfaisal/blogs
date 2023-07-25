# How To Install MediaWiki on Centos 6 4

```CentOS```


Status: Deprecated
This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.
Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.
See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.
The following DigitalOcean tutorial may be of immediate interest, as it outlines installing MediaWiki on a CentOS 7 server:

How To Install MediaWiki on CentOS 7


## Mediawiki



MediaWiki is a free open source wiki program that allows users to create their own personal wiki sites. Originally built for Wikpedia, it is now used by thousands of other projects due to its scalability and high customization.


Before working with MediaWiki, you need to have LAMP installed on your server. If you don’t have the Linux, Apache, MySQL, PHP stack on your server, you can find the tutorial for setting it up here: <a href=“https://www.digitalocean.com/community/articles/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-6”>CentOS LAMP Installation Tutorial</a>. <br/><br/>


# Install MediaWiki on your VPS



At the time of writing, the latest version of MediaWiki is MediaWiki 1.21.2. To check for the latest version of the platform, visit their website and simply alter the code to match the most updated version (if desirable).


<ul>
<li>Go to MediaWiki’s official website and download the latest version of the platform:</li>
<pre>wget http://download.wikimedia.org/mediawiki/1.21/mediawiki-1.21.2.tar.gz</pre>
<li>After the download completes untar the package:</li>
<pre>tar xvzf mediawiki-*.tar.gz</pre>
<li>The default directory for the downloaded contents includes the specified version of the platform-- it may be best to move the contents to a more convenient location:</li>
<pre>sudo mv mediawiki-1.21.2 /etc/mediawiki</pre>
<li>Create a symbolic link between the MediaWiki directory and Apache’s document root:</li>
<pre>sudo ln -s /etc/mediawiki/ /var/www/html</pre>
<li>Restart Apache</li>
<pre>sudo service apache2 restart</pre>
</ul>


# Create a MySQL Database and User



Creating a MySQL database is simple and increases security, as it eliminates the need for sharing the MySQL root information. Follow these steps:


<ul>
<li>Log into MySQL</li>


<pre>mysql -u root -p</pre>


<li>Create a Dedicated Database</li><br/>
You can name your DB whatever you’d like-- here it will be example_wiki.


<pre>create database example_wiki;</pre>


<li>Grant Permissions</li><br/>
Next you will provide a user for the new database with the permissions that MediaWiki requires. Replace “wikiuser” and “password” with your own specifications.


<pre>grant index, create, select, insert, update, delete, alter, lock tables on my_wiki.* to ‘wikiuser’@‘localhost’ identified by ‘password’;</pre>


<li>Finishing Up</li><br/>
Implement changes and quit MySQL.


<pre>FLUSH PRIVILEGES;
exit;</pre>
</ul>


Visit: [domain]/mediawiki/index.php.


# Set Up MediaWiki



MediaWiki will now be configured through your browser’s on-screen instructions.


When you reach the “MySQL settings” section of the setup page, leave the Database Host as localhost; enter in the MySQL database name, username, and password which you setup in the previous steps.


Press continue until you reach the page that says, “Complete!”


<img src=“https://assets.digitalocean.com/articles/mediawiki_centos/img1.png” />


Once the LocalSettings.php file finishes downloading, upload it to /etc/mediawiki or whichever directory contains MediaWiki’s “index.php” file on your VPS.


You can copy the LocalSettings.php file from your computer to the server with SCP (Secure Copy):


```
scp /path/to/LocalSettings.php [username]@[IP Address]:/etc/mediawiki

```


After the file is uploaded, you will be able to access your personal wiki at [domain]/mediawiki!


<div class=“author”>By Adam LaGreca</div>


