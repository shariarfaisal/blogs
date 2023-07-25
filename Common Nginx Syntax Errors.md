# Common Nginx Syntax Errors

```Nginx```

## Introduction


Nginx is a popular web server that can host many large and high-traffic sites on the internet. When setting up an Nginx web server, you typically do so by creating a domain block with configuration details to ensure your site can handle incoming requests. One of the common errors when setting up Nginx involves the syntax in the configuration file, and ensuring it is valid.


The examples in this guide were tested on an Ubuntu 22.04 server, but should be applicable to most Nginx installations, given that it deals mostly within a standardized configuration file. Directories and paths may differ slightly.


In this tutorial, you will learn about syntax errors that may occur in your Nginx configuration file, how to check for them, and how to correct them.


# Inspecting Your Nginx Error Log


The errors and solutions presented in this tutorial are common cases, but not exhaustive. Due to the nature of syntax errors breaking the structure of what Nginx recognizes as valid statements, the following errors are only guidelines for how Nginx responds. Oftentimes, one error can cascade into another, and an error can be symptomatic of a larger or separate issue. Your specific circumstance and setup may vary.


Please note that you can always refer to the Nginx error log to view a running list:


```
sudo cat /var/log/nginx/error.log


```


This tutorial will cover how to break down and understand the parts of an Nginx error message later on.


# Testing Your Configuration File for Errors


For the purposes of this tutorial, we will be referring to an Nginx domain block example with various errors present in the configuration file. These errors are intentional to demonstrate how you can correct them. In general, to verify whether or not you have any syntax errors, you can run the following command:


```
sudo nginx -t


```


If there are no errors, your output will return the following message:


```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

```


If there are errors, you will receive a message that identifies the exact file and line of code where the error is occurring, and the specific syntax issue you need to resolve.


# Identifying Structural Syntax Errors — Semicolons, Curly Braces, and Parameters in an Nginx Configuration File


One of the common errors you may encounter with Nginx has to do with missing characters or incorrect syntax structure. Nginx’s configuration file is focused around directives, and these directives must be declared in a specific way. Otherwise, your configuration file will not be structurally valid.


Directives can be broken into two distinct types, with specific syntax for each. There can be simple directives that contain the directive’s name and parameters, ending with a semicolon. There can also be block directives which are similar to a single directive, but they are demarcated by curly braces { } and can even contain other blocks within it.


When a directive is not structured appropriately, Nginx will fail to recognize it as the directive you intended, and the following errors can result.


## Nginx’s invalid parameter Error


An Nginx configuration file is very particular when it comes to proper structure and syntax. In fact, one issue that can occur with your syntax has to do with the semi-colon ;. For example, consider the following error message:


```
[emerg] invalid parameter "root" in /etc/nginx/sites-enabled/example.com:5
nginx: configuration file /etc/nginx/nginx.conf test failed

```


Before resolving the issue in this example, you should understand the components of Nginx’s error messages. There are certain hints in the error message that help to identify the cause of the error. In this case, [emerg], stands for “emergency”, and represents an error message that has to do with your system being unstable. In this context, this means Nginx has run into an issue that prevents it from working.


This error message further identifies the location this error can be found. Keep in mind that in Nginx setups, you will have a custom configuration file that is symlinked to /etc/nginx/nginx.conf. For example, if you followed our guide on how to install Nginx on Ubuntu 22.04, Step 5 specifically demonstrates this. The exact line in the configuration file with the error is given: /etc/nginx/sites-enabled/example.com:5. You also learn that there is an invalid parameter "root" in that file.


Now that you have the information on where to find your error, you can go ahead and open up the file with your preferred text editor. Here, we’ll use nano:



Note: If you’re ever uncertain, you can typically find all configuration files within the /etc/nginx/ directory. Read more in our tutorial about other important Nginx files and directories to know.

```
sudo nano /etc/nginx/sites-available/example.com


```


Once inside, locate line 5 that the error message referred to. The line is highlighted in the following:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;

       root /var/www/example.com/html
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Now, the error happening here may not be apparent. If you recall from the error message, there is an invalid parameter. As a reminder, parameters are arguments that you provide to Nginx’s directives. While /var/www/example.com/html is the parameter provided in this scenario, it is invalid. There is a missing character from the syntax structure, in this case, a missing semicolon at the end of the line.


