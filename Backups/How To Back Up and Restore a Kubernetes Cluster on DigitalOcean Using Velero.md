# How To Back Up and Restore a Kubernetes Cluster on DigitalOcean Using Velero

```Kubernetes``` ```Backups``` ```Solutions``` ```Object Storage``` ```DigitalOcean``` ```Block Storage```

## Introduction


Velero is a convenient backup tool for Kubernetes clusters that compresses and backs up Kubernetes objects to object storage. It also takes snapshots of your cluster’s Persistent Volumes using your cloud provider’s block storage snapshot features, and can then restore your cluster’s objects and Persistent Volumes to a previous state.


The DigitalOcean Velero Plugin allows you to use DigitalOcean block storage to snapshot your Persistent Volumes, and Spaces to back up your Kubernetes objects. When running a Kubernetes cluster on DigitalOcean, this allows you to quickly back up your cluster’s state and restore it should disaster strike.


In this tutorial we’ll set up and configure the velero command line tool on a local machine, and deploy the server component into our Kubernetes cluster. We’ll then deploy a sample Nginx app that uses a Persistent Volume for logging and then simulate a disaster recovery scenario.


If you’re looking for a managed Kubernetes hosting service, check out our simple, managed Kubernetes service built for growth.


# Prerequisites


Before you begin this tutorial, you should have the following available to you:


On your local computer:


- The kubectl command-line tool, configured to connect to your cluster. You can read more about installing and configuring kubectl in the official Kubernetes documentation.
- The git command-line utility. You can learn how to install git in  Getting Started with Git.

In your DigitalOcean account:


- A DigitalOcean Kubernetes cluster, or a Kubernetes cluster (version 1.7.5 or later) on DigitalOcean Droplets.
- A DNS server running inside of your cluster. If you are using DigitalOcean Kubernetes, this is running by default. To learn more about configuring a Kubernetes DNS service, consult Customizing DNS Service from the official Kuberentes documentation.
- A DigitalOcean Space that will store your backed-up Kubernetes objects. To learn how to create a Space, consult the Spaces product documentation.
- An access key pair for your DigitalOcean Space. To learn how to create a set of access keys, consult How to Manage Administrative Access to Spaces.
- A personal access token for use with the DigitalOcean API. To learn how to create a personal access token, consult How to Create a Personal Access Token. Ensure that the token you create or use has Read/Write permissions or snapshots will not work.

Once you have all of this set up, you’re ready to begin with this guide.


# Step 1 — Installing the Velero Client


The Velero backup tool consists of a client installed on your local computer and a server that runs in your Kubernetes cluster. To begin, we’ll install the local Velero client.


In your web browser, navigate to the Velero GitHub repo releases page, find the release corresponding to your OS and system architecture, and copy the link address. For the purposes of this guide, we’ll use an Ubuntu 18.04 server on an x86-64 (or AMD64) processor as our local machine, and the Velero v1.2.0 release.



Note: To follow this guide, you should download and install v1.2.0 of the Velero client.

Then, from the command line on your local computer, navigate to the temporary /tmp directory and cd into it:


```
cd /tmp


```


Use wget and the link you copied earlier to download the release tarball:


```
wget https://link_copied_from_release_page


```


Once the download completes, extract the tarball using tar (note the filename may differ depending on the release version and your OS):


```
tar -xvzf velero-v1.2.0-linux-amd64.tar.gz


```


The /tmp directory should now contain the extracted velero-v1.2.0-linux-amd64 directory as well as the tarball you just downloaded.


Verify that you can run the velero client by executing the binary:


```
./velero-v1.2.0-linux-amd64/velero help


```


You should see the following help output:


```
OutputVelero is a tool for managing disaster recovery, specifically for Kubernetes
cluster resources. It provides a simple, configurable, and operationally robust
way to back up your application state and associated data.

If you're familiar with kubectl, Velero supports a similar model, allowing you to
execute commands such as 'velero get backup' and 'velero create schedule'. The same
operations can also be performed as 'velero backup get' and 'velero schedule create'.

Usage:
  velero [command]

Available Commands:
  backup            Work with backups
  backup-location   Work with backup storage locations
  bug               Report a Velero bug
  client            Velero client related commands
  completion        Output shell completion code for the specified shell (bash or zsh)
  create            Create velero resources
  delete            Delete velero resources
  describe          Describe velero resources
  get               Get velero resources
  help              Help about any command
  install           Install Velero
  plugin            Work with plugins
  restic            Work with restic
  restore           Work with restores
  schedule          Work with schedules
  snapshot-location Work with snapshot locations
  version           Print the velero version and associated image
. . .

```


