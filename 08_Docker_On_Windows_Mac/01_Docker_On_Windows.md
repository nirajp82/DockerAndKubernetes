# Docker on Windows: Understanding the Options

## Introduction

In this section, we will explore the two primary ways to run Docker on Windows: **Docker Toolbox** and **Docker Desktop for Windows**. Before we dive into these options, it's important to understand a fundamental concept about containers.

Containers share the underlying **OS kernel**, which means that **a Windows container cannot run on a Linux host**, and **vice versa**. This distinction is critical when working with Docker on Windows, as beginners often encounter issues related to this concept.

Let's look at the two available options for running Docker on Windows.

## Option 1: Docker Toolbox

**Docker Toolbox** was the original solution for running Docker on Windows before Docker Desktop was introduced. It is suitable for older systems or situations where newer options aren't available.

### How Docker Toolbox Works

Imagine you have a Windows laptop with no access to Linux systems, either locally or in the cloud, but you want to try Docker. In this scenario, Docker Toolbox helps by using a virtualization approach. You would install software like **Oracle VirtualBox** or **VMware Workstation** to create a **Linux VM** (e.g., Ubuntu or Debian), and then install Docker inside that Linux VM.

### What Docker Toolbox Does

* **Docker Toolbox** essentially allows Docker to run on a **Linux virtual machine (VM)** running on Windows.
* The toolbox includes several components:

  * **Oracle VirtualBox**: Used for virtualization.
  * **Docker Engine**: The core functionality of Docker.
  * **Docker Machine**: A tool to manage virtual machines.
  * **Docker Compose**: A tool for defining and running multi-container Docker applications.
  * **Kitematic**: A user interface for Docker, helping users to manage Docker containers easily.

Once Docker Toolbox is installed, it automatically sets up VirtualBox, deploys a lightweight Linux VM called **Boot2Docker**, and runs Docker inside that VM. This approach makes it easier to start using Docker, especially for those who do not have direct access to Linux environments.

### Requirements for Docker Toolbox

* **Windows 7 (64-bit)** or higher.
* **Virtualization** must be enabled in your BIOS settings.
* Docker Toolbox is considered a **legacy solution**, meaning it’s intended for older systems or those that don’t meet the requirements for **Docker Desktop for Windows**.

## Option 2: Docker Desktop for Windows

**Docker Desktop for Windows** is the modern solution for running Docker on Windows. It uses **Microsoft Hyper-V**, a native virtualization technology in Windows, to run Docker containers. This option is more efficient and seamless compared to Docker Toolbox.

### Key Differences from Docker Toolbox

* Docker Desktop uses **Hyper-V** (native Windows virtualization) instead of Oracle VirtualBox for virtualization.
* During installation, Docker Desktop automatically sets up a Linux system on your Windows host using Hyper-V, eliminating the need for external virtualization software like VirtualBox.

### Requirements for Docker Desktop

* **Windows 10** (Professional or Enterprise Edition).
* **Windows Server 2016**.

Both these operating systems have **Hyper-V** support built-in, making them ideal for Docker Desktop.

### Windows Containers: The Game-Changer

Docker Desktop originally focused on running **Linux containers** on Windows. However, **Microsoft** announced support for **Windows containers** in early 2016, enabling you to run Windows-based applications inside Docker containers.

By default, Docker Desktop for Windows runs **Linux containers**. However, you can switch to **Windows containers** by configuring Docker for Windows accordingly.

### Types of Windows Containers

There are two types of Windows containers you can use, depending on your needs:

1. **Windows Server Containers**:

   * These containers share the **OS kernel** with the underlying Windows host, similar to how Linux containers operate.
   * Provides a secure and isolated environment for running Windows applications.

2. **Hyper-V Isolation**:

   * Each container runs inside its own optimized virtual machine.
   * This approach guarantees **complete kernel isolation** between the container and the host system.

### Windows Base Images

In the Linux world, you have several base images like **Ubuntu**, **Debian**, and **Alpine** to build your containers. In the Windows ecosystem, there are two main base images:

* **Windows Server Core**: A heavier base image with a full Windows Server environment.
* **Nano Server**: A minimal, headless version of Windows Server, designed for lightweight containers. It’s similar to **Alpine** in the Linux world.

### Supported Platforms for Windows Containers

* **Windows Server 2016** (with Nano Server).
* **Windows 10 Professional/Enterprise Edition** (supports Hyper-V isolated containers only).

## Important Considerations

### VirtualBox and Hyper-V Compatibility

It’s essential to understand that **VirtualBox** and **Hyper-V** cannot run simultaneously on the same Windows host. If you're using Docker Toolbox (which relies on VirtualBox) and decide to switch to Docker Desktop (which requires Hyper-V), you’ll need to uninstall VirtualBox first.

Fortunately, Docker provides a **migration guide** to help you transition from Docker Toolbox to Docker Desktop, ensuring a smooth shift from VirtualBox to Hyper-V.

## Conclusion

We’ve explored the two primary options for running Docker on Windows:

1. **Docker Toolbox**: A legacy solution that runs Docker on a Linux VM via Oracle VirtualBox. Ideal for older systems or those not supporting Docker Desktop.
2. **Docker Desktop for Windows**: The modern solution that uses Microsoft Hyper-V for running Docker containers. It offers better performance and flexibility, supporting both Linux and Windows containers.

Remember that Docker Desktop allows you to run **Windows containers**, which enables containerization of Windows-based applications. Docker on Windows has come a long way, and with these options, you have the flexibility to choose what works best for your setup.
