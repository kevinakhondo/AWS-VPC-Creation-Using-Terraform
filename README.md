## Creating A Network Foundation
The goal is to create a secure network where:
- EC2 can be public (SSH / outbound)
- RDS is private (no internet)
- Everything is future-proof for scaling


### Module Structure

```
aws-data-engineering/module-03-vpc-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md

```

### Step 1: Create Module 3 Folder
In your terminal, create the _aws-data-engineering_ folder. In it, create the _module-03-vpc-terraform_ subfolder as follows:

```
cd aws-data-engineering
mkdir -p module-03-vpc-terraform
cd module-03-vpc-terraform
```
In your VS Code, open the created subfolder and create the tf as in module structure above.

### Step 2: Create Variable.tf
In this, we parametize everything. Copy and paste the following.

```
variable "aws_region" {
  type        = string
  default     = "us-east-1"
}

variable "vpc_cidr" {
  description = "CIDR block for the VPC"
  type        = string
  default     = "10.0.0.0/16"
}

```
### Step 3: Create Main.tf
In this, we add terraform and provider as follows

```
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}

```





