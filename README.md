# AWS 3-Tier Architecture Deployment: Java Web Application

![AWS Architecture](https://imgur.com/b9iHwVc.png)

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture Overview](#architecture-overview)
3. [Repository Structure](#repository-structure)
4. [Pre-Requisites](#pre-requisites)
5. [Infrastructure Setup (Terraform)](#infrastructure-setup-terraform)
6. [Application Setup (Java)](#application-setup-java)
7. [Security & Monitoring](#security--monitoring)
8. [Troubleshooting Guide](#troubleshooting-guide)
9. [License](#license)

---

![3-tier Architecture Diagram](https://imgur.com/3XF0tlJ.png)

## Project Overview

This repository provides an educational and practical guide to deploying a production-grade Java web application (a Login App) using AWS's robust **3-Tier Architecture**. By separating the presentation, application, and data layers, this implementation ensures high availability, scalability, and security following cloud-native best practices. 

Unlike manual CLI-based setups, this project leverages **Infrastructure as Code (IaC)** with Terraform to automate the provisioning of all AWS resources, making it reliable, repeatable, and easy to manage.

### Key Features

- **High Availability**: Multi-Availability Zone (AZ) deployment with automated failover capabilities.
- **Auto Scaling**: Dynamic resource allocation for the application tier based on workload demands.
- **Infrastructure as Code (IaC)**: Fully automated infrastructure provisioning using modular Terraform configurations.
- **Security**: Defense-in-depth approach utilizing private subnets, precise security groups, and minimal-privilege architectures.
- **Monitoring**: Comprehensive metrics and logging structured via Amazon CloudWatch.

## Architecture Overview

Our architecture spans across three distinct layers distributed over multiple Availability Zones to guarantee fault tolerance:

1. **Presentation Tier (Frontend)**
   - Traffic flows through a public-facing Application Load Balancer (ALB).
   - Designed to intelligently manage and route incoming client requests to the backend securely.

2. **Application Tier (Backend)**
   - Java Spring Boot/Tomcat application (provided in the `Java-Login-App` folder) running on EC2 instances.
   - Instances are managed automatically by an Auto Scaling Group (ASG) nestled safely within private subnets.

3. **Data Tier (Database)**
   - Amazon RDS for MySQL hosted securely in its own private subnets.
   - Configured with restrictive security groups, allowing traffic solely from the Application Tier.

## Repository Structure

The project is thoughtfully structured to divide concerns between application code and infrastructure code:

- **`Java-Login-App/`**: Contains the complete source code for the Java web application, including Maven configuration (`pom.xml`, `mvnw`) and application logic.
- **`infrastructure/`**: Contains the Terraform configuration files (`main.tf`, `variables.tf`) and sub-modules (vpc, security, rds, alb, asg, monitoring) responsible for provisioning the complete AWS environment.

## Pre-Requisites

Before you begin, ensure you have the following installed and configured on your local machine:

1. **AWS Account & CLI**
   - An active [AWS Account](https://aws.amazon.com/free/).
   - AWS CLI v2 installed and configured (`aws configure`).

2. **Infrastructure Tools**
   - **Terraform** (>= 1.0.0) installed to provision the AWS infrastructure.

3. **Development Tools**
   - **Java JDK 11** or higher.
   - **Maven** (optional, as the Maven Wrapper `mvnw` is included in the project) for building the Java application.
   - **Git** as your version control system.

## Infrastructure Setup (Terraform)

The foundation of the project is provisioned using modular Terraform code located in the `infrastructure` directory.

### 1. Initialize and Configure
Navigate to the infrastructure directory and initialize the Terraform providers and modules:
```bash
cd infrastructure
terraform init
```

Create a `terraform.tfvars` file to define your environment-specific variables:
```hcl
environment        = "dev"
aws_region         = "us-east-1"
vpc_cidr           = "192.168.0.0/16"
public_subnets     = ["192.168.1.0/24", "192.168.2.0/24"]
private_subnets    = ["192.168.3.0/24", "192.168.4.0/24"]
db_name            = "javaapp"
db_username        = "admin"
db_password        = "YourSecurePassword123!"    # Ensure this meets AWS RDS complexity rules
key_name           = "your-ec2-keypair"        # Existing EC2 Key Pair name for SSH access
```

### 2. Plan and Apply
Review the infrastructure changes before applying them:
```bash
terraform plan
```
Once you verify the intended changes, apply the configuration mathematically to AWS:
```bash
terraform apply
```
*Note: This command provisions a VPC, Subnets, NAT Gateways, the ALB, an Auto Scaling Group for EC2 instances, and the RDS MySQL database.*

## Application Setup (Java)

Once the infrastructure successfully spins up, your next natural step is compiling and deploying the application.

### 1. Build the Artifact
Navigate to the application directory and construct the build artifact using the included Maven wrapper:
```bash
cd ../Java-Login-App
./mvnw clean package -DskipTests
```

### 2. Deployment
The compiled output (typically a `.war` or `.jar` artifact found in the `/target` directory) is then deployed to the EC2 instances provisioned in our application tier, running within a container like Apache Tomcat.

Ensure the application is configured to connect to your newly provisioned RDS endpoint. 

## Security & Monitoring

### Security Implementation
- **VPC & Tiered Subnets**: Public subnets house only the ALB and NAT Gateways. Application instances and databases reside strictly secured in private subnets without public IPs.
- **Security Groups**: Granular network rules dynamically ensure that the ALB can only reach the EC2 instances on specific ports, and the Database tier solely accepts SQL port connections stemming out of the Application ASG.

### Monitoring Setup
- **Amazon CloudWatch**: The dedicated Terraform `monitoring` module constructs metrics for CPU utilization, memory thresholds, database connections, and ASG health reporting.
- **Logging**: System setups ensure that foundational logs from the instances and the database are pushed to CloudWatch Logs for centralized debugging and long-term retention.

## Troubleshooting Guide

- **Database Connection Failure**: 
  - Ensure your EC2 instances successfully deployed in the designated private subnets.
  - Check the RDS Security Group specifically to confirm it explicitly allows inbound traffic (Port 3306) from the Application Layer's Security Group.
- **Application Not Responding / Site Unreachable**: 
  - Check the ALB target group health checks in the AWS Console. 
  - If EC2 instances are marked `unhealthy`, the application likely either failed to start up or isn't serving on the port the ALB expects. Use a Bastion host (or AWS Systems Manager Session Manager) to connect securely to the private instance and inspect application container logs.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
