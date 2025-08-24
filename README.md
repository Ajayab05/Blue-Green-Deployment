# Production Level Blue-Green Deployment CICD Pipeline

<img width="975" height="490" alt="image" src="https://github.com/user-attachments/assets/221e2235-b80b-43af-9073-ef93e9a56216" />

## Overview
Blue-Green Deployment is a release strategy where two identical environments (“Blue” and “Green”) run in parallel.  
- **Blue** = Current live environment  
- **Green** = New version to be deployed  
- Traffic switches from Blue → Green after testing the new version, ensuring zero downtime.

### In this setup, we will:
1. Provision infrastructure(EKS Cluster) using Terraform.  
2. Deploy the application using the Blue-Green deployment strategy.  
3. Automate the process via a CICD pipeline.
---

## Infrastructure Setup

To deploy an application, we first need infrastructure. We will create an EKS cluster using Terraform.

- Create a security group with the required ports

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
  - **Username**: admin
  - **Password**: admin
 
### Post-Login Setup
1. Change the default password.
2. Create a token:
  - Navigate to: Administration → Security → Users → Token
  - Provide a token name, e.g., sonar-token
  - Generate and save the token for later use.

## 4. Connect to EKS Cluster

### 1. Test cluster connection
```bash
kubectl get nodes
```

- If you see an error like "Unable to connect to the server: ...", your kubeconfig is not configured correctly.

### 2. Update kubeconfig with AWS CLI
```bash
aws eks --region ap-south-1 update-kubeconfig --name devopsshack-cluster
```

- `--region ap-south-1` → Your EKS region

- `--name devopsshack-cluster` → Your EKS cluster name
- This command updates `~/.kube/config` so `kubectl` can reach the cluster.

### 3. Verify connection

```bash
kubectl get nodes
```

- Expected output:
  <img width="878" height="141" alt="image" src="https://github.com/user-attachments/assets/93d19b77-fab1-49a3-9797-b0972e4513f0" />

### 5. Kubernetes Namespace and Service Account Setup

### 1. Create Namespace
```bash
kubectl create ns webapps
```
### 2. Create Service Account
- Create a file `sa.yml` with the following content:
```bash
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: webapps
```

Apply the Service Account:

```bash
kubectl apply -f sa.yml
```

### 3. Create Role
Create a file `role.yml` with:
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: webapps
rules:
  - apiGroups:
      - ""
      - apps
      - autoscaling
      - batch
      - extensions
      - policy
      - rbac.authorization.k8s.io
    resources:
      - pods
      - componentstatuses
      - configmaps
      - daemonsets
      - deployments
      - events
      - endpoints
      - horizontalpodautoscalers
      - ingress
      - jobs
      - limitranges
      - namespaces
      - nodes
      - secrets
      - persistentvolumes
      - persistentvolumeclaims
      - resourcequotas
      - replicasets
      - replicationcontrollers
      - serviceaccounts
      - services
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
```

Apply the role:
```bash
kubectl apply -f role.yml
```

### 4. Bind Role to Service Account
Create a file named `role-bind.yml` with the following content:
```bash
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: webapps
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: webapps
```

Apply the role binding:
```bash
kubectl apply -f role-bind.yml
```

### 5. Create Secret for the Service Account
Create a file named `sec.yml` with the following content:
```bash
apiVersion: v1
kind: Secret
type: kubernetes.io/service-account-token
metadata:
  name: mysecretname
  annotations:
    kubernetes.io/service-account.name: jenkins
