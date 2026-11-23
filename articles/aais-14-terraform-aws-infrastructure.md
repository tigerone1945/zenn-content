---
title: "第14回 TerraformでAIエージェントのAWS基盤を作る──Infrastructure as Code入門"
emoji: "🏗️"
type: "tech"
topics:
  - AIエージェント
  - Terraform
  - AWS
  - InfrastructureAsCode
  - ECS
published: true
---

# はじめに

第13回では、Data LayerをSQLiteからPostgreSQLへ移行しました。

ここまででApplicationは、

```text
Streamlit
↓
FastAPI
↓
Manager Agent
↓
Function Tools
↓
Repository
↓
PostgreSQL
```

という構成になっています。

ローカル環境ではDocker Composeを使って、

```text
ui
api
db
```

を動かしています。

次に必要なのは、

> **このApplicationをAWS上で動かすためのInfrastructure**

です。

AWS Management Consoleから手作業で作ることもできます。

しかし今回は、

> **TerraformでInfrastructureをコード化する**

方針を採用します。

---

# 今回のゴール

今回のゴールは、

```text
Terraform
↓
AWS Infrastructure
```

を作れる状態にすることです。

第15回で実際にApplicationをDeployする前に、

```text
Network
Container Registry
Container Runtime
Database
Logging
```

の土台を定義します。

---

# 今回作るAWS構成

最終的には次の構成を目指します。

```text
Internet
↓
Application Load Balancer
↓
ECS / Fargate
├─ API Container
└─ UI Container
↓
RDS PostgreSQL

Container Image
↑
ECR

Logs
↓
CloudWatch
```

---

# なぜTerraformなのか

Terraformでは、

```text
VPC
Subnet
Security Group
ECR
ECS
RDS
CloudWatch
```

などを`.tf`ファイルとして定義できます。

つまり、

> **Infrastructureの構成を再現可能なコードにする**

ことができます。

---

# Terraformの基本Workflow

このシリーズでは、Terraform操作を次の順に固定します。

```text
terraform fmt
↓
terraform validate
↓
terraform plan
↓
Human Review
↓
terraform apply
```

いきなり`apply`しません。

特に、

> **planを人が確認してからapplyする**

ことを基本にします。

---

# フォルダー構成

```text
ai-agent-practice/
├── api/
├── ui/
├── terraform/
│   ├── main.tf
│   ├── providers.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── network.tf
│   ├── ecr.tf
│   ├── ecs.tf
│   ├── rds.tf
│   └── logs.tf
├── compose.yaml
├── Dockerfile.api
├── Dockerfile.ui
└── ...
```

---

# Terraformを初期化する

まずTerraformが利用できる環境を準備します。

MacならHomebrewの例として、

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
```

があります。

確認：

```bash
terraform version
```

---

# AWS Providerを定義する

`terraform/providers.tf`：

```hcl
terraform {
  required_version = ">= 1.9.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
}
```

Provider VersionはProject内で固定します。

---

# Variableを定義する

`variables.tf`：

```hcl
variable "aws_region" {
  type        = string
  description = "AWS region"
  default     = "ap-northeast-1"
}

variable "project_name" {
  type        = string
  description = "Project name"
  default     = "ai-agent-practice"
}

variable "db_username" {
  type        = string
  description = "RDS master username"
  sensitive   = true
}

variable "db_password" {
  type        = string
  description = "RDS master password"
  sensitive   = true
}
```

---

# VPCを作る

`network.tf`の最小例です。

```hcl
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true

  tags = {
    Name = "${var.project_name}-vpc"
  }
}
```

実際には、

```text
Public Subnet
Private Subnet
Route Table
Internet Gateway
NAT Gateway
```

などが必要になります。

今回はInfrastructure全体の役割を理解することを優先します。

---

# ECRを定義する

Container Imageの保存先です。

`ecr.tf`：

```hcl
resource "aws_ecr_repository" "api" {
  name = "${var.project_name}-api"

  image_scanning_configuration {
    scan_on_push = true
  }
}

resource "aws_ecr_repository" "ui" {
  name = "${var.project_name}-ui"

  image_scanning_configuration {
    scan_on_push = true
  }
}
```

第15回では、ここへDocker ImageをPushします。

---

# ECS Clusterを定義する

`ecs.tf`：

```hcl
resource "aws_ecs_cluster" "main" {
  name = "${var.project_name}-cluster"
}
```

ApplicationはECS/Fargateで動かします。

---

# CloudWatch Log Groupを作る

`logs.tf`：

```hcl
resource "aws_cloudwatch_log_group" "api" {
  name              = "/ecs/${var.project_name}/api"
  retention_in_days = 14
}

