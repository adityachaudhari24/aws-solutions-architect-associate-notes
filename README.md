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





<details>
<summary>Q. What is the ENI and why it's used? (exam point of view)</summary>

**Elastic Network Interface (ENI)** is a virtual network interface that you can attach to an Amazon EC2 instance in a Virtual Private Cloud (VPC). It enables you to create a management network, use network and security appliances, and create dual-homed instances with workloads/roles on distinct subnets.

**what is attached to the network interface?**
** Security groups, public IP, Elastic IP, private IP,and MAC address. <br>
MAC address means Media Access Control address, which is a unique identifier assigned to network interfaces for communications on the physical network segment. <br>

**Why and When It Is Used:**
- **High Availability:** You can attach multiple ENIs to an EC2 instance, each in a different subnet, to ensure redundancy.
- **Network Segmentation:** ENIs allow you to separate traffic types (e.g., management vs. data traffic) by attaching multiple interfaces to a single instance.
- **Failover Scenarios:** ENIs can be moved between instances to quickly recover from failures.

**Exam Tips:**
**Key Points:**
- ENIs are bound to a specific Availability Zone (AZ) and cannot be moved across AZs.
- You can attach multiple ENIs to an instance, depending on the instance type.
- ENIs retain their attributes (e.g., private IPs, MAC addresses) when detached and reattached to another instance.
- ENIs are useful for creating dual-homed instances (e.g., instances with public and private IPs).

**Common Mistakes:**
- Forgetting that ENIs are AZ-specific.
- Overlooking the fact that ENIs can have multiple private IPs and one public IP.
- Confusing ENIs with Elastic IPs (EIPs), which are public IPs that can be remapped.
</details>

<details>
<summary>Q. What is EC2 hibernate and use of it? scenarios ? 
</summary>

Pauses an EC2 instance and saves its in-memory state (RAM) to the root EBS volume, allowing it to resume later from the exact state. <br>
User Case : 

what is use of hibernation ? <br>
- Save time by preserving the RAM state.
- Resume instances faster than booting from scratch.
- Ideal for long-running applications that need to be paused and resumed quickly. <br>

**Key Prerequisites:**
- Encrypted root EBS volume (mandatory).
- RAM ≤ EBS root volume size (e.g., 16 GiB RAM → root volume ≥ 16 GiB).
- Supported instance families: M5, C5, R5, T3 (not all instance types).

**Hibernate vs. Stop:**
- **Stop:** Terminates RAM state; starts fresh on next launch.
- **Hibernate:** Preserves RAM state; resumes processes.
</details>

## EBS volume
<details>
<summary> Q. what is EBS volume and its use. </summary>
<summary>What is EBS volume and its use?</summary>
EBS volumes are like USB sticks for EC2, providing persistent block storage. They remain intact even after the associated EC2 instance is terminated. EBS volumes can be dynamically attached to or detached from EC2 instances, allowing for flexible storage management. <br>
- **Per AZ:** EBS volumes are specific to an Availability Zone.
- **Attachment:** Can be mounted to multiple EC2 instances but attached to only one instance at a time. <br>
### Exam Tips:
- **Incremental Nature of Snapshots:** Understand that EBS snapshots are incremental. After the initial snapshot, only the changed data blocks are stored in subsequent snapshots.
- **Data Restoration:** When restoring a volume from a snapshot, the new volume will be an exact replica of the original volume at the time the snapshot was taken.
- **Cross-Region and Cross-Account Sharing:** EBS snapshots can be copied across regions and shared between AWS accounts, facilitating data migration and disaster recovery strategies.
- **Cost Considerations:** Since snapshots are stored incrementally, they are cost-effective. However, it's essential to manage and delete unnecessary snapshots to avoid accruing storage costs.
</details>

<details>
<summary>EBS volume types</summary>
**1. General Purpose SSD (gp2) and (gp3)**
- **Use Case:** boot volumes, virtual desktops, dev and test environments.
**2. Provisioned IOPS SSD (io1) and (io2)**
- **Use Case:** critical business applications, large databases, I/O-intensive workloads.

