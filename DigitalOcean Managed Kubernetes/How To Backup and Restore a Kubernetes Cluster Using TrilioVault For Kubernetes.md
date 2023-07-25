# How To Backup and Restore a Kubernetes Cluster Using TrilioVault For Kubernetes

```Backups``` ```DigitalOcean Managed Kubernetes``` ```Kubernetes```

# How To Backup and Restore a Kubernetes Cluster Using TrilioVault For Kubernetes


The author selected the Diversity in Tech Fund to receive a donation as part of the Write for DOnations program.


## Introduction


TrilioVault for Kubernetes (TVK) is a cloud-native solution that secures your application metadata and data by storing on-demand backups in an independent storage repository.


There are a few advantages of using Trilio. With Trilio, you can take full or incremental backups of your cluster and restore them in case of data loss. You can migrate from one cluster to another and run pre- and post-hooks for backup and restore operations. You can also schedule backups and define retention policies for your backups. Finally, you can also use the web management console to inspect your backup and restore operations state in detail (along with many other features).


This article provides instructions for protecting your local Kubernetes cluster deployment or a DigitalOcean Kubernetes Service using TVK, including the stateful or stateless applications deployed on the cluster. In this tutorial, you will deploy TVK to your Kubernetes cluster, create a cluster backup, and recover from the backup.


If you’re looking for a managed Kubernetes hosting service, check out our simple, managed Kubernetes service built for growth.


# Prerequisites


To complete this tutorial, you’ll need the following:


- A DigitalOcean account. If you do not have one, sign up for a new account.
- A DigitalOcean Kubernetes cluster with multiple namespaces. You can create a cluster by following our documentation on How To Create Clusters.
- Doctl for DigitalOcean API interaction. To get started, see our guide on How To Install and Configure doctl.
- Kubectl for Kubernetes interaction. For installation and set up, see the Kubernetes product documentation for Install Tools.
- A DigitalOcean Spaces bucket or any S3-compatible object storage bucket with its access keys. To use a DigitalOcean Spaces bucket, follow our guides on How to Create Spaces and How to Manage Administrative Access with access keys. Save the access and secret keys in a safe place for later use. You can also use NFS export to store the backup.
- Helm for managing TrilioVault Operator releases and upgrades. For installation, see Step 1 of our tutorial, How To Install Software on Kubernetes Clusters with the Helm 3 Package Manager.
- A TrilioVault license saved as a yaml file. This tutorial uses Cluster-scoped installation, which you may need to select when fetching the license. For DigitalOcean users, the TVK installation is free for five years. If you are not using a DigitalOcean Kubernetes cluster, you will need to enroll on the Trilio website to request a TVK license. There are free trials and a free basic version available.

# Step 1 — Configuring the Kubernetes Cluster


In this step, you will check the configuration of your Kubernetes cluster to ensure TrilioVault will work correctly.


For TrilioVault to work correctly and to backup your PersistentVolumeClaim (PVCs), the Kubernetes cluster needs to be configured to support the Container Storage Interface (CSI). By default, the DigitalOcean Managed Kubernetes Service comes with the CSI driver already installed and configured. You can check this using the following command:


```
kubectl get storageclass


```


The output should look similar to this:


```
OutputNAME                         PROVISIONER                 RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
do-block-storage (default)   dobs.csi.digitalocean.com   Delete          Immediate           true                   1d

```


As you can see, the provisioner is dobs.csi.digitalocean.com.


The TrilioVault installation also needs volumeSnapshot Custom Resource Definition (CRD) for successful installation. You can check this using the following command:


```
kubectl get crd | grep volumesnapshot


```


If the volumeSnapshot is already installed, the output will look similar to this:


```
Outputvolumesnapshotclasses.snapshot.storage.k8s.io2022-03-02T07:24:23Z
volumesnapshotcontents.snapshot.storage.k8s.io2022-03-02T07:24:23Z
volumesnapshots.snapshot.storage.k8s.io2022-03-02T07:24:23Z |

```


If volumeSnapshot is not already installed, refer to the documentation for Installing VolumeSnapshot CRDs.


Finally, make sure that the CRD supports the v1 API version, which you can check by running the following command:


```
kubectl get crd volumesnapshots.snapshot.storage.k8s.io -o yaml


```


At the end of the CRD yaml output, you should see a storedVersions list containing v1 values:


```
Output...
- lastTransitionTime: "2022-01-20T07:58:06Z"
    message: approved in https://github.com/kubernetes-csi/external-snapshotter/pull/419
    reason: ApprovedAnnotation
    status: "True"
    type: KubernetesAPIApprovalPolicyConformant
  storedVersions:
  - v1

```


If this is not installed, refer to the documentation for Installing VolumeSnapshot CRDs.


