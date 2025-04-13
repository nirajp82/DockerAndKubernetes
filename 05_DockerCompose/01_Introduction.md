## 📘 What is Docker Compose?

**Docker Compose** is a tool for defining and running **multi-container Docker applications**.  
It uses a YAML file to configure the application’s services, networks, and volumes.

Instead of running each container manually using `docker run`, you define everything in one place and spin it up with a single command.

---

## 🔧 Why Use Docker Compose?

Imagine an app that consists of:

- A **frontend** (Python Flask) - **vote** (Python web app) — users vote cats vs. dogs
- A **worker** (.NET) - **worker** (.NET service) — reads from Redis, updates Postgres
- A **PostgreSQL** database - **db** (Postgres) — stores final vote counts
- A **Redis** cache - **redis** (In-memory store) — stores votes temporarily
- A **results app** (Node.js) - **result** (Node.js app) — displays results

Using just Dockerfiles and `docker run`, you'd have to:
- Manually start each container
- Set up port mappings and links
- Worry about startup order
- Share scripts with teammates

😩 It’s hard to manage and replicate!

---

## 🗳️ Example: The Voting App Architecture

```
+---------------+     +---------+       +------------+
| Voting Web UI | --> |  Redis  | <---> |   Worker   |
|   (Python)    |     +---------+       |  (.NET)    |
+---------------+                         |
                                          v
                                +----------------+
                                |  PostgreSQL DB  |
                                +----------------+
                                          |
                                          v
                                +-----------------+
                                | Results Web App |
                                |    (Node.js)    |
                                +-----------------+
```

---

## 🚫 Problems Using Only `docker run`

```bash
docker run -d --name redis redis
docker run -d --name db postgres
docker run -d --name vote -p 5000:80 --link redis:redis voting-app
docker run -d --name result -p 5001:80 --link db:db result-app
docker run -d --name worker --link redis:redis --link db:db worker-app
```

Issues:
- ❌ Manual linking
- ❌ No easy scaling
- ❌ Hard to replicate
- ❌ Startup order issues

---

## ✅ Docker Compose to the Rescue!

A `docker-compose.yml` defines everything.

```bash
docker-compose up
```

✨ And it just works!

---

## 📄 Compose File – Version 1

```yaml
vote:
  image: voting-app
  ports:
    - "5000:80"
  links:
    - redis

result:
  image: result-app
  ports:
    - "5001:80"
  links:
    - db

worker:
  image: worker-app
  links:
    - redis
    - db

redis:
  image: redis

db:
  image: postgres
```

🧱 Basic but limited.

---

## ⚡ Version 2 – Introducing Networks & Depends On

```yaml
version: '2'
services:
  vote:
    build: ./vote
    ports:
      - "5000:80"
    depends_on:
      - redis
    networks:
      - frontend
      - backend

  result:
    build: ./result
    ports:
      - "5001:80"
    depends_on:
      - db
    networks:
      - frontend
      - backend

  worker:
    build: ./worker
    depends_on:
      - redis
      - db
    networks:
      - backend

  redis:
    image: redis
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

🔗 Cleaner service discovery using DNS.  
🚦 Control service startup order.  
🌐 Isolated networks.

---

## 🚀 Version 3 – Swarm Mode Ready

```yaml
version: '3'
services:
  vote:
    build: ./vote
    ports:
      - "5000:80"
    depends_on:
      - redis
    networks:
      - frontend
      - backend

  result:
    build: ./result
    ports:
      - "5001:80"
    depends_on:
      - db
    networks:
      - frontend
      - backend

  worker:
    build: ./worker
    depends_on:
      - redis
      - db
    networks:
      - backend

  redis:
    image: redis
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

✨ Supports **Docker Swarm** clustering with:

```bash
docker stack deploy -c docker-compose.yml voting-app
```

---

## 🔗 Linking vs Networking

### Linking (Deprecated)

```yaml
links:
  - redis
```

🚫 Hard-coded and deprecated.

---

### Networking (Recommended)

```yaml
networks:
  - backend
```

✅ Built-in DNS, flexible, scalable.

---

## 🧠 Summary

