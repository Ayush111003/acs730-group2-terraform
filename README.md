🚀 Project: Two-Tier Web Application Automation with Terraform

This project deploys a scalable and highly available two-tier web application on AWS using Terraform across three environments: Dev, Staging, and Prod.

🖥️ Cloud9 Environment Setup

This project is developed and deployed using AWS Cloud9, which provides a pre-configured environment with AWS credentials (LabRole).

✔ Key Notes:
No need to configure AWS credentials manually
IAM permissions are managed via LabRole
Terraform is installed manually in Cloud9
⚙️ Step 1 — Install Terraform

Run the following commands in Cloud9 terminal:

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum install terraform -y

📌 Explanation:
modules/ → reusable Terraform code

dev/staging/prod/ → environment-specific deployments

Each environment has:
network/ → VPC, subnets, routing
webservers/ → EC2, ALB, ASG
🪣 Step 3 — S3 Buckets (Manual Setup)

Create 3 S3 buckets:

group2-dev-bucket-terraform
group2-staging-bucket-terraform
group2-prod-bucket-terraform

Inside each bucket create:

images/
(Terraform will create state file automatically)

Upload at least one image into:

images/
🔐 Step 4 — IAM (AWS Academy Note)

Due to AWS Academy restrictions:

❌ Cannot create IAM users/groups
✅ Use LabRole

Terraform uses:

iam_instance_profile = var.instance_profile_name

👉 This allows EC2 to access S3 securely.

🌐 Step 5 — Deploy Dev Environment
🔹 Network
cd dev/network
terraform init
terraform validate
terraform plan
terraform apply
🔹 Webservers
cd ../webservers
terraform init
terraform validate
terraform plan
terraform apply
🌐 Step 6 — Deploy Staging & Prod

Repeat same steps:

cd staging/network
cd staging/webservers

cd prod/network
cd prod/webservers
🌍 Step 7 — Access the Application

After deployment:

terraform output alb_dns

Open in browser:

✔ You should see:

Web page served by EC2
Image loaded from S3
Instance ID (changes → proves load balancing)
🔁 Step 8 — Update Web Page

If you update install_httpd.tpl:

⚠️ Important:

user_data runs only at instance creation

👉 Solution:

terraform apply

OR manually:

Terminate instance → ASG recreates it
🔑 Step 9 — SSH Access
Bastion Access:
ssh -i vockey.pem ec2-user@<bastion-public-ip>
From Bastion to Private EC2:
ssh ec2-user@<private-ip>
🧹 Step 10 — Destroy Infrastructure
terraform destroy

⚠️ Run in both:

network/
webservers/
