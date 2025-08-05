# BLOCKS 

In Terraform, your code is made up of **blocks** — each block describes something about your infrastructure.
Here are the main types:

---

### **1. `provider` block** – *“Who are we talking to?”*

Tells Terraform which cloud/service you’re using and any setup needed.

```hcl
provider "aws" {
  region = "us-east-1"
}
```

---

### **2. `resource` block** – *“What do we want to create?”*

Defines a real thing in the cloud (EC2, S3, VPC, etc.).

```hcl
resource "aws_s3_bucket" "my_bucket" {
  bucket = "my-bucket-name"
  acl    = "private"
}
```

---

### **3. `data` block** – *“Get info about something that already exists.”*

Reads existing cloud resources without creating new ones.

```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]
}
```

---

### **4. `variable` block** – *“Let’s make it flexible.”*

Lets you define input values instead of hardcoding them.

```hcl
variable "region" {
  default = "us-east-1"
}
```

---

### **5. `output` block** – *“Show me the result.”*

Prints important info after running Terraform.

```hcl
output "bucket_name" {
  value = aws_s3_bucket.my_bucket.bucket
}
```

---

### **6. `locals` block** – *“Reusable mini-variables.”*

For small calculated values inside your config.

```hcl
locals {
  bucket_prefix = "project-123"
}
```

---

📌 **Quick analogy**:

* **provider** → which shop you’re buying from.
* **resource** → the item you’re buying.
* **data** → checking if the shop already has it.
* **variable** → option to change item details later.
* **output** → your receipt.
* **locals** → sticky notes for yourself.

---
