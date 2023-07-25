# How To Install Node js on CentOS 8

```JavaScript``` ```Node.js``` ```CentOS``` ```CentOS 8```

## Introduction


Node.js is a JavaScript runtime for server-side programming. It allows developers to create scalable backend functionality using JavaScript, a language many are already familiar with from browser-based web development.


In this guide, we will show you three different ways of getting Node.js installed on a CentOS 8 server:


- using dnf to install the nodejs package from CentOS’s default AppStream repository
- installing nvm, the Node Version Manager, and using it to install and manage multiple versions of node
- building and installing node from source

Most users should use dnf to install the built-in pre-packaged versions of Node. If you’re a developer or otherwise need to manage multiple installed versions of Node, use the nvm method. Building from source is rarely necessary for most users.


# Prerequisites


To complete this tutorial, you will need a server running CentOS 8. We will assume you are logged into this server as a non-root, sudo-enabled user. To set this up, see our Initial Server Setup for CentOS 8 guide.


# Option 1 — Installing Node from the CentOS AppStream Repository


Node.js is available from CentOS 8’s default AppStream software repository. There are multiple versions available, and you can choose between them by enabling the appropriate module stream. First list out the available streams for the nodejs module using the dnf command:


```
sudo dnf module list nodejs


```


```
OutputName                     Stream                   Profiles                                                Summary
nodejs                   10 [d]                   common [d], development, minimal, s2i                   Javascript runtime
nodejs                   12                       common, development, minimal, s2i                       Javascript runtime

```


Two streams are available, 10 and 12. The [d] indicates that version 10 is the default stream. If you’d prefer to install Node.js 12, switch module streams now:


```
sudo dnf module enable nodejs:12


```


You will be prompted to confirm your decision. Afterwards the version 12 stream will be enabled and we can continue with the installation. For more information on working with module streams, see the official CentOS AppStream documentation.


Install the nodejs package with dnf:


```
sudo dnf install nodejs


```


Again, dnf will ask you to confirm the actions it will take. Press y then ENTER to do so, and the software will install.


Check that the install was successful by querying node for its version number:


```
node --version


```


```
Outputv12.13.1

```


Your --version output will be different if you installed Node.js 10 instead.



Note: both available versions of Node.js are long-term support releases, meaning they have a longer guaranteed window of maintenance. See the official Node.js releases page for more lifecycle information.

Installing the nodejs package should also install the npm Node Package Manager utility as a dependency. Verify that it was installed properly as well:


```
npm --version


```


```
Output6.12.1

```


At this point you have successfully instlled Node.js and npm using the CentOS software repositories. The next section will show how to use the Node Version Manager to do so.


# Option 2 — Installing Node Using the Node Version Manager


Another way of installing Node.js that is particularly flexible is to use nvm, the Node Version Manager. This piece of software allows you to install and maintain many different independent versions of Node.js, and their associated Node packages, at the same time.


To install NVM on your CentOS 8 machine, visit the project’s GitHub page. Copy the curl command from the README file that displays on the main page. This will get you the most recent version of the installation script.


Before piping the command through to bash, it is always a good idea to audit the script to make sure it isn’t doing anything you don’t agree with. You can do that by removing the | bash segment at the end of the curl command:


```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh


```


Take a look and make sure you are comfortable with the changes it is making. When you are satisfied, run the command again with | bash appended at the end. The URL you use will change depending on the latest version of NVM, but as of right now, the script can be downloaded and executed by typing:


```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash


```


This will install the nvm script to your user account. To use it, you must first source your .bash_profile file:


```
source ~/.bash_profile


```


Now, you can ask NVM which versions of Node are available:


```
nvm list-remote

```


