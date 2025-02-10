<style>
  .highlighted-text {
    color: #333333; /* Dark gray text color */
    background-color: #ffff99; /* Light yellow background */
    padding: 2px;
    border-radius: 3px;
  }
</style>

# AWS Solutions Architect Associate Notes
Sharing my learnings and notes while I prepare for the exam in my 5 weeks plan.


## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#identity-access-management-iam">Identity Access Management (IAM)</a>

## Week 1 Learnings

## Introduction
TODO - 5 weeks plan 

## Identity Access Management (IAM)
#### Q: What is the difference between policy and role in AWS IAM?
A policy is a document that specifies what actions are allowed or denied on AWS resources—it’s like a rulebook. <br>
A role is an IAM identity that holds one or more policies and can be temporarily assumed by a user or service to gain those permissions.<br>
**Think of it like this:** <br>
Policy = Rules: They define what you can or cannot do. <br>
Role = Job Position: It’s like a temporary job title that comes with a set of rules (policies) that the person or service must follow. <br>

#### Q: How Users, Groups, Roles, and Policies align with each other ?
Users: Assigned credentials (username/password, access keys). <br>
Groups: Users are added to groups (e.g., "Developers"), and policies are attached to groups. <br>
Roles: Assigned to AWS services (like EC2) or users temporarily. Policies are attached to roles. <Br>
Policies: JSON documents that define permissions. They are not standalone identities – they must be attached to a User, Group, or Role. <br>

IAM roles provide temporary credentials (access key, secret key, and token) when assumed, replacing long-term access keys. <br>
<span class="highlighted-text"> Role is like hat, which is being wear by user or service to perform certain tasks.</span> <br>
These credentials are short-lived and used by users, apps, or AWS services to perform tasks securely. <br>
Example reference link : https://www.youtube.com/watch?v=miij_0HkBws <br>

#### Q: What are policies and what are the types of policies ?
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




