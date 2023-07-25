# How To Set Up the code-server Cloud IDE Platform on DigitalOcean Kubernetes

```Kubernetes``` ```Nginx``` ```Let's Encrypt``` ```VS Code```

The author selected the Free and Open Source Fund to receive a donation as part of the Write for DOnations program.


## Introduction


With developer tools moving to the cloud, the creation and adoption of cloud IDE (Integrated Development Environment) platforms is growing. Cloud IDEs allow for real-time collaboration between developer teams to work in a unified development environment that minimizes incompatibilities and enhances productivity. Accessible through web browsers, cloud IDEs are available from every type of modern device. Another advantage of a cloud IDE is the possibility to leverage the power of a cluster, which can greatly exceed the processing power of a single development computer.


code-server is Microsoft Visual Studio Code running on a remote server and accessible directly from your browser. Visual Studio Code is a modern code editor with integrated Git support, a code debugger, smart autocompletion, and customizable and extensible features. This means that you can use various devices, running different operating systems, and always have a consistent development environment on hand.


In this tutorial, you will set up the code-server cloud IDE platform on your DigitalOcean Kubernetes cluster and expose it at your domain, secured with Let’s Encrypt certificates. In the end, you’ll have Microsoft Visual Studio Code running on your Kubernetes cluster, available via HTTPS, and protected by a password.


# Prerequisites


- A DigitalOcean Kubernetes cluster with your connection configured as the kubectl default. Instructions on how to configure kubectl are shown under the Connect to your Cluster step when you create your cluster. To create a Kubernetes cluster on DigitalOcean, see Kubernetes Quickstart.
- The Helm 3 package manager installed on your local machine. Complete Step 1 of the How To Install Software on Kubernetes Clusters with the Helm 3 Package Manager tutorial.
- The Nginx Ingress Controller and Cert-Manager installed on your cluster using Helm in order to expose code-server using Ingress Resources. To do this, follow How to Set Up an Nginx Ingress on DigitalOcean Kubernetes Using Helm.
- A fully registered domain name to host code-server, pointed at the Load Balancer used by the Nginx Ingress. This tutorial will use code-server.your_domain throughout. You can purchase a domain name on Namecheap, get one for free on Freenom, or use the domain registrar of your choice. This domain name must differ from the one used in the How To Set Up an Nginx Ingress on DigitalOcean Kubernetes prerequisite tutorial.

# Step 1 — Installing and Exposing code-server


In this section, you’ll install code-server to your DigitalOcean Kubernetes cluster and expose it at your domain using the Nginx Ingress controller. You will also set up a password for admittance.


As part of the Nginx Ingress Controller prerequisite, you created example Services and an Ingress. You won’t need them in this tutorial, so you can delete them by running the following commands:


```
kubectl delete -f hello-kubernetes-first.yaml
kubectl delete -f hello-kubernetes-second.yaml
kubectl delete -f hello-kubernetes-ingress.yaml


```


The kubectl delete command accepts the file to delete when passed the -f parameter.


You’ll store the deployment configuration on your local machine, in a file named code-server.yaml. Create it using the following command:


```
nano code-server.yaml


```


Add the following lines to the file:


code-server.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: code-server
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: code-server
  namespace: code-server
spec:
  ingressClassName: nginx
  rules:
  - host: code-server.your_domain
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: code-server
            port:
              number: 80
---
apiVersion: v1
kind: Service
metadata:
 name: code-server
 namespace: code-server
spec:
 ports:
 - port: 80
   targetPort: 8080
 selector:
   app: code-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: code-server
  name: code-server
  namespace: code-server
spec:
  selector:
    matchLabels:
      app: code-server
  replicas: 1
  template:
    metadata:
      labels:
        app: code-server
    spec:
      containers:
      - image: codercom/code-server:latest
        imagePullPolicy: Always
        name: code-server
        env:
        - name: PASSWORD
          value: "your_password"

```


This configuration defines a Namespace, a Deployment, a Service, and an Ingress. The Namespace is called code-server and separates the code-server installation from the rest of your cluster. The Deployment consists of one replica of the codercom/code-server Docker image, and an environment variable named PASSWORD that specifies the password for access.


The code-server Service internally exposes the pod (created as a part of the Deployment) at port 80. The Ingress defined in the file specifies that the Ingress Controller is nginx, and that the code-server.your_domain domain will be served from the Service.


Remember to replace your_password with your desired password, and code-server.your_domain with your desired domain, pointed to the Load Balancer of the Nginx Ingress Controller.


Save and close the file.


Then, create the configuration in Kubernetes by running the following command:


```
kubectl apply -f code-server.yaml


```


You’ll see the following output:


```
Outputnamespace/code-server created
ingress.networking.k8s.io/code-server created
service/code-server created
deployment.apps/code-server created

```


You can watch the code-server pod become available by running:


```
kubectl get pods -w -n code-server


```


The output will look similar to this:


```
OutputNAME                           READY   STATUS              RESTARTS   AGE
code-server-6c4745497c-l2n7w   0/1     ContainerCreating   0          12s

