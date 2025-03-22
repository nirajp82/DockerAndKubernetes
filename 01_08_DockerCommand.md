# Docker Commands: Your Ultimate Cheat Sheet 🚀

## **Running Containers** 🏃‍♂️
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

## **Cleanup & Maintenance** 🧹
### **`docker system prune`** - Clean Up Unused Resources
```bash
docker system prune
```

## **Authentication & Registry** 🔑
### **`docker login` / `docker logout`**
```bash
docker login -u username -p password
docker logout
```

## **Final Thoughts** 🏁
Master these commands to become a Docker expert! Check out the official [Docker Docs](https://docs.docker.com/) for more insights. 🚀

