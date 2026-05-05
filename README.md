# Cloud Security Portfolio

This portfolio documents my transition into cloud security threat research through hands-on AWS labs focused on IAM, workload identity, attack-path modelling, and defensive control design.

## Focus Areas

- AWS IAM policy analysis
- Terraform-managed lab infrastructure
- AWS CLI testing
- S3 data access and exfiltration risk
- EC2 workload identity and instance profiles
- IMDSv2 credential exposure concepts
- `iam:PassRole` and `ec2:RunInstances` privilege-escalation modelling
- Least-privilege redesign and guardrail development
- Security documentation and threat modelling

## Labs

### AWS IAM Attack Path Lab

Hands-on lab exploring IAM misconfiguration, credential exposure, S3 access, and least-privilege redesign.

Key concepts:
- Terraform state credential exposure
- Authentication vs authorization
- S3 bucket discovery and object access
- IAM policy scoping
- Blast-radius reduction

Repository:
[aws-iam-attack-path-lab](https://github.com/patrick-cloud-security/aws-iam-attack-path-lab)

## In Progress

### EC2 Workload Identity / IMDSv2 Lab

Planned lab exploring:
- IAM roles and instance profiles
- Temporary STS credentials
- IMDSv2 credential retrieval concepts
- Local replay of compromised workload credentials
- `iam:PassRole` + `ec2:RunInstances` privilege escalation
- Defensive guardrails using explicit denies, role scoping, permission boundaries, and logging

## Background

I am a published scientist and qualified teacher transitioning into cloud security threat research. My background combines scientific research, technical writing, international education experience, and hands-on AWS security labs.

## Security Note

All labs are performed in authorized personal lab environments only. No production systems, customer data, or unauthorized targets are used.
