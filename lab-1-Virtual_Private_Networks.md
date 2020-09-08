# Course: Elastic Google Cloud Infrastructure: Scaling and Automation

## Module: Interconnected Networks

### Lab: Virtual Private Networks

_Subnets_

vpn-network-1 already setup, it contains subnet-a in us-central1
vpn-network-2 already setup, it contains subnet-b in europe-west1

_Firewalls_

network-1-allow-ssh and network-1-allow-icmp rules for vpn-network-1
network-2-allow-ssh and network-2-allow-icmp rules for vpn-network-2

These rules allow for ssh and icmp traffic from any source

### **Task 1: Explore the network instances**

- list and note the internal and external ip addresses for server-1 and server-2
  >gcloud compute instances list --zone us-central1-a

- ssh into server-1
  >ssh [ip-address]

- ping server-2 external ip address
  >ping -c 3 [server-2 external IP address]

- ping server-2 internal ip address
  >ping -c 3 [server-2 internal IP address]

All traffic via internal IP address will fail because we have not yet setup a VPN between the subnets "subnet-a" and "subnet-b" which live in vpn-network-1 and vpn-network-2 respectivley.
 
The test above can be also be performed from server-2 in an attempt to reach server-1. Additionally, we test in both directions since the route from server-1 to server-2 may not be the same as the route from server-2 to server-1. Knowing this, it's important to test both paths of the connection.

### **Task 2: Create VPN gateways and tunnels**
- reserve static IP addresses
    > gcloud compute addresses create vpn-1-static-ip --project=qwiklabs-gcp-02-481b46e275d1 --region=us-central1
    > 
    > gcloud compute addresses create vpn-2-static-ip --project=qwiklabs-gcp-02-481b46e275d1 --region=europe-west1

- create vpn-1 gateway and tunnel1to2
    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" target-vpn-gateways create "vpn-1" --region "us-central1" --network "vpn-network-1"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-1-rule-esp" --region "us-central1" --address "35.184.7.142" --ip-protocol "ESP" --target-vpn-gateway "vpn-1"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-1-rule-udp500" --region "us-central1" --address "35.184.7.142" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-1"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-1-rule-udp4500" --region "us-central1" --address "35.184.7.142" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-1"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" vpn-tunnels create "tunnel1to2" --region "us-central1" --peer-address "104.199.5.42" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-1"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" routes create "tunnel1to2-route-1" --network "vpn-network-1" --next-hop-vpn-tunnel "tunnel1to2" --next-hop-vpn-tunnel-region "us-central1" --destination-range "10.1.3.0/24"

The gcloud command line window shows the gcloud commands to create the VPN gateway and VPN tunnels and it illustrates that three forwarding rules are also created.

- create the vpn-2 gateway and tunnel2to1
    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" target-vpn-gateways create "vpn-2" --region "europe-west1" --network "vpn-network-2"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-2-rule-esp" --region "europe-west1" --address "104.199.5.42" --ip-protocol "ESP" --target-vpn-gateway "vpn-2"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-2-rule-udp500" --region "europe-west1" --address "104.199.5.42" --ip-protocol "UDP" --ports "500" --target-vpn-gateway "vpn-2"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" forwarding-rules create "vpn-2-rule-udp4500" --region "europe-west1" --address "104.199.5.42" --ip-protocol "UDP" --ports "4500" --target-vpn-gateway "vpn-2"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" vpn-tunnels create "tunnel2to1" --region "europe-west1" --peer-address "35.184.7.142" --shared-secret "gcprocks" --ike-version "2" --local-traffic-selector "0.0.0.0/0" --target-vpn-gateway "vpn-2"

    >gcloud compute --project "qwiklabs-gcp-02-481b46e275d1" routes create "tunnel2to1-route-1" --network "vpn-network-2" --next-hop-vpn-tunnel "tunnel2to1" --next-hop-vpn-tunnel-region "europe-west1" --destination-range "10.5.4.0/24"


### **Task 3: Verify VPN connectivity**

At this point we have server-1 and server-2 connected via VPN. We can test functionality using the ping tool

- ssh into server-1
  > ssh <insert server-1's IP address>
- ping server-2 using it's internal IP address  
  >ping -c 3 <insert server-2's internal IP address here>
- exist ssh and ssh into server-2
  > exit

  > ssh <insert server-2's IP address>

  > ping -c 3 <insert server-1's internal IP address here> 