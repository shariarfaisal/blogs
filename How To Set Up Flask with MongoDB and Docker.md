# How To Set Up Flask with MongoDB and Docker

```Docker``` ```Python``` ```Python Frameworks``` ```Nginx``` ```MongoDB```

The author selected the Internet Archive to receive a donation as part of the Write for DOnations program.


## Introduction


Developing web applications can become complex and time consuming when building and maintaining a number of different technologies. Considering lighter weight options designed to reduce complexity and time-to-production for your application can result in a more flexible and scalable solution. As a micro web framework built on Python, Flask provides an extensible way for developers to grow their applications through extensions that can be integrated into projects. To continue the scalability of a developer’s tech stack, MongoDB is a NoSQL database is designed to scale and work with frequent changes. Developers can use Docker to simplify the process of packaging and deploying their applications.


Docker Compose has further simplified the development environment by allowing you to define your infrastructure, including your application services, network volumes, and bind mounts, in a single file. Using Docker Compose provides ease of use over running multiple docker container run commands. It allows you to define all your services in a single Compose file, and with a single command you create and start all the services from your configuration. This ensures that there is version control throughout your container infrastructure. Docker Compose uses a project name to isolate environments from each other, this allows you to run multiple environments on a single host.


In this tutorial you will build, package, and run your to-do web application with Flask, Nginx, and MongoDB inside of Docker containers. You will define the entire stack configuration in a docker-compose.yml file, along with configuration files for Python, MongoDB, and Nginx. Flask requires a web server to serve HTTP requests, so you will also use Gunicorn, which is a Python WSGI HTTP Server, to serve the application. Nginx acts as a reverse proxy server that forwards requests to Gunicorn for processing.


# Prerequisites


To follow this tutorial, you will need the following:


- A non-root user with sudo privileges configured by following the steps in the Initial Server Setup tutorial.
- Docker installed with the instructions from Step 1 and Step 2 of How To Install and Use Docker.
- Docker Compose installed with the instructions from Step 1 of How To Install Docker Compose.

# Step 1 — Writing the Stack Configuration in Docker Compose


Building your applications on Docker allows you to version infrastructure easily depending on configuration changes you make in Docker Compose. The infrastructure can be defined in a single file and built with a single command. In this step, you will set up the docker-compose.yml file to run your Flask application.


The docker-compose.yml file lets you define your application infrastructure as individual services. The services can be connected to each other and each can have a volume attached to it for persistent storage. Volumes are stored in a part of the host filesystem managed by Docker (/var/lib/docker/volumes/ on Linux).


Volumes are the best way to persist data in Docker, as the data in the volumes can be exported or shared with other applications. For additional information about sharing data in Docker, you can refer to How To Share Data Between the Docker Container and the Host.


To get started, create a directory for the application in the home directory on your server:


```
mkdir flaskapp


```


Move into the newly created directory:


```
cd flaskapp


```


Next, create the docker-compose.yml file:


```
nano docker-compose.yml


```


The docker-compose.yml file starts with a version number that identifies the Docker Compose file version. Docker Compose file version 3 targets Docker Engine version 1.13.0+, which is a prerequisite for this setup. You will also add the services tag that you will define in the next step:


docker-compose.yml
```
version: '3'
services:

```


You will now define flask as the first service in your docker-compose.yml file. Add the following code to define the Flask service:


docker-compose.yml
```
. . .
  flask:
    build:
      context: app
      dockerfile: Dockerfile
    container_name: flask
    image: digitalocean.com/flask-python:3.6
    restart: unless-stopped
    environment:
      APP_ENV: "prod"
      APP_DEBUG: "False"
      APP_PORT: 5000
      MONGODB_DATABASE: flaskdb
      MONGODB_USERNAME: flaskuser
      MONGODB_PASSWORD: your_mongodb_password
      MONGODB_HOSTNAME: mongodb
    volumes:
      - appdata:/var/www
    depends_on:
      - mongodb
    networks:
      - frontend
      - backend

```


The build property defines the context of the build. In this case, the app folder that will contain the Dockerfile.


You use the container_name property to define a name for each container. The image property specifies the image name and what the Docker image will be tagged as. The restart property defines how the container should be restarted—in your case it is unless-stopped. This means your containers will only be stopped when the Docker Engine is stopped/restarted or when you explicitly stop the containers. The benefit of using the unless-stopped property is that the containers will start automatically once the Docker Engine is restarted or any error occurs.


