# Docker on Windows: Understanding the Options

## Overview

In this course, we will discuss the options available for running Docker on Windows. Containers share the underlying OS kernel, which means that you cannot run a Windows container on a Linux host or vice versa. This concept is crucial for understanding Docker on Windows, as many beginners face difficulties with this distinction.

We will explore two primary options for Docker on Windows:

1. **Docker Toolbox**
2. **Docker Desktop for Windows**

Let's delve into each of these options in detail.

## Docker Toolbox

**Docker Toolbox** was the original solution for running Docker on Windows. Here's how it works:

### Scenario

Imagine you have a Windows laptop and no access to any Linux system (e.g., no lab or cloud environment). You want to try Docker, but you can't run it natively on Windows. What do you do? The solution was to install a virtualization software (e.g., Oracle VirtualBox or VMware Workstation), deploy a Linux VM (e.g., Ubuntu or Debian), and install Docker inside the Linux VM.

### What Docker Toolbox Does

Docker Toolbox doesn't rely on Windows directly; instead, it uses a Linux virtual machine to run Docker. The Docker Toolbox includes the following components:

* **Oracle VirtualBox** (for virtualization)
* **Docker Engine** (core Docker functionality)
* **Docker Machine** (to manage virtual machines)
* **Docker Compose** (for multi-container Docker applications)
* **Kitematic** (a user interface for Docker)

When you install Docker Toolbox, it installs VirtualBox, deploys a lightweight Linux VM called "Boot2Docker," and runs Docker inside that VM. This setup allows you to get started with Docker quickly and easily.

### Requirements

* **64-bit Windows 7** or higher
* **Virtualization enabled** in BIOS
* Docker Toolbox is a legacy solution and is primarily for systems that do not meet the requirements for Docker Desktop for Windows.

## Docker Desktop for Windows

**Docker Desktop for Windows** is the newer, more powerful option for running Docker on Windows. This option uses **Microsoft Hyper-V**, a native virtualization technology available in Windows, to run Docker containers.

### Key Differences from Docker Toolbox

* Docker Desktop uses **Hyper-V** instead of Oracle VirtualBox for virtualization.
* It automatically creates a Linux system beneath the Windows host, but this system runs on Hyper-V, not VirtualBox.

### Requirements

* **Windows 10 Enterprise** or **Professional Edition**
* **Windows Server 2016**

Both of these operating systems have Hyper-V support built in.

### Windows Containers

Initially, Docker on Windows was only for running **Linux containers**. However, Microsoft announced support for **Windows containers** in early 2016. With Docker Desktop for Windows, you can now run **Windows containers** as well, which allows you to package and run Windows applications inside Docker containers.

By default, Docker Desktop for Windows works with Linux containers, but you can switch to **Windows containers** by configuring Docker accordingly.

### Two Types of Windows Containers

1. **Windows Server Containers**:

   * Similar to Linux containers.
   * Share the OS kernel with the underlying Windows host.
   * Provide security isolation and kernel version compatibility between containers.

2. **Hyper-V Isolation**:

   * Each container runs inside its own highly optimized virtual machine.
   * Guarantees complete kernel isolation between containers and the host.

### Windows Base Images

While Linux uses various base images like **Ubuntu**, **Debian**, and **Alpine**, Windows has two primary options:

* **Windows Server Core**: A heavier base image with a full Windows Server environment.
* **Nano Server**: A minimal, headless deployment option for Windows Server, ideal for lightweight containers. It's similar to **Alpine** in the Linux world.

### Supported Platforms for Windows Containers

* **Windows Server 2016** (with Nano Server)
* **Windows 10 Professional/Enterprise Edition** (only supports Hyper-V isolated containers)

## Important Considerations

### VirtualBox and Hyper-V Coexistence

It's important to note that **VirtualBox** and **Hyper-V** cannot run together on the same Windows host. If you're using Docker Toolbox (which relies on VirtualBox) and later decide to migrate to Docker Desktop (which relies on Hyper-V), you will need to uninstall VirtualBox before using Hyper-V.

Docker provides a **migration guide** on their documentation page to help with the transition from VirtualBox to Hyper-V.

## Conclusion

In this lecture, we've explored two primary options for running Docker on Windows:

* **Docker Toolbox**: A legacy solution using Oracle VirtualBox to run Docker inside a Linux VM.
* **Docker Desktop for Windows**: A modern solution using Microsoft Hyper-V to run Docker containers natively on Windows, with support for both Linux and Windows containers.

Remember that while Docker on Windows initially focused on running Linux containers, with the introduction of Windows containers, you can now containerize Windows-based applications as well.
