# How to Set Up an Nginx Ingress with Cert-Manager on DigitalOcean Kubernetes

```Kubernetes``` ```Security``` ```Solutions``` ```Nginx``` ```Let's Encrypt```

## Introduction


Kubernetes Ingresses allow you to flexibly route traffic from outside your Kubernetes cluster to Services inside of your cluster. This is accomplished using Ingress Resources, which define rules for routing HTTP and HTTPS traffic to Kubernetes Services, and Ingress Controllers, which implement the rules by load balancing traffic and routing it to the appropriate backend Services.


Popular Ingress Controllers include Nginx, Contour, HAProxy, and Traefik. Ingresses provide a more efficient and flexible alternative to setting up multiple LoadBalancer services, each of which uses its own dedicated Load Balancer.


In this guide, we’ll set up the Kubernetes-maintained Nginx Ingress Controller, and create some Ingress Resources to route traffic to several dummy backend services. Once we’ve set up the Ingress, we’ll install cert-manager into our cluster to manage and provision TLS certificates for encrypting HTTP traffic to the Ingress. This guide does not use the Helm package manager. For a guide on rolling out the Nginx Ingress Controller using Helm, consult How To Set Up an Nginx Ingress on DigitalOcean Kubernetes Using Helm.


If you’re looking for a managed Kubernetes hosting service, check out our simple, managed Kubernetes service built for growth.


# Prerequisites


Before you begin with this guide, you should have the following available to you:


- A Kubernetes 1.15+ cluster with role-based access control (RBAC) enabled. This setup will use a DigitalOcean Kubernetes cluster, but you are free to create a cluster using another method.
- The kubectl command-line tool installed on your local machine and configured to connect to your cluster. You can read more about installing kubectl in the official documentation. If you are using a DigitalOcean Kubernetes cluster, please refer to How to Connect to a DigitalOcean Kubernetes Cluster to learn how to connect to your cluster using kubectl.
- A domain name and DNS A records which you can point to the DigitalOcean Load Balancer used by the Ingress. If you are using DigitalOcean to manage your domain’s DNS records, consult How to Manage DNS Records to learn how to create A records.
- The wget command-line utility installed on your local machine. You can install wget using the package manager built into your operating system.

Once you have these components set up, you’re ready to begin with this guide.


# Step 1 — Setting Up Dummy Backend Services


Before we deploy the Ingress Controller, we’ll first create and roll out two dummy echo Services to which we’ll route external traffic using the Ingress. The echo Services will run the hashicorp/http-echo web server container, which returns a page containing a text string passed in when the web server is launched. To learn more about http-echo, consult its GitHub Repo, and to learn more about Kubernetes Services, consult Services from the official Kubernetes docs.


On your local machine, create and edit a file called echo1.yaml using nano or your favorite editor:


```
nano echo1.yaml


```


Paste in the following Service and Deployment manifest:


echo1.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: echo1
spec:
  ports:
  - port: 80
    targetPort: 5678
  selector:
    app: echo1
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo1
spec:
  selector:
    matchLabels:
      app: echo1
  replicas: 2
  template:
    metadata:
      labels:
        app: echo1
    spec:
      containers:
      - name: echo1
        image: hashicorp/http-echo
        args:
        - "-text=echo1"
        ports:
        - containerPort: 5678

```


In this file, we define a Service called echo1 which routes traffic to Pods with the app: echo1 label selector. It accepts TCP traffic on port 80 and routes it to port 5678, http-echo’s default port.


We then define a Deployment, also called echo1, which manages Pods with the app: echo1 Label Selector. We specify that the Deployment should have 2 Pod replicas, and that the Pods should start a container called echo1 running  the  hashicorp/http-echo image. We pass in the text parameter and set it to echo1, so that the http-echo web server returns echo1. Finally, we open port 5678 on the Pod container.


Once you’re satisfied with your dummy Service and Deployment manifest, save and close the file.


Then, create the Kubernetes resources using kubectl apply with the -f flag, specifying the file you just saved as a parameter:


```
kubectl apply -f echo1.yaml


