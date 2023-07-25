# Ubuntu and Debian Package Management Essentials

```Linux Basics``` ```Ubuntu``` ```System Tools``` ```Debian```

## Introduction


Package management is one of the fundamental features of a Linux system. The packaging format and the package management tools differ from distribution to distribution, but most distributions use one of two core sets of tools.


For Red Hat Enterprise Linux-based distributions (such as RHEL itself and Rocky Linux), the RPM packaging format and packaging tools like rpm and yum are common. The other major family, used by Debian, Ubuntu, and related distributions, uses the .deb packaging format and tools like apt and dpkg.


In recent years, there have been more auxiliary package managers designed to run in parallel with the core apt and dpkg tooling: for example, snap provides more portability and sandboxing, and Homebrew, ported from macOS, provides command-line tools which can be installed by individual users to avoid conflicting with system packages.


In this guide, you will learn some of the most common package management tools that system administrators use on Debian and Ubuntu systems. This can be used as a quick reference when you need to know how to accomplish a package management task within these systems.


# Prerequisites


- A Ubuntu 20.04 or Debian server and a non-root user with sudo privileges. You can learn more about how to set up a user with these privileges in our Initial Server Setup with Ubuntu 20.04 guide.

# Step 1 – Debian Package Management Tools Overview


The Debian/Ubuntu ecosystem employs quite a few different package management tools in order to manage software on the system.


Most of these tools are interrelated and work on the same package databases. Some of these tools attempt to provide high-level interfaces to the packaging system, while other utilities concentrate on providing low-level functionality.


## apt


The apt command is probably the most often used member of the apt suite of packaging tools. Its main purpose is interfacing with remote repositories maintained by the distribution’s packaging team and performing actions on the available packages.


The apt suite in general functions by pulling information from remote repositories into a cache maintained on the local system. The apt command is used to refresh the local cache. It is also used to modify the package state, meaning to install or remove a package from the system.


In general, apt will be used to update the local cache, and to make modifications to the live system.



Note: In earlier versions of Ubuntu, the core apt command was known as apt-get. It has been streamlined, but you can still call it with apt-get out of habit or for backwards compatibility.

## apt-cache


Another important member of the apt suite is apt-cache. This utility uses the local cache to query information about the available packages and their properties.


For instance, any time you wish to search for a specific package or a tool that will perform a certain function, apt-cache is a good place to start. It can also be informative on what exact package version will be targeted by a procedure. Dependency and reverse dependency information is another area where apt-cache is useful.


## dpkg


While the previous tools were focused on managing packages maintained in repositories, the dpkg command can also be used to operate on individual .deb packages. The dpkg tool actually is responsible for most of the behind-the-scenes work of the commands above; apt provides additional housekeeping while dpkg interacts with the packages themselves.


Unlike the apt commands, dpkg does not have the ability to resolve dependencies automatically. It’s main feature is the ability to work with .deb packages directly, and its ability to dissect a package and find out more about its structure. Although it can gather some information about the packages installed on the system, you should not use it as a primary package manager. In the next step, you’ll learn about package upgrade best practices.


# Step 2 – Updating the Package Cache and the System


The Debian and Ubuntu package management tools help to keep your system’s list of available packages up-to-date. They also provide various methods of updating packages you currently have installed on your server.


## Updating the Local Package Cache


The remote repositories that your packaging tools rely on for package information are updated all of the time. However, most Linux package management tools are designed, for historical reasons, to work directly with a local cache of this information. That cache needs to be periodically refreshed.


It is usually a good idea to update your local package cache every session before performing other package commands. This will ensure that you are operating on the most up-to-date information about the available software. Some installation commands will fail if you are operating with stale package information.


To update the local cache, use the apt command with the update sub-command:


```
sudo apt update


```


This will pull down an updated list of the available packages in the repositories you are tracking.


## Updating Packages


The apt command distinguishes between two different update procedures. The first update procedure (covered in this section) can be used to upgrade any components that do not require component removal. To learn how to update and allow apt to remove and swap components as necessary, see the section below.


This can be very important when you do not want to remove any of the installed packages under any circumstance. However, some updates involve replacing system components or removing conflicting files. This procedure will ignore any updates that require package removal:


```
sudo apt upgrade


```


The second procedure will update all packages, even those that require package removal. This is often necessary as dependencies for packages change.


Usually, the packages being removed will be replaced by functional equivalents during the upgrade procedure, so this is generally safe. However, it is a good idea to keep an eye on the packages to be removed,  in case some essential components are marked for removal. To perform this action, type:


