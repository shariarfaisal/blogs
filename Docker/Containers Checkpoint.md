# Containers Checkpoint

```Container``` ```Deployment``` ```Docker``` ```Kubernetes``` ```Networking``` ```Scaling```

## Introduction


This checkpoint is intended to help you assess what you learned from our introductory articles to containers, where we introduced the container ecosystem alongside Docker and Kubernetes, two common container solutions. You can use this checkpoint to assess your knowledge on these topics, review key terms and commands, and find resources for continued learning.


Containerization is the process of isolating a development environment at the operating system level in order to create a portable runtime environment. Containers share resources from the host, enabling you to run your application in a predictable and controlled environment. Using containers to abstract the infrastructure and isolate the application allows you to scale your development processes and testing efficiently and consistently.


In this checkpoint, you’ll find two sections that synthesize the central ideas from the introductory articles: a section summarizing the purposes of a container ecosystem and a second on running commands, each including subsections specific to Docker and Kubernetes, respectively. In each of these sections, there are interactive components to help you test your knowledge. At the end of this checkpoint, you will find opportunities for continued learning about containers.


# Resources


- Introduction to Containers
- How To Install and Use Docker on Ubuntu 22.04
- How To Install and Use Docker Compose on Ubuntu 22.04
- How To Use docker exec to Run Commands in a Docker Container
- How To Share Data between Docker Containers on Ubuntu 22.04
- How To Set Up a Private Docker Registry on Ubuntu 20.04
- An Introduction to Kubernetes
- How To Use minikube for Local Kubernetes Development and Testing

# What Is a Container?


Containers provide a predictable and controlled environment for developing your applications. Container engines are what people typically mean when they refer to a container.


To understand containers, it’s important to have familiarity with several key containerization tools.



Containerization Terms To Know
Define each of the following terms, then use the dropdown feature to check your work.

Container engine
Container engines are a complete solution for containerization, and the engine includes the container, the container runtime, and the container image and the tools to build them. The engine might also include container image registries and container orchestration. Docker is one of the most commonly deployed container engines.


Container image
A container image is a template provided to build the environment within the container. When an image is run, the read-only layer of the image is overlaid with a read-write layer that can be adjusted in the individual container instance.
An image registry is a repository that holds container images. You can use an image registry or repository to manage and share your container images.


Container runtimes
Container runtimes manage the startup and execution of a container. There are two groups of container runtimes: Open Container Initiative (OCI) and Container Runtime Initiative.
The Open Container Initiative provides specifications and standards for container formats. The OCI provides a baseline for running a container, such as the commonly used runc.
The Container Runtime Initiative focuses on container orchestration.


Container orchestration
Orchestration involves provisioning, configuration, scaling, scheduling, deployment, monitoring, and more, and it is generally used to automate container deployment.


Many developers prefer containerization due to its portability and predictable performance. Abstracting the infrastructure makes it possible to test an application in the exact same way across multiple machines. This stability in the baseline environment empowers developers to work collaboratively and remotely.



Check Yourself

What are some common goals when using containers?
To ensure a strong development and production environment, you might want to satisfy these needs:

Predictable and portable performance
Efficiency in usage and memory
Statelessness with an option for persistent data
Isolated containers with the ability for cluster networking
Logging errors and outputs during container testing
Application and infrastructure decoupling
Specialization and microservices



When containers are created from the same base image, they provide a reliable environment that can account for your development and testing needs. This consistency enables you to create and destroy containers as needed during your application’s development.


If you are working on a longer term project that requires persistent data in the container, you can mount a volume for persistent data storage and data sharing across containers.


Kubernetes and Docker are two common container solutions. In the sections below, you can assess what you learned about the differences between them through working with both options in the introductory articles.



Check Yourself

What is the difference between Docker and Kubernetes?
Docker is a container engine used most often to run one or two containerized applications at a time.
Kubernetes is an orchestration software that enables you to run many overlapping containers at scale. Kubernetes originally used the Docker runtime but now primarily uses the containerd abstraction layer.


## What Is Docker?


With Docker, you can manage application processes in containers. Docker is commonly used as a container engine and can be run on a variety of operating systems.


You can map your Docker container onto your host machine by mapping the ports or allowing Docker to select a random, likely unused, port. This process simplifies your testing environment. For more on using Docker, you can review The Docker Ecosystem: An Introduction to Common Components.


Docker uses specific terminology to support its system setup. Assess your knowledge with the Docker terms to know:



Docker Terms to Know
Define each of the following terms, then use the dropdown feature to check your work.

Docker Compose
Docker Compose is a command-line tool that allows you to run multi-container application environments as defined by a YAML file. The YAML configuration file will define the setup, often with a web server environment, port redirection, and a shared volume.


Docker Hub
Docker Hub is a specific image registry managed by Docker, the company behind the Docker project.


Docker Image
A Docker image is a template for building containers. There are many common images available for use.


