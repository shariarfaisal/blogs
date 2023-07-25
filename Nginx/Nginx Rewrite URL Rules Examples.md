# Nginx Rewrite URL Rules Examples

```Nginx```

NGINX rewrite rules are used to change entire or a part of the URL requested by a client. The main motive for changing an URL is to inform the clients that the resources they are looking for have changed its location apart from controlling the flow of executing pages in NGINX. The return and rewrite directives in NGINX are used to rewrite URL. Both the directives perform the same function of rewriting URL. However, the rewrite directive is more powerful than the return directive since complex rewriting that requires parsing of URL can be done with rewrite directive only. In this tutorial, we will explore how both return and rewrite directives are used in NGINX to change or rewrite the URL.


# NGINX Return directive


The easiest and cleaner way to rewrite an URL can be done by using the return directive. The return directive must be declared in the server or location context by specifying the URL to be redirected.


## 1. NGINX Return directive in Server context


The return directive in server context is very useful in a situation where you have migrated your site to a new domain and you want to redirect all old URLs to the new domain. Further, it also helps in canonicalization of URL by forcing your site to redirect to either www or non-www version.


```
server {
        listen 80;
        server_name www.olddomain.com;
        return 301 $scheme://www.newdomain.com$request_uri;
}

```


The return directive in the above server context redirect URL destined to site www.olddomain.com to www.newdomain.com. As soon as NGINX receives an URL with www.olddomain.com, it stops processing the page and sends a 301 response code along with rewritten URL to the client. The two variables used in the above return directive are $scheme and $request_uri. The variable $scheme is used to define scheme of the URL (http or https) and the variable $request_uri contains complete URI with parameters if any. Remember both the variable fetches this information from input URL while rewriting the URL.


## 2. Return directive in Location context


In some situation, you may want to redirect pages instead of redirecting domains. The return directive inside the location block enables you to redirect specific pages to a new location.


```
location = /tutorial/learning-nginx {
     return 301 $scheme://example.com/nginx/understanding-nginx
}

```


In the above example, whenever a request URI matches exactly with pattern /tutorial/learning-nginx, NGINX will redirect the same to the new location https://example.com/nginx/understanding-nginx/ You can also redirect everything for a specific path to a new location. The following example shows how to redirect all pages those falls under /tutorial to https://example.com/articles.


```
location /tutorial {
     return 301 $scheme://example.com/articles
}

```


# NGINX Rewrite directive


We can also use rewrite directive to rewrite URL in NGINX. Like return directive, rewrite directive can also be placed in server context as well as in location context. The rewrite directive can perform complicated distinctions between URLs and fetch elements from the original URL that don’t have corresponding NGINX variables thereby making it more useful than return directive. The syntax of rewrite directive is:


```
rewrite regex replacement-url [flag];

```


- regex: The PCRE based regular expression that will be used to match against incoming request URI.
- replacement-url: If the regular expression matches against the requested URI then the replacement string is used to change the requested URI.
- flag: The value of flag decides if any more processing of rewrite directive is needed or not.

Remember, The rewrite directive can return only code 301 or 302. To return other codes, you need to include a return directive explicitly after the rewrite directive


# NGINX Rewrite directive examples


Let us quickly check few rewrite rules to get you started with it starting from rewriting a simple html page to another URL:


## 1. Rewrite static page


Consider a scenario where you want to rewrite an URL for a page say https://example.com/nginx-tutorial to https://example.com/somePage.html. The rewrite directive to do the same is given in the following location block.


```
server {
          ...
          ...
          location = /nginx-tutorial 
          { 
            rewrite ^/nginx-tutorial?$ /somePage.html break; 
          }
          ...
          ...
}

```


Explanation:


- 
The location directive location = /nginx-tutorial tells us that the location block will only match with an URL containing the exact prefix which is /nginx-tutorial.

- 
The NGINX will look for the pattern ^/nginx-tutorial?$ in the requested URL.

- 
To define the pattern, the characters ^,? and $ are used and have special meaning.

- 
^ represents the beginning of the string to be matched.

- 
$ represents the end of the string to be matched.

