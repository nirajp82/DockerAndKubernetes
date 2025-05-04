## 📁 Default Docker File System Location

When Docker is installed, it uses the default directory (in linux):

```
/var/lib/docker
```

This directory includes several subdirectories, each responsible for storing different components like images, containers, and volumes.

### 🗂 Example Directory Structure

```bash
ls /var/lib/docker
```
![image](https://github.com/user-attachments/assets/e5117cac-f69c-42fd-88b0-53bbf1034bf4)

* `aufs/`: Stores the layers for images and containers.
* `containers/`: Stores metadata and logs for each container.
* `image/`: Stores metadata for images.
* `network/`, `plugins/`, etc.: Other Docker internals.

---

## ⚙️ Storage Driver: AUFS

Docker uses a **storage driver** to manage image and container layers. To check which driver is in use:

```bash
# Display a summary of Docker disk usage
docker system df | more
```

**Sample Output Snippet:**

![image](https://github.com/user-attachments/assets/7bc99508-0b19-4c5d-aa31-bbf501f28b1b)

> 🔹 **AUFS** (Advanced Multi-Layered Unification Filesystem) is the default driver on **Ubuntu/Debian** systems.

---

## 📂 AUFS Structure Explained

AUFS uses three main subfolders under `/var/lib/docker/aufs`:
![image](https://github.com/user-attachments/assets/e851f3d8-6b8e-4636-bc4f-4b901366f051)

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
Check AUFS `diff/`:

```bash
ls /var/lib/docker/aufs/diff
```
![image](https://github.com/user-attachments/assets/6bcb256b-0a3a-4940-911c-c093b1b82043)

---

## 🧠 View Image Build History

Use the `docker history` command to trace how an image was built:

```bash
docker history hello-world
```
![image](https://github.com/user-attachments/assets/c754a959-0baf-458b-aa9c-a93a7827e2e1)

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

![image](https://github.com/user-attachments/assets/0ddfa3dc-526e-44e6-b3b5-b815c446c858)

### Build the Image

```bash
docker build -t simple-web-app .
```

🕒 **First-time build** may take several minutes (downloads base images, installs dependencies).

## 🧰 Docker Caching

Rebuild the same image:

```bash
docker build -t simple-web-app .
```

You’ll see messages like Using cache for each step, meaning Docker is reusing previously built layers. Since dependencies were already built earlier, the rebuild completes in about 1 second — it's fast because no new changes require a full rebuild.

**Docker caches image layers**, so it skips unchanged steps during rebuild.

---

## 🕵️ View Image History

```bash
docker history simple-web-app
```

![image](https://github.com/user-attachments/assets/a0d0347e-3f47-4752-a541-16850b5b8825)

## 📊 Checking AUFS Layer Sizes

```bash
du -sh /var/lib/docker/aufs/diff/*
```
![image](https://github.com/user-attachments/assets/6c53d13b-80f5-44ce-9e2a-b5b3900f60d1)

Match large folders (e.g., 376MB) to corresponding layers (e.g., Flask dependencies).

Locate your app code:

* Look for the smallest folders (e.g., 12K)
* `ls` or `cat` to view the actual Python file
---
Find and view the content of app.py file.
![image](https://github.com/user-attachments/assets/91124f80-e196-418a-bff1-ada50ed5dd19)

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
![image](https://github.com/user-attachments/assets/18ec41d4-ca6b-469d-89b0-53f8fa74f031)

---
### 🧊 Docker Images and Size Explained

After building our Docker images,  even though the **second image** (with just a tiny change in application code) shared **most layers with the first**, both images were reported as **467 MB** using `docker images`.

![image](https://github.com/user-attachments/assets/823d7df6-04fc-44ca-9942-1cb7c876946a)

#### 🧾 Why Same Size Despite Small Changes?

Let’s break this down:

* Docker uses a **layered architecture**, where each instruction in the Dockerfile creates a new layer.
* When you make a change (like modifying your app source code), Docker only rebuilds **from the changed instruction onward** — earlier layers are reused from **cache**.
* However, the size shown in `docker images` is **not just the size of the unique layers** in that image. It includes the **cumulative size of all layers** that the image references, even if they are **shared with other images**.

> ✅ **Important Note:**
> The `docker images` output shows the **total size** of an image as if it were to be exported independently — including shared base layers like Ubuntu.

#### 📌 Key Points to Remember

* **Layer Sharing:** Layers are shared across images to save disk space **internally**, but each image reports the full size of its own layer stack.
* **Cache Efficiency:** Docker builds are fast when reusing cached layers. But once a single instruction changes, **all subsequent steps** must be rebuilt.
* **Output of `docker images`:** Shows:

  * **REPOSITORY** – name (e.g., `simple-web-app`)
  * **TAG** – version label (e.g., `latest`)
  * **IMAGE ID** – unique identifier
  * **CREATED** – when it was built
  * **SIZE** – **total virtual size** (sum of all layers in the image)

#### 🧪 Commands Recap

```bash
docker images
```

**Output:**

```
REPOSITORY       TAG       IMAGE ID       CREATED         SIZE
simple-web-app   latest    7f3b7c8922d3   1 minute ago    467MB
sample-web-app2  latest    e6a4bc8322d2   10 seconds ago  467MB
```


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

---------------

In a windows default installation, layers are stored in C:\ProgramData\dockerdesktop and split across the "image" and "windowsfilter" directories. You can change where the layers are stored using the docker-root configuration, as demonstrated in the Docker Engine on Windows documentation.

Ref:

https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/container-storage
