# AWS-RDS-Terraform
Create Postgres RDS using Terraform in AWS

## Folder Structure
In this module, we shall have the following files:

```
module-06-rds-terraform/
├── main.tf
├── variables.tf
├── outputs.tf
```
## Hands On
### Step 1: Create RDS Subfolder
In your terminal, navigate to your root folder and create an RDS Folder _module-06-rds-terraform_.

```
cd aws-data-engineering
mkdir -p module-06-rds-terraform
cd module-06-rds-terraform
```
In it, create the following files:

```
main.tf
variables.tf
outputs.tf
```
### Step 2: Add RDS inputs to variables.tf

```
variable "project_name" {
  description = "Project name for tagging"
  type        = string
}

variable "vpc_id" {
  description = "VPC ID"
  type        = string
}

variable "private_subnet_ids" {
  description = "Private subnet IDs for RDS"
  type        = list(string)
}

variable "db_name" {
  description = "Database name"
  type        = string
  default     = "dataplatform"
}

variable "db_username" {
  description = "Master DB username"
  type        = string
}

variable "db_password" {
  description = "Master DB password"
  type        = string
  sensitive   = true
}

```
### Step 3: Create main.tf
This will contain the PostgreSQL RDS

```
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# RDS SECURITY GROUP
resource "aws_security_group" "rds_sg" {
  name        = "${var.project_name}-rds-sg"
  description = "Allow PostgreSQL access from EC2"
  vpc_id      = var.vpc_id

  ingress {
    description = "PostgreSQL from VPC"
    from_port   = 5432
    to_port     = 5432
    protocol    = "tcp"
    cidr_blocks = ["10.0.0.0/16"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Project = var.project_name
  }
}

# DB SUBNET GROUP

resource "aws_db_subnet_group" "rds_subnet_group" {
  name       = "${var.project_name}-rds-subnet-group"
  subnet_ids = var.private_subnet_ids

  tags = {
    Project = var.project_name
  }
}

# POSTGRESQL RDS INSTANCE
resource "aws_db_instance" "postgres" {
  identifier              = "${var.project_name}-postgres"
  engine                  = "postgres"
  engine_version          = "15.5"
  instance_class          = "db.t3.micro"

  allocated_storage       = 20
  storage_encrypted       = true

  db_name                 = var.db_name
  username                = var.db_username
  password                = var.db_password

  db_subnet_group_name    = aws_db_subnet_group.rds_subnet_group.name
  vpc_security_group_ids  = [aws_security_group.rds_sg.id]

  publicly_accessible     = false
  skip_final_snapshot     = true
  deletion_protection     = false

  tags = {
    Name    = "${var.project_name}-postgres"
    Project = var.project_name
  }
}

```

### Step 4: Create outputs.tf
This is what other variables can use.

```
output "rds_endpoint" {
  description = "PostgreSQL endpoint"
  value       = aws_db_instance.postgres.endpoint
}

output "rds_port" {
  description = "PostgreSQL port"
  value       = aws_db_instance.postgres.port
}

output "rds_db_name" {
  description = "Database name"
  value       = aws_db_instance.postgres.db_name
}

```
### Step 5: Wire the RDS into the Root main.tf

```
module "rds" {
  source             = "./module-06-rds-terraform"
  project_name       = var.project_name
  vpc_id             = module.vpc.vpc_id
  private_subnet_ids = module.vpc.private_subnet_ids
  db_username        = var.db_username
  db_password        = var.db_password
}

```

### Step 6: Apply

From the root folder, apply:

```
terraform init
terraform plan
terraform apply
```



















