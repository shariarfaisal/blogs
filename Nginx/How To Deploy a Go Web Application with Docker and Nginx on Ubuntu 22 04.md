# How To Deploy a Go Web Application with Docker and Nginx on Ubuntu 22 04

```Deployment``` ```Docker``` ```Go``` ```Let's Encrypt``` ```Nginx``` ```Ubuntu 22.04```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Docker is a commonly used containerization software that enables developers to easily package apps along with their environments, allowing for quicker iteration cycles and better resource efficiency while providing the same desired environment on each run. Docker Compose is a container orchestration tool that facilitates modern app requirements. It allows you to run multiple interconnected containers at the same time. Instead of manually running containers, orchestration tools give developers the ability to control, scale, and extend a container simultaneously.


The benefits of using Nginx as a front-end web server are in performance, configurability, and TLS termination, which frees the app from completing these tasks. The nginx-proxy is an automated system for Docker containers that simplifies the process of configuring Nginx as a reverse proxy. Its Let’s Encrypt add-on can accompany the nginx-proxy to automate the generation and renewal of certificates for proxied containers.


In this tutorial, you will deploy an example Go web application with gorilla/mux as the request router and Nginx as the web server, all inside Docker containers that are orchestrated by Docker Compose. You’ll use nginx-proxy with the Let’s Encrypt add-on as the reverse proxy. At the end of this tutorial, you will have deployed a Go web app accessible at your domain with multiple routes, using Docker and secured with Let’s Encrypt certificates.


# Prerequisites


- An Ubuntu 22.04 server with root privileges, and a secondary, non-root account. You can set this up by following this initial server setup guide. For this tutorial, the non-root user is sammy.
- Docker installed by following the first two steps of How To Install Docker on Ubuntu 22.04.
- Docker Compose installed by following the first step of How To Install Docker Compose on Ubuntu 22.04.
- A fully registered domain name. This tutorial will use your_domain throughout. You can get one for free on Freenom, or use the domain registrar of your choice.
- A DNS “A” record with your_domain pointing to your server’s public IP address. You can follow this introduction to DigitalOcean DNS for details on how to add them.
- An understanding of Docker and its architecture. For an introduction to Docker, see The Docker Ecosystem: An Introduction to Common Components.

# Step 1 — Creating an Example Go Web App


In this step, you will set up your workspace and create a simple Go web app, which you’ll later containerize. The Go app will use the powerful gorilla/mux request router, chosen for its flexibility and speed.


For this tutorial, you’ll store all data under ~/go-docker. Run the following command to create this folder:


```
mkdir ~/go-docker


```


Navigate to it:


```
cd ~/go-docker


```


You’ll store your example Go web app in a file named main.go. Create it using your text editor:


```
nano main.go


```


Add the following lines:


~/go-docker/main.go
```
package main

import (
	"fmt"
	"net/http"

	"github.com/gorilla/mux"
)

func main() {
	r := mux.NewRouter()

	r.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "<h1>This is the homepage. Try /hello and /hello/Sammy\n</h1>")
	})

	r.HandleFunc("/hello", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintf(w, "<h1>Hello from Docker!\n</h1>")
	})

	r.HandleFunc("/hello/{name}", func(w http.ResponseWriter, r *http.Request) {
		vars := mux.Vars(r)
		title := vars["name"]

		fmt.Fprintf(w, "<h1>Hello, %s!\n</h1>", title)
	})

	http.ListenAndServe(":80", r)
}

```


You first import the net/http and gorilla/mux packages, which provide HTTP server functionality and routing. The gorilla/mux package implements a more powerful request router and dispatcher, while maintaining interface compatibility with the standard router. You instantiate a new mux router and store it in variable r.


Then, you define three routes: /, /hello, and /hello/{name}. The first (/) serves as the homepage and you include a message for the page. The second (/hello) returns a greeting to the visitor. For the third route (/hello/{name}), you specify that it should take a name as a parameter and show a greeting message with the name inserted.


At the end of your file, you start the HTTP server with http.ListenAndServe and instruct it to listen on port 80, using the router you configured.


Save and close the file.


Before running your Go app, you need to compile and pack it for execution inside a Docker container. Go is a compiled language, so before a program can run, the compiler translates the programming code into executable machine code.


You’ve set up your workspace and created an example Go web app. Next, you will deploy nginx-proxy with an automated Let’s Encrypt certificate provision.


# Step 2 — Deploying nginx-proxy with Let’s Encrypt