**3. Throughput Optimized HDD Hard Disk Drive **
- **Use Case:** big data, data warehousing, log processing.
</details>


<details>
<summary> Q. what is multi attach feature ?</summary>
**Multi-Attach:** Allows a single EBS volume to be attached to multiple EC2 instances within the same AZ. <br>
each instance can read and write data to the volume simultaneously. <br>
**Use Cases:** Shared file systems, clustered databases, and applications requiring high availability. <br>

**Key Points:**
application  must be designed to handle concurrent writes to the volume.
-upto 16 instances can be attached to a single volume.
</details>

<details>
<summary>Q. Explain EC2 instance store</summary>

**Better IO performance**  
If an EC2 instance store stops or terminates, data is lost.

- **Best for:** Buffer, cache, scratch data, temporary content
- **Ideal for:** Low latency, high performance applications

**Example Use Case:**  
A machine learning job processing large datasets where temporary storage is needed to handle intermediate data and the results are later saved to persistent storage (like EBS or S3).

**Exam Tips:**

- **Ephemeral Nature:** EC2 Instance Store is temporary and not persistent. Data is lost when the instance is stopped or terminated, so it should only be used for non-critical, transient data.
- **Not All Instance Types Support It:** Not all EC2 instance types come with instance store volumes. You must select instance types that provide instance store support (e.g., i3, d2, r5d).
- **Cost-effective for Temporary Storage:** Instance stores are typically more cost-effective for workloads needing high-throughput but not requiring data persistence.
- **Common Mistake:** Confusing instance store with persistent storage options like EBS or S3. Instance store is only suitable for ephemeral storage needs.
</details>


<details>
<summary>Q. what is Amazon EFS- Elastic file system</summary>
Its a managed NFS (Network File System) that can be shared across thousands of EC2 instances. <br>
**Use Cases:** Content management, web serving, data sharing, and container storage. <br>
**Key Features:** Scalable, elastic, highly available, and durable. <br>
uses security groups to control access to the file system. <br>
only for linux based instances. not windows. For windows there is separate file system by AWS  <br>
Amazing thing is we do not need to do capacity planning. it scales automatically. (its one of the option and recommended option <br>
</details>



<details>
<summary>Q. EFS vs EBS, use case and exam tips</summary>

**Example Use Case:**

**EFS:**
- Used for applications like content management systems (CMS), big data analytics, or media workflows where multiple EC2 instances need concurrent access to the same data.
- **Example:** A web application with several EC2 instances running web servers that need access to shared media files, such as images or videos.

**EBS:**
- Ideal for applications needing persistent block storage that is attached to a single instance, such as a database or transactional systems.
- **Example:** A database hosted on EC2, where the data must be persistent and only needs to be accessed by one EC2 instance at a time.

**Exam Tips:**

**EFS Key Points:**
- Shared file storage for multiple EC2 instances.
- Uses NFS protocol and supports Linux-based instances.
- Automatically scales as data grows.
- Ideal for distributed applications and content sharing across EC2 instances.

**EBS Key Points:**
- Block storage that can be attached to a single EC2 instance.
- Ideal for databases, transactional applications, or any workloads requiring persistent storage.
- EBS volumes persist even when the EC2 instance is stopped, unlike EFS, which supports multiple simultaneous mounts.

**Common Mistakes:**
- **EFS vs. EBS for shared access:** EFS is the right choice for shared data access across multiple instances. EBS is not designed for multiple instances.
- **Mounting EBS on multiple instances:** EBS volumes can only be mounted on one EC2 instance at a time (except for EBS Multi-Attach, which has limited support).
</details>



<details>
<summary></summary>
</details>




*****************
TEST - IGNORE BELOW FOR NOW
<details>
  <summary>Q. What is the ENI and what are the types of ENI?</summary>
  It's a logical networking component in a VPC that represents a virtual network card. It can be attached to an EC2 instance in the same AZ. <br>
  You can create multiple ENIs and attach them to different instances **on the fly**. <br>
  <code style="color: orange">Bound to specific AZ, cannot be moved to another AZ.</code> <br>
</details>



Questions to answer later
Q. what is bursting meaning ? overall as a concept in the cloud ? <br>