```


You should see the following output:


```
Outputservice/echo1 created
deployment.apps/echo1 created

```


Verify that the Service started correctly by confirming that it has a ClusterIP, the internal IP on which the Service is exposed:


```
kubectl get svc echo1


```


You should see the following output:


```
OutputNAME      TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
echo1     ClusterIP   10.245.222.129   <none>        80/TCP    60s

```


This indicates that the echo1 Service is now available internally at 10.245.222.129 on port 80. It will forward traffic to containerPort 5678 on the Pods it selects.


Now that the echo1 Service is up and running, repeat this process for the echo2 Service.


Create and open a file called echo2.yaml:


echo2.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: echo2
spec:
  ports:
  - port: 80
    targetPort: 5678
  selector:
    app: echo2
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: echo2
spec:
  selector:
    matchLabels:
      app: echo2
  replicas: 1
  template:
    metadata:
      labels:
        app: echo2
    spec:
      containers:
      - name: echo2
        image: hashicorp/http-echo
        args:
        - "-text=echo2"
        ports:
        - containerPort: 5678

```


Here, we essentially use the same Service and Deployment manifest as above, but name and relabel the Service and Deployment echo2. In addition, to provide some variety, we create only 1 Pod replica. We ensure that we set the text parameter to echo2 so that the web server returns the text echo2.


Save and close the file, and create the Kubernetes resources using kubectl:


```
kubectl apply -f echo2.yaml


```


You should see the following output:


```
Outputservice/echo2 created
deployment.apps/echo2 created

```


Once again, verify that the Service is up and running:


```
kubectl get svc


```


You should see both the echo1 and echo2 Services with assigned ClusterIPs:


```
OutputNAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
echo1        ClusterIP   10.245.222.129   <none>        80/TCP    6m6s
echo2        ClusterIP   10.245.128.224   <none>        80/TCP    6m3s
kubernetes   ClusterIP   10.245.0.1       <none>        443/TCP   4d21h

```


Now that our dummy echo web services are up and running, we can move on to rolling out the Nginx Ingress Controller.


# Step 2 — Setting Up the Kubernetes Nginx Ingress Controller


In this step, we’ll roll out v1.1.1 of the Kubernetes-maintained Nginx Ingress Controller. Note that there are several Nginx Ingress Controllers; the Kubernetes community maintains the one used in this guide and Nginx Inc. maintains kubernetes-ingress. The instructions in this tutorial are based on those from the official Kubernetes Nginx Ingress Controller Installation Guide.


The Nginx Ingress Controller consists of a Pod that runs the Nginx web server and watches the Kubernetes Control Plane for new and updated Ingress Resource objects. An Ingress Resource is essentially a list of traffic routing rules for backend Services. For example, an Ingress rule can specify that HTTP traffic arriving at the path /web1 should be directed towards the web1 backend web server. Using Ingress Resources, you can also perform host-based routing: for example, routing requests that hit web1.your_domain.com to the backend Kubernetes Service web1.


In this case, because we’re deploying the Ingress Controller to a DigitalOcean Kubernetes cluster, the Controller will create a LoadBalancer Service that provisions a DigitalOcean Load Balancer to which all external traffic will be directed. This Load Balancer will route external traffic to the Ingress Controller Pod running Nginx, which then forwards traffic to the appropriate backend Services.


We’ll begin by creating the Nginx Ingress Controller Kubernetes resources. These consist of ConfigMaps containing the Controller’s configuration, Role-based Access Control (RBAC) Roles to grant the Controller access to the Kubernetes API, and the actual Ingress Controller Deployment which uses v1.1.1 of the Nginx Ingress Controller image. To see a full list of these required resources, consult the manifest from the Kubernetes Nginx Ingress Controller’s GitHub repo.



