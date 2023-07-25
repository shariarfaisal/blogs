# How To Use Caching with the Yii Framework To Improve Performance

```PHP Frameworks``` ```PHP``` ```Caching```

Amongst the many features that the incredible Yii framework offers, a cache management system is something that couldn’t have been missing.


Yii framework allows us to save both static data and your SQL/Active Record queries, which -if used wisely- can lead to a lot of page loading time saving.


In particular, in this tutorial we are going to learn how to cache Data and Queries.


So here’s how we enable cache in Yii.


# Activate the Cache Component



The first step consists in activating the cache component. Just open your configuration file (located under protected/config/), go to the components array, and add the following code right within the array:


```
'cache'=>array( 
    'class'=>'system.caching.CDbCache'
)

```


By doing so, we are choosing to use CDbCache, which is just one of the Caching Components available with Yii. This particular one stores the cached data in a SQLite database, which makes it extremely easy to set up. And while not being the best choice in terms of performance, it will still make our web application slightly faster.


Another viable and more powerful option is to use the CApcCache component, which makes use of APC, the built-in caching system that comes with the newest versions of PHP.


Since all these Cache components are based on top of the CCache class, you can easily switch from a cache component to another by changing the name of the component (e.g. system.caching.CApcCache), while not having to change any code throughout your application.


# Simple Data Caching



The first and simplest way to use cache is by storing variables. To do that, Yii’s cache component gives you two functions: get() and set().


So we start by setting a value to be cached. To do so, we will also have to assign it a unique ID. For example:


```
// Storing $value in Cache
$value = "This is a variable that I am storing";
$id    = "myValue";
$time  = 30; // in seconds

Yii::app()->cache->set($id, $value, $time);

```


The last value, $time, is not required, although useful in order to avoid storing a value forever when it’s not necessary.


Getting the stored value is trivial:


```
Yii::app()->cache->get($id);

```


Should the value not be found (because it does not exist or because it did expire before), this function will return a false value. Thus, for example, a nice way of checking if a certain value is cached would be:


```
$val = Yii::app()->cache->get($id);
if (!$val):
    // the value is not cached, do something here
else:
    // the value is cached, do something else here
endif;

```


## Delete a cached value



To delete a value that is stored in cache, we can call:


```
Yii::app()->cache->delete($id);

```


If what we need is to clean everything, we will just write:


```
Yii::app()->cache->flush();

```


# Query Caching



Built on top of the Data Caching system, this is a very useful feature, especially for heavy apps that rely intensely on a Database.


The concept of this feature is fairly easy but pretty solid.


Firstly, what we have to do is to define a dependency query. In other words, we define a much simpler and lighter Database Query that we will call before the one that we really need. The reason for doing that is to check if anything has changed since the last time that we executed that query.


If, for example, the data we want to retrieve is a list of Book Authors, our dependency query might well be:


```
SELECT MAX(id) FROM authors

```


By doing so, we will be able to see if any new author has been added since the last time we checked. If no new author has been added, Yii’s Cache component will take the Authors list directly from the cache, without executing again our big query, which could be something like:


```
SELECT authors.*, book.title 
FROM authors 
JOIN book ON book.id = authors.book_id

```


## Yii Query Builder



To use Query Caching with the Yii Query Builder, this is what we have to write [using the Authors’ example showed before]:


```
// big query
$query = ' SELECT authors.*, book.title 
FROM authors 
JOIN book ON book.id = authors.book_id';
// dependency query 
$dependency = new CDbCacheDependency('SELECT MAX(id) FROM authors'); 
// executing query using Yii Query Builder
$result = Yii::app()->db->cache(1000, $dependency)->createCommand($query)->queryAll();

```


The arguments passed to Yii::app()->db->cache() are, respectively, the amount of seconds that the result should be stored for and the dependency query.


As explained before, when running this code, Yii will check for the result of the dependency query before anything else. Should it not find anything, or a different value from the one stored before, it will execute the big query and store the result in cache. Otherwise it will extract the big query result from the cache.


## Active Record



It is also possible to cache the result of a query made using Active Record. The concept remains the same as explained before; but with a different syntax, of course:


```
$dependency = new CDbCacheDependency('SELECT MAX(id) FROM authors');
$authors = Author::model()->cache(1000, $dependency)->with('book')->findAll();

```


## Things to keep in mind



It’s pretty obvious that an application that makes intensive use of caching would need to be well designed in advance, since the risk of serving inconsistent data to the user will increase inevitably.


Also, don’t forget that each caching component might have limits on the amount of data that can be stored. It’s thus a good practice to find out in advance the limit of your caching system.


<div class=“author”>Submitted by: <a href=“https://twitter.com/marcotroisi”>Marco Troisi </a></div>


