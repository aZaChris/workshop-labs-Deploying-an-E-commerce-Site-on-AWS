# AWS E-commerce Workshop

## Lab 1: Network Foundation

### Overview
In this lab, we'll create the network infrastructure for your e-commerce application, including a VPC, subnet, and Internet Gateway in the Frankfurt region.

![Piano_a_mente (1)](https://github.com/user-attachments/assets/f0ff11ea-f143-4bba-a4c5-5e7547ba98ef)

### Step-by-Step Instructions

1. **Create VPC**
   - Navigate to VPC Dashboard in AWS Console
   - Click "Create VPC"
   - Enter CIDR block 10.0.0.0/16
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

## Lab 2: Security Configuration

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

[Continue with Labs 3-5 in same format...]

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
