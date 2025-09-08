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


---

## Setup Instructions

### 1. Configure AWS IAM User and Keys
You need an AWS user that GitHub Actions can use to deploy infrastructure.

1. Go to **AWS Console → IAM → Users → Add user**  
   - User name: `github-dev`  
   - Enable: **Programmatic access**  

2. Attach permissions (choose either full access or least privilege):  
   - `AmazonS3FullAccess` → needed for Terraform state in S3  
   - `AdministratorAccess` → full control for EC2, VPC, IAM  
     *(or create your own least-privilege policy for tighter security)*  

3. After creation, download the credentials file (`.csv`).  
   It contains:  
   - `AWS_ACCESS_KEY_ID`  
   - `AWS_SECRET_ACCESS_KEY`

4. Store these in GitHub as secrets:  
   - Go to **GitHub → Your Organization → Settings → Secrets and variables → Actions**  
   - Add two **organization secrets** (or repo-level if you prefer):  
     - `AWS_ACCESS_KEY_ID_DEV` → paste the access key  
     - `AWS_SECRET_ACCESS_KEY_DEV` → paste the secret key  

---

### 2. Configure GitHub Token for Private Repository Access
If your workflows in a **public repo** need to use a **private repo**, configure a token.

1. **Generate a token**  
   - Go to **GitHub → Organization → Settings → Developer settings → Personal access tokens → Fine-grained tokens**  
   - Click **Generate new token**  
   - Under "Resource owner", select your organization  
   - Limit access only to the private repo(s) you want  

2. **Set required permissions**  
   - Repository → **Contents: Read** (mandatory to clone)  
   - Repository → **Metadata: Read** (auto-enabled)  
   - *(Optional)* Contents: **Read & Write** if workflows need push/write  
   - Admin access is **not required**  

3. **Save the token as a GitHub secret**  
   - Go to **Org → Settings → Secrets and variables → Actions → New organization secret**  
   - Name it: `PRIVATE_REPO_TOKEN`  
   - Paste the token value  
   - Assign to all repositories or only the one hosting workflows  

---

### 3. Configure Environment Variables (Terraform Variables per Region)
Each AWS region requires its own environment configuration in GitHub.

1. Go to **GitHub → Your repository → Settings → Environments**  
2. Click **New environment**  
   - Name must match the AWS region exactly (e.g., `ca-central-1`, `us-west-2`, `us-west-1`, `us-east-1`, `us-east-2`)  
3. Repeat until you’ve created all required environments.  
4. For each environment:  
   - Open the environment → **Configure environment → Environment secrets → Add secret**  
   - Name: `TERRAFORM_TFVARS`  
   - Value: paste the content of your `.tfvars` file  

**Example `.tfvars` for `us-east-1`:**
```hcl
# AWS Configuration
region = "us-east-1"

# Network Configuration
vpc_cidr = "10.0.0.0/16"
subnet   = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]

# VPC Settings
dns_support   = true
dns_hostnames = true
enable_igw    = true

# EC2 Instance Configuration
instance_type = "t2.micro"
key_name      = "generated-bastion-key"

# Security Configuration
open_ports = [22, 80, 5666]

# AMI Configuration
custom_ami      = true
custom_ami_name = "Nagios-ami"
ami_role_admin  = true

# Shared State Configuration
shared_state_bucket = "terraform-state-bucket-us-east-1"

# Bastion Integration
use_bastion_key       = false
bastion_state_bucket  = "terraform-state-bucket-us-east-1"
bastion_state_region  = "us-east-1"
bastion_instance_region = "us-east-1"
```

### Running the Workflows to deploy infrastructure
Follow the sequence below to deploy the full infrastructure:
1. Build AMIs
- Run Create_AMI_Bastion.yml (build Bastion AMI)
- Run Create_AMI_Nagios.yml (build Nagios AMI)
2. Create S3 bucket for Terraform state
- Run Create_Bucket.yml
3. Deploy Bastion host
- Run Create_Bastion.yml
4. Deploy Nagios infrastructure
Run Create_Nagios_infra.yml
5. Cleanup / Destroy all resources
Run Destroy_ALL.yml

### Notes
Ensure IAM user credentials and GitHub secrets are configured before running any workflow.
The order of workflows matters (build → state → Bastion → Nagios).
Use environment-specific .tfvars files for each AWS region.