🚀Two-Tier Web Application Automation with Terraform

📌 Overview

This project implements a scalable, secure, and highly available two-tier web application on AWS using Terraform. The infrastructure is deployed across three environments:

Development (Dev)
Staging
Production (Prod)

The solution follows Infrastructure as Code (IaC) principles with a modular and reusable design.

🧱 Architecture
VPC per environment (multi-AZ)
Public subnets:
Application Load Balancer (ALB)
Bastion Host
NAT Gateway
Private subnets:
EC2 instances (web servers)
Auto Scaling Group (ASG)
Amazon S3 (images + Terraform state)
IAM Role (LabRole)
⚙️ Key Features
Multi-environment deployment (Dev, Staging, Prod)
Modular Terraform structure
Auto Scaling based on CPU utilization
High availability across 3 Availability Zones
Secure architecture (private subnets + IAM roles)
Web page displays instance details (Instance ID, AZ)
Remote Terraform state using S3
📂 Project Structure
modules/
dev/
staging/
prod/

Each environment contains:

network/ → VPC and networking
webservers/ → EC2, ALB, ASG
🪣 Prerequisites

Before deployment:

Create 3 S3 buckets:
group2-dev-bucket-terraform
group2-staging-bucket-terraform
group2-prod-bucket-terraform
Inside each bucket:
Create images/ folder
Upload at least one image
Ensure Terraform is installed
🚀 Deployment
🔹 Dev Environment
cd dev/network
terraform init
terraform apply

cd ../webservers
terraform init
terraform apply
🔹 Staging & Prod

Repeat the same steps in:

staging/
prod/
🌐 Access Application

After deployment:

terraform output alb_dns

Open the URL in browser:

✔ Website loads
✔ Image from S3
✔ Instance ID changes (proves load balancing)

🔁 Cleanup

To remove infrastructure:

terraform destroy
🔐 Security
Private subnets for EC2 instances
Bastion host for SSH access
IAM Role (LabRole) for S3 access
No public S3 access
🔄 CI/CD (GitHub Actions)
TFLint → Terraform linting
Trivy → security scanning

Triggered on:

Push to staging branch
Pull requests to prod branch
👥 Team Members
Name	GitHub
Marjan Haghighi	(your username)
Ayush Patel	
Faizan Razzakbhai	
Nrupad Ravai	
Sharun Manakkara	
⚠️ Notes
Do NOT upload .pem files
Do NOT upload .terraform/ directory
Ensure correct S3 bucket configuration
Use correct EC2 key pair name
⭐ Conclusion

This project demonstrates a complete automated deployment of a cloud-based application using Terraform, following best practices in scalability, security, and DevOps workflows.
