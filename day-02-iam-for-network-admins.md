# Day 2 — IAM for Network Administrators

## Concept

Identity and Access Management (IAM) controls **who** can do **what** in AWS. As a network admin, you'll need to create users/roles for network services, manage access keys for the CLI, and understand how permissions affect network resources.

## Core Components

### 1. User

A **User** represents a person or application.

Examples:
```
sohail
netadmin
developer1
```

Create:
```bash
aws iam create-user --user-name netadmin
```

A user has **no permissions by default**. You must attach a policy to grant access.

### 2. Policy

A **Policy** is a JSON document that defines permissions.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances"
      ],
      "Resource": "*"
    }
  ]
}
```

This policy says: **Allow viewing EC2 instances.**

A policy by itself **does nothing until attached** somewhere (user, group, or role).

### 3. Group

A **Group** is a collection of users.

Examples:
```
NetworkAdmins
Developers
Auditors
```

Create:
```bash
aws iam create-group --group-name NetworkAdmins
```

Add user:
```bash
aws iam add-user-to-group \
  --user-name netadmin \
  --group-name NetworkAdmins
```

Attach policy:
```bash
aws iam attach-group-policy \
  --group-name NetworkAdmins \
  --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
```

Result:
```
NetworkAdmins
 ├─ netadmin
 ├─ sohail
 └─ admin2

Policy:
EC2FullAccess
```

All members **inherit** the same permissions.

### 4. Role

A **Role** is not a user. A role is something that can be **assumed**.

Used by:
- EC2 instances
- Lambda functions
- ECS / EKS services
- Cross-account access
- Temporary users

Example: `EC2MonitoringRole`

Trust policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Meaning: **EC2 instances may use this role.**

### 5. Attach Policy to Role

Create role:
```bash
aws iam create-role \
  --role-name EC2MonitoringRole \
  --assume-role-policy-document file://trust-policy.json
```

Attach policy:
```bash
aws iam attach-role-policy \
  --role-name EC2MonitoringRole \
  --policy-arn arn:aws:iam::aws:policy/CloudWatchReadOnlyAccess
```

Example:
```
Role:    EC2MonitoringRole
Policy:  CloudWatchReadOnlyAccess
```

Now EC2 instances using this role **can access CloudWatch**.

## Typical Flow for Human Users

**Option A (Best Practice):**
```
User → Group → Policy
```

Example:
```
User:    netadmin
Group:   NetworkAdmins
Policy:  VPCAdminPolicy
```

Diagram:
```
netadmin
    ↓
NetworkAdmins
    ↓
VPCAdminPolicy
```

## Typical Flow for EC2

**Option B:**
```
EC2 Instance → Role → Policy
```

Diagram:
```
EC2 Server
     ↓
EC2MonitoringRole
     ↓
CloudWatchReadOnlyAccess
```

**No access keys required.**

## Complete Real Example

**Human Administrator:**
```
User:    sohail
Group:   NetworkAdmins
Policy:  Allow VPC + EC2 + Route53
```

```
sohail
   ↓
NetworkAdmins
   ↓
Permissions
```

**Application on EC2:**
```
Web Server → Role → S3 Read Access
```

```
EC2 Instance
      ↓
WebAppRole
      ↓
AmazonS3ReadOnlyAccess
```

The application automatically gets **temporary credentials from STS**. No `aws configure` and no access keys stored on the server.

## Rule of Thumb

- **User** = person or application identity
- **Group** = collection of users
- **Policy** = permissions document
- **Role** = temporary identity that users/services can assume
- **Attach policies to groups and roles** whenever possible
- **Avoid attaching policies directly to users** unless there is a specific reason

## IAM Policy Structure

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets",
        "ec2:DescribeRouteTables",
        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress"
      ],
      "Resource": "*"
    }
  ]
}
```

## Sample Network Admin Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "vpc:*",
        "elasticloadbalancing:*",
        "route53:*",
        "directconnect:*",
        "cloudfront:*",
        "shield:*",
        "wafv2:*",
        "network-firewall:*"
      ],
      "Resource": "*"
    }
  ]
}
```

## Least Privilege Principle

**Only grant the permissions needed** — not full admin access.

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeVpcs",
    "ec2:DescribeSubnets",
    "ec2:DescribeRouteTables"
  ],
  "Resource": "*"
}
```