Docker Volume
A Docker Volume can be used to persist data across containers, including containers that are stopped or deleted.


Swarm
Docker Swarm mode is a method for managing a cluster, which is called a swarm. SwarmKit is one open-source orchestration toolkit built in Go that can be used for a swarm mode cluster.
This is distinct from a previous tool called Docker Swarm, which is now called Swarm Classic, that refers to the now-archived native clustering for Docker. Swarm Classic was an option for large scale clustering.


Docker containers provide a portable and consistent runtime environment, which can often be deployed to Kubernetes clusters.


## What Is Kubernetes?


With Kubernetes, you can run and manage containerized applications and services across a cluster of machines. Kubernetes is used frequently for scaling containerization needs.


Kubernetes uses specific terminology to support its system setup. Assess your knowledge with the Kubernetes terms to know:



Kubernetes Terms to Know
Define each of the following terms, then use the dropdown feature to check your work.

Deployment
A deployment is a Kubernetes workload that uses replication sets for cycle management.
A replication set is used to manage pods, much like a replication controller that defines the pod template for scaling replica pods.


Node
The servers in a cluster are nodes, with one acting as the control plane for the cluster and other servers functioning as worker nodes. Each node includes a container runtime to manage applications and services in containers on that node.


Pod
Containers that are controlled as a single application are grouped as a pod. Pods are created by a replication controller or a replication set.


Your Kubernetes cluster, which consists of a central server and nodes, includes specific components that help the cluster function and the servers to communicate with each other. For a more granular description of the Kubernetes architecture, you can review An Introduction to Kubernetes.


To facilitate networked configuration across servers in a cluster, Kubernetes requires specific components on the control plane that can also be accessed across the nodes, including:


- A key-value store that can be distributed across nodes, such as etcd.
- An API server (kube-apiserver) that can be used to configure workloads and send commands.
- A controller manager (kube-controller-manager) to manage workload, perform tasks, and regulate the cluster.
- A scheduler (kube-scheduler) to assign workloads to specific nodes.
- A cloud controller (cloud-controller-manager) to interact with the cloud provider, resources, and services on behalf of the cluster.

The nodes also need specific components to communicate with the central server and run their assigned workloads, including:


- A container runtime on each node, such as Docker or runc.
- A communication service like kubelet to and from the control plane; kubelet uses a manifest to to define the workload it receives and manage the node’s work state.
- A small proxy, most commonly kube-proxy, to forward requests to containers.

There are command-line tools for both Docker and Kubernetes, each with its own syntax. You can also manage your containers with Docker Desktop or in a Kubernetes Dashboard accessible via a web browser when deployed.


# Using the Command Line


You began to use the Linux command line with our introductory articles on cloud servers, configured a  web server with the articles on web server solutions, and managed your database with articles on databases.


In the introduction to containers, you have continued to develop familiarity with the command line through commands such as:


- cat to display a file.
- chmod to set permissions with a new tool.
- curl to transfer data with a specified location (the URL).
- echo to display a string passed as an argument.
- env to print all environment variables.
- exit to close the interactive Docker shell.
- mkdir to create new directories.
- pwd to print the present working directory.
- systemctl to manage the Docker daemon.
- tail to print the last ten lines of a file.
- watch to run a specific command consistently (every two seconds by default).
- whoami to print the current user’s username.

You have also worked with the unique command syntax for different container engines, and you used the Homebrew package manager to install minikube for your Kubernetes cluster. In the sections below, you will review the commands that you ran in the introductory articles for Docker and Kubernetes.


## Running Docker Commands


In the articles introducing Docker, you installed and managed a Docker container on an Ubuntu server. By default, the docker command can only be run by the root user, by prepending sudo power, or by a user in the docker group.


You used the docker command to pass options, subcommands, and arguments to your Docker container:


- docker exec to run commands in an active container, using the --workdir flag to specify the directory that a command should be run in, the --user flag to run a command as a different user, and the -e flag to pass an environment variable into the container or the --env-file flag to specify a .env file.
- docker images to review the images you have downloaded to your system.
- docker info to access system-wide information.
- docker ps to review active containers running on your system, using the -a switch to view all containers, both active and inactive, and the -l switch to view the latest container you created.
- docker rename to rename your container.
- docker rm with the container ID or name to remove a container.
- docker run to start a container from a specified image, using the combined -it switches for interactive shell access.
- docker search to search for images available on Docker Hub and docker pull to download a specified image from the registry.
- docker start with the container ID or name to start a stopped container.
- docker stop with the container ID or name to stop a running container.
- docker tag to rename a created image.
- docker volume to manage your data volumes.


Check Yourself
Get the answers using the dropdown feature.

What is the difference between the docker and docker exec commands?
The docker exec command is used to run programs and inspect containers that are currently running, whereas you might run docker run to create new containers.


What is a Dockerfile?
You can build images with a Dockerfile using the docker build command, though you did not run that command in these introductory articles. The Dockerfile is used to build images, whereas running the YAML configuration file with docker compose will manage orchestration.


