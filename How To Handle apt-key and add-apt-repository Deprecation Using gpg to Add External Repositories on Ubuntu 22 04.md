# How To Handle apt-key and add-apt-repository Deprecation Using gpg to Add External Repositories on Ubuntu 22 04

```Linux Basics``` ```Linux Commands``` ```Ubuntu 22.04```

## Introduction


apt-key is a utility used to manage the keys that APT uses to authenticate packages. It’s closely related to the add-apt-repository utility, which adds external repositories using keyservers to an APT installation’s list of trusted sources. However, keys added using apt-key and add-apt-repository are trusted globally by apt. These keys are not limited to authorizing the single repository they were intended for. Any key added in this manner can be used to authorize the addition of any other external repository, presenting an important security concern.


Starting with Ubuntu 20.10, the use of apt-key yields a warning that the tool will be deprecated in the near future; likewise, add-apt-repository will also soon be deprecated. While these deprecation warnings do not strictly prevent the usage of apt-key and add-apt-repository with Ubuntu 22.04, it is not advisable to ignore them.


The current best practice is to use gpg in place of apt-key and add-apt-repository, and in future versions of Ubuntu it will be the only option. apt-key and add-apt-repository themselves have always acted as wrappers, calling gpg in the background. Using gpg directly cuts out the intermediary. For this reason, the gpg method is backwards compatible with older versions of Ubuntu and can be used as a drop-in replacement for apt-key.


This tutorial will outline two procedures that use alternatives to apt-key and add-apt-repository, respectively. First will be adding an external repository using a public key with gpg instead of using apt-key. Second, as an addendum, this tutorial will cover adding an external repository using a keyserver with gpg as an alternative to using add-apt-repository.


# Prerequisites


To complete this tutorial, you will need an Ubuntu 22.04 server. Be sure to set this up according to our initial server setup guide for Ubuntu 22.04, with a non-root user with sudo privileges and a firewall enabled.


# Step 1 — Identifying the Components and Key Format


PGP, or Pretty Good Privacy, is a proprietary encryption program used for signing, encrypting, and decrypting files and directories. PGP files are public key files, which are used in this process to authenticate repositories as valid sources within apt. GPG, or GNU Privacy Guard, is an open-source alternative to PGP. GPG files are usually keyrings, which are files that hold multiple keys. Both of these file types are commonly used to sign and encrypt files.


gpg is GPG’s command line tool that can be used to authorize external repositories for use with apt. However, gpg only accepts GPG files. In order to use this command line tool with PGP files, you must convert them.


Elasticsearch presents a common scenario for key conversion, and will be used as the example for this section. You’ll download a key formatted for PGP and convert it into an apt compatible format with a .gpg file extension. You’ll do this by running the gpg command with the --dearmor flag. Next, you’ll add the repository link to the list of package sources, while attaching a direct reference to your converted key. Finally, you will verify this process by installing the Elasticsearch package.


Projects that require adding repositories with key verification will always provide you with a public key and a repository URI representing its exact location. For our Elasticsearch example, the documentation gives these components on their installation page.


Here are the components given for Elasticsearch:


- Key: https://artifacts.elastic.co/GPG-KEY-elasticsearch
- Repository: https://artifacts.elastic.co/packages/7.x/apt stable main

Next, you have to determine whether you are given a PGP or GPG file to work with. You can inspect at the key file by opening the URL with curl:


```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch


```


This will output the contents of the key file, which starts with the following:


```
Output-----BEGIN PGP PUBLIC KEY BLOCK-----
. . .

```


Despite having GPG in the URL, the first line indicates that this is actually a PGP key file. Take note of this, because apt only accepts the GPG format. Originally, apt-key detected PGP files and converted it into GPG automatically by calling gpg in the background. Step 2 will cover both manual conversion from PGP to GPG, and also what to do when conversion is not needed.


# Step 2  — Downloading the Key and Converting to an apt Compatible File Type