You may notice that many of the other lines in this file also end with a semicolon. A semicolon is necessary any time a line contains a directive. This particular example is the root directive that specifies the root directory that will be used when searching for a file. Having this root directive is necessary for Nginx to be able to find a specific URL path. We will discuss the importance of directives in a later section.


In short, you can fix this error by adding a semicolon to the end of this line to ensure the directive is valid. Your update will be as follows:


/etc/nginx/sites-available/example.com
```
…

       root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

…


```


When you’re finished updating, save and close the file. If you’re using nano you can do so by pressing CTRL + X, then Y, and ENTER.


You can verify your syntax error is fixed by running the sudo nginx -t command.


## Nginx’s unexpected "}" Error


Another common error that may occur with Nginx syntax structure is with curly braces { }. Unlike the previous error that did not specify how to correct the invalid parameter, this error explicitly provides a reason for the error:


```
nginx: [emerg] unexpected "}" in /etc/nginx/sites-enabled/example.com:15
nginx: configuration file /etc/nginx/nginx.conf test failed

```


This error message is another [emerg] type and indicates at line 15 that there is an unexpected curly brace in the same configuration file as before. Using your preferred text editor to open this file, the contents will be the following:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;

       root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
       }

```


Highlighted at the end of this file is the curly brace }. At first glance, it may not be clear what the issue is since the unexpected "}" seems to be present. Upon closer inspection, there is actually a missing curly brace after that one. The proper amount of curly braces is important in an Nginx configuration file because it indicates the opening and closing of a specific block. If you focus in, you will recognize that the curly brace at the end of the file is actually the closing curly brace for the following nested location block:


/etc/nginx/sites-available/example.com
```
…

 location / {
                try_files $uri $uri/ =404;
       }

…

```


If you review the contents of the configuration file even further, it is the server block that is missing its closing curly brace. An Nginx server block is important because it provides the configuration details that Nginx needs to recognize which virtual server will handle the various requests it receives. It’s important to note that location blocks are nested within a server block since a server block takes precedence in handling incoming requests. With this all in mind, you can add the closing curly brace at the end of the file to complete the server block. The contents of this file will now be as follows:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Remember to save and close the file when you’re done. Confirm that this syntax error has been resolved by running sudo nginx -t.


## Nginx’s invalid host Error


This next error differs from the previous two because the error originates in a malformed parameter given to a directive. While ending lines with semicolon ; and closing every curly brace { with a } deals with the overall structure of a configuration file, parameters are custom inputs that can vary depending on your setup’s needs.


It should be noted that host is the effected directive in this scenario, but any directive can be subject to this error, given that invalid parameters are provided. The error message would change accordingly. Providing a valid parameter ensures you avoid errors like the following scenario:


```
[emerg] invalid host in "[::]80" of the "listen" directive in /etc/nginx/sites-enabled/example.com:3
nginx: configuration file /etc/nginx/nginx.conf test failed

```


The emergency error in this example explains that the port 80 directive set up for your host is invalid. This error message further identifies the location this error can be found and the exact line in the configuration file:  /etc/nginx/sites-enabled/example.com:3. Now that you have the information on where to find your error, you can go ahead and open up the file with your preferred text editor. Once inside, locate line 3 that the error message referred to. The line is highlighted in the following:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


The two colons in the brackets represent the IPv6 notation, or 0.0.0.0, and without the additional colon after the bracket, it is unable to bind to port 80. As a result, the listen directive will not work since without the colon it’s unclear which port you want your server to listen to.


In short, the specific syntax error happening here is that there is a missing colon after the bracket [::], making the parameter itself invalid. Once you add the missing colon, the code snippet in your file will read as follows:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;
…


```


After updating this line, make sure to save and close the file, then verify that this syntax error has been corrected by running sudo nginx -t.


Overall, when receiving these syntax errors related to parameters, semicolons ;, or curly braces { }, it’s recommended you pay close attention to the details of the exact location and details provided in the [emerg] message.


# Identifying Directive Misuse — Keywords in an Nginx Configuration File


Aside from the syntax errors that may arise from a missing colon or curly brace, there are possibilities for errors such as misspelled keywords associated with a directive in your configuration file. We briefly mentioned directives in the previous section, but let’s revisit them here.