Note: In this tutorial, we’re following the official installation instructions for the DigitalOcean Provider. You should choose the appropriate manifest file depending on your Kubernetes provider.

To create the resources, use kubectl apply and the -f flag to specify the manifest file hosted on GitHub:


```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/do/deploy.yaml


```


We use apply here so that in the future we can incrementally apply changes to the Ingress Controller objects instead of completely overwriting them. To learn more about apply, consult Managing Resources from the official Kubernetes docs.


You should see the following output:


```
Outputnamespace/ingress-nginx created
serviceaccount/ingress-nginx created
configmap/ingress-nginx-controller created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
service/ingress-nginx-controller-admission created
service/ingress-nginx-controller created
deployment.apps/ingress-nginx-controller created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
serviceaccount/ingress-nginx-admission created

```


This output also serves as a convenient summary of all the Ingress Controller objects created from the deploy.yaml manifest.


Confirm that the Ingress Controller Pods have started:


```
kubectl get pods -n ingress-nginx \
  -l app.kubernetes.io/name=ingress-nginx --watch


```


```
OutputNAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-l2jhk       0/1     Completed   0          13m
ingress-nginx-admission-patch-hsrzf        0/1     Completed   0          13m
ingress-nginx-controller-c96557986-m47rq   1/1     Running     0          13m

```


Hit CTRL+C to return to your prompt.


Now, confirm that the DigitalOcean Load Balancer was successfully created by fetching the Service details with kubectl:


```
kubectl get svc --namespace=ingress-nginx


```


After several minutes, you should see an external IP address, corresponding to the IP address of the DigitalOcean Load Balancer:


```
OutputNAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP       PORT(S)                      AGE
ingress-nginx-controller             LoadBalancer   10.245.201.120   203.0.113.0   80:31818/TCP,443:31146/TCP   14m
ingress-nginx-controller-admission   ClusterIP      10.245.239.119   <none>            443/TCP                      14m

```


Note down the Load Balancer’s external IP address, as you’ll need it in a later step.



Note: By default the Nginx Ingress LoadBalancer Service has service.spec.externalTrafficPolicy set to the value Local, which routes all load balancer traffic to nodes running Nginx Ingress Pods. The other nodes will deliberately fail load balancer health checks so that Ingress traffic does not get routed to them. External traffic policies are beyond the scope of this tutorial, but to learn more you can consult A Deep Dive into Kubernetes External Traffic Policies and Source IP for Services with Type=LoadBalancer from the official Kubernetes docs.

This load balancer receives traffic on HTTP and HTTPS ports 80 and 443, and forwards it to the Ingress Controller Pod. The Ingress Controller will then route the traffic to the appropriate backend Service.


We can now point our DNS records at this external Load Balancer and create some Ingress Resources to implement traffic routing rules.


# Step 3 — Creating the Ingress Resource


Let’s begin by creating a minimal Ingress Resource to route traffic directed at a given subdomain to a corresponding backend Service.


In this guide, we’ll use the test domain example.com. You should substitute this with the domain name you own.


We’ll first create a simple rule to route traffic directed at echo1.example.com to the echo1 backend service and traffic directed at echo2.example.com to the echo2 backend service.


Begin by opening up a file called echo_ingress.yaml in your favorite editor:


```
nano echo_ingress.yaml


```


Paste in the following ingress definition:


echo_ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
spec:
  rules:
  - host: echo1.example.com
    http:
        paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: echo1
              port:
                number: 80
  - host: echo2.example.com
    http:
        paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: echo2
              port:
                number: 80

```


When you’ve finished editing your Ingress rules, save and close the file.


Here, we’ve specified that we’d like to create an Ingress Resource called echo-ingress, and route traffic based on the Host header. An HTTP request Host header specifies the domain name of the target server. To learn more about Host request headers, consult the Mozilla Developer Network definition page. Requests with host echo1.example.com will be directed to the echo1 backend set up in Step 1, and requests with host echo2.example.com will be directed to the echo2 backend.


You can now create the Ingress using kubectl:


```
kubectl apply -f echo_ingress.yaml


