# POD
- A pod is the lowest level of deployment in K8s.
- A definition of how to run a container
- A pod is a GROUP OF ONE OR MORE CONTAINERS.
- To create a pod -> `.yaml` file required
- Containers inside the same pod - share network, share storage -talk to each other using localhost
- K8s allocates clusterIP address to the pod
 ---------
 ref: https://kubernetes.io/pt-br/docs/reference/kubectl/cheatsheet/
 
- To create the Pod : `kubectl create -f <yaml file>`
- When you want to create or update the resource if it already exists :`kubectl apply -f <yaml>`
- To see status of the Pod : `kubectl get pods` or `kubectl get pods -o wide`
- To delete the pod : `kubectl delete <podname>`
  
  -----------------
  
# DEPOLYMENT
- Autohealing and Autoscaling(done by ReplicaSet) properties
- Do not create a pod directly - create it using depolyment - which creates a replicaset which in turn creates pods.

## Creating Deployments:
- Write the .yaml file : `vim test-deployment.yaml`
  - replicas : specifies the no.of pods
  
- Create the deployment: `kubectl apply -f test-deployment.yaml` 
                         `deployment.apps/nginx-deployment created`
- Verify: `kubectl get deploy`
- 
| NAME | READY | UP-TO-DATE | AVAILABLE | AGE |
|---|---|---|---|---|
| nginx-deployment | 3/3 | 3 | 3 | 2m11s |

---

- `kubectl get rs`
- 
| NAME | DESIRED | CURRENT | READY | AGE |
|---|---|---|---|---|
| nginx-deployment-85996f8dbd | 3 | 3 | 3 | 2m22s |

---

- `kubectl get po`
- 
| NAME | READY | STATUS | RESTARTS | AGE |
|---|---|---|---|---|
| expense-tracker-pod | 1/1 | Running | 1 (4m58s ago) | 3h56m |
| nginx-deployment-85996f8dbd-hs5bm | 1/1 | Running | 0 | 2m30s |
| nginx-deployment-85996f8dbd-qmq6s | 1/1 | Running | 0 | 2m30s |
| nginx-deployment-85996f8dbd-wvd4w | 1/1 | Running | 0 | 2m30s |
------

- If a pod in the deployment is deleted - it will be created automatically created
- That is the autohealing nature of Deployment (ensures that the predefined number od pods are always running)
  
   
  

--------------------------------

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
