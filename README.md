# Deploying Camunda 8 on AWS

This provides step-by-step instructions to deploy Camunda 8 on AWS using ECS (Elastic Container Service) and EKS (Elastic Kubernetes Service).


## Prerequisites

- AWS Account with AdministratorAccess.
- Installed tools: [AWS CLI], [Docker], [kubectl], [eksctl], [Helm].


## Deploying Camunda 8 Using ECS

### 1. Create AWS Access Key ID and Secret Access Key

1. Log in to the [AWS Management Console].
2. Navigate to **IAM**:
   - Search for IAM in the search bar.
   - Go to **Users** > **Add Users**.
   - Enter a username.
   - Enable **Programmatic Access**.
3. Assign permissions:
   - Choose **Attach existing policies directly**.
   - Select the `AdministratorAccess` policy.
4. Download the `.csv` file containing:
   - Access Key ID
   - Secret Access Key

### 2. Install AWS CLI

1. Verify the installation:
   ```bash
   aws --version
   ```
2. Configure AWS CLI:
   ```bash
   aws configure
   ```
   Provide the following:
   - Access Key ID
   - Secret Access Key
   - Default region
   - Default output format

### 3. Set Up AWS Resources

#### 3.1 Create a VPC

1. Navigate to **VPC** in the AWS Console.
2. Create a new VPC:
   - Select **Create VPC** > **VPC Only**.
   - Name your VPC.
   - Set IPv4 CIDR block.
3. Create Subnets:
   - Go to **Subnets** > **Create Subnet**.
   - Create a public subnet and a private subnet.
4. Attach an Internet Gateway:
   - Go to **Internet Gateway** > **Create Internet Gateway**.
   - Attach it to your VPC.

#### 3.2 Create Security Groups

1. Navigate to **Security Groups** in the VPC dashboard.
2. Create a new security group with the following inbound rules:
   - HTTP (Port 80) – Source: Anywhere
   - TCP (Port 8080) – Source: Anywhere
   - TCP (Ports 26500–26501) – Source: Anywhere
   - SSH (Port 22) – Source: Your IP

#### 3.3 Set Up RDS (PostgreSQL Database)

1. Navigate to **RDS** in the AWS Console.
2. Create a PostgreSQL database:
   - Engine: PostgreSQL (13+)
   - Instance size: `db.t3.micro`
   - Enable public access
   - Assign your security group
3. Note the database endpoint for configuration.

### 4. Deploy Camunda 8 on AWS

#### 4.1 Set Up ECS Cluster

1. Navigate to **ECS** in the AWS Console.
2. Create a new cluster:
   - Select **Fargate** as the launch type.
   - Name your cluster.

#### 4.2 Prepare Camunda Docker Image

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

#### 4.3 Deploy Camunda in ECS

1. Create a Task Definition:
   - Navigate to **Task Definitions** > **Create New Task Definition**.
   - Select **Fargate** as the launch type.
   - Add the Camunda container:
     - Image: `<ecr_url>/camunda`
     - Port mappings: `8080`, `26500–26501`
2. Deploy the Task:
   - Go to your ECS cluster > **Services** > **Create Service**.
   - Select the Task Definition and desired number of tasks.

### 5. Validate Deployment

1. Access the Camunda Web UI:
   - Navigate to the public IP of the ECS service:
2. Log in using the default credentials:
   - Username: `demo`
   - Password: `demo`

---

## Deploying Camunda 8 Using EKS

### 2. Install Required Tools

1. Install [kubectl].
   ```bash
   kubectl version --client
   ```
2. Install [eksctl].
   ```bash
   eksctl version
   ```
3. Install [Helm].
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
