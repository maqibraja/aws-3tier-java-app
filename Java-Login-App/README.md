# AWS 3-Tier Java Application

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Terraform](https://img.shields.io/badge/Terraform-IaC-purple)
![Java](https://img.shields.io/badge/Java-11-red)
![Spring Boot](https://img.shields.io/badge/Spring%20Boot-2.2.4-brightgreen)

Production-ready 3-tier architecture for deploying Java Spring Boot applications on AWS using Terraform.

![Architecture](https://imgur.com/3XF0tlJ.png)

---

## Architecture Overview

```
┌─────────────────────────────────────────┐
│     PRESENTATION TIER (Public)          │
│   Nginx + NLB + Auto Scaling Group      │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│     APPLICATION TIER (Private)          │
│   Tomcat + NLB + Auto Scaling Group     │
└─────────────────┬───────────────────────┘
                  │
┌─────────────────▼───────────────────────┐
│         DATA TIER (Private)             │
│      RDS MySQL (Multi-AZ)               │
└─────────────────────────────────────────┘
```

### Components

| Tier | AWS Services | Purpose |
|------|--------------|---------|
| **Presentation** | NLB, Nginx, EC2 (ASG) | Handle user requests |
| **Application** | NLB, Tomcat, EC2 (ASG) | Business logic |
| **Data** | RDS MySQL Multi-AZ | Database storage |

---

## Prerequisites

- AWS Account with CLI configured
- Terraform >= 1.0.0
- Java 11+
- Maven

```bash
# Configure AWS
aws configure

# Verify Terraform
terraform version
```

---

## Quick Start

### 1. Deploy Infrastructure

```bash
cd infrastructure

# Initialize
terraform init

# Create variables file
cat > terraform.tfvars << EOF
environment       = "dev"
aws_region        = "us-east-1"
vpc_cidr          = "192.168.0.0/16"
db_username       = "admin"
db_password       = "YourSecurePassword123!"
key_name          = "your-key-pair"
EOF

# Deploy
terraform plan -out=tfplan
terraform apply tfplan
```

### 2. Build Application

```bash
cd ../Java-Login-App

# Build WAR file
mvn clean package

# Run tests
mvn test
```

### 3. Database Setup

```sql
CREATE DATABASE UserDB;
USE UserDB;

CREATE TABLE Employee (
    id INT AUTO_INCREMENT PRIMARY KEY,
    first_name VARCHAR(250),
    last_name VARCHAR(250),
    email VARCHAR(250),
    username VARCHAR(250) UNIQUE,
    password VARCHAR(250),
    regdate TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Project Structure

```
aws-3tier-java-app/
├── README.md
├── infrastructure/           # Terraform configs
│   ├── main.tf
│   ├── variables.tf
│   └── modules/
│       ├── vpc/             # Networking
│       ├── security/        # Security groups
│       ├── rds/             # Database
│       ├── alb/             # Load balancer
│       └── asg/             # Auto Scaling
└── Java-Login-App/          # Spring Boot app
    ├── src/
    ├── pom.xml
    └── settings.xml
```

---

## Key Features

✅ **High Availability** - Multi-AZ deployment  
✅ **Auto Scaling** - Based on CPU/memory metrics  
✅ **Security** - VPC, security groups, encrypted data  
✅ **Monitoring** - CloudWatch metrics & logs  
✅ **Cost Optimized** - Right-sized resources  

---

## Common Commands

```bash
# Check ASG status
aws autoscaling describe-auto-scaling-groups

# View RDS instances
aws rds describe-db-instances

# Check target health
aws elbv2 describe-target-health --target-group-arn <arn>

# View CloudWatch logs
aws logs tail /aws/tomcat/application --follow
```

---

## Troubleshooting

**Can't connect to database?**
```bash
# Check security group allows port 3306
aws ec2 describe-security-groups --group-ids sg-xxx
```

**App not responding?**
```bash
# Check Tomcat logs
tail -f /opt/tomcat/logs/catalina.out

# Verify service status
sudo systemctl status tomcat
```

---

## Cleanup

```bash
cd infrastructure

# Destroy all resources
terraform destroy
```

⚠️ **Warning:** This deletes all infrastructure including the database.

---

## License

MIT License

---

<div align="center">

**Questions?** Create an issue in the repository.

</div>
