# Docker Storage Drivers and File Systems - Practical Guide

## Overview

In this guide, we will explore Docker's storage drivers and file systems, including where Docker stores its data, how the layering system works, and how to persist data beyond container lifecycle. This practical guide will provide examples so you can verify the concepts we discuss.

---

## 1. Docker Data Storage Location

When Docker is installed on a system, it creates a folder structure under `/var/lib/docker`. This is where Docker stores all its data related to containers, images, and volumes. Let's explore this further with a practical verification.

### Practical Steps:
1. **Inspect the Docker Folder Structure**: 
   Run the following command to check where Docker stores its data:
   ```bash
   ls /var/lib/docker
   ```

   You should see multiple subdirectories, including `aufs`, `containers`, `images`, `volumes`, etc., depending on your storage driver.

2. **Inspect Docker Container Data**: 
   If you have running containers, you can check the contents of the containers directory:
   ```bash
   ls /var/lib/docker/containers
   ```
   This will show directories named by the container IDs, containing the data for each container.

---

## 2. Docker's Layered Architecture

Docker images are built using a layered architecture. Each layer in the image corresponds to a Dockerfile instruction, and Docker uses caching to reuse layers from previous builds to make the process more efficient.

### Practical Steps:
1. **Create a Sample Dockerfile**:
   Create a `Dockerfile` with the following content:
   ```Dockerfile
   FROM ubuntu
   RUN apt-get update && apt-get install -y python3
   COPY . /app
   CMD ["python3", "/app/app.py"]
   ```

2. **Build the Image**:
   Build the Docker image using the following command:
   ```bash
   docker build -t my-python-app .
   ```

3. **Inspect Image Layers**:
   After building the image, run the following command to inspect the layers:
   ```bash
   docker history my-python-app
   ```
   This will show the layers created by each instruction in the Dockerfile, including their sizes.

4. **Rebuild the Image**:
   Modify the `COPY` instruction or the source code, and rebuild the image. Docker will reuse layers from the previous build, as shown in the output of the `docker history` command.

---

## 3. Image Layers and Read-Only Architecture

Docker images are immutable once built. The image layers are read-only, and any changes made in a container based on the image are stored in a new writable layer on top.

### Practical Steps:
1. **Run a Container**:
   Start a container using the image you built:
   ```bash
   docker run -it my-python-app bash
   ```

2. **Create a File Inside the Container**:
   Once inside the container, create a new file:
   ```bash
   touch /app/temp.txt
   ```

3. **Exit the Container**:
   Exit the container and inspect its writable layer:
   ```bash
   docker ps -a
   docker inspect <container_id>
   ```

   Notice that the `temp.txt` file is not part of the original image; it's only stored in the container's writable layer.

4. **Remove the Container**:
   When you remove the container:
   ```bash
   docker rm <container_id>
   ```
   The file `temp.txt` will be deleted because it was part of the container's writable layer.

---

## 4. Copy-On-Write Mechanism

When you modify a file inside a container, Docker uses the **Copy-On-Write (COW)** mechanism. The original file from the image remains unchanged in the read-only layer, while the modification occurs in the writable layer.

### Practical Steps:
1. **Modify a File Inside the Container**:
   Run the container again and modify a file that was part of the image:
   ```bash
   docker run -it my-python-app bash
   echo "Hello World" >> /app/app.py
   ```

2. **Inspect the Writable Layer**:
   When you modify the file, Docker copies it to the writable layer. To verify, inspect the container’s filesystem:
   ```bash
   docker inspect <container_id>
   ```

3. **Verify the Changes**:
   You can check if your changes are persistent within the container's writable layer:
   ```bash
   docker exec -it <container_id> cat /app/app.py
   ```

---

## 5. Persisting Data with Volumes

Volumes allow you to persist data outside of the container lifecycle. Even if the container is destroyed, the data stored in the volume remains intact.

### Practical Steps:
1. **Create a Volume**:
   Create a Docker volume:
   ```bash
   docker volume create data_volume
   ```

2. **Run a MySQL Container with Volume**:
   Mount the volume to the MySQL container to store the database data persistently:
   ```bash
   docker run -d -v data_volume:/var/lib/mysql --name mysql-container mysql
   ```

3. **Inspect the Volume**:
   After running the container, you can list the volumes:
   ```bash
   docker volume ls
   ```

4. **Remove the MySQL Container**:
   Stop and remove the container:
   ```bash
   docker stop mysql-container
   docker rm mysql-container
   ```

5. **Verify Data Persistence**:
   After removing the container, create a new container using the same volume:
   ```bash
   docker run -d -v data_volume:/var/lib/mysql --name mysql-container mysql
   ```

   The data from the volume should still be available even though the container was removed and recreated.

---

## 6. Volume Mounting vs. Bind Mounting

Docker supports both volume mounting (managed by Docker) and bind mounting (where a specific host directory is mounted into the container).

### Practical Steps:
1. **Volume Mounting**:
   Mount a Docker-managed volume:
   ```bash
   docker run -d -v /data/mysql:/var/lib/mysql mysql
   ```

2. **Bind Mounting**:
   Mount a host directory to a container:
   ```bash
   docker run -d -v /data/msql:/var/lib/mysql mysql
   ```

3. **Verify Mounts**:
   After running these commands, you can inspect the mounts using:
   ```bash
   docker inspect <container_id>
   ```

---

## 7. Docker Storage Drivers

Docker uses storage drivers to manage how layers and files are stored for containers. The choice of storage driver depends on the operating system and file system capabilities.

### Practical Steps:
1. **Check the Storage Driver**:
   To find out which storage driver Docker is using, run:
   ```bash
   docker info | grep "Storage Driver"
   ```

2. **Storage Driver Options**:
   Docker supports various storage drivers such as AUFS, Overlay2, ZFS, and Device Mapper. The choice of driver can affect performance and compatibility.

---
