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
* `None`
* `overlay`
* `ipvlan`
* `macvlan`

### 1. **Bridge** (Default for containers)

* This is the **default network** Docker uses when you don't specify one.
* It's a **private internal network** created on the Docker host.
* Containers in this network get internal IP addresses in the **172.17.x.x** range.
* Containers connected to the bridge can talk to each other. These containers can communicate with each other using these IP addresses.
* To allow external access, **map ports** from the container to the Docker host.
  > Example: `docker run -p 8080:80 my-web-app` maps container's port 80 to host's port 8080.
* Containers are isolated from the host and other networks unless specifically configured.
* Each container gets its own IP.
* They **can access your LAN and the internet**, but **won’t appear as physical devices** on your LAN.
Here's a diagram showing how the default **Docker bridge network** connects containers to each other, the **host**, and the **internet**:

##### 🖼️ Docker Bridge Network Diagram

![image](https://github.com/user-attachments/assets/9c99018c-6f61-40c1-8720-234b2714700c)

##### 🔍 Key Components

* **docker0**: The bridge interface on the host (`172.17.0.1` by default).
* **Containers**: Get IPs from the bridge subnet and can communicate with each other.
* **NAT (iptables)**: Docker sets up NAT rules so containers can access the **internet via host**.
* **Host ↔ Container communication**: Possible via the `docker0` interface.
* **Inbound from Internet**: Only works if ports are published (`-p 8080:80`), else containers are isolated.

This command creates a custom Docker bridge network with own IP subnet and gateway, instead of using Docker’s default settings. 
```sh
docker network create --driver bridge --subnet 182.18.0.0/24 --gateway 182.18.0.1 wp-mysql-network
```
#### 🧱 What It Does

It creates a **user-defined bridge network** named `wp-mysql-network` with the following characteristics:

| Option                   | Description                                                                                         |
| ------------------------ | --------------------------------------------------------------------------------------------------- |
| `--driver bridge`        | Uses the **bridge** driver — the default for standalone container communication on the same host.   |
| `--subnet 182.18.0.0/24` | Defines a **custom subnet**. This means containers on this network will have IPs like `182.18.0.x`. |
| `--gateway 182.18.0.1`   | Sets the network **gateway** — this is the "router" address for containers inside the network. All containers use it to leave the Docker network (e.g., to talk to the host or the internet). Each container sends outbound traffic to the gateway, and the gateway handles routing it to the outside world.Container A (IP: 182.18.0.2) ───▶ Gateway (182.18.0.1) ───▶ Internet (e.g., Google)|
| `wp-mysql-network`       | The **name** of the new network you’re creating. You’ll use this in `--network` flags.              |

#### 🔌 Why Use It?

Using a **custom bridge network** like this gives you:

* **Fixed IP ranges**: You know what IPs your containers will get — useful for predictable setups.
* **MySQL/WordPress isolation**: Containers in this network can talk only to each other.
* **Better DNS resolution**: Docker provides **internal DNS** when using named user-defined networks.
* **No port publishing required** (for internal communication): Containers can talk via internal IP/hostname.

#### 🧑‍🍳 Real Use Case: WordPress + MySQL

You could use this network to **connect WordPress and MySQL** containers like this:

```bash
# Step 1: Create the network
docker network create \
  --driver bridge \
  --subnet 182.18.0.0/24 \
  --gateway 182.18.0.1 \
  wp-mysql-network

# Step 2: Start the MySQL container
docker run -d \
  --name mysql-db \
  --network wp-mysql-network \
  -e MYSQL_ROOT_PASSWORD=root \
  -e MYSQL_DATABASE=wordpress \
  mysql:5.7

# Step 3: Start the WordPress container
docker run -d \
  --name wp-site \
  --network wp-mysql-network \
  -e WORDPRESS_DB_HOST=mysql-db \
  -e WORDPRESS_DB_USER=root \
  -e WORDPRESS_DB_PASSWORD=root \
  -p 8080:80 \
  wordpress:latest
```

✅ Now, `wp-site` can talk to `mysql-db` using internal DNS (`mysql-db:3306`) — no need to expose ports.

#### 📌 When to Use This Command

Use a custom bridge network like this when:

* You want better control over the IP range.
* You want DNS-based discovery between containers.
* You're running **multi-container apps** like WordPress + MySQL on the same host.
* You want to avoid conflicts with Docker’s default `172.x.x.x` bridge networks.

---
### 2. `host`

### 🧠 What is Docker's *Host Network*?

When a container uses the **host network**, it doesn't get its own virtual network or IP. Instead, it shares the same network as your computer (the **host**). It's like the container is running **directly on your machine's network**.

### 🧩 Key Points (in simple terms):

* 🚫 **No separate network**: The container **does not get its own IP address**.
* 🔗 **Shares the host’s ports**: If your container listens on port `80`, it actually uses your **host's port 80**.
* 🔒 **No isolation**: There’s **no network isolation** between the container and your host.
* ⚠️ **Only one container** can use a specific port at a time (since they share the host’s ports).

### 🧪 Example

If you run:

```bash
docker run --network host nginx
```

Then visiting `http://localhost` on your machine will directly connect to **nginx** inside the container — because it’s using your host’s port 80.

### ⚙️ When to Use It?

* When you need **fast network performance**.
* When your app **requires access to the host’s network interfaces** (e.g., for low-level networking, service discovery, or multicast).
* Use with **care** — you lose the security isolation of container networking.

![image](https://github.com/user-attachments/assets/7829e126-5e93-48b0-bb49-60842a32a6a3)
---

### 3. `None`
In Docker, the `None` network mode is a special configuration that isolates a container from all network communication. When a container is connected to a network with the `None` mode, it does **not** have any access to:

1. **External Networks**: The container cannot communicate with the outside world, including the internet, or other hosts on the local network.

2. **Other Containers**: It also cannot communicate with other containers on the same host, even if those containers are connected to the default bridge network or any other custom networks.

### How `None` Network Mode Works

* **Network Stack Disabled**: The container has no networking capabilities. This means that no interfaces (e.g., `eth0`) are created, and no IP addresses are assigned.
* **Use Cases**: This can be useful when you need a container that performs specific tasks, like running a process that doesn't need network communication, or if you're running a container with a security focus that should be fully isolated from network traffic.

### Example of Using `None` Network Mode


### Real-World Example: Database Backup Container

Imagine you're running a **database backup** task inside a Docker container, and you don't want it to communicate with any other container or external network. This task involves taking a snapshot of the database or writing backup files to the local filesystem. Since the backup process doesn't need any network access and should be isolated for security or efficiency, you could use Docker's `None` network mode.

### ❌ What is *not* possible:

* It **cannot connect to a database server**, whether local (on host) or remote, because that would require networking.
* Even `localhost`, `127.0.0.1`, or hostnames won't resolve — the container has no interface to do so.

### Updated Real-World Example:

Let’s say you're **processing a database dump file** (e.g., a `.sql` or `.bak` file) that’s already been created and stored locally. Your container’s job is just to:

* Compress it,
* Encrypt it,
* Or copy it to an archive directory.

You do **not** need network access because you're dealing with a **file**, not connecting to a database server.

```bash
docker run --network none -v /backups:/data my-processing-container process /data/db-dump.sql
```

Here:

* `--network none` ensures the container can’t communicate with anything.
* `-v /backups:/data` gives it access to the file system where the database dump is stored.
* Your container just processes that file in-place — no need to connect to a database server.

### So to be 100% clear:

> If you're trying to **connect to a live database**, `--network none` will not work.
> But if you're processing **an already-exported database file**, it's a perfect fit for secure, isolated operations.

---
## 4. Overlay Network Mode

### 🧠 Concept:

An **Overlay network** lets containers on **different physical machines (hosts)** talk to each other **as if they were on the same LAN** — even if they're running miles apart in separate data centers or clouds.

Docker handles all the routing and encryption between hosts using **VXLAN tunneling** under the hood.

---

### 📡 Behavior:

* Virtual network **spans multiple Docker hosts**
* Containers communicate as if they’re on the **same local subnet**
* No need to mess with **manual OS-level routes**
* Works great in **Docker Swarm** and multi-node setups

---

### 📦 Real-World Example:

You're building a **microservices application**. The frontend service runs on Host A, and the backend service runs on Host B. You want them to talk without worrying about physical IPs or routing.

Using an **overlay network**, both containers can connect to `my_overlay_net`, and Docker will take care of securely routing traffic between hosts.

```bash
docker network create --driver overlay my_overlay_net
```

Now you can do:

```bash
# On Host A
docker service create --name frontend --network my_overlay_net frontend-image

# On Host B
docker service create --name backend --network my_overlay_net backend-image
```
- You're creating Docker two services (frontend and backend) using Docker Swarm (not regular containers), and you’re connecting them to the same overlay network across multiple hosts.
- They are deployed on different hosts.
- The --network my_overlay_net puts them in the same virtual Docker network, so they can communicate securely and seamlessly, even across hosts.
- Docker Swarm handles the virtual network, routing, and service discovery behind the scenes.

---

### 🔐 Use Case Summary:

* Distributed applications across **multiple Docker hosts**
* **Docker Swarm** for service-to-service communication
* **Cloud-native** deployments where containers span VMs
* Simplifying **scaling and HA (high availability)** setup

---

## 🧬 How It Works (Behind the Scenes):

* Uses **VXLAN** (Virtual Extensible LAN) for tunneling traffic between hosts.
* Docker creates a **virtual switch** on each host and connects it to a mesh.
* No need to touch `iptables`, static routes, or networking config manually.

---

## 🧾 Quick Comparison with Other Networks

| Feature                | Bridge          | MacVlan              | IPVlan           | Overlay                 |
| ---------------------- | --------------- | -------------------- | ---------------- | ----------------------- |
| Host Scope             | Single Host     | Single Host          | Single Host      | **Multiple Hosts**      |
| MAC Address            | NAT’d from Host | Unique per Container | Same as Host     | Internal VXLAN MAC/IP   |
| Needs Promiscuous Mode | No              | Yes                  | No               | No                      |
| Best For               | Local Dev       | Legacy Networks      | Datacenter Rules | **Swarm / Distributed** |
| Example Use            | One-machine dev | MAC-locked apps      | MAC-limited NICs | Cloud-native services   |

---

### 🤝 When to Use Overlay:

* You're deploying a **Swarm** cluster or want **multi-host communication**.
* You need containers on **different servers** to discover and talk to each other.
* You're aiming for **horizontal scaling** and **high availability**.
* You want Docker to handle all **routing, security, and DNS** for you.

![image](https://github.com/user-attachments/assets/cb0bbf12-ab2b-4316-b109-8992e851d2c1)

---
## 🔷 5. MacVlan Network Mode

### 🧠 Concept:

Docker gives the container its **own MAC address** and IP address. It looks like a **completely separate physical device** on the network.

### 📡 Behavior:

* Acts like a real machine connected to the physical switch.
* Appears with a unique MAC and IP.
* No NAT — it talks directly to the network.
* Requires your host's network interface to be in **promiscuous mode**.

### 📦 Real-World Example:

You're running an old license server that only grants access to **specific MAC addresses** — like one of those old-school floating license systems (e.g., MATLAB license servers).

You want your Docker container to appear **exactly** like a separate physical machine with its own MAC, so it passes the licensing check.

### ✅ How to use it:

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  macvlan_net

docker run --rm --network macvlan_net my-container
```

### 🔐 Use Case Summary:

* Legacy networks with MAC restrictions
* Security appliances
* Network scanning or sniffing
* Containers that must look like real devices

---

## 🔶 6. IPVlan Network Mode

### 🧠 Concept:

Docker gives the container a **unique IP address**, but **shares the host’s MAC address**. It behaves like a separate device **only at the IP level**, not MAC level.

### 📡 Behavior:

* All containers use the **same MAC** (host's MAC).
* Each gets a **separate IP**.
* Doesn’t require promiscuous mode — more data center-friendly.

### 📦 Real-World Example:

You're in a **corporate data center** where the switch port **limits the number of MAC addresses** per interface (common in virtualized environments).

Using MacVlan would get blocked. But with IPVlan, you can run multiple containers with different IPs using **only the host’s MAC**, staying within hardware rules.

### ✅ How to use it:

```bash
docker network create -d ipvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=eth0 \
  ipvlan_net

docker run --rm --network ipvlan_net my-container
```

### 🔐 Use Case Summary:

* Data centers or networks that restrict MAC addresses
* High-performance environments without NAT
* Containers needing direct IP-level access, but same physical identity

![image](https://github.com/user-attachments/assets/a36366e8-8e09-4482-accc-fc87444afdfc)

---

## 🧾 Quick Comparison

| Feature            | MacVlan                       | IPVlan                  |
| ------------------ | ----------------------------- | ----------------------- |
| MAC Address        | Unique per container          | Shared with host        |
| Promiscuous Mode   | Required                      | Not required            |
| Visible on LAN     | Yes                           | Yes                     |
| Ideal Use Case     | Legacy MAC-sensitive networks | MAC-restricted networks |
| Layer of Operation | Layer 2 (MAC)                 | Layer 3 (IP)            |
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

## References:
https://spacelift.io/blog/docker-networking 
