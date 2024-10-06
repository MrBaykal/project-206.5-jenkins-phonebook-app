# Project-206.5 Jenkins Phonebook App Deployment on AWS ECR and EKS using CI/CD Pipeline

This project aims to deploy a Phonebook application consisting of two microservices, leveraging Kubernetes clusters. The deployment process is managed by a CI/CD pipeline using **Jenkins**. The Docker images for the services will be built, pushed to an AWS ECR (Elastic Container Registry), and deployed to AWS EKS (Elastic Kubernetes Service) using Kubernetes manifests. This README will guide you through the detailed steps to complete the project.

## Project Overview

The project includes the following:

1. **Two Microservices**:
   - Web Server
   - Result Server

2. **Technologies**:
   - Jenkins for CI/CD Pipeline
   - AWS ECR for Docker image storage
   - AWS EKS for Kubernetes cluster and deployment
   - Docker for containerization
   - Kubernetes for orchestration

3. **Steps Involved**:
   - Build Docker images for both services.
   - Push the images to AWS ECR.
   - Deploy the application on an EKS Cluster using Kubernetes manifests.
   - Automate the entire process using Jenkins pipelines.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Step-by-Step Guide](#step-by-step-guide)
   - [Step 1: Set up GitHub Private Repository](#step-1-set-up-github-private-repository)
   - [Step 2: Configure Jenkins Server](#step-2-configure-jenkins-server)
   - [Step 3: Configure Jenkins Pipeline (Jenkinsfile)](#step-3-configure-jenkins-pipeline-jenkinsfile)
   - [Step 4: Set up AWS ECR Repository](#step-4-set-up-aws-ecr-repository)
   - [Step 5: Build and Push Docker Images](#step-5-build-and-push-docker-images)
   - [Step 6: Deploy to EKS Cluster](#step-6-deploy-to-eks-cluster)
   - [Step 7: Automate CI/CD with Jenkins](#step-7-automate-ci/cd-with-jenkins)
3. [Jenkins Pipeline Stages](#jenkins-pipeline-stages)
4. [Infrastructure as Code: Cluster Configuration](#infrastructure-as-code-cluster-configuration)
5. [Improvements and Best Practices](#improvements-and-best-practices)
6. [Conclusion](#conclusion)

---

## Prerequisites

Before starting this project, ensure that you have the following:

- **AWS Account** with IAM user permissions to create ECR and EKS.
- **GitHub Private Repository** to store your project code and configuration files.
- **Jenkins Server** up and running (either local or on a cloud instance like AWS EC2).
- **Docker** installed on your Jenkins server.
- **AWS CLI**, **eksctl**, and **kubectl** installed.

## Step-by-Step Guide

### Step 1: Set up GitHub Private Repository

1. Create a **private repository** on GitHub.
2. Push the following files to your GitHub repository:
   - `Dockerfile` for both the **web-server** and **result-server**.
   - Kubernetes manifest files for deployment (`deployment.yml`, `service.yml`).
   - `Jenkinsfile` that defines your pipeline stages.
   
3. Generate a **GitHub Token** for authentication from Jenkins.

### Step 2: Configure Jenkins Server

1. Install Jenkins on a server (or EC2 instance).
   - Install necessary **Jenkins Plugins**:
     - `Docker Pipeline`
     - `Kubernetes CLI`
     - `Amazon ECR Plugin`
     - `GitHub Integration Plugin`
   
2. Configure Jenkins to use the **GitHub Token** to access your private repository:
   - Navigate to `Jenkins -> Manage Jenkins -> Credentials`.
   - Add the **GitHub Token** under the credentials for Jenkins.

### Step 3: Configure Jenkins Pipeline (Jenkinsfile)

Your `Jenkinsfile` will define the CI/CD pipeline stages:

1. Set up the environment in the `Jenkinsfile`.
2. Build the Docker images for the **web-server** and **result-server**.
3. Tag and push the images to AWS ECR.
4. Update the **image names** in the Kubernetes manifest files with the ECR image names.
5. Install `eksctl` and `kubectl` on Jenkins to manage EKS.
6. Deploy the updated manifest files to the EKS cluster.

Sample `Jenkinsfile` structure:

```groovy
pipeline {
    agent any
    environment {
        AWS_DEFAULT_REGION = 'us-east-1'
        ECR_REGISTRY = 'AWS_ACCOUNT_ID.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com'
        EKS_CLUSTER_NAME = 'phonebook-cluster'
    }
    stages {
        stage('Clone repository') {
            steps {
                git 'https://github.com/your-username/project-206.5-jenkins-phonebook-app.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("${ECR_REGISTRY}/phonebook-web-server")
                    dockerImage.tag("${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image to ECR') {
            steps {
                script {
                    docker.withRegistry("https://${ECR_REGISTRY}", 'ecr:login') {
                        dockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }
        stage('Update K8s Manifests') {
            steps {
                sh 'sed -i "s|IMAGE_PLACEHOLDER|${ECR_REGISTRY}/phonebook-web-server:${env.BUILD_NUMBER}|g" k8s/deployment.yml'
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}
```

### Step 4: Set up AWS ECR Repository

1. Create a **Private ECR Repository** in your AWS account:
   - Navigate to AWS Console -> ECR -> Create Repository.
   - Name it `phonebook-web-server` and `phonebook-result-server`.

2. In your `Jenkinsfile`, update the `ECR_REGISTRY` variable with the correct AWS account details.

### Step 5: Build and Push Docker Images

1. Jenkins will use the `Dockerfile` to build images for both services.
2. Tag the images according to the build number or version number.
3. Push the images to the **AWS ECR** repository created in Step 4.

### Step 6: Deploy to EKS Cluster

1. Install **eksctl** and **kubectl** on Jenkins to create and manage the EKS cluster:
   - Install using:
     ```bash
     curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/0.62.0/eksctl_Linux_amd64.tar.gz" | tar xz -C /usr/local/bin
     ```

2. Use `eksctl` to create an EKS cluster using a configuration YAML (`clusterconfig.yml`):
   - Example:
     ```yaml
     apiVersion: eksctl.io/v1alpha5
     kind: ClusterConfig
     metadata:
       name: phonebook-cluster
       region: us-east-1
     nodeGroups:
       - name: ng-1
         instanceType: t2.micro
         desiredCapacity: 2
     ```

3. Once the cluster is ready, deploy the Kubernetes manifests (`deployment.yml` and `service.yml`).

### Step 7: Automate CI/CD with Jenkins

The Jenkins pipeline will now:

1. Build Docker images.
2. Push them to AWS ECR.
3. Update Kubernetes manifests.
4. Deploy the app on AWS EKS.

## Jenkins Pipeline Stages

- **Stage 1: Clone Repository**: Pull code from the GitHub repository.
- **Stage 2: Build Docker Image**: Use Docker to build the app's image.
- **Stage 3: Push to ECR**: Push the built image to AWS ECR.
- **Stage 4: Update Kubernetes Manifests**: Replace the image placeholders with the ECR image link.
- **Stage 5: Deploy to EKS**: Apply the Kubernetes manifests to deploy the app on the EKS cluster.

## Infrastructure as Code: Cluster Configuration

You can define your cluster configuration using `eksctl` in a YAML file (`clusterconfig.yml`). This makes your infrastructure easily reproducible and manageable.

Example of `clusterconfig.yml`:

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig
metadata:
  name: phonebook-cluster
  region: us-east-1
nodeGroups:
  - name: ng-1
    instanceType: t3.micro
    desiredCapacity: 2
    maxSize: 3
    minSize: 1
```

## Improvements and Best Practices

- **Docker Image Optimization**: Ensure minimal image size using multistage builds.
- **Kubernetes Best Practices**: Use Helm charts for Kubernetes deployment if the complexity increases.
- **Monitoring**: Implement monitoring using AWS CloudWatch or Prometheus for the cluster.
- **Security**: Use AWS IAM roles for EKS to securely access AWS resources.
  
## Conclusion

This project demonstrates the CI/CD process of deploying a Phonebook app on AWS EKS using Jenkins. By integrating Jenkins pipelines with AWS services such as ECR and EKS, we can automate the deployment process from source code to production. This approach ensures faster and more reliable delivery of new features and updates.
