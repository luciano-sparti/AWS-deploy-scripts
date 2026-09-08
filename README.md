# AWS ECS Fargate Deployment & Lifecycle Automation

A suite of automation scripts and declarative configurations for building, deploying, and managing containerized microservices on **AWS Elastic Container Service (ECS)** with **AWS Fargate**, **Amazon ECR**, **Amazon EFS**, and **CloudWatch/OpenTelemetry** observability.

---

## 📁 Repository Structure

```text
├── variables.sh                     # Centralized AWS environment configuration & sizing parameters
├── service_up.sh                    # Configures ECS Fargate cluster profile and scales service up
├── service_start.sh                 # Starts service tasks on existing ECS clusters
├── service_stop.sh                  # Stops running service tasks
├── service_down.sh                  # Tears down and removes ECS service definitions
├── microservice/                    # Standard microservice deployment definition
│   ├── docker-compose.yml           # App container spec + OpenTelemetry (ADOT) metrics collector sidecar
│   └── ecs-params.yml               # Fargate task sizes, awsvpc networking, EFS encryption & Service Discovery
└── microservice-ECS-OS-Index/       # Specialized OpenSearch indexer service definition
```

---

## 🏗️ Architecture & Features

- **Serverless Compute (AWS Fargate)**: Provision and run containers without managing underlying EC2 instances.
- **Private Networking (`awsvpc`)**: Deploys tasks inside private VPC subnets with attached Security Groups and Service Discovery (private DNS namespaces).
- **Persistent Storage (Amazon EFS)**: Encrypted in-transit volume mounts via EFS Access Points for application configuration persistence.
- **Observability & Metrics**:
  - Direct log streaming to **Amazon CloudWatch Logs** via the `awslogs` log driver.
  - Container and task metrics ingestion via **AWS Distro for OpenTelemetry (ADOT)** collector sidecar (`amazon/aws-otel-collector`).
- **Load Balancing**: Native integration with **Application Load Balancer (ALB)** Target Group ARNs.

---

## ⚙️ Prerequisites

Ensure the following tools are installed and configured on your management machine:

- **[AWS CLI v2](https://aws.amazon.com/cli/)** (with appropriate IAM deployment permissions)
- **[Amazon ECS CLI](https://github.com/aws/amazon-ecs-cli)** (`ecs-cli`)
- **[Docker Engine](https://docs.docker.com/engine/)**

---

## 🚀 Usage Workflow

### 1. Build & Push Docker Images to Amazon ECR

Authenticate Docker to your Amazon ECR registry, tag your local container image, and push:

```bash
# 1. Authenticate Docker with Amazon ECR
aws ecr get-login-password --region <AWS_REGION> | docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.<AWS_REGION>.amazonaws.com

# 2. Tag and push container image
docker tag <LOCAL_IMAGE>:<TAG> <REPOSITORY_URL>/<MICROSERVICE_NAME>:<VERSION>
docker push <REPOSITORY_URL>/<MICROSERVICE_NAME>:<VERSION>
```

### 2. Configure Deployment Environment

Edit `variables.sh` with your AWS infrastructure IDs (VPC, Subnets, EFS, Security Groups, Target Groups, Task CPU/Memory limits):

```bash
# Export infrastructure variables into current shell session
source variables.sh
```

### 3. Deploy & Scale Services (`service_up.sh`)

Deploy the microservice definition to the ECS Fargate cluster:

```bash
export MICROSERVICE_NAME="microservice"
./service_up.sh
```

### 4. Lifecycle Management Commands

| Action | Command | Description |
| :--- | :--- | :--- |
| **Start Service** | `./service_start.sh` | Starts tasks on the target ECS cluster. |
| **Stop Service** | `./service_stop.sh` | Stops all active task instances for maintenance. |
| **Scale / Deploy** | `./service_up.sh` | Sets task desired count (`scale 1`) and deploys updates. |
| **Teardown Service** | `./service_down.sh` | Safely removes the service and deregisters task definitions. |

---

## 🔒 Security Best Practices

- **Never commit active AWS credentials or database passwords** in plaintext files. Use IAM execution roles (`ECS_EXECUTION_ROLE`) and AWS Secrets Manager / Parameter Store for sensitive environment variables.
- Place all ECS tasks within **private subnets** with `assign_public_ip: DISABLED`, routing ingress through ALBs or API Gateways.