```
sudo apt full-upgrade


```


This will update all packages on your system. In the next step, you’ll learn about downloading and installing new packages.


# Step 3 – Downloading and Installing Packages


## Search for Packages


The first step when downloading and installing packages is often to search your distribution’s repositories for the packages you are looking for.


Searching for packages is one operation that targets the package cache for information. In order to do this, use apt-cache search. Keep in mind that you should ensure that your local cache is up-to-date using sudo apt update prior to searching for packages:


```
apt-cache search package


```


Since this procedure is only querying for information, it does not require sudo privileges. Any search performed will look at the package names, as well as the full descriptions for packages.


For instance, if you search for htop, you will see results like these:


```
apt-cache search htop


```


```
Outputaha - ANSI color to HTML converter
htop - interactive processes viewer
libauthen-oath-perl - Perl module for OATH One Time Passwords

```


As you can see, you have a package named htop, but you can also see two other programs, each of which mention htop in the full description field of the package (the description next to the output is only a short summary).


## Install a Package from the Repos


To install a package from the repositories, as well as all of the necessary dependencies, you can use the apt command with the install argument.


The arguments for this command should be the package name or names as they are labeled in the repository:


```
sudo apt install package


```


You can install multiple packages at once, separated by a space:


```
sudo apt install package1 package2


```


If your requested package requires additional dependencies, these will be printed to standard output and you will be asked to confirm the procedure. It will look something like this:


```
sudo apt install apache2


```


```
OutputReading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  apache2-data
Suggested packages:
  apache2-doc apache2-suexec-pristine apache2-suexec-custom
  apache2-utils
The following NEW packages will be installed:
  apache2 apache2-data
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Need to get 236 kB of archives.
After this operation, 1,163 kB of additional disk space will be used.
Do you want to continue [Y/n]?

```


As you can see, even though our install target was the apache2 package, the apache2-data package is needed as a dependency. In this case, you can continue by pressing Enter or “Y”, or cancel by typing “n”.


## Install a Specific Package Version from the Repos


If you need to install a specific version of a package, you can provide the version you would like to target with =, like this:


```
sudo apt install package=version

```


The version in this case must match one of the package version numbers available in the repository. This means utilizing the versioning scheme employed by your distribution. You can find the available versions by using apt-cache policy package:


```
apt-cache policy nginx


```


```
Outputnginx:
  Installed: (none)
  Candidate: 1.18.0-0ubuntu1.2
  Version table:
     1.18.0-0ubuntu1.2 500
        500 http://mirrors.digitalocean.com/ubuntu focal-updates/main amd64 Packages
        500 http://security.ubuntu.com/ubuntu focal-security/main amd64 Packages
     1.17.10-0ubuntu1 500
        500 http://mirrors.digitalocean.com/ubuntu focal/main amd64 Packages

```


## Reconfigure Packages


Many packages include post-installation configuration scripts that are automatically run after the installation is complete. These often include prompts for the administrator to make configuration choices.


If you need to run through these (and additional) configuration steps at a later time, you can use the dpkg-reconfigure command. This command looks at the package passed to it and re-runs any post-configuration commands included within the package specification:


```
sudo dpkg-reconfigure package


```


This will allow you access to the same (and often more) prompts that you ran upon installation.


## Perform a Dry Run of Package Actions


Many times, you will want to see the side effects of a procedure before without actually committing to executing the command. apt allows you to add the -s flag to “simulate” a procedure.


For instance, to see what would be done if you choose to install a package, you can type:


```
apt install -s package


```


This will let you see all of the dependencies and the changes to your system that will take place if you remove the -s flag. One benefit of this is that you can see the results of a process that would normally require root privileges, without using sudo.


For instance, if you want to evaluate what would be installed with the apache2 package, you can type:


```
apt install -s apache2


```


```
OutputNOTE: This is only a simulation!
      apt needs root privileges for real execution.
      Keep also in mind that locking is deactivated,
      so don't depend on the relevance to the real current situation!
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following extra packages will be installed:
  apache2-data
Suggested packages:
  apache2-doc apache2-suexec-pristine apache2-suexec-custom
  apache2-utils
The following NEW packages will be installed:
  apache2 apache2-data
0 upgraded, 2 newly installed, 0 to remove and 0 not upgraded.
Inst apache2-data (2.4.6-2ubuntu2.2 Ubuntu:13.10/saucy-updates [all])
Inst apache2 (2.4.6-2ubuntu2.2 Ubuntu:13.10/saucy-updates [amd64])
Conf apache2-data (2.4.6-2ubuntu2.2 Ubuntu:13.10/saucy-updates [all])
Conf apache2 (2.4.6-2ubuntu2.2 Ubuntu:13.10/saucy-updates [amd64])

```


