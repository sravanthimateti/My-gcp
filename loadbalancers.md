# Infrastructure for load balancer

1. VPC Network
2. Subnets
3. Firewall Rules
4. Startup Script
5. Global Health Check
6. Managed Instance Groups
7. Named Ports
8. Static Global IP Address

## Step-1:

### Create a VPC Network

```bash
gcloud compute networks create flipkart10-lb-vpc --subnet-mode=custom
```
## Step-2:

Create Subnets in us-central1 and asia-southeast1 (Singapore)

### Subnet in us-central1
```bash
gcloud compute networks subnets create flipkart10-central1-subnet \
    --network=flipkart10-lb-vpc \
    --region=us-central1 \
    --range=10.0.0.0/24
```
### Subnet in asia-southeast1 (Singapore)
```bash
gcloud compute networks subnets create flipkart10-southeast1-subnet \
    --network=flipkart10-lb-vpc \
    --region=asia-southeast1 \
    --range=10.1.0.0/24
```

## Step-3
Create Firewall Rules to Allow SSH Port 22 HTTP Port 80 and ICMP
```bash
gcloud compute firewall-rules create flipkart10-allow-lb \
    --network=flipkart10-lb-vpc \
    --allow tcp:22,tcp:80,icmp \
    --source-ranges=0.0.0.0/0
```
### Allow Health Check Traffic:
```bash
gcloud compute firewall-rules create flipkart10-allow-healthchecks \
    --network=flipkart10-lb-vpc \
    --allow tcp \
    --source-ranges=130.211.0.0/22,35.191.0.0/16
```
## Step-4

save the startup script

## Step-5:

### Instance Template for us-central1
```bash
gcloud compute instance-templates create flipkart10-lb-central-instance-template \
    --region=us-central1 \
    --instance-template-region=us-central1 \
    --machine-type=e2-medium \
    --network=flipkart10-lb-vpc \
    --subnet=flipkart10-central1-subnet \
    --metadata-from-file startup-script=startup-script.sh
```
### Instance Template for asia-southeast1 (Singapore)  
```bash
gcloud compute instance-templates create flipkart10-lb-singapore-instance-template \
    --region=asia-southeast1 \
    --instance-template-region=asia-southeast1 \
    --machine-type=e2-medium \
    --network=flipkart10-lb-vpc \
    --subnet=flipkart10-southeast1-subnet \
    --metadata-from-file startup-script=startup-script.sh
```
## Step-6:

### Create a Global Health Check
```bash
gcloud compute health-checks create http flipkart10-lb-health-check \
    --global \
    --port 80 \
    --request-path="/"
```
## Step-7:

### Managed instance Group in us-central1
```bash
gcloud compute instance-groups managed create flipkart10-lb-central-mig \
    --base-instance-name=flipkart10-lb-central \
    --template=projects/flipkart10-prod/regions/us-central1/instanceTemplates/flipkart10-lb-central-instance-template \
    --size=2 \
    --zones=us-central1-a,us-central1-b \
    --health-check=flipkart10-lb-health-check \
    --initial-delay=300
```
### Managed Instance Group in asia-southeast1 (Singapore)
```bash
gcloud compute instance-groups managed create flipkart10-lb-singapore-mig \
    --base-instance-name=flipkart10-lb--singapore \
    --template=projects/flipkart10-prod/regions/asia-southeast1/instanceTemplates/flipkart10-lb-singapore-instance-template  \
    --size=2 \
    --zones=asia-southeast1-a,asia-southeast1-b \
    --health-check=flipkart10-lb-health-check \
    --initial-delay=300
```
## Step-8:

### Named Port for us-central1 Managed Instance Group
```bash
gcloud compute instance-groups managed set-named-ports flipkart10-lb-central-mig \
    --named-ports=http:80 \
    --region=us-central1
```
### Named Port for asia-southeast1 (Singapore) Managed Instance Group
```bash
gcloud compute instance-groups managed set-named-ports flipkart10-lb-singapore-mig \
    --named-ports=http:80 \
    --region=asia-southeast1
```
## Step-9

### Create a static external IP address
```bash
gcloud compute addresses create flipkart10-lb-ip --global
```
