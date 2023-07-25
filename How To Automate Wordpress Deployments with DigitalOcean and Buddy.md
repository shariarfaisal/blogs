# How To Automate Wordpress Deployments with DigitalOcean and Buddy

```Docker``` ```CI/CD``` ```WordPress``` ```PHP``` ```PHP Frameworks```

## Introduction


In this tutorial, you will automate WordPress deployments using Buddy CI/CD, a user-friendly tool offering continuous integration and continuous deployment solutions.


Compared to many other CI/CD tools, Buddy requires less DevOps experience. It allows developers to create delivery pipelines with drag-and-drop actions in a visual GUI. This GUI leverages pre-configured actions (builds, test, deployments, etc.) in an approach similar to DigitalOcean’s interactive Droplet configuration. This means newcomers and expert developers alike can use Buddy to release more software, all while making fewer errors.


Once you’ve completed this tutorial, you will be able to perform a WordPress deployment with a single command from your local terminal. For better insight, you will build a more advanced Sage-based WordPress theme that requires multiple build steps before you can deploy it to the WordPress server.


# Prerequisites


- Docker and Docker Compose installed on your local machine. On Windows, install Docker Desktop for Windows, which includes both tools. On macOS, install Docker Desktop for Mac. For Ubuntu and other Linux operating systems, you can follow our How To Install Docker and How To Install Docker Compose tutorials.
- Git installed on your local machine. Getting Started with Git Version Control is a useful resource.
- An account with an online Git provider like GitHub or GitLab.
- Composer installed on your local machine. This tutorial collection includes instructions for installing Composer on many Linux distributions. On macOS you can download the installer from Composer’s website or use the Homebrew package manager.
- The Yarn package manager installed on your local machine.
- PHP version 7.2+ installed on your local machine.  On Linux, you can follow Step 3 of our tutorial on How To Install Linux, Apache, MySQL, PHP (LAMP) Stack. On macOS you can install PHP using the Homebrew package manager.
- Node.js version 14+ installed on your local machine. This tutorial collection can help you install Node.js on Linux. On macOS, you can also install Node.js using the Homebrew package manager.


Note: This tutorial was tested on Node.js version 14.13.0, npm version 6.14.8, and PHP version 7.4.10.

- A WordPress installation on a DigitalOcean Droplet. This is where you will deploy your customized theme after building it locally. If you are looking to access a ready-made WordPress installation, DigitalOcean Marketplace offers a one-click app to get you started with WordPress. You can also install WordPress on a Droplet by following the instructions in our tutorial, How To Install Wordpress on a LAMP Stack.
- A fully-qualified domain name with an A record pointed to your Wordpress Droplet’s IP. This tutorial can help you set up a hostname with DigitalOcean.
- A DigitalOcean Personal Access Token for the API.

# Step 1 — Installing WordPress with Docker


In this step you will pull the WordPress image from Docker and start your build.


First, verify that Docker is running with the following command:


```
docker info


```


You will receive an output like this:


```
OutputClient:
 Debug Mode: false

Server:
 Containers: 0
  Running: 0
  Paused: 0
  Stopped: 0
 Images: 0
 Server Version: 19.03.12
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
 Logging Driver: json-file
 Cgroup Driver: cgroupfs
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
  ...

```


Now that you’ve verified that Docker is running, download the latest version of the WordPress image:


```
docker pull wordpress


```


Next, create a folder for your project in your workspace:


```
mkdir docker-wordpress-theme


```


Navigate inside your new project folder:


```
cd docker-wordpress-theme


```


Now you need to define your build. Use nano or your preferred text editor to open and create a file called docker-compose.yml:


```
nano docker-compose.yml


```


Add the following definitions to the file. These describe the version of Docker Compose and the services to be launched. In this case, you are launching WordPress and MySQL database. Make sure to replace the highlighted fields with your credentials:


docker-compose.yml
```
version: "3.1"

services:
  wordpress:
    image: wordpress
    restart: always
    ports:
      - 8080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: exampleuser
      WORDPRESS_DB_PASSWORD: examplepass
      WORDPRESS_DB_NAME: exampledb
    volumes:
      - wordpress:/var/www/html
      - ./sage:/var/www/html/wp-content/themes/sage/
  db:
    image: mysql:5.7
    restart: always
    environment:
      MYSQL_DATABASE: exampledb
      MYSQL_USER: exampleuser
      MYSQL_PASSWORD: examplepass
      MYSQL_RANDOM_ROOT_PASSWORD: "1"
    volumes:
      - db:/var/lib/mysql

volumes:
  wordpress:
  db:

```


Here you are defining the images that Docker will launch in the service and then setting ports and environment variables


Take note that you are mounting a folder called sage that you haven’t created yet. This will be your custom theme, which you will now create.


# Step 2 — Creating a Custom WordPress Theme


In this step you will create a custom wordpress theme. You will then create a CI/CD pipeline so that you can push changes you make locally to your Wordpress server with one command.


Let’s start building our custom theme by installing the Sage framework on our local WordPress installation. This theme uses Node.js and Gulp to perform development and build functions. There won’t be any build dependencies installed on the production server – instead, all production build tasks will be performed on Buddy, the remote Continuous Integration server.


