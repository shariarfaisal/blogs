# Deploying Deis with Rigger on DigitalOcean

```Docker``` ```Ubuntu``` ```Clustering``` ```Scaling``` ```CoreOS```

## An Article from Deis


## Introduction


Deis is an open source private Platform as a Service (PaaS) that simplifies deploying and managing your applications on your own servers. By leveraging technologies such as Docker and CoreOS, Deis provides a workflow and scaling features that are similar to that of Heroku, on the hosting provider of your choice. Deis supports applications that can run in a Docker container, and Deis can run on any platform that supports CoreOS.


This guide steps you through the new and improved Deis provisioning process using the Deis project’s new tool called Rigger.


# Preview


If you don’t have much time, this accelerated terminal recording (only around a minute long!) shows what we’ll be up to in the rest of this article:





# Prerequisites


Rigger is designed to handle its own dependency management, but you will need to setup a few things before provisioning a Deis cluster with it. To follow along with this guide at home you’ll need:


- DigitalOcean Personal Access Token to access the DigitalOcean API (follow How To Generate a Personal Access Token) (the token must be read-write)
- An SSH key pair (follow How To Use SSH Keys with DigitalOcean Droplets)

All the command in this tutorial can be run on a local Mac or Linux workstation (OS X >= 10.10 and Debian/Ubuntu were tested). They can also be run on a Droplet, but that is not necessary.


The zip, make, and git utilities need to be installed on whatever workstation you use to provision a Deis cluster with Rigger.


For example, if you are using an Ubuntu system, install them with the following command:


```
sudo apt-get update
sudo apt-get install zip make git


```


The git utility is used through the article to download Rigger and an example application. The zip and make utilities are used by the Rigger provisioning script.


If you are running Mac OS X, you also need to agree to the Xcode license agreement to use git:


```
sudo xcodebuild -license


```



Note: This article was written for Deis version 1.12.0.

# Step 1 — Installing Rigger


To install Rigger, first use git to download it:


```
git clone https://github.com/deis/rigger.git


```


Change to the directory created:


```
cd rigger


```


Then, execute the following command:


```
./rigger


```


When it first runs, you will see the following:


```
OutputDownloading rerun from GitHub...

```


At the end of the output, you will see a list of available commands:


```
OutputAvailable commands in module, "rigger":
checkout: "checkout the Deis repo with version: $VERSION into directory: $DEIS_ROOT"
configure: "initialize a rigger varsfile to use with future commands"
   [ --advanced]: "configure all the nitty gritty details of the infrastructure and Deis deployment"
   [ --provider <>]: "which cloud provider to use to provision a Deis cluster"
   [ --version <>]: "choose what version of Deis to deploy"
create-registry: "Create a local dev registry"
deploy: "Install and Deploy Deis"
destroy: "destroy all infrastructure created by the provision step"
provision: "provision new infrastructure and deploy Deis to it"
   [ --cleanup]: "destroy cluster after action"
setup-clients: "download and stage deisctl and deis clients for your own use"
shellinit: "show the current sourceable environment variables (useful for eval-ing)"
   [ --file <>]: "use a specific file"
shell-reset: "an eval-able output that unsets variables that have been injected by rigger"
   [ --file <>]: "use a specific file"
test: "run a test suite on the provisioned Deis cluster"
   [ --type <smoke>]: "provide a type of test to run"
upgrade: "Tests upgrade path for Deis"
   [ --to <master>]: "Define version of Deis to upgrade to"
   [ --cleanup]: "destroy cluster after action"
    --upgrade-style <graceful>: "choose the style of upgrade you'd like to perform"

```


# Step 2 — Configuring the Deis Deployment


To configure the Deis deployment to use DigitalOcean as the provider and a specific version of Deis, all we need to do is call:


```
./rigger configure --provider "digitalocean" --version "1.12.0"


```


Rigger will then ask you a few questions. It will all look like this:


```
Output-> What DigitalOcean token should I use? DO_TOKEN (no default)
[enter or paste your DigitalOcean token here]
You chose: ******

-> Which private SSH key should be used? SSH_PRIVATE_KEY_FILE [ /Users/sgoings/.ssh/id_dsa ]
1) /Users/sgoings/.ssh/id_dsa
2) ...
#? [enter a number]
You chose: 1) /Users/sgoings/.ssh/id_dsa

... output snipped ...

Enter passphrase for /Users/sgoings/.ssh/id_dsa: [enter your ssh passphrase]

Rigger has been configured on this system using ${HOME}/.rigger/<id>/vars
To use the configuration outside of rigger, you can run:

  source "${HOME}/.rigger/<id>/vars"

```


