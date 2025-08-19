# 🐳 Docker Internals & Container Behavior — Notes

---

## 1️⃣ Container Basics

* A **container = just a process** on the host machine, isolated using:

  * **Namespaces** → isolate resources (PID, network, mount, etc.).
  * **cgroups** → control resources (CPU, memory, I/O).
  * **root filesystem** → built from **image layers**.

👉 **So a container is not a VM**. It shares the host kernel, but runs isolated processes.

---

## 2️⃣ Docker Images & Layers

* An **image** is a stack of **read-only layers** (built using Dockerfile instructions).
* These layers are managed by a storage driver (e.g., `overlay2` in Linux).

### Example from Docker internals (`docker inspect`):

```
LowerDir:  read-only image layers (FROM, RUN, COPY, etc.)
UpperDir:  writable container layer (changes made at runtime)
MergedDir: what the container actually sees
WorkDir:   internal working directory for overlay2
```

---

## 3️⃣ Copy-on-Write (CoW)

* Containers use a **Copy-on-Write** mechanism:

  * If a file exists in an image layer and you read it → it’s used directly.
  * If you modify it → Docker copies it into the writable layer (UpperDir) and applies the change.
  * If you delete it → Docker creates a **whiteout file** in the writable layer to hide it.

👉 This makes containers efficient: many containers can share the same base image layers.

---

## 4️⃣ Writable Layer

* Every container has **one writable layer on top of the image**.
* Anything you write/change/delete in a running container goes here.

### Properties:

* Exists as long as the container exists.
* Deleted when the container is removed.
* Changes **are not automatically saved to the image**.

👉 To persist changes:

1. Use `docker commit` → create new image with your changes.
2. Use `Dockerfile` → rebuild image properly (preferred).
3. Use **volumes** → keep data outside the container.

---

## 5️⃣ Dockerfile: COPY vs ADD

* **COPY**: copy files from host → image.
* **ADD**: same as COPY, but also supports:

  * Extracting tar archives.
  * Downloading from URLs.

👉 Best practice: use `COPY` unless you need `ADD`’s special behavior.

---

## 6️⃣ Volumes

* **Volumes** store data **outside the container’s writable layer**.
* Survive container restarts/removal.
* Perfect for:

  * Databases (`/var/lib/mysql`)
  * User uploads (images, docs)
  * Logs

### Example:

```bash
docker run -d -v mydata:/app/data ubuntu
```

👉 Rule of thumb:

* **Image (Dockerfile)** → static, repeatable app code/config.
* **Volumes** → dynamic runtime data.

---

## 7️⃣ Attached vs Detached Mode

* `docker run -it ...` → **attached mode**

  * You interact directly with container (like a terminal).
  * If you exit → the main process ends → container stops.

* `docker run -dit ...` → **detached mode**

  * Runs in background.
  * Container keeps running even if you disconnect.
  * To interact again: `docker attach <id>` or `docker exec -it <id> bash`.

👉 The difference isn’t about stopping itself — it’s about whether the container keeps running **after you disconnect**.

---

## 8️⃣ What Happens on Host Shutdown?

* Containers are just processes.
* If the host machine shuts down/crashes → containers stop immediately.
* Data persistence:

  * **Image layers** → safe.
  * **Writable layer** → safe if container not deleted.
  * **Volumes** → always safe.

👉 To make containers restart automatically:

```bash
docker run -d --restart always nginx
```

---

## 9️⃣ runc & “Spawning”

* Docker Engine relies on **`runc`** to actually start containers.
* `runc` = low-level tool that **spawns a process inside namespaces & cgroups**.
* "Spawn" = create a new process.
* In Linux, this happens via **fork + exec** system calls.

---

## 🔟 Useful Commands

* **Inspect storage layers**

  ```bash
  docker inspect <container>
  ```
* **List running processes inside container**

  ```bash
  docker exec -it <id> ps -ef
  ```
* **Save changes as image**

  ```bash
  docker commit <container> newimage:tag
  ```
* **Run in attached/detached**

  ```bash
  docker run -it ubuntu bash   # attached
  docker run -dit ubuntu bash  # detached
  ```
* **Mount volume**

  ```bash
  docker run -v mydata:/app/data ubuntu
  ```
* **Check restart policy**

  ```bash
  docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
  ```

---

# ✅ Summary

* A **container is just an isolated process** (via namespaces + cgroups).
* **Images** = read-only layers. **Containers** = image + writable layer.
* **Copy-on-Write (CoW)** makes changes efficient.
* **Writable layer** is ephemeral → removed with container.
* **Volumes** persist runtime data.
* **Attached mode** ties lifecycle to your session, **detached mode** doesn’t.
* **Host crash** = containers stop (like any process), but data persists (if volumes used).
* **runc spawns containers** by creating new processes in isolated environments.

---
