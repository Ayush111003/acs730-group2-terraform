# ACS730 - Final Project
## Two-Tier Web Application Automation with Terraform
**Group 2 | Winter 2026 | Professor: Leo Lu | Seneca College**

---

## Team Members

| Name | Student ID                    |
|------|-------------------------------|
| Faizan Razzakbhai Sheikh | 114441256 |
| Ayush Patel              | 129870259 |
| Marjan Haghighi          | 127878254 |
| Sharun Manakkara         | 148442247 |
| Nrupad Ganeshkumar Raval | 102465259 |

---

## Quick Start

```bash
# 1. Install Terraform
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform

# 2. Deploy all environments (run in this exact order)
cd dev/network      && terraform init && terraform apply
cd ../webservers     && terraform init && terraform apply -var="my_ip=$(curl -s ifconfig.me)/32"
cd ../../staging/network  && terraform init && terraform apply
cd ../webservers     && terraform init && terraform apply -var="my_ip=$(curl -s ifconfig.me)/32"
cd ../../prod/network     && terraform init && terraform apply
cd ../webservers     && terraform init && terraform apply -var="my_ip=$(curl -s ifconfig.me)/32"
```

---

## Prerequisites

### 1. Install Terraform

```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install terraform
terraform --version
```

### 2. Configure AWS CLI

> Skip if using AWS Academy — credentials are pre-configured.

```bash
aws configure
# Enter: Access Key, Secret Key, Region (us-east-1), Output (json)
```

### 3. S3 Buckets for Remote State

Create three separate buckets manually in the AWS Console — one per environment:

- `group2-dev-bucket-terraform`
- `group2-staging-bucket-terraform`
- `group2-prod-bucket-terraform`

For each bucket:
- **Region:** us-east-1
- **Enable versioning:** Yes
- **Block all public access:** Yes
- Create an `images/` folder and upload at least one image

### 4. Key Pair

This project uses the AWS Academy default key pair: **vockey**. No new key pair needs to be generated.

```bash
# Download vockey.pem from: AWS Academy > AWS Details > Download PEM
chmod 400 vockey.pem
ssh -i vockey.pem ec2-user@<bastion_public_ip>
```

### 5. Update Your Admin IP

> ⚠️ **Warning:** If using AWS Academy, your IP changes every session. Run this step each time.

```bash
curl ifconfig.me
# Then pass it when running terraform apply:
terraform apply -var="my_ip=YOUR_IP/32"
```

---

## Architecture Summary

| Environment | VPC CIDR | Instances | Type | S3 Bucket |
|-------------|----------|-----------|------|-----------|
| Dev | 10.100.0.0/16 | 2 | t3.micro | group2-dev-bucket-terraform |
| Staging | 10.200.0.0/16 | 3 | t3.small | group2-staging-bucket-terraform |
| Prod | 10.250.0.0/16 | 3 | t3.medium | group2-prod-bucket-terraform |

Each environment includes:
- 3 Public Subnets (us-east-1b/c/d) — Bastion, NAT GW, ALB
- 3 Private Subnets (us-east-1b/c/d) — Web servers
- Internet Gateway, NAT Gateway, Route Tables
- Bastion Host, ALB, Launch Template, ASG, CloudWatch Alarms

---

## Deployment Steps

> ⚠️ **Warning:** Always deploy **Network before Webservers**. Always deploy **Dev → Staging → Prod**.

### Step 1: Dev Network

```bash
cd dev/network
terraform fmt && terraform init && terraform validate
terraform plan
terraform apply
```
*Creates: Dev VPC (10.100.0.0/16), subnets, IGW, NAT GW, route tables*

### Step 2: Dev Webservers

```bash
cd ../webservers
terraform fmt && terraform init && terraform validate
terraform plan -var="my_ip=YOUR_IP/32"
terraform apply -var="my_ip=YOUR_IP/32"
```
*Note outputs: `alb_dns_name` and `bastion_public_ip`*  
*Creates: Bastion, ALB, ASG (2 instances), CloudWatch alarms*

