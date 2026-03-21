# Repository Guidelines

## Project Structure & Module Organization

This repository is intentionally small. The main deliverable is [`ec2.cfn.yml`](/Users/dceoy/util/aws-cfn-ec2-windows/ec2.cfn.yml), a CloudFormation template that provisions a Windows Server EC2 stack plus supporting VPC, subnet, routing, IAM, and Session Manager access. [`README.md`](/Users/dceoy/util/aws-cfn-ec2-windows/README.md) documents deployment and parameters. CI automation lives under [`.github/workflows`](/Users/dceoy/util/aws-cfn-ec2-windows/.github/workflows), and local helper skills are stored in [`.agents/skills`](/Users/dceoy/util/aws-cfn-ec2-windows/.agents/skills). There is no separate `src/` or `tests/` tree.

## Build, Test, and Development Commands

Use the AWS CLI to deploy or update the stack:

```bash
aws cloudformation deploy --template-file ec2.cfn.yml --stack-name ws-dev-ec2 --capabilities CAPABILITY_NAMED_IAM
```

Run the local QA workflow before opening a PR:

```bash
bash .agents/skills/local-qa/scripts/qa.sh
```

That script formats Markdown with `prettier`, fixes workflow issues with `zizmor`, and runs `yamllint`, `cfn-lint`, `actionlint`, `checkov`, and `trivy`.

## Coding Style & Naming Conventions

Write YAML with two-space indentation and keep keys aligned for readability. Preserve CloudFormation logical ID patterns already used in the template, such as `Ec2Vpc`, `Ec2Subnet`, and `Ec2SecurityGroup`. Parameters use PascalCase with service-prefixed names like `Ec2VolumeSize`. Keep resource names and tags derived from `SystemName` and `EnvType`, and prefer explicit comments only when suppressing a scanner finding.

## Testing Guidelines

There is no unit test suite; validation is lint- and policy-based. Treat `cfn-lint` as the minimum gate for template changes, and run the full QA script for any update to CloudFormation, workflows, or Markdown. When changing infrastructure behavior, include the exact deploy command or a short validation note in the PR description.

## Commit & Pull Request Guidelines

Recent commits use short, imperative subjects such as `Add CI/CD workflows...` and `Harden EC2 security...`. Follow that style, keep the subject line focused on one change, and avoid vague messages like `fix stuff`. PRs should summarize the infrastructure impact, call out parameter or security changes, link related issues, and include relevant command output when QA or deployment behavior changed.

## Security & Configuration Tips

Do not hardcode secrets, key pairs, or open management ports. This stack is designed for Session Manager access instead of RDP. Keep security exceptions narrowly scoped and document any `trivy`, `checkov`, or `zizmor` ignore comments in the PR.
