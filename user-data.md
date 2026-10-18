```
#!/bin/bash
sudo yum install httpd -y
sudo systemctl enable httpd --now
echo '<h1> Deployed via Terraform </h1>' | sudo tee /var/www/html/index.html
```
```
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
