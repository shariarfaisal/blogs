# Install Chrome on Linux Mint - Easy Step-By-Step Guide

```UNIX/Linux```

In this tutorial, we will see how to install Chrome on Linux Mint. Google Chrome is a popular web browser that is suitable for surfing amazing websites like this one. We will also cover a better alternative to Chrome that is easier to install.


# Steps to Install Google Chrome on Linux Mint


Let’s go over the steps to install Google Chrome, which is Google’s version of the original open-source Chromium browser. Since Google Chrome isn’t natively available in the package repositories, we need to add their Linux repos and install the package from there.


## 1. Downloading the Key for Chrome


Before we proceed, install Google’s Linux package signing Key. This key will automatically configure the repository settings necessary to keep your Google Linux applications up-to-date.


```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -

```


Google Chrome Key
## 2. Adding Chrome Repo


For installing Chrome you need to add Chrome repository to your system source. You can do this with the command:


```
sudo add-apt-repository "deb http://dl.google.com/linux/chrome/deb/ stable main"

```


You may also add the repository manually by editing your /etc/apt/sources.list file.


## 3. Run an Apt Update


After you add the Chrome repository in the last step you need to do an apt-update. The command for doing that is:


```
sudo apt update

```


Add Chrome Repo and Update 
## 4. Install Chrome on Linux Mint


After going through the commands above you are ready to finally install Chrome. The command for doing that is :


```
sudo apt install google-chrome-stable

```


Install Google Chrome Stable
This command installs the stable version of Chrome.


While installing you will be prompted to grant permission to proceed with the installation. Press ‘y’ to continue.


That is it! Now you can run Google Chrome by typing in :


```
google-chrome 

```


Or you can use the GUI to go to your applications and find Google Chrome there.


## 5. Uninstalling Chrome


To uninstall Google Chrome use the command :


```
sudo apt remove google-chrome-stable

```


This will successfully remove Google Chrome from your system.


# A better Alternative: Chromium





Chromium is an open-source version of Chrome. It is present by default in the Linux repositories. So you won’t need to add it explicitly.


To install Chromium you just need to run the command :


```
sudo apt install chromium-browser

```


One command and Chromium is ready to go!


# That’s it, folks!


In this tutorial, we saw how Chrome can be installed on Linux systems. Apart from that we also saw Chromium, a better open-source version of Chrome that is readily available on Linux.


