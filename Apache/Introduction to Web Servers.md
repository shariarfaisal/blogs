# Introduction to Web Servers

```Apache``` ```Nginx``` ```Conceptual```

## Introduction


A web server’s primary role is to serve web pages for a website. A web page can be rendered from a single HTML file, or a complex assortment of resources fitted together. If you want to host your web application on the internet, in many cases you will need a web server.


One of the most common use cases for web servers is serving files necessary for rendering a website in a browser. When you visit http://www.digitalocean.com, you begin with entering a URL that starts a request over the internet. This request passes through multiple layers, one or more of which will be a web server. This web server generates a response to your request, which in this case is the DigitalOcean website, specifically the homepage. Ideally, this happens quickly and with 24/7 availability.


While any visitor to DigitalOcean’s homepage will experience it as a single web page, in reality most modern web pages today are a combination of many resources. Web servers act as an intermediary between the backend and the frontend, serving up resources like HTML and CSS files to JSON data, all generated dynamically on the fly or served statically. If you intend to work with websites or online apps in any capacity, it is extremely useful to familiarize yourself with the basics of what a web server is, and how it works.


While the term “web server” can refer to either the software itself or the hardware it exists on, this article refers specifically to web server software. For more details on this difference, check out our introduction to cloud servers.


# Common Use Cases


A web server handles requests on the internet through HTTP and HTTPS protocol, and is also called an HTTP server. A web server is distinct from other types of servers in that it specializes in handling these HTTP and HTTPS requests, differentiating itself from application servers (e.g. Gunicorn) and servers for other protocols (i.e. WSGI). These other servers work as intermediaries for backend programming languages through external libraries, which is a different level of abstraction than web servers.


Here are some common tasks handled by web servers:


- Serves HTML, CSS, and JavaScript files.
- Serves images and videos.
- Handles HTTP error messaging.
- Handles user requests, often concurrently.
- Directs URL matching and rewriting.
- Processes and serves dynamic content.
- Compresses content for optimized data usage and speed.
- Enables browser caching for your static content.

In practical terms, here are some personal projects that would involve a web server:


- You want to make a website.
- You want to make an app that connects to the internet.

This list is by no means comprehensive, and a web server is not strictly limited in the data types it can serve to an end user. For example, a web server that serves web API requests often responds with data in a format such as JSON.


# Goals of a Web Server


Web servers cater to an audience with expectations of speed, availability, reliability, and more. They have a shared purpose of serving content on the internet, and in order to be considered a viable web server solution, the following aspects must be considered:


- Uptime: This refers to the time a web server is online and operational. Websites need to be online at all times to serve users, so a high uptime is the goal. This also translates to stability and predictability. When a user enters a URL or clicks a link to your website, the expected page should load every time, and at any given time. The only exceptions should be planned downtimes for updates or maintenance. A web server that is buggy or crashes at random times adversely affects your users’ experience.
- Speed: Your web pages should load as fast as possible. Users want their request fulfilled immediately, otherwise you risk losing them. On a slow loading web page, even if the user sits through the first load, every subsequent long load will exponentially decrease their willingness to stay or visit again.
- Concurrency: This refers to the handling of multiple requests coming in at the same time. Having too many people trying to visit your website at once seems like a good thing, but this becomes a real problem when load times slow down to a crawl and your whole server crashes. Your physical or virtual server only has so many resources such as RAM and CPU compute power, and web servers must use these resources efficiently.
- Scalability: Scalability refers to either making your existing servers more powerful through vertical scaling, or adding more servers to your setup through horizontal scaling. As you grow your audience, you may reach a point where you need more than one or two small web servers.
- Ease of set up: Getting a project up and running quickly is key to the iteration of your project. A straightforward and repeatable install process is important for the first web server you set up, and the multiple web servers afterwards when you scale up.
- Documentation: Web servers are complex. The most common setups will get you on your feet quickly, but your needs will grow over time. Oftentimes you will need features that are not as commonly used. When that time comes, good documentation is essential to creating custom solutions for your needs.
- Developer support: If the core developers are not committed to their own project, you shouldn’t commit your project to theirs. This includes both plans for long term support for their software, along with immediate short term support they provide in the form of bug fixes and patches.
- Community support: A core development team will handle most of the heavy lifting, but a thriving community contributes to filling in the gaps. With open source projects, this can mean contributions to the actual code base, but a strong community will also answer your questions and help with your specific issues.

While web servers can offer different solutions, the solutions they offer stem from attempts to address these same problems. These problems themselves evolve over time along with the needs and expectations of the end user, making this a living and ever evolving list.


# Selecting a Web Server Solution


The most popular open source web servers are currently Apache and Nginx.


Apache came first, and was built at a time when it was common for multiple websites with their own individual configuration files to all exist on a single web server. Nginx came after Apache, at a time when needs shifted away from serving multiple websites from one server, and instead towards serving one website from one server in an extremely efficient manner under load.


While web servers share the same goals and problems, each solution’s interpretation and implementation will be different. The exact answers to these problems shape the identity of any given web server solution. Nginx and Apache are highlighted here due to their ubiquity, but any web server solution will be opinionated. When selecting a web server, it’s important to keep your own needs in mind for your specific project. That way, even if the landscape of web server offerings change, your method of evaluation stays grounded by your own requirements.


Here are some key differentiators in how web servers attempt to accomplish the goals of a web server:


## Structure of Configuration Files


