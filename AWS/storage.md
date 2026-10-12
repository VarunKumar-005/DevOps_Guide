# AWS Storage Services: EBS, EFS, FSx and S3

## 1. Overview

AWS offers three main storage types. Each solves a different problem.

| Storage type | AWS services | How data is stored and accessed |
|---|---|---|
| **Block storage** | Amazon EBS (and EC2 instance store) | Raw volumes attached to a server, formatted with a file system by the OS |
| **File storage** | Amazon EFS, Amazon FSx | Shared file systems accessed over network protocols (NFS, SMB, Lustre) |
| **Object storage** | Amazon S3 | Objects stored in buckets and accessed through HTTP APIs |

**Quick rule of thumb**

- **EBS:** the "hard disk" of one EC2 instance.
- **EFS:** a shared Linux network drive (NFS) that grows automatically.
- **FSx:** a managed, specialized file system (Windows SMB, Lustre HPC, NetApp ONTAP, OpenZFS).
- **S3:** unlimited object storage for files, backups, data lakes, and static content.

---

## 2. Amazon EBS (Elastic Block Store)

### 2.1 What Is EBS?

Amazon EBS provides scalable, high-performance **block storage** for Amazon EC2. After you attach a volume to an instance, you use it like a local hard drive: store files, install applications, or run a database.

EBS has two main resources:

- **EBS volumes:** storage volumes attached to EC2 instances.
- **EBS snapshots:** point-in-time backups of volumes that persist independently of the volume.

### 2.2 Key Features

- **Multiple volume types:** SSD-backed for transactional workloads, HDD-backed for throughput-intensive workloads.
- **Elastic Volumes:** increase capacity or tune performance with no downtime.
- **Snapshots:** back up volumes and restore them, or copy data across accounts, Regions, and Availability Zones.
- **Encryption:** encrypts volumes, snapshots, and data in transit between instance and volume (uses AWS KMS).
- **Durability:** io2 Block Express offers 99.999% durability; other types offer 99.8% to 99.9%. Data is replicated across multiple servers **within one Availability Zone**.
- **Snapshot Archive:** low-cost tier for snapshots kept 90 days or more.
- **Automation:** Amazon Data Lifecycle Manager automates snapshot creation, retention, and deletion; Recycle Bin restores accidentally deleted snapshots and EBS-backed AMIs.

### 2.3 EBS Volume Types

| Category | Type | Best for |
|---|---|---|
| SSD | **gp3** (General Purpose) | Boot volumes, dev/test, most applications (default choice) |
| SSD | **gp2** (General Purpose, older) | Legacy general workloads |
| SSD | **io2 Block Express / io1** (Provisioned IOPS) | Mission-critical, latency-sensitive databases |
| HDD | **st1** (Throughput Optimized) | Big data, logs, data warehouses with frequent sequential access |
| HDD | **sc1** (Cold HDD) | Lowest cost for infrequently accessed data |

- io2 Block Express offers the highest performance, with average latency under 500 microseconds for 16 KiB I/O.
- st1 and sc1 are rated on 1 MiB I/O; st1 up to 500 MiB/s and sc1 up to 250 MiB/s per volume. HDD volumes cannot be used as boot volumes.
- Multi-Attach (one volume attached to several instances) is supported only on Provisioned IOPS SSD volumes.

### 2.4 Advantages of EBS

- Low latency and consistent performance for databases and OS disks.
- Data persists independently of the instance's life (unlike instance store).
- Easy backup and restore through snapshots.
- Resize and retune without downtime.

### 2.5 Disadvantages of EBS

- **Single-AZ:** a volume can attach only to instances in the same Availability Zone.
- **Mostly one instance at a time** (Multi-Attach is limited to specific volume types and use cases).
- **You pay for provisioned size,** whether or not it is full.
- **Manual management:** formatting, mounting, resizing the file system, and snapshot policies are your responsibility.

### 2.6 When to Use EBS

- Boot and root volumes for EC2.
- Relational and NoSQL databases running on EC2.
- Applications needing low-latency, block-level access.
- Persistent storage for stateful workloads (including ECS tasks on Fargate that use EBS volumes).

