ref: https://github.com/sidpalas/devops-directive-docker-course
# Docker Installation on Ubuntu 22.04

This guide outlines the commands to install Docker Engine, Docker CLI, Buildx, and Docker Compose Plugin on an Ubuntu 22.04 system.

## Step-by-Step Commands

```bash
# 1. Update package index
sudo apt-get update

# 2. Install required packages
sudo apt-get install ca-certificates curl gnupg lsb-release -y

# 3. Create Docker keyrings directory
sudo mkdir -m 0755 -p /etc/apt/keyrings

# 4. Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 5. Set up the Docker repository
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 6. Update package index again
sudo apt-get update

# 7. Install Docker Engine, CLI, containerd, Buildx, and Compose Plugin
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 8. Start and enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# 9. Verify Docker installation
docker version
```
## DOCKER VOLUMES AND MOUNTS
**Persisting Data Produced by the Application:** (we can also pre install dependencies)
- Often, our applications produce data that we need to safely persist (e.g. database data, user uploaded data, etc...) even if the containers are destroyed and recreated. 
- Luckily, Docker (and containers more generally) have a feature to handle this use case called Volumes and mounts.
- Volumes and mounts allow us to specify a location where data should persist beyond the lifecycle of a single container.

### Volume
* A **volume** is a storage mechanism managed entirely by Docker.
* It exists outside the container's filesystem and is stored in Docker's internal directory.
* Use **volumes** when you need to **persist application data**, like **database files or user uploads**, across container restarts or when containers are deleted and recreated.

### Bind Mount (Mount)
- A **bind mount** (or just "mount") is a directory from the **host machine** that is mounted into the container.
- It links a path on the host to a path inside the container.
- Use **mounts** when you need to **share files between your host and container**, such as **source code during development**, or **configuration files** that need to be updated dynamically.

  ----------------------------------------------------------------------------
### 1. using volumes

  $ `docker run -it --rm ubuntu:22.04` # _create a container from ubuntu image_
  
  $ `mkdir my-data`
  
  $ `echo "Hello from the container!" > /my-data/hello.txt`  _#creating a folder and file_
  
  $ `exit`
  
- Now , we exit from the container and then create a new one, the file will not be present.
- 
- To persist the data, we use volumes

  $ `docker volume create my-volume` _# create a named volume_
  
  $ `docker run  -it --rm --mount source=my-volume,destination=/my-data/ ubuntu:22.04` or `docker run  -it --rm -v my-volume:/my-data ubuntu:22.04`  _#Create a container and mount the volume into the container filesystem_

 _# Now we can create and store the file into the location we mounted the volume_
 
 $ `echo "Hello from the container!" > /my-data/hello.txt`
 
 $ `cat my-data/hello.txt`
 
 $ `exit`

 - Create a new container and mount the volume into the container filesystem
   
 $ `docker run  -it --rm --mount source=my-volume,destination=/my-data/ ubuntu:22.04`

 root@4a93beffc351:/# `cat /my-data/hello.txt `
 
Hello from the container!

root@4a93beffc351:/# `exit`

----------------------------------------------------------------------------------------------------------
## creating a simple Python app, containerizing it with Docker, saving the image as a .tar file, and copying it from EC2 to  local machine

$ `mkdir expense-tracker-app`

$ `cd expense-tracker-app`

$ `vim app.py`

$ `vim requirements.txt`

$ `vim Dockerfile`

$ `docker build -t expense-tracker-app .`

$ `docker run -p 5000:5000 expense-tracker-app`

$ `docker save -o expense-tracker-app.tar expense-tracker-app`

$ `scp -i /home/minnu/Downloads/Minnu.pem ubuntu@3.22.61.207:/home/ubuntu/expense-tracker-app/expense-tracker-app.tar /home/minnu/UST/Training/`

- To remove the container: 
$ `docker rm <conatinerID>`