With the gpg method, you must always download the key before adding to the list of package sources. Previously with apt-key, this ordering was not always enforced. Now, you are required to reference the path to the downloaded key file in your sources list. If you have not downloaded the key, you obviously cannot reference an existing path.


With Elasticsearch you are working with a PGP file, so you will convert it to a GPG file format after download. The following example uses curl to download the key, with the download being piped into a gpg command. gpg is called with the --dearmor flag to convert the PGP key into a GPG file format, with -o used to  indicate the file output.


On Ubuntu, the /usr/share/keyrings directory is the recommended location for your converted GPG files, as it is the default location where Ubuntu stores its keyrings. The file is named elastic-7.x.gpg in this example, but any name works:


```
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /usr/share/keyrings/elastic-7.x.gpg


```


This converts the PGP file into the correct GPG format, making it ready to be added to the list of sources for apt.



Note: If the downloaded file was already in a GPG format, you could instead download the file straight to /usr/share/keyrings without converting it using a command like the following example:
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo tee /usr/share/keyrings/elastic-7.x.gpg


In this case, the curl command’s output would be piped into tee to save the file in the correct location.

# Step 3  — Adding the Repository to Your List of Package Sources


With the key downloaded and in the correct GPG file format, you can add the repository to the apt packages source while explicitly linking it to the key you obtained. There are three methods to achieve this, all of which are related to how apt finds sources. apt pulls sources from a central sources.list file, .list files in the sources.list.d directory, and .source files in the sources.list.d directory. Though there is no functional difference between the three options, it is recommended to consider the three options and choose the method that best fits your needs.


## Option 1 — Adding to sources.list Directly


The first method involves inserting a line representing the source directly into /etc/apt/sources.list, the primary file containing apt sources. There are multiple sources in this file, including the default sources that come with Ubuntu. It is perfectly acceptable to edit this file directly, though Option 2 and Option 3 will present a more modular solution that can be easier to edit and maintain.


Open /etc/apt/sources.list with nano or your preferred text editor:


```
sudo nano /etc/apt/sources.list


```


Then add the external repository to the bottom of the file:


/etc/apt/sources.list
```
. . .
deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/elastic-7.x.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main

```


This line contains the following information about the source:


- deb: This specifies that the source uses a regular Debian architecture.
- arch=amd64,arm64 specifies the architectures the APT data will be downloaded to. Here it is amd64 and arm64.
- signed-by=/usr/share/keyrings/elastic-7.x.gpg: This specifies the key used to authorize this source, and here it points towards your .gpg  file stored in /usr/share/keyrings. This portion of the line must be included, while it previously wasn’t required in the apt-key method. This addition is the most critical change in porting away from apt-key, since it ties the key to a singular repository it is allowed to authorize and fixes the original security flaw in apt-key.
- https://artifacts.elastic.co/packages/7.x/apt stable main: This is the URI representing the exact location the data within the repository can be found.
- /etc/apt/sources.list.d/elastic-7.x.list: This is the location and name of the new file to be created.
- /dev/null: This is used when the output of a command is not necessary. Pointing tee to this location omits the output.

Save and exit by hitting CTRL+O then CTRL+X.


## Option 2 — Creating a New .list File in sources.list.d


With this option, you will instead create a new file in the sources.list.d directory. apt parses both this directory and sources.list for repository additions. This method allows you to physically isolate repository additions within separate files. If you ever need to later remove this addition or make edits, you can delete this file instead of editing the central sources.list file. Keeping your additions separate makes it easier to maintain, and editing sources.list can be more error prone in a way that affects other repositories in the file.


To do this, pipe an echo command into a tee command to create this new file and insert the appropriate line. The file is named elastic-7.x.list in the following example, but any name works as long as it is a unique filename in the directory:


```
echo "deb [arch=amd64,arm64 signed-by=/usr/share/keyrings/elastic-7.x.gpg] https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-7.x.list > /dev/null


```


This command is identical to manually creating the file and inserting the appropriate line of text.


## Option 3 — Creating a .sources File in sources.list.d


