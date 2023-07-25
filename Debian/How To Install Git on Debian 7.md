# How To Install Git on Debian 7

```Git``` ```Debian```

## What the Red  Means


Lucky for you, the majority of code in this tutorial can be copy and pasted! The lines that you will actually need to enter or customize will be red in this tutorial.


## About Git


Git is a version control system distributed under the terms of the GNU General Public License v.2 since its release in 2005. It allows for non-linear development of projects and can handle large amounts of data effectively. This is the case because every working directory in Git is a full-fledged repository with complete history and tracking capabilities that are non-dependent on network access or a central server.


The advantages of using Git stem from the way the program stores data—unlike other VCS’s, it is best to think of the storage process as a set of snapshots of a mini filesystem primarily on your local disk, maximizing efficiency and allowing for powerful tools to be built on top of it.


# Install Git with Apt-Get


Install Git with apt-get in one command!


```
sudo apt-get install git-core
```


The end! Just kidding, you still need to configure Git.


If you wish to download the most recent version of Git from the source, follow the next step. Otherwise, skip down to setup.


# Install Git from the Source


Run "apt-get update" to make sure that you download the most recent packages to your VPS. After that is successful, we are going to download all of the required dependencies (line 1). Finally, only after following the two preceding steps, may you move on to installing the latest version of Git via the  google code page (line 2).


Line 1


```
sudo apt-get install libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev build-essential
```


Line 2


```
wget https://github.com/git/git/archive/v1.9.4.tar.gz
```


After it downloads, untar the file and switch into that directory:


```
tar -zxf v1.9.4.tar.gz
```


```
cd git-1.9.4
```


## Global Install


This is a slightly different and more complex process. But do not worry, weary traveler! If you wish to perform a global install it's a two-step process: 1) Install it once as yourself  and 2) Install it once as root.


```
make prefix=/usr/local all
sudo make prefix=/usr/local install
```


# How to Setup Git


After Git is installed you need to copy your username and email in the gitconfig file. Using the nano command "sudo nano ~/.gitconfig" will open a completely blank page, as you have just done a fresh install. Insert the necessary information with the following commands:


```
git config --global user.name "NewUser"
git config --global user.email newuser@example.com
```


You can see all of your settings with this command:


```
git config --list
```


# See Also


To learn more about using git, visit the link, How To Use Git Effectively.


By Adam LaGreca
