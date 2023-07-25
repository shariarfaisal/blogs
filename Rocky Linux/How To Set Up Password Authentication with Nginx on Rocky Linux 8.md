# How To Set Up Password Authentication with Nginx on Rocky Linux 8

```Nginx``` ```Rocky Linux``` ```Rocky Linux 8``` ```Security```

## Introduction


When setting up a web server, there are often sections of the site that you wish to restrict access to. Web applications often provide their own authentication and authorization methods, but the web server itself can be used to restrict access if these are inadequate or unavailable. In this guide, you’ll password protect assets on an Nginx web server running on Rocky Linux 8.


# Prerequisites


To get started, you will need:


- Access to an Rocky Linux 8 server environment with a non-root user with sudo privileges in order to perform administrative tasks. To learn how to create such a user, follow the Rocky Linux 8 initial server setup guide.
- Nginx installed on your system, following Steps 1 and 2 of this guide on how to install Nginx on Rocky Linux 8.

# Step 1 — Creating the Password File


To start out, you need to create a file that will hold your username and password combinations. You can do this by using the OpenSSL utilities that should already be available on your server.


If you have OpenSSL installed on your server, you can create a password file with no additional packages. You will create a hidden file called .htpasswd in the /etc/nginx configuration directory to store your username and password combinations.


You can add a username to the file using this command. sammy is used here as the username, but you can use whatever name you’d like:


```
sudo sh -c "echo -n 'sammy:' >> /etc/nginx/.htpasswd"


```


Next, add an encrypted password entry for the username by typing:


```
sudo sh -c "openssl passwd -apr1 >> /etc/nginx/.htpasswd"


```


You can repeat this process for additional usernames. You can see how the usernames and encrypted passwords are stored within the file by typing:


```
cat /etc/nginx/.htpasswd


```


```
Outputsammy:$apr1$wI1/T0nB$jEKuTJHkTOOWkopnXqC1d1

```


# Step 2 — Configuring Nginx Password Authentication


Now that you have a file with your users and passwords in a format that Nginx can read, you need to configure Nginx to check this file before serving your protected content.


The default text editor that comes with Rocky Linux 8 is vi. vi is an extremely powerful text editor, but it can be somewhat obtuse for users who lack experience with it. You can install a more user-friendly editor such as nano to facilitate editing configuration files on your Rocky Linux 8 server:


```
sudo dnf install nano


```


Now you can use nano to edit your Nginx configuration file:


```
sudo nano /etc/nginx/nginx.conf


```


To set up authentication, you need to decide on the context to restrict. Among other choices, Nginx allows you to set restrictions on the server level or inside a specific location.


This example will be for a server level restriction, so you will add options to the main server{ } block within the file. The auth_basic directive turns on authentication and a realm name to be displayed to the user when prompting for credentials. You will use the auth_basic_user_file directive to point Nginx to the password file you created:


/etc/nginx/nginx.conf
```
. . .
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;
     . . .
}
. . .

```



Note: Depending on which block you place the restrictions in, you can control the granularity of which parts of your site require a password. This alternative example restricts only the document root with a location block, and you can even modify this listing to only target a specific directory within the web space:
/etc/nginx/nginx.conf
…
server {
    listen 80 default_server;

     . . .
   
    location / {
    try_files $uri $uri/ =404;
    auth_basic "Restricted Content";
    auth_basic_user_file /etc/nginx/.htpasswd;
    }
}
…


Save and close the file when you are finished. Restart Nginx to implement your password policy:


```
sudo systemctl restart nginx


```


The directory you specified should now be password protected.


# Step 3 — Confirming the Password Authentication


To confirm that your content is protected, try to access your restricted content in a web browser:


```
http://server_domain_or_IP

```


You should be presented with a username and password prompt:





If you enter the correct credentials, you will be allowed to access the content. If you enter the wrong credentials or hit “Cancel”, you will see the “Authorization Required” error page:





# Conclusion


You should now have everything you need to set up basic authentication for your site. Keep in mind that password protection should be combined with TLS encryption so that your credentials are not sent to the server in plain text. Check out this guide on how to secure Nginx with Let’s Encrypt on Rocky Linux 8


