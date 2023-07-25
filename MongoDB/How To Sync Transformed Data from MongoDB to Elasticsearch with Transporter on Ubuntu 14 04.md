# How To Sync Transformed Data from MongoDB to Elasticsearch with Transporter on Ubuntu 14 04

```Ubuntu``` ```MongoDB``` ```Elasticsearch``` ```Go```


Status: Deprecated
This tutorial is written for an outdated version of Transporter. You can read the Ubuntu 16.04 version of this tutorial instead, which uses a more recent version of Transporter.

## Introduction


Elasticsearch facilitates full text search of your data, while MongoDB excels at storing it. Using MongoDB to store your data and Elasticsearch for search is a common architecture.


Many times, you might find the need to migrate data from MongoDB to Elasticsearch in bulk. Writing your own program for this, although a good exercise, can be a tedious task. There is a wonderful open source utility called Transporter, developed by Compose (a cloud platform for databases), that takes care of this task very efficiently.


This tutorial shows you how to use the open-source utility Transporter to quickly copy data from MongoDB to Elasticsearch with custom transformations.


# Goals


In this article, we are going to cover how to copy data from MongoDB to Elasticsearch on Ubuntu 14.04, using the Transporter utility.


We’ll start with a quick overview showing you how to install MongoDB and Elasticsearch, although we won’t go into detail about data modeling in the two systems. Feel free to skim through the installation steps quickly if you have already installed both of them.


Then we’ll move on to Transporter.


The instructions are similar for other versions of Ubuntu, as well as other Linux distributions.


# Prerequisites


Please complete the following prerequisites.


- Ubuntu 14.04 Droplet
- sudo user

# Step 1 — Installing MongoDB


Import the MongoDB repository’s public key.


```
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10

```


Create a list file for MongoDB.


```
echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' | sudo tee /etc/apt/sources.list.d/mongodb.list

```


Reload the local package database.


```
sudo apt-get update

```


Install the MongoDB packages:


```
sudo apt-get install -y mongodb-org

```


Notice that each package contains the associated version number.


Once the installation completes you can start, stop, and check the status of the service. It will start automatically after installation.


Try to connect to the MongoDB instance running as service:


```
mongo

```


If it is up and running, you will see something like this:


```
MongoDB shell version: 2.6.9
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user

```


This means the database server is running! You can exit now:


```
exit

```


# Step 2 — Installing Java


Java is a prerequisite for Elasticsearch. Let’s install it now.


First, add the repository:


```
sudo apt-add-repository ppa:webupd8team/java

```


Update your package lists again:


```
sudo apt-get update

```


Install Java:


```
sudo apt-get install oracle-java8-installer

```


When prompted to accept the license, select <Ok> and then <Yes>.


# Step 3 — Installing Elasticsearch


Now we’ll install Elasticsearch.


First, create a new directory where you will install the search software, and move into it.


```
mkdir ~/utils
cd ~/utils

```


Visit Elasticsearch’s download page to see the latest version.


Now download the latest version of Elasticsearch. At the time of writing this article, the latest version was 1.5.0.


```
wget https://download.elasticsearch.org/elasticsearch/elasticsearch/elasticsearch-1.5.0.zip

```


Install unzip:


```
sudo apt-get install unzip

```


Unzip the archive:


```
unzip elasticsearch-1.5.0.zip

```


Navigate to the directory where you extracted it:


```
cd elasticsearch-1.5.0

```


Launch Elasticsearch by issuing the following command:


```
bin/elasticsearch

```


It will take a few seconds for Elasticsearch to start up. You’ll see some startup logs as it does. Elasticsearch will now be running in the terminal window.



Note: At some point you may want to run Elasticsearch as a service so you can control it with sudo service elasticsearch restart and similar commands; see this tutorial about Upstart for tips. Alternately, you can install Elasticsearch from Ubuntu’s repositories, although you’ll probably get an older version.

Keep this terminal open. Make another SSH connection to your server in another terminal window and check if your instance is up and running:


