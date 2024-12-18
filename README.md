# Orchestration-and-Scaling
Orchestration and Scaling_HV
Project Link: https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices

Fork this repository. For the update from the main repository, yzou can refer to this link:

https://stackoverflow.com/questions/3903817/pull-new-updates-from-original-github-repository-into-forked-github-repository

<h1> Forking the repository <h1>

<img width="829" alt="image" src="https://github.com/user-attachments/assets/1d624290-1891-4029-97d2-848412a838f9" />

For the update from the main repository

![image](https://github.com/user-attachments/assets/713d0020-8074-420d-9efa-c279485815aa)

Followed the above steps 

There are no changes 

![image](https://github.com/user-attachments/assets/bd3e69b0-f426-4bfa-b0d2-57d92c80e6d1)

Step 1: Set Up the AWS Environment

1. Set Up AWS CLI and Boto3:
- Install AWS CLI and configure it with AWS credentials.
- Install Boto3 for Python and configure it.
This is done using the AWS terraform script (main.tf) which has been uploaded to the GIT repository.

Using the terraform script everything has been done and completed.

Step 2: Prepare the MERN Application

1. Containerize the MERN Application:
   
- Ensure the MERN application is containerized using Docker. Create a Dockerfile for each component (frontend and backend).
This is done using the AWS terraform script (main.tf) which has been uploaded to the GIT repository.
Using the terraform script everything has been done and completed.

<img width="696" alt="image" src="https://github.com/user-attachments/assets/510f7cfa-56c5-42f2-b295-82ba9abd1c16" />

2. Push Docker Images to Amazon ECR:
- Build Docker images for the frontend and backend.
- Create an Amazon ECR repository for each image.
- Push the Docker images to their respective ECR repositories.
All the docker containers are created using the AWS Terraform script it self as shown below and they are sent to the ECR.
Images of the Docker container: All the 3 containers are up and running

<img width="449" alt="image" src="https://github.com/user-attachments/assets/d42331b4-36ea-4541-9703-14e3e44e278a" />

Mutable repositories are created and are stored at 

<img width="449" alt="image" src="https://github.com/user-attachments/assets/e7ef64b4-0b25-4d66-a014-a2d83bb983c6" />

When I open the ECR repository I can see the below 3 Images as expected 1 for the front end and the other for the 2 Microservices

<img width="446" alt="image" src="https://github.com/user-attachments/assets/600812d5-f2c2-40cf-a47b-0c3c4af8172d" />

I have written a shell script which will push the contents of the repositories to ECR
```
#!/bin/bash

# Variables
AWS_REGION="us-west-2"                  # AWS region
AWS_ACCOUNT_ID="975050024946"    # Replace with your AWS account ID
REPO_NAME="samplemern-all-images"       # Name of the ECR repository

# Authenticate Docker with ECR
echo "Logging into ECR..."
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com

# Ensure the repository exists (idempotent creation)
echo "Ensuring repository $REPO_NAME exists..."
aws ecr describe-repositories --repository-names $REPO_NAME --region $AWS_REGION >/dev/null 2>&1 || aws ecr create-repository --repository-name $REPO_NAME --region $AWS_REGION

# Process all running containers
docker ps --format '{{.ID}} {{.Image}} {{.Names}}' | while read -r CONTAINER_ID IMAGE NAME; do
    # Tag for ECR
    ECR_IMAGE="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$NAME"
    
    echo "Tagging image $IMAGE for container $NAME as $ECR_IMAGE..."
    docker tag $IMAGE $ECR_IMAGE

    echo "Pushing $ECR_IMAGE to ECR..."
    docker push $ECR_IMAGE
    echo "Successfully pushed $ECR_IMAGE."
done

echo "All images have been pushed to the ECR repository $REPO_NAME."
```

Step 3: Version Control
1. Use AWS CodeCommit:
   - Create a CodeCommit repository.
   - Push the MERN application source code to the CodeCommit repository.
It has been agreed that there is no necessity to use Code Commit as it has been deprecated.
Step 4: Continuous Integration with Jenkins
1. Set Up Jenkins:
   - Install Jenkins on an EC2 instance.
   - Configure Jenkins with necessary plugins.
2. Create Jenkins Jobs:
   - Create Jenkins jobs for building and pushing Docker images to ECR.
   - Trigger the Jenkins jobs whenever there's a new commit in the CodeCommit repository.
It has been agreed that there is no necessity to use Code Commit as it has been deprecated.
I have created a Jenkins job which will be executed manually for now for moving the containers to ECR. The manual trigger of the Jenkins job will ensure that the shell script will be executed which will trigger the pushing of the container images to the AWS ECR.

Getting started in Jenkins within the Amazon EC2 Instance 

![image](https://github.com/user-attachments/assets/d18c243e-4708-4a63-9c35-0e8700b3fe85)

Creating a Jenkins Job 

<img width="1181" alt="image" src="https://github.com/user-attachments/assets/64a3983a-0373-4237-ae81-2efd523a13af" />

The pipeline had run fine after sorting the permission issues 

![image](https://github.com/user-attachments/assets/7aa95a72-b4ae-4bce-8f72-eeb68a109626)

Jenkins pipeline configuration code:

```
pipeline {
    agent any
    environment {
        AWS_REGION = 'us-west-2'
        AWS_ACCOUNT_ID = '975050024946'
        SCRIPT_PATH = '/home/ubuntu/SampleMERNwithMicroservices/push_all_to_ecr.sh'
    }
    stages {
        stage('Prepare') {
            steps {
                script {
                    echo 'Ensuring AWS CLI and Docker are accessible...'
                    sh 'aws --version'
                    sh 'docker --version'
                }
            }
        }
        stage('Push to ECR') {
            steps {
                script {
                    echo 'Running the ECR push script...'
                    sh """
                    sudo chmod +x ${SCRIPT_PATH}
                    sudo ${SCRIPT_PATH}
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Job completed!'
        }
    }
}
```

Jenkins jobs have updated the ECR Elastic container registry as expected by running the shell script 

<img width="445" alt="image" src="https://github.com/user-attachments/assets/e120db30-0f3f-4489-a87b-62905d33ca0d" />












