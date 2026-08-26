# NGINX Load Balancer on AWS EC2 with Terraform

Deploy a fully functional NGINX load balancer across real AWS EC2 instances using Terraform. 

## Overview

This project demonstrates a production-like load balancing setup on AWS:

- 2 Backend Servers running NGINX, each serving unique content
- 1 Load Balancer distributing traffic between backends using round-robin
- All provisioned via Terraform in your real AWS account
- Free-tier eligible when using t2.micro instances

### Key Features

- Dynamic AMI lookup (always uses latest Ubuntu 24.04)
- Auto-generates SSH key pair
- User-data scripts for automatic NGINX installation
- Default VPC setup (no networking complexity)
- Public IPs for easy testing
- Clean terraform destroy for complete removal

## Architecture
 [Filename](diagram/architecture-diagram.png)

## Prerequisites

- Terraform (v1.0+)
- AWS CLI configured with credentials
- AWS account (free-tier eligible recommended)
- Basic understanding of EC2, security groups, and NGINX

### AWS Setup

```
# Configure AWS CLI with your credentials
aws configure
# Enter: Access Key ID, Secret Key, region (e.g., us-east-1)
```

Note: Create an IAM user with programmatic access in AWS Console -> IAM -> Users -> Create user -> Attach AdministratorAccess (for learning purposes) or scope permissions down later.

## Quick Start

```
# Clone the repository
git clone https://github.com/zaahrraa/nginx-lb-terraform.git
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
+-- main.tf                          # Main Terraform configuration
+-- outputs.tf                       # Output variables
+-- templates/
|   +-- backend-userdata.sh.tpl      # Backend server setup script
|   +-- lb-userdata.sh.tpl           # Load balancer setup script
+-- README.md                        # This file
```

## Configuration

### Default Settings

| Parameter | Value | Description |
|-----------|-------|-------------|
| Region | us-east-1 | AWS region for deployment |
| Instance Type | t2.micro | Free-tier eligible EC2 size |
| AMI | Latest Ubuntu 24.04 | Automatically fetched |
| Backend Count | 2 | Number of backend servers |
| SSH Key | Auto-generated | Saved as nginx-lb-key.pem |
| Security Groups | Open HTTP/SSH | Customize for production |

### Customization

To modify the configuration, edit the variables directly in main.tf:

```
# Change region
provider "aws" {
  region = "us-west-2"  # Change to your preferred region
}

# Change instance type
instance_type = "t3.micro"  # Or any other instance type

# Restrict SSH access (recommended for production)
cidr_blocks = ["YOUR_IP_ADDRESS/32"]  # Replace with your actual IP
```

## Testing the Load Balancer

### Command Line Test for Linux / macOS (Bash)

```
$LB_IP = (terraform output -raw lb_public_ip)
1..6 | ForEach-Object { 
    $response = Invoke-WebRequest -Uri "http://$LB_IP" -UseBasicParsing
    Write-Host "Request $_ : $($response.Content)" 
}
```

Expected Output:
```
Request 1: Hello from Backend Server A
Request 2: Hello from Backend Server B
Request 3: Hello from Backend Server A
Request 4: Hello from Backend Server B
Request 5: Hello from Backend Server A
Request 6: Hello from Backend Server B
Request 7: Hello from Backend Server A
Request 8: Hello from Backend Server B
Request 9: Hello from Backend Server A
Request 10: Hello from Backend Server B
```

### Command Line Test for Windows (PowerShell)

```
# Get the load balancer IP
$LB_IP = terraform output -raw lb_public_ip

# Test with Invoke-WebRequest (Recommended)
1..6 | ForEach-Object {
    $response = Invoke-WebRequest -Uri "http://$LB_IP" -UseBasicParsing
    Write-Host "Request $_ : $($response.Content.Trim())"
}
```

Expected PowerShell Output:
```
Request 1 : Hello from Backend Server A
Request 2 : Hello from Backend Server B
Request 3 : Hello from Backend Server A
Request 4 : Hello from Backend Server B
Request 5 : Hello from Backend Server A
Request 6 : Hello from Backend Server B
```

Alternative PowerShell Method:
```
# Using WebClient for simpler output
$LB_IP = terraform output -raw lb_public_ip
$webClient = New-Object System.Net.WebClient

1..6 | ForEach-Object {
    $response = $webClient.DownloadString("http://$LB_IP").Trim()
    Write-Host "Request $_ : $response"
}
```

### Command Line Test for Windows (Git Bash / WSL)

If you have Git Bash or WSL installed, you can use the bash commands:

```
# In Git Bash or WSL terminal
LB_IP=$(terraform output -raw lb_public_ip)
for i in {1..6}; do
    echo "Request $i: $(curl -s http://$LB_IP)"
done
```

### Browser Test

Open http://LB_PUBLIC_IP in your browser and refresh multiple times to see the alternating responses.

### Quick One-Liner Tests

Bash (Linux/macOS/Git Bash):
```
# One-liner to test 6 requests
for i in {1..6}; do curl -s http://$(terraform output -raw lb_public_ip); echo " (Request $i)"; done
```

PowerShell (Windows):
```
# One-liner for quick testing
$ip=terraform output -raw lb_public_ip; 1..6 | %{ Write-Host "Request $_ : $(Invoke-WebRequest -Uri http://$ip -UseBasicParsing).Content.Trim()" }
```

### Troubleshooting PowerShell Commands

If you get System.Xml.XmlDocument instead of the actual content:

Fix: Use Invoke-WebRequest instead of Invoke-RestMethod:

```
# This returns XML objects (incorrect)
Invoke-RestMethod -Uri "http://$LB_IP"

# This returns plain text content (correct)
Invoke-WebRequest -Uri "http://$LB_IP" -UseBasicParsing
```

## Cleanup

```
# Destroy all resources
terraform destroy
# Type 'yes' when prompted

# Verify in AWS Console
# EC2 -> Instances -> Check no instances running

# Remove local files (optional)
rm nginx-lb-key.pem
rm nginx-lb-key.pem.pub
rm -rf .terraform/
rm terraform.tfstate*
```


