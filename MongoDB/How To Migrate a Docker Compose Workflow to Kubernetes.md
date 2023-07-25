# How To Migrate a Docker Compose Workflow to Kubernetes

```Kubernetes``` ```Docker``` ```Node.js``` ```MongoDB``` ```Microservices```

## Introduction


When building modern, stateless applications, containerizing your application’s components is the first step in deploying and scaling on distributed platforms. If you have used Docker Compose in development, you will have modernized and containerized your application by:


- Extracting necessary configuration information from your code.
- Offloading your application’s state.
- Packaging your application for repeated use.

You will also have written service definitions that specify how your container images should run.


To run your services on a distributed platform like Kubernetes, you will need to translate your Compose service definitions to Kubernetes objects. This will allow you to scale your application with resiliency. One tool that can speed up the translation process to Kubernetes is kompose, a conversion tool that helps developers move Compose workflows to container orchestrators like Kubernetes or OpenShift.


In this tutorial, you will translate Compose services to Kubernetes objects using kompose. You will use the object definitions that kompose provides as a starting point and make adjustments to ensure that your setup will use Secrets, Services, and PersistentVolumeClaims in the way that Kubernetes expects. By the end of the tutorial, you will have a single-instance Node.js application with a MongoDB database running on a Kubernetes cluster. This setup will mirror the functionality of the code described in Containerizing a Node.js Application with Docker Compose and will be a good starting point to build out a production-ready solution that will scale with your needs.


If you’re looking for a managed Kubernetes hosting service, check out our simple, managed Kubernetes service built for growth.


# Prerequisites


- A Kubernetes 1.10+ cluster with role-based access control (RBAC) enabled. This setup will use a DigitalOcean Kubernetes cluster, but you are free to create a cluster using another method.
- The kubectl command-line tool installed on your local machine or development server and configured to connect to your cluster. You can read more about installing kubectl in the official documentation.
- Docker installed on your local machine or development server. If you are working with Ubuntu 18.04, follow Steps 1 and 2 of How To Install and Use Docker on Ubuntu 18.04; otherwise, follow the official documentation for information about installing on other operating systems. Be sure to add your non-root user to the docker group, as described in Step 2 of the linked tutorial.
- A Docker Hub account. For an overview of how to set this up, refer to this introduction to Docker Hub.

# Step 1 — Installing kompose


To begin using kompose, navigate to the project’s GitHub Releases page, and copy the link to the current release (version 1.18.0 as of this writing). Paste this link into the following curl command to download the latest version of kompose:


```
curl -L https://github.com/kubernetes/kompose/releases/download/v1.18.0/kompose-linux-amd64 -o kompose


```


For details about installing on non-Linux systems, please refer to the installation instructions.


Make the binary executable:


```
chmod +x kompose


```


Move it to your PATH:


```
sudo mv ./kompose /usr/local/bin/kompose


```


To verify that it has been installed properly, you can do a version check:


```
kompose version


```


If the installation was successful, you will see output like the following:


```
Output1.18.0 (06a2e56)

```


With kompose installed and ready to use, you can now clone the Node.js project code that you will be translating to Kubernetes.


# Step 2 — Cloning and Packaging the Application


To use our application with Kubernetes, we will need to clone the project code and package the application so that the kubelet service can pull the image.


Our first step will be to clone the node-mongo-docker-dev repository from the DigitalOcean Community GitHub account. This repository includes the code from the setup described in Containerizing a Node.js Application for Development With Docker Compose, which uses a demo Node.js application to demonstrate how to set up a development environment using Docker Compose. You can find more information about the application itself in the series From Containers to Kubernetes with Node.js.


Clone the repository into a directory called node_project:


```
git clone https://github.com/do-community/node-mongo-docker-dev.git node_project


```


Navigate to the node_project directory:


```
cd node_project


```


The node_project directory contains files and directories for a shark information application that works with user input. It has been modernized to work with containers: sensitive and specific configuration information has been removed from the application code and refactored to be injected at runtime, and the application’s state has been offloaded to a MongoDB database.


For more information about designing modern, stateless applications, please see Architecting Applications for Kubernetes and Modernizing Applications for Kubernetes.


The project directory includes a Dockerfile with instructions for building the application image. Let’s build the image now so that you can push it to your Docker Hub account and use it in your Kubernetes setup.