You’re all done with the hard part!


# Step 3 — Profit!


Or more accurately: run rigger to provision infrastructure on DigitalOcean and then deploy Deis!


All we need to do is execute:


```
./rigger provision


```



Warning: If you are running Rigger on Mac OS X, you might see the following error message:
Agreeing to the Xcode/iOS license requires admin privileges, please re-run as root via sudo.

If so, you need to agree to the Xcode license with the sudo xcodebuild -license command as mentioned in the Prerequisite section.

The Deis provisioning process with Rigger on DigitalOcean takes about 15 minutes and goes like this:


1. Terraform is automatically downloaded and installed into ${HOME}/.rigger for use by Rigger
2. Deis clients (deis and deisctl) are downloaded into ${HOME}/.rigger/<id>/bins
3. Terraform is used to provision 3 CoreOS Droplets in DigitalOcean
4. DEISCTL_TUNNEL is determined by investigating one of the newly provisioned DigitalOcean Droplets
5. xip.io is used to set up a simple DNS entry point to the cluster
6. deisctl install platform is executed
7. deisctl start platform is executed


Warning: If you see the following errors when provisioning the infrastructure on DigitalOcean, make sure your DO Access Token is read-write:
3 error(s) occurred:

* digitalocean_droplet.deis.1: Error creating droplet: Error creating droplet: API Error: 403 Forbidden
* digitalocean_droplet.deis.0: Error creating droplet: Error creating droplet: API Error: 403 Forbidden
* digitalocean_droplet.deis.2: Error creating droplet: Error creating droplet: API Error: 403 Forbidden

You can then run the ./rigger provision again.

# Step 4 — Playtime!


After you’ve created your Deis cluster using Rigger, you should deploy an app to it!


First, get back to some free directory space:


```
cd ../


```


Next, grab an example app from the Deis project:


```
git clone https://github.com/deis/example-nodejs-express.git


```


Change into the newly created directory:


```
cd example-nodejs-express


```


Load all the rigger environment variables into this shell:


```
source "${HOME}/.rigger/<id>/vars"


```


Then register an administrative account to this Deis cluster:


```
deis auth:register http://deis.${DEIS_TEST_DOMAIN}


```


You will be prompted for some information to create the account:


```
Outputusername: [ enter a username ]
password: [ enter a password ]
password (confirm): [ enter the same password ]
email: [ enter an email for this user ]
Registered <username>
Logged in as <username>

```


Add your public key to the Deis cluster:


```
deis keys:add


```


You will see the following:


```
OutputFound the following SSH public keys:
1) deiskey.pub deiskey
2) id_dsa.pub sgoings
0) Enter path to pubfile (or use keys:add <key_path>)
Which would you like to use with Deis? [ enter number ] 

```


You should pick the public key that goes along with the private key you chose during the rigger configure step.


Add a git remote to point at the Deis cluster:


```
deis apps:create


```


You will see the following:


```
OutputCreating Application... done, created hearty-kingfish
Git remote deis added
remote available at ssh://git@deis.${DEIS_TEST_DOMAIN}:2222/hearty-kingfish.git

```


Now, push it!


```
git push deis master


```


This might take a while. You should eventually see the following at the end of the output:


```
Output-----> Launching...
       done, hearty-kingfish:v2 deployed to Deis

       http://hearty-kingfish.${DEIS_TEST_DOMAIN}

       To learn more, use `deis help` or visit http://deis.io

```


Go ahead and load that URL in your browser! (the app is pretty simple, it just prints out: “Powered by Deis”)


# Step 5 — Knocking it all down!


Once you’ve played around with your fancy new Deis cluster a bit… it’d probably be a good idea to tear it all down, eh? That’s simple.


Go back to the rigger directory:


```
cd ../rigger


```


Then, destroy it:


```
./rigger destroy


```


# Conclusion


In this guide, you were able to see where the Deis team is headed to make the lives of developers, operators, and open source contributors easier. Provisioning a Deis cluster is now a breeze with Rigger, thanks to the winning combination of Terraform under the hood and lightning fast DigitalOcean as the infrastructure provider.


