# 🏗️ Módulo 11: Terraform - Infrastructure as Code

> **Objetivo:** Dominar provisionamento de infraestrutura como código, essencial para DevOps moderno.

---

## 📌 Por que Terraform?

- **IaC padrão de mercado** (~65% das empresas usam)
- **Multi-cloud:** AWS, Azure, GCP, Kubernetes
- **Versionamento:** Infraestrutura no Git
- **Reprodutibilidade:** Ambientes idênticos
- **Auditoria:** Quem mudou o quê e quando

---

## 🎯 O que você vai aprender

| Seção | Tópico | Importância |
|-------|--------|-------------|
| 11.1 | Conceitos IaC | 🔴 Crítico |
| 11.2 | HCL Básico | 🔴 Crítico |
| 11.3 | Providers | 🔴 Crítico |
| 11.4 | Resources | 🔴 Crítico |
| 11.5 | Variables e Outputs | 🔴 Crítico |
| 11.6 | State Management | 🔴 Crítico |
| 11.7 | Modules | 🟡 Importante |
| 11.8 | Workspaces | 🟡 Importante |
| 11.9 | Terraform + LocalStack | 🔴 Crítico |
| 11.10 | CI/CD com Terraform | 🟡 Importante |

---

## 11.1 Conceitos IaC

### Infraestrutura Tradicional vs IaC

```
┌─────────────────────────────────────────────────────────────────┐
│                    TRADICIONAL (Console)                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Developer ───► AWS Console ───► Clica, configura ───► Recursos │
│                                                                  │
│  ❌ Não reproduzível                                             │
│  ❌ Não versionado                                               │
│  ❌ Propenso a erros                                             │
│  ❌ Sem auditoria                                                │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                    IaC (Terraform)                               │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Developer ───► Código .tf ───► Git ───► CI/CD ───► Recursos    │
│                                                                  │
│  ✅ Reproduzível (exato em qualquer ambiente)                   │
│  ✅ Versionado (Git history)                                    │
│  ✅ Code Review (PRs para infra)                                │
│  ✅ Auditável (quem, quando, o quê)                             │
└─────────────────────────────────────────────────────────────────┘
```

### Workflow Terraform

```
┌─────────┐     ┌─────────┐     ┌─────────┐     ┌─────────┐
│  Write  │────►│  Plan   │────►│  Apply  │────►│ Destroy │
│  (.tf)  │     │(preview)│     │(execute)│     │(cleanup)│
└─────────┘     └─────────┘     └─────────┘     └─────────┘
     │               │               │               │
     ▼               ▼               ▼               ▼
  Código HCL    Mostra mudanças   Cria/atualiza   Remove tudo
                 sem aplicar      recursos
```

---

## 11.2 HCL (HashiCorp Configuration Language)

### Sintaxe Básica

```hcl
# Bloco de recurso
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
  
  tags = {
    Name        = "Web Server"
    Environment = "production"
  }
}

# Bloco de data (ler dados existentes)
data "aws_ami" "ubuntu" {
  most_recent = true
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# Variável
variable "instance_type" {
  description = "Tipo da instância EC2"
  type        = string
  default     = "t3.micro"
}

# Output
output "instance_ip" {
  description = "IP público da instância"
  value       = aws_instance.web.public_ip
}

# Local
locals {
  common_tags = {
    Project     = "pix-system"
    ManagedBy   = "terraform"
    Environment = var.environment
  }
}
```

### Tipos de Dados

```hcl
# String
variable "region" {
  type    = string
  default = "sa-east-1"
}

# Number
variable "instance_count" {
  type    = number
  default = 2
}

# Bool
variable "enable_monitoring" {
  type    = bool
  default = true
}

# List
variable "availability_zones" {
  type    = list(string)
  default = ["sa-east-1a", "sa-east-1b"]
}

# Map
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t3.micro"
    prod = "t3.medium"
  }
}

# Object
variable "database_config" {
  type = object({
    engine         = string
    engine_version = string
    instance_class = string
    storage        = number
  })
  default = {
    engine         = "postgres"
    engine_version = "15.3"
    instance_class = "db.t3.micro"
    storage        = 20
  }
}
```

---

## 11.3 Providers

### Configuração do Provider AWS