The third method writes to a .sources file instead of a .list file. This method is relatively new, and uses the deb822 multiline format that is less ambiguous compared to the deb . . . declaration, though is functionally identical. Create a new file:


```
sudo nano /etc/apt/sources.list.d/elastic-7.x.sources


```


Then add the external repository using the deb822 format:


/etc/apt/sources.list.d/elastic-7.x.sources
```
Types: deb
Architectures: amd64 arm64
Signed-By: /usr/share/keyrings/elastic-7.x.gpg
URIs: https://artifacts.elastic.co/packages/7.x/apt
Suites: stable
Components: main

```


Save and exit after you’ve inserted the text.


This is analogous to the one-line format, and doing a line-by-line comparison shows that the  information in both is identical, just organized differently. One thing to note is that this format doesn’t use commas when there are multiple arguments (such as with amd64,arm64), and instead uses spaces.


Next you will verify this process by doing a test installation.


# Step 4  — Installing the Package from the External Repository


You must call apt update in order to prompt apt to look through the main sources.list file, and all the .list and .sources files in sources.list.d. Calling apt install without an update first will cause a failed install, or an installation of an out-of-date default package from apt.


Update your repositories:


```
sudo apt update


```


Then install your package:


```
sudo apt install elasticsearch


```


Nothing changes in this step compared to the apt-key method. Once this command finishes you will have completed the installation.


# Addendum - Adding an External Repository Using a Keyserver


This section will briefly go over using gpg with a keyserver instead of a public key to add an external repository. The process is nearly identical to the public key method, with the difference being how gpg is called.


add-apt-repository is the keyserver based counterpart to apt-key, and both are up for deprecation. This scenario uses different components. Instead of a key and repository, you are given a keyserver URL and key ID. In this case, you can download from the keyserver directly into the appropriate .gpg format without having to convert anything. Because add-apt-repository will soon be deprecated, you will instead use gpg to download to a file while overriding the default gpg behavior of importing to an existing keyring.


Using the open-source programming language R as an example, here are the given components, which can also be found in the installation instructions on the official project site:


- Keyserver: keyserver.ubuntu.com
- Key ID: E298A3A825C0D65DFD57CBB651716619E084DAB9
- Repository: https://cloud.r-project.org/bin/linux/ubuntu jammy-cran40/

First, download from the keyserver directly using gpg. Be aware that depending on download traffic, this download command may take a while to complete:


```
sudo gpg --homedir /tmp --no-default-keyring --keyring /usr/share/keyrings/R.gpg --keyserver keyserver.ubuntu.com --recv-keys E298A3A825C0D65DFD57CBB651716619E084DAB9


```


This command includes the following flags, which are different from using gpg with a public key:


- --no-default-keyring combined with  --keyring allows outputting to a new file instead of importing into an existing keyring, which is the default behavior of gpg in this scenario.
- --keyserver combined with --recv-keys provides the specific key and location you’re downloading from.
- --homedir is used to overwrite the gpg default location for creating temporary files. gpgneeds to create these files to complete the command, otherwise gpg will attempt to write to /root which causes a permission error. Instead, this command places the temporary files in the appropriate /tmp directory.

Next, add the repository to a .list file. This is done in the exact same manner as adding an external repository using a public key by piping an echo command into a tee command:


```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/R.gpg] https://cloud.r-project.org/bin/linux/ubuntu jammy-cran40/" | sudo tee /etc/apt/sources.list.d/R.list > /dev/null


```


Next, update your list of repositories:


```
sudo apt update


```


Then you can install the package:


```
sudo apt install r-base


```


Using gpg to add external repositories is similar between public keys and keyservers, with the difference being how you call gpg.


# Conclusion


Adding an external repository using a public key or a keyserver can be done through gpg, without using apt-key or add-apt-repository as an intermediary. Use this method to ensure your process does not become obsolete in future Ubuntu versions, as apt-key and add-apt-repository are deprecated and will be removed in a future version. Adding external repositories using gpg ensures that a key will only be used to authorize a single repository as you intend.