You get all of the information about the packages and versions that would be installed, without having to complete the actual process.


This also works with other procedures, like doing system upgrades:


```
apt -s dist-upgrade


```


By default, apt will prompt the user for confirmation for many processes. This includes installations that require additional dependencies, and package upgrades.


In order to bypass these upgrades, and default to accepting any of these prompts, you can pass the -y flag when performing these operations:


```
sudo apt install -y package


```


This will install the package and any dependencies without further prompting from the user. This can be used for upgrade procedures as well:


```
sudo apt dist-upgrade -y


```


## Fix Broken Dependencies and Packages


There are times when an installation may not finish successfully due to dependencies or other problems. One common scenario where this may happen is when installing a .deb package with dpkg, which does not resolve dependencies.


The apt command can attempt to sort out this situation by passing it the -f command.


```
sudo apt install -f


```


This will search for any dependencies that are not satisfied and attempt to install them to fix the dependency tree. If your installation complained about a dependency problem, this should be your first step in attempting to resolve it. If you aren’t able to resolve an issue this way, and you installed a third-party package, you should remove it and look for a newer version that is more actively maintained.


## Download Package from the Repos


There are main instances where it may be helpful to download a package from the repositories without actually installing it. You can do this by running apt with the download argument.


Because this is only downloading a file and not impacting the actual system, no sudo privileges are required:


```
apt download package


```


This will download the specified package(s) to the current directory.


## Install a .deb Package


Although most distributions recommend installing software from their maintained repositories, some vendors supply raw .deb files which you can install on your system.


In order to do this, you use dpkg. dpkg is mainly used to work with individual packages. It does not attempt to perform installs from the repository, and instead looks for .deb packages in the current directory, or the path supplied:


```
sudo dpkg --install debfile.deb


```


It is important to note that the dpkg tool does not implement any dependency handling. This means that if there are any unmet dependencies, the installation will fail. However, it marks the dependencies needed, so if all of the dependencies are available within the repositories, you can satisfy them by typing this afterwards:


```
sudo apt install -f


```


This will install any unmet dependencies, including those marked by dpkg. In the next step, you’ll learn about removing some of the packages you’ve installed.


# Step 4 – Removing Packages and Deleting Files


This section will discuss how to uninstall packages and clean up the files that may be left behind by package operations.


## Uninstall a Package


In order to remove an installed package, you use apt remove. This will remove most of the files that the package installed to the system, with one notable exception.


This command leaves configuration files in place so that your configuration will remain available if you need to reinstall the application at a later date. This is helpful because it means that any configuration files that you customized won’t be removed if you accidentally get rid of a package.


To complete this operation, you need to provide the name of the package you wish to uninstall:


```
sudo apt remove package


```


The package will be uninstalled with the exception of your configuration files.


## Uninstall a Package and All Associated Configuration Files


If you wish to remove a package and all associated files from your system, including configuration files, you can use apt purge.


Unlike the remove command mentioned above, the purge command removes everything. This is useful if you do not want to save the configuration files or if you are having issues and want to start from a clean slate.


Keep in mind that once your configuration files are removed, you won’t be able to get them back:


```
sudo apt purge package


```


Now, if you ever need to reinstall that package, the default configuration will be used.


## Remove Any Automatic Dependencies that are No Longer Needed


When removing packages from your system with apt remove or apt purge, the package target will be removed. However, any dependencies that were automatically installed in order to fulfill the installation requirements will remain behind.


In order to automatically remove any packages that were installed as dependencies that are no longer required by any packages, you can use the autoremove command:


```
sudo apt autoremove


```


If you wish to remove all of the associated configuration files from the dependencies being removed, you will want to add the --purge option to the autoremove command. This will clean up configuration files as well, just like the purge command does for a targeted removal:


```
sudo apt --purge autoremove


```


## Clean Obsolete Package Files


As packages are added and removed from the repositories by a distribution’s package maintainers, some packages will become obsolete.


The apt tool can remove any package files on the local system that are associated with packages that are no longer available from the repositories by using the autoclean command.


This will free up space on your server and remove any potentially outdated packages from your local cache.


```
sudo apt autoclean


```


In the next step, you’ll learn more ways of querying packages without necessarily installing them.


