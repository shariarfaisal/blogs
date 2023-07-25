# How To Set Up Basic HTTP Authentication With Nginx on CentOS 7

```Security``` ```Nginx``` ```CentOS```

## Introduction


Nginx is one of the leading web servers in active use. It and its commercial edition, Nginx Plus, are developed by Nginx, Inc.


In this tutorial, you’ll learn how to restrict access to an Nginx-powered website using the HTTP basic authentication method on Ubuntu 14.04. HTTP basic authentication is a simple username and (hashed) password authentication method.


# Prerequisites


To complete this tutorial, you’ll need the following:


- 
One CentOS 7 Droplet with a sudo non-root user, which you can set up by following this initial server setup tutorial.

- 
Nginx installed and configured on your server, which you can do by following this Nginx installation tutorial.


# Step 1 — Installing HTTPD Tools


You’ll need the htpassword command to configure the password that will restrict access to the target website. This command is part of the httpd-tools package, so the first step is to install that package.


```
sudo yum install -y httpd-tools


```


# Step 2 — Setting Up HTTP Basic Authentication Credentials


In this step, you’ll create a password for the user running the website.


That password and the associated username will be stored in a file that you specify. The password will be encrypted and the name of the file can be anything you like. Here, we use the file /etc/nginx/.htpasswd and the username nginx.


To create the password, run the following command.


```
sudo htpasswd -c /etc/nginx/.htpasswd nginx


```


You can check the contents of the newly-created file to see the username and hashed password.


```
cat /etc/nginx/.htpasswd


```


Example /etc/nginx/.htpasswd
```
nginx:$apr1$ilgq7ZEO$OarDX15gjKAxuxzv0JTrO/

```


# Step 3 — Updating the Nginx Configuration


Now that you’ve created the HTTP basic authentication credential, the next step is to update the Nginx configuration for the target website to use it.


HTTP basic authentication is made possible by the auth_basic and auth_basic_user_file directives. The value of auth_basic is any string, and will be displayed at the authentication prompt; the value of auth_basic_user_file is the path to the password file that was created in Step 2.


Both directives should be in the configuration file of the target website, which is normally located in the /etc/nginx/ directory. Open that file using nano or your favorite text editor.


```
sudo nano /etc/nginx/nginx.conf


```


Under the server section, add both directives:


/etc/nginx/nginx.conf
```
. . .
server {
	listen       80 default_server;
	listen       [::]:80 default_server;
	server_name  _;
	root         /usr/share/nginx/html;

	auth_basic "Private Property";
	auth_basic_user_file /etc/nginx/.htpasswd;
. . .

```


Save and close the file.


# Step 4 — Testing the Setup


To apply the changes, first reload Nginx.


```
sudo systemctl reload nginx


```


Now try accessing the website you just secured by going to http://your_server_ip/ in your favorite browser. You should be presented with an authentication window (which says “Private Property”, the string we set for auth_basic), and you will not be able to access the website until you enter the correct credentials. If you enter the username and password you set, you’ll see the default Nginx home page.


# Conclusion


You’ve just completed basic access restriction for an Nginx website. More information about this technique and other means of access restriction are available in Nginx’s documentation.


