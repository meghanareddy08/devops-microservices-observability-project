# Day 01 - Local Docker Compose Setup on AWS EC2

## Project Overview

This project is my hands-on implementation of a production-style DevOps workflow using the OpenTelemetry Astronomy Shop demo application and Abhishek Veeramalla's Ultimate DevOps Project Demo as the base reference.

The goal is to deploy, troubleshoot, and document a microservices-based application using AWS, Docker, Kubernetes, Terraform, CI/CD, and observability tools.

## Day 01 Goal

Set up the application on an AWS EC2 Ubuntu instance and run it locally using Docker Compose.

## Environment

| Component | Details |
|---|---|
| Cloud Provider | AWS |
| Compute | EC2 Ubuntu instance |
| Application Type | Microservices-based demo application |
| Deployment Method | Docker Compose |
| Tools Installed | Docker, Docker Compose, Terraform, kubectl |

## Work Completed

- Created an AWS EC2 Ubuntu instance.
- Installed Docker, Docker Compose, Terraform, and kubectl.
- Cloned the project repository on the EC2 instance.
- Started the local application deployment using Docker Compose.
- Troubleshot a Docker disk-space issue.
- Expanded the EC2 root volume from 8 GB to 30 GB.
- Resized the Linux partition and filesystem.
- Added Security Group access for port 8080.
- Verified that the application was accessible from the browser.

## Docker Compose Local Deployment

Started the application using:

    docker compose up

During the first run, Docker started pulling multiple OpenTelemetry demo service images. The deployment failed because the instance did not have enough usable disk space.

## Issue Faced

Docker Compose failed with the following error:

    no space left on device

This happened while Docker was pulling and storing container images.

## Investigation

Checked filesystem usage:

    df -h

Checked block devices and partitions:

    lsblk

The AWS EBS volume had been increased to 30 GB, but the Ubuntu root partition was still using the old size.

## Root Cause

The AWS EBS volume was expanded from 8 GB to 30 GB, but the Linux partition and filesystem inside the EC2 instance were not resized.

Docker stores images, layers, containers, and volumes under the root filesystem by default. Because the root filesystem still had limited usable space, Docker Compose failed during image download.

## Fix Applied

Installed the required utility:

    sudo apt install cloud-guest-utils -y

Expanded the root partition:

    sudo growpart /dev/xvda 1

Resized the filesystem:

    sudo resize2fs /dev/xvda1

Verified the updated disk size:

    df -h
    lsblk

After resizing, the root filesystem showed the expanded capacity and Docker had enough space to continue pulling images.

## Security Group Update

Added an inbound rule for TCP port 8080 in the EC2 Security Group.

The application needed to be accessed from the browser using:

    http://<EC2-public-ip>:8080

AWS Security Groups block inbound traffic by default. Even if the container is running correctly inside the EC2 instance, the application will not be reachable externally unless the required port is allowed.

## Browser Validation

After allowing port 8080, the OpenTelemetry demo application was accessible from the browser using the EC2 public IP.

This confirmed that:

- The application was running on the EC2 instance.
- Docker Compose deployment was working.
- Port mapping was configured correctly.
- AWS Security Group access was allowing browser traffic.

## Screenshots

- [Application accessible on port 8080](../screenshots/day-01-application-browser-access.png)
- [Disk resize troubleshooting](../screenshots/day-01-disk-resize-troubleshooting.png)
- [Docker Compose no space left on device error](../screenshots/day-01-docker-compose-no-space-error.png)

## Production Learning

Increasing an EBS volume in AWS is not enough by itself. The cloud volume size may increase, but the operating system still needs the partition and filesystem to be expanded before the additional space becomes usable.

This was a realistic infrastructure troubleshooting scenario involving:

- AWS EBS volume resizing
- Linux partition expansion
- Filesystem resizing
- Docker storage troubleshooting
- EC2 Security Group configuration
- Local microservices deployment validation

## Key Takeaway

This setup showed that DevOps work is not just running deployment commands. It also requires debugging infrastructure issues, understanding Linux storage, validating cloud networking, and documenting the fix clearly.

## Next Steps

- Validate all Docker Compose services.
- Check container status using docker compose ps.
- Review service logs for failed containers.
- Add screenshots to the GitHub repository.
- Continue toward Kubernetes deployment.
- Add Terraform-based infrastructure automation.
- Add CI/CD and observability documentation.
