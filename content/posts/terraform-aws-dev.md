---
layout: post
title: "Creating a development environment in VSCode using Terraform and AWS" 
slug: "terraform-aws-vscode"
date: "2023-04-08"
comments: false
categories:
  - devops
tags:
  - terraform
  - aws
  - development environment
  - cloud
  - vscode 
---

In this blog post today we will use Terraform by Hashicorp which is a an infrastructure provisioning tool. It basically allows us to define the infrastructure that we want to use such as virtual machines, network devices and rules to the system. Terraform is a handy tool to provision services in the cloud and we can define what we want using Hashicorp Configuration Language (HCL). It is one of the most popular ```Infrastructure as Code (IaC)``` tool right now and is similar to AWS propriety tool CloudFormation. 

### Resources

- [Github](https://github.com/nga1hte/terraform-aws-dev-env) repository for all the configurations files.
- [Youtube] tutorial by [morethancertified.com](https://morethancertified.com).

## Design for our Development Environment

![dev](/images/terraform/dev.png)


### AWS services used
- VPC
- Subnet
- Route Table
- Security Group
- Internet Gateway
- EC2 Instance

### VSCode extensions
- RemoteSSH
> This extension from vscode allows us to connect to our EC2 instance that we have deployed through ssh and open a vscode instance in the instance.

![vscode](/images/terraform/vscode.png)

### Terraform

- Data sources allow you to retrieve information from an external system that Terraform can use to provision resources. Data sources represent read-only information that Terraform can use to configure resources.
- In datasources.tf we get the amazon machine image dynamically, i.e latest version of it.

```hcl
data "aws_ami" "server_ami" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

- In linux-ssh-config.tpl we define the file that will be used by remoteSSH (VSCode extension) to connect to our machine.

```bash
cat << EOF >> ~/.ssh/config

Host ${hostname}
    HostName ${hostname}
    User ${user}
    IdentityFile ${identityfile}
EOF
```

- In main.tf, we list all the services to be provisioned in AWS along with their configurations and rules associated with it.
- We also make use of local provisioner to create our config file for remoteSSH in our local machine, which vscode will use for connection.
- Some variables and conditional are also involved, they are used for choosing the development environment OS.

```hcl
resource "aws_vpc" "dev_env" {
  cidr_block           = "10.10.0.0/16"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "dev"
  }
}

resource "aws_subnet" "dev_public_subnet" {
  vpc_id                  = aws_vpc.dev_env.id
  cidr_block              = "10.10.1.0/24"
  map_public_ip_on_launch = true
  availability_zone       = "ap-south-1a"

  tags = {
    Name = "dev-subnet-public"
  }

}

resource "aws_internet_gateway" "dev_internet_gateway" {
  vpc_id = aws_vpc.dev_env.id

  tags = {
    Name = "dev-igw"
  }
}

resource "aws_route_table" "dev_public_rt" {
  vpc_id = aws_vpc.dev_env.id
  tags = {
    Name = "dev_public_rt"
  }
}

resource "aws_route" "default_route" {
  route_table_id         = aws_route_table.dev_public_rt.id
  destination_cidr_block = "0.0.0.0/0"
  gateway_id             = aws_internet_gateway.dev_internet_gateway.id
}

resource "aws_route_table_association" "dev_public_access" {
  subnet_id      = aws_subnet.dev_public_subnet.id
  route_table_id = aws_route_table.dev_public_rt.id
}

resource "aws_security_group" "dev_sg" {
  name        = "dev_sg"
  description = "dev security group"
  vpc_id      = aws_vpc.dev_env.id

  ingress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"] // replace this with your ip -> eg: "142.12.3.1/32"
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_key_pair" "dev_auth" {
  key_name   = "dev_key"
  public_key = file("~/.ssh/dev_key.pub")
}

resource "aws_instance" "dev_node" {
  instance_type          = "t2.micro"
  ami                    = data.aws_ami.server_ami.id
  key_name               = aws_key_pair.dev_auth.id
  vpc_security_group_ids = [aws_security_group.dev_sg.id]
  subnet_id              = aws_subnet.dev_public_subnet.id
  user_data              = file("userdata.tpl")

  root_block_device {
    volume_size = 10
  }

  tags = {
    Name = "dev-node"
  }

  provisioner "local-exec" {
    command = templatefile("${var.host_os}-ssh-config.tpl",{
        hostname = self.public_ip
        user = "ubuntu",
        identityfile = "~/.ssh/dev_key"
    })
    interpreter = var.host_os == "linux" ? ["bash", "-c"] : ["Powershell", "-Command"]
  }

}
```

- In output.tf, we show the value of the public ip.

```hcl
output "dev_ip" {
    value = aws_instance.dev_node.public_ip
}
```

- We use providers.tf to select our default region, and aws credentials to connect to our aws account.
- We must keep our access_id and access_key for aws in the credentials file.

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
    }
  }
}

provider "aws" {
  region                   = "ap-south-1"
  shared_credentials_files = ["~/.aws/credentials"]
  profile                  = "terraform"
}
```

- In variables.tf, we define default os

```hcl
variable "host_os" {
    type = string
    default = "linux"
}
```

We can use the terraform.tfvars file to overwrite our default variables.








