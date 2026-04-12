# aws-cfn-ec2-minimal

AWS CloudFormation templates for Windows Server and Linux on Amazon EC2

## Overview

This repository provides four CloudFormation templates:

- `vpc.cfn.yml` creates the shared networking and security resources.
- `iam.cfn.yml` creates the IAM role and instance profile used by the EC2 instance.
- `ec2-windows.cfn.yml` launches the Windows Server 2025 EC2 instance into that existing infrastructure.
- `ec2-linux.cfn.yml` launches the Amazon ECS-optimized Amazon Linux 2023 x86_64 EC2 instance into that existing infrastructure.

Both EC2 templates import the matching subnet, security group, and instance profile
by default using the shared `${SystemName}-${EnvType}-*` naming convention.

The VPC and IAM stacks export shared `${SystemName}-${EnvType}-*` names, while each
EC2 template publishes OS-specific export names and an OS-specific EC2 `Name` tag
so Windows and Linux resources do not collide.

The EC2 instances are accessible via [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html) with the companion IAM role, so no RDP, SSH, or key pair is required.
Both EC2 templates require IMDSv2, attach encrypted root volumes, associate public IP addresses, and omit `UserData` by default unless you provide `Ec2UserData`.
The VPC template creates a dual-stack public subnet, internet connectivity for IPv4 and IPv6, S3 and DynamoDB gateway endpoints, a network ACL that explicitly denies inbound SSH and RDP, and a security group with no ingress rules and outbound HTTPS only.

## Template

| File                  | Description                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------ |
| `vpc.cfn.yml`         | Dual-stack VPC, public subnet, routing, gateway endpoints, network ACL, and security group |
| `iam.cfn.yml`         | IAM role and instance profile for Systems Manager access                                   |
| `ec2-windows.cfn.yml` | Windows Server 2025 EC2 instance                                                           |
| `ec2-linux.cfn.yml`   | Amazon ECS-optimized Amazon Linux 2023 x86_64 EC2 instance                                 |

## Parameters for `vpc.cfn.yml`

| Parameter            | Default       | Description                               |
| -------------------- | ------------- | ----------------------------------------- |
| `SystemName`         | `min`         | System name used for naming and tags      |
| `EnvType`            | `dev`         | Environment type used for naming and tags |
| `Ec2VpcCidrBlock`    | `10.0.0.0/16` | IPv4 CIDR block for the VPC               |
| `Ec2SubnetCidrBlock` | `10.0.0.0/24` | IPv4 CIDR block for the public subnet     |

## Parameters for `iam.cfn.yml`

| Parameter    | Default | Description                               |
| ------------ | ------- | ----------------------------------------- |
| `SystemName` | `min`   | System name used for naming and tags      |
| `EnvType`    | `dev`   | Environment type used for naming and tags |

## Parameters for `ec2-windows.cfn.yml`

| Parameter                   | Default                    | Description                                                                                       |
| --------------------------- | -------------------------- | ------------------------------------------------------------------------------------------------- |
| `SystemName`                | `min`                      | System name used for naming and tags                                                              |
| `EnvType`                   | `dev`                      | Environment type used for naming and tags                                                         |
| `Ec2SubnetId`               | `''`                       | Optional subnet ID override; empty imports `${SystemName}-${EnvType}-ec2-subnet`                  |
| `Ec2SecurityGroupId`        | `''`                       | Optional security group ID override; empty imports `${SystemName}-${EnvType}-ec2-sg`              |
| `Ec2IamInstanceProfileName` | `''`                       | Optional instance profile override; empty imports `${SystemName}-${EnvType}-iam-instance-profile` |
| `Ec2InstanceType`           | `m7i-flex.large`           | EC2 instance type                                                                                 |
| `Ec2WindowsAmiId`           | Latest Windows Server 2025 | SSM parameter for the AMI ID                                                                      |
| `Ec2VolumeSize`             | `64`                       | Root EBS volume size in GiB                                                                       |
| `Ec2VolumeType`             | `gp3`                      | Root EBS volume type                                                                              |
| `Ec2UserData`               | `''`                       | Optional EC2 UserData script; empty omits `UserData`                                              |

## Parameters for `ec2-linux.cfn.yml`