The environment property contains the environment variables that are passed to the container. You need to provide a secure password for the environment variable MONGODB_PASSWORD. The volumes property defines the volumes the service is using. In your case the volume appdata is mounted inside the container at the /var/www directory. The depends_on property defines a service that Flask depends on to function properly. In this case, the flask service will depend on mongodb since the mongodb service acts as the database for your application. depends_on ensures that the flask service only runs if the mongodb service is running.


The networks property specifies frontend and backend as the networks the flask service will have access to.


With the flask service defined, you’re ready to add the MongoDB configuration to the file. In this example, you will use the official 4.0.8 version mongo image. Add the following code to your docker-compose.yml file following the flask service:


docker-compose.yml
```
. . .
  mongodb:
    image: mongo:4.0.8
    container_name: mongodb
    restart: unless-stopped
    command: mongod --auth
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongodbuser
      MONGO_INITDB_ROOT_PASSWORD: your_mongodb_root_password
      MONGO_INITDB_DATABASE: flaskdb
      MONGODB_DATA_DIR: /data/db
      MONDODB_LOG_DIR: /dev/null
    volumes:
      - mongodbdata:/data/db
    networks:
      - backend

```


The container_name for this service is mongodb with a restart policy of unless-stopped. You use the command property to define the command that will be executed when the container is started. The command mongod --auth will disable logging into the MongoDB shell without credentials, which will secure MongoDB by requiring authentication.


The environment variables MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD create a root user with the given credentials, so be sure to replace the placeholder with a strong password.


MongoDB stores its data in /data/db by default, therefore the data in the /data/db folder will be written to the named volume mongodbdata for persistence. As a result you won’t lose your databases in the event of a restart. The mongoDB service does not expose any ports, so the service will only be accessible through the backend network.


Next, you will define the web server for your application. Add the following code to your docker-compose.yml file to configure Nginx:


docker-compose.yml
```
. . .
  webserver:
    build:
      context: nginx
      dockerfile: Dockerfile
    image: digitalocean.com/webserver:latest
    container_name: webserver
    restart: unless-stopped
    environment:
      APP_ENV: "prod"
      APP_NAME: "webserver"
      APP_DEBUG: "false"
      SERVICE_NAME: "webserver"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - nginxdata:/var/log/nginx
    depends_on:
      - flask
    networks:
      - frontend

```


Here you have defined the context of the build, which is the nginx folder containing the Dockerfile. With the image property, you specify the image used to tag and run the container. The ports property will configure the Nginx service to be publicly accessible through :80 and :443 and volumes mounts the nginxdata volume inside the container at /var/log/nginx directory.


You’ve defined the service on which the web server service depends_on as flask. Finally the networks property defines the networks web server service will have access to the frontend.


Next, you will create bridge networks to allow the containers to communicate with each other. Append the following lines to the end of your file:


docker-compose.yml
```
. . .
networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

```


You defined two networks—frontend and backend—for the services to connect to. The front-end services, such as Nginx, will connect to the frontend network since it needs to be publicly accessible. Back-end services, such as MongoDB, will connect to the backend network to prevent unauthorized access to the service.


Next, you will use volumes to persist the database, application, and configuration files. Since your application will use the databases and files, it is imperative to persist the changes made to them. The volumes are managed by Docker and stored on the filesystem. Add this code to the docker-compose.yml file to configure the volumes:


docker-compose.yml
```
. . .
volumes:
  mongodbdata:
    driver: local
  appdata:
    driver: local
  nginxdata:
    driver: local

```


The volumes section declares the volumes that the application will use to persist data. Here you have defined the volumes mongodbdata, appdata, and nginxdata for persisting your MongoDB databases, Flask application data, and the Nginx web server logs, respectively. All of these volumes use a local driver to store the data locally. The volumes are used to persist this data so that data like your MongoDB databases and Nginx webserver logs could be lost once you restart the containers.


Your complete docker-compose.yml file will look like this:


