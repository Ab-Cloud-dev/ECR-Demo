
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

Authenticate the docker client to the registry created in the previous step using the below commands.
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <account_id>.dkr.ecr.us-east-1.amazonaws.com

Build the docker file.
docker build -t crypto-app .

Tag the Image built so we can push it to the specified ECR registry.
docker tag crypto-app:latest <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest

Push the image to the ECR.
docker push <account_id>.dkr.ecr.us-east-1.amazonaws.com/crypto-app:latest
