# WordPress on AWS CloudFormation

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![AWS](https://img.shields.io/badge/AWS-CloudFormation-orange.svg)
![WordPress](https://img.shields.io/badge/WordPress-Latest-blue.svg)

## Table of Contents

- [Project Overview](#project-overview)
- [Solution Architecture](#solution-architecture)
- [Features](#features)
- [Prerequisites](#prerequisites)
- [Deployment Instructions](#deployment-instructions)
  - [AWS Console Deployment](#aws-console-deployment)
  - [AWS CLI Deployment](#aws-cli-deployment)
  - [CI/CD with Jenkins](#cicd-with-jenkins)
- [IAM Permissions](#iam-permissions)
- [Usage & Management](#usage--management)
- [Monitoring](#monitoring)
- [Cost Optimization](#cost-optimization)
- [Troubleshooting](#troubleshooting)
- [Cleanup](#cleanup)
- [Contributing](#contributing)
- [License](#license)
- [References](#references)

## Project Overview

This project is part of the **PGP Cloud Computing & AWS Solutions Architect** program. It demonstrates the deployment of a fully functional WordPress website on AWS using Infrastructure as Code (CloudFormation).

The solution deploys a complete LAMP stack (Linux, Apache, MySQL, PHP) on a single EC2 instance with automated WordPress installation, CloudWatch monitoring, and cost optimization features through scheduled start/stop functionality.

### Project Structure

```
project-PGP-CC-AWS-Solutions-Architect/
├── README.md              # This comprehensive documentation
├── template.yaml          # Main CloudFormation template
├── Jenkinsfile           # CI/CD pipeline configuration
├── iAmRole.json          # IAM policy for deployment permissions
├── LICENSE               # MIT License
└── .git/                 # Git version control
```

### Technologies Used
- **AWS CloudFormation** - Infrastructure as Code
- **Amazon EC2** - Compute instance (t3.medium)
- **Amazon Linux 2** - Operating system
- **Apache HTTP Server** - Web server
- **PHP 8.4** - Server-side scripting
- **MariaDB 10.5** - Database server
- **WordPress** - Content Management System
- **CloudWatch** - Monitoring and alerting
- **Lambda** - Serverless functions for automation
- **EventBridge** - Scheduled events
- **Jenkins** - CI/CD pipeline automation

### Estimated Monthly Cost
- **EC2 t3.medium (if running 24/7)**: ~$30-35/month
- **With scheduled shutdown (12 hours/day)**: ~$15-18/month
- **Additional services** (CloudWatch, Lambda, EIP): <$10/month

## Solution Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Internet      │────│  Security Group  │────│   EC2 Instance  │
│   Gateway       │    │  (HTTP/HTTPS/SSH)│    │   (t3.medium)   │
│                 │    │                  │    │   WordPress     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                │                        │
                                │                        ▼
┌─────────────────┐    ┌────────▼──────────┐    ┌─────────────────┐
│  CloudWatch     │    │     Lambda        │    │   MariaDB       │
│  Alarms         │────│   Functions       │    │   Database      │
│  (CPU > 80%)    │    │ (Start/Stop EC2)  │    │   (Local)       │
└─────────────────┘    └───────────────────┘    └─────────────────┘
                                │                        │
                                ▼                        │
                       ┌──────────────────┐              │
                       │   EventBridge    │              │
                       │   Rules          │              │
                       │  (6AM/9PM CRON)  │              │
                       └──────────────────┘              │
                                                          ▼
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Jenkins       │────│     IAM Role     │────│  Elastic IP     │
│  CI/CD Pipeline │    │   Permissions    │    │  (Static IP)    │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### Key Components

1. **EC2 Instance**: Hosts the complete LAMP stack with WordPress (t3.medium)
2. **Security Group**: Controls inbound traffic (ports 22, 80, 443)
3. **Elastic IP**: Provides persistent public IP address
4. **CloudWatch Alarm**: Monitors CPU utilization and triggers notifications
5. **Lambda Functions**: Automate instance start/stop for cost optimization
6. **EventBridge Rules**: Schedule instance operations (6 AM start, 9 PM stop)
7. **IAM Role**: Provides necessary permissions for Lambda execution
8. **Jenkins Pipeline**: Automates deployment and infrastructure updates
9. **VPC/Subnet**: Network isolation and security boundaries

## Features

✅ **Automated WordPress Installation**
- Latest WordPress version download and configuration
- MariaDB database setup with secure credentials
- Apache web server configuration
- PHP 8.4 with required extensions

✅ **Security**
- Security group with minimal required ports
- SSH access for administration
- Database credentials configurable

✅ **Monitoring & Alerting**
- CloudWatch CPU utilization monitoring
- Configurable alarm thresholds
- SNS integration for notifications

✅ **Cost Optimization**
- Automated instance shutdown at 9 PM
- Automated instance startup at 6 AM
- Elastic IP for persistent public IP address

✅ **Infrastructure as Code**
- Version controlled CloudFormation templates
- Reproducible deployments
- Easy stack updates and rollbacks

## Prerequisites

### AWS Account Setup
1. **AWS Account** with administrative privileges
2. **EC2 Key Pair** in your target region
3. **AWS CLI** installed and configured (optional for CLI deployment)
4. **CloudFormation** service access

### Required IAM Permissions
Your AWS user/role needs permissions for:
- EC2 (create instances, security groups)
- IAM (create roles and policies)
- Lambda (create and manage functions)
- CloudWatch (create alarms)
- EventBridge (create rules)
- CloudFormation (create and manage stacks)

### System Requirements
- **Region**: Any AWS region with EC2 availability
- **VPC**: Existing VPC with public subnet (required parameter)
- **Instance Type**: t3.medium (can be changed)

## Deployment Instructions

### AWS Console Deployment

1. **Navigate to CloudFormation Console**
   ```
   AWS Console → CloudFormation → Create Stack
   ```

2. **Upload Template**
   - Select "Upload a template file"
   - Choose `template.yaml`
   - Click "Next"

3. **Configure Parameters**
   ```
   Stack Name: wordpress-stack
   KeyName: [Select your EC2 Key Pair]
   VpcId: [Select your VPC ID]
   SubnetId: [Select a public subnet ID]
   ```

4. **Review and Deploy**
   - Acknowledge IAM resource creation
   - Click "Create Stack"
   - Wait for deployment completion (~10-15 minutes)

### AWS CLI Deployment

1. **Clone Repository**
   ```bash
   git clone git@github.com:AmChella/cf-simplylearn.git
   cd cf-simplylearn
   ```

2. **Deploy Stack**
   ```bash
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name wordpress-stack \
     --parameter-overrides \
       KeyName=your-key-pair-name \
       VpcId=vpc-xxxxxxxxx \
       SubnetId=subnet-xxxxxxxxx \
     --capabilities CAPABILITY_NAMED_IAM CAPABILITY_IAM \
     --region us-east-1
   ```

3. **Monitor Deployment**
   ```bash
   aws cloudformation describe-stacks \
     --stack-name wordpress-stack \
     --query 'Stacks[0].StackStatus'
   ```

4. **Get Website URL**
   ```bash
   aws cloudformation describe-stacks \
     --stack-name wordpress-stack \
     --query 'Stacks[0].Outputs[?OutputKey==`WebsiteURL`].OutputValue' \
     --output text
   ```

### CI/CD with Jenkins

This project includes a Jenkins pipeline for automated deployment. The `Jenkinsfile` defines a complete CI/CD workflow.

#### Pipeline Features

- **Automatic Checkout**: Pulls the latest code from the repository
- **Template Validation**: Validates CloudFormation template syntax
- **Packaging**: Packages Lambda functions and uploads artifacts to S3
- **Deployment**: Deploys the CloudFormation stack with predefined parameters
- **Error Handling**: Archives artifacts and provides detailed error reporting

#### Environment Variables

The pipeline uses the following environment variables:

| Variable | Value | Description |
|----------|-------|-------------|
| `AWS_REGION` | us-east-1 | Target AWS region for deployment |
| `S3_BUCKET` | slean-cft-artifacts-bucket | S3 bucket for storing artifacts |
| `STACK_NAME` | my-wordpress-stack | CloudFormation stack name |
| `TEMPLATE` | template.yaml | CloudFormation template file |

#### Prerequisites for Jenkins

1. **Jenkins Server** with AWS CLI configured
2. **AWS IAM Role** with appropriate permissions (see IAM section below)
3. **S3 Bucket** for storing packaged artifacts
4. **Jenkins Plugins**: AWS Pipeline, CloudFormation Plugin

#### Pipeline Stages

1. **Checkout**: Downloads source code from repository
2. **Debug**: Displays current directory and AWS caller identity
3. **Validate**: Validates CloudFormation template syntax
4. **Package**: Packages template and uploads to S3 if needed
5. **Deploy**: Deploys the CloudFormation stack

```bash
# Example: Trigger pipeline manually
# The pipeline automatically uses these parameter overrides:
KeyName=Aadvik
VpcId=vpc-0791ac266363f50f2
SubnetId=subnet-0ee037a1617af0674
```

### Parameter Reference

| Parameter | Description | Default | Required |
|-----------|-------------|---------|----------|
| `KeyName` | EC2 Key Pair name for SSH access | - | Yes |
| `VpcId` | VPC where EC2 will be launched | - | Yes |
| `SubnetId` | Public subnet where EC2 will be launched | - | Yes |

## IAM Permissions

This project includes a comprehensive IAM policy (`iAmRole.json`) that defines the minimum required permissions for deploying and managing the WordPress infrastructure.

### Required AWS Services Permissions

The IAM policy grants permissions for:

#### CloudFormation Operations
- Create, update, delete, and describe stacks
- Manage change sets and templates
- Validate templates

#### EC2 Operations
- Launch and terminate instances
- Create and manage security groups
- Manage Elastic IP addresses
- Describe EC2 resources

#### Lambda Operations
- Create, update, and delete Lambda functions
- Manage function permissions
- Configure function settings

#### EventBridge (CloudWatch Events)
- Create and manage scheduled rules
- Configure rule targets
- Enable/disable rules

#### CloudWatch Operations
- Create and manage alarms
- Configure alarm actions

#### IAM Operations
- Create and manage service roles
- Attach and detach policies
- Pass roles to AWS services

#### S3 Operations (for CI/CD)
- Upload and download objects from artifact bucket
- List bucket contents

### Setting Up IAM Role

1. **Create IAM Role**:
   ```bash
   aws iam create-role \
     --role-name WordPress-CloudFormation-Role \
     --assume-role-policy-document file://trust-policy.json
   ```

2. **Attach Custom Policy**:
   ```bash
   aws iam put-role-policy \
     --role-name WordPress-CloudFormation-Role \
     --policy-name WordPress-Deployment-Policy \
     --policy-document file://iAmRole.json
   ```

3. **Use Role in Deployment**:
   ```bash
   aws cloudformation deploy \
     --template-file template.yaml \
     --stack-name wordpress-stack \
     --role-arn arn:aws:iam::ACCOUNT-ID:role/WordPress-CloudFormation-Role \
     --capabilities CAPABILITY_NAMED_IAM CAPABILITY_IAM
   ```

### Security Best Practices

- ✅ **Principle of Least Privilege**: Policy grants only necessary permissions
- ✅ **Resource Constraints**: Some permissions limited to specific resource patterns
- ✅ **Service-Specific**: Permissions scoped to required AWS services only
- ⚠️ **Production Note**: Consider further restricting resources by tags or naming patterns

## Usage & Management

### Accessing WordPress

1. **Get the Public URL**
   - Check CloudFormation Outputs tab
   - Or use AWS CLI command above
   - Format: `http://ec2-xx-xx-xx-xx.region.compute.amazonaws.com`

2. **Initial WordPress Setup**
   - Navigate to the public URL
   - Follow WordPress installation wizard
   - Database details are pre-configured:
     - Database Name: `wordpress`
     - Username: `admin`
     - Password: `admin` (change this in production!)

3. **SSH Access**
   ```bash
   ssh -i your-key.pem ec2-user@[public-ip-address]
   ```

### Database Management

```bash
# Connect to MariaDB
sudo mysql -u admin -p

# WordPress database
USE wordpress;
SHOW TABLES;
```

### File Management

```bash
# WordPress files location
cd /var/www/html

# Apache configuration
sudo systemctl status httpd
sudo systemctl restart httpd

# View logs
sudo tail -f /var/log/httpd/access_log
sudo tail -f /var/log/httpd/error_log
```

## Monitoring

### CloudWatch Integration

- **CPU Utilization Alarm**: Triggers when CPU > 80% for 5 minutes
- **Instance Status**: Monitor via EC2 console
- **Application Logs**: Available in `/var/log/httpd/`

### Custom Monitoring

```bash
# Check system resources
top
df -h
free -m

# Monitor Apache
sudo systemctl status httpd
sudo netstat -tlnp | grep :80
```

## Cost Optimization

### Automatic Scheduling

- **Instance stops**: Every day at 9:00 PM UTC
- **Instance starts**: Every day at 6:00 AM UTC
- **Savings**: ~50% reduction in EC2 costs

### Manual Control

```bash
# Stop instance
aws ec2 stop-instances --instance-ids [instance-id]

# Start instance
aws ec2 start-instances --instance-ids [instance-id]

# Modify schedule (update cron expressions in template)
ScheduleExpression: "cron(0 21 * * ? *)"
```

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| **Stack Creation Failed** | Check CloudFormation Events tab, verify IAM permissions |
| **WordPress Not Loading** | Check security group rules, verify Apache is running |
| **Database Connection Error** | Verify MariaDB is running: `sudo systemctl status mariadb` |
| **SSH Connection Failed** | Verify key pair name, check security group port 22 |
| **High CPU Alarm** | Check WordPress plugins, optimize database, consider scaling up |

### Debug Commands

```bash
# Check services status
sudo systemctl status httpd
sudo systemctl status mariadb

# View installation logs
sudo cat /var/log/cloud-init-output.log

# Test database connection
sudo mysql -u admin -padmin -e "SHOW DATABASES;"

# Check disk space
df -h
```

### Stack Events

```bash
# View recent stack events
aws cloudformation describe-stack-events \
  --stack-name wordpress-stack \
  --max-items 10
```

## Cleanup

⚠️ **Important**: This will permanently delete all resources and data.

### AWS Console
1. Go to CloudFormation console
2. Select your stack
3. Click "Delete"
4. Confirm deletion

### AWS CLI
```bash
aws cloudformation delete-stack --stack-name wordpress-stack

# Monitor deletion
aws cloudformation describe-stacks --stack-name wordpress-stack
```

### Manual Cleanup (if needed)
```bash
# List remaining resources
aws ec2 describe-instances --filters "Name=tag:aws:cloudformation:stack-name,Values=wordpress-stack"
```

## Contributing

1. **Fork the repository**
2. **Create a feature branch**: `git checkout -b feature/improvement`
3. **Make your changes**
4. **Test deployment** in your AWS account
5. **Submit a pull request**

### Development Guidelines
- Follow AWS CloudFormation best practices
- Update documentation for any new features
- Test in multiple AWS regions
- Ensure cost optimization features remain intact

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## References

### AWS Documentation
- [AWS CloudFormation User Guide](https://docs.aws.amazon.com/cloudformation/)
- [Amazon EC2 User Guide](https://docs.aws.amazon.com/ec2/)
- [Amazon CloudWatch User Guide](https://docs.aws.amazon.com/cloudwatch/)
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/)

### WordPress Resources
- [WordPress Installation Guide](https://wordpress.org/support/article/how-to-install-wordpress/)
- [WordPress Security](https://wordpress.org/support/article/hardening-wordpress/)

### AWS Best Practices
- [Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Cost Optimization Pillar](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/)
- [Security Best Practices](https://aws.amazon.com/security/security-resources/)

By Chella S
---

**Project**: PGP Cloud Computing & AWS Solutions Architect  
**Author**: Chellapandi Soundarapandian  
**Repository**: [cf-simplylearn](https://github.com/AmChella/cf-simplylearn)  
**Last Updated**: August 2024
