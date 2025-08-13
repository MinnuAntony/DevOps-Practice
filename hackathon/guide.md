
* **Terraform** → EC2 with your name in tags & hostname-friendly naming
* **Shell script** → Only the steps needed for your case, minimal but clear comments on where to change things
* **Execution order** → So you don’t get lost mid-way

---

## **1️⃣ Terraform Code — Create EC2 for Kubernetes Master**

Save this as `ec2-master.tf`

```hcl
terraform {
  required_version = ">= 1.0.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region                  = "ap-south-1" # Change region if needed
  shared_credentials_files = ["~/.aws/credentials"]
}

resource "aws_instance" "k8s_master" {
  ami           = "ami-0abcdef1234567890"  # <-- Change this to the Ubuntu AMI ID for your region
  instance_type = "t2.medium"              # Medium for kubeadm master

  key_name = "your-keypair-name"           # <-- Change to your key pair name in AWS

  tags = {
    Name = "Minnu-K8s-Master"              # Tag with your name to avoid confusion
  }
}
```

**💡 Note:**

* To find AMI:

  ```bash
  aws ec2 describe-images --owners 099720109477 --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-*-20.04-amd64-server-*" --region ap-south-1 --query 'Images[*].[ImageId,Name]' --output table
  ```

---

## **2️⃣ Shell Script — Master Node Setup**

Save this as `master-setup.sh`

```bash
#!/bin/bash
# ===== Kubernetes Master Node Setup =====
# Make sure you run this as root or with sudo

# 1. Set hostname (change "kmaster" if needed)
hostnamectl set-hostname kmaster

# 2. Initialize Kubernetes cluster
# Replace <your-private-ip> with this EC2's private IP
# Replace pod-network-cidr if you use a different CNI
kubeadm init \
  --apiserver-advertise-address=<your-private-ip> \
  --pod-network-cidr=192.168.0.0/16

# 3. Configure kubectl for the ubuntu user
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config

# 4. Install Weave Net CNI (matches the pod CIDR above)
kubectl apply -f "https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml"

# 5. (Optional) Remove control-plane taint so master can run workloads
# Skip if your trainer says so
kubectl describe node | grep Taint
kubectl get node
kubectl taint node ip-172-31-80-216 node-role.kubernetes.io/control-plane:NoSchedule-


#kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

**💡 Where to change things:**

* `<your-private-ip>` → Get it by:

  ```bash
  hostname -I | awk '{print $1}'
  ```
* CIDR `192.168.0.0/16` → Must match your chosen CNI plugin

---

## **3️⃣ Execution Order**

1. **Run Terraform** to create EC2

   ```bash
   terraform init
   terraform apply -auto-approve
   ```
2. **SSH into the EC2**

   ```bash
   ssh -i your-key.pem ubuntu@<public-ip>
   ```
3. **(If image is pre-baked)** → Just run the shell script:

   ```bash
   chmod +x master-setup.sh
   ./master-setup.sh
   ```
4. **Verify cluster is running**

   ```bash
   kubectl get nodes
   ```

---

If you want, I can also make a **worker node Terraform + join script** so you can expand later.
Do you want me to add that too?
