Here's a complete README.md for your NGINX Load Balancer project:

---

# NGINX Load Balancer on AWS EC2 with Terraform

Deploy a fully functional NGINX load balancer across real AWS EC2 instances using Terraform. 

## Overview

This project demonstrates a production-like load balancing setup on AWS:

- **2 Backend Servers** running NGINX, each serving unique content
- **1 Load Balancer** distributing traffic between backends using round-robin
- **All provisioned via Terraform** in your real AWS account
- **Free-tier eligible** when using `t2.micro` instances

### Key Features

- ✅ Dynamic AMI lookup (always uses latest Ubuntu 24.04)
- ✅ Auto-generates SSH key pair
- ✅ User-data scripts for automatic NGINX installation
- ✅ Default VPC setup (no networking complexity)
- ✅ Public IPs for easy testing
- ✅ Clean `terraform destroy` for complete removal

##  Architecture

```
Internet
    │
    ▼
┌─────────────────────────┐
│  NGINX Load Balancer    │  (t2.micro, public IP)
│  - Round-robin routing  │
└─────────┬───────────────┘
          │
    ┌─────┴─────┐
    ▼           ▼
┌─────────┐ ┌─────────┐
│ Backend │ │ Backend │  (t2.micro, private IPs)
│ Server A│ │ Server B│
│ "Hello  │ │ "Hello  │
│ from A" │ │ from B" │
└─────────┘ └─────────┘
```

##  Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) (v1.0+)
- [AWS CLI](https://aws.amazon.com/cli/) configured with credentials
- AWS account (free-tier eligible recommended)
- Basic understanding of EC2, security groups, and NGINX

### AWS Setup

```bash
# Configure AWS CLI with your credentials
aws configure
# Enter: Access Key ID, Secret Key, region (e.g., us-east-1)
```

> **Note:** Create an IAM user with programmatic access in AWS Console → IAM → Users → Create user → Attach `AdministratorAccess` (for learning purposes) or scope permissions down later.

## Quick Start

```bash
# Clone the repository
git clone https://github.com/yourusername/nginx-lb-terraform.git
cd nginx-lb-terraform/terraform

# Initialize Terraform
terraform init

# Review what will be created
terraform plan

# Deploy the infrastructure
terraform apply
# Type 'yes' when prompted

# Wait 30-60 seconds for instances to initialize

# Test the load balancer
curl http://$(terraform output -raw lb_public_ip)
```

## Project Structure

```
terraform/
├── main.tf                          # Main Terraform configuration
├── outputs.tf                       # Output variables
├── templates/
│   ├── backend-userdata.sh.tpl      # Backend server setup script
│   └── lb-userdata.sh.tpl           # Load balancer setup script
└── README.md                        # This file
```

##  Configuration

### Default Settings

| Parameter | Value | Description |
|-----------|-------|-------------|
| Region | `us-east-1` | AWS region for deployment |
| Instance Type | `t2.micro` | Free-tier eligible EC2 size |
| AMI | Latest Ubuntu 24.04 | Automatically fetched |
| Backend Count | 2 | Number of backend servers |
| SSH Key | Auto-generated | Saved as `nginx-lb-key.pem` |
| Security Groups | Open HTTP/SSH | Customize for production |

### Customization

To modify the configuration, edit the variables directly in `main.tf`:

```hcl
# Change region
provider "aws" {
  region = "us-west-2"  # Change to your preferred region
}

# Change instance type
instance_type = "t3.micro"  # Or any other instance type

# Restrict SSH access (recommended for production)
cidr_blocks = ["YOUR_IP_ADDRESS/32"]  # Replace with your actual IP
```

##  Testing the Load Balancer

### Command Line Test

```bash
# Get the load balancer IP
LB_IP=$(terraform output -raw lb_public_ip)

# Send 10 requests to see round-robin distribution
for i in {1..10}; do 
    echo "Request $i: $(curl -s http://$LB_IP)"
done
```

**Expected Output:**
```
Request 1: Hello from Backend Server A
Request 2: Hello from Backend Server B
Request 3: Hello from Backend Server A
Request 4: Hello from Backend Server B
...and so on
```

### Browser Test

Open `http://<LB_PUBLIC_IP>` in your browser and refresh multiple times to see the alternating responses.

## Cleanup

```bash
# Destroy all resources
terraform destroy
# Type 'yes' when prompted

# Verify in AWS Console
# EC2 → Instances → Check no instances running

# Remove local files (optional)
rm nginx-lb-key.pem
rm nginx-lb-key.pem.pub
rm -rf .terraform/
rm terraform.tfstate*
```






