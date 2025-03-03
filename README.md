# AWS Solutions Architect Associate Notes

Study material
1. exam guide : https://d1.awsstatic.com/training-and-certification/docs-sa-assoc/AWS-Certified-Solutions-Architect-Associate_Exam-Guide.pdf
2. Stefan Marek's AWS Certified Solutions Architect Associate course on Udemy
3. AWS Skill builder.
4. Whiz labs AWS Certified Solutions Architect Associate tests
5. AWS white papers and website documentation.

## Table of Contents
1. <a href="#introduction">Introduction</a>
2. <a href="#identity-access-management-iam">Identity Access Management (IAM)</a>
3. <a href="#EC2">Elastic Cloud Compute EC2</a>
4. <a href="#EBS-volume">EBS volume</a>
5. <a href="#High-Availability-and-Scalability-ELB-and-ASG">High Availability and Scalability ELB and ASG</a>
6. <a href="#RDS-Aurora-ElastiCache">RDS Aurora ElastiCache</a>
7. <a href="#Route-53">Route 53</a>
8. <a href="#Amazon-S3-and-other-storage-services">Amazon S3 and other storage services</a>
9. <a href="#CloudFront-and-AWS-Global-Accelerator">CloudFront and AWS Global Accelerator</a>
10. <a href="#SQS-SNS">SQS SNS</a>
11. <a href="#ECS-EKS">ECS EKS</a>
12. <a href="#Lambda">Lambda</a>
13. <a href="#DynamoDB">DynamoDB</a>
14. <a href="#API-Gateway">Api Gateway</a>
15. <a href="#Step Functions">Step Functions</a>
16. <a href="#Cognito">cognito</a>
17. <a href="#Databases-in-aws-and-analytics">Databases In AWS and analytics</a>
18. <a href="#machine-learning">Machine Learning</a>
19. <a href="#AWS-Security">AWS Security</a>
20. <a href="#VPC">VPC</a>
21. <a href="#Disaster-Recovery-and-migrations">Disaster Recovery</a>


## Introduction

- AWS solution architect associate exam focus on below 4 areas
  - Designing and deploying scalable and secure cloud solutions
  - Designing and deploying high availability and fault tolerant cloud solutions
  - Designing and deploying cost effective cloud solutions
  - Designing and deploying secure cloud solutions.

Below are the topics mostly as per Stefan Marek's course on Udemy and additional notes I made using AWS skill builder content and using chatGPT.
Hope it helps whoever is referring this notes along with their own preparations.


## Identity Access Management (IAM)


<details>
<summary>🎯Q. AWS Organization </summary>

- AWS Organizations is a service that helps you centrally manage and govern multiple AWS accounts.
- It allows you to create and manage accounts, apply policies (like Service Control Policies - SCPs), and consolidate billing. It’s ideal for enterprises needing to enforce security, compliance, and cost management across multiple accounts.

A company, XYZ Corp, has three departments: HR, Finance, and IT. They create an AWS Organization with a Management Account and three Member Accounts (one for each department).

- They use Service Control Policies (SCPs) to restrict access to certain services (e.g., Finance cannot use EC2).
- They enable Consolidated Billing to track costs across all accounts.
- They use AWS IAM Identity Center (formerly AWS SSO) to provide single sign-on access to all accounts.

Common Mistakes: ⚠️

- `Misunderstanding SCPs`: SCPs don’t grant permissions; ⭐they only restrict them⭐. Permissions are still managed by IAM policies.
- Overlooking the Management Account: It has full access to all member accounts and can’t be restricted by SCPs.
- Assuming SCPs affect all services: Some AWS services (e.g., AWS Support) are exempt from SCPs.

</details>

<details>
<summary> 🎯Q: what is Users, Groups, Roles, and Policies? IMP - remember role is HAT/CAP which can be put on group or users</summary>

- Users: Assigned credentials (username/password, access keys). <br>
- Groups: Users are added to groups (e.g., "Developers"), and policies are attached to groups. 
- Roles: Assigned to AWS services (like EC2) or users temporarily. Policies are attached to roles. 
- Policies: JSON documents that define permissions.

- IAM roles provide temporary credentials (access key, secret key, and token) when assumed, replacing long-term access keys. <br>
<span class="highlighted-text"> Role is like hat, which is being wear by user or service to perform certain tasks.</span> <br>
These credentials are short-lived and used by users, apps, or AWS services to perform tasks securely. <br>
Example reference link : https://www.youtube.com/watch?v=miij_0HkBws <br>
</details>

<details>
<summary>🎯Q: What are policies and what are the types of policies ?</summary>

- Policies are JSON documents that define permissions in AWS. They specify what actions are allowed/denied on which resources under what conditions.

- simple policy example below - 
- (Allows listing objects in example_bucket only if the prefix is "home/")

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

- Different types of policies in AWS
    - `Identity-based policies`: Attached to users, groups, or roles.
    - `Resource-based policies`: Attached to resources (e.g., S3 bucket policies).
    - `Organization policies`: Applied at the AWS Organization level.
    - `ACLs`: Legacy rules for resource access, primarily for S3 and DynamoDB.
    - `Session policies`: Temporary permissions applied during role sessions (e.g., STS assume-role).

### AWS IAM Policy Evaluation Logic

- IAM Policy Evaluation Logic:
  - Default: Deny (requests are denied unless explicitly allowed).
  - Explicit DENY overrides any ALLOW.
  - For cross-account access, both identity-based (if applicable) and resource-based policies must allow the action.
  - If a user is part of multiple groups, the most permissive policy applies.
  - Policies are evaluated in the following order: 
    - Identity-based policies
    - Resource-based policies
    - Organizations SCPs
    - Session policies
    - ACLs (legacy, not recommended)
  - If a policy doesn't explicitly allow or deny an action, the default is to deny.

Exam Tips: ⭐

- Explicit DENY always wins – even if other policies ALLOW.
- Resource-based policies (e.g., S3 bucket policies) require the principal to be explicitly specified.

Common Mistakes: ⚠️

- If there is explicit deny in any policy then it will be denied irrespective of other policies.
- Forgetting that S3 bucket access often requires both IAM policies (identity) and bucket policies (resource) to align.

</details>

<details>
<summary>🎯 Q: Two types of authorization model RBAC and ABAC what they are and difference between them?</summary>

- RBAC (Role-Based Access Control) assigns permissions based on user roles, while ABAC (Attribute-Based Access Control) grants access based on attributes (e.g., user, resource, environment) and policies.

Simple end-to-end real-world example for better visualization: ⭐

- RBAC: In a company, a "Manager" role has access to approve budgets, while an "Employee" role can only view them.
- ABAC: A policy allows access to a file only if the user’s department matches the file’s department tag and the access time is during business hours.


Exam Tips: ⭐

- RBAC is simpler and ideal for static environments with well-defined roles.
- ABAC is more flexible and scalable, suitable for dynamic environments with complex access requirements.
- ⭐ABAC uses attributes like user ID, resource tags, and time of day to make access decisions.⭐
- Understand how AWS IAM supports both models: RBAC through policies attached to roles and ABAC through conditions in policies.

Common Mistakes: ⚠️

- Confusing RBAC with ABAC: RBAC is role-centric, while ABAC is attribute-centric.
- Overlooking the scalability of ABAC for large, dynamic systems.
- Ignoring the importance of defining clear attributes and policies in ABAC.
</details>

<details><summary>🎯Q: AWS Control Tower?</summary>

- AWS Control Tower is a service that simplifies setting up and governing a secure, multi-account AWS environment. It automates best practices and provides guardrails to enforce compliance.

Simple end-to-end real-world example for better visualization: ⭐
- A company wants to manage multiple AWS accounts for different departments (e.g., HR, Finance, IT). AWS Control Tower sets up a landing zone, creates accounts, and applies guardrails to ensure compliance (e.g., preventing public S3 bucket access).

Exam Tips: ⭐

- AWS Control Tower is ideal for managing multi-account AWS environments.
- Guardrails are policies that enforce compliance (e.g., preventive vs. detective guardrails).
- It integrates with AWS Organizations for account management and governance.
- Understand the difference between mandatory and elective guardrails.

Common Mistakes: ⚠️

- Assuming AWS Control Tower is only for small-scale environments (it’s designed for multi-account setups).
- Overlooking the importance of guardrails in enforcing compliance.
- Confusing AWS Control Tower with AWS Organizations ⭐(Control Tower builds on Organizations).⭐

</details>

<details>
<summary>🎯Q: what is security token service ? </summary>

- AWS Security Token Service (STS) generates temporary, scoped credentials (lasting minutes to hours) to delegate access to AWS resources without exposing long-term keys.

Common Mistakes

❌ Assuming STS works only for human users (used by AWS services like Lambda).
❌ Overlooking the need for trust policies in IAM roles when using AssumeRole.
❌ Using STS for static credentials (e.g., CI/CD pipelines → use IAM roles instead).
- `Credentials expire` (default: 1 hour; max 12 hours for IAM roles).
- Always use roles instead of long-term keys for dynamic workloads (e.g., Lambda, EC2).

</details>

## EC2

<details> <summary>🎯Q: EC2 Placement Groups types and use cases?</summary>

- `Cluster`: Instances are grouped into a single Availability Zone (AZ) for ⭐low latency.⭐
  - PRO : low latency, high performance.
  - limitation : If AZ fails, all instances fail.

- `Spread`: Instances are spread across distinct hardware (max 7 instances per AZ) for fault tolerance. 
  - here each EC2 instance is on different hardware(⭐even when its in same AZ⭐) and in different AZs as well.
  - PRO : fault tolerance.
  - limitation : limited to 7 instances per AZ.
  - **UseCase** : critical applications, where each instance must be isolated from failure from each other.
  
- `Partition`: Instances are divided into partitions, each isolated from failures (used for large distributed systems like Hadoop).
   - Same Availability Zone (AZ): All instances within a partition placement group are in the same AZ.
   - Same Region: All instances within a partition placement group are in the same AWS region.
   - Isolation: Each partition is isolated from the failures of other partitions, providing fault tolerance.
   - 

- `cluster` is single AZ, `spread` is multiple AZs, `partition` is multiple AZs and multiple regions.

Simple real-world example for end-to-end visualization: ⭐

- Cluster: A high-performance computing (HPC) workload requiring fast communication between instances (e.g., financial modeling).
- Spread: Hosting mission-critical applications (e.g., healthcare systems) where downtime is unacceptable.
- Partition: A distributed database (e.g., Cassandra) where partitions ensure failure isolation.

Exam Cheat Sheet ⭐

| Type | Best For | AZs | Failure Risk |
|------|-----------|-----|--------------|
| Cluster | Speed (HPC, gaming) | Single AZ | Entire group if rack fails |
| Spread | Critical small apps | Multiple AZs | Only 1 instance per failure |
| Partition | Big data (Hadoop) | Multiple AZs | Only 1 partition (subset of nodes) |

Brain Hack for Exam⭐
- Cluster = "Cram together" → Speed.
- Spread = "Spread out" → Safety.
- Partition = "Divide and conquer" → Scale + Partial safety.



Exam Tip:

- "Low latency" → Cluster
- "Fault tolerance" → Spread
-  "Large distributed apps"  → Partition

</details>

<details>
<summary>🎯Q: Different instances types in EC2 ?</summary>
Instance Types:

- `General-purpose` (e.g., T3, M6g) for balanced workloads.
- `Compute-optimized` (e.g., C5) for CPU-intensive tasks.
- `Memory-optimized` (e.g., R5) for in-memory databases.
</details>


<details>
<summary>🎯Q. What is the ENI and why it's used? </summary>

- An Elastic Network Interface (ENI) is a virtual network interface that enables communication between an Amazon EC2 instance and other AWS resources or on-premises networks. 
- It provides network connectivity, IP addresses (IPv4/IPv6), security groups, and MAC addresses to instances.

**what is attached to the network interface?**
** Security groups, public IP, Elastic IP, private IP,and MAC address. <br>
MAC address means Media Access Control address, which is a unique identifier assigned to network interfaces for communications on the physical network segment. <br>

**Why and When It Is Used:**
- **High Availability:** You can attach multiple ENIs to an EC2 instance, each in a different subnet, to ensure redundancy.
- **Network Segmentation:** ENIs allow you to separate traffic types (e.g., management vs. data traffic) by attaching multiple interfaces to a single instance.
- **Failover Scenarios:** ENIs can be moved between instances to quickly recover from failures.

**Exam Tips:**
**Key Points:**
- ENIs are ⭐bound to a specific Availability Zone (AZ) and cannot be moved across AZs⭐.
- You can attach multiple ENIs to an instance, depending on the instance type.
- ⭐ENIs retain their attributes (e.g., private IPs, MAC addresses) when detached and reattached to another instance.⭐
- ENIs are useful for creating ⭐dual-homed instances⭐ (e.g., instances with public and private IPs).

**Common Mistakes:**
- Forgetting that ENIs are AZ-specific.
- Overlooking the fact that ENIs ⭐can have multiple private IPs and one public IP⭐.
- Confusing ENIs with Elastic IPs (EIPs), which are public IPs that can be remapped.
</details>


<details>
<summary>🎯Q. What is EC2 hibernate and use of it? scenarios ?</summary>

- It Pauses an EC2 instance and saves its in-memory state (RAM) to the root EBS volume, allowing it to resume later from the exact state. 


what is use of hibernation ? <br>
- Save time by preserving the RAM state.
- Resume instances faster than booting from scratch.
- Ideal for long-running applications that need to be paused and resumed quickly. <br>

**Key Prerequisites:**
- ⭐Encrypted root EBS volume (mandatory).⭐
- RAM ≤ EBS root volume size (e.g., 16 GiB RAM → root volume ≥ 16 GiB).
- Supported instance families: M5, C5, R5, T3 (not all instance types).

**Hibernate vs. Stop:**
- **Stop:** ⭐Terminates RAM state;⭐ starts fresh on next launch.
- **Hibernate:** ⭐Preserves RAM state;⭐ resumes processes.
</details>


## EBS volume
<details>
<summary>Q: What is EBS volume and its use?</summary>

- **EBS volumes** are like USB sticks for EC2, providing persistent block storage. 
- They remain intact even after the associated EC2 instance is terminated. 
- EBS volumes can be ⭐dynamically attached to or detached from EC2 instances⭐, allowing for flexible storage management.
- **Per AZ:** EBS volumes are specific to an Availability Zone.
- **Attachment:** Can be mounted to multiple EC2 instances but attached to only one instance at a time.

### Exam Tips:
- **Incremental Nature of Snapshots:** Understand that EBS snapshots are incremental. After the initial snapshot, only the changed data blocks are stored in subsequent snapshots.
- **Data Restoration:** When restoring a volume from a snapshot, the new volume will be an exact replica of the original volume at the time the snapshot was taken.
- **Cross-Region and Cross-Account Sharing:** EBS snapshots can be copied across regions and shared between AWS accounts, facilitating data migration and disaster recovery strategies.
- **Cost Considerations:** ⭐Since snapshots are stored incrementally, they are cost-effective⭐. However, it's essential to manage and delete unnecessary snapshots to avoid accruing storage costs.
</details>

<details>
<summary>🎯Q . what are different EBS volume types</summary>

**1. General Purpose SSD (gp2) and (gp3)**
- **Use Case:** Boot volumes, virtual desktops, dev and test environments.

**2. Provisioned IOPS SSD (io1) and (io2)**
- **Use Case:** Critical business applications, large databases, I/O-intensive workloads.

**3. Throughput Optimized HDD (st1)**
- **Use Case:** Big data, data warehousing, log processing.

**4. Cold HDD (sc1)**
- **Use Case:** Infrequently accessed data, lower cost storage.

**5. Magnetic (standard)**
- **Use Case:** Previous generation, low-cost storage for infrequently accessed data.
</details>


<details>
<summary>🎯Q: What is the multi-attach feature?</summary>

- Allows a single EBS volume to be attached to multiple EC2 instances within the same AZ. 
- Each instance can read and write data to the volume simultaneously.

**Use Cases:**
- Shared file systems
- Clustered databases
- Applications requiring high availability

Exam Tips: ⭐
- Multi-Attach is only supported for io1 and io2 volume types.
- All instances must be in the ⭐same Availability Zone⭐
- Ideal for clustered workloads like databases (e.g., Oracle RAC, SQL Server Failover Cluster).
- Not supported for boot volumes—only for data volumes.
- Use Multi-Attach for high availability and low-latency shared storage scenarios.

Common Mistakes: ⚠️
- Assuming Multi-Attach works with all EBS volume types (it’s only for io1/io2).
- Trying to attach the volume to instances in different Availability Zones.
- Using Multi-Attach for boot volumes (it’s not supported).

**Key Points:**
- Application must be designed to handle concurrent writes to the volume.
- Up to 16 instances can be attached to a single volume.

</details>

<details>
<summary>🎯Q. Explain EC2 instance store</summary>

- **EC2 Instance Store** provides temporary block-level storage for EC2 instances.
- If an EC2 instance store stops or terminates, data is lost.
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


<summary>🎯Q: What is Amazon EFS - Elastic File System?</summary>
**Amazon EFS** is a managed NFS (Network File System) that can be shared across thousands of EC2 instances.

**Use Cases:**
- Content management
- Web serving
- Data sharing
- Container storage

**Key Features:**
- Scalable, elastic, highly available, and durable
- Uses security groups to control access to the file system
- Only for Linux-based instances (for Windows, there is a separate file system by AWS)
- No need for capacity planning as it scales automatically (recommended option)
</details>



<details>
<summary>🎯Q. EFS vs EBS, use case and exam tips</summary>
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
<summary>🎯Q: Why is the "Delete On Termination" attribute enabled by default for the root volume but disabled for other EBS volumes?</summary>

**Root Volume:**
- **Reason:** The root volume contains the operating system and is essential for the instance to boot. When an instance is terminated, it is often desirable to delete the root volume to avoid unnecessary storage costs and to ensure that the instance is completely removed.
- **Default Behavior:** Enabled by default to automatically clean up the root volume when the instance is terminated.

