---
layout: single
title:  "Docker AWS"
categories: aws
---
# Aim

# AWS Components and structure 

# Testing

Make note of the task version as this increments if a new image is loaded successfully to the AWS docker registry.

Watch the codepipeline graphic which shows which stage the process is going through Can talk around 5 mins to go through all of the stages    

You should see the task version increment
Go to cluster and you should see 1 task running.

Now go to the load balancer and check if its working:

http://[ec2hostname].compute-1.amazonaws.com/


# Common problems:

If when trying to access a web page the browser waits a long time before saying connection failed it means there is a firewall rule some where which is stopping the request getting to your docker instance. This could be in a number of places:

 * Firewall in the load balancer:
   * Go to EC2 -> Load Balancers -> Select the balancer you want
   * Click on the 'Security group:' link
   * Highlight the security group you are looking at (not sure why this isn't defaultly selected. )
   * Click the inbound tab
   * Make sure port 80 is open to the world
 * Firewall on your EC2 instance.
   * To ensure your application load balancer can connect to your docker instance you have to add a rule to your ec2 docker instance to give it access to all ports.
   * EC2 -> Instances (left hand menu) -> select the ec2 which is in the ECS Cluster
   * Look for `Security groups` and click link
   * Click inbound tab and ensure the following rule is present:

```Type: All traffic, Protocol: All, port range: 0 - 65535, Source: custom name of Security group given in the firewall above,  Description: something that make sesnse to you. ```

# Useful Resources:
 * Tutorial on deploying docker components from github https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-cd-pipeline.html
 * Setting up dynamic port mapping on containers: https://aws.amazon.com/premiumsupport/knowledge-center/dynamic-port-mapping-ecs/

# Idea of Costs to bare in mind:
Note this may not be all of them so investigate them fully
  * CodePipeline: https://aws.amazon.com/codepipeline/pricing/
  * CodeBuilder: https://aws.amazon.com/codebuild/pricing/
  * S3 storage for build artefacts: https://aws.amazon.com/s3/pricing/
  * Image registry pricing: https://aws.amazon.com/ecr/pricing/
  * EC2 pricing: https://aws.amazon.com/ec2/pricing/
  * Elastic load balancer: https://aws.amazon.com/elasticloadbalancing/pricing/


# Incomplete step by step guide

AWS Deployment process:
 * Elastic Container Service:
    * Create repository
        * note repository URI:

```Repository URI 420715881449.dkr.ecr.us-east-1.amazonaws.com/imageapi ```

    * Create task
        * Create new task
        * Select EC2 instance in this example (need to test Fargate as an option)
        * Name: name of task
        * no task role
        * Network mode: default
        * add container:
            * add Name
            * Add image -> repo uri above with :latest
            * Memory limit -> going for 300MB note this can cause issues with deployment if EC2 instance doesn't have the available memory. Docs from AWS:

```
If you specify a hard limit (memory), your container will be killed if it attempts to exceed that limit. If you specify a soft limit (memoryReservation), ECS reserves that amount of memory for your container; however, the container can request up to the hard limit (if specified) or all of the available memory on the container instance, whichever is reached first. If you specify both, the hard limit must be greater than the soft limit.
```
            * Port mapping -> map internal docker port to external ec2 container port. Note Host Port is the EC2 port and container port is docker port. Set the host port to 0 if you want to have dynamic port mapping. Note this requires a load balancer.
            * Leave Environment settings as is.
            * Leave network settings as is.
            * Leave Storage and logging
            * Security click here if you need the docker to run with privilege settings.
            * Resource limit -> leave as is
            * Docker labels -> leave as is
        * Leave task options; Constraint and volumes as is.

-- need to create cluster somehow.

     * Create Service
        * Click on cluster
        * Click on create in service tab
            * Launch type EC2
            * Select task definition you just created.
            * Cluster select the desired cluster although this is probably already set
            * Service Name
            * Number of tasks - amount of copies of the docker instance to keep runnning. In this case use 1.
            * Minimum health percent. Set to 0%. If this is set above 0% it means you won't be able to deploy the docker instance if you only have one ec2 host. More than 0% means it won't deploy a new version of the docker unless one copy is running. On a single ec2 instance when the new instance tries to deploy it fails due to a clash of ports.
            * Task placement - Default AZ Balanced Spread.
            * load balancer
                * Possible to not use a load balancer but in this exmaple I'm going to create a application load balancer. This might solve some of the deployment issues with port clashes..
                * Go to the ec2 management console which is linked next to load balancer.
                * select:
                 * Internet facing
                 * Ip address type ipv4
                 * Listeners http port 80
                 * Availability zones - must choose 2 just going for first two.. ensure VPC matches the VPC of your docker host
                 * Not using https
                 * Using default security group.

                 * Configure routing:
                 * Target group - new target group
                 * Protocol - http
                 * Port -80
                 * Target type instance
                 * Health checks http path /

                 * Targets: select none
         * Now add load balancer to service.
         * Listener port:
            * click add listener then fill in the following:
            * Listener protocol Should this be 8000?
            * protocol: HTTP
            * Target group name - select target group we created earlier
            * Target group protocol HTTP
            * Target type instance
            * Path pattern - not sure what this does...
            * Evaluation order - default
            * Health check path - /
        * Auto scalling - not going to enable this now.
        * Create service.

    * Open up port on EC2 instance.
    * Select docker instance EC2 -> Instances -> DockerHost
    * Click Security groups in  instance information
    * Click inbound tab then click edit to create a new rule.
        * Custom TCP  
        * Port 8001
        * Anywhere.
        * add description
        * Note could add a range so don't have to go through this for every docker instance.

With that setup we now need to connect github so it will create an image and load it to the AWS image repository every time a commit is done to the master branch.             

First we need to add a buildspec.yml to tell code pipeline how to build the application. In the following example change:

 * ```REPOSITORY_URI``` to the repository URI of your image (go to Elastic Container and select repository and you should see the repository URI)
 * ```IMAGE_NAME``` this is the short name of the image.
 * line 26 change container name to the container you created earlier. (Elastic Container Service -> Task Definitions -> select task -> select latest task version -> Container Definitions (heading) -> Container name )

```
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws --version
      - $(aws ecr get-login --region $AWS_DEFAULT_REGION --no-include-email)
      - IMAGE_NAME="imageapi"
      - REPOSITORY_URI=420715881449.dkr.ecr.us-east-1.amazonaws.com/imageapi
      - IMAGE_TAG=prod_$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)
  build:
    commands:
      - echo Build started on `date`
      - echo Building the Docker image...
      - echo Image_tag $IMAGE_TAG
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG
  post_build:
    commands:
      - echo Build completed on `date`
      - echo Pushing the Docker images...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing image definitions file...
      - printf '[{"name":"imageapi-container","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json
artifacts:
    files: imagedefinitions.json
```

 1. Create Code Pipeline (CodePipeline -> Create Pipeline (button))
    * Name - imageapi-Pipeline
    * Source provider
        * github
        * connect to github, select repo and branch.
    * Build provider: AWS CodeBuilder
        * Create a new build project
            * name: image-api-builder
            * Environment
                * Use an image managed by AWS CodeBuild
                * choose an OS: Ubuntu
                * Runtime*: Docker
                * Version: (latest) aws/codebuild/docker:17.09.0
                * Build specification: Use the buildspec.yml in the source code root directory
            * Service role:
                    * need to work this one out... try creating new for now.
            * Advanced -> Enviroment variables use this if you want to make the `buildspec.yml` use Environment variables configured here.
        * Save build project
    * Deploy: Deployment provider* Amazon ECS            
        * Cluster name* select cluster name
        * Service name*: select service name that you created
        * Image filename: imagedefinitions.json ( this is written out in the `buildspec.yml` above.)
    * Need to look at role creation - AWS Service Role (AWS-CodePipeline-Service)
    * Create pipeline
