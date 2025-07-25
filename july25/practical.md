## BUILDING AN IMAGE - PUSHING IT INTO DOCKER HUB - RUNNING A POD  USING THE IMAGE

1. Install Docker CE (Community Edition) from Docker's official repository: `apt install docker-ce docker-ce-cli containerd.io`
2. Check Docker service status                                            : `systemctl status docker`
3. Verify Docker installation version                                     : `docker --version`

4. Securely copying the app from your local machine to a remote server using scp with an SSH key :
   
  `scp -i minnunv.pem -r /home/minnu/UST/Training/expense-tracker-app ubuntu@3.86.144.53:/home/ubuntu`
  
5. Build a Docker image from the current directory using the `Dockerfile` and tags it as `minnuantony/expense-tracker-app:latest`.

  `docker build -t minnuantony/expense-tracker-app:latest .`
  
6. To enter your Docker Hub username and password to authenticate your Docker client with Docker Hub :

  `docker login`
  
7. Upload the tagged Docker image to the Docker Hub repository under the user `minnuantony` :

  ` docker push minnuantony/expense-tracker-app:latest`
  
8. Downloads the specified Docker image from Docker Hub to your local machine :
   
  `docker pull minnuantony/expense-tracker-app:latest`

9. Create the pod : `kubectl apply -f expense-tracker-pod.yaml`

10. `kubectl describe pod expense-tracker-pod` : found taint

11. Remove taint : `kubectl taint nodes kmaster node-role.kubernetes.io/control-plane-`

12. `kubectl logs expense-tracker-pod`

13. `curl 10.32.0.4:5000`
