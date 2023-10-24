# Create a custom VPC, security groups and NACL using AWS CDK with python

## Step 1: Introduction

AWS Cloud Development Kit (AWS CDK) helps in provisioning cloud infrastructre using common programming languages to model application.
It supports several programming languages such as TypeScript, JavaScript, Python, Java, Go, and .NET and provides high-level easy to use contructs. It also allows a fallback on low-level CloudFormation contructs.

## Step 2: Environment Setup

In this project we are going to build a custom VPC, public and a private subnets in AWS using Python. First we setup the environment using Python

## Python

This steps involves setting up AWS CDK version 1.58 on Ubuntu Linux 20.04. We would install the required dependencies and the use pip (Python package manager) to install the CDK.

1. Install Node.js: (AWS CDK requires Node.js), we first update and upgrade the system


```bash
sudo apt update
sudo apt upgrade
```
2. Install Node.js with NVM (Node Version Manager):
NVM is a popular tool for managing multiple Node.js versions on your system. It allows you to easily switch between different Node.js versions. To install Node.js 18.x using NVM, follow these steps:

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
```
a. Close and reopen the termminal
```bash
source ~/.bashrc

```
b. Verify that NVM is installed correctly by running:

```bash
nvm --version
```
2. Install Node.js version 18.0.0:
Once NVM is installed, you can use it to install Node.js version 18.0.0 with the following command:
```bash
nvm install 18.0.0
```

3. Set Node.js version 18.0.0 as the active version and as the default and check version

```bash
nvm use 18.0.0
nvm alias default 18.0.0
node -v
```
4. Install AWS CLI(Optional). The AWS CLI is useful in managing AWS resources.

```bash
curl "https://d1vvhvl2y92vvt.cloudfront.net/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip #make sure you have unzip installed
sudo ./aws/install
```

Verify the installation:

```bash
aws --version
```

5. Install AWS CDK:
Use npm (Node Package Manager) to install the AWS CDK and check version:

```bash
npm install -g aws-cdk@1.58
cdk --version
```

6. Configure AWS Credentials (if not already done):
If you haven't configured AWS credentials on your Ubuntu system, you can set them up by running: Add YOUR aws access ID, secrte ID and deafult region.

```bash
aws configure
```

7. Create a CDK Application:
Having installed AWS CDK, create a new directory for the project
```bash
mkdir my-cdk-app
cd my-cdk-app
```
a. Initialize a new CDK app. This iwll setup a basic CDK  project in Python

```bash
cdk init app --language=python
```

8. Activate a Python Virtual Environment:

```bash
python3 -m venv .env
source .env/bin/activate
```
Note: You might need to install the Python environment first

```bash
sudo apt-get install python3-venv
python3 -m venv .env
source .env/bin/activate
```

9. Install CDK Dependencies(Python)

```bash
pip install -r requirements.txt
```

10. Bootstrap the CDK Environment:
You can proceed to bootstrap your CDK environment: This creates AWS resources such as S3 bucket to host the cloud assembly assets, and application artifacts and creates AWS CDKToolkit. It also creates the IAM roles, policies to facilitate deploying AWS resources using CDK.

```bash
cdk bootstrap
```

## Step 3: Create a Basic AWS VPC Stack

Open the CDK app in the folder `my-cdk-app` and change directory to the `my_cdk_app` folder and create a file `custom_subnet_vpc_stack.py` and add the code below.


```python
from aws_cdk import core
import aws_cdk.aws_ec2 as ec2