## Managed vs Inline Policies

| | Managed Policies | Inline Policies |
|---|---|---|
| **Created by** | AWS or you | You, embedded directly |
| **Attached to** | Multiple users, groups, or roles | Exactly one entity |
| **Reusability** | Reuse across many entities | Not reusable |
| **ARN** | Has its own ARN | No ARN |
| **Use case** | Standard permissions (e.g. `AmazonVPCReadOnlyAccess`) | Entity-specific, custom logic |
| **List commands** | `list-attached-user-policies` | `list-user-policies` |

**Rule of thumb:** Prefer managed policies for shared permissions. Use inline policies for one-off, entity-specific rules that should never be reused.

## CLI Setup for Network Admin

```bash
# Configure CLI with IAM user access keys
aws configure

# Verify who you are
aws sts get-caller-identity

# Check effective permissions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/network-admin \
  --action-names ec2:CreateVpc ec2:DeleteVpc
```

## Key CLI Commands

### Users

```bash
# List all IAM users
aws iam list-users

# Create a user
aws iam create-user --user-name netadmin

# Get user details
aws iam get-user --user-name netadmin

# Create access keys for CLI usage
aws iam create-access-key --user-name netadmin

# List access keys for a user
aws iam list-access-keys --user-name netadmin

# Get when an access key was last used
aws iam get-access-key-last-used --access-key-id AKIAIOSFODNN7EXAMPLE

# Create console password for a user
aws iam create-login-profile \
  --user-name netadmin \
  --password P@ssw0rd123! \
  --password-reset-required

# Update console password
aws iam update-login-profile \
  --user-name netadmin \
  --password NewP@ssw0rd456!

# Delete console password
aws iam delete-login-profile --user-name netadmin

# Delete an access key
aws iam delete-access-key \
  --user-name netadmin \
  --access-key-id AKIAIOSFODNN7EXAMPLE

# Delete a user (must remove from groups and detach policies first)
aws iam delete-user --user-name netadmin
```

### Groups

```bash
# List all groups
aws iam list-groups

# Create a group
aws iam create-group --group-name NetworkAdmins

# Add user to group
aws iam add-user-to-group --group-name NetworkAdmins --user-name netadmin

# Remove user from group
aws iam remove-user-from-group --group-name NetworkAdmins --user-name netadmin

# List users in a group
aws iam get-group --group-name NetworkAdmins

# Delete a group (must be empty first)
aws iam delete-group --group-name NetworkAdmins
```

### Policies

```bash
# List all managed policies (AWS + customer)
aws iam list-policies

# List only customer-managed policies
aws iam list-policies --scope Local

# List only AWS-managed policies
aws iam list-policies --scope AWS

# Get policy details
aws iam get-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess

# Get the actual policy document (JSON)
aws iam get-policy-version \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess \
  --version-id v1

# See which users/groups/roles are using a policy
aws iam list-entities-for-policy \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess

# Create a custom managed policy
aws iam create-policy \
  --policy-name NetworkReadOnly \
  --policy-document file://network-readonly-policy.json

# Delete a customer-managed policy (must not be attached anywhere)
aws iam delete-policy \
  --policy-arn arn:aws:iam::123456789012:policy/NetworkReadOnly
```

### Attaching / Detaching Policies

```bash
# Attach managed policy to group
aws iam attach-group-policy \
  --group-name NetworkAdmins \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess

# List policies attached to a group
aws iam list-attached-group-policies --group-name NetworkAdmins

# Detach policy from group
aws iam detach-group-policy \
  --group-name NetworkAdmins \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess

# Attach managed policy to user
aws iam attach-user-policy \
  --user-name netadmin \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess

# List policies attached to a user
aws iam list-attached-user-policies --user-name netadmin

# Detach policy from user
aws iam detach-user-policy \
  --user-name netadmin \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess

# Attach managed policy to role
aws iam attach-role-policy \
  --role-name NetworkMonitorRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess

# List policies attached to a role
aws iam list-attached-role-policies --role-name NetworkMonitorRole

# Detach policy from role
aws iam detach-role-policy \
  --role-name NetworkMonitorRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess
```

