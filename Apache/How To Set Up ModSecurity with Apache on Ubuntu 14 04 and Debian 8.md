# How To Set Up ModSecurity with Apache on Ubuntu 14 04 and Debian 8

```Apache``` ```Security``` ```Ubuntu``` ```Debian```

## Introduction


ModSecurity is a free web application firewall (WAF) that works with Apache, Nginx and IIS. It supports a flexible rule engine to perform simple and complex operations and comes with a Core Rule Set (CRS) which has rules for SQL injection, cross site scripting, Trojans, bad user agents, session hijacking and a lot of other exploits. For Apache, it is loaded as an additional module which makes it easy to install and configure.


# Prerequisites


To follow this tutorial, you will need:


- 
A Ubuntu 14.04 or Debian 8 Droplet.

- 
A standard user account with sudo privileges, which you can set up by following the initial server setup tutorial for Ubuntu 14.04 or Debian 8.

- 
A LAMP stack, which you can install by following the tutorial for Ubuntu 14.04 or Debian 8.


# Step 1 — Installing ModSecurity


In this step, we will install ModSecurity.


First, update the package index files.


```
sudo apt-get update


```


Then, install ModSecurity.


```
sudo apt-get install libapache2-mod-security2 -y


```


You can verify that the ModSecurity module was loaded using the following command.


```
sudo apachectl -M | grep --color security2


```


If the output reads security2_module (shared), this indicates that the module was loaded.


ModSecurity’s installation includes a recommended configuration file which has to be renamed.


```
sudo mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf


```


Finally, reload Apache.


```
sudo service apache2 reload


```


A new log file for ModSecurity will be created in the Apache logs directory at /var/log/apache2/modsec_audit.log.


# Step 2 — Configuring ModSecurity


Out of the box, ModSecurity doesn’t do anything because it needs rules to work. In this step, we will first enable some configuration directives.


To find and replace the configuration directives in this step, we will be using sed, a stream editor. You can read the sed tutorial series to learn more about the tool.


## Basic Directives to Enable


The default ModSecurity configuration file is set to DetectionOnly, which logs requests according to rule matches and does not block anything. This can be changed by editing the modsecurity.conf file and modifying the SecRuleEngine directive. If you are trying this out on a production server, change this directive only after testing all your rules.


```
sudo sed -i "s/SecRuleEngine DetectionOnly/SecRuleEngine On/" /etc/modsecurity/modsecurity.conf


```


The SecResponseBodyAccess directive configures whether response bodies are buffered (i.e. read by ModSecurity). This is only necessary if data leakage detection and protection are required. Therefore, leaving it on will use up Droplet resources and also increase the logfile size, so we will turn it off.


```
sudo sed -i "s/SecResponseBodyAccess On/SecResponseBodyAccess Off/" /etc/modsecurity/modsecurity.conf


```


## Optional Directives to Modify


There are other directives you may want to customize by editing /etc/modsecurity/modsecurity.conf. The SecRequestBodyLimit and SecRequestBodyNoFilesLimit directives limit the maximum data that can be posted to your web application.


In particular, the SecRequestBodyLimit directive specifies the maximum POST data size. If anything larger is sent by a client the server will respond with a 413 Request Entity Too Large error. If your web application does not have any file uploads, this value can be left as it is. The pre-configured value specified in the configuration file is 13107200 bytes (12.5MB). If you want to change this value, look for the following line modsecurity.conf:


Optional `modsecurity.conf` directive change
```
SecRequestBodyLimit 13107200

```


Similarly, the SecRequestBodyNoFilesLimit limits the size of POST data minus file uploads. This value should be set as low as possible to reduce susceptibility to denial of service (DoS) attacks when someone is sending request bodies of very large sizes. The pre-configured value in the configuration file is 131072 bytes (128KB). If you want to change this value, look for the following line modsecurity.conf:


Optional `modsecurity.conf` directive change
```
SecRequestBodyNoFilesLimit 131072

```


