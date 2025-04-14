
# Deploying VPC, EC2, AND RDS using Terraform

As a DevOps and Cloud Engineer, one of the key responsibilities is to reduce manual effort and ensure infrastructure is automated, consistent, and scalable. Instead of writing repetitive Terraform code for every project, I wanted to build simple but powerful Terraform modules that could be reused across real-world cloud environments.

In this project, I used Terraform modules to automate the provisioning of core AWS services like VPC, EC2, and RDS. These modules are designed to be modular, configurable, and production-friendly, making them easy to plug into any infrastructure setup with just a few lines of code.

Let’s dive into how I structured these modules and used them to build a clean, automated AWS environment.



 Prerequisites: 
● AWS Account: Ensure you have an active AWS account. 
● AWS CLI: Install and configure the AWS Command Line Interface (CLI) 
with appropriate credentials. 
● Terraform: Install Terraform on your local machine.


step-1 : create an EC2 instance and connect through CLI 
step-2 : create new directory for Terraform
step-3 : Inside the directory terraform create .tf file for modules
        mkdir Terraform
        vim VPC.tf
step-4 write this in the module 
## important to specify the region where you are going to create resources
    provider "aws" { 
        region = "us-west-2"  # Specify your desired AWS region 
    } 
    resource "aws_vpc" "main_vpc" { 
        cidr_block = "10.0.0.0/16" 
         tags = { 
            Name = "main_vpc" 
        } 
    }

## after creating vpc we have to create subnets
    resource "aws_subnet" "public_subnet" { 
        vpc_id = aws_vpc.main_vpc.id 
        cidr_block = "10.0.1.0/24" 
        availability_zone = "us-west-2a" 
        tags = { 
             Name = "public_subnet" 
                } 
    } 
    resource "aws_subnet" "private_subnet" { 
        vpc_id = aws_vpc.main_vpc.id          
        cidr_block = "10.0.2.0/24" 
        availability_zone = "us-west-2a" 
        tags = { 
            Name = "private_subnet" 
        } 
    } 

##done with subnets now let connect to internet but how? the answer is through Internet gateway 

    resource "aws_internet_gateway" "igw" { 
        vpc_id = aws_vpc.main_vpc.id 
        tags = { 
            Name = "main_igw" 
        } 
    } 
## after creating Internet gateway we have to do subnet assciation in the route route_table
    resource "aws_route_table" "public_route_table" { 
        vpc_id = aws_vpc.main_vpc.id 
        route { 
            cidr_block = "0.0.0.0/0" 
            gateway_id = aws_internet_gateway.igw.id 
        } 
        tags = { 
        Name = "public_route_table" 
                } 
    } 
    resource "aws_route_table_association" "public_subnet_association" { 
        subnet_id = aws_subnet.public_subnet.id 
        route_table_id = aws_route_table.public_route_table.id 
    }