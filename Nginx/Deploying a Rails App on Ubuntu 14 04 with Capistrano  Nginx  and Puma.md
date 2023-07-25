# Deploying a Rails App on Ubuntu 14 04 with Capistrano  Nginx  and Puma

```Ubuntu``` ```Ruby on Rails``` ```Nginx```

## Introduction


Rails is an open source web application framework written in Ruby. It follows the Convention over Configuration philosophy by making assumptions that there is the ‘best’ way of doing things. This allows you to write less code while accomplishing more without having you go through endless config files.


Nginx is a high performance HTTP server, reverse proxy, and a load balancer known for its focus on concurrency, stability, scalability, and low memory consumption. Like Nginx, Puma is another extremely fast and concurrent web server with a very small memory footprint but built for Ruby web applications.


Capistrano is a remote server automation tool focusing mainly on Ruby web apps. It’s used to reliably deploy web apps to any number of remote machines by scripting arbitrary workflows over SSH and automate common tasks such as asset pre-compilation and restarting the Rails server.


In this tutorial we’ll install Ruby and Nginx on a DigitalOcean Ubuntu Droplet and configure Puma and Capistrano in our web app. Nginx will be used to capture client requests and pass them over to the Puma web server running Rails. We’ll use Capistrano to automate common deployment tasks, so every time we have to deploy a new version of our Rails app to the server, we can do that with a few simple commands.


# Prerequisites


To follow this tutorial, you must have the following:


- Ubuntu 14.04 x64 Droplet
- A non-root user named deploy with sudo privileges (Initial Server Setup with Ubuntu 14.04 explains how to set this up.)
- Working Rails app hosted in a remote git repository that’s ready to be deployed

Optionally, for heightened security, you can disable root login via SSH and change the SSH port number as described in Initial Server Setup with Ubuntu 14.04.



Warning: After disabling root login, make sure you can SSH to your Droplet as the deploy user and use sudo for this user before closing the root SSH session you opened to make these changes.

All the commands in this tutorial should be run as the deploy user. If root access is required for the command, it will be preceded by sudo.


# Step 1 — Installing Nginx


Once the VPS is secure, we can start installing packages. Update the package index files:


```
sudo apt-get update


```


Then, install Nginx:


```
sudo apt-get install curl git-core nginx -y


```


# Step 2 — Installing Databases


Install the database that you’ll be using in your Rails app. Since there are lots of databases to choose from, we won’t cover them in this guide. You can see instructions for major ones here:


- MySQL
- PostgreSQL
- MongoDB

Also be sure to check out:


- How To Use MySQL with Your Ruby on Rails Application on Ubuntu 14.04
- How To Use PostgreSQL with Your Ruby on Rails Application on Ubuntu 14.04

# Step 3 — Installing RVM and Ruby


We won’t be installing Ruby directly. Instead, we’ll use a Ruby Version Manager. There are lots of them to choose from (rbenv, chruby, etc.), but we’ll use RVM for this tutorial. RVM allows you to easily install and manage multiple rubies on the same system and use the correct one according to your app. This makes life much easier when you have to upgrade your Rails app to use a newer ruby.


Before installing RVM, you need to import the RVM GPG Key:


```
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3


```


Then install RVM to manage our Rubies:


```
curl -sSL https://get.rvm.io | bash -s stable


```


This command uses curl to download the RVM Installation script from https://get.rvm.io. The -sSL option is composed of three flags:


- -s tells curl to download the file in ‘silent mode’
- -S tells curl to show an error message if it fails
- -L tells curl to follow all HTTP redirects while retrieving the installation script

Once downloaded, the script is piped to bash. The -s option passes stable as an argument to the RVM Installation script to download and install the stable release of RVM.



Note: If the second command fails with the message “GPG signature verification failed”, that means the GPG Key has changed, simply copy the command from the error output and run it to download the signatures. Then run the curl command for the RVM Installation.

We need to load the RVM script (as a function) so we can start using it. We then need to run the requirements command to automatically install required dependencies and files for RVM and Ruby to function properly:


```
source ~/.rvm/scripts/rvm
rvm requirements


```


We can now install the Ruby of our choice. We’ll be installing the latest Ruby 2.2.1 (at the time of writing) as our default Ruby:


```
rvm install 2.2.1
rvm use 2.2.1 --default


```


# Step 4 — Installing Rails and Bundler


Once Ruby is set up, we can start installing Rubygems. We’ll start by installing the Rails gem that will allow your Rails application to run, and then we’ll install bundler which can read your app’s Gemfile and automatically install all required gems.


To install Rails and Bundler:


```
gem install rails -V --no-ri --no-rdoc
gem install bundler -V --no-ri --no-rdoc


```


Three flags were used:


- -V (Verbose Output): Prints detailed information about Gem installation
- --no-ri - (Skips Ri Documentation): Doesn’t install Ri Docs, saving space and making installation fast
- --no-rdoc - (Skips RDocs): Doesn’t install RDocs, saving space and speeding up installation


