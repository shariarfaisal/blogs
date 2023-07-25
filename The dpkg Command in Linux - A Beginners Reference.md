# The dpkg Command in Linux - A Beginners Reference

```UNIX/Linux```

Let’s discuss the dpkg command in Linux in this article. Packages help in delivering or installing any application on a Linux system. Essentially, packages are compressed archive of the files and dependencies required to install a program or service.


These packages are used when you want to install a new program or service on their system. All the packages on a system are stored in a local ‘repository’.


This repository can be accessed by a package management service whenever required. Let’s talk about one of those package management utilities, the dpkg command in Linux today.


# What is the dpkg command?


Essentially, the man page describes it like this: “dpkg is a tool to install, build, remove and manage Debian packages.”


We use the dpkg command to interact with packages on our system. It is controlled fully with the help of command-line parameters and the first parameter is referred to as the action parameter that is used to direct what to do. This parameter may or may not be followed by any other parameter.


Later, a new tool named aptitude was designed to provide a more user-friendly, interactive front-end for the users to manage packages without the complexity of the dpkg command. It interacts with the dpkg interface on behalf of the user. Now, let’s try and understand the dpkg command in Linux.


# The Basics of the dpkg Command in Linux


Here’s what the basic syntax of the dpkg command looks like:


```
dpkg [options] [.deb package name]

```


The dpkg command provides a long list of options to customise the data we receive while analysing our network. Here is a list of some of the most popular dpkg options.











Option
Function


-i OR --install
Install a package using the dpkg command. The command will extract all control files for the specified package, remove any previously installed older instance of the package, and install the new package on our system.


-r OR --remove
Remove an installed package from our system. It removes every file belonging to the specific package except the configuration files. This can be seen as the uninstallation option.


-P OR --purge
An alternative way to remove an installed package from our system. It completely removes every fie belonging to the specific package, including the configuration files. This can be seen as the ‘complete uninstallation’ option.


--update-avail
Uhe information of the dpkg command about available packages in its repositories. If new packages are available, they are synced from the official repositories.


--merge-avail
Merge the information of the dpkg command about available packages in its repositories with previously available information. It is usually run right after the previous command.


--help
Display the help page for the dpkg command and exit.




These are some of the most commonly used options for the dpkg command and you can explore more by displaying the help options in your terminal.


# Using the dpkg command


Let us explore the common uses of the dpkg command. As the command works the same for both Debian and Ubuntu systems, we will only mention Ubuntu in this tutorial from now on.


## 1. Installing a package


The most basic use of the dpkg command in Ubuntu is a package installation. We can install a deb package in Ubuntu or Debian using the dpkg -i command option.


Here’s how you’d install a package.


```
sudo dpkg -i [package name]

```


We’re installing the VLC player on our Ubuntu system. Have a look at the below screenshot for what the installation looks like on screen.


Dpkg Command
You can also install multiple packages at the same time by specifiying the package names separated by spaces.


## 2. Removing a package


When you no longer need a program or service on your system, there is no use keeping it.


The dpkg command has got us covered here as well.


We can uninstall a program or service from our system using the dpkg -r option.


Let’s remove the VLC player that we installed for this demonstration.


```
sudo dpkg -r [package name]

```


Look at the below screenshot to see how dpkg triggers changes for all the dependent menus, desktop icons, etc similar to the apt command.


Dpkg R Vlc
## 3. Updating your repositories


The dpkg repository stores all the packages available for installation on your Ubuntu or Debian Linux distribution.


However, as these packages are stored locally you can often end up having old versions of packages for a program when newer versions have already been released. This causes a need for a method to update your repositories.


Guess what? The dpkg --update-avail option has got you covered.


It checks the online repositories and downloads all the updated packages to your local repository.


Let’s update our local repositories to the latest version:


```
sudo dpkg --update-avail

```


# Ending notes


That brings us to the end of our topic for the day. This is all you’d need for the most part when using the dpkg command in Linux. Most regular users would not need more than these three options for the command. However, if you’re a power user, you can run man dpkg and get complete details of everything the command can do.