```


You’ll see the following output confirming the Ingress creation:


```
Outputingress.networking.k8s.io/echo-ingress created

```


To test the Ingress, navigate to your DNS management service and create A records for echo1.example.com and echo2.example.com pointing to the DigitalOcean Load Balancer’s external IP. The Load Balancer’s external IP is the external IP address for the ingress-nginx Service, which we fetched in the previous step. If you are using DigitalOcean to manage your domain’s DNS records, consult How to Manage DNS Records to learn how to create A records.


Once you’ve created the necessary echo1.example.com and echo2.example.com DNS records, you can test the Ingress Controller and Resource you’ve created using the  curl command line utility.


From your local machine, curl the echo1 Service:


```
curl echo1.example.com


```


You should get the following response from the echo1 service:


```
Outputecho1

```


This confirms that your request to echo1.example.com is being correctly routed through the Nginx ingress to the echo1 backend Service.


Now, perform the same test for the echo2 Service:


```
curl echo2.example.com


```


You should get the following response from the echo2 Service:


```
Outputecho2

```


This confirms that your request to echo2.example.com is being correctly routed through the Nginx ingress to the echo2 backend Service.


At this point, you’ve successfully set up a minimal Nginx Ingress to perform virtual host-based routing. In the next step, we’ll install cert-manager to provision TLS certificates for our Ingress and enable the more secure HTTPS protocol.


# Step 4 — Installing and Configuring Cert-Manager


In this step, we’ll install v1.7.1 of cert-manager into our cluster. cert-manager is a Kubernetes add-on that provisions TLS certificates from Let’s Encrypt and other certificate authorities (CAs) and manages their lifecycles. Certificates can be automatically requested and configured by annotating Ingress Resources, appending a tls section to the Ingress spec, and configuring one or more Issuers or ClusterIssuers to specify your preferred certificate authority. To learn more about Issuer and ClusterIssuer objects, consult the official cert-manager documentation on Issuers.


Install cert-manager and its Custom Resource Definitions (CRDs) like Issuers and ClusterIssuers by following the official installation instructions. Note that a namespace called cert-manager will be created into which the cert-manager objects will be created:


```
kubectl apply -f https://github.com/jetstack/cert-manager/releases/download/v1.7.1/cert-manager.yaml


```


You should see the following output:


```
Outputcustomresourcedefinition.apiextensions.k8s.io/certificaterequests.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/certificates.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/challenges.acme.cert-manager.io created
customresourcedefinition.apiextensions.k8s.io/clusterissuers.cert-manager.io created
. . .
deployment.apps/cert-manager-webhook created
mutatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created
validatingwebhookconfiguration.admissionregistration.k8s.io/cert-manager-webhook created

```


To verify our installation, check the cert-manager Namespace for running pods:


```
kubectl get pods --namespace cert-manager


```


```
OutputNAME                                       READY   STATUS    RESTARTS   AGE
cert-manager-578cd6d964-hr5v2              1/1     Running   0          99s
cert-manager-cainjector-5ffff9dd7c-f46gf   1/1     Running   0          100s
cert-manager-webhook-556b9d7dfd-wd5l6      1/1     Running   0          99s

```


This indicates that the cert-manager installation succeeded.


Before we begin issuing certificates for our echo1.example.com and echo2.example.com domains, we need to create an Issuer, which specifies the certificate authority from which signed x509 certificates can be obtained. In this guide, we’ll use the Let’s Encrypt certificate authority, which provides free TLS certificates and offers both a staging server for testing your certificate configuration, and a production server for rolling out verifiable TLS certificates.


Let’s create a test ClusterIssuer to make sure the certificate provisioning mechanism is functioning correctly. A ClusterIssuer is not namespace-scoped and can be used by Certificate resources in any namespace.


Open a file named staging_issuer.yaml in your favorite text editor:


```
nano staging_issuer.yaml