| Feature         | Docker Run                      | Docker Compose            |
|-----------------|----------------------------------|----------------------------|
| Startup         | Manual                          | `docker-compose up`       |
| Linking         | `--link` (deprecated)           | Networks + DNS            |
| Startup Order   | You manage                      | `depends_on`              |
| Multiple Apps   | Shell scripts                   | YAML config               |
| Scaling         | Manual                          | Easy                      |
| Port Mapping    | Manually                        | Configurable in YAML      |

# 📚 Docker Compose Versions – Explained

## ⚙️ Version 1 – The Beginning (Pre-1.6)

### ✅ Features:
- Simple YAML structure (no top-level `version` key)
- Defines services and their basic attributes (like `image`, `ports`, `links`)

### ❌ Limitations:
| Limitation | Description |
|------------|-------------|
| ❌ No built-in networks | Containers use Docker’s default bridge network, which is shared and flat. |
| ❌ `links` are required | Manual linking with `links`, which is now deprecated. |
| ❌ No `depends_on` | You had to manage container startup order manually. |
| ❌ No top-level structure | You can’t define networks or volumes globally. |
| ❌ Not swarm-ready | Can’t be used in Docker Swarm Mode. |

### 🚨 Result:
- Good for simple development environments
- Becomes fragile and unscalable for more than 2–3 containers

---

## ⚡ Version 2 – Real Networks & Orchestration (Docker Compose 1.6+)

### ✅ What it Introduced:
| Feature         | Description |
|-----------------|-------------|
| 🔗 **Networks** | Define and isolate traffic between services. Each service can belong to one or more user-defined networks. |
| 🚦 **depends_on** | Control basic startup order. Useful in development environments. |
| 🏗️ **Top-level structure** | Now includes `version`, `services`, `networks`, and `volumes`. |
| 🛠️ **Improved syntax** | More consistent and YAML-compliant than v1. |
| 💬 **DNS-based service discovery** | Built-in DNS resolves service names to container IPs, no need for `--link`. |

### 🛠 Problem Solved:
- ❌ Manual `--link` is replaced with ✔️ built-in networks  
- ❌ Fragile startup order is improved with ✔️ `depends_on`  
- ❌ Lack of modularity is improved with ✔️ global `networks` and `volumes`

### ⚠️ Caveat:
- **Still not compatible with Docker Swarm.**  
This version works great for **local development and testing**, but not for distributed, clustered deployment.

---

## 🚀 Version 3 – Production & Swarm Mode (Compose 1.13+)

### ✅ What it Added:
| Feature | Description |
|---------|-------------|
| 📦 **Docker Swarm support** | Use the same Compose file with `docker stack deploy` to deploy services across clusters. |
| 🔄 **Replicas, update policies** | Add Swarm-specific configs like scaling, rolling updates, and placement constraints. |
| 📁 **Configs and secrets** | Securely store credentials and sensitive data in a Swarm-compatible way. |
| 🔧 **Deploy block** | Define deployment rules such as restart policies, limits, replicas, etc. |

### 🛠 Problem Solved:
- ❌ No production deployment support in v2  
- ❌ No native scaling or rolling updates  
- ❌ No secrets/configs handling  
→ ✔️ Solved with Swarm Mode + `docker stack deploy`

### ⚠️ Limitations:
| Limitation | Description |
|------------|-------------|
| ⚠️ `depends_on` doesn’t wait for readiness | You must handle health checks yourself in Swarm. |
| ⚠️ More restrictive | Some features from v2 like `build` context behave differently or are ignored in Swarm. |
| ⚠️ No support for conditionals | No logic or conditional loading in YAML. |

---

## 🧠 Summary Comparison

| Feature              | Version 1               | Version 2                   | Version 3                   |
|----------------------|--------------------------|------------------------------|------------------------------|
| Format Style         | Flat YAML               | Top-level structure          | Top-level structure          |
| Networking           | Default bridge + `links`| User-defined networks        | User-defined networks        |
| Service Discovery    | `links` (deprecated)     | DNS via service name         | DNS via service name         |
| depends_on           | ❌                       | ✅ Startup order only         | ⚠️ Startup only, not readiness |
| Volumes              | Inline only             | Top-level volumes            | Top-level volumes            |
| Swarm Support        | ❌                       | ❌                            | ✅ Yes (`docker stack deploy`) |
| Secrets / Configs    | ❌                       | ❌                            | ✅ Swarm-only                 |
| Scaling / Placement  | ❌                       | ❌                            | ✅ In `deploy:` block         |
| Production Use       | ❌                       | ⚠️ Local dev/testing only     | ✅ Cluster-ready              |

