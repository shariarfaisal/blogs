# How To Install Couchbase from source with Git and Make on a Debian 7 VPS

```NoSQL``` ```Git``` ```Debian```

# Introduction



Couchbase is an open-source, distributed key-value based NoSQL database. It comes in two flavors: Enterprise Edition (EE) and Community Edition (CE). Usually first to be released with new updates and bug fixes after testing and QA processes, the Enterprise Edition is the most up-to-date edition of Couchbase. Following suit, the Community Edition gets released shortly afterwards.


In this DigitalOcean guide, we will discuss how Installing from source (compiling) works. We will be making use of various tools that (should) come with your Linux distribution of choice. Here, we are targeting Debian based systems in particular. However, upon simple installation of the other items listed below, you should be able to follow the rest to achieve our goal of Installing Couchbase from source. Installation of the necessary external tools are also explained in this article.


## Installing from source


Building applications on Unix systems can appear scary to some but it is generally easier than you think. Although it should be noted that there are other tools to achieve the same task, we will be using GNU make to build Couchbase here. GNU make is one of the most widespread utilities because it has been built into Unix systems since its introduction in late 70s. It became really popular for its system-independent nature and its ability to combine commands and instructions into a single file, which are referred as makefiles. For more information on make visit http://en.wikipedia.org/wiki/Make_(software).


Many system administrators choose to build software from source as it can help to solve problems caused by deb/rpm (pre-made) packages. It also allows you to customize the installation process, to have multiple versions of the same application on a single system and to use the desired one without worrying about pre-built binaries (compiled files).


## Version Control and Couchbase


Version control systems are used to manage changes to projects’ files, enabling you to track them and their origins. One of the most popular ones is called Git. It is a distributed version control system created to manage the Linux Kernel project itself. Being widely popular, it is used by many others. Our target application, Couchbase, uses Github to host its Git repository online. From there, we checkout a branch (a version of Couchbase) to start working on it. Checking out will download a copy of source code on our local system which then we will build using make.


## To Remember


Different Linux distributions come with different default software packages needed for managing the system. It’s also important to remember that your system’s default package manager won’t be aware of applications installed this way. The responsibility to maintain them (update, upgrade etc.) remains with you.


With all that said, let’s begin!


# Taking Care of the Dependencies and Preparing the System



We’re going to update the default software of our Linux system and enrich its toolbase with the following items:


- Git (version control tool, as mentioned above)
- Repo (complimentary repository management tool, born off Android project)
- Python (an object oriented programming language’s and its C-Based interpreter)
- cURL (a library for downloads and web requests)

In order to update the system, run the following commands on Debian and Ubuntu:


```
 sudo aptitude update
 sudo aptitude upgrade

```


To install Repo application (ref. http://source.android.com/source/downloading.html)


We need to make sure that we have a bin directory to put Repo’s executables. In order to do that, run the following to create it and include it in our path:


```
 mkdir ~/bin
 PATH=~/bin:$PATH

```


Download the Repo tool using cURL tool and ensure that it’s executable (via chmod):


```
 curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
 chmod a+x ~/bin/repo

```


Couchbase, depending on the distribution, will need various libraries and applications, including, but not limited to:


- automake, libtool, pkg-config, check, libssl-dev, sqlite3, libevent-dev, libglib2.0-dev, libcurl4-openssl-dev, erlang-nox, erlang-dev, erlang-src, ruby, libmozjs-dev (Debian), xulrunner-dev (Ubuntu), libicu-dev, libv8-dev, libcloog-ppl0, libsnappy-dev, python-minimal

In order to install these libraries on Debian and Ubuntu, run the following commands successively:


```
 aptitude install -y --without-recommends build-essential automake libtool pkg-config check libssl-dev sqlite3 libevent-dev libglib2.0-dev libcurl4-openssl-dev erlang-nox curl erlang-dev erlang-src ruby libmozjs-dev libicu-dev libv8-dev libcloog-ppl0 libsnappy-dev

 aptitude install -y --without-recommends git-core

```


Please Note (Ubuntu):
Since Ubuntu lacks the libmozjs-dev library, you will also need to install the xulrunner-dev library with the following command in addition to the above:


```
 aptitude install -y xulrunner-dev

```


# Entering the Building Stage



Git:


We now need to let Git know on our local system who we are before checking out the branch of Couchbase from Github. So let’s run the following:


```
 git config --global user.email your@email.addr
 git config --global user.name  your_name

```


Folder preparations and the downloading of the Couchbase source


As of October 2013, Couchbase’s recent releases have the following manifests which we can use to clone the source from:


- rel-2.1.1.xml, rel-2.2.0.xml, rel-2.2.1.xml, rel-3.0.0.xml

Let’s begin with preparing the Couchbase source download and installation folder using mkdir:


```
 mkdir couchbase
 cd couchbase

```


Now we may clone the Couchbase 2.1.1 release using the the manifest file via Repo tool with the init and sync commands:


```
 repo init -u git://github.com/couchbase/manifest.git -m rel-2.1.1.xml
 repo sync

```


# Building From Source



After the above steps, we’re now ready to build source code using make on Debian. However, for Ubuntu, we will need to specify and link with xulrunner library.


Debian:


We can directly run the make command and start the building procedure.


```
 make

```


Ubuntu:


Let’s verify that xulrunner-dev library is installed and check its version with apt-cache for linking with make tool:


```
 apt-cache policy xulrunner-dev

```


And in order to find the location of xulrunner on the system path in /usr/include/ and /usr/lib/, let’s use the list tool with directory information option ls -ld:


```
 ls -ld /usr/lib/xulrunner-devel*
 ls -ld /usr/include/xulrunner*

```


Now we can prepare the make command accordingly, linking with xulrunner folder and its correct version:**


```
 make couchdb_EXTRA_OPTIONS='--with-js-include=/usr/include/xulrunner-17.0 --with-js-lib=/usr/lib/xulrunner-devel-17.0/sdk/lib/'

```


# Finishing



Setting system limits for Couchbase


After the successful build, we need to set limits on our system which Couchbase needs:


```
 ulimit -n 10240 ulimit -c unlimited

```


Running and configuring Couchbase:


We are ready to run the Couchbase server using the binary file we built residing in the /bin directory of the installation folder.


```
 ./install/bin/couchbase-server

```


And to configure the Couchbase installation and join it to clusters, we can access the web console by visiting the port 8091, located at the host address (IP or hostname of our VPS):


```
URL: http://host.addr:8091

```


# Finally



Trouble shooting


For common errors where you might encounter, please visit Couchbase for the troubleshooting common errors page currently located at:


http://www.couchbase.com/docs/couchbase-manual-2.1.0/couchbase-troubleshooting-common-errors.html


For further details about building and manifesting files, please refer to the Couchbase Git repository manifest located at:


https://github.com/couchbase/manifest


<div class=“author”>Submitted by: <a href=“https://twitter.com/ostezer”>O.S. Tezer</div>


