# DigitalOcean Blueprints  Getting Up and Running with Node js  MySQL Replication  and Cachet

```Databases``` ```MySQL``` ```Blueprint``` ```DigitalOcean``` ```DigitalOcean Cloud Firewalls``` ```Block Storage``` ```Node.js``` ```Ansible``` ```Solutions``` ```Terraform``` ```Automated Setups```


Access Blueprints Repositories
Node.js Web Application
MySQL Group Replication with ProxySQL
Status Page Application

## Introduction


DigitalOcean Blueprints provide you with fast and flexible infrastructure to support you as you scale. You can leverage and incorporate Blueprints as part of your development workflow to spend more time crafting code and less time setting up your infrastructure.


# What are Blueprints?


DigitalOcean Blueprints offer automated multi-server infrastructure setups. The goal of each Blueprint is to give developers a way to streamline the infrastructure setup process so they can spend more time bringing ideas and projects to life.


Blueprints can be the foundation of a project or a component in a multi-server environment. As a starting point for further work, Blueprints leave configuration and content creation within developers’ hands while giving them a tool for getting started quickly.


# Available Blueprints


Each Blueprint uses Terraform and Ansible to create an infrastructure setup with DigitalOcean products that addresses a different use case:


- 
Node.js Web Application: This Blueprint can be used to set up a two-node infrastructure with Nginx, Node.js, and MongoDB. The web and application layers are deployed on one server, while the database is located on the other. Data from the database is stored on a block storage device, and Cloud Firewalls are configured in front of each server to regulate traffic.

- 
MySQL Group Replication with ProxySQL: This Blueprint provides a replicated database group setup using MySQL group replication and ProxySQL. The cloned setup creates a three-node replication database layer to handle project data. It also creates a ProxySQL server that is configured to manage queries and changes to the project’s backend.

- 
Status Page Application: This Blueprint creates a status page using Cachet, an open-source status page application, and a two-node infrastructure. One of the two servers runs MySQL, while the other runs the Cachet application with Nginx and PHP-FRM. The two servers communicate over DigitalOcean’s private network, and customizable Cloud Firewalls are in place to further restrict access. Nginx is also configured with SSL/TLS certificates using Let’s Encrypt.


Each of these Blueprints can lay the groundwork for various use cases and provide a pattern that can be modified based on your needs.


# How To Use Blueprints


Each Blueprint will be ready to clone and use after a few prerequisites are in place. You will need:


- Docker installed on your local machine or on a control Droplet. To install Docker locally, you can follow the community edition download guidelines. If you would prefer to use a control Droplet, you can get started quickly with the DigitalOcean Docker One-Click Application.
- Git installed locally. If you are using the Docker One-Click image on a control Droplet, then Git will already be installed.
- A DigitalOcean account and API Key.

With these prerequisites in place, you will be able to take the following steps to get each Blueprint up and running:


1. 
Clone the repository.

2. 
Configure definitions and credentials for the Docker image and local repository.

3. 
Create your infrastructure.


From here, you will be able to customize your infrastructure and adapt it to your needs and use cases.


# Next Steps


A good first step in putting the Blueprints to use will be to read each project’s README.md in full. There, you will find detailed instructions for installation, as well as discussions of how to test, customize, and deprovision your infrastructure.



Access Blueprints Repositories
Node.js Web Application
MySQL Group Replication with ProxySQL
Status Page Application