```
curl -XGET http://localhost:9200

```


9200 is the default port for Elasticsearch. If everything goes well, you will see output similar to that shown below:


```
{
  "status" : 200,
  "name" : "Northstar",
  "cluster_name" : "elasticsearch",
  "version" : {
    "number" : "1.5.0",
    "build_hash" : "927caff6f05403e936c20bf4529f144f0c89fd8c",
    "build_timestamp" : "2015-03-23T14:30:58Z",
    "build_snapshot" : false,
    "lucene_version" : "4.10.4"
  },
  "tagline" : "You Know, for Search"
}

```



Note: For the later part of this article, when you will be copying data, make sure that Elasticsearch is running (and on port 9200).

# Step 4 — Installing Mercurial


Next we’ll install the revision control tool Mercurial.


```
sudo apt-get install mercurial

```


Verify that Mercurial is installed correctly:


```
hg

```


You will get the following output if it is installed correctly:


```
Mercurial Distributed SCM

basic commands:

. . .


```


# Step 5 — Installing Go


Transporter is written in the Go language. So, you need to install golang on your system.


```
sudo apt-get install golang

```


For Go to work properly, you need to set the following environment variables:


Create a folder for Go from your $HOME directory:


```
mkdir ~/go; echo "export GOPATH=$HOME/go" >> ~/.bashrc

```


Update your path:


```
echo "export PATH=$PATH:$HOME/go/bin:/usr/local/go/bin" >> ~/.bashrc

```


Log out of your current SSH session and log in again. You can close just the session where you’ve been working and keep the Elasticsearch session running. This step is crucial for your environment variables to get updated. Log in again, and verify that your variable has been added:


```
echo $GOPATH

```


This should display the new path for Go. In our case, it will be:


```
/home/sammy/go

```


If it does not display the path correctly, please double-check the steps in this section.


Once our $GOPATH is set correctly, we need to check that Go is installed correctly by building a simple program.


Create a file named hello.go and put the following program in it. You can use any text editor you want. We are going to use the nano text editor in this article. Type the following command to create a new file:


```
nano ~/hello.go

```


Now copy this brief “Hello, world” program below to the newly opened file. The entire point of this file is to help us verify that Go is working.


```
package main;
import "fmt"

func main() {
    fmt.Printf("Hello, world\n")
}

```


Once done, press CTRL+X to exit the file. It will prompt you to save the file. Press Y and then press ENTER. it will ask you if you want to change the file name. Press ENTER again to save the current file.


Then, from your home directory, run the file with Go:


```
go run hello.go

```


You should see this output:


```
Hello, world

```


If you see the “Hello, world” message, then Go is installed correctly.


Now go to the $GOPATH directory and create the subdirectories src, pkg and bin. These directories constitute a workspace for Go.


```
cd $GOPATH
mkdir src pkg bin

```


- src contains Go source files organized into packages (one package per directory)
- pkg contains package objects
- bin contains executable commands

# Step 6 — Installing Git


We’ll use Git to install Transporter. Install Git with the following command:


```
sudo apt-get install git

```


# Step 7 — Installing Transporter


Now create and move into a new directory for Transporter. Since the utility was developed by Compose, we’ll call the directory compose.


```
mkdir -p $GOPATH/src/github.com/compose
cd $GOPATH/src/github.com/compose

```


This is where compose/transporter will be installed.


Clone the Transporter GitHub repository:


```
git clone https://github.com/compose/transporter.git

```


Move into the new directory:


```
cd transporter

```


Take ownership of the /usr/lib/go directory:


```
sudo chown -R $USER /usr/lib/go

```


Make sure build-essential is installed for GCC:


```
sudo apt-get install build-essential

```


Run the go get command to get all the dependencies:


```
go get -a ./cmd/...

```


This step might take a while, so be patient. Once it’s done you can build Transporter.


```
go build -a ./cmd/...

```


If all goes well, it will complete without any errors or warnings. Check that Transporter is installed correctly by running this command:


```
transporter

```


