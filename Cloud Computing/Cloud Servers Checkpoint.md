# Cloud Servers Checkpoint

```Cloud Computing``` ```Linux Basics```

## Introduction


This checkpoint is intended to help you assess what you have learned from our introductory articles to Cloud Servers, where we introduced cloud computing, cloud servers, and the Linux command line. You can use this checkpoint to assess your knowledge on these topics, review key terms and commands, and find resources for continued learning.


Cloud computing typically uses virtualization for hosting needs. This abstraction from physical hardware (often on-premises) means that projects can be built and maintained at scale without making large financial and time investments to maintain your own hardware. Once you have learned the fundamentals of cloud servers, you will be ready to begin exploring other key concepts and technologies of cloud computing, such as databases, containers, web servers, and security.


In this checkpoint, you’ll find three sections that synthesize the central topics across the articles in the Cloud Servers section: defining the cloud and its delivery models, using the Linux command line, and using SSH with remote servers. You can test your knowledge with interactive components. At the end of this checkpoint, you will find opportunities for continued learning and Linux server management.


## Resources


- Cloud Servers: An Introduction.
- A General Introduction to Cloud Computing.
- Initial Server Setup.
- A Linux Command Line Primer.
- SSH Essentials: Working with SSH Servers, Clients, and Keys.
- How to Choose a Linux Distribution.
- An Introduction to Cloud Hosting.

# What Is The Cloud?


Cloud computing is the delivery of computing resources as a service, meaning that the resources are owned and managed by the cloud provider rather than the end user. You’ve likely used the cloud to watch streaming media, store your personal data such as photos and files, or even create your own web apps or other projects.


In A General Introduction to Cloud Computing, you learned about cloud computing as it is defined by the National Institute of Standards and Technology (NIST), a non-regulatory agency of the United States Department of Commerce.



Check Yourself
What are the five essential characteristics of cloud computing?

Get the answers with the dropdown feature.
NIST defines the following as the five key principles of cloud computing:

On-demand self-service
Broad network access
Resource pooling
Rapid elasticity
Measured service



These characteristics are relevant across all types of cloud environments: public cloud, private cloud, hybrid cloud, and multicloud.


## Terms to Know


Through each of the articles, you have developed a vocabulary with common terms related to cloud computing.



Check Yourself
Define each of the following terms, then use the dropdown feature to check your work.

Server
A server is the computer hardware or software that can run services to other computers and that allows client machines to function.
See the glossary entry for server for a longer definition.


Virtual Private Server
A virtual private server, or VPS, is a virtual server that emulates a real computer with its own operating system. The software on a virtual machine is allocated by a host and disconnected from the computer’s hardware.
They are sometimes called virtual machines, or VMs. When hosted in the cloud, they are sometimes called cloud servers or remote servers.


Virtualization
Virtualization is a process that abstracts computer environments from physical hardware, making it possible to host in the cloud. This process facilitates the relationship between the virtual servers that apps and websites can be hosted on and the physical hosts that manage the virtual servers.


Hypervisor
A hypervisor is software that deploys, manages, and grants resources to virtual servers under its control. The physical hardware that a hypervisor is running on is referred to as a host. The hypervisor shares the host’s resources among various guest VMs.
An Introduction to Cloud Hosting specifies four common hypervisors available today. Can you name them?
See the glossary entry for hypervisor for a longer definition.


Kernel
The kernel is the foundation of a computer’s operating system. The kernel facilitates memory allocation as well as device and resource management.
See the glossary entry for kernel for a longer definition.


## Cloud Delivery


You can also identify how cloud resources are provided through delivery models such as Infrastructure as a Service (IaaS), Platform as a Service (PaaS), and Software as a Service (SaaS).


IaaS provides complete control over your infrastructure without maintaining your own hardware. Benefits include flexible hosting, scaling with demand, and building across multiple datacenters.


With PaaS, you use deployment platforms on your cloud provider’s back-end infrastructure. Benefits include predictable scaling, pre-configured runtime environments, and a simplified experience with API integrations.


SaaS provides software applications in cloud environments. You can access the software but not its production, maintenance, or modification. As a result, users can use the platform directly without needing to install or maintain software on their device.



Check Yourself
Match the following products to their delivery model:



Adobe Creative Cloud
DigitalOcean Managed Databases like MongoDB and MySQL
Microsoft Azure




