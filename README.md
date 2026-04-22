# AWS VPC Infrastructure with Terraform

![Terraform](https://img.shields.io/badge/Terraform-v1.x-7B42BC?logo=terraform&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-EC2%20%7C%20VPC%20%7C%20ALB-FF9900?logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

A production-style AWS VPC infrastructure provisioned entirely via **Terraform** — zero manual console steps. Includes public/private subnets, NAT gateway, EC2 instances, security groups, and an Application Load Balancer for high availability.

---

## Architecture

```
Internet
    │
    ▼
Internet Gateway
    │
    ▼
┌─────────────────────────────────────┐
│            VPC (10.0.0.0/16)        │
│                                     │
│  ┌──────────────┐  ┌─────────────┐  │
│  │ Public Subnet│  │Public Subnet│  │
│  │  (AZ-1a)     │  │  (AZ-1b)   │  │
│  │   ALB        │  │   ALB      │  │
│  └──────┬───────┘  └──────┬──────┘  │
│         │                 │         │
│         ▼                 ▼         │
│  ┌──────────────┐  ┌─────────────┐  │
│  │Private Subnet│  │Private Subnet│ │
│  │  (AZ-1a)     │  │  (AZ-1b)   │  │
│  │  EC2         │  │  EC2       │  │
│  └──────────────┘  └─────────────┘  │
│                                     │
│         NAT Gateway                 │
└─────────────────────────────────────┘
```

---

## What's Provisioned

| Resource | Details |
|---|---|
| VPC | Custom CIDR, DNS enabled |
| Subnets | 2 public + 2 private across 2 AZs |
| Internet Gateway | Attached to VPC for public traffic |
| NAT Gateway | Allows private instances to reach internet |
| Route Tables | Separate tables for public and private subnets |
| EC2 Instances | Deployed in private subnets |
| Security Groups | Least-privilege inbound/outbound rules |
| NACLs | Subnet-level traffic control |
| ALB | Distributes traffic across EC2 instances |

---

## Project Structure

```
aws-vpc-terraform/
├── main.tf               # Root module — wires everything together
├── variables.tf          # Input variable definitions
├── outputs.tf            # Output values (VPC ID, ALB DNS, etc.)
├── terraform.tfvars      # Variable values (region, CIDR, etc.)
├── modules/
│   ├── vpc/              # VPC, subnets, IGW, NAT, route tables
│   ├── ec2/              # EC2 instances, key pair, user data
│   └── security-groups/  # SG rules for ALB, EC2, NAT
└── README.md
```

---

## Prerequisites

- [Terraform](https://developer.hashicorp.com/terraform/downloads) v1.0+
- AWS CLI configured (`aws configure`)
- IAM user/role with EC2, VPC, ELB permissions

---

## Usage

### 1. Clone the repo

```bash
git clone https://github.com/Ramana175/aws-vpc-terraform.git
cd aws-vpc-terraform
```

### 2. Update variables

Edit `terraform.tfvars` with your values:

```hcl
region         = "ap-south-1"
vpc_cidr       = "10.0.0.0/16"
public_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
private_subnets = ["10.0.3.0/24", "10.0.4.0/24"]
instance_type  = "t2.micro"
```

### 3. Initialise and apply

```bash
terraform init
terraform plan
terraform apply
```

Full infrastructure is up in **under 8 minutes**.

### 4. Destroy when done

```bash
terraform destroy
```

---

## Key Design Decisions

- **Reusable modules** — VPC, EC2, and security group logic are separated into modules so they can be reused across environments (dev/staging/prod)
- **Private subnet for compute** — EC2 instances have no public IPs; all traffic goes through the ALB
- **Least-privilege security groups** — only required ports open; no `0.0.0.0/0` on EC2 directly
- **NAT Gateway** — allows private instances to pull updates without exposing them to the internet

---

## Outputs

After `terraform apply`, you'll see:

```
vpc_id          = "vpc-xxxxxxxxxxxxxxxxx"
alb_dns_name    = "my-alb-123456789.ap-south-1.elb.amazonaws.com"
private_ec2_ips = ["10.0.3.x", "10.0.4.x"]
```

---

## Author

**S. Venkata Ramana**
[LinkedIn](https://linkedin.com/in/venkata-ramana-sanga-176936401) · [GitHub](https://github.com/Ramana175)
