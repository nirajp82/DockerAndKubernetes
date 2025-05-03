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

```bash
# Run a container with automatic restarts for a Docker registry service
docker run -d -p 5000:5000 --restart=always --name my-registry registry:2
```

- `-d` → Runs the container in **detached mode** (in the background)
- `-p 5000:5000` → Maps **port 5000** on your local machine to **port 5000** inside the container
- `--restart=always` → Ensures the container **always restarts** if it stops, crashes, or the system reboots
- `--name my-registry` → Gives the container the name **`my-registry`** for easy reference
- `registry:2` → Uses the **Docker registry image**, version 2

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

| **Feature** | **`docker exec`** | **`docker run`** |
|-------------|-------------------|------------------|
| Runs inside a running container? | ✅ Yes | ❌ No (creates a new container) |
| Requires an existing container? | ✅ Yes | ❌ No |
| Useful for debugging? | ✅ Yes | ❌ No |
| Runs additional commands in an existing environment? | ✅ Yes | ❌ No |
| Common use case | Debugging, inspecting, or running commands inside a running container | Starting a fresh interactive session from an image |
| Key difference | Attaches to an **already running** container | **Creates and starts** a new container instance |
| Example | `docker exec -it my-container bash` | `docker run -it ubuntu bash` |
| Summary | Used to run commands **inside an existing** running container | Used to **create and run** a brand new container from an image |

 
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

## 🔍 Inspecting Containers with `docker inspect`

The `docker inspect` command gives detailed, low-level information about Docker containers (both running and stopped) in JSON format.

### 🧰 Syntax
```bash
docker inspect <container-id or container-name>
```

---

### 🧠 What It Shows

| Section              | Description                                                                 |
|----------------------|-----------------------------------------------------------------------------|
| `Id`                 | Unique ID of the container                                                  |
| `Image`              | ID of the image the container is based on                                   |
| `Name`               | Container name                                                              |
| `State.Status`       | Current state: `running`, `exited`, or `created`                            |
| `State.StartedAt`    | When the container started                                                  |
| `State.FinishedAt`   | When the container stopped (for exited containers)                          |
| `Mounts`             | Volume or bind mounts in the container                                      |
| `Config.Env`         | Environment variables passed to the container                               |
| `NetworkSettings.IPAddress` | IP address assigned to the container (for bridge network mode)        |
| `HostConfig.PortBindings`   | Port mappings between host and container                              |

---

### ✅ Examples

#### **Inspect a running container**
```bash
docker inspect my-running-container
```
You’ll see:
- `State.Status`: `running`
- IP address and network settings
- Active mounts and environment variables

#### **Inspect a stopped container**
```bash
docker inspect my-stopped-container
```
You’ll see:
- `State.Status`: `exited`
- `FinishedAt`: timestamp of when it stopped
- Other configs (image, env, volumes, etc.) still visible

---

### 🎯 Filter Specific Info (with `jq` or `--format`)

#### Show only container status:
```bash
docker inspect --format='{{.State.Status}}' my-container
```

#### Show only the container IP address:
```bash
docker inspect --format='{{.NetworkSettings.IPAddress}}' my-container
```

---

### 🧪 Bonus: Use `jq` for Pretty Output
```bash
docker inspect my-container | jq '.[0].State'
```


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
### `docker history` 

The `docker history` command shows the history of an image, displaying information about each layer of the image, such as the commands that were executed to create it, its size, and timestamps. This is useful for understanding the build process and the content of a Docker image.

Here’s how the `docker history` command is explained:

```bash
# Show the history of a specific Docker image
docker history <image-name>
```

- `<image-name>` → Specifies the image whose history you want to view (e.g., `ubuntu`, `nginx:latest`, or a custom image like `my-app:v1`).

```bash
docker history ubuntu:latest
```

Sure! Here's an explanation of the `docker system df` command in the same style:

---

### `docker system df`
The `docker system df` command shows how much disk space Docker is using and what is consuming that space. It gives you a breakdown of disk usage for images, containers, volumes, and build cache. This is helpful for identifying large or unused items you might want to clean up to free space.

Here’s how the `docker system df` command is explained:

```bash
# Display a summary of Docker disk usage
docker system df
```

* **Images** → Shows how many images are on your system, how many are in use, how many are dangling (unused), and how much total space they consume.
* **Containers** → Lists the number of containers (running, stopped, or exited) and their associated disk usage.
* **Local Volumes** → Displays the number of volumes and how much space they occupy.
* **Build Cache** → Shows how much disk space is used by the build cache, which can accumulate during Docker image builds.

**Commonly Used Flags:**

These are optional parameters that modify the behavior of Docker commands. For example:

* `-v` or `--verbose` → Enables verbose output, providing more detailed information about what the command is doing.
* `-a` or `--all` → Includes all items (e.g., stopped containers or unused images), not just the active ones.
* `-f` or `--filter` → Filters output based on specific conditions (e.g., by name, status, or label).
* `--format` → Lets you customize the output format using Go templates, often used for scripting or cleaner displays.

These flags can be combined with many Docker commands to refine the output or change how the command operates.


