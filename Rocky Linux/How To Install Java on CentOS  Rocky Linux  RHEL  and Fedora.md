# How To Install Java on CentOS  Rocky Linux  RHEL  and Fedora

```Rocky Linux``` ```Java``` ```CentOS``` ```Fedora```

## Introduction


This tutorial will show you how to install Java on current versions of RPM-based Linux distributions: Red Hat Enterprise Linux, CentOS, Fedora, and Rocky Linux. Java is a popular programming language and software platform that allows you to run many server-side applications.


This tutorial covers installing the latest, default version of Java, as well as cherry-picking any older versions for installation, and switching between multiple versions in your environment as needed.


# Prerequisites


Before you begin this guide, you should have a regular, non-root user with sudo privileges configured on your server – this is the user that you should log in to your server as. You can learn how to configure a regular user account by following the steps in our initial server setup guide for Rocky Linux 8.


# Step 1 – Installing OpenJDK


There are three different editions of the Java Platform: Standard Edition (SE), Enterprise Edition (EE), and Micro Edition (ME). This tutorial is focused on Java SE (Java Platform, Standard Edition). Almost all open source Java software is designed to run with Java SE.


There are two different Java SE packages that can be installed: the Java Runtime Environment (JRE) and the Java Development Kit (JDK). JRE is an implementation of the Java Virtual Machine (JVM), which allows you to run compiled Java applications and applets. The JDK includes the JRE as well as other software that is required for writing, developing, and compiling Java applications and applets.


There are also two different implementations of Java: OpenJDK and Oracle Java. Both implementations are based largely on the same code but OpenJDK, the reference implementation of Java, is fully open source while Oracle Java contains some proprietary code. Most Java applications will work fine with either but you should use whichever implementation your software calls for.


You may install various versions and releases of Java on a single system, but most people only need one installation. With that in mind, try to only install the version of Java that you need to run or develop your application(s).


This section will show you how to install the prebuilt OpenJDK JRE and JDK packages using the yum package manager. yum is the default package manager for distributions that use RPM packages.


To install the OpenJDK using yum, you can run sudo yum install java:


```
sudo yum install java


```


By default, trying to install java without specifying a version will resolve to the most common stable version of the OpenJDK JRE. As you can see from this output, as of this writing, that is java-1.8.0-openjdk:


```
OutputLast metadata expiration check: 0:02:38 ago on Tue 22 Feb 2022 04:57:59 PM UTC.
Dependencies resolved.
========================================================================================
 Package                     Arch   Version                             Repo       Size
========================================================================================
Installing:
 java-1.8.0-openjdk          x86_64 1:1.8.0.322.b06-2.el8_5             appstream 341 k
Installing dependencies:
 alsa-lib                    x86_64 1.2.5-4.el8                         appstream 488 k
 atk                         x86_64 2.28.1-1.el8                        appstream 270 k
 avahi-libs                  x86_64 0.7-20.el8                          baseos     61 k
 copy-jdk-configs            noarch 4.0-2.el8                           appstream  29 k
 cups-libs                   x86_64 1:2.2.6-40.el8                      baseos    432 k
 fribidi                     x86_64 1.0.4-8.el8                         appstream  88 k
…

```


Multiple dependencies will also be provided along with Java. At the confirmation prompt, enter y then press Enter to continue with the installation. You may also be prompted to accept signing keys for the repositories you are installing from:


```
OutputImporting GPG key 0x6D745A60:
 Userid     : "Release Engineering <infrastructure@rockylinux.org>"
 Fingerprint: 7051 C470 A929 F454 CEBE 37B7 15AF 5DAC 6D74 5A60
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-rockyofficial
Is this ok [y/N]:

```


Enter y then press Enter again.


You should now have a working Java installation. In order to confirm this, you can run java -version, to check the version of Java that is now available in your environment:


```
java -version


```



Note: Most of the time, command-line arguments are preceded by one dash for single-letter arguments, or two dashes for full-word arguments. Java follows a different convention of using one dash for all arguments, in this case, -version.