**Other EBS Volumes:**
- **Reason:** Additional EBS volumes are often used to store important data that may need to persist beyond the lifecycle of the instance. Automatically deleting these volumes could result in data loss.
- **Default Behavior:** Disabled by default to ensure that data stored on these volumes is preserved even if the instance is terminated.
</details>

<details>
<summary>🎯Q: Can you launch an EC2 instance using an AMI from another AWS Region?</summary>

**Answer:** No, you cannot directly launch an EC2 instance using an AMI from another AWS Region. ⭐AMIs are unique to each AWS Region.⭐

**Solution:** You can copy the AMI to the target AWS Region and then use it to create your EC2 instances.
</details>

## High Availability and Scalability ELB and ASG
- High Availability (HA) ensures your application remains operational even if components fail,
- Scalability ensures your application can handle increased load by adding resources.

<details>
<summary>what is scale-up and scale-down and scale-out and scale-in means ?</summary>
This is a key concept in cloud computing and is often tested in the exam.
`Scalability`: The ability to increase or decrease resources based on demand. <br>
`vertical scaling is scale-up and horizontal scaling is scale-out`. <br>
Scale-up: Increasing the size of an instance (e.g., upgrading from t2.micro to t2.large). <br>
Scale-down: Decreasing the size of an instance (e.g., downgrading from m5.xlarge to t3.medium). <br>
Scale-out: Adding more instances to distribute the load. <br>
Scale-in: Removing instances to reduce the load. <br>
</details>


<details>
<summary>🎯 Q. what are the types of the load balancer?</summary>
Load Balancer : it's a service that distributes incoming application or network traffic across multiple targets, such as EC2 instances, containers, and IP addresses, in multiple Availability Zones. <br>
-its per region service. <br>

Types of load balancer (4 types):
1. Application Load Balancer (ALB) : Layer 7, HTTP/HTTPS, intelligent routing, and microservices.
   - supports HTTP2 and websocket. 
   -  routing to different target groups based on URL, hostname and query string headers
   - `supports containerized applications and microservices`
2. Network Load Balancer (NLB) : Layer 4, TCP/UDP, high performance, and static IP. <br>
    - Forward TCP and UDP traffic to your instances.
    - handle millions of requests per second.
    - ⭐ **`NLP has one static IP per AZ and supports assigning Elastic IP address`**
    - Target groups can be EC2 instances, IP addresses (must be private) and ⭐Application Load Balancers.
3. Gateway Load Balancer : Deploy, scale, and manage third-party virtual appliances. <br>
    - Deploy, scale, and manage third-party virtual appliances such as firewalls, intrusion detection and prevention systems, and deep packet inspection systems in the cloud.
    - Uses the GENEVE on port 6081 protocol to encapsulate and forward traffic to virtual appliances.
4. Classic Load Balancer : Legacy, Layer 4/7, HTTP/HTTPS, and TCP. <br>
</details>

<details>
<summary>🎯Q. what is Sticky Sessions(Session Affinity) </summary>
Sticky Sessions (Session Affinity): Ensures that a client's requests are always directed to the same target. <br>
2 types <br>
    - Load balancer-generated cookie: The load balancer generates a cookie to track the session. <br>
    - Application-generated cookie: The application generates a cookie to track the session. <br>
</details>

<details>
<summary>🎯Q. what is cross zone balancing ? </summary>
Cross-Zone Load Balancing is a feature of AWS Elastic Load Balancers (ELBs) that distributes incoming traffic ⭐evenly⭐ across all registered instances in all Availability Zones (AZs) within a region. 
This ensures that no single AZ is overwhelmed with traffic while others remain underutilized. <br>
Know that cross-zone load balancing is ⭐enabled by default for Application Load Balancers but needs to be manually enabled for Network Load Balancers.⭐
</details>


<details>
<summary>🎯🔥Q. what is Auto Scaling group? what are scaling policies ?</summary>

**Definition:** Is a service that automatically adjusts the number of instances in a group based on demand or a predefined schedule. <br>
- **Scale-out:** Increase the number of instances based on demand. <br>
- **Scale-in:** Decrease the number of instances based on demand. <br>
- **Automatically replaces unhealthy instances** and also registers new instances to the load balancer.
- Minimum capacity : always running instances
- Desired capacity : Target number of instances
- Maximum capacity : Maximum number of instances that can be running. <br>

<details>
<summary>Example scenarios to understand desired capacity below</summary>
Example Scenario 1: E-commerce Website

Minimum Capacity: 3 instances
Maximum Capacity: 10 instances
Desired Capacity: 5 instances
Explanation:

The ASG will always keep at least 3 instances running to handle basic traffic.
It can scale up to a maximum of 10 instances if needed.
The ASG aims to maintain 5 instances under normal load. If traffic increases, it might scale out to 7 or 8 instances until the load decreases, at which point it will scale back down to 5.
Example Scenario 2: Seasonal Application

Minimum Capacity: 2 instances
Maximum Capacity: 15 instances
Desired Capacity: 10 instances
Explanation:

During peak season, the application typically needs 10 instances to handle traffic.
The ASG starts with 2 instances running (minimum).
As traffic increases, it scales up to reach the desired capacity of 10. If traffic spikes even more, it can scale up to the maximum of 15 instances.
Example Scenario 3: Fluctuating Workload

Minimum Capacity: 1 instance
Maximum Capacity: 5 instances
Desired Capacity: 1 instance
Explanation:

The application has low traffic most of the time, so it starts with a desired capacity of 1 instance.
If traffic increases, the ASG can scale up to 5 instances if needed, but it will try to maintain 1 instance during low traffic periods.
If traffic drops, it will scale down to the minimum of 1 instance.
Key Takeaway

Desired Capacity is the target number of instances the ASG tries to maintain based on current load. It can change dynamically based on traffic, but it will always respect the minimum and maximum capacities set for the ASG.
</details>

**Understand scaling policies:** ⭐
- **Target Tracking:** Automatically adjusts the number of instances to maintain a target metric (e.g., CPU utilization).
- **Step Scaling:** Scales based on predefined thresholds (e.g., scale out if CPU > 70%).
- **Simple Scaling:** Manually adjusts the number of instances based on a single scaling adjustment.
- **Scheduled Scaling:** Scales based on a schedule (e.g., scale out during peak hours, specify the time).

**Exam Tips:** ⭐
- `Understand Scaling Policies`: Focus on Target Tracking and Step Scaling, as they are commonly tested.
- `Multi-AZ Deployment`: Always configure ASGs across multiple Availability Zones (AZs) for high availability.
- `Health Checks`: Ensure ASG integrates with Elastic Load Balancer (ELB) health checks for instance health monitoring.

**Common Mistakes:** ⚠️
- Not Defining Minimum/Maximum Capacity: Failing to set these can lead to excessive costs or insufficient resources.
- Ignoring Cooldown Periods: Not configuring cooldown periods can result in rapid scaling actions, causing instability.
- ⭐Overlooking Instance Termination Policies⭐: Not setting termination policies can lead to unintended instance removal. <br>
  ⭐Termination policies determine which instances to terminate when scaling in (removing instances) in an Auto Scaling Group. here are common termination policies:⭐
  - Default Policy: AWS chooses the instance with the oldest launch configuration or template.
  - Newest Instances: Prioritizes terminating the most recently launched instances.
  - Lowest Cost: Chooses instances based on cost metrics, such as Spot Instances.
  - Health Status: Terminates unhealthy instances first.
  - Availability Zone Balance: Ensures balanced instance distribution across Availability Zones.
</details>

<details>
<summary>🎯🔥 Q. Load Balancer, Target Group and Auto Scaling Group how they are related to each other? </summary>

`Load Balancer (LB)`: Distributes incoming traffic across multiple targets (e.g., EC2 instances).
`Target Group (TG)`: Routes requests to registered targets (instances/lambda) via rules. LB directs traffic to TGs.
`Auto Scaling Group (ASG)`: Automatically adjusts EC2 capacity based on demand. Launches/terminates instances.
 ⭐`Relation` ⭐: ASG scales instances, registers them to a TG, and LB routes traffic to the TG.<br>

⭐ Exam Tips: ⭐
- ASG + LB = High availability + scalability.
- Target Groups decouple LB from instances (e.g., LB can route to different TGs).
- Scaling policies prioritize ⭐Desired first⭐, then adjust within min/max. This means that the ASG will always try to maintain the Desired Capacity, but it can scale up to the Maximum Capacity if needed.


Common Mistakes: ⚠️
- Forgetting to link ASG to a TG (traffic won’t reach scaled instances). - ⭐means ASG should be linked to TG and TG should be linked to LB⭐.
- Setting min ≥ max or desired outside min/max range.
- Confusing LB health checks (for traffic routing) with ASG health checks (for instance replacement).- means ⭐LB health checks are for routing traffic and ASG health checks are for instance replacement.⭐
</details>


🎯 ⭐One liners notes⭐ 🎯
- Only Network Load Balancer provides both ⭐static DNS name and static IP⭐. While, Application Load Balancer provides a static DNS name, but it does NOT provide a static IP 🚫or even also not provide a way to attach elastic IP.🚫
- To get the client's IP address to application code, ALB adds an `X-Forwarded-For header` to the request., `X-forarded-port`-for client's port number, `X-forarded-proto`-for client's protocol.
- `network load balancer` cannot be a target in target groups however `application load balancer` can be a target in target groups.
- Server Name Indication (SNI) allows you to expose multiple HTTPS applications each with its own SSL certificate on the same listener.
- ⭐You can configure the Auto Scaling Group to determine the EC2 instance's health based on Application Load Balancer Health Checks instead of EC2 Status Checks (default)⭐. When an EC2 instance fails the ALB Health Checks, it is marked unhealthy and will be terminated while the ASG launches a new EC2 instance.
- The NLB supports HTTP health checks as well as TCP and HTTPS


## RDS Aurora ElastiCache
<details>
<summary>🎯Q. What are advantages of using RDS(relational database service) instead DB in the EC2 ?</summary>

- it's a managed service which provides automated provisioning, backups, patching, monitoring, and scaling of databases.
- provides multi-AZ setup for Disaster Recovery.
- it's a managed service which provides automated provisioning, backups, patching, monitoring, and scaling of databases.
- provides multi-AZ setup for Disaster Recovery.
- automated backups with point-in-time recovery (up to 35 days).
- monitoring dashboards with CloudWatch.
- storage backed by EBS (gp2 or io1).
- cannot SSH into RDS instances (managed service).
</details>


<details>
<summary>🎯🔥Q. Difference between read replica and RDS Multi AZ in RDS? (very important feature)</summary>

- `Read Replica`: ⭐Asynchronous copy⭐ for scaling read workloads; can be in same/different regions; manual promotion to standalone.
- `Multi-AZ`: ⭐Synchronous replication⭐ to a standby in another AZ for high availability; automatic failover during outages.
  Exam Tips: ⭐

**Read Replica:**
- Used for read scaling, not for high availability.
- Can have multiple read replicas (up to 5 for MySQL, PostgreSQL, MariaDB).
- Can be promoted to a standalone database, but this breaks the replication.
- Does not support automatic failover.

**Multi-AZ:**
- Used for high availability and disaster recovery.
- Only one standby replica in a different AZ.
- Supports automatic failover with minimal downtime (1-2 minutes).
- Synchronous replication ensures zero data loss.

**Key Differences:**
- **Read Replica:** Async replication, used for read scaling.
- **Multi-AZ:** Sync replication, used for failover and high availability.