docker-compose.yml
```
version: '3'
services:

  flask:
    build:
      context: app
      dockerfile: Dockerfile
    container_name: flask
    image: digitalocean.com/flask-python:3.6
    restart: unless-stopped
    environment:
      APP_ENV: "prod"
      APP_DEBUG: "False"
      APP_PORT: 5000
      MONGODB_DATABASE: flaskdb
      MONGODB_USERNAME: flaskuser
      MONGODB_PASSWORD: your_mongodb_password
      MONGODB_HOSTNAME: mongodb
    volumes:
      - appdata:/var/www
    depends_on:
      - mongodb
    networks:
      - frontend
      - backend

  mongodb:
    image: mongo:4.0.8
    container_name: mongodb
    restart: unless-stopped
    command: mongod --auth
    environment:
      MONGO_INITDB_ROOT_USERNAME: mongodbuser
      MONGO_INITDB_ROOT_PASSWORD: your_mongodb_root_password
      MONGO_INITDB_DATABASE: flaskdb
      MONGODB_DATA_DIR: /data/db
      MONDODB_LOG_DIR: /dev/null
    volumes:
      - mongodbdata:/data/db
    networks:
      - backend

  webserver:
    build:
      context: nginx
      dockerfile: Dockerfile
    image: digitalocean.com/webserver:latest
    container_name: webserver
    restart: unless-stopped
    environment:
      APP_ENV: "prod"
      APP_NAME: "webserver"
      APP_DEBUG: "true"
      SERVICE_NAME: "webserver"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - nginxdata:/var/log/nginx
    depends_on:
      - flask
    networks:
      - frontend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge

volumes:
  mongodbdata:
    driver: local
  appdata:
    driver: local
  nginxdata:
    driver: local

```


Save the file and exit the editor after verifying your configuration.


You’ve defined the Docker configuration for your entire application stack in the docker-compose.yml file. You will now move on to writing the Dockerfiles for Flask and the web server.


# Step 2 — Writing the Flask and Web Server Dockerfiles


With Docker, you can build containers to run your applications from a file called Dockerfile. The Dockerfile is a tool that enables you to create custom images that you can use to install the software required by your application and configure your containers based on your requirements. You can push the custom images you create to Docker Hub or any private registry.


In this step, you’ll write the Dockerfiles for the Flask and web server services. To get started, create the app directory for your Flask application:


```
mkdir app


```


Next, create the Dockerfile for your Flask app in the app directory:


```
nano app/Dockerfile


```


Add the following code to the file to customize your Flask container:


app/Dockerfile
```
FROM python:3.6.8-alpine3.9

LABEL MAINTAINER="FirstName LastName <example@domain.com>"

ENV GROUP_ID=1000 \
    USER_ID=1000

WORKDIR /var/www/

```


In this Dockerfile, you are creating an image on top of the 3.6.8-alpine3.9 image that is based on Alpine 3.9 with Python 3.6.8 pre-installed.


The ENV directive is used to define the environment variables for our group and user ID.
Linux Standard Base (LSB) specifies that UIDs and GIDs 0-99 are statically allocated by the system. UIDs 100-999 are supposed to be allocated dynamically for system users and groups. UIDs 1000-59999 are supposed to be dynamically allocated for user accounts. Keeping this in mind you can safely assign a UID and GID of 1000, furthermore you can change the UID/GID by updating the GROUP_ID and USER_ID to match your requirements.


The WORKDIR directive defines the working directory for the container. Be sure to replace the LABEL MAINTAINER field with your name and email address.


Add the following code block to copy the Flask application into the container and install the necessary dependencies:


app/Dockerfile
```
. . .
ADD ./requirements.txt /var/www/requirements.txt
RUN pip install -r requirements.txt
ADD . /var/www/
RUN pip install gunicorn

```


The following code will use the ADD directive to copy files from the local app directory to the /var/www directory on the container. Next, the Dockerfile will use the RUN directive to install Gunicorn and the packages specified in the requirements.txt file, which you will create later in the tutorial.


The following code block adds a new user and group and initializes the application:


app/Dockerfile
```
. . .
RUN addgroup -g $GROUP_ID www
RUN adduser -D -u $USER_ID -G www www -s /bin/sh

USER www

EXPOSE 5000

CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "wsgi"]

```


By default, Docker containers run as the root user. The root user has access to everything in the system, so the implications of a security breach can be disastrous. To mitigate this security risk, this will create a new user and group that will only have access to the /var/www directory.


This code will first use the addgroup command to create a new group named www. The -g flag will set the group ID to the ENV GROUP_ID=1000 variable that is defined earlier in the Dockerfile.


