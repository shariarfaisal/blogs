# What is a CDN 

```CDN``` ```Glossary```

A content delivery network (CDN) is a geographically distributed group of servers optimized to deliver static content to end users. This static content can be almost any sort of data, but CDNs are most commonly used to deliver web pages and their related files, streaming video and audio, and large software packages.


A CDN consists of multiple points of presence (PoPs) in various locations, each consisting of several edge servers that  cache assets from an origin, or host server. The CDN routes user requests to the nearest edge server, which then serves users the appropriate content. If, however, the edge server does not have the assets cached or the cached assets have expired, the CDN will fetch and cache the latest version from either another nearby CDN edge server or your origin servers.


This allows geographically dispersed users to minimize the number of hops needed to receive static content, fetching the content directly from a nearby edge’s cache. The result is significantly decreased latencies and packet loss, faster page load times, and drastically reduced load on your origin infrastructure.


For more resources about CDNs, please visit:


- Using a CDN to Speed Up Static Content Delivery
- How to Speed Up WordPress Asset Delivery Using DigitalOcean Spaces CDN

A complete list of our caching-related tutorials, questions, and other educational resources can be found on our CDN tag page.


