
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


Thanks for sharing your app code and Dockerfile — looks clean and straightforward!

Here’s a quick **summary** of what you already have and what’s perfect for your CI setup:

---

### ✔️ Your Current Setup Recap:

* **Dockerfile**: Uses Python 3.12 slim, installs requirements, copies app files, and runs `app.py`. Perfect.
* **Flask app (`app.py`)**: A simple expense tracker using SQLite, serving on port 5000.
* **requirements.txt**: Contains `flask` (make sure it's the only dependency or add more if needed).
* **Port**: Exposes 5000 (Flask default).

---

### 🛠️ What You Should Check / Add Before CI

1. **SQLite database**:

   * Since your app uses a local file `expenses.db`, remember that containers are ephemeral.
   * **For production**, consider persisting the DB with a volume or moving to a cloud DB.
   * For now, it will create the DB fresh on each container start — fine for learning/demo.


### 🚀 Next: GitHub Actions CI Pipeline for Your App

You already have everything to build and push your Docker image!

Here’s a **minimal GitHub Actions workflow** that will:

* Build your Docker image from your repo’s Dockerfile
* Push it to Docker Hub as `minnuantony/exp-track:latest`

```yaml
name: CI - Build and Push Docker Image

on:
  push:
    branches:
      - main

jobs:
  build-and-push:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        push: true
        tags: minnuantony/exp-track:latest
```

---

### 🔐 What You Need to Do:

1. Add your Docker Hub credentials as **GitHub Secrets** in your repo:

| Secret Name      | Description                              |
| ---------------- | ---------------------------------------- |
| DOCKER\_USERNAME | Your Docker Hub username                 |
| DOCKER\_PASSWORD | Your Docker Hub password or access token |

2. Commit and push this workflow file to `.github/workflows/ci.yml`.

3. Push to your main branch — GitHub Actions will build & push the Docker image automatically.

---

### After CI, we can work on the **CD** part:

* Deploy your new image to your Kubernetes cluster automatically
* Use `kubectl` commands in GitHub Actions to update the deployment with new image tags

---

Want me to help you draft the **deployment YAML** for Kubernetes and the **CD workflow** next?

Got it! I’ll give you:

* Fully customized **Kubernetes manifests** with deployment named `et-deploy` and service named `et-service`
* A **complete GitHub Actions workflow** that builds your Docker image, pushes it, then deploys (updates) your app on the cluster using these manifest names

---

### 1️⃣ Kubernetes Manifests

Save these as `deployment.yaml` and `service.yaml` in your repo root or a `k8s/` folder.

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: et-deploy
spec:
  replicas: 1
  selector:
    matchLabels:
      app: et-app
  template:
    metadata:
      labels:
        app: et-app
    spec:
      containers:
      - name: et-container
        image: minnuantony/exp-track:latest
        ports:
        - containerPort: 5000
```

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: et-service
spec:
  type: NodePort
  selector:
    app: et-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
      nodePort: 30007
```

---

### 2️⃣ Full GitHub Actions Workflow `.github/workflows/ci-cd.yaml`

```yaml
name: CI/CD - Build, Push & Deploy to Kubernetes

on:
  push:
    branches:
      - main

jobs:
  build-push-deploy:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2

    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        push: true
        tags: minnuantony/exp-track:latest

    - name: Setup kubectl
      uses: azure/setup-kubectl@v3

    - name: Configure kubeconfig
      run: |
        echo "${{ secrets.KUBE_CONFIG_DATA }}" | base64 --decode > $HOME/.kube/config

    - name: Apply Kubernetes manifests
      run: |
        kubectl apply -f deployment.yaml
        kubectl apply -f service.yaml

    - name: Update deployment image
      run: |
        kubectl set image deployment/et-deploy et-container=minnuantony/exp-track:latest
        kubectl rollout status deployment/et-deploy
```

---

### 3️⃣ Will the naming changes affect previous files?

* **No changes needed** in your Dockerfile or app.
* The only things affected are the **deployment and service resource names** in Kubernetes.
* The names `et-deploy` and `et-service` are **independent** from your previous setup.
* Just ensure when you do manual `kubectl` commands or check pods/services, you use these names.

---

### 4️⃣ Final Notes

* Add your Kubernetes cluster kubeconfig as a GitHub secret called `KUBE_CONFIG_DATA` (base64 encoded).
* Make sure your EC2 nodes allow inbound traffic on port `30007` (NodePort).
* You can change the `nodePort` in the service YAML if `30007` conflicts with anything.

---

If you want, I can also help with commands to base64-encode your kubeconfig or any other step — just ask!

Awesome! Here’s how you can **base64-encode your kubeconfig file** and add it as a GitHub secret:

---

### How to base64-encode your kubeconfig file

1. On your local machine or EC2 where your kubeconfig lives (usually `~/.kube/config`):

```bash
base64 ~/.kube/config > kubeconfig.b64
```

2. Open the generated `kubeconfig.b64` file with any text editor.

3. Copy the entire content (it’s a long single-line string).

Great! Here’s what you can do next:

1. **Base64 encode your kubeconfig and add it as a GitHub secret `KUBE_CONFIG_DATA`** (as I explained).

2. **Commit and push your Kubernetes manifests (`deployment.yaml` and `service.yaml`) and the GitHub Actions workflow `.github/workflows/ci-cd.yaml`** to your repo’s main branch.

3. **Trigger the workflow by pushing a commit or just pushing the workflow file itself.**

4. **Monitor the Actions tab** in GitHub to watch your CI/CD run — building your image, pushing to Docker Hub, and deploying/updating your Kubernetes app.

5. **Access your app in browser:**
   `http://<your-ec2-public-ip>:30007`

---

If you want, I can also provide commands or tips to:

* Troubleshoot if pods don’t come up or rollout fails
* Open security group ports for NodePort access
* Check logs (`kubectl logs`) and pod status

Just say the word! How would you like to proceed?

---

### Add it as a GitHub secret

1. Go to your GitHub repo → **Settings** → **Secrets and variables** → **Actions**

2. Click **New repository secret**

3. Name it: `KUBE_CONFIG_DATA`

4. Paste the base64 string from step 3 into the value field.

5. Save the secret.

---

### Now your GitHub Actions workflow can decode and use it with:

```yaml
echo "${{ secrets.KUBE_CONFIG_DATA }}" | base64 --decode > $HOME/.kube/config
```

---

If you want, I can help you troubleshoot or write a little helper script for this. Otherwise, you’re all set to commit the workflow and deploy automatically!

---

Ready for the next step or want me to help you verify your setup?


