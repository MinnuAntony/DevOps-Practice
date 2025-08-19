https://chatgpt.com/share/68a4afc0-e288-800e-b54b-504ecf797d91
# Kubernetes Pods: Internal Creation Flow (Deep Dive)

> A practical, end‑to‑end walkthrough of what *actually* happens inside a Kubernetes cluster when you create a Pod (or a Deployment that creates Pods)—from `kubectl apply` to packets hitting your container.

---

## 0) Kubernetes Architecture (quick but complete)

**Control plane (brains)**

* **kube‑apiserver**: Front door; validates requests, runs authn/authz/admission, persists state to etcd, exposes a watch/stream interface to everyone else.
* **etcd**: Strongly consistent, MVCC key‑value store; holds the *desired* and *observed* state of all API objects.
* **kube‑controller‑manager**: A bundle of controllers (Deployment, ReplicaSet, Job, StatefulSet, Node, EndpointSlice, ResourceQuota, etc.). Each runs a reconcile loop: *current vs desired → take actions*.
* **kube‑scheduler**: Assigns Pods without a node to a specific Node based on filters (feasibility) and scores (best fit). Handles preemption when necessary.
* **cloud‑controller‑manager (optional/managed)**: Talks to the cloud provider (e.g., ELB creation, Node lifecycle). In managed services (EKS/GKE/AKS) it’s often provided/hidden.

**Node plane (muscle)**

* **kubelet**: Per‑node agent. Watches the API for Pods bound to its node and drives the container runtime to realize them. Reports status, runs probes, mounts volumes.
* **Container runtime (via CRI)**: Typically containerd or CRI‑O. Creates the Pod sandbox (infra container), pulls images, runs containers, manages cgroups/namespaces.
* **CNI plugin**: Programs pod networking (veth pairs, IPAM, routes/NAT), and enforces NetworkPolicy (iptables/eBPF via Calico, Cilium, etc.).
* **kube‑proxy**: Implements Service VIPs and load‑balancing on each node (iptables or IPVS). Some CNIs replace/augment this.
* **CSI drivers (node + controller pieces)**: Attach/mount/block/ephemeral volumes.

---

## 1) The Request: from your shell to the cluster

1. **You apply YAML** (e.g., a Deployment or a naked Pod):

   * `kubectl` sends a REST call to **kube‑apiserver**.
2. **Authentication** (x509 client cert, bearer token, OIDC/JWT, etc.).
3. **Authorization** (RBAC/ABAC/webhook): are you allowed to create this object?
4. **Admission chain** (ordered):

   * **Mutating** (may change your object): defaults, LimitRanger, sidecar injectors (Istio/Linkerd), PSP replacement (Pod Security Admission defaults), image pull secrets injection, tolerations defaults, etc.
   * **Validating** (must accept): ResourceQuota, PodSecurity Admission (baseline/restricted), custom policy engines (Kyverno/Gatekeeper), topology constraints,
     and any org‑specific validations.
5. **Persistence**: The (possibly mutated) object is written to **etcd** with a new `resourceVersion`.

> If you created a **Deployment**, the Deployment controller immediately ensures a **ReplicaSet**, which then creates **Pod** objects (with `ownerReferences`). From here, the rest of this flow focuses on Pods.

---

## 2) Controller loops kick in (if not a naked Pod)

* **Deployment controller** computes desired ReplicaSets and replica counts.
* **ReplicaSet controller** compares desired vs actual Pods → creates/deletes Pods to match.
* Each created Pod initially has **no `spec.nodeName`** (i.e., unscheduled).

---

## 3) Scheduling: choosing a node

1. **Scheduler watches** the API for unscheduled Pods.
2. **Filter (feasibility)** across candidate nodes:

   * Node readiness/taints & Pod **tolerations**
   * **Resources** (CPU/memory/extended resources, PodOverhead)
   * **Affinity/anti‑affinity** & **topology spread** constraints
   * **Volume feasibility** (PVC binding, CSI capabilities, zone/az constraints)
   * **Node selectors/labels** & RuntimeClass/node features (GPU, SGX, kata, etc.)
