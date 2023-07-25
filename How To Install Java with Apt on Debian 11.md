# How To Install Java with Apt on Debian 11

```Debian``` ```Debian 11``` ```Java```

## Introduction


Java and the JVM (Java Virtual Machine) are required for many kinds of software, including Tomcat, Jetty, Glassfish, Cassandra and Jenkins.


In this guide, you will install different versions of the Java Runtime Environment (JRE) and the Java Developer Kit (JDK) using apt. You’ll install OpenJDK as well as the official JDK from Oracle. Then, you’ll select the version you wish to use for your projects. When you’re finished, you’ll be able to use the JDK to develop software or use the Java Runtime to run software.


# Prerequisites


To follow this tutorial, you will need:


- One Debian 11 server with a non-root, sudo-enabled user. You can set this up by following our Debian 11 initial server setup guide.

# Step 1 — Installing Java


Installing Java comes with two main components. The JDK provides essential software tools to develop in Java, such as a compiler and debugger. The JRE is used to actually execute Java programs. Furthermore, there are two main installation options of Java to choose from. OpenJDK is the open-source implementation of Java and comes packaged with Debian. Oracle JDK is the original version of Java and is fully maintained by Oracle, the developers of Java.


Both of these versions are officially recognized by Oracle. Both are also developed by Oracle, but OpenJDK has the addition of community contributions due to its open-source nature. However, starting with Java 11 the two options are now functionally identical as detailed by Oracle. The choice between which to install comes down to choosing the appropriate licensing for your circumstance. Additionally, OpenJDK has the option to install the JRE separately, while OracleJDK comes packaged with its JRE.


## Option 1—Installing the Default JRE/JDK


One option for installing Java is to use the version packaged with Debian. By default, Debian 11 includes OpenJDK version 11, which is an open-source variant of the JRE and JDK, and is compatible with Java 11.


Java 11 is the current Long Term Support version of Java.


To install the OpenJDK version of Java, first update your apt package index:


```
sudo apt update


```


Next, check if Java is already installed:


```
java -version


```


If Java is not currently installed, you’ll receive the following message:


```
Output-bash: java: command not found

```


Execute the following command to install the default JRE from OpenJDK 11:


```
sudo apt install default-jre


```


The JRE will allow you to run almost all Java software.


Verify the installation with the following:


```
java -version


```


The following output will return:


```
Outputopenjdk version "11.0.16" 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8-post-Debian-1deb11u1)
OpenJDK 64-Bit Server VM (build 11.0.16+8-post-Debian-1deb11u1, mixed mode, sharing)

```


You may need the JDK in addition to the JRE in order to compile and run some specific Java-based software. To install the JDK, execute the following command, which will also install the JRE:


```
sudo apt install default-jdk


```


Verify that the JDK is installed by checking the version of javac, the Java compiler:


```
javac -version


```


You’ll receive the following output:


```
Outputjavac 11.0.16

```


Next, you will learn how to install Oracle’s official JDK and JRE.


## Option 2 — Installing Oracle JDK 11


Oracle’s licensing agreement for Java doesn’t allow automatic installation through package managers. To install the official Oracle JDK, you must create an Oracle account and manually download the JDK to add a new package repository for the version you’d like to use. Then you can use apt to install it with help from a third-party installation script. Oracle JDK comes with the JRE included, so you don’t need to install that separately.


The version of Oracle’s JDK you need to download must match the version of the installer script. To find what version you need, visit the oracle-java11-installer page. The location of your package is in the following figure:





In this image, the version of the script is 11.0.13. Therefore, you need Oracle JDK 11.0.13. The version number may vary depending on when you’re installing the software.
You don’t need to download anything from this page since you’ll be downloading the installation script through apt shortly.


Next, visit the Archive Downloads  and locate the version that matches the one you need.





From this list, choose the Linux x64 compressed archive .tar.gz package:





You’ll be presented with a screen asking you to accept the Oracle license agreement. Select the checkbox to accept the license agreement and press the Download button. Your download will begin. You may need to log in to your Oracle account one more time before the download starts.


Once the file has downloaded, you’ll need to transfer it to your server. On your local machine, upload the file to your server. On macOS, Linux, or Windows using the Windows Subsystem for Linux, use the scp command to transfer the file to the home directory of your sammy user. The following command assumes you’ve saved the Oracle JDK file to your local machine’s Downloads folder:


```
scp Downloads/jdk-11.0.13_linux-x64_bin.tar.gz sammy@your_server_ip:~


```


