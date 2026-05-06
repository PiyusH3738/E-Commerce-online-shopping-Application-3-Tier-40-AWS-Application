# 🏗️ AWS 3-Tier VPC Architecture — Terraform

> A production-ready, fully modular Infrastructure-as-Code project that provisions a scalable, secure 3-tier architecture on AWS — built entirely with Terraform.

---

## 📌 Project Description

This project demonstrates end-to-end cloud infrastructure provisioning using Terraform, following AWS best practices for a **3-tier architecture**: a public-facing load balancer tier, a private compute tier with auto-scaling, and an isolated database tier — all spanning multiple availability zones for high availability.

Designed to mirror real-world production environments, this project showcases skills in:
- **Infrastructure as Code (IaC)** with modular Terraform design
- **AWS networking** — VPCs, subnets, route tables, NAT/Internet gateways
- **Security** — least-privilege IAM, security group layering, private subnet isolation
- **High availability** — multi-AZ deployment across us-west-2a, 2b, and 2c
- **Auto Scaling** — dynamic EC2 fleet management with ALB health checks

---

## 🏛️ Architecture Overview

```
                        Internet
                           │
                    ┌──────▼──────┐
                    │   ALB (80)  │   ← Public Subnets (10.0.101–103.0/24)
                    └──────┬──────┘
                           │
             ┌─────────────▼─────────────┐
             │   Auto Scaling Group       │   ← Private Subnets (10.0.1–3.0/24)
             │   EC2 t2.micro (1–3)       │
             │   Port 8080                │
             └─────────────┬─────────────┘
                           │
             ┌─────────────▼─────────────┐
             │   RDS MySQL 8.0            │   ← DB Subnets (10.0.21–23.0/24)
             │   db.t2.micro · 10 GB      │
             │   Not publicly accessible  │
             └────────────────────────────┘

           VPC: 10.0.0.0/16 │ Region: us-west-2 │ AZs: a, b, c
```

---

## 📁 Module Structure

```
3-tier-architecture/
├── main.tf                  # Root module — wires networking, autoscaling, database
├── variables.tf
├── outputs.tf
└── modules/
    ├── networking/          # VPC, subnets, IGW, NAT GW, route tables, security groups
    ├── autoscaling/         # ALB, target group, launch template, ASG, IAM instance profile
    └── database/            # RDS instance, DB subnet group, random password generation
```

---

## ⚙️ Resources Provisioned (40 total)

| Category | Resources |
|---|---|
| **Networking** | VPC, 9 subnets (3 public / 3 private / 3 DB), IGW, NAT GW, 2 route tables, 9 route table associations, Elastic IP |
| **Security** | 3 security groups (LB, web server, database) |
| **Compute** | Launch template, Auto Scaling Group, IAM role + policy + instance profile |
| **Load Balancing** | Application Load Balancer, target group (port 8080), HTTP listener (port 80) |
| **Database** | RDS MySQL 8.0 instance, DB subnet group, auto-generated password |

---

## 🔒 Security Design

| Layer | Rule |
|---|---|
| **ALB SG** | Accepts inbound HTTP (port 80) from `0.0.0.0/0` |
| **Web Server SG** | Accepts port 8080 from ALB SG only; SSH from VPC CIDR (`10.0.0.0/16`) only |
| **DB SG** | Accepts port 3306 from Web Server SG only; no public access |
| **IAM Policy** | EC2 instances granted `rds:*` and `logs:*` — nothing else |
| **RDS** | `publicly_accessible = false`; lives entirely in isolated DB subnets |

---

## 🚀 Getting Started

### Prerequisites
- [Terraform](https://developer.hashicorp.com/terraform/downloads) >= 1.0
- AWS CLI configured with appropriate credentials
- AWS account with permissions to provision VPC, EC2, RDS, and IAM resources

### Deploy

```bash
# Clone the repo
git clone https://github.com/<your-username>/3-tier-vpc.git
cd 3-tier-vpc

# Initialize providers and modules
terraform init

# Preview the execution plan
terraform plan

# Deploy (approx. 8–10 minutes)
terraform apply
```

### Access the Application

After a successful apply, Terraform outputs:

```
lb_dns_name = "my-3-tier-architecture-<id>.us-west-2.elb.amazonaws.com"
db_password = <sensitive>
```

Hit the `lb_dns_name` in your browser to reach the web tier.

To retrieve the database password:
```bash
terraform output db_password
```

### Destroy

```bash
terraform destroy
```

---

## 📤 Outputs

| Output | Description |
|---|---|
| `lb_dns_name` | Public DNS of the Application Load Balancer |
| `db_password` | Auto-generated 16-character RDS admin password *(sensitive)* |

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| **Terraform** | Infrastructure provisioning and state management |
| **AWS VPC** | Network isolation and routing |
| **EC2 + Auto Scaling** | Elastic compute with health-check-driven scaling |
| **Application Load Balancer** | Traffic distribution across EC2 instances |
| **RDS MySQL 8.0** | Managed relational database |
| **IAM** | Least-privilege access control for EC2 |
| **cloud-init** | EC2 bootstrap configuration via user data |

---

## 📚 Concepts Demonstrated

- **Modular Terraform** — reusable, single-responsibility modules
- **Remote module sourcing** — uses `terraform-aws-modules/vpc`, `terraform-aws-modules/alb`, and `terraform-in-action` registry modules
- **Dependency management** — implicit resource dependencies via reference chaining
- **Sensitive output handling** — DB password marked sensitive, never exposed in plan output
- **Multi-AZ high availability** — all tiers replicated across 3 AZs
- **Infrastructure lifecycle** — plan → apply → destroy with zero manual steps

---

## 📄 License

MIT — free to use, fork, and adapt.