```hcl
# versions.tf
terraform {
  required_version = ">= 1.5.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
    kubernetes = {
      source  = "hashicorp/kubernetes"
      version = "~> 2.23"
    }
  }
  
  # Backend para state remoto
  backend "s3" {
    bucket         = "pix-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "sa-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

# Provider AWS
provider "aws" {
  region = var.aws_region
  
  default_tags {
    tags = {
      Project     = "pix-system"
      ManagedBy   = "terraform"
      Environment = var.environment
    }
  }
}

# Provider AWS para LocalStack (desenvolvimento)
provider "aws" {
  alias = "localstack"
  
  region                      = "sa-east-1"
  access_key                  = "test"
  secret_key                  = "test"
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true
  
  endpoints {
    s3             = "http://localhost:4566"
    sqs            = "http://localhost:4566"
    sns            = "http://localhost:4566"
    secretsmanager = "http://localhost:4566"
    iam            = "http://localhost:4566"
  }
}
```

---

## 11.4 Resources

### VPC Completa

```hcl
# vpc.tf

# VPC
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = {
    Name = "${var.project_name}-vpc"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "main" {
  vpc_id = aws_vpc.main.id
  
  tags = {
    Name = "${var.project_name}-igw"
  }
}

# Subnets Públicas
resource "aws_subnet" "public" {
  count                   = length(var.availability_zones)
  vpc_id                  = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 8, count.index)
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = {
    Name = "${var.project_name}-public-${count.index + 1}"
    Type = "public"
  }
}

# Subnets Privadas
resource "aws_subnet" "private" {
  count             = length(var.availability_zones)
  vpc_id            = aws_vpc.main.id
  cidr_block        = cidrsubnet(var.vpc_cidr, 8, count.index + 10)
  availability_zone = var.availability_zones[count.index]
  
  tags = {
    Name = "${var.project_name}-private-${count.index + 1}"
    Type = "private"
  }
}

# NAT Gateway (para subnet privada acessar internet)
resource "aws_eip" "nat" {
  count  = length(var.availability_zones)
  domain = "vpc"
  
  tags = {
    Name = "${var.project_name}-nat-eip-${count.index + 1}"
  }
}

resource "aws_nat_gateway" "main" {
  count         = length(var.availability_zones)
  allocation_id = aws_eip.nat[count.index].id
  subnet_id     = aws_subnet.public[count.index].id
  
  tags = {
    Name = "${var.project_name}-nat-${count.index + 1}"
  }
}
```

### RDS PostgreSQL

```hcl
# rds.tf

resource "aws_db_subnet_group" "main" {
  name       = "${var.project_name}-db-subnet-group"
  subnet_ids = aws_subnet.private[*].id
  
  tags = {
    Name = "${var.project_name}-db-subnet-group"
  }
}

resource "aws_security_group" "rds" {
  name        = "${var.project_name}-rds-sg"
  description = "Security group for RDS"
  vpc_id      = aws_vpc.main.id
  
  ingress {
    from_port       = 5432
    to_port         = 5432
    protocol        = "tcp"
    security_groups = [aws_security_group.app.id]
  }
  
  tags = {
    Name = "${var.project_name}-rds-sg"
  }
}

resource "aws_db_instance" "main" {
  identifier = "${var.project_name}-postgres"
  
  engine         = "postgres"
  engine_version = "15.3"
  instance_class = var.db_instance_class
  
  allocated_storage     = 20
  max_allocated_storage = 100
  storage_type          = "gp3"
  storage_encrypted     = true
  
  db_name  = "pix"
  username = var.db_username
  password = var.db_password
  
  multi_az               = var.environment == "prod"
  db_subnet_group_name   = aws_db_subnet_group.main.name
  vpc_security_group_ids = [aws_security_group.rds.id]
  
  backup_retention_period = 7
  backup_window           = "03:00-04:00"
  maintenance_window      = "Mon:04:00-Mon:05:00"
  
  skip_final_snapshot = var.environment != "prod"
  
  tags = {
    Name = "${var.project_name}-postgres"
  }
}
```

### SQS e SNS

