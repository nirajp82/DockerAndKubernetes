# Understanding Docker Commands: A Comprehensive Guide
* `docker run`: The docker run command is used to create and start a container from a specified Docker image. The general syntax is:

```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```

### **Breaking it Down**
1. **`docker run`** → This is the command itself.
2. **`[OPTIONS]`** → These are optional flags to modify the container’s behavior (e.g., `-d` for detached mode, `-p` to publish ports, etc.).
3. **`IMAGE`** → This is the name of the Docker image you want to run (e.g., `nginx`, `ubuntu:latest`).
4. **`[COMMAND]` `[ARG...]`** → Optional command and arguments that override the image’s default behavior.

---

### **Common Options**
| Option | Description |
|---------|-------------|
| `-d` | Run the container in detached mode (background) |
| `-it` | Run interactively with a TTY (useful for debugging) |
| `--name mycontainer` | Assigns a name (`mycontainer`) to the container |
| `-p 8080:80` | Maps port 8080 on the host to port 80 in the container |
| `-v /host/path:/container/path` | Mounts a volume (bind mount) from the host to the container |
| `--rm` | Removes the container after it stops |
| `-e ENV_VAR=value` | Sets environment variables inside the container |
| `--network network_name` | Connects the container to a specific network |
| `--restart always` | Ensures the container restarts if it stops unexpectedly |

---

### **Examples**
- **Run an interactive Ubuntu container**
   ```bash
   docker run -it ubuntu /bin/bash
   ```
   - `-it` → Interactive mode with a terminal
   - `ubuntu` → Uses the Ubuntu image
   - `/bin/bash` → Runs the Bash shell inside the container

- **Run Nginx in detached mode with port mapping**
   ```bash
   docker run -d -p 8080:80 nginx
   ```
   - `-d` → Detached mode (runs in the background)
   - `-p 8080:80` → Maps host port 8080 to container port 80
   - `nginx` → Uses the Nginx image

- **Run a container and remove it after it stops**
   ```bash
   docker run --rm ubuntu echo "Hello, Docker!"
   ```
   - `--rm` → Removes the container after execution
   - `ubuntu` → Uses the Ubuntu image
   - `echo "Hello, Docker!"` → Runs a simple command inside the container

- **Run a container with a mounted volume**
   ```bash
   docker run -v /data:/app/data ubuntu ls /app/data
   ```
   - `-v /data:/app/data` → Mounts `/data` from the host to `/app/data` in the container
   - `ls /app/data` → Lists the contents of `/app/data` inside the container
 