A directive that affects server performance is SecRequestBodyInMemoryLimit. This directive is pretty much self-explanatory; it specifies how much of “request body” data (POSTed data) should be kept in the memory (RAM), anything more will be placed in the hard disk (just like swapping). Because Droplets use SSDs, this is not much of an issue. However, this can be changed if you have RAM to spare. The pre-configured value for this directive is 128KB. If you want to change this value, look for the following line modsecurity.conf:


Optional `modsecurity.conf` directive change
```
SecRequestBodyInMemoryLimit 131072

```


# Step 3 — Testing an SQL Injection


Before configuring some rules, we will create a PHP script which is vulnerable to SQL injection to test ModSecurity’s protection.



Note: this is a basic PHP login script with no session handling or form sanitation. It is just used as an example to test the SQL injection and ModSecurity’s rules. It will be removed before the end of the tutorial.

First, access the MySQL prompt.


```
mysql -u root -p


```


Here, create a MySQL database called sample and connect to it.


```
create database sample;
connect sample;


```


Then create a table containing some credentials — the username sammy and the password password.


```
create table users(username VARCHAR(100),password VARCHAR(100));
insert into users values('sammy','password');


```


Finally, exit the MySQL prompt.


```
quit;


```


Next, create the login script in Apache’s document root.


```
sudo nano /var/www/html/login.php


```


Paste the following PHP script into the file. Be sure to change the MySQL password in the script below to the one you set earlier so the script can connect to the database:


/var/www/html/login.php
```

<html>
<body>
<?php
    if(isset($_POST['login']))
    {
        $username = $_POST['username'];
        $password = $_POST['password'];
        $con = mysqli_connect('localhost','root','your_mysql_password','sample');
        $result = mysqli_query($con, "SELECT * FROM `users` WHERE username='$username' AND password='$password'");
        if(mysqli_num_rows($result) == 0)
            echo 'Invalid username or password';
        else
            echo '<h1>Logged in</h1><p>This is text that should only be displayed when logged in with valid credentials.</p>';
    }
    else
    {
?>
        <form action="" method="post">
            Username: <input type="text" name="username"/><br />
            Password: <input type="password" name="password"/><br />
            <input type="submit" name="login" value="Login"/>
        </form>
<?php
    }
?>
</body>
</html>

```


This script will display a login form. Open your browser and navigate to http://your_server_ip/login.php to see it. If you enter the correct pair of credentials, e.g. sammy in the Username field and password in the Password field, you will see the message This is text that only gets displayed when logged in with valid credentials. If you navigate back to the login screen and use incorrect credentials, you will see the message Invalid username or password.


The next job is to try an SQL injection to bypass the login page. Enter the following for the username field.


SQL injection username
```
' or true -- 

```


Note that there should be a space after --  for this injection to work. Leave the password field empty and hit the login button. The script shows the message meant for authenticated users! In the next step, we will prevent this.


# Step 4 — Setting Up Rules


In this step, we will set up some ModSecurity rules.


## Enabling the CRS


To make things easier, there are a lot of rules which are already installed along with ModSecurity. These are called the CRS (Core Rule Set) and are located in the /usr/share/modsecurity-crs directory. To load these rules, we need to configure Apache to read .conf files in these directories, so open the security2.conf file for editing.


```
sudo nano /etc/apache2/mods-enabled/security2.conf

```


Add the two following directives, highlighted in red, inside before the last line in the file (</IfModule>).


Updated security2.conf
```
        IncludeOptional /etc/modsecurity/*.conf
        IncludeOptional "/usr/share/modsecurity-crs/*.conf"
        IncludeOptional "/usr/share/modsecurity-crs/activated_rules/*.conf"
</IfModule>

```


Save and close the file.


## Excluding Directories/Domains (Optional)


Sometimes it makes sense to exclude a particular directory or a domain name if it is running an application, like phpMyAdmin, as ModSecurity will block SQL queries. It is also better to exclude admin backends of CMS applications like WordPress. If you’re following this tutorial on a fresh server, you can skip this step.