```


Paste in the following ClusterIssuer manifest:


staging_issuer.yaml
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
 name: letsencrypt-staging
 namespace: cert-manager
spec:
 acme:
   # The ACME server URL
   server: https://acme-staging-v02.api.letsencrypt.org/directory
   # Email address used for ACME registration
   email: your_email_address_here
   # Name of a secret used to store the ACME account private key
   privateKeySecretRef:
     name: letsencrypt-staging
   # Enable the HTTP-01 challenge provider
   solvers:
   - http01:
       ingress:
         class:  nginx

```


Here we specify that we’d like to create a ClusterIssuer called letsencrypt-staging, and use the Let’s Encrypt staging server. We’ll later use the production server to roll out our certificates, but the production server rate-limits requests made against it, so for testing purposes you should use the staging URL.


We then specify an email address to register the certificate, and create a Kubernetes Secret called letsencrypt-staging to store the ACME account’s private key. We also use the HTTP-01 challenge mechanism. To learn more about these parameters, consult the official cert-manager documentation on Issuers.


Roll out the ClusterIssuer using kubectl:


```
kubectl create -f staging_issuer.yaml


```


You should see the following output:


```
Outputclusterissuer.cert-manager.io/letsencrypt-staging created

```


We’ll now repeat this process to create the production ClusterIssuer. Note that certificates will only be created after annotating and updating the Ingress resource provisioned in the previous step.


Open a file called prod_issuer.yaml in your favorite editor:


```
nano prod_issuer.yaml

```


Paste in the following manifest:


prod_issuer.yaml
```
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
  namespace: cert-manager
spec:
  acme:
    # The ACME server URL
    server: https://acme-v02.api.letsencrypt.org/directory
    # Email address used for ACME registration
    email: your_email_address_here
    # Name of a secret used to store the ACME account private key
    privateKeySecretRef:
      name: letsencrypt-prod
    # Enable the HTTP-01 challenge provider
    solvers:
    - http01:
        ingress:
          class: nginx

```


Note the different ACME server URL, and the letsencrypt-prod secret key name.


When you’re done editing, save and close the file.


Roll out this Issuer using kubectl:


```
kubectl create -f prod_issuer.yaml


```


You should see the following output:


```
Outputclusterissuer.cert-manager.io/letsencrypt-prod created

```


Now that we’ve created our Let’s Encrypt staging and prod ClusterIssuers, we’re ready to modify the Ingress Resource we created above and enable TLS encryption for the echo1.example.com and echo2.example.com paths.


If you’re using DigitalOcean Kubernetes, you first need to implement a workaround so that Pods can communicate with other Pods using the Ingress. If you’re not using DigitalOcean Kubernetes, you can skip ahead to Step 6.


# Step 5 — Enabling Pod Communication through the Load Balancer (optional)


Before it provisions certificates from Let’s Encrypt, cert-manager first performs a self-check to ensure that Let’s Encrypt can reach the cert-manager Pod that validates your domain. For this check to pass on DigitalOcean Kubernetes, you need to enable Pod-Pod communication through the Nginx Ingress load balancer.


To do this, we’ll create a DNS A record that points to the external IP of the cloud load balancer, and annotate the Nginx Ingress Service manifest with this subdomain.


Begin by navigating to your DNS management service and create an A record for workaround.example.com pointing to the DigitalOcean Load Balancer’s external IP. The Load Balancer’s external IP is the external IP address for the ingress-nginx Service, which we fetched in Step 2. If you are using DigitalOcean to manage your domain’s DNS records, consult How to Manage DNS Records to learn how to create A records. Here we use the subdomain workaround but you’re free to use whichever subdomain you prefer.