**Common Mistakes:** ⚠️
- Assuming Read Replica provides high availability (it does not; it's for read scaling).
- Thinking Multi-AZ improves read performance (it does not; it's for failover only).
- 🔥 Forgetting that Read Replica can be in a different region, while Multi-AZ is within the same region.

**Real-World Example:**
A news website uses Read Replicas to handle heavy read traffic from users browsing articles.
They also use Multi-AZ to ensure the database remains available during AZ outages or maintenance.
</details>



<details>
<summary>🎯Q. Amazon Aurora </summary>

- `Amazon Aurora` Amazon Aurora is a fully managed, MySQL/PostgreSQL-compatible relational database engine optimized for the cloud.
- offers upto 5 times better performance than MySQL and 3 times better performance than PostgreSQL.
- 6 copies of data across 3 AZs.
- `Writer endpoint` : This is the endpoint that you use for write operations.
- `Reader endpoint` : This is the endpoint that you use for read operations.
- custom endpoints can be created for specific use cases. Example : reporting endpoint.
- `Aurora Serverless` : automatically scales up or down based on the application's needs.
- `Global aurora` : allows to replicate data across multiple regions.
- `Aurora machine learning` : allows to integrate machine learning models with the database.
- `Aurora Backtrack` : allows to rewind the database to a specific point in time without using backups.

⭐Simple real-world example: ⭐
An e-commerce platform uses Aurora for its transaction database. Aurora’s automatic scaling (up to 128 TiB) handles Black Friday traffic spikes, while Multi-AZ deployments and read replicas ensure high availability. Backtracking fixes accidental data deletions, and Aurora Serverless manages unpredictable workloads during flash sales.

⭐Exam Tips: ⭐
- `Replicas`: Aurora supports up to 15 read replicas (vs. RDS’s 5), and replicas can be promoted to primary in seconds.
- `Storage`: Auto-scaling storage (10 GB increments up to 128 TiB) with 6-way replication across 3 AZs for durability.
- `Serverless`: Use Aurora Serverless for intermittent/unpredictable workloads (e.g., dev/test environments).
- `Backtracking`: Rewind the database to a specific time (seconds/minutes) without restoring backups.
- `Global Database`: Cross-region replication with <1-second latency (for disaster recovery/low-latency reads).

Common Mistakes: ⚠️
- `Confusing Aurora with DynamoDB`: Aurora is relational; DynamoDB is NoSQL.
- `Overlooking cost`: Aurora is more expensive than RDS but justifies cost for high scalability/performance.
- `Ignoring backtracking vs. snapshots`: **Backtracking is for short-term recovery (hours/days)**, while snapshots are for long-term.

</details>

<details>
<summary>🎯Q. Amazon RDS & Aurora Backup: </summary>

- RDS: Offers automated backups (enabled by default, retained up to 35 days) and manual DB snapshots (stored in S3). Supports point-in-time recovery.
- Aurora: Automatically backs up data to S3 continuously (retained up to 35 days). Supports database cloning and cross-region snapshots.

Simple real-world example: ⭐

- RDS Backup: An e-commerce app uses RDS automated backups for daily recovery. A manual snapshot is taken before a major sale event.
- Aurora Backup: A financial app uses Aurora’s continuous backup to recover from accidental data deletion in seconds.

Exam Tips: ⭐

Backup Differences: ⭐
- Aurora backups are continuous; RDS backups are daily with transaction logs.
- Aurora snapshots are instantaneous (no performance impact) due to storage clustering.
- Retention: Automated backups (RDS/Aurora) max at 35 days; manual snapshots persist until deleted.
- Use `RDS Read Replicas` when you need to scale out read-heavy workloads.
- Use `RDS Multi-AZ` when you need high availability and automatic failover for critical applications.

Common Mistakes: ⚠️
- Assuming Aurora snapshots are slow: They’re instant and storage-based (unlike RDS volume snapshots).
- Confusing RDS Custom with regular RDS: Custom allows SSH access; standard RDS doesn’t.
</details>


<details>
<summary>🎯Q. RDS Proxy what it is and why its important and in which situations to use ? </summary>

- `Amazon RDS Proxy` is a fully managed database proxy that sits between your application and RDS databases. 
- ⭐It allows you to pool and share database connections⭐, improving scalability and security.
- Never publicaly accessible and only accessible from within the VPC. 
- handles failover, connection pooling, and read/write splitting.

⭐Simple real-world example for better end-to-end visualization: ⭐ 💡🔥
Imagine a travel booking app using AWS Lambda (serverless) during peak holiday seasons. 
Thousands of users trigger Lambda functions simultaneously, each opening a new database connection. 
Without RDS Proxy, the database gets overwhelmed by too many connections, slowing down or crashing.
🔥 **With RDS Proxy, connections are reused efficiently**, Lambda functions share pooled connections, and failovers (e.g., during maintenance) happen without disrupting users.


Exam Tips: ⭐

- `Use RDS Proxy for serverless applications` (e.g., Lambda) or apps with frequent connection churn to prevent database overload.
- `Ideal for improving scalability (connection pooling)` and reducing database costs (fewer open connections).
- `Seamless failover`: RDS Proxy reroutes traffic to a standby DB instance during primary DB failures (e.g., Multi-AZ deployments).
- `Security`: Integrates with IAM for authentication and avoids exposing databases publicly (only accessible within your VPC).

Common Mistakes: ⚠️

- Assuming RDS Proxy is publicly accessible. ⭐It only works within your VPC.⭐
- Using it for non-RDS databases (e.g., self-managed MySQL on EC2). RDS Proxy supports only RDS/Aurora.
- Overlooking IAM role setup: Proxy requires IAM roles for authentication, not just database credentials.
</details>


<details>
<summary>🎯Q. Amazon Elasticache IMP notes and use cases </summary>

- To get managed Redis or Memcached in-memory data store.
- used to improve latency and throughput for read-heavy workloads.
- supports replication and multi-AZ for high availability.
- helps to make application stateless by storing session data in Elasticache.
- as its managed aws takes care of patching, monitoring, and backups.

<details>
<summary>Click to expand image 1</summary>
<img src="img_1.png" alt="Image 1" width="891">
</details>

<details>
<summary>Click to expand image 2</summary>
<img src="img_2.png" alt="Image 2" width="825">
</details>

Simple real-world example: ⭐
An e-commerce platform uses ElastiCache for Redis to cache product listings and user sessions. This reduces latency during peak traffic and ensures sessions persist during server failures.

Exam Tips: ⭐
- Redis vs. Memcached: Redis supports persistence, clustering, and complex data structures; Memcached scales horizontally for simple caching.
- Multi-AZ & Failover: Redis offers automatic failover; Memcached does not.
- Encryption: Redis supports in-transit/at-rest encryption; Memcached only in-transit.
- VPC Peering: Not supported; use VPC endpoints for cross-VPC access.

Common Mistakes: ⚠️
- Assuming Memcached supports clustering or persistence (it does not).
- Forgetting ElastiCache does not reduce RDS storage costs (it only reduces query load).
</details>

<details>
<summary>🎯🔥Q. Important ports to know (revise before exam) </summary>

Exam Tips: ⭐
- Port 22 = SSH (secure shell access) and SFTP (secure file transfer).
- Port 443 = HTTPS (secure web traffic), Port 80 = HTTP (unsecured web traffic).
- RDS ports: PostgreSQL (5432), MySQL (3306), Oracle (1521), MSSQL (1433).
- Aurora uses MySQL (3306) or PostgreSQL (5432) ports based on engine type.
- Security Groups require port rules, not IP protocols (e.g., allow port 3306 for MySQL, not "MySQL" as a service).
- FTP uses port 21 (control), but SFTP (port 22) is more secure.

Common Mistakes: ⚠️
- Confusing MySQL (3306) with PostgreSQL (5432) ports.
- Assuming SFTP uses port 21 (it uses 22, same as SSH).
- Forgetting Aurora inherits ports from its compatible engine (MySQL = 3306, PostgreSQL = 5432).
- Mixing up HTTPS (443) with SSL-based database connections (e.g., RDS uses its default port even with SSL).
- Opening all ports (0.0.0.0/0) instead of restricting to specific ports for security.
</details>


🎯 ⭐One liners notes⭐ 🎯
- `RDS Read Replicas` **add new endpoints with their own DNS name.** We need to change our application to reference them individually to balance the read load. However Multi-AZ keeps the same connection string regardless of which database is up.
- 🔥`RDS read replica` uses asynchronous replication, while `RDS Multi-AZ` uses synchronous replication.
- You can not create encrypted Read Replicas from an unencrypted RDS DB instance.




## Route 53
<details>
<summary>🎯Q. what is DNS ? Route 53 ? and hosted zone?</summary>

- `DNS (Domain Name System)` is a distributed system that translates domain names (e.g., www.example.com) into IP addresses.
- `Route 53` is a scalable and highly available Domain Name System (DNS) web service.
- `DNS Terminologies`
  - `Domain Name`: Human-readable name (e.g., www.example.com).
  - `IP Address`: Numeric address (e.g.,
  - `Domain Registrar`: Company that manages domain registration. (Amazon Route 53, GoDaddy, etc.)
  - `DNS records`: types of DNS records (e.g., A, CNAME, MX, NS, SOA, TXT).
    - A (Address): Maps domain to IPv4 address.
    - AAAA (IPv6 Address): Maps domain to IPv6 address.
    - CNAME (Canonical Name): Maps domain to another domain (e.g., www.example.com to example.com).
    - MX (Mail Exchange): Maps domain to mail servers.
    - NS (Name Server): Maps domain to authoritative name servers.
    - SOA (Start of Authority): Contains authoritative information about the domain.
    - TXT (Text): Contains arbitrary text information.
    - PTR (Pointer): Maps IP address to domain name (reverse lookup). and many more..
  - `zone file`: A file that contains DNS records for a domain.
  - `name server`: A ⭐server that stores DNS records for a domain.⭐
  - `root name server`: A root name server stores the DNS root zone and directs DNS queries to the appropriate top-level domain (TLD) name servers. ( its dot (.) domain)
  - `top-level domain (TLD)`: The last part of a domain name (e.g., .com, .org, .net).
  - `second level domain`: The part of a domain name before the TLD (e.g., example.com).
  - `subdomain`: A domain that is part of a larger domain (e.g., blog.example.com).
  - `fully qualified domain name (FQDN)`: A domain name that includes all levels (e.g., www.example.com).
  - `recursive DNS resolver`: A server that queries authoritative name servers on behalf of clients.
  - `authoritative name server`: A server that stores DNS records for a domain. ( Amazon Route 53, GoDaddy, etc.) - aka : Authoritative DNS.
  
`Route 53 - Records`
- helps to route traffic to the internet resources like EC2 instances, S3 buckets, ELB, etc.
- each record contains 
  - Domain/Subdomain name - e.g., example.com
  - Record type - e.g., A, AAAA, CNAME, MX, etc.
  - value - e.g., IP address, domain name, etc.
  - routing policy - means how to route the traffic to the resources. e.g., simple, weighted, latency, failover, geolocation, multivalue, etc.
  - TTL - time to live, how long the record is cached by the resolver(client).
    - means, if the TTL is 60 seconds, the resolver(client) will cache the record for 60 seconds and after that, it will query the authoritative name server to get the updated record.
- supports **A, AAAA, CNAME, MX, NS,** SOA, TXT, PTR, SRV, SPF, NAPTR, CAA, and more.
  - A : maps domain to IPv4 address.
  - AAAA : maps domain to IPv6 address.
  - CNAME : maps domain to another domain.
  - MX : maps domain to mail servers.
  - NS : maps domain to authoritative name servers. (example : ns-123.awsdns-12.com) 

`Hosted Zone` : A container for DNS records for a domain.
- `Public Hosted Zone` : Used to route traffic on the internet. Example : www.example.com
  - means you have a domain www.example.com and you want to route the traffic to your internet resources like EC2 instances, S3 buckets, ELB, etc. you create the public hosted zone in the route 53 and add the records to route the traffic to the resources.
- `Private Hosted Zone` : Used to route traffic within an Amazon VPC. Example : www.example.internal
  - means you have a domain www.example.internal and you want to route the traffic within the VPC to the resources like EC2 instances, S3 buckets, ELB, etc. you create the private hosted zone in the route 53 and add the records to route the traffic to the resources.
- </details>


<details>
<summary>🎯Q. what are routing policies for Route 53? </summary>

- Defines how Route 53 responds to DNS queries based on the routing configuration.
- `Simple Routing`: Maps a domain to a single resource (e.g., an IP address). 
- `Weighted Routing`: Distributes traffic based on weights assigned to resources.
- `Latency Routing`: Routes traffic based on the lowest network latency for end users.
- `Failover Routing`: Directs traffic to a standby resource ⭐during an outage.
- `Geolocation Routing`: Routes traffic based on the geographic ⭐location of the user⭐.
  - Examples:  
  - Users from Europe are routed to europe.example.com.
  - Users from North America are routed to us.example.com.
- `Geoproximity Routing`: Routes traffic based on the geographic ⭐location of the user and resources.⭐
  - defined bias to route the traffic to the specific resources. ( check stephan's video for excellent diagram explanation)
  - Examples
  - Users closer to the US East region are routed to us-east.example.com.
  - Users closer to the US West region are routed to us-west.example.com.
  - You can ⭐apply a bias⭐ to route more traffic to us-west.example.com even if users are closer to the US East region.
- `Multivalue Answer Routing`: Returns multiple values in response to DNS queries.
    - Simple Routing is suitable for straightforward use cases where you only need to map a domain to a single resource.
    - we can provide health checks for the resources in the multivalue answer routing.
</details>



<details>
<summary>🎯Q. What is Alias record ? how its different then CNAME? (IMPORTANT) </summary>

- An Alias Record in Route 53 is a special DNS record that maps a domain name (e.g., example.com) directly to an AWS resource (e.g., S3 bucket, CloudFront distribution, or another Route 53 record). Unlike CNAME records, Alias records can be used for the root domain (apex zone) and are free, with Route 53 automatically updating the record if the target’s IP changes.
- Simple Real-World Example: ⭐
  - Scenario: You host a static website on an S3 bucket named my-website-bucket. 
  - Goal: Point example.com (root domain) to the S3 bucket. 
  - Solution: Create an Alias record in Route 53 for example.com that aliases to the S3 bucket’s endpoint (my-website-bucket.s3-website-us-east-1.amazonaws.com). 
  - Result: Users accessing example.com are routed to the S3 bucket, and Route 53 automatically handles IP changes if AWS updates the S3 endpoint.

Exam Tips: ⭐

- Alias vs. CNAME: Alias records work for root domains (e.g., example.com), while CNAMEs only work for subdomains (e.g., www.example.com).
- AWS Resources Only: Alias records can only point to AWS services (e.g., ELB, CloudFront, S3, RDS) or other Route 53 records.
- Cost-Free: Alias queries to AWS resources are free, unlike CNAMEs, which incur standard DNS query costs.
- Automatic Updates: Alias records dynamically resolve to the target’s IPs, ensuring no manual updates if the target changes.


Common Mistakes: ⚠️

- `Using CNAME for Root Domain`: Attempting to use a CNAME for example.com (violates DNS standards; Alias is required).
- `Alias for Non-AWS Resources`: Trying to alias to external services (e.g., a third-party server)—use CNAME instead.
- `Confusing Alias with A Records`: Using an A record with an AWS resource’s IP (IPs can change; Alias ensures reliability).
  - If you want to map AWS elastic IP still you have to use the A record , as Alias record is only for the AWS resources like ELB, CloudFront, S3, etc.

additional exam specific notes:

- Q. When would you use an Alias record instead of a CNAME?
    - Answer: When pointing a root domain (e.g., example.com) to an AWS resource like S3 or CloudFront.
- Q. Can an Alias record point to an on-premises server?
    - Answer: No—Alias records only work with AWS resources or other Route 53 records.
- Q. What happens if an S3 bucket’s IP changes after configuring an Alias?
    - Answer: ⭐Route 53 automatically updates the Alias record’s IP⭐, ensuring no downtime.

</details>


<details>
<summary>🎯Q. Route 53 health checks </summary>

- Route 53 Health Checks monitor the availability and performance of AWS resources (e.g., EC2 instances, load balancers) or custom endpoints. They enable automated DNS failover by rerouting traffic to healthy resources when failures are detected. Health checks can evaluate HTTP/HTTPS status codes, TCP connectivity, or CloudWatch alarms.
- `Types of Health Checks`:
    - `Endpoint Health Check`: Monitors the health of an endpoint (e.g., a website URL).
    - `CloudWatch Alarm Health Check`: Monitors the status of a CloudWatch alarm.
    - `Calculated Health Check`: Combines multiple health checks to determine overall health.

Simple real-world example: ⭐
A company hosts a web app on an EC2 instance behind an Application Load Balancer (ALB) in us-east-1 and a backup ALB in us-west-2.
They configure Route 53 health checks to send HTTP requests to /health on both ALBs every 30 seconds.
If the primary ALB fails to respond twice consecutively, Route 53 marks it unhealthy and switches DNS traffic to the backup ALB, ensuring minimal downtime.

Exam Tips: ⭐

- Health checks can monitor endpoints (HTTP/HTTPS/TCP), CloudWatch alarms (for private resources), or other health checks (calculated checks).
- Use Failover Routing Policy to route traffic to a secondary resource if the primary fails.
- ⭐Health checks require publicly accessible endpoints unless using CloudWatch alarms for private resources.⭐
- Configure failure thresholds (e.g., 3 failures in 3 checks) to avoid false positives.

Common Mistakes: ⚠️

- Assuming health checks work for ⭐private resources⭐ without CloudWatch integration (Route 53 can’t access private IPs directly).
- Forgetting to configure correct status codes (e.g., expecting HTTP 200 instead of 3xx).
- Overlooking request intervals (default: 30 seconds; faster checks cost more).
</details>



🎯 ⭐One liners notes⭐ 🎯
- except for Alias records, TTL is mandatory for each record in the route 53.
- alias records are used to route traffic to AWS resources like ELB, CloudFront, S3, etc. without any additional cost and TTL is not required for alias records.
- you cannot set ALias for EC2 instances, RDS, etc. you can only set alias for AWS resources like ELB, CloudFront, S3, etc.
- 



## Amazon S3 and other storage services

<details>
<summary>🎯Q. What is Amazon S3 and how data gets stored in S3 </summary>

- Amazon S3 (Simple Storage Service) is a scalable, secure, and highly available object storage service designed to store and retrieve any amount of data from anywhere. Data is stored as objects in buckets (containers), each identified by a unique key. S3 supports versioning, encryption, lifecycle policies, and multiple storage classes (e.g., S3 Standard, Glacier) for cost optimization.
- Exam Tips: ⭐
  - Bucket names are globally unique across all AWS accounts and must follow DNS naming rules.
  - S3 is object storage (not block storage like EBS or file storage like EFS).
  - Encryption options: SSE-S3 (AWS-managed keys), SSE-KMS (customer-managed keys), SSE-C (customer-provided keys).
  - Access control: Use bucket policies, ACLs, or IAM policies to manage permissions.
  - Storage classes: Know differences (e.g., S3 Standard for frequent access, Glacier for archival).

Common Mistakes: ⚠️
- Confusing S3 with block/file storage: S3 is for objects (e.g., images, backups), not databases/OS disks.
- Forgetting bucket name uniqueness: Names must be unique globally, not just within your account.
- Ignoring storage class tradeoffs: Glacier has retrieval delays (minutes to hours), while S3 Standard-IA is for infrequent but immediate access.
- Overlooking versioning: ⭐Without versioning, overwritten objects are permanently lost.⭐
</details>


<details>
<summary>🎯Q. Amazon S3 Security  </summary>

- Amazon S3 security ensures data protection through encryption, access controls, and compliance
- Objects are secured using:
  - Encryption: At rest (SSE-S3, SSE-KMS, SSE-C) or in transit (SSL/TLS).
  - Access Policies: Bucket policies, IAM policies, ACLs, and VPC Endpoints.
  - Versioning & MFA Delete: Prevent accidental deletion/modification.
  - Block Public Access: Global settings to enforce "no public access" by default.
</details>


<details>
<summary>🎯Q. Amazon S3 Replication  </summary>

- Amazon S3 Replication automatically copies objects across S3 buckets. (versioning must be enabled) for both of these options)
  - Cross-Region Replication (CRR): Copies objects to a bucket in a different AWS region (e.g., for disaster recovery).
  - Same-Region Replication (SRR): Copies objects within the same region (e.g., for compliance, aggregation, or latency reduction).

Simple end-to-end real-world example: ⭐
- A global e-commerce company uses:
  - CRR to replicate order data from us-east-1 to eu-west-1 for disaster recovery.
  - SRR to aggregate logs from multiple buckets in us-east-1 into a central bucket for analytics.
  - Lifecycle policies to archive logs to S3 Glacier after replication.

Common Mistakes: ⚠️

- After replication is enabled only new objects are replicated, existing objects are not replicated. Optionally you can replicate the existing objects by using the `S3 batch replication` feature.

</details>

<details>
<summary>🎯Q. Amazon S3 storage classes , durability and availibility </summary>

- Different storage classes and their characteristics:
  - `S3 Standard`: 99.99% availability, frequent access. ⭐used for frequently accessed data.
  - `S3 Intelligent-Tiering`: Automatically moves objects between tiers based on usage. ⭐used for unpredictable access patterns.
  - `S3 Standard-IA` (Infrequent Access): 99.9% availability, lower cost for less frequent access. ⭐used for infrequently accessed data.
  - `S3 One Zone-IA`: 99.5% availability, stores data in one AZ (cheaper but less resilient). ⭐used for non-critical data.
  - `S3 Glacier` (Instant, Flexible, Deep Archive): For archival (retrieval times: minutes to hours). ⭐used for archival data.

- Amazon S3 storage classes are cost-optimized tiers for storing objects based on access frequency and retrieval requirements. All classes provide 99.999999999% (11 9s) durability, but availability varies:
   - `S3 Standard`: 99.99% availability (frequent access).
   - `S3 Intelligent-Tiering`: Automatically moves objects between tiers (frequent, infrequent, archive) based on usage.
   - `S3 Standard-IA` (Infrequent Access): 99.9% availability (lower cost for less frequent access).
   - `S3 One Zone-IA`: 99.5% availability (stores data in one AZ; cheaper but less resilient).
   - `S3 Glacier` (Instant, Flexible, Deep Archive): For archival (retrieval times: minutes to hours).

- Simple end-to-end real-world example: ⭐
  - A video streaming platform uses:
    - S3 Standard: Stores newly uploaded videos (frequent access).
    - S3 Intelligent-Tiering: Automatically moves older videos to lower-cost tiers as views decline.
    - S3 Glacier Flexible Retrieval: Archives raw footage (retrievable in 1-5 minutes).
    - S3 One Zone-IA: Stores transcoded logs (non-critical data, cost-sensitive).

Exam Tips: ⭐

- `Durability ≠ Availability`: All classes have 11 9s durability, but availability varies (e.g., One Zone-IA has 99.5% vs. Standard’s 99.99%).
- `Glacier retrieval options`: Instant (ms), Flexible (mins to hrs), Deep Archive (12+ hrs).
- `Intelligent-Tiering avoids` retrieval fees and monitors access patterns.
- `One Zone-IA is riskier`: Data is lost if the AZ fails.
- Lifecycle policies automate transitioning objects between classes.

Common Mistakes: ⚠️

- Assuming all classes have the same availability (e.g., One Zone-IA is cheaper but less available).
- Confusing Glacier retrieval tiers (e.g., using Deep Archive for urgent retrievals).
- Overlooking Intelligent-Tiering for unpredictable access patterns.
- Ignoring cost tradeoffs (e.g., Standard-IA has lower storage costs but higher retrieval fees).

</details>


<details>
<summary>🎯Q. Amazon S3 lifecycle rules - important things to know </summary>

- Amazon S3 lifecycle rules automate transitioning objects between storage classes (e.g., Standard → Glacier) or deleting objects based on age. Key actions:
   - `Transitions`: Move objects to cheaper storage classes (e.g., after 30 days).
   - `Expiration`: Permanently delete objects or delete expired delete markers (for versioned buckets).

Simple end-to-end real-world example: ⭐

- A media streaming service uses lifecycle rules to optimize costs:
- Transition rule: Move videos from S3 Standard to S3 Standard-IA after 30 days (reduced access frequency).
- Archive rule: Transition to S3 Glacier Flexible Retrieval after 90 days (long-term archival).
- Expiration rule: Delete raw footage logs after 365 days (compliance requirements).
- Incomplete upload cleanup: Abort multipart uploads not completed in 7 days.
  
Exam Tips: ⭐

- Minimum age for transitions:
- Standard → Standard-IA/One Zone-IA: 30 days.
- Standard → Glacier/Deep Archive: 90 days (varies by storage class).
- Intelligent-Tiering: No minimum age (immediate tiering).
- Versioning impacts expiration: Expiration deletes non-current versions if configured.
- Expiration vs. Transition: Expiration deletes objects; Transition changes storage class.
- Incomplete uploads: Lifecycle rules can abort uploads stuck for a specified period.


Common Mistakes: ⚠️

- `Ignoring minimum age requirements` (e.g., trying to transition to Glacier before 90 days).
- Confusing expiration with transition (e.g., using expiration to move objects to Glacier).
- `Overlooking versioning`: Expiration rules may not delete all versions unless configured.
- `Assuming lifecycle rules apply retroactively` (rules only affect objects uploaded after creation).
</details>


<details>
<summary>🎯Q.what is S3 requester pays ? who pays by default for access and networking cost ? owner or requester? </summary>

- Amazon `S3 Requester Pays feature` allows bucket owners to charge requesters for data transfer and request costs. 
- By default, bucket owners pay for all costs associated with their S3 buckets (e.g., storage, data transfer, requests). 
- With Requester Pays, the requester pays for data transfer and request costs when accessing the bucket. 
- This feature is useful for sharing data with external users or charging for data access.
- requester must be authenticated by IAM to access the bucket with requester pays enabled.
</details>

<details>
<summary>🎯Q. different methods of object encryption in S3 bucke ?</summary>

- `Server-Side Encryption (SSE)`: Encrypts objects at rest using AWS-managed keys or customer-provided keys.
  - `SSE-S3`: Uses  AES-256 encryption  and AWS-managed keys. - Enabled by default.
  - `SSE-KMS`: Uses AWS Key Management Service
  - `SSE-C`: Uses customer-provided keys (must be managed by the customer).
  - `Encryption in Transit`: Encrypts data in transit using SSL/TLS.
     - encryption in flight is enabled by default in S3, is also called as `SSL/TLS` encryption.
  - DSSE-KMS - DSSE-KMS is just "double encryption based on KMS".
</details>


<details>
<summary>🎯Q. S3 - CORS important things to know </summary>

- CORS is web security feature that restricts cross-origin HTTP requests (e.g., from a different domain).
- preflight request is sent by the browser to check if the server allows the request. (Do not miss Stephan's diagram explanation for this)
- S3 CORS configuration allows you to control which domains can access your S3 resources.
</details>

<details>
<summary>🎯Q. What is S3 object lock ?</summary>

- Amazon S3 Object Lock prevents objects from being deleted or modified for a fixed period or indefinitely. 
- It enforces Write-Once-Read-Many (WORM) compliance. Key features:
    - Retention Modes:
      - `Governance Mode`: Objects can’t be deleted/modified unless users have special permissions.
      - `Compliance Mode`: No one (including root) can delete/modify objects until the retention period expires.
      - `Retention Period`: Fixed duration (e.g., 7 years) set for objects.
      - `Legal Hold`: Blocks deletion/modification indefinitely (no retention period required).

Simple end-to-end real-world example: ⭐
 - A financial institution must comply with SEC regulations:
 - Enable versioning & Object Lock on the bucket.
 - Governance Mode: Apply a 3-year retention period for audit logs (internal admins can override with permissions).
 - Compliance Mode: Apply a 7-year retention for tax records (no override allowed).
 - Legal Hold: Place on specific documents during litigation (indefinite protection).

Exam Tips: ⭐

- ⭐Object Lock requires versioning⭐ to be enabled on the bucket.
- Governance vs. Compliance Mode: Governance allows override with permissions; Compliance does not.
- ⭐Legal Hold works independently of retention periods⭐ (can be applied/removed anytime).
- Use cases: Regulatory compliance (e.g., FINRA, SEC), legal holds, ⭐ransomware protection (write once and never deletes.⭐)
- Retention period starts when the object is created (not when the lock is applied).

Common Mistakes: ⚠️

- Confusing Governance and Compliance Modes: ⭐Assuming root can bypass Compliance Mode (they cannot).⭐
- Forgetting to enable versioning: Object Lock will fail without it.
- Assuming Legal Hold requires a retention period: It works independently. - means you can apply legal hold without any retention period.
  - example : if you are in the middle of the litigation and you want to protect the data from deletion, you can apply legal hold without any retention period. even if retention period is set to 7 years, the object will not be deleted until the legal hold is removed.
- Overlooking retroactive locks: Existing objects aren’t locked automatically unless configured.

</details>


<details>
<summary>🎯Q. AWS Snow family important notes  </summary>

- The AWS Snow Family includes physical devices ⭐(Snowcone, Snowball Edge, Snowmobile)⭐ designed for secure, offline data transfer and edge computing in environments with limited connectivity or massive data volumes.

Real-World Example: ⭐
- A media company with 500 TB of raw footage in a remote location uses Snowball Edge (Storage Optimized) to transfer data to AWS. They process some data on-device using EC2 instances (edge computing) before shipping. For a 100 PB data lake migration, they use Snowmobile (truck-sized storage).

Exam Tips: ⭐

- Device Selection:
    - `Snowcone`: Smallest (8 TB), ideal for edge computing/light data transfer.
    - `Snowball Edge`: Larger (80 TB) with compute/storage options (Compute Optimized vs. Storage Optimized).
    - `Snowmobile`: Exabyte-scale (100 PB+).
Use Cases:
    - Offline data transfer (slow/no internet).
    - Edge computing (IoT, ML inference, preprocessing).
    - Encryption: All data is automatically encrypted with AWS KMS.
    - Data Transfer: Jobs are tracked via AWS OpsHub (GUI) or CLI.

Common Mistakes: ⚠️

- Confusing Snowball Edge with Storage Gateway:
    - Snowball = one-time transfer; Storage Gateway = ongoing hybrid storage.
- Assuming online transfer (e.g., DataSync) is always better:
    - Snowball is cheaper/faster for 10+ TB.
- Ignoring edge computing capabilities:
    - ⭐Snowball Edge can run EC2 instances or Lambda functions.⭐


</details>

<details>
<summary>🎯Q. what is AWS FSx ? what are different services/product types ? </summary>

- AWS FSx is a fully managed file storage service that simplifies deploying and managing high-performance file systems. 
- It supports multiple file system types optimized for specific workloads:
    - FSx for Windows File Server: SMB-based, integrates with Active Directory, ideal for Windows workloads.
    - FSx for Lustre: Designed for HPC, machine learning, and analytics, optimized for fast processing of large datasets.
    - FSx for NetApp ONTAP: Enterprise-grade NAS storage with features like snapshots, replication, and multi-protocol support.
    - FSx for OpenZFS: Fully managed OpenZFS file system for Linux-based applications requiring POSIX compliance.

Common Mistakes: ⚠️

- Confusing EFS and FSx: Using EFS for Windows workloads (EFS is Linux/NFS-only).
- Overlooking Lustre’s temporary nature: Assuming Lustre is ideal for long-term storage (it’s optimized for compute-heavy, short-term tasks).
</details>


<details>
<summary>🎯Q. AWS Gateway </summary>

- AWS Storage Gateway is a hybrid cloud storage service that connects on-premises environments with AWS Cloud storage. 
- It provides `seamless integration between on-premises applications and AWS storage services like Amazon S3, Amazon S3 Glacier,` Amazon EBS, and AWS Backup. 
- It `supports three types of gateways: File Gateway, Volume Gateway, and Tape Gateway.`

Exam Tips: ⭐

- Understand the three types of gateways:
    - `File Gateway`: For storing files as objects in S3.
    - `Volume Gateway`: For block storage, either cached (primary data in S3, frequently accessed data locally) or stored (entire dataset on-premises, asynchronously backed up to AWS).
    - `Tape Gateway`: For virtual tape libraries, used for backup and archival.
    - Storage Gateway is ideal for hybrid cloud scenarios, enabling on-premises applications to use AWS storage.
      It supports data encryption in transit (SSL/TLS) and at rest (AWS KMS).
</details>

<details>
<summary>🎯Q. Difference AWS Transfer Family, AWS Storage Gateway and DataSync </summary>

- `AWS Transfer Family`: Focus on its use for ⭐external file transfers⭐ using SFTP, FTPS, or FTP.
- `AWS Storage Gateway`: Understand its hybrid cloud nature and the three types (File, Volume, Tape).
- `AWS DataSync`: Remember it’s ⭐optimized for fast, automated data transfers⭐ between on-premises and AWS storage.
- Use Transfer Family for external file sharing, Storage Gateway for hybrid cloud integration, and DataSync for large-scale data migration.

Common Mistakes: ⚠️

- Confusing AWS Transfer Family with AWS DataSync. Transfer Family is for external file transfers, while DataSync is for large-scale data migration.
- Overlooking the hybrid cloud capabilities of AWS Storage Gateway. It’s not just for cloud storage but bridges on-premises and AWS.
</details>

🎯 ⭐One liners notes⭐ 🎯
- Explicit DENY in an IAM Policy will take precedence over an S3 bucket policy.
- S3 batch operations : used for bulk operations like copying objects, deleting objects, etc.
- S3 lens - used for analyzing the S3 bucket and its objects.
- ⭐`byte range fetches`⭐ - used to download a specific range of bytes from the object.
- `MFA delete` - used to enable the MFA delete for the versioned bucket to prevent accidental deletion of the objects.
- Glacier volt - used to retrieve the data from the glacier vault based on WOMP(Write Once, read Many times) policy.
- `S3 object lock` - used to lock the object for a specific period of time to prevent accidental deletion or modification based on WOMP(write once, many times) policy.
- With SSE-KMS, the encryption happens in AWS, and the encryption keys are managed by AWS but you have full control over the rotation policy of the encryption key. Encryption keys stored in AWS.
- MFA Delete forces users to use MFA codes before deleting S3 objects. It's an extra level of security to prevent accidental deletions.
- legal hold - used to protect the object from deletion or modification ⭐indefinitely⭐. even if the retention period is set to 7 years, the object will not be deleted until the legal hold is removed.
- AWS Transfer Family: Used to transfer files to AWS storage services like S3, EFS, and FSx for Windows File Server.
- `AWS Storage Gateway`: Used to connect on-premises environments with AWS Cloud storage, enabling hybrid cloud storage solutions.
- Data-Sync : move large amounts of data between on-premises storage and AWS storage services like S3, EFS, and FSx.
- Snowball Edge `comes with computing capabilities and allows you to pre-process the data while it's being moved into Snowball`.


## CloudFront and AWS Global Accelerator

<details>
<summary>🎯Q. what is CloudFront ?</summary>

- AWS Cloudfront is Content Delivery Network (CDN) service that accelerates the delivery of web content to users worldwide.
-  It improves the read performance and content is cached at the edge locations.
- "Edge Location" : this is the location where the content will be cached. this is separate to an AWS Region/AZ.
- cloudfront origin can be S3 bucket, EC2 instance, ELB, or Route 53.

Simple end-to-end real-world example: ⭐
- A company hosts a website with product images stored in an S3 bucket in us-east-1. Users in Europe experience slow load times. By deploying CloudFront:
- For dynamic content (e.g., personalized recommendations), Lambda@Edge modifies responses at the edge.

Exam Tips: ⭐

- `Edge vs. Regional Caches`: Edge locations cache content globally; regional caches (e.g., S3) are per-region.
    - regional cache means the cache is stored in the specific region only. reginal cache is achieved using the cloudfront regional edge cache.
- `Origin Types`: Supports S3, ALB, EC2, on-premises, or custom HTTP servers.
- `TTL & Cache Behaviors`: Control caching duration and path-based routing (e.g., /images/* cached longer than /api/*).
- `Security`: SSL/TLS via ACM, OAI (Origin Access Identity) for secure S3 access, WAF/Shield for DDoS.
- `Lambda@Edge`: Customize content at edge (e.g., A/B testing, header modification).
    - Lambda@Edge is used to modify the request or response at the edge locations itself
    - simple real scenario :  You have a website that serves different content based on the user's country. You want to use Lambda@Edge to modify the response headers to include a custom header indicating the user's country.
- `Cost`: Data transfer out charges apply; invalidations are paid.
- `Geo-Restriction`: Allow/block countries via whitelist/blacklist.

Common Mistakes: ⚠️

- `Assuming all content is cached`: Dynamic content (e.g., APIs) requires explicit caching configuration.
- `Long TTLs without versioning`: Leads to stale content; use image_v2.jpg instead of invalidating.
- `Confusing CloudFront with S3 Cross-Region Replication`: CloudFront caches; replication duplicates data.
- `Overlooking HTTPS redirects`: To ensure CloudFront enforces HTTPS and avoids mixed-content issues, you can configure a CloudFront distribution to redirect HTTP requests to HTTPS.

</details>

<details>
<summary>🎯Q. AWS global accelerator notes </summary>

- AWS Global Accelerator is a networking service that improves the availability and performance of applications by directing traffic through AWS’s global network infrastructure. 
- It ⭐uses static anycast IP addresses⭐ to route user requests to the nearest edge location, then optimally routes traffic to healthy endpoints (e.g., EC2, ALB, NLB) across AWS regions. 
- Key features include `built-in health checks`, automatic failover, and DDoS protection via AWS Shield.

Simple end-to-end real-world example for better visualization: ⭐
A global e-commerce app hosts backend servers in us-east-1 (N. Virginia) and ap-southeast-1 (Singapore). Without Global Accelerator, users in Europe might connect directly to the Singapore region, causing high latency.
With Global Accelerator:

- Users in Europe hit the nearest AWS edge location (e.g., Frankfurt).
- Traffic is routed over AWS’s backbone (faster than public internet) to the optimal healthy endpoint (e.g., us-east-1 if Singapore is overloaded).
- If the Singapore ALB fails health checks, traffic automatically shifts to the Virginia ALB, ensuring minimal downtime.

Exam Tips: ⭐

Global Accelerator vs. CloudFront:
- Use Global Accelerator for TCP/UDP traffic, non-HTTP use cases (e.g., gaming, IoT), or apps needing static IPs.
- Use CloudFront for HTTP/HTTPS content caching (static/dynamic) and latency reduction for web apps.
- `Static Anycast IPs`: Provides two static IPs from different AWS regions. Ideal for whitelisting scenarios (e.g., APIs, on-premises firewall rules).
- `Health Checks`: Monitors endpoint health (e.g., ALB, EC2) and reroutes traffic if unhealthy. Faster failover than Route 53.
- `Performance`: ⭐Uses AWS backbone network⭐, reducing jitter and latency compared to public internet routing.
- `Integration`: Works with Elastic IPs, EC2, ALB, NLB, and supports cross-region endpoints.


Common Mistakes: ⚠️

- Confusing Global Accelerator with CloudFront (e.g., using Accelerator for caching static content).
- ⭐Assuming it requires DNS changes⭐ (it uses IPs; Route 53 manages DNS). means you don't have to change the DNS records to use the global accelerator you just have to use the static anycast IP address provided by the global accelerator.
- Overlooking that it does not cache content (unlike CloudFront).
- `Forgetting it’s global`, not region-specific. You don’t deploy it per region.
- `Ignoring the cost impact`: Global Accelerator charges for data transfer and hourly usage (unlike CloudFront’s per-request model).
</details>


<details>
<summary>🎯Q. Difference AWS Global Accelerator vs CloudFront </summary>

- `AWS Global Accelerator optimizes traffic routing for TCP/UDP applications using static IPs and AWS’s backbone network`, focusing on availability and performance for non-HTTP use cases.
- `Amazon CloudFront is a CDN` (Content Delivery Network) that caches static/dynamic HTTP/HTTPS content at edge locations, reducing latency for end users.

Simple end-to-end real-world example: ⭐

- `Global Accelerator`: A multiplayer gaming app uses UDP for real-time communication. Global Accelerator routes traffic from players in Asia to the nearest healthy game server in ap-northeast-1 (Tokyo) via AWS’s backbone, ensuring low latency and failover during outages.
- `CloudFront`: A news website caches images and articles at edge locations. A user in Paris retrieves content from the Frankfurt edge location instead of the origin server in us-east-1, reducing load times.

Exam Tips: ⭐

⭐Protocols & Use Cases:⭐
- Global Accelerator: Ideal for TCP/UDP (e.g., gaming, IoT, APIs) and apps needing static IPs.
- CloudFront: Designed for HTTP/HTTPS (e.g., websites, video streaming).

⭐Caching:⭐
- CloudFront caches content at edge locations
- Global Accelerator does not cache (only optimizes routing). IP Addresses:

⭐IP Addresses:⭐
- Global Accelerator `provides static anycast IPs`.
- CloudFront uses `domain-specific URLs` (e.g., d123.cloudfront.net).

⭐Failover Speed:⭐
- Global Accelerator reroutes traffic in seconds using health checks. 
- CloudFront relies on TTL-based DNS failover (slower).

Common Mistakes: ⚠️

- Using CloudFront for UDP traffic (it only supports HTTP/HTTPS).
- Assuming Global Accelerator caches content (it only accelerates routing).
- Overlooking cost differences:
    - CloudFront charges per request/data transfer out.
    - Global Accelerator charges hourly + data processing fees.
- Confusing edge locations:
    - CloudFront caches content at edges.
    - `Global Accelerator uses edges **only for traffic entry, not storage.**`
</details>



## SQS SNS
Decople using - SQA - Queue model, SNS - Pub-Sub, Kinesis - Streaming

<details>
<summary>🎯Q. Difference SQS VS SNS VS Kinesis ? </summary>

Definition: ⭐

- `SQS`: Simple Queue Service for decoupled, scalable message queuing ⭐(pull-based).⭐
- `SNS`: Simple Notification Service for pub/sub messaging ⭐(push-based).⭐
- `Kinesis`: ⭐Real-time streaming⭐ for large-scale data with ordered, replayable records.
   - `Kinesis Data Streams`: For real-time data ingestion and processing.
   - `Kinesis Data Firehose`: ⭐⭐⭐⭐⭐ For data delivery to S3, Redshift, Elasticsearch, etc. ⭐⭐⭐⭐⭐
   - `Kinesis Data Analytics`: For real-time analytics on streaming data.

⭐Simple Real-World Example:⭐
Imagine an e-commerce app:

- `SQS`: Processes orders one-by-one (e.g., payment service pulls orders from a queue).
- `SNS`: Sends order confirmation emails/SMS (1 event → multiple subscribers).
- `Kinesis`: Analyzes real-time user clickstreams (e.g., tracking page views for live dashboards).

Exam Tips: ⭐

- SQS:
    - Polling (pull), retention (14 days), DLQ for failed messages.
    - Use for async processing ⭐with 1 consumer per message.⭐
    - supports long polling, short polling, and visibility timeout(means how long the message will be invisible to other consumers after it is consumed by one consumer).
- SNS:
    - Push to multiple subscribers (SQS, Lambda, HTTP).
    - `Fanout pattern`: 1 SNS topic → multiple SQS queues.
    - `No Persistence`: Messages are not stored; use SQS Subscribers for durability.
- Kinesis:
    - Shards define capacity; ordered records, replayable for 24h-365 days.
    - `Use for real-time analytics` (e.g., clickstreams, logs).
    - Data Retention: Default 24h, extendable to 365 days (extra cost)

Common Mistakes: ⚠️

- Using SQS for broadcasting (use SNS + SQS fanout instead).
- Assuming SNS stores messages (no retention; pair with SQS for durability).
- `Confusing Kinesis with SNS/SQS`: Kinesis handles high-throughput streams, not simple notifications.


⭐Key Differentiators for Exam Scenarios:⭐

- Order Matters?
  - Yes → Kinesis or SQS FIFO.
  - No → SQS Standard.

- Multiple Consumers?
  - Same Data: Kinesis (multiple apps read independently).
  - Different Data: SNS (fanout to SQS/Lambda).

- Real-Time vs. Batched:
  - Real-Time: Kinesis.
  - Async Batched: SQS.
  - 
</details>

<details>
<summary>🎯Q. Amazon MQ </summary>

- Amazon MQ is a managed message broker service that supports industry-standard protocols (MQTT, AMQP, STOMP, OpenWire) for legacy applications. It’s ideal for migrating existing messaging systems to the cloud without rewriting code.

- Simple Real-World Example:
  - A financial institution migrates its on-premises stock trading app (using Apache ActiveMQ/JMS) to AWS. They use Amazon MQ to retain compatibility with JMS APIs, allowing their app to send/receive orders via queues/topics without code changes.

Exam Tips: ⭐

- Use Amazon MQ when migrating legacy apps reliant on MQ protocols (e.g., JMS, AMQP).
- Supports ActiveMQ and RabbitMQ engines (managed brokers).
- Integrates with on-premises systems via VPN/Direct Connect (hybrid architecture).
- High availability: Deploys brokers in Multi-AZ mode for failover.
- Not serverless: You provision brokers (unlike SQS/SNS).


Common Mistakes: ⚠️

- Choosing Amazon MQ for new cloud-native apps (use SQS/SNS/Kinesis instead).
- Assuming it’s fully serverless (requires broker provisioning, unlike SQS).
- Forgetting VPC setup: Amazon MQ runs in your VPC for security.

</details>

⭐One liners notes⭐
- SQS, SNS, and Kinesis are all regional services, meaning they are available across all AZs within a specific region.
- Amazon MQ is for legacy protocol compatibility; SQS/SNS are for cloud-native apps.




## ECS EKS

<details>
<summary>🎯Q.what is ECS (Elastic Compute Service) and important notes. </summary>

- Amazon ECS (Elastic Container Service) is a fully managed container orchestration service that simplifies deploying, managing, and scaling Docker containers. 
- It supports two launch modes: EC2 (manage your own EC2 instances) and Fargate (serverless, no infrastructure management). 
- ECS auto-scaling adjusts tasks or instances based on demand, and integrates with AWS services like ALB, CloudWatch, and IAM.

Real-World Example: ⭐
A company runs a microservices-based e-commerce app:

- Frontend service (Fargate): Serverless containers scale automatically during peak traffic.
- Backend service (EC2 mode): Runs on reserved EC2 instances for cost savings, with cluster auto-scaling.
- Load balancer: ALB routes traffic to services.
- Auto Scaling Service-level scaling for tasks (CPU utilization) + EC2 instance scaling for the cluster.
  means : if the CPU utilization of the task is high, then the ECS will automatically scale the task to handle the load.


⭐Exam Tips: ⭐

- `Fargate vs EC2`:
    - Fargate: No EC2 management, pay per task (vCPU/memory), ideal for unpredictable workloads.
    - EC2 mode: Control over instances, cheaper for steady workloads, requires ASG for scaling.
  
- `Auto Scaling`:
    - `Service Auto Scaling`: Scales tasks (horizontal) based on CloudWatch metrics.
    - `Cluster Auto Scaling (EC2 only)`: Scales EC2 instances via ASG.

- `Task Definitions`: Blueprint for containers (image, CPU/memory, networking).
    - `Networking`: Fargate requires VPC (`each task gets an ENI`); EC2 uses host/awsvpc mode.
    - `IAM Roles`: Assign roles at the task level (not EC2 instance role).


⚠️Common Mistakes: ⚠️

- ⭐Confusing service auto-scaling (tasks) with cluster auto-scaling (EC2 instances)⭐.
- Forgetting Fargate tasks require VPC configuration (subnets, security groups).
- Using EC2 instance roles instead of task roles for container permissions.
- `Overprovisioning CPU/memory in Fargate` (billed for allocated resources, not usage).

</details>

<details>
<summary>🎯Q. ECR, EKS, AppRunner notes</summary>

- Amazon ECR: Private Docker container registry for storing, managing, and deploying container images.
- Amazon EKS: Managed Kubernetes service for running containerized applications using Kubernetes.
- AWS App Runner: Fully managed service to deploy containerized web apps/APIs without infrastructure management.
- ECS: Elastic Container Service for running containers with AWS-native orchestration. Supports Fargate (serverless) and EC2 (self-managed instances) launch modes.


Real-World Example: ⭐
A company builds a microservices-based e-commerce app:

- ECR: Store Docker images for product, cart, and payment services.
- EKS: Orchestrate Kubernetes clusters for scalable, complex microservices with custom networking.
- App Runner: Deploy the frontend (React) container with automatic scaling and HTTPS.
- ECS (Fargate): Run backend APIs with serverless containers, auto-scaled based on CPU usage.


Exam Tips: ⭐

ECR vs EKS vs App Runner:
- Use ECR for storing container images (like Docker Hub for AWS).
- Use EKS for Kubernetes-specific workloads (e.g., Helm charts, custom scaling).
- Use App Runner for simple, fully managed deployments (CI/CD integrated).

ECS Auto Scaling:
- Target Tracking: Scale ECS tasks based on CloudWatch metrics (CPU/Memory).
- Scheduled Scaling: Predictable traffic patterns (e.g., peak hours).

Fargate vs EC2:
- Fargate = no EC2 management (serverless), higher cost.
- EC2 mode = control over instances, cheaper for large workloads.


Common Mistakes: ⚠️

- Confusing ECS (AWS-native) with EKS (Kubernetes).
- Assuming App Runner supports non-containerized apps (it’s only for containers).
- Overlooking Fargate cost implications for long-running, high-traffic apps.
   - because fargate cost is based on the vCPU and memory allocated to the task, so if you allocate more vCPU and memory to the task, then you will be charged more.
- Forgetting that ECR requires IAM policies (not resource-based policies) for access control.

</details>

## Lambda

<details>
<summary>🎯Q. what is lambda? lambda@edge? </summary>

- AWS Lambda is serverless compute for running code without managing servers. Lambda@Edge allows running Lambda functions at CloudFront edge locations for low-latency responses. Concurrency refers to the number of Lambda executions running simultaneously (default: 1,000/account/region).

Simple real-world example: ⭐
Imagine an e-commerce website:

- Lambda: Processes orders (backend API) when a user checks out.
- Lambda@Edge: Dynamically resizes product images based on the user’s device type (e.g., mobile vs desktop) before the image is cached by CloudFront.
- Concurrency: During Black Friday, Lambda scales to 500 concurrent executions to handle traffic spikes.

Common Mistakes: ⚠️

- Thinking Lambda@Edge functions can access VPC resources (they can’t).
- Forgetting Lambda@Edge has shorter timeouts (e.g., 5 seconds for viewer request events).
- Ignoring cold starts (initial latency) for time-sensitive applications.
- Not setting reserved concurrency for mission-critical functions, leading to resource starvation.

</details>


<details>
<summary>🎯Q. What is Lambda cold start and important points to know for solution architecture? </summary>

- A Lambda cold start is the latency introduced when AWS Lambda initializes a new execution environment (runtime, code, dependencies) for a function that hasn’t been recently used. This occurs during the first invocation or after periods of inactivity.

Simple real-world example: ⭐
A ride-sharing app uses Lambda for fare calculation:
- Cold Start: A user opens the app at 5 AM (low traffic). Lambda must initialize a new instance, causing a 1-2 second delay.
- Warm Start: Subsequent requests reuse the initialized instance, responding in milliseconds.


Exam Tips: ⭐

- Factors increasing cold starts:
    - Runtimes: Java/.NET have longer cold starts vs. Python/Node.js.
    - Package size: Larger deployment packages (e.g., 250MB+) slow initialization. 
    - VPCs: Lambda in a VPC adds ENI setup time (~10+ seconds).

- Mitigation strategies:
    - Use Provisioned Concurrency (pre-warms instances).
    - Minimize dependencies & use layers for reusable code.
    - `Avoid VPCs unless necessary`.
    - `Use SnapStart (for Java functions)` to cache runtime state.

Common Mistakes: ⚠️

- Assuming all runtimes have the same cold start duration (Java vs. Python).
- Overlooking Provisioned Concurrency for latency-sensitive apps (e.g., APIs).
- Deploying Lambda in a VPC for non-critical use cases, increasing cold starts.
- Ignoring initialization code optimization (e.g., moving heavy setup outside the handler).
   meaning : if you have some heavy initialization code that is not required to be executed every time the lambda function is invoked, then you can move that code outside the handler so that it will be executed only once when the lambda function is initialized.

</details>

## DynamoDB

<details>
<summary>🎯Q. Dynamo DB important notes </summary>

- Amazon DynamoDB is a fully managed, serverless NoSQL database service (key-value & document) offering ⭐single-digit millisecond performance⭐, ⭐automatic scaling⭐, ⭐built-in high availability with multi-AZ replication⭐, and features like global tables, TTL, and streams. ⭐It's ideal for scalable, low-latency applications.⭐

⭐Real-World Example: ⭐
An e-commerce platform uses DynamoDB to:

- Store product catalogs (partition key = ProductID) and user sessions (TTL deletes expired sessions).
- Use DAX to cache frequently viewed product pages, reducing read latency.
- `Enable global tables for active-active replication` across regions (US/EU) for low-latency access.
   active-active replication means the data is replicated across multiple regions and the data can be read and written from any region.
- `Process real-time order updates via DynamoDB Streams`, triggering Lambda to update inventory.
- Restore accidentally deleted data using `point-in-time recovery`.
  how this is achieved ? : point-in-time recovery is achieved by enabling the point-in-time recovery feature in the dynamoDB table, this feature allows you to restore the data to any point in time within the retention period.


Exam Tips: ⭐

- `DAX vs ElastiCache`: DAX(DynamoDB Accelerator) is purpose-built for DynamoDB (caches query/results), while ElastiCache (Redis/Memcached) is general-purpose (caches app data, API responses).
- `Global Tables`: Require DynamoDB Streams enabled for cross-region replication (active-active).
- `TTL`: Automatically deletes expired items (e.g., sessions, logs) to reduce costs.
- `Stream Processing`: Triggers Lambda for real-time analytics/auditing (e.g., fraud detection).
- `Point-in-Time Recovery`: Restores table to any point in the last 35 days (protects against accidental writes/deletes).
- Scan vs Query:
    - Query: Efficient (uses partition/sort keys).
    - Scan: Inefficient (reads entire table) – avoid in production.
- Analytics Integration:
    - Use DynamoDB Accelerator (DAX) for read-heavy apps.
    - `For complex analytics, export data to Amazon Redshift/Athena via S3.`
- Billing: On-demand vs. provisioned capacity (provisioned requires RCU/WCU planning).
- DAX: Use for read-heavy apps needing microsecond latency (caches frequent queries).
- Streams: Capture real-time data changes (e.g., trigger Lambda for order processing).

Common Mistakes: ⚠️

- Using ElastiCache instead of DAX for DynamoDB-specific caching.
- Forgetting to enable DynamoDB Streams for global tables.
   means : if you want to use the global tables feature in the dynamoDB, then you have to enable the dynamoDB streams in the table ⭐because the global tables feature uses the dynamoDB streams to replicate the data across the regions⭐.
- Assuming TTL deletes data instantly (can take up to 48 hours).
- Over-provisioning RCU/WCU instead of enabling auto-scaling.
- Ignoring partition key design, leading to "hot partitions" and throttling.

</details>

## API Gateway

<details>
<summary>🎯Q. what is API gateway and important notes </summary>

- Amazon API Gateway is a fully managed service for creating, deploying, and managing RESTful and `WebSocket APIs`.
- It acts as a "front door" for applications to access data, business logic, or functionality from backend services (e.g., Lambda, EC2, HTTP endpoints).

Exam Tips: ⭐

- `Endpoint Types`:
    - `Edge-optimized`: Uses CloudFront globally. Best for public APIs with geographically distributed users.
    - `Regional`: Serves requests from a single region. Use for APIs where clients are in the same region.
    - `Private`: Accessible only within a VPC via VPC Endpoint (Interface type).
- `Caching`: API Gateway can cache responses to reduce backend load.
- `Security`: Integrates with IAM, Cognito, or custom authorizers for authentication/authorization.
- `Throttling`: Limits requests per second to prevent abuse.
- `Monitoring`: CloudWatch metrics/logs, X-Ray tracing, and usage plans for API keys.


</details>

## Step Functions

<details>
<summary>🎯Q. Template 1 </summary>

- API Step Functions refers to using AWS Step Functions ⭐(a serverless orchestration service)⭐ to design, automate, and coordinate multi-step workflows involving AWS services, APIs, and custom logic. 
- These workflows (state machines) handle complex processes like retries, error handling, parallel execution, and human approval steps.


Simple Real-World Example: ⭐
Use Case: E-commerce Order Processing

- Start → A customer places an order via an API Gateway.
- Check Inventory → Step Functions invokes a Lambda function to verify product availability in DynamoDB.
- Process Payment → Call an external payment API (e.g., Stripe) via Lambda.
- Parallel Execution →
  - Update order status in DynamoDB.
  - Send an order confirmation email via SNS.
- Handle Errors → If payment fails, trigger a Lambda to notify the customer via SQS.
- End → Order workflow completes.


Exam Tips: ⭐

- `Step Functions vs. Lambda`: Use Step Functions for multi-step workflows; Lambda for single-task functions.
- `State Types`: Know key states (Task, Choice, Parallel, Wait, Fail) and their use cases.
- `Error Handling`: Use Retry and Catch blocks to manage service failures (e.g., Lambda throttling).

Execution Types:
- `Standard Workflow`: Long-running (up to 1 year), auditable (e.g., order processing).
- `Express Workflow`: Short-lived (5 mins), high-volume (e.g., IoT data processing).


Common Mistakes: ⚠️

- `Direct Service Calls`: Assuming Step Functions can call all AWS services directly (e.g., DynamoDB requires a Lambda proxy).
- `Execution Time`: Using Standard Workflow for high-frequency, short tasks (use Express instead).
- `State Machine Limits`: Forgetting max execution duration (1 year for Standard, 5 mins for Express).
</details>

## Cognito

<details>
<summary>🎯Q. cognito notes </summary>

- Amazon Cognito is an AWS service that provides user authentication, authorization, and management for web/mobile apps. It has two main components:
    - `User Pools`: Secure user directories for sign-up/sign-in (authentication).
    - `Identity Pools`: Grant temporary AWS credentials to access AWS services (authorization).

Simple real-world example: ⭐
A travel app lets users sign up with email or Google.
- `User Pool` verifies credentials (e.g., email/password or Google token) and returns a JWT.
- `Identity Pool` uses the JWT to grant temporary AWS credentials, allowing the app to upload trip photos to an S3 bucket.

Exam Tips: ⭐

- `User Pools ≠ Identity Pools`: User Pools handle authentication (identity verification), while Identity Pools handle authorization (AWS resource access).
- `Federation`: Cognito supports social (Google/Facebook) and enterprise (SAML/OIDC) identity providers.
- `MFA & Security`: User Pools enforce MFA, password policies, and account recovery.
- Cognito Sync is deprecated; use AppSync for syncing user data across devices.

Common Mistakes: ⚠️

- Assuming User Pools alone grant AWS access (they don’t; use Identity Pools for credentials).
- Forgetting to configure IAM roles for Identity Pools, leading to permission errors.
- `Confusing Cognito with IAM roles for EC2/Lambda` (Cognito is for end-users, IAM roles for AWS services).

</details>


⭐One liners notes⭐
- DynamoDB Accelerator (DAX) is a fully managed, highly available, in-memory cache for DynamoDB that delivers up to 10x performance improvement. It caches the most frequently used data, thus offloading the heavy reads on hot keys off your DynamoDB table, hence preventing the "ProvisionedThroughputExceededException" exception.
- DynamoDB Streams allows you to capture a ⭐time-ordered sequence of item-level modifications in a DynamoDB table⭐. It's integrated with AWS Lambda so that you create triggers that automatically respond to events in real-time.
- An Edge-Optimized API Gateway is best for geographically distributed clients. API requests are routed to the nearest CloudFront Edge Location which improves latency. The API Gateway still lives in one AWS Region.
- Security groups are used exclusively for VPC-based resources – ⭐primarily EC2 instances (and also for other services like RDS, ELB, Lambda, etc. ⭐when they run inside a VPC⭐).⭐




## Databases in AWS and analytics

<details>
<summary>🎯Q. Database types and use cases</summary>

- `Relational (RDS, Aurora)`: Structured data with ACID transactions (e.g., customer records).
- `NoSQL (DynamoDB)`: Flexible schemas, high scalability (e.g., user sessions).
- `In-Memory (ElastiCache)`: Low-latency caching (e.g., real-time leaderboards).
- `Document (DocumentDB)`: JSON-like data (e.g., product catalogs).
- `Graph (Neptune)`: Relationships (e.g., fraud detection).
- `Time-Series (Timestream)`: Time-stamped data (e.g., IoT sensors).
- `Warehouse (Redshift)`: Analytics (e.g., business reports).

Simple Real-World Example: ⭐
An e-commerce platform uses:

- RDS (MySQL): Orders, payments (ACID compliance).
- DynamoDB: Product inventory (scales for flash sales).
- Redshift: Sales trend analysis (petabyte-scale queries).
- ElastiCache (Redis): Carts and recommendations (sub-millisecond latency).
- Neptune: "Customers who bought this also bought..." (graph relationships).
- Timestream: Monitor user activity timestamps (clickstream analytics).

Exam Tips: ⭐

- Key Scenarios:
    - `DynamoDB`: Serverless apps, high throughput, low-latency (e.g., gaming leaderboards)., single digit miliseconds latency.
    - `Redshift vs. RDS`: Redshift for analytics (OLAP), RDS for transactions (OLTP).
    - `Aurora`: High availability (⭐6 copies across AZs⭐), auto-scaling.
    - `ElastiCache`: Use when latency is critical (session stores, real-time dashboards).
    - `Timestream`: IoT, DevOps monitoring (time-ordered data).


Common Mistakes: ⚠️

- Using RDS for unstructured data → Use DynamoDB or DocumentDB.
- Confusing Redshift (analytics) with RDS (transactions).
- Ignoring caching → Results in unnecessary database load/high latency.
- Choosing DynamoDB for complex joins → Relational DBs are better here.
- Overlooking Neptune for relationship-heavy data (e.g., social networks).
</details>


<details>
<summary>🎯Q. DocumentDB </summary>

- Amazon DocumentDB is a fully managed, MongoDB-compatible database service designed for storing, querying, and processing JSON-based document workloads. It separates compute and storage layers (like Aurora) for scalability, high availability, and automatic storage scaling.

Simple real-world example: ⭐
- A social media app stores user profiles with dynamic attributes (e.g., interests, friends, posts). Each profile is a JSON document with nested structures. Using DocumentDB:
- Schema flexibility: Add new fields (e.g., "recent_activity") without downtime.
- Complex queries: Use MongoDB syntax to find users in a specific city with interests in "travel."
- Scalability: Auto-scaling storage and read replicas handle spikes during viral posts.


Exam Tips: ⭐

- MongoDB compatibility: Supports MongoDB 3.6/4.0 APIs—ideal for migrating MongoDB apps to AWS.
- Use cases: JSON documents, unstructured/semi-structured data, apps needing aggregation pipelines.
- Storage: Auto-scaling (starts at 10 GB, grows in 10 GB increments) with 6 replicas across AZs.
- High availability: Multi-AZ with up to 15 read replicas; failover <30 sec.


Common Mistakes: ⚠️

- Confusing with DynamoDB: DynamoDB is key-value/document (limited query flexibility), DocumentDB excels at complex queries.
- Assuming full MongoDB parity: Some MongoDB features (e.g., transactions, $lookup) may have limitations.
- Ignoring compatibility versions: If the question mentions MongoDB 5.0+, DocumentDB is not the answer.
- ⭐DocumentDB doesn't have a Serverless option and it doesn't have a global database feature.⭐

</details>

<details>
<summary>🎯Q. Amazon Neptune </summary>

- Amazon Neptune is a fully managed graph database service optimized for storing and querying highly connected data using RDF (Resource Description Framework) and Property Graph models. It supports query languages like Gremlin (for property graphs) and SPARQL (for RDF), enabling efficient traversal of complex relationships.

Exam Tips: ⭐

- Use Neptune for relationship-heavy data: Fraud detection, social networks, recommendation engines, knowledge graphs.
- Query languages: Gremlin (property graph) vs. SPARQL (RDF) – know which to use based on the data model.
- `High availability`: Neptune replicates data to 3 AZs and supports up to 15 read replicas.
- Not a replacement: Avoid using it for simple key-value (DynamoDB) or tabular data (RDS).
- Security: IAM integration, encryption at rest (AWS KMS), and VPC isolation are key features.


Common Mistakes: ⚠️

- Confusing Neptune with other databases: Using Neptune for non-graph use cases (e.g., transactional apps needing SQL).
- Overlooking scaling: ⭐Neptune scales vertically (instance size) and horizontally (read replicas), but not automatically like DynamoDB.⭐
- Forgetting backups: Automated backups are enabled by default (retention: 1-35 days), but cross-region backups require manual setup.

</details>


<details>
<summary>🎯Q. Amazon Keyspaces (for apache cassandra) </summary>

- Amazon Keyspaces (for Apache Cassandra) is a managed, serverless, wide-column NoSQL database service compatible with Apache Cassandra. It handles scaling, replication, and maintenance, ideal for apps needing high scalability and Cassandra-like data models (e.g., time-series, IoT).

Simple real-world example: ⭐
- A logistics company tracks real-time GPS data from 50,000 delivery trucks. Each truck sends location updates every 10 seconds. Using Amazon Keyspaces:
- Data is stored in a wide-column format (e.g., vehicle_id as partition key, timestamp as clustering column).
- Serverless scaling handles fluctuating write/read traffic.
- Cassandra Query Language (CQL) is used for queries like "Get last 10 locations for truck-X."
- No infrastructure management: AWS handles backups, patches, and replication across 3 AZs.


Exam Tips: ⭐

- Serverless & Managed: No provisioning; pay-per-use pricing.
- Cassandra Compatibility: Uses CQL (not SQL) and Cassandra drivers.
- Use Cases: Time-series, high-scale apps (IoT, clickstreams).
- VS DocumentDB:
     - Keyspaces: Wide-column (rows with dynamic columns).
     - DocumentDB: Document-based (JSON-like, hierarchical data).


Common Mistakes: ⚠️

- Confusing with DynamoDB/DocumentDB: ⭐Keyspaces ≠ key-value (DynamoDB) or document DB (DocumentDB).⭐
- Ignoring CQL: Writing SQL queries (e.g., SELECT * FROM table WHERE non-key column=...) will fail.
- Overlooking Partition Keys: Poorly designed partition keys lead to throttling (e.g., hot partitions).
</details>

<details>
<summary>🎯Q. Amazon timestream </summary>

- Amazon Timestream is a serverless time-series database optimized for fast ingestion, efficient storage, and analytical queries on time-stamped data (e.g., IoT, DevOps metrics). It automates retention, tiered storage (hot/warm layers), and scales seamlessly.

Simple real-world example: ⭐
- A smart factory uses Timestream to analyze sensor data (temperature, vibration) from 10,000 machines. Data is ingested every second, stored cost-effectively, and queried to detect anomalies (e.g., "Which machines exceeded 100°C in the last hour?"). Older data is auto-archived to warm storage, reducing costs.


Exam Tips: ⭐

- Use Timestream for time-series data (metrics, IoT) requiring time-based aggregations (e.g., AVG() over 5-minute intervals).
- Differentiate from Keyspaces: Keyspaces is a wide-column DB (Apache Cassandra-compatible) for general-purpose apps; Timestream is purpose-built for time-series.
- Serverless & auto-scaling: No capacity planning; handles petabytes with tiered storage (hot for recent, warm for historical).
- SQL-like querying: Supports time-series functions (e.g., time_binned_avg, smooth).

</details>


<details>
<summary>🎯Q. Amazon Athena important points (its OLAP) </summary>

- Amazon Athena is a serverless, interactive query service that uses standard SQL to analyze data directly in Amazon S3. It supports various formats (CSV, JSON, Parquet, etc.) and is ideal for ad-hoc querying, log analysis, and cost-effective big data analytics.


Simple end-to-end real-world example: ⭐
A retail company stores daily sales logs (CSV files) in Amazon S3. Using Athena:
 - Define a table schema in AWS Glue Data Catalog.
 - Run SQL queries directly on S3 data (e.g., SELECT product_id, SUM(sales) FROM sales_data GROUP BY product_id).
 - Visualize results in Amazon QuickSight for sales trends.
 - No data movement or infrastructure setup required!

⭐Exam Tips: ⭐

- Serverless: No servers to manage; pay per query (based on data scanned).
- S3 Integration: Directly queries data in S3 (supports structured/semi-structured data).
- Cost Optimization: Use columnar formats (e.g., Parquet) and partition data to reduce scanned data.
- AWS Glue: Often paired with Athena for schema/ETL management.
- Key Differentiator: Athena vs. Amazon Keyspaces (managed Cassandra DB for NoSQL) vs. RDS/Aurora (OLTP databases).

Common Mistakes: ⚠️

- Assuming Athena is for ⭐transactional workloads (OLTP)⭐ – it’s designed for OLAP (analytics).
- Not compressing/partitioning data, leading to high query costs.
- Confusing Athena with Redshift (Athena = ad-hoc S3 queries; Redshift = data warehouse for complex ETL/analytics).
- Forgetting that Athena uses schema-on-read (data format/partitioning matters!).

</details>

<details>
<summary>🎯Q. what is RedShift ? why its used ? </summary>

- Amazon Redshift is a fully-managed, petabyte-scale data warehousing service optimized for analytics workloads. (OLAP)
- It uses columnar storage, Massively Parallel Processing (MPP), and advanced compression to deliver fast query performance on large datasets. It integrates with BI tools (e.g., Tableau) and supports SQL for complex analytics.


Simple real-world example: ⭐
An e-commerce company stores sales transactions in Amazon RDS (OLTP) and customer logs in S3. They use Redshift to:

- Ingest data from RDS (via CDC) and S3 (via COPY commands).
- Transform/aggregate data (e.g., daily sales per region).
- Analyze trends using Amazon QuickSight dashboards for business decisions.

Exam Tips: ⭐

- Redshift vs. RDS: Redshift is for OLAP (analytics), RDS for OLTP (transactions).
- Columnar storage improves performance for aggregation queries (e.g., SUM, COUNT).
- Compression reduces storage costs and speeds up queries (automatically applied).


Common Mistakes: ⚠️

- Using Redshift for OLTP workloads (instead of RDS/Aurora).
- Assuming Redshift Serverless is always cheaper (cost depends on usage patterns; compare with provisioned clusters).

</details>


<details>
<summary>🎯Q. Opensearch notes </summary>

- Amazon OpenSearch Service (successor to Amazon Elasticsearch) is a managed service for real-time search, analytics, and visualization of structured/unstructured data. It supports use cases like log analytics, full-text search, and operational monitoring.
- Built on OpenSearch (forked from Elasticsearch), it integrates with AWS services (e.g., Kinesis, CloudWatch) and offers scalability, security (IAM/VPC), and Kibana for dashboards.


Common Mistakes: ⚠️

- Using OpenSearch for transactional workloads (it’s optimized for search/analytics, not ACID compliance).
- Overlooking index management (poorly designed indices lead to high costs/performance issues).
- Confusing UltraWarm (for read-only, historical data) with hot storage (for real-time data).

</details>


<details>
<summary>🎯Q. Amazon QuickSight </summary>

- Amazon QuickSight is AWS’s fully managed, cloud-native business intelligence (BI) service for creating interactive dashboards, visualizations, and ad-hoc analyses. 
- It is serverless, scales automatically, and `uses a pay-per-session pricing model` (charges apply only when users access dashboards or reports).
- It integrates with AWS data sources (S3, RDS, Athena, Redshift) and third-party tools, using SPICE (Super-fast, Parallel, In-memory Calculation Engine) for fast query performance.


Simple real-world example for end-to-end visualization: ⭐
A retail company stores sales data in Amazon S3. They use QuickSight to:

- Connect to S3 and create a dataset.
- Prepare data (clean, transform) using QuickSight’s UI.
- Build dashboards with charts (e.g., sales by region, YoY growth).
- Share dashboards with regional managers via email/URL.
- Managers access dashboards on any device, paying only for their active sessions.

</details>

<details>

<summary>🎯Q. Amazon glue </summary>

- Amazon Glue is a `serverless ETL` (Extract, Transform, Load) service that automates data preparation and integration. 

- Key components:
  - `Data Catalog`: Metadata repository for all data assets.
  - `Crawlers`: Automatically discover and catalog data schemas. (e.g., from S3, RDS).
  - `Jobs`: PySpark/Spark code for ETL workflows.
  - `Glue Studio`: Visual ETL interface.
  - `Glue ETL`: Generates and runs data transformation code.


Real-World Example: ⭐
A company aggregates CSV logs from IoT devices in S3. Use case:

- Glue Crawler scans S3 paths, infers schema, and populates the Data Catalog with metadata.
- Glue Job transforms data (e.g., cleans duplicates, converts to Parquet) and writes to a processed S3 bucket.
- QuickSight visualizes results.


Exam Tips: ⭐

- Glue vs. Lambda:
    - Use Glue for heavy/long ETL jobs (Spark-based, serverless, scales automatically).
    - Use Lambda for quick, event-driven tasks (e.g., trigger on S3 upload, 15-min max runtime).
- Glue Data Catalog acts as a central schema store for Athena, Redshift Spectrum, and EMR.
- Glue is serverless; no infrastructure to manage (vs. EC2/EMR).
- Glue Elastic Views: Combines data across sources (e.g., merge RDS + DynamoDB) via SQL.

Common Mistakes: ⚠️

- Thinking Lambda replaces Glue for ETL (Lambda lacks Spark’s scalability for large datasets).
- Forgetting to run/configure Crawlers before querying data with Athena (no schema = errors).
- Assuming Glue Jobs are real-time (they’re batch/ETL-focused; use Kinesis for real-time).
- Overlooking job bookmarks (prevents reprocessing old data in incremental workflows). 
  - job bookmarks helps to ensure that incremental data processing is handled efficiently by remembering which data has already been processed. This is particularly useful when dealing with large datasets or streaming data, as it avoids reprocessing data that has already been processed in previous job runs.

</details>

<details>
<summary>🎯Q. AWS data lake what it is?</summary>

- A data lake is a centralized repository (like Amazon S3) `storing structured/unstructured data at any scale`.
- AWS Lake Formation is a `service simplifying data lake setup, managing ingestion, cleaning, security, and governance.`
  simple example

Simple real-world example: ⭐
A retail company builds a data lake in S3 to analyze customer behavior:
- Raw data ingestion: Store clickstream logs, sales data, social media feeds in S3
- Processing: Use Glue for ETL to clean/transform data into analytics-ready format
- Analysis: Query with Athena/Redshift Spectrum, visualize in QuickSight
- ML/AI: Use SageMaker to build recommendation models on processed data
- Security: `Lake Formation manages access controls and data governance`

Exam Tips: ⭐

- `Blueprints` = Predefined templates for ingesting data (e.g., RDS, DynamoDB).
- `Lake Formation centralizes permissions` (integrates with IAM/LF-Tags for column/row-level security).
- Uses AWS Glue for ETL, crawlers for schema discovery.
- not built in storage but uses S3 for storage
- not built in analytics but uses Athena, Redshift, QuickSight for analytics.

Common Mistakes: ⠀⚠️

- Assuming Lake Formation replaces S3/Glue (it’s a management layer).
- Confusing blueprints with Glue workflows (blueprints simplify ingestion, Glue handles ETL).
- Assuming Lake Formation is a data warehouse (it’s a data lake management tool).
</details>

<details>
<summary>🎯Q. Amazon managed streaming for apache kafka </summary>

- Amazon MSK (Managed Streaming for Apache Kafka) is a fully managed service for Apache Kafka, simplifying real-time data streaming for applications like analytics, monitoring, and event-driven architectures.

Real-World Example: ⭐
Ride-Sharing App:

- Producers: Driver/passenger apps send real-time GPS data to MSK.
- MSK Topics: Streams data to topics like ride-locations or fare-updates.
- Consumers: Services process data for ETA calculations, surge pricing, and dashboards.
- S3/Redshift: Processed data is stored for analytics.

Exam Tips: ⭐
- Use MSK for Kafka-compatible ecosystems, existing Kafka tools (e.g., Kafka Connect), or complex event routing.
- Use Kinesis for serverless streaming, simpler setups, or native AWS integrations (e.g., Lambda, Firehose).
- MSK requires cluster management (brokers, storage); Kinesis scales automatically with shards.

Common Mistakes: ⚠️

- kinesis vs MSK : kinesis is serverless and MSK is not.
- Assuming MSK is entirely serverless (you manage brokers/zookeepers).
- Assuming MSK is a drop-in replacement for Kafka (it’s a managed service, not a Kafka cluster).


</details>

## Machine Learning

<details>
<summary>🎯Q. AWS AI/ML services for different aspects of content processing:</summary>

- `Rekognition`: Image/video analysis (objects, faces, text detection)
- `Transcribe`: Speech-to-text conversion
- `Polly`: Text-to-speech conversion
- `Translate`: Text translation between languages
- `Lex`: Conversational AI chatbots
- `Connect`: Cloud contact center service : call center, chatbot.
- `comprehend`: Natural language processing (sentiment analysis, entity recognition)
- `sagemaker`: Machine learning (training, inference, deployment)`
- `kendra` : Enterprise search and knowledge management. (comprehend vs kendra : comprehend is for text analysis, kendra is for search and knowledge management, kendra index documents and provides search capabilities
- `personalize` : Personalized recommendations , amazon like recommendations .
Simple end-to-end real-world example: ⭐
- `textract` : extract text from documents, images, and PDFs.


- Global Customer Service Portal:
- Video support ticket → Rekognition analyzes video content → Transcribe converts speech to text → Translate converts to required language → Polly reads back in target language → Lex handles customer queries → Connect routes to appropriate agent


Exam Tips: ⭐

- Remember Transcribe is speech-TO-text, Polly is text-TO-speech
- Lex handles conversational AI (chatbots), while Connect manages contact center operations
- Rekognition can analyze both images AND videos
- Translate works with text only, not speech
- These services can be combined for end-to-end solutions 
- Connect integrates natively with Lex for automated interactions


Common Mistakes: ⚠️

- Confusing Transcribe and Polly's functions (direction of conversion)
- Mixing up Lex (for understanding intent) with Translate (for language translation)  
- Assuming Translate can directly translate speech (`needs Transcribe first`)
- Thinking Rekognition can only process images (⭐it handles videos too⭐)
- Forgetting that Connect is needed for full contact center functionality

</details>

<details>
<summary>🎯Q. Amazon Sagemaker </summary>

- Amazon SageMaker is a fully managed service that `enables building, training, and deploying machine learning models at scale`. 
- It provides the entire ML workflow from data labeling, preparation, feature engineering, training, tuning, deployment, and monitoring.

Simple end-to-end real world example: ⭐
Retail Demand Forecasting:

- Upload historical sales data to S3
- `Use SageMaker Notebooks for data preprocessing`
- Select built-in XGBoost algorithm for forecasting
- Train model using spot instances for cost optimization
- Deploy model to SageMaker endpoint
- Integrate with retail application for real-time predictions

Exam Tips: ⭐

- SageMaker provides built-in algorithms (no need to create from scratch)
- Notebooks run on managed instances (no infrastructure management) 
- Training can use spot instances for cost savings
- Endpoints provide real-time inference (but manual scaling by default)
- Supports automatic model tuning (hyperparameter optimization)
- Ground Truth for data labeling
- Can integrate with AWS services like S3, ECR, Lambda

</details>


⭐ One Liner Notes⭐
- Amazon DynamoDB can not be used to store big objects. ⭐The maximum item size in DynamoDB is 400KB.⭐
- DocumentDB is fully managed MongoDB compatible.


## AWS Security

<details>
<summary>🎯Q. What is Encryption in transit means? </summary>

- Encryption in transit refers to securing data as it moves between systems (e.g., client-server, server-server) using encryption protocols like TLS/SSL.
- It ensures that data is not readable by unauthorized parties during transmission.

Simple end-to-end real world example: ⭐
- A user sends a password over the internet to a website.
- The website uses TLS to encrypt the password before sending it to the server.
- The server decrypts the password and uses it to authenticate the user.

Exam Tips: ⭐
- Encryption in transit is about securing data during transmission.
- It's different from encryption at rest, which secures data while it's stored.
</details>


<details>
<summary>🎯Q. KMS Overview </summary>

- AWS Key Management Service (KMS) is a fully managed service that allows you to create and control encryption keys used to encrypt your data.
- It integrates with other AWS services to protect data at rest and in transit. - KMS supports encryption, decryption, and key rotation.
- It provides a secure and durable key storage, and you `can audit key usage with CloudTrail`.
- KMS is integrated with AWS services like S3, EBS, RDS, Redshift, and more.- KMS supports Bring Your Own Key (BYOK) for customer-managed keys.

- Types of keys:
    - symmetric keys: used for encryption and decryption
    - asymmetric keys: used for digital signatures and key exchange
    - Symmetric key encryption uses the same key for both encryption and decryption.
    - Asymmetric key encryption uses a pair of keys: a public key for encryption and a private key for decryption.

Exam Tips: ⭐

- KMS supports two types of keys: Symmetric (AES-256) and Asymmetric (RSA/ECC). Symmetric keys are used for encryption/decryption, while asymmetric keys are used for signing/verification or encryption/decryption.
  - `KMS keys are region-specific and do not replicate across regions`. If you need cross-region encryption, you must create keys in each region.
- KMS integrates with AWS services like S3, EBS, RDS, and Lambda for seamless encryption.
- Key policies control access to KMS keys. `Always ensure the root user has full access to avoid lockout`.
- Use Automatic Key Rotation for symmetric keys to enhance security without manual intervention.
- handling multi region specific keys : 
  - `Cross-Region Replication`: Manually copy encrypted data to another region. (TODO understand more)
  - `Multi-Region Keys`: Create a multi-region key in each region and use them for encryption/decryption.
  
</details>

<details>
<summary>🎯Q. AWS secret manager </summary>

- AWS Secrets Manager is a service that helps you securely store, manage, and retrieve sensitive information such as database credentials, API keys, and passwords. 
- It also supports automatic rotation of secrets, reducing the risk of compromised credentials. 
- In contrast, AWS Systems Manager Parameter Store is a service for storing configuration data and secrets, but it lacks built-in secret rotation and has fewer integrations with AWS services.


Simple end-to-end real-world example for better visualization: ⭐
- Imagine you’re running a web application that connects to a MySQL database. Instead of hardcoding the database credentials in your application code, you store them in AWS Secrets Manager. The service encrypts the credentials and allows your application to retrieve them securely via API calls. Additionally, Secrets Manager automatically rotates the database password every 30 days, ensuring your application always uses updated credentials without manual intervention. For non-sensitive configuration data like environment variables, you can use Parameter Store, which is cost-effective but doesn’t support rotation

Exam Tips: ⭐

- Use Secrets Manager for sensitive data like database credentials, especially when automatic rotation is needed
- Use Parameter Store for non-sensitive configuration data or when cost efficiency is a priority
- Remember that Secrets Manager integrates directly with AWS services like RDS, while Parameter Store does not
- `Know the pricing difference`: Secrets Manager charges per secret, while Parameter Store is free for standard parameters

Common Mistakes: ⚠️

- Mistake 1: Using Parameter Store for sensitive data that requires automatic rotation, which it doesn’t support  .
- Mistake 2: Overpaying by using Secrets Manager for non-sensitive data that could be stored in Parameter Store.

</details>


<details>
<summary>🎯Q. AWS Certificate Manager </summary>

- AWS Certificate Manager (ACM) simplifies provisioning, managing, and deploying SSL/TLS certificates to secure applications with HTTPS.
- 

Exam Tips: ⭐

- ACM certificates ⭐must be in the same AWS region⭐ as the resource (e.g., ALB, CloudFront).
- ⭐`CloudFront` requires certificates in `us-east-1`⭐, even if your infrastructure is elsewhere.⭐
- ACM provides free public certificates; private certificates cost extra.
- Supports wildcard certificates (e.g., *.example.com).
- Automatic renewal for ACM-managed certificates.


Common Mistakes: ⚠️

- Forgetting regional restrictions (e.g., using an ACM certificate from us-west-2 for a CloudFront distribution).
- Attempting to use ACM for EC2 instances directly (requires manual certificate installation).

</details>


<details>
<summary>🎯Q. AWS WAF (Web application firewall) </summary>


- AWS WAF (Web Application Firewall) is a managed service that protects web applications from common exploits by filtering and monitoring HTTP/HTTPS traffic using customizable rules. 
- It integrates with services like CloudFront, Application Load Balancer (ALB), and API Gateway.


Exam Tips: ⭐
- WAF is a ⭐ stateless firewall⭐ , so it doesn’t maintain session state.
- Use WAF to protect against common web attacks, not for DDoS protection.
- WAF supports both managed rules and custom rules, allowing you to fine-tune protection.
- ⭐ OWASP Top 10:⭐  Focus on rules mitigating SQLi, XSS, and HTTP floods.
- Cost: Costs accrue per rule and per request processed—optimize rule sets to avoid unnecessary expenses.


Common Mistakes: ⚠️

- Assuming AWS WAF alone provides full security (combine with Shield for DDoS protection, Security Groups, etc.).
- Misconfiguring rule priorities (e.g., allowing a broad rule to override a specific blocking rule).
- Overlooking regional vs. global deployment (e.g., using a regional WAF for a global CloudFront distribution).

Exam Questions (Critical Types):

- Primary functions of AWS WAF: Filter web traffic, block exploits (SQLi/XSS), and enable custom rule-based protection.
- Positive vs. Negative Security Models:
  - Positive: Allows only pre-approved traffic (whitelisting).
  - Negative: Blocks known malicious traffic (blacklisting).
- Integration with AWS services: Works with CloudFront/ALB for traffic inspection, logs to CloudWatch/S3, and pairs with AWS Shield for DDoS mitigation.
</details>


<details>
<summary>🎯Q. AWS Shield important notes </summary>

- AWS Shield is a managed Distributed Denial of Service (DDoS) protection service that safeguards applications running on AWS.

Simple End-to-End Real-World Example:

- An online banking app uses AWS Shield Advanced:
- During a volumetric DDoS attack targeting its public-facing API Gateway.
- Shield Advanced automatically detects and mitigates traffic spikes.
- Provides real-time attack visibility via AWS WAF dashboards.
- Ensures continuous uptime for customers during the attack.

Exam Tips:

- Standard vs. Advanced:
    - Standard: Free, always enabled for AWS resources (e.g., CloudFront, ELB).
    - Advanced: Paid, includes enhanced DDoS mitigation, 24/7 SOC support, and financial protection against scaling costs during attacks.
- Use Cases: Protect mission-critical apps (e.g., gaming, financial services).
- Integration: Works with CloudFront, Route 53, and Elastic IPs for comprehensive protection.


Common Mistakes: ⚠️

- Assuming Shield Standard provides full DDoS protection (Advanced is needed for complex attacks).
- Overlooking Shield Advanced’s SLA-backed 99.99% availability guarantee.
</details>


<details>
<summary>🎯Q.  AWS GuardDuty, INspector, Macie </summary>

- `AWS GuardDuty`: A threat detection service monitoring AWS accounts, VPCs, and S3 logs for malicious activity (e.g., unauthorized API calls, compromised instances).
- `Amazon Inspector`: Automated `vulnerability assessment` for EC2 instances, containers, and serverless functions, focusing on network exposure and software vulnerabilities.
- `AWS Macie`: ML-powered service to discover/classify `sensitive data` (e.g., PII, credit card numbers) in S3 buckets and enforce data protection policies.


Exam Tips Key Differentiators: ⭐

- GuardDuty → Behavioral analysis (e.g., crypto-mining, credential compromise).
- Inspector → Vulnerability scoring (CVSS-based) and compliance checks (CIS benchmarks).
- Macie → Data classification (supports custom identifiers) and GDPR/HIPAA compliance.
- Integration: All three integrate with AWS Security Hub for centralized alerts.
- `Cost Model`: GuardDuty charges `per GB` of log data analyzed; Macie `per GB` of data scanned.

Common Mistakes

❌ Assuming GuardDuty requires manual log analysis (it’s fully managed).
❌ Overlooking Inspector’s need for an agent on EC2 instances.
❌ Assuming Macie works only with S3 (it also supports Redshift and Aurora).

</details>

One liners ⭐
- parameter store has versioning capability
- GuardDuty does not scan CloudWatch logs - because it is designed to analyze specific AWS data sources directly,If you need to analyze CloudWatch Logs for potential security threats, you can use other AWS services like Amazon Detective, AWS Security Hub, or integrate with third-party tools for log analysis.
- AWS Firewall Manager is a security management service that allows you to centrally configure and manage firewall rules across your accounts and applications in AWS Organizations.



## VPC

<details>
<summary>🎯Q. CIDR, subnet mask, Private Vs PublicIP </summary>

- `CIDR` (Classless Inter-Domain Routing): A notation for specifying IP ranges (e.g., 10.0.0.0/16), where the suffix (/16) defines the network portion. Replaces rigid class-based subnetting.
- `Subnet Mask`: A 32-bit number (e.g., 255.255.255.0) that splits an IP into network/host parts. A /24 mask = 24 network bits + 8 host bits.
- `Private IP`: Non-routable on the internet (e.g., 10.0.0.0–10.255.255.255, 172.16.0.0–172.31.255.255, 192.168.0.0–192.168.255.255). Used internally in AWS VPCs.
- `Public IP`: Globally unique, routable on the internet. Assigned to AWS resources like EC2 instances or NAT gateways.


Real-World Example ⭐
Scenario: A company deploys a web app with a database.⭐

- `VPC`: Created with CIDR 10.0.0.0/16 (65,536 total IPs).
- `Public Subnet`: 10.0.1.0/24 (256 IPs) for a web server with a public IP (203.0.113.5).
- `Private Subnet`: 10.0.2.0/24 for a database with only a private IP (10.0.2.100).
- `NAT Gateway`: In the public subnet to allow the private database to access the internet for updates.


Exam Tips ⭐

- CIDR Prefixes: Memorize common ones:
    - /32 = 1 IP
    - /24 = 256 IPs (254 usable)
    - /16 = 65,536 IPs
    - /8 = 16,777,216 IPs (16,777,214 usable)
- Subnet Mask: Understand how it works (e.g., 255.255.255.0 = 24 bits).

Common Mistakes

- `Mixing private/public ranges`: Public IPs can’t start with 10.x.x.x or 192.168.x.x.
- `Subnet Size`: Forgetting that AWS reserves 5 IPs (e.g., a /24 has 251 usable IPs, not 256).
</details>

<details>
<summary>🎯Q. VPC Subnets, Internet and Nat gateway and route tables what they are and how they are connected/work together ? </summary>

- VPC subnets partition your network, Internet Gateway (IGW) enables public internet access, NAT Gateway allows private subnets to initiate outbound internet traffic, and route tables define traffic rules between these components.

Simple end-to-end real-world example: ⭐
- Imagine a 2-tier web app:
  - Public Subnet: Hosts a load balancer (uses IGW for inbound/outbound internet traffic).
  - Private Subnet: Hosts web servers (route table directs outbound traffic to NAT Gateway) and a database (no internet access).
  - Route Tables:
    - Public subnet route: 0.0.0.0/0 → IGW
    - Private subnet route: 0.0.0.0/0 → NAT Gateway

Exam Tips: ⭐

- `IGW` = Bidirectional public internet access ⭐(used by public subnets).⭐
- `NAT Gateway` = Outbound-only internet for private subnets (⭐placed in public subnet⭐).
- `Route tables are subnet-specific`, default route (0.0.0.0/0) determines internet access.
- NAT Gateway requires an ⭐Elastic IP⭐ and scales automatically (vs. NAT instances).


Common Mistakes: ⚠️

- Confusing IGW (public) with NAT Gateway (private).
- Forgetting to ⭐associate subnets with the correct route table.⭐
- Placing NAT Gateway in a private subnet (⭐must be in public⭐).
</details>


<details>
<summary>🎯Q. What is NACL and security groups?  </summary>

- `NACL (Network ACL)`: ⭐Stateless⭐ subnet-level firewall (rules for inbound/outbound traffic).
- `Security Group`: ⭐Stateful⭐ instance-level firewall (acts as virtual firewall for EC2 instances).
- Ephemeral Ports: Temporary ports (1024-65535) used for outbound responses. 

Simple end-to-end real-world example: ⭐

- A web server (EC2) in a public subnet:
  - Security Group: Allows inbound HTTP(80)/HTTPS(443) from 0.0.0.0/0.
  - NACL: Blocks specific IPs at subnet level but allows ephemeral ports for responses.
  - IGW: Attached to VPC for public internet access.
  - Route Table: Directs 0.0.0.0/0 traffic to IGW.

Exam Tips: ⭐

- NACL = Stateless (explicitly allow inbound/outbound), Security Group = Stateful (automatic return traffic).
  - `You must explicitly configure both ALLOW and DENY rules for NACLs`, and they are evaluated in numerical order (Rule #100 is evaluated before Rule #200).
- Ephemeral ports must be allowed in NACL outbound rules for responses.
- Security Groups deny by default; NACLs explicitly allow/deny (evaluated in rule-number order).
  - ⭐`Everything is DENIED by default`⭐. You must explicitly ALLOW what you want (like opening specific ports). You cannot create DENY rules.
- IGW enables public internet; NAT enables private subnet outbound traffic; Route Tables define traffic paths.

</details>


<details>
<summary>🎯Q. VPC endpoints notes what it is ? </summary>

- A VPC Endpoint is a private connection between your VPC and supported AWS services without using the internet, NAT devices, VPN, or AWS Direct Connect. It ensures secure and efficient communication within the AWS network.

Simple end-to-end real-world example for better visualization: ⭐

- Imagine you have an application running in a private subnet of your VPC that needs to access an S3 bucket. Instead of routing traffic through the internet or a NAT gateway, you create a Gateway VPC Endpoint for S3. This allows your application to securely access the S3 bucket directly within the AWS network, reducing latency and avoiding internet-related security risks.

Exam Tips: ⭐

- There are two types of VPC Endpoints:
    - `Gateway Endpoint`: **Supports only S3 and DynamoDB**. Automatically adds a route to your VPC route table.
    - `Interface Endpoint`: Supports most AWS services (e.g., SQS, SNS, KMS). Uses an Elastic Network Interface (ENI) with a private IP address.
- `VPC Endpoints are region-specific` and cannot be used across regions.
- `Endpoint Policies` control access (like IAM policies).
- Interface Endpoints incur additional costs, while Gateway Endpoints are free.
- ⭐Gateway Endpoints only work within the same region.⭐
- ⭐Interface Endpoints require security groups (gateway endpoints do not).⭐


Common Mistakes: ⚠️

- Assuming VPC Endpoints work across regions (they are region-specific).
- Using a NAT gateway or internet gateway when a VPC Endpoint would be more secure and cost-effective.
- Forgetting to configure Endpoint Policies to restrict access to specific resources or actions.
- Forgetting to update route tables for gateway endpoints.
- Overlooking endpoint policies, leading to unintended access.
</details>



<details>
<summary>🎯Q. Site to Site VPN notes and its comparision with others connection when it comes to connecting AWS with on-premise </summary>

- `Site-to-Site VPN` establishes an encrypted IPSec connection between an on-premises network and AWS VPC, using a Virtual Private Gateway (VGW) as the AWS-side endpoint.
- It uses a VGW(Virtual Private Gateway) and a Customer Gateway (CGW) on-premises, with a VPN tunnel for secure communication.
- `Direct Connect` is about `dedicated private connections` with high bandwidth, while `Site-to-Site VPN` is about quick, encrypted connections over the internet.
- AWS private connect is about dedicated private connections with high bandwidth (without internet), while site-to-site VPN is about quick, encrypted connections over the internet.

Direct Connect AND SIte-to-Site VPN ⭐

- This combination provides the highest availability for hybrid architecture, though at higher cost than either solution alone.

- site-to-site VPN when connection is over internet
- direct connect when connection is not over internet
</details>


<details>
<summary>🎯Q. Transit Gateway notes </summary>

- AWS Transit Gateway acts as a network hub to connect multiple VPCs, VPNs, and Direct Connect connections, simplifying large-scale network architectures (regional resource with cross-region peering).

Exam Tips: ⭐

- `Regional scope`: 1 Transit Gateway per region (use inter-region peering for cross-region)
- `Max attachments`: 5,000 VPCs/Transit Gateway (vs VPC peering's 125 limit)


Common Mistakes: ⚠️

- Transit Gateway is the preferred choice for complex network architectures over VPC Peering (mesh connections vs hub-spoke) 
- Assuming transitive routing works automatically (requires proper route table setup)

</details>



One liner notes ⭐
- Addressing For the VPC (10.0.0.0/16), it allocates IPs from 10.0.0.0 to 10.0.255.255.
- The subnet(10.0.0.0/24) allocates the IP address ranging from 10.0.0.0 to 10.0.0.255.
- Bastian host is EC2
- Everything is DENIED by default. You must explicitly ALLOW what you want (like opening specific ports). You cannot create DENY rules.
- analogy to understand important conponents
    - `NACL` = Building's main entrance security (checks everyone both entering AND leaving)
    - `Security Group` = Individual office door access (once you're allowed in, you can freely exit)
    - `Route Table` = Building's directory showing which way to go
    - `Internet Gateway (IGW)` = Main building entrance to public street
    - `NAT Gateway` = One-way door that lets people inside make external calls but prevents external access
- REMEMBER NAT Gateways are for outbound traffic from private subnets to the internet. (NOT INBOUND)
- max CIDR size in AWS is /16.
- A Dedicated Direct Connect connection supports ⭐1Gbps and 10Gbps.⭐
- ⭐AWS VPN CloudHub⭐ allows you to securely communicate with multiple sites using AWS VPN. It operates on a simple hub-and-spoke model that you can use with or without a VPC.


## Disaster Recovery and migrations

<details>
<summary>🎯Q. RPO and RTO what they are ? and what are different Disaster recovery strategies ? </summary>

- RPO is how much data loss are you willing to tolerate in case of disaster? (e.g., "How much data can we afford to lose?").)
- RTO is how quickly you can recover from a disaster? (e.g., "How fast must systems recover?").

Simple Real-World Example: ⭐
An e-commerce platform:

- Backs up transactions every 1 hour → RPO = 1 hour.
- Restores systems within 4 hours after outage → RTO = 4 hours.

⭐⭐ Disaster Recovery Strategies: ⭐⭐

- `Backup & Restore (High RTO/RPO, low cost)`: Manual restoration from backups (e.g., S3 → EC2).
- `Pilot Light (Mid RTO/RPO)`: Core services (e.g., DB) run in standby; others scale on-demand.
- `Warm Standby (Low RTO/RPO)`: Partial replica always running (e.g., Multi-AZ RDS).
- `Multi-Site Active-Active (~0 RTO/RPO, high cost)`: Fully replicated across regions (e.g., Global DynamoDB).

Exam Tips: ⭐

- RPO focuses on data loss; RTO focuses on downtime.
- AWS strategies sorted by speed: Backup → Pilot Light → Warm Standby → Multi-Site.
- Cost ↑ as RTO/RPO ↓ (e.g., Multi-Site is expensive but fastest).
- RPO depends on backup/replication frequency (e.g., hourly snapshots → 1hr RPO).

Common Mistakes: ⚠️

- Confusing RTO (time to recover) with RPO (data loss tolerance).
- Assuming "Cold Standby" (no pre-provisioned resources) has lower RTO than Warm Standby.
- Overlooking replication lag (e.g., RPO ≠ backup interval if replication is asynchronous).

</details>


<details>
<summary>🎯Q. DMS - Database Migration Service  (DMS) notes </summary>

- AWS DMS (Database Migration Service) is a managed service for migrating relational/S3 databases to AWS or between sources with minimal downtime. Supports homogeneous (e.g., MySQL to MySQL) and heterogeneous (e.g., Oracle to Aurora) migrations using Schema Conversion Tool (SCT).

Exam Tips: ⭐

- CDC (Change Data Capture) enables near-real-time replication during migration
- Use SCT for schema/stored proc conversion in heterogeneous migrations
- Supports cross-account/region/VPC migrations with proper networking
- DMS avoids downtime by synchronizing changes during migration
- Critical sources: RDS, S3, DynamoDB, MongoDB, on-prem DBs

</details>

<details>
<summary>🎯Q. Transferring large amount of Data to AWS ? different options  </summary>

- AWS provides multiple services for transferring large data: AWS Snow Family (physical devices), Direct Connect (dedicated network), S3 Transfer Acceleration (fast uploads), DataSync (automated migration), and Storage Gateway (hybrid storage). Choose based on data size, network bandwidth, and security needs.

Real-World Example: ⭐

- A media company with 2PB of video archives uses:
  - Snowmobile (truck-sized storage) to transfer 1.5PB offline.
  - Snowball Edge for 500TB of sensitive footage (encrypted, air-gapped transfer).
  - DataSync to keep new files synced to S3 via Direct Connect.

Exam Tips: ⭐

- `Snowball vs. Snowmobile`: Use Snowball for 10TB–10PB, Snowmobile for 100PB+.
- `Offline vs. Online`: Slow/unstable networks → Snow Family; fast networks → S3 Transfer Acceleration.
- `Cost`: Snow Family avoids egress fees; Direct Connect reduces cloud data transfer costs.
- `Encryption`: Snow devices use hardware encryption; DataSync uses TLS in transit.


Common Mistakes: ⚠️

- Choosing S3 Transfer Acceleration for 100TB+ data (use Snowball instead).
- Ignoring bandwidth throttling when scheduling transfers (causes delays).
- Forgetting DataSync’s auto-retry and compression features for recurring transfers.
</details>

<br>
<br>
********************************************* IGNORE BELOW THIS LINES




<details>
<summary>🎯Q. Template 1 </summary>
</details>

<details>
<summary>🎯🔥Q. Template 2 </summary>

<details>
Emojis used
⭐ - For important points
🔥 - For hot/important exam topics
💡 - For key concepts/tips
⚠️ - For warnings/common mistake
🎯 - For exam targets/focus areas/ question 
🚀 - For advanced topics .
🚫 - For indicating something that cannot be used or a concerning point
</details>




**Definition:**

**Key Features:**
- Point 1
- Point 2

**Exam Tips:** ⭐
- Important point 1
- Important point 2

**Common Mistakes:** ⚠️
- Mistake 1
- Mistake 2


</details>
*************************** IMPORTANT questions and answers ***************************
🚀 - Random notes 🚀
<details>
<summary>🎯Q. Difference between AutoScaling and LoadBalancing and how to use them to build a highly available and highly scalable applications ? </summary>

- `High Availability`: Always consider Multi-AZ deployments for databases and Auto Scaling for compute resources.
- `Scalability`: Use services like Aurora, DynamoDB, and S3 that can scale automatically with demand.

Simple end-to-end real world example: ⭐
An e-commerce website using:
- Multi-AZ RDS for database redundancy
- Auto Scaling EC2 instances across AZs
- ALB for distributing traffic
- CloudFront for content delivery
- Route 53 for DNS failover


1. Auto Scaling 
**Purpose:**
- Automatically adjusts compute resources (e.g., EC2 instances) based on demand
- Maintains performance during traffic spikes
- Reduces costs during low traffic periods

**Key Features:**
- Scaling Policies: Rules for scaling up/down based on metrics
- Target Tracking: Maintains specific metric targets (e.g., CPU at 50%)
- Scheduled Scaling: Scales at specific times (e.g., business hours)
- Health Checks: Replaces unhealthy instances

**Services:**
- EC2 Auto Scaling: For EC2 instances
- AWS Auto Scaling: For multiple services (EC2, ECS, DynamoDB, Aurora)

2. Load Balancing
**Purpose:** 
- Distributes traffic across multiple targets
- Improves fault tolerance and availability
- Prevents resource overload

**Key Features:**
- Even traffic distribution
- Health monitoring
- SSL/TLS termination
- Session stickiness

**Services (ELB Types):**
- Application Load Balancer (ALB): Layer 7, HTTP/HTTPS
- Network Load Balancer (NLB): Layer 4, TCP/UDP
- Gateway Load Balancer (GWLB): For virtual appliances

Key Differences: ⭐

| Feature | Auto Scaling | Load Balancing |
|---------|--------------|----------------|
| Purpose | Adjusts resource count | Distributes traffic |
| Focus | Resource scaling | Traffic routing |
| Metrics | CPU, memory, custom | Health, latency, requests |
| Examples | EC2 AS, AWS AS | ALB, NLB, GWLB |

How They Work Together:
- Auto Scaling ensures optimal instance count
- Load Balancing distributes traffic across instances
- Traffic increase → Auto Scaling adds instances → Load Balancer includes new instances
- Traffic decrease → Auto Scaling removes instances → Load Balancer adjusts routing

Exam Tips: ⭐
- `Auto Scaling` = scaling resources up/down
- `Load Balancing` = distributing traffic
- `Often used together for HA/scalability`
- Know Load Balancer types and use cases
</details>

<details>
<summary>🎯Q.what improves the fault tolerance of the application and make application more available ?  </summary>

- Combine Multi-AZ, Load Balancing, Auto Scaling, and Global Resilience for maximum fault tolerance and availability.

- Multi-AZ Deployments:
  - Services: RDS, Aurora, ElastiCache, EC2 (Auto Scaling).
  - Example: Deploy RDS instances across multiple Availability Zones (AZs) for database redundancy.

- Load Balancing:
  - Services: ALB, NLB, GWLB.
  - Example: Use ALB to distribute traffic across EC2 instances in different AZs.

- Auto Scaling:
  - Services: EC2 Auto Scaling, `AWS Auto Scaling (for DynamoDB, Aurora, S3)`
  - Example: Automatically add EC2 instances during traffic spikes and remove them during low traffic.

- Global Resilience:
  - Services: Route 53, Aurora Global Database, DynamoDB Global Tables.
  - Example: Use Route 53 with failover routing to direct traffic to a healthy AWS Region.

- Backup & Recovery:
  - Services: S3, AWS Backup, RDS Snapshots.
  - Example: Regularly back up RDS databases to S3 for disaster recovery.

- Monitoring & Alerts:
  - Services: CloudWatch, CloudTrail.
  - Example: Set up CloudWatch alarms to notify you of high CPU usage or instance failures.

</details>

<details>
<summary>🎯Q. TODO : Why Aurora DB is fault-tolerant and RDS is not ? also why Aurora is more scalable then RDS ?</summary>
</details>



Q. how AWS Auto Scaling (for DynamoDB, Aurora, S3) happens how its differnt then EC2 autoscaling ? do we have special optinos for AWS autoscaling ?
Q. Aurora Global Database, DynamoDB Global Tables works internally ? how globalness is achieved then other normal offerings for RDBMS and NoSql alternatives ?
Q. EventBridge vs SQS ?
Q. Difference between AWS Config vs CLoudTrail vs CloudWatch .


🚀 - Questions to answer later <br>
Q. what is bursting meaning ? overall as a concept in the cloud ? <br> what does it mean by burst workload?
Q. what is SSL/TLS certifications ? who maintains it , generates it? IMP things to know about these certificates ? how they work actually? <br>
Q. Difference between IPV4 vs IPV6 and why we have IPV6 ? <br>
Q. Differences SNS, SES, PinPoint services.


validate
- scalability means to serve increases or decreased load efficiently by either increaing or decreasing resources or compute power
- availability means to serve the request ⭐without any downtime⭐ which may cause due to hardware failure, software failure, network failure etc.

⭐⭐⭐⭐⭐⭐ Miscellaneous questions and answers  ⭐⭐⭐⭐⭐⭐
<details>
<summary>🎯Q. selecting correct EBS volume - notes </summary>

# EBS Volume Types - Quick Reference 🎯
## io1/io2 (Provisioned IOPS SSD) ⭐
- **Key Point**: Choose for high-performance, mission-critical workloads
- **Performance**:
  - Up to 64,000 IOPS
  - 160 MB/s throughput
- **Best for**:
  - Databases (especially production)
  - OLTP workloads 
  - Latency-sensitive applications
- ⚠️ Remember: If you need > 16,000 IOPS, always choose io1/io2
## io2 Block Express 🚀
- **Key Point**: Highest performer in EBS family
- **Performance**: Up to 256,000 IOPS
- Think of it as "io2 on steroids"
## gp2/gp3 (General Purpose SSD) 💡
- **Key Point**: Default choice for most workloads
- **gp3 features**:
  - Baseline 3,000 IOPS (free)
  - Can scale up to 16,000 IOPS
- **Best for**:
  - Cost-effective performance
  - Development/test environments
  - Virtual desktops
## st1 (Throughput Optimized HDD) 📊
- **Key Point**: For frequently accessed, sequential workloads
- **Best for**:
  - Big Data
  - Data warehouses
  - Log processing
- Remember: Think "S" for Sequential and "Streaming"
- General Purpose SSD (gp3) includes 3,000 IOPS at no additional cost independent of volume size.
## sc1 (Cold HDD) ❄️
- **Key Point**: Lowest cost per GB
- **Best for**:
  - Infrequently accessed data
  - Archive storage
  - Lowest cost requirement
- Remember: Think "C" for Cold storage
## Quick Decision Guide for Exam 🎯

1. Need high IOPS (>16,000)?
→ Choose io1/io2
2. Need highest possible performance?
→ Choose io2 Block Express
3. Need cost-effective general purpose?
→ Choose gp3
4. Big sequential data, frequent access?
→ Choose st1
5. Lowest cost, infrequent access?
→ Choose sc1
## Common Exam Traps ⚠️
- Don't use HDD (st1/sc1) for boot volumes
- Don't use HDD for random access patterns  
- Remember gp3 is newer and more cost-effective than gp2
- For high-performance databases, always lean towards io1/io2
</details>

<details>
<summary>🎯🔥Q. Lambda max execution time? and max memory? </summary>

- Lambda function execution time is limited to 15 minutes. <br>
- Lambda memory(RAM) limit is 10 GB. meaning you can allocate upto 10 GB of memory to your lambda function. <br>
</details>

<details>
<summary>🎯🔥Q. Globally resilient vs Regional Resilient vs AZ reliant services list those and important things to know about them </summary>

- Globally Resilient Services
  - Services that operate across multiple AWS Regions simultaneously
  - Data is automatically replicated across all AWS Regions
  - Can withstand entire region failure
  - Key Examples:
    - IAM (Identity and Access Management)
    - Route 53 (DNS Service)
    - CloudFront (CDN)
    - WAF (Web Application Firewall)

- Regional Resilient Services
  - Services that automatically replicate data across all AZs in a Region
  - Can withstand AZ failures within the region
  - Continue operating even if one or more AZs fail
  - Key Examples:
    - S3 (Simple Storage Service)
    - DynamoDB
    - SQS (Simple Queue Service)
    - ECS (Elastic Container Service)
    - EFS (Elastic File System)

AZ Resilient Services
  - Services that run in a single AZ
  - Need manual configuration for multi-AZ resilience
  - Key Examples:
    - EC2 instances (by default)
    - EBS volumes
    - RDS Single-AZ deployments
</details>

⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐


<details>
<summary>🎯🔥Q. Template 2 </summary>
</details>

<details>
<summary>🎯Q. Template 1 </summary>
</details>

⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐⭐


⭐⭐⭐⭐⭐⭐ one liners ⭐⭐⭐⭐⭐⭐

⭐⭐⭐⭐⭐⭐ Miscellaneous  ⭐⭐⭐⭐⭐⭐
- The only way to retrieve `instance metadata` is to use the link-local address, which is 169.254.169.254.
- for granting cross-account access always choose the resource based policies, because it has `Principle` element (dentity-based policies alone cannot grant cross-account access)



⭐⭐⭐⭐⭐⭐ Design Secure Architectures ⭐⭐⭐⭐⭐⭐

- SCPs ≠ IAM Policies (SCPs set boundaries; IAM grants permissions within those boundaries), SCPs require AWS Organizations
- Control Tower = `Governance Framework`; SCPs = `Permission Enforcement Mechanism`
- ❌ "Never create access keys for the AWS account root user" - AWS IAM Documentation.
- 🎯 STS(Security Token Service) is temporary credentials upto 12 hours for IAM roles mainly used for `role switching(same account), cross-account access,SSO, SAML` (Key STS APIs: AssumeRole, AssumeRoleWithSAML, AssumeRoleWithWebIdentity)
-  Always prefer roles over IAM users for programmatic access (e.g., EC2, Lambda).
- NAT gateways cannot work without Internet gateway. Flow is (Private Subnet ---> NATGW ---> IGW ---> Internet).
- Subnets are inside VPC and route-tables and network ACLs are attached to the subnets.

 


