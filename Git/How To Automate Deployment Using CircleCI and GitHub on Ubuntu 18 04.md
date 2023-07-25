# How To Automate Deployment Using CircleCI and GitHub on Ubuntu 18 04

```Ubuntu``` ```CI/CD``` ```Git```

The author selected International Medical Corps to receive a donation as part of the Write for DOnations program.


## Introduction


Continuous Integration/Continuous Deployment (CI/CD) is a development practice that allows software teams to build, test, and deploy applications easier and quicker on multiple platforms. CircleCI is a popular automation platform that allows you to build and maintain CI/CD workflows for your projects.


Having continuous deployment is beneficial in many ways. It helps to standardize the deployment steps of an application and to protect it from un-recorded changes. It also helps avoid performing repetitive steps and lets you focus more on development. With CircleCI, you can have a single view across all your different deployment processes for development, testing, and production.


In this tutorial, you’ll build a Node.js app locally and push it to GitHub. Following that, you’ll configure CircleCI to connect to a virtual private server (VPS) that’s running Ubuntu 18.04, and you’ll go through the steps to set up your code for auto-deployment on the VPS. By the end of the article, you will have a working CI/CD pipeline where CircleCI will pick up any code you push from your local environment to the GitHub repo and deploy it on your VPS.


# Prerequisites


Before you get started, you’ll need to have the following:


- A GitHub account, which you can make at the GitHub website.
- One Ubuntu 18.04 server set up by following this Initial Server Setup for Ubuntu 18.04 tutorial, including a sudo non-root user and a firewall.
- A CircleCI account, which you can make at the CircleCI website.
- It will be helpful to know the basics of git version control. You can use How To Create a Pull Request on GitHub and Introduction to Git: Installation, Usage, and Branches to learn the fundamentals.
- Both in the local environment and on the remote server, you will need a development environment running Node.js; this tutorial was tested on Node.js version 10.22.0 and npm version 6.14.6. To install this on macOS or Ubuntu 18.04, follow the steps in How to Install Node.js and Create a Local Development Environment on macOS or the Installing Using a PPA section of How To Install Node.js on Ubuntu 18.04.

# Step 1 — Creating a Local Node Project


In this step, you’ll create a Node.js project locally that you will use during this tutorial as an example application. You’ll push this to a repo on GitHub later.


Go ahead and run these commands on your local terminal so that you can set up a quick Node development environment.


First, create a directory for the test project:


```
mkdir circleci-test


```


Change into the new directory:


```
cd circleci-test


```


Follow this up by initializing a npm environment to pull the dependencies if you have any. The -y flag will auto-accept every prompt thrown by npm init:


```
npm init -y


```


For more information on npm, check out our How To Use Node.js Modules with npm and package.json tutorial.


Next, create a basic server that serves Hello World! when someone accesses any route. Using a text editor, create a file called app.js in the root directory of the project. This tutorial will use nano:


```
nano app.js 


```


Add the following code to the app.js file:


circleci-test/app.js
```
const http = require('http');

http.createServer(function (req, res) {
  res.write('Hello World!'); 
  res.end(); 
}).listen(8080, '0.0.0.0'); 

```


This sample server uses the http package to listen to any incoming requests on port 8080 and fires a request listener function that replies with the string Hello World.


Save and close the file.


You can test this on your local machine by running the following command from the same directory in the terminal. This will create a Node process that runs the server (app.js):


```
node app.js


```


Now visit the http://localhost:8080 URL in your browser. Your browser will render the string Hello World!. Once you have tested the app, stop the server by pressing CTRL+C on the same terminal where you started the Node process.


You’ve now set up your sample application. In the next step, you will add a configuration file in the project so that CircleCI can use it for deployment.


# Step 2 — Adding a Config File to Your Project


CircleCI executes workflows according to a configuration file in your project folder. In this step, you will create that file to define the deployment workflow.


Create a folder called .circleci in the root directory of your project:


```
mkdir .circleci


```


Add a new file called config.yml in it:


```
nano .circleci/config.yml


```


This will open a file with the YAML file extension. YAML is a language that is often used for configuration management.


Add the following configuration to the new config.yml file:


circleci-test/.circleci/config.yml
```
version: 2.1

# Define the jobs we want to run for this project
jobs:
  pull-and-build:
    docker:
      - image: arvindr226/alpine-ssh
    steps:
      - checkout
      - run: ssh -oStrictHostKeyChecking=no -v $USER@$IP "./deploy.sh"

# Orchestrate our job run sequence
workflows:
  version: 2
  build-project:
    jobs:
      - pull-and-build:
          filters:
            branches:
              only:
                - main

```


Save this file and exit the text editor.


This file tells the CircleCI pipeline the following:


- There is a job called pull-and-build whose steps involve spinning up a Docker container, SSHing from it to the VPS, and then running the deploy.sh file.
- The Docker container serves the purpose of creating a temporary environment for executing the commands mentioned in the steps section of the config. In this case, all you need to do is SSH into the VPS and run sh deploy.sh command, so the environment needs to be lightweight but still allow the SSH command. The Docker image arvindr226/alpine-ssh is an Alpine Linux image that supports SSH.
- deploy.sh is a file that you will create in the VPS. It will run every time as a part of the deployment process and will contain steps specific to your project.
- In the workflows section, you inform CircleCI that it needs to perform this job based on some filters, which in this case is that only changes to the main branch will trigger this job.

Next, you will commit and push these files to a GitHub repository. You will do this by running the following commands from the project directory.


First, initialize the Node.js project directory as a git repo:


```
git init


```


Go ahead and add the new changes to the git repo:


```
git add .


```


Then commit the changes:


```
git commit -m "initial commit"


```


If this is the first time committing, git will prompt you to run some git config commands to identify you.


From your browser navigate to GitHub and log in with your GitHub account. Create a new repository called circleci-test without a README or license file. Once you’ve created the repository, return to the command line to push your local files to GitHub.


To follow GitHub protocol, rename your branch main with the following command:


```
git branch -M main


```


Before you push the files for the first time, you need to add GitHub as a remote repository. Do that by running:


```
git remote add origin https://github.com/GitHub_username/circleci-test


```


Follow this with the push command, which will transfer the files to GitHub:


```
git push -u origin main


```


You have now pushed your code to GitHub. In the next step, you’ll create a new user in the VPS that will execute the steps in the pull-and-build part.


# Step 3 — Creating a New User for Deployment


Now that you have the project ready, you will create a deployment user in the VPS.


Connect to your VPS as your sudo user


```
ssh your_username@your_server_ip


```


Next, create a new user that doesn’t use a password for login using the useradd command.


```
sudo useradd -m -d /home/circleci -s /bin/bash circleci


```


This command creates a new user on the system. The -m flag instructs the command to create a home directory specified by the -d flag.


circleci will be the new deployment user in this case. For security purposes, you are not going to add this user to the sudo group, since the only job of this user is to create an SSH connection from the VPS to the CircleCI network and run the deploy.sh script.


Make sure that the firewall on your VPS is open to port 8080:


```
sudo ufw allow 8080


```


You now need to create an SSH key, which the new user can use to log in. You are going to create an SSH key with no passphrase, or else CircleCI will not be able to decrypt it. You can find more information in the official CircleCI documentation. Also, CircleCI expects the format of the SSH keys to be in the PEM format, so you are going to enforce that while creating the key pair.


Back on your local system, move to your home folder:


```
cd


```


Then run the following command:


```
ssh-keygen -m PEM -t rsa -f .ssh/circleci


```


This command creates an RSA key with the PEM format specified by the -m flag and the key type specified by the -t flag. You also specify the -f to create a new key pair called circleci and circleci.pub. Specifying the name will avoid overwriting your existing id_rsa file.


Print out the new public key:


```
cat ~/.ssh/circleci.pub


```


This outputs the public key that you generated. You will need to register this public key in your VPS. Copy this to your clipboard.


Back on the VPS, create a .ssh directory for the circleci user:


```
sudo mkdir /home/circleci/.ssh


```


Here you’ll add the public key you copied from the local machine into a file called authorized_keys:


```
sudo nano /home/circleci/.ssh/authorized_keys


```


Add the copied public key here, save the file, and exit the text editor.


Give the circleci user its directory permissions so that it doesn’t run into permission issues during deployment.


```
sudo chown -R circleci:circleci /home/circleci


```


Verify if you can log in as the new user by using the private key. Open a new terminal on your local system and run:


```
ssh circleci@your_server_ip -i ~/.ssh/circleci


```


You will now log in as the circleci user into your VPS. This shows that the SSH connection is successful. Next, you will connect your GitHub repo to CircleCI.


# Step 4 — Adding Your GitHub Project to CircleCI


In this step, you’ll connect your GitHub account to your CircleCI account and add the circleci-test project for CI/CD. If you signed up with your GitHub account, then your GitHub will be automatically linked with your CircleCI account. If not, head over to https://circleci.com/account and connect it.


To add your circleci-test project, navigate to your CircleCI project dashboard at https://app.circleci.com/projects/project-dashboard/github/your_username:





Here you will find all the projects from GitHub listed. Click on Set Up Project for the project circleci-test. This will bring you to the project setup page:





You’ll now have the option to set the config for the project, which you have already set in the repo. Since this is already set up, choose the Use Existing Config option. This will bring up a popup box confirming that you want to build the pipeline:





From here, go ahead and click on Start Building. This will bring you to the circleci-test pipeline page. For now, this pipeline will fail. This is because you must first update the SSH keys for your project.


Navigate to the project settings at https://app.circleci.com/settings/project/github/your_username/circleci-test and select the SSH keys section on the left.


Retrieve the private key named circleci you created earlier from your local machine by running:


```
cat ~/.ssh/circleci


```


Copy the output from this command.


Under the Additional SSH Keys section, click on the Add SSH Key button.





This will open up a window asking you to enter the hostname and the SSH key. Enter a hostname of your choice, and add in the private SSH key that you copied from your local environment.


CircleCI will now be able to log in as the new circleci user to the VPS using this key.


The last step is to provide the username and IP of the VPS to CircleCI. In the same Project Settings page, go to the Environment Variables tab on the left:





Add an environment variable named USER with a value of circleci and IP with the value of the IP address of your VPS (or domain name of your VPS, if you have a DNS record).


Once you’ve created these variables, you have completed the setup needed for CircleCI. Next, you will give the circleci user access to GitHub via SSH.


# Step 5 — Adding SSH Keys to GitHub


You now need to provide a way that the circleci user can authenticate with GitHub so that it can perform git operations like git pull.


To do this, you will create an SSH key for this user to authenticate against GitHub.


Connect to the VPS as the circleci user:


```
ssh circleci@your_server_ip -i ~/.ssh/circleci


```


Create a new SSH key pair with no passphrase:


```
ssh-keygen -t rsa 


```


Then output the public key:


```
cat ~/.ssh/id_rsa.pub


```


Copy the output, then head over to your circleci-test GitHub repo’s deploy key settings at https://github.com/your_username/circleci-test/settings/keys.


Click on Add deploy key to add the copied public key. Fill the Title field with your desired name for the key, then add the copied public key in the Key field. Finally, click the Add key button to add the key to your account.


Now that the circleci user has access to your GitHub account, you’ll use this SSH authentication to set up your project.


# Step 6 — Setting Up the Project on the VPS


Now for setting up the project, you are going to clone the repo and make the initial setup of the project on the VPS as the circleci user.


On your VPS, run the following command:


```
git clone git@github.com:your_username/circleci-test.git


```


Navigate into it:


```
cd circleci-test


```


First, install the dependencies:


```
npm install


```


Now test the app out by running the server you built:


```
node app.js


```


Head over to your browser and try the address http://your_vps_ip:8080. You will receive the output Hello World!.


Stop this process with CTRL+C and use pm2 to run this app as a background process.


Install pm2 so that you can run the Node app as an independent process. pm2 is a versatile process manager written in Node.js. Here it will help you keep the sample Node.js project running as an active process even after you log out of the server. You can read a bit more about this in the How To Set Up a Node.js Application for Production on Ubuntu 18.04 tutorial.


```
npm install -g pm2


```



Note:  On some systems such as Ubuntu 18.04, installing an npm package globally can result in a permission error, which will interrupt the installation. Since it is a security best practice to avoid using sudo with npm install, you can instead resolve this by changing npm’s default directory. If you encounter an EACCES error, follow the instructions at the official npm documentation.

You can use the pm2 start command to run the app.js file as a Node process. You can name it app using the --name flag to identify it later:


```
pm2 start app.js --name "app"


```


You will also need to provide the deployment instructions. These commands will run every time the circleci user deploys the code.


Head back to the home directory since that will be the path the circleci user will land in during a successful login attempt:


```
cd ~


```


Go ahead and create the deploy.sh file, which will contain the deploy instructions:


```
nano deploy.sh


```


You will now use a Bash script to automate the deployment:


deploy.sh
```
#!/bin/bash

#replace this with the path of your project on the VPS
cd ~/circleci-test

#pull from the branch
git pull origin main

# followed by instructions specific to your project that you used to do manually
npm install
export PATH=~/.npm-global/bin:$PATH
source ~/.profile

pm2 restart app

```


This will automatically change the working directory to the project root, pull the code from GitHub, install the dependencies, then restart the app. Save and exit the file.


Make this file an executable by running:


```
chmod u+x deploy.sh


```


Now head back to your local machine and make a quick change to test it out. Change into your project directory:


```
cd circleci-test


```


Open up your app.js file:


```
nano circleci-test/app.js


```


Now add in the following highlighted line:


circleci-test/app.js
```
const http = require('http');

http.createServer(function (req, res) {
  res.write('Foo Bar!');
  res.end();
}).listen(8080, '0.0.0.0');

```


Save the file and exit the text editor.


Add this change and commit it:


```
git add .
git commit -m "modify app.js"


```


Now push this to your main branch:


```
git push origin main


```


This will trigger a new pipeline for deployment. Navigate to https://app.circleci.com/pipelines/github/your_username to view the pipeline in action.


Once it’s successful, refresh the browser at http://your_vps_ip:8080. Foo Bar! will now render in your browser.


# Conclusion


These are the steps to integrate CircleCI with your GitHub repository and Linux-based VPS. You can modify the deploy.sh for more specific instructions related to your project.


If you would like to learn more about CI/CD, check out our CI/CD topic page. For more on setting up workflows with CircleCI, head over to the CircleCI documentation.


