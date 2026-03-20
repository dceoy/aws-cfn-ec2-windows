# aws-cfn-ec2-windows

AWS CloudFormation template for Windows Server on Amazon EC2

## Overview

This repository provides a CloudFormation template that launches a Windows Server 2022 EC2 instance pre-configured for [MetaTrader 5](https://www.metatrader5.com/). The template handles networking (security group with RDP access), IAM (SSM-managed instance role), and automated software installation via a PowerShell UserData script. It installs [winget](https://github.com/microsoft/winget-cli) (Windows Package Manager) and uses it to provision the following tools:

- Windows Terminal
- PowerShell
- Vim
- Visual Studio Code
- PowerToys
- Git
- uv
- Google Chrome

## Template

| File                  | Description                                        |
| --------------------- | -------------------------------------------------- |
| `ec2-windows.cfn.yml` | Windows Server 2022 EC2 instance with MetaTrader 5 |

## Parameters

| Parameter      | Default                    | Description                                |
| -------------- | -------------------------- | ------------------------------------------ |
| `ProjectName`  | `mt5`                      | Project name used for resource tagging     |
| `VpcId`        | _(required)_               | VPC ID where the instance will be launched |
| `SubnetId`     | _(required)_               | Subnet ID for the instance                 |
| `InstanceType` | `t3.large`                 | EC2 instance type                          |
| `WindowsAmiId` | Latest Windows Server 2022 | SSM parameter for the AMI ID               |
| `KeyPairName`  | _(required)_               | EC2 key pair for password decryption       |
| `RdpCidr`      | _(required)_               | CIDR block allowed for RDP access          |
| `VolumeSize`   | `100`                      | Root EBS volume size in GiB                |
| `VolumeType`   | `gp3`                      | Root EBS volume type                       |

## Usage

```bash
aws cloudformation deploy \
  --template-file ec2-windows.cfn.yml \
  --stack-name mt5-windows \
  --parameter-overrides \
    VpcId=vpc-xxxxxxxx \
    SubnetId=subnet-xxxxxxxx \
    KeyPairName=my-key-pair \
    RdpCidr=203.0.113.1/32 \
  --capabilities CAPABILITY_IAM
```

After the stack is created, retrieve the administrator password using the EC2 console or the AWS CLI:

```bash
aws ec2 get-password-data \
  --instance-id <InstanceId> \
  --priv-launch-key my-key-pair.pem
```

Connect to the instance via RDP and MetaTrader 5 will be available on the desktop.
