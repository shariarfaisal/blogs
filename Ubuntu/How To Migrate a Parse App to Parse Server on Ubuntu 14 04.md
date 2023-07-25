# How To Migrate a Parse App to Parse Server on Ubuntu 14 04

```Ubuntu``` ```Nginx``` ```Let's Encrypt``` ```MongoDB```

## Introduction


Parse is a Mobile Backend as a Service platform, owned by Facebook since 2013.  In January of 2016, Parse announced that its hosted services would shut down completely on January 28, 2017.


Fortunately, Parse has also released an open source API server, compatible with the hosted service’s API, called Parse Server. Parse Server is under active development, and seems likely to attract a large developer community.  It can be be deployed to a range of environments running Node.js and MongoDB.


This guide focuses on migrating a pre-existing Parse application to a standalone instance of Parse Server running on Ubuntu 14.04.  It uses TLS/SSL encryption for all connections, using a certificate provided by Let’s Encrypt, a new Certificate Authority which offers free certificates.  It includes a few details specific to DigitalOcean and Ubuntu 14.04, but should be broadly applicable to systems running recent Debian-derived GNU/Linux distributions.



Warning: It is strongly recommended that this procedure first be tested with a development or test version of the app before attempting it with a user-facing production app.  It is also strongly recommended that you read this guide in conjunction with the official migration documentation.

# Prerequisites


This guide builds on How To Run Parse Server on Ubuntu 14.04.  It requires the following:


- An Ubuntu 14.04 server, configured with a non-root sudo user
- Node.js 5.6.x
- MongoDB 3.0.x
- A domain name pointing at the server
- A Parse App to be migrated
- Nginx installed and configured with SSL using Let’s Encrypt certificates. How To Secure Nginx with Let’s Encrypt on Ubuntu 14.04 will walk you through the process.

The target server should have enough storage to handle all of your app’s data.  Since Parse compresses data on their end, they officially recommend that you provision at least 10 times as much storage space as used by your hosted app.


# Step 1 – Configure MongoDB for Migration


Parse provides a migration tool for existing applications.  In order to make use of it, we need to open MongoDB to external connections and secure it with a copy of the TLS/SSL certificate from Let’s Encrypt.  Start by combining fullchain1.pem and privkey1.pem into a new file in /etc/ssl:


```
sudo cat /etc/letsencrypt/archive/domain_name/{fullchain1.pem,privkey1.pem} | sudo tee /etc/ssl/mongo.pem


```



You will have to repeat the above command after renewing your Let’s Encrypt certificate.  If you configure auto-renewal of the Let’s Encrypt certificate, remember to include this operation.

Make sure mongo.pem is owned by the mongodb user, and readable only by its owner:


```
sudo chown mongodb:mongodb /etc/ssl/mongo.pem
sudo chmod 600 /etc/ssl/mongo.pem


```


Now, open /etc/mongod.conf in nano (or your text editor of choice):


```
sudo nano /etc/mongod.conf


```


Here, we’ll make several important changes.


First, look for the bindIp line in the net: section, and tell MongoDB to listen on all addresses by changing 127.0.0.1 to 0.0.0.0.  Below this, add SSL configuration to the same section:


/etc/mongod.conf
```
# network interfaces
net:
  port: 27017
  bindIp: 0.0.0.0
  ssl:
    mode: requireSSL
    PEMKeyFile: /etc/ssl/mongo.pem

```


Next, under # security, enable client authorization:


/etc/mongod.conf
```
# security
security:
  authorization: enabled

```


Finally, the migration tool requires us to set the failIndexKeyTooLong parameter to false:


/etc/mongod.conf
```

setParameter:
  failIndexKeyTooLong: false

```



Note: Whitespace is significant in MongoDB configuration files, which are based on YAML.  When copying configuration values, make sure that you preserve indentation.

Exit and save the file.


Before restarting the mongod service, we need to add a user with the admin role.  Connect to the running MongoDB instance:


```
mongo --port 27017


```


Create an admin user and exit.  Be sure to replace sammy with your desired username and password with a strong password.


```
use admin
db.createUser({
  user: "sammy",
  pwd: "password",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
exit

```


Restart the mongod service:


```
sudo service mongod restart


```


# Step 2 – Migrate Application Data from Parse


Now that you have a remotely-accessible MongoDB instance, you can use the Parse migration tool to transfer your app’s data to your server.


## Configure MongoDB Credentials for Migration Tool


We’ll begin by connecting locally with our new admin user:


```
mongo --port 27017 --ssl --sslAllowInvalidCertificates --authenticationDatabase admin --username sammy --password


```


You should be prompted to enter the password you set earlier.


Once connected, choose a name for the database to store your app’s data.  For example, if you’re migrating an app called Todo, you might use todo.  You’ll also need to pick another strong password for a user called parse.


From the mongo shell, give this user access to database_name:


```
use database_name
db.createUser({ user: "parse", pwd: "password", roles: [ "readWrite", "dbAdmin" ] })


```


## Initiate Data Migration Process


In a browser window, log in to Parse, and open the settings for your app.  Under General, locate the Migrate button and click it:





You will be prompted for a MongoDB connection string.  Use the following format:


```
mongodb://parse:password@your_domain_name:27017/database_name?ssl=true

```


For example, if you are using the domain example.com, with the user parse, the password foo, and a database called todo, your connection string would look like this:


```
mongodb://parse:foo@example.com:27017/todo?ssl=true

```


Don’t forget ?ssl=true at the end, or the connection will fail.  Enter the connection string into the dialog like so:





Click Begin the migration.  You should see progress dialogs for copying a snapshot of your Parse hosted database to your server, and then for syncing new data since the snapshot was taken.  The duration of this process will depend on the amount of data to be transferred, and may be substantial.








## Verify Data Migration


Once finished, the migration process will enter a verification step.  Don’t finalize the migration yet.  You’ll first want to make sure the data has actually transferred, and test a local instance of Parse Server.





Return to your mongo shell, and examine your local database.  Begin by accessing database_name and examining the collections it contains:


```
use database_name


```


```
show collections


```


```
Sample Output for Todo AppTodo
_Index
_SCHEMA
_Session
_User
_dummy
system.indexes

```


You can examine the contents of a specific collection with the .find() method:


```
db.ApplicationName.find()


```


```
Sample Output for Todo App> db.Todo.find()
{ "_id" : "hhbrhmBrs0", "order" : NumberLong(1), "_p_user" : "_User$dceklyR50A", "done" : false, "_acl" : { "dceklyR50A" : { "r" : true, "w" : true } }, "_rperm" : [ "dceklyR50A" ], "content" : "Migrate this app to my own server.", "_updated_at" : ISODate("2016-02-08T20:44:26.157Z"), "_wperm" : [ "dceklyR50A" ], "_created_at" : ISODate("2016-02-08T20:44:26.157Z") }

```


Your specific output will be different, but you should see data for your app.  Once satisfied, exit mongo and return to the shell:


```
exit


```


# Step 3 – Install and Configure Parse Server and PM2


With your app data in MongoDB, we can move on to installing Parse Server itself, and integrating with the rest of the system.  We’ll give Parse Server a dedicated user, and use a utility called PM2 to configure it and ensure that it’s always running.


## Install Parse Server and PM2 Globally


Use npm to install the parse-server utility, the pm2 process manager, and their dependencies, globally:


```
sudo npm install -g parse-server pm2


```


## Create a Dedicated Parse User and Home Directory


Instead of running parse-server as root or your sudo user, we’ll create a system user called parse:


```
sudo useradd --create-home --system parse


```


Now set a password for parse:


```
sudo passwd parse


```


You’ll be prompted to enter a password twice.


Now, use the su command to become the parse user:


```
sudo su parse


```


Change to parse’s home directory:


```
cd ~


```


## Write or Migrate a Cloud Code File


Create a cloud code directory:


```
mkdir -p ~/cloud


```


Edit /home/parse/cloud/main.js:


```
nano ~/cloud/main.js


```


For testing purposes, you can paste the following:


/home/parse/cloud/main.js
```
Parse.Cloud.define('hello', function(req, res) {
  res.success('Hi');
});

```


Alternatively, you can migrate any cloud code defined for your application by copying it from the Cloud Code section of your app’s settings on the Parse Dashboard.


Exit and save.


## Retrieve Keys and Write /home/parse/ecosystem.json


PM2 is a feature-rich process manager, popular with Node.js developers.  We’ll use the pm2 utility to configure our parse-server instance and keep it running over the long term.


You’ll need to retrieve some of the keys for your app.  In the Parse dashboard, click on App Settings followed by Security & Keys:





Of these, only the Application ID and Master Key are required.  Others (client, JavaScript, .NET, and REST API keys) may be necessary to support older client builds, but, if set, will be required in all requests.  Unless you have reason to believe otherwise, you should begin by using just the Application ID and Master Key.


With these keys ready to hand, edit a new file called /home/parse/ecosystem.json:


```
nano ecosystem.json


```


Paste the following, changing configuration values to reflect your MongoDB connection string, Application ID, and Master Key:


```
{
  "apps" : [{
    "name"        : "parse-wrapper",
    "script"      : "/usr/bin/parse-server",
    "watch"       : true,
    "merge_logs"  : true,
    "cwd"         : "/home/parse",
    "env": {
      "PARSE_SERVER_CLOUD_CODE_MAIN": "/home/parse/cloud/main.js",
      "PARSE_SERVER_DATABASE_URI": "mongodb://parse:password@your_domain_name:27017/database_name?ssl=true",
      "PARSE_SERVER_APPLICATION_ID": "your_application_id",
      "PARSE_SERVER_MASTER_KEY": "your_master_key",
    }
  }]
}

```


The env object is used to set environment variables.  If you need to configure additional keys, parse-server also recognizes the following variables:


- PARSE_SERVER_COLLECTION_PREFIX
- PARSE_SERVER_CLIENT_KEY
- PARSE_SERVER_REST_API_KEY
- PARSE_SERVER_DOTNET_KEY
- PARSE_SERVER_JAVASCRIPT_KEY
- PARSE_SERVER_DOTNET_KEY
- PARSE_SERVER_FILE_KEY
- PARSE_SERVER_FACEBOOK_APP_IDS

Exit and save ecosystem.json.


Now, run the script with pm2:


```
pm2 start ecosystem.json


```


```
Sample Output...
[PM2] Spawning PM2 daemon
[PM2] PM2 Successfully daemonized
[PM2] Process launched
┌───────────────┬────┬──────┬──────┬────────┬─────────┬────────┬─────────────┬──────────┐
│ App name      │ id │ mode │ pid  │ status │ restart │ uptime │ memory      │ watching │
├───────────────┼────┼──────┼──────┼────────┼─────────┼────────┼─────────────┼──────────┤
│ parse-wrapper │ 0  │ fork │ 3499 │ online │ 0       │ 0s     │ 13.680 MB   │  enabled │
└───────────────┴────┴──────┴──────┴────────┴─────────┴────────┴─────────────┴──────────┘
 Use `pm2 show <id|name>` to get more details about an app

```


Now tell pm2 to save this process list:


```
pm2 save


```


```
Sample Output[PM2] Dumping processes

```


The list of processes pm2 is running for the parse user should now be stored in /home/parse/.pm2.


Now we need to make sure the parse-wrapper process we defined earlier in ecosystem.json is restored each time the server is restarted.  Fortunately, pm2 can generate and install a script on its own.


Exit to your regular sudo user:


```
exit


```


Tell pm2 to install initialization scripts for Ubuntu, to be run as the parse user, using /home/parse as its home directory:


```
sudo pm2 startup ubuntu -u parse --hp /home/parse/


```


Output
```
[PM2] Spawning PM2 daemon
[PM2] PM2 Successfully daemonized
[PM2] Generating system init script in /etc/init.d/pm2-init.sh
[PM2] Making script booting at startup...
[PM2] -ubuntu- Using the command:
      su -c "chmod +x /etc/init.d/pm2-init.sh && update-rc.d pm2-init.sh defaults"
 System start/stop links for /etc/init.d/pm2-init.sh already exist.
[PM2] Done.

```


# Step 4 – Install and Configure Nginx


We’ll use the Nginx web server to provide a reverse proxy to parse-server, so that we can serve the Parse API securely over TLS/SSL.


In the prerequisites, you set up the default server to respond to your domain name, with SSL provided by Let’s Encrypt certificates. We’ll update this configuration file with our proxy information.


Open /etc/nginx/sites-enabled/default in nano (or your editor of choice):


```
sudo nano /etc/nginx/sites-enabled/default


```


Within the main server block (it should already contain a location / block) add another location block to handle the proxying of /parse/ URLs:


/etc/nginx/sites-enabled/default
```
. . .
        # Pass requests for /parse/ to Parse Server instance at localhost:1337
        location /parse/ {
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-NginX-Proxy true;
                proxy_pass http://localhost:1337/;
                proxy_ssl_session_reuse off;
                proxy_set_header Host $http_host;
                proxy_redirect off;
        }

```


Exit the editor and save the file.  Restart Nginx so that changes take effect:


```
sudo service nginx restart


```


```
Output * Restarting nginx nginx
   ...done.

```


# Step 5 – Test Parse Server


At this stage, you should have the following:


- A TLS/SSL certificate, provided by Let’s Encrypt
- MongoDB, secured with the Let’s Encrypt certificate
- parse-server running under the parse user on port 1337, configured with the keys expected by your app
- pm2 managing the parse-server process under the parse user, and a startup script to restart pm2 on boot
- nginx, secured with the Let’s Encrypt certificate, and configured to proxy connections to https://your_domain_name/parse to the parse-server instance

It should now be possible to test reads, writes, and cloud code execution using curl.



Note: The curl commands in this section should be harmless when used with a test or development app.  Be cautious when writing data to a production app.

## Writing Data with a POST


You’ll need to give curl several important options:





Option
Description




-X POST
Sets the request type, which would otherwise default to GET


-H "X-Parse-Application-Id: your_application_id"
Sends a header which identifies your application to parse-server


-H "Content-Type: application/json"
Sends a header which lets parse-server know to expect JSON-formatted data


-d '{json_data}
Sends the data itself




Putting these all together, we get:


```
curl -X POST \
  -H "X-Parse-Application-Id: your_application_id" \
  -H "Content-Type: application/json" \
  -d '{"score":1337,"playerName":"Sammy","cheatMode":false}' \
  https://your_domain_name/parse/classes/GameScore

```


Sample Output
```
{"objectId":"YpxFdzox3u","createdAt":"2016-02-18T18:03:43.188Z"}

```


## Reading Data with a GET


Since curl sends GET requests by default, and we’re not supplying any data, you should only need to send the Application ID in order to read some sample data back:


```
curl -H "X-Parse-Application-Id: your_application_id" https://your_domain_name/parse/classes/GameScore


```


Sample Output
```
{"results":[{"objectId":"BNGLzgF6KB","score":1337,"playerName":"Sammy","cheatMode":false,"updatedAt":"2016-02-17T20:53:59.947Z","createdAt":"2016-02-17T20:53:59.947Z"},{"objectId":"0l1yE3ivB6","score":1337,"playerName":"Sean Plott","cheatMode":false,"updatedAt":"2016-02-18T03:57:00.932Z","createdAt":"2016-02-18T03:57:00.932Z"},{"objectId":"aKgvFqDkXh","score":1337,"playerName":"Sean Plott","cheatMode":false,"updatedAt":"2016-02-18T04:44:01.275Z","createdAt":"2016-02-18T04:44:01.275Z"},{"objectId":"zCKTgKzCRH","score":1337,"playerName":"Sean Plott","cheatMode":false,"updatedAt":"2016-02-18T16:56:51.245Z","createdAt":"2016-02-18T16:56:51.245Z"},{"objectId":"YpxFdzox3u","score":1337,"playerName":"Sean Plott","cheatMode":false,"updatedAt":"2016-02-18T18:03:43.188Z","createdAt":"2016-02-18T18:03:43.188Z"}]}

```


## Executing Example Cloud Code


A simple POST with no real data to https://your_domain_name/parse/functions/hello will run the hello() function defined in /home/parse/cloud/main.js:


```
curl -X POST \
  -H "X-Parse-Application-Id: your_application_id" \
  -H "Content-Type: application/json" \
  -d '{}' \
  https://your_domain_name/parse/functions/hello

```


Sample Output
```
{"result":"Hi"}

```


If you have instead migrated your own custom cloud code, you can test with a known function from main.js.


# Step 6 – Configure Your App for Parse Server and Finalize Migration


Your next step will be to change your client application itself to use the Parse Server API endpoint.  Consult the official documentation on using Parse SDKs with Parse Server.  You will need the latest version of the SDK for your platform.  As with the curl-based tests above, use this string for the server URL:


```
https://your_domain_name/parse

```


Return to the Parse dashboard in your browser and the Migration tab:





Click the Finalize button:





Your app should now be migrated.


# Conclusion and Next Steps


This guide offers a functional starting point for migrating a Parse-hosted app to a Parse Server install on a single Ubuntu system, such as a DigitalOcean droplet.  The configuration we’ve described should be adequate for a low-traffic app with a modest userbase.  Hosting for a larger app may require multiple systems to provide redundant data storage and load balancing between API endpoints.  Even small projects are likely to involve infrastructure considerations that we haven’t directly addressed.


In addition to reading the official Parse Server documentation and tracking the GitHub issues for the project when troubleshooting, you may wish to explore the following topics:


- The DigitalOcean New Ubuntu 14.04 Server Checklist
- Securing Nginx with Let’s Encrypt on Ubuntu 14.04
- Installing Node.js on Ubuntu 14.04
- DigitalOcean Droplet Backups

