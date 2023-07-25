# How To Install Node js on Debian 10

```Node.js``` ```Debian``` ```Debian 10```

## Introduction


Node.js is a JavaScript runtime for server-side programming. It allows developers to create scalable backend functionality using JavaScript, a language many are already familiar with from browser-based web development.


In this guide, you will learn how to install Node.js on a Debian 10 server three different ways:


- Using the default Debian repository
- Using a PPA software repository
- Using NVM (Node Version Manager) to install and activate different versions of Node.

For many users, using apt with the default repository will be sufficient. If you need specific newer (or legacy) versions of Node, you should use the PPA repository. If you are actively developing Node applications and need to switch between versions frequently, choose the NVM method.


# Prerequisites


Before you begin, you should have a non-root user with sudo privileges set up on your system. You can learn how to set this up by following the initial server setup for Debian 10 tutorial.


# Installing Node.js with Apt from the Default Repositories


Debian contains a version of Node.js in its default repositories that can be used to provide a consistent experience across multiple systems. At the time of writing, the version in the repositories is 10.24.0. This will not be the latest version, but it should be stable and sufficient for quick experimentation with the language.



Warning: The version of Node.js included with Debian 10, version 10.24.0, is unsupported and unmaintained. You should not use this version in production, and should refer to one of the other sections in this tutorial to install a more recent version of Node.

To get Node.js from the default Debian software repository, you can use the apt package manager. First, refresh your local package index:


```
sudo apt update


```


Then install the Node.js package:


```
sudo apt install nodejs


```


To verify that the installation was successful, run the node command with the -v flag to get the version:


```
node -v


```


```
Outputv10.24.0

```


If the package in the repositories suits your needs, this is all you need to do to get set up with Node.js. In most cases, you’ll also want to also install npm, the Node.js package manager. You can do this by installing the npm package with apt:


```
sudo apt install npm


```


This will allow you to install modules and packages to use with Node.js.


At this point you have successfully installed Node.js and npm using apt and the default Ubuntu software repositories. The next section will show how to use an alternate repository to install different versions of Node.js.


# Installing Node.js with Apt Using a NodeSource PPA


To work with a more recent version of Node.js, you can install from a PPA (personal package archive) maintained by NodeSource. This is an alternate repository that still works with apt, and will have more up-to-date versions of Node.js than the official Debian repositories. NodeSource has PPAs available for multiple Node versions. Refer to the NodeSource documentation for more information on the available versions.


From your home directory, use curl to retrieve the installation script for your preferred Node.js version. If you do not have curl installed, you can install it before proceeding to the next step with this command:


```
sudo apt install curl


```


With curl installed, you can begin your Node.js installation. This example installs version 16.x. You can replace 16.x with your preferred version.


```
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install nodejs


```


You don’t need to install a separate package for npm in this case, as it is included in the nodejs package.


Verify the installation by running node with the -v version option:


```
node -v


```


```
Outputv16.14.2

```


npm uses a configuration file in your home directory to keep track of updates. It will be created the first time you run npm. Execute this command to verify that npm is installed:


```
npm -v


```


```
Output8.5.0

```


# Installing Node Using the Node Version Manager


An alternative to installing Node.js through apt is to use a tool called nvm, which stands for “Node Version Manager”. Rather than working at the operating system level, nvm works at the level of an independent directory within your user’s home directory. This means that you can install multiple self-contained versions of Node.js without affecting the entire system.


Controlling your environment with nvm allows you to access the newest versions of Node.js while also retaining and managing previous releases. It is a different utility from apt, however, and the versions of Node.js that you manage with it are distinct from those you manage with apt.


To install nvm on Debian 10, follow the installation instructions on the README file from the NVM Github repository.


The URL may change depending on the latest version of nvm, but as of this writing, the script can be downloaded and executed by typing:


```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash


```


This will install the nvm script to your user account. To use it, you must first source your .bashrc file. This allows you to execute the code:


```
source ~/.bashrc


```


