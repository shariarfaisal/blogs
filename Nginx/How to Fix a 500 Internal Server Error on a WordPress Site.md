# How to Fix a 500 Internal Server Error on a WordPress Site

```WordPress``` ```Nginx```

## Introduction


The 500 Internal Server Error code can be an ambiguous one when maintaining a WordPress installation, and issues in PHP or the web server) could likely be the culprit. If you are receiving a 500 error on your WordPress installation, this tutorial will share solutions to help you identify, solve, and verify that the changes you made were successful in getting your WordPress site running smoothly again.


# Step 1 — Identifying and Replicating the Issue


An Internal Server Error HTTP code indicates that the server is having an issue, but cannot be specific about what sort of issue it’s having. Using this knowledge about the 500 Internal Server Error code, let’s take a look at the error message:


```
`HTTP Error 500 NGINX`

```


To solve this problem, the first step is to replicate and monitor the error. If you recently enabled, changed settings, or upgraded a plugin, there is a chance the plugin is the culprit of your issues.


## Deactivating WordPress Plugins


You may want to start your audit by disabling your plugins one by one and seeing if this changes anything.


To deactivate your plugins temporarily, navigate to your WordPress dashboard and select Plugins. In your list of plugins, locate the Deactivate button and select it to start the process of disabling your plugin. Repeat this process for each plugin you have activated.





## Auditing Web Server Logs


As mentioned before, the 500 Internal Server Error on WordPress sites can happen for a wide variety of reasons, all related to the back end server. Auditing your web server logs can be a helpful practice to identify the issue or what may have caused it in the first place.


To audit your server log, enter the following in the command line:


```
tail -f /var/log/nginx/error.log


```


After entering, reload your current WordPress page to see if more information on the error is shown.


If you still can’t identify the specific code that is triggering this error, the issue might come from an incompatible or damaged installation of either WordPress or PHP on the server. In the next step, you’ll see how to upgrade WordPress and PHP to make sure this is not what’s causing your error.


# Step 2 — Updating Your Installation


To make sure the 500 Internal Server Error encountered on your WordPress installation doesn’t come from a damaged or incompatible installation of either WordPress or PHP, you’ll need to check your currently installed versions and update them accordingly. Keeping your web server and your WordPress installation up to date is a good security practice and should be incorporated as a regular maintenance task.


## Updating WordPress


When you’re experiencing a 500 Internal Server Error, you may have limited access to your site, to update WordPress automatically. If the error is not preventing you from accessing your WordPress admin panel, log in to your /wp-admin dashboard. Because WordPress automatically sends notifications on new updates available, there may be a notification at the top of your dashboard:





If there is no notification, you can update your WordPress installation by visiting the Updates section, and selecting Update when prompted to update your WordPress site.


After the update, move to Step 3 to test for the 500 error. If you are still experiencing the error, return to this step to update your version of PHP.


If you aren’t able to log into your dashboard because of the 500 error, you’ll need to perform a manual WordPress update via the command line.


## Updating PHP


To update your version of PHP on your WordPress installation, you’ll need to check your hosting provider’s steps to accessing and updating the PHP version on your installation. Some providers allow for updates via cpanel, while others require updates on their platform. Consult with your hosting provider’s documentation to learn more about how to update the PHP on your WordPress installation.


You can also manually update your installation - learn more about this process and why updating PHP for WordPress sites is important on WordPress’ official documentation.


After you’ve successfully updated your WordPress installation and/or version of PHP, it’s time to move to Step 3 to test for errors.


# Step 3 — Testing for Errors


To test for errors after updating your WordPress installation and/or PHP version, try accessing your domain.


If you encounter the 500 error again and have successfully updated your version of PHP as well as your WordPress installation, you’ll need to check with your hosting provider to dive deeper into issues with your server that may exist beyond your site.


If you’ve successfully resolved the 500 error, you’ll have also updated your installation to ward against commonly experienced bugs and security vulnerabilities. It’s a good practice to keep both your WordPress installation and PHP versions updated for this reason, and can prevent 500 errors from occurring in the future.


## Conclusion


In this tutorial, we successfully performed troubleshooting a 500 error on a WordPress installation, commonly experienced when either the WordPress installation or PHP version is damaged or outdated.


For more information on error codes and how to solve them, visit our tutorial, “How to Troubleshoot Common HTTP Codes”.


