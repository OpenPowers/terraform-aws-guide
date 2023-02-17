状态文件 - terraform.tfstate
===========================

## 知识点

* Terraform 的状态文件 - terraform.tfstate

## 官网

https://www.terraform.io/language/state

## 说明讲解

### tfstate 是什么？

.tfstate 文件是 Terraform 管理现有资源状态的文件。

### 为什么要有 tfstate

每当执行 `terraform plan` 的时候， Terraform 都会为我们比较本地资源的变化和服务器资源的不同。

这点和 CloudFormation 不一样， CloudFormation 是存在云端的。

大家请想一个问题, 为什么我们可以通过 `terraform apply` 建立出的 EC2, 马上可以通过 `terraform destroy` 进行销毁呢？ 

Terraform 是怎么知道该销毁哪台 EC2 的? 会不会将云中的其他 EC2 销毁呢？ 

如果没有状态管理， 这就很危险。

### terraform.tfstate文件的作用

* *terraform.tfstate*记录了**当前tf管理的资源的状态**。
* 如果我们在此之前执行了`terraform destroy` 命令，把资源都销毁了，那么*terraform.tfstate*文件里面是看不到任何resources信息的，因为该文件记录的是当前的tf管理资源的状态。
* 但是，Terraform为我们提供了上一次通过tf管理资源的*terraform.tfstate.backup*文件。

### 一起来看一下 tfstate 文件样例

```bash
$ cat terraform.tfstate
```

*terraform.tfstate*

```json
{
  "version": 4,
  // 采用版本
  "terraform_version": "1.1.4",
  // 运行次数
  "serial": 9,
  // 项目唯一ID。如果本地用tf管理多个项目，但是因为ID是唯一的，这会确保不同的tf不会影响到云端的同一个资源
  "lineage": "a81e9642-ae12-9afc-ab52-1bb1ca1d5b66",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      // 资源类型
      "type": "aws_instance",
      // 资源名称(TF使用)
      "name": "myweb_server2",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            // AMI-ID
            "ami": "ami-0218d08a1f9dac831",
            // EC2 ARN
            "arn": "arn:aws:ec2:ap-northeast-1:168571836196:instance/i-06f67dc122136c534",
            // EC2 各种属性设置 ...
            "associate_public_ip_address": true,
            "availability_zone": "ap-northeast-1a",
            "capacity_reservation_specification": [
              {
                "capacity_reservation_preference": "open",
                "capacity_reservation_target": []
              }
            ],
            "cpu_core_count": 1,
            "cpu_threads_per_core": 1,
            "credit_specification": [
              {
                "cpu_credits": "standard"
              }
            ],
            "disable_api_termination": false,
            "ebs_block_device": [],
            "ebs_optimized": false,
            "enclave_options": [
              {
                "enabled": false
              }
            ],
            "ephemeral_block_device": [],
            "get_password_data": false,
            "hibernation": false,
            "host_id": null,
            "iam_instance_profile": "",
            "id": "i-06f67dc122136c534",
            "instance_initiated_shutdown_behavior": "stop",
            "instance_state": "running",
            "instance_type": "t2.micro",
            "ipv6_address_count": 0,
            "ipv6_addresses": [],
            "key_name": "",
            "launch_template": [],
            "metadata_options": [
              {
                "http_endpoint": "enabled",
                "http_put_response_hop_limit": 1,
                "http_tokens": "optional"
              }
            ],
            "monitoring": false,
            "network_interface": [],
            "outpost_arn": "",
            "password_data": "",
            "placement_group": "",
            "placement_partition_number": null,
            "primary_network_interface_id": "eni-04777e6b3d682c7e8",
            "private_dns": "ip-172-31-45-191.ap-northeast-1.compute.internal",
            "private_ip": "172.31.45.191",
            "public_dns": "ec2-13-231-129-70.ap-northeast-1.compute.amazonaws.com",
            "public_ip": "13.231.129.70",
            "root_block_device": [
              {
                "delete_on_termination": true,
                "device_name": "/dev/xvda",
                "encrypted": false,
                "iops": 100,
                "kms_key_id": "",
                "tags": {},
                "throughput": 0,
                "volume_id": "vol-013199e72886ccebd",
                "volume_size": 8,
                "volume_type": "gp2"
              }
            ],
            "secondary_private_ips": [],
            "security_groups": [
              "default"
            ],
            "source_dest_check": true,
            "subnet_id": "subnet-0647f7f6ae12f3d59",
            "tags": {
              "Name": "learnaws-ec2-from-terraform2"
            },
            "tags_all": {
              "Name": "learnaws-ec2-from-terraform2"
            },
            "tenancy": "default",
            "timeouts": null,
            "user_data": "4d496b03906764122c3ddb6f8bba0bddb432ec11",
            "user_data_base64": null,
            "volume_tags": null,
            "vpc_security_group_ids": [
              "sg-0e114289b84c9418a"
            ]
          },
          "sensitive_attributes": [],
          "private": "eyJlMmJmYjczMC1lY2FhLTExZTYtOGY4OC0zNDM2M2JjN2M0YzAiOnsiY3JlYXRlIjo2MDAwMDAwMDAwMDAsImRlbGV0ZSI6MTIwMDAwMDAwMDAwMCwidXBkYXRlIjo2MDAwMDAwMDAwMDB9LCJzY2hlbWFfdmVyc2lvbiI6IjEifQ=="
        }
      ]
    }
  ]
}
```

## 强烈建议

* 在进行源代码管理时，务必把*terraform.tfstate*文件也管理进去。
* 因为一旦*terraform.tfstate*文件丢失，tf就不知道该管理哪些资源
* *terraform.tfstate*虽然是自动生成自动维护，但确是使用tf管理云资源最最重要的文件，一定不可以丢失。