To disable ModSecurity for a complete VirtualHost, place the following directives inside the <VirtualHost>[...]</VirtualHost> block in its virtual host file.


```
<IfModule security2_module>
    SecRuleEngine Off
</IfModule>

```


For omitting a particular directory (for example, /var/www/wp-admin):


```
<Directory "/var/www/wp-admin">
    <IfModule security2_module>
        SecRuleEngine Off
    </IfModule>
</Directory>

```


If you don’t want to completely disable ModSecurity in a directory, use the SecRuleRemoveById directive to remove a particular rule or rule chain by specifying its ID.


```
<LocationMatch "/wp-admin/update.php">
    <IfModule security2_module>
        SecRuleRemoveById 981173
    </IfModule>
</LocationMatch>

```


## Activating the SQL Injection Rule


Next, we will activate the SQL injection rule file. The required rule files should be symlinked to activated_rules directory, which is similar to Apache’s mods-enabled directory. Change to the activated_rules directory.


```
cd /usr/share/modsecurity-crs/activated_rules/


```


Then create a symbolic link from the modsecurity_crs_41_sql_injection_attacks.conf file.


```
sudo ln -s ../base_rules/modsecurity_crs_41_sql_injection_attacks.conf .


```


Finally, reload Apache for the rules to take effect.


```
sudo service apache2 reload


```


Now open the login page we created earlier and try using the same SQL injection query on the username field. Because we changed the SecRuleEngine directive to On in step 2,  a 403 Forbidden error is displayed. (If SecRuleEngine was left to the DetectionOnly option, the injection will be successful but the attempt would be logged in the modsec_audit.log file.)


Because this PHP login script is only meant to test ModSecurity, you should remove it now that the test is done.


```
sudo rm /var/www/html/login.php


```


# Step 5 — Writing Your Own Rules


In this section, we’ll create a rule chain which blocks the request if certain words commonly associated with spam are entered in a HTML form.


First, we’ll create an example PHP script which gets the input from a text box and displays it back to the user. Open a file called form.php for editing.


```
sudo nano /var/www/html/form.php


```


Paste the following code:


/var/www/html/form.php
```
<html>
    <body>
        <?php
            if(isset($_POST['data']))
                echo $_POST['data'];
            else
            {
        ?>
                <form method="post" action="">
                        Enter something here:<textarea name="data"></textarea>
                        <input type="submit"/>
                </form>
        <?php
            }
        ?>
    </body>
</html>

```


Custom rules can be added to any of the configuration files or placed in ModSecurity directories. We’ll place our rules in a separate new file called modsecurity_custom_rules.conf.


```
sudo nano /etc/modsecurity/modsecurity_custom_rules.conf

```


Paste the following in this file. The two words we’re blocking are blockedword1 and blockedword2.


modsecurity_custom_rules.conf
```
SecRule REQUEST_FILENAME "form.php" "id:'400001',chain,deny,log,msg:'Spam detected'"
SecRule REQUEST_METHOD "POST" chain
SecRule REQUEST_BODY "@rx (?i:(blockedword1|blockedword2))"

```


The syntax for SecRule is SecRule VARIABLES OPERATOR [ACTIONS]. Here we used the chain action to match variables REQUEST_FILENAME with form.php, REQUEST_METHOD with POST, and REQUEST_BODY with the regular expression (@rx) string (blockedword1|blockedword2). The ?i: does a case insensitive match. On a successful match of all these three rules, the ACTION is to deny and log with the msg "Spam detected." The chain action simulates the logical AND to match all the three rules.


Save the file and reload Apache.


```
sudo service apache2 reload

```


Open http://your_server_ip/form.php in the browser. If enter text containing blockedword1 or blockedword2, you will see a 403 page.


Because this PHP form script is only meant to test ModSecurity, you should remove it now that the test is done.


```
sudo rm /var/www/html/form.php


```


# Conclusion


In this tutorial, you have learned how to install and configure ModSecurity, and add custom rules. To learn more, you can check out the official ModSecurity documentation.


