# Create AWS VPC, Subnets, NAT Gateway, NACL, Elastic IP and Route Table using AWS Cloudformation

## Introduction

AWS CloudFormation is a service provided by Amazon Web Services (AWS) that enables users to define and provision their AWS infrastructure and applications in a declarative manner, using templates. In other words, CloudFormation allows you to create and manage AWS resources and services using code, rather than manually configuring them through the AWS Management Console or command-line tools. One good thing about cloudformation is that it figures out what AWS resources dependent on.

In this project we are going to create a Cloudformation template by creating a cloudformation yaml file and then upload it to AWS S3 bucket. We would use this template to create an AWS VPC, a public and a private subnet and an AWS NACL (Network Access Control List) which would be associated to the public subnet. Cloudformation would then automatically create elastic IP connected to both public subnets, route tables for both public and private subnets and internet gateway also connected to the public subnet.

## Step 1: 
 - Create S3 bucket: This will be used to store the cloudformation template.
 1. Search for S3 bucket in the AWS management console and click `Create Bucket`
 2. Give the bucket a name, choose a region e.g. "us-east-1" and leave the default settings.
 3. Click create bucket.

## Step 2: 
- Create cloudformation stack
1. Search for Cloudformation in the AWS management console and click `Create stack`, choose (with new resource - standard).
3. Create a file e.g. `customvpc.yml` and save it on your local pc and add the code below inside it. This is the cloudformation template that would create the AWS resources we mentioned above.  The ip range for the vpc is `10.0.0.0/16`, the vpc name is `MyVpc`, the ip range for the public subnets are `10.0.0.0/24` and `10.0.1.0/24` and would be created in two availability zones `us-east-1a` and `us-east-1b` respectively. While the ip range for the private subnets are `10.0.2.0/24` and `10.0.3.0/24`.

`MyVpcPublicSubnetSubnet1Subnet60D1320D` - Defines the first public subnet with its CIDR block, the associated VPC, availability zone, and public IP mapping. It also has tags that identify its purpose.

`MyVpcPublicSubnetSubnet1RouteTable00654ADB` - Specifies a route table associated with the first public subnet, identified by tags.

`MyVpcPublicSubnetSubnet1RouteTableAssociation2CCE9CDC` - Associates the first public subnet with the corresponding route table

`MyVpcPublicSubnetSubnet1DefaultRoute2D379878` - Defines a default route in the route table that points to an Internet Gateway.

`MyVpcPublicSubnetSubnet1EIP5C2C4ED5` - Allocates an Elastic IP address for the first public subnet.

`MyVpcPublicSubnetSubnet1NATGateway9744F529` - Creates a NAT gateway in the first public subnet and associates it with the allocated Elastic IP..

