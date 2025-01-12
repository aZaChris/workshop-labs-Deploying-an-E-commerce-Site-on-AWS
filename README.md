# Workshop: Deploying an E-commerce Site on AWS

This guide walks through deploying a scalable e-commerce website on AWS using various services including VPC, EC2, S3, and IAM.

## Architecture Overview

![Piano_a_mente (1)](https://github.com/user-attachments/assets/e0c5e99b-be04-48ef-bf09-15befa64633f)



## Prerequisites

- AWS Account
- Basic knowledge of AWS services
- AWS CLI installed and configured
- Access to AWS Management Console

## Detailed Setup Steps

### 1. VPC Setup

```bash
# Create VPC
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=ecommerce-vpc}]'

# Create Private Subnet
aws ec2 create-subnet \
    --vpc-id <vpc-id> \
    --cidr-block 10.0.0.0/26 \
    --availability-zone eu-central-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=ecommerce-private-subnet}]'

# Create Internet Gateway
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=ecommerce-igw}]'

# Attach Internet Gateway to VPC
aws ec2 attach-internet-gateway \
    --internet-gateway-id <igw-id> \
    --vpc-id <vpc-id>
```

### 2. Security Group Configuration

```bash
# Create Security Group
aws ec2 create-security-group \
    --group-name ecommerce-sg \
    --description "Security group for e-commerce site" \
    --vpc-id <vpc-id>

# Add inbound rules
aws ec2 authorize-security-group-ingress \
    --group-id <security-group-id> \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id <security-group-id> \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0
```

### 3. EC2 Instance Deployment

```bash
# Launch EC2 instance
aws ec2 run-instances \
    --image-id ami-0a261c0e5f51090b1 \
    --instance-type t2.micro \
    --subnet-id <subnet-id> \
    --security-group-ids <security-group-id> \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ecommerce-server}]'

# Allocate Elastic IP
aws ec2 allocate-address

# Associate Elastic IP
aws ec2 associate-address \
    --instance-id <instance-id> \
    --allocation-id <eip-allocation-id>
```

### 4. S3 Bucket Setup

```bash
# Create S3 bucket
aws s3api create-bucket \
    --bucket your-ecommerce-bucket \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1

# Upload website files
aws s3 cp website.zip s3://your-ecommerce-bucket/
```

### 5. IAM Role Configuration

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::your-ecommerce-bucket",
                "arn:aws:s3:::your-ecommerce-bucket/*"
            ]
        }
    ]
}
```

## Initial Setup Script

```bash
#!/bin/bash
# EC2 instance setup script

# Update system
sudo yum update -y

# Install required packages
sudo yum install -y httpd wget unzip

# Start and enable Apache
sudo systemctl start httpd
sudo systemctl enable httpd

# Download website from S3
aws s3 cp s3://your-ecommerce-bucket/website.zip /tmp/
unzip /tmp/website.zip -d /var/www/html/

# Set permissions
sudo chown -R apache:apache /var/www/html/
```

## Verification Steps

1. Check VPC Configuration:
   - Verify subnet CIDR block is /26
   - Confirm Internet Gateway is attached
   - Validate route table entries

2. EC2 Instance Check:
   - Verify instance is running
   - Confirm Elastic IP association
   - Test SSH access

3. Website Access:
   - Navigate to Elastic IP address in browser
   - Verify website loads correctly
   - Check S3 access from EC2 instance

## Troubleshooting

Common issues and solutions:

1. Website not accessible:
   - Check security group rules
   - Verify Apache is running
   - Confirm Elastic IP association

2. S3 access issues:
   - Validate IAM role permissions
   - Check S3 bucket policy
   - Verify instance profile attachment

3. Network connectivity:
   - Confirm route table configuration
   - Check Internet Gateway attachment
   - Verify CIDR blocks

## Clean Up

```bash
# Remove resources when no longer needed
aws ec2 release-address --allocation-id <eip-allocation-id>
aws ec2 terminate-instances --instance-ids <instance-id>
aws s3 rb s3://your-ecommerce-bucket --force
aws ec2 delete-security-group --group-id <security-group-id>
aws ec2 detach-internet-gateway --internet-gateway-id <igw-id> --vpc-id <vpc-id>
aws ec2 delete-internet-gateway --internet-gateway-id <igw-id>
aws ec2 delete-subnet --subnet-id <subnet-id>
aws ec2 delete-vpc --vpc-id <vpc-id>
```