The adduser -D -u $USER_ID -G www www -s /bin/sh lines creates a www user with a user ID of 1000, as defined by the ENV variable. The -s flag creates the user’s home directory if it does not exist and sets the default login shell to /bin/sh. The -G flag is used to set the user’s initial login group to www, which was created by the previous command.


The USER command defines that the programs run in the container will use the www user. Gunicorn will listen on :5000, so you will open this port with the EXPOSE command.


Finally, the CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "wsgi"] line runs the command to start the Gunicorn server with four workers listening on port 5000. The number should generally be between 2–4 workers per core in the server, Gunicorn documentation recommends (2 x $num_cores) + 1 as the number of workers to start with.


Your completed Dockerfile will look like the following:


app/Dockerfile
```
FROM python:3.6.8-alpine3.9

LABEL MAINTAINER="FirstName LastName <example@domain.com>"

ENV GROUP_ID=1000 \
    USER_ID=1000

WORKDIR /var/www/

ADD . /var/www/
RUN pip install -r requirements.txt
RUN pip install gunicorn

RUN addgroup -g $GROUP_ID www
RUN adduser -D -u $USER_ID -G www www -s /bin/sh

USER www

EXPOSE 5000

CMD [ "gunicorn", "-w", "4", "--bind", "0.0.0.0:5000", "wsgi"]

```


Save the file and exit the text editor.


Next, create a new directory to hold your Nginx configuration:


```
mkdir nginx


```


Then create the Dockerfile for your Nginx web server in the nginx directory:


```
nano nginx/Dockerfile


```


Add the following code to the file to create the Dockerfile that will build the image for your Nginx container:


nginx/Dockerfile
```
FROM alpine:latest

LABEL MAINTAINER="FirstName LastName <example@domain.com>"

RUN apk --update add nginx && \
    ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log && \
    mkdir /etc/nginx/sites-enabled/ && \
    mkdir -p /run/nginx && \
    rm -rf /etc/nginx/conf.d/default.conf && \
    rm -rf /var/cache/apk/*

COPY conf.d/app.conf /etc/nginx/conf.d/app.conf

EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]

```


This Nginx Dockerfile uses an alpine base image, which is a tiny Linux distribution with a minimal attack surface built for security.