```yaml
Resources:
  MyVpcF9F0CA6F:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsHostnames: true
      EnableDnsSupport: true
      InstanceTenancy: default
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/Resource
  MyVpcPublicSubnetSubnet1Subnet60D1320D:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.0.0/24
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-east-1a
      MapPublicIpOnLaunch: true
      Tags:
        - Key: aws-cdk:subnet-name
          Value: PublicSubnet
        - Key: aws-cdk:subnet-type
          Value: Public
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/Subnet
  MyVpcPublicSubnetSubnet1RouteTable00654ADB:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/RouteTable
  MyVpcPublicSubnetSubnet1RouteTableAssociation2CCE9CDC:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId:
        Ref: MyVpcPublicSubnetSubnet1RouteTable00654ADB
      SubnetId:
        Ref: MyVpcPublicSubnetSubnet1Subnet60D1320D
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/RouteTableAssociation
  MyVpcPublicSubnetSubnet1DefaultRoute2D379878:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId:
        Ref: MyVpcPublicSubnetSubnet1RouteTable00654ADB
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId:
        Ref: MyVpcIGW5C4A4F63
    DependsOn:
      - MyVpcVPCGW488ACE0D
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/DefaultRoute
  MyVpcPublicSubnetSubnet1EIP5C2C4ED5:
    Type: AWS::EC2::EIP
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/EIP
  MyVpcPublicSubnetSubnet1NATGateway9744F529:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId:
        Fn::GetAtt:
          - MyVpcPublicSubnetSubnet1EIP5C2C4ED5
          - AllocationId
      SubnetId:
        Ref: MyVpcPublicSubnetSubnet1Subnet60D1320D
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet1/NATGateway
  MyVpcPublicSubnetSubnet2Subnet122ADB1B:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.1.0/24
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-east-1b
      MapPublicIpOnLaunch: true
      Tags:
        - Key: aws-cdk:subnet-name
          Value: PublicSubnet
        - Key: aws-cdk:subnet-type
          Value: Public
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/Subnet
  MyVpcPublicSubnetSubnet2RouteTableC647F413:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/RouteTable
  MyVpcPublicSubnetSubnet2RouteTableAssociation7AF8666E:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId:
        Ref: MyVpcPublicSubnetSubnet2RouteTableC647F413
      SubnetId:
        Ref: MyVpcPublicSubnetSubnet2Subnet122ADB1B
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/RouteTableAssociation
  MyVpcPublicSubnetSubnet2DefaultRouteAFC76296:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId:
        Ref: MyVpcPublicSubnetSubnet2RouteTableC647F413
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId:
        Ref: MyVpcIGW5C4A4F63
    DependsOn:
      - MyVpcVPCGW488ACE0D
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/DefaultRoute
  MyVpcPublicSubnetSubnet2EIPF93892AD:
    Type: AWS::EC2::EIP
    Properties:
      Domain: vpc
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/EIP
  MyVpcPublicSubnetSubnet2NATGateway883B336D:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId:
        Fn::GetAtt:
          - MyVpcPublicSubnetSubnet2EIPF93892AD
          - AllocationId
      SubnetId:
        Ref: MyVpcPublicSubnetSubnet2Subnet122ADB1B
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PublicSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PublicSubnetSubnet2/NATGateway
  MyVpcPrivateSubnetSubnet1SubnetE8BD536C:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.2.0/24
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-east-1a
      MapPublicIpOnLaunch: false
      Tags:
        - Key: aws-cdk:subnet-name
          Value: PrivateSubnet
        - Key: aws-cdk:subnet-type
          Value: Private
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1/Subnet
  MyVpcPrivateSubnetSubnet1RouteTableD918A34F:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1/RouteTable
  MyVpcPrivateSubnetSubnet1RouteTableAssociation2811D7AF:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId:
        Ref: MyVpcPrivateSubnetSubnet1RouteTableD918A34F
      SubnetId:
        Ref: MyVpcPrivateSubnetSubnet1SubnetE8BD536C
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1/RouteTableAssociation
  MyVpcPrivateSubnetSubnet1DefaultRoute58825934:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId:
        Ref: MyVpcPrivateSubnetSubnet1RouteTableD918A34F
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId:
        Ref: MyVpcPublicSubnetSubnet1NATGateway9744F529
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet1/DefaultRoute
  MyVpcPrivateSubnetSubnet2SubnetE3BFCF91:
    Type: AWS::EC2::Subnet
    Properties:
      CidrBlock: 10.0.3.0/24
      VpcId:
        Ref: MyVpcF9F0CA6F
      AvailabilityZone: us-east-1b
      MapPublicIpOnLaunch: false
      Tags:
        - Key: aws-cdk:subnet-name
          Value: PrivateSubnet
        - Key: aws-cdk:subnet-type
          Value: Private
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2/Subnet
  MyVpcPrivateSubnetSubnet2RouteTable83C86AF7:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2/RouteTable
  MyVpcPrivateSubnetSubnet2RouteTableAssociationE7A10A88:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      RouteTableId:
        Ref: MyVpcPrivateSubnetSubnet2RouteTable83C86AF7
      SubnetId:
        Ref: MyVpcPrivateSubnetSubnet2SubnetE3BFCF91
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2/RouteTableAssociation
  MyVpcPrivateSubnetSubnet2DefaultRoute0FED656C:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId:
        Ref: MyVpcPrivateSubnetSubnet2RouteTable83C86AF7
      DestinationCidrBlock: 0.0.0.0/0
      NatGatewayId:
        Ref: MyVpcPublicSubnetSubnet2NATGateway883B336D
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/PrivateSubnetSubnet2/DefaultRoute
  MyVpcIGW5C4A4F63:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: CustomSubnetStack/MyVpc
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/IGW
  MyVpcVPCGW488ACE0D:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
      InternetGatewayId:
        Ref: MyVpcIGW5C4A4F63
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyVpc/VPCGW
  MyNACLCFE459E6:
    Type: AWS::EC2::NetworkAcl
    Properties:
      VpcId:
        Ref: MyVpcF9F0CA6F
    Metadata:
      aws:cdk:path: CustomSubnetStack/MyNACL/Resource
  PublicSubnetNACLAssociation:
    Type: AWS::EC2::SubnetNetworkAclAssociation
    Properties:
      NetworkAclId:
        Ref: MyNACLCFE459E6
      SubnetId:
        Ref: MyVpcPublicSubnetSubnet1Subnet60D1320D
    Metadata:
      aws:cdk:path: CustomSubnetStack/PublicSubnetNACLAssociation
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Modules: aws-cdk=2.100.0,@aws-cdk/assets=1.58.0,@aws-cdk/aws-apigateway=1.58.0,@aws-cdk/aws-applicationautoscaling=1.58.0,@aws-cdk/aws-autoscaling=1.58.0,@aws-cdk/aws-autoscaling-common=1.58.0,@aws-cdk/aws-autoscaling-hooktargets=1.58.0,@aws-cdk/aws-batch=1.58.0,@aws-cdk/aws-certificatemanager=1.58.0,@aws-cdk/aws-cloudformation=1.58.0,@aws-cdk/aws-cloudfront=1.58.0,@aws-cdk/aws-cloudwatch=1.58.0,@aws-cdk/aws-codebuild=1.58.0,@aws-cdk/aws-codecommit=1.58.0,@aws-cdk/aws-codeguruprofiler=1.58.0,@aws-cdk/aws-codepipeline=1.58.0,@aws-cdk/aws-cognito=1.58.0,@aws-cdk/aws-ec2=1.58.0,@aws-cdk/aws-ecr=1.58.0,@aws-cdk/aws-ecr-assets=1.58.0,@aws-cdk/aws-ecs=1.58.0,@aws-cdk/aws-ecs-patterns=1.58.0,@aws-cdk/aws-efs=1.58.0,@aws-cdk/aws-elasticloadbalancing=1.58.0,@aws-cdk/aws-elasticloadbalancingv2=1.58.0,@aws-cdk/aws-events=1.58.0,@aws-cdk/aws-events-targets=1.58.0,@aws-cdk/aws-iam=1.58.0,@aws-cdk/aws-kinesis=1.58.0,@aws-cdk/aws-kms=1.58.0,@aws-cdk/aws-lambda=1.58.0,@aws-cdk/aws-logs=1.58.0,@aws-cdk/aws-route53=1.58.0,@aws-cdk/aws-route53-targets=1.58.0,@aws-cdk/aws-s3=1.58.0,@aws-cdk/aws-s3-assets=1.58.0,@aws-cdk/aws-sam=1.58.0,@aws-cdk/aws-secretsmanager=1.58.0,@aws-cdk/aws-servicediscovery=1.58.0,@aws-cdk/aws-sns=1.58.0,@aws-cdk/aws-sns-subscriptions=1.58.0,@aws-cdk/aws-sqs=1.58.0,@aws-cdk/aws-ssm=1.58.0,@aws-cdk/aws-stepfunctions=1.58.0,@aws-cdk/cloud-assembly-schema=1.58.0,@aws-cdk/core=1.58.0,@aws-cdk/custom-resources=1.58.0,@aws-cdk/cx-api=1.58.0,@aws-cdk/region-info=1.58.0,jsii-runtime=Python/3.10.12


```

