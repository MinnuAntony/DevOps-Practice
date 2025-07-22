1. `kubeadm init`  
   ➔ Initializes the Kubernetes control-plane (master node).

2. `kubectl get node`  
   ➔ Checks the status of the node; initially shows as "NotReady".

3. `kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml`  
   ➔ Deploys the Weave CNI (Container Network Interface) for pod networking.

4. `watch -n 1 kubectl get po -n kube-system`  
   ➔ Continuously monitors the status of system pods until all are Running.

5. `kubectl describe node | grep -i taint`  
   ➔ Checks if there is a taint on the control-plane node that prevents scheduling pods.

6. `kubectl get node`  
   ➔ Gets the node name needed for the next step.

7. `kubectl taint node <node name> node-role.kubernetes.io/control-plane:NoSchedule-`  
   ➔ Removes the default taint from the control-plane node to allow scheduling workloads on it.

8. `kubectl run nginx --image=nginx`  
   ➔ Runs a simple nginx pod to test the cluster.

9. `kubectl get po`  
   ➔ Lists all pods running in the default namespace.

10. `kubectl get po -o wide`  
    ➔ Lists pods with additional details, including their assigned IP addresses.

11. `curl <ip>`  
    ➔ Verifies nginx is running by sending a request to the pod's IP; expects nginx default response.