Make sure you are in your project folder:


```
cd docker-wordpress-theme


```


Use Composer to create a new Sage theme:


```
composer create-project roots/sage


```


With everything properly configured, the following output will appear:


```
OutputInstalling roots/sage (9.0.9)
  - Installing roots/sage (9.0.9): Loading from cache
Created project in /home/mike/Projects/buddy/github/sage
Loading composer repositories with package information
Installing dependencies (including require-dev) from lock file
Package operations: 29 installs, 0 updates, 0 removals
  - Installing composer/installers (v1.6.0): Downloading (100%)
  - Installing symfony/polyfill-mbstring (v1.10.0): Downloading (100%)
  - Installing symfony/contracts (v1.0.2): Downloading (100%)
  - ..........

```


The installer will then ask you to select the framework to load:


```
Output- Theme Name > Sage Starter Theme
- Theme URI > https://roots.io/sage/
- Theme Name [Sage Starter Theme]:
- Theme Description > Sage is a WordPress starter theme.
- Theme Version > 9.0.9
- Theme Author > Roots
- Theme Author URI > https://roots.io/
- Local development URL of WP site > http://localhost:8080
- Path to theme directory > /wp-content/themes/sage
- Which framework would you like to load? [Bootstrap]:
    [0] None
    [1] Bootstrap
    [2] Bulma
    [3] Foundation
    [4] Tachyons
    [5] Tailwind

```



Note: Make sure sure that the local development URL matches the port.

Press 1 to select the Bootstrap framework. You will be asked for permission to overwrite a couple of files. Type y to confirm and proceed:


```
Output Are you sure you want to overwrite the following files?
 - scripts/autoload/_bootstrap.js
 - styles/autoload/_bootstrap.scss
 - styles/common/_variables.scss
 - styles/components/_comments.scss
 - styles/components/_forms.scss
 - styles/components/_wp-classes.scss
 - styles/layouts/_header.scss

 (yes/no) [no]:

```


You now have the foundations of a custom WordPress theme. In the next step you will build and launch the theme, and then you will version it using Git.


# Step 3 — Building and Launching a Custom WordPress Theme


In this step you will install all your build dependencies, create a production build, and lauch WordPress in a local Docker container.


Navigate to the Sage folder:


```
cd ./sage


```


Install the node-sass binary to prevent installation failure (the rest of package.json will be installed, too):


```
yarn add node-sass -D


```


Run a production build that will compile all Sass/SCSS files and minify CSS and JS:


```
yarn build:production


```


With the build generated, exit the theme folder and launch your WordPress instance using Docker Compose:


```
cd ..
docker-compose up -d


```


Launching WordPress in the Docker environment should only take a few seconds. Now open the URL http://localhost:8080 in a web browser to access your local WordPress site. Since this is the first time you are launching WordPress, you will be prompted to create an Admin account. Create one now.


Once you have created an account and are logged in, head over to Appearance > Themes page on the dashboard. You will find several pre-installed themes including the Sage theme we’ve just created. Click the Activate button to set it as the current theme. Your home page will look something like this:





You have now built and activated a custom theme. In the next step, you will put your project under version control.


# Step 4 — Uploading a WordPress Project to a Remote Repository


Version control is a cornerstone of the CI/CD workflow. In this step, you will upload your project to a remote Git repository that the Buddy platform can access. Buddy integrates with many popular Git providers, including:


- GitHub
- GitLab
- Bitbucket
- Privately-hosted Git repositories

Create a remote repository on the platform of your choice. For the purpose of this guide we’ll use GitHub. Click here to read how you can create a new repo using the Github UI.


Then, in your terminal, initialize Git in your project’s remote directory:


```
git init


```


Add the newly created remote repository. Replace the highlighted section with your own repository’s URL:


```
git add remote https://github.com/user-name/your-repo-name.git

```


Before you push your project, there are some files that you want to exclude from version control.


Create a file called .gitignore:


```
nano .gitignore


```


Add the following filenames:


./.gitignore
```
.cache-loader
composer.phar
dist
node_modules
vendor

```


Save and close the file.


Now you are ready to add your project under version control and commit the files to your repository on GitHub:


```
git add .
git commit -m 'my sage project'
git push


```


You have now built a custom WordPress theme using the Sage framework and then pushed the code to a remote repository. Now you will automate the deployment of this theme to your WordPress server using Buddy.


# Step 5 — Automating WordPress Deployment with Buddy


If you haven’t used Buddy before, sign up with your Git provider credentials or email address. There’s a 14-day trial with no limit on resources, and a free plan with 5 projects and 120 executions/month once it’s over, which is more than enough for our needs.


Start by synchronizing Buddy with your repository. In the Buddy UI, click Create a new project, select your Git provider, and choose the repository that you created in the first section of this article.


Next, you will be prompted to create your delivery pipeline. A pipeline is a set of actions that perform tasks on your repository code, like builds, tests, or deployments.


The key settings to configure are:


- Branch from which Buddy will deploy your code – in this case, set it to master
- Pipeline trigger mode – set it to On push to automatically execute the pipeline on every push to the selected branch.

