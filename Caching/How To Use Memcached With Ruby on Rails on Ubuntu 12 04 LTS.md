# How To Use Memcached With Ruby on Rails on Ubuntu 12 04 LTS

```Ubuntu``` ```Ruby on Rails``` ```Caching``` ```Ruby```











# Status: Deprecated


This article covers a version of Ubuntu that is no longer supported.  If you are currently operate a server running Ubuntu 12.04, we highly recommend upgrading or migrating to a supported version of Ubuntu:


- Upgrade to Ubuntu 14.04.
- Upgrade from Ubuntu 14.04 to Ubuntu 16.04
- Migrate the server data to a supported version

Reason:
Ubuntu 12.04 reached end of life (EOL) on April 28, 2017 and no longer receives security patches or updates.  This guide is no longer maintained.


See Instead:
This guide might still be useful as a reference, but may not work on other Ubuntu releases.  If available, we strongly recommend using a guide written for the version of Ubuntu you are using.   You can use the search functionality at the top of the page to find a more recent version.


## Introduction


Memcached is a very fast in-memory object caching system that can make Rails run much faster with very few changes.


Prerequisites:


This tutorial assumes you have already installed Ruby on Rails and Memcached.  If not, the tutorials are linked below:


- How to Install Ruby on Rails on Ubuntu 12.04 LTS (Precise Pangolin) with RVM | DigitalOcean

- How to Install and Use Memcache on Ubuntu 12.04 | DigitalOcean

It also assumes that you have your Rails application up and running and ready to optimize using Memcached.


# Install the Dalli Gem


The first thing we will have to do is install Mike Perham's Dalli Gem:


```
gem install dalli
```


If you use Bundler, then add gem 'dalli' to your Gemfile and run bundle install.


This will be our super fast and feature packed way of interacting with Memcached.


# Configure Rails


The first step to configuring Rails to use memcached is to edit your config/environments/production.rb and add this line to tell Rails to use Dalli:


```
config.cache_store = :dalli_store
```


Next, we will tell ActionController to perform caching. Add this line to the same file:


```
config.action_controller.perform_caching = true
```


Now, restart your Rails application as you normally would.


# Change Your Rails Application


To take advantage of the changes we've just made, the Rails application will need to be updated. There are two major ways to take advantage of the speed up memcached will give you.


## Add Cache Control Headers


The easiest way to take advantage of memcached is to add a Cache-Control header to one of your actions. This will let Rack::Cache store the result of that action in memcached for you. If you had the following action in app/controllers/slow_controller.rb:


```
def slow_action
  sleep 15
  # todo - print something here
end

```


We can add the following line to tell Rack::Cache to store the result for five minutes:


```
def slow_action
  expires_in 5.minutes
  sleep 15
  # todo - print something here
end

```


Now, when you execute this action the second time, you'll see that it's significantly faster. Rails only has to execute it once every five minutes to update Rack::Cache.


Please note that this will set the Cache-Control header to public. If you have certain actions that only one user should see, use expires_in 5.minutes, :public => false. You will also have to determine what the appropriate time is to cache your responses, this varies from application to application.


If you would like to learn more about HTTP Caching, check out Mark Nottingham's Caching Tutorial for Web Authors and Webmasters.


## Store Objects in Memcached


If you have a very expensive operation or object that you must create each time, you can store and retrieve it in memcached. Let's say your action looks like this:


```
def slow_action
  slow_object = create_slow_object
end

```


We can store the result in memcached by changing the action like this:


```
def slow_action
  slow_object = Rails.cache.fetch(:slow_object) do 
      create_slow_object
  end
end

```


Rails will ask memcached for the object with a key of 'slow_object'; if it doesn't find that object, it will execute the block given and write the object back into it.


## Fragment Caching


Fragment caching is a Rails feature that lets you choose which parts of your application are the most dynamic and need to be optimized. You can easily cache any part of a view surrounding it in a cache block:


```
<% # app/views/managers/index.html.erb  %>
<% cache manager do %>
  Manager's Direct Reports:
  <%= render manager.employees %>
<% end %> 

<% # app/views/employees/_employee.html.erb %>
<% cache employee do %>
    Employee Name: <%= employee.name %>
    <%= render employee.incomplete_tasks %>
<% end %>

<% # app/views/tasks/_incomplete_tasks.html.erb %>
<% cache task do %>
    Task: <%= task.title %>
    Due Date: <%= task.due_date %>
<% end %>

```


The above technique is called Russian Doll caching alluding to the traditional Russian nesting dolls. Rails will then cache these fragments to memcached and since we added the model into the cache statement this cache object's key will change when the object changes. The problem this creates though is when a task gets updated:


```
@todo.completed!
@todo.save!

```


Since we are nesting cache objects inside of cache objects, Rails won't know to expire the cache fragments that rely on this model. This is where the ActiveRecord touch keyword comes in handy:


```
class Employee < ActiveRecord::Base
  belongs_to :manager, touch: true
end

class Todo < ActiveRecord::Base
  belongs_to :employee, touch: true
end

```


Now when a Todo model is updated, it will expire its cache fragments plus notify the Employee model that it should update its fragments too. Then the Employee fragment will notify the Manager model and after this, the cache expiration process is complete.


There is one additional problem that Russian Doll caching creates for us. When deploying a new application, Rails doesn't know when to check that a view template has changed. If we update our task listing view partial:


```
<% # app/views/tasks/_incomplete_tasks.html.erb %>
<% cache task do %>
    Task: <%= task.title %>
    Due Date: <%= task.due_date %>
    <p><%= task.notes %></p>
<% end %>

```


Rails won't expire the cache fragments that use view partial. Before you would have to add version numbers to your cache statements but now there is a gem called cache_digests that automatically adds in an MD5 hash of the template file to the cache key. If you update the partial and restart your application, the cache key will no longer match since the MD5 of the view template file has changed and Rails will render that template again. It also handles the dependencies between template files so, in the above example, it will expire all our cache objects up the dependency chain if the _incomplete_tasks.html.erb is updated.


This feature is automatically included in Rails version 4.0. To use this gem in your Rails 3 project, type the following command:


```
gem install cache_digests
```


Or if you use Bundler, add this line to your Gemfile:


```
gem 'cache_digests'
```


# Advanced Rails and Memcached Setup


The Dalli Ruby Gem is very powerful and takes care of spreading keys across a cluster of memcached servers, which distributes the load and increases your memcached capacity. If you have multiple web servers in your web tier, you can install memcached on each of those servers and add them all to your config/environments/production.rb:


```
config.cache_store = :dalli_store, 'web1.example.com', 'web2.example.com', 'web3.example.com'
```


This will use consistent hashing to spread the keys across the available memcached servers.


Article Submitted by: Andrew Williams