- 
? represents non greedy modifier. Non greedy modifier will stop searching for pattern once a match have been found.

- 
If the requested URI contains the above pattern then somePage.html will be used as a replacement.

- 
Since the rewrite rule ends with a break, the rewriting also stops, but the rewritten request is not passed to another location.


## 2. Rewrite dynamic page


Now consider a dynamic page https://www.example.com/user.php?id=11 where the dynamic part is id=11(userid). We want the URL to be rewritten to https://exampleshop.com/user/11. If you have 10 users then there is a need of 10 rewrite rules for every users if you follow the last method of rewriting URLs. Instead, It is possible to capture elements of the URL in variables and use them to construct a single rewrite rule that will take care of all the dynamic pages.


```
server {
          ...
          ...
          location = /user.php 
          { 
            rewrite user.php?id=$1 ^user/([0-9]+)/?$ break; 
          }
          ...
          ...
}

```


Explanation:


- The location directive location = /user tells NGINX to match the location block with an URL containing the exact prefix which is /user.
- The NGINX will look for the pattern ^user/([0-9]+)/?$ in the requested URL.
- The regular expression within square bracket [0-9]+ contains a range of characters between 0 and 9. The + sign signifies matching one or more of the preceding characters. Without the + sign, the above regular expression will match with only 1 character like 5 or 8 but not with 25 or 44.
- The parenthesis ( ) in the regular expression refers to the back-reference. The $1 in the replacement URL user.php?id=$1 refers to this back-reference.

For example, if https://www.example.com/user/24 is the input URL then the user id 24 will match with the range in the back-reference resulting in the following substitution: https://www.example.com/user.php?id=24


## 3. Advance URL Rewriting


Let us proceed with another example where we want the URL https://www.example.com/user.php?user_name=john to be rewritten to https://www.example.com/user/login/john. Unlike previous rewrite rule, The dynamic part of the URL user_name=john now contains range of alphabetic characters. The rewrite rule for this scenario is given below:


```
server {
          ...
          ...
          location = /user.php 
            { 
                rewrite user.php?user_name=$1 ^user/login/([a-z]+)/?$ break;           
            }
          ...
          ...
  }

```


Explanation:


- The location directive location = /user/login/john tells NGINX to match the location block with an URL containing the exact prefix which is /user/login/john.
- The NGINX will look for the pattern ^user/login/([a-z]+)/?$ in the requested URL.
- The regular expression within square bracket [a-z]+ contains range of characters between a to z. The + sign signifies matching one or more of the preceding characters. Without + sign, the above regular expression will match with only 1 character like a or c but not with john or doe.
- The parenthesis ( ) in the regular expression refers to the back-reference. The $1 in the replacement URL user.php?user_name=$1 refers to this back-reference.

For example, if the input URL is https://www.example.com/user/login/john then the user name “john” will match with the range in the back-reference resulting in the following substitution: https://www.example.com/user.php?user\_name=john


## 4. Rewrite with multiple back reference


In this example, we will also find out how to rewrite an URL by using multiple backreferences. Let us assume the input URL is https://example.com/tutorial/linux/wordpress/file1 and we want to rewrite the URL to https://example.com/tutorial/linux/cms/file1.php. If you closely look at the input URL, it starts with /tutorial, and somewhere later in the path the string wordpress needs to be replaced with string cms. Further, a file extension (php) also needs to be appended at the end of the filename. The rewrite rule for this scenario is given below:


```
server {
          ...
          ...
          location /tutorial
          {
             rewrite ^(/tutorial/.*)/wordpress/(\w+)\.?.*$ $1/cms/$2.php last;
          }
          ...
          ...
  }

```


Explanation:


- The first back reference ^(/tutorial/.*) in the regular expression used to match any input URL starting with /tutorial/foo
- The second back reference (\w+) is used to capture the file name only without extension.
- The above two back references are used in the replacement URL using $1 and $2
- The last keyword instructs NGINX to stop parsing of more rewrite conditions, even on the next location match !

# Summary


You can now rewrite URL using either rewrite or return directive. The rewrite examples used in this tutorial are simple and easy to understand. You can now proceed with writing more complex rewrite rules!


