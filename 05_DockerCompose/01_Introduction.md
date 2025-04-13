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

---

## ✅ Final Thoughts

Docker Compose gives you:
- Simplicity ✅
- Reusability ✅
- Portability ✅
- Collaboration ✅