### Inline Policies

```bash
# Create inline policy for a group
aws iam put-group-policy \
  --group-name NetworkAdmins \
  --policy-name NetworkFullAccess \
  --policy-document file://network-admin-policy.json

# List inline policies for a group
aws iam list-group-policies --group-name NetworkAdmins

# Get inline policy document for a group
aws iam get-group-policy \
  --group-name NetworkAdmins \
  --policy-name NetworkFullAccess

# Delete inline policy from a group
aws iam delete-group-policy \
  --group-name NetworkAdmins \
  --policy-name NetworkFullAccess

# Create inline policy for a user
aws iam put-user-policy \
  --user-name netadmin \
  --policy-name VpcReadOnly \
  --policy-document file://vpc-readonly.json

# List inline policies for a user
aws iam list-user-policies --user-name netadmin

# Get inline policy document for a user
aws iam get-user-policy \
  --user-name netadmin \
  --policy-name VpcReadOnly

# Delete inline policy from a user
aws iam delete-user-policy \
  --user-name netadmin \
  --policy-name VpcReadOnly

# Create inline policy for a role
aws iam put-role-policy \
  --role-name NetworkMonitorRole \
  --policy-name CloudWatchReadOnly \
  --policy-document file://cloudwatch-readonly.json

# List inline policies for a role
aws iam list-role-policies --role-name NetworkMonitorRole

# Get inline policy document for a role
aws iam get-role-policy \
  --role-name NetworkMonitorRole \
  --policy-name CloudWatchReadOnly

# Delete inline policy from a role
aws iam delete-role-policy \
  --role-name NetworkMonitorRole \
  --policy-name CloudWatchReadOnly
```

### Roles

```bash
# List all roles
aws iam list-roles

# Create role for EC2
aws iam create-role \
  --role-name NetworkMonitorRole \
  --assume-role-policy-document file://trust-policy.json

# Get role details
aws iam get-role --role-name NetworkMonitorRole

# Update role trust policy
aws iam update-assume-role-policy \
  --role-name NetworkMonitorRole \
  --policy-document file://updated-trust-policy.json

# Delete a role (must detach all policies first)
aws iam delete-role --role-name NetworkMonitorRole
```

### Simulate Permissions

```bash
# Check if a user can perform specific actions
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:user/network-admin \
  --action-names ec2:CreateVpc ec2:DeleteVpc

# Check with resource constraints
aws iam simulate-principal-policy \
  --policy-source-arn arn:aws:iam::123456789012:role/NetworkMonitorRole \
  --action-names ec2:Describe* \
  --resource-arns arn:aws:ec2:us-east-1:123456789012:*
```

### Account-Level Commands

```bash
# See account alias
aws iam list-account-aliases

# List MFA devices
aws iam list-mfa-devices --user-name netadmin

# Enable MFA device
aws iam enable-mfa-device \
  --user-name netadmin \
  --serial-number arn:aws:iam::123456789012:mfa/netadmin \
  --authentication-code1 123456 \
  --authentication-code2 789012

# List server certificates
aws iam list-server-certificates
```

## Console

**IAM → Users** — create and manage users
**IAM → Roles** — create and assign roles
**IAM → Policies** — create custom policies
**IAM → Access Analyzer** — analyze public/ cross-account access

## Gotchas

- IAM is **global** — not region-specific
- Changes can take **seconds to minutes** to propagate
- Root user has **full unrestricted access** — enable MFA, use it only for account-level tasks
- **Access keys are displayed once** — save them immediately
- Delete unused access keys regularly
- Use **IAM Access Analyzer** to detect unintended public access
- Roles are preferred over long-term access keys

## Design Notes

- Create separate IAM users for each network admin
- Use groups for permission management
- Use roles for EC2 instances instead of storing keys on them
- Enable CloudTrail to audit all IAM actions
- Enable MFA for all users
- Rotate access keys every 90 days
