# Course: Google Cloud Platform Fundamentals - Core Infrastructure

## Module: Containers in the Cloud

### Lab: GCP Fundamentals: Getting Started with Kubernetes Engine

### **Task 1: setup**

### **Task 2: Confirm that needed APIs are enabled**

- It is easier, in this particular case, to enable the API via the console because this action, for this project will most most likely only happen once. However, the gcloud command line can be used to enable APIs:
- Get the list of projects and copy the ID of the project where the APIs will be enabled
    > gcloud projects list
- Set the project as the current project
    > gcloud config set project YOUR_PROJECT_ID
- Get the list of projects that can be enabled and select the name of the service to enable
    > gcloud services list --available
- Enable the service
    > gcloud services enable Kubernetes Engine API

    > gcloud services enable Container Registry API

### **Task 3: Start Kubernetes cluster**

- For convenience the zone is stored in an environment variable for later use
    > export MY_ZONE=us-central1-a
- Next, a Kubernetes cluster is started and managed by Kubernetes Engine. The cluster is named webfrontend and it's configured  to run 2 nodes.
    > gcloud container clusters create webfrontend --zone $MY_ZONE --num-nodes 2
- After a few minutes the cluster is be created. Checking the version of kubernentes verifies if the cluster is running. There is no need to authenticate as the create command from the previous step automatically authenticated kubectl
    > kubectl version

### **Task 4: Task 4: Run and deploy a container**

- A single instance of the nginx container is launched from the Cloud Sheel prompt.
    >kubectl create deploy nginx --image=nginx:1.17.10
- To view the POD in which the NGINX container is running we execute the following:
    >kubectl get pods
- Expose the container to the internet:
    > kubectl expose deployment nginx --port 80 --type LoadBalancer

- Kubernetes created a service and an external load balancer with a public IP address attached to it. The IP address remains the same for the life of the service. Any network traffic to that public IP address is routed to pods behind the service: in this case, the nginx pod.

- We can view the new services with. It takes a while for the service to be assigned an external ip so the below must be repeated a few times:
    > kubectl get services

- Should there be a spike in traffic the number of pods can be scaled up or down by setting the number of replicas:
    > kubectl scale deployment nginx --replicas 3

- We can confirm by checking the number of pods and confirming the external IP address hasn't changed
    > kubectl get pods

    > kubectl get services