Note: You can also install a specific version of Rails according to your requirements by using the -v flag:
gem install rails -v '4.2.0' -V --no-ri --no-rdoc



# Step 5 — Setting up SSH Keys


Since we want to set up smooth deployments, we’ll be using SSH Keys for authorization. First shake hands with GitHub, Bitbucket, or any other Git Remote where the codebase for your Rails app is hosted:


```
ssh -T git@github.com
ssh -T git@bitbucket.org


```


Don’t worry if you get a Permission denied (publickey) message. Now, generate a SSH key (a Public/Private Key Pair) for your server:


```
ssh-keygen -t rsa 


```


Add the newly created public key (~/.ssh/id_rsa.pub) to your repository’s deployment keys:


- Instructions for Github
- Instructions for Bitbucket

If all the steps were completed correctly, you should now be able to clone your git repository (over the SSH Protocol, not HTTP) without entering your password:


```
git clone git@example.com:username/appname.git


```


If you need a sample app for testing, you can fork the following test app specifically created for this tutorial: Sample Rails App on GitHub


The git clone command will create a directory with the same name as your app. For example, a directory named testapp_rails will be created.


We are cloning only to check if our deployment keys are working, we don’t need to clone or pull our repository every time we push new changes. We’ll let Capistrano handle all that for us. You can now delete this cloned directory if you want to.


Open a terminal on your local machine. If you don’t have a SSH Key for your local computer, create one for it as well. In your local terminal session:


```
ssh-keygen -t rsa 


```


Add your local SSH Key to your Droplet’s Authorized Keys file (remember to replace the port number with your customized port number):


```
cat ~/.ssh/id_rsa.pub | ssh -p your_port_num deploy@your_server_ip 'cat >> ~/.ssh/authorized_keys'


```


# Step 6 — Adding Deployment Configurations in the Rails App


On your local machine, create configuration files for Nginx and Capistrano in your Rails application. Start by adding these lines to the Gemfile in the Rails App:


Gemfile
```

group :development do
    gem 'capistrano',         require: false
    gem 'capistrano-rvm',     require: false
    gem 'capistrano-rails',   require: false
    gem 'capistrano-bundler', require: false
    gem 'capistrano3-puma',   require: false
end

gem 'puma'

```


Use bundler to install the gems you just specified in your Gemfile. Enter the following command to bundle your Rails app:


```
bundle


```


After bundling, run the following command to configure Capistrano:


```
cap install


```


This will create:


- Capfile in the root directory of your Rails app
- deploy.rb file in the config directory
- deploy directory in the config directory

Replace the contents of your Capfile with the following:


Capfile
```
# Load DSL and Setup Up Stages
require 'capistrano/setup'
require 'capistrano/deploy'

require 'capistrano/rails'
require 'capistrano/bundler'
require 'capistrano/rvm'
require 'capistrano/puma'

# Loads custom tasks from `lib/capistrano/tasks' if you have any defined.
Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }

```


This Capfile loads some pre-defined tasks in to your Capistrano configuration files to make your deployments hassle-free, such as automatically:


- Selecting the correct Ruby
- Pre-compiling Assets
- Cloning your Git repository to the correct location
- Installing new dependencies when your Gemfile has changed

Replace the contents of config/deploy.rb with the following, updating fields marked in red with your app and Droplet parameters:


config/deploy.rb
```

# Change these
server 'your_server_ip', port: your_port_num, roles: [:web, :app, :db], primary: true

set :repo_url,        'git@example.com:username/appname.git'
set :application,     'appname'
set :user,            'deploy'
set :puma_threads,    [4, 16]
set :puma_workers,    0

# Don't change these unless you know what you're doing
set :pty,             true
set :use_sudo,        false
set :stage,           :production
set :deploy_via,      :remote_cache
set :deploy_to,       "/home/#{fetch(:user)}/apps/#{fetch(:application)}"
set :puma_bind,       "unix://#{shared_path}/tmp/sockets/#{fetch(:application)}-puma.sock"
set :puma_state,      "#{shared_path}/tmp/pids/puma.state"
set :puma_pid,        "#{shared_path}/tmp/pids/puma.pid"
set :puma_access_log, "#{release_path}/log/puma.error.log"
set :puma_error_log,  "#{release_path}/log/puma.access.log"
set :ssh_options,     { forward_agent: true, user: fetch(:user), keys: %w(~/.ssh/id_rsa.pub) }
set :puma_preload_app, true
set :puma_worker_timeout, nil
set :puma_init_active_record, true  # Change to false when not using ActiveRecord

## Defaults:
# set :scm,           :git
# set :branch,        :master
# set :format,        :pretty
# set :log_level,     :debug
# set :keep_releases, 5

## Linked Files & Directories (Default None):
# set :linked_files, %w{config/database.yml}
# set :linked_dirs,  %w{bin log tmp/pids tmp/cache tmp/sockets vendor/bundle public/system}

