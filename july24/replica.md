# ReplicaSet in Kubernetes

A **ReplicaSet** is like a guardian that watches over your pods.  
Its job is to make sure you **always have the exact number of pods you want** running.

## What it Does

- Monitors your pods constantly  
- If a pod dies, it creates a new one  
- If there are too many pods, it removes extras  
- If there are too few pods, it creates more  

---

## Creating a ReplicaSet

Step 1: Save ReplicaSet YAML  :  `vim replicaset.yaml`

Step 2: Deploy ReplicaSet     :  `kubectl apply -f replicaset.yaml`

Step 3: Check ReplicaSet & Pods   : `kubectl get rs`  or `kubectl get pods`
