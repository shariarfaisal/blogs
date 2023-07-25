# How To Use Dig  Whois    Ping on an Ubuntu VPS to Query DNS Data

```Linux Basics``` ```Ubuntu``` ```DNS``` ```Linux Commands``` ```CentOS``` ```Fedora``` ```Debian```

Dig is a networking tool that can query DNS servers for information.  It can be very helpful for diagnosing problems with domain pointing and is a good way to verify that your configuration is working.


In this article, we will discuss how to use dig to verify your domain name settings and return data about how the internet sees your domain.  We will also examine a few other related tools like "whois" and "ping".


We will be using an Ubuntu 12.04 VPS to test the commands in this guide, but any modern Linux distribution should function in a similar way.


# How to Use Dig


The most basic way to use dig is to specify the domain we wish to query:


```
dig example.com
```


We can test "duckduckgo.com" to find out what kind of information it returns:


```
dig duckduckgo.com
```


```
; <<>> DiG 9.8.1-P1 <<>> duckduckgo.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64399
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;duckduckgo.com.		IN	A

;; ANSWER SECTION:
duckduckgo.com.	99	IN	A	107.21.1.61
duckduckgo.com.	99	IN	A	184.72.106.253
duckduckgo.com. 99	IN	A	184.72.106.52
duckduckgo.com.	99	IN	A	184.72.115.86

;; Query time: 33 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Fri Aug 23 14:26:17 2013
;; MSG SIZE  rcvd: 96
```


This is a lot of information.  Let's examine it in smaller chunks:


```
; <<>> DiG 9.8.1-P1 <<>> duckduckgo.com
;; global options: +cmd
```


The lines above act as a header for the query performed.  It is possible to run dig in batch mode, so proper labeling of the output is essential to allow for correct analysis.


```
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 64399
;; flags: qr rd ra; QUERY: 1, ANSWER: 4, AUTHORITY: 0, ADDITIONAL: 0
```


The next section gives us a technical summary of our query results.  We can see that the query was successful, certain flags were used, and that 4 "answers" were received.


```
;; QUESTION SECTION:
;duckduckgo.com.		IN	A

;; ANSWER SECTION:
duckduckgo.com.	99	IN	A	107.21.1.61
duckduckgo.com.	99	IN	A	184.72.106.253
duckduckgo.com.	99	IN	A	184.72.106.52
duckduckgo.com.	99	IN	A	184.72.115.86
```


The above section of the output contains the actual results we were looking for.  It restates the query and then returns the matching DNS records for that domain name.


Here, we can see that there are four "A" records for "duckduckgo.com".  By default, "A" records are returned.  This gives us the IP addresses that the domain name resolves to.


The "99" is the TTL (time to live) before the DNS server rechecks the association between the domain name and the IP address.  The "IN" means the class of the record is a standard internet class.


```
;; Query time: 33 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Fri Aug 23 14:26:17 2013
;; MSG SIZE  rcvd: 96
```


These lines simply provide some statistics about the actual query results.  The query time can be indicative of problems with the DNS servers.


# How to Use Dig to Test DNS Records


If you've set up a domain name with DigitalOcean, you can use dig to query the information.


To test that your "A" records are set correctly, type:


```
dig your_domain_name.com
```


If you would like to check if your mail servers are directing correctly, type:


```
dig your_domain_name.com MX
```


In general, you can issue the type of record that you would like to query after the domain name in the query.


If you would like to receive information about all of the records, type:


```
dig your_domain_name.com ANY
```


This will return any records that match the base domain you set up.  These will include "SOA" records, "NS" records, "A" records, and "MX" records.


Note: Due to the way that TTL and DNS works, it sometimes takes awhile for the changes you create to propagate to the name servers.  If you have created a record and do not see it, wait until the TTL reaches 0 to see if your record shows up.


If you would like to only return the actual IP the domain points to, you can type:


```
dig your_domain_name.com +short
```


## Using the "host" Command Instead of "dig"


An alternative to dig is a command called "host".  This command functions in a very similar way to dig, with many of the same options.


The basic syntax is:


```
host domain_name_or_IP_address
```


Notice how you do not need a flag to change the functionality from regular DNS lookup to reverse DNS lookup.


As with dig, you can specify the type of record you are interested in.  This is done with the "-t" flag.


To return the mx records of google, you can type:


```
host -t mx google.com
```


```
google.com mail is handled by 50 alt4.aspmx.l.google.com.
google.com mail is handled by 10 aspmx.l.google.com.
google.com mail is handled by 40 alt3.aspmx.l.google.com.
google.com mail is handled by 30 alt2.aspmx.l.google.com.
google.com mail is handled by 20 alt1.aspmx.l.google.com.
```


Other types of records can be retrieved just as easily.


You can return all records with the "-a" flag.  I will not post the output of this command, because it can be quite long:


```
host -a google.com
```


If you need additional information about the host, you can turn on verbose output with the "-v" flag:


```
host -v google.com
```


This will provide extended information.


# Using Other Tools to Query DNS Information


## Ping


A simple way to check if your domain name is resolving correctly is "ping".


It's usage is incredibly simple:


```
ping your_domain_name.com
```


```
PING your_domain_name.com (192.241.160.34) 56(84) bytes of data.
64 bytes from 192.241.160.34: icmp_req=1 ttl=64 time=0.026 ms
64 bytes from 192.241.160.34: icmp_req=2 ttl=64 time=0.038 ms
64 bytes from 192.241.160.34: icmp_req=3 ttl=64 time=0.037 ms
. . .
```


This will continue to output information until you type "CTRL-C".


You can also tell the software to ping only a certain number of times.  This will ping 3 times:


```
ping -c 3 your_domain_name.com
```


```
PING your_domain_name.com (192.241.160.34) 56(84) bytes of data.
64 bytes from 192.241.160.34: icmp_req=1 ttl=64 time=0.027 ms
64 bytes from 192.241.160.34: icmp_req=2 ttl=64 time=0.059 ms
64 bytes from 192.241.160.34: icmp_req=3 ttl=64 time=0.042 ms

--- your_domain_name.com ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.027/0.042/0.059/0.015 ms
```


This command can be used to simply check that the domain name resolves to the IP address that you assigned.


## Whois


The whois protocol returns information about registered domain names, including the name servers they are configured to work with.


While most of the information concerns the registration of the domain, it can be helpful to see that the name servers are returned correctly.


Run the command like this:


```
whois your_domain_name.com
```


This will return a long list of information.  The formatting will vary based on the whois server that contains the information.


Towards the bottom, you can usually see the domain servers that provide the domain forwarding to the correct IP addresses.


# Conclusion


While dig, ping, and whois are all simple tools that perform basic checks on your domain names, they can be incredibly useful.  When you are setting up your domain names, having a terminal open with dig handy can save you a lot of guesswork.


By Justin Ellingwood
