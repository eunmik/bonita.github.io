---
layout: post
title: "[AWS Hands-On] Break a Monolith Application into MSA"
date: 2022-03-25
excerpt: "AWS Hands-On"
tags: ["AWS", "Hands-On"]
comments: true
---


[Module One - Containerize the Monolith | AWS](https://aws.amazon.com/getting-started/hands-on/break-monolith-app-microservices-ecs-docker-ec2/module-one/?nc1=h_ls)

# Overview

In this Hands-On Tutorial, I am going to follow step by step to understand how to break a Montolith Application into Microservices. 

With this tutorial, a monolithic node.js application will be deployed to a Docker container which I can also understand how docker works. 

and then I will decouple the application into microservices without any **DOWNTIME.**

In this page, all the steps I do  will be shared with lots of screenshots!

# Module 1 - Containerize the Monolith

In this module, you will build the container image for your monolithic node.js application and push it to Amazon Elastic Container Registry.

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/Untitled.png" />

## Step 1. Get Setup

### 1. Have a AWS Account

### 2. Install Docker (macOS)

```bash
brew install --cask docker
```

--cask : to install GUI application 

### 3. Install AWS CLI (macOS)

[Installing or updating the latest version of the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

```bash
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg ./AWSCLIV2.pkg -target /
```

```bash
% aws --version
aws-cli/2.4.27 Python/3.8.8 Darwin/21.4.0 exe/x86_64 prompt/off
```

## Step 2. Download & Open the project

[https://github.com/awslabs/amazon-ecs-nodejs-microservices](https://github.com/awslabs/amazon-ecs-nodejs-microservices)

```bash
git clone https://github.com/awslabs/amazon-ecs-nodejs-microservices.git
```

## Step 3. Provision a Repository

### 1. Select Create Repository (레파지토리 생성)

<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.02.31.png">

### 2. On the Create repository page, enter the following name your repository: *api*.

**⚐ Note:** Under **Tag immutability**, leave the default settings.

![스크린샷 2022-03-22 오전 10.06.25.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.06.25.png">

### 3. Select **Create repository**.

![스크린샷 2022-03-22 오전 10.08.29.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.08.29.png">

## Step 4. Build & Push the Docker image

### **Use the terminal to authenticate Docker log in:**

### 1. configure your authentication (if it’s your first time to use aws cli)

[](https://us-east-1.console.aws.amazon.com/iamv2/home)

- Add Users on IAM Console Page

![스크린샷 2022-03-22 오전 10.12.57.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.12.57.png">

- type user name and select AWS credential type (this time, I chose Access Key)
    
    ![스크린샷 2022-03-22 오전 10.13.55.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.13.55.png">
    
- select how to set permissions : add user to group, copy permissions from existing user, or attach existing policies directly
    
    I created admin group with AdministratorAccess 
    
    ![스크린샷 2022-03-22 오전 10.14.52.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.14.52.png">
    
- skip add tags
- select create user
    
    ![스크린샷 2022-03-22 오전 10.16.24.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.16.24.png">
    
- save Access Key ID and secret access key to configure aws cli
    
    ![스크린샷 2022-03-22 오전 10.17.00.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.17.00.png">
    
- configure aws cli
    
    ```bash
    aws configure 
    ```
    
    ![스크린샷 2022-03-22 오전 10.19.01.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.19.01.png">
    

### 2. Docker Log in

```bash
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 431072036772.dkr.ecr.us-west-2.amazonaws.com/api
```

If the authentication was successful, you will receive the confirmation message: **Login Succeeded**.

### 3. Build the image

move to the directory ~/amazon-ecs-nodejs-microservices/2-containerized/services/api

To build the image, run the following command in the terminal: *docker build -t api* .

**⚐ Note:** **The period (**.**) after *api* is needed.

```bash
docker build -t api .
```

```bash
[+] Building 22.9s (9/9) FINISHED                                                                                                                             
 => [internal] load build definition from Dockerfile                                                                                                     0.2s
 => => transferring dockerfile: 149B                                                                                                                     0.0s
 => [internal] load .dockerignore                                                                                                                        0.1s
 => => transferring context: 2B                                                                                                                          0.0s
 => [internal] load metadata for docker.io/mhart/alpine-node:7.10.1                                                                                      3.7s
 => [internal] load build context                                                                                                                        0.1s
 => => transferring context: 3.69kB                                                                                                                      0.0s
 => [1/4] FROM docker.io/mhart/alpine-node:7.10.1@sha256:d334920c966d440676ce9d1e6162ab544349e4a4359c517300391c877bcffb8c                                7.6s
 => => resolve docker.io/mhart/alpine-node:7.10.1@sha256:d334920c966d440676ce9d1e6162ab544349e4a4359c517300391c877bcffb8c                                0.0s
 => => sha256:d334920c966d440676ce9d1e6162ab544349e4a4359c517300391c877bcffb8c 740B / 740B                                                               0.0s
 => => sha256:54c58132ca42ed07226e0bf19282863c6aa0ce53205baa1d90242d5ccf01491d 6.27kB / 6.27kB                                                           0.0s
 => => sha256:019300c8a437a2d60248f27c206795930626dfe7ddc0323d734143bd5eb131a6 1.97MB / 1.97MB                                                           0.5s
 => => sha256:c44ae64c93a279c2790c6c2a1e2a153dcae72c4e9396ee0752d76f2c04321d74 17.43MB / 17.43MB                                                         2.6s
 => => extracting sha256:019300c8a437a2d60248f27c206795930626dfe7ddc0323d734143bd5eb131a6                                                                1.8s
 => => extracting sha256:c44ae64c93a279c2790c6c2a1e2a153dcae72c4e9396ee0752d76f2c04321d74                                                                4.5s
 => [2/4] WORKDIR /srv                                                                                                                                   0.4s
 => [3/4] ADD . .                                                                                                                                        0.1s
 => [4/4] RUN npm install                                                                                                                               10.2s
 => exporting to image                                                                                                                                   0.5s
 => => exporting layers                                                                                                                                  0.5s
 => => writing image sha256:023ee736e7eb3a93a1c1d0a265de19dd4979b982b718b7cdb48a97885ceb10bf                                                             0.0s 
 => => naming to docker.io/library/api
```

### 4. Tag the Image

After the build completes, tag the image so you can push it to the repository: 

**⚐ Note:** Replace the *[account-ID]* and *[region]* placeholders with your specific information.

```bash
docker tag api:latest [account-ID].dkr.ecr.[region].amazonaws.com/api:v1
```

```bash
docker tag api:latest 431072036772.dkr.ecr.us-west-2.amazonaws.com/api:v1
```

you check if the images is built successfully following command *docker images*

```bash
docker images
REPOSITORY                                         TAG       IMAGE ID       CREATED         SIZE
431072036772.dkr.ecr.us-west-2.amazonaws.com/api   v1        023ee736e7eb   3 minutes ago   59.9MB
api                                                latest    023ee736e7eb   3 minutes ago   59.9MB
```

### 5. Push the Image to the Repository

Push the image to Amazon ECR by running: 

**⚐ Note:** replace the [account-ID] and [region] placeholders with your specific information.

```bash
docker push [account-id].dkr.ecr.[region].amazonaws.com/api:v1
```

```bash
docker push 431072036772.dkr.ecr.us-west-2.amazonaws.com/api:v1
```

```bash
The push refers to repository [431072036772.dkr.ecr.us-west-2.amazonaws.com/api]
dfed9ca1db03: Pushed 
43fccc911ce1: Pushed 
5f70bf18a086: Pushed 
3e893534526a: Pushed 
040fd7841192: Pushed
```

If you navigate to your Amazon ECR repository, you should see your image tagged *v1*.

![스크린샷 2022-03-22 오전 10.34.25.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오전_10.34.25.png">

# Module 2 : Deploy the Monolith

In this module, you will use Amazon Elastic Container Service (Amazon ECS) to instantiate a managed cluster of EC2 compute instances and deploy your image as a container running on the cluster.

![Untitled]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/Untitled%201.png">

## Step 1. Launch an ECS Cluster using AWS CloudFormation

Create an Amazon ECS cluster deployed behind an Application Load Balancer.

### 1. Go to AWS CloudFormation Console and Click Create stack

[](https://us-west-2.console.aws.amazon.com/cloudformation/home?region=us-west-2#/)

![스크린샷 2022-03-22 오후 7.43.17.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.43.17.png">

### 2. **Upload a template file** and choose the [ecs.yml](https://github.com/awslabs/amazon-ecs-nodejs-microservices/blob/master/2-containerized/infrastructure/ecs.yml) file from the GitHub project at *amazon-ecs-nodejs-microservice/2-containerized/infrastructure/ecs.yml* then select **Next**.

![스크린샷 2022-03-22 오후 7.45.52.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.45.52.png">

### 3. For the stack name, enter *BreakTheMonolith-Demo*. Verify that the other parameters have the following values:

1. Desired Capacity = *2*
2. InstanceType = *t2.micro*
3. MaxSize = *2*

![스크린샷 2022-03-22 오후 7.47.20.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.47.20.png">

### 3. For configure stack options and advanced options, keep them default and Select Next

### 4. Tick the check box and Create stack

![스크린샷 2022-03-22 오후 7.48.44.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.48.44.png">

You will see your stack with the status CREATE_IN_PROGRESS

![스크린샷 2022-03-22 오후 7.49.23.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.49.23.png">

After few minutes (maybe about 5 minutes) you can check the progress 

![스크린샷 2022-03-22 오후 7.51.43.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.51.43.png">

Or You can use command line to create stack

```
 aws cloudformation deploy \
   --template-file infrastructure/ecs.yml \
   --region [region] \
   --stack-name BreakTheMonolith-Demo \
   --capabilities CAPABILITY_NAMED_IAM
```

## Step 2. Check your cluster is running

[https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/clusters](https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/clusters)

![스크린샷 2022-03-22 오후 7.55.59.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_7.55.59.png">

- Select the cluster **BreakTheMonolith-Demo**, then select the **Tasks** tab to verify that there are no tasks running.

![스크린샷 2022-03-22 오후 8.00.50.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.00.50.png">

- Select the **ECS Instances** tab to verify there are two Amazon EC2 instances created by the AWS CloudFormation template.

![스크린샷 2022-03-22 오후 8.02.12.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.02.12.png">

## Step 3. Write a Task Definition

### 1. Select Task Denitions

![스크린샷 2022-03-22 오후 8.05.18.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.05.18.png">

### 2. Create new Task Denition and Select launch type EC2

![스크린샷 2022-03-22 오후 8.06.43.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.06.43.png">

### 3. Enter task definition name : api

![스크린샷 2022-03-22 오후 8.08.16.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.08.16.png">

### 4. Add Container

![스크린샷 2022-03-22 오후 8.11.55.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.11.55.png">

Scroll down to ENVIRONMENT

![스크린샷 2022-03-22 오후 8.12.20.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.12.20.png">

![스크린샷 2022-03-22 오후 8.13.03.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.13.03.png">

### 5. Create

![스크린샷 2022-03-22 오후 8.13.47.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.13.47.png">

![스크린샷 2022-03-22 오후 8.14.31.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.14.31.png">

## Step 4. Configure the Application Load Balancer : Target Group

The [Application Load Balancer (ALB)](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html)
 lets your service accept incoming traffic. The ALB automatically routes traffic to container instances running on your cluster using them as a [target group](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-target-groups.html)

### 1. Check VPC Name

[https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName)

![스크린샷 2022-03-22 오후 8.19.10.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.19.10.png">

### 2. Configure the ALB Target Group

[Module Two - Deploy the Monolith | AWS](https://aws.amazon.com/getting-started/hands-on/break-monolith-app-microservices-ecs-docker-ec2/module-two/)

1. create target group 

![스크린샷 2022-03-22 오후 8.21.55.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.21.55.png">

1. Configure the following Target Group parameters (for the parameters not listed below, keep the default values):
- For the **Target group name**, enter *api*.
- For the **Protocol**, select **HTTP**.
- For the **Port**, enter *80*.
- For the VPC, select the value that matches the one from the Load Balancer description. .
    
    This is most likely NOT your default VPC
    
    ![스크린샷 2022-03-22 오후 8.23.51.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.23.51.png">
    
- Access the **Advanced health check settings** and edit the following parameters as needed:
    - For **Healthy threshold**, enter *2*.
    - For **Unhealthy threshold**, enter *2*.
    - For **Timeout**, enter *5*.
    - For **Interval**, enter *6*.
        
        ![스크린샷 2022-03-22 오후 8.24.48.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.24.48.png">
        
- Create

## Step 5. Configure the Application Load Balancer : Listener

The ALB [listener](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) checks for incoming connection requests to your ALB.

[https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:)

### 1. add Listener

![스크린샷 2022-03-22 오후 8.27.21.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.27.21.png">

### 2. create

![스크린샷 2022-03-22 오후 8.28.12.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.28.12.png">

![스크린샷 2022-03-22 오후 8.28.45.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.28.45.png">

## Step 6. Deploy the Monolith as a Service

Deploy the monolith as a service into the cluster.

[https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2](https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2)

### 1. Creae Service

![스크린샷 2022-03-22 오후 8.30.18.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.30.18.png">

### 2. Configure service

![스크린샷 2022-03-22 오후 8.31.03.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.31.03.png">

### 3. Configure Load balancing

![스크린샷 2022-03-22 오후 8.32.19.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.32.19.png">

### 4. Add container to load balancer

![스크린샷 2022-03-22 오후 8.33.39.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.33.39.png">

### 5. Review and Create

![스크린샷 2022-03-22 오후 8.34.57.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.34.57.png">

## Step 7. Test your Monolith

Validate your deployment by checking if the service is available from the internet and pinging it.

[](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName)

Copy DNS and paste into a new browser

![스크린샷 2022-03-22 오후 8.39.57.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.39.57.png">

![스크린샷 2022-03-22 오후 8.37.12.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_8.37.12.png">

**See Each Part of the Service:** The node.js application routes traffic to each worker based on the URL. To see a worker, simply add the worker name *api/[worker-name]* to the end of the DNS Name as follows:

- http://*[DNS name]*/api/users
- http://*[DNS name]*/api/threads
- http://*[DNS name]*/api/posts

You can also add a record number at the end of the URL to drill down to a particular record. For example: *http://[DNS name]/api/posts/1* or *http://[DNS name]/api/users/2*

# Module 3 : Break the Monolith

In this module, you will break the node.js application into several interconnected services and push each service's image to an Amazon Elastic Container Registry (Amazon ECR) repository.

![Untitled]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/Untitled%202.png">

## Step 1. Provision the ECR Repositories

In the previous two modules, you deployed your application as a monolith using a single service and a single container image repository. To deploy the application as three microservices, you will need to provision three repositories (one for each service) in Amazon ECR.

- make three more repositories (users, posts, threads)

![스크린샷 2022-03-22 오후 10.23.48.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_10.23.48.png">

![스크린샷 2022-03-22 오후 10.24.28.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_10.24.28.png">

## Step 2. Build and Push Images for Each Service

- Build Images
    
    ```bash
    docker build -t [service-name] ./[service-name]
    ```
    
    ```bash
    docker build -t posts ./posts
    docker build -t users ./users
    docker build -t threads ./threads
    ```
    
- Tag Images
    
    ```bash
    docker tag [service-name]:latest [account-ID].dkr.ecr.[region].amazonaws.com/[service-name]:v1
    ```
    
    ```bash
    docker tag posts:latest 431072036772.dkr.ecr.us-west-2.amazonaws.com/posts:v1
    docker tag users:latest 431072036772.dkr.ecr.us-west-2.amazonaws.com/users:v1
    docker tag threads:latest 431072036772.dkr.ecr.us-west-2.amazonaws.com/threads:v1
    ```
    
- Push Images
    
    ```bash
    docker push [account-id].dkr.ecr.[region].amazonaws.com/[service-name]:v1
    ```
    
    ```bash
    docker push 431072036772.dkr.ecr.us-west-2.amazonaws.com/posts:v1
    docker push 431072036772.dkr.ecr.us-west-2.amazonaws.com/threads:v1
    docker push 431072036772.dkr.ecr.us-west-2.amazonaws.com/users:v1
    ```
    
    # Module 4 : Deploy Microservices
    
    In this module, you will deploy your node.js application as a set of interconnected services behind an Application Load Balancer (ALB). Then, you will use the ALB to seamlessly shift traffic from the monolith to the microservices.
    
    ![Untitled]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/Untitled%203.png">
    

## Step 1. Write Task Definition for your Services

You will deploy three new services to the cluster that you launched in [Module 2](https://aws.amazon.com/getting-started/hands-on/break-monolith-app-microservices-ecs-docker-ec2/module-two/). Like Module 2, you will write [Task Definitions](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definitions.html) for each service.

- create task using JSON Configuration for each task (users, posts, threads)
    
    ![스크린샷 2022-03-22 오후 10.49.07.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_10.49.07.png">
    
    ![스크린샷 2022-03-22 오후 10.46.04.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-22_오후_10.46.04.png">
    
    ```bash
    {
        "containerDefinitions": [
            {
                "name": "[service-name]",
                "image": "[account-id].dkr.ecr.[region].amazonaws.com/[service-name]:[tag]",
                "memoryReservation": "256",
                "cpu": "256",
                "essential": true,
                "portMappings": [
                    {
                        "hostPort": "0",
                        "containerPort": "3000",
                        "protocol": "tcp"
                    }
                ]
            }
        ],
        "volumes": [],
        "networkMode": "bridge",
        "placementConstraints": [],
        "family": "[service-name]"
    }
    ```
    

## Step 2. Configure the Application Load Balancer : Target Group

As in [Module 2](https://aws.amazon.com/getting-started/hands-on/break-monolith-app-microservices-ecs-docker-ec2/module-two/)
, configure a target group for each service (posts, threads, and users). A target group allows traffic to correctly reach a specified service. You will configure the target groups using AWS CLI

- Configure the target group
    
    ```bash
    aws elbv2 create-target-group --region [region] --name [service-name] --healthy-threshold-count 2 --unhealthy-threshold-count 2 --health-check-timeout-seconds 5 --health-check-interval-seconds 6 --protocol HTTP --port 80 --vpc-id [vpc-attribute] 
    ```
    
    ```bash
    aws elbv2 create-target-group --region us-west-2 --name threads  --healthy-threshold-count 2 --unhealthy-threshold-count 2 --health-check-timeout-seconds 5 --health-check-interval-seconds 6 --protocol HTTP --port 80 --vpc-id vpc-0a2ebb545b80b6307
    ```
    
    ![스크린샷 2022-03-23 오전 10.16.46.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_10.16.46.png">
    

## Step 3. Configure Listener Rules

The [listener](http://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) checks for incoming connection requests to your ALB in order to route traffic appropriately. 

Right now, all four of your services (monolith and your three microservices) are running behind the same load balancer. To make the transition from monolith to microservices, you will start routing traffic to your microservices and stop routing traffic to your monolith.

### Update Listenr rules

1. click View/edit rules under Listeners tab

![스크린샷 2022-03-23 오전 10.20.31.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_10.20.31.png">

1. Insert Rule

![스크린샷 2022-03-23 오전 11.22.57.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.22.57.png">

![스크린샷 2022-03-23 오전 11.26.14.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.26.14.png">

## Step 4. Deploy your Monolith

Deploy the three microservices (posts, threads, and users) to your cluster. Repeat these steps for each of your three microservices:

[](https://us-west-2.console.aws.amazon.com/ecs/home?region=us-west-2#/clusters/BreakTheMonolith-Demo-ECSCluster-o8iQP3xrqL4M/services)

### 1. Select Create on Services Tab

![스크린샷 2022-03-23 오전 11.28.24.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.28.24.png">

### 2. Configure service

![스크린샷 2022-03-23 오전 11.30.17.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.30.17.png">

### 3. Configure load balancing

![스크린샷 2022-03-23 오전 11.31.50.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.31.50.png">

### 4. Add container to load balance

![스크린샷 2022-03-23 오전 11.33.12.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오전_11.33.12.png">

### 5. Create

![스크린샷 2022-03-23 오후 1.02.25.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.02.25.png">

## Step 5. Switch Over Traffic to your Microservices

Your microservices are now running, but all traffic is still flowing to your monolith service. To reroute traffic to the microservices, take the following steps to update the listener rules:

[](https://us-west-2.console.aws.amazon.com/ec2/v2/home?region=us-west-2#LoadBalancers:sort=loadBalancerName)

### 1.  Edit Listener

![스크린샷 2022-03-23 오후 1.16.57.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.16.57.png">

### 2. Delete api Rule

![스크린샷 2022-03-23 오후 1.17.40.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.17.40.png">

### 3. Update default Rule to forward to drop-traffic

![스크린샷 2022-03-23 오후 1.19.30.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.19.30.png">

![스크린샷 2022-03-23 오후 1.21.49.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.21.49.png">

### Disable the Monolith

1. Update api Service

![스크린샷 2022-03-23 오후 1.22.53.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.22.53.png">

1. Change Number of tasks to 0

![스크린샷 2022-03-23 오후 1.23.38.png]<img src ="https://eunmik.github.io/bonita.blog/assets/img/2022/0325/스크린샷_2022-03-23_오후_1.23.38.png">

**You have now fully transitioned your node.js from the monolith to microservices, without any downtime!**

## **Step 6. Validate your Deployment**

- http://[DNS name]/api/users
- http://[DNS name]/api/threads
- http://[DNS name]/api/posts