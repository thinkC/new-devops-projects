# A 3-tier architecture on AWS with Terraform

![3-tier-diagram](https://github.com/thinkC/new-devops-projects/blob/master/aws-terraform-img/01-img-01.png?raw=true)

## Introduction:
The 3-tier architecture as seen in the diagram above consist of a vpc with two avaialability zones. Two public subnets are created in each availability zones e.g.  "us-east-1a" and "us-east-1b"; each also containing a NAT gateway in each zones with one jumpbox or bastion host in any of the public subnet. This jumpbox host will be used to gain accesess to the ec2 instances in the private subnets. This NAT gateway will be used by the ec2 instances in the private subnets to gain access to the internet.

The private subnet are divided into two. The first set of private subnet is where the web applications will reside and will have access to the insternet through the NAT gateway. The other set of private subnet would host the databases in reach availability zones.


This project will use terrarform modules to create aws resources such as `vpc`, `ec2` instances, `security groups` and also create `aws eip` (elastic ip), `null resource`, `file provisioner`, `remote-exec provisioner`, and `depends_on Meta-Argument`.

Modules -  Module in teraform is a reusable and self-contained collection of Terraform configuration files, variables, and resources. Modules enable you to organize your infrastructure code into logical, reusable components, making your Terraform configurations more maintainable, modular, and easier to share across different projects or teams.

Null_Resource - As the name suggest null resouces in terraform is a resource type that does not represent a physical infrastructure resource like a virtual machine or a network component. Instead, it's a mechanism for using Terraform to run provisioners or perform other actions without creating an actual resource in the infrastructure. Null resources are often used for tasks that are not directly related to infrastructure creation, such as running local or remote commands, generating files, or executing scripts.

You can read more on others below:

- [null_resource](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource)
- [file provisioner](https://www.terraform.io/docs/language/resources/provisioners/file.html)
- [remote-exec provisioner](https://www.terraform.io/docs/language/resources/provisioners/remote-exec.html)
- [depends_on Meta-Argument](https://www.terraform.io/docs/language/meta-arguments/depends_on.html)
- [local-exec provisioner](https://www.terraform.io/docs/language/resources/provisioners/local-exec.html)

## Tasks:

Create the following files in the root folder of the project. Below is the folder structure.:
`versions.tf`, `generic-variables.tf`, `locals.tf`, `vpc-variables.tf`, `vpc-module.tf`, `vpc-outputs.tf`,`securitygroup-variables.tf`, `security-group-outputs.tf`, `securitygroup-jumpboxsg.tf`, `securitygroup-privatesg.tf`, `datasource-ami.tf`, `ec2instance-variables.tf`, `ec2instance-outputs.tf`, `ec2instance-jumpbox.tf`, `ec2instance-private.tf`, `elasticip.tf`, `nullresource-provisioners.tf`, `app1-install.sh`, `ec2instance.auto.tfvars`, `vpc.auto.tfvars`.

## Project Folder structure:

.
├── 01-AWS-3Tier-App
│   └── terraform-manifest
│       ├── README.md
│       ├── a1-versions.tf
│       ├── a2-generic-variables.tf
│       ├── a3-locals.tf
│       ├── a4-01-vpc-variables.tf
│       ├── a4-02-vpc-module.tf
│       ├── a4-03-vpc-outputs.tf
│       ├── a5-01-securitygroup-variables.tf
│       ├── a5-02-security-group-outputs.tf
│       ├── a5-03-securitygroup-jumpboxsg.tf
│       ├── a5-04-securitygroup-privatesg.tf
│       ├── a6-01-datasource-ami.tf
│       ├── a7-01-ec2instance-variables.tf
│       ├── a7-02-ec2instance-outputs.tf
│       ├── a7-03-ec2instance-jumpbox.tf
│       ├── a7-04-ec2instance-private.tf
│       ├── a8-elasticip.tf
│       ├── a9-nullresource-provisioners.tf
│       ├── app1-install.sh
│       ├── ec2instance.auto.tfvars
│       ├── private-key
│       └── vpc.auto.tfvars
└── README.md

_versions.tf_
```hcl
# Terraform Block
terraform {
 
 required_version = ">= 1.0"  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      # version = "~> 5.0"  
      version = "~> 3.0"
    }
     null = {
      source = "hashicorp/null"
      # version = "~> 5.0"
      version = "~> 3.0"
    }
  }
}

# Provider Block
provider "aws" {
  region  = var.aws_region
  profile = "default"
}

```

_generic-variables.tf_

```hcl

# AWS Region
variable "aws_region" {
  description = "Region in which AWS Resources to be created"
  type = string
  default = "us-east-1"  
}

# Environment Variable
variable "environment" {
  description = "Environment Variable used as a prefix"
  type = string
  default = "dev"
}

# Department
variable "department" {
  description = "Department in the large organization this Infrastructure belongs"
  type = string
  default = "Sales"
}

```

_locals.tf_

```hcl
locals {
  owners = var.department
  environment = var.environment
  name = "${local.owners}-${local.environment}"
  common_tags = {
    owners = local.owners
    environment = local.environment
  }
}
```

_vpc-variables.tf_
```hcl
# vpc name
variable "vpc_name" {
  description = "vpc name"
  type = string
  default = "myvpc"
}

# vpc cidr block
variable "vpc_cidr_block" {
  description = "vpc cidr block"
  type = string
  default = "10.0.0.0/16"
}

# vpc availability zones
variable "vpc_availability_zones" {
  description = "vpc avialability zone"
  type = list(string)
  default = [ "us-east-1a", "us-east-1b" ]
}

# vpc public subnets
variable "vpc_public_subnets" {
 description = "vpc public subnets"
 type = list(string)
 default = [ "10.0.101.0/24", "10.0.102.0/24" ]
}

# vpc private subnets
variable "vpc_private_subnets" {
  description = "vpc private subnets"
  type = list(string)
  default = [ "10.0.1.0/24", "10.0.2.0/24" ]
}

# vpc database subnets
variable "vpc_database_subnets" {
  description = "vpc databse subnets"
  type = list(string)
  default = [ "10.0.151.0/24", "10.0.152.0/24" ]
}

# vpc create database subnets group - true / false
variable "vpc_create_database_subnet_group" {
  description = "vpc create subnet group"
  type = bool
  default = true
}

# vpc create database subnets route table - true / false
variable "vpc_create_database_subnet_route_table" {
  description = "vpc create subnet route table"
  type = bool
  default = true
}

# enable NAT gateway (true / false)
variable "vpc_enable_nat_gateway" {
  description = "Enable NAT Gateways for Private Subnets Outbound Communication"
  type = bool
  default = true
}

# create svpc single NAT Gateway (true / false)
variable "vpc_single_nat_gateway" {
  description = "enable single nat gateway in one availability zone" # thiisis to save cost for demo
  type = bool
  default = true
}
```
_vpc_module.tf
```hcl
module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "2.78.0"

    # vpc details
    name = "${local.name}-${var.vpc_name}"
    cidr = "${var.vpc_cidr_block}"
    azs = "${var.vpc_availability_zones}"
    public_subnets = "${var.vpc_public_subnets}"
    private_subnets = "${var.vpc_private_subnets}" 

    # database subnets
    database_subnets = var.vpc_database_subnets
    create_database_subnet_group = var.vpc_create_database_subnet_group
    create_database_subnet_route_table = var.vpc_create_database_subnet_route_table

    # nat gateway
   enable_nat_gateway = var.vpc_enable_nat_gateway
   single_nat_gateway = var.vpc_single_nat_gateway

#    vpc dns parameters
enable_dns_hostnames = true
enable_dns_support = true

# tags
tags = local.common_tags
vpc_tags = local.common_tags

public_subnet_tags = {
    Type = "Public Subnets"
}

private_subnet_tags = {
    Type = "Purivate Subnets"
}

database_subnet_tags = {
    Type = "Private Database Subnets"
}
}
```

_vpc-outputs.tf_

```hcl
# VPC
output "vpc_id" {
  description = "The ID of the VPC"
  value       = module.vpc.vpc_id
}

# Subnets
output "private_subnets" {
  description = "List of IDs of private subnets"
  value       = module.vpc.private_subnets
}

output "public_subnets" {
  description = "List of IDs of public subnets"
  value       = module.vpc.public_subnets
}

output "database_subnets" {
  description = "List of IDs of database subnets"
  value       = module.vpc.database_subnets
}

# NAT gateways
output "nat_public_ips" {
  description = "List of public Elastic IPs created for AWS NAT Gateway"
  value       = module.vpc.nat_public_ips
}

# vpc azs
output "azs" {
  description = "list of availability zones"
  value = module.vpc.azs
}
```

_securitygroup-variables.tf_

```hcl
# 

variable "jumpbox_sg" {
  description = "security group for jumpbox"
  type = string
  default = "public_jumpbox-sg"
}

variable "private_sg_name" {
  description = "private security group name"
  type = string
  default = "private-sg"
}
```

_security-group-outputs.tf_

```hcl
# Public Jummpbox Host Security Group Outputs

output "public_jumpbox_sg_group_id" {
  description = "The ID of the security group"
  value       = module.public_jumpbox_sg.this_security_group_id
}

output "public_jumpbox_sg_group_vpc_id" {
  description = "The VPC ID"
  value       = module.public_jumpbox_sg.this_security_group_vpc_id
}

output "public_jumpbox_sg_group_name" {
  description = "The name of the security group"
  value       = module.public_jumpbox_sg.this_security_group_name
}

# Private EC2 Instances Security Group Outputs
output "private_sg_id" {
  description = "The ID of the security group"
  value       = module.private_sg.this_security_group_id

}

output "private_sg_group_vpc_id" {
  description = "The description of the security group"
  value       = module.private_sg.this_security_group_vpc_id
}

## private_sg_group_name
output "private_sg_group_name" {
  description = "The name of the security group"
  value       = module.private_sg.this_security_group_name
}
```

_securitygroup-jumpboxsg.tf_

This is a terraform module named `public_jumpbox_sg` which creates a security group for public jumpbox or bastion host. This security group allows SSH access (port 22) from any source (0.0.0.0/0) and allows all egress traffic.

```hcl
module "public_jumpbox_sg" {
  source = "terraform-aws-modules/security-group/aws"
  version = "3.18.0"

  name        = var.jumpbox_sg
  description = "Security group with all available arguments set (this is just an example)"
  vpc_id      = module.vpc.vpc_id

  ingress_cidr_blocks = ["0.0.0.0/0"]
  ingress_rules = ["ssh-tcp", "http-80-tcp"]

  egress_rules = ["all-all"] # open to all
  tags = local.common_tags

 
}


```

_securitygroup-privatesg.tf_

This is a terraform module names `private_sg` and is used to create security group for private EC2 instances in private subnets. The security group will allow SSH (port 22) and HTTP (port 80) from within the VPC's IP range (10.10.0.0/16). It will also allow egress traffic.

```hcl
module "private_sg" {
  source = "terraform-aws-modules/security-group/aws"
  version = "3.18.0"

  name        = var.private_sg_name
  description = "security group for private"
  vpc_id      = module.vpc.vpc_id

  ingress_cidr_blocks = ["10.10.0.0/16"]
  ingress_rules = ["ssh-tcp", "http-80-tcp"]

  egress_rules = ["all-all"] # open to all
  tags = local.common_tags

 
}


```

_datasource-ami.tf_

```hcl
# Get latest AMI ID for Amazon Linux2 OS
data "aws_ami" "amzlinux2" {
  most_recent = true
  owners = [ "amazon" ]
  filter {
    name = "name"
    values = [ "al2023-ami-2023.2.20231011.0-*-x86_64" ]
  }
  filter {
    name = "root-device-type"
    values = [ "ebs" ]
  }
  filter {
    name = "virtualization-type"
    values = [ "hvm" ]
  }
  filter {
    name = "architecture"
    values = [ "x86_64" ]
  }
}
```

_ec2instance-variables.tf_

```hcl
# AWS EC2 Instance Terraform Variables
# EC2 Instance Variables

# AWS EC2 Instance Type
variable "instance_type" {
  description = "EC2 Instance Type"
  type = string
  default = "t2.micro"  
}

# AWS EC2 Instance Key Pair
variable "instance_keypair" {
  description = "AWS EC2 Key pair that need to be associated with EC2 Instance"
  type = string
  default = "aws_hostkey"
}

# AWS EC2 Private Instance Count
variable "private_instance_count" {
  description = "AWS EC2 Private Instances Count"
  type = number
  default = 1  
}
```

_ec2instance-outputs.tf_


```hcl
## ec2_jumpbox_public_instance_ids

output "ec2_jumpbox_public_instance_ids" {
  description = "List of IDs of instances"
  value = module.ec2_public.id
}

# ec2_jumpbox_public_ip
output "ec2_jumpbox_public_ip" {
  description = "List of public IP addresses assigned to the instances"
  value = module.ec2_public.public_ip
}

# Private EC2 Instances
## ec2_private_instance_ids
output "ec2_private_instance_ids" {
  description = "List of IDs of instances"
  value       = module.ec2_private.id
}
## ec2_private_ip
output "ec2_private_ip" {
  description = "List of private IP addresses assigned to the instances"
  value       = module.ec2_private.private_ip 
}
```

_ec2instance-jumpbox.tf_

```hcl
# AWS EC2 Instance Terraform Module
# Bastion Host - EC2 Instance that will be created in VPC Public Subnet#

## https://registry.terraform.io/modules/terraform-aws-modules/ec2-instance/aws/latest

module "ec2_public" {
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "2.17.0"

    # insert the 10 required variables here
  name                   = "${var.environment}-JumpBox"
#   name                   = "${local.name}-Jumpbox"
  #instance_count         = 5
  ami                    = data.aws_ami.amzlinux2.id
  instance_type          = var.instance_type
  key_name               = var.instance_keypair
  #monitoring             = true
  subnet_id              = module.vpc.public_subnets[0]
  vpc_security_group_ids = [module.public_jumpbox_sg.this_security_group_id]
  tags = local.common_tags
}
```

_ec2instance-private.tf_

The key `depends_on` is used here to that the instance is not created until the vpc is created.

```hcl
# AWS EC2 Instance Terraform Module
# EC2 Instances that will be created in VPC Private Subnets

module "ec2_private" {
  depends_on = [ module.vpc ] # VERY VERY IMPORTANT else userdata webserver provisioning will fail
  source  = "terraform-aws-modules/ec2-instance/aws"
  version = "2.17.0"

    # insert the 10 required variables here
  name                   = "${var.environment}-vm"
# name                   = "${local.name}-vm"
  ami                    = data.aws_ami.amzlinux2.id
  instance_type          = var.instance_type
  key_name               = var.instance_keypair
  #monitoring             = true
  vpc_security_group_ids = [module.private_sg.this_security_group_id]
# subnet_id              = module.vpc.private_subnets[0]
subnet_ids = [
    module.vpc.private_subnets[0],
    module.vpc.private_subnets[1]
]
instance_count         = var.private_instance_count
user_data = file("${path.module}/app1-install.sh")
tags = local.common_tags
}
```

_elasticip.tf_
The key `depends_on` is also used here to allow the jumpbox instance to be created before assign an elastic IP to the jumpbox.

```hcl
resource "aws_eip" "jumpbox_eip" {
  depends_on = [ module.ec2_public, module.vpc ]
  instance = module.ec2_public.id[0]
  vpc = true
  tags = local.common_tags
}
```


_nullresource-provisioners.tf_

```hcl
# create null resource and provisioners

resource "null_resource" "name" {
  # Connection Block for Provisioners to connect to EC2 Instance
  connection {
    type = "ssh"
    host = aws_eip.jumpbox_eip.public_ip
    user = "ec2-user"
    password = ""
    private_key = file("private-key/aws_hostkey.pem")
  }

  ## File Provisioner: Copies the aws_hostkey.pem file to /tmp/aws_hostkey.pem

  provisioner "file" {
    source      = "private-key/aws_hostkey.pem"
    destination = "/tmp/aws_hostkey.pem"  
  }
## Remote Exec Provisioner: Using remote-exec provisioner fix the private key permissions on Bastion Host
  provisioner "remote-exec" {
    inline = [
      "sudo chmod 400 /tmp/aws_hostkey.pem"
    ]
  }
}
```

_ec2instance.auto.tfvars_

The purpose of using `ec2instance.auto.tfvars` is to allow terraform to automatically load variables without requiring manual input during execution of `terraform apply` or `terraform plan` commands. It's a way to provide default values for variables and allow users to set those values without explicitly specifying them on the command line or within a separate .tfvars file. So this this case this `ec2instance.auto.tfvars` will overwrite the values set in `ec2instance-variables.tf` or `ec2instance-jumpbox.tf`.

```hcl
instance_type = "t2.micro"
instance_keypair = "aws_hostkey"
private_instance_count = 2
```


_vpc.auto.tfvars_

```hcl
# VPC Variables
vpc_name = "myvpc"
vpc_cidr_block = "10.0.0.0/16"
vpc_availability_zones = ["us-east-1a", "us-east-1b"]
vpc_public_subnets = ["10.0.101.0/24", "10.0.102.0/24"]
vpc_private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
vpc_database_subnets= ["10.0.151.0/24", "10.0.152.0/24"]
vpc_create_database_subnet_group = true 
vpc_create_database_subnet_route_table = true   
vpc_enable_nat_gateway = true  
vpc_single_nat_gateway = true
```

## Execute terrafom commands

```hcl
# Initialize terraform
terraform init

# validate terraform
terraform validate

# plan terraform
terraform plan

apply terraform

terraform apply
```

## Connect to Jumpbox host and test private instances

```bash
ssh -i aws-hostkey.pem ec2-user@<public-ip-address-for-jumpbox-host>

# placehoder
# Curl Test for Bastion EC2 Instance to Private EC2 Instances
curl  http://<Private-Instance-1-Private-IP>
curl  http://<Private-Instance-2-Private-IP>


# Connect to Private EC2 Instances from Bastion EC2 Instance
ssh -i /tmp/terraform-key.pem ec2-user@<Private-Instance-1-Private-IP>
cd /var/www/html
ls -lrta
Observation: 
1) Should find index.html
2) Should find app1 folder
3) Should find app1/index.html file
4) Should find app1/metadata.html file
5) If required verify same for second instance too.
6) # Additionalyy To verify userdata passed to Instance
curl http://169.254.169.254/latest/user-data
```

## Clean-up

```hcl
#terraform destroy
terraform destroy -auto-approve

# Clean-Up
rm -rf .terraform*
rm -rf terraform.tfstate*
```