In the RUN directive you are installing nginx as well as creating symbolic links to publish the error and access logs to the standard error (/dev/stderr) and output (/dev/stdout). Publishing errors to standard error and output is a best practice since containers are ephemeral, doing this the logs are shipped to docker logs and from there you can forward your logs to a logging service like the Elastic stack for persistance. After this is done, commands are run to remove the default.conf and /var/cache/apk/* to reduce the size of the resulting image. Executing all of these commands in a single RUN decreases the number of layers in the image, which also reduces the size of the resulting image.


The COPY directive copies the app.conf web server configuration inside of the container. The EXPOSE directive ensures the containers listen on ports :80 and :443, as your application will run on :80 with :443 as the secure port.


Finally, the CMD directive defines the command to start the Nginx server.


Save the file and exit the text editor.


Now that the Dockerfile is ready, you are ready to configure the Nginx reverse proxy to route traffic to the Flask application.


# Step 3 — Configuring the Nginx Reverse Proxy


In this step, you will configure Nginx as a reverse proxy to forward requests to Gunicorn on :5000. A reverse proxy server is used to direct client requests to the appropriate back-end server. It provides an additional layer of abstraction and control to ensure the smooth flow of network traffic between clients and servers.


Get started by creating the nginx/conf.d directory:


```
mkdir nginx/conf.d


```


To configure Nginx, you need to create an app.conf file with the following configuration in the nginx/conf.d/ folder. The app.conf file contains the configuration that the reverse proxy needs to forward the requests to Gunicorn.


```
nano nginx/conf.d/app.conf


```


Put the following contents into the app.conf file:


nginx/conf.d/app.conf
```
upstream app_server {
    server flask:5000;
}

server {
    listen 80;
    server_name _;
    error_log  /var/log/nginx/error.log;
    access_log /var/log/nginx/access.log;
    client_max_body_size 64M;

    location / {
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
        gzip_static on;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_buffering off;
        proxy_redirect off;
        proxy_pass http://app_server;
    }
}

```


This will first define the upstream server, which is commonly used to specify a web or app server for routing or load balancing.


Your upstream server, app_server, defines the server address with the server directive, which is identified by the container name flask:5000.


The configuration for the Nginx web server is defined in the server block. The listen directive defines the port number on which your server will listen for incoming requests. The error_log and access_log directives define the files for writing logs. The proxy_pass directive is used to set the upstream server for forwarding the requests to http://app_server.


Save and close the file.


With the Nginx web server configured, you can move on to creating the Flask to-do API.


# Step 4 — Creating the Flask To-do API


Now that you’ve built out your environment, you’re ready to build your application. In this step, you will write a to-do API application that will save and display to-do notes sent in from a POST request.


Get started by creating the requirements.txt file in the app directory:


```
nano app/requirements.txt


```


This file is used to install the dependencies for your application. The implementation of this tutorial will use Flask, Flask-PyMongo, and requests. Add the following to the requirements.txt file:


app/requirements.txt
```
Flask==1.0.2
Flask-PyMongo==2.2.0
requests==2.20.1

```


Save the file and exit the editor after entering the requirements.


Next, create the app.py file to contain the Flask application code in the app directory:


```
nano app/app.py


```


In your new app.py file, enter in the code to import the dependencies:


app/app.py
```
import os
from flask import Flask, request, jsonify
from flask_pymongo import PyMongo

```


The os package is used to import the environment variables. From the flask library you imported the Flask, request, and jsonify objects to instantiate the application, handle requests, and send JSON responses, respectively. From flask_pymongo you imported the PyMongo object to interact with the MongoDB.


Next, add the code needed to connect to MongoDB:


app/app.py
```
. . .
application = Flask(__name__)

application.config["MONGO_URI"] = 'mongodb://' + os.environ['MONGODB_USERNAME'] + ':' + os.environ['MONGODB_PASSWORD'] + '@' + os.environ['MONGODB_HOSTNAME'] + ':27017/' + os.environ['MONGODB_DATABASE']

mongo = PyMongo(application)
db = mongo.db

```


The Flask(__name__) loads the application object into the application variable. Next, the code builds the MongoDB connection string from the environment variables using os.environ. Passing the application object in to the PyMongo() method will give you the mongo object, which in turn gives you the db object from mongo.db.


Now you will add the code to create an index message:


app/app.py
```
. . .
@application.route('/')
def index():
    return jsonify(
        status=True,
        message='Welcome to the Dockerized Flask MongoDB app!'
    )

```


The @application.route('/') defines the / GET route of your API. Here your index() function returns a JSON string using the jsonify method.


Next, add the /todo route to list all to-do’s:


app/app.py
```
. . .
@application.route('/todo')
def todo():
    _todos = db.todo.find()

    item = {}
    data = []
    for todo in _todos:
        item = {
            'id': str(todo['_id']),
            'todo': todo['todo']
        }
        data.append(item)

    return jsonify(
        status=True,
        data=data
    )

```


The @application.route('/todo') defines the /todo GET route of your API, which returns the to-dos in the database. The db.todo.find() method returns all the to-dos in the database. Next, you iterate over the _todos to build an item that includes only the id and todo from the objects appending them to a data array and finally returns them as JSON.


Next, add the code for creating the to-do:


app/app.py
```
. . .
@application.route('/todo', methods=['POST'])
def createTodo():
    data = request.get_json(force=True)
    item = {
        'todo': data['todo']
    }
    db.todo.insert_one(item)

    return jsonify(
        status=True,
        message='To-do saved successfully!'
    ), 201

```


The @application.route('/todo') defines the /todo POST route of your API, which creates a to-do note in the database. The request.get_json(force=True) gets the JSON that you post to the route, and item is used to build the JSON that will be saved in the to-do. The db.todo.insert_one(item) is used to insert one item into the database. After the to-do is saved in the database you return a JSON response with a status code of 201 CREATED.


Now you add the code to run the application:


app/app.py
```
. . .
if __name__ == "__main__":
    ENVIRONMENT_DEBUG = os.environ.get("APP_DEBUG", True)
    ENVIRONMENT_PORT = os.environ.get("APP_PORT", 5000)
    application.run(host='0.0.0.0', port=ENVIRONMENT_PORT, debug=ENVIRONMENT_DEBUG)

```


The condition __name__ == "__main__" is used to check if the global variable, __name__, in the module is the entry point to your program, is "__main__", then run the application. If the __name__ is equal to "__main__" then the code inside the if block will execute the app using this command application.run(host='0.0.0.0', port=ENVIRONMENT_PORT, debug=ENVIRONMENT_DEBUG).


Next, we get the values for the ENVIRONMENT_DEBUG and ENVIRONMENT_PORT from the environment variables using os.environ.get(), using the key as the first parameter and default value as the second parameter. The application.run() sets the host, port, and debug values for the application.


The completed app.py file will look like this:


app/app.py
```
import os
from flask import Flask, request, jsonify
from flask_pymongo import PyMongo

application = Flask(__name__)

application.config["MONGO_URI"] = 'mongodb://' + os.environ['MONGODB_USERNAME'] + ':' + os.environ['MONGODB_PASSWORD'] + '@' + os.environ['MONGODB_HOSTNAME'] + ':27017/' + os.environ['MONGODB_DATABASE']

mongo = PyMongo(application)
db = mongo.db

@application.route('/')
def index():
    return jsonify(
        status=True,
        message='Welcome to the Dockerized Flask MongoDB app!'
    )

@application.route('/todo')
def todo():
    _todos = db.todo.find()

    item = {}
    data = []
    for todo in _todos:
        item = {
            'id': str(todo['_id']),
            'todo': todo['todo']
        }
        data.append(item)

    return jsonify(
        status=True,
        data=data
    )

@application.route('/todo', methods=['POST'])
def createTodo():
    data = request.get_json(force=True)
    item = {
        'todo': data['todo']
    }
    db.todo.insert_one(item)

    return jsonify(
        status=True,
        message='To-do saved successfully!'
    ), 201

if __name__ == "__main__":
    ENVIRONMENT_DEBUG = os.environ.get("APP_DEBUG", True)
    ENVIRONMENT_PORT = os.environ.get("APP_PORT", 5000)
    application.run(host='0.0.0.0', port=ENVIRONMENT_PORT, debug=ENVIRONMENT_DEBUG)

```


Save the file and exit the editor.


Next, create the wsgi.py file in the app directory.


```
nano app/wsgi.py


```


The wsgi.py file creates an application object (or callable) so that the server can use it. Each time a request comes, the server uses this application object to run the application’s request handlers upon parsing the URL.


Put the following contents into the wsgi.py file, save the file, and exit the text editor:


app/wsgi.py
```
from app import application

if __name__ == "__main__":
  application.run()

```


This wsgi.py file imports the application object from the app.py file and creates an application object for the Gunicorn server.


The to-do app is now in place, so you’re ready to start running the application in containers.


# Step 5 — Building and Running the Containers


Now that you have defined all of the services in your docker-compose.yml file and their configurations, you can start the containers.


Since the services are defined in a single file, you need to issue a single command to start the containers, create the volumes, and set up the networks. This command also builds the image for your Flask application and the Nginx web server. Run the following command to build the containers:


```
docker-compose up -d


```


When running the command for the first time, it will download all of the necessary Docker images, which can take some time. Once the images are downloaded and stored in your local machine, docker-compose will create your containers. The -d flag daemonizes the process, which allows it to run as a background process.


Use the following command to list the running containers once the build process is complete:


```
docker ps


```


You will see output similar to the following:


```
OutputCONTAINER ID        IMAGE                           COMMAND                  CREATED             STATUS              PORTS                                      NAMES
f20e9a7fd2b9        digitalocean.com/webserver:latest   "nginx -g 'daemon of…"   2 weeks ago         Up 2 weeks          0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   webserver
3d53ea054517        digitalocean.com/flask-python:3.6   "gunicorn -w 4 --bin…"   2 weeks ago         Up 2 weeks          5000/tcp                                   flask
96f5a91fc0db        mongo:4.0.8                     "docker-entrypoint.s…"   2 weeks ago         Up 2 weeks          27017/tcp                                  mongodb

```


The CONTAINER ID is a unique identifier that is used to access containers. The IMAGE defines the image name for the given container. The NAMES field is the service name under which containers are created, similar to CONTAINER ID these can be used to access containers. Finally, the STATUS provides information regarding the state of the container whether it’s running, restarting, or stopped.


You’ve used the docker-compose command to build your containers from your configuration files. In the next step, you will create a MongoDB user for your application.


# Step 6 — Creating a User for Your MongoDB Database


By default, MongoDB allows users to log in without credentials and grants unlimited privileges. In this step, you will secure your MongoDB database by creating a dedicated user to access it.


To do this, you will need the root username and password that you set in the docker-compose.yml file environment variables MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD for the mongodb service. In general, it’s better to avoid using the root administrative account when interacting with the database. Instead, you will create a dedicated database user for your Flask application, as well as a new database that the Flask app will be allowed to access.


To create a new user, first start an interactive shell on the mongodb container:


```
docker exec -it mongodb bash


```


You use the docker exec command in order to run a command inside a running container along with the -it flag to run an interactive shell inside the container.


Once inside the container, log in to the MongoDB root administrative account:


```
mongo -u mongodbuser -p


```


You will be prompted for the password that you entered as the value for the MONGO_INITDB_ROOT_PASSWORD variable in the docker-compose.yml file. The password can be changed by setting a new value for the MONGO_INITDB_ROOT_PASSWORD in the mongodb service, in which case you will have to re-run the docker-compose up -d command.


Run the show dbs; command to list all databases:


```
show dbs;


```


You will see the following output:


```
Outputadmin    0.000GB
config   0.000GB
local    0.000GB
5 rows in set (0.00 sec)

```


The admin database is a special database that grants administrative permissions to users. If a user has read access to the admin database, they will have read and write permissions to all other databases. Since the output lists the admin database, the user has access to this database and can therefore read and write to all other databases.


Saving the first to-do note will automatically create the MongoDB database. MongoDB allows you to switch to a database that does not exist using the use database command. It creates a database when a document is saved to a collection. Therefore the database is not created here; that will happen when you save your first to-do note in the database from the API. Execute the use command to switch to the flaskdb database:


```
use flaskdb


```


Next, create a new user that will be allowed to access this database:


```
db.createUser({user: 'flaskuser', pwd: 'your password', roles: [{role: 'readWrite', db: 'flaskdb'}]})
exit


```


This command creates a user named flaskuser with readWrite access to the flaskdb database. Be sure to use a secure password in the pwd field. The user and pwd here are the values you defined in the docker-compose.yml file under the environment variables section for the flask service.


Log in to the authenticated database with the following command:


```
mongo -u flaskuser -p your password --authenticationDatabase flaskdb


```


Now that you have added the user, log out of the database.


```
exit


```


And finally, exit the container:


```
exit


```


You’ve now configured a dedicated database and user account for your Flask application. The database components are ready, so now you can move on to running the Flask to-do app.


# Step 7 — Running the Flask To-do App


Now that your services are configured and running, you can test your application by navigating to http://your_server_ip in a browser. Additionally, you can run curl to see the JSON response from Flask:


```
curl -i http://your_server_ip


```


You will receive the following response:


```
Output{"message":"Welcome to the Dockerized Flask MongoDB app!","status":true}

```


The configuration for the Flask application is passed to the application from the docker-compose.yml file. The configuration regarding the database connection is set using the MONGODB_* variables defined in the environment section of the flask service.


To test everything out, create a to-do note using the Flask API. You can do this with a POST curl request to the /todo route:


```
curl -i -H "Content-Type: application/json" -X POST -d '{"todo": "Dockerize Flask application with MongoDB backend"}' http://your_server_ip/todo


```


This request results in a response with a status code of 201 CREATED when the to-do item is saved to MongoDB:


```
Output{"message":"To-do saved successfully!","status":true}

```


You can list all of the to-do notes from MongoDB with a GET request to the /todo route:


```
curl -i http://your_server_ip/todo


```


```
Output{"data":[{"id":"5c9fa25591cb7b000a180b60","todo":"Dockerize Flask application with MongoDB backend"}],"status":true}

```


With this, you have Dockerized a Flask API running a MongoDB backend with Nginx as a reverse proxy deployed to your servers. For a production environment you can use sudo systemctl enable docker to ensure your Docker service automatically starts at runtime.


# Conclusion


In this tutorial, you deployed a Flask application with Docker, MongoDB, Nginx, and Gunicorn. You now have a functioning modern stateless API application that can be scaled. Although you can achieve this result by using a command like docker container run, the docker-compose.yml simplifies your job as this stack can be put into version control and updated as necessary.


From here you can also take a look at our further Python Framework tutorials.


