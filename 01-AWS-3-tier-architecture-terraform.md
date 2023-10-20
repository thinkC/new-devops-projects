# A 3-tier architecture on AWS with Terraform

![3-tier-diagram](https://github.com/thinkC/new-devops-projects/blob/master/aws-terraform-img/01-img-01.png?raw=true)

## Step-01: Introduction
The 3-tier architecture as seen in the diagram above consist of a vpc with two avaialability zones. Two public subnets are created in each availability zones each also containing a NAT gateway in each zones with one jumpbox or bastion host in any of the public subnet. This jumpbox host will be used to gain accesess to the ec2 instances in the private subnets. This NAT gateway will be used by the ec2 instances in the private subnets to gain access to the internet.

The private subnet are divided into two. The first set of private subnet is where the web applications will reside and will have access to the insternet through the NAT gateay. The other set of private subnet would host the databases in reach avaialability zones.



This project will use terrarform modules to create aws resources such as `vpc`, `ec2` instances, `security groups` and also touch on `aws eip` (elastic ip), `null resource`, `file provisioner`, `remote-exec provisioner`, and `depends_on Meta-Argument`.

Modules -  Module in teraform is a reusable and self-contained collection of Terraform configuration files, variables, and resources. Modules enable you to organize your infrastructure code into logical, reusable components, making your Terraform configurations more maintainable, modular, and easier to share across different projects or teams.

Null_Resource - As the name suggest null resouces in terraform is a resource type that does not represent a physical infrastructure resource like a virtual machine or a network component. Instead, it's a mechanism for using Terraform to run provisioners or perform other actions without creating an actual resource in the infrastructure. Null resources are often used for tasks that are not directly related to infrastructure creation, such as running local or remote commands, generating files, or executing scripts.

You can read more on others below:

- [null_resource](https://registry.terraform.io/providers/hashicorp/null/latest/docs/resources/resource)
- [file provisioner](https://www.terraform.io/docs/language/resources/provisioners/file.html)
- [remote-exec provisioner](https://www.terraform.io/docs/language/resources/provisioners/remote-exec.html)
- [depends_on Meta-Argument](https://www.terraform.io/docs/language/meta-arguments/depends_on.html)
- [local-exec provisioner](https://www.terraform.io/docs/language/resources/provisioners/local-exec.html)
## Tasks:

Create the following files:
`versions.tf`, `generic-variables.tf`, `locals.tf`, `vpc-variables.tf`, `vpc-module.tf`, `vpc-outputs.tf`,`securitygroup-variables.tf`, `security-group-outputs.tf`, `securitygroup-jumpboxsg.tf`, `securitygroup-privatesg.tf`, `datasource-ami.tf`, `ec2instance-variables.tf`, `ec2instance-outputs.tf`, `ec2instance-jumpbox.tf`, `ec2instance-private.tf`, `elasticip.tf`, `nullresource-provisioners.tf`, `app1-install.sh`, `ec2instance.auto.tfvars`, `vpc.auto.tfvars`.

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