namespace :puma do
  desc 'Create Directories for Puma Pids and Socket'
  task :make_dirs do
    on roles(:app) do
      execute "mkdir #{shared_path}/tmp/sockets -p"
      execute "mkdir #{shared_path}/tmp/pids -p"
    end
  end

  before :start, :make_dirs
end

namespace :deploy do
  desc "Make sure local git is in sync with remote."
  task :check_revision do
    on roles(:app) do
      unless `git rev-parse HEAD` == `git rev-parse origin/master`
        puts "WARNING: HEAD is not the same as origin/master"
        puts "Run `git push` to sync changes."
        exit
      end
    end
  end

  desc 'Initial Deploy'
  task :initial do
    on roles(:app) do
      before 'deploy:restart', 'puma:start'
      invoke 'deploy'
    end
  end

  desc 'Restart application'
  task :restart do
    on roles(:app), in: :sequence, wait: 5 do
      invoke 'puma:restart'
    end
  end

  before :starting,     :check_revision
  after  :finishing,    :compile_assets
  after  :finishing,    :cleanup
  after  :finishing,    :restart
end

# ps aux | grep puma    # Get puma pid
# kill -s SIGUSR2 pid   # Restart puma
# kill -s SIGTERM pid   # Stop puma

```


This deploy.rb file contains some sane defaults that work out-of-the-box to help you manage your app releases and automatically perform some tasks when you make a deployment:


- Uses production as the default environment for your Rails app
- Automatically manages multiple releases of your app
- Uses optimized SSH options
- Checks if your git remotes are up to date
- Manages your app’s logs
- Preloads the app in memory when managing Puma workers
- Starts (or restarts) the Puma server after finishing a deployment
- Opens a socket to the Puma server at a specific location in your release

You can change all options depending on your requirements. Now, Nginx needs to be configured. Create config/nginx.conf in your Rails project directory, and add the following to it (again, replacing with your parameters):


config/nginx.conf
```

upstream puma {
  server unix:///home/deploy/apps/appname/shared/tmp/sockets/appname-puma.sock;
}

server {
  listen 80 default_server deferred;
  # server_name example.com;

  root /home/deploy/apps/appname/current/public;
  access_log /home/deploy/apps/appname/current/log/nginx.access.log;
  error_log /home/deploy/apps/appname/current/log/nginx.error.log info;

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  try_files $uri/index.html $uri @puma;
  location @puma {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;

    proxy_pass http://puma;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 10M;
  keepalive_timeout 10;
}

```


Like the previous file, this nginx.conf contains defaults that work out-of-the-box with the configurations in your deploy.rb file. This listens for traffic on port 80 and passes on the request to your Puma socket, writes nginx logs to the ‘current’ release of your app, compresses all assets and caches them in the browser with maximum expiry, serves the HTML pages in the public folder as static files, and sets default maximum Client Body Size and Request Timeout values.


# Step 7 — Deploying your Rails Application


If you are using your own Rails app, commit the changes you just made, and push them to remote from your Local Machine:


```
git add -A
git commit -m "Set up Puma, Nginx & Capistrano"
git push origin master


```



Note: If this is the first time using GitHub from this system, you might have to issue the following commands with your GitHub username and email address:
git config --global user.name 'Your Name'
git config --global user.email you@example.com



Again, from your local machine, make your first deployment:


```
cap production deploy:initial


```


This will push your Rails app to the Droplet, install all required gems for your app, and start the Puma web server. This may take anywhere between 5-15 minutes depending on the number of Gems your app uses. You will see debug messages as this process occurs.


If everything goes smoothly, we’re now ready to connect your Puma web server to the Nginx reverse proxy.


On the Droplet, Symlink the nginx.conf to the sites-enabled directory:


```
sudo rm /etc/nginx/sites-enabled/default
sudo ln -nfs "/home/deploy/apps/appname/current/config/nginx.conf" "/etc/nginx/sites-enabled/appname"


```


Restart the Nginx service:


```
sudo service nginx restart


```


You should now be able to point your web browser to your server IP and see your Rails app in action!


# Normal Deployments


Whenever you make changes to your app and want to deploy a new release to the server, commit the changes, push to your git remote like usual, and run the deploy command:


```
git add -A
git commit -m "Deploy Message"
git push origin master
cap production deploy


```



Note: If you make changes to your config/nginx.conf file, you’ll have to reload or restart your Nginx service on the server after deploying your app:
sudo service nginx restart



# Conclusion


Okay, so by now you would be running a Rails app on your Droplet with Puma as your Web Server as well as Nginx and Capistrano configured with basic settings. You should now take a look at other docs that can help you optimize your configurations to get the maximum out of your Rails application:


- Puma Configurations
- Puma DSL in Capistrano
- Local Asset Pre-compilation in Capistrano
- Automatically restart Puma workers based on RAM
- Optimizing Nginx for High Traffic Loads

