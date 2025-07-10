# Embedded Systems Overview

## What is an Embedded System?

An **embedded system** is a combination of hardware and software designed to perform a **specific task**.  
Key characteristics:

- Equipped with a processing unit.
- Includes temporary memory and permanent storage.
- Part of a larger system.
- Customized for specific functionality.
- Lower cost compared to general-purpose systems.

Each component in such a system can be considered an embedded system when it has its own processor and memory.

---

#  Kernel Configuration: `make menuconfig` vs. Yocto Approach

When working with Linux kernel customization for embedded systems, there are two main ways to configure the kernel:

---

## . `make menuconfig` — Traditional Kernel Configuration

This is the **standard method** used when you're working directly with the Linux kernel source **outside** of Yocto.

#  What is Yocto Project?

The **Yocto Project** is an open-source collaboration project that helps developers **create custom Linux distributions** for embedded systems, regardless of the hardware architecture (e.g., ARM, x86).

> Yocto is not a Linux distribution — it's a set of tools for building your own.

---

##  What Can Be Customized in the Kernel with Yocto?

When using Yocto to build an embedded Linux image, you can **deeply customize the Linux kernel** to fit your device's requirements — from performance to memory footprint.

Here are the major kernel aspects you can customize:

###  1. **Boot Time Optimization**
- Remove unnecessary drivers or features to reduce kernel size.
- Disable debugging or logging features to speed up boot.
- Optimize init system and enable fast boot techniques like `read-only rootfs` or `initramfs`.

###  2. **Memory Usage**
- Disable memory-hungry subsystems not used by your application.
- Use lightweight libraries and a small `C` library like `musl` or `uClibc`.
- Optimize kernel page size and memory allocators (e.g., SLUB vs SLOB).

###  3. **Filesystem Type**
- Choose from multiple filesystems:
  - `ext4`, `squashfs`, `jffs2`, `ubifs`, `cpio`, etc.
- Read-only or read-write depending on use case.
- Enable/disable journaling to save flash wear.

###  4. **Device Drivers**
- Only include drivers relevant to your hardware (I2C, SPI, GPIO, UART, USB, etc.).
- Disable unneeded drivers to reduce image size and attack surface.
- Add support for custom drivers via `.bbappend` and `.cfg`.

###  5. **Security Features**
- Enable kernel hardening options:
  - Stack protector
  - ASLR (Address Space Layout Randomization)
  - SELinux or AppArmor
- Remove legacy or unused protocols to reduce attack surface.

###  6. **Processor and Architecture Specific Settings**
- Set correct CPU type (e.g., Cortex-A53, A72).
- Optimize for single-core or multi-core systems.
- Enable hardware floating point, NEON support, etc.

###  7. **Kernel Features and Subsystems**
- Networking stack features (IPv6, VLAN, etc.)
- Filesystem features (overlayfs, squashfs compression)
- USB gadget support, HID, CAN, BLE, etc.
- Real-time kernel features (PREEMPT-RT patch)

### 8. **Init System**
- Use minimal init systems like `busybox`, `systemd`, or `SysV`.
- Customize init scripts to boot directly into your application.

---

#  Desktop Kernel vs  Embedded Kernel

Understanding the difference between a desktop and embedded Linux kernel helps in making design decisions based on system requirements.

---

##  Key Differences

| Feature / Attribute            |  Desktop Kernel                          |   Embedded Kernel                         |
|-------------------------------|-------------------------------------------|--------------------------------------------|
| **Purpose**                   | General-purpose (laptops, PCs, servers)   | Application-specific (IoT, industrial, etc.)|
| **Size**                      | Large (many modules & drivers)            | Small, optimized for specific hardware     |
| **Drivers**                   | Includes support for a wide range         | Only includes required drivers             |
| **User Interface**            | Supports full GUI environments            | Often headless or minimal UI               |
| **Boot Time**                 | Not a priority (20–60 seconds)            | Critical (needs fast boot: < 5 seconds)    |
| **Memory Usage**              | Assumes large RAM (GBs)                   | Optimized for low RAM (MBs)                |
| **Filesystem**                | ext4, btrfs, xfs                          | squashfs, ubifs, jffs2, cramfs             |
| **Init System**               | systemd, upstart                          | busybox init, systemd (minimal), SysVinit  |
| **Security Model**            | Multi-user, complex permissions           | Often single-user or sandboxed application |
| **Update Mechanism**          | Package manager (apt, rpm)                | Image-based (OTA) or atomic updates        |
| **Real-Time Support**         | Optional, not default                     | Frequently patched with PREEMPT-RT         |
| **Build Process**             | Binary packages (Ubuntu, Fedora, etc.)    | Custom-built using Yocto, Buildroot        |

---


#  How Linux Boots (Boot Process Overview)