3. **Score (ranking)** feasible nodes: balance/pack, affinity weights, image locality, etc.
4. **PreBind** (e.g., finalize volume bindings).
5. **Bind**: Scheduler writes a **Binding** (or sets `spec.nodeName`) → Pod is now assigned to a node.
6. **(Optional) Preemption**: if no feasible nodes, may evict lower‑priority Pods to make room.

An **Event** sequence appears on the Pod (e.g., *Scheduled*), visible via `kubectl describe pod`.

---

## 4) Kubelet turns desired state into containers

On the chosen node, **kubelet** is watching the API and sees the new bound Pod:

1. **Pod sandbox creation** (via CRI):

   * Runtime creates an *infra/pause* container which holds the Pod’s Linux namespaces (net, ipc, uts, pid if ShareProcessNamespace, etc.).
   * **Security** applied: cgroups (QoS classes: Guaranteed/Burstable/BestEffort), seccomp/AppArmor, SELinux, user/group mappings, capabilities, Linux `oom_score_adj`.
2. **Networking (CNI)**:

   * Runtime calls the CNI plugin’s `ADD` for the sandbox → creates a veth pair, connects to the node bridge or overlay, assigns Pod IP (via IPAM), sets routes.
   * NetworkPolicy rules programmed (iptables or eBPF). Pod now has a stable **cluster‑routable IP**.
3. **Volumes (CSI & local)**:

   * **PVCs**: If using PersistentVolumeClaims, the CSI **controller** may have attached the volume; the **node plugin** stages & publishes (mounts) it.
   * **ConfigMap/Secret/DownwardAPI/Projected** volumes are materialized on the node (tmpfs) and mounted into the sandbox.
   * `subPath`, `readOnly`, fsGroup, access modes are applied.
4. **Images**:

   * For each container image, kubelet asks the CRI **ImageService** to pull if needed (policy `Always/IfNotPresent/Never`). Uses `imagePullSecrets`/registry creds.
5. **Init containers** run **sequentially** until all succeed.
6. **App containers** start **in parallel** (unless `startupProbe` gates).

   * Env assembled (literal, from Secret/ConfigMap, Downward API), command/args expanded, working dir set.
   * Lifecycle hooks: `postStart` executed after process start.

Pod phase transitions: **Pending → ContainerCreating → Running** (assuming probes pass).

---

## 5) Probes, readiness, and Service wiring

* **startupProbe** (optional): gating long bootstrap times; until it succeeds, liveness/readiness are disabled.
* **livenessProbe**: failing → kubelet restarts the container.
* **readinessProbe**: failing → container remains running, but **Endpoints/EndpointSlices** will **exclude** this Pod from Service load‑balancing.
* **kube‑proxy** (or CNI dataplane) programs **Service VIPs** (ClusterIP/NodePort/IPVS) and selects **ready** endpoints.
* **Ingress/LoadBalancer** (if used): cloud‑controller provisions external LB; traffic flows to nodes → Service → Pod.

Result: once readiness is **True**, cluster traffic can reach your Pod via Services.

---

## 6) Observability of the lifecycle

Typical **Events** you’ll see (view with `kubectl describe pod <name>`):

* *Created* (by ReplicaSet) → *Scheduled* (by scheduler) → *Pulling* image → *Pulled* → *Created* container → *Started* container → *(Probes)*.

Useful commands:

* `kubectl get deploy,rs,pods -o wide` (owner chain, node placement)
* `kubectl describe pod <pod>` (events, conditions, volumes)
* `kubectl logs <pod> [-c <container>]` (app logs)
* `kubectl get events --sort-by=.metadata.creationTimestamp`
* `kubectl get endpointslices,endpoints svc/<service>` (readiness wiring)

---

## 7) Pod termination (for completeness)

1. You delete the Pod (or scale down a Deployment). API server sets `deletionTimestamp`.
2. **Endpoints** remove the Pod (stops new traffic).
3. **kubelet** sends **SIGTERM** to each container and waits **`terminationGracePeriodSeconds`**.

   * `preStop` hooks run here.
   * After the grace period, kubelet sends **SIGKILL**.
4. Volumes unmounted; CNI `DEL` tears down networking; sandbox removed.
5. Final status pushed to the API (phase `Succeeded`/`Failed`) and object is garbage‑collected (respecting finalizers if any).

---