Now that you’ve created a DNS record pointing to the Ingress load balancer, annotate the Ingress LoadBalancer Service with the do-loadbalancer-hostname annotation. Open a file named ingress_nginx_svc.yaml in your favorite editor and paste in the following LoadBalancer manifest:


ingress_nginx_svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    service.beta.kubernetes.io/do-loadbalancer-enable-proxy-protocol: 'true'
    service.beta.kubernetes.io/do-loadbalancer-hostname: "workaround.example.com"
  labels:
    helm.sh/chart: ingress-nginx-4.0.6
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/version: 1.1.1
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/component: controller
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  type: LoadBalancer
  externalTrafficPolicy: Local
  ports:
    - name: http
      port: 80
      protocol: TCP
      targetPort: http
    - name: https
      port: 443
      protocol: TCP
      targetPort: https
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/component: controller

```


This Service manifest was extracted from the complete Nginx Ingress manifest file that you installed in Step 2. Be sure to copy the Service manifest corresponding to the Nginx Ingress version you installed; in this tutorial, this is 1.1.1. Also be sure to set the do-loadbalancer-hostname annotation to the workaround.example.com domain.


When you’re done, save and close the file.


Modify the running ingress-nginx-controller Service using kubectl apply:


```
kubectl apply -f ingress_nginx_svc.yaml

```


You should see the following output:


```
Outputservice/ingress-nginx-controller configured

```


This confirms that you’ve annotated the ingress-nginx-controller service and Pods in your cluster can now communicate with one another using this ingress-nginx-controller Load Balancer.


# Step 6 — Issuing Staging and Production Let’s Encrypt Certificates


To issue a staging TLS certificate for our domains, we’ll annotate echo_ingress.yaml with the ClusterIssuer created in Step 4. This will use ingress-shim to automatically create and issue certificates for the domains specified in the Ingress manifest.


Open up echo_ingress.yaml in your favorite editor:


```
nano echo_ingress.yaml


```


Add the following to the Ingress resource manifest:


echo_ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-staging"
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - echo1.example.com
    - echo2.example.com
    secretName: echo-tls
  rules:
    - host: echo1.example.com
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: echo1
                port:
                  number: 80
    - host: echo2.example.com
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: echo2
                port:
                  number: 80

```


Here we add an annotation to set the cert-manager ClusterIssuer to letsencrypt-staging, the test certificate ClusterIssuer created in Step 4. We also add an annotation that describes the type of ingress, in this case nginx.


We also add a tls block to specify the hosts for which we want to acquire certificates, and specify a secretName. This secret will contain the TLS private key and issued certificate. Be sure to swap out example.com with the domain for which you’ve created DNS records.


When you’re done making changes, save and close the file.


We’ll now push this update to the existing Ingress object using kubectl apply:


```
kubectl apply -f echo_ingress.yaml


```


You should see the following output:


```
Outputingress.networking.k8s.io/echo-ingress configured

```


You can use kubectl describe to track the state of the Ingress changes you’ve just applied:


```
kubectl describe ingress


```


```
OutputEvents:
  Type    Reason             Age               From                      Message
  ----    ------             ----              ----                      -------
  Normal  UPDATE             6s (x3 over 80m)  nginx-ingress-controller  Ingress default/echo-ingress
  Normal  CreateCertificate  6s                cert-manager              Successfully created Certificate "echo-tls"

```


Once the certificate has been successfully created, you can run a describe on it to further confirm its successful creation:


```
kubectl describe certificate


```


You should see the following output in the Events section:


```
OutputEvents:
  Type    Reason     Age    From          Message
  ----    ------     ----   ----          -------
  Normal  Requested  64s    cert-manager  Created new CertificateRequest resource "echo-tls-vscfw"
  Normal  Issuing    40s    cert-manager  The certificate has been successfully issued

```


This confirms that the TLS certificate was successfully issued and HTTPS encryption is now active for the two domains configured.


We’re now ready to send a request to a backend echo server to test that HTTPS is functioning correctly.


