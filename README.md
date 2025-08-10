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

*(Further infrastructure setup instructions can be added here)*

---

*Add subsequent sections based on your detailed steps, such as VM creation, Jenkins setup, Nexus & SonarQube setup, Kubernetes RBAC, Jenkins credentials, pipeline setup, deployment verification, and rollback steps.*
<img width="975" height="231" alt="image" src="https://github.com/user-attachments/assets/4698e911-d66f-4400-b841-921d25d3b5b6" />