```
Outputopenjdk version "1.8.0_322"
OpenJDK Runtime Environment (build 1.8.0_322-b06)
OpenJDK 64-Bit Server VM (build 25.322-b06, mixed mode)

```


The interactions between Java naming conventions and Linux package naming conventions can be somewhat confusing. Earlier in this tutorial, we clarified the difference between the full JDK environment for development, and the JRE environment for running Java applications. Although OpenJDK is the name of the open source distribution of Java, you have only actually installed the OpenJDK JRE. In order to install the full OpenJDK JDK, you should install the corresponding package with -devel appended onto its name. This is a common convention for development packages for other programming environments, which Java also follows, although the terminology overlaps awkwardly here.


As before, you can install java-devel to get the default version, or specify java-1.8.0-openjdk-devel:


```
sudo yum install java-devel


```


```
OutputDigitalOcean Droplet Agent                               63 kB/s | 3.3 kB     00:00
Dependencies resolved.
========================================================================================
 Package                     Arch      Version                       Repository    Size
========================================================================================
Installing:
 java-1.8.0-openjdk-devel    x86_64    1:1.8.0.322.b06-2.el8_5       appstream    9.8 M

Transaction Summary
========================================================================================
Install  1 Package

Total download size: 9.8 M
Installed size: 41 M
Is this ok [y/N]:

```


After installing this package, you should have a full OpenJDK environment which can compile and run any Java software that does not have specific version incompatibilities. In the next section, you’ll install and manage other versions of Java.


# Step 2 – Installing Other OpenJDK Releases


More recently, OpenJDK changed its version numbering scheme to track more closely with Oracle Java releases. In order to install a newer version of OpenJDK, you can specify the version number in the package name, just like with 1.8.0. For example, in order to install OpenJDK 17, you can yum install java-17-openjdk:


```
sudo yum install java-17-openjdk


```


```
OutputLast metadata expiration check: 0:03:36 ago on Tue 22 Feb 2022 05:42:44 PM UTC.
Dependencies resolved.
========================================================================================
 Package                      Arch       Version                    Repository     Size
========================================================================================
Installing:
 java-17-openjdk              x86_64     1:17.0.2.0.8-4.el8_5       appstream     244 k
Installing dependencies:
 adwaita-cursor-theme         noarch     3.28.0-2.el8               appstream     646 k
 adwaita-icon-theme           noarch     3.28.0-2.el8               appstream      11 M
 at-spi2-atk                  x86_64     2.26.2-1.el8               appstream      88 k
 at-spi2-core                 x86_64     2.28.0-1.el8               appstream     168 k
 colord-libs                  x86_64     1.4.2-1.el8                appstream     234 k
 java-17-openjdk-headless     x86_64     1:17.0.2.0.8-4.el8_5       appstream      41 M
 lcms2                        x86_64     2.9-2.el8                  appstream     163 k
…

```


As before, you can install the full JDK environment by appending -devel to the package name. However, after this, running java programs will still use the OpenJDK 1.8.0 version that you installed earlier by default, which you can confirm by running java -version again:


```
java -version


```


```
Outputopenjdk version "1.8.0_322"
OpenJDK Runtime Environment (build 1.8.0_322-b06)
OpenJDK 64-Bit Server VM (build 25.322-b06, mixed mode)

```


In the next step, you’ll manage installed versions of Java.


# Step 3 – Setting Your Default Java Version


If you installed multiple versions of Java, you may want to set one as your default (i.e. the one that will run when a user runs the java command). Additionally, some applications require certain environment variables to be set to locate which installation of Java to use.


The alternatives command, which manages default commands through symbolic links, can be used to select the default Java version. To list the available versions of Java that can be managed by alternatives, use alternatives –config java:


```
sudo alternatives --config java


```


The output should list both versions of Java that you’ve installed:


```
outputThere are 2 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
*+ 1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre/bin/java)
   2           java-17-openjdk.x86_64 (/usr/lib/jvm/java-17-openjdk-17.0.2.0.8-4.el8_5.x86_64/bin/java)

Enter to keep the current selection[+], or type selection number:


```


Enter the a selection number to choose which java executable should be used by default. It will rearrange the necessary symbolic links on your system to ensure that the java command points to the correct set of libraries. You can re-run this command as needed, and the output of java -version should change accordingly:


```
java -version


```


```
Outputopenjdk version "17.0.2" 2022-01-18 LTS
OpenJDK Runtime Environment 21.9 (build 17.0.2+8-LTS)
OpenJDK 64-Bit Server VM 21.9 (build 17.0.2+8-LTS, mixed mode, sharing)

```


Many Java applications also use the JAVA_HOME or JRE_HOME environment variables to determine which java executable to use.


For example, if you installed Java to (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre/bin (i.e. your java executable is located at <^>(/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre/bin/java), you could set your JAVA_HOME environment variable in a bash shell or script like so:


```
export JAVA_HOME=(/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre


```



Note: The JAVA_HOME environment variable prefers that you set the path to your Java installation ending in the /jre directory. This convention can change from one variable to the next, so it’s best to carefully check examples when making changes.

If you want JAVA_HOME to be set for every user on the system by default, add the previous line to the /etc/environment file. You can append it to the file using echo and >> shell redirection, in order to avoid having to edit the /etc/environment file directly, by running this command:


```
sudo sh -c "echo export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre >> /etc/environment"


```


In the next step, you’ll install Oracle’s proprietary Java alongside your OpenJDK versions.


# Step 4 – Installing Oracle Java


This section of the guide will show you how to install Oracle Java JRE and JDK (64-bit), the latest release of these packages at the time of this writing.



Note:
If you are using the interactive terminal on this page, you will not be able to download and install Oracle Java into the environment.

Throughout this section we will be using the wget command to download the Oracle Java software packages. wget may not be included by default on your Linux distribution, so in order to follow along you will need to install it by running:


```
sudo yum install wget


```



You must accept the Oracle Binary Code License Agreement for Java SE, which is one of the included steps, before installing Oracle Java.


Note: In order to install Oracle Java, you will need to go to the Oracle Java Downloads Page, accept the license agreement, and copy the download link of the appropriate Linux x86 .rpm package. Substitute the copied download link in place of the highlighted part of the wget command.

Change to your home directory and download the Oracle Java RPM with these commands:


```
cd ~
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "https://download.oracle.com/java/17/latest/jdk-17_linux-x64_bin.rpm"


```


Then install the RPM with yum localinstall (if you downloaded a different release, substitute the filename here):


```
sudo yum localinstall jdk-17_linux-x64_bin.rpm


```


At the confirmation prompt, enter y then press Enter to continue with the installation.


You may delete the archive file that you downloaded earlier:


```
rm ~/jdk-17_linux-x64_bin.rpm


```


You can now re-run the alternatives command and you should see a third option to use Oracle Java:


```
sudo alternatives --config java


```


```
outputThere are 3 programs which provide 'java'.

  Selection    Command
-----------------------------------------------
   1           java-1.8.0-openjdk.x86_64 (/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.322.b06-2.el8_5.x86_64/jre/bin/java)
 + 2           java-17-openjdk.x86_64 (/usr/lib/jvm/java-17-openjdk-17.0.2.0.8-4.el8_5.x86_64/bin/java)
*  3           /usr/java/jdk-17.0.2/bin/java

Enter to keep the current selection[+], or type selection number:


```


The steps in this tutorial should be sufficient in order to install and run any available version of Java depending on your use case.


# Conclusion


In this tutorial, you installed and managed multiple versions of Java using the yum package manager, the alternatives command, and environment variables. These are all fundamental aspects of Linux environment management, and Java provides an especially good example of working with them because of its many different versions.


Next, you may want to learn how to use Java in other contexts.


