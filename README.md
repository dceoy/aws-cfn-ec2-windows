# aws-cfn-ec2-windows

AWS CloudFormation template for Windows Server on Amazon EC2

## Overview

This repository provides a CloudFormation template that launches a Windows Server 2022 EC2 instance.
The template creates a complete networking stack (VPC, public subnet, internet gateway, egress-only internet gateway, route table) and an EC2 instance accessible via [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) — no RDP or key pair required.
It installs [winget](https://github.com/microsoft/winget-cli) (Windows Package Manager) and uses it to provision the several applications.

## Template

| File          | Description                      |
| ------------- | -------------------------------- |
| `ec2.cfn.yml` | Windows Server 2022 EC2 instance |

## Parameters

| Parameter            | Default                    | Description                                           |
| -------------------- | -------------------------- | ----------------------------------------------------- |
| `SystemName`         | `ws`                       | System name used for resource naming and tagging      |
| `EnvType`            | `dev`                      | Environment type used for resource naming and tagging |
| `Ec2VpcCidrBlock`    | `10.0.0.0/16`              | IPv4 CIDR block for the VPC                           |
| `Ec2SubnetCidrBlock` | `10.0.0.0/24`              | IPv4 CIDR block for the public subnet                 |
| `Ec2InstanceType`    | `t3.large`                 | EC2 instance type                                     |
| `Ec2WindowsAmiId`    | Latest Windows Server 2022 | SSM parameter for the AMI ID                          |
| `Ec2VolumeSize`      | `100`                      | Root EBS volume size in GiB                           |
| `Ec2VolumeType`      | `gp3`                      | Root EBS volume type                                  |

## Usage

```bash
aws cloudformation deploy \
  --template-file ec2.cfn.yml \
  --stack-name ws-dev-ec2 \
  --capabilities CAPABILITY_NAMED_IAM
```

After the stack is created, start a Session Manager session using the AWS CLI:

```bash
aws ssm start-session --target <Ec2InstanceId>
```
