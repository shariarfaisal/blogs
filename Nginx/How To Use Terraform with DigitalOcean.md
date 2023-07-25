# How To Use Terraform with DigitalOcean

```Terraform``` ```System Tools``` ```API``` ```Nginx``` ```Load Balancing``` ```DigitalOcean Managed Load Balancers``` ```DNS``` ```Configuration Management```

## Introduction


Terraform is a tool for building and managing infrastructure in an organized way. You can use it to manage DigitalOcean Droplets, Load Balancers, and even DNS entries, in addition to a large variety of services offered by other providers. Terraform uses a command-line interface and can run from your desktop or a remote server.


Terraform works by reading configuration files that describe the components that make up your application environment or datacenter. Based on the configuration, it generates an execution plan that describes what it will do to reach the desired state. You then use Terraform to execute this plan to build the infrastructure. When changes to the configuration occur, Terraform can generate and execute incremental plans to update the existing infrastructure to the newly described state.


In this tutorial, you’ll install Terraform and use it to create an infrastructure on DigitalOcean that consists of two Nginx servers that are load balanced by a DigitalOcean Load Balancer. Then, you’ll use Terraform to add a DNS entry on DigitalOcean that points to your Load Balancer. This will help you get started with using Terraform, and give you an idea of how you can use it to manage and deploy a DigitalOcean-based infrastructure that meets your own needs.



Note: This tutorial has been tested with Terraform 1.1.3.

# Prerequisites


To complete this tutorial, you’ll need:


- A DigitalOcean account. If you do not have one, sign up for a new account.
- A DigitalOcean Personal Access Token, which you can create via the DigitalOcean control panel. Instructions to do that can be found at: How to Create a Personal Access Token.
- A password-less SSH key added to your DigitalOcean account, which you can create by following How To Use SSH Keys with DigitalOcean Droplets. When you add the key to your account, remember the name you give it, as you’ll need it in this tutorial. (For Terraform to accept the name of your key, it must start with a letter or underscore and may contain only letters, digits, underscores, and dashes.)
- A personal domain pointed to DigitalOcean’s nameserver, which you can do by following the tutorial, How To Point to DigitalOcean Nameservers From Common Domain Registrars.

# Step 1 — Installing Terraform


Terraform is a command-line tool that you run on your desktop or on a remote server. To install it, you’ll download it and place it on your PATH so you can execute it in any directory you’re working in.


First, download the appropriate package for your OS and architecture from the official Downloads page. If you’re on macOS or Linux, you can download Terraform with curl.


On macOS, use this command to download Terraform and place it in your home directory:


```
curl -o ~/terraform.zip https://releases.hashicorp.com/terraform/1.1.3/terraform_1.1.3_darwin_amd64.zip 


```


On Linux, use this command:


```
curl -o ~/terraform.zip https://releases.hashicorp.com/terraform/1.1.3/terraform_1.1.3_linux_amd64.zip 


```


Create the ~/opt/terraform directory:


```
mkdir -p ~/opt/terraform


```


Then, unzip Terraform to ~/opt/terraform using the unzip command. On Ubuntu, you can install unzip using apt:


```
sudo apt install unzip


```


Use it to extract the downloaded archive to the ~/opt/terraform directory by running:


```
unzip ~/terraform.zip -d ~/opt/terraform


```


Finally, add ~/opt/terraform to your PATH environment variable so you can execute the terraform command without specifying the full path to the executable.


On Linux, you’ll need to redefine PATH in .bashrc, which runs when a new shell opens. Open it for editing by running:


```
nano ~/.bashrc


```



Note: On macOS, add the path to the file .bash_profile if using Bash, or to .zshrc if using ZSH.

To append Terraform’s path to your PATH, add the following line at the end of the file:


.bashrc
```
export PATH=$PATH:~/opt/terraform

```


Save and close the file when you’re done.


Now all of your new shell sessions will be able to find the terraform command. To load the new PATH into your current session, run the following command if you’re using Bash on a Linux system:


```
. ~/.bashrc


```


If you’re using Bash on macOS, execute this command instead:


```
. .bash_profile


```


If you’re using ZSH, run this command:


```
. .zshrc


```


To verify that you have installed Terraform correctly, run the terraform command with no arguments:


```
terraform


```


You will see output that is similar to the following:


```
OutputUsage: terraform [global options] <subcommand> [args]

The available commands for execution are listed below.
The primary workflow commands are given first, followed by
less common or more advanced commands.

Main commands:
  init          Prepare your working directory for other commands
  validate      Check whether the configuration is valid
  plan          Show changes required by the current configuration
  apply         Create or update infrastructure
  destroy       Destroy previously-created infrastructure

All other commands:
  console       Try Terraform expressions at an interactive command prompt
  fmt           Reformat your configuration in the standard style
  force-unlock  Release a stuck lock on the current workspace
  get           Install or upgrade remote Terraform modules
  graph         Generate a Graphviz graph of the steps in an operation
  import        Associate existing infrastructure with a Terraform resource
  login         Obtain and save credentials for a remote host
  logout        Remove locally-stored credentials for a remote host
  output        Show output values from your root module
  providers     Show the providers required for this configuration
  refresh       Update the state to match remote systems
  show          Show the current state or a saved plan
  state         Advanced state management
  taint         Mark a resource instance as not fully functional
  test          Experimental support for module integration testing
  untaint       Remove the 'tainted' state from a resource instance
  version       Show the current Terraform version
  workspace     Workspace management

Global options (use these before the subcommand, if any):
  -chdir=DIR    Switch to a different working directory before executing the
                given subcommand.
  -help         Show this help output, or the help for a specified subcommand.
  -version      An alias for the "version" subcommand.

```


These are the commands that Terraform accepts. The output gives you a brief description, and you’ll learn more about them throughout this tutorial.


Now that Terraform is installed, let’s configure it to work with DigitalOcean’s resources.


# Step 2 — Configuring Terraform for DigitalOcean


Terraform supports a variety of service providers through providers you can install. Each provider has its own specifications, which generally map to the API of its respective service provider.


The DigitalOcean provider lets Terraform interact with the DigitalOcean API to build out infrastructure. This provider supports creating various DigitalOcean resources, including the following:


- digitalocean_droplet: Droplets (servers)
- digitalocean_loadbalancer: Load Balancers
- digitalocean_domain: DNS domain entries
- digitalocean_record: DNS records

Terraform will use your DigitalOcean Personal Access Token to communicate with the DigitalOcean API and manage resources in your account. Don’t share this key with others, and keep it out of scripts and version control. Export your DigitalOcean Personal Access Token to an environment variable called DO_PAT by running:


```
export DO_PAT="your_personal_access_token"


```


This will make using it in subsequent commands easier and keep it separate from your code.



Note: If you’ll be working with Terraform and DigitalOcean often, add this line to your shell configuration files using the same approach you used to modify your PATH environment variable in the previous step.

Create a directory that will store your infrastructure configuration by running the following command:


```
mkdir ~/loadbalance


```


Navigate to the newly created directory:


```
cd ~/loadbalance


```


Terraform configurations are text files that end with the .tf file extension. They are human-readable and they support comments. (Terraform also supports JSON-format configuration files, but they won’t be covered here.) Terraform will read all of the configuration files in your working directory in a declarative manner, so the order of resource and variable definitions do not matter. Your entire infrastructure can exist in a single configuration file, but you should separate the configuration files by resource type to maintain clarity.


The first step to building an infrastructure with Terraform is to define the provider you’re going to use.


To use the DigitalOcean provider with Terraform, you have to tell Terraform about it and configure the plugin with the proper credential variables. Create a file called provider.tf, which will store the configuration for the provider:


```
nano provider.tf


```


Add the following lines into the file to tell Terraform that you want to use the DigitalOcean provider, and instruct Terraform where to find it:


~/loadbalance/provider.tf
```
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

```


Then, define the following variables in the file so you can reference them in the rest of your configuration files:


- do_token: your DigitalOcean Personal Access Token.
- pvt_key: private key location, so Terraform can use it to log in to new Droplets and install Nginx.

You will pass the values of these variables into Terraform when you run it, rather than by hard-coding the values here. This makes the configuration more portable.


To define these variables, add these lines to the file:


~/loadbalance/provider.tf
```
...
variable "do_token" {}
variable "pvt_key" {}

```


Then, add these lines to configure the DigitalOcean provider and specify the credentials for your DigitalOcean account by assigning the do_token to the token argument of the provider:


~/loadbalance/provider.tf
```
...
provider "digitalocean" {
  token = var.do_token
}

```


Finally, you’ll want to have Terraform automatically add your SSH key to any new Droplets you create. When you added your SSH key to DigitalOcean, you gave it a name. Terraform can use this name to retrieve the public key. Add these lines, replacing terraform with the name of the key you provided in your DigitalOcean account:


~/loadbalance/provider.tf
```
...
data "digitalocean_ssh_key" "terraform" {
  name = "terraform"
}

```


Your completed provider.tf file will look like this:


~/loadbalance/provider.tf
```
terraform {
  required_providers {
    digitalocean = {
      source = "digitalocean/digitalocean"
      version = "~> 2.0"
    }
  }
}

variable "do_token" {}
variable "pvt_key" {}

provider "digitalocean" {
  token = var.do_token
}

data "digitalocean_ssh_key" "terraform" {
  name = "terraform"
}

```


When you’re done, save and close the file.



Note: Setting the TF_LOG environment variable to 1 will enable detailed logging of what Terraform is trying to do. You can set it by running:
export TF_LOG=1



Initialize Terraform for your project by running:


```
terraform init


```


This will read your configuration and install the plugins for your provider. You’ll see that logged in the output:


```
OutputInitializing the backend...

Initializing provider plugins...
- Finding digitalocean/digitalocean versions matching "~> 2.0"...
- Installing digitalocean/digitalocean v2.16.0...
- Installed digitalocean/digitalocean v2.16.0 (signed by a HashiCorp partner, key ID F82037E524B9C0E8)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

```


If you happen to get stuck, and Terraform is not working as you expect, you can start over by deleting the terraform.tfstate file and manually destroying the resources that were created (e.g., through the control panel).


Terraform is now configured and can be connected to your DigitalOcean account. In the next step, you’ll use Terraform to define a Droplet that will run an Nginx server.


# Step 3 — Defining the First Nginx Server


You can use Terraform to create a DigitalOcean Droplet and install software on the Droplet once it spins up. In this step, you’ll provision a single Ubuntu 20.04 Droplet and install the Nginx web server using Terraform.


Create a new Terraform configuration file called www-1.tf, which will hold the configuration for the Droplet:


```
nano www-1.tf


```


Insert the following lines to define the Droplet resource:


~/loadbalance/www-1.tf
```
resource "digitalocean_droplet" "www-1" {
    image = "ubuntu-20-04-x64"
    name = "www-1"
    region = "nyc3"
    size = "s-1vcpu-1gb"
    ssh_keys = [
      data.digitalocean_ssh_key.terraform.id
    ]

```


In the preceding configuration, the first line defines a digitalocean_droplet resource named www-1. The rest of the lines specify the Droplet’s attributes, including the data center it will be residing in and the slug that identifies the size of the Droplet you want to configure. In this case you’re using s-1vcpu-1gb, which will create a Droplet with one CPU and 1GB of RAM. (Visit this size slug chart to see the available slugs you can use.)


The ssh_keys section specifies a list of public keys you want to add to the Droplet. In this case you’re specifying the key you defined in provider.tf. Ensure the name here matches the name you specified in provider.tf.


When you run Terraform against the DigitalOcean API, it will collect a variety of information about the Droplet, such as its public and private IP addresses. This information can be used by other resources in your configuration.


If you are wondering which arguments are required or optional for a Droplet resource, please refer to the official Terraform documentation: DigitalOcean Droplet Specification.


To set up a connection that Terraform can use to connect to the server via SSH, add the following lines at the end of the file:


~/loadbalance/www-1.tf
```
...
connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }

```


These lines describe how Terraform should connect to the server, so Terraform can connect over SSH to install Nginx. Note the use of the private key variable var.pvt_key—you’ll pass its value in when you run Terraform.


Now that you have the connection set up, configure the remote-exec provisioner, which you’ll use to install Nginx. Add the following lines to the configuration to do just that:


~/loadbalance/www-1.tf
```
...
provisioner "remote-exec" {
    inline = [
      "export PATH=$PATH:/usr/bin",
      # install nginx
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
  }
}

```


Note that the strings in the inline array are the commands that the root user will run to install Nginx.


The completed file looks like this:


~/loadbalance/www-1.tf
```
resource "digitalocean_droplet" "www-1" {
  image = "ubuntu-20-04-x64"
  name = "www-1"
  region = "nyc3"
  size = "s-1vcpu-1gb"
  ssh_keys = [
    data.digitalocean_ssh_key.terraform.id
  ]
  
  connection {
    host = self.ipv4_address
    user = "root"
    type = "ssh"
    private_key = file(var.pvt_key)
    timeout = "2m"
  }
  
  provisioner "remote-exec" {
    inline = [
      "export PATH=$PATH:/usr/bin",
      # install nginx
      "sudo apt update",
      "sudo apt install -y nginx"
    ]
  }
}

```


Save the file and exit the editor. You’ve defined the server, and are ready to deploy it, which you’ll now do.


# Step 4 — Using Terraform to Create the Nginx Server


Your current Terraform configuration describes a single Nginx server. You’ll now deploy the Droplet exactly as it’s defined.


Run the terraform plan command to see the execution plan, or what Terraform will attempt to do to build the infrastructure you described. You will have to specify the values for your DigitalOcean Access Token and the path to your private key, as your configuration uses this information to access your Droplet to install Nginx. Run the following command to create a plan:


```
terraform plan \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa" 


```



Warning: The terraform plan command supports an -out parameter to save the plan. However, the plan will store API keys, and Terraform does not encrypt this data. When using this option, you should explore encrypting this file if you plan to send it to others or leave it at rest for an extended period of time.

You’ll see output similar to this:


```
OutputTerraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.www-1 will be created
  + resource "digitalocean_droplet" "www-1" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + graceful_shutdown    = false
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "www-1"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = (known after apply)
      + region               = "nyc3"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + ssh_keys             = [
          + "...",
        ]
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.

───────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.

```


The + resource "digitalocean_droplet" "www-1"  line means that Terraform will create a new Droplet resource called www-1, with the details that follow it. That’s exactly what should happen, so run terraform apply command to execute the current plan:


```
terraform apply \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


You’ll get the same output as before, but this time, Terraform will ask you if you want to proceed:


```
Output...
Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

```


Enter yes and press ENTER. Terraform will provision your Droplet:


```
Outputdigitalocean_droplet.www-1: Creating...

```


After a bit of time, you’ll see Terraform installing Nginx with the remote-exec provisioner, and then the process will complete:


```
Output
digitalocean_droplet.www-1: Provisioning with 'remote-exec'...

....

digitalocean_droplet.www-1: Creation complete after 1m54s [id=your_www-1_droplet_id]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
...

```


Terraform has created a new Droplet called www-1 and installed Nginx on it. If you visit the public IP address of your new Droplet, you’ll see the Nginx welcome screen. The public IP was displayed when the Droplet was created, but you can always view it by looking at Terraform’s current state. Terraform updates the state file terraform.tfstate every time it executes a plan or refreshes its state.


To view the current state of your environment, use the following command:


```
terraform show terraform.tfstate


