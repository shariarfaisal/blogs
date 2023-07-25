# How To Manage Your Kubernetes Configurations with Kustomize

```Docker``` ```Kubernetes``` ```Configuration Management``` ```Open Source```

The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


Deploying applications to Kubernetes can sometimes feel cumbersome. You deploy some Pods, backed by a Deployment, with accessibility defined in a Service. All of these resources require YAML files for proper definition and configuration.


On top of this, your application might need to communicate with a database, manage web content, or set logging verbosity. Further, these parameters may need to differ depending on the environment to which you are deploying. All of this can result in a sprawling codebase of YAML definitions, each with one- or two-line changes that are difficult to pinpoint.


Kustomize is an open-source configuration management tool developed to help address these concerns. Since Kubernetes 1.14, kubectl fully supports Kustomize and kustomization files.


In this guide, you will build a small web application and then use Kustomize to manage your configuration sprawl. You will deploy your app to development and production environments with different configurations. You will also layer these variable configurations using Kustomize’s bases and overlays so that your code is easier to read and thus easier to maintain.


If you’re looking for a managed Kubernetes hosting service, check out our simple, managed Kubernetes service built for growth.


# Prerequisites


For this tutorial, you will need:


- A Kubernetes 1.14+ cluster with your connection configuration set as the kubectl default. To create a Kubernetes cluster on DigitalOcean, read our Kubernetes Quickstart. To connect to the cluster, read How to Connect to a DigitalOcean Kubernetes Cluster.
- kubectl installed on your local machine. Follow this tutorial on getting started with Kubernetes: A kubectl Cheat Sheet to install it.

# Step 1 — Deploying Your Application without Kustomize


Before deploying your app with Kustomize, you will first deploy it more traditionally. In this case, you will deploy a development version of sammy-app—a static web application hosted on Nginx. You will store your web content as data in a ConfigMap, which you will mount on a Pod in a Deployment. Each of these will require a separate YAML file, which you will now create.


First, make a folder for your application and all of its configuration files. This is where you’ll run all of the commands in this tutorial.


Create a new folder in your home directory and navigate inside:


```
mkdir ~/sammy-app && cd ~/sammy-app


```


Now use your preferred text editor to create and open a file called configmap.yml:


```
nano configmap.yml


```


Add the following content:


~/sammy-app/configmap.yml
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: sammy-app
  namespace: default
data:
  body: >
    <html>
      <style>
        body {
          background-color: #222;
        }
        p {
          font-family:"Courier New";
          font-size:xx-large;
          color:#f22;
          text-align:center;
        }
      </style>
      <body>
        <p>DEVELOPMENT</p>
      </body>
    </html>

```


This specification creates a new ConfigMap object. You are naming it sammy-app and saving some HTML web content inside data:.


Save and close the file.


Now create and open a second file called deployment.yml:


```
nano deployment.yml


```


Add the following content:


~/sammy-app/deployment.yml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
  labels:
    app: sammy-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: sammy-app
  template:
    metadata:
      labels:
        app: sammy-app
    spec:
      containers:
      - name: server
        image: nginx:1.17
        volumeMounts:
          - name: sammy-app
            mountPath: /usr/share/nginx/html
        ports:
        - containerPort: 80
          protocol: TCP
        resources:
          requests:
            cpu: 100m
            memory: "128M"
          limits:
            cpu: 100m
            memory: "256M"
        env:
        - name: LOG_LEVEL
          value: "DEBUG"
      volumes:
      - name: sammy-app
        configMap:
          name: sammy-app
          items:
          - key: body
            path: index.html

```


This specification creates a new Deployment object. You are adding the name and label of sammy-app, setting the number of replicas to 1, and specifying the object to use the Nginx version 1.17 container image. You are also setting the container’s port to 80, defining cpu and memory requests and limitations, and setting your logging level to DEBUG.


Save and close the file.


Now deploy these two files to your Kubernetes cluster. To create multiple Objects from stdin, pipe the cat command to kubectl:


```
cat configmap.yml deployment.yml | kubectl apply -f -


```


Wait a few moments and then use kubectl to check the status of your application:


```
kubectl get pods -l app=sammy-app


```


You will eventually see one Pod with your application running and 1/1 containers in the READY column:


```
OutputNAME                         READY   STATUS    RESTARTS   AGE
sammy-app-56bbd86cc9-chs75   1/1     Running   0          8s

```


Your Pod is running and backed by a Deployment, but you still cannot access your application. First, you need to add a Service.


Create and open a third YAML file called service.yml:


```
nano service.yml


```


Add the following content:


~/sammy-app/service.yml
```
---
apiVersion: v1
kind: Service
metadata:
  name: sammy-app
  labels:
    app: sammy-app
spec:
  type: LoadBalancer
  ports:
  - name: sammy-app-http
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: sammy-app

```


