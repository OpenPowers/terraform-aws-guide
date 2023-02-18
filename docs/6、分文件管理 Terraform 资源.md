分文件管理资源
=============

## 知识点

* 分文件管理 Terraform 资源

## 实战演习/说明讲解

+ 拆分 EC2 资源描述文件
+ 部署确认

## 操作步骤

### 修改main.tf文件仅保留基础设置

*main.tf*

```terraform
###########################################################
# Terraform 基本设置
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.74"
    }
  }
  required_version = ">= 1.1.4"
}

###########################################################
# 提供商设置(云平台)
provider "aws" {
  profile = "learnaws"
  region  = "ap-northeast-1"
}
```

### 拆分 EC2 资源描述文件

```bash
$ vi ec2.tf
```

*ec2.tf*

```terraform
###########################################################
# EC2 资源设置
resource "aws_instance" "myweb_server2" {
  ami           = "ami-0218d08a1f9dac831"
  instance_type = "t3.micro"
  tags = {
    Name = "learnaws-ec2-from-terraform2"
  }
}
```

### 部署确认

* Terraform会把当前工程目录下所有.tf文件都会获取，不存在入口文件一说

```bash
# 目录初始化
$ terraform init
# 检验 tf 文件
$ terraform validate
# 实施计划, 准备资源
$ terraform plan
# 应用部署
$ terraform apply
# 摧毁系统
$ terraform destroy
```

