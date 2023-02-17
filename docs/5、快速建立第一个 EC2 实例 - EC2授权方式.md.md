# 快速建立第一个 EC2 实例 - EC2授权方式

## 知识点

* 使用 Terraform 在 Amazon Linux 2@EC2 上快速建立第一个 EC2 实例

## 实战演习/说明讲解

+ 准备 tf 模版
+ 在 EC2 上执行tf，部署 EC2 实例
+ 动作确认

## 操作步骤

### 准备 tf 模版

*main.ts*

```bash
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
	# 这里写default即可，因为我们的EC2上没有设置aws cli，而是给这台EC2进行了角色授权
  profile = "default"
  region  = "ap-northeast-1"
}

###########################################################
# 生成的资源设置
resource "aws_instance" "myweb_server" {
  ami           = "ami-0218d08a1f9dac831"
  instance_type = "t2.micro"
  # 增加一个 KeyPair 设置
  key_name      = "learnaws-ssh-key"
  tags = {
    Name = "learnaws-ec2-from-terraform"
  }
}
```

### 在 EC2 上部署 EC2 实例

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

### 动作确认

Done.