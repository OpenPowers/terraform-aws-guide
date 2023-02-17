一、Terraform CLI 工具的安装
=======================

## 知识点

* Terraform CLI 工具的安装 - @macOS @Linux

## 官网

https://www.terraform.io/downloads

## 操作步骤

### 从官网下载安装

https://releases.hashicorp.com/terraform/

这里选择：`terraform_1.3.8_darwin_amd64.zip`

```bash
$ unzip terraform_x.xx.x_darwin_amd64.zip
$ mv terraform /usr/local/bin
```

### 在 macOS 上安装 Terraform CLI 命令行工具

https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started

```bash
#######################################
# 使用 Homebrew 安装 Terraform CLI
$ brew tap hashicorp/tap								# 注册hashicorp信息库
$ brew install hashicorp/tap/terraform	# 安装terraform
# 确认安装
$ terraform -help
$ terraform version
# 各种子命令确认
$ terraform -help init									# 初始化
$ terraform -help validate							# 验证Terraform配置文件的语法是否正确
$ terraform -help plan									# 
$ terraform -help apply
$ terraform -help destroy

#######################################
# Homebrew 本体更新
$ brew update
# 更新 Terraform CLI
$ brew upgrade hashicorp/tap/terraform

#######################################
# Tab 自动完成(可选安装, 有安全隐患, 可跳过)
## Bash 的场合
# $ touch ~/.bashrc
## Zsh  的场合
$ touch ~/.zshrc
# 安装 Terraform 自动完成包
$ terraform -install-autocomplete
```

[macos系统的Homebrew包管理工具官网](https://brew.sh/)

### 在 Linux 上安装 Terraform CLI 命令行工具

```bash
# 安装 yum 管理工具 - yum-config-manager
$ sudo yum install -y yum-utils
# 注册 HashiCorp 信息库
$ sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
# 安装 terraform
$ sudo yum -y install terraform
# 确认安装
$ terraform -help
$ terraform version
```



二、AWS CLI 命令行工具安装及设置
=====================

## 实战演习/说明讲解

+ 安装 AWS CLI 命令行工具
+ 设置授权（方式一） - 本地授权
+ 设置授权（方式二） - EC2 角色授权

## 操作步骤

### 安装 AWS CLI 命令行工具

https://docs.aws.amazon.com/zh_cn/cli/latest/userguide/install-cliv2.html

+ Docker
+ Linux
+ macOS
+ Windows

```bash
# 版本确认
$ aws --version
# 确认本地授权文件
$ cat ~/.aws/credentials
```

### 设置授权（方式一） - 本地授权

* 登陆AWS console，进入“IAM”页面，点击 “用户” ，添加“用户”并设置勾选访问类型“编程访问”

* 设置相应权限

* 创建用户，并保存访问密钥对（注意只显示一次）

* 授权CLI身份
  
  ```bash
  # 授权CLI身份(指定profile方式 - AWS Best Practices)
  $ aws configure --profile learnaws
  >AWS Access Key ID [None]: FFUff89898u89vxv
  AWS Secret Access Key [None]: Ahy789yvhjxlFDoajsdflajia90sdfjsaa
  >Default region name [None]: ap-northeast-1
  Default output format [None]: json
  # 授权身份存储位置(******内容涉及敏感信息******)
  $ ls ~/.aws
  # 列出所有配置数据
  $ aws configure list --profile learnaws
  # 列出可用配置文件名称
  $ aws configure list-profiles
  # 列出当前区的VPC
  $ aws ec2 describe-vpcs --profile learnaws
  # 查询过滤显示
  $ aws ec2 describe-vpcs --profile learnaws --query 'Vpcs[].VpcId'
  # 列出S3存储桶
  $ aws s3 ls --profile learnaws
  ```
  
  ```bash
  # 此命令来验证您的 AWS 访问密钥是否正确，以及您是否已获得必要的 AWS 资源访问权限。如果该命令返回您的身份信息(账户ID、用户ID、ARN)，表示您已通过身份验证，并且可以在 AWS 上执行其他操作。如果出现错误，说明您的访问密钥无效或您没有必要的 AWS 资源访问权限。
  $ aws sts get-caller-identity
  ```
  
  

### 设置授权（方式二） - 给EC2进行角色授权

使用下面的AWS CloudFormation 模版*awsNodeJSEC2.yml*，已经设置好 EC2 权限

```yaml
Parameters:
  MyKeyName:
    Description: Name of EC2 KeyPair
    Type: AWS::EC2::KeyPair::KeyName
    Default: learn-aws-ssh-key
    ConstraintDescription: must input

Resources:
  awsNodeJSEC2:
    Type: AWS::EC2::Instance
    Properties:
      KeyName: !Ref MyKeyName
      AvailabilityZone:
        { "Fn::Select": ["0", { "Fn::GetAZs": { "Ref": "AWS::Region" } }] }
      ImageId: ami-0218d08a1f9dac831
      InstanceType: t3.micro
      SecurityGroups:
        - !Ref awsNodeJSSSHWEBSG
      IamInstanceProfile: !Ref awsNodeJSMyInstanceProfile
      BlockDeviceMappings:
        - DeviceName: /dev/xvda
          Ebs:
            VolumeType: gp2
            VolumeSize: 8
            DeleteOnTermination: true
            Encrypted: false
      UserData:
        Fn::Base64: |
          #!/bin/bash
          yum update -y
          timedatectl set-timezone "Asia/Tokyo"
          localectl set-locale LANG=ja_JP.UTF-8          
          touch /home/ec2-user/.hushlogin
          chown ec2-user:ec2-user /home/ec2-user/.hushlogin
  
          curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "/tmp/awscliv2.zip"
          unzip /tmp/awscliv2.zip  -d /tmp/awscliv2/
          /tmp/awscliv2/aws/install

          curl -sL https://rpm.nodesource.com/setup_14.x | bash -
          yum install -y gcc-c++ make
          yum install -y nodejs

  awsNodeJSSSHWEBSG:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: SSH and Web PORT
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  awsNodeJSAccessRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          - Effect: "Allow"
            Principal:
              Service:
                - "ec2.amazonaws.com"
            Action:
              - "sts:AssumeRole"
      Path: "/"
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/AdministratorAccess

  awsNodeJSMyInstanceProfile:
    Type: "AWS::IAM::InstanceProfile"
    Properties:
      Path: "/"
      Roles:
        - !Ref awsNodeJSAccessRole

  awsNodeJSEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref awsNodeJSEC2
```





# 三、VSCode 插件安装

> 在 VSCode 中安装 Terraform 插件

## 官网

https://marketplace.visualstudio.com/items?itemName=HashiCorp.terraform

## 操作步骤

### 在 VSCode 中安装 Terraform 插件

在 VSCode 中搜索扩展 HashiCorp Terraform， 然后安装。