### **`docker pull` Command**
The `docker pull` command is used to download (pull) a Docker image from a remote registry, typically [Docker Hub](https://hub.docker.com/), but it can also be from private registries.

### **Syntax**
```bash
docker pull [OPTIONS] IMAGE[:TAG|@DIGEST]
```

### **Breaking it Down**
1. **`docker pull`** → The command to fetch an image.
2. **`[OPTIONS]`** → Optional flags to modify the behavior (e.g., `--quiet` for silent mode).
3. **`IMAGE`** → The name of the image you want to download (e.g., `nginx`).
4. **`[:TAG]` or `[@DIGEST]`** → Optional version identifiers:
   - `:TAG` → Specifies the image version (e.g., `ubuntu:22.04`).
   - `@DIGEST` → A unique identifier for an exact image version.

---

### **Examples**
#### **1. Pull the latest version of an image**
```bash
docker pull ubuntu
```
- By default, Docker pulls the `latest` tag if no version is specified.
- Equivalent to:
  ```bash
  docker pull ubuntu:latest
  ```

#### **2. Pull a specific version of an image**
```bash
docker pull nginx:1.21
```
- Downloads version `1.21` of the `nginx` image.

#### **3. Pull an image using a digest**
```bash
docker pull ubuntu@sha256:xyz123...
```
- Ensures you get an exact version of the image, even if the `latest` tag changes.

#### **4. Pull an image from a private registry**
```bash
docker pull myregistry.com/myimage:latest
```
- Downloads an image from a custom registry (`myregistry.com`).

#### **5. Pull an image without progress output**
```bash
docker pull --quiet redis
```
- `--quiet` suppresses progress output.

---

### **Check Pulled Images**
After pulling an image, you can verify it with:
```bash
docker images
```

* `docker ps`: 
List All Running Containers. The `docker ps` command provides a snapshot of all your currently running containers. It offers a quick overview of the containers' names, IDs, statuses, and resource usage.

* `docker ps -a`: 
List All Containers (Stopped and Running). Adding the `-a` flag expands the scope to include all containers, both running and stopped. This command gives you a comprehensive view of your container ecosystem.

* `docker ps -a -q`: 
List Only Container IDs. For scripting and automation purposes, you can use `docker ps -a -q` to extract only the IDs of all containers. This minimalist output is handy when you need to pass container IDs as parameters to other commands.

* `docker images [OPTIONS] [REPOSITORY[:TAG]]`: 
Lists all Docker images stored locally on your system. The `docker ps -a` command, on the other hand, is used to list all Docker containers, regardless of their current state (Including both running and stopped containers).

* `docker stop container-id`: 
Stop a Running Container. To gracefully stop a running container, use `docker stop` followed by the container's ID. Docker sends a SIGTERM signal to the main process inside the container, allowing the process to shut down gracefully. The container is given a specific amount of time (default is 10 seconds) to stop and perform any necessary cleanup operations before it is forcefully terminated. Please note, if a container does not respond to the SIGTERM signal within the specified timeout period, Docker will forcefully terminate the container using a SIGKILL signal, which can result in data corruption or loss if the application is in the middle of critical operations.

* `docker kill container-id`: 
The `docker kill` command is used to forcefully stop a running container in a Docker environment. Unlike the `docker stop` command, which sends a SIGTERM signal to the container, allowing it to shut down gracefully, `docker kill` immediately terminates the container by sending a SIGKILL signal. This abrupt termination does not give the container any opportunity to clean up or execute shutdown procedures, making it a forceful and instantaneous way to stop a container.

* `docker start container-id`: Start a Stopped Container
When a container is in a stopped state, use `docker start` along with the container's ID to initiate it. This command reactivates the container, making it operational again.

* `docker restart container-id`: Restart a Running Container
To restart an already running container, employ `docker restart` followed by the container's ID. Docker will stop the container, then immediately start it again, ensuring a fresh state.

* `docker port container-id`: List Port Mappings of a Container
With `docker port`, you can inspect the port mappings of a specific container. It reveals the internal ports mapped to the host system, facilitating network configuration.
### **`docker exec` Command**
The `docker exec` command is used to run a command inside an already running container. This is useful for debugging, interacting with applications inside containers, or running additional processes.

---

### **Syntax**
```bash
docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
```

### **Breaking it Down**
1. **`docker exec`** → The command to execute a process inside a running container.
2. **`[OPTIONS]`** → Flags that modify behavior (e.g., `-it` for interactive mode).
3. **`CONTAINER`** → The name or ID of the running container.
4. **`COMMAND [ARG...]`** → The command and its arguments to run inside the container.

---

### **Common Options**
| Option | Description |
|---------|-------------|
| `-d` | Runs the command in the background (detached mode) |
| `-i` | Keeps STDIN open (useful for interactive commands) |
| `-t` | Allocates a pseudo-TTY (terminal) |
| `-u username` | Runs the command as a specific user inside the container |
| `--privileged` | Grants extended privileges inside the container |

---

### **Examples**
#### **1. Open an interactive Bash shell in a running container**
```bash
docker exec -it mycontainer /bin/bash
```
- `-it` → Interactive mode with a terminal.
- `mycontainer` → The name (or ID) of the running container.
- `/bin/bash` → Starts a Bash shell inside the container.

#### **2. Run a simple command inside a container**
```bash
docker exec mycontainer ls /app
```
- Lists the contents of the `/app` directory inside `mycontainer`.

#### **3. Run a command as a specific user**
```bash
docker exec -u root mycontainer whoami
```
- Runs `whoami` as the `root` user inside the container.

#### **4. Execute a command in a detached mode**
```bash
docker exec -d mycontainer touch /tmp/newfile
```
- Runs `touch /tmp/newfile` inside `mycontainer` without waiting for output.

---

### **Difference Between `docker exec` and `docker run`**
| Feature | `docker exec` | `docker run` |
|---------|--------------|--------------|
| Runs inside a running container? | ✅ Yes | ❌ No (creates a new container) |
| Requires an existing container? | ✅ Yes | ❌ No |
| Useful for debugging? | ✅ Yes | ❌ No |
| Runs additional commands in an existing environment? | ✅ Yes | ❌ No |
 
* `docker exec -it container-id /bin/sh`  or `docker container exec -it f42d redis-cli`
Run a command in a running container. Interactive terminal Connect to linux container and execute commands in container.
For ex: If we want to get access to the redis CLI for the running redis container we can achieve this using the exec command.
![image](https://github.com/nirajp82/DockerAndKubernetes/assets/61636643/2dd18b7f-baf2-4ab8-a2b8-d32b8eb21d49)
Learn more at https://github.com/nirajp82/DockerAndKubernetes/blob/main/03_02_Linux_Std_Comm_Channel.md

* `docker rm container-id` or `name`: Remove a Stopped Container
To clean up your system, use `docker rm` followed by the container's ID or name. This command removes a stopped container, freeing up resources.

* `docker rm -f container-id` or `name`: Remove a Running Container Forcefully
Forcibly remove a running container by adding the `-f` flag to `docker rm`. Be cautious when using this command, as it terminates the container immediately, potentially leading to data loss.

* `docker rmi image-id`
The `docker rmi` command removes one or more specified images from your local system. Replace `image-id` with the ID of the image you want to remove. This is useful for cleaning up unused or unwanted images to free up disk space. We must stop and delete all dependent containers to be able to delete an image.

* `docker pull image-info`: Pull an Image from Docker Hub
To fetch an image from the Docker Hub repository, employ `docker pull` followed by the image's name. This command downloads the specified image, making it available for creating containers.
  - `docker pull nirajp82/my_first_docker_image`: Pull a Specific Image from Docker Hub. If you want a particular image, specify the repository and image name in the `docker pull` command. This allows you to download customized or specific images for your projects.

* `docker logout`
`docker logout` command logs out the Docker registry associated with the Docker CLI. It is essential for security and managing different Docker registries. After logging out, you need to log in again to perform any registry-related operations.

* `docker login -u username -p password`
Use `docker login` to authenticate with a Docker registry. Replace `username` with your Docker Hub username and `password` with your Docker Hub password. This command allows you to pull and push images to your Docker Hub account.

* `docker stats`
The `docker stats` command provides a live stream of container resource usage statistics. It displays CPU, memory, network, and disk I/O usage for all running containers. This is helpful for monitoring the performance of your containers.

* `docker top container-id or name`
With `docker top`, you can view the running processes inside a specific container. Provide the `container-id` or `name` as an argument, and Docker will show you the active processes within that container.

* `docker version`
`docker version` provides detailed information about your Docker installation, including the Docker client and server version, build date, and other relevant details. It helps you verify the version you are running and check for updates.

* `docker container run -d image_name`
The `docker container run` command creates and runs a new container based on the specified `image_name`. The `-d` flag runs the container in detached mode, allowing it to run in the background. This is commonly used for running applications as services.

* `docker system prune`
`docker system prune` removes all stopped containers, unused networks, dangling images, and build cache. It helps clean up your system by removing unused resources, freeing up disk space, and improving system performance.

* `docker logs <container id> --follow`
Adding the `--follow` flag to `docker logs` allows you to continuously stream the logs of a container, displaying new log messages as they are generated. This is useful for real-time monitoring of container activities.

* `docker logs <container id> --since`
The `--since` flag in `docker logs` allows you to filter log output to show only the logs generated since a specific timestamp. Specify the timestamp, and Docker will display logs generated after that time.

* `docker logs <container id> --tail`
`--tail` flag limits the number of lines displayed in the log output. For example, `docker logs <container id> --tail=50` shows only the last 50 lines of logs for the specified container.

* `docker logs <container id> --timestamps`
Adding `--timestamps` to `docker logs` displays timestamps along with the log entries. This helps you understand when each log entry was generated, providing context to the log output.

* `docker logs <container id> --details`
`docker logs` command shows the logs generated by a container. Use the `--details` flag to include extra details such as timestamps and container IDs in the log output.

* `docker logs <container id> --until`
The `--until` flag in `docker logs` filters log output to show only the logs generated before a specific timestamp. Specify the timestamp, and Docker will display logs generated before that time.

These Docker commands provide essential functionalities for managing containers, monitoring resources, and analyzing container logs effectively. Understanding their usage is crucial for efficient Docker container orchestration.

*For more detailed information, check out the official [Docker Documentation](https://docs.docker.com/).*


# docker exec vs docker run
Both `docker exec -it <container-id> sh` and `docker run -it <container-id> sh` commands can be used to access an interactive shell inside a running Docker container, but they serve different purposes and have distinct behaviors:

### 1. `docker exec -it <container-id> sh`:

- **Usage:** `docker exec -it <container-id> sh` is used to run a new command (`sh` in this case) inside an already running container. It does not start a new container; instead, it attaches an interactive shell to the existing container specified by `<container-id>`.
  
- **Purpose:**
  - **Interacting with Running Containers:** This command is useful for accessing the shell of a container that is already running. You can use it to debug, inspect, or modify the container's state without disrupting its execution.
  
- **Example:**
  ```bash
  docker exec -it <container-id> sh
  ```

### 2. `docker run -it <container-id> sh`:

- **Usage:** `docker run -it <container-id> sh` creates a new container instance using the specified `<container-id>` and starts an interactive shell (`sh`) within the new container. This command starts a fresh container and provides an interactive shell prompt for immediate use.

- **Purpose:**
  - **Starting a New Interactive Container:** This command is useful when you want to spin up a new container and interact with it immediately. It's handy for scenarios where you need to experiment, test, or troubleshoot within an isolated environment.

- **Example:**
  ```bash
  docker run -it <container-id> sh
  ```

### Key Differences:

1. **Existing Container vs. New Container:**
   - `docker exec -it <container-id> sh` interacts with an **existing running container**.
   - `docker run -it <container-id> sh` creates a **new container** based on the specified image, then interacts with it.

2. **Use Cases:**
   - `docker exec -it <container-id> sh` is suitable for working with containers that are already running and need immediate interaction.
   - `docker run -it <container-id> sh` is useful when you want to spin up a fresh container and start interacting with it right away, especially for testing or debugging purposes.

3. **Container Lifecycle:**
   - `docker exec` does not affect the container's lifecycle; the container keeps running after the interactive session ends.
   - `docker run` creates a new container instance, which will be terminated when the interactive shell session is closed unless the container runs another long-lived process.

In summary, `docker exec -it <container-id> sh` is for interacting with existing containers, while `docker run -it <container-id> sh` is for starting a new container with an interactive shell session. Choose the appropriate command based on whether you need to work with an already running container or require a fresh, interactive container environment.
