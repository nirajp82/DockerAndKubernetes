# Docker Commands: Your Ultimate Cheat Sheet 

## **Running Containers** 
### **`docker run`** - Create and Start a Container
The `docker run` command is used to create and start a container from a specified Docker image.
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
#### **Breaking it Down**
- **`docker run`** → The command itself.
- **`[OPTIONS]`** → Modify container behavior (e.g., `-d` for detached mode, `-p` to publish ports).
- **`IMAGE`** → Name of the image to use (e.g., `nginx`, `ubuntu:latest`).
- **`[COMMAND] [ARG...]`** → Optional command to override the image’s default behavior.

#### **Common Options**
| Option | Description |
|---------|-------------|
| `-d` | Run the container in detached mode (background) |
| `-it` | Run interactively with a TTY (useful for debugging) |
| `--name mycontainer` | Assigns a name (`mycontainer`) to the container |
| `-p 8080:80` | Maps port 8080 on the host to port 80 in the container |
| `-v /host/path:/container/path` | Mounts a volume from the host to the container |
| `--rm` | Removes the container after it stops |
| `-e ENV_VAR=value` | Sets environment variables inside the container |
| `--network network_name` | Connects the container to a specific network |
| `--restart always` | Ensures the container restarts if it stops unexpectedly |

#### **Examples**
```bash
# Run an interactive Ubuntu container
docker run -it ubuntu /bin/bash
```
- `-it` → Interactive mode with a terminal
- `ubuntu` → Uses the Ubuntu image
- `/bin/bash` → Runs the Bash shell inside the container

```bash
# Run Nginx in detached mode with port mapping
docker run -d -p 8080:80 nginx
```
- `-d` → Detached mode (runs in the background)
- `-p 8080:80` → Maps host port 8080 to container port 80
- `nginx` → Uses the Nginx image

```bash
# Run a container and remove it after it stops
docker run --rm ubuntu echo "Hello, Docker!"
```
- `--rm` → Removes the container after execution
- `ubuntu` → Uses the Ubuntu image
- `echo "Hello, Docker!"` → Runs a simple command inside the container

```bash
# Run a container with a mounted volume
docker run -v /data:/app/data ubuntu ls /app/data
```
- `-v /data:/app/data` → Mounts `/data` from the host to `/app/data` in the container
- `ls /app/data` → Lists the contents of `/app/data` inside the container

## **Managing Containers** 🔄
### **`docker ps`** - List Running Containers
```bash
docker ps
```
Displays currently running containers.

### **`docker ps -a`** - List All Containers
```bash
docker ps -a
```
Shows both running and stopped containers.

### **`docker ps -a -q`** - List Only Container IDs
```bash
docker ps -a -q
```
Outputs only container IDs (useful for scripting).

### **`docker start` / `docker stop` / `docker restart`**
```bash
docker start container-id
docker stop container-id
docker restart container-id
```
- `start` → Start a stopped container
- `stop` → Gracefully stop a running container
- `restart` → Stop and restart a running container

### **`docker kill`** - Force Stop a Container
```bash
docker kill container-id
```
Immediately stops a container without a graceful shutdown.

### **`docker rm`** - Remove Containers
```bash
docker rm container-id  # Remove stopped container
docker rm -f container-id  # Force remove running container
```

