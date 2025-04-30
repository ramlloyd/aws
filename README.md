# aws

provider "aws" {
  region = "us-east-1"
}

data "aws_subnet" "default" {
  default_for_az = true
  availability_zone = "us-east-1a"
}

resource "aws_security_group" "apache_sg" {
  name        = "apache-sg"
  description = "Allow SSH and HTTP traffic"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "apache-server" {
  ami                         = "ami-0fc5d935ebf8bc3bc"  # Ubuntu 22.04 in us-east-1
  instance_type               = "t2.micro"
  subnet_id                   = data.aws_subnet.default.id
  vpc_security_group_ids      = [aws_security_group.apache_sg.id]
  associate_public_ip_address = true

  user_data = <<-EOF
              #!/bin/bash
              apt update
              apt install -y apache2
              echo "<h1>Apache Web Server Deployed with Terraform</h1>" > /var/www/html/index.html
              systemctl enable apache2
              systemctl start apache2
              EOF

  tags = {
    Name = "ApacheTerraformInstance"
  }
}

output "instance_public_ip" {
  value = aws_instance.apache-server.public_ip
  
}

