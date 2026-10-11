# Amazon EC2 (Elastic Compute Cloud)

## 1. What Is EC2?

Amazon EC2 is an AWS service that provides resizable virtual servers called **instances** in the cloud.  
It allows users to run operating systems, applications, websites, APIs, and workloads without purchasing physical servers.

![Amazon EC2 Architecture](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/EC2-instance.png)

**Official documentation:** [What is Amazon EC2?](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)

---

## 2. Advantages of EC2

- **Scalability:** Increase or decrease computing capacity according to workload.
- **Flexibility:** Choose different operating systems, CPU, memory, and instance types.
- **Cost Efficiency:** Pay for computing resources according to the selected pricing model and usage.
- **Full Control:** Configure the operating system, software, network, and applications.
- **Integration:** Works with AWS services such as EBS, S3, IAM, VPC, CloudWatch, and Elastic Load Balancing.
- **High Availability Options:** Deploy instances across multiple Availability Zones.

## 3. Disadvantages of EC2

- **Management Overhead:** Users are responsible for configuring and maintaining the guest operating system.
- **Security Responsibility:** Operating systems, applications, access controls, and security groups must be configured correctly.
- **Cost Management:** Unused instances, storage volumes, snapshots, and network resources can generate charges.
- **Scaling Complexity:** Automatic scaling requires suitable configuration and monitoring.
- **Availability Planning:** A single instance can become a single point of failure.
- **Technical Knowledge:** Requires knowledge of Linux or Windows, networking, security, and troubleshooting.

## 4. Responsibilities in EC2

| AWS Responsibilities | Customer Responsibilities |
|---|---|
| Physical data centers | Guest operating system configuration |
| Physical servers and hardware | OS updates and security patches |
| Underlying virtualization infrastructure | Application installation and maintenance |
| Physical network infrastructure | Security group and network configuration |
| Infrastructure-level availability | Application availability and recovery planning |

**Key concept:** EC2 follows the AWS Shared Responsibility Model. AWS secures the underlying cloud infrastructure, while customers secure and manage their workloads within it.

Reference: [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)

---

## 5. When to Use EC2

- Hosting websites, web applications, and APIs.
- Running applications that require operating system-level access.
- Deploying custom software and legacy applications.
- Running databases or specialized workloads that need server control.
- Performing development, testing, and DevOps experiments.
- Running applications that need specific CPU, memory, GPU, or networking configurations.

## 6. When Not to Use EC2

- **Event-driven functions:** Consider AWS Lambda for short-lived, event-triggered workloads.
- **Containers without server management:** Consider ECS with AWS Fargate.
- **Simple managed web application deployment:** Consider AWS App Runner or Elastic Beanstalk.
- **Static websites:** Consider Amazon S3 with CloudFront.
- **Workloads requiring minimal server administration:** Prefer a suitable managed AWS service when it meets the requirements.

**Note:** EC2 is not inherently unsuitable for these workloads. Other services may reduce operational effort.

---

# 7. Steps to Create an EC2 Instance

## Step 1: Open the EC2 Console

1. Sign in to the AWS Management Console.
2. Open the Amazon EC2 service.
3. Select an appropriate AWS Region.
4. Choose **Instances → Launch instances**.

![AWS EC2 Console](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/EC2-instance.png)

[Official Guide: Get Started with EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)

## Step 2: Configure the Instance

1. Enter an instance name.
2. Select an Amazon Machine Image (AMI), such as Amazon Linux or Ubuntu.
3. Choose an instance type based on CPU and memory requirements.
4. Create or select a key pair if using SSH key-based access.
5. Select the VPC and subnet.
6. Configure the security group.
7. Configure the root EBS volume and any additional storage.

![EC2 Instance Components](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/instance-explained.png)

[Official Guide: Launch an EC2 Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html)

## Step 3: Configure the Security Group

Security groups act as virtual firewalls for EC2 instances.

Typical rules:

| Protocol | Port | Purpose |
|---|---:|---|
| SSH | 22 | Linux administration |
| HTTP | 80 | Web traffic |
| HTTPS | 443 | Encrypted web traffic |
| RDP | 3389 | Windows remote administration |

- Allow SSH or RDP only from trusted sources.
- Allow HTTP and HTTPS when the application needs public web access.
- Do not expose administrative ports to everyone unless there is a specific, justified need.

