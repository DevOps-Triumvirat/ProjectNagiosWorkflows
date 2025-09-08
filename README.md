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
