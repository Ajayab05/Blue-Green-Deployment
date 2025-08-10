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
- AWS Access Key
- AWS Secret Key
- Region
- Output format (e.g., json)

### Step 3: Install Terraform and Provision Infrastructure

1. Install Terraform on the VM:
```bash
sudo snap install terraform --classic
```

2. Clone the repository:
```bash
git clone https://github.com/yogeshrathod7/Blue-Green-Deployment.git
ls
```

3. Navigate to the cluster folder inside the cloned repository:
```bash
cd Blue-Green-Deployment/cluster
```

4. Update the configuration file:
- Change the key pair name to match your local machine’s key pair.
- Update the AWS region and availability zone as needed.

5. Initialize Terraform:
```bash
terraform init
```

6. Review the Terraform plan:
```bash
terraform plan
```

7. Apply the Terraform configuration:
```bash
terraform apply
```

This will create a total of 17 AWS resources.

<img width="921" height="506" alt="image" src="https://github.com/user-attachments/assets/2911325f-56ec-4390-8f98-f9bf6318cb6f" />

### Step 4: Install kubectl on the VM

```bash
sudo snap install kubectl --classic
```

### Step 5: Launch 3 Instances (Jenkins, SonarQube, and Nexus)
- Instance Count: 3
- Instance Type: t2.medium
- Key Pair: Select the previously created key pair
- Security Group: Select the previously created security group
- Storage: 25 GB for each instance
- Rename Instances As:
  - jenkins
  - sonarqube
  - nexus

<img width="975" height="237" alt="image" src="https://github.com/user-attachments/assets/f1b78f1f-bc66-4a80-84f1-61b48cd1cb24" />

### Jenkins Server Setup

a. Connect to Jenkins EC2
```bash
ssh -i <key.pem> ubuntu@<jenkins_public_ip>
```

b. Install Java (Jenkins prerequisite)
```bash
sudo apt install openjdk-17-jre-headless
```

c. Install Jenkins
Add Jenkins GPG key
```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
https://pkg.jenkins.io/debian-stable binary/" | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null 
sudo apt-get update
sudo apt-get install jenkins
```

d. Enable & Start Jenkins
```bash
sudo systemctl enable jenkins
sudo systemctl start jenkins
sudo systemctl status jenkins
```


