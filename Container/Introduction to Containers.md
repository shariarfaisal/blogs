# Introduction to Containers

```Container``` ```Docker``` ```Conceptual```

## Introduction


A container is a unit of software that packages code with its required dependencies in order to run in an isolated, controlled environment. This allows you to run an application in a way that is predictable, repeatable, and without the uncertainty of inconsistent code execution across diverse development and production environments. A container will always start up and run code the same way, regardless of what machine it exists on.


Containers provide benefits on multiple levels. In terms of physical allocation of resources, containers provide efficient distribution of compute power and memory. However, containers also enable new design paradigms by strengthening the separation of application code and infrastructure code. This design separation also enables specialization within teams of developers, allowing developers to focus on their strengths.


Considering the breadth of what containers bring to the table, this article will bring you up to speed with the common, related terminology. After a general introduction to the benefits of containers, you will learn the nuance between terms such as “container runtimes”, “container orchestration”, and “container engines”. You will then apply this towards understanding the goals of containers, the specific problems they’re trying to solve, and the current solutions available.



Note:
This article will abstract the feature sets and benefits of containers away from specific products where it makes sense. However, learning how to use containers will ultimately involve learning how to use Docker. Docker containers are not only one of the most popular stand-alone container solutions in their own right, but are also the de facto standard for many container related technologies.
Additionally, the focus here will only be on containers specialized in isolating single application processes, which is the contemporary implementation of containers most people currently associate with container technology. While container technologies such as LXC existed prior to technologies such as Docker, and even acted as the base of early development for parts of Docker, LXC is different enough in design and practical usage to be considered out of the scope of this article.

# Benefits of Containers


Containers are efficient with their consumption of resources — such as CPU compute power and memory — compared to options such as virtual machines. They also offer design advantages both in how they isolate applications and how they abstract the application layer away from the infrastructure layer.


At a project level, this abstraction enables teams and team members to work on their piece of the project without the risk of interfering with other parts of the project. Application developers can focus on application development, while infrastructure engineers can focus on infrastructure. Separating the responsibilities of a team in this case also separates the code, meaning application code won’t break infrastructure code and vice versa.


Containers provide a smoother transition between development and production for teams. For example, if you need to run a Node.js application on a server, one option is to install Node.js directly. This is a straightforward solution when you are a singular developer working on a singular server. However, once you begin collaborating with multiple developers and deploying to multiple environments, that singular installation of Node.js can differ across different team members using even slightly differing development environments.


At a development level, the portability of containers enables infrastructure design that doesn’t have to account for unpredictable application code execution. In this context, containers enable developers to:


- Run an application in the exact same way, across multiple machines. This means containers in local development and production can be reliably predictable, and containers can be shared between developers.
- Run multiple containers as a connected application stack. This means applications can be designed and broken down into microservices as opposed to a monolithic design. For more on this, check out rearchitecting monolithic applications to microservices.

These features may make containers appear similar to virtual machines, but the difference is in their underlying design and subsequent efficiency. Modern container technologies are designed specifically to avoid the heavy resource requirements of virtual machines. While containers share the same principles of portability and repeatability, they are designed at a different level of abstraction. Containers skip the virtualization of hardware and the kernel, which are the most resource intensive parts of a virtual machine, and instead rely on the underlying hardware and kernel of the host machine.


As a result, containers are comparatively lightweight, with lower resource requirements. Container technology has subsequently fostered a rich ecosystem to support container centered development, including:


- Building of container images.
- Storing images in registries and repositories.
- Persistence of data through data volumes.
- Integrated networking solutions.

Strictly speaking, these tools and benefits are auxiliary features on the edges of container technology. However, these are often bundled together as complete container solutions due to the ubiquity of their need.


# Defining Container Terminology


Due to their broad set of use cases, “containers” can refer to multiple things. To help you understand the nuance between the interconnected concepts, here are some key terms that are often confused or used interchangeably in error:


- Operating System: This is the software that manages all other software on your computer, along with the hardware. Often abbreviated as “OS”, an example of this is Linux, which itself has many distributions such as Ubuntu.
- Kernel: This is the component of the OS that specializes in the most basic, low-level interfacing with a machine’s hardware. It translates the requests for physical resources between software processes and the hardware. Resources include compute power from the CPU, memory allocation from RAM, and I/O from the hard disk.
- Containers: This is a unit of software that packages code with its required dependencies in order to run in an isolated, controlled environment. They virtualize an OS, but not a kernel.
- Container runtimes: This manages the start up and existence of a container, along with how a container actually executes code. Regardless of what container solution you choose, you are essentially choosing between different types of container runtimes as a foundation. Furthermore, complete container solutions often use multiple runtimes in conjunction. Container runtimes themselves are divided into two groups:

Open Container Initiative (OCI): This is the baseline runtime that creates and runs a container, with little else. An example is runc, which is the most common, used by Docker, and written in Golang. An alternative is crun from Red Hat written in C, and is meant to be faster and lightweight.
Container Runtime Interface: This runtime is more focused on container orchestration. Examples include the early dockershim which has steadily been dropped in favor of the more fully featured containerd from Docker, or the alternative CRI-O from Red Hat.


- Open Container Initiative (OCI): This is the baseline runtime that creates and runs a container, with little else. An example is runc, which is the most common, used by Docker, and written in Golang. An alternative is crun from Red Hat written in C, and is meant to be faster and lightweight.
- Container Runtime Interface: This runtime is more focused on container orchestration. Examples include the early dockershim which has steadily been dropped in favor of the more fully featured containerd from Docker, or the alternative CRI-O from Red Hat.
- Container Image: These are packages of software required by containers that contain the code, runtime, system libraries, and dependencies. Usually they start from an image of an OS like Ubuntu. These images can be built manually, or can be retrieved from an image registry.