```


As soon as the status becomes Running, code-server has finished installing to your cluster.


You can now Navigate to your domain in your browser. You’ll see the login prompt for code-server.





code-server is asking you for your password. Enter the one you set in the previous step and press Enter IDE. You’ll now enter code-server and immediately see its editor GUI.





You’ve installed code-server to your Kubernetes cluster and made it available at your domain. You have also verified that it requires you to log in with a password. Now, you’ll move on to secure it with free Let’s Encrypt certificates using Cert-Manager.


# Step 2 — Securing the code-server Deployment


In this section, you will secure your code-server installation by applying Let’s Encrypt certificates to your Ingress, which Cert-Manager will automatically create. After completing this step, your code-server installation will be accessible via HTTPS.


Open code-server.yaml for editing:


```
nano code-server.yaml


```


Add the highlighted lines to your file, making sure to replace the example domain with your own:


code-server.yaml
```
apiVersion: v1
kind: Namespace
metadata:
  name: code-server
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: code-server
  namespace: code-server
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - code-server.your_domain
    secretName: codeserver-prod
  rules:
    - host: code-server.your_domain
    http:
      paths:
      - backend:
          service:
            name: code-server
            port:
              number: 80
...

```


First, you specify that this Ingress will use the cluster-issuer letsencrypt-prod to provision certificates, which you created as a part of the prerequisites. Then, you specify the domains that will be secured under the tls section, as well as your name for the Secret holding them.


Save and close the file.


Apply the changes to your Kubernetes cluster by running the following command:


```
kubectl apply -f code-server.yaml


```


You’ll need to wait a few minutes for Let’s Encrypt to provision your certificate. In the meantime, you can track its progress by looking at the output of the following command:


```
kubectl describe certificate codeserver-prod -n code-server


```


When it finishes, the end of the output will look similar to this:


```
OutputEvents:
  Type    Reason     Age   From          Message
  ----    ------     ----  ----          -------
  Normal  Issuing    44m   cert-manager  Issuing certificate as Secret does not exist
  Normal  Generated  44m   cert-manager  Stored new private key in temporary Secret resource "codeserver-prod-m7r8v"
  Normal  Requested  44m   cert-manager  Created new CertificateRequest resource "codeserver-prod-sc7xm"
  Normal  Issuing    44m   cert-manager  The certificate has been successfully issued

```


You can now refresh your domain in your browser. You’ll see the padlock to the left of the address bar in your browser signifying that the connection is secure.


In this step, you have configured the Ingress to secure your code-server deployment. Now, you can review the code-server user interface.


# Step 3 — Exploring the code-server Interface


In this section, you’ll explore some of the features of the code-server interface. Since code-server is Visual Studio Code running in the cloud, it has the same interface as the standalone desktop edition.


On the left-hand side of the IDE, there is a vertical row of six buttons opening the most commonly used features in a side panel known as the Activity Bar.





This bar is customizable; you can reorder these views or remove them from the bar. By default, the first view opens the Explorer panel that provides tree-like navigation of the project’s structure. You can manage your folders and files here—creating, deleting, moving, and renaming them as necessary. The next view provides access to a search and replace functionality.


Following this, in the default order, is your view of the source control systems, like Git. Visual Studio Code also supports other source control providers, and you can find further instructions for source control workflows with the editor in this documentation. Because the currently empty project (that is opened by default) is not initialized as a Git repository, Visual Studio Code offers you to initialize it as such.





The debugger option on the Activity Bar provides all the common actions for debugging in the panel. Visual Studio Code comes with built-in support for the Node.js runtime debugger and any language that transpiles to Javascript. For other languages you can install extensions for the required debugger. You can save debugging configurations in the launch.json file.





The final view in the Activity Bar provides a menu to access available extensions on the Marketplace.





The central part of the GUI is your editor, which you can separate by tabs for your code editing. You can change your editing view to a grid system or to side-by-side files.





After creating a new file through the File menu, an empty file will open in a new tab. Once saved, the file’s name will be viewable in the Explorer side panel. Creating folders can be done by right clicking on the Explorer sidebar and selecting New Folder. You can expand a folder by clicking on its name as well as dragging and dropping files and folders to upper parts of the hierarchy to move them to a new location.





You can access a terminal by pressing CTRL+SHIFT+`, or by pressing on Terminal in the hamburger menu, and selecting New Terminal. The terminal will open in a lower panel and its working directory will be set to the project’s workspace, which contains the files and folders shown in the Explorer side panel.


If you wish to destroy the deployment on your cluster, run the following command:


```
kubectl delete -f code-server.yaml


```


You’ve explored a high-level overview of the code-server interface and reviewed some of the most commonly used features.


# Conclusion


You now have code-server, a versatile cloud IDE, installed on your DigitalOcean Kubernetes cluster. You can work on your source code and documents with it individually or collaborate with your team. Running a cloud IDE on your cluster provides more power for testing, downloading, and more thorough or rigorous computing. For further information see the Visual Studio Code documentation on additional features and detailed instructions on other components of code-server.