In this step, you confirmed that your Kubernetes configuration is prepared for installing TrilioVault, which you will do in the next step.


# Step 2 — Installing TrilioVault for Kubernetes


In this step, you will deploy TrilioVault for a local Kubernetes Cluster and manage TVK installations via Helm. Backup data will be stored in the S3-compatible bucket that you created as part of the Prerequisites.


The TrilioVault application can be installed in multiple ways, depending on the Kubernetes cluster distribution. In this tutorial, you will install TrilioVault using Helm via the triliovault-operator chart.


This tutorial uses the Cluster-scoped installation type for the tvk application. With this type of installation, TVK can protect all applications across namespaces. (In contrast, with a Namespace-scoped installation, TVK can only protect applications deployed in that namespace.)


To install TrilioVault for Kubernetes, you will need a license, which you requested as a part of the prerequisites. When fetching the TVK license, you may need to select Cluster-scoped installation.


## Installing TrilioVault Using Helm


To install TrilioVault via Helm, first add the TrilioVault Helm repository and list the available charts using the following command:


```
helm repo add triliovault-operator http://charts.k8strilio.net/trilio-stable/k8s-triliovault-operator
helm repo update triliovault-operator
helm search repo triliovault-operator


```


The output looks similar to the following:


```
OutputNAME                                            CHART VERSION   APP VERSION     DESCRIPTION
triliovault-operator/k8s-triliovault-operator   2.10.3          2.10.3          K8s-TrilioVault-Operator is an operator designe...

```


Finally, install TrilioVault for Kubernetes using helm:


```
helm install triliovault-operator triliovault-operator/k8s-triliovault-operator \
  --namespace tvk \
  --set installTVK.ingressConfig.host="demo-tutorial.tvk-doks.com" \
  --create-namespace


```


This command installs the triliovault-operator and the TriloVault Manager (TVM) Custom Resource using the default parameters provided in the TrilioVault Helm values file, triliovault-values.yaml.


- 
TVK Operator: TVK has a Helm-based Operator, which is managed by a CRD called TrilioVault Manager. The TVK Operator takes care of the lifecycle of the application and auto-recovery in case one of the application components goes down.

- 
TVK Manager: The TVK application contains several CRDs and their controllers. It has its own webhook server that manages the validation and mutation of its CRD instances. Controllers reconcile the events generated by the operations done on the Custom Resources.


This tutorial uses the default values in the TrilioVault Helm values file (triliovault-values.yaml), including the following:


- installTVK.applicationScope: The scope for the TVK installation can be Cluster or Namespaced. This parameter protects applications either across the ‘Cluster’ or ‘Namespace’, and the TVK license is also generated based on the installation scope. This tutorial uses Cluster-scoped installation.
- installTVK.ingressConfig.host: The domain name for the TVK UI hostname, which is demo-tutorial.tvk-doks.com. Users will access the TVK Management Console through this domain name.
- installTVK.ComponentConfiguration.ingressController.service.type: The service type to access the TVK UI, such as NodePort or LoadBalancer.


Note: To see all available options, you could inspect the TrilioVault Helm values file. For more information, please check the TVK documentation for configuration options.

Run the following command to check your tvk deployment:


```
helm ls -n tvk


```


This command will list the helm repositories you’ve added:


```
OutputNAME                    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                           APP VERSION
triliovault-manager-tvk tvk             1               2022-08-18 08:19:50.409742366 +0000 UTC deployed        k8s-triliovault-2.10.3          2.10.3
triliovault-operator    tvk             1               2022-08-18 08:15:51.618651231 +0000 UTC deployed        k8s-triliovault-operator-2.10.3 2.10.3

```


The STATUS column should display deployed.


Next, verify that TrilioVault is up and running. Run the following command, which will show the status of the tvk installation:


```
kubectl get deployments -n tvk


```


This command will display the deployments in the tvk namespace.


The output will look similar to the following:


```
OutputNAME                                            READY   UP-TO-DATE   AVAILABLE   AGE
k8s-triliovault-admission-webhook               1/1     1            1           83s
k8s-triliovault-control-plane                   1/1     1            1           83s
k8s-triliovault-exporter                        1/1     1            1           83s
k8s-triliovault-ingress-nginx-controller        1/1     1            1           83s
k8s-triliovault-web                             1/1     1            1           83s
k8s-triliovault-web-backend                     1/1     1            1           83s
triliovault-operator-k8s-triliovault-operator   1/1     1            1           4m22s

```


The READY column displays how many deployments are available following the pattern of ready/desired. All the deployments pods are in the READY state, which means that tvk was installed successfully. Next, you will check your TrilioVault license type and validity.


## Checking the TrilioVault Application License


As part of the prerequisites, you requested a TrilioVault license for the Cluster-scoped installation type and saved it as a yaml file. In this section, you will apply your TrilioVault license, confirm its status, and inspect its fields.


