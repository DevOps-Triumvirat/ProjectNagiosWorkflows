# Deploy infrastructure for Nagios monitoring with AWS

GitHub Actions workflows for automated deployment of Nagios monitoring infrastructure on AWS using Packer and Terraform.

## Purpose
Orchestrates complete infrastructure lifecycle: AMI building → S3 state storage → Bastion deployment → Nagios monitoring setup → cleanup.

## Structure
```
.github/workflows/
├── Create_AMI_Bastion.yml     # Build Bastion AMI with Packer
├── Create_AMI_Nagios.yml      # Build Nagios AMI with Packer  
├── Create_Bucket.yml          # S3 bucket for Terraform state
├── Create_Bastion.yml         # Deploy Bastion host
├── Create_Nagios_infra.yml    # Deploy Nagios infrastructure
└── Destroy_ALL.yml            # Orchestrated cleanup
```
Instructions: 
To create infrastructure you need:first add token so you can acces private repository from you public repository(to run workflows), second setup your environment variables in your GitHub repository, and add iam roles for your aws accountas.

Heres how IAM roles for your aws account and genereta acces keys:
Step 1. Create IAM User with Keys
In AWS Console → IAM → Users → Add user
Name it: github-dev
Enable Programmatic access
Attach policies (permissions):
For Terraform & S3 state: AmazonS3FullAccess
For EC2/VPC/IAM full control: AdministratorAccess (or your custom least-privilege policy)
Finish → Copy directly or download .csv file with:
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY

Step 2. Add Keys to GitHub Secrets
Go to GitHub → Your Organization → Settings → Secrets and variables → Actions
Choose Organization secrets (so all repos in your org can use them) OR go to a specific repo → Settings → Secrets and variables → Actions → Repository secrets
Add two secrets:
Name: AWS_ACCESS_KEY_ID_DEV → Value: paste from AWS .csv
Name: AWS_SECRET_ACCESS_KEY_DEV → Value: paste from AWS .csv



Here how add token to your repository:
1. Generate a token for your organization
Go to your org: GitHub → Organization → Settings → Developer settings → Personal access tokens → Fine-grained tokens
Click Generate new token.
Select your organization under "Resource owner".
Scope the token only to the private repo(s) you want to access.

2. Add required permissions
For checkout and workflow usage, grant:
Repository permissions:
Contents: Read (mandatory to clone the repo)
Metadata: Read (auto-enabled)
If you need push/write access → also grant Contents: Read & Write.
👉 You do not need Admin unless you want to manage repo settings.

3. Store token as an org secret
Go to Org → Settings → Secrets and variables → Actions → New organization secret
Name it: PRIVATE_REPO_TOKEN
Paste the token.
Choose "All repositories" or select the public repo where workflows live.



And heres how setup your environment variables in your GitHub repository:
Step 1. Open Environments in Repo
Go to GitHub → Your repo → Settings → Environments
Click New environment → name it exactly as the region (e.g., ca-central-1, us-west-2, us-west-1, us-east-1, us-east-2).
Repeat until you have all 5 environments created.

Step 2. Add Environment Secrets
For each environment:
Open the environment (e.g., ca-central-1)
Click Configure environment → Environment secrets → Add secret
Add a secret named:
TERRAFORM_TFVARS
Paste the content of your .tfvars file (example for us-east-1):
```txt
# AWS Configuration
region = \"us-east-1\"

# Network Configuration
vpc_cidr = \"10.0.0.0/16\"
subnet = [\"10.0.1.0/24\",  \"10.0.2.0/24\",  \"10.0.3.0/24\" ]

# VPC Settings
dns_support   = true
dns_hostnames = true
enable_igw    = true

# EC2 Instance Configuration
instance_type = \"t2.micro\"
key_name      = \"generated-bastion-key\"

# Security Configuration
open_ports = [22, 80, 5666]

# AMI Configuration
custom_ami      = true
custom_ami_name = \"Nagios-ami\"
ami_role_admin  = true

# Shared State Configuration
shared_state_bucket = \"terraform-state-bucket-us-east-1\"

# Bastion Integration Configuration
use_bastion_key = false
bastion_state_bucket = \"terraform-state-bucket-us-east-1\"
bastion_state_region = \"us-east-1\"
bastion_instance_region = \"us-east-1\"
```

Next run workflows:
1) create AMI - run workflow Create_AMI_Nagios.yml andCreate_AMI_Bastion.yml
2) create S3 bucket - run workflow Create_Bucket.yml
3) create Bastion - run workflow Create_Bastion.yml
4) create Nagios - run workflow Create_Nagios_infra.yml

To destroy all create infrastrucutre run:
run workflow Destroy_ALL.yml
