# AWS E-commerce Workshop

> ⚠️ **DISCLAIMER: Only for demo in sandbox environment**
> This workshop is designed for learning purposes and should only be executed in a sandbox/demo AWS environment. Do not run in production environment BECAUSE IT WILL COST YOU MONEY.

The "AWS E-commerce Workshop" is a hands-on lab focused on building a scalable cloud infrastructure for an e-commerce application using AWS. The primary objective is to learn how to create and manage key components such as a Virtual Private Cloud (VPC), subnets, Internet Gateways, EC2 instances, security groups, and S3 storage configuration. This project enables participants to gain essential skills for designing secure, efficient, and optimized cloud architectures for web applications, providing a solid foundation for implementing real-world e-commerce solutions.

![Group_78](https://github.com/user-attachments/assets/4a54d053-2232-4a22-9f0e-5e118a3ea2d0)



<div style="background-color: #6B46C1; color: white; padding: 20px; border-radius: 8px; margin: 20px 0;">
<div style="background-color: rgba(0, 0, 0, 0.3); padding: 15px; border-radius: 6px; margin: 10px 0;">

## Step 1: Network Foundation

### Overview
In this lab, we'll create the network infrastructure for your e-commerce application, including a VPC, subnet, and Internet Gateway in the Ohio region.

### Step-by-Step Instructions

1. **Create VPC** 
   - Navigate to VPC Dashboard in AWS Console
   - Click "Create VPC"
   - Enter CIDR block 10.0.0.0/26
   - Enable DNS hostnames
   - Add tag: Name=ecommerce-vpc

```bash
# Create VPC
aws ec2 create-vpc \
    --cidr-block 10.0.0.0/16 \
    --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=ecommerce-vpc}]' \
    --instance-tenancy default \
    --enable-dns-hostnames

# Store VPC ID
VPC_ID=$(aws ec2 describe-vpcs \
    --filters Name=tag:Name,Values=ecommerce-vpc \
    --query 'Vpcs[0].VpcId' \
    --output text)
```

**Validation:**
- Check VPC creation in console
- Verify CIDR block is 10.0.0.0/16
- Confirm DNS hostnames enabled
```bash
aws ec2 describe-vpcs --vpc-ids $VPC_ID
```

2. **Create Private Subnet**
   - Within your VPC, create new subnet
   - Use CIDR block 10.0.0.0/26
   - Select Frankfurt (eu-central-1a)
   - Tag: Name=ecommerce-private-subnet

```bash
# Create subnet
aws ec2 create-subnet \
    --vpc-id $VPC_ID \
    --cidr-block 10.0.0.0/26 \
    --availability-zone eu-central-1a \
    --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=ecommerce-private-subnet}]'

# Store subnet ID
SUBNET_ID=$(aws ec2 describe-subnets \
    --filters Name=tag:Name,Values=ecommerce-private-subnet \
    --query 'Subnets[0].SubnetId' \
    --output text)
```

**Validation:**
- Verify subnet creation
- Check CIDR range
- Confirm AZ assignment
```bash
aws ec2 describe-subnets --subnet-ids $SUBNET_ID
```

3. **Create and Attach Internet Gateway**
   - Create new Internet Gateway
   - Attach to your VPC
   - Tag: Name=ecommerce-igw

```bash
# Create Internet Gateway
aws ec2 create-internet-gateway \
    --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=ecommerce-igw}]'

# Store IGW ID
IGW_ID=$(aws ec2 describe-internet-gateways \
    --filters Name=tag:Name,Values=ecommerce-igw \
    --query 'InternetGateways[0].InternetGatewayId' \
    --output text)

# Attach to VPC
aws ec2 attach-internet-gateway \
    --vpc-id $VPC_ID \
    --internet-gateway-id $IGW_ID
```

**Validation:**
- Check IGW creation
- Verify VPC attachment
```bash
aws ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID
```


## Step 2: Security Configuration

### Overview
Configure security groups and network access controls for your e-commerce infrastructure.

### Step-by-Step Instructions

1. **Create Security Group**
   - Navigate to EC2 Dashboard > Security Groups
   - Create in your VPC
   - Name: ecommerce-sg
   - Description: E-commerce security rules

```bash
# Create security group
aws ec2 create-security-group \
    --group-name ecommerce-sg \
    --description "E-commerce security group" \
    --vpc-id $VPC_ID

# Store security group ID
SG_ID=$(aws ec2 describe-security-groups \
    --filters Name=group-name,Values=ecommerce-sg \
    --query 'SecurityGroups[0].GroupId' \
    --output text)
```

**Validation:**
- Verify security group creation
- Check VPC association
```bash
aws ec2 describe-security-groups --group-ids $SG_ID
```

2. **Configure Security Rules**
   - Add inbound rules for HTTP and SSH
   - Configure outbound rules

```bash
# Add inbound rules
aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 80 \
    --cidr 0.0.0.0/0

aws ec2 authorize-security-group-ingress \
    --group-id $SG_ID \
    --protocol tcp \
    --port 22 \
    --cidr 0.0.0.0/0
```

**Validation:**
- Verify inbound rules
- Check port configurations
```bash
aws ec2 describe-security-group-rules \
    --filters Name=group-id,Values=$SG_ID
```

</div>
</div>

<div style="background-color: #ED8936; color: white; padding: 20px; border-radius: 8px; margin: 20px 0;">
<div style="background-color: rgba(0, 0, 0, 0.3); padding: 15px; border-radius: 6px; margin: 10px 0;">

## Step 3: EC2 Instance Setup

### Overview
Deploy and configure the EC2 instance that will host your e-commerce application.

### Step-by-Step Instructions

1. **Launch EC2 Instance**
   - Choose Amazon Linux 2 AMI
   - Select t2.micro instance type
   - Place in private subnet
   - Attach security group

```bash
# Create key pair for SSH access
aws ec2 create-key-pair \
    --key-name ecommerce-key \
    --query 'KeyMaterial' \
    --output text > ecommerce-key.pem

chmod 400 ecommerce-key.pem

# Launch EC2 instance
aws ec2 run-instances \
    --image-id ami-0a261c0e5f51090b1 \
    --instance-type t2.micro \
    --subnet-id $SUBNET_ID \
    --security-group-ids $SG_ID \
    --key-name ecommerce-key \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=ecommerce-server}]'

# Store instance ID
INSTANCE_ID=$(aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=ecommerce-server" \
    --query 'Reservations[0].Instances[0].InstanceId' \
    --output text)
```

**Validation:**
- Verify instance running status
- Check instance tags
```bash
aws ec2 describe-instances --instance-ids $INSTANCE_ID
```

2. **Configure Elastic IP**
   - Allocate new Elastic IP
   - Associate with EC2 instance

```bash
# Allocate Elastic IP
aws ec2 allocate-address \
    --domain vpc

# Store Elastic IP allocation ID
EIP_ID=$(aws ec2 describe-addresses \
    --query 'Addresses[-1].AllocationId' \
    --output text)

# Associate with instance
aws ec2 associate-address \
    --instance-id $INSTANCE_ID \
    --allocation-id $EIP_ID
```

**Validation:**
- Check Elastic IP association
- Test SSH connectivity
```bash
aws ec2 describe-addresses --allocation-ids $EIP_ID

# Get public IP
PUBLIC_IP=$(aws ec2 describe-addresses \
    --allocation-ids $EIP_ID \
    --query 'Addresses[0].PublicIp' \
    --output text)

# Test SSH connection
ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP
```

</div>
</div>

<div style="background-color: #438a26; color: white; padding: 20px; border-radius: 8px; margin: 20px 0;">
<div style="background-color: rgba(0, 0, 0, 0.3); padding: 15px; border-radius: 6px; margin: 10px 0;">

## Step 4: S3 Storage Configuration

### Overview
Set up S3 storage for your website files and configure necessary permissions.

### Step-by-Step Instructions

1. **Create S3 Bucket**
   - Choose unique bucket name
   - Select Frankfurt region
   - Configure basic settings

```bash
# Create S3 bucket
aws s3api create-bucket \
    --bucket your-ecommerce-bucket-name \
    --region eu-central-1 \
    --create-bucket-configuration LocationConstraint=eu-central-1

# Enable versioning
aws s3api put-bucket-versioning \
    --bucket your-ecommerce-bucket-name \
    --versioning-configuration Status=Enabled
```

**Validation:**
- Verify bucket creation
- Check region setting
```bash
aws s3api get-bucket-location --bucket your-ecommerce-bucket-name
aws s3api get-bucket-versioning --bucket your-ecommerce-bucket-name
```

2. **Configure IAM Role**
   - Create role for EC2
   - Attach S3 access policy

```bash
# Create IAM role
cat << EOF > trust-policy.json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
EOF

aws iam create-role \
    --role-name ecommerce-ec2-role \
    --assume-role-policy-document file://trust-policy.json

# Create S3 access policy
cat << EOF > s3-policy.json
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
                "arn:aws:s3:::your-ecommerce-bucket-name",
                "arn:aws:s3:::your-ecommerce-bucket-name/*"
            ]
        }
    ]
}
EOF

aws iam put-role-policy \
    --role-name ecommerce-ec2-role \
    --policy-name S3Access \
    --policy-document file://s3-policy.json
```

**Validation:**
- Check role creation
- Verify policy attachment
```bash
aws iam get-role --role-name ecommerce-ec2-role
aws iam get-role-policy --role-name ecommerce-ec2-role --policy-name S3Access
```

## Step 5: Website Deployment
### Overview
Deploy the e-commerce website to your EC2 instance and configure the web server.

### Step-by-Step Instructions

1. **Configure Web Server**
   - Install Apache and PHP
   - Set up virtual host
   - Configure permissions

```bash
# Connect to instance and run setup
### User
# Execute <Website> with this script
# Edit before Running >>> <Website>
# use 2> log.txt for troubleshoot

# Sys Update e Cleanse
sleep 30;
sudo yum remove -y awscli;
sleep 2;
sudo yum update;
sleep 5;

### AWSCLIV2 Install

sudo curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip";
sleep 2;
sudo unzip awscliv2.zip;
sleep 2;
sudo ./aws/install;
sleep 2;

### Install and Run WebServer

sudo yum install httpd;
sleep 2;
sudo yum install polkit;
sleep 2;
sudo systemctl enable httpd;
sleep 2;
sudo systemctl start httpd;
sleep 2;
sudo chown ec2-user:ec2-user /var/www/html/;
cd /var/www/html/;
sleep 5;

### Download WebSite's Files from S3

/usr/local/bin/aws s3 sync s3://<bucketname>/ /var/www/html/;
sleep 5;
sudo unzip <Website.zip>;
sleep 2;
sudo mv <Website>/* .;
sleep 2;

# Cleanse var/www/html/

sudo rm -rf <Website.zip>;
sudo rmdir <Website>;

echo 'Deploy successfull'
```

**Validation:**
- Check Apache status
- Verify PHP installation
```bash
ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP 'sudo systemctl status httpd'
```

2. **Deploy Website Files**
   - Upload files to S3
   - Download to EC2
   - Configure website

```bash
# Upload website files to S3
aws s3 cp ./website-files s3://your-ecommerce-bucket-name/ --recursive

# Download and deploy on EC2
ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP << 'EOF'
aws s3 cp s3://your-ecommerce-bucket-name/ /var/www/html/ --recursive
sudo systemctl restart httpd
EOF
```

**Validation:**
- Test website access
- Check error logs
```bash
# Check website accessibility
curl -I http://$PUBLIC_IP

# Check error logs
ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP 'sudo tail /var/log/httpd/error_log'
```

</div>
</div>

## Final Validation
### Overview
Perform comprehensive testing of the entire setup.

### Validation Steps

1. **Network Connectivity**
```bash
# Test VPC connectivity
aws ec2 describe-vpc-attribute \
    --vpc-id $VPC_ID \
    --attribute enableDnsHostnames

# Test subnet routing
aws ec2 describe-route-tables \
    --filters "Name=vpc-id,Values=$VPC_ID"
```

2. **Security Configuration**
```bash
# Verify security group rules
aws ec2 describe-security-group-rules \
    --filters Name=group-id,Values=$SG_ID

# Check instance security groups
aws ec2 describe-instance-attribute \
    --instance-id $INSTANCE_ID \
    --attribute groupSet
```

3. **Website Functionality**
```bash
# Check HTTP response
curl -I http://$PUBLIC_IP

# Test PHP functionality
echo "<?php phpinfo(); ?>" | ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP \
    'cat > /var/www/html/info.php'
curl http://$PUBLIC_IP/info.php
```

## Troubleshooting Guide
### Common Issues and Solutions

1. **Cannot Connect to EC2**
- Check security group rules
- Verify Elastic IP association
- Confirm key pair permissions
```bash
aws ec2 describe-instance-attribute \
    --instance-id $INSTANCE_ID \
    --attribute groupSet
```

2. **Website Not Accessible**
- Check Apache status
- Verify file permissions
- Review error logs
```bash
ssh -i "ecommerce-key.pem" ec2-user@$PUBLIC_IP << 'EOF'
sudo systemctl status httpd
ls -la /var/www/html
sudo tail /var/log/httpd/error_log
EOF
```

3. **S3 Access Issues**
- Check IAM role
- Verify bucket policy
- Test S3 connectivity
```bash
aws iam simulate-principal-policy \
    --policy-source-arn arn:aws:iam::ACCOUNT_ID:role/ecommerce-ec2-role \
    --action-names s3:GetObject \
    --resource-arns arn:aws:s3:::your-ecommerce-bucket-name/*
```

## Clean Up Instructions
### Overview
Remove all created resources to avoid unwanted charges.

```bash
# Delete EC2 instance
aws ec2 terminate-instances --instance-ids $INSTANCE_ID

# Release Elastic IP
aws ec2 release-address --allocation-id $EIP_ID

# Delete security group
aws ec2 delete-security-group --group-id $SG_ID

# Detach and delete Internet Gateway
aws ec2 detach-internet-gateway \
    --internet-gateway-id $IGW_ID \
    --vpc-id $VPC_ID
aws ec2 delete-internet-gateway --internet-gateway-id $IGW_ID

# Delete subnet and VPC
aws ec2 delete-subnet --subnet-id $SUBNET_ID
aws ec2 delete-vpc --vpc-id $VPC_ID
```

**Validation:**
- Verify all resources deleted
- Check AWS Console for remaining resources
```bash
aws ec2 describe-vpcs --vpc-ids $VPC_ID
aws ec2 describe-subnets --subnet-ids $SUBNET_ID
aws ec2 describe-internet-gateways --internet-gateway-ids $IGW_ID
aws ec2 describe-security-groups --group-ids $SG_ID
```
