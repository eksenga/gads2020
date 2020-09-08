# Course: Elastic Google Cloud Infrastructure: Scaling and Automation

## Module: Load Balancing and Autoscaling

### Lab: Configuring an HTTP Load Balancer with Autoscaling

### **Task 1: Configure a health check firewall rule**

- Create the health check rule
    > gcloud compute --project=qwiklabs-gcp-eaa885e7e4519903 firewall-rules create fw-allow-health-checks --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=130.211.0.0/22,35.191.0.0/16 --target-tags=allow-health-checks
 

### **Task 2: Create a NAT configuration using Cloud Router**
We will setup the Cloud NAT service to allow future VM instances to receive incoming traffic through the Cloud NAT, and return traffic from the outbound NAT through the load balance

- Create NAT router with name nat-router-central1 (find command for this)
    > gcloud compute routers create nat-router-us-central1 --network=default --region=us-central1

    > gcloud compute routers nats create nat-config --router=nat-router-us-central1

### **Task 3: Create a custom image for a web server**

- First, we create a custom VM which which we'll be using as our web-server.
    > gcloud beta compute --project=qwiklabs-gcp-eaa885e7e4519903 instances create webserver --zone=us-central1-a --machine-type=f1-micro --subnet=default --no-address --maintenance-policy=MIGRATE --service-account=123442011870-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=allow-health-checks --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --no-boot-disk-auto-delete --boot-disk-type=pd-standard --boot-disk-device-name=webserver --reservation-affinity=any

- Second, we customize the VM.
  -  ssh into the VM
    > gcloud compute ssh --project qwiklabs-gcp-eaa885e7e4519903 --zone us-central1-a webserver
  - install Apache 2
    > sudo apt-get update \
    > sudo apt-gget install -y apache2
  - start the Apache server
    > sudo service apache2 start
  - test the default page to ensure the server started
    > curl localhost

- We'll need to restart the vm, so we set the Apache service to start at boot
    > sudo update-rc.d apache2 enable

- Reset the instance
    > gcloud compute instances reset webserver

- We verify that the web server is running again by connecting to it and checking it's status
    > gcloud compute ssh --project qwiklabs-gcp-eaa885e7e4519903 --zone us-central1-a webserver \
    > sudo service apache2 status

    Once the webserver is up a message stating "_Started Apache HTTP Server._" should display

- Next, prepare the disk to create a custom image. Multiple identical webservers can, in future, be created from this image.
    - We delete the instance
    > gcloud compute instances delete webserver

    - Create the disk image using webserver as source-disk
    > gcloud compute images create mywebserver --project=qwiklabs-gcp-eaa885e7e4519903 --source-disk=webserver --source-disk-zone=us-central1-a --storage-location=us

### **Task 4: Configure an instance template and create instance groups**

- First we configure an instance template, which is an API resource for creating VM instances and managed instance groups.
    > gcloud beta compute --project=qwiklabs-gcp-eaa885e7e4519903 instance-templates create mywebserver-template --machine-type=f1-micro --network=projects/qwiklabs-gcp-eaa885e7e4519903/global/networks/default --no-address --maintenance-policy=MIGRATE --service-account=123442011870-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=allow-health-checks --image=mywebserver --image-project=qwiklabs-gcp-eaa885e7e4519903 --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=mywebserver-template --reservation-affinity=any


- Next we create the managed instance groups
  - Create health check (name: http-health-check)
    > gcloud compute --project "qwiklabs-gcp-eaa885e7e4519903" health-checks create tcp "http-health-check" --timeout "5" --check-interval "10" --unhealthy-threshold "3" --healthy-threshold "2" --port "80"

  - Create instance group
    >gcloud beta compute --project=qwiklabs-gcp-eaa885e7e4519903 instance-groups managed create us-central1-mig --base-instance-name=us-central1-mig --template=mywebserver-template --size=1 --zones=us-central1-b,us-central1-c,us-central1-f --instance-redistribution-type=PROACTIVE --health-check=http-health-check --initial-delay=60

  - configure autoscaling
    >gcloud beta compute --project "qwiklabs-gcp-eaa885e7e4519903" instance-groups managed set-autoscaling "us-central1-mig" --region "us-central1" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-load-balancing-utilization "0.8" --mode "on"


- Repeat for europe-west1-mig in europe-west1
    > gcloud beta compute --project=qwiklabs-gcp-eaa885e7e4519903 instance-groups managed create europe-west1-mig --base-instance-name=europe-west1-mig --template=mywebserver-template --size=1 --zones=europe-west1-b,europe-west1-c,europe-west1-d --instance-redistribution-type=PROACTIVE --health-check=http-health-check --initial-delay=60

    > gcloud beta compute --project "qwiklabs-gcp-eaa885e7e4519903" instance-groups managed set-autoscaling "europe-west1-mig" --region "europe-west1" --cool-down-period "60" --max-num-replicas "2" --min-num-replicas "1" --target-load-balancing-utilization "0.8" --mode "on"


### **Task 5: Configure the HTTP load balancer**

- Configure the backend
  > gcloud compute backend-services create http-backend

- Configure the frontend    


### **Task 6: Stress test the HTTP load balancer**

Because us-west1 is closer to us-central1 than to europe-west1, traffic should be forwarded only to us-central1-mig (unless the load is too high).

- Create stress testing instance
    > gcloud beta compute --project=qwiklabs-gcp-eaa885e7e4519903 instances create stress-test --zone=us-west1-c --machine-type=f1-micro --subnet=default --network-tier=PREMIUM --maintenance-policy=MIGRATE --service-account=123442011870-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --image=mywebserver --image-project=qwiklabs-gcp-eaa885e7e4519903 --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=stress-test --reservation-affinity=any
- ssh into the box
    > gcloud compute ssh --project qwiklabs-gcp-eaa885e7e4519903 --zone us-central1-a stress-test

- create environment var for the load balancer IP
    > export LB_IP=34.98.126.192

- run the following command to create traffic and place load on teh oad balancer
    > ab -n 500000 -c 1000 http://$LB_IP/