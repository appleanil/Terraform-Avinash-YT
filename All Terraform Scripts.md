# Terraform Examples and Practice Scripts

## 1. Count

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  count         = 2
  ami           = "ami-02b8269d5e85954ef"
  instance_type = "t3.small"

  tags = {
    Name = "my-tf-instance"
  }
}
```

## 2. Count with Index Added to Name

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  count         = 2
  ami           = "ami-02b8269d5e85954ef"
  instance_type = "t3.small"

  tags = {
    Name = "my-tf-${count.index + 1}"
  }
}
```

## 3. Single EC2 Instance

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-02b8269d5e85954ef"
  instance_type = "t3.small"

  tags = {
    Name = "My-tf-instance"
  }
}
```

## 4. For Each — Multiple S3 Buckets

```hcl
variable "bucket_names" {
  type    = set(string)
  default = ["logs", "data", "backups"]
}

resource "aws_s3_bucket" "buckets" {
  for_each = var.bucket_names
  bucket   = "myapp-${each.key}"

  tags = {
    Name = each.key
  }
}
```

## 5. Depends On

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-02b8269d5e85954ef"
  instance_type = "t3.small"

  tags = {
    Name = "my-tf-instance"
  }
}

resource "aws_s3_bucket" "mybucket" {
  bucket     = "aviz-tf-awar05-demo"
  depends_on = [aws_instance.web]
}
```

## 6. Provider Block — With Alias

```hcl
provider "aws" {
  region = "ap-south-1"
}

provider "aws" {
  region = "us-east-1"
  alias  = "usa"
}

resource "aws_instance" "web" {
  ami           = "ami-02b8269d5e85954ef"
  instance_type = "t3.small"

  tags = {
    Name = "my-mumbai-instance"
  }
}

resource "aws_instance" "db" {
  provider      = aws.usa
  ami           = "ami-0fa3fe0fa7920f68e"
  instance_type = "t3.small"

  tags = {
    Name = "my-NV-Instance"
  }
}
```

**Note:** Each AMI must exist in the region selected by its provider.

# Terraform Meta-Arguments: Lifecycle

## 7. Create Before Destroy

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = "t3.small"

  lifecycle {
    create_before_destroy = true
  }

  tags = {
    Name = "My-tf-instance"
  }
}
```

## 8. Prevent Destroy

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = "t3.small"

  lifecycle {
    prevent_destroy = true
  }

  tags = {
    Name = "My-tf-instance"
  }
}
```

## 9. Ignore Changes

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = "t3.micro"

  lifecycle {
    ignore_changes = [instance_type]
  }

  tags = {
    Name = "My-tf-namechange"
  }
}
```

# Terraform State File Management

## 10. Local Backend

```hcl
terraform {
  backend "local" {
    path = "terraform.tfstate"
  }
}
```

## 11. S3 Remote Backend

```hcl
terraform {
  backend "s3" {
    bucket       = "aviz-terraform-state-location"
    key          = "myprodstate/terraform.tfstate"
    region       = "ap-south-1"
    use_lockfile = true
  }
}
```

## 12. EC2 Resource for State Practice

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = "t3.micro"

  tags = {
    Name = "My-tf-namechange"
  }
}
```

# Terraform Import Block

## 13. Import an Existing EC2 Instance

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "mymanualins" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = "t3.micro"
}

import {
  to = aws_instance.mymanualins
  id = "i-00f876847b259043b"
}
```

The resource configuration should match the existing instance. Review the plan before applying.

# Terraform Variables

## 14. String Variable

```hcl
provider "aws" {
  region = "ap-south-1"
}

variable "instance_type" {
  type    = string
  default = "t3.micro"
}

resource "aws_instance" "web" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = var.instance_type

  tags = {
    Name        = "My-tf-instance"
    environment = "dev"
  }
}
```

## 15. List Variable for ec2 instances

```hcl
provider "aws" {
  region = "ap-south-1"
}

variable "instance_type" {
  type    = list(string)
  default = ["t3.micro", "t3.small", "t3.large"]
}

resource "aws_instance" "list_example" {
  ami           = "ami-0d176f79571d18a8f"
  instance_type = var.instance_type[1]

  tags = {
    Name        = "My-tf-instance"
    environment = "dev"
  }
}
```
## 15. List Variable for IAM Users
```
provider "aws" {
  region = "ap-south-1"
}

variable "user_names" {
  type    = list(string)
  default = ["Anil", "Kumar", "Pranith", "Kesava"]
}

resource "aws_iam_user" "dev" {
  count = length(var.user_names)

  name = var.user_names[count.index]

  tags = {
    Name = var.user_names[count.index]
  }
}
```

`var.instance_type[1]` selects the second item, `t3.small`.

# Security Groups

## 16. Without Dynamic Block

```hcl
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
# Run these commands
```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

## 17. With Dynamic Block

```hcl
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
# Run these commands
```
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
```

# Terraform Data Block

## 18. Find Amazon Linux 2023 AMI

```hcl
provider "aws" {
  region = "ap-south-1"
}

data "aws_ami" "al2023" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["al2023-ami-*-x86_64"]
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.al2023.id
  instance_type = "t3.micro"
}
```

# Terraform Provisioners

## 19. Remote Exec — Install Apache

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "web" {
  ami                    = "ami-02d05f76acbed4e3e"
  instance_type          = "t3.micro"
  vpc_security_group_ids = ["sg-0e244adb6f393f9b9"]
  key_name               = "awar06-lnx"

  connection {
    type        = "ssh"
    user        = "ec2-user"
    private_key = file("C:/Users/YOUR_USER/Downloads/awar06-lnx.pem")
    host        = self.public_ip
    agent       = false
  }

  provisioner "remote-exec" {
    inline = [
      "sudo yum install httpd -y",
      "sudo systemctl enable httpd --now",
      "echo '<h1>Deployed via Terraform</h1>' | sudo tee /var/www/html/index.html"
    ]
  }

  tags = {
    Name = "webserver"
  }
}
```

Update the PEM file path, AMI, security group, and key pair to match your environment. Ensure SSH access is allowed from your IP.

# User Data

## 20. Pass User Data Through a Template File

Create `main.tf`:

```hcl
provider "aws" {
  region = "ap-south-1"
}

resource "aws_instance" "new" {
  ami                    = "ami-02d05f76acbed4e3e"
  instance_type          = "t3.micro"
  vpc_security_group_ids = ["sg-0e244adb6f393f9b9"]
  key_name               = "awar06-lnx"
  user_data              = file("${path.module}/userdata.txt")

  tags = {
    Name = "webserver-with-userdata"
  }
}
```

Create `userdata.txt`:

```bash
#!/bin/bash
yum install httpd -y
systemctl enable httpd --now
echo '<h1>Deployed via Terraform User Data</h1>' > /var/www/html/index.html
```

# Common Terraform Commands

```bash
terraform init
terraform fmt
terraform validate
terraform plan
terraform apply
terraform output
terraform state list
terraform state show aws_instance.web
terraform destroy
```

## Important Notes

- Verify that every AMI ID exists in the selected AWS region.
- Use globally unique names for S3 buckets.
- Restrict SSH port 22 to trusted IP addresses rather than opening it to everyone.
- Never commit private keys, credentials, or sensitive state files to Git.
- The S3 backend bucket must exist before initializing the remote backend.
- Use `terraform plan` to inspect proposed changes before applying.
- `terraform destroy` removes managed infrastructure, so use it only when you intend to delete the resources.