Using the docker build command, build the image with the -t flag, which allows you to tag it with a memorable name. In this case, tag the image with your Docker Hub username and name it node-kubernetes or a name of your own choosing:


```
docker build -t your_dockerhub_username/node-kubernetes .


```


The . in the command specifies that the build context is the current directory.


It will take a minute or two to build the image. Once it is complete, check your images:


```
docker images


```


You will see the following output:


```
OutputREPOSITORY                                TAG                 IMAGE ID            CREATED             SIZE
your_dockerhub_username/node-kubernetes   latest              9c6f897e1fbc        3 seconds ago       90MB
node                                      10-alpine           94f3c8956482        12 days ago         71MB

```


Next, log in to the Docker Hub account you created in the prerequisites:


```
docker login -u your_dockerhub_username 


```


When prompted, enter your Docker Hub account password. Logging in this way will create a ~/.docker/config.json file in your user’s home directory with your Docker Hub credentials.


Push the application image to Docker Hub with the docker push command. Remember to replace your_dockerhub_username with your own Docker Hub username:


```
docker push your_dockerhub_username/node-kubernetes


```


You now have an application image that you can pull to run your application with Kubernetes. The next step will be to translate your application service definitions to Kubernetes objects.


# Step 3 — Translating Compose Services to Kubernetes Objects with kompose


Our Docker Compose file, here called docker-compose.yaml, lays out the definitions that will run our services with Compose. A service in Compose is a running container, and service definitions contain information about how each container image will run. In this step, we will translate these definitions to Kubernetes objects by using kompose to create yaml files. These files will contain specs for the Kubernetes objects that describe their desired state.


We will use these files to create different types of objects: Services, which will ensure that the Pods running our containers remain accessible; Deployments, which will contain information about the desired state of our Pods; a PersistentVolumeClaim to provision storage for our database data; a ConfigMap for environment variables injected at runtime; and a Secret for our application’s database user and password. Some of these definitions will be in the files kompose will create for us, and others we will need to create ourselves.


First, we will need to modify some of the definitions in our docker-compose.yaml file to work with Kubernetes. We will include a reference to our newly-built application image in our nodejs service definition and remove the bind mounts, volumes, and additional commands that we used to run the application container in development with Compose. Additionally, we’ll redefine both containers’ restart policies to be in line with the behavior Kubernetes expects.


Open the file with nano or your favorite editor:


```
nano docker-compose.yaml


```


The current definition for the nodejs application service looks like this:


~/node_project/docker-compose.yaml
```
...
services:
  nodejs:
    build:
      context: .
      dockerfile: Dockerfile
    image: nodejs
    container_name: nodejs
    restart: unless-stopped
    env_file: .env
    environment:
      - MONGO_USERNAME=$MONGO_USERNAME
      - MONGO_PASSWORD=$MONGO_PASSWORD
      - MONGO_HOSTNAME=db
      - MONGO_PORT=$MONGO_PORT
      - MONGO_DB=$MONGO_DB 
    ports:
      - "80:8080"
    volumes:
      - .:/home/node/app
      - node_modules:/home/node/app/node_modules
    networks:
      - app-network
    command: ./wait-for.sh db:27017 -- /home/node/app/node_modules/.bin/nodemon app.js
...

```


Make the following edits to your service definition:


- Use your node-kubernetes image instead of the local Dockerfile.
- Change the container restart policy from unless-stopped to always.
- Remove the volumes list and the command instruction.

The finished service definition will now look like this:


~/node_project/docker-compose.yaml
```
...
services:
  nodejs:
    image: your_dockerhub_username/node-kubernetes
    container_name: nodejs
    restart: always
    env_file: .env
    environment:
      - MONGO_USERNAME=$MONGO_USERNAME
      - MONGO_PASSWORD=$MONGO_PASSWORD
      - MONGO_HOSTNAME=db
      - MONGO_PORT=$MONGO_PORT
      - MONGO_DB=$MONGO_DB 
    ports:
      - "80:8080"
    networks:
      - app-network
...

```


Next, scroll down to the db service definition. Here, make the following edits:


- Change the restart policy for the service to always.
- Remove the .env file. Instead of using values from the .env file, we will pass the values for our MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD to the database container using the Secret we will create in Step 4.

The db service definition will now look like this:


~/node_project/docker-compose.yaml
```
...
  db:
    image: mongo:4.1.8-xenial
    container_name: db
    restart: always
    environment:
      - MONGO_INITDB_ROOT_USERNAME=$MONGO_USERNAME
      - MONGO_INITDB_ROOT_PASSWORD=$MONGO_PASSWORD
    volumes:  
      - dbdata:/data/db   
    networks:
      - app-network
...  

```


Finally, at the bottom of the file, remove the node_modules volumes from the top-level volumes key. The key will now look like this:


~/node_project/docker-compose.yaml
```
...
volumes:
  dbdata:

```


Save and close the file when you are finished editing.


Before translating our service definitions, we will need to write the .env file that kompose will use to create the ConfigMap with our non-sensitive information. Please see Step 2 of Containerizing a Node.js Application for Development With Docker Compose for a longer explanation of this file.


In that tutorial, we added .env to our .gitignore file to ensure that it would not copy to version control. This means that it did not copy over when we cloned the node-mongo-docker-dev repository in Step 2 of this tutorial. We will therefore need to recreate it now.


Create the file:


```
nano .env


```


kompose will use this file to create a ConfigMap for our application. However, instead of assigning all of the variables from the nodejs service definition in our Compose file, we will add only the MONGO_DB database name and the MONGO_PORT. We will assign the database username and password separately when we manually create a Secret object in Step 4.


Add the following port and database name information to the .env file. Feel free to rename your database if you would like:


~/node_project/.env
```
MONGO_PORT=27017
MONGO_DB=sharkinfo

```


Save and close the file when you are finished editing.


You are now ready to create the files with your object specs. kompose offers multiple options for translating your resources. You can:


- Create yaml files based on the service definitions in your docker-compose.yaml file with kompose convert.
- Create Kubernetes objects directly with kompose up.
- Create a Helm chart with kompose convert -c.

For now, we will convert our service definitions to yaml files and then add to and revise the files kompose creates.


Convert your service definitions to yaml files with the following command:


```
kompose convert


```


You can also name specific or multiple Compose files using the -f flag.


After you run this command, kompose will output information about the files it has created:


```
OutputINFO Kubernetes file "nodejs-service.yaml" created 
INFO Kubernetes file "db-deployment.yaml" created 
INFO Kubernetes file "dbdata-persistentvolumeclaim.yaml" created 
INFO Kubernetes file "nodejs-deployment.yaml" created 
INFO Kubernetes file "nodejs-env-configmap.yaml" created 

```


These include yaml files with specs for the Node application Service, Deployment, and ConfigMap, as well as for the dbdata PersistentVolumeClaim and MongoDB database Deployment.


These files are a good starting point, but in order for our application’s functionality to match the setup described in Containerizing a Node.js Application for Development With Docker Compose we will need to make a few additions and changes to the files kompose has generated.


# Step 4 — Creating Kubernetes Secrets


In order for our application to function in the way we expect, we will need to make a few modifications to the files that kompose has created. The first of these changes will be generating a Secret for our database user and password and adding it to our application and database Deployments. Kubernetes offers two ways of working with environment variables: ConfigMaps and Secrets. kompose has already created a ConfigMap with the non-confidential information we included in our .env file, so we will now create a Secret with our confidential information: our database username and password.


The first step in manually creating a Secret will be to convert your username and password to base64, an encoding scheme that allows you to uniformly transmit data, including binary data.


Convert your database username:


```
echo -n 'your_database_username' | base64


```


Note down the value you see in the output.


Next, convert your password:


```
echo -n 'your_database_password' | base64


```


Take note of the value in the output here as well.


Open a file for the Secret:


```
nano secret.yaml


```



Note: Kubernetes objects are typically defined using YAML, which strictly forbids tabs and requires two spaces for indentation. If you would like to check the formatting of any of your yaml files, you can use a linter or test the validity of your syntax using kubectl create with the --dry-run and --validate flags:
kubectl create -f your_yaml_file.yaml --dry-run --validate=true


In general, it is a good idea to validate your syntax before creating resources with kubectl.

Add the following code to the file to create a Secret that will define your MONGO_USERNAME and MONGO_PASSWORD using the encoded values you just created. Be sure to replace the dummy values here with your encoded username and password:


~/node_project/secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: mongo-secret
data:
  MONGO_USERNAME: your_encoded_username
  MONGO_PASSWORD: your_encoded_password

