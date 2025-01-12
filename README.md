# AWS E-commerce Workshop
The "AWS E-commerce Workshop" is a hands-on lab focused on building a scalable cloud infrastructure for an e-commerce application using AWS. The primary objective is to learn how to create and manage key components such as a Virtual Private Cloud (VPC), subnets, Internet Gateways, EC2 instances, security groups, and S3 storage configuration. This project enables participants to gain essential skills for designing secure, efficient, and optimized cloud architectures for web applications, providing a solid foundation for implementing real-world e-commerce solutions.


![Piano_a_mente (1)](https://github.com/user-attachments/assets/f0ff11ea-f143-4bba-a4c5-5e7547ba98ef)

> 🟣 **NETWORK FOUNDATION**
> 
> ## Step 1: Network Foundation
> 
> ### Overview
> In this lab, we'll create the network infrastructure for your e-commerce application, including a VPC, subnet, and Internet Gateway in the Frankfurt region.
> 
> ### Step-by-Step Instructions
> 
> 1. **Create VPC**
>    - Navigate to VPC Dashboard in AWS Console
>    - Click "Create VPC"
>    - Enter CIDR block 10.0.0.0/26
>    - Enable DNS hostnames
>    - Add tag: Name=ecommerce-vpc
> 
> ```bash
> # Create VPC
> aws ec2 create-vpc \
>     --cidr-block 10.0.0.0/16 \
>     --tag-specifications 'ResourceType=vpc,Tags=[{Key=Name,Value=ecommerce-vpc}]' \
>     --instance-tenancy default \
>     --enable-dns-hostnames
> ```
> 
> ## Step 2: Security Configuration
> // ... contenuto della sezione Security Configuration ...

> 🟧 **EC2 SETUP**
> 
> ## Step 3: EC2 Instance Setup
> // ... tutto il contenuto della sezione EC2 Instance Setup ...

> 🟩 **STORAGE CONFIGURATION**
> 
> ## Step 4: S3 Storage Configuration
> // ... tutto il contenuto della sezione S3 Storage Configuration ...

## Step 5: Website Deployment
// ... resto del file invariato ...
