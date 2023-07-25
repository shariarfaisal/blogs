# How To Install Drupal with Nginx on an Ubuntu 13 04 VPS

```Ubuntu``` ```Nginx``` ```Drupal```

<h2>Introduction</h2>


<p>Drupal is a free and open-source content management framework (CMF) written in PHP and distributed under the GNU General Public License. It is used as a back-end system for at least 2.1% of all websites worldwide. As of August 2013, there are more than 22,900 free community-contributed add-ons, known as contributed modules, available to alter and extend Drupal’s core capabilities and add new features or customize Drupal’s behavior and appearance.</p>


<h2>Initial Setup</h2>


<p>In this tutorial, we will be using an Ubuntu 13.04 VPS.  The following instructions require the user to have root privileges on your virtual private server. You can see how to set that up <a href = “https://www.digitalocean.com/community/articles/initial-server-setup-with-ubuntu-12-04”>here</a> (steps 3 and 4).</p>


<p>In order to work with Drupal, you need to have LEMP installed on your VPS. If you don’t have the Linux, Nginx, MySQL, PHP stack on your cloud server, you can find the tutorial for setting it up <a href = “https://www.digitalocean.com/community/articles/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-12-04”>here</a>.</p>


<p>Only once you have the user and required software should you proceed to install Drupal.</p>


<h2>1) Download Drupal</h2>


<p>Download the latest version of Drupal from the Drupal website by using this command.</p>


<pre>
wget http://ftp.drupal.org/files/projects/drupal-7.23.tar.gz
</pre>


<p>Unzip the downloaded Drupal file in your home directory:</p>


<pre>
tar xzvf drupal-7.23.tar.gz
</pre>


<p>Now the Unzipped files will be in the folder drupal-7.23.</p>


<h2>2) Create Drupal Database and User</h2>


<p>Now we want to create a new MySQL database for Drupal. Login into your MySQL shell by using the command:</p>


<pre>
mysql -u root -p
</pre>


<p>Then enter your MySQL root password, which will drop you in the MySQL Shell. Don’t forget to add semicolons to end of MySQL querys.</p>


<p>Now let’s create a database for Drupal by using this query. Here I am naming the Database <b>drupal</b>-- you may give it any name you like.</p>


<pre>
CREATE DATABASE drupal;
Query OK, 1 row affected (0.00 sec)
</pre>


<p>At this point, we need to create the new user. You can use any name:</p>


<pre>
CREATE USER drupaluser@localhost;
Query OK, 0 rows affected (0.02 sec)
</pre>


<p>Set the password for your new user:</p>


<pre>
SET PASSWORD FOR drupaluser@localhost= PASSWORD(“<span class=“highlight”>password</span>”);
Query OK, 0 rows affected (0.00 sec)
</pre>


<p>Now we want to grant all permissions to the created drupal user. Without this we cannot proceed:</p>


<pre>
GRANT ALL PRIVILEGES ON drupal.* TO drupaluser@localhost IDENTIFIED BY ‘<span class=“highlight”>password</span>’;
Query OK, 0 rows affected (0.00 sec)
</pre>


<p>Refresh MySQL:</p>


<pre>
FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)
</pre>


<p>Finally, exit from the MySQL Shell:</p>


<pre>
exit
</pre>


<h2>3) Copying the files</h2>


<p>The default server directory in Ubuntu 13.04 is /usr/share/nginx/html/.</p>


<p>Create a new directory <b>drupal</b> in “/usr/share/nginx/html/”:</p>


<pre>
sudo mkdir /usr/share/nginx/html/drupal
</pre>


<p>Copy the drupal files to your server directory from home:</p>


<pre>
cd ~
sudo mv drupal-7.23/* /usr/share/nginx/html/drupal/
</pre>


<h2>4) Configuring Drupal</h2>


<p>Copy the default configuration as settings.php:</p>


<pre>
sudo cp /usr/share/nginx/html/drupal/sites/default/default.settings.php /usr/share/nginx/html/drupal/sites/default/settings.php
</pre>


<p>Now make the settings.php file writable by changing the permissions:</p>


<pre>
sudo chmod a+w /usr/share/nginx/html/drupal/sites/default/settings.php
</pre>


<p>Change permissions for the settings directory:</p>


<pre>
sudo chmod a+w /usr/share/nginx/html/drupal/sites/default
</pre>


<p>We need a specific php module to proceed with Drupal installation. Download and install by using this command:</p>


<pre>
sudo apt-get install php5-gd
</pre>


<p>After it is installed, you will need to restart the php5-fpm service:</p>


<pre>
sudo service php5-fpm restart
</pre>


<h2>5) Configuring Nginx</h2>


<p>We need to setup Drupal virtual host for nginx. Copy the default host for Drupal:</p>


<pre>
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/drupal
</pre>


<p>Open the nginx virtual host for Drupal.</p>


<pre>
sudo nano /etc/nginx/sites-available/drupal
</pre>


<p>The configuration should include the changes as below.</p>


<pre>
server {
listen   80;
root /usr/share/nginx/html/drupal;
index index.php index.html index.htm;
server_name 162.243.9.129;
location / {
try_files $uri $uri/ /index.php?q=$uri&$args;
}
error_page 404 /404.html;
error_page 500 502 503 504 /50x.html;
location = /50x.html {
root /usr/share/nginx/html/drupal;
}
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9$
location ~ .php$ {
#fastcgi_pass 127.0.0.1:9000;
# With php5-fpm:
fastcgi_pass unix:/var/run/php5-fpm.sock;
fastcgi_index index.php;
include fastcgi_params;


```
             }

```


</pre>


<p>Here are the Changes:</p>


<ol>
<li>Change the root to /usr/share/nginx/html/drupal.</li>
<li>Change the server_name from localhost to your domain name or IP address.</li>
<li>Change the “try_files $uri $uri/ /index.html;” line to “try_files $uri $uri/ /index.php?q=$uri&$args;” in order to enable Drupal Permalinks with nginx.</li>
</ol>


<h2>Step Six - Activate the Configuration</h2>


<p>Next enable the Drupal configuration:</p>


<pre>
sudo ln -s /etc/nginx/sites-available/drupal /etc/nginx/sites-enabled/drupal
</pre>


<p>And remove the default configuration:</p>


<pre>
sudo rm /etc/nginx/sites-enabled/default
</pre>


<p>Restart nginx:</p>


<pre>
sudo service nginx restart
</pre>


<h2>7) Installation</h2>


<p>Now open the IP address or domain in your browser followed by “/drupal” and continue the installation.</p>


<div class=“author”>Submitted by: <a href=“http://www.learn2crack.com”>Raj Amal</a></div>