```


We have named the Secret object mongo-secret, but you are free to name it anything you would like.


Save and close this file when you are finished editing. As you did with your .env file, be sure to add secret.yaml to your .gitignore file to keep it out of version control.


With secret.yaml written, our next step will be to ensure that our application and database Pods both use the values we added to the file. Let’s start by adding references to the Secret to our application Deployment.


Open the file called nodejs-deployment.yaml:


```
nano nodejs-deployment.yaml


```


The file’s container specifications include the following environment variables defined under the env key:


~/node_project/nodejs-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
...
    spec:
      containers:
      - env:
        - name: MONGO_DB
          valueFrom:
            configMapKeyRef:
              key: MONGO_DB
              name: nodejs-env
        - name: MONGO_HOSTNAME
          value: db
        - name: MONGO_PASSWORD
        - name: MONGO_PORT
          valueFrom:
            configMapKeyRef:
              key: MONGO_PORT
              name: nodejs-env
        - name: MONGO_USERNAME

```


We will need to add references to our Secret to the MONGO_USERNAME and MONGO_PASSWORD variables listed here, so that our application will have access to those values. Instead of including a configMapKeyRef key to point to our nodejs-env ConfigMap, as is the case with the values for MONGO_DB and MONGO_PORT, we’ll include a secretKeyRef key to point to the values in our mongo-secret secret.


Add the following Secret references to the MONGO_USERNAME and MONGO_PASSWORD variables:


~/node_project/nodejs-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
...
    spec:
      containers:
      - env:
        - name: MONGO_DB
          valueFrom:
            configMapKeyRef:
              key: MONGO_DB
              name: nodejs-env
        - name: MONGO_HOSTNAME
          value: db
        - name: MONGO_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_PASSWORD
        - name: MONGO_PORT
          valueFrom:
            configMapKeyRef:
              key: MONGO_PORT
              name: nodejs-env
        - name: MONGO_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_USERNAME

```


Save and close the file when you are finished editing.


Next, we’ll add the same values to the db-deployment.yaml file.


Open the file for editing:


```
nano db-deployment.yaml


```


In this file, we will add references to our Secret for following variable keys: MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD. The mongo image makes these variables available so that you can modify the initialization of your database instance. MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD together create a root user in the admin authentication database and ensure that authentication is enabled when the database container starts.


Using the values we set in our Secret ensures that we will have an application user with root privileges on the database instance, with access to all of the administrative and operational privileges of that role. When working in production, you will want to create a dedicated application user with appropriately scoped privileges.


Under the MONGO_INITDB_ROOT_USERNAME and MONGO_INITDB_ROOT_PASSWORD variables, add references to the Secret values:


~/node_project/db-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
...
    spec:
      containers:
      - env:
        - name: MONGO_INITDB_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_PASSWORD        
        - name: MONGO_INITDB_ROOT_USERNAME
          valueFrom:
            secretKeyRef:
              name: mongo-secret
              key: MONGO_USERNAME
        image: mongo:4.1.8-xenial
...

```


Save and close the file when you are finished editing.


With your Secret in place, you can move on to creating your database Service and ensuring that your application container only attempts to connect to the database once it is fully set up and initialized.


# Step 5 — Creating the Database Service and an Application Init Container


Now that we have our Secret, we can move on to creating our database Service and an Init Container that will poll this Service to ensure that our application only attempts to connect to the database once the database startup tasks, including creating the MONGO_INITDB user and password, are complete.


For a discussion of how to implement this functionality in Compose, please see Step 4 of Containerizing a Node.js Application for Development with Docker Compose.


Open a file to define the specs for the database Service:


```
nano db-service.yaml  


```


Add the following code to the file to define the Service:


~/node_project/db-service.yaml
```
apiVersion: v1
kind: Service
metadata:
  annotations: 
    kompose.cmd: kompose convert
    kompose.version: 1.18.0 (06a2e56)
  creationTimestamp: null
  labels:
    io.kompose.service: db
  name: db
spec:
  ports:
  - port: 27017
    targetPort: 27017
  selector:
    io.kompose.service: db
status:
  loadBalancer: {}

```


The selector that we have included here will match this Service object with our database Pods, which have been defined with the label io.kompose.service: db by kompose in the db-deployment.yaml file. We’ve also named this service db.


Save and close the file when you are finished editing.


Next, let’s add an Init Container field to the containers array in nodejs-deployment.yaml. This will create an Init Container that we can use to delay our application container from starting until the db Service has been created with a Pod that is reachable. This is one of the possible uses for Init Containers; to learn more about other use cases, please see the official documentation.