# Step 5 – Getting Information about Packages


Each package contains a large amount of metadata that can be accessed using the package management tools. This section will demonstrate some common ways to get information about available and installed packages.


To show detailed information about a package in your distribution’s repositories, you can use the apt-cache show. The target of this command is a package name within the repository:


```
apt-cache show nginx


```


This will display information about any installation candidates for the package in question. Each candidate will have information about its dependencies, version, architecture, conflicts, the actual package file name, the size of the package and installation, and a detailed description among other things.


```
OutputPackage: nginx
Architecture: all
Version: 1.18.0-0ubuntu1.2
Priority: optional
Section: web
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Original-Maintainer: Debian Nginx Maintainers <pkg-nginx-maintainers@lists.alioth.debian.org>
Bugs: https://bugs.launchpad.net/ubuntu/+filebug
Installed-Size: 44
Depends: nginx-core (<< 1.18.0-0ubuntu1.2.1~) | nginx-full (<< 1.18.0-0ubuntu1.2.1~) | nginx-light (<< 1.18.0-0ubuntu1.2.1~) | nginx-extras (<< 1.18.0-0ubuntu1.2.1~), nginx-core (>= 1.18.0-0ubuntu1.2) | nginx-full (>= 1.18.0-0ubuntu1.2) | nginx-light (>= 1.18.0-0ubuntu1.2) | nginx-extras (>= 1.18.0-0ubuntu1.2)
Filename: pool/main/n/nginx/nginx_1.18.0-0ubuntu1.2_all.deb
…

```


To show additional information about each of the candidates, including a full list of reverse dependencies (a list of packages that depend on the queried package), use the showpkg command instead. This will include information about this package’s relationship to other packages:


```
apt-cache showpkg package


```


## Show Info about a .deb Package


To show details about a .deb file, you can use the --info flag with the dpkg command. The target of this command should be the path to a .deb file:


```
dpkg --info debfile.deb


```


This will show you some metadata about the package in question. This includes the package name and version, the architecture it was built for, the size and dependencies required, a description and conflicts.


To specifically list the dependencies (packages this package relies on) and the reverse dependencies (the packages that rely on this package), you can use the apt-cache utility.


For conventional dependency information, you can use the depends sub-command:


```
apt-cache depends nginx


```


```
Outputnginx
 |Depends: nginx-core
 |Depends: nginx-full
 |Depends: nginx-light
  Depends: nginx-extras
 |Depends: nginx-core
 |Depends: nginx-full
 |Depends: nginx-light
  Depends: nginx-extras

```


This will show information about every package that is listed as a hard dependency, suggestion, recommendation, or conflict.


If you need to find out which packages depend on a certain package, you can pass that package to apt-cache rdepends:


```
apt-cache rdepends package


```


## Show Installed and Available Package Versions


Often, there are multiple versions of a package within the repositories, with a single default package. To see the available versions of a package you can use apt-cache policy:


```
apt-cache policy package


```


This will show you which version is installed (if any), the package that will be installed by default if you do not specify a version with the installation command, and a table of package versions, complete with the weight that indicates each version’s priority.


This can be used to determine what version will be installed and which alternatives are available. Because this also lists the repositories where each version is located, this can be used for determining if any extra repositories are superseding the packages from the default repositories.


## Show Installed Packages with dpkg -l


To show the packages installed on your system, you have a few separate options, which vary in format and verbosity of output.


The first method involves using either the dpkg or the dpkg-query command with the -l flag. The output from both of these commands is identical. With no arguments, it gives a list of every installed or partially installed package on the system. The output will look like this:


```
dpkg -l


```


```
OutputDesired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name                                        Version                                 Architecture Description
+++-===========================================-=======================================-============-=====================================================================================================================
ii  account-plugin-generic-oauth                0.10bzr13.03.26-0ubuntu1.1              amd64        GNOME Control Center account plugin for single signon - generic OAuth
ii  accountsservice                             0.6.34-0ubuntu6                         amd64        query and manipulate user account information
ii  acl                                         2.2.52-1                                amd64        Access control list utilities
ii  acpi-support                                0.142                                   amd64        scripts for handling many ACPI events
ii  acpid                                       1:2.0.18-1ubuntu2                       amd64        Advanced Configuration and Power Interface event daemon
. . .

```


The output continues for every package on the system. At the top of the output, you can see the meanings of the first three characters on each line. The first character indicates the desired state of the package. It can be:


- u: Unknown
- i: Installed
- r: Removed
- p: Purged
- h: Version held

