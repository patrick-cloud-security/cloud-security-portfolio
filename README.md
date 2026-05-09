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

### EC2 Workload Identity / IMDSv2 Credential Replay Lab

Hands-on AWS lab demonstrating how an EC2 instance receives temporary STS credentials through IMDSv2, how those credentials can be replayed outside the instance, and how least-privilege IAM design limits blast radius.

Repository: [ec2-workload-identity-imdsv2-lab](https://github.com/patrick-cloud-security/ec2-workload-identity-imdsv2-lab)

Skills demonstrated:
- EC2 IAM roles and instance profiles
- IMDSv2 credential retrieval
- STS temporary credentials
- AWS CLI local replay testing
- Least-privilege IAM policy design
- Cloud attack-path analysis
- Guardrail design

### Cloud Threat / Control Matrix

A research-style matrix mapping AWS attack paths to dangerous permissions, impacts, preventive controls, and detection ideas.

[View matrix](./cloud-threat-control-matrix.md)

### STRIDE Threat Model: EC2 Workload Identity / IMDSv2

Formal STRIDE threat model for the EC2 workload identity lab, covering spoofing, tampering, repudiation, information disclosure, denial of service, and elevation of privilege risks.

[View threat model](./stride-threat-model-ec2-imdsv2.md)

## Background

I am a published scientist and qualified teacher transitioning into cloud security threat research. My background combines scientific research, technical writing, international education experience, and hands-on AWS security labs.

## Security Note

All labs are performed in authorized personal lab environments only. No production systems, customer data, or unauthorized targets are used.
