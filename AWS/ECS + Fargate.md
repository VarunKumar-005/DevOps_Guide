# Amazon ECS, AWS Fargate and Related Container Services

## 1. What Is Amazon ECS?

Amazon Elastic Container Service (ECS) is a fully managed container orchestration service. It deploys, manages, and scales containerized applications without requiring you to run your own cluster management software.

ECS has three layers:

- **Capacity:** the infrastructure where containers run.
- **Controller:** the ECS scheduler, which deploys and manages the applications running on containers.
- **Provisioning:** the tools you use to interact with the scheduler (console, CLI, SDKs, CDK).

![The three Amazon ECS layers: capacity, controller, and provisioning.](https://docs.aws.amazon.com/de_de/AmazonECS/latest/developerguide/images/ecs-layers.png)

*Figure: Amazon ECS layers. Source: [What is Amazon ECS?](https://docs.aws.amazon.com/de_de/AmazonECS/latest/developerguide/Welcome.html)*

### Capacity options

| Capacity | Who manages servers? | Typical use |
|---|---|---|
| **AWS Fargate** | AWS (serverless) | Most web apps, APIs, microservices, batch jobs |
| **Amazon EC2** | You | Custom AMIs, GPUs, daemon tasks, cost tuning at scale |
| **ECS Managed Instances** | AWS manages the EC2 instances | Need specific EC2 instance types without patching and scaling them yourself |
| **ECS Anywhere (external)** | You | On-premises servers or VMs registered to a cluster |

**Official documentation:** [Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)

---

## 2. Core ECS Concepts

| Concept | Meaning |
|---|---|
| **Container image** | Packaged application, usually stored in Amazon ECR |
| **Task definition** | Blueprint (JSON) describing images, CPU, memory, ports, roles, logging, and volumes |
| **Task** | A running instance of a task definition (for example a batch job that runs and exits) |
| **Service** | Keeps a desired number of tasks running (long-running apps), replaces failed tasks, and integrates with load balancers |
| **Cluster** | Logical grouping of tasks, services, and capacity |
| **Container agent** | Runs on EC2 container instances to communicate with ECS (not needed on Fargate) |
| **Capacity provider** | Links a cluster to capacity (Fargate, Fargate Spot, or an EC2 Auto Scaling group) |

---

## 3. Advantages of ECS

- **Fully managed control plane:** no control plane to install, patch, or scale.
- **Choice of capacity:** run on Fargate, EC2, or Managed Instances.
- **Deep AWS integration:** works with IAM, VPC, ALB/NLB, CloudWatch, ECR, Secrets Manager, and EventBridge.
- **Simple model:** fewer concepts than Kubernetes (cluster, task definition, service).
- **Built-in scaling:** Service Auto Scaling adjusts task count; cluster auto scaling manages EC2 capacity.
- **No extra orchestration fee:** you pay for the underlying capacity you use.

## 4. Disadvantages of ECS

- **AWS lock-in:** ECS is proprietary, so task definitions are not portable to other clouds (unlike Kubernetes).
- **Smaller ecosystem:** fewer third-party tools and add-ons than Kubernetes.
- **Learning curve:** networking (awsvpc), IAM roles (task role vs execution role), and task definitions take time to learn.
- **Capacity trade-offs:** EC2 launch type means you manage servers; Fargate limits low-level control.
- **Cost surprises:** idle services, NAT gateways, load balancers, and logs add cost.

---

## 5. Shared Responsibility: ECS on EC2 vs ECS on Fargate

| Responsibility | ECS on EC2 | ECS on Fargate |
|---|---|---|
| Physical infrastructure | AWS | AWS |
| Control plane (scheduler) | AWS | AWS |
| Host OS patching and AMI | **You** | AWS |
| Cluster capacity and scaling of instances | **You** | AWS |
| Container image and app code | You | You |
| Task definition, CPU, and memory sizing | You | You |
| IAM roles, security groups, and network design | You | You |
| Logging and monitoring setup | You | You |

**Key concept:** with Fargate, AWS takes over the servers and host OS. You still own the container image, permissions, networking, and application security.

---

## 6. When to Use ECS and When Not To

### Use ECS when

- Running containerized web apps, APIs, and microservices on AWS.
- Running scheduled or batch container jobs.
- You want container orchestration without managing Kubernetes.
- You want tight AWS integration with minimal operational overhead.

### Consider alternatives when

- **Kubernetes portability or ecosystem is required:** use Amazon EKS.
- **Short, event-driven functions:** use AWS Lambda.
- **A simple managed web service from source or image:** use AWS App Runner.
- **Small, predictable projects at a flat monthly price:** consider Amazon Lightsail.
- **Static websites:** use Amazon S3 with CloudFront.

---

# 7. AWS Fargate in Detail

## 7.1 What Is Fargate?

AWS Fargate is a serverless compute engine for containers that works with Amazon ECS (and EKS). You no longer provision, configure, or scale clusters of virtual machines. There is no need to choose server types, decide when to scale, or optimize cluster packing.

You package your application in containers, specify CPU and memory, define networking and IAM policies, and launch it. AWS provisions the compute.

## 7.2 How Fargate Works

1. Build a container image and push it to Amazon ECR.
2. Create a **task definition** with `requiresCompatibilities` set to `FARGATE`, plus task-level CPU and memory.
3. Create an **ECS cluster** (a logical grouping; no servers to add).
4. Run a **task** or create a **service** with the Fargate launch type or a Fargate capacity provider.
5. Fargate provisions an isolated compute environment, attaches an elastic network interface (ENI) in your VPC, pulls the image, and starts the containers.
6. When the task stops, its compute and ephemeral storage are released.

## 7.3 Task Isolation

Each Fargate task has its own isolation boundary. It does **not** share the underlying kernel, CPU resources, memory resources, or elastic network interface with another task.

## 7.4 Task CPU and Memory Sizes

Fargate requires CPU and memory to be set at the **task level**. Only these combinations are valid:

| CPU | Memory | OS support |
|---|---|---|
| 256 (0.25 vCPU) | 512 MiB, 1 GB, 2 GB | Linux |
| 512 (0.5 vCPU) | 1 GB to 4 GB | Linux |
| 1024 (1 vCPU) | 2 GB to 8 GB | Linux, Windows |
| 2048 (2 vCPU) | 4 GB to 16 GB (1 GB steps) | Linux, Windows |
| 4096 (4 vCPU) | 8 GB to 30 GB (1 GB steps) | Linux, Windows |
| 8192 (8 vCPU)* | 16 GB to 60 GB (4 GB steps) | Linux |
| 16384 (16 vCPU)* | 32 GB to 120 GB (8 GB steps) | Linux |
| 32768 (32 vCPU)* | 60 GB, 120 GB, 244 GB | Linux |

*Requires Linux platform version 1.4.0 or later.

## 7.5 Operating Systems and CPU Architectures

- **Linux:** Amazon Linux 2 based platform; supports **X86_64** and **ARM64** (Graviton).
- **Windows:** Windows Server 2019 and 2022 (Full and Core); **X86_64 only**, minimum 1 vCPU.

## 7.6 Networking on Fargate

- Fargate tasks **always use the `awsvpc` network mode**. Each task gets its own ENI.
- You must specify **subnets** and **security groups** for the task.
- To pull images from the internet, a task needs a public IP (public subnet) or a **NAT gateway** (private subnet). Alternatively, use **ECR interface VPC endpoints (AWS PrivateLink)** so image pulls stay private.
- For load balancer target groups, choose target type **`ip`** (not `instance`).

```json
"networkConfiguration": {
   "awsvpcConfiguration": {
      "assignPublicIp": "ENABLED",
      "securityGroups": [ "sg-12345678" ],
      "subnets": [ "subnet-12345678" ]
   }
}
```

## 7.7 Storage on Fargate

| Storage | Persistence | Use |
|---|---|---|
| **Ephemeral task storage (bind mounts)** | Deleted when the task stops | Scratch data, temp files |
| **Amazon EFS** | Persistent, shared across tasks | Shared files, CMS uploads |
| **Amazon EBS** | Durable block storage attached to a task | Data-intensive workloads |

## 7.8 Logging and Monitoring

- Supported log drivers: **`awslogs`** (CloudWatch Logs), **`splunk`**, and **`awsfirelens`**.
- ECS events go to **EventBridge** for near real-time notifications.
- Tasks publish lifecycle timestamps (for example image pull time) that help you tune startup.
- CloudWatch usage metrics track Fargate usage against service quotas.

```json
"logConfiguration": {
   "logDriver": "awslogs",
   "options": {
      "awslogs-group": "/ecs/fargate-task-definition",
      "awslogs-region": "us-east-1",
      "awslogs-stream-prefix": "ecs"
   }
}
```

## 7.9 IAM Roles for Fargate Tasks

| Role | Used by | Purpose |
|---|---|---|
| **Task execution role** | ECS/Fargate agent | Pull images from ECR, write logs to CloudWatch, read secrets for injection |
| **Task role** | Your application code | Call AWS services such as S3 or DynamoDB with least privilege |

## 7.10 Fargate Spot

Fargate Spot runs **interruption-tolerant** tasks on spare capacity at a discount compared to regular Fargate. When AWS needs the capacity back, tasks are interrupted with a **two-minute warning**. Use it for batch jobs, queue workers, CI tasks, and stateless workers. Do not use it for the only copy of a critical service.

## 7.11 Platform Versions

A Fargate platform version is a specific runtime environment (kernel plus container runtime versions).

- New revisions are released for kernel/OS updates, features, and security fixes.
- A running task stays on one revision for its whole life; **new tasks always get the latest revision**.
- If a security issue is found, AWS can schedule affected tasks for **retirement**, so design services to tolerate task replacement.

## 7.12 Faster Startup with SOCI (Lazy Loading)

Seekable OCI (SOCI) lets Fargate start containers **before the whole image is downloaded**.

- Requires Linux platform version 1.4.0 and images in Amazon ECR.
- Needs a SOCI index stored beside the image.
- Most useful for images larger than about 250 MiB compressed.

## 7.13 Load Balancing with Fargate

| Load balancer | Layer | Use |
|---|---|---|
| **Application Load Balancer (ALB)** | 7 (HTTP/HTTPS) | Web apps and APIs, path-based routing (recommended default) |
| **Network Load Balancer (NLB)** | 4 (TCP/UDP) | High-performance TCP/UDP; UDP needs platform 1.4+ |
| **Gateway Load Balancer (GWLB)** | 3/4 | Firewalls and inspection appliances |

## 7.14 Fargate Limitations

These task definition parameters are **not supported** on Fargate: `privileged`, `gpu`, `placementConstraints`, `dockerSecurityOptions`, `ipcMode`, `links`, `extraHosts`, `dnsServers`, `dnsSearchDomains`, `disableNetworking`, `maxSwap`, and `swappiness`.

Other limits:

- No privileged containers and no direct host access.
- Only the `CAP_SYS_PTRACE` Linux capability can be added.
- `devices`, `sharedMemorySize`, and `tmpfs` are not supported.
- Only bind mount volumes are supported; `dockerVolumeConfiguration` is not.
- No GPU support in task definitions.

## 7.15 Advantages of Fargate

- **No server management:** no patching, AMI maintenance, or cluster packing.
- **Right-sized billing:** pay for the CPU and memory requested by tasks, billed per second (one-minute minimum).
- **Strong task isolation:** one task per isolation boundary.
- **Quick scaling:** add tasks without scaling instances first.
- **Less operational work:** a good fit for small teams and DevOps pipelines.

## 7.16 Disadvantages of Fargate

- **Less control:** no host access, privileged mode, or GPUs.
- **Higher unit price at steady scale:** well-utilized EC2 or Reserved/Savings Plans capacity can cost less for constant, heavy workloads.
- **Task size limits:** fixed CPU/memory combinations.
- **Cold-start time:** image pulls and ENI setup add startup delay.
- **Network cost items:** NAT gateways, public IPv4 addresses, and data transfer can add up.

---

# 8. Fargate vs EC2 vs Managed Instances

| Feature | Fargate | ECS on EC2 | ECS Managed Instances |
|---|---|---|---|
| Server management | None | You manage instances | AWS manages instances |
| Instance type choice | No (task-level CPU/memory) | Full | Wide EC2 instance range |
| GPU support | No | Yes | Yes |
| Privileged containers | No | Yes | Depends on configuration |
| Scaling | Per task | Tasks plus instances | Tasks plus managed capacity |
| Pricing basis | vCPU and memory per task | EC2 instance hours | EC2 instance based |
| Operational effort | Lowest | Highest | Medium |
| Best for | Simplicity, spiky load, small teams | Cost tuning, special hardware | EC2 features without OS patching |

![An Auto Scaling group within a VPC.](https://docs.aws.amazon.com/images/autoscaling/ec2/userguide/images/asg-basic-arch.png)

*Figure: With the EC2 launch type, ECS cluster capacity is backed by an EC2 Auto Scaling group like this one. Fargate removes this layer. Source: [What is Amazon EC2 Auto Scaling?](https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html)*

---

# 9. Service Combinations Used with ECS and Fargate

| Service | Role in the architecture |
|---|---|
| **Amazon ECR** | Stores and scans container images |
| **Elastic Load Balancing (ALB/NLB)** | Distributes traffic across tasks |
| **Amazon VPC** | Subnets, route tables, security groups, NAT gateway, VPC endpoints |
| **AWS IAM** | Task execution role, task role, and user permissions |
| **Amazon CloudWatch** | Logs, metrics, alarms, Container Insights |
| **Application Auto Scaling** | Service Auto Scaling for task count (target tracking on CPU, memory, or request count) |
| **AWS Secrets Manager / SSM Parameter Store** | Inject secrets and configuration into containers |
| **Amazon EFS / EBS** | Persistent storage for tasks |
| **Amazon EventBridge** | Scheduled tasks and ECS state change events |
| **Amazon Route 53** | DNS for load balancers |
| **AWS CloudFormation / CDK / Terraform** | Infrastructure as code |
| **CodePipeline / GitHub Actions** | CI/CD that builds, pushes, and deploys new task definition revisions |
| **AWS GuardDuty** | Detects suspicious activity on containers and workloads |
| **AWS Backup** | Back up EFS and EBS data |

## Common Architecture Patterns

1. **Web application:** Route 53, then ALB (public subnets), then Fargate service (private subnets), then RDS or DynamoDB.
2. **CI/CD pipeline:** GitHub Actions or CodePipeline builds the image, pushes it to ECR, registers a new task definition, and updates the ECS service (rolling deployment).
3. **Batch or queue worker:** SQS or EventBridge triggers Fargate (or Fargate Spot) tasks to process jobs and exit.
4. **Hybrid capacity:** one cluster using a Fargate capacity provider for baseline traffic and Fargate Spot for burst or batch work.
5. **Observability:** CloudWatch Logs plus Container Insights, or Prometheus and Grafana for custom metrics.

---

# 10. Comparing ECS with Other AWS Compute Services

| Service | Best for | Management effort |
|---|---|---|
| **EC2** | Full server control | High |
| **ECS + EC2** | Containers with custom instances | Medium to high |
| **ECS + Fargate** | Containers without servers | Low |
| **EKS** | Kubernetes workloads and portability | Medium to high |
| **App Runner** | Simple web apps and APIs from image or source | Very low |
| **Lambda** | Short event-driven functions | Very low |
| **Lightsail** | Small, predictable-cost projects | Low |

---

# 11. Steps to Run a Container on ECS with Fargate (Console)

## Step 1: Prepare Prerequisites

1. Have an AWS account with permissions to use ECS, IAM, and VPC.
2. Make sure a **task execution role** is available (the console can create it if you have the permissions).
3. Prepare a container image (this tutorial uses a public `httpd` image).
4. Make sure the security group allows **inbound HTTP on port 80**. Allowing `0.0.0.0/0` is acceptable only for short test environments.

## Step 2: Create the Cluster

1. Open the ECS console.
2. Choose **Clusters → Create cluster**.
3. Enter a unique cluster name.
4. (Optional) Turn on **Container Insights** and add tags.
5. Choose **Create**.

## Step 3: Create a Task Definition

1. Choose **Task definitions → Create new task definition**.
2. Select **Fargate** as the launch type, with `awsvpc` networking.
3. Set task size (for example 0.25 vCPU and 512 MiB).
4. Add the container name, image, and port mappings.
5. Choose **Create**.

Example task definition:

```json
{
  "family": "sample-fargate",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "fargate-app",
      "image": "public.ecr.aws/docker/library/httpd:latest",
      "portMappings": [
        { "containerPort": 80, "hostPort": 80, "protocol": "tcp" }
      ],
      "essential": true
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

## Step 3.1: (Optional) Build and Push Your Own Image to ECR

```bash
# Create a repository
aws ecr create-repository --repository-name my-app

# Authenticate Docker to ECR
aws ecr get-login-password --region REGION | \
  docker login --username AWS --password-stdin ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com

# Build, tag, and push
docker build -t my-app .
docker tag my-app:latest ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/my-app:latest
docker push ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/my-app:latest
```

Replace `REGION` and `ACCOUNT_ID` with your values.

## Step 4: Create the Service

1. Open your cluster and choose the **Services** tab, then **Create**.
2. Select the task definition family and enter a service name.
3. Under **Environment**, choose launch type **FARGATE**.
4. Set **Desired tasks** to `1` (or more).
5. Under **Networking**, choose subnets and a security group that allows the app port.
6. (Optional) Attach an **Application Load Balancer** with an `ip` target group.
7. Choose **Create**.

## Step 5: Verify the Service

1. Open the cluster, then the service, then the **Tasks** tab.
2. Open the task and check that its status is `RUNNING`.
3. Under **Configuration**, open the **Public IP** address to view the app.
4. Check logs in CloudWatch Logs.

## Step 6: Update, Scale, or Stop

- **Update:** register a new task definition revision and update the service (rolling deployment).
- **Scale:** change desired count or configure Service Auto Scaling.
- **Stop:** set desired count to `0`, or delete the service.
- **Clean up:** delete the service, then the cluster, and remove any load balancers, NAT gateways, and unused log groups to avoid charges.

**Official guide:** [Create an Amazon ECS Linux task for Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)

---

# 12. Steps to Run a Fargate Task with the AWS CLI

```bash
# 1. Create a cluster
aws ecs create-cluster --cluster-name fargate-cluster

# 2. Register a task definition
aws ecs register-task-definition --cli-input-json file://fargate-task.json

# 3. List task definitions
aws ecs list-task-definitions

# 4. Create a service on Fargate
aws ecs create-service \
  --cluster fargate-cluster \
  --service-name fargate-service \
  --task-definition sample-fargate:1 \
  --desired-count 2 \
  --launch-type FARGATE \
  --network-configuration "awsvpcConfiguration={subnets=[subnet-abcd1234],securityGroups=[sg-abcd1234]}"

# 5. Check the service
aws ecs list-services --cluster fargate-cluster
aws ecs describe-services --cluster fargate-cluster --services fargate-service

# 6. Clean up
aws ecs update-service --cluster fargate-cluster --service fargate-service --desired-count 0
aws ecs delete-service --cluster fargate-cluster --service fargate-service --force
aws ecs delete-cluster --cluster fargate-cluster
```

Replace the subnet and security group IDs with your own.

**Official guide:** [Create a Fargate task with the AWS CLI](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_AWSCLI_Fargate.html)

---

# 13. Best Practices for ECS and Fargate

- Run tasks in **private subnets** behind an ALB; use NAT gateways or VPC endpoints for outbound access.
- Use **least-privilege task roles**; keep the execution role separate from the task role.
- Store secrets in **Secrets Manager or SSM Parameter Store**, never in images or plain environment variables.
- Right-size task CPU and memory using CloudWatch metrics.
- Use **multiple Availability Zones** and set a minimum of 2 tasks for production services.
- Use **Fargate Spot** only for interruption-tolerant work.
- Keep images small, scan them in ECR, and consider **SOCI** for large images.
- Send logs to CloudWatch and set alarms on service CPU, memory, and unhealthy task counts.
- Design for **task replacement** (platform version retirement and Spot interruptions).
- Define everything with **IaC** (CloudFormation, CDK, or Terraform).
- Use **tags** and **cost allocation** to track spend per service.

---

# 14. Cost Awareness

- **Fargate:** charged for the vCPU and memory resources your tasks request, per second with a one-minute minimum, plus any extra ephemeral storage.
- **ECS on EC2:** charged for the EC2 instances (and EBS) whether or not they are fully used.
- **Other charges:** load balancers, NAT gateways, data transfer, CloudWatch Logs, public IPv4 addresses, ECR storage, and EFS or EBS storage.
- Savings options: Fargate Spot, ARM64 (Graviton) tasks, right-sizing, scaling to zero for non-production environments, and Compute Savings Plans.

Pricing pages: [Amazon ECS pricing](https://aws.amazon.com/ecs/pricing) | [AWS Fargate pricing](https://aws.amazon.com/fargate/pricing)

---

# 15. ECS and Fargate Interview Questions and Answers

### Basic Questions

**1. What is Amazon ECS?**
A fully managed container orchestration service for deploying, managing, and scaling containerized applications on AWS.

**2. What is AWS Fargate?**
A serverless compute engine for containers. You define CPU, memory, networking, and IAM, and AWS provisions and manages the compute.

**3. What is a task definition?**
A JSON blueprint that describes container images, CPU, memory, ports, roles, logging, and volumes.

**4. What is the difference between a task and a service?**
A task is a running instance of a task definition (often short-lived). A service keeps a desired number of tasks running and replaces failed ones.

**5. What is an ECS cluster?**
A logical grouping of tasks, services, and capacity.

**6. What is Amazon ECR?**
A managed container image registry used to store, scan, and distribute images.

### Intermediate Questions

**7. What is the difference between the task execution role and the task role?**
The execution role is used by ECS/Fargate to pull images, write logs, and fetch secrets. The task role is used by your application code to call AWS services.

**8. Which network mode does Fargate use?**
Only `awsvpc`; every task gets its own ENI, subnet placement, and security groups.

**9. Why must you use target type `ip` for a Fargate load balancer target group?**
Because tasks in `awsvpc` mode have their own ENI and IP address and are not associated with an EC2 instance.

**10. What is Fargate Spot?**
Discounted Fargate capacity for interruption-tolerant tasks; tasks get a two-minute warning before interruption.

**11. What is the difference between Fargate and EC2 launch types?**
With Fargate, AWS manages the servers and you size tasks. With EC2, you manage instances (AMI, patching, scaling) and get more control, such as GPUs and privileged containers.

**12. How do you persist data for Fargate tasks?**
Use Amazon EFS (shared, persistent) or EBS volumes. Ephemeral task storage is deleted when the task stops.

### Scenario-Based Questions

**13. A Fargate task keeps stopping right after it starts. What would you check?**
Check the task's stopped reason, CloudWatch Logs, the container exit code, image pull errors (ECR permissions, NAT or VPC endpoint, public IP), the execution role, memory limits (OOM), and health check settings.

**14. A Fargate task in a private subnet cannot pull its image. Why?**
It needs a route to the internet through a NAT gateway or ECR interface VPC endpoints (plus S3 gateway endpoint for image layers), and the execution role must allow ECR access.

**15. How would you make an ECS web application highly available?**
Run at least two tasks across multiple Availability Zones behind an ALB, enable health checks, use Service Auto Scaling, and use a multi-AZ database.

**16. How can you reduce Fargate costs?**
Right-size CPU and memory, use Fargate Spot for tolerant workloads, use ARM64 tasks, scale down or stop non-production services, and consider Savings Plans.

**17. When would you choose EC2 over Fargate?**
When you need GPUs, privileged containers, specific instance types or host access, or when steady, high utilization makes reserved EC2 capacity cheaper.

**18. How do you deploy a new version of a service without downtime?**
Register a new task definition revision and update the service for a rolling deployment behind a load balancer with health checks, or use a blue/green deployment strategy.

**19. How do you securely give a container a database password?**
Store it in Secrets Manager or SSM Parameter Store and reference it in the task definition; the execution role fetches it at launch.

**20. ECS or EKS: how do you choose?**
Choose ECS for simplicity and tight AWS integration. Choose EKS when you need Kubernetes APIs, portability, or its ecosystem.

---

# 16. Revision Checklist

- [ ] Understand ECS layers: capacity, controller, provisioning.
- [ ] Explain clusters, task definitions, tasks, and services.
- [ ] Compare Fargate, EC2, and Managed Instances capacity.
- [ ] Remember valid Fargate CPU and memory combinations.
- [ ] Understand `awsvpc` networking, subnets, security groups, and `ip` target groups.
- [ ] Know the task execution role vs the task role.
- [ ] Push an image to ECR and run it on Fargate.
- [ ] Attach an Application Load Balancer to a Fargate service.
- [ ] Configure CloudWatch Logs and Container Insights.
- [ ] Set up Service Auto Scaling.
- [ ] Try Fargate Spot for a batch workload.
- [ ] Know Fargate limitations (no privileged mode, no GPU).
- [ ] Review costs and delete unused clusters, NAT gateways, and load balancers.

## Official Resources

- [Amazon ECS Developer Guide](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html)
- [Architect for AWS Fargate for Amazon ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/AWS_Fargate.html)
- [Task definition differences for Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-tasks-services.html)
- [Create an ECS Linux task for Fargate](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/getting-started-fargate.html)
- [Use load balancing with ECS services](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-load-balancing.html)
- [Amazon ECS pricing](https://aws.amazon.com/ecs/pricing)
- [AWS Fargate pricing](https://aws.amazon.com/fargate/pricing)
- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