You also used the --help option to access options available to various subcommands, such as the following switches:


- --name to give a name to a container.
- --rm to create a container that removes itself when it’s stopped.
- -d will detach the container from the terminal to run it in the background.
- -v to manage your Docker Volume by naming it or to create a bindmount (when specifying a / or ~/).

Docker also provides the Docker Compose CLI tool for managing multi-container environments. You set up a YAML configuration file to create a web server environment with port redirection and a shared volume, and you ran the following commands to manage your containers:


- docker compose up runs the containerized environment.
- docker compose ps provides information about running containers and port redirection.
- docker compose logs accesses the logs for your container.
- docker compose pause pauses the container and docker compose unpause resumes it.
- docker compose stop will stop the container.
- docker compose down removes the containers, networks, and volumes associated with the environment.


Check Yourself
Get the answers using the dropdown feature.

What is the main difference between using the docker command and running docker compose?
The docker command runs all subcommands in the command line, whereas docker compose runs a YAML file to supply configuration data that can be used for multiple container environments. Containers started with docker compose can also share networks and data volumes.


What makes Docker Compose a beneficial tool to use?
When you have a larger deployment that has multiple containers running in parallel, you can write one YAML file to set up the container configurations and run docker compose to issue commands to all the components and control them as a group. As a result, docker compose helps you scale your container management as your applications grow and you need a strong orchestration setup.


To manage your images with your private Docker Registry and Docker Hub, you ran these commands:


- docker login to log in with the -u switch for your username.
- docker commit to commit a new Docker image with the -m switch to provide a commit message and the -a switch to specify the author.
- docker push to push your images, including to your own repository.
- docker pull to pull your images to a new machine.

For persistent data, you also set up a Docker Volume. You used the docker volume command with the create subcommand to create a new volume, the inspect subcommand to verify the volume on your system, the ls subcommand to list volumes, and :ro appended to the volume name for read-only permissions. You might next share data between the Docker container and the host.


Docker is one common container engine; Kubernetes is an orchestration platform that can run the Docker container engine.


## Running Kubernetes Commands


In the articles introducing Kubernetes, you ran the minikube companion with a Docker framework to simulate a Kubernetes cluster running on a single machine, which enabled you to access the browser dashboard for your Kubernetes cluster.


You ran the following minikube commands:


- minikube start to start the tool and enable kubectl, optionally using the -p or --profile option to specify a cluster.
- minikube dashboard to access the Kubernetes dashboard with automatic port forwarding, using the --url option to aid port forwarding from a remote server via SSH tunneling.
- minikube service with a specified service and the --url option to retrieve a URL for a service that is running.
- minikube config to manage your cluster with subcommands like set memory, get profile,
- minikube delete to delete the service in order to redeploy it.
- minikube mount to mount a directory from your local file system into the cluster temporarily, using the local_path:minikube_host_path syntax to specify which directory and where in the container.
- minikube profile with a specified cluster to switch the active profile.

You also used the kubectl command with the following subcommands:


- kubectl get pods for a list of all the pods running in your cluster, using the -A argument to find all namespaces.
- kubectl create deployment to create a named deployment as a service, using the --image option to call a specified remote image.
- kubectl expose deployment to expose the named deployment, specifying a port with the --port option and the --type option.
- kubectl get service to check if a specified service is running.
- kubectl get nodes to list the active node(s) in the cluster, using the --kubeconfig option to specify a different YAML configuration file.

To get started with a managed Kubernetes service, you can review the DigitalOcean Kubernetes Quickstart documentation.


# What’s Next?


With a stronger understanding of containers and popular database container engines, you can containerize your development environment for consistency when building your applications. That may seem like too simple of an answer for such a complex topic, but you now have a container ecosystem (or two!) in which you can experiment, build apps, and scale projects.


To build out more of your Docker ecosystem, you can follow these tutorials next:


- Containerizing a Node.js Application for Development With Docker Compose
- How To Secure a Containerized Node.js Application with Nginx, Let’s Encrypt, and Docker Compose

If you prefer to get right into the action, try the DigitalOcean Marketplace Docker One-Click Solution, which starts a Droplet with Docker installed. You can also review DigitalOcean product documentation on How to Deploy from Container Images.


To continue developing your Kubernetes cluster, you might review these articles:


- Architecting Applications for Kubernetes
- Modernizing Applications for Kubernetes
- Best Practices for Rearchitecting Monolithic Applications to Microservices
- Our series From Containers to Kubernetes with NodeJS or with Django

If you’d like to free up your resources for development, you can migrate to the DigitalOcean Managed Kubernetes (DOKS) service. You can also use the DigitalOcean Container Registry (DOCR) for additional support with your Docker containers and DigitalOcean Kubernetes clusters.


With your newfound knowledge of containers, you are ready to continue your cloud journey with security measures. If you haven’t yet, check out our introductory articles on cloud servers, web servers, and databases.