Once you add the pipeline, you’ll need to create four actions:


1. A PHP action that will install the required PHP packages
2. A Node action that will download the dependencies and prepare a build for deployment
3. A Droplet action that will upload the build code directly to your DO Droplet
4. An SSH action with a script that will activate your theme.

Based on the contents of your repository, Buddy will automatically suggest the actions to perform. Select PHP from the list.





Clicking the action will open its configuration panel. Enter the following commands in the terminal section:


```
# navigate to theme directory
cd sage

# install php packages
composer validate
composer install

```


Save and run the pipeline to ensure it works:






Note: Buddy uses isolated containers with preinstalled frameworks for builds. The downloaded dependencies are cached in the container, meaning you don’t have to download them again. Think of it as a local development environment that remains consistent for everybody on the team.

Next, add the Node.js action. In order for the theme to display properly, we’ll need to compile and minify assets, i.e. SCSS/SASS and JavaScript files.


First, set Environment to node latest.


Now you must add several commands. These commands will install the necessary dependencies and perform your build.


Add them to the terminal box just like before:


```
# navigate to theme directory
cd sage

# install packages
yarn install

# Create production build
yarn build:production

```


Once again, save and run the action to ensure it works.


Next, add the Droplet action right after the Node.js build. If you’ve never used DigitalOcean with Buddy before, a wizard will appear that will guide you through the integration. Once you’ve completed this step, define the authentication details as follows:


- 
Set the Source path to sage.

- 
Choose Buddy’s SSH key authentication mode as that is the easiest one to set up. Just log in to your Droplet server via SSH and execute the commands displayed in Buddy’s key code snippet.


After you execute those commands, go back to the browser and click the Remote path browse button – you will be able to navigate your Droplet’s filesystem and access the correct deployment folder. The default path will be /var/www/html/wp-content/themes/sage.


You will also need to visit the Ignore paths section and provide the following to prevent uploading of Node.js dependencies:


```
.cache-loader/
node_modules/

```


When done, click the Test action button to verify that everything’s been properly configured.


Last, you’ll add one more action to activate your theme on the WordPress Droplet with a WP-CLI command. On your pipeline page, add the SSH action and input the following command in the commands section:


```
sudo -u www-data -- wp theme activate sage/resources


```


Ensure you have set the correct Working directory setting – otherwise, the command won’t work.


Since you already configured Buddy’s SSH key in the previous setup, you don’t need to do anything else. Alternatively, you can select private SSH key and then you can upload your DigitalOcean private key and use that to connect to your Droplet. Buddy’s SSH key is simpler and just as secure.


Your complete pipeline will now contain 4 actions: PHP > Node > Droplet > SSH. Click the Run Pipeline button to test out all the actions at once. You will receive a green check mark for each stage:





On the first execution, Buddy will deploy all files from the repository to the selected revision. Future deployments will only update files that have changed or were deleted. This feature significantly reduces upload time because you don’t have to deploy everything from scratch on every update.


Go to your hosted WordPress dashboard and refresh the Themes page. You will see your Sage theme. Activate it now.


Your hosted home page will now match your local home page.


Our pipeline is built and our local and remote machines are synced. Now, let’s test the entire workflow.


# Step 6 — Testing Buddy’s Auto-Deployment Workflow


In this step you will make a small change to your theme and then deploy those changes to your WordPress server.


Go back to your local terminal and run this yarn command:


```
yarn start


```


This will start a live proxy development server at localhost:3000. Any changes you make to your theme will get automatically reflected in this window. The page on localhost:8080 will remain unchanged until you run the production build script.


Let’s test out our pipeline by making some minor changes to our CSS.


Open the main.scss file for your Sage theme:


```
nano ./sage/resources/assets/styles/main.scss


```


Insert the following code to introduce some subtle green color and an underline to the website’s font:


./sage/resources/assets/styles/main.scss
```
.brand {
  @extend .display-3;

  color: #013d30;
}

.entry-title {
  @extend .display-4;

  a {
    color: #015c48;
    text-decoration: underline;
  }
}

.page-header {
  display: none;
}

```


Save and close the file.


Commit these changes and upload them to your repo:


```
git add .
git commit -m "minor style changes"
git push


```


Once the code is uploaded to the repository, Buddy will automatically trigger your pipeline and execute all actions one by one:


Wait for the pipeline to finish and then refresh your WordPress Droplet’s home page to see your updates.





Your pipeline is now pushing changes from your local machine to GitHub to Buddy to your production WordPress server, all triggered by one git command.


# Conclusion


Buddy is a very user friendly and powerful CI/CD tool. Buddy even has a video that shows just how quickly you can create pipelines using their interface.


By automating your development workflow, you can focus on implementing styles and features for your custom theme or plugin without wasting time on manual deployments. The CI/CD workflow can also significantly reduce the risk of manual errors. In addition, automation allows you to further enhance the quality of your code by running unit tests and analysis tools, such as PHP Sniffer, on every change.


You can take this tutorial even further by setting up an advanced branching strategy and a staging server, where you can perform quality control checks before you deploy new code to the production server. This way you can release better software more often without losing the momentum.