```
. . .
       v12.13.0   (LTS: Erbium)
       v12.13.1   (LTS: Erbium)
       v12.14.0   (LTS: Erbium)
       v12.14.1   (LTS: Erbium)
       v12.15.0   (LTS: Erbium)
       v12.16.0   (LTS: Erbium)
       v12.16.1   (Latest LTS: Erbium)
        v13.0.0
        v13.0.1
        v13.1.0
        v13.2.0
        v13.3.0
        v13.4.0
        v13.5.0
        v13.6.0
        v13.7.0
        v13.8.0
        v13.9.0
       v13.10.0
       v13.10.1
       v13.11.0
       v13.12.0

```


It’s a very long list! You can install a version of Node by typing any of the release versions you see. For instance, to get version v13.6.0, you can type:


```
nvm install v13.6.0


```


You can see the different versions you have installed by typing:


```
nvm list

```


```
Output->      v13.6.0
default -> v13.6.0
node -> stable (-> v13.6.0) (default)
stable -> 13.6 (-> v13.6.0) (default)

```


This shows the currently active version on the first line (->      v13.6.0), followed by some named aliases and the versions that those aliases point to.



Note: if you also have a version of Node installed through the CentOS software repositories, you may see a system -> v12.13.1 (or some other version number) line here. You can always activate the system version of Node using nvm use system.

Additionally, you’ll see aliases for the various long-term support (or LTS) releases of Node:


```
Outputlts/* -> lts/erbium (-> N/A)
lts/argon -> v4.9.1 (-> N/A)
lts/boron -> v6.17.1 (-> N/A)
lts/carbon -> v8.17.0 (-> N/A)
lts/dubnium -> v10.19.0 (-> N/A)
lts/erbium -> v12.16.1 (-> N/A)

```


We can install a release based on these aliases as well. For instance, to install the latest long-term support version, erbium, run the following:


```
nvm install lts/erbium


```


```
OutputDownloading and installing node v12.16.1...
. . .
Now using node v12.16.1 (npm v6.13.4)

```


You can switch between installed versions with nvm use:


```
nvm use v13.6.0

```


```
Now using node v13.6.0 (npm v6.13.4)

```


You can verify that the install was successful using the same technique from the other sections, by typing:


```
node --version

```


```
Outputv13.6.0

```


The correct version of Node is installed on our machine as we expected. A compatible version of npm is also available.


# Option 3 — Installing Node from Source


Another way to install Node.js is to download the source code and compile it yourself.


To do so, use your web browser to navigate to the official Node.js download page, right-click on the Source Code link and click Copy Link Address or whichever similar option your browser gives you.


Back in your SSH session, first make sure you’re in a directory you can write to. We’ll use the current user’s home directory:


```
cd ~


```


Then type curl, paste the link that you copied from the website, and follow it with | tar xz:


```
curl https://nodejs.org/dist/v12.16.1/node-v12.16.1.tar.gz | tar xz


```


This will use the curl utility to download the source, then pipe it directly to the tar utility, which will extract it into the current directory.


Move into the newly created source directory:


```
cd node-v*


```


There are a few packages that we need to download from the CentOS repositories in order to compile the code. Use dnf to install these now:


```
sudo dnf install gcc-c++ make python2


```


You will be prompted to confirm the installation. Type y then ENTER to do so. Now, we can configure and compile the software:


```
./configure
make -j4


```


The compilation will take quite a while (around 30 minutes on a four-core server). We’ve used the -j4 option to run four parallel compilation processes. You can omit this option or update the number based on the number of processor cores you have available.


When compilation is finished, you can install the software onto your system by typing:


```
sudo make install


```


To check that the installation was successful, ask Node to display its version number:


```
node --version


```


```
v12.16.1

```


If you see the correct version number, then the installation was completed successfully. By default Node also installs a compatible version of npm, so that should be available as well.


# Conclusion


In this tutorial we’ve shown how to install Node.js using the CentOS AppStream software repository, using Node Version Manager, and by compiling from source.


If you’d like more information on programming in JavaScript, please read our related tutorial series:


- How To Code in Javascript: a comprehensive overview of the JavaScript language, applicable to both the browser and Node.js
- How To Code in Node.js: a series of exercises that teach the basics of using Node.js

