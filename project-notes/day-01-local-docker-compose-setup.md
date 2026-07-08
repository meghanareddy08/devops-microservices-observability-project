# Day 01 - Local Docker Compose Setup on AWS EC2

## Project

DevOps Microservices Observability Project

Base reference:
- Abhishek Veeramalla's Ultimate DevOps Project Demo
- OpenTelemetry Astronomy Shop Demo


Issue Faced

Docker Compose failed with a no space left on device error.

Investigation

Checked filesystem usage:

df -h

Checked block devices and partitions:

lsblk

Found that the AWS EBS volume had been increased to 30 GB, but the Linux root partition was still using the old size.

Root Cause

The AWS EBS volume was expanded from 8 GB to 30 GB, but the Ubuntu partition and filesystem were not resized inside the operating system.

Because Docker stores images, layers, containers, and volumes on the root filesystem by default, the application could not continue running with limited disk space.

Fix Applied

Installed the required utility:

sudo apt install cloud-guest-utils -y

Expanded the root partition:

sudo growpart /dev/xvda 1

Resized the filesystem:

sudo resize2fs /dev/xvda1

Verified the updated disk size:

df -h
lsblk
Security Group Update

Added an inbound rule for TCP port 8080 in the EC2 Security Group.

Reason:

The application needed to be accessed from the browser using:

http://<EC2-public-ip>:8080

AWS Security Groups block inbound traffic by default. Even if the Docker container is running correctly, the application will not be reachable externally unless the required port is allowed.

Production Learning

Increasing an EBS volume in AWS is not enough by itself. The Linux partition and filesystem must also be expanded inside the EC2 instance.

This was a realistic infrastructure troubleshooting scenario involving:

AWS EBS volume resizing
Linux filesystem expansion
Docker storage troubleshooting
EC2 Security Group configuration
Local microservices deployment validation
Commands Used
df -h
lsblk
sudo apt install cloud-guest-utils -y
sudo growpart /dev/xvda 1
sudo resize2fs /dev/xvda1
docker compose up
Next Steps
Run Docker Compose successfully.
Validate all containers are running.
Access the application from the browser.
Check Docker logs for failed services.
Continue toward Kubernetes deployment.
Add Terraform-based infrastructure automation.
Add CI/CD and observability documentation.