This specification creates a new Service object called sammy-app. For most cloud providers, setting spec.type to LoadBalancer will provision a load balancer. DigitalOcean Managed Kubernetes (DOKS), for instance, will provision a DigitalOcean LoadBalancer to make your application available to the Internet. spec.ports will target TCP port 80 for any Pod with the sammy-app label.


Save and close the file.


Now deploy the Service to your Kubernetes cluster:


```
kubectl apply -f service.yml


```


Wait a few moments and then use kubectl to check the status of your application:


```
kubectl get services -w


```


Eventually, a public IP will appear for your Service under the EXTERNAL-IP column. A unique IP will appear in the place of your_external_ip:


```
OutputNAME         TYPE           CLUSTER-IP       EXTERNAL-IP         PORT(S)        AGE
kubernetes   ClusterIP      10.245.0.1       <none>              443/TCP        7h26m
sammy-app    LoadBalancer   10.245.186.235   <pending>           80:30303/TCP   65s
sammy-app    LoadBalancer   10.245.186.235   your_external_ip   80:30303/TCP   2m29s

```


Copy the IP address that appears and enter it in your web browser. You will see the DEVELOPMENT version of your application.





From your terminal, type CTRL + C to stop watching your Services.


In this step, you deployed a development version of sammy-app to Kubernetes. In Steps 2 and 3, you will use Kustomize to redeploy a development version of sammy-app and then deploy a production version with slightly different configurations. Using this new workflow, you will see how well Kustomize can manage configuration changes and simplify your development workflow.


# Step 2 — Deploying Your Application with Kustomize


In this step, you will deploy the exact same application, but in the form that Kustomize expects instead of the default Kubernetes manner.


Your filesystem currently looks like this:


```
sammy-app/
├── configmap.yml
├── deployment.yml
└── service.yml

```


To make this application deployable with Kustomize, you need to add one file, kustomization.yml. Do so now:


```
nano kustomization.yml


```


At a minimum, this file should specify what resources to manage when running kubectl with the -k option, which will direct kubectl to process the kustomization file.


Add the following content:


~/sammy-app/kustomization.yml
```
---
resources:
- configmap.yml
- deployment.yml
- service.yml

```


Save and close the file.


Now, before deploying again, delete your existing Kubernetes resources from Step 1:


```
kubectl delete deployment/sammy-app service/sammy-app configmap/sammy-app


```


And deploy them again, but this time with Kustomize:


```
kubectl apply -k .


```


Instead of providing the -f option to kubectl to direct Kubernetes to create resources from a file, you provide -k and a directory (in this case, . denotes the current directory). This instructs kubectl to use Kustomize and to inspect that directory’s kustomization.yml.


This creates all three resources: the ConfigMap, Deployment, and Service. Use the get pods command to check your deployment:


```
kubectl get pods -l app=sammy-app


```


You will again see one Pod with your application running and 1/1 containers in the READY column.


Now rerun the get services command. You will also see your Service with a publicly-accessible EXTERNAL-IP:


```
kubectl get services -l app=sammy-app


```


You are now successfully using Kustomize to manage your Kubernetes configurations. In the next step, you will deploy sammy-app to production with a slightly different configuration. You will also use Kustomize to manage these variances.


# Step 3 — Managing Application Variance with Kustomize


Configuration files for Kubernetes resources can really start to sprawl once you start dealing with multiple resource types, especially when there are small differences between environments (like development versus production, for example). You might have a deployment-development.yml and deployment-production.yml instead of just a deployment.yml. The situation might be similar for all of your other resources, too.


Imagine what might happen when a new version of the Nginx Docker image is released, and you want to start using it. Perhaps you test the new version in deployment-development.yml and want to proceed, but then you forget to update deployment-production.yml with the new version. Suddenly, you’re running a different version of Nginx in development than you are in production. Small configuration errors like this can quickly break your application.


Kustomize can greatly simplify these management issues. Remember that you now have a filesystem with your Kubernetes configuration files and a kustomization.yml:


```
sammy-app/
├── configmap.yml
├── deployment.yml
├── kustomization.yml
└── service.yml

```


Imagine that you are now ready to deploy sammy-app to production. You’ve also decided that the production version of your application will differ from its development version in the following ways:


- replicas will increase from 1 to 3.
- container resource requests will increase from 100m CPU and 128M memory to 250m CPU and 256M memory.
- container resource limits will increase from 100m CPU and 256M memory to 1 CPU and 1G memory.
- the LOG_LEVEL environment variable will change from DEBUG to INFO.
- ConfigMap data will change to display slightly different web content.

To begin, create some new directories to organize things in a more Kustomize-specific way:


```
mkdir base


```


This will hold your “default” configuration—your base. In your example, this is the development version of sammy-app.


Now move your current configuration in sammy-app/ into this directory:


```
mv configmap.yml deployment.yml service.yml kustomization.yml base/


```


Then make a new directory for your production configuration. Kustomize calls this an overlay. Think of overlays as layers on top of the base—they always require a base to function:


```
mkdir -p overlays/production


```


Create another kustomization.yml file to define your production overlay:


```
nano overlays/production/kustomization.yml


```


Add the following content:


~/sammy-app/overlays/production/kustomization.yml
```
---
bases:
- ../../base
patchesStrategicMerge:
- configmap.yml
- deployment.yml

```


This file will specify a base for the overlay and what strategy Kubernetes will use to patch the resources. In this example, you will specify a strategic-merge-style patch to update the ConfigMap and Deployment resources.


Save and close the file.


And finally, add new deployment.yml and configmap.yml files into the overlays/production/ directory.


Create the new deployment.yml file first:


```
nano overlays/production/deployment.yml


```


Add the following to your file. The highlighted sections denote changes from your development configuration:


~/sammy-app/overlays/production/deployment.yml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sammy-app
  namespace: default
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: server
        resources:
          requests:
            cpu: 250m
            memory: "256M"
          limits:
            cpu: 1
            memory: "1G"
        env:
        - name: LOG_LEVEL
          value: "INFO"

```


Notice the contents of this new deployment.yml. It contains only the TypeMeta fields used to identify the resource that changed (in this case, the Deployment of your application), and just enough remaining fields to step into the nested structure to specify a new field value, e.g., the container resource requests and limits.


Save and close the file.


Now create a new configmap.yml for your production overlay:


```
nano /overlays/production/configmap.yml

```


Add the following content:


~/sammy-app/overlays/production/configmap.yml
```
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: sammy-app
  namespace: default
data:
  body: >
    <html>
      <style>
        body {
          background-color: #222;
        }
        p {
          font-family:"Courier New";
          font-size:xx-large;
          color:#22f;
          text-align:center;
        }
      </style>
      <body>
        <p>PRODUCTION</p>
      </body>
    </html>

```


Here you have changed the text to display PRODUCTION instead of DEVELOPMENT. Note that you also changed the text color from a red hue #f22 to a blue hue #22f. Consider how difficult it could be to locate and track such minor changes if you were not using a configuration management tool like Kustomize.


Your directory structure now looks like this:


```
sammy-app/
├── base
│   ├── configmap.yml
│   ├── deployment.yml
│   ├── kustomization.yml
│   └── service.yml
└── overlays
    └── production
        ├── configmap.yml
        ├── deployment.yml
        └── kustomization.yml

```


You are ready to deploy using your base configuration. First, delete the existing resources:


```
kubectl delete deployment/sammy-app service/sammy-app configmap/sammy-app


```


Deploy your base configuration to Kubernetes:


```
kubectl apply -k base/


```


Inspect your deployment:


```
kubectl get pods,services -l app=sammy-app


```


You will see the expected base configuration, with the development version visible on the EXTERNAL-IP of the Service:


```
OutputNAME                             READY   STATUS    RESTARTS   AGE
pod/sammy-app-5668b6dc75-rwbtq   1/1     Running   0          21s

NAME                TYPE           CLUSTER-IP       EXTERNAL-IP            PORT(S)        AGE
service/sammy-app   LoadBalancer   10.245.110.172   your_external_ip   80:31764/TCP   7m43s

```


Now deploy your production configuration:


```
kubectl apply -k overlays/production/


```


Inspect your deployment again:


```
kubectl get pods,services -l app=sammy-app


```


You will see the expected production configuration, with the production version visible on the EXTERNAL-IP of the Service:


```
OutputNAME                             READY   STATUS    RESTARTS   AGE
pod/sammy-app-86759677b4-h5ndw   1/1     Running   0          15s
pod/sammy-app-86759677b4-t2dml   1/1     Running   0          17s
pod/sammy-app-86759677b4-z56f8   1/1     Running   0          13s

NAME                TYPE           CLUSTER-IP       EXTERNAL-IP            PORT(S)        AGE
service/sammy-app   LoadBalancer   10.245.110.172   your_external_ip   80:31764/TCP   8m59s

```


Notice in the production configuration that there are 3 Pods in total instead of 1. You can view the Deployment resource to confirm that the less-apparent changes have taken effect, too:


```
kubectl get deployments -l app=sammy-app -o yaml


```


Visit your_external_ip in a browser to view the production version of your site.





You are now using Kustomize to manage application variance. Thinking back to one of your original problems, if you now wanted to change the Nginx image version, you would only need to modify deployment.yml in the base, and your overlays that use that base will also receive that change through Kustomize. This greatly simplifies your development workflow, improves readability, and reduces the likelihood of errors.


# Conclusion


In this tutorial, you built a small web application and deployed it to Kubernetes. You then used Kustomize to simplify the management of your application’s configuration for different environments. You reorganized a set of nearly duplicate YAML files into a layered model. This will reduce errors, reduce manual configuration, and keep your work more recognizable and maintainable.


This, however, only scratches the surface of what Kustomize offers. There are dozens of official examples and plenty of in-depth technical documentation to explore if you are interested in learning more.