Open the nodejs-deployment.yaml file:


```
nano nodejs-deployment.yaml


```


Within the Pod spec and alongside the containers array, we are going to add an initContainers field with a container that will poll the db Service.


Add the following code below the ports and resources fields and above the restartPolicy in the nodejs containers array:


~/node_project/nodejs-deployment.yaml
```
apiVersion: extensions/v1beta1
kind: Deployment
...
    spec:
      containers:
      ...
        name: nodejs
        ports:
        - containerPort: 8080
        resources: {}
      initContainers:
      - name: init-db
        image: busybox
        command: ['sh', '-c', 'until nc -z db:27017; do echo waiting for db; sleep 2; done;']
      restartPolicy: Always
...               

```


This Init Container uses the BusyBox image, a lightweight image that includes many UNIX utilities. In this case, we’ll use the netcat utility to poll whether or not the Pod associated with the db Service is accepting TCP connections on port 27017.


This container command replicates the functionality of the wait-for script that we removed from our docker-compose.yaml file in Step 3. For a longer discussion of how and why our application used the wait-for script when working with Compose, please see Step 4 of Containerizing a Node.js Application for Development with Docker Compose.


Init Containers run to completion; in our case, this means that our Node application container will not start until the database container is running and accepting connections on port 27017. The db Service definition allows us to guarantee this functionality regardless of the exact location of the database container, which is mutable.


Save and close the file when you are finished editing.


With your database Service created and your Init Container in place to control the startup order of your containers, you can move on to checking the storage requirements in your PersistentVolumeClaim and exposing your application service using a LoadBalancer.


# Step 6 — Modifying the PersistentVolumeClaim and Exposing the Application Frontend


Before running our application, we will make two final changes to ensure that our database storage will be provisioned properly and that we can expose our application frontend using a LoadBalancer.


First, let’s modify the storage resource defined in the PersistentVolumeClaim that kompose created for us. This Claim allows us to dynamically provision storage to manage our application’s state.


To work with PersistentVolumeClaims, you must have a StorageClass created and configured to provision storage resources. In our case, because we are working with DigitalOcean Kubernetes, our default StorageClass provisioner is set to dobs.csi.digitalocean.com  — DigitalOcean Block Storage.


We can check this by typing:


```
kubectl get storageclass


```


If you are working with a DigitalOcean cluster, you will see the following output:


```
OutputNAME                         PROVISIONER                 AGE
do-block-storage (default)   dobs.csi.digitalocean.com   76m

```


If you are not working with a DigitalOcean cluster, you will need to create a StorageClass and configure a provisioner of your choice. For details about how to do this, please see the official documentation.


When kompose created dbdata-persistentvolumeclaim.yaml, it set the storage resource to a size that does not meet the minimum size requirements of our provisioner. We will therefore need to modify our PersistentVolumeClaim to use the minimum viable DigitalOcean Block Storage unit: 1GB. Please feel free to modify this to meet your storage requirements.


Open dbdata-persistentvolumeclaim.yaml:


```
nano dbdata-persistentvolumeclaim.yaml


```


Replace the storage value with 1Gi:


~/node_project/dbdata-persistentvolumeclaim.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  creationTimestamp: null
  labels:
    io.kompose.service: dbdata
  name: dbdata
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
status: {}

```


Also note the accessMode: ReadWriteOnce means that the volume provisioned as a result of this Claim will be read-write only by a single node. Please see the documentation for more information about different access modes.


Save and close the file when you are finished.


Next, open nodejs-service.yaml:


```
nano nodejs-service.yaml


```


We are going to expose this Service externally using a DigitalOcean Load Balancer. If you are not using a DigitalOcean cluster, please consult the relevant documentation from your cloud provider for information about their load balancers. Alternatively, you can follow the official Kubernetes documentation on setting up a highly available cluster with kubeadm, but in this case you will not be able to use PersistentVolumeClaims to provision storage.


Within the Service spec, specify LoadBalancer as the Service type:


~/node_project/nodejs-service.yaml
```
apiVersion: v1
kind: Service
...
spec:
  type: LoadBalancer
  ports:
...

