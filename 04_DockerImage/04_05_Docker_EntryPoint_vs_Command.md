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

## 🔄 How Do You Specify a Different Command?

You can **override** the default command by appending a new command in the `docker run` command.

For example:

```bash
docker run ubuntu sleep 5
```

This runs the `sleep` program, waits for 5 seconds, and then exits.

---

## 📦 Making the Change Permanent

If you want the image to **always run** the `sleep` command, you can create a custom image:

### Dockerfile Example:

```dockerfile
FROM ubuntu
CMD ["sleep", "5"]
```

> Note: The preferred format is **JSON array form** where:
> - The **first element** is the executable (e.g., `sleep`)
> - The following elements are arguments (e.g., `"5"`)

Then, build the image:

```bash
docker build -t ubuntu-sleeper .
```

And run it:

```bash
docker run ubuntu-sleeper
```

This will **always sleep for 5 seconds** and then exit.

---

## ⚙️ Changing the Sleep Duration

Right now, the sleep time is **hardcoded** to 5 seconds.

You could override it at runtime:

```bash
docker run ubuntu-sleeper sleep 10
```

This works, but it's not very clean. The image name `ubuntu-sleeper` already implies it will sleep.

Ideally, you'd want:

```bash
docker run ubuntu-sleeper 10
```

This should automatically invoke `sleep 10`.

---

## 🚪 Introducing ENTRYPOINT

This is where the `ENTRYPOINT` instruction comes in.

The `ENTRYPOINT` is **like `CMD`**, but with an important difference:

- The command-line arguments passed to `docker run` are **appended** to the `ENTRYPOINT`.
- With `CMD`, the arguments **replace** the default command entirely.

---

### Dockerfile Example Using ENTRYPOINT

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
```

Then running:

```bash
docker run ubuntu-sleeper 10
```

Will effectively run:

```bash
sleep 10
```

---

## ⚠️ What If No Arguments Are Passed?

If you run:

```bash
docker run ubuntu-sleeper
```

Then only the `sleep` command is executed **without an argument**, and you’ll see:

```text
sleep: missing operand
```

To fix this, you can combine `ENTRYPOINT` and `CMD`.

---

## ✅ ENTRYPOINT + CMD Together

```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```

This way:

- `docker run ubuntu-sleeper` → runs `sleep 5`
- `docker run ubuntu-sleeper 10` → runs `sleep 10`

> Reminder: To make this work correctly, **both `ENTRYPOINT` and `CMD` should be in JSON (exec) format**.

---

## 🔄 Overriding ENTRYPOINT at Runtime

What if you really want to change the `ENTRYPOINT` during runtime?

You can do so with the `--entrypoint` flag.

```bash
docker run --entrypoint sleep2.0 ubuntu-sleeper 10
```

This overrides the image’s entrypoint, so the container will run:

```bash
sleep2.0 10
```

---

## 🧾 Summary

- `CMD` defines default arguments or commands. It’s **fully overridden** by CLI input.
- `ENTRYPOINT` defines the main command. CLI arguments are **appended**.
- You can combine `ENTRYPOINT` and `CMD` for flexible behavior.
- Use the **JSON array syntax** for consistency and reliability.
- You can override `ENTRYPOINT` at runtime using the `--entrypoint` flag.

---

That’s it for this lecture. See you in the next one! 🚀
```

---

Let me know if you want this as a downloadable `README.md` file or want to create a GitHub repo around it!
