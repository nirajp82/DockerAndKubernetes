# Docker Networking Explained

Docker is a containerization platform that uses OS-level virtualization to package software applications and their dependencies into reusable units called **containers**. Docker containers can be run on any host with Docker or an equivalent container runtime installed, whether locally on your laptop or in a remote cloud.

Docker includes a **networking system** for managing communications between containers, your Docker host, and the outside world. Several different network types are supported, facilitating a variety of common use cases.

In this article, we’ll explain how Docker networks function, then cover the basics of using them with your container deployments.

---

## 📋 We Will Cover:

* Docker Network Types
* How Docker Networking Works
* Using Docker Networks
* Docker Compose Networking
* Key Takeaways

---

## 🧠 Docker Network Types

Docker networks configure communications between neighboring containers and external services. Containers must be connected to a Docker network to receive any network connectivity. The communication routes available to the container depend on the network connections it has.

Docker comes with **five built-in network drivers** that implement core networking functionality:

* `bridge`
* `host`
* `overlay`
* `ipvlan`
* `macvlan`

### 1. `bridge`

* Software-based bridge between your host and the container.
* Containers connected to the bridge can talk to each other.
* Containers are isolated from the host and other networks unless specifically configured.
* Each container gets its own IP.
* They **can access your LAN and the internet**, but **won’t appear as physical devices** on your LAN.

### 2. `host`

* Containers **share the host's network stack** directly.
* No separate IP is assigned to the container.
* Ports exposed by the container are **bound directly to your host's network interface**.
* Example: A container listening on port 80 will bind to `<your_host_ip>:80`.

### 3. `overlay`

* **Distributed networks** that span multiple Docker hosts.
* Essential for Docker Swarm but also usable outside of Swarm for **multi-host communication**.
* No OS-level routing configuration needed.
* Supports **scaling and high availability**.

### 4. `ipvlan`

* Advanced networking mode with **fine-grained control** over IPs and VLAN tagging.
* Useful when integrating Docker with **physical networks**.
* Better performance than bridge due to **dedicated interfaces**.

### 5. `macvlan`

* Allows containers to appear as **physical devices** on the network.
* Each container gets a **unique MAC address**.
* Requires **dedicated physical interface** from host.
* Ideal for **monitoring or low-level networking tools**.

---

## ❓ Which Network Type Should I Use?

| Use Case                           | Recommended Network |
| ---------------------------------- | ------------------- |
| Most common, simple scenarios      | `bridge`            |
| Accessing host network directly    | `host`              |
| Cross-host container communication | `overlay`           |
| Container as physical device       | `macvlan`           |
| Advanced VLAN/IP management        | `ipvlan`            |

Docker also supports **third-party network plugins** like:

* **Kuryr** – integrates with OpenStack Neutron
* **Weave** – focuses on security, service discovery, and fault tolerance

You can also set `--network none` to **disable networking**, improving container isolation for untrusted workloads.

---

## ⚙️ How Docker Networking Works

Docker uses your host’s network stack to implement its networking system. Here’s how:

* Docker manipulates **iptables** rules to route traffic.
* Every container has its own **network namespace**, isolating it at the kernel level.
* Virtual interfaces on your host allow containers to send/receive traffic.

You don’t need to configure iptables manually — Docker handles it.

---

## 🆚 Docker Networking vs. VM Networking

| Feature     | Docker                            | VM                       |
| ----------- | --------------------------------- | ------------------------ |
| Isolation   | Network namespaces                | Full networking stack    |
| NAT-based?  | Bridge (like NAT)                 | Usually full NAT         |
| Flexibility | Plugins/macvlan for complex cases | More flexible by default |

Docker offers the tools to **replicate advanced topologies**, including macvlan and third-party plugins.

---

## 🧪 Using Docker Networks

Let’s dive into using Docker’s networking system in practice.

### 🔧 Creating Networks

```bash
docker network create demo-network -d bridge
```

Output:

```
50ed05634f6a3312e56700ef683ca39df44bfc826e2e4da9179c2593c79910f9
```

> You've created a bridge network. Now let’s connect containers.

---

### 🔗 Connecting Containers to Networks

```bash
# Terminal 2
docker run -it --rm --name container1 --network demo-network busybox:latest
```

```bash
# Terminal 3
docker run -it --rm --name container2 busybox:latest
```

Attempt to ping:

```bash
# inside container1
ping container2  # ❌ fails
```

Now, connect container2 to the network:

```bash
docker network connect demo-network container2
```

Try ping again:

```bash
# inside container1
ping container2  # ✅ success
```

---

### 🌐 Using Host Networking

```bash
docker run -d --name nginx --network host nginx:latest
```

Now test from host:

```bash
curl localhost:80  # Should return the NGINX welcome page
```

---

### 🚫 Disabling Networking

```bash
docker run -it --rm --network none busybox:latest
```

Inside the container:

```bash
ping google.com  # ❌ fails
```

---

### 🔄 Removing Containers from Networks

```bash
docker network disconnect demo-network container2
```

---

### 📜 Managing Networks

List:

```bash
docker network ls
```

Delete:

```bash
docker network rm demo-network
```

Prune unused:

```bash
docker network prune
```

---

## 🧱 Using Networks With Docker Compose

Here’s a simple `docker-compose.yml` file:

```yaml
version: "3"
services:
  app:
    image: php:7.2-apache
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: changeme
```

Launch:

```bash
docker compose up -d
```

Check networks:

```bash
docker network ls
```

Verify connectivity:

```bash
docker compose exec -it app bash
apt update && apt install iputils-ping -y
ping mysql
```

---

### ➕ Adding Extra Networks

```yaml
version: "3"
services:
  app:
    image: php:7.2-apache
    networks:
      - db
  helper:
    image: custom-image:latest
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: changeme
    networks:
      - db
networks:
  db:
```

This ensures that `app` and `mysql` can talk to each other, but `helper` is isolated.

---

## 📌 Key Points

* Docker networks allow **flexible communication** between containers, hosts, and external services.
* There are **five core drivers**, each designed for specific scenarios.
* Networks use **host kernel features** like namespaces and iptables.
* **Compose automates** network creation and container wiring.
* You can **disable networking** entirely for security-hardened workloads.

---

## 📚 You Might Also Like

* **[Kubernetes Networking Explained](https://kubernetes.io/docs/concepts/cluster-administration/networking/)**
* **Common Infrastructure Challenges and How to Solve Them**
* **16 DevOps Best Practices to Follow**
* **Top Most Useful CI/CD Tools for DevOps**


