# Docker on Mac: Understanding the Options

## Introduction

Just like on Windows, there are multiple ways to run Docker on a Mac. However, it’s important to note that Docker on macOS is designed to run **Linux containers** on a Mac host. At the time of writing, there are no **Mac-based Docker images** or **Mac containers**. This distinction is important, as it affects how Docker operates on macOS.

We’ll explore two primary options for running Docker on Mac:

1. **Docker Toolbox**
2. **Docker Desktop for Mac**

Let’s walk through each of these options.

## Option 1: Docker Toolbox

**Docker Toolbox** was the original method for running Docker on macOS before Docker Desktop was introduced. This option is still available, particularly for older systems or environments that do not meet the requirements for Docker Desktop.

### How Docker Toolbox Works

Docker Toolbox runs **Linux containers** on macOS via a **Linux VM** created using **Oracle VirtualBox**. Just like Docker on Windows, it doesn't run macOS-based containers or images. Instead, it uses a Linux VM to enable the execution of Linux containers.

### What Docker Toolbox Does

* **Docker Toolbox** installs and configures **Oracle VirtualBox**, which is used for virtualization.
* The toolbox includes several components:

  * **Oracle VirtualBox**: A virtualization tool for macOS.
  * **Docker Engine**: The core Docker functionality.
  * **Docker Machine**: Manages virtual machines.
  * **Docker Compose**: Used to define and run multi-container applications.
  * **Kitematic**: A user interface for Docker to help users manage containers visually.

Once installed, Docker Toolbox sets up VirtualBox, deploys a lightweight Linux VM called **Boot2Docker**, and runs Docker within that VM. This setup allows you to get started with Docker on macOS quickly.

### Requirements for Docker Toolbox

* **macOS 10.8 (Mountain Lion)** or newer.
* Virtualization enabled (usually enabled by default on modern Macs).
* Docker Toolbox is considered a **legacy solution** and is intended for systems that cannot support Docker Desktop for Mac.

## Option 2: Docker Desktop for Mac

**Docker Desktop for Mac** is the newer, more efficient solution for running Docker on macOS. It uses a native macOS virtualization technology called **HyperKit** to run Docker containers, providing a more seamless and integrated experience compared to Docker Toolbox.

### Key Differences from Docker Toolbox

* Docker Desktop removes the need for **Oracle VirtualBox** and uses **HyperKit**, a macOS-native virtualization tool.
* During installation, Docker Desktop automatically sets up a Linux VM using HyperKit, which then runs Docker containers.

### Requirements for Docker Desktop

* **macOS Sierra (10.12)** or newer.
* **Mac hardware**: Must be from **2010 or newer**.

### Running Linux Containers on Mac

Just like with Docker Toolbox, Docker Desktop for Mac is designed to run **Linux containers**. There are no macOS-specific containers or images available at this time.

## Conclusion

To summarize, there are two options for running Docker on macOS:

1. **Docker Toolbox**: The legacy solution that uses **Oracle VirtualBox** to run Docker inside a Linux VM. Suitable for older systems or those that do not meet the requirements for Docker Desktop.
2. **Docker Desktop for Mac**: The modern solution that uses **HyperKit** for virtualization, providing a more seamless and efficient experience. This option requires macOS Sierra or newer and is ideal for most users.

In both cases, the focus is on running **Linux containers** on macOS, as there is no support for macOS-based containers or images. If you're using an older version of macOS, Docker Toolbox is still a viable option, but Docker Desktop is generally recommended for newer systems due to its better performance and integration.
