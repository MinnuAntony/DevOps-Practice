# AWS EC2 Dashboard – Theory Notes

The **EC2 Dashboard** in AWS Management Console is the central place to manage **Elastic Compute Cloud (EC2)** instances and related networking, security, and storage resources.

---

## 1. **EC2 Instances**
- **Definition:** Virtual machines running in AWS cloud.
- **Key actions:**
  - Launch, stop, start, reboot, terminate.
  - View instance details such as:
    - **AMI (Amazon Machine Image)** used
    - **Instance type** (e.g., t2.micro)
    - **State** (running, stopped, terminated)
    - **Public & Private IP addresses**
    - **Attached Security Groups**
- **Use cases:** Web servers, app servers, databases, bastion hosts, etc.

---

## 2. **AMI (Amazon Machine Images)**
- Preconfigured templates for launching instances.
- Contains:
  - OS (Linux, Windows, etc.)
  - Optional software (e.g., LAMP stack)
- Types:
  - AWS-provided (Amazon Linux, Ubuntu, Windows)
  - Marketplace AMIs
  - Custom AMIs created by you.

---

## 3. **Instance Types**
- Define **compute capacity** (vCPU, memory).
- Families:
  - **General Purpose** – Balanced CPU, memory (t2.micro, t3.small)
  - **Compute Optimized** – High CPU performance
  - **Memory Optimized** – Large RAM
  - **Storage Optimized** – High disk throughput
- Pricing depends on size and type.

---

## 4. **Key Pairs**
- Required for **SSH (Linux)** or **RDP (Windows)** access.
- `.pem` file for Linux SSH.
- `.ppk` file for Windows PuTTY.
- Region-specific – must be created in the same region as your EC2.

---

## 5. **Security Groups**
- Virtual firewalls for controlling inbound/outbound traffic.
- **Inbound rules:** Allow traffic **to** instance.
- **Outbound rules:** Allow traffic **from** instance.
- **Stateful** – if inbound is allowed, return traffic is automatically allowed.

---

## 6. **Elastic IPs**
- Static public IPv4 addresses.
- Useful for consistent DNS mapping.
- Free while attached to a **running** instance.
- Charged if allocated but **not attached**.

---

## 7. **Volumes (EBS – Elastic Block Store)**
- Persistent storage for EC2.
- Attached to an instance as a root volume or additional volumes.
- Can be detached and re-attached to other instances.
- Pay per GB provisioned per month.

---

## 8. **Snapshots**
- Backup of EBS volumes.
- Used to restore data or create new volumes.
- Stored in Amazon S3 (managed by AWS, not directly visible).

---

## 9. **Launch Templates**
- Predefined configurations for launching EC2 quickly.
- Contains:
  - AMI ID
  - Instance type
  - Key pair
  - Security group
  - Subnet/VPC settings
- Ensures consistency in multiple deployments.

---

## 10. **Placement Groups**
- Control instance placement across hardware for:
  - **Cluster** – Low-latency, high-bandwidth
  - **Spread** – Isolate instances across hardware
  - **Partition** – Large distributed workloads

---

## 11. **Tags**
- Key-value pairs for resource identification.
- Example:
  - `Name: Web-Server`
  - `Environment: Production`
- Useful for billing, automation, and organization.

---

## 12. **Monitoring**
- **Basic monitoring:** 5-minute metrics (CPU, network, disk).
- **Detailed monitoring:** 1-minute metrics (paid).
- Graphs available in dashboard for quick insight.

---

## 13. **Elastic Load Balancers (Optional from EC2 Dashboard)**
- Distribute traffic across multiple EC2 instances.
- Improve scalability & availability.
- Types: Application, Network, Gateway load balancers.

---

## 14. **Auto Scaling (Optional from EC2 Dashboard)**
- Automatically increase or decrease EC2 count based on load.
- Uses Launch Templates or Launch Configurat
