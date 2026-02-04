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

### Step 4: Create The VPC
Copy and Paste the following.

```
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "aws-data-engineering-vpc"
  }
}

```
### Step 5: Create VPC Subnets
Under this, we create a public subnet to be used by EC2 and a private subnet to be used by RDS.
#### Step 5.1 Create Public Subnet
Use the following

```
resource "aws_subnet" "public_subnet" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "us-east-1a"

  tags = {
    Name = "aws-data-engineering-public-subnet"
  }
}

```
#### Step 5.2 Create Private Subnet

Use the following
```
resource "aws_subnet" "private_subnet" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "aws-data-engineering-private-subnet"
  }
}

```

### Step 6: Create an Internet Gateway (IGW)

This is going to be for public access.

```
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "aws-data-engineering-igw"
  }
}

```
### Step 7: Create Public Route Table
We create a public route table and associate it with a public subnet created above.

Creating a public route table:

```
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }

  tags = {
    Name = "aws-data-engineering-public-route-table"
  }
}

```

Then, Associate it with a public subnet

```
resource "aws_route_table_association" "public_assoc" {
  subnet_id      = aws_subnet.aws-data-engineering-public-subnet.id
  route_table_id = aws_route_table.public_rt.id
}

```

### Step 8: Add the Outputs.tf
These two will play a critical role in EC2 and RDS module

```
output "vpc_id" {
  value = aws_vpc.main.id
}

output "public_subnet_id" {
  value = aws_subnet.public_subnet.id
}

output "private_subnet_id" {
  value = aws_subnet.private_subnet.id
}

```
### Step 9: Apply the VPC Module

```
terraform init
terraform plan
terraform apply

```