## Nginx’s unknown directive Error


As mentioned previously, the foundation of an Nginx configuration file is built on directives. Nginx has a plethora of directives to choose from, but there are some main ones needed in your configuration file. However, there are errors that can occur with the directive itself, specifically if the keyword is not written exactly as it should be. Here’s an example of the error message you might receive:


```
nginx: [emerg] unknown directive "serve_name" in /etc/nginx/sites-enabled/example.com:8
nginx: configuration file /etc/nginx/nginx.conf test failed

```


This error identifies an unknown directive serve_name within your configuration file on line 8. When you open up your configuration file with your preferred text editor your contents will list the following:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        serve_name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


In this example, the error that’s being triggered is a result of the misspelling of “server” for the server_name directive. In this case, there’s a missing “r” and for that reason, this single directive cannot be recognized. This is a significant error because the server_name directive provides the specific server names that the server block will refer to when given the request. Without this directive correctly functioning, the request cannot be fulfilled. This may be a seemingly small typo error, but it disrupts the syntax, and causes an error. You can update this code snippet in your configuration file to the following:


/etc/nginx/sites-available/example.com
```
…

        server_name example.com www.example.com;

…


```


After you save and close this file, you can verify it’s working with the sudo nginx -t command.


## Nginx’s directive is not allowed here Error


Now, let’s say you had an error with the same directive, but this time the message reads as the following:


```
nginx: [emerg] "server" directive is not allowed here in /etc/nginx/sites-enabled/example.com:8
nginx: configuration file /etc/nginx/nginx.conf test failed

```


You may notice that even though this error is happening in the same location as the previous one, the cause of the issue is detailed in this message. Therefore, it’s important to understand what the error message is indicating. As opposed to the previous error message regarding the “unknown directive”, this error states that the "server" directive is not allowed. You can open up your configuration file with your preferred text editor to inspect its contents:


/etc/nginx/sites-available/example.com
```
server {
        listen 80;
        listen [::]:80;

        root /var/www/example.com/html;
        index index.html index.htm index.nginx-debian.html;

        server name example.com www.example.com;

        location / {
                try_files $uri $uri/ =404;
        }
}

```


Here the word “server” is spelled correctly, but the directive itself is not the exact wording for the server_name directive with the missing underscore demarcation. Therefore, server is perceived as a duplicate, which is not allowed as stated by the error message. This is related to the distinction between the single and block directives we discussed earlier.


The error being triggered here is because the single directive server name is only reading server and thus clashing with the first server block directive at the beginning of the file. This is a tricky syntax error that can occur among block directives and nested single directives because of Nginx’s hierarchical structure in a configuration file. You can correct this error by adding the underscore _ to correctly represent the exact directive keyword for server_name:


/etc/nginx/sites-available/example.com
```
…

        server_name example.com www.example.com;

…


```


Once you’ve made this correction, save and close the file. Then check that the syntax is valid with sudo nginx -t. Unless there are other errors present in your configuration file, this should return a syntax is ok message.



Note: When it comes to Nginx configuration files, there is some flexibility with the amount of spacing or separation of lines that won’t trigger any errors. However, anything explicitly related to a directive will return an error since the exact wording or syntax structure is necessary to function properly.

A final reminder to keep in mind is that if your configuration file contains multiple errors, the error messages you receive will only return one at a time in consecutive order. This means that if you receive an error message about one issue, correct it, and then run the syntax check command again, the next error will return in your output. This will continue to happen until all errors are corrected.


You’ve now learned about other syntax errors that may occur with specific directives in your configuration file. It’s important to note that all of these examples are represented in a terminal environment. If you prefer, there are options to use a code editor such as Visual Studio Code that can check for and highlight the errors in your code without triggering multiple error messages, allowing you to correct them all at once, or avoid them altogether early on.


# Conclusion


In this tutorial you learned about Nginx’s common syntax errors and how to correct them. Even though there are many Nginx syntax errors you may encounter, these examples provided a few possibilities and solutions for the most common ones. If you’re interested in learning more about the Nginx configuration file you can check out our tutorial. You can also check out other content on Nginx with our tagged Community page, or jump into learning how to install Nginx on Ubuntu 22.04.