The Linux boot process consists of several stages that transform a powered-off device into a fully running Linux system.

---

##  Linux Boot Stages 

1. **Power On / Reset**    

2. **Bootloader (e.g., GRUB, U-Boot)**  

3. **Linux Kernel**  

4. **Init System (e.g., systemd, SysVinit, BusyBox)**   

5. **User Space / Shell / Application**  


### 1️ Power-On and Reset (ROM Code)
- Processor starts executing from a fixed ROM location.
- Initializes clocks, RAM, and basic peripherals.
- Loads the **first-stage bootloader** (if required).

### 2️ Bootloader (e.g., GRUB, U-Boot)
Responsible for:
- Initializing board-specific hardware (RAM, UART, SD card, etc.)
- Loading the **Linux kernel** into RAM
- Loading the **Device Tree Blob** (`.dtb`) for ARM-based systems
- Loading an **initramfs/initrd** (optional)
- Passing control to the kernel

**Desktop**: GRUB loads `vmlinuz`  
**Embedded**: U-Boot loads `zImage` or `uImage`

---

### 3️ Linux Kernel Initialization
Once loaded:
- Sets up memory management (MMU, paging)
- Initializes device drivers (console, storage, network, etc.)
- Mounts the **root filesystem**
- Starts the **init process** (PID 1)

📄 Files loaded:
- Kernel image (`vmlinuz`, `zImage`, `uImage`)
- Optional: `initrd` / `initramfs`
- Device Tree (`.dtb`) — ARM only

---

### 4️ Init System (PID 1)
The **first user-space process** started by the kernel.

- Reads config files (`/etc/inittab`, `/etc/systemd/`, etc.)
- Starts system services (networking, sshd, logging, GUI)
- Mounts additional filesystems
- Launches user applications or login shell

Examples:
- `systemd` (desktop)
- `BusyBox init`, `SysVinit` (embedded)

---

### 5️ User Space & Applications
Finally:
- You get a login prompt, shell, or GUI
- In embedded systems, it may auto-launch a specific application (e.g., GUI, kiosk app, sensor logic)

---