It’s important that you secure your app with HTTPS. To accomplish this, you’ll deploy nginx-proxy via Docker Compose, along with its Let’s Encrypt add-on. This setup secures Docker containers proxied using nginx-proxy and takes care of securing your app through HTTPS by automatically handling TLS certificate creation and renewal.


You’ll be storing the Docker Compose configuration for nginx-proxy in a file named nginx-proxy-compose.yaml. Create it by running:


```
nano nginx-proxy-compose.yaml


```


Add the following lines to the file:


~/go-docker/nginx-proxy-compose.yaml
```
version: '3'

services:
  nginx-proxy:
    restart: always
    image: jwilder/nginx-proxy
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/etc/nginx/vhost.d"
      - "/usr/share/nginx/html"
      - "/var/run/docker.sock:/tmp/docker.sock:ro"
      - "/etc/nginx/certs"

  letsencrypt-nginx-proxy-companion:
    restart: always
    image: jrcs/letsencrypt-nginx-proxy-companion
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
    volumes_from:
      - "nginx-proxy"

```


In this file, you define two containers: one for nginx-proxy and one for its Let’s Encrypt add-on (letsencrypt-nginx-proxy-companion). For the proxy, you specify the image jwilder/nginx-proxy, expose and map HTTP and HTTPS ports, and define volumes that will be accessible to the container for persisting Nginx-related data.


In the second block, you name the image for the Let’s Encrypt add-on configuration. Then, you configure access to Docker’s socket by defining a volume and then the existing volumes from the proxy container to inherit. Both containers have the restart property set to always, which instructs Docker to always keep them up (in the case of a crash or a system reboot).


Save and close the file.


Deploy the nginx-proxy by running:


```
docker compose -f nginx-proxy-compose.yaml up -d


```


Docker Compose accepts a custom named file via the -f flag. The up command runs the containers, and the -d flag (detached mode) instructs it to run the containers in the background.


You’ll receive an output like this:


```
Output[+] Running 21/21
 ⠿ letsencrypt-nginx-proxy-companion Pulled                            6.8s
   ⠿ df9b9388f04a Pull complete                                        3.1s
   ⠿ 6c6cfd4eaf5b Pull complete                                        3.9s
   ⠿ 870307501973 Pull complete                                        4.3s
   ⠿ e8ff3435d14f Pull complete                                        4.5s
   ⠿ 5b78ba945919 Pull complete                                        4.8s
   ⠿ 973b2ca26006 Pull complete                                        5.0s
 ⠿ nginx-proxy Pulled                                                  8.1s
   ⠿ 42c077c10790 Pull complete                                        3.9s
   ⠿ 62c70f376f6a Pull complete                                        5.5s
   ⠿ 915cc9bd79c2 Pull complete                                        5.6s
   ⠿ 75a963e94de0 Pull complete                                        5.7s
   ⠿ 7b1fab684d70 Pull complete                                        5.7s
   ⠿ db24d06d5af4 Pull complete                                        5.8s
   ⠿ e917373dbecf Pull complete                                        5.9s
   ⠿ 11e2be9775e9 Pull complete                                        5.9s
   ⠿ 9996fa75bc02 Pull complete                                        6.1s
   ⠿ d37674efdf77 Pull complete                                        6.3s
   ⠿ a45d84576e75 Pull complete                                        6.3s
   ⠿ a13c1f42faf7 Pull complete                                        6.4s
   ⠿ 4f4fb700ef54 Pull complete                                        6.5s
[+] Running 3/3
 ⠿ Network go-docker_default                                Created    0.1s
 ⠿ Container go-docker-nginx-proxy-1                        Started    0.5s
 ⠿ Container go-docker-letsencrypt-nginx-proxy-companion-1  Started    0.8s

```


You’ve deployed nginx-proxy and its Let’s Encrypt companion using Docker Compose. Next, you’ll create a Dockerfile for your Go web app.


# Step 3 — Dockerizing the Go Web App


In this section, you will prepare a Dockerfile containing instructions on how Docker will create an immutable image for your Go web app. Docker builds an immutable app image, similar to a snapshot of the container, using the instructions found in the Dockerfile. The image’s immutability guarantees the same environment each time a container, based on the particular image, is run.


Create the Dockerfile with your text editor:


```
nano Dockerfile


```


Add the following lines:


~/go-docker/Dockerfile
```
FROM golang:alpine AS build
RUN apk --no-cache add gcc g++ make git
WORKDIR /go/src/app
COPY . .
RUN go mod init webserver
RUN go mod tidy
RUN GOOS=linux go build -ldflags="-s -w" -o ./bin/web-app ./main.go

FROM alpine:3.17
RUN apk --no-cache add ca-certificates
WORKDIR /usr/bin
COPY --from=build /go/src/app/bin /go/bin
EXPOSE 80
ENTRYPOINT /go/bin/web-app --port 80

```


This Dockerfile has two stages. The first stage uses the golang:alpine base, which contains pre-installed Go on Alpine Linux.


Then you install gcc, g++, make, and git as the necessary compilation tools for your Go app. You set the working directory to /go/src/app, which is under the default GOPATH. You also copy the content of the current directory into the container. The first stage concludes with recursively fetching the packages used from the code and compiling the main.go file for release without symbol and debug info (by passing -ldflags="-s -w").



Note: When you compile a Go program, it keeps a separate part of the binary that would be used for debugging; however, this extra information uses memory and is not necessary to preserve when deploying to a production environment.

The second stage bases itself on alpine:3.17 (Alpine Linux 3.17). It installs trusted CA certificates, copies the compiled app binaries from the first stage to the current image, exposes port 80, and sets the app binary as the image entry point.


Save and close the file.


You’ve created a Dockerfile for your Go app that will fetch its packages, compile it for release, and run it upon container creation. In the next step, you will create the Docker Compose yaml file and test the app by running it in Docker.


# Step 4 — Creating and Running the Docker Compose File


Now, you’ll create the Docker Compose config file and write the necessary configuration for running the Docker image you created in the previous step. Then, you will run it and check if it works correctly. In general, the Docker Compose config file specifies the containers, settings, networks, and volumes that the app requires. You can also specify that these element start and stop as one.


You will store the Docker Compose configuration for the Go web app in a file named go-app-compose.yaml. Create it by running:


```
nano go-app-compose.yaml


```


Add the following lines to this file:


~/go-docker/go-app-compose.yaml
```
version: '3'
services:
  go-web-app:
    restart: always
    build:
      dockerfile: Dockerfile
      context: .
    environment:
      - VIRTUAL_HOST=your_domain
      - LETSENCRYPT_HOST=your_domain

```


Replace your_domain both times with your domain name. Save and close the file.


This Docker Compose configuration contains one container (go-web-app), which will be your Go web app. It builds the app using the Dockerfile you’ve created in the previous step and takes the current directory, which contains the source code, as the context for building. Furthermore, it sets two environment variables: VIRTUAL_HOST and LETSENCRYPT_HOST. nginx-proxy uses VIRTUAL_HOST to know from which domain to accept the requests. LETSENCRYPT_HOST specifies the domain name for generating TLS certificates and must be the same as VIRTUAL_HOST, unless you specify a wildcard domain.


Now run your Go web app in the background via Docker Compose:


```
docker compose -f go-app-compose.yaml up -d


```


An output like this will print (this output has been truncated for readability):


```
OutputCreating network "go-docker_default" with the default driver
Building go-web-app
Step 1/13 : FROM golang:alpine AS build
 ---> b97a72b8e97d
...
Successfully built 71e4b1ef2e25
Successfully tagged go-docker_go-web-app:latest
...
[+] Running 1/1
 ⠿ Container go-docker-go-web-app-1  Started 

```


If you review the output presented after running the command, Docker logs every step of building the app image according to the configuration in your Dockerfile.


You can now navigate to https://your_domain/ to access your homepage. At your web app’s home address, you can access the page as a result of the / route you defined in the first step.





Now navigate to https://your_domain/hello. The message you defined in your code for the /hello route from Step 1 will load.





Finally, append a name to your web app’s address to test the other route, like: https://your_domain/hello/Sammy.






Note: If you receive an error about invalid TLS certificates, wait a few minutes for the Let’s Encrypt add-on to provision the certificates. If you are still getting errors after a short time, double check what you’ve entered against the commands and configuration in this step.

You’ve created the Docker Compose file and written configuration for running your Go app inside a container. To finish, you navigated to your domain to check that the gorilla/mux router setup serves requests to your Dockerized Go web app correctly.


# Conclusion


You have now successfully deployed your Go web app with Docker and Nginx on Ubuntu 22.04. With Docker, maintaining applications is not as time-consuming because the environment the app is executed in is guaranteed to be the same each time it’s run. The gorilla/mux package has excellent documentation and offers more sophisticated features, such as naming routes and serving static files. For more control over the Go HTTP server module, such as defining custom timeouts, visit the official docs.


