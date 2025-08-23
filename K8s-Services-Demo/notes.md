# Kubernetes Service

A **Kubernetes Service** is an abstraction that provides a stable IP  to a set of Pods, enabling reliable communication and automatic load balancing across them.


## Why do we need Services?

* A **Pod** is the smallest deployable unit in Kubernetes.
* Pods Are Ephemeral meaning they are **not permanent** – they are created, destroyed, and recreated all the time by controllers like **ReplicaSets**, **Deployments**, or due to failures.

* A Pod gets a new IP address whenever it is replaced. Kubernetes **does not guarantee stable Pod IPs** across Pod restarts.

* If clients (other Pods, users, or external systems) connected **directly** to Pod IPs:
  * Connections would **break** whenever Pods are recreated.
  * You’d need to constantly update your configs with new IPs → impractical.

---

##  **The Solution → Services**

* A **Service** provides a **stable or static virtual IP (ClusterIP)**.
* Clients connect to the **Service**, not directly to Pods.
* The Service automatically routes traffic to whichever Pods are healthy and available (via selectors + kube-proxy).

---

# Types of Kubernetes Services

Kubernetes provides several types of services to meet different use cases:

- **ClusterIP**:  
  This is the default type of service in Kubernetes. It provides a stable IP address within the cluster that other Pods can use to access the service. This type of service is useful for **internal communication within the cluster**.

- **NodePort**:  
  This type of service exposes the service on a static port on each node's IP address. It is useful when you need to **access the service from outside the cluster**.

- **LoadBalancer**:  
  This type of service exposes the service on an **external load balancer**. This is useful when you need to **distribute traffic across multiple nodes**.

## 1. **ClusterIP**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

* **Type**: `ClusterIP` (default)
* **What it does**: Assigns a **virtual internal IP** to the Service.
* **Accessibility**: Only reachable **inside the cluster**.
* **Use case**: Internal communication between microservices.
* **Key point**: Pods behind the Service can scale up/down, but the Service IP remains stable.

---

## 2. **NodePort**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

* **Type**: `NodePort`
* **What it does**: Exposes the Service on **all cluster nodes** at a static port (here `30080`).
* **Accessibility**: Can be accessed **from outside the cluster** via `<NodeIP>:30080`.
* **Use case**: Quick testing, demos, or exposing apps without a cloud load balancer.
* **Key point**: Works on **any node**, but the port range is limited (30000–32767).

---

## 3. **LoadBalancer**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

* **Type**: `LoadBalancer`
* **What it does**: Requests a **cloud provider load balancer** 
* **Accessibility**: External clients can access the Service using the **cloud LB’s IP or DNS**.
* **Use case**: Production apps needing **stable external endpoints**, auto-scaling, and built-in load balancing.
* **Key point**: Combines NodePort + ClusterIP internally; the cloud provider handles the external IP.

---

### 1. Pods (from an `nginx` Deployment)

```bash
$ kubectl get po
NAME                          READY   STATUS    RESTARTS   AGE
nginx-deployment-6d8f7b9c8d-abc12   1/1     Running   0          3m
nginx-deployment-6d8f7b9c8d-def34   1/1     Running   0          3m
nginx-deployment-6d8f7b9c8d-ghi56   1/1     Running   0          3m
```

--

### 2. Services

```bash
$ kubectl get svc
NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
nginx-service        ClusterIP      10.96.120.15    <none>          80/TCP         3m
nginx-nodeport       NodePort       10.96.248.42    <none>          80:30080/TCP   3m
nginx-loadbalancer   LoadBalancer   10.96.157.89    34.123.45.67    80:31922/TCP   3m
kubernetes           ClusterIP      10.96.0.1       <none>          443/TCP        10d
```

* **ClusterIP (`nginx-service`)** → only has a **stable internal IP** (`10.96.120.15`)
* **NodePort (`nginx-nodeport`)** → maps `ClusterIP:80` to **node IPs on port 30080**
* **LoadBalancer (`nginx-loadbalancer`)** → cloud provider assigns an **EXTERNAL-IP** (`34.123.45.67`) that maps to internal `ClusterIP` + auto-chosen NodePort (`31922`)
---

## When and Why to Use Each

| Service Type     | Accessibility         | Use Case / Why                                                                        |
| ---------------- | --------------------- | ------------------------------------------------------------------------------------- |
| **ClusterIP**    | Internal only         | For microservices communicating inside the cluster.                                   |
| **NodePort**     | External via NodeIP   | Quick dev/test access or when no cloud LB is available.                               |
| **LoadBalancer** | External via cloud LB | Production apps needing reliable external endpoints and cloud-managed load balancing. |

---

![Kubernetes LoadBalancer Service](https://bsdnet.github.io/images/kubernetes-service-loadbalancer.png)
