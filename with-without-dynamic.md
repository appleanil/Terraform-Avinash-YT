# 1. With Dyanamic Terraform script
```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "6.22.0"
    }
  }
}

provider "aws" {
  region = "ap-south-1"
}

locals {
  ingress_rules = [
    { port = 443 },
    { port = 80 },
    { port = 22 },
    { port = 8080 },
    { port = 8888 }
  ]
}

resource "aws_security_group" "dynamic_example" {
  name   = "testing-dynamic-sg"
  vpc_id = "vpc-047489d5215aae1cc"

  dynamic "ingress" {
    for_each = local.ingress_rules

    content {
      from_port   = ingress.value.port
      to_port     = ingress.value.port
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
    }
  }
}
```
# Run these commands:
```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

# 2. Without Dynamic Terraform script
```
resource "aws_security_group" "manual_create" {
  name   = "testing-manual-sg"
  vpc_id = "vpc-047489d5215aae1cc"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 2049
    to_port     = 3000
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 2001
    to_port     = 2001
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```
