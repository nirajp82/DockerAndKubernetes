# Docker Storage Drivers and File Systems

Docker's storage mechanisms are crucial for understanding how data is managed, stored, and persisted within containers.

## 1. Docker Data Storage Location

When Docker is installed on a system, it creates a folder structure to store all its data related to containers, images, volumes, and other components.

### For Linux

By default, Docker on Linux stores its data under `/var/lib/docker`. This is where Docker manages containers, images, volumes, and networks.

#### Practical Steps:

1. **Inspect the Docker Folder Structure**: 
   To inspect the folder structure where Docker stores data, run the following command:
   ```bash
   ls /var/lib/docker
   ```
   ![image](https://github.com/user-attachments/assets/d5c6187f-68ce-4a11-8bc5-f24430eff95e)

   You should see multiple subdirectories based on your storage driver (e.g., `aufs`, `containers`, `images`, `volumes`). The directory layout depends on the storage driver used.

2. **Inspect Docker Container Data**: 
   You can check the `containers` directory to see the data associated with each container:
   ```bash
   ls /var/lib/docker/containers
   ```
   This will show directories named by container IDs, containing the container's data and logs.
   ![image](https://github.com/user-attachments/assets/e197c1a3-822d-498e-8ec3-a71818872875)
   
---

## 2. Docker's Layered Architecture

Docker images are built using a **layered architecture**. Each layer in the image corresponds to a Dockerfile instruction. Docker uses caching to reuse layers from previous builds, making the build process faster and more efficient.

### Practical Steps:

1. **Create a Sample Dockerfile**:
   Here’s an example of a Dockerfile:
   ```Dockerfile
   FROM ubuntu
   RUN apt-get update && apt-get install -y python3
   COPY . /app
   CMD ["python3", "/app/app.py"]
   ```

2. **Build the Image**:
   Build the image with the following command:
   ```bash
   docker build -t my-python-app .
   ```

3. **Inspect Image Layers**:
   To see the layers created by each Dockerfile instruction, run:
   ```bash
   docker history my-python-app
   ```

   This will show a detailed breakdown of the layers and their sizes.

4. **Rebuild the Image**:
   If you modify the `COPY` instruction or the source code, Docker will reuse layers from previous builds. Rebuild the image and inspect the layers again to see the changes:
   ```bash
   docker build -t my-python-app .
   docker history my-python-app
   ```

---

## 3. Image Layers and Read-Only Architecture

![image](https://github.com/user-attachments/assets/41fefcd9-65b2-4a52-91ac-07072176fba4)

Docker images are **immutable** once built. The image layers are **read-only**, and any changes made in a container created from the image are stored in a new writable layer on top.

### Practical Steps:

1. **Run a Container**:
   Run the container based on your built image:
   ```bash
   docker run -it my-python-app bash
   ```

2. **Create a File Inside the Container**:
   Once inside the container, create a new file:
   ```bash
   touch /app/temp.txt
   ```

3. **Exit the Container**:
   Exit the container and inspect the container’s data:
   ```bash
   docker ps -a
   docker inspect <container_id>
   ```

   You’ll notice that `temp.txt` is not part of the original image. It's only stored in the container's writable layer.

4. **Remove the Container**:
   When the container is removed, the file `temp.txt` will also be deleted:
   ```bash
   docker rm <container_id>
   ```

---

## 4. Copy-On-Write Mechanism

Docker uses the **Copy-On-Write (COW)** mechanism when you modify a file inside a container.  When a container changes a file, Docker doesn’t touch the original image file. Instead, it makes a copy in the container’s writable layer and edits that. The original stays safe and read-only, like a master copy.

![image](https://github.com/user-attachments/assets/ec5bb19f-7387-4d42-8b35-59bfcd23d997)

### Practical Steps:

1. **Modify a File Inside the Container**:
   Run the container again and modify a file from the image:
   ```bash
   docker run -it my-python-app bash
   echo "Hello World" >> /app/app.py
   ```

2. **Inspect the Writable Layer**:
   Docker copies the file to the writable layer when you modify it. To inspect this, run:
   ```bash
   docker inspect <container_id>
   ```

3. **Verify the Changes**:
   Check if your changes are persistent within the container’s writable layer:
   ```bash
   docker exec -it <container_id> cat /app/app.py
   ```

---

## 5. Persisting Data with Volumes

Volumes provide a way to persist data **outside of the container lifecycle**. Even if the container is destroyed, data stored in volumes remains intact.

### Why Persist Data with Volumes?

While Docker allows data to be stored in containers’ writable layers, **those layers are ephemeral**. This means that data is lost once the container is deleted. Volumes, on the other hand, provide persistent storage that outlasts the container's lifecycle.

### Practical Steps:

1. **Create a Volume**:
   To create a Docker volume, run:
   ```bash
   docker volume create data_volume
   ```

2. **Run a MySQL Container with Volume**:
   Mount the volume to the MySQL container to store database data persistently:
   ```bash
   docker run -d -v data_volume:/var/lib/mysql --name mysql-container mysql
   ```

   Here, the volume `data_volume` is mounted to `/var/lib/mysql` inside the container where MySQL stores its data.

3. **Inspect the Volume**:
   After running the container, you can list the volumes:
   ```bash
   docker volume ls
   ```

4. **Remove the MySQL Container**:
   Stop and remove the container to verify data persistence:
   ```bash
   docker stop mysql-container
   docker rm mysql-container
   ```

5. **Verify Data Persistence**:
   After removing the container, create a new container using the same volume:
   ```bash
   docker run -d -v data_volume:/var/lib/mysql --name mysql-container mysql
   ```

   The data in the volume should still be available even though the container was removed and recreated.

---

## 6. Volume Mounting vs. Bind Mounting

Docker supports two types of mounts: **volume mounting** (managed by Docker) and **bind mounting** (where a specific host directory is mounted into the container).

### Volume Mounting

- In volume mounting, Docker manages the storage location for you, ensuring persistence and flexibility across different systems.
- Docker picks the storage spot and manages it. It’s clean, portable, and works the same across different machines. Best for persistent, container-friendly data.

### Bind Mounting

- With bind mounting, a host directory is directly mounted into a container, allowing for more control but potentially less portability between systems.
- We have to tell Docker exactly where on your machine to connect. Great for development, but can break on other systems since paths may differ.

#### Practical Steps:

🔹 **Volume Mounting** – *Let Docker manage the volume*:

```bash
docker run -d -v mysql-data:/var/lib/mysql --name mysql-container mysql
```

* `mysql-data` is a **Docker-managed volume**.
* Use `docker volume ls` to see it.
* Good for portability and persistence.
* Docker stores volumes under: `/var/lib/docker/volumes/<volume-name>/_data`
 
🔹 **Bind Mounting** – *Use a specific host directory*:

```bash
docker run -d -v /data/mysql:/var/lib/mysql --name mysql-container mysql
```

* `/data/mysql` is a **host path**.
* Changes go directly to your filesystem.
* Great for local development but less portable.

3. **Verify Mounts**:
   After running the container, verify the mounts with:
   ```bash
   docker inspect <container_id>
   ```
![image](https://github.com/user-attachments/assets/4afb3e02-96b4-4b64-bc43-68af7276534c)
---

## 7. Docker Storage Drivers

Docker uses **storage drivers** to manage how layers and files are stored for containers. The choice of storage driver depends on the operating system and file system capabilities, and it can impact performance and compatibility.

Absolutely! Here's a simplified, memorable version of **Linux Storage Drivers** in Docker:

---

### 🧱 **Storage Drivers — "How Docker Layers Your Files"**

Think of storage drivers as **the engine under Docker’s file system** — they control how image layers and container changes are stored.

### 🔧 **Common Linux Storage Drivers**

#### 1. **AUFS** – *“The Classic Stack”*

* AUFS (Another Union File System) is a **union file system** used in Linux, similar to OverlayFS. It allows you to **combine multiple directories** (layers) into one unified view. stacks layers like pancakes 🍴.
* Was popular in early Docker versions.
* Fast and flexible, but **not always supported on newer kernels**.
* AUFS: Supports multiple lower layers and one upper writable layer. It can merge many layers (up to 128 or more), but it has a more complex internal structure.

### Key points about AUFS:

* **Layers**: AUFS allows you to stack several directories (Multiple lower layers) on top of each other, with one writable upper layer.
* **Read-Only**: The lower layers are typically read-only, while the top layer is where changes (writes) are made.
* **Copy-on-Write**: When a file in a lower layer is modified, AUFS copies it to the upper layer and then makes the changes, preserving the original file in the lower layer.

AUFS is commonly used in **Docker** before OverlayFS became popular. It helps maintain efficient storage while allowing modification without altering the original layers.


#### 2. **Overlay2** – *“The Modern Default”*

* Default on most modern Linux distros.
* Uses the **overlay** feature in Linux to combine layers.
* **Better performance and fewer bugs** than AUFS.
* OverlayFS: Supports only two layers — one lower (read-only) and one upper (writable). It’s simpler and more efficient with a focus on two layers.
* OverlayFS: Part of the mainline Linux kernel since version 3.18, meaning it’s natively supported and doesn’t need external patches.

#### OverlayFS in Linux is like a **transparent cover** over a base file system.

Imagine this:

* You have a **base folder** (lower layer) that contains original files (like a read-only template).
* You add a **top folder** (upper layer) that starts out empty — this is **writable**.
* **OverlayFS** merges them into a **single virtual view** (called a "merged" directory).

#### What happens when you access files?

##### 1. **Reading a file**

* If a file exists in the upper layer, you see that one.
* If not, it reads from the lower layer.
* **You never modify the original lower files.**

##### 2. **Writing or editing a file**

* If you write to a file from the lower layer, OverlayFS **copies it** to the upper layer (called "copy-up"), then modifies it **only in the upper layer**.
* The lower layer stays untouched.
* **When editing**, if you change a part of a file (e.g., `"abc"` to `"abd"`), OverlayFS copies the whole file to the upper layer and updates the necessary part.

##### 3. **Deleting a file**

* OverlayFS creates a **"whiteout" marker** in the upper layer to hide the lower-layer file.
* The **original file isn’t deleted** — it's just hidden from the merged view. The file still exists in the lower layer but isn’t accessible via the merged directory.

##### 4. **Reverting a deletion (or undoing a change)**

* If you delete a file from the upper layer, it’s only hidden in the merged view.
* If you recreate the file in the upper layer, OverlayFS will show the updated version, and the **lower layer file** is restored.
* If the file was never edited, it will show the lower layer version.

#### Why is this useful?

This is the **core of how Docker images and containers work**:

* Docker images are **read-only layers** (base).
* Containers are **writable layers** (upper).
* When you run a container, any change goes to the upper layer, leaving the image layer untouched.

🧠 **Think:** Fast, stable, and widely recommended.

#### 3. **ZFS** – *“The Feature Beast”*

* Built-in **snapshots, compression, and checksums**.
* Very powerful, but heavier and needs setup.
* Often used in **enterprise or power-user setups**.

🧠 **Think:** Luxury car — powerful but needs maintenance.

#### 4. **Device Mapper** – *“The Raw Block Manager”*

* Works at a low level with block devices.
* Flexible but **complex and slower** for some workloads.
* Not the first choice unless you need its features.

🧠 **Think:** Manual gear transmission — you better know what you're doing.


| Driver        | Use Case                      | Speed | Notes                        |
| ------------- | ----------------------------- | ----- | ---------------------------- |
| AUFS          | Legacy systems                | ⚡⚡    | Needs kernel support         |
| Overlay2      | Modern default (recommended)  | ⚡⚡⚡   | Fast and efficient           |
| ZFS           | Enterprise, advanced features | ⚡⚡    | Complex setup, rich features |
| Device Mapper | Low-level control             | ⚡     | Slower, more complex         |


### Storage Drivers on Windows

On Windows, Docker primarily uses **LCOW** (Linux Containers on Windows) when running Linux containers. However, Docker supports **Windows containers** as well, using storage drivers like **NTFS** and **ReFS** depending on the system's configuration.

#### Practical Steps:

1. **Check the Storage Driver**:
   To find out which storage driver Docker is using, run:
   ```bash
   docker info | grep "Storage Driver"
   docker info | more
   ```
   ![image](https://github.com/user-attachments/assets/b39dc718-8430-4d9c-aa61-716a49b12829)

2. **Storage Driver Options**:
   Docker supports multiple storage drivers, each with different characteristics. Depending on your operating system and use case, you may need to choose the appropriate driver for better performance or compatibility.