Run the following wget command to send a request to echo1.example.com and print the response headers to STDOUT:


```
wget --save-headers -O- echo1.example.com


```


You should see the following output:


```
Output. . .
HTTP request sent, awaiting response... 308 Permanent Redirect
. . .
ERROR: cannot verify echo1.example.com's certificate, issued by ‘ERROR: cannot verify echo1.example.com's certificate, issued by ‘CN=(STAGING) Artificial Apricot R3,O=(STAGING) Let's Encrypt,C=US’:
  Unable to locally verify the issuer's authority.
To connect to echo1.example.com insecurely, use `--no-check-certificate'.

```


This indicates that HTTPS has successfully been enabled, but the certificate cannot be verified as it’s a fake temporary certificate issued by the Let’s Encrypt staging server.


Now that we’ve tested that everything works using this temporary fake certificate, we can roll out production certificates for the two hosts echo1.example.com and echo2.example.com. To do this, we’ll use the letsencrypt-prod ClusterIssuer.


Update echo_ingress.yaml to use letsencrypt-prod:


```
nano echo_ingress.yaml


```


Make the following change to the file:


echo_ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: echo-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    kubernetes.io/ingress.class: "nginx"
spec:
  tls:
  - hosts:
    - echo1.example.com
    - echo2.example.com
    secretName: echo-tls
  rules:
    - host: echo1.example.com
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: echo1
                port:
                  number: 80
    - host: echo2.example.com
      http:
        paths:
          - pathType: Prefix
            path: "/"
            backend:
              service:
                name: echo2
                port:
                  number: 80

```


Here, we update the ClusterIssuer name to letsencrypt-prod.


Once you’re satisfied with your changes, save and close the file.


Roll out the changes using kubectl apply:


```
kubectl apply -f echo_ingress.yaml


```


```
Outputingress.networking.k8s.io/echo-ingress configured

```


Wait a couple of minutes for the Let’s Encrypt production server to issue the certificate. You can track its progress using kubectl describe on the certificate object:


```
kubectl describe certificate echo-tls


```


Once you see the following output, the certificate has been issued successfully:


```
Output Normal  Issuing    28s                 cert-manager  Issuing certificate as Secret was previously issued by ClusterIssuer.cert-manager.io/letsencrypt-staging
  Normal  Reused     28s                 cert-manager  Reusing private key stored in existing Secret resource "echo-tls"
  Normal  Requested  28s                 cert-manager  Created new CertificateRequest resource "echo-tls-49gmn"
  Normal  Issuing    2s (x2 over 4m52s)  cert-manager  The certificate has been successfully issued

```


We’ll now perform a test using curl to verify that HTTPS is working correctly:


```
curl echo1.example.com


```


You should see the following:


```
Output<html>
<head><title>308 Permanent Redirect</title></head>
<body>
<center><h1>308 Permanent Redirect</h1></center>
<hr><center>nginx/1.15.9</center>
</body>
</html>

```


This indicates that HTTP requests are being redirected to use HTTPS.


Run curl on https://echo1.example.com:


```
curl https://echo1.example.com


```


You should now see the following output:


```
Outputecho1

```


You can run the previous command with the verbose -v flag to dig deeper into the certificate handshake and to verify the certificate information.


At this point, you’ve successfully configured HTTPS using a Let’s Encrypt certificate for your Nginx Ingress.


# Conclusion


In this guide, you set up an Nginx Ingress to load balance and route external requests to backend Services inside of your Kubernetes cluster. You also secured the Ingress by installing the cert-manager certificate provisioner and setting up a Let’s Encrypt certificate for two host paths.


There are many alternatives to the Nginx Ingress Controller. To learn more, consult Ingress controllers from the official Kubernetes documentation.


For a guide on rolling out the Nginx Ingress Controller using the Helm Kubernetes package manager, consult How To Set Up an Nginx Ingress on DigitalOcean Kubernetes Using Helm.


