
# Clone the repository:

git clone https://github.com/your-username/microservices-project.git


## Build the Docker Image

Navigate to the directory containing the Dockerfile and run the command docker build -t your-image-name:tag ..
 Replace your-image-name:tag with the desired name and tag for your image.
 
## Run the Docker Container
 
 After building the image, run it using docker run -d -p local-port:container-port your-image-name:tag, adjusting local-port and container-port as necessary for your application.
 
## Test the Application: 
 
 With the container running, test your application by accessing it via the local port you specified, using tools like curl, Postman, or your web browser, depending on the nature of your application.


Deploying the Docker to AWS ECS steps

# First Create a New ECR Repository

Navigate to Amazon ECR
Click the Create repository button.
Choose a visibility setting: select Private to restrict access to the repository.
Enter a name for your repository
a. Name = crypto-app
b. Keep the rest of the options as default and click on Create repository


```
root@aws-client ~ on ☁️  (us-east-1) ➜  aws ecr create-repository \
  --repository-name "crypto-app" \
  --image-scanning-configuration scanOnPush=true \
  --region us-east-1
  
```
  
 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
 ## Pushing the docker image to ECR 

 
  Navigate to the crypto-app directory on the VS code terminal and build the docker image using the docker file. Name the docker image crypto-app.
In the ECR registry page, select the registry created. Click on view push commands and use commands to perform the task on the terminal provided.

```

#Authenticate the docker client to the registry created in the previous step using the below commands.
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com

#Build the docker file.
docker build -t crypto-app .

#Tag the Image built so we can push it to the specified ECR registry.
docker tag crypto-app:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest

#Push the image to the ECR.
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest
```


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

##  create an Application Load Balancer (ALB) and a target group.


1) Create an Application Load Balancer (ALB) and a target group. 
Assign the target group to the ECS service in order to serve the application through the Application Load Balancer.
Verify that the application is accessible through the Application Load Balancer Endpoint.


aws elbv2 create-target-group --name "crypto-app" --target-type "instance" --health-check-path "/" --healthy-threshold-count "5" --unhealthy-threshold-count "2" --health-check-timeout-seconds "5" --health-check-interval-seconds "30" --matcher '{"HttpCode":"200"}' --port "5000" --protocol "HTTP" --protocol-version "HTTP1" --ip-address-type "ipv4" --health-check-port "traffic-port" --health-check-protocol "HTTP" --vpc-id "vpc-0c8da31da25c03d00" 
aws elbv2 register-targets --target-group-arn "arn:aws:elasticloadbalancing:us-east-1:533267155812:targetgroup/crypto-app/0ab82fafc7f12495" --targets '{"Port":5000,"Id":"i-085495f37b2689b02"}' 

Create an Application Load Balancer (ALB) with the following details:

```
1 Name: microservices-alb
2 Scheme: internet-facing
3 Listener: HTTP on port 80
4 Target group: crypto-app
5 Health check:
6 Port: 5000
7 Status code: 200
8 Path: /
9 Security group (Select existing): microservices-sg
10 VPC: default
11 Application load balancer listener rule:
12 Path: / -> Target Group: crypto-app

```

 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

##  create an ECS service

In the ECS console, navigate to the Clusters section and select the microservices-cluster.

Create a new service named crypto-app using the latest task definition for crypto-app.

Under the Load balancing section:

```
select the check box
Select the container to load balance: crypto-app:5000
Choose the Application Load Balancer and select microservices-alb from the dropdown.
Use the existing HTTP:80 listener for the Listener.
For the Target group name, select the existing crypto-app target group.
Keep the rest of the settings as default and click on the Create button.

```

Note: It may take a few minutes for the service to be up and running, so please be patient.

You can validate the service status by checking the service details:

After service creation, click on the service name crypto-app and check the status of the service.
In Health and Metrics, Deployments current state should be completed. In case it is not, you can check errors in the Events tab.
For looking into application logs, click on the Logs tab


 -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

##  Access the application through the Application Load Balancer 

Open a web browser and enter the Endpoint URL of the Application Load Balancer.

You can find the DNS URL of the Application Load Balancer in the AWS Management Console, under EC2 > Load Balancers.
Select the microservices-alb load balancer and look for the DNS name field.
You should see the application running with the login page.