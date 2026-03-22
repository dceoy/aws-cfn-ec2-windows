# aws-cfn-ec2-minimal

AWS CloudFormation templates for Windows Server on Amazon EC2

## Overview

This repository provides two CloudFormation templates:

- `vpc.cfn.yml` creates the shared networking and security resources.
- `ec2.cfn.yml` launches the Windows Server 2025 EC2 instance and its IAM resources into that existing infrastructure.

The instance is accessible via [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) — no RDP or key pair required.
It installs [winget](https://github.com/microsoft/winget-cli) (Windows Package Manager) and uses it to provision several applications.

## Template

| File          | Description                                                     |
| ------------- | --------------------------------------------------------------- |
| `vpc.cfn.yml` | VPC, subnet, routing, network ACL, and security group resources |
| `ec2.cfn.yml` | Windows Server 2025 EC2 instance plus IAM role and profile      |

## Parameters for `vpc.cfn.yml`

| Parameter            | Default       | Description                               |
| -------------------- | ------------- | ----------------------------------------- |
| `SystemName`         | `fte`         | System name used for naming and tags      |
| `EnvType`            | `dev`         | Environment type used for naming and tags |
| `Ec2VpcCidrBlock`    | `10.0.0.0/16` | IPv4 CIDR block for the VPC               |
| `Ec2SubnetCidrBlock` | `10.0.0.0/24` | IPv4 CIDR block for the public subnet     |

## Parameters for `ec2.cfn.yml`

| Parameter            | Default                    | Description                                                       |
| -------------------- | -------------------------- | ----------------------------------------------------------------- |
| `SystemName`         | `fte`                      | System name used for naming and tags                              |
| `EnvType`            | `dev`                      | Environment type used for naming and tags                         |
| `Ec2SubnetId`        | none                       | Existing subnet ID from `vpc.cfn.yml`                             |
| `Ec2SecurityGroupId` | none                       | Existing security group ID from `vpc.cfn.yml`                     |
| `Ec2InstanceType`    | `m7i-flex.large`           | EC2 instance type                                                 |
| `Ec2WindowsAmiId`    | Latest Windows Server 2025 | SSM parameter for the AMI ID                                      |
| `Ec2VolumeSize`      | `64`                       | Root EBS volume size in GiB                                       |
| `Ec2VolumeType`      | `gp3`                      | Root EBS volume type                                              |
| `Ec2UserData`        | `''`                       | Optional full UserData script that replaces the default bootstrap |

## Usage

Deploy the support stack first:

```bash
aws cloudformation deploy \
  --template-file vpc.cfn.yml \
  --stack-name fte-dev-ec2-support
```

Then fetch the required output values and deploy the EC2 stack:

```bash
SUPPORT_STACK=fte-dev-ec2-support

EC2_SUBNET_ID=$(aws cloudformation describe-stacks \
  --stack-name "$SUPPORT_STACK" \
  --query "Stacks[0].Outputs[?OutputKey=='Ec2SubnetId'].OutputValue" \
  --output text)

EC2_SECURITY_GROUP_ID=$(aws cloudformation describe-stacks \
  --stack-name "$SUPPORT_STACK" \
  --query "Stacks[0].Outputs[?OutputKey=='Ec2SecurityGroupId'].OutputValue" \
  --output text)

aws cloudformation deploy \
  --template-file ec2.cfn.yml \
  --stack-name fte-dev-ec2 \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    Ec2SubnetId="$EC2_SUBNET_ID" \
    Ec2SecurityGroupId="$EC2_SECURITY_GROUP_ID"
```

Use matching `SystemName` and `EnvType` values across both stacks whenever you override the defaults.

If you previously deployed the intermediate split where IAM lived in the networking stack, plan a staged migration or stack recreation before adopting this layout because the IAM role and instance profile now belong to `ec2.cfn.yml`.

Set `Ec2UserData` only when you want to replace the built-in bootstrap script. Custom values should provide a complete Windows UserData script, including the `<powershell>...</powershell>` wrapper and a `cfn-signal.exe` call so the stack can satisfy the instance `CreationPolicy`.

After the EC2 stack is created, start a Session Manager session using the AWS CLI:

```bash
aws ssm start-session --target <Ec2InstanceId>
```
