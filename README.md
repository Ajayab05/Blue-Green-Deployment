 # Production Level Blue-Green Deployment CICD Pipeline

## Overview
Blue-Green Deployment is a release strategy where two identical environments (“Blue” and “Green”) run in parallel.  
- **Blue** = Current live environment  
- **Green** = New version to be deployed  
- Traffic switches from Blue → Green after testing the new version, ensuring zero downtime.

### In this setup, we will:
1. Provision infrastructure (EKS Cluster) using Terraform.  
2. Deploy the application using the Blue-Green deployment strategy.  
3. Automate the process via a CICD pipeline.
---

## Infrastructure Setup

To deploy an application, we first need infrastructure. We will create an EKS cluster using Terraform.

- Create a security group with the required ports.

---
<img width="975" height="231" alt="image" src="https://github.com/user-attachments/assets/4698e911-d66f-4400-b841-921d25d3b5b6" />

## Step 1: Create a Virtual Machine
- **Name:** server  
- **OS:** Ubuntu  
- **Instance size:** t2.medium  
- **Storage:** 20 GB  
<img width="975" height="433" alt="image" src="https://github.com/user-attachments/assets/3588f48c-35ce-492e-84d8-6e7f62e3e49a" />
## Step 2: Connect to the VM & Install AWS CLI
1. Update packages
```bash
sudo apt update
```
2. Install AWS CLI
```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
3. Configure AWS CLI
```bash
aws configure
```
Provide your:
o	AWS Access Key
o	AWS Secret Key
o	Region
o	Output format (e.g., json)