AWS Elastic Beanstalk
Heroku
Streaming services like Netflix and Spotify




DigitalOcean App Platform
Managed Kubernetes on DigitalOcean
Virtual communication tools like Google Workspace, Slack, and Zoom




Compare your answers using the dropdown feature.



Delivery Model
Product




IaaS
Managed Kubernetes on DigitalOcean, Managed Databases (like MongoDB and MySQL), Microsoft Azure, and more


PaaS
AWS Elastic Beanstalk, DigitalOcean App Platform, Heroku, and more


SaaS
Adobe Creative Cloud, Google Workspace, Netflix, Slack, Spotify, Zoom, and more





You can now explain what the cloud is and describe why it has become ubiquitous in the modern day. You know the benefits and considerations necessary when building projects in the cloud, as well as which types of projects are available in which cloud delivery model. To build projects in the cloud, many developers use Linux-based virtual machines.


# Using the Command Line


In A Linux Command Line Primer, you began the journey to love your terminal. With the initial server setup, you configured a Linux environment with SSH, a firewall configured with ufw, a package manager, and a non-root user with sudo privileges.


You can now navigate the command line interface (CLI) on both your local machine and a remote server using commands such as:


- cat to review file contents.
- cd to move between directories.
- curl to transfer data using URL syntax.
- echo to display strings of text.
- ls to list files.
- mkdir to make new folders.
- mv to move or rename files.
- nano to create and edit text files.
- pwd to review your current working directory path.
- rm and rmdir to delete files and folders.
- sudo to run commands as a superuser.
- usermod to change user permissions.

And options (also known as flags or switches) like:


- -a to list all files, including hidden files.
- -h or --human-readable to print memory sizes in a human readable format.
- -l to print extra details about files.
- -o to output text to a file.
- -r to run commands recursively.

You can review all the commands you have run in your terminal with the history command. You can also use the man command in Linux to display user manuals or the --help flag to review additional information about any command.


Once you’ve chosen your Linux distribution, you can explore tutorials in the Linux Basics series, manage processes on your Linux server, and otherwise monitor your server resources. If you are running a Linux-based remote server, you will use SSH to access and perform operations on the remote server from your local terminal.


# Using SSH


The Secure Shell protocol (SSH) allows you to log in to a remote server and run command line execution from an unsecured network.


In SSH Essentials, you generated an SSH key pair to connect to your remote server. The SSH key provides a secure access credential when using SSH login. Your keys are stored in an authorized key file, usually in the /.ssh directory in each user’s home directory.


Alongside ssh and ssh-keygen, you can use the rsync (remote sync) and scp (secure copy program) commands for file transfer between systems. During your initial server setup, you used rsync to copy files between users, but you can also use it to copy files between systems.



Check Yourself
What is the difference between scp and rsync?

Review the answer with the dropdown feature.
Both scp and rsync copy files: scp between hosts on a network using SSH; rsync on the local host or bidirectionally between the local host and a remote host. Both encrypt file transfer when used with SSH, and rsync is known for its delta-transfer algorithm, which results in optimized transfer speed.
With scp, you select which files and directories are to be transferred, whereas rsync transfers all files and directories initially and then only the changed files and directories. You can use additional options with rsync, such as the --archive, --verbose, and --compress flags.
The Secure File Transfer Protocol (sftp) is another option for file transfer, but it is not used frequently these days as scp and rsync are more flexible.


# What’s Next?


You can host a cloud server on a DigitalOcean Droplet. Once you’re familiar with the fundamentals of Linux, you can try securing your VPS and setting up Fail2ban to protect your server. You’ll also want to decide which package management system to use.


If you’d like to develop your Linux skills further, follow these tutorials:


- How To Use Find and Locate to Search for Files on Linux
- Understanding Systemd Units and Unit Files
- Using Grep & Regular Expressions to Search for Text Patterns in Linux
- How To Use Bash’s Job Control to Manage Foreground and Background Processes
- Command Line Basics: Symbolic Links
- How To Partition and Format Storage Devices in Linux
- How To Add Swap Space on Ubuntu 22.04

If you run into issues as you work, you can also troubleshoot common site issues.


With your newfound cloud knowledge, you are well-equipped to continue your cloud journey with web servers, databases, containers, and security.