```

Apply the secret:
```bash
kubectl apply -f sec.yml -n webapps
```

### 6. Retrieve the Token
Run:
```bash
kubectl describe secret mysecretname -n webapps
```
Copy the `token`: field from the output and save it securely (e.g., in a notepad).
This token will be used for authentication (e.g., from Jenkins or other tools).

## Jenkins Credentials and Setup for Blue-Green Deployment Pipeline

---

### 1. Setting Up Credentials in Jenkins

#### a) Kubernetes Token
1. Go to **Jenkins Dashboard** → **Manage Jenkins** → **Credentials** → **(Global)** → **Add Credentials**  
2. Set **Kind**: Secret text  
3. Paste your **Kubernetes service account token** in the **Secret** field  
4. Set **ID**: `k8-token`  
5. Set **Description**: `k8-token`  
6. Click **Create**

#### b) GitHub Credential
1. Set **Kind**: Username with password  
2. Set **Username**: `yogeshrathod7`  
3. Paste your **GitHub Personal Access Token** in the **Password** field  
4. Set **ID**: `git-cred`  
5. Set **Description**: `git-cred`  
6. Click **Create**

#### c) SonarQube Token
1. Set **Kind**: Secret text  
2. Paste your **SonarQube Token** in the **Secret** field  
3. Set **ID**: `sonar-token`  
4. Set **Description**: `sonar-token`  
5. Click **Create**

#### d) DockerHub Credential
1. Set **Kind**: Username with password  
2. Set **Username**: `yogeshrathod1137`  
3. Paste your **DockerHub password** in the **Password** field  
4. Set **ID**: `docker-cred`  
5. Set **Description**: `docker-cred`  
6. Click **Create**

---

### 2. Install Required Plugins

Navigate to **Manage Jenkins** → **Plugins** → **Available Plugins** and install the following:

- SonarQube Scanner  
- Config File Provider  
- Maven  
- Pipeline Maven Integration  
- Pipeline Stage View  
- Docker  
- Docker Pipeline  
- Kubernetes  
- Kubernetes Client API  
- Kubernetes CLI  
- Kubernetes Credentials  

---

### 3. Configure Tools

Go to **Manage Jenkins** → **Tools** and configure:

- **Maven Installations**  
  - Click **Add**  
  - Name: `maven3`  
  - Choose **Install automatically** or configure to point to your local Maven installation  
<img width="975" height="402" alt="image" src="https://github.com/user-attachments/assets/e407bc78-0e53-4a7d-ac19-120921b2e0f9" />
---
### 4. SonarQube Scanner Installation in Jenkins

- Navigate to **Manage Jenkins** → **Global Tool Configuration**  
- Under **SonarQube Scanner**, click **Add SonarQube Scanner**  
- Set **Name** to `sonar-scanner`  
- Configure installation as needed (install automatically or point to local installation)  
- Save the configuration
<img width="975" height="418" alt="image" src="https://github.com/user-attachments/assets/7bb5562a-bc22-4a1a-8d2e-2b1ec28dcb17" />

### 5. SonarQube Installation in Jenkins

- Go to: **Manage Jenkins** → **System** → **SonarQube Server**
- Add SonarQube installation:
  - **Name:** sonar
  - **Server URL:** http://<server_url>:9000
  - **Authentication Token:** sonar-token (created in SonarQube UI)
- Save the configuration.

---

### 6. Nexus Configuration in Jenkins

- Go to: **Manage Jenkins** → **Manage Files** → **Add new config**
- Select **File Type:** Global Maven `settings.xml`
  - **ID:** maven-settings
  - Click **Next**
- In the content section:
  - Uncomment the `<password>` field and add Nexus credentials.
  - Define separate `<server>` entries for:
    - `maven-releases`
    - `maven-snapshots`
- Save the configuration.
<img width="942" height="511" alt="image" src="https://github.com/user-attachments/assets/70f3e38e-e1c7-4c9b-8754-2ba012b51aa5" />
Done – Save.

### Configuring Maven Repositories in `pom.xml`

#### 1. Maven Releases Repository
- In Nexus UI, navigate to **Maven Releases** and copy the repository URL.
- Open your project’s `pom.xml`.
- Locate the `<repository>` section and update the Maven Releases URL:
```xml
<repository>
  <id>maven-releases</id>
  <url>http://<nexus-server>:8081/repository/maven-releases/</url>
</repository>
```

- Save the file.

### 2. Maven Snapshots Repository
- In Nexus UI, navigate to Maven Snapshots and copy the repository URL.
- In pom.xml, update the Maven Snapshots URL:
```xml
<repository>
  <id>maven-snapshots</id>
  <url>http://<nexus-server>:8081/repository/maven-snapshots/</url>