At this point you should move the velero executable out of the temporary /tmp directory and add it to your PATH. To add it to your PATH on an Ubuntu system, simply copy it to /usr/local/bin:


```
sudo mv velero-v1.2.0-linux-amd64/velero /usr/local/bin/velero


```


You’re now ready to configure secrets for the Velero server and then deploy it to your Kubernetes cluster.


# Step 2 — Configuring Secrets


Before setting up the server component of Velero, you will need to prepare your DigitalOcean Spaces keys and API token. Again navigate to the temporary directory /tmp using the cd command:


```
cd /tmp


```


Now we’ll download a copy of the Velero plugin for DigitalOcean. Visit the plugin’s Github releases page and copy the link to the file ending in .tar.gz.


Use wget and the link you copied earlier to download the release tarball:


```
wget https://link_copied_from_release_page


```


Once the download completes, extract the tarball using tar (again note that the filename may differ depending on the release version):


```
tar -xvzf v1.0.0.tar.gz


```


The /tmp directory should now contain the extracted velero-plugin-1.0.0 directory as well as the tarball you just downloaded.


Next we’ll cd into the velero-plugin-1.0.0 directory:


```
cd velero-plugin-1.0.0


```


Now we can save the access keys for our DigitalOcean Space and API token for use as a Kubernetes Secret. First, open up the examples/cloud-credentials file using your favorite editor.


```
nano examples/cloud-credentials


```


The file will look like this:


/tmp/velero-plugin-1.0.0/examples/cloud-credentials
```
[default]
aws_access_key_id=<AWS_ACCESS_KEY_ID>
aws_secret_access_key=<AWS_SECRET_ACCESS_KEY>

```


Edit the <AWS_ACCESS_KEY_ID> and <AWS_SECRET_ACCESS_KEY> placeholders to use your DigitalOcean Spaces keys. Be sure to remove the < and > characters.


The next step is to edit the 01-velero-secret.patch.yaml file so that it includes your DigitalOcean API token. Open the file in your favourite editor:


```
nano examples/01-velero-secret.patch.yaml


```


It should look like this:


```
---
apiVersion: v1
kind: Secret
stringData:
digitalocean_token: <DIGITALOCEAN_API_TOKEN>
type: Opaque

```


Change the entire <DIGITALOCEAN_API_TOKEN> placeholder to use your DigitalOcean personal API token. The line should look something like digitalocean_token: 18a0d730c0e0..... Again, make sure to remove the < and > characters.


# Step 3 — Installing the Velero Server


A Velero installation consists of a number of Kubernetes objects that all work together to create, schedule, and manage backups. The velero executable that you just downloaded can generate and install these objects for you. The velero install command will perform the preliminary set-up steps to get your cluster ready for backups. Specifically, it will:


- 
Create a velero Namespace.

- 
Add the velero Service Account.

- 
Configure role-based access control (RBAC) rules to grant permissions to the velero Service Account.

- 
Install Custom Resource Definitions (CRDs) for the Velero-specific resources: Backup, Schedule, Restore, Config.

- 
Register Velero Plugins to manage Block snapshots and Spaces storage.


We will run the velero install command with some non-default configuration options. Specifically,  you will to need edit each of the following settings in the actual invocation of the command to match your Spaces configuration:


- --bucket velero-backups: Change the velero-backups value to match the name of your DigitalOcean Space. For example if you called your Space ‘backup-bucket’, the option would look like this: --bucket backup-bucket
- --backup-location-config s3Url=https://nyc3.digitaloceanspaces.com,region=nyc3: Change the URL and region to match your Space’s settings. Specifically, edit both nyc3 portions to match the region where your Space is hosted. For example, if your Space is hosted in the fra1 region, the line would look like this: --backup-location-config s3Url=https://fra1.digitaloceanspaces.com,region=fra1. The identifiers for regions are: nyc3, sfo2, sgp1, and fra1.

Once you are ready with the appropriate bucket and backup location settings, it is time to install Velero. Run the following command, substituting your values where required:


```
velero install \
  --provider velero.io/aws \
  --bucket velero-backups \
  --plugins velero/velero-plugin-for-aws:v1.0.0,digitalocean/velero-plugin:v1.0.0 \
  --backup-location-config s3Url=https://nyc3.digitaloceanspaces.com,region=nyc3 \
  --use-volume-snapshots=false \
  --secret-file ./examples/cloud-credentials


```


You should see the following output:


```
OutputCustomResourceDefinition/backups.velero.io: attempting to create resource
CustomResourceDefinition/backups.velero.io: created
CustomResourceDefinition/backupstoragelocations.velero.io: attempting to create resource
CustomResourceDefinition/backupstoragelocations.velero.io: created
CustomResourceDefinition/deletebackuprequests.velero.io: attempting to create resource
CustomResourceDefinition/deletebackuprequests.velero.io: created
CustomResourceDefinition/downloadrequests.velero.io: attempting to create resource
CustomResourceDefinition/downloadrequests.velero.io: created
CustomResourceDefinition/podvolumebackups.velero.io: attempting to create resource
CustomResourceDefinition/podvolumebackups.velero.io: created
CustomResourceDefinition/podvolumerestores.velero.io: attempting to create resource
CustomResourceDefinition/podvolumerestores.velero.io: created
CustomResourceDefinition/resticrepositories.velero.io: attempting to create resource
CustomResourceDefinition/resticrepositories.velero.io: created
CustomResourceDefinition/restores.velero.io: attempting to create resource
CustomResourceDefinition/restores.velero.io: created
CustomResourceDefinition/schedules.velero.io: attempting to create resource
CustomResourceDefinition/schedules.velero.io: created
CustomResourceDefinition/serverstatusrequests.velero.io: attempting to create resource
CustomResourceDefinition/serverstatusrequests.velero.io: created
CustomResourceDefinition/volumesnapshotlocations.velero.io: attempting to create resource
CustomResourceDefinition/volumesnapshotlocations.velero.io: created
Waiting for resources to be ready in cluster...
Namespace/velero: attempting to create resource
Namespace/velero: created
ClusterRoleBinding/velero: attempting to create resource
ClusterRoleBinding/velero: created
ServiceAccount/velero: attempting to create resource
ServiceAccount/velero: created
Secret/cloud-credentials: attempting to create resource
Secret/cloud-credentials: created
BackupStorageLocation/default: attempting to create resource
BackupStorageLocation/default: created
Deployment/velero: attempting to create resource
Deployment/velero: created
Velero is installed! ⛵ Use 'kubectl logs deployment/velero -n velero' to view the status.

```


You can watch the deployment logs using the kubectl command from the output. Once your deploy is ready, you can proceed to the next step, which is configuring the server. A successful deploy will look like this (with a different AGE column):


```
kubectl get deployment/velero --namespace velero


```


```
OutputNAME     READY   UP-TO-DATE   AVAILABLE   AGE
velero   1/1     1            1           2m

```


At this point you have installed the server component of Velero into your Kubernetes cluster as a Deployment. You have also registered your Spaces keys with Velero using a Kubernetes Secret.



Note: You can specify the kubeconfig that the velero command line tool should use with the --kubeconfig flag. If you don’t use this flag, velero will check the KUBECONFIG environment variable and then fall back to the kubectl default (~/.kube/config).

# Step 4 — Configuring snapshots


When we installed the Velero server, the option --use-volume-snapshots=false was part of the command. Since we want to take snapshots of the underlying block storage devices in our Kubernetes cluster, we need to tell Velero to use the correct plugin for DigitalOcean block storage.


Run the following command to enable the plugin and register it as the default snapshot provider:


```
velero snapshot-location create default --provider digitalocean.com/velero


```


You will see the following output:


```
OutputSnapshot volume location "default" configured successfully.

```


# Step 5 — Adding an API token


In the previous step we created block storage and object storage objects in the Velero server. We’ve registered the digitalocean/velero-plugin:v1.0.0 plugin with the server, and installed our Spaces secret keys into the cluster.


The final step is patching the cloud-credentials Secret that we created earlier to use our DigitalOcean API token. Without this token the snapshot plugin will not be able to authenticate with the DigitalOcean API.


