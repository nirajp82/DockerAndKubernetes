### Containerization in Docker

Let’s explore how Docker containers work and how Docker achieves **application isolation** using **namespaces**.

#### What is a Namespace?

In Docker, **namespaces** are used to provide isolation for different components of a container’s environment. Each namespace confines a specific aspect of the container's environment, such as the processes it runs, network settings, or filesystem mounts. This isolation ensures that containers feel like they are independent systems, even though they are running on the same host system.

There are several types of namespaces in Docker, including:

- **Process ID (PID) Namespace**: Isolates process IDs within containers.
- **Network Namespace**: Provides network isolation for containers.
- **Mount Namespace**: Isolates the filesystem for containers.
- **IPC Namespace**: Isolates inter-process communication.
- **User Namespace**: Provides user and group isolation.

For this explanation, we will focus on the **Process ID (PID) Namespace** to understand how containers run their processes independently.

---

### Process ID (PID) Namespace

When a Linux system starts, it initializes the system with a single root process (PID 1), which then spawns other processes. You can see the running processes by using the `ps` command. Every process on the system has a unique **process ID (PID)**, which helps the system track and manage them.

When Docker creates a container, it essentially sets up a **child environment** that needs to think of itself as an independent system. For this, Docker uses **PID namespaces**.

Here’s how it works:

1. When you run a Docker container, it starts with its own **process namespace**. This means that, inside the container, processes believe they are running in a standalone system. The first process to run inside the container will have a PID of 1, making it the **root process** of that container.
   
2. However, these container processes are still running on the **host system**. The difference is that the **host system** and the **container** have separate process namespaces. Therefore, processes inside the container have their own set of PIDs, distinct from those on the host system.

#### Example: NGINX Container

Let’s run an **NGINX** server inside a container to see this in action.

1. First, run the following command to start an **NGINX** container:
   ```
   docker run -d --name nginx-container nginx
   ```

2. **Inside the container**: The `nginx` process will run with PID 1. To verify this, you can execute the following command to enter the container:
   ```
   docker exec -it nginx-container ps -aux
   ```

   This shows the list of running processes inside the container, with **NGINX** listed as the root process (PID 1):
   ```
   USER       PID  %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
   root         1  0.0  0.0  37836  3124 ?        Ss   10:25   0:00 nginx: master process /usr/sbin/nginx
   ```

3. **On the host system**: If you list all processes running on the host, you’ll see the **NGINX** process running, but with a different PID. You can do this with the `ps` command:
   ```
   ps aux | grep nginx
   ```

   You’ll see output like this, with a different PID on the host system:
   ```
   root      1457  0.0  0.0  37836  3124 ?        Ss   10:25   0:00 nginx: master process /usr/sbin/nginx
   ```

Even though both the container and the host are running the same NGINX service, they have separate PIDs within their respective namespaces. The container thinks that it is an independent system, running its own set of processes, with **PID 1** being the NGINX process.

---

### Resource Management and Isolation

Containers share the **host system’s resources** (like CPU and memory), but Docker provides ways to **restrict** the amount of resources each container can use. By default, containers are not restricted in terms of resource usage and may end up utilizing all available resources on the host.

Docker uses **control groups (cgroups)** to manage and limit the resources allocated to each container.

#### Example: Limiting CPU and Memory Usage

1. **CPU Limit**:
   You can limit the CPU usage of a container to a certain percentage. For example, to limit the **NGINX** container to 50% of the CPU, use the following command:
   ```
   docker run -d --name nginx-limited --cpus="0.5" nginx
   ```

   This command ensures the container doesn't use more than 50% of the CPU at any given time.

2. **Memory Limit**:
   You can limit the amount of **memory** a container can use. For example, to restrict the **NGINX** container to 100 MB of memory, use:
   ```
   docker run -d --name nginx-memory-limited --memory="100m" nginx
   ```

   This ensures the container can use no more than 100 MB of memory.

By using these options, Docker ensures that containers don’t overwhelm the host system's resources and helps in creating a more controlled environment for running applications.

---

### Conclusion

To summarize, **Docker containers** use **namespaces** to isolate various aspects of their environments, such as processes, network settings, and file systems. The **PID namespace** allows containers to have their own isolated process space, making it appear as if they are running independently, even though they are still sharing the same host system.

The **NGINX** example demonstrated how Docker uses PID namespaces to isolate processes, making the container think it has its own root process (PID 1). Docker also provides options to limit resource usage, such as CPU and memory, to ensure containers don’t exhaust host resources.