### INTRODUCTION TO YOCTO 
![Image](https://github.com/user-attachments/assets/86f25d42-cd86-4dc0-a0e9-b74ded2dce5a)

#  Yocto Build System Overview

##  What is a Build System?

A **build system** automates the process of:

- Taking source code  
- Compiling it  
- Packaging it into **binaries**  
- Creating a bootable **Linux image** or **SDK**

In the **Yocto Project**, the build system consists of tools like **BitBake**, **OpenEmbedded**, and configuration **metadata** that guide the entire build process.

---

##  What is OpenEmbedded?

**OpenEmbedded (OE)** is a **build framework** for embedded Linux systems.

- It defines **recipes** (instructions for building packages).
- Manages **dependencies** and **cross-compilation**.
- Provides **layers** (collections of recipes) to customize builds for different hardware.

> Think of it as the "**database** + **rules**" for building embedded Linux distributions.

---

##  What is BitBake?

**BitBake** is the **build engine** of the Yocto Project and OpenEmbedded.

- It reads **metadata** (recipes, configurations).
- Decides **what to build and how**.
- Calls the appropriate **compiler/toolchain** to build the code.

>  BitBake is like the `make` tool for Yocto — it interprets instructions and does the actual building.

---

##  What are Binaries?

**Binaries** are the compiled output files generated from source code.

Yocto/BitBake generates:

- **Kernel image** (e.g., `zImage`, `Image`, `uImage`)
- **Bootloader** (e.g., `U-Boot`)
- **Root filesystem** (`rootfs.ext4`, `rootfs.tar.gz`)
- **Device Tree Blobs** (`.dtb`)
- **Kernel modules** (`.ko`)
- **Application binaries** (`/usr/bin/myapp`)

---
---
### Yocto Project ###
  
## Introduction to Yocto project

==> Role of Yocto Project
- The Yocto Project is a toolset and framework.
- The Yocto Project is a open_source Collabrative project, that helps developers build custom Linux-based operating systems for embedded systems (like smart meters, routers, automotive devices, industrial machines, etc.).

==> Core Components and Terminology
```
+----------------------------+
|   Your Custom Layers       |  ← Your own apps, configs, and board support
|   (meta-myproject, etc.)   |
+----------------------------+
| Community/Third-Party Layers | ← meta-raspberrypi, meta-qt5, etc.
+----------------------------+
|           Poky             |  ← Reference distro (includes BitBake + metadata)
+----------------------------+
|       BitBake Engine       |  ← Executes tasks using recipes
+----------------------------+
|     OpenEmbedded Core      |  ← Shared base recipes, classes, configs
+----------------------------+
|   Source Code / Packages   |  ← Kernel, BusyBox, toolchain, libraries
+----------------------------+
```
- Source code: This is the raw software you want to build into your embedded Linux images.
- OpenEmbedded: This is the **build system** and the primary software ecosystem used by the Yocto Project to generate embedded Linux distributions.
OpenEmbedded consists of a collection of **scripts and Python libraries** that manage operations on software sources and packages for embedded Linux development.
- BitBake: This is the **build engine** that powers the OpenEmbedded build system.
- Poky: Poky is the reference build system provided by the Yocto Project. It includes everything needed to build a custom embedded Linux OS.
  - it include's:
    - BitBake
    - Metadata
      - Recipe (.bb) Describe hoe to fetch, configure, compile, and install packages.
      - class (.bbclass) Reusable funtions and build logic.
      - config files (.conf) it Defines variables, machine settings, distro settings, etc...
      - Default layers- like a meta, meta_poky, meta_Yocto_bsp 
    - Custom Layer: This is where your own project code goes.

---
---
    1. What is Yocto Project?
    2. Explain the Core Components of Yocto Project?
    3. What are layers in Yocto?
    4. What is Poky and explain it's main components?
---


## Yocto Project Architecture

<img src="/Architeture.png" alt="Yocto_Project.md" width="600"/>

I. Inputs to the OpenEmbedded Build System
---

1. User Configuration: These are conf files that users configure to set parameters like the machine type for the build and the layers to be included.
2. Metadata: This encompasses all the layers and recipe files contained within those layers.
3. Recipe files describe how BitBake executes tasks for each software package, such as fetching, unpacking, patching, compiling, and installing.
4. Machine Configuration: Describes everything related to the platform used in the build and compiler tuning options. Examples include:
   - Architecture
   - Device tree
   - Kernel command line arguments
   - Kernel config file
5. Policy Configuration: Customizable settings to change how the build system behaves.
6. Source Mirrors: Sources from where the build system retrieves source code. These can be Git repositories, tarball archives, or local sources. The build system downloads software source packages from these mirrors based on configuration.

--------------------------------------------------------------------------------

II. OpenEmbedded Build System Tasks (Inside the Blue Box)
---

1. Source Fetching:
   - Described in the metadata using Python and Bash scripts within recipes and patches.
   - Downloads or clones source code from specified source mirrors.
   - Archives and Git repositories are stored in a downloads folder.
3. Unpack and Extract:
   - After fetching, sources are unpacked and extracted into different folders.
4. Patch Application:
   - Any modifications to software package sources are typically done via patches for ease of portability and application.
   - The OpenEmbedded system applies all instructed patches at this stage. Patches are often created to migrate software to a specific         platform or resolve build errors.
5. Configuration and Compilation:
   - Compilation is typically performed by GCC.
   - If other build systems (e.g., GNU Autoconf) are used, extra configuration is done before compilation to generate makefiles.
   - Compilation then proceeds using make commands and GCC.
   - Binary outputs are deployed to a separate folder.
6. Output Analysis:
   - OpenEmbedded gathers binary outputs and prepares files for package generation.
7. Package Generation:
   - Depending on the desired package manager output, packages are generated (e.g., RPM generation, Deb package outputs).
8. Quality Assurance Checks:
   - Applied to the generated packages and the binaries within the software package archive.
9. Software Package Outputs (First Stage Output):
   - If all checks are complete, the build system produces software package outputs, represented as a green box.
   - This is the first stage of OpenEmbedded's capability, where it creates package feeds or archives containing pre-compiled binaries         for a specific platform using configuration files and metadata.
   - These package archives can be copied to an already running Linux system of the same architecture and installed using package manager commands.

III. Software Package Feeds (First Stage Output)
---

- If everything passes QA, the system generates pre-built software packages (feeds):
  - Copy them to a running embedded Linux system
  - Install with package managers (opkg, dpkg, or rpm)
- This output makes it easy to update systems in the field without rebuilding the whole image.

IV. Final Outputs (Second Stage)
---
From the software package feeds, the following final outputs are generated:
1. System Image
- Bootable OS image for the target board
  - Contains:
      - Bootloader
      - Kernel
      - Root filesystem
- This is the end goal — to run Linux on your embedded hardware.

2. Application Development SDK
- A toolchain for developers to build user-space applications.
  - Includes:
      - Cross-compiler
      - Prebuilt libraries
      - Target sysroot
- Enables easy development of apps without setting up complex environments.

BitBake & Dependency Handling
---
1. BitBake is the build engine that runs the tasks described in each recipe.
- BitBake Commands:
  - Build a single package:
    - bitbake openssl
  - bitbake openssl
    - bitbake core-image-minimal
2. Dependency Handling:
  - If package A depends on B, BitBake builds B first.
  - Ensures correct build order automatically.






