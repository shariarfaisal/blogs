# How To Install and Use LinuxBrew on a Linux VPS

```Ubuntu``` ```Miscellaneous``` ```Debian``` ```CentOS```


Status: Deprecated
This article is deprecated and no longer maintained.
Reason
Homebrew now provides mainline support for Linux.
See Instead
This article may still be useful as a reference, but may not work or follow best practices. We strongly recommend using a recent article written for the operating system you are using.

How to Install and Use Homebrew on Linux


## Intro



LinuxBrew is a Linux-fork of the popular Mac OS X HomeBrew package manager.


LinuxBrew is package-management-software, which enables installing packages from source, on top on the system’s default package management (e.g. “apt/deb” in Debian/Ubuntu and “yum/rpm” in CentOS/RedHat).


# Why Use LinuxBrew ?



- 
HomeBrew was originally developed for Mac OS X (which does not have a standard open-source package-management system). It superceded package-managements such as MacPorts and Fink. LinuxBrew is homebrew ported to Linux.

- 
Most Linux distributions have a good package management system (e.g. “apt/deb” in Debian/Ubuntu and “yum/rpm” in CentOS/RedHat), however


Packages in the standard repositories are often older than the latest available versions, and


Many open-source packages are not available in the standard repositories (e.g. common bioinformatics tools).



- 
Packages in the standard repositories are often older than the latest available versions, and

- 
Many open-source packages are not available in the standard repositories (e.g. common bioinformatics tools).

- 
LinuxBrew provides a repository of software installation recipes (packages are installed from source and compiled on the local machine) to complement the packages from the distribution’s standard repository.

- 
LinuxBrew provides an easy method to build your own repositories (i.e. list of open-source packages tailored to your needs).

- 
LinuxBrew installs software in user-specified directory (not system-wide), and does not require sudo access.

- 
LinuxBrew (and HomeBrew) integrates very well with GitHub, enabling sharing of installation recipes easily.


Especially with DigitalOcean, which (at the time of this writing) does not provide sharable Droplet Images (with custom-configured installed software), a LinuxBrew repository can provide a quick method to install specific packages and versions on a standard Linux machine.


# The Gist of LinuxBrew



Simply put, LinuxBrew takes care of downloading the tar.gz file and running ./configure && make && make install for you (or whichever commands are needed to install the package).


A LinuxBrew Formula is a Ruby script which defines where to find the tar.gz file, how to build the package, and how to install it.


A formula file can be as simple as hmmer.rb (a bioinformatics tool):


```
class Hmmer < Formula
  homepage 'http://hmmer.janelia.org/'
  url 'http://selab.janelia.org/software/hmmer3/3.1b1/hmmer-3.1b1.tar.gz'

  def install
    system "./configure", "--prefix=#{prefix}"
    system "make"
    system "make install"
  end
end

```


Or as complicated as emacs.rb.


Once a formula file is properly defined, installing the package is simply a matter of running:


```
$ brew install FORMULA

```


# Preparing for LinuxBrew - Debian/Ubuntu



For Debian/Ubuntu-based systems, run the following commands:


```
$ sudo apt-get update
$ sudo apt-get upgrade -y
$ sudo sudo apt-get install -y build-essential make cmake scons curl git \
                               ruby autoconf automake autoconf-archive \
                               gettext libtool flex bison \
                               libbz2-dev libcurl4-openssl-dev \
                               libexpat-dev libncurses-dev

```


# Preparing for LinuxBrew - CentOS/RedHat



For RedHat/CentOS-based systems, run the following commands:


```
$ sudo yum update -y
$ sudo yum groupinstall -y "Development Tools"
$ sudo yum install -y \
        autoconf automake19 libtool gettext \
        git scons cmake flex bison \
        libcurl-devel curl \
        ncurses-devel ruby bzip2-devel expat-devel

```


# Installing LinuxBrew



Installing LinuxBrew is simply a matter of cloning the LinuxBrew Repository.


## Step 1 - Clone LinuxBrew



To keep things tidy, clone LinuxBrew into a hidden directory in the user’s home directory:


```
$ git clone https://github.com/Homebrew/linuxbrew.git ~/.linuxbrew

```


But any other directory would work just as well.


## Step 2 - Update environment variables



The next step is to add LinuxBrew to the user’s environment variables.


Add the following lines to the end of the user’s ~/.bashrc file:


```
# Until LinuxBrew is fixed, the following is required.
# See: https://github.com/Homebrew/linuxbrew/issues/47
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:$PKG_CONFIG_PATH
## Setup linux brew
export LINUXBREWHOME=$HOME/.linuxbrew
export PATH=$LINUXBREWHOME/bin:$PATH
export MANPATH=$LINUXBREWHOME/man:$MANPATH
export PKG_CONFIG_PATH=$LINUXBREWHOME/lib64/pkgconfig:$LINUXBREWHOME/lib/pkgconfig:$PKG_CONFIG_PATH
export LD_LIBRARY_PATH=$LINUXBREWHOME/lib64:$LINUXBREWHOME/lib:$LD_LIBRARY_PATH

```


NOTE: If you installed LinuxBrew to a different directory, change the path in LINUXBREWHOME above.


## Step 3 - Test installation



To ensure those changes take effect, log-out and log-in again. The shell should then use these new settings.


To test these new settings, try:


```
$ which brew
/home/ubuntu/.linuxbrew/bin/brew
$ echo $PKG_CONFIG_PATH
/home/ubuntu/.linuxbrew/lib64/pkgconfig:/home/ubuntu/.linuxbrew/lib/pkgconfig:/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig:/usr/lib64/pkgconfig:/usr/lib/pkgconfig:/usr/lib/x86_64-linux-gnu/pkgconfig:/usr/lib64/pkgconfig:/usr/share/pkgconfig:

```


# Installing Packages with LinuxBrew



## Which packages are available?



Type brew search to see the list of all available packages (all the packages that the current installation of LinuxBrew knows about – see below about adding repositories).


Type brew search WORD to see all the packages (called Formulas in HomeBrew jargon) which contain WORD. Example:


```
$ brew search xml
blahtexml       libnxml   libxml2     xml-coreutils   xml2        xmlrpc-c
html-xml-utils  libwbxml  libxmlsec1  xml-security-c  xmlcatmgr   xmlsh
libmxml         libxml++  tinyxml     xml-tooling-c   xmlformat   xmlstarlet

```


## Install a package



To install a package, run brew install PACKAGE.


Example, installing jq - JSON processor:


```
$ brew install jq
==> Downloading http://stedolan.github.io/jq/download/source/jq-1.3.tar.gz
==> ./configure
==> make
/home/ubuntu/.linuxbrew/Cellar/jq/1.3: 7 files, 256K, built in 10 seconds
$ which jq
/home/ubuntu/.linuxbrew/bin/jq
$ jq --version
jq version 1.3

```


LinuxBrew’s usefulness is apparent: While Ubuntu has jq in the latest repositories, its version is old (1.2). Debian Stable and Testing don’t have jq package at all. LinuxBrew’s version is the most recent one (1.3). Addionally, LinuxBrew installs the program to a path which will not conflict with the system’s default location.


# Adding Existing HomeBrew Repositories



HomeBrew/LinuxBrew repositories are called TAPS. These are simply GitHub repositories containing Ruby scripts (‘Formulas’). The HomeBrew Githab User has several common repositories.


Example: adding the homebrew-science repository (containing many useful open-source scientific programs) and the HomeBrew-Games repository:


```
$ brew tap homebrew/science
Cloning into '/home/ubuntu/.linuxbrew/Library/Taps/homebrew-science'...
Tapped 237 formula
$ brew tap homebrew/games
Cloning into '/home/ubuntu/.linuxbrew/Library/Taps/homebrew-games'...
Tapped 57 formula

```


List available taps:


```
$ brew tap
homebrew/science
homebrew/games

```


Install any package from those repositories:


```
$ brew install gnu-go
==> Downloading http://ftpmirror.gnu.org/gnugo/gnugo-3.8.tar.gz
#################################################################
==> ./configure --prefix=/home/ubuntu/.linuxbrew/Cellar/gnu-go/3.8 --with-readline=/usr/lib
==> make install
/home/ubuntu/.linuxbrew/Cellar/gnu-go/3.8: 9 files, 7.0M, built in 60 seconds

```


# Updating TAPs and Packages



To download any updates to Formulas, run:


```
$ brew update

```


To upgrade packages (if updates are available), run:


```
$ brew upgrade PACKAGE

```


# Creating Custom/Private TAPs (Repositories)



A HomeBrew TAP/Repository is simply a collection of Formulas – Ruby scripts stored in local files or in GitHub repositories.


## Formulas in local files



To install a formula from a local file, run:


```
$ brew install /full/path/to/file.rb

```


This is useful when creating (and debugging) a new formula.


## Formulas in GitHub repositories



To create a custom TAP repository in github, Create a new GitHub repository (in your user’s github account) and name it homebrew-NAME. It must start with ‘homebrew-’ to work as a HomeBrew/LinuxBrew tap. NAME can be any name you want.


Example:


GitHub user agordon has a HomeBrew repository named gordon, the full URL is: https://github.com/agordon/homebrew-gordon.


To use this repository (“tap it”):


```
$ brew tap agordon/gordon
Cloning into '/home/ubuntu/.linuxbrew/Library/Taps/agordon-gordon'...
Warning: Could not tap agordon/gordon/libestr over Homebrew/homebrew/libestr
Warning: Could not tap agordon/gordon/coreutils over Homebrew/homebrew/coreutils
Tapped 12 formula

```


NOTES


1. 
brew tap used the username agordon and the repository suffix gordon (suffix of ‘homebrew-gordon’) and deduced the github URL to access.

2. 
Formulas in custom repostiries can conflict with formulas in the official HomeBrew repositories. That is perfectly normal. See below on how to install such packages.


To install non-conflicting packages from custom repositories, run:


```
$ brew install libjson

```


To install packages from specific taps, run:


```
$ brew install agordon/gordon/coreutils

```


# More information



NOTE: When reading HomeBrew-related information, keep in mind that HomeBrew was developed for Mac OS X.


LinuxBrew (the linux-port of HomeBrew) have many commonalities with HomeBrew, but also some linux-specific differences.


HomeBrew Wiki


HomeBrew FAQ


HomeBrew Formula Cookbook


HomeBrew Troublehsooting


LinuxBrew WebSite


LinuxBrew Known Issues


<div class=“author”>Submitted by <a href=“https://github.com/agordon”>Assaf Gordon</a></div>


