# 📦 Understanding Docker File System & Storage Drivers

## 🔍 Overview

This guide walks you through **how Docker stores data**, specifically the **file system structure**, **storage drivers**, and **how image layers and containers** are physically stored on disk. It also demonstrates **how caching works during image builds** and how to inspect Docker image history.

---

## 📁 Default Docker File System Location

When Docker is installed, it uses the default directory:

```
/var/lib/docker
```

This directory includes several subdirectories, each responsible for storing different components like images, containers, and volumes.

### 🗂 Example Directory Structure

```bash
ls /var/lib/docker
```

Typical output:

```
aufs  containers  image  network  plugins  swarm  ...
```

* `aufs/`: Stores the layers for images and containers.
* `containers/`: Stores metadata and logs for each container.
* `image/`: Stores metadata for images.
* `network/`, `plugins/`, etc.: Other Docker internals.

---

## ⚙️ Storage Driver: AUFS

Docker uses a **storage driver** to manage image and container layers. To check which driver is in use:

```bash
docker info
```

**Sample Output Snippet:**

```
Storage Driver: aufs
Docker Root Dir: /var/lib/docker/aufs
```

> 🔹 **AUFS** (Advanced Multi-Layered Unification Filesystem) is the default driver on **Ubuntu/Debian** systems.

---

## 📂 AUFS Structure Explained

AUFS uses three main subfolders under `/var/lib/docker/aufs`:

| Folder    | Description                             |
| --------- | --------------------------------------- |
| `diff/`   | Actual content of each image layer.     |
| `layers/` | Metadata about how layers are stacked.  |
| `mnt/`    | Mount points for container filesystems. |

---

## 🐳 Pulling a Sample Image

Let's pull the official **hello-world** image:

```bash
docker pull hello-world
```

To confirm:

```bash
docker images
```

**Output:**

```
REPOSITORY     TAG       IMAGE ID       CREATED         SIZE
hello-world    latest    2cb0d9787c4d   5 weeks ago     13.3kB
```

Check AUFS `diff/`:

```bash
ls /var/lib/docker/aufs/diff
```

You’ll find a folder representing a single layer with a script named `hello`.

---

## 🧠 View Image Build History

Use the `docker history` command to trace how an image was built:

```bash
docker history hello-world
```

This reveals:

* `COPY` step (adding the hello script)
* `CMD` step (running the script)

---

## 🛠️ Manually Run a Script from a Layer (Advanced/Hack)

Navigate to the AUFS diff folder and manually run the script stored in the image layer (not generally recommended):

```bash
cd /var/lib/docker/aufs/diff/<layer_id>
./hello
```

---

## 🧪 Building a Custom Docker Image (Flask App)

### Sample Flask App Structure

```bash
app.py
Dockerfile
```

### Dockerfile Example

```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip3 install flask
COPY app.py /opt/app.py
ENTRYPOINT ["python3", "/opt/app.py"]
```

### Build the Image

```bash
docker build -t simple-web-app .
```

🕒 **First-time build** may take several minutes (downloads base images, installs dependencies).

Check the image:

```bash
docker images
```

Expected output:

```
REPOSITORY       TAG       IMAGE ID       CREATED          SIZE
simple-web-app   latest    abcdef123456   a few seconds ago  467MB
```

---

## 🧰 Docker Caching

Rebuild the same image:

```bash
docker build -t simple-web-app .
```

⚡ Output will show `Using cache` for each instruction — rebuild completes in \~1 second.

**Docker caches image layers**, so it skips unchanged steps during rebuild.

---

## 🕵️ View Image History

```bash
docker history simple-web-app
```

Example output:

```
SIZE      CREATED BY
366MB     RUN pip3 install flask
...
```

---

## 📊 Checking AUFS Layer Sizes

```bash
du -sh /var/lib/docker/aufs/diff/*
```

Match large folders (e.g., 376MB) to corresponding layers (e.g., Flask dependencies).

Locate your app code:

* Look for the smallest folders (e.g., 12K)
* `ls` or `cat` to view the actual Python file

---

## 🔁 Rebuilding with Modified Code

Make a **minor change** in `app.py`.

Create a new Dockerfile if filename changed (e.g., `Dockerfile2`):

```Dockerfile
FROM ubuntu
RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip3 install flask
COPY app2.py /opt/app2.py
ENTRYPOINT ["python3", "/opt/app2.py"]
```

Build with the custom Dockerfile:

```bash
docker build -f Dockerfile2 -t simple-web-app-2 .
```

🚀 Output will show first few layers **using cache**, while last steps (COPY, ENTRYPOINT) are rebuilt.

---

## 📌 Important Notes

1. **Docker Layer Caching Works Top-Down:**

   * If step 4 changes, all steps **after** it are rebuilt.
   * Only **unchanged instructions from the top** are cached.

2. **Image Size Consistency:**

   * Even if only 1KB of code changes, Docker still shows total image size (including base layers).

3. **AUFS Folder Contents:**

   * All actual file contents are in `/var/lib/docker/aufs/diff/`
   * You can inspect file sizes using `du -sh`
   * Each folder is a **layer**, and the contents map to instructions in Dockerfile.

4. **Avoid manual tampering** in `/var/lib/docker` — it’s for internal use.

---

## ✅ Summary

| Action                     | Command                                   |
| -------------------------- | ----------------------------------------- |
| Check Docker info          | `docker info`                             |
| View images                | `docker images`                           |
| Check AUFS layer folder    | `ls /var/lib/docker/aufs/diff/`           |
| Build Docker image         | `docker build -t <name> .`                |
| Use a different Dockerfile | `docker build -f Dockerfile2 -t <name> .` |
| Inspect image history      | `docker history <image>`                  |
| View AUFS layer sizes      | `du -sh /var/lib/docker/aufs/diff/*`      |

---

## 🧠 Tips to Remember

* Docker **stores all its data in `/var/lib/docker`**.
* AUFS is a **layered filesystem**, each Dockerfile instruction = one layer.
* Layers are reused during builds using Docker's **build cache**.
* Storage drivers like AUFS control **how Docker manages file contents** on disk.
