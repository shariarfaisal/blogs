# How To Install an Upstream Version of  Node js on Ubuntu 12 04

```Ubuntu``` ```Node.js```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## About Node.js


Node.js. is a system that uses event-driven (as opposed to thread-based) programming to build scalable applications and network programs. It is especially helpful for building web servers.  Written in Javascript, node.js was created in 2009 and is currently the second most popular repository on github.


# Setup


Feel free to skip this section if you have a compiler and curl installed on your droplet. These steps are included since both a compiler and curl are required for the node.js installation itself. Additionally, you will need to have sudo privileges on your virtual private server for the next three commands (the actual installation does not require them).


Go ahead and run an apt-get update before starting to install any of the requirements.


```
sudo apt-get update
```


Once the update finishes, install a compiler on your VPS.


```
sudo apt-get install build-essential
```


Additionally, be sure to download curl, which we will need to perform the installation itself.


```
sudo apt-get install curl
```


Once those two components have downloaded, you are all ready to install node.js.


# Install node.js and NPM (Node Package Manager)


I found the method described below to be the easiest way to install node.js. According to their site, there are eight ways to install node.js, and you can check out the other possibilities if you prefer.


This specific installation is very straightforward and installs node.js on the user's local system alone.


Start by changing the path to include commands from the ~/local/bin directory.


```
 echo 'export PATH=$HOME/local/bin:$PATH' >> ~/.bashrc
```


Follow up by sourcing the .bashrc file:


```
. ~/.bashrc
```


Create two new directories for the installation:


```
mkdir ~/local
mkdir ~/node-latest-install
```


Switch into the latest-install folder:


```
cd ~/node-latest-install
```


Run curl to get the node.js tarball and subsequently untar it.


```
curl http://nodejs.org/dist/node-latest.tar.gz | tar xz --strip-components=1
```


Once that has completed, you can go ahead and start the installation process, limiting it to the local user. This ensures that you will not need sudo later.


```
./configure --prefix=~/local
```


Run make install, but be warned: it does take a while.


```
make install
```


Finish up by downloading the the node package manager through curl:


```
curl -L https://npmjs.org/install.sh | sh
```


After you are all done, you can quickly check which version you have installed on your virtual private server.


```
node -v
```


By Etel Sverdlov