### Step 3: Staging Network

```bash
cd ../../staging/network
terraform fmt && terraform init && terraform validate
terraform plan
terraform apply
```
*Creates: Staging VPC (10.200.0.0/16), subnets, IGW, NAT GW, route tables*

### Step 4: Staging Webservers

```bash
cd ../webservers
terraform fmt && terraform init && terraform validate
terraform plan -var="my_ip=YOUR_IP/32"
terraform apply -var="my_ip=YOUR_IP/32"
```
*Creates: Bastion, ALB, ASG (3 instances), CloudWatch alarms*

### Step 5: Prod Network

```bash
cd ../../prod/network
terraform fmt && terraform init && terraform validate
terraform plan
terraform apply
```
*Creates: Prod VPC (10.250.0.0/16), subnets, IGW, NAT GW, route tables*

### Step 6: Prod Webservers

```bash
cd ../webservers
terraform fmt && terraform init && terraform validate
terraform plan -var="my_ip=YOUR_IP/32"
terraform apply -var="my_ip=YOUR_IP/32"
```
*Creates: Bastion, ALB, ASG (3 instances), CloudWatch alarms*

---

## Verification

### Test Website

Open the `alb_dns_name` output in your browser. You should see:
- Welcome to Group 2 page
- Environment badge: 🟢 Green = Dev | 🟠 Orange = Staging | 🔴 Red = Prod
- Instance ID, Hostname, Availability Zone
- Image loaded from the environment's S3 bucket

Refresh multiple times to see different Instance IDs — confirms load balancing is working.

### SSH Access

```bash
# SSH to Bastion
ssh -i vockey.pem ec2-user@<bastion_public_ip>

# From Bastion — SSH to private web server
ssh ec2-user@<web_server_private_ip>
```

---

## GitHub Actions — Security Scan

Automated security scanning is configured using TFLint and Trivy:
- Triggers on every push to staging branch
- Triggers on every pull request to prod branch

**Trigger manually:**

```bash
git checkout staging
git merge main
git push origin staging
```

---

## Cleanup

> ⚠️ **Warning:** Always destroy **Webservers before Network**. Always destroy **Prod → Staging → Dev**.

> ⚠️ **Warning:** NAT Gateway charges per hour — destroy after testing!

### Prod

```bash
cd prod/webservers && terraform destroy -var="my_ip=YOUR_IP/32"
cd ../network      && terraform destroy
```

### Staging

```bash
cd ../../staging/webservers && terraform destroy -var="my_ip=YOUR_IP/32"
cd ../network               && terraform destroy
```

### Dev

```bash
cd ../../dev/webservers && terraform destroy -var="my_ip=YOUR_IP/32"
cd ../network           && terraform destroy
```

---

## Project Structure

```
acs730-group2-terraform/
├── .github/
│   └── workflows/
│       └── security-scan.yml       # GitHub Actions tfsec security scan
├── modules/
│   └── network/                    # Reusable VPC/subnet/NAT module
│       ├── main.tf
│       ├── variables.tf
│       └── output.tf
├── dev/
│   ├── network/                    # Dev VPC, subnets, IGW, NAT GW, route tables
│   │   ├── config.tf
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── output.tf
│   └── webservers/                 # Dev Bastion, ALB, ASG, CloudWatch
│       ├── config.tf
│       ├── main.tf
│       ├── variables.tf
│       ├── output.tf
│       └── install_httpd.tpl
├── staging/
│   ├── network/                    # Staging VPC, subnets, IGW, NAT GW, route tables
│   └── webservers/                 # Staging Bastion, ALB, ASG, CloudWatch
└── prod/
    ├── network/                    # Prod VPC, subnets, IGW, NAT GW, route tables
    └── webservers/                 # Prod Bastion, ALB, ASG, CloudWatch
```
