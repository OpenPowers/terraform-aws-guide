Output Values - 定义输出内容
==========================

## 知识点

* 当我们每次执行`terraform apply`命令后，会输出很多日志，我们可以进行定义输出内容，把比较关心的日志进行输出

* 定义 Terraform 管理的云端资源的输出内容

## 官网

https://www.terraform.io/language/values/outputs

## 实战演习/说明讲解

+ 定于资源输出显示的内容
+ 运行模版确认动作

## 操作步骤

### 目录结构

```bash
.
├── ec2-variables.tf
├── ec2-aws_ami.tf
├── ec2-sg.tf
├── ec2.tf
├── main.tf
└── ec2-outputs.tf
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

### 编写 TF 资源描述文件

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

### 定义资源输出显示的内容

*ec2-outputs.tf*

```terraform
output "instance_publicip" {
  # EC2 公有IP
  value = aws_instance.myweb_server2.public_ip
}
output "instance_availability_zone" {
  # 部署的可用区
  value = aws_instance.myweb_server2.availability_zone
}
output "instance_ami" {
  # 使用的 AMI
  value = aws_instance.myweb_server2.ami
}
output "instance_private_ip" {
  # EC2 私有网断 IP
  value = aws_instance.myweb_server2.private_ip
}
output "instance_cpu_core_count" {
  # EC2 CPU 核心数
  value = aws_instance.myweb_server2.cpu_core_count
}
output "instance_instance_type" {
  # EC2 实例类型
  value = aws_instance.myweb_server2.instance_type
}

# 最后输出的样式如下:
# instance_ami = "ami-0d9f3ffed238c343f"
# instance_availability_zone = "ap-northeast-1c"
# instance_cpu_core_count = "1"
# instance_instance_type = "t3.micro"
# instance_private_ip = "172.31.13.114"
# instance_publicip = "18.183.155.128"
```

_EC2资源属性参照_

https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance

### 运行模版确认动作

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
```