| Parameter                   | Default                            | Description                                                                                       |
| --------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------- |
| `SystemName`                | `min`                              | System name used for naming and tags                                                              |
| `EnvType`                   | `dev`                              | Environment type used for naming and tags                                                         |
| `Ec2SubnetId`               | `''`                               | Optional subnet ID override; empty imports `${SystemName}-${EnvType}-ec2-subnet`                  |
| `Ec2SecurityGroupId`        | `''`                               | Optional security group ID override; empty imports `${SystemName}-${EnvType}-ec2-sg`              |
| `Ec2IamInstanceProfileName` | `''`                               | Optional instance profile override; empty imports `${SystemName}-${EnvType}-iam-instance-profile` |
| `Ec2InstanceType`           | `m7i-flex.large`                   | EC2 instance type                                                                                 |
| `Ec2LinuxAmiId`             | Latest ECS-optimized AL2023 x86_64 | SSM parameter for the AMI ID                                                                      |
| `Ec2VolumeSize`             | `64`                               | Root EBS volume size in GiB                                                                       |
| `Ec2VolumeType`             | `gp3`                              | Root EBS volume type                                                                              |
| `Ec2UserData`               | `''`                               | Optional EC2 UserData shell script; empty omits `UserData`                                        |

## Usage

Deploy the VPC stack first:

```bash
aws cloudformation deploy --template-file vpc.cfn.yml --stack-name min-dev-ec2-vpc
```

Then deploy the IAM stack:

```bash
aws cloudformation deploy --template-file iam.cfn.yml --stack-name min-dev-ec2-iam --capabilities CAPABILITY_NAMED_IAM
```

Each stack exports its outputs under predictable names:

- `vpc.cfn.yml`
  - `${SystemName}-${EnvType}-ec2-vpc`
  - `${SystemName}-${EnvType}-ec2-subnet`
  - `${SystemName}-${EnvType}-ec2-sg`
  - `${SystemName}-${EnvType}-ec2-igw`
  - `${SystemName}-${EnvType}-ec2-eigw`
  - `${SystemName}-${EnvType}-ec2-s3-vpce`
  - `${SystemName}-${EnvType}-ec2-dynamodb-vpce`
- `iam.cfn.yml`
  - `${SystemName}-${EnvType}-iam-instance-profile`
  - `${SystemName}-${EnvType}-iam-role`
- `ec2-windows.cfn.yml`
  - `${SystemName}-${EnvType}-ec2-windows-instance`
  - `${SystemName}-${EnvType}-ec2-windows-instance-private-ip`
  - `${SystemName}-${EnvType}-ec2-windows-instance-public-ip`
- `ec2-linux.cfn.yml`
  - `${SystemName}-${EnvType}-ec2-linux-instance`
  - `${SystemName}-${EnvType}-ec2-linux-instance-private-ip`
  - `${SystemName}-${EnvType}-ec2-linux-instance-public-ip`

Both EC2 templates still import the same shared VPC and IAM exports, but their EC2
export names and instance `Name` tags are now OS-specific. If you deploy both variants for the same
`${SystemName}-${EnvType}`, use distinct CloudFormation stack names.

With the default parameters, either EC2 template automatically imports these exported values:

- `${SystemName}-${EnvType}-ec2-subnet`
- `${SystemName}-${EnvType}-ec2-sg`
- `${SystemName}-${EnvType}-iam-instance-profile`

Deploy the Windows stack:

```bash
aws cloudformation deploy --template-file ec2-windows.cfn.yml --stack-name min-dev-ec2
```

Deploy the Linux stack:

```bash
aws cloudformation deploy --template-file ec2-linux.cfn.yml --stack-name min-dev-ec2
```

If you need to pin different existing resources, you can still supply explicit overrides:

```bash
aws cloudformation deploy --template-file <ec2-template-file> --stack-name min-dev-ec2 --parameter-overrides Ec2SubnetId=<subnet-id> Ec2SecurityGroupId=<security-group-id> Ec2IamInstanceProfileName=<instance-profile-name>
```

Use matching `SystemName` and `EnvType` values across all three stacks whenever you rely
on the automatic imports. Deploy or update `vpc.cfn.yml` and `iam.cfn.yml` before
your chosen EC2 template so the expected exports exist.

If you previously deployed the older layout where IAM lived in `ec2.cfn.yml`, plan a staged migration or stack recreation before adopting this split because the IAM role and instance profile now belong to `iam.cfn.yml`.

Set `Ec2UserData` only when you want CloudFormation to pass custom instance initialization commands:

- For `ec2-windows.cfn.yml`, provide a complete Windows UserData payload in the format you want the instance to receive.
- For `ec2-linux.cfn.yml`, provide the shell script content you want CloudFormation to base64-encode and pass as `UserData`.

Neither EC2 template defines a `CreationPolicy` or appends a `cfn-signal` step, so stack completion does not wait for `UserData` execution to finish.

After the EC2 stack is created, start a Session Manager session using the AWS CLI:

```bash
aws ssm start-session --target <Ec2InstanceId>
```
