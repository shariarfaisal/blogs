# How To Install Ruby on Rails on an Debian 7 0  Wheezy  VPS using RVM

```Ruby on Rails``` ```Debian```

## Introduction



Ruby on Rails is one of the most popular full-stack web frameworks for easily getting your application or website up and running.


There are a number of ways to install a Rails stack on a Debian machine.  This guide will cover how to do so with RVM (Ruby Version Manager).


RVM allows you to easily manage multiple Ruby versions and provides a great upgrade path when a newer version becomes stable.


# Step One –– Install RVM



We will be installing RVM from the project’s website, so we will not be relying on Debian’s default repositories for the actual installation.


However, RVM uses the package manager to install dependencies for the programs it manages, so we should make sure those repositories are up-to-date:


```
sudo apt-get update

```


Next, we will install RVM, Ruby, and Rails in a single command.  This assumes that you want a single-user installation, which will be applicable in most cases.  Do this as a regular user, not as root:


```
\curl -L https://get.rvm.io | bash -s stable --rails

```


This will take awhile to finish.  When complete, you will have a stable Ruby instance and the Rails framework installed.


Once the process is complete, we need to make our environment aware of the new RVM installation.  This will happen automatically when we open new terminal windows from now on:


```
source ~/.rvm/scripts/rvm

```


# Step Two –– Get Ruby Environment Information



You can easily get info from RVM about what Ruby version is being used currently and what the file paths are for related applications and services.


Run the info command to get these details:


```
rvm info

```



```
ruby-2.0.0-p247:

  system:
    uname:       "Linux RVM 3.2.0-4-amd64 #1 SMP Debian 3.2.41-2+deb7u2 x86_64 GNU/Linux"
    system:      "debian/7/x86_64"
    bash:        "/bin/bash => GNU bash, version 4.2.37(1)-release (x86_64-pc-linux-gnu)"
    zsh:         " => not installed"
    . . .

```


We can find info about the current and available Ruby versions installed by issuing the command:


```
rvm list

```



```
rvm rubies

=* ruby-2.0.0-p247 [ x86_64 ]

# => - current
# =* - current && default
#  * - default

```


To find other Ruby versions that are available for installation, type:


```
rvm list known

```



```
# MRI Rubies
[ruby-]1.8.6[-p420]
[ruby-]1.8.7[-p374]
[ruby-]1.9.1[-p431]
[ruby-]1.9.2[-p320]
[ruby-]1.9.3[-p448]
[ruby-]2.0.0-p195
[ruby-]2.0.0[-p247]
. . .

```


You can then install the desired Ruby by typing:


<pre>
rvm install <span class=“highlight”>version_of_ruby</span>
</pre>


To switch over to a different Ruby, type:


<pre>
rvm use <span class=“highlight”>version_of_ruby</span>
</pre>


# Step Three –– Create a Sample Rails App



You can test that Rails is installed correctly by creating a sample app, which will live in your home directory.


This will create an application called sample:


```
cd ~
rails new sample

```


We need to enter the app directory in order to have the correct environment for launching the Rails development server:


```
cd sample

```


You can try to start the server now, but it will not execute correctly:


```
rails s

```



```
/home/demo/.rvm/gems/ruby-2.0.0-p247/gems/execjs-2.0.2/lib/execjs/runtimes.rb:51:in `autodetect': Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes. (ExecJS::RuntimeUnavailable)
	from /home/demo/.rvm/gems/ruby-2.0.0-p247/gems/execjs-2.0.2/lib/execjs.rb:5:in `<module:ExecJS>'

```


Rails isn’t finding an appropriate JavaScript executable.  We will install two gems (Ruby packages) that will provide the necessary dependencies, into our application environment.


Edit the Gemfile file, which contains the specifications of what the application requires:


```
nano Gemfile

```


Add the following two lines at the end:


```
gem 'execjs'
gem 'therubyracer'

```


Save and close the file.


Now, we will run the command that will ensure that all of those dependencies are installed:


```
bundle install

```


Finally, we can run the Rails development server to take a look at our app:


```
rails s

```


Point your browser to your server’s IP address, followed by “:3000”:


<pre>
<span class=“highlight”>your_IP_or_domain</span>:3000
</pre>


You should see the default Rails page:


<img style=“border:2px solid black; display:block;margin-left:auto;margin-right:auto” src=“https://assets.digitalocean.com/articles/rvm_debian/rails_page.png” alt =“RVM Rails default page” />


This means you’re well on your way to deploying your app or website.  Congratulations, you have Ruby on Rails installed!


# Conclusion



Although there are several ways to install Ruby and Rails on Debian, this is probably the most preferred method, due to its flexibility and its insistence on keeping application environments separated.


RVM has many additional features that can provide structure and control over your development environment.  The project’s site provides great documentation on how to best utilize this powerful tool.


<div class=“author”>By Justin Ellingwood</div>


