# STRIDE Threat Model: EC2 Workload Identity / IMDSv2 Lab

## System Overview

This threat model analyses an AWS EC2 workload identity pattern where an EC2 instance receives temporary STS credentials through an IAM role attached via an instance profile.

The lab demonstrates that if a process on the EC2 instance is compromised, an attacker may be able to retrieve temporary role credentials from the Instance Metadata Service v2 (IMDSv2), replay those credentials outside the instance, and call AWS APIs as the attached role.

The main security question is:

> If the EC2 workload credentials are stolen, what can the attacker do?

## Protected Assets

| Asset | Why it matters |
|---|---|
| EC2 IAM role credentials | Temporary STS credentials can be used to call AWS APIs as the workload role. |
| IAM permissions attached to the role | These permissions determine the blast radius after credential theft. |
| S3 bucket names and object data | Bucket discovery and object access can lead to data exposure or exfiltration. |
| AWS account metadata | EC2, IAM, and Secrets Manager enumeration can help attackers map the environment. |
| SSH private key | If exposed, it may allow direct access to instances. |
| Terraform state | State files may reveal infrastructure details, ARNs, outputs, or sensitive values. |

## Trust Boundaries

| Trust Boundary | Description | Security Concern |
|---|---|---|
| Internet / SSH access → EC2 instance | Temporary SSH access was used to access the lab instance. | If SSH access is exposed too broadly, an attacker may gain shell access to the instance. |
| EC2 workload → IMDSv2 | Processes running on the instance can request an IMDSv2 token and query metadata. | If a workload is compromised, the attacker may retrieve temporary role credentials. |
| EC2 role credentials → AWS control plane | Temporary STS credentials are used to call AWS APIs. | Stolen credentials can be replayed outside the instance until they expire. |
| IAM role → AWS resources | The role’s identity policy determines what AWS resources can be accessed. | Over-permissive policies increase blast radius after credential theft. |
| Terraform working directory → GitHub | Terraform code is safe to publish, but state files and keys are not. | Accidentally committing state, keys, or credentials can expose sensitive information. |

## STRIDE Analysis

| STRIDE Category | Threat in This Lab | Example Attack Path | Impact | Controls |
|---|---|---|---|---|
| Spoofing | Attacker uses stolen EC2 role credentials outside the instance. | Retrieve temporary STS credentials from IMDSv2 and configure them as a local AWS CLI profile. | AWS API calls appear as the EC2 workload role, not the attacker’s original identity. | Least-privilege role design, short-lived credentials, CloudTrail monitoring for unusual source IPs. |
| Tampering | Attacker modifies cloud resources if the role has write permissions. | If `s3:PutObject` is allowed broadly, attacker uploads or overwrites objects. | Data integrity loss, poisoned application data, malicious file uploads. | Scope write permissions to specific prefixes, enable versioning, validate uploads, separate write/read roles. |
| Repudiation | It may be hard to prove who used stolen temporary credentials. | Attacker replays EC2 role credentials from another machine. | Activity is attributed to the assumed role session unless logs are reviewed carefully. | CloudTrail, source IP monitoring, session context review, alerting on unusual API calls. |
| Information Disclosure | Attacker reads sensitive cloud data or metadata. | Use stolen credentials to list buckets, read S3 objects, or retrieve secrets if allowed. | Exposure of bucket names, objects, secrets, internal architecture, or credentials. | Least privilege, specific resource ARNs, Secrets Manager scoping, deny unnecessary enumeration. |
| Denial of Service | Attacker abuses allowed permissions to disrupt services or create cost. | Upload large objects, trigger resource-heavy API calls, or consume service quotas. | Cost increase, service degradation, quota exhaustion. | Quotas, billing alerts, scoped permissions, rate limits, monitoring. |
| Elevation of Privilege | Attacker uses role permissions to gain a stronger identity. | Combine `iam:PassRole` with `ec2:RunInstances` to launch compute with a privileged role. | Attacker obtains higher-privilege credentials through the new instance’s metadata service. | Restrict `iam:PassRole`, restrict `ec2:RunInstances`, use permission boundaries/SCPs, alert on PassRole + RunInstances. |

## Highest-Risk Threats

### 1. Stolen workload credentials replayed outside the instance

If an attacker gains shell access or code execution on the EC2 instance, they may be able to request an IMDSv2 token, retrieve temporary STS credentials for the attached IAM role, and replay those credentials from another environment.

In the lab, replayed credentials were able to call:

- `sts:GetCallerIdentity`
- `s3:ListAllMyBuckets`

They were denied:

- `ec2:DescribeInstances`
- `iam:ListRoles`
- `secretsmanager:ListSecrets`

This demonstrates that credential theft is serious, but the blast radius is determined by the IAM permissions attached to the role.

### 2. Over-permissive workload role

If the EC2 role had broad permissions such as `secretsmanager:GetSecretValue` on `Resource="*"`, stolen credentials could be used to retrieve secrets such as database passwords, API keys, or application tokens.

If the role had `iam:PassRole` and `ec2:RunInstances`, an attacker could potentially launch a new EC2 instance with a more privileged role and retrieve those higher-privilege credentials through IMDS.

### 3. Unsafe Terraform/Git handling

Terraform state files can contain sensitive information such as resource identifiers, outputs, and sometimes credentials or secrets. Accidentally committing `terraform.tfstate`, private keys, or temporary credentials to GitHub would create a separate credential exposure path.

## Recommended Controls

| Control | Purpose |
|---|---|
| Require IMDSv2 | Reduces risk from simple metadata access and older SSRF-style attacks. |
| Attach least-privilege workload roles | Limits what stolen EC2 credentials can do. |
| Avoid broad `Resource="*"` permissions | Prevents one compromised workload from accessing unrelated resources. |
| Scope S3 access by bucket and prefix | Separates bucket discovery, object listing, object read, and object write permissions. |
| Scope Secrets Manager access to specific secret ARNs | Prevents stolen workload credentials from retrieving unrelated secrets. |
| Restrict `iam:PassRole` | Prevents attackers from passing privileged roles to new compute. |
| Restrict `ec2:RunInstances` | Reduces compute-based privilege escalation paths. |
| Monitor CloudTrail assumed-role activity | Helps detect role use from unusual locations or unexpected API calls. |
| Alert on denied API calls from workload roles | Denied calls may indicate probing after credential theft. |
| Keep Terraform state out of Git | Prevents accidental exposure of infrastructure details or sensitive values. |
| Remove temporary SSH access after testing | Reduces direct shell access risk to lab instances. |

## Lessons Learned

- EC2 instances should be treated as cloud identities, not just virtual machines.
- If an EC2 workload is compromised, the attached IAM role credentials may also be compromised.
- IMDSv2 improves metadata protection, but it does not remove the need for least-privilege IAM.
- Temporary STS credentials can be replayed outside the instance until they expire.
- The impact of credential theft depends on the permissions attached to the role.
- Formal threat modelling helps connect technical attack paths to security controls.

- 