resource "aws_cloudwatch_log_group" "ui" {
  name              = "/ecs/${var.project_name}/ui"
  retention_in_days = 14
}
```

第16回でMonitoringを詳しく扱います。

---

# RDS PostgreSQLを定義する

`rds.tf`の考え方です。

```hcl
resource "aws_db_instance" "postgres" {
  identifier = "${var.project_name}-postgres"

  engine         = "postgres"
  engine_version = "16"

  instance_class        = "db.t4g.micro"
  allocated_storage     = 20
  max_allocated_storage = 100

  db_name  = "ai_agent"
  username = var.db_username
  password = var.db_password

  publicly_accessible = false

  skip_final_snapshot = true
}
```

これは学習用の最小例です。

本番では、

```text
DB Subnet Group
Security Group
Backup
Deletion Protection
Multi-AZ
Encryption
Final Snapshot
```

などを検討します。

---

# RDSはPrivateにする

重要なのは、

```hcl
publicly_accessible = false
```

です。

基本構成では、

```text
Internet
↓
ALB
↓
ECS
↓
RDS
```

とし、

> **RDSをInternetへ直接公開しない**

構成にします。

---

# Security Groupで通信範囲を絞る

概念的には、

```text
ALB SG
↓ port 8000 / 8501
ECS SG

ECS SG
↓ port 5432
RDS SG
```

とします。

RDSの5432を、

```text
0.0.0.0/0
```

へ開放しないようにします。

---

# Terraform init

`terraform`フォルダーへ移動します。

```bash
cd terraform
```

初期化：

```bash
terraform init
```

Provider Pluginなどが準備されます。

---

# fmt

```bash
terraform fmt -recursive
```

Formattingを揃えます。

---

# validate

```bash
terraform validate
```

Configurationの構文・内部整合性を確認します。

---

# plan

```bash
terraform plan
```

ここで、

```text
何が作られるか
何が変更されるか
何が削除されるか
```

を確認します。

---

# planはHuman Review Point

Infrastructureでは、

```text
terraform apply
```

の前に、

> **planをレビューする**

ことが非常に重要です。

特に、

```text
destroy
replace
Security Group変更
RDS変更
```

などがないか確認します。

---

# apply

内容を確認したら、

```bash
terraform apply
```

を実行します。

学習環境でもAWS Resourceには費用が発生する可能性があります。

不要になったら必ずResourceを確認してください。

---

# destroy

学習環境を削除するときは、

```bash
terraform plan -destroy
```

で削除内容を確認してから、

```bash
terraform destroy
```

を実行します。

---

# StateをGitへ入れない

TerraformはStateを持ちます。

```text
terraform.tfstate
```

にはInfrastructure情報が含まれます。

`.gitignore`には、

```gitignore
.terraform/
*.tfstate
*.tfstate.*
.terraform.lock.hcl
```

と機械的に全部除外するのではなく、`.terraform.lock.hcl`は通常Version Control対象にする運用も一般的です。

最低限、

```gitignore
.terraform/
*.tfstate
*.tfstate.*
```

はRepositoryへ不用意に入れないようにします。

---

# Passwordをtfvarsへ直書きしない

今回Variableとして、

```text
db_username
db_password
```

を定義しました。

しかしGit管理する`.tf`や`.tfvars`へ、本物のPasswordを直書きしないようにします。

第16回では、

```text
AWS Secrets Manager
```

を使います。

---

# 今回のArchitecture

```text
Terraform
│
├─ Network
│  ├─ VPC
│  ├─ Subnets
│  └─ Security Groups
│
├─ ECR
│  ├─ api
│  └─ ui
│
├─ ECS
│  └─ Fargate
│
├─ RDS
│  └─ PostgreSQL
│
└─ CloudWatch
   └─ Log Groups
```

---

# 第14回で理解しておきたい5つのこと

```text
1. Infrastructure as Code
AWS構成をコード化する

2. Provider
TerraformとAWSを接続する

3. Resource
AWS Resourceを宣言する

4. plan
変更内容を事前確認する

5. State
TerraformがInfrastructure状態を管理する
```

---

# 今回あえてやらなかったこと

| 機能 | 今回 | 扱う回 |
|---|---:|---:|
| Terraform | ○ | 第14回 |
| ECR定義 | ○ | 第14回 |
| ECS基盤 | ○ | 第14回 |
| RDS基盤 | ○ | 第14回 |
| Image Push | × | 第15回 |
| ECS Deploy | × | 第15回 |
| Secrets Manager | × | 第16回 |
| Monitoring | × | 第16回 |

---

# まとめ

今回は、AIエージェントApplicationのAWS基盤をTerraformで定義しました。

中心は、

```text
terraform fmt
↓
terraform validate
↓
terraform plan
↓
Human Review
↓
terraform apply
```

というWorkflowです。

InfrastructureもApplication Codeと同じように、

> **再現可能・Review可能なArtifactとして管理する**

ことが重要です。

---

# 次回予告

## 第15回：Docker化したAIエージェントをAWSへ──ECR・ECS/Fargate・RDSで動かす

次回は、

```text
Docker Image
↓
ECR
↓
ECS / Fargate
↓
RDS
```

というDeploy Flowを実行します。

---

# 関連記事・教材

## AIエージェント実践ロードマップ

SDD・設計・実装・PoC・AWS本番までの全体像はこちら。

https://zenn.dev/tigerone1945/articles/aais-00-ai-agent-implementation-roadmap-hub

---

# 参考

Terraform AWS Provider：

https://registry.terraform.io/providers/hashicorp/aws/latest/docs

Terraform Providers：

https://developer.hashicorp.com/terraform/language/providers

Terraform RDS Tutorial：

https://developer.hashicorp.com/terraform/tutorials/aws/aws-rds
