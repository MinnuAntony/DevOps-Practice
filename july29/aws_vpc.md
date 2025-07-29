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

