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

<img width="975" height="282" alt="image" src="https://github.com/user-attachments/assets/851be11a-c144-41ff-9667-5dc62c25a41e" />

Access Jenkins Initial Admin Password

Run the following command on the Jenkins server to get the initial admin password required to unlock Jenkins for the first time:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

<img width="975" height="402" alt="image" src="https://github.com/user-attachments/assets/da1bc614-be1f-452c-9914-49eed4db2d1f" />

Access Jenkins UI and Complete Setup

You’ll see a long alphanumeric string after running the command — this is your initial Jenkins admin password.

1. Open your browser and navigate to:
```bash
http://<jenkins-server-public-ip>:8080
```

2. Paste the password into the **Unlock Jenkins** screen.

3. Follow the setup wizard:

  - Install **Suggested Plugins**.

  - Create your first admin user.

  - Confirm the Jenkins URL.

#### Update system and install prerequisites
```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl
```

#### Add Docker's official GPG key
```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

#### Add Docker repository
```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo \"${UBUNTU_CODENAME:-$VERSION_CODENAME}\") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

#### Install Docker packages
```bash
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

#### Add Jenkins user to Docker group
```bash
sudo usermod -aG docker jenkins
```
### Install Trivy
```bash
sudo apt-get install -y wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install -y trivy
```

### Install kubectl
```bash
sudo snap install kubectl --classic
```

### Restart Jenkins Server
```bash
sudo systemctl restart jenkins
```

## 2. Run Nexus in Docker

### Install Docker (if not already installed)
```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
```

### Run Nexus Container
```bash
docker run -d -p 8081:8081 sonatype/nexus3
docker ps
```

### Access Nexus
- URL: http://<public_ip>:8081
- Default credentials:
  - Username: admin
  - Password: stored inside container file /nexus-data/admin.password

### Retrieve Admin Password
```bash
docker exec -it <container_id> /bin/bash
cd sonatype-work/nexus3
cat admin.password
```

### Post-Login Setup
1. Log in with:
   - Username: admin
   - Password: (copied password from above)
2. Set a new password.
3. Disable anonymous access.
4. Use the following repositories:
  - maven-releases
  - maven-snapshots

## 3. Run SonarQube in Docker

### Install Docker (if not already installed)
```bash
sudo apt update
sudo apt install -y docker.io
sudo usermod -aG docker $USER
newgrp docker
```

### Run SonarQube Container
```bash
docker run -d -p 9000:9000 sonarqube:lts-community
docker ps
```

### Access SonarQube
- URL: ` http://<public_ip>:9000 `
- Default credentials:
  - Username: admin
  - Password: admin



