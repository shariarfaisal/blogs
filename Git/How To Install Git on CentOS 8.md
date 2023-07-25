# How To Install Git on CentOS 8

```Git``` ```Open Source``` ```CentOS```

## Introduction


Version control systems are an indispensable part of modern software development. Versioning allows you to keep track of your software at the source level. You can track changes, revert to previous stages, and branch to create alternate versions of files and directories.


One of the most popular version control systems currently available is Git. Many projects’ files are maintained in a Git repository, and sites like GitHub, GitLab, and Bitbucket help to facilitate software development project sharing and collaboration.


In this guide, we will go through how to install and configure Git on a CentOS 8 server. We will cover how to install the software two different ways: via the built-in package manager and via source. Each of these approaches has their own benefits depending on your specific needs.


# Prerequisites


You will need a CentOS 8 server with a non-root superuser account.


To set this up, you can follow our Initial Server Setup Guide for CentOS 8.


With your server and user set up, you are ready to begin.


# Installing Git with Default Packages


Our first option to install Git is via CentOS’s default packages.


This option is best for those who want to get up and running quickly with Git, those who prefer a widely-used stable version, or those who are not looking for the newest available options. If you are looking for the most recently release, you should jump to the section on installing from source.


We will be using the open-source package manager tool DNF, which stands for Dandified YUM the next-generation version of the Yellowdog Updater, Modified (that is, yum). DNF is a package manager that is now the default package manager for Red Hat based Linux systems like CentOS. It will let you install, update, and remove software packages on your server.


First, use the DNF package management tools to update your local package index.


```
sudo dnf update -y


```


The -y flag is used to alert the system that we are aware that we are making changes, preventing the terminal from prompting us to confirm.


With the update complete, you can install Git:


```
sudo dnf install git -y


```


You can confirm that you have installed Git correctly by running the following command:


```
git --version


```


```
Outputgit version 2.18.2

```


With Git successfully installed, you can now move on to the Setting Up Git section of this tutorial to complete your setup.


# Installing Git from Source


A more flexible method of installing Git is to compile the software from source. This takes longer and will not be maintained through your package manager, but it will allow you to download the latest release and will give you some control over the options you include if you wish to customize.


Before you begin, you need to install the software that Git depends on. This is all available in the default repositories, so we can update our local package index and then install the packages.


```
sudo dnf update -y
sudo dnf install gettext-devel openssl-devel perl-CPAN perl-devel zlib-devel gcc autoconf -y


```


After you have installed the necessary dependencies, create a temporary directory and move into it. This is where we will download our Git tarball.


```
mkdir tmp
cd /tmp


```


From the Git project website, we can navigate to the Red Hat Linux distribution tarball list available at https://mirrors.edge.kernel.org/pub/software/scm/git/ and download the version you would like. At the time of writing, the most recent version is 2.26.0, so we will download that for demonstration purposes. We’ll use curl and output the file we download to git.tar.gz.


```
curl -o git.tar.gz https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.26.0.tar.gz


```


Unpack the compressed tarball file:


```
tar -zxf git.tar.gz


```


Next, move into the new Git directory:


```
cd git-*


```


Now, you can make the package and install it by typing these two commands:


```
make prefix=/usr/local all
sudo make prefix=/usr/local install


```


With this complete, you can be sure that your install was successful by checking the version.


```
git --version


```


```
Outputgit version 2.26.0

```


With Git successfully installed, you can now complete your setup.


# Setting Up Git


Now that you have Git installed, you should configure it so that the generated commit messages will contain your correct information.


This can be achieved by using the git config command. Specifically, we need to provide our name and email address because Git embeds this information into each commit we do. We can go ahead and add this information by typing:


```
git config --global user.name "Your Name"
git config --global user.email "youremail@domain.com"


```


We can display all of the configuration items that have been set by typing:


```
git config --list


```


```
Outputuser.name=Your Name
user.email=youremail@domain.com
...

```


The information you enter is stored in your Git configuration file, which you can optionally edit by hand with a text editor like this:


```
vi ~/.gitconfig


```


~/.gitconfig contents
```
[user]
  name = Your Name
  email = youremail@domain.com

```


Press ESC then :q to exit the text editor.


There are many other options that you can set, but these are the two essential ones needed. If you skip this step, you’ll likely see warnings when you commit to Git. This makes more work for you because you will then have to revise the commits you have done with the corrected information.


# Conclusion


You should now have Git installed and ready to use on your system.


To learn more about how to use Git, check out these articles and series:


- How To Use Git Effectively
- How To Use Git Branches
- An Introduction to Open Source