Now, you can list the available Node versions using nvm:


```
nvm list-remote


```


This command will produce a long output:


```
Output...
       v15.11.0
       v15.12.0
       v15.13.0
       v15.14.0
        v16.0.0
        v16.1.0
        v16.2.0
        v16.3.0
        v16.4.0
        v16.4.1
        v16.4.2
        v16.5.0
        v16.6.0
        v16.6.1
        v16.6.2
        v16.7.0
        v16.8.0
        v16.9.0
        v16.9.1
       v16.10.0
       v16.11.0
       v16.11.1
       v16.12.0
       v16.13.0   (LTS: Gallium)
       v16.13.1   (LTS: Gallium)
       v16.13.2   (LTS: Gallium)
       v16.14.0   (LTS: Gallium)
       v16.14.1   (LTS: Gallium)
       v16.14.2   (Latest LTS: Gallium)
        v17.0.0
        v17.0.1
...

```


You can install a version of Node by typing any of the release versions you see. For example, to install version v14.10.0, you can type:


```
nvm install v14.10.0


```


You can view the different versions you have installed by typing:


```
nvm ls


```


```
Output->     v14.10.0
         system
default -> v14.10.0
iojs -> N/A (default)
unstable -> N/A (default)
node -> stable (-> v14.10.0) (default)
stable -> 14.10 (-> v14.10.0) (default)
...

```


This shows the currently active version on the first line (-> v14.10.0), followed by some named aliases and the versions that those aliases point to.



Note: If you also have a version of Node.js installed through apt, you may see a system entry here. You can activate the system installed version of Node using nvm use system.

Additionally, this output lists aliases for the various long-term support (LTS) releases of Node:


```
Output. . .
lts/* -> lts/fermium (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.24.1 (-> N/A)
lts/erbium -> v12.22.11 (-> N/A)
lts/fermium -> v14.19.1 (-> N/A)
lts/gallium -> v16.14.2 (-> N/A)

```


We can install a release based on these aliases as well. For instance, to install the latest long-term support version, gallium, run the following:


```
nvm install lts/gallium


```


```
OutputDownloading and installing node v16.14.2...
...
Now using node v16.14.2 (npm v8.5.0)

```


You can verify that the installation was successful by typing:


```
node -v


```


```
Outputv16.14.2

```


If you wish to use a particular Node version as a default, type the following with the version of your choosing:


```
nvm alias default 14.10.0


```


This version will be automatically selected when you start a new session in Node. You can also reference it by the alias like this:


```
nvm use default


```


```
OutputNow using node v14.10.0 (npm v6.14.8)

```


Each version of Node.js will keep track of its own packages and has npm available to manage these.


# Removing Node.js


You can uninstall Node.js using apt or nvm, depending on the version you want to target. To remove versions installed from the Debian repository or from the PPA, you will need to work with the apt utility at the system level.


To remove either of these versions, type the following:


```
sudo apt remove nodejs


```


This command will remove the package and the configuration files.


To uninstall a version of Node.js that you have enabled using nvm, first determine whether or not the version you would like to remove is the current active version:


```
nvm current


```


```
Outputv16.14.2

```


If the version you are targeting is not the current active version, you can run this command with the version you want to remove:


```
nvm uninstall node_version_to_remove


```


This command will uninstall the selected version of Node.js.


If the version you would like to remove is the current active version, you must first deactivate nvm to enable your changes:


```
nvm deactivate


```


You can now uninstall the current version using the nvm uninstall command with your current version of Node.js.


```
nvm uninstall current_node_version


```


This will remove all files associated with the targeted version of Node.js except the cached files that can be used for reinstallation.


# Conclusion


There are quite a few ways to get up and running with Node.js on your Debian 10 server. Your circumstances will dictate which of the above methods is best for your needs. While using the packaged version in the Debian repository is an option for experimentation, installing from a PPA and working with a NodeSource PPA  or nvm offers additional flexibility.


For more information on programming with Node.js, please refer to our tutorial series How To Code in Node.js.


