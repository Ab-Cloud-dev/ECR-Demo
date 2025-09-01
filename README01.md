# This will be divided in two phases 

## 1) Fist testing the project locally.
## 2) Then deploying the image to ECS via ECR and accessing the environment via AWS ALP

# Phase- 01  First, Testing the Image locally

### Clone the repository:

git clone https://github.com/Ab-Cloud-dev/ECR-Demo.git


### Build the Docker Image

Navigate to the directory containing the Dockerfile and run the command.Replace your-image-name:tag with the desired name and tag for your image. 

I am giving my-crypto-app as name and tag as v1

```bash
docker build -t my-crypto-app:v1
```

 
## Run the Docker Container
 
 After building the image, run it using **docker run -d -p local-port:container-port your-image-name:tag**, adjusting local-port and container-port as necessary for your application.

 ```bash
docker run -d -p 3000:5000 my-crypto-app:v1
```
 
## Test the Application: 
 
 With the container running, you can test your application by accessing it via the local port you specified, using tools like curl, Postman, or your web browser, depending on the nature of your application.

 I am accessing the container over the port 3000



## Phase-2 deploying the image to ECS via ECR and accessing the environment via AWS ALP



## First Create a New ECR Repository

Navigate to Amazon ECR
Click the Create repository button.
Choose a visibility setting: select Private to restrict access to the repository.
Enter a name for your repository
a. Name = crypto-app
b. Keep the rest of the options as default and click on Create repository


```
 aws ecr create-repository   --repository-name "crypto-app"  --image-scanning-configuration scanOnPush=true --region us-east-1
  
```
  
 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Pushing the docker image to ECR 

In the ECR registry page, select the registry created. Click on view push commands and use commands to perform the task on the terminal provided.
Authenticate the docker client to the registry created in the previous step using the below commands.

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com
```

As we already have image, we will be pushing the image to ECR.

First Tag the Image built so we can push it to the specified ECR registry.
```
docker tag crypto-app:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest
```

Then Push the image to the ECR.

```
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest

```

 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Create an Repo
 
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ##  edit the buildspec.yml file.

In the pre_build commands section, we need to update the below lines. Copy the URI of ECR created in before step and update the below commands.

- aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 666234783044.dkr.ecr.us-east-1.amazonaws.com/crypto-app
- REPOSITORY_URI=666234783044.dkr.ecr.us-east-1.amazonaws.com/crypto-app
Run the following commands:
```
git add .
git commit -m"kodekloud-student"
git push origin main
```

 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Configure connection  to github
 
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Configure the build using the AWS CodeBuild service.
 
 Navigate to the CodeBuild service in the AWS console.
Click on create project.
Name the project ascodebuild-crypto-app.
In the source section, click on the drop-down menu and select Gitlab Self Managed.
Select Use override credentials for this project only
In connections bar click on drop-down and select crypto-app.
For the repository section, select cyrpto-app rep usrl end point from drop-down menu.
In the Environment section, for the service role section, select Existing service role and select codebuild-service-role and leave the rest of the options default.
Under the build spec section, select option Use a buildspec file and enter the name as buildspec.yml.
Leave the rest of the options as default and click on Create build project.

 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## start build 
 
 Now click on start build on the right corner of the project that you created. while the build is happening continue to explore the project page. 
 Once the build is completed you can see the docker image is uploaded to ECR crypto-app created in this lab.
 
 
 
 
 
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Configure connection  to github
 
 
 
 In this lab, you will create an ECS cluster and deploy a service on AWS ECS. You will use the EC2 launch type for the ECS cluster.

Deploy the Docker image manually using the AWS Management Console on ECS.
The ECS cluster should be ready, and you will create a task definition and service.
After that, you will deploy the service on AWS ECS, all via the console.


  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Create an ECS Cluster
To create an ECS cluster named microservices-cluster with the EC2 launch type, you would follow these steps in the AWS Management Console:

Sign in to the AWS Management Console using the provided credentials.
Navigate to the Amazon ECS service page.
Click on "Clusters" in the left navigation pane.
Click the "Create Cluster" button.
Provide the following details:

Cluster configuration:
Cluster name: microservices-cluster
Infrastructure configuration:
Amazon EC2 Instance
On-Demand Instance
Operating System: Amazon Linux 2
Instance type: t3a.micro
Instance role: ecs-microservices-cluster
Min/Max/Desired: 1/1/3
SSH key pair: None
Root EBS volume size: 30 GB
Networking configuration:
VPC: Default VPC
Subnets: All Default Subnets
Security group (Existing): microservices-sg
Auto-assign public IP: Enabled
Finally, click "Create" to create the ECS cluster.

Run CodeBuild Project
In a new browser tab, navigate to the AWS CodeBuild service page.

While the cluster is being created, you can proceed with the following tasks:

Run Build for CodeBuild project codebuild-crypto-app to build the docker image and push it to the ECR repository.

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## create a task definition
 
 o create a task definition named crypto-app with the specified details, follow these steps:

Navigate to the Amazon ECS service page within the AWS Management Console.
Click on "Task Definitions" in the left navigation pane.
Click the "Create new Task Definition" button.
Select "Amazon EC2 instances" for the launch type compatibility and click "Next step".
Enter crypto-app as the name of the task definition.
For "Task Execution Role", select ecsTaskExecutionRole.
Set "Task memory (GB)" to 0.5 GB and "Task CPU (vCPU)" to 0.25 vCPU.
Click "Add container" to define the container for this task.
For "Container name", enter crypto-app.
In the "Image" field, enter the Image URL you got from the ECR repository for crypto-app.
Under "Port mappings", set the "Container port": 5000, "Port name": 5000 of protocol HTTP.
For "Log configuration", select "Auto-configure CloudWatch Logs". Specify the Log Group Name as /ecs/crypto-app.
Click "Create" to create the task definition.


  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ##  create an ECS service

To create an ECS service named crypto-app with the specified details, follow these steps:

Navigate back to the Amazon ECS service page within the AWS Management Console.
Click on "Clusters" in the left navigation pane and select your microservices-cluster.
On the "Services" tab, click "Create".
Select the crypto-app task definition of the latest revision.
Enter crypto-app as the service name.
Set the number of desired tasks to 1.
Under "Network configuration", select the default VPC and default subnets. For the security group, choose the existing microservices-sg.
Click "Create Service" to finish the setup.

Note: It may take a few minutes for the service to be up and running, so please be patient.

You can validate the service status by checking the service details:

After service creation, click on the service name crypto-app and check the status of the service.
In Health and Metrics, Deployments current state should be completed. In case it is not, you can check errors in the Events tab.
For looking into application logs, click on the Logs tab