You should see output like this:


```
usage: transporter [--version] [--help] <command> [<args>]

Available commands are:
    about    Show information about database adaptors
    eval     Eval javascript to build and run a transporter application
    
. . .
    

```


So the installation is complete. Now, we need some test data in MongoDB that we want to sync to Elasticsearch.


Troubleshooting:


If you get the following error:


```
transporter: command not found

```


This means that your $GOPATH was not added to your PATH variable. Check that you correctly executed the command:


```
echo "export PATH=$PATH:$HOME/go/bin:/usr/local/go/bin" >> ~/.bashrc

```


Try logging out and logging in again. If the error still persists, use the following command instead:


```
$GOPATH/bin/transporter

```


# Step 8 — Creating Sample Data


Now that we have everything installed, we can proceed to the data syncing part.


Connect to MongoDB:


```
mongo

```


You should now see the MongoDB prompt, >. Create a database named foo.


```
use foo

```


Insert some sample documents into a collection named bar:


```
db.bar.save({"firstName": "Robert", "lastName": "Baratheon"});
db.bar.save({"firstName": "John", "lastName": "Snow"});

```


Select the contents you just entered:


```
db.bar.find().pretty();

```


This should display the results shown below (ObjectId will be different on your machine):


```
{
	"_id" : ObjectId("549c3ef5a0152464dde10bc4"),
	"firstName" : "Robert",
	"lastName" : "Baratheon"
}
{
	"_id" : ObjectId("549c3f03a0152464dde10bc5"),
	"firstName" : "John",
	"lastName" : "Snow"
}

```


Now you can exit from the database:


```
exit

```


A bit of terminology:


- A database in MongoDB is analogous to an index in Elasticsearch
- A collection in MongoDB is analogous to a type in Elasticsearch

Our ultimate goal is to sync the data from the bar collection of the foo database from MongoDB to the bar type of the foo index in Elasticsearch.


# Step 9 — Configuring Transporter


Now, we can move on to the configuration changes to migrate our data from MongoDB to Elasticsearch. Transporter requires a config file (config.yaml), a transform file (myTransformation.js), and an application file (application.js)


- The config file specifies the nodes, types, and URIs
- The application file specifies the data flow from source to destination and optional transformation steps
- The transform file applies transformations to the data


Note:  All the commands in this section assume that you are executing the commands from the transporter directory.

Move to the transporter directory:


```
cd ~/go/src/github.com/compose/transporter

```


## Config File


You can take a look at the example config.yaml file if you like. We’re going to back up the original and then replace it with our own contents.


```
mv test/config.yaml test/config.yaml.00

```


The new file is similar but updates some of the URIs and a few of the other settings to match what’s on our server. Let’s copy the contents from here and paste into the new config.yaml file. Use nano editor again.


```
nano test/config.yaml

```


Copy the contents below into the file. Once done, save the file as described earlier.


```
# api:
#   interval: 60s
#   uri: "http://requestb.in/13gerls1"
#   key: "48593282-b38d-4bf5-af58-f7327271e73d"
#   pid: "something-static"
nodes:
  localmongo:
    type: mongo
    uri: mongodb://localhost/foo
  es:
    type: elasticsearch
    uri: http://localhost:9200/
  timeseries:
    type: influx
    uri: influxdb://root:root@localhost:8086/compose
  debug:
    type: file
    uri: stdout://
  foofile:
    type: file
    uri: file:///tmp/foo

```


Notice the nodes section. We have tweaked the localmongo and es nodes slightly compared to the original file. Nodes are the various data sources and destinations. Type defines the type of node. E.g.,


- mongo means it’s a MongoDB instance/cluster
- elasticsearch means it’s an Elasticsearch node
- file means it’s a plain text file

uri will give the API endpoint to connect with the node. The default port will be used for MongoDB (27017) if not specified. Since we need to capture data from the foo database of MongoDB, the URI should look like this:


```
mongodb://localhost/foo

```


Similarly, the URI for Elasticsearch will look like:


```
http://localhost:9200/

```


Save the config.yaml file. You don’t need to make any other changes.


## Application File


Now, open the application.js file in the test directory.


```
nano test/application.js

```


Replace the sample contents of the file with the contents shown below:


```
Source({name:"localmongo", namespace:"foo.bar"})
.transform({filename: "transformers/addFullName.js"})
.save({name:"es", namespace:"foo.bar"});

```


Save the file and exit. Here’s a brief explanation of our pipeline.


- Source(options) identifies the source from which to fetch data
- transform specifies what transformation to apply on each record
- save(options) identifies where to save data

Options include:


- name: name of the node as it appears in the config.yaml file
- namespace: identifies the database and table name; it must be qualified by a dot ( . )

## Transformation File


Now, the last piece of the puzzle is the transformation. If you recall, we stored two records in MongoDB with firstName and lastName. This is where you can see the real power of transforming data as you sync it from MongoDB to Elasticsearch.


Let’s say we want the documents being stored in Elasticsearch to have another field called fullName. For that, we need to create a new transform file, test/transformers/addFullName.js.


```
nano test/transformers/addFullName.js

```


Paste the contents below into the file. Save and exit as described earlier.


```
module.exports = function(doc) {
  doc._id = doc._id['$oid']; 
  doc["fullName"] = doc["firstName"] + " " + doc["lastName"];
  return doc
}

```


The first line is necessary to tackle the way Transporter handles MongoDB’s ObjectId() field. The second line tells Transporter to concatenate firstName and lastName to form fullName.


This is a simple transformation for the example, but with a little JavaScript you can do more complex data manipulation as you prepare your data for searching.


# Step 10 — Executing the Transformation


Now that we are done with the setup, it’s time to sync and transform our data.


Make sure Elasticsearch is running! If it isn’t, start it again in a new terminal window:


```
~/utils/elasticsearch-1.5.0/bin/elasticsearch

```


In your original terminal, make sure you are in the transporter directory:


```
cd ~/go/src/github.com/compose/transporter

```


Execute the following command to copy the data:


```
transporter run --config ./test/config.yaml ./test/application.js

```


The run command of Transporter expects two arguments. First is the config file and second is the application file. If all goes well, the command will complete without any errors.


Check Elasticsearch to verify that the data got copied, with our transformation:


```
curl -XGET localhost:9200/foo/bar/_search?pretty=true

```


You will get a result like this:


```
{
  "took" : 10,
  "timed_out" : false,
  "_shards" : {
    "total" : 5,
    "successful" : 5,
    "failed" : 0
  },
  "hits" : {
    "total" : 2,
    "max_score" : 1.0,
    "hits" : [ {
      "_index" : "foo",
      "_type" : "bar_full_name",
      "_id" : "549c3ef5a0152464dde10bc4",
      "_score" : 1.0,
      "_source":{"_id":"549c3ef5a0152464dde10bc4","firstName":"Robert","fullName":"Robert Baratheon","lastName":"Baratheon"}
    }, {
      "_index" : "foo",
      "_type" : "bar_full_name",
      "_id" : "549c3f03a0152464dde10bc5",
      "_score" : 1.0,
      "_source":{"_id":"549c3f03a0152464dde10bc5","firstName":"John","fullName":"John Snow","lastName":"Snow"}
    } ]
  }
}

```


Notice the field fullName, which contains the firstName and lastName concatenated by a space in between — our transformation worked.


## Conclusion


Now we know how to use Transporter to copy data from MongoDB to Elasticsearch, and how to apply transformations to our data while syncing. You can apply much more complex transformations in the same way. Also, you can chain multiple transformations in the pipeline.


It’s a good practice that if you are doing multiple transformations, keep them in separate files, and chain them. This way, you are making each one of your transformations usable independently for the future.


So, that’s pretty much it. You can check out the Transporter project on GitHub to stay updated for the latest changes in the API.


You might also want to check out this tutorial about basic CRUD operations in Elasticsearch.