As part of the prerequisites, you downloaded a free license from Trilio’s website and saved it as a yaml file. Now, apply it using the below command:


```
kubectl apply -f your_license_filename.yaml -n tvk


```


Next, check whether the license is installed and in the Active state on your cluster:


```
kubectl get license -n tvk


```


The output looks similar to the following:


```
OutputNAME            STATUS   MESSAGE                                   CURRENT NODE COUNT   GRACE PERIOD END TIME   EDITION     CAPACITY   EXPIRATION TIME        MAX NODES
your_license   Active   Cluster License Activated successfully.   1                                            Basic       100        2023-04-25T00:00:00Z   1

```


Check the STATUS, which should be Active. You can also verify your license type in the EDITION column and the license expiration in the EXPIRATION TIME column.


The license is managed via a special CRD called the License object. You can inspect it by running the below command, replacing the highlighted portion with your license name, as shown in the previous output:


```
kubectl describe license your_license -n tvk


```


The output looks similar to the following:


```
OutputName:         your_license
Namespace:    tvk
Labels:       <none>
Annotations:  generation: 4
              triliovault.trilio.io/creator: trilio.user@trilio.io
              triliovault.trilio.io/instance-id: 1350188a-9289-49ba-9086-553e8cd7cabe
              triliovault.trilio.io/updater:
                [{"username":"system:serviceaccount:tvk:k8s-triliovault","lastUpdatedTimestamp":"2022-04-21T09:50:40.530365762Z"},{"username":"0c9f7f19-c4...
API Version:  triliovault.trilio.io/v1
Kind:         License
Metadata:
  Creation Timestamp:  2022-04-06T08:07:16Z
...
Status:
  Condition:
    Message:         Cluster License Activated successfully.
    Status:           Active
    Timestamp:        2022-04-06T08:07:17Z
  Current CPU Count:  6
  Max CP Us:          6
  Message:            Cluster License Activated successfully.
  Properties:
    Active:                        true
    Capacity:                     100
    Company:                       TRILIO-KUBERNETES-LICENSE-GEN-BASIC
    Creation Timestamp:            2022-04-21T00:00:00Z
    Edition:                      Basic
    Expiration Timestamp:         2027-04-25T00:00:00Z
    Kube UID:                      1350188a-9289-49ba-9086-553e8cd7cabe
    License ID:                    TVAULT-7f70e73e-c158-11ec-990f-0cc47a9fd48e
    Maintenance Expiry Timestamp:  2027-04-25T00:00:00Z
    Number Of Users:               -1
    Purchase Timestamp:            2022-04-21T00:00:00Z
    Scope:                         Cluster
...

```


Check the Message and Capacity fields, as well as Edition.


The above output will also tell you when the license is going to expire in the Expiration Timestamp field, as well as the Scope (Cluster-based in this case). More details can be found on the TrilioVault for Kubernetes licensing documentation page.


In this step, you applied your TVK license and confirmed its status. Next, you will explore the TVK web console, which will help you manage Targets, backups, restorations, and more.


# Step 3 — Accessing the TVK Management Console


In this step, you’ll access the TVK Management Console, where you can create Targets and manage operations such as backups and restoration via a GUI. While you can manage operations from the CLI via kubectl and CRDs, the TVK management console simplifies common tasks via point-and-click operations and provides better visualization and inspection of TVK cluster objects.


In the previous section on Installing TrilioVault Using Helm, you installed and configured the required components for the web management console. To access the console and explore the features it offers, you’ll export the kubeconfig file for your Kubernetes cluster and port-forward the ingress controller service for TVK.


Begin by exporting the kubeconfig file for your Kubernetes cluster. This step is required so that the web console can authenticate you.


If you are using the DigitalOcean Kubernetes service, you can follow these steps to export your kubeconfig file.


List the available clusters:


```
doctl k8s cluster list


```


Save the cluster configuration to YAML, replacing the highlighted values your with cluster’s name:


```
doctl kubernetes cluster kubeconfig show YOUR_CLUSTER_NAME_ > config_YOUR_CLUSTER_NAME_.yaml


```



Note: If you have only one cluster, the below command can be used:
DOKS_CLUSTER_NAME="$(doctl k8s cluster list --no-header --format Name)"
doctl kubernetes cluster kubeconfig show $DOKS_CLUSTER_NAME > config_${DOKS_CLUSTER_NAME}.yaml



Be sure to keep the generated kubeconfig file safe because it contains sensitive data such as the token and user details that are used to access your cluster. Store the file in a non-public location and consider using a password management application or encrypted format.


Now that you have your kubeconfig file for authentication, you will set up the port-forward to access the Management Console.


First, identify the ingress-nginx-controller service from the tvk namespace. You can do this by running the following command to list services in the tvk namespace:


```
kubectl get svc -n tvk


```


The output looks similar to the following:


```
OutputNAME                                                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
k8s-triliovault-admission-webhook                               ClusterIP   10.245.202.17    <none>        443/TCP                      13m
k8s-triliovault-ingress-nginx-controller                        NodePort    10.245.192.140   <none>        80:32448/TCP,443:32588/TCP   13m
k8s-triliovault-ingress-nginx-controller-admission              ClusterIP   10.3.20.89       <none>        443/TCP                      13m
k8s-triliovault-web                                             ClusterIP   10.245.214.13    <none>        80/TCP                       13m
k8s-triliovault-web-backend                                     ClusterIP   10.245.10.221    <none>        80/TCP                       13m
triliovault-operator-k8s-triliovault-operator-webhook-service   ClusterIP   10.245.186.59    <none>        443/TCP                      16m

```


Search for the k8s-triliovault-ingress-nginx-controller line, and notice that it listens on port 80 in the PORT(s) column.


The TrilioVault ingress service can be configured with NodePort or LoadBalancer, which you’ll see in the TYPE column. If it’s set to NodePort, then you’ll need to make sure that the domain is accessible. If you are using NodePort to access the TVK Management Console, the TrilioVault ingress service won’t be able to connect directly to the external network without resolving the domain name that is used to access the TVK Console. To solve this problem, you’ll add the IP address and domain name entry in the etc/hosts file to resolve the domain name to the IP.


To do this, open the /etc/hosts file for editing and add this entry:


/etc/hosts
```
127.0.0.1 demo-tutorial.tvk-doks.com

```


demo-tutorial.tvk-doks.com is the domain name set for the TrilioVault ingress Controller service. You will use this domain to access the TVK Management Console.


Save and close the file.


Next, create the port-forward for the TVK ingress controller service:


```
kubectl port-forward svc/k8s-triliovault-ingress-nginx-controller 8080:80 -n tvk &


```


You can now access the console in your web browser by navigating to: http://demo-tutorial.tvk-doks.com:8080. When asked for the kubeconfig file, please select the one that you created in this section.



Note: TVK uses the kubeconfig file to generate a token for authentication. It does not store the user details present in the kubeconfig file.

In this step, you set up access to the TVK management console. For more information about the console’s available features, please take a look at the TVK Web Management Console official documentation.


In the next step, you will define TrilioVault’s storage backend, which is called a target.


# Step 4 — Creating a TrilioVault Target to Store Backups


TrilioVault needs to know where to store your backups, which is referred to as a target. The following target types are supported: S3 and NFS. This tutorial uses the S3 storage type. More information is available in the Backup Target section in TVK Documentation.


To access S3 storage, each target needs to know the bucket’s credentials, which are stored in a secret. In this step, you will create a TrilioVault target for backups and a secret to store your bucket’s credentials.


To get started, you will create the Kubernetes secret containing your target S3 bucket credentials. Using nano or your favorite text editor, create a file called trilio-s3-target-secret and add the following code, making sure to replace the highlighted values with your DigitalOcean Spaces access key and secret key:


trilio-s3-target-secret.yaml
```
apiVersion: v1
kind: Secret
metadata:
  name: trilio-s3-target-secret
  namespace: tvk
type: Opaque
stringData:
  accessKey: your_bucket_access_key
  secretKey: your_bucket_secret_key

```


The secret name is trilio-s3-target-secret. This will be referenced in the spec.objectStoreCredentials.credentialSecret field of the target manifest you’ll create next. The secret can be in the same namespace where TrilioVault was installed (defaults to tvk) or in another namespace of your choice. (Just be sure that the namespace you use is correctly referenced.)


Save and close the file.


To apply this manifest and create the secret, run the following command:


```
kubectl apply -f trilio-s3-target-secret.yaml -n tvk


```



Note: Alternatively, you can create the secret by running the following command, replacing the placeholder values with your DigtalOcean bucket access key and secret key:
kubectl create secret generic trilio-s3-target-secret \
 --namespace=tvk \
 --from-literal=accessKey="your_bucket_access_key" \
 --from-literal=secretKey="your_bucket_secret_key"



Your output will look like this:


```
Outputsecret/trilio-s3-target-secret created

```


Next, you’ll create a manifest for the target. Create a new file called trilio-s3-target.yaml and add the following code block. Replace the highlighted values for bucketName, region, and url with information about your DigitalOcean bucket, which you can find on your bucket’s control panel.


trilio-s3-target.yaml
```
apiVersion: triliovault.trilio.io/v1
kind: Target
metadata:
  name: trilio-s3-target
  namespace: tvk
spec:
  type: ObjectStore
  vendor: Other
  enableBrowsing: true
  objectStoreCredentials:
    bucketName: your_bucket_name
    region: your_bucket_region           # e.g.: nyc1 or us-east-1
    url: https://nyc1.digitaloceanspaces.com      # update the region to match your bucket
    credentialSecret:
      name: trilio-s3-target-secret
      namespace: tvk
  thresholdCapacity: 10Gi

```