## **Working Inside a Container** 🛠
### **`docker exec`** - Run Commands in a Running Container
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```
#### **Common Options**
| Option | Description |
|---------|-------------|
| `-d` | Runs the command in the background |
| `-i` | Keeps STDIN open |
| `-t` | Allocates a pseudo-TTY |
| `-u username` | Runs the command as a specific user |

#### **Examples**
```bash
docker exec -it container-id /bin/bash  # Open interactive shell
docker exec mycontainer ls /app  # List files inside container
docker exec -u root mycontainer whoami  # Run command as root
```

* `docker exec -it container-id /bin/sh`  or `docker container exec -it f42d redis-cli`
Run a command in a running container. Interactive terminal Connect to linux container and execute commands in container.
For ex: If we want to get access to the redis CLI for the running redis container we can achieve this using the exec command.

- ![image](https://github.com/nirajp82/DockerAndKubernetes/assets/61636643/2dd18b7f-baf2-4ab8-a2b8-d32b8eb21d49)

### **Difference Between `docker exec` and `docker run`**
| Feature | `docker exec` | `docker run` |
|---------|--------------|--------------|
| Runs inside a running container? | ✅ Yes | ❌ No (creates a new container) |
| Requires an existing container? | ✅ Yes | ❌ No |
| Useful for debugging? | ✅ Yes | ❌ No |
| Runs additional commands in an existing environment? | ✅ Yes | ❌ No |
| Common use case | Debugging, inspecting, or running commands inside a running container | Starting a fresh interactive session from an image |
| Summary | - Use `docker exec -it` when you need to interact with an already running container. | Use `docker run -it` when you want to create and start a new container interactively.|

 
## **Handling Images** 🎨
### **`docker pull`** - Download an Image
```bash
docker pull IMAGE[:TAG|@DIGEST]
```
#### **Examples**
```bash
docker pull ubuntu  # Latest version
docker pull nginx:1.21  # Specific version
docker pull myregistry.com/myimage:latest  # From a private registry
```

### **`docker images`** - List Available Images
```bash
docker images
```

### **`docker rmi`** - Remove an Image
```bash
docker rmi image-id
```

## **Networking & Ports** 🌐
### **`docker port`** - View Port Mappings
```bash
docker port container-id
```

## **Logs & Monitoring** 📊
### **`docker logs`** - View Logs of a Container
```bash
docker logs container-id
docker logs container-id --follow  # Stream live logs
docker logs container-id --tail=50  # Show last 50 lines
```

### **`docker stats`** - Monitor Resource Usage
```bash
docker stats
```

### **`docker top`** - View Processes Inside a Container
```bash
docker top container-id
```

## **Cleanup & Maintenance** 
### **`docker system prune`** - Clean Up Unused Resources
```bash
docker system prune
```

## **Authentication & Registry** 
### **`docker login` / `docker logout`**
```bash
docker login -u username -p password
docker logout
```

## **docker version** 
- docker version provides detailed information about your Docker installation, including the Docker client and server version, build date, and other relevant details. It helps you verify the version you are running and check for updates.

## 🚀 Understanding `docker stop` vs `docker kill`

### `docker stop` - Gracefully Stops a Running Container
The `docker stop` command allows a running container to shut down properly.
- It **sends a SIGTERM** signal to the container’s main process.
- The container gets time to **clean up resources** before stopping.
- If the process does not terminate within **10 seconds**, Docker **forcefully kills** it with **SIGKILL**.

**Example:**
```sh
docker stop container-id
```

### `docker kill` - Immediately Terminates a Container
The `docker kill` command abruptly stops a running container **without a grace period**.
- It **sends a SIGKILL** signal, terminating the process immediately.
- This might result in **data corruption or incomplete shutdown**.

**Example:**
```sh
docker kill container-id
```

### When to Use?
| Scenario                           | Use `docker stop` | Use `docker kill` |
|------------------------------------|------------------|------------------|
| Allow graceful shutdown            | ✅ Yes          | ❌ No           |
| Application is unresponsive        | ❌ No           | ✅ Yes          |
| Avoid data loss                    | ✅ Yes          | ❌ No           |

---

## 🛠️ `docker exec` vs `docker run`

| Feature                      | `docker exec`                        | `docker run` |
|------------------------------|--------------------------------------|-------------|
| Runs inside an existing container? | ✅ Yes | ❌ No (creates a new container) |
| Requires a running container? | ✅ Yes | ❌ No |
| Runs additional commands in an active environment? | ✅ Yes | ❌ No |
| Best for debugging/interacting with live containers? | ✅ Yes | ❌ No |

### `docker exec` - Run a Command in a Running Container
Used for **interacting** with an already running container.

**Example:** Open an interactive shell inside a running container:
```sh
docker exec -it container-id /bin/bash
```

### `docker run` - Start a New Container
Used to **create and run** a new container from an image.

**Example:** Start a new Ubuntu container interactively:
```sh
docker run -it ubuntu /bin/bash
```

**Key Difference:**
- `docker exec` **attaches to an existing container**.
- `docker run` **creates a new container instance**.

---

## 📥 Understanding `docker pull`

The `docker pull` command fetches images from Docker Hub or private registries.

### Syntax:
```sh
docker pull [OPTIONS] IMAGE[:TAG|@DIGEST]
```

### Examples:
- Pull the latest Ubuntu image:
  ```sh
  docker pull ubuntu
  ```
- Pull a specific version of Nginx:
  ```sh
  docker pull nginx:1.21
  ```
- Pull an image using a digest:
  ```sh
  docker pull ubuntu@sha256:xyz123...
  ```
- Pull from a private registry:
  ```sh
  docker pull myregistry.com/myimage:latest
  ```

---

## 🔄 Standard Input/Output (I/O) in Docker

Docker containers interact with input/output streams using:
- `stdin` - Standard input (used to send data to a container)
- `stdout` - Standard output (container’s normal output)
- `stderr` - Standard error (container error logs)

### Running a container while keeping `stdin` open
```sh
docker run -i ubuntu cat
```

### Running an interactive shell (`stdin`, `stdout`, `stderr` enabled)
```sh
docker run -it ubuntu /bin/bash
```

---

## 📜 Advanced Logging with `docker logs`

The `docker logs` command helps view logs from a container.

### Common Flags:
| Flag | Description |
|------|-------------|
| `--follow` | Stream logs live |
| `--since TIME` | Show logs since a timestamp (e.g., `--since 10m`) |
| `--tail=50` | Show the last 50 lines of logs |
| `--timestamps` | Show timestamps on each log entry |

### Example Commands:
- View container logs:
  ```sh
  docker logs container-id
  ```
- Continuously stream logs:
  ```sh
  docker logs --follow container-id
  ```
- Show logs from the last 30 minutes:
  ```sh
  docker logs --since 30m container-id
  ```
- Show only the last 100 lines:
  ```sh
  docker logs --tail=100 container-id
  ```

---

## 🧹 Cleaning Up Docker Resources

### `docker system prune` - Remove Unused Resources
```sh
docker system prune
```
This removes:
- **Stopped containers**
- **Dangling images**
- **Unused networks**
- **Build cache**

Add the `-a` flag to remove **all unused images**:
```sh
docker system prune -a
```

---

## 📊 Monitoring Docker Performance

### `docker stats` - View Container Resource Usage
```sh
docker stats
```
Displays live CPU, memory, network, and disk usage for all running containers.

### `docker top` - Show Running Processes in a Container
```sh
docker top container-id
```
Lists all active processes inside a running container.

---

## 🔐 Docker Authentication

### `docker login` - Authenticate with a Docker Registry
```sh
docker login -u username -p password
```
After logging in, you can push/pull private images.

### `docker logout` - Log Out from a Docker Registry
```sh
docker logout
```
This ensures your credentials are removed for security.

---


## **Final Thoughts** 🏁
Master these commands to become a Docker expert! Check out the official [Docker Docs](https://docs.docker.com/) for more insights. 