---

## 3. Amazon EFS (Elastic File System)

### 3.1 What Is EFS?

Amazon EFS is a **managed, shared file system** for Linux workloads, accessed over **NFS**. You create a file system, mount it on EC2 instances (or containers, Lambda, on-premises servers), and read and write data. Capacity grows and shrinks automatically as you add and remove files.

### 3.2 How EFS Works

- **File system:** the primary resource.
- **Mount targets:** network endpoints in your VPC. You create **one mount target per Availability Zone**, and a file system can have mount targets in **only one VPC at a time**. Mount targets are designed to be highly available.
- **Access points:** application-specific entry points into a file system with enforced user identity and root directory.
- **DNS name:** `file-system-id.efs.aws-region.amazonaws.com`, used in the mount command.
- **Hybrid access:** mount EFS on on-premises servers over AWS Direct Connect or Site-to-Site VPN.
- **Backups:** integrates with AWS Backup.

![Regional EFS file system with mount targets in three Availability Zones within a VPC, used by EC2 instances.](https://docs.aws.amazon.com/efs/latest/ug/images/efs-ec2-how-it-works-Regional_china-world.png)

*Figure: A Regional EFS file system shared by EC2 instances across three Availability Zones. Source: [How Amazon EFS works](https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html)*

![One Zone EFS file system with a single mount target in the same Availability Zone.](https://docs.aws.amazon.com/efs/latest/ug/images/efs-ec2-how-it-works-OneZone.png)

*Figure: A One Zone EFS file system with a single mount target. Source: [How Amazon EFS works](https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html)*

### 3.3 File System Types

| Type | Availability | Use |
|---|---|---|
| **Regional** | Data stored across multiple Availability Zones | Production workloads needing high availability |
| **One Zone** | Data stored in a single Availability Zone | Dev/test, lower cost, data that can be recreated |

### 3.4 Advantages of EFS

- **Shared access:** thousands of clients can use the same file system at once.
- **Elastic:** no capacity planning; you pay for the storage you use.
- **Multi-AZ by default** (Regional type).
- **POSIX-compatible** for Linux applications.
- **Integrates** with EC2, ECS, EKS, Lambda, and AWS Backup.
- **Encryption** at rest and in transit.

### 3.5 Disadvantages of EFS

- **Linux only (NFS):** not for Windows clients; use FSx for Windows File Server instead.
- **Higher latency than EBS** because it is accessed over the network.
- **Higher cost per GB than S3 or EBS** for large, rarely accessed data.
- **Network setup required:** mount targets, security groups (NFS port 2049), and a mount helper or NFS client.

### 3.6 When to Use EFS

- Shared content for web servers, CMS uploads, and media.
- Shared storage for containers on ECS, EKS, and Fargate.
- Home directories and developer tools.
- Machine learning and analytics jobs that read shared datasets.
- Lift-and-shift of apps that need a shared NFS file system.

---

## 4. Amazon FSx

### 4.1 What Is FSx?

Amazon FSx provides **fully managed third-party file systems** with native compatibility and feature sets. You choose the file system that matches your protocol and performance needs.

### 4.2 FSx File System Types

| FSx type | Protocol / clients | Best for |
|---|---|---|
| **FSx for Windows File Server** | SMB, NTFS, Active Directory | Windows applications, shared drives, Microsoft workloads |
| **FSx for Lustre** | Lustre (Linux clients) | HPC, machine learning training, media processing, EDA |
| **FSx for NetApp ONTAP** | NFS, SMB, iSCSI | Migrating NetApp or NAS workloads; multi-protocol access |
| **FSx for OpenZFS** | NFS | Linux workloads migrating from ZFS or needing snapshots and cloning |

### 4.3 Key Details per Type

**FSx for Windows File Server**

- Built on Windows Server; supports SMB, NTFS, and Microsoft Active Directory integration.
- Connects to EC2, WorkSpaces, AppStream 2.0, and VMware Cloud on AWS.
- Supports Single-AZ and Multi-AZ deployments with automated backups.

**FSx for Lustre**

- High-performance parallel file system for compute-intensive workloads, with throughput in the hundreds of GB/s.
- **Scratch** file systems: temporary storage, a single copy of data, and data loss if the underlying server fails.
- **Persistent** file systems: long-term storage with replication.
- Integrates with S3 so you can process S3 data with a high-performance file system.

**FSx for NetApp ONTAP**

- Multi-protocol access (NFS, SMB, iSCSI) from Linux, Windows, and macOS.
- Enterprise features: snapshots, cloning, replication (SnapMirror), and data tiering.

**FSx for OpenZFS**

- NFS access with ZFS features such as snapshots and cloning.

### 4.4 Advantages of FSx

- Fully managed: AWS handles hardware, patching, and backups.
- Native compatibility with Windows, Lustre, NetApp, and ZFS applications.
- Very high performance options (Lustre).
- Encryption at rest, with network access controlled by security groups.
- Multi-AZ options for Windows, ONTAP, and OpenZFS.

### 4.5 Disadvantages of FSx

- **More complex choices:** deployment type, throughput, storage tier, and caching differ by file system type.
- **Higher cost** than EFS or S3 for simple storage needs.
- **Specialized skills** (Windows AD, Lustre, ONTAP) help with tuning.
- **Lustre is single-AZ only,** and scratch deployments do not replicate data.

### 4.6 When to Use FSx

- Windows file shares with Active Directory (FSx for Windows).
- HPC, simulation, ML training, and video rendering (FSx for Lustre).
- Moving NetApp or on-premises NAS workloads to AWS (FSx for ONTAP).
- Linux workloads that need ZFS snapshots and cloning (FSx for OpenZFS).

---

## 5. Amazon S3 (Simple Storage Service)

### 5.1 What Is S3?

Amazon S3 is an **object storage** service with industry-leading scalability, availability, security, and performance. It stores data as **objects** inside **buckets** and is used for data lakes, websites, mobile apps, backup and restore, archives, and analytics.

### 5.2 Core Concepts

| Concept | Meaning |
|---|---|
| **Bucket** | Container for objects; the name is unique in its namespace and the Region is chosen at creation |
| **Object** | A file plus the metadata that describes it (up to 50 TB per object according to the current S3 documentation) |
| **Key** | Unique identifier of an object within a bucket (for example `photos/puppy.jpg`) |
| **Version ID** | Identifier generated when S3 Versioning is enabled |
| **Prefix** | Part of the key used to organize objects; S3 is flat, and "folders" are a console convention |

Example object URL: `https://amzn-s3-demo-bucket.s3.us-west-2.amazonaws.com/photos/puppy.jpg`

### 5.3 Bucket Types

- **General purpose buckets:** recommended for most use cases.
- **Directory buckets:** low-latency (S3 Express One Zone) and data-residency use cases.
- **Table buckets:** tabular data in Apache Iceberg format for analytics.
- **Vector buckets:** store and query vector embeddings for ML.

### 5.4 Storage Classes

| Storage class | Use |
|---|---|
| **S3 Standard** | Frequently accessed, mission-critical data |
| **S3 Express One Zone** | Single-digit millisecond latency in a single Availability Zone |
| **S3 Intelligent-Tiering** | Unknown or changing access patterns; moves data automatically |
| **S3 Standard-IA / One Zone-IA** | Infrequently accessed data |
| **S3 Glacier Instant Retrieval** | Archive with millisecond access |
| **S3 Glacier Flexible Retrieval** | Archive with retrieval in minutes to hours |
| **S3 Glacier Deep Archive** | Lowest cost, long-term archive |

### 5.5 Key Features

- **Strong consistency:** read-after-write for PUT and DELETE in all Regions. Updates to a single key are atomic.
- **Versioning:** preserve, retrieve, and restore every version of an object.
- **Lifecycle rules:** transition objects between storage classes or expire them.
- **Object Lock:** WORM (write-once-read-many) protection against deletion or overwrite.
- **Replication:** copy objects to other buckets in the same or different Regions.
- **Batch Operations:** run actions on millions or billions of objects.
- **Event notifications:** trigger Lambda, SNS, or SQS when objects change.
- **Analytics:** Storage Lens, Storage Class Analysis, and S3 Inventory.

![Two clients writing to the same S3 key; both reads return the latest data after writes finish.](https://docs.aws.amazon.com/images/AmazonS3/latest/userguide/images/consistency1.png)

*Figure: S3 strong consistency. Once both writes finish, every later read returns the newest data. Source: [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)*

### 5.6 S3 Security

- Buckets and objects are **private by default**, and Block Public Access is on by default at the bucket level.
- Use **IAM policies** and **bucket policies** (JSON, limited to 20 KB) for access control.
- **S3 Access Points** give named endpoints with dedicated policies for shared datasets.
- **ACLs are disabled by default** (Object Ownership "bucket owner enforced"); prefer policies.
- Use **IAM Access Analyzer for S3** to review who can access your data.
- Enable encryption (SSE-S3 or SSE-KMS) and log with CloudTrail and server access logs.

### 5.7 Advantages of S3

- Practically unlimited capacity and very high durability.
- Pay only for what you use, with storage classes to cut cost.
- Accessible from anywhere over HTTPS using the console, CLI, SDKs, and REST API.
- Rich security, replication, and lifecycle features.
- Works with almost every AWS service (CloudFront, Athena, Lambda, EMR, and others).

### 5.8 Disadvantages of S3

- **Not a POSIX file system:** no in-place partial edits of an object and no file locking; objects are replaced as a whole.
- **Higher latency than EBS or EFS** for small, frequent reads and writes (except S3 Express One Zone).
- **Concurrent writers:** the last write wins; build your own locking if needed.
- **Request and data transfer charges** can matter at scale.
- **Misconfigured policies** can expose data publicly.

### 5.9 When to Use S3

- Static website assets (with CloudFront).
- Backups, archives, and disaster recovery copies.
- Data lakes and analytics inputs.
- Application uploads, images, video, and logs.
- Software artifacts, container build caches, and Terraform state (with locking elsewhere).

---

## 6. Comparison: EBS vs EFS vs FSx vs S3

| Feature | EBS | EFS | FSx | S3 |
|---|---|---|---|---|
| **Storage type** | Block | File (NFS) | File (SMB, Lustre, NFS, iSCSI) | Object |
| **Access method** | Attached to EC2, formatted by OS | Mounted over NFS | Mounted over SMB, Lustre, NFS, or iSCSI | HTTPS API, SDK, CLI |
| **Scope** | Single Availability Zone | Regional (multi-AZ) or One Zone | Single-AZ or Multi-AZ (varies by type) | Regional (objects across multiple AZs, except One Zone classes) |
| **Shared by many clients** | Mostly one instance (Multi-Attach limited) | Yes, thousands of clients | Yes | Yes (via API) |
| **OS support** | Linux and Windows (via EC2) | Linux | Windows, Linux, macOS (varies by type) | Any (API based) |
| **Capacity model** | Provisioned size | Elastic, grows automatically | Provisioned (varies by type) | Unlimited |
| **Typical latency** | Lowest | Low (network file system) | Low to very low (Lustre) | Higher; milliseconds (single-digit ms for Express One Zone) |
| **Pricing basis** | Provisioned GB (plus IOPS and throughput for some types) | GB used (plus throughput options) | Provisioned storage, throughput, and backups | GB stored, requests, and data transfer |
| **Best for** | OS disks, databases | Shared Linux files, containers | Windows shares, HPC, NetApp, ZFS | Backups, data lakes, static content, archives |
| **Backup method** | Snapshots | AWS Backup | Automated backups | Versioning, replication, lifecycle |

### Quick Decision Guide

| Need | Choose |
|---|---|
| Boot disk or database volume for one EC2 instance | **EBS** |
| Shared Linux file system for many servers or containers | **EFS** |
| Windows file share with Active Directory | **FSx for Windows File Server** |
| HPC or ML training with extreme throughput | **FSx for Lustre** |
| Migrate NetApp or multi-protocol (NFS + SMB + iSCSI) storage | **FSx for NetApp ONTAP** |
| Backups, static website files, data lake, archives | **S3** |
| Cheapest long-term archive | **S3 Glacier Deep Archive** |
| Temporary scratch data on the host that may be lost | **EC2 instance store** |

---

## 7. Storage with EC2, ECS and Fargate

| Compute | EBS | EFS | FSx | S3 |
|---|---|---|---|---|
| **EC2** | Attach volumes (same AZ) | Mount with NFS or the EFS mount helper | Mount (SMB, Lustre, NFS, iSCSI) | Access via SDK/CLI or tools such as Mountpoint for Amazon S3 |
| **ECS on Fargate** | Supported for tasks | Supported for persistent shared storage | Not available as a Fargate task volume | Access through the SDK using a task role |
| **ECS on EC2** | Via the instance or ECS-managed volumes | Supported | FSx for Windows supported for Windows tasks | Access through the SDK using a task role |
| **Lambda** | Not supported | Supported (mount EFS) | Not supported | Event source and SDK access |

**Reminder:** Fargate tasks also get ephemeral task storage (bind mounts), which is **deleted when the task stops**. Use EFS or EBS for data that must persist.

---

## 8. Steps to Use Each Storage Service

### 8.1 Create and Attach an EBS Volume (Console)

1. Open the EC2 console and choose **Elastic Block Store → Volumes → Create volume**.
2. Choose the volume type (for example gp3), size, and **the same Availability Zone as your instance**.
3. Enable encryption if required, then choose **Create volume**.
4. Select the volume, choose **Actions → Attach volume**, and pick the instance.

### 8.2 Format and Mount the EBS Volume (Linux)

```bash
# List block devices and find the new volume
lsblk

# Check whether it already has a file system (new volumes are empty)
sudo file -s /dev/nvme1n1

# Create a file system (only on a NEW, empty volume)
sudo mkfs -t xfs /dev/nvme1n1

# Mount it
sudo mkdir /data
sudo mount /dev/nvme1n1 /data

# Verify
df -h
```

Device names vary (for example `/dev/xvdf` or `/dev/nvme1n1`). **Never run `mkfs` on a volume that already contains data.**

### 8.3 Create an EBS Snapshot (CLI)

```bash
aws ec2 create-volume --availability-zone AZ_NAME --size 20 --volume-type gp3
aws ec2 attach-volume --volume-id vol-XXXX --instance-id i-XXXX --device /dev/sdf
aws ec2 create-snapshot --volume-id vol-XXXX --description "daily backup"
```

### 8.4 Create and Mount an EFS File System

1. Open the EFS console and choose **Create file system** (Regional for production).
2. Select the VPC and create **mount targets** in your subnets.
3. Attach a security group that **allows inbound NFS (TCP 2049)** from your instances' security group.
4. Install the EFS client on the instance and mount the file system.

```bash
# Amazon Linux: install the EFS mount helper
sudo yum install -y amazon-efs-utils

# Mount (replace the file system ID)
sudo mkdir /mnt/efs
sudo mount -t efs -o tls fs-0123456789abcdef0:/ /mnt/efs

# Verify
df -h | grep efs
```

CLI equivalent:

```bash
aws efs create-file-system --performance-mode generalPurpose --encrypted
aws efs create-mount-target --file-system-id fs-XXXX --subnet-id subnet-XXXX --security-groups sg-XXXX
```

### 8.5 Create an FSx File System (Console Outline)

1. Open the FSx console and choose **Create file system**.
2. Select the type: Windows File Server, Lustre, NetApp ONTAP, or OpenZFS.
3. Choose the deployment type (Single-AZ or Multi-AZ where available), storage capacity, and throughput.
4. Select the VPC, subnets, and security groups.
5. For Windows File Server, join it to AWS Managed Microsoft AD or your self-managed Active Directory.
6. Create the file system and wait until its status is **Available**.
7. Mount it: SMB drive mapping on Windows, `mount -t lustre` for Lustre, or NFS or iSCSI for ONTAP and OpenZFS.

### 8.6 Create and Use an S3 Bucket (CLI)

```bash
# Create a bucket (names must be unique in their namespace)
aws s3 mb s3://my-unique-bucket-name

# Upload, list, and download
aws s3 cp report.pdf s3://my-unique-bucket-name/docs/report.pdf
aws s3 ls s3://my-unique-bucket-name/docs/
aws s3 cp s3://my-unique-bucket-name/docs/report.pdf ./report.pdf

# Sync a folder
aws s3 sync ./site s3://my-unique-bucket-name/site

# Turn on versioning
aws s3api put-bucket-versioning \
  --bucket my-unique-bucket-name \
  --versioning-configuration Status=Enabled

# Share an object temporarily (presigned URL, valid for 1 hour)
aws s3 presign s3://my-unique-bucket-name/docs/report.pdf --expires-in 3600
```

### 8.7 Clean Up

- **EBS:** unmount, detach, and delete unused volumes; delete old snapshots.
- **EFS:** unmount, delete mount targets, then delete the file system.
- **FSx:** take a final backup if needed, then delete the file system.
- **S3:** empty the bucket (including old versions) before deleting it.

---

## 9. Data Protection, Security and Backup

| Area | EBS | EFS | FSx | S3 |
|---|---|---|---|---|
| **Encryption at rest** | KMS | KMS | Yes | SSE-S3 or SSE-KMS |
| **Encryption in transit** | Between instance and volume | TLS with the mount helper | Varies by type | HTTPS |
| **Network control** | Attached to an instance | VPC security groups on mount targets | Security groups | Bucket policies, VPC endpoints |
| **Backup** | Snapshots, Data Lifecycle Manager | AWS Backup | Automated backups | Versioning, replication, Object Lock |
| **Accidental delete recovery** | Recycle Bin (snapshots, AMIs) | AWS Backup restore | Backup restore | Versioning |

---

## 10. Cost Awareness

- **EBS:** you pay for what you **provision** (GB, and IOPS or throughput on some types). Snapshots, unattached volumes, and old snapshots keep costing money.
- **EFS:** you pay for storage used, with cheaper infrequent-access classes and throughput options.
- **FSx:** you pay for provisioned storage, throughput, and backups, which makes it the most configuration-dependent.
- **S3:** you pay for storage, requests, retrievals (for colder classes), and data transfer. Lifecycle rules and Intelligent-Tiering reduce cost.
- **General tips:** delete unused volumes, snapshots, and file systems; pick the right storage class; use gp3 instead of gp2 where it fits; and set lifecycle policies on S3 buckets.

Pricing pages: [Amazon EBS pricing](https://aws.amazon.com/ebs/pricing/) | [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)

---

## 11. Best Practices

- Use **gp3** as the default EBS volume type and enable **encryption by default**.
- Automate EBS snapshots with **Data Lifecycle Manager**.
- Use **Regional EFS** for production and One Zone only for non-critical data.
- Allow NFS (port 2049) only from the security groups that need it.
- Choose the FSx type that matches your protocol, and use **Multi-AZ** for production Windows and ONTAP file systems.
- Keep S3 **Block Public Access** on, use bucket policies and least-privilege IAM roles, and enable **versioning** for important data.
- Add **S3 lifecycle rules** to move or expire old data.
- Use **IAM roles** (instance profiles, task roles) instead of long-lived access keys.
- Monitor with **CloudWatch** and review spend with Cost Explorer.

---

## 12. Storage Interview Questions and Answers

### Basic Questions

**1. What is Amazon EBS?**
Block storage volumes attached to EC2 instances; they persist independently of the instance and can be backed up with snapshots.

**2. What is Amazon EFS?**
A managed, elastic, shared NFS file system for Linux workloads, mountable by many clients across Availability Zones.

**3. What is Amazon S3?**
A highly scalable object storage service that stores objects in buckets, accessed through HTTP APIs.

**4. What is Amazon FSx?**
A managed service that provides third-party file systems: Windows File Server, Lustre, NetApp ONTAP, and OpenZFS.

**5. What is the difference between block, file, and object storage?**
Block is raw volumes managed by an OS file system, file is a shared hierarchical file system over a network protocol, and object is flat key-value storage accessed through an API.

**6. What is an EBS snapshot?**
A point-in-time backup of a volume that is stored independently and can restore new volumes or be copied across Regions and accounts.

### Intermediate Questions

**7. Can an EBS volume be attached to an instance in a different Availability Zone?**
No. The volume and instance must be in the same Availability Zone. To move data, create a snapshot and restore a volume in the target AZ.

**8. What is the difference between EBS and instance store?**
EBS persists independently of the instance. Instance store is physically attached storage that is lost when the instance stops or terminates.

**9. What is the difference between EFS and EBS?**
EBS is block storage in one AZ, usually attached to one instance. EFS is shared NFS storage accessed over the network by many instances across AZs.

**10. When would you choose FSx over EFS?**
When you need Windows SMB with Active Directory, Lustre for HPC, NetApp ONTAP features, or OpenZFS. EFS only serves Linux over NFS.

**11. What are the main S3 storage classes?**
Standard, Express One Zone, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier Instant Retrieval, Glacier Flexible Retrieval, and Glacier Deep Archive.

**12. Is S3 strongly consistent?**
Yes. S3 provides strong read-after-write consistency for PUT and DELETE requests in all Regions. Bucket configuration changes are eventually consistent.

### Scenario-Based Questions

**13. A web application on several EC2 instances must share uploaded files. Which storage?**
EFS for a shared POSIX file system, or S3 if the app can use object APIs (usually the more scalable and cheaper option).

**14. A database on EC2 needs consistent low latency. Which storage?**
EBS, using gp3 or io2 Block Express depending on the IOPS requirement.

**15. How do you reduce S3 costs for data that is rarely accessed?**
Use lifecycle rules to move objects to Standard-IA or Glacier classes, or use Intelligent-Tiering for unknown access patterns, and expire data you no longer need.

**16. How do you securely give an EC2 application access to an S3 bucket?**
Attach an IAM role with least-privilege permissions to the instance and use the SDK. Keep Block Public Access enabled, and avoid embedding access keys.

**17. How can you recover a file that was overwritten or deleted in S3?**
Enable S3 Versioning ahead of time; then restore a previous version or remove the delete marker.

**18. A Fargate task needs data that survives task restarts. What do you use?**
Amazon EFS (shared, persistent) or an EBS volume. Ephemeral task storage is deleted when the task stops.

**19. How do you protect data from accidental deletion or overwrite across EBS, EFS, and S3?**
Snapshots with Data Lifecycle Manager and Recycle Bin (EBS), AWS Backup (EFS and FSx), and Versioning plus Object Lock (S3).

**20. You need a Windows file share for a corporate application. Which service?**
Amazon FSx for Windows File Server, integrated with Active Directory.

---

## 13. Revision Checklist

- [ ] Explain block vs file vs object storage.
- [ ] List EBS volume types and when to use each.
- [ ] Create, attach, format, mount, and snapshot an EBS volume.
- [ ] Explain EFS mount targets, Regional vs One Zone, and NFS port 2049.
- [ ] Mount an EFS file system with the mount helper.
- [ ] Name the four FSx file system types and their protocols.
- [ ] Explain Lustre scratch vs persistent deployments.
- [ ] Create an S3 bucket, upload objects, and enable versioning.
- [ ] Know S3 storage classes, lifecycle rules, and Object Lock.
- [ ] Understand S3 Block Public Access, bucket policies, and IAM roles.
- [ ] Compare EBS, EFS, FSx, and S3 on scope, access, and cost.
- [ ] Know which storage works with EC2, ECS on Fargate, and Lambda.
- [ ] Review costs and delete unused volumes, snapshots, file systems, and buckets.

## Official Resources

- [What is Amazon EBS?](https://docs.aws.amazon.com/ebs/latest/userguide/what-is-ebs.html)
- [Amazon EBS volume types](https://aws.amazon.com/ebs/volume-types/)
- [How Amazon EFS works](https://docs.aws.amazon.com/efs/latest/ug/how-it-works.html)
- [Amazon FSx vs FSx for NetApp ONTAP comparison](https://aws.amazon.com/compare/fsx-and-fsx-for-netapp-ontap/)
- [What is Amazon S3?](https://docs.aws.amazon.com/AmazonS3/latest/userguide/Welcome.html)
- [Amazon EBS pricing](https://aws.amazon.com/ebs/pricing/)
- [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)
- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
