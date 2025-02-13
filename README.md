# AWS Solutions Architect Associate Notes
Sharing my learnings and notes while I prepare for the exam.


## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#identity-access-management-iam">Identity Access Management (IAM)</a>
3. <a href="#EC2">Elastic Cloud Compute EC2</a>



## Introduction
TODO

## Identity Access Management (IAM)
#### Q: what is Users, Groups, Roles, and Policies? 
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
Policies are JSON documents defining permissions for users (IAM) or resources (S3, Lambda).

simple policy example below : <br>
(Allows listing objects in example_bucket only if the prefix is "home/")

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

#### 6 Policy Types
**Identity-Based Policies**  
Attached to users/groups/roles.  
*Example:* AmazonS3FullAccess policy lets a user (e.g., "Alice") manage S3.

**Resource-Based Policies**  
Attached to resources (S3, Lambda).  
*Example:* S3 bucket policy allowing another account to read objects.

**Permissions Boundaries**  
Set max permissions for a user/role.  
*Example:* Boundary allowing only s3:GetObject, even if other policies grant more.

**Session Policies**  
Temporary permissions for role sessions.  
*Example:* Assume a role with STS, limiting actions to s3:ListBucket for 1 hour.

**Service Control Policies (SCPs)**  
AWS Organizations guardrails.  
*Example:* Block s3:DeleteBucket across all accounts in an OU.

**ACLs**  
Legacy resource access rules.  
*Example:* S3 object ACL set to public-read for open access.

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


#### Q: Test - ignore this for now
<details>
<summary>Click for example and explanation</summary>
This is the summary block 
</details>



## EC2

#### Q: EC2 Placement Groups types and use cases?
1. **Cluster:** Single AZ, same rack (low latency). <br> 
   Use Cases : applications which require low latency and high network throughput. <br>
2. **Partition:** Isolated partitions per rack (e.g., Hadoop/Cassandra). <br>
    Use Cases : Large distributed apps like Hadoop, Cassandra, Kafka <br>
    Scenario : A company is running a large-scale web application with multiple microservices. They need to ensure that the failure of one partition does not affect the others. By using a Partition placement group, they can isolate the instances of each microservice into different partitions, each on separate racks, to achieve fault tolerance and high availability. <br>
3. **Spread:** Separate hardware per instance (critical apps). <br>
    Use Cases : Critical apps that need high availability.
    Scenario : A financial services company is running a critical trading application that requires high availability and fault tolerance. They need to ensure that the failure of a single hardware component does not affect the entire application. By using a Spread placement group, they can place each instance on separate hardware to minimize the risk of simultaneous failures and ensure continuous operation. <br>


**Key Limits:**
- Cluster groups risk AZ failure.
- Spread groups support only 7 instances per AZ (EC2) or 5 (other services like EBS).
- Can’t merge groups or move existing instances into a group.

**Exam Tip:**
- Always prioritize "low latency" → Cluster
- "Fault tolerance" → Spread
- "Large distributed apps" → Partition

#### Q: what is the ENI and what are the types of ENI?
It's a logical networking component in a VPC that represents a virtual network card. It can be attached to an EC2 instance in the same AZ. <br> 
You can create multiple ENIs and attach them to different instances **on the fly**. <br>
<code style="color: orange">Bound to specific AZ, cannot be moved to another AZ.</code> <br>

#### Q. what is attached to the network interface?
**Answer :** Security groups, public IP, Elastic IP, and MAC address. <br>
MAC address means Media Access Control address, which is a unique identifier assigned to network interfaces for communications on the physical network segment. <br>


#### EC2 ad-hoc notes
**ENI (Elastic Network Interface):** This is for only specific AZ cannot be moved to another AZ. <br>

<details> </details>
#### Q. what is attached to the network interface?


<details>
  <summary>#### Q. What is the ENI and what are the types of ENI?</summary>
  It's a logical networking component in a VPC that represents a virtual network card. It can be attached to an EC2 instance in the same AZ. <br>
  You can create multiple ENIs and attach them to different instances **on the fly**. <br>
  <code style="color: orange">Bound to specific AZ, cannot be moved to another AZ.</code> <br>
</details>