## 8) Key mechanics & nuances (often asked in interviews)

* **Desired vs Observed state**: Controllers and kubelet continuously reconcile; the cluster self‑heals to the declared spec.
* **QoS & cgroups**: Resource `requests` drive scheduling; `limits` + requests determine QoS classes and cgroup settings; OOM killer honors QoS.
* **Eviction manager**: On resource pressure (memory/disk/inodes), kubelet evicts lower‑priority/best‑effort pods to keep the node healthy.
* **Taints/Tolerations** and **(anti)affinity**: Coarse vs fine‑grained placement controls; topology spread keeps replicas across zones/racks.
* **Admission evolution**: PodSecurityPolicy (deprecated) replaced by **Pod Security Admission** (baseline/restricted/profiles) + ecosystem policies (Kyverno/Gatekeeper).
* **RuntimeClass**: Choose alternative runtimes (gVisor/kata) for stronger isolation; may affect scheduling & overhead.
* **OwnerReferences & GC**: Pods created by RS/Job carry owners; when owners go away, dependents are garbage‑collected.
* **EndpointSlice over Endpoints**: Scales better; kube‑proxy consumes slices for programming dataplane.

---

## 9) Minimal end‑to‑end timeline (annotated)

```text
kubectl → apiserver → authn → authz → admission (mutate/validate) → etcd (persist)
               ↓
     controllers reconcile (Deployment → ReplicaSet → Pod)
               ↓
         scheduler filters/scores/binds (sets nodeName)
               ↓
  kubelet sees bound Pod → create sandbox → CNI add (IP) → CSI mount → pull images
               ↓
          run init containers → run app containers → probes OK
               ↓
       kube-proxy/Service picks endpoints → traffic flows
```

---

## 10) Managed clusters (EKS/GKE/AKS) specifics

* **Control plane** often hidden and auto‑scaled; version and API features still standard.
* **LoadBalancers/Ingress**: created by controllers talking to your cloud; for EKS, Service type=LB plus **AWS Load Balancer Controller** handles ALB/NLB.
* **Storage**: CSI drivers (EBS, Filestore, Azure Disk) handle dynamic provisioning; `StorageClass` chooses reclaim/policy and parameters.
* **Identity**: EKS commonly uses **IRSA** (IAM Roles for Service Accounts) for Pod‑level AWS permissions projected via tokens.

---

## 11) Where failures usually occur (and what they mean)

* **Rejected at admission**: policy blocks (security context, hostPath, privilege). Fix spec or policy.
* **`Pending` forever**: scheduler can’t place (insufficient CPU/mem, taints, affinity conflicts, PVC not bound).
* **`ImagePullBackOff`**: bad image ref/registry auth; check `imagePullSecrets` and tag.
* **`CreateContainerError`**: bad command/volume mount/permission; see container logs & events.
* **Probes failing**: container restarts or excluded from traffic; verify paths, ports, timeouts.
* **`CrashLoopBackOff`**: app crashes; add `startupProbe`/tune `restartPolicy` or fix app.

---

## 12) Mental model TL;DR

* Kubernetes is a **declarative state machine**. You write *what*; controllers/kubelet figure out *how*.
* Pod creation is a **pipeline**: API accept → persist → reconcile → schedule → realize on node → become ready → receive traffic.

---

### Appendix A — Glossary

* **Pod**: Smallest deployable unit; one or more containers sharing network & (optionally) PID/IPC namespaces and volumes.
* **ReplicaSet/Deployment**: Higher‑level controllers that create/manage Pods.
* **Service**: Stable virtual IP & DNS for discovering reachable **ready** Pods.
* **EndpointSlice**: Records of which Pod IP\:port backs a Service.
* **CNI/CSI**: Plugin interfaces for networking/storage.
* **Probe**: Health checks (startup/liveness/readiness).
* **Taint/Toleration**: Node marks that repel Pods unless they tolerate.
* **Affinity/Anti‑affinity**: Placement constraints among Pods and Nodes.

---

## Want practice?

* Create a small Deployment with an initContainer + readiness probe + PVC. Watch events: `kubectl get events -A --watch`.
* Break things on purpose (wrong image tag, missing toleration) and observe the **exact** stage that fails—then map it back to the pipeline above.