---

## ✅ TL;DR — Which One Should You Use?

| Use Case                      | Recommended Version |
|------------------------------|---------------------|
| Quick local setup            | Version 2           |
| Learning Docker networking   | Version 2           |
| Building microservices apps  | Version 2 or 3      |
| Production deployments       | Version 3 (Swarm)   |
| Need Swarm/cluster features  | Version 3           |

---

## 🎓 Key Takeaways for Beginners:

- **Start with v2** to learn networks and service relationships.
- **Upgrade to v3** when you're ready for **Swarm, scaling, and production**.
- Remember:  
  - `links:` is outdated  
  - `depends_on:` is not bulletproof  
  - Custom `networks:` are your **best friend**

---

# 🐳 **Docker Compose — From Novice to Pro (with Networking Deep Dive)**

---

## 📘 What is Docker Compose?

**Docker Compose** is a tool for defining and running **multi-container Docker applications**.  
It uses a YAML file to configure the application’s services, networks, and volumes.

Instead of running each container manually using `docker run`, you define everything in one place and spin it up with a single command.

---

## 🔧 Why Use Docker Compose?

Imagine an app that consists of:

- A **frontend** (Python Flask) - **vote** (Python web app) — users vote cats vs. dogs  
- A **worker** (.NET) - **worker** (.NET service) — reads from Redis, updates Postgres  
- A **PostgreSQL** database - **db** (Postgres) — stores final vote counts  
- A **Redis** cache - **redis** (In-memory store) — stores votes temporarily  
- A **results app** (Node.js) - **result** (Node.js app) — displays results

Using just Dockerfiles and `docker run`, you'd have to:
- Manually start each container
- Set up port mappings and links
- Worry about startup order
- Share scripts with teammates

😩 It’s hard to manage and replicate!

---

## 🗳️ Example: The Voting App Architecture

```
+---------------+     +---------+       +------------+
| Voting Web UI | --> |  Redis  | <---> |   Worker   |
|   (Python)    |     +---------+       |  (.NET)    |
+---------------+                         |
                                          v
                                +----------------+
                                |  PostgreSQL DB  |
                                +----------------+
                                          |
                                          v
                                +-----------------+
                                | Results Web App |
                                |    (Node.js)    |
                                +-----------------+
```

---

## 🚫 Problems Using Only `docker run`

```bash
docker run -d --name redis redis
docker run -d --name db postgres
docker run -d --name vote -p 5000:80 --link redis:redis voting-app
docker run -d --name result -p 5001:80 --link db:db result-app
docker run -d --name worker --link redis:redis --link db:db worker-app
```

Issues:
- ❌ Manual linking  
- ❌ No easy scaling  
- ❌ Hard to replicate  
- ❌ Startup order issues  

---

## ✅ Docker Compose to the Rescue!

A `docker-compose.yml` defines everything.

```bash
docker-compose up
```

✨ And it just works!

---

## 📄 Compose File – Version 1

```yaml
vote:
  image: voting-app
  ports:
    - "5000:80"
  links:
    - redis

result:
  image: result-app
  ports:
    - "5001:80"
  links:
    - db

worker:
  image: worker-app
  links:
    - redis
    - db

redis:
  image: redis

db:
  image: postgres
```

🧱 Basic but limited.

---

## ⚡ Version 2 – Introducing Networks & Depends On

