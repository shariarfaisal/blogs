# How To Install Elasticsearch 1 7  Logstash 1 5  and Kibana 4 1  ELK Stack  on CentOS 7

```Logging``` ```Nginx``` ```Monitoring``` ```CentOS```


Note: This tutorial is for an older version of the ELK stack, which is not compatible with the latest version. The latest version of this tutorial is available at How To Install Elasticsearch, Logstash, and Kibana (ELK Stack) on CentOS 7.

## Introduction


In this tutorial, we will go over the installation of the Elasticsearch ELK Stack on CentOS 7—that is, Elasticsearch 1.7.3, Logstash 1.5.4, and Kibana 4.1.1. We will also show you how to configure it to gather and visualize the syslogs of your systems in a centralized location. Logstash is an open source tool for collecting, parsing, and storing logs for future use. Kibana is a web interface that can be used to search and view the logs that Logstash has indexed. Both of these tools are based on Elasticsearch.


Centralized logging can be very useful when attempting to identify problems with your servers or applications, as it allows you to search through all of your logs in a single place. It is also useful because it allows you to identify issues that span multiple servers by correlating their logs during a specific time frame.


It is possible to use Logstash to gather logs of all types, but we will limit the scope of this tutorial to syslog gathering.


# Our Goal


The goal of the tutorial is to set up Logstash to gather syslogs of multiple servers, and set up Kibana to visualize the gathered logs.


Our Logstash / Kibana setup has four main components:


- Logstash: The server component of Logstash that processes incoming logs
- Elasticsearch: Stores all of the logs
- Kibana: Web interface for searching and visualizing logs, which will be proxied through Nginx
- Logstash Forwarder: Installed on servers that will send their logs to Logstash, Logstash Forwarder serves as a log forwarding agent that utilizes the lumberjack networking protocol to communicate with Logstash




We will install the first three components on a single server, which we will refer to as our Logstash Server. The Logstash Forwarder will be installed on all of the client servers that we want to gather logs for, which we will refer to collectively as our Client Servers.


# Prerequisites


To complete this tutorial, you will require root access to an CentOS 7 VPS. Instructions to set that up can be found here (steps 3 and 4): Initial Server Setup with CentOS 7.


If you would prefer to use Ubuntu instead, check out this tutorial: How To Install ELK on Ubuntu 14.04.


The amount of CPU, RAM, and storage that your Logstash Server will require depends on the volume of logs that you intend to gather. For this tutorial, we will be using a VPS with the following specs for our Logstash Server:


- OS: CentOS 7
- RAM: 4GB
- CPU: 2

In addition to your Logstash Server, you will want to have a few other servers that you will gather logs from.


Let’s get started on setting up our Logstash Server!


# Install Java 8


Elasticsearch and Logstash require Java, so we will install that now. We will install a recent version of Oracle Java 8 because that is what Elasticsearch recommends. It should, however, work fine with OpenJDK, if you decide to go that route. Following the steps in this section means that you accept the Oracle Binary License Agreement for Java SE.


Change to your home directory and download the Oracle Java 8 (Update 65) JDK RPM with these commands:


```
cd ~
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u65-b17/jdk-8u65-linux-x64.rpm"


```


Then install the RPM with this yum command (if you downloaded a different release, substitute the filename here):


```
sudo yum localinstall jdk-8u65-linux-x64.rpm


```


Now Java should be installed at /usr/java/jdk1.8.0_65/jre/bin/java, and linked from /usr/bin/java.


You may delete the archive file that you downloaded earlier:


```
rm ~/jdk-8u65-linux-x64.rpm


```


Now that Java 8 is installed, let’s install ElasticSearch.


# Install Elasticsearch


Elasticsearch can be installed with a package manager by adding Elastic’s package repository.


Run the following command to import the Elasticsearch public GPG key into rpm:


```
sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch


```


Create and edit a new yum repository file for Elasticsearch:


```
sudo vi /etc/yum.repos.d/elasticsearch.repo


```


Add the following repository configuration:


/etc/yum.repos.d/elasticsearch.repo
```
[elasticsearch-1.7]
name=Elasticsearch repository for 1.7.x packages
baseurl=http://packages.elastic.co/elasticsearch/1.7/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1


```


Save and exit.


Install Elasticsearch with this command:


```
sudo yum -y install elasticsearch


```


Elasticsearch is now installed.  Let’s edit the configuration:


```
sudo vi /etc/elasticsearch/elasticsearch.yml


```


You will want to restrict outside access to your Elasticsearch instance (port 9200), so outsiders can’t read your data or shutdown your Elasticsearch cluster through the HTTP API. Find the line that specifies network.host, uncomment it, and replace its value with “localhost” so it looks like this:


elasticsearch.yml excerpt (updated)
```
network.host: localhost

```


