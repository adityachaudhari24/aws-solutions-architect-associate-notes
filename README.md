# AWS Solutions Architect Associate Notes
Sharing my learnings and notes while I prepare for the exam in my 5 weeks plan.


## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#identity-access-management-iam">Identity Access Management (IAM)</a>

## Week 1 Learnings

## Introduction
TODO - 5 weeks plan 

## Identity Access Management (IAM)
#### Q: How Users, Groups, Roles, and Policies explained ?
<details>
Users: Assigned credentials (username/password, access keys). <br>
Groups: Users are added to groups (e.g., "Developers"), and policies are attached to groups. <br>
Roles: Assigned to AWS services (like EC2) or users temporarily. Policies are attached to roles. <Br>
Policies: JSON documents that define permissions. They are not standalone identities – they must be attached to a User, Group, or Role. <br>

IAM roles provide temporary credentials (access key, secret key, and token) when assumed, replacing long-term access keys. <br>
<span class="highlighted-text"> Role is like hat, which is being wear by user or service to perform certain tasks.</span> <br>
These credentials are short-lived and used by users, apps, or AWS services to perform tasks securely. <br>
Example reference link : https://www.youtube.com/watch?v=miij_0HkBws <br>
</details>

#### Q: What are policies and what are the types of policies ?
<details>
Access in AWS is managed by creating policies and attaching them to IAM identities (users, groups of users, or roles) or AWS resources. 
A policy is an object in AWS that, when associated with an identity or resource, defines their permissions. AWS evaluates these policies when an IAM principal (user or role) makes a request regardless if it is from the AWS Management Console, the AWS CLI, or the AWS API. <br>

simple policy example below :

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::example_bucket",
            "Condition": {
                "StringEquals": {
                    "s3:prefix": "home/"
                }
            }
        }
    ]
}
```
Policies have below elements :
- version - Version of the policy language
- id - Policy ID **(optional)**
- statement - List of statements
  - sid - Statement ID **(optional)**
  - Effect - whether the statement allows or denies access
  - Principal - account/user/role to which policy is attached to
  - Action - List of actions that are allowed or denied
  - Resource - List of resources to which the action is applied to
  - Condition - Condition when the policy is in effect **(optional)**

#### There are 6 types of policies all of these polices are evaluated before a request is either allowed or denied.
`1. Identity-Based Policies `<br>
**What they are:** Personal rulebooks attached to IAM identities. <br>
**Example 1:** The AWS Managed Policy AmazonS3FullAccess—allows an IAM user (e.g., "Alice") full control over S3. <br>
**Example 2:** The AWS Managed Policy AmazonEC2ReadOnlyAccess—assigned to a group (e.g., "Developers") so all members can view EC2 resources without modification. <br>

`2. Resource-Based Policies `<br>
**What they are:** Policies attached directly to AWS resources, like locks on a door. <br>
**Example 1:** An S3 bucket policy that grants another AWS account (say, a backup account) permission to read objects—this policy is written in JSON and attached to the bucket. <br>
**Example 2:** A Lambda function resource policy that permits Amazon API Gateway to invoke the function—ensuring only designated services can trigger it. <br>


`3. Permissions Boundaries`
**What they are:** Hard ceilings that limit the maximum permissions an identity can have. <br>
**Example 1:** A custom boundary policy (e.g., S3ReadOnlyBoundary) attached to a role, ensuring that even if other policies grant extra privileges, only S3 read actions (like s3:GetObject) are allowed.
**Example 2:** A custom boundary (e.g., EC2StartStopBoundary) attached to an IAM user, which restricts actions to only starting and stopping EC2 instances—even if additional policies provide broader access. <br>

`4. Session Policies`
**What they are:** Temporary filters applied during a session when a role is assumed. <br>
**Example 1:** When an IAM user assumes a role via AWS STS (using aws sts assume-role), a session policy may restrict actions to only allow s3:ListBucket and s3:GetObject on a specific bucket. <br>
**Example 2:** During a temporary session for managing databases, a session policy can limit permissions to read-only operations on RDS (for example, allowing only rds:DescribeDBInstances). <br>

`5. Service Control Policies (SCPs)`
**What they are:** Organization-wide policies that set the outer limits for what member accounts can do. <br>
**Example 1:** An SCP applied to an Organizational Unit (OU) that denies s3:DeleteBucket across all accounts—preventing accidental deletion of critical storage. <br>
**Example 2:** An SCP that blocks IAM changes (such as denying iam:CreatePolicy, iam:PutUserPolicy, and iam:DeletePolicy) to maintain a strict security baseline across the organization.   <br>

`6. Access Control Lists (ACLs)`
**What they are:** Simple permission lists used on specific resources for basic access control. <br>
**Example 1:** An S3 object ACL set to public-read on an image file—allowing anyone to view it. <br>
**Example 2:** A VPC network ACL configured to allow inbound HTTP (port 80) and HTTPS (port 443) traffic while denying other ports—acting as a basic firewall. <br>

Below is the flow in which these policies are evaluated before a request is either allowed or denied.
![img.png](images/6iampolicytypes.png). <br>
</details>

#### Q: Two types of authorization model RBAC and ABAC what they are and difference between them?
IMP - Recognize when to use ABAC (tags, scaling) vs. RBAC (static roles). Always check for aws:ResourceTag or aws:PrincipalTag in policies.
<details>
Both are IAM strategies to manage permissions, but they work differently. Let’s break them down with simple examples and exam-focused insights.

### 1. RBAC (Role-Based Access Control)

**Definition:** Assign permissions based on predefined roles (e.g., "Admin," "Developer").

**How It Works:**

- Create IAM roles with policies that specify exact AWS resources (e.g., S3 buckets, EC2 instances).
- Users/groups are assigned these roles.

**Example:**

**Scenario:** A company has two S3 buckets: `projectx-data` and `projecty-data`.

**Role:** `ProjectX-Developer`

**Policy:** Allows read/write access only to `projectx-data`.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": ["s3:*"],
            "Resource": [
                "arn:aws:s3:::projectx-data",
                "arn:aws:s3:::projectx-data/*"
            ]
        }
    ]
}
```
Exam Tip:
RBAC is ideal for static environments where resources don’t change often.
If a new bucket projectz-data is added, you must update the policy to include it.

### 2. ABAC (Attribute-Based Access Control)

**Definition:** Permissions are based on tags (attributes) attached to users/resources.

**How It Works:**

- Define policies that use conditions like `aws:ResourceTag` or `aws:PrincipalTag`.
- Access is granted if tags match.

**Example:**

**Scenario:** Developers should only access EC2 instances tagged with their team’s name (e.g., Team=Frontend).

- **User Tag:** `Team=Frontend` (assigned to the IAM user).
- **Resource Tag:** `Team=Frontend` (assigned to EC2 instances).

**ABAC Policy:**

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "ec2:*",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "aws:ResourceTag/Team": "${aws:PrincipalTag/Team}"
                }
            }
        }
    ]
}
```
**Exam Tip:**  
ABAC is scalable for dynamic environments (e.g., auto-scaling EC2 instances). No policy updates needed when new resources are created—just apply the correct tags.

**Key Differences for the Exam:**

| **RBAC** | **ABAC** |
|----------|----------|
| Permissions tied to roles with explicit resource ARNs. | Permissions tied to tags on users/resources. |
| Best for fixed, predictable resources. | Best for dynamic, rapidly changing resources. |
| Requires policy updates for new resources. | Automatically applies to new tagged resources. |


</details>


#### Q: Test
<details>
<summary>Click to expand</summary>
</details>


