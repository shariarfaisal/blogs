# An Introduction to Proxies

```Apache``` ```Conceptual``` ```Nginx```

## Introduction


A proxy, also called a proxy server, is a server software that sits as an intermediary between a client and server on the internet. Without a proxy, a client would send a request for a resource directly to a server, and then the server would serve the resource directly back to the client. While this approach is straightforward to understand and implement, adding proxies provides benefits in the form of increased performance, privacy, security, and more. As an additional pass-through layer, a proxy acts as a gatekeeper of the internet between clients and servers.


Generally speaking, the combined package of server hardware with installed proxy software is also often referred to as a proxy server. However, this article will focus on proxies traditionally defined as software, and in the context of web servers. You will get a breakdown of the two main types, a forward proxy and a reverse proxy. Each type has a different use case, often confused because of the similar naming convention.


This article will provide you with an understanding of what proxies and their subtypes are, and how they are useful in common setups. By reading this article, you will be able to identify the circumstances in which a proxy is beneficial, and choose the correct solution between forward proxy and reverse proxy in any given situation.


# Understanding Forward Proxies


A forward proxy, also called an open proxy, acts as a representative for a client that is trying to send a request through the internet to an origin server. In this scenario, all attempts to send requests by the client will instead be sent to the forward proxy. The forward proxy, in the client’s stead, will examine the request. First, it will determine if this client is authorized to send requests through this specific forward proxy. It will then reject the request or forward it to the origin server. The client has no direct access to the internet; it can only reach what the forward proxy allows it to access.


A common use case of forward proxies is to gain increased privacy or anonymity on the internet. A forward proxy accesses the internet in place of a client, and in that process it can use a different IP address than the client’s original IP address.


Depending on how it has been configured, a forward proxy can grant a number of features, allowing you to:


- Avoid ad tracking.
- Circumvent surveillance.
- Identify restrictions based on your geolocation.

Forward proxies are also used in systems for centralized security and permission based access, such as in a workplace. When all internet traffic passes through a common forward proxy layer, an administrator can allow only specific clients access to the internet filtered through a common firewall. Instead of maintaining firewalls for the client layer that can involve many machines with varying environments and users, a firewall can be placed at the forward proxy layer.


Keep in mind that forward proxies must be manually set up in order to be used, whereas reverse proxies can go unnoticed by the client. Depending on whether the IP address of a client is passed on to the origin server by the forward proxy, privacy and anonymity can be granted or left transparent.


There are several options to consider for forward proxies:


- Apache: A popular open-source web server that offers forward proxy functionality.
- Nginx: Another popular open-source web server with forward proxy functionality.
- Squid: An open-source forward proxy that uses the HTTP protocol. This option doesn’t include an entire web server solution. You can check out our guide on how to set up Squid proxy for private connections on Ubuntu 20.04.
- Dante: A forward proxy that uses the SOCKS protocol instead of HTTP, making it more suitable for use cases such as with peer-to-peer traffic. You may also want to check out how to set up Dante proxy for private connections on Ubuntu 20.04

# Understanding Reverse Proxies


A reverse proxy acts as a representative of a web server, handling incoming requests from clients on its behalf. This web server can be a single server or multiple servers. Additionally, it can be an application server such as Gunicorn. In either scenario, a request would come in from a client through the internet at large. Normally, this request will go directly to the web server that has the resources the client is requesting. Instead, a reverse proxy acts as an intermediary, isolating the web server from direct interaction with the open internet.


From a client’s perspective, interacting with a reverse proxy is no different from interacting with the web server directly. It is functionally the same, and the client cannot tell the difference. The client requests a resource and then receives it, without any extra configuration required by the client.


Reverse proxies grant features such as:


- Centralized security for the web server layer.
- Directing incoming traffic based on rules you can configure.
- Added functionality for caching.

While centralized security is a benefit of both forward and reverse proxies, reverse proxies provide this to the web server layer and not the client layer. Instead of focusing on maintaining firewalls at the web server layer, which may contain multiple servers with different configurations, the majority of firewall security can be focused at the reverse proxy layer. Additionally, removing the responsibility of interfacing with a firewall and interfacing with client requests away from web servers allows them to focus solely on serving resources.


In the case of multiple servers existing behind a reverse proxy, the reverse proxy also handles directing which requests go to which server. Multiple web servers might be serving the same resource, each serving different kinds of resources, or some combination of the two. These servers can use the HTTP protocol as a conventional web server, but can also include application server protocols such as FastCGI. You can configure a reverse proxy to direct clients to specific servers depending on the resource requested, or to follow certain rules regarding traffic load.


Reverse proxies can also take advantage of their placement in front of web servers by offering caching functionality. Large static assets can be configured with caching rules to avoid hitting web servers on every request, with some solutions offering an option to serve static assets directly without touching the web server at all. Furthermore, the reverse proxy can handle compression of these assets.


The popular Nginx web server is also a popular reverse proxy solution. While the Apache web server also has reverse proxy feature, it is an additional feature for Apache whereas Nginx was originally built for and focuses on reverse proxy functionality.


# Differentiating Forward Proxy and Reverse Proxy Use Cases


Because “forward” and “reverse” come with connotations of directionality and misleading comparisons with “incoming” and “outgoing” traffic, these labels can be confusing because both kinds of proxies handle requests and responses. Instead, a better way to differentiate between forward and reverse proxies is to examine the needs of the application you’re building.


A reverse proxy is useful when building a solution to serve web applications on the internet. They represent your web servers in any interactions with the internet.


A forward proxy is useful when placed in front of client traffic for your personal use or in a workplace environment. They represent your client traffic in any interactions with the internet.


Differentiating by use case instead of focusing on the similar naming conventions will help you avoid confusion.


# Conclusion


This article defined what a proxy is along with the two main types being the forward proxy and the reverse proxy. Practical use cases and an exploration of beneficial features was used to differentiate forward proxies and reverse proxies. If you’d like to explore implementation of proxies, you can check out our guide on how to configure Nginx as a web server and reverse proxy for Apache on one Ubuntu 20.04 Server.


