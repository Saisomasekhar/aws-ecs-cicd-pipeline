# AWS ECS CI/CD Pipeline using GitHub Actions, Docker & Amazon ECR

![AWS](https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-blue?logo=githubactions)
![Docker](https://img.shields.io/badge/Docker-Containerization-blue?logo=docker)
![Amazon ECR](https://img.shields.io/badge/Amazon%20ECR-Container%20Registry-orange?logo=amazonaws)
![Amazon ECS](https://img.shields.io/badge/Amazon%20ECS-Container%20Deployment-orange?logo=amazonaws)

## 📌 Project Overview

This project demonstrates an end-to-end **CI/CD pipeline for deploying a containerized application on AWS** using **GitHub Actions, Docker, Amazon ECR, and Amazon ECS**.

The pipeline automates the application delivery process from a developer pushing source code to GitHub through Docker image creation, publishing the image to Amazon ECR, and deploying the containerized application on Amazon ECS.

### CI/CD Flow

```text
Developer
    |
    | Git Push
    v
GitHub Repository
    |
    | Workflow Trigger
    v
GitHub Actions
    |
    | Build Application
    | Build Docker Image
    | Authenticate with AWS
    | Push Docker Image
    v
Amazon ECR
    |
    | Pull Container Image
    v
Amazon ECS
    |
    v
Running Container
```

---

## 🏗️ Architecture Diagram

![AWS ECS CI/CD Architecture](![alt text](image.png))

> Place the project architecture image at `screenshots/architecture.png` for the image to display correctly on GitHub.

---

## 🎯 Project Objectives

- Automate application deployment using CI/CD.
- Integrate GitHub with GitHub Actions.
- Containerize the application using Docker.
- Build Docker images automatically.
- Push Docker images to Amazon ECR.
- Deploy the containerized application using Amazon ECS.
- Reduce manual deployment activities.
- Implement a repeatable and consistent DevOps workflow.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|---|---|
| AWS | Cloud platform |
| Git | Version control |
| GitHub | Source code repository |
| GitHub Actions | CI/CD automation |
| Docker | Application containerization |
| Amazon ECR | Docker image registry |
| Amazon ECS | Container deployment |
| AWS IAM | Authentication and authorization |
| Amazon CloudWatch | Logging and monitoring |

---

## 🔄 CI/CD Pipeline Workflow

### 1. Developer

The developer makes changes to the application and pushes the code to GitHub.

```bash
git add .
git commit -m "Update application"
git push origin main
```

### 2. GitHub

GitHub stores the source code and triggers the GitHub Actions workflow when code is pushed to the configured branch.

### 3. GitHub Actions

GitHub Actions performs the CI/CD automation:

- Checkout source code
- Build the application
- Build Docker image
- Authenticate with AWS
- Login to Amazon ECR
- Tag Docker image
- Push image to ECR
- Deploy/update the ECS service

### 4. Docker

Docker packages the application and its dependencies into a portable container image.

Example:

```bash
docker build -t my-application .
```

### 5. Amazon ECR

The Docker image is tagged and pushed to an Amazon ECR repository.

Example:

```bash
docker tag my-application:latest \
<ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/my-application:latest
```

Push the image:

```bash
docker push \
<ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com/my-application:latest
```

### 6. Amazon ECS

Amazon ECS retrieves the container image from ECR and runs it as an ECS task.

The ECS service manages the desired number of running tasks and can be integrated with an Application Load Balancer for application traffic.

---

## 📁 Project Structure

```text
aws-ecs-cicd-pipeline/
│
├── README.md
├── Dockerfile
├── .dockerignore
├── .gitignore
│
├── .github/
│   └── workflows/
│       └── cicd.yml
│
├── src/
│   └── application/
│
├── screenshots/
│   └── architecture.png
│
└── docs/
    └── deployment-guide.md
```

---

## 🐳 Dockerfile

A basic Dockerfile can be structured as:

```dockerfile
FROM nginx:alpine

COPY . /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

> Replace this example with the Dockerfile required by the actual application.

---

## ⚙️ GitHub Actions Workflow

Create:

```text
.github/workflows/cicd.yml
```

Example:

```yaml
name: Build and Deploy to ECS

on:
  push:
    branches:
      - main

env:
  AWS_REGION: us-east-1
  ECR_REPOSITORY: my-application
  ECS_CLUSTER: my-ecs-cluster
  ECS_SERVICE: my-ecs-service

permissions:
  contents: read

jobs:
  build-and-deploy:

    runs-on: ubuntu-latest

    steps:

      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: ${{ env.AWS_REGION }}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build Docker image
        run: |
          docker build \
            -t ${{ env.ECR_REPOSITORY }}:${{ github.sha }} .

      - name: Tag Docker image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          docker tag \
            ${{ env.ECR_REPOSITORY }}:${{ github.sha }} \
            $ECR_REGISTRY/${{ env.ECR_REPOSITORY }}:${{ github.sha }}

      - name: Push Docker image to ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        run: |
          docker push \
            $ECR_REGISTRY/${{ env.ECR_REPOSITORY }}:${{ github.sha }}

      - name: Deploy to ECS
        run: |
          aws ecs update-service \
            --cluster ${{ env.ECS_CLUSTER }} \
            --service ${{ env.ECS_SERVICE }} \
            --force-new-deployment
```

> For a production deployment, the ECS task definition should normally be updated to reference the newly pushed image tag before deploying.

---

## 🔐 AWS Authentication & Security

AWS credentials should never be hard-coded in the GitHub Actions workflow.

For a basic lab setup, GitHub repository secrets can be used:

```text
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
```

### Recommended Production Approach

For production environments, use **GitHub Actions OIDC with an AWS IAM role** instead of long-lived AWS access keys.

```text
GitHub Actions
      |
      | OIDC
      v
AWS IAM Role
      |
      v
AWS Services
```

Follow the principle of least privilege when creating IAM permissions.

---

## ☁️ Amazon ECR

Create an ECR repository:

```bash
aws ecr create-repository \
  --repository-name my-application \
  --region us-east-1
```

Verify the repository:

```bash
aws ecr describe-repositories \
  --repository-names my-application \
  --region us-east-1
```

List images:

```bash
aws ecr list-images \
  --repository-name my-application \
  --region us-east-1
```

---

## 🚀 Amazon ECS

The ECS environment consists of:

```text
ECS Cluster
    |
    +-- ECS Service
            |
            +-- Task Definition
                    |
                    +-- Container
                            |
                            +-- ECR Image
```

The ECS task definition controls:

- Container image
- CPU
- Memory
- Port mappings
- Environment variables
- IAM roles
- Logging configuration

---

## 🌐 Optional Application Load Balancer

For a production-style architecture, ECS can be integrated with an Application Load Balancer.

```text
Internet
   |
   v
Application Load Balancer
   |
   v
ECS Service
   |
   +----------------+
   |                |
   v                v
ECS Task 1       ECS Task 2
   |                |
   +-------+--------+
           |
           v
        ECR Image
```

---

## 📊 Monitoring and Logging

Amazon CloudWatch can be used to monitor ECS workloads.

Useful information includes:

- CPU utilization
- Memory utilization
- Running task count
- Desired task count
- ECS deployment status
- Container logs
- Application logs

Typical logging flow:

```text
ECS Container
      |
      v
CloudWatch Logs
```

---

## 🧪 Testing the Pipeline

### Clone the repository

```bash
git clone https://github.com/<YOUR_USERNAME>/aws-ecs-cicd-pipeline.git
cd aws-ecs-cicd-pipeline
```

### Make a change

```bash
git add .
git commit -m "Update application"
git push origin main
```

### Verify GitHub Actions

Open:

```text
GitHub Repository
      ↓
Actions
      ↓
Build and Deploy to ECS
```

Verify that the workflow completes successfully.

### Verify ECR

```bash
aws ecr list-images \
  --repository-name my-application \
  --region us-east-1
```

### Verify ECS

```bash
aws ecs describe-services \
  --cluster my-ecs-cluster \
  --services my-ecs-service \
  --region us-east-1
```

---

## 🔍 Troubleshooting

### GitHub Actions AWS authentication failure

Check:

- AWS credentials or OIDC configuration
- AWS region
- IAM permissions
- GitHub repository secrets

### Docker build failure

Run the build locally:

```bash
docker build -t my-application .
```

Check:

- Dockerfile
- Application dependencies
- Build context
- Required files

### ECR push failure

Verify AWS identity:

```bash
aws sts get-caller-identity
```

Authenticate with ECR:

```bash
aws ecr get-login-password \
  --region us-east-1 | \
docker login \
  --username AWS \
  --password-stdin \
  <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```

### ECS task fails to start

Check:

- ECS task definition
- ECR image URI and tag
- ECS task execution role
- Security groups
- Subnets
- Container port mappings
- CloudWatch logs

---

## 🔒 Security Best Practices

- Never commit AWS credentials to GitHub.
- Use GitHub Secrets or AWS OIDC.
- Follow IAM least-privilege permissions.
- Use private networking where appropriate.
- Restrict security group access.
- Expose only required ports.
- Use versioned Docker image tags.
- Scan container images for vulnerabilities.
- Enable CloudWatch logging.
- Regularly review IAM permissions.

---

## 📈 CI/CD Pipeline Stages

| Stage | Tool | Description |
|---|---|---|
| Source Code | GitHub | Stores application code |
| CI/CD | GitHub Actions | Automates the pipeline |
| Build | GitHub Actions | Builds the application |
| Containerization | Docker | Creates container image |
| Registry | Amazon ECR | Stores Docker image |
| Deployment | Amazon ECS | Runs container |
| Monitoring | CloudWatch | Monitors application |

---

## 🎓 Key DevOps Concepts Demonstrated

- CI/CD
- Git workflows
- GitHub Actions
- Docker containerization
- Docker image management
- Amazon ECR
- Amazon ECS
- AWS IAM
- Container deployment
- CloudWatch logging
- AWS security
- Automated deployments

---

## 🚀 Future Enhancements

The project can be extended with:

- GitHub Actions OIDC
- Terraform infrastructure provisioning
- Amazon ECS Fargate
- Application Load Balancer
- HTTPS with AWS Certificate Manager
- Route 53 DNS
- ECS Auto Scaling
- CloudWatch alarms
- Trivy container image scanning
- SonarQube code quality analysis
- Blue/Green deployment
- Canary deployment
- Automated rollback
- Slack or email deployment notifications

---

## 📌 Project Highlights

### Source Control

**GitHub**

### CI/CD

**GitHub Actions**

### Containerization

**Docker**

### Container Registry

**Amazon ECR**

### Container Deployment

**Amazon ECS**

### Monitoring

**Amazon CloudWatch**

---

## 👨‍💻 Author

**Sai Somasekhar**

AWS DevOps Engineer

### Skills

```text
AWS
DevOps
Docker
Kubernetes
Terraform
Git
GitHub
GitHub Actions
Amazon ECS
Amazon ECR
CI/CD
Linux
```

---

## ⭐ Project Purpose

This project was created for hands-on learning and demonstrating practical experience with **AWS, Docker, GitHub Actions, Amazon ECR, Amazon ECS, and CI/CD automation**.

---

## 📜 License

This project is intended for educational and portfolio purposes.