Here is an explanation for the above configuration:


- spec.type: Type of target for backup storage (S3 is ObjectStore).
- spec.vendor: Third-party storage vendor hosting the target (for DigitalOcean Spaces, you’ll need to use Other).
- spec.enableBrowsing: Enable browsing for the target.
- spec.objectStoreCredentials: Defines required credentials (via credentialSecret) to access the S3 storage, as well as other parameters such as bucket region and name.
- spec.thresholdCapacity: Maximum threshold capacity to store backup data.

Note that the credentialSecret name matches the secret you just created.


Save and close the manifest file.


Now, create the target object using kubectl:


```
kubectl apply -f trilio-s3-target.yaml -n tvk


```


Your output will look like this:


```
Outputtarget.triliovault.trilio.io/trilio-s3-target created

```


TrilioVault will spawn a worker job named trilio-s3-target-validator, which is responsible for validating your S3 bucket (such as availability, permissions, and so on). If the job finishes successfully, the bucket is considered to be healthy, or available, and the trilio-s3-target-validator job resource is deleted afterward.


Now, check if the target resource created earlier is healthy by running the following command and passing in the name of the target:


```
kubectl get target trilio-s3-target -n tvk


```


The output will look similar to this:


```
OutputNAME               TYPE          THRESHOLD CAPACITY   VENDOR   STATUS      BROWSING ENABLED
trilio-s3-target   ObjectStore   10Gi                 Other    Available   true

```


The STATUS column value is Available, meaning that the target is in a healthy state.


You can also validate the target status using the TVK Management Console. After logging in, select Backup & Recovery and then click Targets to view.





If the status shows as Available, you have successfully configured the S3 target object.


However, if there’s an issue in the configuration, the status will show Unavailable. In such cases, the S3 target validator job is left up and running so that you can inspect the logs and find the possible issue. In case the target object fails to become healthy, you can inspect the logs from the trilio-s3-target-validator Pod to find the issue.


To check the logs, you’ll begin by finding the target validator:


```
kubectl get pods -n tvk | grep trilio-s3-target-validator


```


The output will look similar to this, but with a unique identifier:


```
Outputtrilio-s3-target-validator-tio99a-6lz4q 1/1 Running 0 104s

```


Using the target validator from the previous output, fetch the data logs using the following command:


```
kubectl logs pod/trilio-s3-target-validator-tio99a-6lz4q -n tvk


```


The output will look similar to this (notice the exception as an example):


```
Output...
INFO:root:2021-11-24 09:06:50.595166: waiting for mount operation to complete.
INFO:root:2021-11-24 09:06:52.595772: waiting for mount operation to complete.
ERROR:root:2021-11-24 09:06:54.598541: timeout exceeded, not able to mount within time.
ERROR:root:/triliodata is not a mountpoint. We can't proceed further.
Traceback (most recent call last):
  File "/opt/tvk/datastore-attacher/mount_utility/mount_by_target_crd/mount_datastores.py", line 56, in main
    utilities.mount_datastore(metadata, datastore.get(constants.DATASTORE_TYPE), base_path)
  File "/opt/tvk/datastore-attacher/mount_utility/utilities.py", line 377, in mount_datastore
    mount_s3_datastore(metadata_list, base_path)
  File "/opt/tvk/datastore-attacher/mount_utility/utilities.py", line 306, in mount_s3_datastore
    wait_until_mount(base_path)
  File "/opt/tvk/datastore-attacher/mount_utility/utilities.py", line 328, in wait_until_mount
    base_path))
Exception: /triliodata is not a mountpoint. We can't proceed further.
...

```


For additional help debugging, or if you encounter issues while creating the target, check the Troubleshooting section of the documentation, or contact support.


In this step, you configured a TrilioVault target and created a secret to provide your bucket’s credentials. Next, you will perform backup and restore operations, thus covering a disaster recovery scenario.


# Step 5 — Backing up and Restoring the Kubernetes Cluster


In this step, you will perform a backup of your Kubernetes cluster. You will then delete the namespaces and use the backup to restore all the important applications to those namespaces. You will perform a cluster restore operation via location from the target. The same flow applies when you need to perform cluster migration.


The main idea here is to perform a complete cluster backup by including all-important namespaces, which hold your essential applications and configurations. It is not a full cluster backup and restore, but rather a multi-namespace backup and restore operation. In practice, this is all that is needed because everything is namespaced in Kubernetes.


## Creating the Kubernetes Cluster Backup


In this section, you will create a multi-namespace backup using a ClusterBackupPlan CRD that targets all-important namespaces from your Kubernetes cluster.