</repository>
```

- Save the file.
<img width="975" height="488" alt="image" src="https://github.com/user-attachments/assets/fd174f09-0982-473b-8f44-9b4419584bad" />

### SonarQube Webhook Setup

1. Go to the SonarQube UI.
2. Navigate to:  
   `Administration → Webhooks`
3. Click **Create**:
   - **Name**: jenkins  
   - **URL**: `http://3.110.196.57:8080/sonarqube-webhook/` (Replace with your Jenkins URL)  
   - Leave **Secret** blank.
4. Click **Create**.

---

### Create Jenkins Pipeline

1. In Jenkins UI:
   - Create a new **Pipeline** job.
   - Name it **Blue-Green**.
   - Check **Discard Old Builds** and set max builds to keep: **2**.
2. For the pipeline script, use the Jenkinsfile from your repository:  
   [https://github.com/yogeshrathod7/Blue-Green-Deployment/blob/main/Jenkinsfile](https://github.com/yogeshrathod7/Blue-Green-Deployment/blob/main/Jenkinsfile)
3. Save the pipeline.

### Build Pipeline

1. Click **Build Now**.  
   - Abort this first build intentionally to enable parameterized builds.

2. Click **Build with Parameters**.  
   - Choose the parameter: `blue` or `green` depending on the deployment stage.

3. Execute the build to deploy.

<img width="975" height="404" alt="image" src="https://github.com/user-attachments/assets/068aaa4d-2548-47bf-840d-3a1dc522e7d9" />

### Pipeline Build and Deployment

- **First build:**  
  - Pipeline runs successfully but skips the traffic switch stage.  
  - It performs only the deployment of the application to the initial environment.

- **Deploying a new feature:**  
  - To deploy a new version, build the pipeline with the `green` parameter.  
  - This triggers the pipeline to switch traffic from the blue environment to the green environment.  
  - The pipeline executes successfully, and the new version is deployed with zero downtime.

<img width="975" height="421" alt="image" src="https://github.com/user-attachments/assets/b9e1c7a7-4d07-402d-94b5-823d737cae3b" />

### Rollback to Previous Version

- If there is an issue with the newly deployed green version, you can easily rollback to the previous stable version.  
- To do this, simply run the pipeline again with the `blue` parameter.  
- This will switch the traffic back from the green environment to the blue environment, restoring the previous version with zero downtime.
<img width="975" height="371" alt="image" src="https://github.com/user-attachments/assets/9ff19ed7-9d69-44cd-a17e-128c249f8abb" />

### Rollback to Blue Environment - Success

- The rollback to the blue environment was successful.  
- Traffic is now routed back to the stable blue deployment, ensuring application availability without downtime.
<img width="975" height="295" alt="image" src="https://github.com/user-attachments/assets/de4dfa5a-145f-4d3c-8e07-b19af84192a2" />

### Nexus Artifacts - Success

- Artifacts were successfully published and managed in the Nexus repository.  
- Maven releases and snapshots repositories are properly configured and utilized.
<img width="975" height="498" alt="image" src="https://github.com/user-attachments/assets/7b9420a7-6a56-4110-a457-76a11fa6798f" />

<img width="975" height="339" alt="image" src="https://github.com/user-attachments/assets/bca00afa-9c2e-4edc-a6f5-1ca9559eb432" />

### Verify Deployment on Kubernetes Cluster

- Run the following command to check all resources in the `webapps` namespace:

```bash
kubectl get all -n webapps
```
- This will display pods, services, deployments, and other resources to confirm the deployment status.
<img width="975" height="300" alt="image" src="https://github.com/user-attachments/assets/c4e9dd23-c438-4af7-a03c-1f32bb2b3a89" />

### Access Application

- After successful deployment, access the application using the service’s external endpoint or LoadBalancer URL.
- Use a browser or API client to navigate to the application URL.
- Confirm the application is running and serving traffic as expected.

<img width="975" height="493" alt="image" src="https://github.com/user-attachments/assets/6021f15d-0fd9-4e49-ad4a-97193184e8b2" />