Web servers store their settings in configuration files. You can customize your web servers by editing these files. The storing and organization of configuration files is an opinionated, structural issue that splits web server products.


The main divide is between centralization and decentralization. Decentralized configuration files allows for a granular level of control on a filesystem level, which stems from a need to host multiple websites on one server. Centralized configurations don’t focus on hosting multiple websites on one server, and instead focuses on efficiently serving a single website. These configurations rely on URI pattern matching, which is the matching of URLs to filenames and other unique identifiers, instead of relying on matching against the directory structure of a web server.


Apache’s .htaccess files facilitates a decentralized configuration as a feature, and every design decision flows from this focus on the filesystem with a granular level of control. Nginx does not have the same filesystem focus, and focuses on URI pattern matching and a centralized configuration.


## Handling Concurrency


The physical and virtual servers you run web servers on have limited resources such as RAM and CPU processing. How your web server fundamentally manages its requests will have the largest impact on efficiently using your resources. A single request can spawn an entire process per request, or it can be handled on an event-driven basis. The capacity of your web server to handle multiple simultaneous requests efficiently is tied to fundamental design decisions.


Apache handles requests by spawning processes, which consumes resources at a rate that can become a problem under load. Nginx’s event-driven system handling system uses less resources and can be more performant under load.


## Serving Static Content


Besides web pages, web servers get requests for other resources such as images, videos, CSS files, and JavaScript files. Since these items are always the same regardless of who requests them, this type of content is referred to as static. Oftentimes a web page itself is just an HTML file that isn’t customized to the person requesting it, and is also treated as static content. Web servers can also compress this static content for better load times.


Nginx excels as serving static content due to its event-driven request handling. Apache can also serve static content, but in most setups, not at the same speed and efficiency under load compared to Nginx.


## Serving Dynamic Content


When content is changed, processed, and customized depending on who is requesting it, the content is referred to as dynamic. For example, after you log in to a website, often the website will dynamically populate your username in the top navigation bar. Dynamic content adds extra complexity because it forces the web server to handle many requests uniquely at the time it receives it. Content tailored per request cannot be served to everyone, and cannot be universally cached.


Processing dynamic content internally removes an extra layer of abstraction that would normally require handing off the request to an external library. Apache natively implements dynamic content processing, with popular solution stacks such as LAMP (Linux, Apache, MySQL, PHP). Nginx is more language agnostic but requires external libraries such as PHP-FPM to act as a similar solution for use cases such as LAMP stack.


## Reverse Proxy Capability


A reverse proxy sits in front of a traditional web server, becoming an intermediary server that routes HTTP request traffic to web servers behind it. A reverse proxy becomes the gateway that directs traffic between web servers and the internet at large, and often is the layer that directly interfaces with a firewall. While most web servers have reverse proxy capability, Nginx was built and optimized from the ground up to be a robust reverse proxy server.


Nginx’s importance in real world usage hinges heavily on its reverse proxy features and efficiency. Many server setups place multiple traditional web servers behind an Nginx reverse proxy, using Nginx to determine which web server to send the request to based on load or rule configuration. This intermediary role allows it to even pair with Apache in some setups, sitting as a reverse proxy in front of a traditional Apache web server.


## Support Ecosystem


Nginx and Apache both have strong support from their respective development teams and community. Being the most popular open source web servers, learning resources are plentiful. Apache is supported by the Apache, a non-profit organization, and will always be free to use. Nginx’s core is open source, but desirable features are locked behind their Nginx Plus product offering with features such as upstream health checks, session persistence, and advanced monitoring.


# Alternatives to the Traditional Web Server


If you want a server that is ready at all times to respond to an incoming HTTP request, then a web server accomplishes this task best. As you stray further from focusing on serving HTTP requests, web servers will become less of an ideal solution. This is especially true for the auxiliary features that web servers provide. For example, features such as caching may be more efficiently handled at the reverse proxy or CDN levels, depending on the setup.


Additionally, as developers have shifted their priorities in dedicating development resources to managing web servers, solutions such as serverless, headless CMS, and Jamstack have emerged in response. These solutions don’t require a self-hosted web server, instead abstracting out the web server layer to external services. For developers who don’t require granular or advanced control of the web server layer, development time can be focused elsewhere. For more, check out this article on Jamstack with headless CMS or implementing full stack Jamstack with DigitalOcean’s App Platform.


# Conclusion


In this article, you’ve gone through a basic primer of what web servers are, how they’re used, and the problems they’re trying to solve. Equipped with this knowledge, you dove into the current landscape of web server solutions, and applied your knowledge towards finding the solution that fits your needs specifically. To learn more about how to set up and use a web server, check out the rest of our Cloud Curriculum series on web servers.


## Additional Resources


Tutorials:


- How to Install Apache: Step-by-step instructions for setting up your first Apache server. This solution excels with decentralized configuration for granular control, and internal handling of dynamic web pages with hooks into popular programming languages such as PHP.
- How to Install Nginx: Step-by-step instructions for setting up your first Nginx server. This solution excels with centralized configuration, serving of static assets, acting as a reverse proxy, and handling high concurrency traffic.
- Apache vs Nginx: Practical Considerations: A more in-depth look at the two major players in the web server solution landscape.

DigitalOcean Products:


- DigitalOcean Droplets: Virtual private servers for you to test and deploy web servers.
- DigitalOcean Functions: Serverless solution that can be an alternative to virtual private servers. Skip the maintenance of servers, focus on your application code.

