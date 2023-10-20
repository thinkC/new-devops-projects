# A 3-tier architecture on AWS with Terraform

![3-tier-diagram]()

## Step-01: Introduction

This project will use terrarform modules to create aws resources such as `vpc`, `ec2` instances, `security groups` and also touch on `aws eip` (elastic ip), `null resource`, `file provisioner`, `remote-exec provisioner`, and `depends_on Meta-Argument`.

Modules -  Module is a reusable and self-contained collection of Terraform configuration files, variables, and resources. Modules enable you to organize your infrastructure code into logical, reusable components, making your Terraform configurations more maintainable, modular, and easier to share across different projects or teams.