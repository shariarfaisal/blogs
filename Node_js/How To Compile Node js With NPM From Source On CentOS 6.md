# How To Compile Node js With NPM From Source On CentOS 6

```Node.js``` ```CentOS```










# Status: Deprecated


This article covers a version of CentOS that is no longer supported. If you are currently operating a server running CentOS 6, we highly recommend upgrading or migrating to a supported version of CentOS.


Reason:
CentOS 6 reached end of life (EOL) on November 30th, 2020 and no longer receives security patches or updates. For this reason, this guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other CentOS releases. If available, we strongly recommend using a guide written for the version of CentOS you are using.


## Introduction


(At the time of this writing, the current stable release is v0.10.15 for Node.js and 1.3.5 for NPM)


Now compiling Node.js from source can seem a little daunting. It may even stir up feelings of fear, after all, it's just not as normal as it used to be. But fear not, it's much easier than you think and it will take about 15 minutes from start to finish.


One good reason to compile it yourself is that you can get the most up-to-date or even a beta release version. CentOS is not known for having the latest release versions of packages, so this may have to be something you know how to do. However, there is a big pitfall to this approach; there isn't a package manager to alert you when there is an update. A second, but minor, pitfall is that when there is an update you have uninstall the old version then compile the new source the same way you would in this tutorial. Keep that in mind, and it may be prudent to sign up for their email newsletter to get alerts of when a new version is released.


This is a beginner level tutorial, it assumes the following:


- You have a fresh CentOS 6 (x64) VPS (from DigitalOcean)
- You have root level privileges

If you use 'sudo', you should know to prepend it to any commands specified.


# Prerequisites


We need a few things installed that don't come with the barebones install DigitalOcean provides. So let's install them to make sure we can compile source code. It will also list about two dozen dependency packages, just accept them and let the system install them all.


```
yum install gcc gcc-c++ automake autoconf libtoolize make
```


# Get the Source Code


We need to download the source code from Node's website, and they make it super simple. There's a big green button on the home page that will force a download. Now, we don't need to download it to our local computer, we need it on the VPS. On your VPS, change to the directory "opt". Why change to that particular directory? Well, in the old days, "/opt" was used by UNIX vendors like AT&T, Sun, DEC and 3rd-party vendors to hold "Option" packages; i.e. packages that you might have paid extra money for. I found that answer eloquently explained here.


```
cd /opt
```


This directory is empty, so it's a good place to keep your source code that you want to compile. But now we need to get the package on the VPS. We use the tool 'wget' to download it:


```
wget http://nodejs.org/dist/v0.10.15/node-v0.10.15.tar.gz
```


Now that the package is on your VPS, extract it from the tar file.


```
tar zxvf node-v0.10.15.tar.gz
```


The option used are as follows: 'z' is to specify that the tar has been gzipped, 'x' for extraction, 'v' is for versbose, 'f' is to specify an archive file. These options combined will let tar know that the tarball file is gzipped and needs to be extracted.


Since we now have the unzipped source code, we need to get in to the directory and get things installed. So move into that directory:


```
cd node-v0.10.15
```


This folder contains raw files that need to be compiled specifically for your particular architecture and kernel. We have to create some files to give instructions during the compile. Don't worry, this is done very easily. Most source code packages like this come with a bash script called "configure" that will create all these files and set the options for you. So run the configure script:


```
./configure
```


It should only take a few seconds to complete, and it will output a lot of stuff. Unless you see anything that says "failed" or "exit code" followed by a bunch of errors, then it complete successfully. I'll assume it was successful, so we'll move on to the next step.


We need to have all the code compiled from their raw format, it's a really simple command to run.


```
make
```


Now go grab a drink, make a sandwhich, or possibly take a short nap. This can take roughly five minutes. Think about that, you could spin up a few VPS in the time it takes to have this compilied. It goes through all the dependecy files and compile them, link them together, and set it up to be used by the system. It will flash a bunch of long commands on your screen. You don't have to do anything, it runs all by itself. You can sit and watch it, but I personally recommend getting up and leaving the computer for a few minutes while it runs, your time will be better spent.


Once it is complete, there is only one more step. You're almost there so don't give up now. The final step is to actually install it on your system by having all the compiled files moved into various other folders on your VPS so the system can use them. This is a quick process, and once again the process is taken care of by a single command.


```
make install
```


That should only take a few moments to be done, and once it's done you can verify it has installed both Node.js and NPM:


```
node --version
v0.10.15

npm --version
1.3.5

```


If you see the versions as specified above, then that's it. You're all done and have successfully compiled Node.js from scratch! You can start installing node modules through NPM or if you already have an app on your VPS you can start node to serve it up.


