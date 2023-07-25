# How To Install Composer on Debian 11 [Quickstart]

```Debian``` ```Debian 11``` ```PHP``` ```Quickstart```

## Introduction


In this quickstart guide, you’ll install Composer on a Debian 11 server.


For a more detailed version of this tutorial, with more explanations of each step, please refer to How To Install and Use Composer on Debian 11.


# Prerequisites


To follow this guide, you will need:


- One Debian 11 server with a sudo non-root user. To set this up, you can follow our Initial Server Setup with Debian 11 tutorial.

# Step 1 — Installing the Dependencies


In addition to dependencies that may already be included within your Debian 11 system, Composer requires php-cli to execute PHP scripts in the command line, and unzip to extract zipped archives.


Begin by updating the package manager cache:


```
sudo apt update


```


Next, install the dependencies. You will need curl to download Composer and php-cli for installing and running it. The php-mbstring package is necessary to provide functions for a library you’ll be using in this tutorial. git is used by Composer for downloading project dependencies, and unzip is for extracting zipped packages. Everything can be installed with the following command:


```
sudo apt install curl php-cli php-mbstring git unzip


```


With all the dependencies installed, now you can install Composer.


# Step 2 — Download and Install Composer


Make sure you’re in your home directory, then retrieve the Composer installer using curl:


```
cd ~
curl -sS https://getcomposer.org/installer -o /tmp/composer-setup.php


```


Next, you’ll verify that the downloaded installer matches the SHA-384 hash for the latest installer found on the Composer Public Keys / Signatures page.


Using curl, fetch the latest signature and store it in a shell variable:


```
HASH=`curl -sS https://composer.github.io/installer.sig`


```


Now execute the following PHP code to verify that the installation script is safe to run:


```
php -r "if (hash_file('SHA384', '/tmp/composer-setup.php') === '$HASH') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"


```


You’ll receive the following output:


Output
```
Installer verified

```



Note: If the output says Installer corrupt, you should repeat the download and verification process until you have a verified installer. This is why checksums are so useful. If there are any changes in your copy of the file, you can quickly tell by comparing the checksum to the original.

The following command will download and install Composer as a [system-wide command](You can learn more about how adding Composer to your PATH works) named composer, under /usr/local/bin:


```
sudo php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer


```


You’ll see output similar to this:


```
OutputAll settings correct for using Composer
Downloading...

Composer (version 2.3.5) successfully installed to: /usr/local/bin/composer
Use it: php /usr/local/bin/composer

```


To test your installation, run:


```
composer


```


```
Output   ______
  / ____/___  ____ ___  ____  ____  ________  _____
 / /   / __ \/ __ `__ \/ __ \/ __ \/ ___/ _ \/ ___/
/ /___/ /_/ / / / / / / /_/ / /_/ (__  )  __/ /
\____/\____/_/ /_/ /_/ .___/\____/____/\___/_/
                    /_/
Composer version 2.3.5 2022-04-13 16:43:00

Usage:
  command [options] [arguments]

Options:
  -h, --help                     Display help for the given command. When no command is given display help for the list command
  -q, --quiet                    Do not output any message
  -V, --version                  Display this application version
      --ansi|--no-ansi           Force (or disable --no-ansi) ANSI output
  -n, --no-interaction           Do not ask any interactive question
      --profile                  Display timing and memory usage information
      --no-plugins               Whether to disable plugins.
      --no-scripts               Skips the execution of all scripts defined in composer.json file.
  -d, --working-dir=WORKING-DIR  If specified, use the given directory as working directory.
      --no-cache                 Prevent use of the cache
  -v|vv|vvv, --verbose           Increase the verbosity of messages: 1 for normal output, 2 for more verbose output and 3 for debug
...

```


This verifies that Composer was successfully installed on your system and is available system-wide.


# Conclusion


Here are links to more detailed guides related to this tutorial:


In this tutorial, you were able to quickly install Composer on your Debian 11 server. You can find a more detailed explanation of this process in our How To Install and Use Composer on Debian 11 tutorial.


