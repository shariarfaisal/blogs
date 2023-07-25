# Upgrading Debian to Unstable

```Debian```

## Introduction


This article will guide you through the process of upgrading your fresh installation of Debian to Unstable (Sid) version.


Don't let the name fool you.  While Debian Unstable might sometimes happen to introduce some bugs or regressions with it's updates, it surely isn't as 'Unstable' as name would suggest.  Debian Unstable is made mostly of stable releases of software packages, not development versions as one might think at first.


Adding further to this, regular Ubuntu releases are based on Debian Unstable and their LTS (Long Term Support) releases are based on Debian Testing.


Biggest benefits of upgrading to Unstable are:


- Fresh versions of OS base
- Fresh versions of packages (software, libraries, etc.)
- Latest bugfixes and security updates

Before you proceed:


Please be aware that the steps in this tutorial should only be performed on a clean installation of Debian.  We are also going to show how to enable 'contrib' and 'non-free' repositories.  You can read about them here.  Additionally, if you wish to enable those repositories, please read "Step 2" carefully.


# Step One


Before you get to the upgrading process, you should determine if it's safe for you to do so right now.  Because Debian Unstable is often updated, it might happen that it's repository is currently being under maintenance and some packages may fail to install.  To help you determine whether it is safe or not to perform the upgrade, you can check out
Debian Weather to see if you can proceed further.


When you confirm that you are safe to proceed, you can start by logging into your server as root.


# Step Two


Open the configuration file in editor:


```
nano /etc/apt/sources.list
```


The file should look similar to this:


```
deb http://ftp.us.debian.org/debian squeeze main
deb http://security.debian.org/ squeeze/updates main
```


For the first line, change "squeeze main" at the end of the line with "sid main".


```
deb http://ftp.us.debian.org/debian sid main
```


Then, replace the entire second line with the following command:


```
deb http://ftp.us.debian.org/debian squeeze main
```


Now, if you want to enable 'contrib' and 'non-free' repositories, add a third line and insert 'contrib non-free'.


```
deb http://ftp.us.debian.org/debian sid main contrib non-free
```


To save your changes, press:


```
Ctrl+O
```


Then confirm with:


```
Enter/Return
```


Finally, close the editor by pressing:


```
Ctrl+X
```


# Step Three


Update your package lists by typing:


```
apt-get update
```


Now, to upgrade, here are the following instructions:


Run:


```
apt-get dist-upgrade
```


Then confirm that you want to continue with the process and let it run.  Don't hide the terminal as you will likely encounter a few questions during the upgrade.  If this dialog window appears:


```
 ----------------------------| Configuring libc6 |----------------------------
 |                                                                           | 
 | There are services installed on your system which need to be restarted    | 
 | when certain libraries, such as libpam, libc, and libssl, are upgraded.   | 
 | Since these restarts may cause interruptions of service for the system,   | 
 | you will normally be prompted on each upgrade for the list of services    | 
 | you wish to restart.  You can choose this option to avoid being           | 
 | prompted; instead, all necessary restarts will be done for you            | 
 | automatically so you can avoid being asked questions on each library      | 
 | upgrade.                                                                  | 
 |                                                                           | 
 | Restart services during package upgrades without asking?                  | 
 |                                                                           | 
 |                    <Yes>                       <No>                       | 
 |                                                                           | 
 -----------------------------------------------------------------------------
```


Select <Yes>.


For the rest of dialogs and questions, if you don't know what to select, just keep the default selections and continue.


Sometimes during an upgrade process, an error will pop-up, usually about that there is an unmet dependency or process is locked.  If you encounter this, try to restart the upgrading process by issuing:


```
apt-get dist-upgrade
```


If you are still having issues with the upgrade process, enter:


```
apt-get -f install
```


Then run the update command again.


```
apt-get dist-upgrade
```


If you encounter any more errors, try to use methods provided above again to see if it will solve them.  If problems continue to persist and mentioned methods fail, it might be easier to rebuild your server and restart the procedure - that's why the process should be performed on a clean install.


Once the process have been successfully completed, reboot your server.


```
reboot
```


Give it some time to reboot and reconnect.  If everything goes well, you are now running Debian Unstable!


