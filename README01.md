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



# <h1 style="color: red;">Phase-2 Deploying the image to ECS via ECR and accessing the environment via AWS ALP</h1>



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
 ## Pushing the Docker image to ECR 

In the ECR registry page, select the registry created. Click on **View Push Commands** and use commands to perform the task on the terminal provided.
Authenticate the docker client to the registry created in the previous step using the below commands.

```
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com
```

As we already have an image, we will be pushing the image to ECR.

First, tag the Image built so we can push it to the specified ECR registry.
```
docker tag crypto-app:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest
```

Then Push the image to the ECR.

```
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest

```

 
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Create an ECS Cluster
To create an ECS cluster named microservices-cluster with the EC2 launch type, you would follow these steps in the AWS Management Console:

Sign in to the AWS Management Console using the provided credentials.Navigate to the Amazon ECS service page.
Click on "Clusters" in the left navigation pane.Click the "Create Cluster" button.
Provide the following details:

```
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
```


  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## create a task definition
 
 o create a task definition named crypto-app with the specified details, follow these steps:

Navigate to the Amazon ECS service page within the AWS Management Console.Click on "Task Definitions" in the left navigation pane.
Click the "Create new Task Definition" button.

```
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
```

  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ##  create an ECS service

To create an ECS service named crypto-app with the specified details, follow these steps:

1. Navigate back to the Amazon ECS service page within the AWS Management Console.
2. Click on "Clusters" in the left navigation pane and select your microservices-cluster.
3. On the "Services" tab, click "Create".
4. Select the crypto-app task definition of the latest revision.
5. Enter crypto-app as the service name.
6. Set the number of desired tasks to 1.
7. Under "Network configuration", select the default VPC and default subnets. For the security group, choose the existing microservices-sg.
8. Click "Create Service" to finish the setup.

Note: It may take a few minutes for the service to be up and running, so please be patient.

You can validate the service status by checking the service details:

After service creation, click on the service name crypto-app and check the status of the service.
In Health and Metrics, Deployments current state should be completed. In case it is not, you can check errors in the Events tab.
For looking into application logs, click on the Logs tab






-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Create a Repo
 # Configure the build using the AWS CodeBuild service.
 
Step:-3 : Create AWS Code Build Project

1.Navigate to AWS Codebuild console and click on “create project”.

2. Provide a name for it.

3. Under source select github as a source provider.

4. Select Connect using OAuth.


5. After this it will ask for permissions and github login do all the stuff.


6. Under GitHub repo, select the one your application code relies.

7. Under Environment leave all of them as default.

8. Under Buildspec select “Use a buildspec file” and provide the name as “buildspec.yaml”.

9. Under Artifacts Use an already created s3 bucket.

10. Click on “Update project”.

11. In IAM click on the role that the codebuild created.

12. Give “AmazonSSMFullAccess” to access the parameters in Systems Manager and “AWSS3FullAccess” to upload the artifacts.

13. Click on “Start build”.
Upon successful build it will look like:

 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## start build 
 
In a new browser tab, navigate to the AWS CodeBuild service page.

While the cluster is being created, you can proceed with the following tasks:

Run Build for CodeBuild project codebuild-crypto-app to build the docker image and push it to the ECR repository.
  -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ##  edit the buildspec.yml file.

In the pre-build commands section, we need to update the lines below.
```
AWS_ACCOUNT_ID: "123456789012"
AWS_DEFAULT_REGION: "us-west-1"
ECR_REPOSITORY_NAME: "my-app-repo"
ECS_CONTAINER_NAME: "my-app-container"
```

And also Copy the URI of the ECR created in before step and update the below commands.
```
- aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 666234783044.dkr.ecr.us-east-1.amazonaws.com/crypto-app
- REPOSITORY_URI=666234783044.dkr.ecr.us-east-1.amazonaws.com/crypto-app
```
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
 
Step:-3 : Create AWS Code Build Project

1.Navigate to AWS Codebuild console and click on “create project”.

2. Provide a name for it.

3. Under source select github as a source provider.

4. Select Connect using OAuth.


5. After this it will ask for permissions and github login do all the stuff.


6. Under GitHub repo, select the one your application code relies.

7. Under Environment leave all of them as default.

8. Under Buildspec select “Use a buildspec file” and provide the name as “buildspec.yaml”.

9. Under Artifacts Use an already created s3 bucket.

10. Click on “Update project”.

11. In IAM click on the role that the codebuild created.

12. Give “AmazonSSMFullAccess” to access the parameters in Systems Manager and “AWSS3FullAccess” to upload the artifacts.

13. Click on “Start build”.
Upon successful build it will look like:

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