```hcl
# messaging.tf

# SQS Queue
resource "aws_sqs_queue" "transfer_events" {
  name                       = "${var.project_name}-transfer-events"
  delay_seconds              = 0
  max_message_size           = 262144
  message_retention_seconds  = 1209600  # 14 days
  receive_wait_time_seconds  = 20       # Long polling
  visibility_timeout_seconds = 60
  
  # Dead Letter Queue
  redrive_policy = jsonencode({
    deadLetterTargetArn = aws_sqs_queue.transfer_events_dlq.arn
    maxReceiveCount     = 3
  })
  
  tags = {
    Name = "${var.project_name}-transfer-events"
  }
}

resource "aws_sqs_queue" "transfer_events_dlq" {
  name                      = "${var.project_name}-transfer-events-dlq"
  message_retention_seconds = 1209600
  
  tags = {
    Name = "${var.project_name}-transfer-events-dlq"
  }
}

# SNS Topic
resource "aws_sns_topic" "pix_events" {
  name = "${var.project_name}-events"
  
  tags = {
    Name = "${var.project_name}-events"
  }
}

# SNS → SQS Subscription
resource "aws_sns_topic_subscription" "transfer_events" {
  topic_arn = aws_sns_topic.pix_events.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.transfer_events.arn
  
  filter_policy = jsonencode({
    eventType = ["TRANSFER_COMPLETED", "TRANSFER_FAILED"]
  })
}

# Policy para SNS publicar no SQS
resource "aws_sqs_queue_policy" "transfer_events" {
  queue_url = aws_sqs_queue.transfer_events.id
  
  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Effect    = "Allow"
        Principal = "*"
        Action    = "sqs:SendMessage"
        Resource  = aws_sqs_queue.transfer_events.arn
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.pix_events.arn
          }
        }
      }
    ]
  })
}
```

---

## 11.5 Variables e Outputs

### Variables

```hcl
# variables.tf

variable "project_name" {
  description = "Nome do projeto"
  type        = string
  default     = "pix-system"
}

variable "environment" {
  description = "Ambiente (dev, staging, prod)"
  type        = string
  
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment deve ser dev, staging ou prod."
  }
}

variable "aws_region" {
  description = "Região AWS"
  type        = string
  default     = "sa-east-1"
}

variable "vpc_cidr" {
  description = "CIDR block da VPC"
  type        = string
  default     = "10.0.0.0/16"
}

variable "availability_zones" {
  description = "Lista de AZs"
  type        = list(string)
  default     = ["sa-east-1a", "sa-east-1b"]
}

variable "db_username" {
  description = "Username do banco"
  type        = string
  sensitive   = true
}

variable "db_password" {
  description = "Password do banco"
  type        = string
  sensitive   = true
}

variable "db_instance_class" {
  description = "Classe da instância RDS"
  type        = string
  default     = "db.t3.micro"
}
```

### Outputs

```hcl
# outputs.tf

output "vpc_id" {
  description = "ID da VPC"
  value       = aws_vpc.main.id
}

output "public_subnet_ids" {
  description = "IDs das subnets públicas"
  value       = aws_subnet.public[*].id
}

output "private_subnet_ids" {
  description = "IDs das subnets privadas"
  value       = aws_subnet.private[*].id
}

output "rds_endpoint" {
  description = "Endpoint do RDS"
  value       = aws_db_instance.main.endpoint
}

output "rds_hostname" {
  description = "Hostname do RDS"
  value       = aws_db_instance.main.address
}

output "sqs_queue_url" {
  description = "URL da fila SQS"
  value       = aws_sqs_queue.transfer_events.url
}

output "sns_topic_arn" {
  description = "ARN do tópico SNS"
  value       = aws_sns_topic.pix_events.arn
}
```

### tfvars

```hcl
# terraform.tfvars (NÃO commitar senhas!)
project_name = "pix-system"
environment  = "prod"
aws_region   = "sa-east-1"

# prod.tfvars
db_instance_class = "db.t3.medium"

# Senhas via variável de ambiente ou Secrets Manager
# export TF_VAR_db_password="xxx"
```

---

## 11.6 State Management

### State Local vs Remoto

```
┌─────────────────────────────────────────────────────────────────┐
│                       STATE LOCAL                                │
├─────────────────────────────────────────────────────────────────┤
│  terraform.tfstate (arquivo local)                               │
│  ❌ Não compartilhável                                           │
│  ❌ Sem locking                                                  │
│  ❌ Risco de perda                                               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                       STATE REMOTO (S3)                          │
├─────────────────────────────────────────────────────────────────┤
│  s3://bucket/terraform.tfstate                                   │
│  ✅ Compartilhável (equipe)                                     │
│  ✅ Locking (DynamoDB)                                          │
│  ✅ Versionado                                                  │
│  ✅ Encriptado                                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Configurar Backend S3

```hcl
# backend.tf
terraform {
  backend "s3" {
    bucket         = "pix-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "sa-east-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # Para locking
  }
}

# Criar bucket e tabela (uma vez, manualmente ou com outro tf)
resource "aws_s3_bucket" "terraform_state" {
  bucket = "pix-terraform-state"
  
  lifecycle {
    prevent_destroy = true
  }
}

