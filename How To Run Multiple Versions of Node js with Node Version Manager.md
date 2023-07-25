# How To Run Multiple Versions of Node js with Node Version Manager

```Node.js```

## Introduction


If you work on multiple Node.js projects, you’ve probably run into this one time or another. You have the latest and greatest version of Node.js installed, and the project you’re about to work on requires an older version. In those situations, the Node Version Manager (nvm) is a great tool to use, allowing you to install multiple versions of Node.js and switch between them as you see fit.


In this tutorial, you will install nvm and learn to install, remove, and switch between different versions of Node.js.


# Prerequisites


To complete this tutorial, you will need the following:


- The latest version of Node installed on your machine. To install Node on macOS, follow the steps outlined in this How to Install Node.js and Create a Local Development Environment on macOS tutorial.

# Step 1 — Getting Started


To get started, you will need to install the Node Version Manager, or nvm, on your system. You can install it manually by running the following command:


```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash


```


If you prefer wget, you can run this command:


```
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash


```


Once installed, close your terminal application for changes to take effect. You will also need to add a couple of lines to your bash shell startup file. This file might have the name .bashrc, .bash_profile, or .zshrc depending on your operating system. To do this, reopen your terminal app and run the following commands:


```
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"


```


With nvm installed, you can now install and work with multiple versions of Node.js.


# Step 2 — Installing Multiple Node.js Versions


Now that you have nvm installed, you can install a few different versions of Node.js:


```
nvm install 0.10


```


After running this command, this is the output that will display in your terminal app:


```
OutputDownloading and installing node v0.10.48...
Downloading https://nodejs.org/dist/v0.10.48/node-v0.10.48-darwin-x64.tar.xz...
######################################################################### 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v0.10.48 (npm v2.15.1)

```


You can also install Node version 8 and version 12:


```
nvm install 8
nvm install 12


```


Upon running each command, nvm will download the version of Node.js from the official website and install it. Once installed, it will also set the version you just installed as the active version.


If you were to run node --version after each of the aforementioned commands, you’d see the most recent version of the respective major version.


nvm isn’t limited to major versions either. You could also run nvm install 12.0.0 to explicitly install the specific 12.0.0 version of Node.js.


# Step 3 — Listing Installed Node.js Versions


With a handful of different versions of Node.js installed, we can run nvm with the ls argument to list out everything we have installed:


```
nvm ls


```


The output produced by running this command might look something like this:


```
Output    v0.10.48
        v4.9.1
    v6.10.3
    v6.14.4
        v8.4.0
    v8.10.0
    v10.13.0
    v10.15.0
    v10.15.3
    ->      v12.0.0
    v12.7.0
        system
default -> v10.15 (-> v10.15.3)
node -> stable (-> v12.7.0) (default)
stable -> 12.7 (-> v12.7.0) (default)
iojs -> N/A (default)
unstable -> N/A (default)
lts/* -> lts/dubnium (-> N/A)
lts/argon -> v4.9.1
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.16.0 (-> N/A)
lts/dubnium -> v10.16.0 (-> N/A)

```


Your output will probably differ depending on how many versions of Node.js you have installed on your machine.


The little -> indicates the active version, and default -> indicates the default version of Node.js. The default version of Node is the version that will be available when you open a new shell. system corresponds with the version of Node.js installed outside of nvm on your system.


You may want to change the version of Node.js that your machine defaults to. You can also use nvm to accomplish this.


# Step 4 — Setting a Default Node.js Version


Even with juggling multiple versions, there’s a good chance you have one version that you would prefer to run the majority of the time. Often times, that would be the latest stable version of Node.js. During the time of the release of this tutorial, the latest stable version of Node.js is version 15.1.0.


To set the latest stable version as your default, run:


```
nvm alias default stable


```


After running this command, this will be the output you see:


```
Outputdefault -> stable (-> v15.1.0)

```


You may also have a specific version number you would like to set as your default. To alias default to a specific version, run:


```
nvm alias default 10.15


```


```
Outputdefault -> 10.15 (-> v10.15.3)

```


Now every time you open a new shell, that version of Node.js will be immediately available. Some work you do may require different versions of Node.js. This is something nvm can help you with as well.


# Step 5 — Switching Between Node.js Versions


To switch to a different version of Node.js, use the nvm command use followed by the version of Node.js you would like to use:


```
nvm use 0.10


```


This is the output you will see:


```
OutputNow using node v0.10.48 (npm v2.15.1)

```


You can even switch back to your default version:


```
nvm use default


```


At this point, you have installed several versions of Node.js. You can use nvm to uninstall any unwanted version of Node.js you may have.


# Step 6 — Removing Node.js Versions


You may have several versions of Node.js installed due to working on a variety of projects on your machine.


Fortunately, you can remove Node.js versions just as easily as you installed them:


```
nvm uninstall 0.10


```


This is the output that will display after running this command:


```
OutputUninstalled node v0.10.48

```


Unfortunately, when you specify a major or minor version, nvm will only uninstall the latest installed version that matches the version number.


So, if you have two different versions of Node.js version 6 installed, you have to run the uninstall command for each version:


```
Output$ nvm uninstall 6
Uninstalled node v6.14.4

$ nvm uninstall 6
Uninstalled node v6.10.3

```


It’s worth noting that you can’t remove a version of Node.js that is currently in use and active.


You may want to return to your system’s default settings and stop using nvm. The next step will explain how to do this.


# Step 7 — Unloading Node Version Manager


If you would like to completely remove nvm from your machine, you can use the unload command:


```
nvm unload


```


If you would still like to keep nvm on your machine, but you want to return to your system’s installed version of Node.js, you can make the switch by running this command:


```
nvm use system


```


Now your machine will return to the installed version of Node.js.


# Conclusion


Working on multiple projects that use different versions of Node.js doesn’t have to be a nightmare. Node Version Manager makes the process seamless. If you would like to avoid having to remember to switch versions, you can take things a step further by creating a .nvmrc file in your project’s root:


```
$ echo "12" > .nvmrc


```


As a next step, you can learn to create your very own Node.js program with this How To Write and Run Your First Program in Node.js tutorial.


