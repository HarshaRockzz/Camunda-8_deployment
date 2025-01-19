# Deploying Camunda 8 on AWS

This guide provides step-by-step instructions to deploy Camunda 8 on AWS using ECS (Elastic Container Service) and EKS (Elastic Kubernetes Service). Each section covers prerequisites, configurations, and validation steps.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Deploying Camunda 8 Using ECS](#deploying-camunda-8-using-ecs)
   - [AWS Account Setup](#1-create-an-aws-account)
   - [AWS Access Key Setup](#2-create-aws-access-key-id-and-secret-access-key)
   - [Install AWS CLI](#3-install-aws-cli)
   - [Set Up AWS Resources](#4-set-up-aws-resources)
   - [Deploy Camunda on ECS](#5-deploy-camunda-8-on-aws)
   - [Validate Deployment](#6-validate-deployment)
3. [Deploying Camunda 8 Using EKS](#deploying-camunda-8-using-eks)
   - [Install Required Tools](#2-install-required-tools)
   - [Set Up EKS Cluster](#3-set-up-eks-cluster)
   - [Deploy Camunda Using Helm](#5-deploy-camunda-8-on-eks)
4. [Validation](#7-validate-the-deployment)

---

## Prerequisites

- AWS Account with AdministratorAccess.
- Installed tools: [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html), [Docker](https://docs.docker.com/get-docker/), [kubectl](https://kubernetes.io/docs/tasks/tools/), [eksctl](https://eksctl.io/), [Helm](https://helm.sh/docs/intro/install/).

---

## Deploying Camunda 8 Using ECS

### 1. Create an AWS Account

1. Visit the [AWS Sign-Up Page](https://aws.amazon.com/).
2. Provide the following details:
   - Email address
   - Password
   - AWS account name (e.g., "CamundaDeployment")
3. Complete the billing process and identity verification via phone OTP.
4. Choose the Free Tier Plan (if eligible).

### 2. Create AWS Access Key ID and Secret Access Key

1. Log in to the [AWS Management Console](https://aws.amazon.com/console/).
2. Navigate to **IAM**:
   - Search for IAM in the search bar.
   - Go to **Users** > **Add Users**.
   - Enter a username (e.g., `camunda-admin`).
   - Enable **Programmatic Access**.
3. Assign permissions:
   - Choose **Attach existing policies directly**.
   - Select the `AdministratorAccess` policy.
4. Download the `.csv` file containing:
   - Access Key ID
   - Secret Access Key

### 3. Install AWS CLI

1. Download and install AWS CLI from the [official guide](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
2. Verify the installation:
   ```bash
   aws --version
   ```
3. Configure AWS CLI:
   ```bash
   aws configure
   ```
   Provide the following:
   - Access Key ID
   - Secret Access Key
   - Default region (e.g., `us-east-1`)
   - Default output format (`json`)

### 4. Set Up AWS Resources

#### 4.1 Create a VPC

1. Navigate to **VPC** in the AWS Console.
2. Create a new VPC:
   - Select **Create VPC** > **VPC Only**.
   - Name your VPC (e.g., `camunda-vpc`).
   - Set IPv4 CIDR block: `10.0.0.0/16`.
3. Create Subnets:
   - Go to **Subnets** > **Create Subnet**.
   - Create a public subnet (`10.0.1.0/24`) and a private subnet (`10.0.2.0/24`).
4. Attach an Internet Gateway:
   - Go to **Internet Gateway** > **Create Internet Gateway**.
   - Attach it to your VPC.

#### 4.2 Create Security Groups

1. Navigate to **Security Groups** in the VPC dashboard.
2. Create a new security group with the following inbound rules:
   - HTTP (Port 80) – Source: Anywhere
   - TCP (Port 8080) – Source: Anywhere
   - TCP (Ports 26500–26501) – Source: Anywhere
   - SSH (Port 22) – Source: Your IP

#### 4.3 Set Up RDS (PostgreSQL Database)

1. Navigate to **RDS** in the AWS Console.
2. Create a PostgreSQL database:
   - Engine: PostgreSQL (13+)
   - Instance size: `db.t3.micro`
   - Enable public access
   - Assign your security group
3. Note the database endpoint for configuration.

### 5. Deploy Camunda 8 on AWS

#### 5.1 Set Up ECS Cluster

1. Navigate to **ECS** in the AWS Console.
2. Create a new cluster:
   - Select **Fargate** as the launch type.
   - Name your cluster (e.g., `camunda-cluster`).

#### 5.2 Prepare Camunda Docker Image

1. Install Docker and pull the Camunda image:
   ```bash
   docker pull camunda/camunda-platform:latest
   ```
2. Push the image to AWS ECR:
   - Create an ECR repository:
     ```bash
     aws ecr create-repository --repository-name camunda
     ```
   - Tag and push the image:
     ```bash
     docker tag camunda/camunda-platform:latest <your_ecr_url>/camunda
     docker push <your_ecr_url>/camunda
     ```

#### 5.3 Deploy Camunda in ECS

1. Create a Task Definition:
   - Navigate to **Task Definitions** > **Create New Task Definition**.
   - Select **Fargate** as the launch type.
   - Add the Camunda container:
     - Image: `<your_ecr_url>/camunda`
     - Port mappings: `8080`, `26500–26501`
2. Deploy the Task:
   - Go to your ECS cluster > **Services** > **Create Service**.
   - Select the Task Definition and desired number of tasks.

### 6. Validate Deployment

1. Access the Camunda Web UI:
   - Navigate to the public IP of the ECS service:
     ```
     http://<public-ip>:8080
     ```
2. Log in using the default credentials:
   - Username: `demo`
   - Password: `demo`

---

## Deploying Camunda 8 Using EKS

### 2. Install Required Tools

1. Install [kubectl](https://kubernetes.io/docs/tasks/tools/).
   ```bash
   kubectl version --client
   ```
2. Install [eksctl](https://eksctl.io/).
   ```bash
   eksctl version
   ```
3. Install [Helm](https://helm.sh/docs/intro/install/).
   ```bash
   helm version
   ```

### 3. Set Up EKS Cluster

1. Create an EKS cluster:
   ```bash
   eksctl create cluster \
     --name camunda-cluster \
     --region us-east-1 \
     --nodes 3 \
     --node-type t3.medium
   ```
2. Verify the cluster:
   ```bash
   kubectl get nodes
   ```

### 5. Deploy Camunda 8 on EKS

#### 5.1 Add Camunda Helm Repository

1. Add and update the Helm chart repository:
   ```bash
   helm repo add camunda https://helm.camunda.io
   helm repo update
   ```

#### 5.2 Prepare Custom `values.yaml`

Example `values.yaml`:
```yaml
global:
  postgresql:
    enabled: false
  camunda:
    enabled: true

postgresql:
  externalDatabase:
    host: <your-rds-endpoint>
    user: <db-username>
    password: <db-password>
    database: camunda

zeebe:
  broker:
    gateway:
      enable: true

operate:
  replicas: 1
```

#### 5.3 Install Camunda Using Helm

1. Deploy Camunda:
   ```bash
   helm install camunda camunda/camunda-platform -f values.yaml --namespace camunda --create-namespace
   ```
2. Verify the deployment:
   ```bash
   kubectl get pods -n camunda
   ```

---

## Validation

### Expose Camunda Services

1. Use a Kubernetes LoadBalancer:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: camunda
     namespace: camunda
   spec:
     type: LoadBalancer
     ports:
     - port: 8080
       targetPort: 8080
       protocol: TCP
     selector:
       app.kubernetes.io/name: camunda-platform
   ```
2. Apply the service:
   ```bash
   kubectl apply -f service.yaml
   ```
3. Retrieve the external IP:
   ```bash
   kubectl get svc -n camunda
   ```

### Access the Camunda Web UI

1. Use the external IP provided by the LoadBalancer:
   ```
   http://<external-ip>:8080
   ```
2. Log in using the default credentials:
   - Username: `demo`
   - Password: `demo`

---

## Challenges and Solutions

Document challenges faced and their solutions during deployment.

---

## License

This project is licensed under the MIT License. See `LICENSE` for details.