resource "aws_s3_bucket_versioning" "terraform_state" {
  bucket = aws_s3_bucket.terraform_state.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_dynamodb_table" "terraform_locks" {
  name         = "terraform-locks"
  billing_mode = "PAY_PER_REQUEST"
  hash_key     = "LockID"
  
  attribute {
    name = "LockID"
    type = "S"
  }
}
```

### Comandos de State

```bash
# Listar recursos no state
terraform state list

# Mostrar recurso específico
terraform state show aws_db_instance.main

# Mover recurso (renomear)
terraform state mv aws_db_instance.main aws_db_instance.postgres

# Remover do state (sem destruir)
terraform state rm aws_instance.old

# Importar recurso existente
terraform import aws_s3_bucket.existing my-existing-bucket
```

---

## 11.7 Modules

### Estrutura de Módulo

```
modules/
├── vpc/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── README.md
├── rds/
│   ├── main.tf
│   ├── variables.tf
│   └── outputs.tf
└── eks/
    ├── main.tf
    ├── variables.tf
    └── outputs.tf
```

### Criar Módulo

```hcl
# modules/vpc/main.tf
resource "aws_vpc" "this" {
  cidr_block           = var.cidr_block
  enable_dns_hostnames = true
  enable_dns_support   = true
  
  tags = merge(var.tags, {
    Name = "${var.name}-vpc"
  })
}

resource "aws_subnet" "public" {
  count                   = length(var.public_subnets)
  vpc_id                  = aws_vpc.this.id
  cidr_block              = var.public_subnets[count.index]
  availability_zone       = var.availability_zones[count.index]
  map_public_ip_on_launch = true
  
  tags = merge(var.tags, {
    Name = "${var.name}-public-${count.index + 1}"
  })
}

# modules/vpc/variables.tf
variable "name" {
  type = string
}

variable "cidr_block" {
  type    = string
  default = "10.0.0.0/16"
}

variable "public_subnets" {
  type = list(string)
}

variable "availability_zones" {
  type = list(string)
}

variable "tags" {
  type    = map(string)
  default = {}
}

# modules/vpc/outputs.tf
output "vpc_id" {
  value = aws_vpc.this.id
}

output "public_subnet_ids" {
  value = aws_subnet.public[*].id
}
```

### Usar Módulo

```hcl
# main.tf
module "vpc" {
  source = "./modules/vpc"
  
  name               = "pix-system"
  cidr_block         = "10.0.0.0/16"
  public_subnets     = ["10.0.1.0/24", "10.0.2.0/24"]
  availability_zones = ["sa-east-1a", "sa-east-1b"]
  
  tags = local.common_tags
}

module "rds" {
  source = "./modules/rds"
  
  name           = "pix-database"
  vpc_id         = module.vpc.vpc_id
  subnet_ids     = module.vpc.private_subnet_ids
  instance_class = "db.t3.micro"
  
  tags = local.common_tags
}

# Usando módulo público do Terraform Registry
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "~> 19.0"
  
  cluster_name    = "pix-cluster"
  cluster_version = "1.28"
  
  vpc_id     = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnet_ids
}
```

---

## 11.8 Workspaces

### Para Múltiplos Ambientes

```bash
# Criar workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Listar
terraform workspace list

# Trocar
terraform workspace select prod

# Qual workspace atual?
terraform workspace show
```

### Usar no Código

```hcl
# Configuração baseada no workspace
locals {
  environment = terraform.workspace
  
  instance_types = {
    dev     = "t3.micro"
    staging = "t3.small"
    prod    = "t3.medium"
  }
  
  instance_counts = {
    dev     = 1
    staging = 2
    prod    = 3
  }
}

resource "aws_instance" "app" {
  count         = local.instance_counts[local.environment]
  instance_type = local.instance_types[local.environment]
  # ...
}
```

---

## 11.9 Terraform + LocalStack

### Configuração Completa

```hcl
# localstack.tf
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region                      = "sa-east-1"
  access_key                  = "test"
  secret_key                  = "test"
  skip_credentials_validation = true
  skip_metadata_api_check     = true
  skip_requesting_account_id  = true
  
  endpoints {
    s3             = "http://localhost:4566"
    sqs            = "http://localhost:4566"
    sns            = "http://localhost:4566"
    secretsmanager = "http://localhost:4566"
    dynamodb       = "http://localhost:4566"
    iam            = "http://localhost:4566"
    sts            = "http://localhost:4566"
  }
}

