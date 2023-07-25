# How To Install and Use the Yarn Package Manager for Node js

```JavaScript``` ```Node.js``` ```Development```

## Introduction


Yarn is a package manager for Node.js that focuses on speed, security, and consistency. It was originally created to address some issues with the popular NPM package manager. Though the two package managers have since converged in terms of performance and features, Yarn remains popular, especially in the world of React development.


Some of the unique features of Yarn are:


- A per-project caching mechanism, that can greatly speed up subsequent installs and builds
- Consistent, deterministic installs that guarantee the structure of installed libraries are always the same
- Checksum testing of all packages to verify their integrity
- “Workspaces”, which facilitate using Yarn in a monorepo (multiple projects developed in a single source code repository)

In this tutorial you will install Yarn globally, add Yarn to a specific project, and learn some basic Yarn commands.


# Prerequisites


Before installing and using the Yarn package manager, you will need to have Node.js installed. To see if you already have Node.js installed, type the following command into your local command line terminal:


```
node -v


```


If you see a version number, such as v12.16.3 printed, you have Node.js installed. If you get a command not found error (or similar phrasing), please install Node.js before continuing.


To install Node.js, follow our tutorial for Ubuntu, Debian, CentOS, or macOS.


Once you have Node.js installed, proceed to Step 1 to install the Yarn package manager.


# Step 1 — Installing Yarn Globally


Yarn has a unique way of installing and running itself in your JavaScript projects. First you install the yarn command globally, then you use the global yarn command to install a specific local version of Yarn into your project directory. This is necessary to ensure that everybody working on a project (and all of the project’s automated testing and deployment tooling) is running the exact same version of yarn, to avoid inconsistent behaviors and results.


The Yarn maintainers recommend installing Yarn globally by using the NPM package manager, which is included by default with all Node.js installations. Use the -g flag with npm install to do this:


```
sudo npm install -g yarn


```


After the package installs, have the yarn command print its own version number. This will let you verify it was installed properly:


```
yarn --version


```


```
Output1.22.11

```


Now that you have the yarn command installed globally, you can use it to install Yarn into a specific JavaScript project.


# Step 2 — Installing Yarn in Your Project


If you are using Yarn to work with an existing Yarn-based project, you can skip this step. The project should already be set up with a local version of Yarn and all the configuration files necessary to use it.


If you are setting up a new project of your own, you’ll want to configure a project-specific version of Yarn now.


First, navigate to your project directory:


```
cd ~/my-project


```


If you don’t have a project directory, you can make a new one with mkdir and then move into it:


```
mkdir my-project
cd my-project


```


Now use the yarn set command to set the version to berry:


```
yarn set version berry


```


This will download the current, actively developed version of Yarn – berry – save it to a .yarn/releases/ directory in your project, and set up a .yarnrc.yml configuration file as well:


```
OutputResolving berry to a url...
Downloading https://github.com/yarnpkg/berry/raw/master/packages/berry-cli/bin/berry.js...
Saving it into /home/sammy/my-project/.yarn/releases/yarn-berry.cjs...
Updating /home/sammy/my-project/.yarnrc.yml...
Done!

```


Now try the yarn --version command again:


```
yarn --version


```


```
Output3.0.0

```


You’ll see the version is 3.0.0 or higher. This is the latest release of Yarn.



Note: if you cd out of your project directory and run yarn --version again, you’ll once again get the global Yarn’s version number, 1.22.11 in this case. Every time you run yarn, you are using the globally installed version of the command. The global yarn command first checks to see if it’s in a Yarn project directory with a .yarnrc.yml file, and if it is, it hands the command off to the project-specific version of Yarn configured in the project’s yarnPath setting.

Your project is now set up with a project-specific version of Yarn. Next we’ll look at a few commonly used yarn commands to get started with.


# Using Yarn


Yarn has many subcommands, but you only need a few to get started. Let’s look at the first subcommands you’ll want to use.


## Getting Help


When starting out with any new tool, it’s useful to learn how to access its online help. In Yarn the --help flag can be added to any command to get more information:


```
yarn --help


```


This will print out overall help for the yarn command. To get more specific information about a subcommand, add --help after the subcommand:


```
yarn install --help


```


This would print out details on how to use the yarn install command.


## Starting a New Yarn Project


If you’re starting a project from scratch, use the init subcommand to create the Yarn-specific files you’ll need:


```
yarn init


```


This will add a package.json configuration file and a yarn.lock file to your directory. The package.json contains configuration and your list of module dependencies. The yarn.lock file locks those dependencies to specific versions, making sure that the dependency tree is always consistent.


## Installing all of a Project’s Dependencies


To download and install all the dependencies in an existing Yarn-based project, use the install subcommand:


```
yarn install


```


This will download and install the modules you need to get started.


## Adding a New Dependency to a Project


Use the add subcommand to add new dependencies to a project:


```
yarn add package-name


```


This will download the module, install it, and update your package.json and yarn.lock files.


## Updating Your .gitignore File for Yarn


Yarn stores files in a .yarn folder inside your project directory. Some of these files should be checked into version control and others should be ignored. The basic .gitignore configuration for Yarn follows:


.gitignore
```
.yarn/*
!.yarn/patches
!.yarn/releases
!.yarn/plugins
!.yarn/sdks
!.yarn/versions
.pnp.*

```


This ignores the entire .yarn directory, and then adds in some exceptions for important folders, including the releases directory which contains your project-specific version of Yarn.


For more details on how to configure Git and Yarn, please refer to the official Yarn documentation on .gitignore.


# Conclusion


In this tutorial you installed Yarn and learned about a few yarn subcommands. For more information on using Yarn, take a look at the official Yarn CLI documentation.


For more general Node.js and JavaScript help, please visit our Node.js and JavaScript tag pages, where you’ll find relevant tutorials, tech talks, and community Q&A.