Image Registry: This is a solution for container image storage and sharing. The most prominent example is Docker Hub, but alternative public image registries exist and private image registries can be implemented.
Image Repository: This is a specific location for a container image within an image registry. It contains the latest image along with the history of it within a registry.


- Image Registry: This is a solution for container image storage and sharing. The most prominent example is Docker Hub, but alternative public image registries exist and private image registries can be implemented.
- Image Repository: This is a specific location for a container image within an image registry. It contains the latest image along with the history of it within a registry.
- Container engines: Complete solutions for container technologies, an example being Docker. When people discuss container technology, what they’re often referring to are container engines. This includes the container, the container runtime underlying it, the container image and the tools to build them, and potentially can include container image registries and container orchestration. An alternative to Docker would be a stack of Podman, Buildah, and Skopeo from Red Hat.
- Container Orchestration: Automation of container deployment. Orchestration involves provisioning, configuration, scheduling, scaling, monitoring, deployment, and more. Kubernetes is an example of a popular container orchestration solution.

While these terms all refer to specific aspects of container technology, don’t be surprised if an informal discussion of containers confuses some of these hardline definitions. These definitions are important for your learning path, but be aware that people sometimes use these terms more loosely in everyday conversation.


# Goals of Containers


Container solutions share some common problems that must be addressed in order to be successful:


- Portability: Self-contained applications bundled with dependencies allow containers to be predictable. Code is executed the exact same way in any machine, since containers bring their own repeatable environment.
- Efficiency: Containers not only leverage the existing kernel of their host machine, but often only exist long enough to execute the code they are responsible for. Additionally, containers are often designed to only exist as long as the process or application they run is needed, and stops once a task is complete.
- Statelessness: Statelessness is an ideal in container design that strives for code to always run the exact same way, without requiring knowledge of past or future code executions. However, this isolating approach has been pushed back by real world needs, and has materialized with most container engines providing a solution for persistent data, often in the form of volume storage that can be shared between containers.
- Networking: While containers are often completely isolated from one another, there are many cases where containers must communicate with each other, with the host machine, or with a different cluster of containers. Docker provides both bridge networking for containers that exist under the same instance of Docker to communicate, and overlay networking for containers across different Docker instances.
- Logging: By nature, containers are difficult to inspect. A solution for logging errors and outputs makes large scale container deployments more manageable.
- Application and infrastructure decoupling: Application logic should start and stop within the container responsible for it. The management and deployment of the containers themselves in different environments can be the focus of infrastructure teams.
- Specialization and microservices: Separation of responsibility spread across multiple containers enables the use of microservices instead of monolithic apps.

While containers can offer different solutions, the solutions they offer stem from attempts to answer the same problems. These problems themselves evolve over time along with the needs and expectations of the end user, making this list ever evolving.


# The Container Landscape


Realistically speaking, choosing a container is choosing a container engine. Most container engines use a combination of runc as their OCI runtime in conjunction with containerd as their CRI. As previously mentioned, the current landscape is dominated by Docker. Whether or not you choose Docker as your primary deployment solution, your learning path will likely cross with Docker.


The closest competitor is Red Hat’s solution stack of Podman, which manages the runtime of containers; Buildah, which builds container images; and Skopeo, which is an interface between container images and image registries.


Ultimately, the differences between these solutions hinge on how they handle root access and their usage of daemons. Docker requires root access in order to function, and this is an extra level of access that widens the plane of potential security concern. Docker also requires a daemon to always be always alive. The Podman solution stack requires neither of these things.


# Container Orchestration


The next level of container deployment is automation, and container orchestration automates many steps by always adjusting towards an ideal, defined state across a deployment. As a whole orchestration involves provisioning, configuration, scheduling, scaling, monitoring, deployment, and more.


While a full dive into container orchestration is beyond the scope of this article, two prominent players are Docker with Docker Compose and Docker Swarm mode, and Kubernetes. In roughly order of complexity, Docker Compose is a container orchestration solution that deals with multi-container deployments on a single host. When there are multiple hosts involved, Docker Swarm mode is required.


Kubernetes is a purpose-built container orchestration solution. Whereas Docker’s orchestration solutions are balanced against their focus on the core container components, Kubernetes is scoped for extensibility and granular control in orchestration. This results in a tradeoff where Kubernetes deployments are more complex.


# Conclusion


This article outlined what containers are and what benefits they bring to the table. Container technology is rapidly evolving, and knowing the terminology will be crucial to your learning journey. LIkewise, understanding the goals of containers and the landscape of competitors that led to their creation allows you to make an informed decision on adopting this technology for your specific needs. To learn more about how to set up and use containers, check out the rest of our Cloud Curriculum series on web containers.


## Additional Resources


Tutorials:


- How to Install and Use Docker: A great starting point for learning to work with container technology. Docker containers are a standard across the industry, and this walks you through both installation and basic usage.
- How To Install Docker Compose: If you want to start automating management of your containers, Docker Compose is a good entry point into container orchestration.
- An Introduction to Kubernetes: A more involved solution for container orchestration, Kubernetes gives you scalability and efficiency for large container deployments.

DigitalOcean Products:


- DigitalOcean Marketplace’s Docker One-Click Solution: An alternative to installing Docker yourself manually, this solution starts a DigitalOcean droplet with Docker already installed.
- DigitalOcean Container Registry: A private container registry that integrates with DigitalOcean’s droplets and managed Kubernetes offerings.
- Deploying to Container Images with DigitalOcean App Platform: A fully managed solution to build and scale apps, allowing you to focus on your application’s code without worrying about infrastructure.

