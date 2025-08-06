## **Aim & Purpose**

We want to:
* **Run a Docker container** on Amazon ECS.
* Learn **how ECS works in real life** (Clusters → Task Definitions → Tasks → Services).
* Use **Fargate** (serverless) so we don’t have to manage EC2 instances.

We’ll:
* Create an ECS Cluster.
* Push a Docker image to Amazon ECR.
* Create a Task Definition.
* Create a Service to keep our container running.
* Access our running app.

---
## **Prerequisites**

✅ AWS CLI installed and configured (`aws configure`)
✅ Docker installed locally
✅ IAM permissions for ECS, ECR, IAM, CloudWatch, VPC
✅ A public subnet and security group in your VPC that allows HTTP (port 80)

---
Got it ✅ — I’ll make this **clear and structured** for you.

---

## **Aim & Purpose**

You are deploying a **simple static website** (served by **Nginx**) to **AWS ECS Fargate** so that:

* It runs **serverlessly** (no EC2 to manage).
* It can be **accessed from the internet**.
* It is **scalable** (ECS will keep the desired number of containers running).
* It is **integrated with AWS services** like ECR (image storage) and CloudWatch (logging).

---

## **Steps to Follow**

---

### **1️⃣ Prepare Docker Image**

* **Goal:** Package your web app (HTML) into a container image.
* **What you do:**

  * Create a `Dockerfile`:

    ```dockerfile
    FROM nginx:alpine
    COPY index.html /usr/share/nginx/html/
    EXPOSE 80
    CMD ["nginx", "-g", "daemon off;"]
    ```
  * Create `index.html` with your website content.

---

### **2️⃣ Push Image to Amazon ECR**

* **Goal:** Store your image in Amazon ECR so ECS can pull it.
* **What you do:**

  ```bash
  # Create ECR repository
  aws ecr create-repository --repository-name webapp

  # Authenticate Docker to ECR
  aws ecr get-login-password --region us-west-2 \
    | docker login --username AWS --password-stdin 123456789012.dkr.ecr.us-west-2.amazonaws.com

  # Build Docker image
  docker build -t webapp .

  # Tag image for ECR
  docker tag webapp:latest 123456789012.dkr.ecr.us-west-2.amazonaws.com/webapp:latest

  # Push image to ECR
  docker push 123456789012.dkr.ecr.us-west-2.amazonaws.com/webapp:latest
  ```

---

### **3️⃣ Create ECS Task Definition**

* **Goal:** Tell ECS **how to run your container**.
* **What you do:**

  * Save this JSON to a file `task-def.json`:

    ```json
    {
      "family": "webapp-task",
      "networkMode": "awsvpc",
      "requiresCompatibilities": ["FARGATE"],
      "cpu": "256",
      "memory": "512",
      "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
      "containerDefinitions": [
        {
          "name": "webapp",
          "image": "123456789012.dkr.ecr.us-west-2.amazonaws.com/webapp:latest",
          "portMappings": [
            { "containerPort": 80, "protocol": "tcp" }
          ],
          "essential": true,
          "logConfiguration": {
            "logDriver": "awslogs",
            "options": {
              "awslogs-create-group": "true",
              "awslogs-group": "/ecs/webapp",
              "awslogs-region": "us-west-2",
              "awslogs-stream-prefix": "ecs"
            }
          }
        }
      ]
    }
    ```
  * Register it:

    ```bash
    aws ecs register-task-definition --cli-input-json file://task-def.json
    ```

---

### **4️⃣ Create ECS Service**

* **Goal:** Run and manage your container in ECS so it stays running and can scale.
* **What you do:**

  ```bash
  aws ecs create-service \
    --cluster default \
    --service-name webapp-service \
    --task-definition webapp-task:1 \
    --desired-count 2 \
    --launch-type FARGATE \
    --network-configuration "awsvpcConfiguration={subnets=[subnet-12345678,subnet-87654321],securityGroups=[sg-abcdef01],assignPublicIp=ENABLED}"
  ```

  * Replace:

    * `subnet-12345678` & `subnet-87654321` → your **public subnet IDs**
    * `sg-abcdef01` → a **security group** allowing inbound **HTTP (port 80)**

---

### **5️⃣ Verify & Access Website**

* Find running tasks:

  ```bash
  aws ecs list-tasks --cluster default
  ```
* Get task details (including public IP):

  ```bash
  aws ecs describe-tasks --cluster default --tasks <task-id>
  ```
* Open in browser:

  ```
  http://<public-ip>
  ```