class CustomSubnetStack(core.Stack):

    def __init__(self, scope: core.Construct, id: str, **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        # Create a VPC with custom subnets
        vpc = ec2.Vpc(
            self,
            "MyVpc",
            max_azs=2,
            cidr="10.0.0.0/16",
            subnet_configuration=[
                ec2.SubnetConfiguration(
                    subnet_type=ec2.SubnetType.PUBLIC,
                    name="PublicSubnet",
                    cidr_mask=24,
                ),
                ec2.SubnetConfiguration(
                    subnet_type=ec2.SubnetType.PRIVATE,
                    name="PrivateSubnet",
                    cidr_mask=24,
                ),
            ],
        )

        # Create a Network ACL
        my_nacl = ec2.NetworkAcl(
            self,
            "MyNACL",
            vpc=vpc,
        )

        # Associate the Network ACL with the public subnet
        ec2.CfnSubnetNetworkAclAssociation(
            self,
            "PublicSubnetNACLAssociation",
            subnet_id=vpc.select_subnets(subnet_type=ec2.SubnetType.PUBLIC).subnet_ids[0],
            network_acl_id=my_nacl.network_acl_id,
        )

app = core.App()
CustomSubnetStack(app, "CustomSubnetStack")
app.synth()

```

In the root of the application locate the `setup.py` file and below in the install_requirement list. This would install the required module for teh application

```bash
  "aws-cdk.core==1.58.0",
  "aws_cdk.aws_ec2==1.58.0"
```

At the root of the parent folder `my-cdk-app` update the file `app.py` and add the code below:

```python
app = core.App()
CustomSubnetStack(app, "CustomSubnetStack",
            env=core.Environment(account  = os.environ["CDK_DEFAULT_ACCOUNT"],
            region   = os.environ["CDK_DEFAULT_REGION"]))
```

The code in `app.py` should now look as below:

```python
#!/usr/bin/env python3
import os
from aws_cdk import core
from my_cdk_tutorial_app.custom_subnet_vpc_stack import CustomSubnetStack

app = core.App()
CustomSubnetStack(app, "CustomSubnetStack",
                env=core.Environment(account  = os.environ["CDK_DEFAULT_ACCOUNT"],
                region   = os.environ["CDK_DEFAULT_REGION"]))
app.synth()

```
After activating the environment, we need to install the required dependencies in the `requirements.txt` file. 

```bash
pip install -r requirements.txt
```

We can run the code below to check the list of stacks for the application. This should list the stacks


```bash
cdk ls
```

The output should look as below:
 
```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-app$ cdk ls
CustomSubnetStack
```

Next is to then deploy the application

```bash
cdk deploy OR cdk deploy <name of the app> e.g. cdk deploy CustomSubnetStack
```

This will create the following:

- A VPC called `MyVpc` with cidr block `10.0.0.0/16`
- A public subnet and a private subnet with cidr ...
- A NACL named `MyNACL` in the custom VPC .
- Associates the Public subnet with the NACL. This is to control inbound and outbount traffic from the internet.

output:

```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ cdk deploy CustomSubnetStack

✨  Synthesis time: 1.82s

CustomSubnetStack: deploying... [1/1]
CustomSubnetStack: creating CloudFormation changeset...
[█████████████████████████████████████████████·············] (21/27)

3:38:07 PM | CREATE_IN_PROGRESS   | AWS::CloudFormation::Stack            | CustomSubnetStack
3:38:31 PM | CREATE_IN_PROGRESS   | AWS::EC2::NatGateway                  | MyVpc/PublicSubnetSubnet2/NATGateway
3:38:31 PM | CREATE_IN_PROGRESS   | AWS::EC2::NatGateway                  | MyVpc/PublicSubnetSubnet1/NATGateway
3:38:42 PM | CREATE_IN_PROGRESS   | AWS::EC2::SubnetNetworkAclAssociation | PublicSubnetNACLAssociation
```

```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ cdk deploy CustomSubnetStack

✨  Synthesis time: 1.82s

CustomSubnetStack: deploying... [1/1]
CustomSubnetStack: creating CloudFormation changeset...

 ✅  CustomSubnetStack

✨  Deployment time: 159.54s

Stack ARN:
arn:aws:cloudformation:us-east-1:447865183182:stack/CustomSubnetStack/c3d8cee0-71b1-11ee-aa90-0ad86e720ecb

✨  Total time: 161.37s


(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ 

```

## Step 4: Clean-up

To destroy the infrastructure we just created run the code below to destroy it.

```bash
cdk destroy CustomSubnetStack
```
output:


```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ cdk destroy CustomSubnetStack
Are you sure you want to delete: CustomSubnetStack (y/n)? y
CustomSubnetStack: destroying... [1/1]
```


```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ cdk destroy CustomSubnetStack
Are you sure you want to delete: CustomSubnetStack (y/n)? y
CustomSubnetStack: destroying... [1/1]
3:45:08 PM | DELETE_IN_PROGRESS   | AWS::EC2::SubnetRouteTableAssociation | MyVpc/PrivateSubne...teTableAssociation
3:45:08 PM | DELETE_IN_PROGRESS   | AWS::EC2::SubnetRouteTableAssociation | MyVpc/PrivateSubne...teTableAssociation
3:45:08 PM | DELETE_IN_PROGRESS   | AWS::EC2::SubnetRouteTableAssociation | MyVpc/PublicSubnet...teTableAssociation
3:45:08 PM | DELETE_IN_PROGRESS   | AWS::EC2::SubnetRouteTableAssociation | MyVpc/PublicSubnet...teTableAssociation
3:45:08 PM | DELETE_IN_PROGRESS   | AWS::EC2::SubnetNetworkAclAssociation | PublicSubnetNACLAssociation
3:45:10 PM | DELETE_IN_PROGRESS   | AWS::EC2::NatGateway                  | MyVpc/PublicSubnetSubnet1/NATGateway
3:45:10 PM | DELETE_IN_PROGRESS   | AWS::EC2::NatGateway                  | MyVpc/PublicSubnetSubnet2/NATGateway
3:45:10 PM | DELETE_IN_PROGRESS   | AWS::EC2::VPCGatewayAttachment        | MyVpc/VPCGW
```


```bash
(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$ cdk destroy CustomSubnetStack
Are you sure you want to delete: CustomSubnetStack (y/n)? y
CustomSubnetStack: destroying... [1/1]

 ✅  CustomSubnetStack: destroyed

(.env) user@DESKTOP-9P075RF:~/my-cdk-practice-app$
```

## Step 5: Verify Output

Custom vpc created

![vpc](https://github.com/thinkC/new-devops-projects/blob/master/aws-cdk-img/python-vpc1.png?raw=true)

Route Table

![route table](https://github.com/thinkC/new-devops-projects/blob/master/aws-cdk-img/python-cdk-rt.png?raw=true)

NAT Gateway

![nat](https://github.com/thinkC/new-devops-projects/blob/master/aws-cdk-img/python-cdk-nat.png?raw=true)

Elastic IP

![elastic ip](https://github.com/thinkC/new-devops-projects/blob/master/aws-cdk-img/python-cdk-elastic-ip.png?raw=true).


## Conclusion:

AWS VPC was successfully created with AWS CDK using python.

