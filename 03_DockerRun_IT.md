### **Understanding Interactive Mode (`-it`) in Docker**  

The `-it` flag in Docker enables **interactive mode**, allowing a container to accept user input and display output in a terminal-like environment. It consists of two options:  

- **`-i` (interactive)** → Keeps standard input (STDIN) open, allowing the user to provide input.  
- **`-t` (pseudo-TTY)** → Allocates a terminal so the output is formatted properly.  

Let’s explore how this works using a simple **shell script** inside a Docker container.  

---

### **Example: Simple Prompt Application**  

#### **Script (`greet.sh`)**  
```bash
#!/bin/bash
echo "Enter your name:"
read name
echo "Pranam $name"
```
- This script prompts for the user’s name and then prints a greeting.  
- When run **directly on a local machine**, it works as expected:  

#### **Expected Output (On Local Machine)**
```
Enter your name:
John
Pranam John
```

---

## **1️⃣ Running Without `-it`**  

```bash
docker run myimage
```

🔹 **What happens?**  
- The script **does not prompt for input** and **immediately exits**.  
- The expected output is **never shown**.  

🔹 **Why?**  
- Docker **does not attach standard input (STDIN)** by default.  
- The container starts, runs the script, and exits **without waiting** for user input.  

#### **Output (Without `-it`)**
```
( No prompt appears )
```
---

## **2️⃣ Running With `-i` (Interactive Mode)**  

```bash
docker run -i myimage
```

🔹 **What happens?**  
- Now, the container **accepts input** when the user types.  
- However, **the prompt (`Enter your name:`) is missing!**  

🔹 **Why?**  
- `-i` keeps **STDIN open**, so Docker listens for user input.  
- But since **no terminal (TTY) is attached**, the output is not properly formatted.  

#### **Output (With `-i`)**
```
( Cursor blinking, waiting for input but no visible prompt )
John
Pranam John
```
💡 **Problem:** The script **works**, but the prompt is **invisible**, making it unclear what the container is asking for.  

---

## **3️⃣ Running With `-it` (Interactive Terminal Mode)**  

```bash
docker run -it myimage
```

🔹 **What happens?**  
- The script **works exactly as expected**!  
- The **prompt appears**, waits for input, and prints the greeting.  

🔹 **Why?**  
- `-i` → Keeps **STDIN open**, so user input is accepted.  
- `-t` → Allocates a **pseudo-terminal (TTY)**, allowing proper **prompt visibility and output formatting**.  

#### **Output (With `-it`)**
```
Enter your name:
John
Pranam John
```
✅ **Now the container behaves just like running the script on a local machine!**  

---

## **Key Takeaways**
| Command | Behavior | Issue |
|---------|----------|---------|
| `docker run myimage` | **Script exits immediately** | No input is taken |
| `docker run -i myimage` | **Accepts input but no prompt appears** | No terminal attached |
| `docker run -it myimage` | **Works correctly (prompt + input)** | ✅ Fully interactive |

🔹 **Always use `-it` for interactive applications** like shells, prompts, and command-line tools! 🚀
