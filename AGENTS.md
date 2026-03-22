# Repository Guidelines

## Overview

The main deliverables are [`vpc.cfn.yml`](/Users/dceoy/util/aws-cfn-ec2-minimal/vpc.cfn.yml), [`iam.cfn.yml`](/Users/dceoy/util/aws-cfn-ec2-minimal/iam.cfn.yml), [`ec2-windows.cfn.yml`](/Users/dceoy/util/aws-cfn-ec2-minimal/ec2-windows.cfn.yml), and [`ec2-ubuntu.cfn.yml`](/Users/dceoy/util/aws-cfn-ec2-minimal/ec2-ubuntu.cfn.yml). The VPC template provisions the VPC, subnet, routing, and security prerequisites, the IAM template creates the EC2 role and instance profile, and the EC2 templates launch either a Windows Server or Ubuntu instance with Session Manager access using those existing resources.

## Deployment Instructions

Use the AWS CLI to deploy or update the stacks:

```bash
aws cloudformation deploy --template-file vpc.cfn.yml --stack-name min-dev-ec2-vpc
aws cloudformation deploy --template-file iam.cfn.yml --stack-name min-dev-ec2-iam --capabilities CAPABILITY_NAMED_IAM
aws cloudformation deploy --template-file ec2-windows.cfn.yml --stack-name min-dev-ec2 --parameter-overrides Ec2SubnetId=<subnet-id> Ec2SecurityGroupId=<security-group-id> Ec2IamInstanceProfileName=<instance-profile-name>
aws cloudformation deploy --template-file ec2-ubuntu.cfn.yml --stack-name min-dev-ec2 --parameter-overrides Ec2SubnetId=<subnet-id> Ec2SecurityGroupId=<security-group-id> Ec2IamInstanceProfileName=<instance-profile-name>
```

Both EC2 templates reuse the same shared imports, but they now publish OS-specific EC2 output export names. If you deploy both for the same `${SystemName}-${EnvType}`, use different CloudFormation stack names.

## Code Quality & Validation

**IMPORTANT**: Run the following on each change before committing.

- **format and lint**: Use the `local-qa` skill.

## Coding Style & Naming Conventions

- Write YAML with two-space indentation and keep keys aligned for readability.
- Preserve CloudFormation logical ID patterns already used in the template, such as `Ec2Vpc`, `Ec2Subnet`, and `Ec2SecurityGroup`.
- Parameters use PascalCase with service-prefixed names like `Ec2VolumeSize`.
- Keep resource names and tags derived from `SystemName` and `EnvType`, and prefer explicit comments only when suppressing a scanner finding.

## Commit & Pull Request Guidelines

- Run QA checks using `local-qa` skill before committing or creating a PR.
- Branch names use appropriate prefixes on creation (e.g., `feature/...`, `bugfix/...`, `refactor/...`, `docs/...`, `chore/...`).
- When instructed to create a PR, create it as a draft with appropriate labels by default.

## Code Design Principles

Always prefer the simplest design that works.

- **KISS**: Choose straightforward solutions and avoid unnecessary abstraction.
- **DRY**: Remove duplication when it improves clarity and maintainability.
- **YAGNI**: Do not add features, hooks, or flexibility until they are needed.
- **SOLID/Clean Code**: Apply these as tools, only when they keep the design simpler and easier to change.

## Development Methodology

Keep delivery incremental, test-backed, and easy to review.

- Make small, safe, reversible changes.
- Prefer `Red -> Green -> Refactor`.
- Do not mix feature work and refactoring in the same commit.
- Refactor when it improves clarity or removes real duplication (Rule of Three).
- Keep tests fast, focused, and self-validating.
