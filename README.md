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

