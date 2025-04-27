
# Docker Registry - Complete Guide

## 📦 What is a Docker Registry?

A **Docker Registry** is a **centralized storage system** where **Docker images** are stored, shared, and managed.

When you **run** a container, Docker first looks for the image **locally**.  
- If the image is **not found locally**, Docker will **pull it from a Docker registry** (such as Docker Hub or a private registry).

When you **build** a container image, it is created **locally** on your machine.  
- To make the image available to others or to deploy it from different machines, you must **manually push** the image to a Docker registry.

In simple terms:
- **Running** a container: Docker **pulls** the image from a registry (if it's not available locally).
- **Building** an image: You **create it locally**; you **must push** it manually to a registry if needed.

---
## 🧠 Understanding with an Example

Let's look at a basic example:

```bash
docker run nginx
```

When you run this command, Docker will **pull the `nginx` image** and **start a container** based on it.  

But where exactly is this `nginx` image coming from?

---
## 🔍 Breaking Down Docker Image Naming

Docker images follow a specific naming convention:

```text
[registry-url/]username/repository:tag
```

When you simply specify `nginx`, Docker assumes:

- **Registry**: `docker.io` (default Docker Hub registry)
- **Username**: `library` (an official Docker-maintained namespace)
- **Repository**: `nginx`
- **Tag**: `latest` (by default, if no tag is specified)

So the **full expanded image name** is:

```text
docker.io/library/nginx:latest
```

> 🧠 **Note:**  
> - `library/` is used for **official images** (like nginx, ubuntu, redis, etc.).  
> - When you push your own images, `username/imagename` structure is followed.  

---
## 📡 Popular Public Registries

Besides Docker Hub (`docker.io`), there are other major registries:

| Provider | Registry URL | Notes |
|:---------|:-------------|:------|
| Docker Hub | docker.io | Default public registry |
| Google Container Registry (GCR) | gcr.io | Used heavily by Kubernetes |
| AWS Elastic Container Registry (ECR) | Account-specific | Private by default |
| Azure Container Registry (ACR) | Account-specific | Private by default |

Anyone can **pull public images** without authentication.

---
## 🔒 Private Registries

When you have **in-house applications** that you do not want to expose to the public, you can:

- Use cloud-provided private registries (AWS ECR, GCR, ACR).
- Host your **own private registry**.

✅ Private registries require **authentication** before you can push or pull images.

---

## 🚪 Accessing Private Registries

First, **login** to the private registry:

```bash
docker login myprivateregistry.com
```

You’ll be prompted for a **username** and **password**.

Then, to **pull or push** images:

```bash
docker pull myprivateregistry.com/myusername/myimage:tag
docker push myprivateregistry.com/myusername/myimage:tag
```

If you **don't log in**, you will get an error like:

```text
Error response from daemon: pull access denied for myimage, repository does not exist or may require 'docker login'
```

---
## 🛠️ Setting Up Your Own Private Registry

If you don't want to depend on cloud providers, you can **run your own Docker Registry server**.

Luckily, **Docker Registry** itself is available as a container!

### Running a local registry:

```bash
docker run -d -p 5000:5000 --name registry registry:2
```

- `registry:2` is the Docker image for the official Registry application.
- It will expose the registry service on **port 5000** of your machine.

---

## 📤 Pushing an Image to Your Local Private Registry

### Step 1: Tag your local image
Suppose you have a local image with tag called `myapp:latest`.  
You need to tag it so Docker knows to push it to your private registry:

```bash
docker tag myapp:latest localhost:5000/myapp
```

Got it! Here's a more detailed and rephrased version with all the key information:

---

### Step 1: Tag Your Local Image for the Registry

If you have a local image called `myapp:latest`, you need to tag it with the address of your private registry so Docker knows where to push it. You can do this by using the `docker tag` command:

```bash
docker tag myapp:latest localhost:5000/myapp
```

This command **creates a new tag** (` localhost:5000/myapp`) for the existing image `myapp:latest`,  This new name (tag) includes the registry address (`localhost:5000/myapp`).  

This new tag is essential because **Docker requires that the image be tagged with the registry address** before it can be pushed to that registry.

To **push** the image to the registry, you must first create a tag that includes the registry's address — **this step is mandatory** before using the `docker push` command.

---
### Step 2: Push the image

After **tagging** the image in Step 1 (with the registry address), the next step is to **push** the image to the registry.

```bash
docker push localhost:5000/myapp
```

Here’s how it works:

- **`docker push`** uploads your local image to the registry you specified.
- The **`localhost:5000/myapp`** is the **tag** you created in Step 1, which includes the registry address (`localhost:5000`) and the image name (`myapp`).
- By running this command, Docker **sends the image** that was previously tagged with `localhost:5000/myapp` to your private registry running on `localhost:5000`.

### How Step 1 and Step 2 Work Together

- **Step 1:** You used the `docker tag` command to give your local image the **correct registry address** (`localhost:5000/myapp`).
- **Step 2:** Once the image has been tagged properly, you can **push** it to your registry using `docker push`. This step **transfers** the image from your local Docker storage to the registry.

### Summary of the Process:
1. **Tag the image** with the registry address using `docker tag` — this creates a reference that includes the registry's location.
2. **Push the image** to that registry using `docker push` — this uploads the image to the private registry so it can be accessed later.

---
## 📥 Pulling an Image from Your Local Private Registry

From the same machine (or from another machine in the same network), you can pull it:

```bash
docker pull localhost:5000/myapp
```

If accessing from a different host, replace `localhost` with the **Docker host's IP address** or domain name:

```bash
docker pull 192.168.1.100:5000/myapp
```

---
## 📚 Summary

- **Docker Registry** stores Docker images.
- **Docker Hub (docker.io)** is the default public registry.
- **Other registries** include GCR, AWS ECR, Azure ACR.
- **Private registries** can be cloud-hosted or self-hosted.
- **Authentication** is required for private registries.
- You can **run your own private Docker Registry** using the `registry` image.
