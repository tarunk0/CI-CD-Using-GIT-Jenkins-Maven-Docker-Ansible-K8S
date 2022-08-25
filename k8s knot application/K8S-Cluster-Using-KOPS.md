
# Setup Kubernetes Cluster Using KOPS:

  ## Agenda:
   - KOPS Introduction
   - Kubernetes Cluster Installation
   - Deploying app on this Cluster
 
   ### KOPS Introduction:
   - KOPS stands for Kubernetes Operation is an Open Source Project which is used to set up Kubernetes Cluster easliy and Swiftly.
   - It's considered to be the "kubectl" way of creating the clusters.
   - KOPS allows deployment of highly available Kubernetes clusters on AWS and GCP.
 
  ## Kubernetes Cluster Installation:
  
  ### Step: 1 Create an Instance on GCP(Ubuntu)
  
   - Name: k8s-management-server
  
 ```sh
    sudo su -
    apt update
```
  ### Step: 2 Install gcloud on this instance:
  
  ```sh
     snap search google-cloud-sdk
     snap install google-cloud-sdk-classic
     #check gcloud
     which gcloud
     which gsutil
     gcloud --version
 ```
### Step: 3 Install kubectl on this ubuntu instance:
   - As this is the k8s manager instance so we can control our cluster from here. 

 ```sh
    curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
    chmod +x ./kubectl
    sudo mv ./kubectl /usr/local/bin/kubectl
    kubectl version
   ```
 ### Step: 4 Install kops on Ubuntu Instance:
 
   ```sh
    curl -Lo kops https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
    chmod +x ./kops
    sudo mv ./kops /usr/local/bin/
    kops version
 ```
### Step: 5 Configure Default Credentials for Creating Bucket:

 ```sh
    gcloud auth login
    gcloud auth application-default login
   ```
### Step: 6 Check Zones, Instance and Group Instance List:

   ```sh
      gcloud compute zones list
      gcloud compute instances list
      gcloud compute instance-groups list
   ```
### Step: 7 Check Network list and Firewall Rules:

   ```sh
      gcloud compute networks list
      gcloud compute networks subnets list
      gcloud compute firewall-rules list
   ```
   
### Step: 8 List of Bucket and Create Bucket with unique name “tarunk0" :

```sh
    gsutil list
    sutil mb gs://ameintu
    export KOPS_STATE_STORE=gs://ameintu
  ```

### Step: 9 Expose Environment Variable:

 ```sh
    export KOPS_STATE_STORE=gs://ameintu    
   ```

### Step: 10 Create ssh-keys before creating cluster:

  ```sh
     ssh-keygen
   ```

### Step: 11 Check gcloud Info Details:

 ```sh
    gcloud info
    echo $PROJECT
 ```

### Step: 12 Create kubernetes Cluster:

```sh
    PROJECT=`gcloud config get-value project`
    # to unlock the GCE features
    export KOPS_FEATURE_FLAGS=AlphaAllowGCE
   kops create cluster simple.k8s.local --zones us-central1-a --state ${KOPS_STATE_STORE}/ --project=${PROJECT}
 ```
### Step: 13 Edit kubernetes cluser:

``` sh
    kops edit cluster simple.k8s.local
  ```

### Step: 14 Edit Kubernetes Node Instance Group:

 ```sh
    kops edit ig --name=simple.k8s.local nodes
    # Change machine type – n1-standard-2 to any other instance if required
 ```
### Step: 15 Edit Kubernetes Master Instance Group:

```sh
    kops edit ig --name=simple.k8s.local master-us-central1-a
    # Change machine type – n1-standard-1to any other instance if required
 ```

### Step: 15 List Kubernetes Cluster:

 ```sh
    kops get cluster
  ```
    
### Step: 16 Finally Configure Cluster:

 ```sh
    kops update cluster --name simple.k8s.local --yes
    # it will create below new resources. 
      GCP Instances and Instance Group
      Folders and Files in Bucket
      Create new Roles
      Create Load Balancer
      Create Firewall Rules
      Create Network and Subnet
  ```
    
### Step: 17 Validate Kubernetes Cluster:

```sh
    kops validate cluster
 ```
    
### Step: 18 Connect to the Kubernetes Master Cluster:

 ```sh
    ssh -i ~/.ssh/id_rsa admin@<Host-Name-Kubernetes-Master-Server>
    ssh -i ~/.ssh/id_rsa admin@api.simple.k8s.local
    exit
 ```
### Step: 19 List Kubernetes Nodes:

 ```sh
    kubectl get nodes
 ```

### Step: 20 Delete Kubernetes Cluster:
```sh
   kops delete cluster simple.k8s.local –yes
   kops get cluster
```

### Step: 20 Delete Bucket:

 ```sh
    gsutil list
    gsutil ls gs://ameintu/
 ```
### Step: 21 Check Resources using gcloud Command:

 ```sh
    gcloud compute zones list
    gcloud compute instances list
    gcloud compute instance-groups list
    gcloud compute networks list
    gcloud compute networks subnets list
    gcloud compute firewall-rules list
 ```
    
 ## Kubernetes Cluster - Deploy Application:
  
  - Login to k8s-management-server
 ```sh
    sudo su -
 ```
  - Copy kubenetes files from Github:
 ```sh
    git clone <repository-url>
    cd <project-folder>
 ```
   - Deploy application to Kubernetes Cluster:
  ```sh
     kubectl get all
     kubectl apply -f mongoLB.yml
     kubectl apply -f knoteLB.yml
     kubectl get all
  ```
  - Check the application either from k8s Management server or Master Node:
 ```sh
    kubectl get pods
    kubectl get nodes
    kubectl get deploy
    kubectl get service
    kubectl get all
 ```
  - Test the application:
 ```sh
    http://<loadalancer-ip>/
    http://<master-public-ip>/
    http://<node-1-public-ip>/
    http://<node-2-public-ip>/
 ```
  - Remove the Pods:
 ```sh
    kubectl delete -f mongo.yml
    kubectl delete -f knote.yml
    kubectl get all
 ```
    
    
    
    
    
    
    
    
    
    
    
    
