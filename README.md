# Elastic Beanstalk practice guide

This guide contains instructions and tips to practice aws elastic beanstalk from the command line to deploy with docker image stored in ECR

## Prerequisites

To practice elastic beanstalk from the command line on your local environment clone the repo and install the eb cli first using these instructions:
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html

Create an IAM role named "aws-elasticbeanstalk-ec2-role" with following permissions:
- AWSElasticBeanstalkMulticontainerDocker
- AWSElasticBeanstalkWebTier
- AWSElasticBeanstalkWorkerTier
- EC2InstanceProfileForImageBuilderECRContainerBuilds

Also, a role named "aws-elasticbeanstalk-service-role" should be automatically created for you on first eb create command
If not created, make sure you have this role with following permissions:
- AWSElasticBeanstalkEnhancedHealth
- AWSElasticBeanstalkManagedUpdatesCustomerRolePolicy


## Docker locally

To test starting the app in a docker container locally follow these steps:
1. build docker image
```bash
docker build -t eb-java-docker . 
```
2. run  docker container
```bash
docker run -p 8001:8001 eb-java-docker
```

## Steps for ECR deployment (take these steps before the current one):

1. build docker image locally (optionally add --platform linux/amd64 tag to build image on mac with silicon processor)
```bash
docker build -t eb-java-docker . 
```

2. make sure your AWS user has access to ecr:GetAuthorizationToken and ecr:CreateRepository actions

In order to do so, you can add a custom policy to your user's group with following permissions:

{
    "Version": "2012-10-17",
        "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:CreateRepository",
                "ecr:BatchCheckLayerAvailability",
                "ecr:GetDownloadUrlForLayer",
                "ecr:GetRepositoryPolicy",
                "ecr:DescribeRepositories",
                "ecr:ListImages",
                "ecr:DescribeImages",
                "ecr:BatchGetImage",
                "ecr:InitiateLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:CompleteLayerUpload",
                "ecr:PutImage"
            ],
            "Resource": "*"
        }
    ]
}

3. authenticate to ECR
```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

4. get image id with docker images command, copy the image id
5. tag image (change the image name/tag)
```bash
docker tag <image_id> <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<image_name>
```

6. push image to ECR repo:
```bash
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/<image_name>
```

7. create an additional project with ed-java-external name and add Dockerrun.aws.json file in the project directory
## Current steps:

8. initialize git repo for new project, copy .gitignore file and .ebextensions directory there
9. init eb env
```bash
eb init
```
- choose us-east-1 region
- create new application
- choose app name
- choose docker platform

10. create eb environment
```bash
eb create --single
```
11. Open the app in the browser:
```bash
eb open
```

12. Terminate the environment to clean up resources:
```bash
eb terminate
```

## Basic EB commands

To init the application with name and region:
```bash
eb init -p <platform> <app-name> --region <region>
```

To create env without an elastic load balancer (with a single instance):
```bash
eb create java-env --single     
```

for example, to init the application in java platform with name java-tutorial and region us-east-1 use this command:
```bash
eb init -p corretto java-tutorial --region us-east-1
```

To deploy/update:
```bash
eb deploy <optional-env-name>
```

To open the app in the browser:
```bash
eb open
```

To terminate the environment:
```bash
eb terminate
```

To check the current status:
```bash
eb status
```

To see the logs when the app is ready:
```bash
eb logs
```

To clone the environment for blue-green deployment
```bash
eb clone
```

To swap the environment URLs for blue-green deployment
```bash
eb swap <old-env-name> --destination_name <new-env-name>
```

***
## Deployment options:

Add this .config file for setting immutable deploy:
```yaml
option_settings:
  aws:elasticbeanstalk:command:
    DeploymentPolicy: Immutable
    HealthCheckSuccessThreshold: Warning
    IgnoreHealthCheck: true
    Timeout: "600"
```

