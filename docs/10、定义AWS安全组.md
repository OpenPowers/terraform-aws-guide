为EC2定义AWS安全组
=====================

## 知识点

> 之前我们部署的EC2都是用的默认安全组，在实际环境中，要根据每台EC2的职能使用不同的安全组

* 使用 Terraform 定义安全组

## 官网

https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group

## 实战演习/说明讲解

+ 编写安全组定义文件
+ 编写 TF 资源描述文件， 引用安全组定义
+ 部署确认

## 操作步骤

### 目录结构

```bash
.
├── ec2-variables.tf
├── ec2-sg.tf
├── ec2.tf
└── main.tf
```

### 编写变量定义文件

*ec2-variables.tf*

```bash
# AWS Region
variable "aws_region" {
  description = "AWS Region"
  type = string
  default = "ap-northeast-1"
}

# AWS EC2 Instance Type
variable "instance_type" {
  description = "EC2 Instance Type"
  type = string
  default = "t3.micro"  
}

# AWS EC2 Instance Key Pair
variable "ami_id" {
  description = "AMI Image ID"
  type = string
  default = "ami-0218d08a1f9dac831"
}
```

### 编写安全组定义文件

*ec2-sg.tf*

```bash
# Create Security Group
resource "aws_security_group" "learnaws-sg-web-ssh" {
  # 安全组的名称
  name        = "learnaws-sg-web-ssh"
  description = "Web and SSH"
  # 安全组的入口描述
  ingress {
    description = "Allow Port 80"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    # 允许范围，这里是所有ip都可访问
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "Allow Port 443"
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  ingress {
    description = "Allow Port 22"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
    # cidr_blocks = ["12.15.55.32/32"]
  }
  # 安全组的出口描述
  egress {
    description = "Allow all ip and ports outbound"
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "learnaws-sg-web-ssh"
  }
}
```

### 编写 TF 资源描述文件， 引用安全组定义

*ec2.tf*

```bash
###########################################################
# EC2 资源设置
resource "aws_instance" "myweb_server2" {
  ami           = var.ami_id
  instance_type = var.instance_type
  # 为EC2指定安全组的ids
  vpc_security_group_ids = [aws_security_group.learnaws-sg-web-ssh.id]

  tags = {
    Name = "learnaws-ec2-from-terraform-cli"
  }
}
```

### 编写 TF 项目描述文件

*main.tf*

```bash
###########################################################
# Terraform 基本设置
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.71"
    }
  }
  required_version = ">= 1.1.3"
}

###########################################################
# 提供商设置(云平台)
provider "aws" {
  profile = "learnaws"
  region  = var.aws_region
}
```

### 部署确认

```bash
# 目录初始化
$ terraform init
# 检验 tf 文件
$ terraform validate
# 实施计划, 准备资源
$ terraform plan
# 应用部署
$ terraform apply
$ terraform apply -auto-approve
# 摧毁系统
$ terraform destroy
$ terraform destroy -auto-approve
```

