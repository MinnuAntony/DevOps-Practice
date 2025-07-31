# VPC

In AWS a VPC is a logically isolated portion  of the cloud into which you can deploy your own resources such as Amazon EC2 instances.

# IPv4 Addressing Primer
## What is an IP Address?

An **IP address** (Internet Protocol address) is a unique identifier assigned to a device on a network. It is used by computers to communicate with each other.

---

## Web Server and IP Address

- When a **web server** is available on the internet, it will have a **network interface**.
- Attached to that interface is an **IP address**.
- For internet-facing services like a web server, this IP address is typically a **public IP address**.

---

## Client-Side Communication Flow

1. **You** are on your computer at home.
2. You open a **web browser** and type in a domain name like `example.com`.
3. While humans use names like `example.com`, computers use IP addresses.

---

## Role of DNS (Domain Name System)

- DNS is responsible for translating **domain names** into **IP addresses**.
- Your computer contacts a **DNS server** and asks:  
  “What is the IP address of `example.com`?”
- Once it gets the IP address, your computer can then connect directly to the web server.

---

## Why This Matters

- **Domain names** are easier for humans to remember.
- **IP addresses** are used behind the scenes for actual communication between devices.

This process is called **name resolution** — converting a user-friendly domain name into a computer-friendly IP address.


--------

Certainly. Here is a clean, professional version of the notes **without emojis**, well-structured for understanding and future reference:

---

# Understanding the Structure of an IP Address

## 1. What is an IP Address?

An IP address is a unique identifier assigned to a device on a network, used for communication.
Example: `192.168.0.1`

This format is called **Dotted Decimal Notation**, where the IP address is represented as four numbers separated by dots.

---

## 2. Dotted Decimal Notation and Binary Octets

Each of the four parts (called **octets**) is an 8-bit binary number.

* Each octet can have a value from 0 to 255.
* Each bit within an octet represents a value based on its position:

| Bit Position (from left) | 1   | 2  | 3  | 4  | 5 | 6 | 7 | 8 |
| ------------------------ | --- | -- | -- | -- | - | - | - | - |
| Binary Weight            | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

### Example Breakdown:

* `192` in binary is `11000000` → 128 + 64 = 192
* `168` = 128 + 32 + 8 = 168
* `0` = `00000000`
* `1` = `00000001`

Understanding this binary structure is important for subnetting and address calculations.

---

## 3. Network ID and Host ID

Every IP address is divided into two parts:

* **Network ID**: Identifies the network the device belongs to.
* **Host ID**: Identifies the specific device within the network.

All devices on the same network share the same Network ID, but have different Host IDs.

---

## 4. Subnet Masks

A **subnet mask** determines which part of an IP address is the network portion and which is the host portion.

### Example:

* Subnet Mask: `255.255.255.0`
  In binary: `11111111.11111111.11111111.00000000`

Here:

* Bits set to **1** represent the **network portion**.
* Bits set to **0** represent the **host portion**.

So, the first 3 octets are the network, and the last octet is used for identifying hosts.

---

## 5. CIDR Notation

Instead of writing the full subnet mask, we use **CIDR (Classless Inter-Domain Routing)** notation.

* Example: `192.168.0.0/24`
  This means the first 24 bits are for the network.

CIDR is widely used in cloud platforms such as AWS.

---

## 6. IP Address Classes (Classful Addressing)

Historically, IP addresses were divided into five classes. The most used are Class A, B, and C.

| Class | Starting Address | Ending Address  | Default Subnet Mask | Number of Hosts  |
| ----- | ---------------- | --------------- | ------------------- | ---------------- |
| A     | 1.0.0.0          | 126.255.255.255 | 255.0.0.0 (/8)      | 16,777,214 hosts |
| B     | 128.0.0.0        | 191.255.255.255 | 255.255.0.0 (/16)   | 65,534 hosts     |
| C     | 192.0.0.0        | 223.255.255.255 | 255.255.255.0 (/24) | 254 hosts        |

* `127.x.x.x` is reserved for loopback (localhost).
* First and last addresses in any subnet (all 0s and all 1s) are **not assignable**.

---

## 7. Private IP Address Ranges

These IP ranges are reserved for internal network use and cannot be routed on the public internet.

| Class | Private IP Range                  |
| ----- | --------------------------------- |
| A     | `10.0.0.0` – `10.255.255.255`     |
| B     | `172.16.0.0` – `172.31.255.255`   |
| C     | `192.168.0.0` – `192.168.255.255` |

These ranges are commonly used in homes, organizations, and VPCs in cloud platforms.

---

## 8. Classless Inter-Domain Routing (CIDR)

CIDR allows more flexible IP allocation compared to traditional class-based addressing.

* It lets us **customize subnet masks** beyond fixed class rules.
* It helps avoid IP wastage by assigning only as many addresses as needed.
* CIDR is expressed using a suffix like `/24`, `/20`, etc., indicating how many bits are used for the network.

---


# VPC (Virtual Private Cloud)

A **logically isolated network** in AWS (like your own private data center).

- You choose the **CIDR range** (e.g., `10.0.0.0/24`).

## Inside a VPC, you have:

- **Subnets** (Public & Private)
- **Route tables**
- **Gateways** (Internet Gateway / NAT Gateway)
- **Security controls** (Security Groups, Network ACLs)

---

##  Subnets

A **sub‑division** of your VPC IP range.

- **Public subnet** → Can access the internet via **Internet Gateway**.
- **Private subnet** → No direct internet access  
  (can use **NAT Gateway** or **Bastion Host**).

---

##  Bastion Host

- An **EC2 instance** in the **public subnet** used to SSH into private subnet instances.
- Improves security by **keeping private resources hidden** from the internet.