To start with the cluster backup operation, you will create a ClusterBackupPlan, which defines a set of resources to backup. The specification includes the backup schedule, backup target, and the resources to backup. Resources can be defined in the form of Helm release, Operators, or just bare Kubernetes API resources.


Using your text editor, create a ClusterBackupPlan manifest file called k8s-cluster-backup-plan.yaml. Add the following code block, which is a typical manifest for targeting multiple namespaces:


k8s-cluster-backup-plan.yaml
```
apiVersion: triliovault.trilio.io/v1
kind: ClusterBackupPlan
metadata:
  name: k8s-cluster-backup-plan
  namespace: tvk
spec:
  backupConfig:
    target:
      name: trilio-s3-target
      namespace: tvk
  backupComponents:
    - namespace: wordpress
    - namespace: mysqldb
    - namespace: etcd

```


Make sure the namespaces listed in the backupComponents are present on the cluster.


You may notice that kube-system (or other Kubernetes cluster-related namespaces) is not included in backupComponents. Usually, those are not required unless there is a special case requiring some settings to be persisted at that level.


Save and close the file.


Now, create the ClusterBackupPlan resource using kubectl:


```
kubectl apply -f k8s-cluster-backup-plan.yaml


```


Your output will look like this:


```
Outputclusterbackupplan.triliovault.trilio.io/k8s-cluster-backup-plan created

```


Now, inspect the ClusterBackupPlan status using kubectl:


```
kubectl get clusterbackupplan k8s-cluster-backup-plan -n tvk


```


The output looks similar to this:


```
OutputNAME                              TARGET             ...   STATUS
k8s-cluster-backup-plan           trilio-s3-target   ...   Available

```


Check the STATUS column value, which should be set to Available.


You can also see the ClusterBackupPlan status using TVK Management Console. After logging in, select Backup & Recovery and then select Backupplans to view.





At this point, you have created a ClusterBackupPlan. Next, you will create a ClusterBackup, which is a configuration pointing to the actual ClusterBackupPlan in spec.clusterBackupPlan.name. ClusterBackupPlan will always remain the same; you can create multiple backups by refreshing that into multiple ClusterBackup manifest files.


Now, create a ClusterBackup manifest file called k8s-cluster-backup.yaml. Add the following code block:


k8s-cluster-backup.yaml
```
apiVersion: triliovault.trilio.io/v1
kind: ClusterBackup
metadata:
  name: k8s-cluster-backup
  namespace: tvk
spec:
  type: Full
  clusterBackupPlan:
    name: k8s-cluster-backup-plan
    namespace: tvk

```


Save and close the file.


Finally, create the ClusterBackup resources, using kubectl:


```
kubectl apply -f k8s-cluster-backup.yaml


```


By applying the ClusterBackup manifest, the backup process will be triggered.


Your output will look like the following:


```
Outputclusterbackup.triliovault.trilio.io/k8s-cluster-backup created

```


Now, inspect the ClusterBackup status, using kubectl:


```
kubectl get clusterbackup k8s-cluster-backup -n tvk


```


The output looks similar to :


```
OutputNAME                 BACKUPPLAN               BACKUP TYPE   STATUS      ...   PERCENTAGE COMPLETE
k8s-cluster-backup   k8s-cluster-backup-plan  Full          Available   ...   100

```


Check the STATUS column value, which should be set to Available, as well as the PERCENTAGE COMPLETE set to 100.


You can also see the cluster backup status using the TVK Management Console. From the main dashboard, select Monitoring and then TrilioVault Monitoring from the left pane.





It will take a while for the full cluster backup to finish, depending on how many namespaces, associated resources, and data present on the PVCs are involved in the process. If the output looks like the above, then all the important application namespaces included in the backup plan were backed up successfully.


You can also open the web console main dashboard and inspect the multi-namespace backup. From the main dashboard, select Backup & Recovery and then Namespaces. On the upper right, you can toggle between a list view a honeycomb structure:





In the honeycomb view, all the important namespaces that were part of the backup are highlighted.


In this section, you created a cluster backup. In the next sections, you will delete the namespaces, and then restore them from the backup.


## Deleting the Namespaces


Now that you have a cluster backup, you’ll delete your namespaces so that you can restore them from the backup in a later step. To delete your namespaces, run the following commands, replacing the highlighted namespaces as needed.


```
kubectl delete ns wordpress
kubectl delete ns mysqldb
kubectl delete ns etcd


```


Your output will look like this:


```
Outputnamespace "wordpress" deleted
namespace "mysqldb" deleted
namespace "etcd" deleted

```


Now that your namespaces are deleted, you’ll restore the backup.


## Restoring the Backup with the Management Console


