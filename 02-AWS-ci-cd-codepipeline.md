# CI/CD With AWS Pipeline

## Introduction

In this project we are going to demostrate using Continuous Integration and Continuous Deployment" (CI/CD) and deploying an application to production using AWS Elastic Beanstalk, AWS CloudFormation, AWS CodeCommit, and AWS CodePipeline.

What is AWS Code Pipeline? 

AWS CodePipeline is a continuous delivery service that enables you to model, visualize, and automate the steps required to release your software. source - AWS website.
We would create a git repo on AWS Codecommit. We would give add the application on our local pc to the codecommit repo. To do this we would grant access from our local pc to the repository by creating a special username and passsword created specficcally forf Git connection to CodeCommit. The user (non root and no admin access) we would create will be given IAM access for CodeCommit.

## Step 1: Create a repository on AWS 
1. Search for `codecommit` on AWS management console and click _create repository.
2. Give it a name e.g. _mycodepipelineapp__.
3. Open the AWS management console, and go to IAM dashboard and go to `Groups` and click `Create New Group`.
4. Name the group e.g. `mycodecommitter`.
5. Still on the same page under `Attach Permission Polices`, search for `AWSCodeCommitFullAccess`,and then create group. This gives access to the user we would add to the group to be able to perform code commit operation. Because this is a demostration, we are giving full access but in real life , restrict access to the repository we created earlier.
6. Click on `Users` and click on `Create new user` and give it a name e.g. `codepipelineuser` and click next.
7. Under permission options, click `Add user to group` and unser `user groups` , select my `mycodecommitter` group we created earlier.
8. Click next and create user. Now the user have permission to access CodeCommit.
9. Next we create CodeCommit credentails for the user. Click on `Users` and click on the user we just created and click on the `Security credentials` tab.
10. Scroll down to `HTTPS Git credentials for AWS CodeCommit` and click generate credentials. This would generate a password we would use to access CodeCommit. Copy this and keep as we would not have access to it again.

## Step 2: Clone and push application file to CodeCommit.
 Here we would clone the application we would use in this project and then push it to CodeCommit.
1. Yu can either fork the repo or clone it. To clone the repo and save it on your PC. `https://github.com/thinkC/aws-codepipeline-react-app`Run the command below

```bash
mkdir my-code-pipeline
cd my-code-pipeline
git init # initialize the folder
git clone https://github.com/thinkC/aws-codepipeline-react-app
```
2. Next we would add AWS AWS remote to be able to push the code to CodeComit. Go back to the CodeCommit and copy the remote URL for the repo. Click the `Clone URL` DROP-DOWN AND SELECT `HTTPS`.
3. On the command line add as below.

```bash
git add aws-codepipeline-react-app
```

```bash
git remote add aws https://git-codecommit.us-east-1.amazonaws.com/v1/repos/mycodepipelineapp
```

```bash
git commit -m "my first commit"
```

```bash
git push aws master
```
It would prompt you for username and password. You the username and password you saved earlier and this would copy the files to the AWS repo.

## Step 3: Use a Cloudformation template to create an AWS Elastic Beanstalk (EB)

Elastic Beanstalk is an orchestration service which is used in deploying applications to orchestrate various AWS services. The Cloudformation template is located at the root of the git repo `.aws\myreactapp-eb-app.template` 


````bash
aws cloudformation create-stack --stack-name myreactapp-eb --template-body file://.aws/myreactapp-eb-app.template --capabilities CAPABILITY_IAM
``

output 

```bash
{
    "StackId": "arn:aws:cloudformation:us-east-1:447865183182:stack/myreactapp-eb/8957ad30-72ac-11ee-95cb-1220fd670719"
}

```


## Create code pipeline

name - myreactappcodepipeline

servicerole- AWSCodePipelineServiceRole-us-east-1-myreactappcodepipeline















codepipelineuser-at-447865183182
7DKVNC54gUc8VJsDK5s/izZQUja7KrAnslH+6Hws7pU=