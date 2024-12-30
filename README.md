# Elastic Beanstalk practice guide

## Prerequisites

To practice elastic beanstalk from the command line on your local environment clone the repo and install the eb cli first using these instructions:
https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html

## Steps for ECR deployment (take these steps before the current one):

1. build docker image locally
```bash
docker build -t eb-java-docker . 
```

2. make sure your AWS user has access to ecr:GetAuthorizationToken and ecr:CreateRepository actions

3. authenticate to ECR
```bash
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
```

4. get image id with docker images command
5. tag image
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
eb create --signle
```

## Basic commands

After the eb cli is configured, use the following commands to deploy the app:

To init the application in docker platform with name docker-tutorial and region us-east-1:
```bash
eb init -p docker docker-tutorial --region us-east-1
```

To create env without an elastic load balancer (with a single instance):
```bash
eb create docker-env --single     
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

