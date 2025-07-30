# Amazon EKS (Elastic Kubernetes Service)

## What is EKS?

- **EKS** is AWS's **managed Kubernetes service**.
- It allows you to run Kubernetes clusters without managing the control plane infrastructure.
- **Managed Control Plane**:
  - AWS provisions and manages all **master nodes**.
  - Installs and maintains core Kubernetes components:
    - API Server
    - Scheduler
    - Controller Manager
    - etcd (key-value store)
- **Customer Responsibility**:
  - You only manage the **worker nodes** (the nodes where your application runs).
  - AWS handles high availability, scalability, and upgrades for the control plane.