In this section, you will use the TVK web console to restore all the important applications from your backup. The restore process will validate the target where the backup is stored. TVK will connect to the target repository to pull the backup files using datamover and metamover pods. TVK will create the Kubernetes application that was pulled from the backup storage.


To get started with the restore operation, you’ll first need to create your target manifest again. Since the existing namespaces were deleted, the target custom resources created in those namespaces were also deleted. This means that there is no target custom resource present on the cluster.


Configure the TVK target as described in Creating a TrilioVault Target to Store Backups, and point it to the same S3 bucket where your backup data is located. Make sure that target browsing is enabled.


Then, navigate to Backup & Recovery and then Targets with Namespace: All  selected:





To list the available backups, click the Actions button on the right and then select the Launch Browser option from the drop-down menu.





For this to work, the target must have the enableBrowsing flag set to true.


Selecting Launch Browser will bring up the Target Browser:





Now, click on the k8s-cluster-backup-plan item from the list of backup plans. A sub-window will appear on the right showing information about the backup, including its status.


Click and expand the k8s-cluster-backup item from the right sub-window:





To start the restore process, click on the Restore button.


Next, you’ll see a pop-up window with some options for your restore process. To understand the different options for the restore process, you can find details about each flag in the Restore Flags section of the TrilioVault documentation.





Enter a Restore Name for the restore custom resource creation. Once you provide the name, you can map all the important namespaces from the backup to the namespaces where you want to restore the data. In this case, select Namespace Configuration in the configuration pane and then select the desired namespaces.





Once the restore process has started, you can monitor its progress. A progress window will be displayed similar to the following:





After a while, when the progress is Completed, then the multi-namespace restore operation has completed successfully.


In this section, you restored your backup using the Management Console. In the next section, you will confirm that the restore operation was successful.


## Checking the DOKS Cluster Applications State


In this section, you will make sure that the restore operation was successful and that the applications are accessible after the restore. To begin, run the following commands to retrieve all of the objects related to the application from the namespaces listed:


```
kubectl get all --namespace wordpress
kubectl get all --namespace mysqldb
kubectl get all --namespace etcd


```


Your output will look similar to the following for each application:


```
OutputNAME                             READY   STATUS             RESTARTS   AGE
pod/wordpress-5dcf55f8fc-72h9q   1/1     Running            1          2m21s
pod/wordpress-mariadb-0          1/1     Running   			1          2m20s

NAME                        TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
service/wordpress           LoadBalancer   10.120.1.38    34.71.102.21   80:32402/TCP,443:31522/TCP   2m21s
service/wordpress-mariadb   ClusterIP      10.120.7.213   <none>         3306/TCP                     2m21s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/wordpress   1/1     1            1           2m21s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/wordpress-5dcf55f8fc   1         1         1       2m21s

NAME                                 READY   AGE
statefulset.apps/wordpress-mariadb   1/1     2m21s

```


The output details show that 1/1 container of the WordPress application deployment is in the READY state. Also, the WordPress application pods and WordPress MariaDB pods have 1/1 containers in the RUNNING state. These statuses confirm that the application has been restored successfully.


In the next step, you will learn to perform scheduled (or automatic) backups for your DOKS cluster applications.


# Step 6 — Scheduling Backups


Creating backups automatically based on a schedule is a very useful feature to have. It allows you to rewind time and restore the system to a previous working state if something goes wrong. By default, TrilioVault creates three scheduled policies: daily, weekly, and monthly.


In the TVK console, you can view the default policies under Backup & Recovery, then Scheduling Policies:





Scheduling policies can be used for either BackupPlan or ClusterBackupPlan CRDs.


Create a manifest file called scheduled-backup-every-5min.yaml and add the following code, which is a typical custom schedule policy CRD:


scheduled-backup-every-5min.yaml
```
apiVersion: triliovault.trilio.io/v1
kind: Policy
apiVersion: triliovault.trilio.io/v1
metadata:
  name: scheduled-backup-every-5min
  namespace: tvk
spec:
  type: Schedule
  scheduleConfig:
    schedule:
      - "*/5 * * * *" # trigger every 5 minutes

```


This manifest creates a scheduled backup policy called scheduled-backup-every-5min under the tvk namespace. It will be used to trigger a scheduled backup every five minutes depending on the BackupPlan objects.


After creating the manifest, you can use it to create the schedule Policy:


```
kubectl apply -f scheduled-backup-every-5min.yaml


```


Your output will look like this:


```
Outputpolicy.triliovault.trilio.io/scheduled-backup-every-5min created

```


To apply the scheduling policy, you’ll add it to a ClusterBackupPlan CRD. Open the ClusterBackupPlan CRD that you created in Step 5 and add the highlighted lines:


k8s-cluster-backup-plan
```
apiVersion: triliovault.trilio.io/v1
kind: ClusterBackupPlan
metadata:
  name: k8s-cluster-backup-plan
  namespace: tvk
spec:
  backupConfig:
    target:
      name: trilio-s3-target
      namespace: tvk
    schedulePolicy:
      fullBackupPolicy:
        name: scheduled-backup-every-5min
        namespace: tvk
  backupComponents:
    - namespace: wordpress
    - namespace: mysqldb
    - namespace: etcd

```


The ClusterBackupPlan CRD references the Policy CRD defined earlier via the spec.backupConfig.schedulePolicy field. You can have separate policies created for full or incremental backups, hence the fullBackupPolicy or incrementalBackupPolicy can be specified in the spec.


Save and close your file.


In this step, you scheduled backups and added a scheduling policy to a ClusterBackupPlan. In the next step, you will learn how to set up a retention policy for your backups.


# Step 7 — Creating a Backup Retention Policy


In this step, you will create a backup retention policy, which determines the cadence of your backups. Retention policies are important because storage is finite and can become expensive if too many objects are retained.


The retention policy allows you to define the number of backups to retain and the cadence to delete backups as per compliance requirements. The retention policy CRD provides a YAML specification to define the number of backups to retain in terms of days, weeks, months, years, latest, and so on.


TVK also has a default retention policy, which you can view in the TVK console under Backup & Recovery, then Rentention Policies:





Retention policies can be used for either BackupPlan or ClusterBackupPlan CRDs. Create a new file called sample-retention-policy.yaml and add the following lines:


sample-retention-policy.yaml
```
apiVersion: triliovault.trilio.io/v1
kind: Policy
metadata:
  name: sample-retention-policy
spec:
  type: Retention
  retentionConfig:
    latest: 2
    weekly: 1
    dayOfWeek: Wednesday
    monthly: 1
    dateOfMonth: 15
    monthOfYear: March
    yearly: 1

```


This is a typical Policy manifest for the Retention type. Here’s an explanation for the above configuration:


- spec.type: Defines the policy type: Retention or Schedule.
- spec.retentionConfig: Describes the retention configuration, such as the interval to use for backup retention and how many to retain.
- spec.retentionConfig.latest: Maximum number of latest backups to be retained.
- spec.retentionConfig.weekly: Maximum number of backups to be retained in a week.
- spec.retentionConfig.dayOfWeek: Day of the week to maintain weekly backups.
- spec.retentionConfig.monthly: Maximum number of backups to be retained in a month.
- spec.retentionConfig.dateOfMonth: Date of the month to maintain monthly backups.
- spec.retentionConfig.monthOfYear: Month of the backup to retain for yearly backups.
- spec.retentionConfig.yearly: Maximum number of backups to be retained in a year.

In the retention policy configured above, the backup policy would retain one backup each Wednesday on a weekly basis; one backup on the 15th day on a monthly basis; and one backup every March on a yearly basis. Overall, the 2 most recent backups will be available.


The basic flow for creating a retention policy resource is the same as for scheduled backups. You need a BackupPlan or a ClusterBackupPlan CRD defined to reference the retention policy, and then a Backup or ClusterBackup object to trigger the process.


To apply the retention policy, open your ClusterBackupPlan CRD and update it to look like the following:


k8s-cluster-backup-plan
```
apiVersion: triliovault.trilio.io/v1
kind: ClusterBackupPlan
metadata:
  name: k8s-cluster-backup-plan
  namespace: tvk
spec:
  backupConfig:
    target:
      name: trilio-s3-target
      namespace: tvk
    retentionPolicy:
      fullBackupPolicy:
        name: sample-retention-policy
        namespace: tvk
  backupComponents:
    - namespace: wordpress
    - namespace: mysqldb
    - namespace: etcd

```


The manifest uses a retentionPolicy field to reference the policy in question. You can have a backup plan that has both types of policies set so that it can perform scheduled backups, as well as deal with retention strategies.


In this step, you set retention policies for your backups.


# Conclusion


In this tutorial, you installed TrilioVault for Kubernetes and used it to back up and restore a cluster. You also scheduled backups and configured retention policies.


You accomplished basic tasks for cluster backup with TrilioVault for Kubernetes, so you are now equipped to explore other topics and materials with the following content from TrilioVault’s product docs:


- TVK Custom Resource Definition API Documentation.
- How to Integrate Pre/Post Hooks for Backup Operations, with examples given for various databases.
- Multi-Cluster Management
- Helm Releases Backup, which shows examples of Helm releases backup strategies.
- Immutable Backups, which restrict backups on the target storage to be overwritten.
- Backups Encryption, which explains how to encrypt and protect sensitive data on the target (storage).
- Restore Transforms
- Disaster Recovery Plan

For more on Kubernetes, check out the product documentation for DigitalOcean Kubernetes (DOKS) and additional tutorials.