The second character indicates the actual status of the package as known to the packaging system. These can be:


- n: Not installed
- i: Installed
- c: Configuration files are present, but the application is uninstalled.
- u: Unpacked. The files are unpacked, but not configured yet.
- f: The package is half installed, meaning that there was a failure part way through an installation that halted the operation.
- w: The package is waiting for a trigger from a separate package
- p: The package has been triggered by another package.

The third character, which will be a blank space for most packages, only has one potential other option:


- r: This indicates that a re-installation is required. This usually means that the package is broken and in a non-functional state.

The rest of the columns contain the package name, version, architecture, and a description.


## Show Install States of Filtered Packages


If you add a search pattern after the -l pattern, dpkg will list all packages (whether installed or not) that contain that pattern. For instance, you can search for YAML processing libraries here:


```
dpkg -l libyaml*


```


```
OutputDesired=Unknown/Install/Remove/Purge/Hold
| Status=Not/Inst/Conf-files/Unpacked/halF-conf/Half-inst/trig-aWait/Trig-pend
|/ Err?=(none)/Reinst-required (Status,Err: uppercase=bad)
||/ Name            Version      Architecture Description
+++-===============-============-============-===================================
ii  libyaml-0-2:amd 0.1.4-2ubunt amd64        Fast YAML 1.1 parser and emitter li
ii  libyaml-dev:amd 0.1.4-2ubunt amd64        Fast YAML 1.1 parser and emitter li
un  libyaml-perl    <none>                    (no description available)
un  libyaml-syck-pe <none>                    (no description available)
ii  libyaml-tiny-pe 1.51-2       all          Perl module for reading and writing

```


As you can see from the first column, the third and fourth results are not installed. This gives you every package that matches the pattern, as well as their current and desired states.


An alternative way to render the packages that are installed on your system is with dpkg –get-selections.


This provides a list of all of the packages installed or removed but not purged:


```
dpkg --get-selections


```


To differentiate between these two states, you can pipe output from dpkg to  awk in order to filter by state. To see only installed packages, type:


```
dpkg --get-selections | awk '$2 ~ /^install/'


```


To get a list of removed packages that have not had their configuration files purged, you can instead type:


```
dpkg --get-selections | awk '$2 !~ /^install/'


```


You may also want to learn more about piping command output through awk,


## Search Installed Packages


To search your installed package base for a specific package, you can add a package filter string after the --get-selections option. This supports matching with wildcards. Again, this will show any packages that are installed or that still have configuration files on the system:


```
dpkg --get-selections libz*


```


You can, once again, filter using the awk expressions from the last section.


## List Files Installed by a Package


To find out which files a package is responsible for, you can use the -L flag with the dpkg command:


```
dpkg -L package


```


This will print out the absolute path of each file that is controlled by the package. This will not include any configuration files that are generated by processes within the package.


To find out which package is responsible for a certain file in your filesystem, you can pass the absolute path to the dpkg command with the -S flag.


This will print out the package that installed the file in question:


```
dpkg -S /path/to/file


```


Keep in mind that any files that are moved into place by post installation scripts cannot be tied back to the package with this technique.


## Find Which Package Provides a File Without Installing It


Using dpkg, you can find out which package owns a file using the -S option. However, there are times when you may need to know which package provides a file or command, even if you may not have the associated package installed.


To do so, you will need to install a utility called apt-file. This maintains its own database of information, which includes the installation path of every file controlled by a package in the database.


Install the apt-file package as normal:


```
sudo apt update
sudo apt install apt-file


```


Now, update the tool’s database and search for a file by typing:


```
sudo apt-file update
sudo apt-file search /path/to/file


```


This will only work for file locations that are installed directly by a package. Any file that is created through post-installation scripts cannot be queried. In the next step, you’ll learn how to import and export lists of installed packages.


# Step 6 – Transferring Package Lists Between Systems


Many times, you may need to back up the list of installed packages from one system and use it to install an identical set of packages on a different system. This is also helpful for backup purposes. This section will demonstrate how to export and import package lists.


If you need to replicate the set of packages installed on one system to another, you will first need to export your package list.


You can export the list of installed packages to a file by redirecting the output of dpkg --get-selections to a text file:


```
dpkg --get-selections > ~/packagelist.txt


```


You may also want to learn more about input and output redirection.


This list can then be copied to the second machine and imported.


You also may need to back up your sources lists and your trusted key list. You can back up your sources by creating a new directory and copying them over from the system configuration in /etc/apt/:


```
mkdir ~/sources
cp -R /etc/apt/sources.list* ~/sources


```


Any keys which you’ve added in order to install packages from third-party repositories can be exported using apt-key exportall:


```
apt-key exportall > ~/trusted_keys.txt


```


You can now transfer the packagelist.txt file, the sources directory, and the trusted_keys.txt file to another computer to import.


## Import Package List


If you have created a package list using dpkg --get-selections as demonstrated above, you can import the packages on another computer using the dpkg command as well.


First, you need to add the trusted keys and implement the sources lists you copied from the first environment. Assuming that all of the data you backed up has been copied to the home directory of the new computer, you could type:


```
sudo apt-key add ~/trusted_keys.txt
sudo cp -R ~sources/* /etc/apt/


```


Next, clear the state of all non-essential packages from the new computer. This will ensure that you are applying the changes to a clean slate. This must be done with the root account or sudo privileges:


```
sudo dpkg --clear-selections


```


This will mark all non-essential packages for deinstallation. You should update the local package list so that your installation will have records for all of the software you plan to install. The actual installation and upgrade procedure will be handled by a tool called dselect.


You should ensure that the dselect tool is installed. This tool maintains its own database, so you also need to update that before you can continue:


```
sudo apt update
sudo apt install dselect
sudo dselect update


```


Next, you can apply the package list on top of the current list to configure which packages should be kept or downloaded:


```
sudo dpkg --set-selections < packagelist.txt


```


This sets the correct package states. To apply the changes, run apt dselect-upgrade:


```
sudo apt dselect-upgrade


```


This will download and install any necessary packages. It will also remove any packages marked for deselection. In the end, your package list should match that of the previous computer, although configuration files will still need to be copied or modified. You may want to use a tool like etckeeper to migrate configuration files from the /etc directory.


In the next and final step, you’ll learn about working with third party package repositories.


# Step 7 – Adding Repositories and PPAs


Although the default set of repositories provided by most distributions are generally the most maintainable, there are times when additional sources may be helpful. In this section, you’ll learn how to configure your packaging tools to consult additional sources.


An alternative to traditional repositories on Ubuntu are PPAs, or personal package archives. Other Linux flavors typically use different, but similar, concepts of third-party repositories. Usually, PPAs have a smaller scope than repositories and contain focused sets of applications maintained by the PPA owner.


Adding PPAs to your system allows you to manage the packages they contain with your usual package management tools. This can be used to provide more up-to-date packages that are not included with the distribution’s repositories. Take care that you only add PPAs that you trust, as you will be allowing a non-standard maintainer to build packages for your system.


To add a PPA, you can use the add-apt-repository command. The target should include the label ppa:, followed by the PPA owner’s name on Launchpad, a slash, and the PPA name:


```
sudo add-apt-repository ppa:owner_name/ppa_name


```


You may be asked to accept the packager’s key. Afterwards, the PPA will be added to your system, allowing you to install the packages with the normal apt commands. Before searching for or installing packages, make sure to update your local cache with the information about your new PPA:


```
sudo apt update


```


You can also edit your repository configuration directly. You can either edit the /etc/apt/sources.list file or place a new list in the /etc/apt/sources.list.d directory. If you go this latter route, the filename you create must end in .list:


```
sudo nano /etc/apt/sources.list.d/new_repo.list


```


Inside the file, you can add the location of the new repository by using the following format:


/etc/apt/sources.list.d/new_repo.list
```
deb_or_deb-src url_of_repo release_code_name_or_suite component_names

```


The different parts of the repository specification are:


- deb or deb-src: This identifies the type of repository. Conventional repositories are marked with deb, while source repositories begin with deb-src.
- url: The main URL for the repository. This should be the location where the repository can be found.
- release code name or suite: This is usually the code name of your distribution’s release, but it can be whatever name is used to identify a specific set of packages created for your version of the distribution.
- component names: The labels for the selection of packages you wish to have available. This is often a distinction provided by the repository maintainer to express something about the reliability or licensing restrictions of the software it contains.

You can add these lines within the file. Most repositories will contain information about the exact format that should be used. On some other Linux distributions, you can add additional repository sources by actually installing packages which contain only a configuration file for that repository – which is consistent with the way that package managers are designed to work.


# Conclusion


Package management is perhaps the single most important aspect of administering a Linux system. There are many other package management operations that you can perform, but this tutorial has provided a baseline of Ubuntu fundamentals, many of which are generalizable to other distributions with minor changes.


Next, you may want to learn more about package management on other platforms.


