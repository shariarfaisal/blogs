# How to Use Ansible to Install and Set Up Docker on Ubuntu 18 04

```Docker``` ```Automated Setups``` ```Ansible``` ```Ubuntu 18.04``` ```Configuration Management```

## Introduction


Server automation now plays an essential role in systems administration, due to the disposable nature of modern application environments. Configuration management tools such as Ansible are typically used to streamline the process of automating server setup by establishing standard procedures for new servers while also reducing human error associated with manual setups.


Ansible offers a simple architecture that doesn’t require special software to be installed on nodes. It also provides a robust set of features and built-in modules which facilitate writing automation scripts.


This guide explains how to use Ansible to automate the steps contained in our guide on How To Install and Use Docker on Ubuntu 18.04. Docker is an application that simplifies the process of managing containers, resource-isolated processes that behave in a similar way to virtual machines, but are more portable, more resource-friendly, and depend more heavily on the host operating system.


# Prerequisites


In order to execute the automated setup provided by the playbook in this guide, you’ll need:


- One Ansible control node: an Ubuntu 18.04 machine with Ansible installed and configured to connect to your Ansible hosts using SSH keys. Make sure the control node has a regular user with sudo permissions and a firewall enabled, as explained in our Initial Server Setup guide. To set up Ansible, please follow our guide on How to Install and Configure Ansible on Ubuntu 18.04.
- One or more Ansible Hosts: one or more remote Ubuntu 18.04 servers previously set up following the guide on How to Use Ansible to Automate Initial Server Setup on Ubuntu 18.04.


Before proceeding, you first need to make sure your Ansible control node is able to connect and execute commands on your Ansible host(s). For a connection test, check Step 3 of How to Install and Configure Ansible on Ubuntu 18.04.

# What Does this Playbook Do?


This Ansible playbook provides an alternative to manually running through the procedure outlined in our guide on How To Install and Use Docker on Ubuntu 18.04. Set up your playbook once, and use it for every installation after.


Running this playbook will perform the following actions on your Ansible hosts:


1. Install aptitude, which is preferred by Ansible as an alternative to the apt package manager.
2. Install the required system packages.
3. Install the Docker GPG APT key.
4. Add the official Docker repository to the apt sources.
5. Install Docker.
6. Install the Python Docker module via pip.
7. Pull the default image specified by default_container_image from Docker Hub.
8. Create the number of containers defined by the container_count variable, each using the image defined by default_container_image, and execute the command defined in default_container_command in each new container.

Once the playbook has finished running, you will have a number of containers created based on the options you defined within your configuration variables.


To begin, log into a sudo enabled user on your Ansible control node server.


# Step 1 — Preparing your Playbook


The playbook.yml file is where all your tasks are defined. A task is the smallest unit of action you can automate using an Ansible playbook. But first, create your playbook file using your preferred text editor:


```
nano playbook.yml


```


This will open an empty YAML file. Before diving into adding tasks to your playbook, start by adding the following:


playbook.yml
```
---
- hosts: all
  become: true
  vars:
    container_count: 4
    default_container_name: docker
    default_container_image: ubuntu
    default_container_command: sleep 1

```


Almost every playbook you come across will begin with declarations similar to this. hosts declares which servers the Ansible control node will target with this playbook. become states whether all commands will be done with escalated root privileges.


vars allows you to store data in variables. If you decide to change these in the future, you will only have to edit these single lines in your file. Here’s a brief explanation of each variable:


- container_count: The number of containers to create.
- default_container_name: Default container name.
- default_container_image: Default Docker image to be used when creating containers.
- default_container_command: Default command to run on new containers.


Note: If you want to see the playbook file in its final finished state, jump to Step 5. YAML files can be particular with their indentation structure, so you may want to double-check your playbook once you’ve added all your tasks.

# Step 2 — Adding Packages Installation Tasks to your Playbook


By default, tasks are executed synchronously by Ansible in order from top to bottom in your playbook. This means task ordering is important, and you can safely assume one task will finish executing before the next task begins.


All tasks in this playbook can stand alone and be re-used in your other playbooks.


Add your first tasks of installing aptitude, a tool for interfacing with the Linux package manager, and installing the required system packages. Ansible will ensure these packages are always installed on your server:


playbook.yml
```
  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true

```


Here, you’re using the apt Ansible builtin module to direct Ansible to install your packages. Modules in Ansible are shortcuts to execute operations that you would otherwise have to run as raw bash commands. Ansible safely falls back onto apt for installing packages if aptitude is not available, but Ansible has historically preferred aptitude.


You can add or remove packages to your liking. This will ensure all packages are not only present, but on the latest version, and done after an update with apt is called.


# Step 3 — Adding Docker Installation Tasks to your Playbook


Your task will install the latest version of Docker from the official repository. The Docker GPG key is added to verify the download, the official repository is added as a new package source, and Docker will be installed. Additionally, the Docker module for Python will be installed as well:


playbook.yml
```
    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Install Docker Module for Python
      pip:
        name: docker

```


You’ll see that the apt_key and apt_repository built-in Ansible modules are first pointed at the correct URLs, then tasked to ensure they are present. This allows installation of the latest version of Docker, along with using pip to install the module for Python.


# Step 4 — Adding Docker Image and Container Tasks to your Playbook


The actual creation of your Docker containers starts here with the pulling of your desired Docker image. By default, these images come from the official Docker Hub. Using this image, containers will be created according to the specifications laid out by the variables declared at the top of your playbook:


playbook.yml
```
    - name: Pull default Docker image
      docker_image:
        name: "{{ default_container_image }}"
        source: pull

    - name: Create default containers
     .docker_container:
        name: "{{ default_container_name }}{{ item }}"
        image: "{{ default_container_image }}"
        command: "{{ default_container_command }}"
        state: present
      with_sequence: count={{ container_count }}

```


docker_image is used to pull the Docker image you want to use as the base for your containers. docker_container allows you to specify the specifics of the containers you create, along with the command you want to pass them.


with_sequence is the Ansible way of creating a loop, and in this case it will loop the creation of your containers according to the count you specified. This is a basic count loop, so the item variable here provides a number representing the current loop iteration. This number is used here to name your containers.


# Step 5 — Reviewing your Complete Playbook


Your playbook should look roughly like the following, with minor differences depending on your customizations:


playbook.yml
```
---
- hosts: all
  become: true
  vars:
    container_count: 4
    default_container_name: docker
    default_container_image: ubuntu
    default_container_command: sleep 1d

  tasks:
    - name: Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true

    - name: Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu bionic stable
        state: present

    - name: Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Install Docker Module for Python
      pip:
        name: docker

    - name: Pull default Docker image
      docker_image:
        name: "{{ default_container_image }}"
        source: pull

    - name: Create default containers
      docker_container:
        name: "{{ default_container_name }}{{ item }}"
        image: "{{ default_container_image }}"
        command: "{{ default_container_command }}"
        state: present
      with_sequence: count={{ container_count }}


```


Feel free to modify this playbook to best suit your individual needs within your own workflow. For example, you could use the docker_image module to push images to Docker Hub or the docker_container module to set up container networks.



Note: This is a gentle reminder to be mindful of your indentations. If you run into an error, this is very likely the culprit. YAML suggests using 2 spaces as an indent, as was done in this example.

Once you’re satisfied with your playbook, you can exit your text editor and save.


# Step 6 — Running your Playbook


You’re now ready to run this playbook on one or more servers. Most playbooks are configured to be executed on every server in your inventory by default, but you’ll specify your server this time.


To execute the playbook only on server1, connecting as sammy, you can use the following command:


```
ansible-playbook playbook.yml -l server1 -u sammy


```


The -l flag specifies your server and the -u flag specifies which user to log into on the remote server. You will get output similar to this:


```
Output. . .
changed: [server1]

TASK [Create default containers] *****************************************************************************************************************
changed: [server1] => (item=1)
changed: [server1] => (item=2)
changed: [server1] => (item=3)
changed: [server1] => (item=4)

PLAY RECAP ***************************************************************************************************************************************
server1              	: ok=9	changed=8	unreachable=0	failed=0	skipped=0	rescued=0	ignored=0   



```



Note: For more information on how to run Ansible playbooks, check our Ansible Cheat Sheet Guide.

This indicates your server setup is complete! Your output doesn’t have to be exactly the same, but it is important that you have zero failures.


When the playbook is finished running, log in via SSH to the server provisioned by Ansible to check if the containers were successfully created.


Log in to the remote server with:


```
ssh sammy@your_remote_server_ip


```


And list your Docker containers on the remote server:


```
sudo docker ps -a


```


You should see output similar to this:


```
OutputCONTAINER ID    	IMAGE           	COMMAND         	CREATED         	STATUS          	PORTS           	NAMES
a3fe9bfb89cf    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker4
8799c16cde1e    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker3
ad0c2123b183    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker2
b9350916ffd8    	ubuntu          	"sleep 1d"      	5 minutes ago   	Created                             	docker1

```


This means the containers defined in the playbook were created successfully. Since this was the last task in the playbook, it also confirms that the playbook was fully executed on this server.


# Conclusion


Automating your infrastructure setup can not only save you time, but it also helps to ensure that your servers will follow a standard configuration that can be customized to your needs. With the distributed nature of modern applications and the need for consistency between different staging environments, automation like this has become a central component in many teams’ development processes.


In this guide, you demonstrated how to use Ansible to automate the process of installing and setting up Docker on a remote server. Because each individual typically has different needs when working with containers, we encourage you to check out the official Ansible documentation for more information and use cases of the docker_container Ansible module.


If you’d like to include other tasks in this playbook to further customize your initial server setup, please refer to our introductory Ansible guide Configuration Management 101: Writing Ansible Playbooks.