4. Upload the Template to AWS S3 . This is where AWS CloudFormation can access the template. Note the S3 URL. Click on the s3 bucket created earlier and click `upload`. Click the `Add files` button or drag the file.
5. Back to cloudformation, choose `Template is ready` and select `Upload a template file.` Provide the S3 bucket URL or upload the template file directly.
6. Specify Stack Details: Enter a stack name, and fill out any parameters or tags required by your template.
7. Configure Stack Options: You can configure stack options like permissions, rollback settings, and stack policy. This step is optional.
8. Review: Review the stack details to ensure everything is configured correctly.
9. Create the Stack: After reviewing, create the stack.
10. Monitor the Stack Creation: The CloudFormation stack will create the resources specified in your template, including the VPC, subnets, NAT and others. You can monitor the progress in the CloudFormation console.
11. Access Your EC2 Instance: Once the stack creation is complete, you can find the VPC, public and private subnets. NAT etc. in the CloudFormation outputs.

## Step 3: Verify results
Vpc

![vpc](https://github.com/thinkC/new-devops-projects/blob/master/aws-cloudformation/img5.png?raw=true)

Internet Gateway
![internet gateway](https://github.com/thinkC/new-devops-projects/blob/master/aws-cloudformation/img6.png?raw=true)

Elastic IPs
![elastic ips](https://github.com/thinkC/new-devops-projects/blob/master/aws-cloudformation/img7.png?raw=true)

NACL

![nacl](https://github.com/thinkC/new-devops-projects/blob/master/aws-cloudformation/img8.png?raw=true)

Public and Private subnets

![subnets](https://github.com/thinkC/new-devops-projects/blob/master/aws-cloudformation/img9.png?raw=true)

## Step 4: Clean-up

We can now destroy the resource. Go to the cloudfomation page, click on the stack you created and click `delete` , confirm deletion and delete. This would delete all the resources created. In addition, go to the S3 page and click on the s3 bucket you created, first click on emplty the content , confirm it and then go ahead to delete the s3 bucket.


Conclusion:

By following these steps we were able to create a custom VPC, Public and Private subnets, NAT Gateway, Internet Gateway, Routables and its association , Elastic IPs using AWS cloudformation.