Once the file upload is complete, return to your server and add the third-party repository that will help you install Oracle’s Java.


First, import the signing key used to verify the software you’re about to install:


```
sudo gpg --homedir /tmp --no-default-keyring --keyring /usr/share/keyrings/oracle-jdk11-installer.gpg --keyserver keyserver.ubuntu.com --recv-keys EA8CACC073C3DB2A


```


You’ll receive the following output:


```
Outputgpg: keybox '/usr/share/keyrings/oracle-jdk11-installer.gpg' created
gpg: /tmp/trustdb.gpg: trustdb created
gpg: key EA8CACC073C3DB2A: public key "Launchpad PPA for Linux Uprising" imported
gpg: Total number processed: 1
gpg:           	imported: 1

```


Next, add the repository to your list of package sources:


```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/oracle-jdk11-installer.gpg] https://ppa.launchpadcontent.net/linuxuprising/java/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/oracle-jdk11-installer.list > /dev/null


```


Update your package list to make the new software available for installation:


```
sudo apt update


```


The installer will look for the Oracle JDK you downloaded in /var/cache/oracle-jdk11-installer-local. First create this directory:


```
sudo mkdir -p /var/cache/oracle-jdk11-installer-local/


```


Then, move the Oracle JDK archive there:


```
sudo cp jdk-11.0.13_linux-x64_bin.tar.gz /var/cache/oracle-jdk11-installer-local/


```


Finally, install the package:


```
sudo apt install oracle-java11-installer-local


```


The installer will first ask you to accept the Oracle license agreement. Accept the agreement, and then the installer will extract the Java package and install it.


Now, you will learn how to select the version of Java you want to use.


# Step 2—Managing Java


You can have multiple Java installations on one server. You can configure which version is the default for use on the command line by using the update-alternatives command:


```
sudo update-alternatives --config java


```


This is the following output if you’ve installed both versions of Java in this tutorial:


```
OutputThere are 2 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                         Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   1111      manual mode
* 2            /usr/lib/jvm/java-11-oracle/bin/java          1091      manual mode

Press <enter> to keep the current choice[*], or type selection number:

```


Choose the number associated with the Java version to use it as the default, or press ENTER to leave the current settings in place.


You can do this for other Java commands, such as the compiler (javac):


```
sudo update-alternatives --config javac


```


Other commands for which this command can be run include, but are not limited to: keytool, javadoc, and jarsigner.


# Step 3 — Setting the JAVA_HOME Environment Variable


Many programs written in Java use the JAVA_HOME environment variable to determine which Java installation location.


To set this environment variable,  first determine where Java is installed. Use the update-alternatives command:


```
sudo update-alternatives --config java


```


This output will list each installation of Java along with its installation path:


```
Output  Selection    Path                                         Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   1111      auto mode
  1            /usr/lib/jvm/java-11-openjdk-amd64/bin/java   1111      manual mode
* 2            /usr/lib/jvm/java-11-oracle/bin/java          1091      manual mode

```


In this case, the installation paths are as follows:


- Oracle Java 11 is located at /usr/lib/jvm/java-11-oracle/bin/java.
- OpenJDK 11 is located at /usr/lib/jvm/java-11-openjdk-amd64/bin/java.

These paths show the path to the java executable.


Next, copy the path for your preferred installation, excluding the trailing bin/java component. Then open /etc/environment using nano or your preferred text editor:


```
sudo nano /etc/environment


```


This file may be blank initially. At the end of the file, add the following line, making sure to replace the highlighted path with your own copied path, and not include the bin/ portion of the path:


/etc/environment
```
JAVA_HOME="/usr/lib/jvm/java-11-openjdk-amd64"

```


Modifying this file will set the JAVA_HOME path for all users on your system.


Save the file and exit the editor. If you’re using nano, you can do this by pressing CTRL + X, Y, then ENTER.


Now reload this file to apply the changes to your current session:


```
source /etc/environment


```


Verify that the environment variable is set:


```
echo $JAVA_HOME


```


Your output will return the path you previously set:


```
Output/usr/lib/jvm/java-11-openjdk-amd64

```


Other users will need to execute the command source /etc/environment or log out and log back in to apply this setting.


# Conclusion


In this tutorial, you installed multiple versions of Java and learned how to manage them. Now you can install software that runs on Java, such as Tomcat, Jetty, Glassfish, Cassandra, or Jenkins.