```


When we create the nodejs Service, a load balancer will be automatically created, providing us with an external IP where we can access our application.


Save and close the file when you are finished editing.


With all of our files in place, we are ready to start and test the application.


# Step 7 — Starting and Accessing the Application


It’s time to create our Kubernetes objects and test that our application is working as expected.


To create the objects we’ve defined, we’ll use kubectl create with the -f flag, which will allow us to specify the files that kompose created for us, along with the files we wrote. Run the following command to create the Node application and MongoDB database Services and Deployments, along with your Secret, ConfigMap, and PersistentVolumeClaim:


```
kubectl create -f nodejs-service.yaml,nodejs-deployment.yaml,nodejs-env-configmap.yaml,db-service.yaml,db-deployment.yaml,dbdata-persistentvolumeclaim.yaml,secret.yaml


```


You will see the following output indicating that the objects have been created:


```
Outputservice/nodejs created
deployment.extensions/nodejs created
configmap/nodejs-env created
service/db created
deployment.extensions/db created
persistentvolumeclaim/dbdata created
secret/mongo-secret created

```


To check that your Pods are running, type:


```
kubectl get pods


```


You don’t need to specify a Namespace here, since we have created our objects in the default Namespace. If you are working with multiple Namespaces, be sure to include the -n flag when running this command, along with the name of your Namespace.


You will see the following output while your db container is starting and your application Init Container is running:


```
OutputNAME                      READY   STATUS              RESTARTS   AGE
db-679d658576-kfpsl       0/1     ContainerCreating   0          10s
nodejs-6b9585dc8b-pnsws   0/1     Init:0/1            0          10s

```


Once that container has run and your application and database containers have started, you will see this output:


```
OutputNAME                      READY   STATUS    RESTARTS   AGE
db-679d658576-kfpsl       1/1     Running   0          54s
nodejs-6b9585dc8b-pnsws   1/1     Running   0          54s

```


The Running STATUS indicates that your Pods are bound to nodes and that the containers associated with those Pods are running. READY indicates how many containers in a Pod are running. For more information, please consult the documentation on Pod lifecycles.



Note:
If you see unexpected phases in the STATUS column, remember that you can troubleshoot your Pods with the following commands:
kubectl describe pods your_pod
kubectl logs your_pod



With your containers running, you can now access the application. To get the IP for the LoadBalancer, type:


```
kubectl get svc


```


You will see the following output:


```
OutputNAME         TYPE           CLUSTER-IP       EXTERNAL-IP      PORT(S)        AGE
db           ClusterIP      10.245.189.250   <none>           27017/TCP      93s
kubernetes   ClusterIP      10.245.0.1       <none>           443/TCP        25m12s
nodejs       LoadBalancer   10.245.15.56     your_lb_ip       80:30729/TCP   93s

```


The EXTERNAL_IP associated with the nodejs service is the IP address where you can access the application. If you see a <pending> status in the EXTERNAL_IP column, this means that your load balancer is still being created.


Once you see an IP in that column, navigate to it in your browser: http://your_lb_ip.


You should see the following landing page:





Click on the Get Shark Info button. You will see a page with an entry form where you can enter a shark name and a description of that shark’s general character:





In the form, add a shark of your choosing. To demonstrate, we will add Megalodon Shark to the Shark Name field, and Ancient to the Shark Character field:





Click on the Submit button. You will see a page with this shark information displayed back to you:





You now have a single instance setup of a Node.js application with a MongoDB database running on a Kubernetes cluster.


# Conclusion


The files you have created in this tutorial are a good starting point to build from as you move toward production. As you develop your application, you can work on implementing the following:


- Centralized logging and monitoring. Please see the relevant discussion in Modernizing Applications for Kubernetes for a general overview. You can also look at How To Set Up an Elasticsearch, Fluentd and Kibana (EFK) Logging Stack on Kubernetes to learn how to set up a logging stack with Elasticsearch, Fluentd, and Kibana. Also check out An Introduction to Service Meshes for information about how service meshes like Istio implement this functionality.
- Ingress Resources to route traffic to your cluster. This is a good alternative to a LoadBalancer in cases where you are running multiple Services, which each require their own LoadBalancer, or where you would like to implement application-level routing strategies (A/B & canary tests, for example). For more information, check out How to Set Up an Nginx Ingress with Cert-Manager on DigitalOcean Kubernetes and the related discussion of routing in the service mesh context in An Introduction to Service Meshes.
- Backup strategies for your Kubernetes objects. For guidance on implementing backups with Velero (formerly Heptio Ark) with DigitalOcean’s Kubernetes product, please see How To Back Up and Restore a Kubernetes Cluster on DigitalOcean Using Heptio Ark.