```


This will show you the public IP address of your Droplet.


```
Outputresource "digitalocean_droplet" "www-1" {
    backups              = false
    created_at           = "..."
    disk                 = 25
    id                   = "your_www-1_droplet_id"
    image                = "ubuntu-20-04-x64"
    ipv4_address         = "your_www-1_server_ip"
    ipv4_address_private = "10.128.0.2"
    ...

```


Navigate to http://your_www-1_server_ip in your browser to verify your Nginx server is running.



Note: If you modify your infrastructure outside of Terraform, your state file will be out of date. If your resources are modified outside of Terraform, you’ll need to refresh the state file to bring it up to date. This command will pull the updated resource information from your provider(s):
terraform refresh \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"



In this step, you’ve deployed the Droplet that you’ve described in Terraform. You’ll now create a second one.


# Step 5 — Creating the Second Nginx Server


Now that you have described an Nginx server, you can add a second quickly by copying the existing server’s configuration file and replacing the name and hostname of the Droplet resource.


You can do this manually, but it’s faster to use the sed command to read the www-1.tf file, substitute all instances of www-1 with www-2, and create a new file called www-2.tf. Here is the sed command to do that:


```
sed 's/www-1/www-2/g' www-1.tf > www-2.tf


```


You can learn more about sed by visiting Using sed.


Run terraform plan again to preview the changes that Terraform will make:


```
terraform plan \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


The output shows that Terraform will create the second server, www-2:


```
OutputTerraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_droplet.www-2 will be created
  + resource "digitalocean_droplet" "www-2" {
      + backups              = false
      + created_at           = (known after apply)
      + disk                 = (known after apply)
      + id                   = (known after apply)
      + image                = "ubuntu-20-04-x64"
      + ipv4_address         = (known after apply)
      + ipv4_address_private = (known after apply)
      + ipv6                 = false
      + ipv6_address         = (known after apply)
      + locked               = (known after apply)
      + memory               = (known after apply)
      + monitoring           = false
      + name                 = "www-2"
      + price_hourly         = (known after apply)
      + price_monthly        = (known after apply)
      + private_networking   = true
      + region               = "nyc3"
      + resize_disk          = true
      + size                 = "s-1vcpu-1gb"
      + ssh_keys             = [
          + "...",
        ]
      + status               = (known after apply)
      + urn                  = (known after apply)
      + vcpus                = (known after apply)
      + volume_ids           = (known after apply)
      + vpc_uuid             = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
...

```


Run terraform apply again to create the second Droplet:


```
terraform apply \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


As before, Terraform will ask you to confirm you wish to proceed. Review the plan again and type yes to continue.


After some time, Terraform will create the new server and display the results:


```
Outputdigitalocean_droplet.www-2: Creation complete after 1m47s [id=your_www-2_droplet_id]
...
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.

```


Terraform created the new server, while not altering the existing one. You can repeat this step to add additional Nginx servers.


Now that you have two Droplets running Nginx, you’ll define and deploy a load balancer to split traffic between them.


# Step 6 — Creating the Load Balancer


You’ll use a DigitalOcean Load Balancer, which the official Terraform provider supports, to route traffic between the two web servers.


Create a new Terraform configuration file called loadbalancer.tf:


```
nano loadbalancer.tf


```


Add the following lines to define the Load Balancer:


~/loadbalance/loadbalancer.tf
```
resource "digitalocean_loadbalancer" "www-lb" {
  name = "www-lb"
  region = "nyc3"

  forwarding_rule {
    entry_port = 80
    entry_protocol = "http"

    target_port = 80
    target_protocol = "http"
  }

  healthcheck {
    port = 22
    protocol = "tcp"
  }

  droplet_ids = [digitalocean_droplet.www-1.id, digitalocean_droplet.www-2.id ]
}

```


The Load Balancer definition specifies its name, the datacenter it will be in, the ports it should listen on to balance traffic, configuration for the health check, and the IDs of the Droplets it should balance, which you fetch using Terraform variables. Save and close the file.


Run terraform plan command again to review the new execution plan:


```
terraform plan \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


You’ll see several lines of output, including the following lines:


```
Output...
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # digitalocean_loadbalancer.www-lb will be created
  + resource "digitalocean_loadbalancer" "www-lb" {
      + algorithm                        = "round_robin"
      + disable_lets_encrypt_dns_records = false
      + droplet_ids                      = [
          + ...,
          + ...,
        ]
      + enable_backend_keepalive         = false
      + enable_proxy_protocol            = false
      + id                               = (known after apply)
      + ip                               = (known after apply)
      + name                             = "www-lb"
      + redirect_http_to_https           = false
      + region                           = "nyc3"
      + size_unit                        = (known after apply)
      + status                           = (known after apply)
      + urn                              = (known after apply)
      + vpc_uuid                         = (known after apply)

      + forwarding_rule {
          + certificate_id   = (known after apply)
          + certificate_name = (known after apply)
          + entry_port       = 80
          + entry_protocol   = "http"
          + target_port      = 80
          + target_protocol  = "http"
          + tls_passthrough  = false
        }

      + healthcheck {
          + check_interval_seconds   = 10
          + healthy_threshold        = 5
          + port                     = 22
          + protocol                 = "tcp"
          + response_timeout_seconds = 5
          + unhealthy_threshold      = 3
        }

      + sticky_sessions {
          + cookie_name        = (known after apply)
          + cookie_ttl_seconds = (known after apply)
          + type               = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
...

```


This means that the www-1 and www-2 Droplets already exist, and Terraform will create the www-lb Load Balancer.


Run terraform apply to build the Load Balancer:


```
terraform apply \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


Once again, Terraform will ask you to review the plan. Approve the plan by entering yes to continue.


Once you do, you’ll see output that contains the following lines, truncated for brevity:


```
Output...
digitalocean_loadbalancer.www-lb: Creating...
...
digitalocean_loadbalancer.www-lb: Creation complete after 1m18s [id=your_load_balancer_id]
...
Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
...

```


Use terraform show terraform.tfstate to locate the IP address of your Load Balancer:


```
terraform show terraform.tfstate


```


You’ll find the IP under the www-lb entry:


```
Output...
# digitalocean_loadbalancer.www-lb:
resource "digitalocean_loadbalancer" "www-lb" {
    algorithm                = "round_robin"
    disable_lets_encrypt_dns_records = false
    droplet_ids              = [
        your_www-1_droplet_id,
        your_www-2_droplet_id,
    ]
    enable_backend_keepalive = false
    enable_proxy_protocol    = false
    id                       = "your_load_balancer_id"
    ip                       = "your_load_balancer_ip"
    name                     = "www-lb"
    ...

```


Navigate to http://your_load_balancer_ip in your browser and you’ll see an Nginx welcome screen because the Load Balancer is sending traffic to one of the two Nginx servers.


You’ll now learn how to configure DNS for your DigitalOcean account using Terraform.


# Step 7 — Creating DNS Domains and Records


In addition to Droplets and Load Balancers, Terraform can also create DNS domain and record domains. For example, if you want to point your domain to your Load Balancer, you can write the configuration describing that relationship.



Note: Use your own, unique domain name or Terraform will be unable to deploy the DNS resources. Be sure your domain is pointed to DigitalOcean nameservers.

Create a new file to describe your DNS:


```
nano domain_root.tf


```


Add the following domain resource, replacing your_domain with your domain name:


~/loadbalance/domain_root.tf
```
resource "digitalocean_domain" "default" {
   name = "your_domain"
   ip_address = digitalocean_loadbalancer.www-lb.ip
}

```


Save and close the file when you’re done.


You can also add a CNAME record that points www.your_domain to your_domain. Create a new file for the CNAME record:


```
nano domain_cname.tf


```


Add these lines to the file:


domain_cname.tf
```
resource "digitalocean_record" "CNAME-www" {
  domain = digitalocean_domain.default.name
  type = "CNAME"
  name = "www"
  value = "@"
}

```


Save and close the file when you’re done.


To add the DNS entries, run terraform plan followed by terraform apply, as with the other resources.


Navigate to your domain name and you’ll see an Nginx welcome screen because the domain is pointing to the Load Balancer, which is sending traffic to one of the two Nginx servers.


# Step 8 — Destroying Your Infrastructure


Although not commonly used in production environments, Terraform can also destroy infrastructure that it created. This is mainly useful in development environments that are deployed and destroyed multiple times.


First, create an execution plan to destroy the infrastructure by using terraform plan -destroy:


```
terraform plan -destroy -out=terraform.tfplan \
  -var "do_token=${DO_PAT}" \
  -var "pvt_key=$HOME/.ssh/id_rsa"


```


Terraform will output a plan with resources marked in red, and prefixed with a minus sign, indicating that it will delete the resources in your infrastructure.


Then, use terraform apply to run the plan:


```
terraform apply terraform.tfplan


```


Terraform will proceed to destroy the resources, as indicated in the generated plan.


# Conclusion


In this tutorial, you used Terraform to build a load-balanced web infrastructure on DigitalOcean, with two Nginx web servers running behind a DigitalOcean Load Balancer. You know how to create and destroy resources, view the current state, and use Terraform to configure DNS entries.


Now that you understand how Terraform works, you can create configuration files that describe a server infrastructure for your own projects. The example in this tutorial is a good starting point that demonstrates how you can automate the deployment of servers. If you already use provisioning tools, you can integrate them with Terraform to configure servers as part of their creation process instead of using the provisioning method used in this tutorial.


Terraform has many more features, and can work with other providers. Check out the official Terraform Documentation to learn more about how you can use Terraform to improve your own infrastructure.


This tutorial is part of the How To Manage Infrastructure with Terraform series. The series covers a number of Terraform topics, from installing Terraform for the first time to managing complex projects.


