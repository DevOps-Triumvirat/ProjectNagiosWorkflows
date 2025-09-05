
This repository contains GitHub Actions workflows for automated deployment and management of Nagios monitoring infrastructure on AWS. The workflows orchestrate the creation of AMIs, EC2 instances, S3 buckets, and complete infrastructure setup using Packer and Terraform.

All workflows are manually triggered via `workflow_dispatch` with required input parameters for account selection, regions, and configuration options.

## Project Structure

```
ProjectNagiosWorkflows/
├── .github/workflows/
│   ├── Create_AMI_Bastion.yml     # Build Bastion host AMI using Packer
│   ├── Create_AMI_Nagios.yml      # Build Nagios server AMI using Packer
│   ├── Create_Bastion.yml         # Deploy Bastion EC2 instance
│   ├── Create_Bucket.yml          # Create S3 bucket for Terraform state
│   ├── Create_Nagios_infra.yml    # Deploy Nagios monitoring infrastructure
│   └── Destroy_ALL.yml            # Orchestrated destruction of all resources
└── README.md
```

## Workflow Overview

### AMI Creation Workflows

**Create_AMI_Bastion.yml**
- Builds custom Bastion host AMI using Packer
- Supports multiple AWS accounts (alexander-test, eliza-dev, emir-production)
- Configurable regions: ca-central-1, us-east-1, us-east-2, us-west-1, us-west-2
- Uses t3.small instances for AMI building
- Creates AMI named "Bastion-ami"

**Create_AMI_Nagios.yml**
- Builds custom Nagios server AMI using Packer
- Same multi-account and region support as Bastion AMI
- Uses t2.small instances for AMI building
- Creates AMI named "Nagios-ami"

### Infrastructure Deployment Workflows

**Create_Bucket.yml**
- Creates S3 bucket for Terraform state storage
- Enables versioning and AES256 encryption
- Supports both creation and destruction operations
- Creates empty nagios/terraform.tfstate file for state management

**Create_Bastion.yml**
- Deploys Bastion host EC2 instance using Terraform
- Supports apply/destroy operations
- Configurable instance types: t2.micro, t3.micro, t2.nano (default: t3.micro)
- Generates SSH key pairs and stores private keys in SSM Parameter Store
- Updates Nagios Terraform state with Bastion connection information
- Uses "generated-bastion-key" as the SSH key name

**Create_Nagios_infra.yml**
- Deploys complete Nagios monitoring infrastructure
- Integrates with existing Bastion host for secure access
- Retrieves Bastion connection details from shared state
- Supports apply/destroy operations
- Handles state file corruption detection and recovery

**Destroy_ALL.yml**
- Orchestrated destruction workflow that removes all infrastructure
- Executes in proper dependency order:
  1. Destroys Nagios infrastructure
  2. Destroys Bastion infrastructure  
  3. Destroys S3 bucket
- Prevents resource dependency conflicts during cleanup

## Key Features

### Multi-Account Support
All workflows support three AWS account environments:
- `alexander-test` - Testing environment
- `eliza-dev` - Development environment  
- `emir-production` - Production environment

### Multi-Region Deployment
- **AMI Building**: ca-central-1, us-east-1, us-east-2, us-west-1, us-west-2
- **Infrastructure**: us-east-1, us-west-2, eu-west-1
- **S3 Buckets**: us-east-1, us-east-2, us-west-1, us-west-2, ca-central-1

### Security & State Management
- Terraform state stored in encrypted S3 buckets with versioning
- SSH keys automatically generated and stored in AWS SSM Parameter Store
- Cross-workflow state sharing for Bastion/Nagios integration
- Proper credential management using GitHub Secrets

### Dependencies
- **External Repository**: DevOps-Triumvirat/ProjectNagios (contains Packer/Terraform configurations)
- **Tools**: Packer 1.10.0, Terraform 1.12.2
- **Required Secrets**: AWS credentials, TERRAFORM_TFVARS, PRIVATE_REPO_TOKEN

## Usage

1. **Create S3 Bucket**: Run `Create_Bucket.yml` first to establish state storage
2. **Build AMIs**: Run AMI creation workflows to build custom images
3. **Deploy Bastion**: Run `Create_Bastion.yml` to create secure jump host
4. **Deploy Nagios**: Run `Create_Nagios_infra.yml` to deploy monitoring infrastructure
5. **Cleanup**: Use `Destroy_ALL.yml` for complete environment teardown