# Recursos funcionam normalmente!
resource "aws_s3_bucket" "documents" {
  bucket = "pix-documents"
}

resource "aws_sqs_queue" "events" {
  name = "pix-events"
}

resource "aws_secretsmanager_secret" "database" {
  name = "pix/database"
}

resource "aws_secretsmanager_secret_version" "database" {
  secret_id = aws_secretsmanager_secret.database.id
  secret_string = jsonencode({
    username = "pix"
    password = "localdev123"
    host     = "localhost"
    port     = 5432
    database = "pix"
  })
}
```

### Script de Setup

```bash
#!/bin/bash
# setup-local.sh

# Subir LocalStack
docker-compose -f docker-compose.localstack.yml up -d

# Aguardar LocalStack
echo "Aguardando LocalStack..."
sleep 10

# Aplicar Terraform
cd terraform/localstack
terraform init
terraform apply -auto-approve

echo "Infraestrutura local criada!"
```

---

## 11.10 CI/CD com Terraform

### GitHub Actions

```yaml
# .github/workflows/terraform.yml
name: Terraform

on:
  push:
    branches: [main]
    paths:
      - 'terraform/**'
  pull_request:
    branches: [main]
    paths:
      - 'terraform/**'

env:
  TF_VERSION: '1.5.0'
  AWS_REGION: 'sa-east-1'

jobs:
  plan:
    name: Terraform Plan
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: terraform/prod
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Format Check
        run: terraform fmt -check
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan -out=tfplan
      
      - name: Upload Plan
        uses: actions/upload-artifact@v3
        with:
          name: tfplan
          path: terraform/prod/tfplan
  
  apply:
    name: Terraform Apply
    needs: plan
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    environment: production
    defaults:
      run:
        working-directory: terraform/prod
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v3
        with:
          terraform_version: ${{ env.TF_VERSION }}
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ${{ env.AWS_REGION }}
      
      - name: Download Plan
        uses: actions/download-artifact@v3
        with:
          name: tfplan
          path: terraform/prod
      
      - name: Terraform Init
        run: terraform init
      
      - name: Terraform Apply
        run: terraform apply -auto-approve tfplan
```

---

## 🎓 Perguntas de Entrevista

### Nível Pleno

1. **"O que é Terraform e por que usar?"**
   - IaC para provisionar infraestrutura
   - Versionável, reproduzível, auditável
   - Multi-cloud

2. **"Qual a diferença entre terraform plan e apply?"**
   - Plan: simula mudanças sem aplicar
   - Apply: executa as mudanças

3. **"O que é o state do Terraform?"**
   - Arquivo que mapeia recursos reais aos declarados
   - Deve ser armazenado remotamente (S3)
   - Precisa de locking para evitar conflitos

### Nível Senior

4. **"Como você organiza Terraform para múltiplos ambientes?"**
   - Workspaces ou diretórios separados
   - Módulos reutilizáveis
   - tfvars por ambiente

5. **"Como garantir segurança em IaC?"**
   - Secrets via variáveis de ambiente ou Secrets Manager
   - State encriptado
   - Code review em PRs
   - Terraform Cloud/Enterprise para governança

---

## 📚 Comandos Essenciais

```bash
# Inicialização
terraform init              # Baixa providers e módulos
terraform init -upgrade     # Atualiza providers

# Planejamento
terraform plan              # Mostra mudanças
terraform plan -out=plan    # Salva plan em arquivo

# Execução
terraform apply             # Aplica mudanças
terraform apply plan        # Aplica plan salvo
terraform apply -auto-approve  # Sem confirmação

# Destruição
terraform destroy           # Remove tudo
terraform destroy -target=aws_instance.web  # Remove específico

# Formatação
terraform fmt               # Formata arquivos
terraform fmt -check        # Verifica formatação

# Validação
terraform validate          # Valida sintaxe

# Debug
TF_LOG=DEBUG terraform apply  # Logs detalhados
```

---

## 📚 Recursos Adicionais

- [Terraform Documentation](https://developer.hashicorp.com/terraform/docs)
- [Terraform AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)
- [Terraform Best Practices](https://www.terraform-best-practices.com/)
- [LocalStack + Terraform](https://docs.localstack.cloud/user-guide/integrations/terraform/)

---

**Próximo:** Aplicar tudo no [ROTEIRO_IMPLEMENTACAO.md](ROTEIRO_IMPLEMENTACAO.md) - Sprints 8-11
