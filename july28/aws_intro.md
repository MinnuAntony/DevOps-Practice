# AWS Cloud Introduction
AWS (Amazon Web Services) is a cloud computing platform that lets you rent infrastructure (servers, storage, networking) instead of owning physical machines.

* Think of AWS as a **pay-as-you-go data center**.
* You can run web apps, store data, analyze logs, host databases, etc.
* Services are categorized as:
    * **Compute**: EC2 (virtual servers)
    * **Storage**: S3 (object storage), EBS (block storage)
    * **Networking**: VPC, Load Balancers
    * **Databases**: RDS, DynamoDB
    * **Security**: IAM (Identity Access Management)

---

# Networking Stack in AWS
The networking layer in AWS is how you isolate, route, and secure your infrastructure. It’s similar to how traditional networking works (IP, routers, firewalls), but in a cloud environment.

* **Layers**:
    * **VPC (Virtual Private Cloud)** – your private network in AWS.
    * **Subnets** – divide your VPC into smaller network zones.
    * **Route Tables** – define how traffic flows.
    * **Internet Gateway** – allows traffic between your VPC and the internet.
    * **NAT Gateway** – lets private subnets reach the internet securely.
    * **Security Groups / NACLs** – firewall-like rules for security.

---

# VPC (Virtual Private Cloud)
Think of a VPC as your own private data center in the cloud.

* You control IP ranges, subnets, routing, and security.
* Every AWS account gets a default VPC, or you can create custom ones.

**Example**:
* CIDR Block (IP range): `10.0.0.0/16` → provides ~65,000 IPs.

---

# Subnet
Subnets are slices of your VPC.

* They can be:
    * **Public**: has access to the internet via Internet Gateway.
    * **Private**: no direct access to the internet.

**Example**:
* Public Subnet: `10.0.1.0/24`
* Private Subnet: `10.0.2.0/24`

---

# Bastion Host (Jump Server)
A bastion host is a secure entry point to your private network.

* It is a public-facing EC2 instance used to SSH into instances in a private subnet.
* After logging into the bastion, you can SSH into internal machines.

**Why use it?**
* You don't expose all your instances to the internet.
* Central point for monitoring and security.