We could use the kubectl edit command to modify the Velero Deployment object with a reference to the API token. However, editing complex YAML objects by hand can be tedious and error prone. Instead, we’ll use the kubectl patch command since Kubernetes supports patching objects. Let’s take a quick look at the contents of the patch files that we’ll apply.


The first patch file is the examples/01-velero-secret.patch.yaml file that you edited earlier. It is designed to add your API token to the secrets/cloud-credentials Secret that already contains your Spaces keys. cat the file:


```
cat examples/01-velero-secret.patch.yaml


```


It should look like this (with your token in place of the <DIGITALOCEAN_API_TOKEN> placeholder):


examples/01-velero-secret.patch.yaml
```
. . .
---
apiVersion: v1
kind: Secret
stringData:
  digitalocean_token: <DIGITALOCEAN_API_TOKEN>
type: Opaque

```


Now let’s look at the patch file for the Deployment:


```
cat examples/02-velero-deployment.patch.yaml


```


You should see the following YAML:


examples/02-velero-deployment.patch.yaml
```
. . .
---
apiVersion: v1
kind: Deployment
spec:
  template:
    spec:
      containers:
      - args:
        - server
        command:
        - /velero
        env:
        - name: DIGITALOCEAN_TOKEN
          valueFrom:
            secretKeyRef:
              key: digitalocean_token
              name: cloud-credentials
        name: velero

```


This file indicates that we’re patching a Deployment’s Pod spec that is called velero. Since this is a patch we do not need to specify an entire Kubernetes object spec or metadata. In this case the Velero Deployment is already configured using the cloud-credentials secret because the velero install command created it for us. So all that this patch needs to do is register the digitalocean_token as an environment variable with the already deployed Velero Pod.


Let’s apply the first Secret patch using the kubectl patch command:


```
kubectl patch secret/cloud-credentials -p "$(cat examples/01-velero-secret.patch.yaml)" --namespace velero


```


You should see the following output:


```
Outputsecret/cloud-credentials patched

```


Finally we will patch the Deployment. Run the following command:


```
kubectl patch deployment/velero -p "$(cat examples/02-velero-deployment.patch.yaml)" --namespace velero


```


You will see the following if the patch is successful:


```
Outputdeployment.apps/velero patched

```


Let’s verify the patched Deployment is working using kubectl get on the velero Namespace:


```
kubectl get deployment/velero --namespace velero


```


You should see the following output:


```
OutputNAME     READY   UP-TO-DATE   AVAILABLE   AGE
velero   1/1     1            1           12s

```


At this point Velero is running and fully configured, and ready to back up and restore your Kubernetes cluster objects and Persistent Volumes to DigitalOcean Spaces and Block Storage.


In the next section, we’ll run a quick test to make sure that the backup and restore functionality works as expected.


# Step 6 — Testing Backup and Restore Procedure


Now that we’ve successfully installed and configured Velero, we can create a test Nginx Deployment, with a Persistent Volume and Service. Once the Deployment is running we will run through a backup and restore drill to ensure that Velero is configured and working properly.


Ensure you are still working in the /tmp/velero-plugin-1.0.0 directory. The examples directory contains a sample Nginx manifest called nginx-example.yaml.


Open this file using your editor of choice:


```
nano examples/nginx-example.yaml


```


You should see the following text:


```
Output. . .
---
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-example
  labels:
    app: nginx

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx-logs
  namespace: nginx-example
  labels:
    app: nginx
spec:
  storageClassName: do-block-storage
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deploy
  namespace: nginx-example
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      volumes:
        - name: nginx-logs
          persistentVolumeClaim:
           claimName: nginx-logs
      containers:
      - image: nginx:stable
        name: nginx
        ports:
        - containerPort: 80
        volumeMounts:
          - mountPath: "/var/log/nginx"
            name: nginx-logs
            readOnly: false

---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx-svc
  namespace: nginx-example
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx
  type: LoadBalancer

```


In this file, we observe specs for:


- An Nginx namespace called nginx-example
- An Nginx Deployment consisting of a single replica of the nginx:stable container image
- A 5Gi Persistent Volume Claim (called nginx-logs), using the do-block-storage StorageClass
- A LoadBalancer Service that exposes port 80

Create the objects using kubectl apply:


```
kubectl apply -f examples/nginx-example.yaml


```


You should see the following output:


```
Outputnamespace/nginx-example created
persistentvolumeclaim/nginx-logs created
deployment.apps/nginx-deploy created
service/nginx-svc created

```


Check that the Deployment succeeded:


```
kubectl get deployments --namespace=nginx-example


```


You should see the following output:


```
OutputNAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           1m23s

```


Once Available reaches 1, fetch the Nginx load balancer’s external IP using kubectl get:


```
kubectl get services --namespace=nginx-example


```


You should see both the internal CLUSTER-IP and EXTERNAL-IP for the my-nginx Service:


```
OutputNAME        TYPE           CLUSTER-IP      EXTERNAL-IP      PORT(S)        AGE
nginx-svc   LoadBalancer   10.245.147.61   159.203.48.191   80:30232/TCP   3m1s

```


Note the EXTERNAL-IP and navigate to it using your web browser.


You should see the following NGINX welcome page:





This indicates that your Nginx Deployment and Service are up and running.


Before we simulate our disaster scenario, let’s first check the Nginx access logs (stored on a Persistent Volume attached to the Nginx Pod):


Fetch the Pod’s name using kubectl get:


```
kubectl get pods --namespace nginx-example


```


```
OutputNAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-694c85cdc8-vknsk   1/1     Running   0          4m14s

```


Now, exec into the running Nginx container to get a shell inside of it:


```
kubectl exec -it nginx-deploy-694c85cdc8-vknsk --namespace nginx-example -- /bin/bash


```


Once inside the Nginx container, cat the Nginx access logs:


```
cat /var/log/nginx/access.log


```


You should see some Nginx access entries:


```
Output10.244.0.119 - - [03/Jan/2020:04:43:04 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0" "-"
10.244.0.119 - - [03/Jan/2020:04:43:04 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0" "-"

```


Note these down (especially the timestamps), as we will use them to confirm the success of the restore procedure. Exit the pod:


```
exit


```


We can now perform the backup procedure to copy all nginx Kubernetes objects to Spaces and take a Snapshot of the Persistent Volume we created when deploying Nginx.


We’ll create a backup called nginx-backup using the velero command line client:


```
velero backup create nginx-backup --selector app=nginx


```


The --selector app=nginx instructs the Velero server to only back up Kubernetes objects with the app=nginx Label Selector.


You should see the following output:


```
OutputBackup request "nginx-backup" submitted successfully.
Run `velero backup describe nginx-backup` or `velero backup logs nginx-backup` for more details.

```


Running velero backup describe nginx-backup --details should provide the following output after a short delay:


```
OutputName:         nginx-backup
Namespace:    velero
Labels:       velero.io/backup=nginx-backup
              velero.io/pv=pvc-6b7f63d7-752b-4537-9bb0-003bed9129ca
              velero.io/storage-location=default
Annotations:  <none>

Phase:  Completed

Namespaces:
  Included:  *
  Excluded:  <none>

Resources:
  Included:        *
  Excluded:        <none>
  Cluster-scoped:  auto

Label selector:  app=nginx

Storage Location:  default

Snapshot PVs:  auto

TTL:  720h0m0s

Hooks:  <none>

Backup Format Version:  1

Started:    2020-01-02 23:45:30 -0500 EST
Completed:  2020-01-02 23:45:34 -0500 EST

Expiration:  2020-02-01 23:45:30 -0500 EST

Resource List:
  apps/v1/Deployment:
    - nginx-example/nginx-deploy
  apps/v1/ReplicaSet:
    - nginx-example/nginx-deploy-694c85cdc8
  v1/Endpoints:
    - nginx-example/nginx-svc
  v1/Namespace:
    - nginx-example
  v1/PersistentVolume:
    - pvc-6b7f63d7-752b-4537-9bb0-003bed9129ca
  v1/PersistentVolumeClaim:
    - nginx-example/nginx-logs
  v1/Pod:
    - nginx-example/nginx-deploy-694c85cdc8-vknsk
  v1/Service:
    - nginx-example/nginx-svc

Persistent Volumes:
  pvc-6b7f63d7-752b-4537-9bb0-003bed9129ca:
    Snapshot ID:        dfe866cc-2de3-11ea-9ec0-0a58ac14e075
    Type:               ext4
    Availability Zone:  
    IOPS:               <N/A>

```


