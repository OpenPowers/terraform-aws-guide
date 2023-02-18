EC2 数据源 - 使用最新的 AMI 数据源
===============================

## 知识点

* 之前我们通过TF启动的EC2都是写死的AMI，而amazon为我们维护了一套AMI，并且随时可能会发生更新，一旦amazon把AMI ID进行更新，此时如何反应到本地的TF维护文件当中呢？不可能每次都要手动进行修改。
  那么有没有一种方式，只要amazon的云端AMI ID发生更新，咱们再通过TF部署时自动引用最新的AMI？

* 利用 EC2 数据源， 保持使用最新的 AMI 镜像

## 官网

https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami

## 实战演习/说明讲解

+ 编写 EC2 AMI 数据源文件， 始终保持使用最新的 AMI
+ 编写 TF 资源描述文件， 引用最新的 AMI 数据源
+ 部署确认

## 操作步骤

### 目录结构

```bash
.
├── ec2-variables.tf
├── ec2-aws_ami.tf
├── ec2-sg.tf
├── ec2.tf
└── main.tf
```

### 编写变量定义文件

*ec2-variables.tf*

```terraform
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
```

### 编写 EC2 AMI 数据源文件， 始终保持使用最新的 AMI

*ec2-aws_ami.tf*

```terraform
data "aws_ami" "myami" {
  # 最新使用
  most_recent = true
  # amazon官方认证
  owners      = ["amazon"]

  # 镜像文件名称过滤
  filter {
    name   = "name"
    values = ["amzn-ami-hvm-*-x86_64-gp2"]
  }

  # 根设备过滤条件
  filter {
    name   = "root-device-type"
    values = ["ebs"]
  }
}
```

### 编写安全组定义文件

*ec2-sg.tf*

```terraform
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

### 编写 TF 资源描述文件， 引用最新的 AMI 数据源

*ec2.tf*

```terraform
###########################################################
# EC2 资源设置
resource "aws_instance" "myweb_server2" {
  # 引用 AMI 数据源
  ami           = data.aws_ami.myami.id
  instance_type = var.instance_type
  vpc_security_group_ids = [aws_security_group.learnaws-sg-web-ssh.id]

  tags = {
    Name = "learnaws-ec2-from-terraform"
  }
}
```

### 编写 TF 项目描述文件

*main.tf*

```terraform
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