```yaml
version: '2'
services:
  vote:
    build: ./vote
    ports:
      - "5000:80"
    depends_on:
      - redis
    networks:
      - frontend
      - backend

  result:
    build: ./result
    ports:
      - "5001:80"
    depends_on:
      - db
    networks:
      - frontend
      - backend

  worker:
    build: ./worker
    depends_on:
      - redis
      - db
    networks:
      - backend

  redis:
    image: redis
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

🔗 Cleaner service discovery using DNS  
🚦 Control service startup order  
🌐 Isolated networks

---

## 🚀 Version 3 – Swarm Mode Ready

```yaml
version: '3'
services:
  vote:
    build: ./vote
    ports:
      - "5000:80"
    depends_on:
      - redis
    networks:
      - frontend
      - backend

  result:
    build: ./result
    ports:
      - "5001:80"
    depends_on:
      - db
    networks:
      - frontend
      - backend

  worker:
    build: ./worker
    depends_on:
      - redis
      - db
    networks:
      - backend

  redis:
    image: redis
    networks:
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

✨ Supports **Docker Swarm** clustering with:

```bash
docker stack deploy -c docker-compose.yml voting-app
```

---

## 🔗 Linking vs Networking

### Linking (Deprecated)

```yaml
links:
  - redis
```

🚫 Hard-coded and deprecated.

---

### Networking (Recommended)

```yaml
networks:
  - backend
```

✅ Built-in DNS, flexible, scalable.

---

## 🌐 Deep Dive: Docker Compose Networking (Beginner to Pro)

### 🧠 What is a Docker Network?

A **Docker network** is like a virtual LAN. Containers on the same network can:
- Discover each other using **service names** (no IPs)
- Communicate over a private network
- Be isolated from the outside world or other services

---

### 🧱 Types of Networks

| Type         | Use Case                            |
|--------------|--------------------------------------|
| `bridge`     | Default. Local host-only container comms. |
| `host`       | Shares host network (less secure).   |
| `none`       | Disables networking.                 |
| `overlay`    | Swarm multi-host clusters.           |
| `macvlan`    | Maps containers to real network interfaces. |

---

### 💬 DNS-based Service Discovery

Every service gets registered in a DNS:
- `redis` → resolves to Redis container’s IP
- `db` → resolves to Postgres IP
- No need for `--link`, just use `redis:6379`

---

### 🔄 Custom Networks in Compose

```yaml
networks:
  frontend:
  backend:
```

- `frontend`: Used by UI services (`vote`, `result`)
- `backend`: Used by internal services (`worker`, `db`, `redis`)

🔒 Services on separate networks **can’t talk to each other unless explicitly added**.

---

### 🧪 Real-World Example

```yaml
services:
  vote:
    networks: [frontend, backend]
  redis:
    networks: [backend]
```

`vote` ↔ `redis`: ✅ (shared backend)  
`vote` ↔ `result`: ❌ (not on same network)

---

### 📦 Inspect Docker Networks

List networks:

```bash
docker network ls
```

Inspect details:

```bash
docker network inspect <network_name>
```

See connected containers, IPs, aliases, etc.

---

### 🧠 Advanced: Aliases

```yaml
db:
  networks:
    backend:
      aliases:
        - postgres-db
        - primary-db
```

Now services can connect using any alias above. Great for legacy apps or migrations.

---

### 🛡️ Pro Tips: Network Isolation

✔️ Use **separate networks** for:
- `frontend` (web traffic)
- `backend` (sensitive services)
- `monitoring` (metrics)

✔️ Prevent unintended access  
✔️ Follows microservice security best practices

---

### ⏳ Caveat: depends_on ≠ Wait for Ready

```yaml
depends_on:
  - db
```

⚠️ This only ensures `db` **starts**, not that it’s **ready**. Use:
- `restart: on-failure`
- Health checks
- `wait-for-it.sh` or `dockerize` scripts

---

## 🧠 Summary

| Feature         | Docker Run                      | Docker Compose            |
|-----------------|----------------------------------|----------------------------|
| Startup         | Manual                          | `docker-compose up`       |
| Linking         | `--link` (deprecated)           | Networks + DNS            |
| Startup Order   | You manage                      | `depends_on`              |
| Multiple Apps   | Shell scripts                   | YAML config               |
| Scaling         | Manual                          | Easy                      |
| Port Mapping    | Manually                        | Configurable in YAML      |
| Networking      | Manual + fragile                | Built-in, scalable        |
| Discovery       | Container IPs or links          | Service names via DNS     |

