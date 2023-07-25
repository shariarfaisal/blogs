# How To Install asdf to Manage Multiple Programming Language Runtime Versions on Ubuntu 22 04

```Node.js``` ```Ubuntu 22.04```

## Introduction


asdf is a command line interface tool, or CLI tool, for managing different runtime versions across multiple programming languages. It unifies all the runtimes under one configuration file, and uses a plugin structure to manage everything with one tool. As an example, you can install Node.js, but then have asdf as a central repository of plugins with each plugin being maintained either officially or by community contributors.


In this tutorial, you will install the asdf core and the Node.js plugin with build dependencies, which is the minimum required for functionality. You will then install Node.js and manage the version you want to use, depending on your desired scope.


# Prerequisites


- An Ubuntu 22.04 server, set up according to our initial server setup guide for Ubuntu 22.04, with a non-root user with sudo privileges and a firewall enabled.

# Step 1 — Installing asdf Core


asdf relies on the installation of a core that, alone, does not have functionality. The asdf core relies on separate plugins that are specific to a given programming language or program. Most commonly, it is used to install and manage multiple versions of a given programming language. It’s recommended that you download the asdf core with git, which comes installed with Ubuntu 22.04. To get the latest version of asdf, clone the latest branch from the asdf repository:


```
git clone https://github.com/asdf-vm/asdf.git ~/.asdf --branch v0.10.2


```


asdf requires a unique installation depending on the combination of shell type and method it was downloaded. By default, Ubuntu uses Bash for its shell, which uses the ~/.bashrc file for configuration and customization. To enable the usage of the asdf command, you will have to add the following line:


```
echo ". $HOME/.asdf/asdf.sh" >> ~/.bashrc


```


Next, make sure your changes are applied to your current session:


```
source ~/.bashrc


```



Note: If you are using ZSH instead of Bash, you can add the same line but to the ~/.zshrc file instead.

With the core installed, you can now install the plugin.


# Step 2 — Installing the asdf Node.js Plugin and Build Dependencies


Installing the plugin for Node.js for asdf is not the same as installing Node.js by itself. That will occur in the next step. As mentioned previously, the minimum requirements for a usable asdf setup is the asdf core and at least one plugin. Once you install this plugin, you can use it to install the runtime it handles.


Every asdf plugin is maintained separately.  Some are maintained by the core asdf team, but most are community maintained. Every asdf plugin has its own repository and dependencies that need to be installed. You must check each plugin repository, such as with the Node.js plugin repository. This plugin in particular is officially maintained by the asdf team.


To install the plugin, use the following asdf plugin add command:


```
asdf plugin add nodejs https://github.com/asdf-vm/asdf-nodejs.git


```


For this Node.js plugin, the dependencies are mentioned in the “Use” section of their “README” file. Within that section, the explicit dependencies are linked to in the official Node.js repositories’ section on building Node.js. This must be done manually because asdf is a solution targeted at multiple operating systems, with each one having their own unique dependencies and methods to install them. This can also vary from plugin to plugin. For this plugin on Ubuntu, you need to install these dependencies. Begin by updating your apt source index:


```
sudo apt update


```


Then, you can install the required dependencies:


```
sudo apt install python3 g++ make python3-pip


```


For this Node.js plugin, depending on the version you need installed, it picks either pre-compiled binaries or compiles binaries from source. If you happen to choose a version that requires compiling from source, the aforementioned dependencies are required.


With the plugin successfully installed, next you can install Node.js.


# Step 3 — Installing Node.js


You can install multiple Node.js versions, choosing from the latest or any specified versions. To install the latest version of Node.js, enter the following:


```
asdf install nodejs latest


```


```
OutputTrying to update node-build... ok
Downloading node-v18.10.0-linux-x64.tar.gz...
-> https://nodejs.org/dist/v18.10.0/node-v18.10.0-linux-x64.tar.gz
Installing node-v18.10.0-linux-x64...
Installed node-v18.10.0-linux-x64 to /home/sammy/.asdf/installs/nodejs/18.10.0

```


Installing the latest version is a shortcut provided by asdf, it is not a special version. asdf identifies and enforces versions by their exact numbers. To install a specific version of Node.js, enter the following:


```
asdf install nodejs 16.16.0


```


```
OutputTrying to update node-build... ok
Downloading node-v16.16.0-linux-x64.tar.gz...
-> https://nodejs.org/dist/v16.16.0/node-v16.16.0-linux-x64.tar.gz
Installing node-v16.16.0-linux-x64...
Installed node-v16.16.0-linux-x64 to /home/sammy/.asdf/installs/nodejs/16.16.0

```


With these two versions installed, you can check all the versions you have with the following:


```
asdf list nodejs


```


```
Output  16.16.0
  18.10.0

```


Additionally, if you ever want to remove a version, you can use the uninstall command with a specific version target:


```
asdf uninstall nodejs 16.16.0


```


Now that Node.js is installed, you can choose the version you want active.


# Step 4 — Choosing the Active Node.js Version


asdf can set the version of Node.js at three different levels: local, global, and shell. If you only want to set the Node.js version for your project’s working directory, run the following:


```
asdf local nodejs latest


```


Setting the current version at the global level acts at the user level for your system:


```
asdf global nodejs latest


```


If you only want to set the version for the current shell session, enter the following:


```
asdf shell nodejs latest


```


Now you have a complete installation of Node.js using asdf, with the ability to switch to the version you need at the scope that you want.


# Conclusion


In this tutorial you installed the asdf core, the asdf Node.js plugin, then Node.js itself. asdf enables multiple versions of a runtime to be installed, and you choose the version at different levels of scope from globally to working project directory. If you’re interested in a conventional installation of Node.js, check our this tutorial on how to install Node.js on Ubuntu 22.04.


