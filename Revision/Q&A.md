# 🔑 Practical Q\&A – Docker & Kubernetes

---

### **Containers & Docker Basics**

**Q1. Why do we need containers when we already have Virtual Machines?**
👉 VMs virtualize *hardware* (each VM has its own OS kernel), containers virtualize *processes* (all share the host kernel but are isolated).

* Containers are lightweight, start in milliseconds, use less memory.
* VMs provide stronger isolation but are heavier.
  **In practice:** Containers are used for microservices and CI/CD pipelines; VMs still used for strong security and legacy apps.

---

**Q2. Why not just run applications directly on the host instead of using containers?**
👉 Without containers:

* Dependency conflicts (“works on my machine” problem).
* Hard to ensure consistent environments across dev → prod.
* No resource isolation; one app can hog CPU/memory.
  **Containers solve** this with namespaces + cgroups → consistent, isolated environments.

---

**Q3. Why is Docker so popular if containers can run without it?**
👉 Docker gave developers a **complete ecosystem**:

* Easy CLI (`docker build`, `docker run`).
* Standardized image format + Docker Hub registry.
* Abstracted low-level runtimes (containerd, runc).
  That “developer experience” made containers mainstream.

---

### **Namespaces & cgroups**

**Q4. Why do containers “feel” like separate machines if they are just processes?**
👉 Namespaces give each process its own view (own PID tree, own filesystem, own IP).
cgroups ensure fair resource usage. Together they **simulate mini-machines**, but under one Linux kernel.

---

**Q5. What happens if you don’t use cgroups in containers?**
👉 A single container could hog CPU/memory and bring down the whole node.
Ex: A runaway process with memory leak could starve other apps.
That’s why cgroups are critical for production.

---

### **Docker Runtime Flow**

**Q6. Why does Docker need containerd and runc? Why not talk directly to the kernel?**
👉 Separation of concerns:

* `dockerd` → API, build, registry.
* `containerd` → container lifecycle, image unpacking.
* `runc` → actually calls Linux syscalls to create namespaces, cgroups.
  This modularity allows Kubernetes to use containerd/CRI-O **without needing Docker**.

---

**Q7. Why does Kubernetes drop Docker support (v1.24+)?**
👉 Kubernetes only needs a runtime that implements CRI (Container Runtime Interface).
Docker itself doesn’t implement CRI → kubelet can’t talk to it directly.
Instead:

* Kubernetes talks to containerd/CRI-O.
* Docker internally used containerd anyway → Docker became an extra unnecessary layer.

---

### **Docker Images & Layers**

**Q8. Why are Docker images built in layers instead of one big blob?**
👉 Layers bring efficiency:

* Reuse unchanged layers across builds (fast rebuild).
* Multiple containers share base layers (save space).
* Pushing/pulling only transfers missing layers.
  Without layers → every small change would rebuild/upload entire image.

---

**Q9. Why not make image layers writable?**
👉 Images are immutable for consistency.

* If base image changes → reproducibility breaks.
* Writable images would lead to “snowflake” containers.
  That’s why containers add a **thin writable layer on top** instead (Copy-on-Write).

---

**Q10. Why is Copy-on-Write important?**
👉 CoW avoids duplication:

* All containers share read-only image layers.
* Only modified files are copied into container’s writable layer.
  This saves disk space and speeds up container startup.

---

**Q11. Why does Docker use OverlayFS (lowerdir/upperdir/workdir/merged)?**
👉 OverlayFS provides union filesystem:

* `lowerdir` = read-only image layers.
* `upperdir` = writable container layer.
* `merged` = final view inside container.
  It’s efficient because only changes go into `upperdir`.
  Without OverlayFS → Docker would need to copy full images per container.

---

### **Kubernetes Relationship**

**Q12. Why can’t Kubernetes run containers directly?**
👉 Kubernetes is an orchestrator, not a runtime.

* K8s decides *what/where* to run.
* The container runtime (containerd/CRI-O) actually runs the process.
  This loose coupling makes Kubernetes runtime-agnostic.

---

**Q13. Why did Kubernetes replace Docker with containerd/CRI-O?**
👉 Docker was never a CRI-compatible runtime. Kubernetes used a “shim” called `dockershim` → extra complexity.
Now Kubernetes talks directly to containerd/CRI-O → simpler, faster, cleaner.

---

### **Self-Healing**

**Q14. Why is Kubernetes called “self-healing”?**
👉 Because K8s continuously reconciles desired state vs actual state:

* Pod crash → kubelet restarts.
* Node dies → Pods rescheduled elsewhere.
* Liveness probe fails → container restarted.
* Deployment requires 3 replicas → ensures always 3 running.
  This automation reduces manual ops overhead.

---

**Q15. Why not just monitor containers manually with scripts instead of K8s self-healing?**
👉 Manual monitoring = reactive, error-prone, delayed.
Kubernetes is **proactive + automatic**, continuously enforcing desired state in real-time.
That’s why K8s is essential at scale.

---

### **Kubernetes Architecture**

**Q16. Why does Kubernetes need etcd?**
👉 etcd is the **single source of truth**.

* Stores desired state (from YAMLs).
* Stores actual state (reported by nodes).
  Controllers read from etcd → reconcile → update cluster.
  Without etcd → cluster state consistency is impossible.

---

**Q17. Why can’t the kubelet act alone without the control plane?**
👉 Kubelet only executes instructions on a node.
It doesn’t know the cluster-wide desired state.
Control plane (API server, controllers, scheduler) ensures global orchestration.

---

**Q18. Why is kube-proxy needed if Pods already have IPs?**
👉 Because Pods are ephemeral — IPs change on restarts/rescheduling.
kube-proxy ensures stable **Service IPs** and load-balancing across Pods.
Without it → clients would need to track dynamic Pod IPs → chaos.

---
