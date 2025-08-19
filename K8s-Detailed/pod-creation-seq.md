
## 🔹 Pod Creation Event Flow

1. **User submits Pod spec**

   * You run `kubectl apply -f pod.yaml`.
   * `kubectl` sends the Pod spec (YAML/JSON) → **API server** (HTTP request).

---

2. **API server validates & persists**

   * API server authenticates request → checks RBAC.
   * Validates Pod spec.
   * Stores Pod object in **etcd** with state: `Pending`.
   * Returns response to `kubectl` → “Pod created”.

---

3. **Scheduler detects unscheduled Pod**

   * **kube-scheduler** is watching the API server.
   * It sees the new Pod without a `nodeName`.
   * Runs scheduling logic (filters & scores nodes).
   * Chooses a node (say `node-1`).
   * Updates the Pod object in API server with `nodeName=node-1`.

---

4. **Kubelet on chosen node reacts**

   * **kubelet** on `node-1` is watching API server for Pods assigned to it.
   * It sees the new Pod scheduled to its node.
   * It pulls the container image(s) using the **container runtime** (containerd/CRI-O).
   * Sets up networking via **CNI plugin**.
   * Mounts storage volumes if defined.
   * Starts containers.

---

5. **Kubelet updates Pod status**

   * Initially sets status → `ContainerCreating`.
   * Once container is running → status → `Running`.
   * Sends status updates back to API server.
   * API server stores these updates in etcd.

---

6. **Controllers watch and reconcile**

   * If the Pod was part of a **Deployment / ReplicaSet**, the respective controller checks:

     * Desired replicas vs actual replicas.
     * Ensures more Pods are created if needed.
   * Service controllers update Endpoints (so Services know which Pods are available).

---

7. **Cluster state converges**

   * Pod is now `Running`.
   * Other components (like Service, Ingress, DNS) update themselves to route traffic.
   * You can run `kubectl get pods` → API server fetches current Pod status from etcd and shows it.

---

## 🔹 Summary Timeline

1. User → `kubectl apply` → API server
2. API server → validates & stores Pod in etcd (`Pending`)
3. Scheduler → assigns node → updates Pod spec
4. Kubelet on that node → creates container(s)
5. Kubelet → reports status → API server updates etcd
6. Controllers update higher-level objects (ReplicaSet, Service, etc.)
7. Pod → Running & ready to serve traffic

---

⚡️So, **Pod creation is not one component doing all the work** — it’s a chain of *event-driven reactions*:
API server ↔ etcd ↔ scheduler ↔ kubelet ↔ controllers.

---

