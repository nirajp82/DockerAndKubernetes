### Docker: Commands, Arguments, and Entrypoints

## 📘 Simple Scenario

Say you were to run a Docker container from an **Ubuntu** image.

```bash
docker run ubuntu
```

When you run the above command, it starts an instance of the Ubuntu image **and exits immediately**.

Now, if you were to list the running containers:

```bash
docker ps
```

You **won’t see** the container running.

If you list **all containers**, including those that are stopped:

```bash
docker ps -a
```

You will see that the container you ran is in an **exited state**.

---

## 🤔 Why Is That?

Unlike virtual machines, **containers are not meant to host a full operating system**.

Containers are meant to run a **specific task or process**, such as:
- Hosting a web server,
- Running an application server or database,
- Performing computations or analysis tasks.

Once the task is complete, the container **exits**.

> A container only lives as long as the process inside it is alive.

If the web service inside the container is stopped or crashes, the container exits.

---

## 🧭 Who Defines What Process Runs in the Container?

If you look at the **Dockerfile** for popular Docker images like **NGINX** or **MySQL**, you will see an instruction called `CMD`.

The `CMD` instruction (short for **command**) defines the **default program** that runs when the container starts.

- For **NGINX**, it starts the `nginx` command.
- For **MySQL**, it starts the `mysqld` command.

```docker
FROM ubuntu:latest
RUN apt-get -y update && apt-get -y install nginx
COPY content/index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```
---

## 🛠️ What Happens with the Ubuntu Image?

When we tried to run a container with the **plain Ubuntu image**, we were actually starting a container that launches the **bash** shell as the default command.

But **bash is not a service**, it's an interactive shell that **expects a terminal**.

If it doesn’t find a terminal, it **exits**.

So when we ran the Ubuntu container earlier, Docker:
1. Created a container from the Ubuntu image.
2. Launched the `bash` program (by default).
3. But didn’t attach a terminal.
4. So `bash` exited.
5. Therefore, the container exited.

---

Sure, let me explain it in simpler terms!

### **ENTRYPOINT vs CMD in Docker**

Both `ENTRYPOINT` and `CMD` are used in a Dockerfile to define what command or application will be run inside the container when it's started. But they work slightly differently, especially in how they handle additional arguments passed to `docker run`.

---

### **CMD** (Command)
- **Purpose**: Defines the default command to run inside the container if no command is specified when running the container.
- **Replacement behavior**: If you provide a command when you run the container (`docker run <container_name> <new_command>`), the `CMD` will be replaced entirely by the command you provide.

#### Example of CMD:
```Dockerfile
CMD ["echo", "Hello, World!"]
```

- Running `docker run <container_name>` will execute `echo Hello, World!`.
- Running `docker run <container_name> echo Goodbye!` will override the `CMD` and run `echo Goodbye!` instead.

### **ENTRYPOINT** (Entry Point)
- **Purpose**: Defines a fixed command to run when the container starts, and **any additional arguments passed** via `docker run` will be **added** to this command.
- **Appended behavior**: If you specify any command or arguments when running the container, those arguments are **appended** to the `ENTRYPOINT`.

#### Example of ENTRYPOINT:
```Dockerfile
ENTRYPOINT ["echo"]
```

- Running `docker run <container_name>` will execute `echo` (but nothing will be printed because no arguments are provided).
- Running `docker run <container_name> Hello, World!` will execute `echo Hello, World!`.

### **Key Difference**: 

- **CMD** provides a default command, but it can be **completely overridden** if you pass new arguments when running the container.
- **ENTRYPOINT** locks in the command to run, and any additional arguments you pass are **appended** to the entrypoint command.

---

### **Combining ENTRYPOINT and CMD**
You can combine both `ENTRYPOINT` and `CMD` in the Dockerfile. This allows you to set a fixed entrypoint (with `ENTRYPOINT`) and a default argument (with `CMD`). If you provide arguments when running the container, the `ENTRYPOINT` remains the same, but the arguments from `CMD` can be overridden.

#### Example:
```Dockerfile
ENTRYPOINT ["echo"]
CMD ["Hello, World!"]
```

- Running `docker run <container_name>` will execute `echo Hello, World!` (because `CMD` provides the argument).
- Running `docker run <container_name> Goodbye!` will execute `echo Goodbye!` (overriding `CMD`).

---

### **Summary**:
- **ENTRYPOINT**: Defines the main command and always runs when the container starts. Additional arguments are appended.
- **CMD**: Defines default arguments for the command, but can be replaced entirely if new arguments are provided at runtime.

Does that clear things up?