![AWS Security Group Rules](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/security-group.png)

[Official Guide: Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/creating-security-group.html)

## Step 4: Launch the Instance

1. Review the selected AMI, instance type, networking, security group, and storage.
2. Choose **Launch instance**.
3. Open **Instances**.
4. Wait until the instance state becomes `Running`.
5. Confirm that its status checks pass.

## Step 5: Connect to the Instance

### Option A: SSH from Linux

```bash
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@PUBLIC_IP
```

Replace `my-key.pem` and `PUBLIC_IP` with your private key filename and instance public IP address.

The username depends on the selected AMI. For Ubuntu, the default username is commonly `ubuntu`; for Amazon Linux, it is commonly `ec2-user`.

### Option B: EC2 Instance Connect

1. Select the instance in the EC2 console.
2. Choose **Connect**.
3. Select **EC2 Instance Connect**, if supported and configured.
4. Choose **Connect**.

[Official Guide: Connect to Your EC2 Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect.html)

## Step 6: Verify the Instance

Run these commands after connecting to a Linux instance:

```bash
# Display operating system information
cat /etc/os-release

# Display CPU information
lscpu

# Display memory usage
free -h

# Display disk and filesystem usage
df -h

# Display IP addresses
ip addr
```

## Step 7: Stop or Terminate the Instance

- **Stop:** Shuts down the instance while normally preserving EBS volumes. Storage charges may continue.
- **Start:** Starts a stopped instance again. Its public IPv4 address may change unless a persistent address is configured.
- **Reboot:** Restarts the operating system.
- **Terminate:** Permanently deletes the instance. EBS volume deletion depends on the volume's termination settings.

**Important:** Review attached volumes and their deletion settings before terminating an instance.

[Official Guide: EC2 Instance Lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html)

---

# 8. Important EC2 Components

| Component | Purpose |
|---|---|
| AMI | Template used to launch an instance |
| Instance Type | Defines compute, memory, and networking capabilities |
| Key Pair | Provides credentials for supported connection methods |
| VPC | Logically isolated virtual network |
| Subnet | IP address range within a VPC |
| Security Group | Controls inbound and outbound traffic |
| EBS | Persistent block storage |
| Elastic IP | Static public IPv4 address that can be associated with supported resources |
| IAM Role | Grants AWS permissions to applications running on an instance |
| CloudWatch | Collects metrics and supports monitoring and alarms |
| Auto Scaling | Adjusts EC2 capacity based on configured policies |

---

# 9. EC2 Interview Questions and Answers

### Basic Questions

**1. What is Amazon EC2?**

**2. What is an EC2 instance?**

**3. What is an AMI?**

**4. What is an instance type?**

**5. What is a key pair?**

**6. What is a security group?**

### Intermediate Questions

**7. What is the difference between stopping and terminating an instance?**

**8. What is the difference between a public IP and a private IP?**

**9. What is the difference between EBS and instance store?**

**10. What is Auto Scaling?**

**11. What is an Elastic IP?**

**12. What is the difference between a reboot and a stop/start operation?**

### Scenario-Based Questions

**13. An EC2 instance is running but cannot be accessed through SSH. What would you check?**

**14. How would you improve the availability of an EC2-hosted website?**

**15. How can you reduce EC2 costs?**

**16. How would you securely allow an EC2 application to access S3?**

**17. Can two EC2 instances share the same EBS volume?**

**18. What happens when an EC2 instance's workload increases?**


---

# 10. Revision Checklist

- [ ] Understand EC2, AMIs, and instance types.
- [ ] Launch a Linux instance.
- [ ] Configure a secure key pair and security group.
- [ ] Connect through SSH or EC2 Instance Connect.
- [ ] Understand EBS volumes and snapshots.
- [ ] Practice stopping, starting, rebooting, and terminating.
- [ ] Understand public IPs, private IPs, VPCs, and subnets.
- [ ] Learn IAM roles and least-privilege permissions.
- [ ] Explore CloudWatch monitoring and Auto Scaling.
- [ ] Review billing and remove unused learning resources.

## Official Resources

- [Amazon EC2 User Guide](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/)
- [Launch an EC2 Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/LaunchingAndUsingInstances.html)
- [EC2 Best Practices](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-best-practices.html)
- [Amazon EC2 Pricing](https://aws.amazon.com/ec2/pricing/)
- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