This output indicates that nginx-backup completed successfully. The list of resources shows each of the Kubernetes objects that was included in the backup. The final section shows the PersistentVolume was also backed up using a filesystem snapshot.


To confirm from within the DigitalOcean Cloud Control Panel, navigate to the Space containing your Kubernetes backup files.


You should see a new directory called nginx-backup containing the Velero backup files.


Using the left-hand navigation bar, go to Images and then Snapshots. Within Snapshots, navigate to Volumes. You should see a Snapshot corresponding to the PVC listed in the above output.


We can now test the restore procedure.


Let’s first delete the nginx-example Namespace. This will delete everything in the Namespace, including the Load Balancer and Persistent Volume:


```
kubectl delete namespace nginx-example


```


Verify that you can no longer access Nginx at the Load Balancer endpoint, and that the nginx-example Deployment is no longer running:


```
kubectl get deployments --namespace=nginx-example


```


```
OutputNo resources found in nginx-example namespace.

```


We can now perform the restore procedure, once again using the velero client:


```
velero restore create --from-backup nginx-backup


```


Here we use create to create a Velero Restore object from the nginx-backup object.


You should see the following output:


```
OutputRestore request "nginx-backup-20200102235032" submitted successfully.
Run `velero restore describe nginx-backup-20200102235032` or `velero restore logs nginx-backup-20200102235032` for more details.


```


Check the status of the restored Deployment:


```
kubectl get deployments --namespace=nginx-example


```


```
OutputNAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           58s

```


Check for the creation of a Persistent Volume:


```
 kubectl get pvc --namespace=nginx-example


```


```
OutputNAME         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS       AGE
nginx-logs   Bound    pvc-6b7f63d7-752b-4537-9bb0-003bed9129ca   5Gi        RWO            do-block-storage   75s

```


The restore also created a LoadBalancer. Sometimes the Service will be re-created with a new IP address. You will need to find the EXTERNAL-IP address again:


```
kubectl get services --namespace nginx-example


```


```
OutputNAME        TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
nginx-svc   LoadBalancer   10.245.15.83   159.203.48.191   80:31217/TCP   97s

```


Navigate to the Nginx Service’s external IP once again to confirm that Nginx is up and running.


Finally, check the logs on the restored Persistent Volume to confirm that the log history has been preserved post-restore.


To do this, once again fetch the Pod’s name using kubectl get:


```
kubectl get pods --namespace nginx-example


```


```
OutputNAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-694c85cdc8-vknsk   1/1     Running   0          2m20s

```


Then exec into it:


```
kubectl exec -it nginx-deploy-694c85cdc8-vknsk --namespace nginx-example -- /bin/bash


```


Once inside the Nginx container, cat the Nginx access logs:


```
cat /var/log/nginx/access.log


```


```
Output10.244.0.119 - - [03/Jan/2020:04:43:04 +0000] "GET / HTTP/1.1" 200 612 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0" "-"
10.244.0.119 - - [03/Jan/2020:04:43:04 +0000] "GET /favicon.ico HTTP/1.1" 404 153 "-" "Mozilla/5.0 (X11; Linux x86_64; rv:72.0) Gecko/20100101 Firefox/72.0" "-"

```


You should see the same pre-backup access attempts (note the timestamps), confirming that the Persistent Volume restore was successful. Note that there may be additional attempts in the logs if you visited the Nginx landing page after you performed the restore.


At this point, we’ve successfully backed up our Kubernetes objects to DigitalOcean Spaces, and our Persistent Volumes using Block Storage Volume Snapshots. We simulated a disaster scenario, and restored service to the test Nginx application.


# Conclusion


In this guide we installed and configured the Velero Kubernetes backup tool on a DigitalOcean-based Kubernetes cluster. We configured the tool to back up Kubernetes objects to DigitalOcean Spaces, and back up Persistent Volumes using Block Storage Volume Snapshots.


Velero can also be used to schedule regular backups of your Kubernetes cluster for disaster recovery. To do this, you can use the velero schedule command. Velero can also be used to migrate resources from one cluster to another.


To learn more about DigitalOcean Spaces, consult the official Spaces documentation. To learn more about Block Storage Volumes, consult the Block Storage Volume documentation.


This tutorial builds on the README found in StackPointCloud’s ark-plugin-digitalocean GitHub repo.