Save and exit elasticsearch.yml.


Now start Elasticsearch:


```
sudo systemctl start elasticsearch


```


Then run the following command to start Elasticsearch automatically on boot up:


```
sudo systemctl enable elasticsearch


```


Now that Elasticsearch is up and running, let’s install Kibana.


# Install Kibana


The Kibana package shares the same GPG Key as Elasticsearch, and we already installed that public key, so let’s create and edit a new Yum repository file for Kibana:


Create and edit a new yum repository file for Kibana:


```
sudo vi /etc/yum.repos.d/kibana.repo


```


Add the following repository configuration:


/etc/yum.repos.d/kibana.repo
```
[kibana-4.1]
name=Kibana repository for 4.1.x packages
baseurl=http://packages.elastic.co/kibana/4.1/centos
gpgcheck=1
gpgkey=http://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1


```


Save and exit.


Install Kibana with this command:


```
sudo yum -y install kibana


```


Open the Kibana configuration file for editing:


```
sudo vi /opt/kibana/config/kibana.yml


```


In the Kibana configuration file, find the line that specifies host, and replace the IP address (“0.0.0.0” by default) with “localhost”:


kibana.yml excerpt (updated)
```
host: "localhost"

```


Save and exit. This setting makes it so Kibana will only be accessible to the localhost. This is fine because we will install an Nginx reverse proxy, on the same server, to allow external access.


Now start the Kibana service, and enable it:


```
sudo systemctl start kibana
sudo chkconfig kibana on


```


Before we can use the Kibana web interface, we have to set up a reverse proxy. Let’s do that now, with Nginx.


# Install Nginx


Because we configured Kibana to listen on localhost, we must set up a reverse proxy to allow external access to it. We will use Nginx for this purpose.


Note: If you already have an Nginx instance that you want to use, feel free to use that instead. Just make sure to configure Kibana so it is reachable by your Nginx server (you probably want to change the host value, in /opt/kibana/config/kibana.yml, to your Kibana server’s private IP address). Also, it is recommended that you enable SSL/TLS.


Add the EPEL repository to yum:


```
sudo yum -y install epel-release


```


Now use yum to install Nginx and httpd-tools:


```
sudo yum -y install nginx httpd-tools


```


Use htpasswd to create an admin user, called “kibanaadmin” (you should use another name), that can access the Kibana web interface:


```
sudo htpasswd -c /etc/nginx/htpasswd.users kibanaadmin


```


Enter a password at the prompt. Remember this login, as you will need it to access the Kibana web interface.


Now open the Nginx configuration file in your favorite editor. We will use vi:


```
sudo vi /etc/nginx/nginx.conf


```


