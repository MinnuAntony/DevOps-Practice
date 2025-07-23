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

------------------------------------------------------------------
# DOCKER NETWORKING


## 1. Network Drivers

Docker uses network drivers to manage communication between containers and the outside world.

| Driver   | Description |
|----------|-------------|
| `bridge` | Default for standalone containers on a single host. Private internal network. |
| `host`   | Shares the host’s network stack. No isolation. |
| `none`   | Disables networking for the container. |
| `overlay` | Enables multi-host networking (Swarm mode). |

---

## 2. Bridge Network (Default for Docker)

Docker creates a bridge network named `bridge` by default.

- Containers on the same bridge can communicate using container names (via DNS).

### Example:
```bash
docker network create my-bridge
docker run -d --network my-bridge --name app1 nginx
docker run -d --network my-bridge --name app2 busybox
```

## 3. Container Communication

Same Network: Communicate by container name.

Different Networks: Cannot communicate unless connected to both networks explicitly.

## 4. Exposing Ports

Ports are exposed using the -p or --publish flag.

Format: host_port:container_port
Example:
`docker run -p 8080:80 nginx`

## 5. Inspecting Networks

View all networks:

`docker network ls`

Inspect a network:

`docker network inspect <network-name>`

## 6. Host Network

The container uses the host's network stack directly.

No network isolation.

Useful for high performance or direct access to host ports.

Example:

`docker run --network host nginx`

## 7. DNS in Docker
Docker provides a built-in DNS server.

Containers on the same user-defined network can resolve each other by name.

## 8. Custom Networks
Custom user-defined networks allow:

  - Automatic DNS-based name resolution.

  - Attaching or disconnecting containers dynamically.

Example :
$ `docker network create mycust-net`
`295f711258a8a3d8f00250cab6d3fae057ac6d8611f1ca8d8671e80bd91e7053`
$ `docker network ls`
| NETWORK ID    | NAME        | DRIVER | SCOPE |
|---------------|-------------|--------|--------|
| bc8c09a36455  | bridge      | bridge | local  |
| ac7d8c4854a0  | host        | host   | local  |
| 295f711258a8  | mycust-net  | bridge | local  |
| 74111fdb039a  | none        | null   | local  |

$ `docker run -d --name secure_cust --network mycust-net nginx:latest `
`f6f3ef4a468f7f6b2628e96cb5474d3a4ffcae305c978ac7caafaf5abc8f75f7`

$` docker ps`
| CONTAINER ID | IMAGE        | COMMAND                  | CREATED         | STATUS        | PORTS   | NAMES        |
|--------------|--------------|--------------------------|------------------|---------------|---------|--------------|
| f6f3ef4a468f | nginx:latest | "/docker-entrypoint.…"   | 4 seconds ago    | Up 3 seconds  | 80/tcp  | secure_cust  |
| 807ad56b0de5 | nginx        | "/docker-entrypoint.…"   | 9 minutes ago    | Up 9 minutes  | 80/tcp  | logout       |
| 3a501f40e29e | nginx        | "/docker-entrypoint.…"   | 10 minutes ago   | Up 10 minutes | 80/tcp  | login        |