Find the default server block (starts with server {), the last configuration block in the file, and delete it. When you are done, the last two lines in the file should look like this:


nginx.conf excerpt
```
    include /etc/nginx/conf.d/*.conf;
}

```


Save and exit.


Now we will create an Nginx server block in a new file:


```
sudo vi /etc/nginx/conf.d/kibana.conf


```


Paste the following code block into the file. Be sure to update the server_name to match your server’s name:


/etc/nginx/conf.d/kibana.conf
```
server {
    listen 80;

    server_name example.com;
            
    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;        
    }
}


```


Save and exit. This configures Nginx to direct your server’s HTTP traffic to the Kibana application, which is listening on localhost:5601. Also, Nginx will use the htpasswd.users file, that we created earlier, and require basic authentication.


Now start and enable Nginx to put our changes into effect:


```
sudo systemctl start nginx
sudo systemctl enable nginx


```



Note: This tutorial assumes that SELinux is disabled. If this is not the case, you may need to run the following command for Kibana to work properly: sudo setsebool -P httpd_can_network_connect 1

Kibana is now accessible via your FQDN or the public IP address of your Logstash Server i.e. http://logstash_server_public_ip/. If you go there in a web browser, after entering the “kibanaadmin” credentials, you should see a Kibana welcome page which will ask you to configure an index pattern. Let’s get back to that later, after we install all of the other components.


# Install Logstash


The Logstash package shares the same GPG Key as Elasticsearch, and we already installed that public key, so let’s create and edit a new Yum repository file for Logstash:


```
sudo vi /etc/yum.repos.d/logstash.repo


```


Add the following repository configuration:


/etc/yum.repos.d/logstash.repo
```
[logstash-1.5]
name=logstash repository for 1.5.x packages
baseurl=http://packages.elasticsearch.org/logstash/1.5/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1


```


Save and exit.


Install Logstash 1.5 with this command:


```
sudo yum -y install logstash


```


Logstash is installed but it is not configured yet.


# Generate SSL Certificates


Since we are going to use Logstash Forwarder to ship logs from our Servers to our Logstash Server, we need to create an SSL certificate and key pair. The certificate is used by the Logstash Forwarder to verify the identity of Logstash Server.


Now you have two options for generating your SSL certificates. If you have a DNS setup that will allow your client servers to resolve the IP address of the Logstash Server,  use Option 2. Otherwise, Option 1 will allow you to use IP addresses.


## Option 1: IP Address


If you don’t have a DNS setup—that would allow your servers, that you will gather logs from, to resolve the IP address of your Logstash Server—you will have to add your Logstash Server’s private IP address to the subjectAltName (SAN) field of the SSL certificate that we are about to generate. To do so, open the OpenSSL configuration file:


```
sudo vi /etc/pki/tls/openssl.cnf


```


Find the [ v3_ca ] section in the file, and add this line under it (substituting in the Logstash Server’s private IP address):


openssl.cnf excerpt
```
subjectAltName = IP: logstash_server_private_ip


```


Save and exit.


Now generate the SSL certificate and private key in the appropriate locations (/etc/pki/tls/), with the following commands:


```
cd /etc/pki/tls
sudo openssl req -config /etc/pki/tls/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt


```


The logstash-forwarder.crt file will be copied to all of the servers that will send logs to Logstash but we will do that a little later. Let’s complete our Logstash configuration. If you went with this option, skip option 2 and move on to Configure Logstash.


## Option 2: FQDN (DNS)


If you have a DNS setup with your private networking, you should create an A record that contains the Logstash Server’s private IP address—this domain name will be used in the next command, to generate the SSL certificate. Alternatively, you can use a record that points to the server’s public IP address. Just be sure that your servers (the ones that you will be gathering logs from) will be able to resolve the domain name to your Logstash Server.


Now generate the SSL certificate and private key, in the appropriate locations (/etc/pki/tls/…), with the following command (substitute in the FQDN of the Logstash Server):


```
cd /etc/pki/tls
sudo openssl req -subj '/CN=logstash_server_fqdn/' -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-forwarder.key -out certs/logstash-forwarder.crt


```


The logstash-forwarder.crt file will be copied to all of the servers that will send logs to Logstash but we will do that a little later. Let’s complete our Logstash configuration.


# Configure Logstash


Logstash configuration files are in the JSON-format, and reside in /etc/logstash/conf.d. The configuration consists of three sections: inputs, filters, and outputs.


Let’s create a configuration file called 01-lumberjack-input.conf and set up our “lumberjack” input (the protocol that Logstash Forwarder uses):


```
sudo vi /etc/logstash/conf.d/01-lumberjack-input.conf


```


Insert the following input configuration:


/etc/logstash/conf.d/01-lumberjack-input.conf
```
input {
  lumberjack {
    port => 5043
    type => "logs"
    ssl_certificate => "/etc/pki/tls/certs/logstash-forwarder.crt"
    ssl_key => "/etc/pki/tls/private/logstash-forwarder.key"
  }
}


```


Save and quit. This specifies a lumberjack input that will listen on tcp port 5043, and it will use the SSL certificate and private key that we created earlier.


Now let’s create a configuration file called 10-syslog.conf, where we will add a filter for syslog messages:


```
sudo vi /etc/logstash/conf.d/10-syslog.conf


```


Insert the following syslog filter configuration:


/etc/logstash/conf.d/10-syslog.conf
```
filter {
  if [type] == "syslog" {
    grok {
      match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      add_field => [ "received_at", "%{@timestamp}" ]
      add_field => [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match => [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }
}


```


Save and quit. This filter looks for logs that are labeled as “syslog” type (by a Logstash Forwarder), and it will try to use “grok” to parse incoming syslog logs to make it structured and query-able.


Lastly, we will create a configuration file called 30-lumberjack-output.conf:


```
sudo vi /etc/logstash/conf.d/30-lumberjack-output.conf


```


Insert the following output configuration:


/etc/logstash/conf.d/30-lumberjack-output.conf
```
output {
  elasticsearch { host => localhost }
  stdout { codec => rubydebug }
}


```


Save and exit. This output basically configures Logstash to store the logs in Elasticsearch.


With this configuration, Logstash will also accept logs that do not match the filter, but the data will not be structured (e.g. unfiltered Nginx or Apache logs would appear as flat messages instead of categorizing messages by HTTP response codes, source IP addresses, served files, etc.).


If you want to add filters for other applications that use the Logstash Forwarder input, be sure to name the files so they sort between the input and the output configuration (i.e. between 01- and 30-).


Restart and enable Logstash to put our configuration changes into effect:


```
sudo systemctl restart logstash


```


Now that our Logstash Server is ready, let’s move onto setting up Logstash Forwarder.


# Set Up Logstash Forwarder (Add Client Servers)


Do these steps for each CentOS or RHEL 7 server that you want to send logs to your Logstash Server. For instructions on installing Logstash Forwarder on Debian-based Linux distributions (e.g. Ubuntu, Debian, etc.), refer to the Build and Package Logstash Forwarder section of the Ubuntu variation of this tutorial.


## Copy SSL Certificate and Logstash Forwarder Package


On Logstash Server, copy the SSL certificate to Client Server (substitute the client server’s IP address, and your own login):


```
scp /etc/pki/tls/certs/logstash-forwarder.crt user@server_private_IP:/tmp


```


After providing the login credentials, ensure that the certificate copy was successful. It is required for communication between the client servers and the Logstash server.


## Install Logstash Forwarder Package


On Client Server, create run the following command to import the Elasticsearch public GPG key into rpm:


```
sudo rpm --import http://packages.elastic.co/GPG-KEY-elasticsearch


```


Create and edit a new yum repository file for Logstash Forwarder:


```
sudo vi /etc/yum.repos.d/logstash-forwarder.repo


```


Add the following repository configuration:


/etc/yum.repos.d/logstash-forwarder.repo
```
[logstash-forwarder]
name=logstash-forwarder repository
baseurl=http://packages.elastic.co/logstashforwarder/centos
gpgcheck=1
gpgkey=http://packages.elasticsearch.org/GPG-KEY-elasticsearch
enabled=1


```


Save and exit.


Then install the Logstash Forwarder package:


```
sudo yum -y install logstash-forwarder


```


Now copy the Logstash server’s SSL certificate into the appropriate location (/etc/pki/tls/certs):


```
sudo cp /tmp/logstash-forwarder.crt /etc/pki/tls/certs/


```


## Configure Logstash Forwarder


On Client Server, create and edit Logstash Forwarder configuration file, which is in JSON format:


```
sudo vi /etc/logstash-forwarder.conf


```


Under the network section, add the following lines into the file, substituting in your Logstash Server’s private IP address for logstash_server_private_IP:


logstash-forwarder.conf excerpt 1 of 2
```
    "servers": [ "logstash_server_private_IP:5043" ],
    "ssl ca": "/etc/pki/tls/certs/logstash-forwarder.crt",
    "timeout": 15

```


Under the files section (between the square brackets), add the following lines,


logstash-forwarder.conf excerpt 2 of 2
```
    {
      "paths": [
        "/var/log/messages",
        "/var/log/secure"
       ],
      "fields": { "type": "syslog" }
    }

```


Save and quit. This configures Logstash Forwarder to connect to your Logstash Server on port 5043 (the port that we specified an input for earlier), and uses the SSL certificate that we created earlier. The paths section specifies which log files to send (here we specify messages and secure), and the type section specifies that these logs are of type "syslog* (which is the type that our filter is looking for).


Note that this is where you would add more files/types to configure Logstash Forwarder to other log files to Logstash on port 5043.


Now start and enable Logstash Forwarder to put our changes into place:


```
sudo systemctl start logstash-forwarder
sudo chkconfig logstash-forwarder on


```


Now Logstash Forwarder is sending your syslog messages and secure files to your Logstash Server! Repeat this section for all of the other servers that you wish to gather logs for.


# Connect to Kibana


When you are finished setting up Logstash Forwarder on all of the servers that you want to gather logs for, let’s look at Kibana, the web interface that we installed earlier.


In a web browser, go to the FQDN or public IP address of your Logstash Server. After entering the “kibanaadmin” credentials, you should see a page prompting you to configure an index pattern:





Go ahead and select @timestamp from the dropdown menu, then click the Create button to create the first index.


Now click the Discover link in the top navigation bar. By default, this will show you all of the log data over the last 15 minutes. You should see a histogram with log events, with log messages below:





Right now, there won’t be much in there because you are only gathering syslogs from your client servers. Here, you can search and browse through your logs. You can also customize your dashboard.


Try the following things:


- Search for “root” to see if anyone is trying to log into your servers as root
- Search for a particular hostname (search for host: "hostname")
- Change the time frame by selecting an area on the histogram or from the menu above
- Click on messages below the histogram to see how the data is being filtered

Kibana has many other features, such as graphing and filtering, so feel free to poke around!


# Conclusion


Now that your syslogs are centralized via Elasticsearch and Logstash, and you are able to visualize them with Kibana, you should be off to a good start with centralizing all of your important logs. Remember that you can send pretty much any type of log to Logstash, but the data becomes even more useful if it is parsed and structured with grok.


To improve your new ELK stack, you should look into gathering and filtering your other logs with Logstash, and creating Kibana dashboards. These topics are covered in the second and third tutorials in this series. Also, if you are having trouble with your setup, follow our How To Troubleshoot Common ELK Stack Issues tutorial.


Scroll down for links to learn more about using your ELK stack!


