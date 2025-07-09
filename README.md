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
#  Yocto Project Basic Terminology

##  General Terms

| Term            | Description |
|------------------|-------------|
| **Yocto Project** | An open-source project that provides templates, tools, and methods to create custom embedded Linux distributions. It’s **not a Linux distro**, but a **build system framework**. |
| **Poky**          | The **reference distribution** of the Yocto Project. It combines **BitBake**, **OpenEmbedded-Core**, and sample configurations. Poky is the default build environment. |
| **BitBake**       | The **build engine** (like `make`) that reads recipes and executes tasks to fetch, configure, compile, and package software. |
| **OpenEmbedded**  | A **layered build framework** used by Yocto to organize recipes and configurations. Yocto uses **OpenEmbedded-Core (OE-Core)** as its base metadata layer. |

---

##  Metadata Structure Terms

| Term       | Description |
|------------|-------------|
| **Metadata** | A collection of instructions, configurations, and descriptions that tell BitBake how to build packages. Includes `.bb`, `.bbappend`, `.inc`, `.conf`, and `layer.conf` files. |

---

##  Files in Metadata

| File Type        | Description |
|------------------|-------------|
| **`.bb` (Recipe)**       | A recipe file that defines how to **fetch, configure, build, and install** a single software package. |
| **`.bbclass` (Class)**   | A class file that contains **reusable functions or variables** shared across recipes. For example, `autotools.bbclass` adds default steps for Autotools-based packages. |
| **`.bbappend` (Append)** | Used to **modify or extend** an existing recipe (usually from another layer) without editing the original `.bb` file. |
| **`.inc` (Include)**     | An include file with **shared variables or functions**. Recipes or config files can include this using `require` or `include`. Helps avoid duplication. |
| **`.conf` (Config)**     | Defines **build-time settings**, such as target machine, distro, or build features. Examples: `local.conf`, `distro.conf`, `machine.conf`. |

---

##  Layers

| Term       | Description |
|------------|-------------|
| **Layer**  | A directory that groups related metadata (**recipes**, **configs**, **classes**). Layers help organize and isolate features (e.g., `meta-networking`, `meta-qt5`). |

Each layer usually contains:
- `recipes-*` folders (e.g., `recipes-core/`)
- `conf/layer.conf` to register the layer
- Optional `.bb`, `.bbappend`, `.bbclass` files

---

### YOCTO PROJECT ARCHITECTURE
![Image](https://github.com/user-attachments/assets/ede9cadf-f8c4-4920-91db-fe0b828d9167)

---
## How Yocto Fetches Source Files (with SRC_URI)
Where SRC_URI Is Defined
You define it inside a BitBake recipe file (.bb), like this:

```bitbake

SRC_URI = "git://github.com/uclibc/uclibc-ng.git;protocol=https;branch=master"
```
This tells BitBake to fetch the source code from the specified Git repository using HTTPS and the master branch.

Once a Git Repo is Downloaded — Where Is It Stored?
When BitBake fetches a Git repository (as defined in SRC_URI), it stores it in a structured subdirectory inside DL_DIR (typically downloads/).

 Git Repository Storage Path
Yocto stores cloned Git repositories in:
```bash
<build-dir>/downloads/git2/<sanitized-git-url>/
```
 Example
If your SRC_URI is:

```bitbake
SRC_URI = "git://github.com/uclibc/uclibc-ng.git;protocol=https;branch=master"
```
Then Yocto saves the repo under:

```bash
<build-dir>/downloads/git2/github.com.uclibc.uclibc-ng.git/
```
## What Happens If Fetch Fails in Yocto?
# BitBake Raises an Error
```bitbake
ERROR: Fetcher failure for URL: 'git://github.com/uclibc/uclibc-ng.git;protocol=https;branch=master'
ERROR: Unable to fetch URL from any source.

```
##  User Configuration

###  Path:
```bash
build/conf/local.conf
```

---

###  Role:

-  **Set target machine:**
  ```conf
  MACHINE = "raspberrypi4-64"
  ```

-  **Configure download paths:**
  - `DL_DIR` – Directory where downloaded source archives are stored
  - `SSTATE_DIR` – Shared state cache location

-  **Define mirror behavior:**
  - `PREMIRRORS` – Fetch from alternate sources before trying upstream

-  **Set build parallelism:**
  - `BB_NUMBER_THREADS` – Max BitBake tasks to run in parallel
  - `PARALLEL_MAKE` – Threads used by `make` within a single task


---
 ## Sample local.conf (Generated)
 ```bitbake
# Yocto Project Local Configuration File

# Set target machine
MACHINE ?= "raspberrypi4-64"

# Set download and sstate-cache directories
DL_DIR ?= "${TOPDIR}/downloads"
SSTATE_DIR ?= "${TOPDIR}/sstate-cache"

# Use downloaded mirrors before trying the original sources
PREMIRRORS_prepend = "\
git://.*/.*  http://my-mirror.local/git/ \n \
http://.*/.*  http://my-mirror.local/http/ \
"

# Set build parallelism
BB_NUMBER_THREADS = "4"
PARALLEL_MAKE = "-j4"
```
## What is Metadata in Yocto?
In Yocto, metadata refers to all the configuration files, recipes, and patches that describe:

What software to build

Where to fetch the source

How to configure, compile, and install it

What to include in the final Linux image

## Types of Metadata Files

.bb files (BitBake Recipes)
These are the core building blocks.

Define how to fetch, build, and install software packages.

Example: myapp_1.0.bb
 ```bitbake
DESCRIPTION = "My custom app"
LICENSE = "MIT"
SRC_URI = "git://github.com/user/myapp.git;protocol=https;branch=main"

S = "${WORKDIR}/git"

do_compile() {
    oe_runmake
}

do_install() {
    install -d ${D}${bindir}
    install -m 0755 myapp ${D}${bindir}
}

```
##.bbappend files (Recipe Extensions)
Used to modify existing recipes without editing them directly.

Example: Add extra patches or build flags.

Example: busybox
```bitbake
SRC_URI += "file://my_busybox_patch.patch"
```
##  `patches/` Directory in Yocto

###  What Is a Patch?

A **patch** is a file that contains specific changes you want to apply to existing source code.

> It tells the build system:  
> _"Go to this file and make this exact change."_

Patches are commonly used in Yocto to:

-  Fix bugs
-  Add or remove features
-  Customize open-source code **without directly modifying the original source**

---

###  Real Examples of Patches

---

###  Enable a Custom Kernel Module

**Problem**: You want the kernel to compile your own driver.

**Patch Example** (added to the kernel's Makefile):

```diff
 obj-$(CONFIG_MY_DRIVER) += my_driver.o
```

---

###  Add a Missing Header File

**Problem**: Build error — `‘memset’ was not declared`

**Patch** (added to `main.c`):

```diff
 #include <stdio.h>
+#include <string.h>  //  Added this to fix the error
```

 
---
## Machine(BSP CONFIGURATION)
It includes:

The board (e.g., Raspberry Pi 4, BeagleBone, etc.)

Bootloader settings

Kernel configuration

Device tree files


##  Example: Raspberry Pi 4 (64-bit)

```conf
# File: meta-raspberrypi/conf/machine/raspberrypi4-64.conf

DESCRIPTION = "Raspberry Pi 4 64-bit"
require conf/machine/include/rpi-base.inc

KERNEL_DEVICETREE = "bcm2711-rpi-4-b.dtb"

UBOOT_MACHINE = "rpi_4_defconfig"

SERIAL_CONSOLE = "115200 ttyAMA0"
```


##  Policy Configuration Examples in Yocto

---

###  Filesystem Type

```conf
IMAGE_FSTYPES = "ext4"
```

 Choose what image format to generate (e.g., `ext4`, `wic`, `tar.gz`).

---

###  Locale and Timezone

```conf
IMAGE_LINGUAS = "en-us"
TIMEZONE = "Asia/Kolkata"
```

 Set language and system timezone.

---

### . Licenses and Compliance

```conf
INCOMPATIBLE_LICENSE = "GPLv3"
```

 Prevents packages with unwanted licenses from being included.

---

###  Default Users and Passwords

```bitbake
EXTRA_USERS_PARAMS = "usermod -P mypass root;"
```
##  .rpm, .deb, and .ipk Are Types of Packages — Not the Package Itself

> A **package** refers to a **bundle of built software** — such as binaries, libraries, documentation, config files, etc.  
> `.rpm`, `.deb`, and `.ipk` are just **different formats** in which that bundle is **compressed, structured, and delivered**.

---

##  Package Format Comparison

| **Package Format** | **Common Use**                     | **Managed By**       |
|--------------------|------------------------------------|-----------------------|
| `.rpm`             | Red Hat, Fedora, Yocto (optional)  | `rpm`, `yum`, `dnf`   |
| `.deb`             | Debian, Ubuntu, Yocto (optional)   | `dpkg`, `apt`         |
| `.ipk`             | OpenEmbedded, Yocto (default)      | `opkg`                |

---

##  Summary

- All three formats serve the same purpose: **install software packages** on a system.
- They differ in:
  - **Packaging structure**
  - **Compression method**
  - **Supported package manager**
- In **Yocto**, the format is chosen using `PACKAGE_CLASSES` in `local.conf`:

```conf
PACKAGE_CLASSES ?= "package_ipk"
```

You can also use:

```conf
PACKAGE_CLASSES ?= "package_rpm"
PACKAGE_CLASSES ?= "package_deb"
```
##  Real Examples of Software Packages

###  1. Application Packages (Runtime)

| **Package Name** | **Description**                              |
|------------------|----------------------------------------------|
| `busybox`        | A lightweight Unix command-line toolset      |
| `dropbear`       | A small SSH server and client                |
| `curl`           | Tool to transfer data via HTTP/FTP           |
| `nano`           | Text editor for terminal                     |
| `python3`        | Python 3 interpreter                         |
| `htop`           | Interactive process viewer                   |

---

##  What Are Package Feeds in Yocto?

###  Definition:
Package feeds are **directories or remote servers** containing the **binary packages** (`.ipk`, `.deb`, `.rpm`) that Yocto builds and stores — ready to be installed later on the target device using a package manager (`opkg`, `apt`, or `rpm`).

---

###  Where Are They Located?

After building an image with BitBake, you’ll find the package feeds in:

```bash
tmp/deploy/ipk/
tmp/deploy/rpm/
tmp/deploy/deb/
```

Each of these contains architecture-specific folders with the actual packages and index files (like `Packages.gz`) used by package managers.


## From Package Feeds  Image & SDK

# BitBake Builds Packages

BitBake recipes (`.bb` files) fetch source code using `SRC_URI`.

The code is then:

1. Configured
2. Compiled
3. Installed

Finally, it's split into multiple **binary packages**.
 **Output** → `.ipk`, `.deb`, or `.rpm`

 **Stored in**:
```bash
tmp/deploy/ipk/
```

These are your **Package Feeds**.

---

##  Root Filesystem Creation (Image)

BitBake then uses these package feeds to construct the **root filesystem (rootfs)** of the embedded Linux image.

- `IMAGE_INSTALL` controls **which packages** go into the image.

 ##  Adding Packages to Your Image with `IMAGE_INSTALL`

You specify which packages should be included in your final image using the `IMAGE_INSTALL` variable.

This is usually done in a configuration file like:

- `conf/local.conf` (for temporary or quick changes)
- or in your **custom image recipe** (`.bb` file)

---

###  Example in `conf/local.conf` or image recipe:

```conf
IMAGE_INSTALL += "busybox dropbear curl nano"
```

This tells Yocto to include the following packages in your root filesystem:

- `busybox` – core Unix utilities
- `dropbear` – lightweight SSH server
- `curl` – command-line tool for HTTP/FTP
- `nano` – terminal-based text editor
 

BitBake pulls these packages from:

- `tmp/deploy/ipk/` (package feed)
 **Filesystem types can be**:

- `ext4`
- `wic`
- `tar.gz`
- etc.

##  Assembling Everything into a Final Bootable Image

This is where Yocto’s **`wic` tool** comes into play — if enabled via the `IMAGE_FSTYPES` variable.

###  Enable `wic` Image Format:
```conf
IMAGE_FSTYPES = "wic ext4"
```

---

###  What Does `wic` Do?

The `wic` tool **combines multiple components** into a single, bootable disk image.

It takes:

- **Bootloader** (e.g., `u-boot`)
- **Linux Kernel** (e.g., `Image`)
- **Root Filesystem** (e.g., `.ext4`)

 And assembles them into a complete bootable image, such as:

- `.wic`
- `.sdimg`
- `.img`

---

###  Example Output Location:
```bash
tmp/deploy/images/<machine>/core-image-minimal-<machine>.wic
```

This `.wic` file can be directly written to an SD card or eMMC for booting your embedded device.

##  What Is the SDK in Yocto?

The **SDK (Software Development Kit)** is a standalone toolchain and environment that allows you to build applications for your embedded image **outside** of Yocto.

###  It includes:
-  Cross-compiler (GCC, binutils)
-  Libraries and headers for the target system
-  Environment setup script to configure the build environment

---

## ⚙️ How SDK Is Generated from Package Feeds

---

###  1. BitBake Builds Packages First

All the packages (`.ipk`, `.deb`, `.rpm`) are built using BitBake recipes:

```bash
bitbake core-image-minimal
```

This stores binary packages in:

```bash
tmp/deploy/ipk/
```

These packages include:

-  Libraries (e.g., `libc`, `libssl`)
-  Binaries (e.g., `busybox`, `dropbear`)
-  Dev files (e.g., `libssl-dev`, `zlib-dev`)

 These **dev packages** are used when generating the SDK.

---

###  2. Generate the SDK

You generate the SDK with the following command:

```bash
bitbake core-image-minimal -c populate_sdk
```

This will:

- Build a cross-toolchain
- Collect all necessary headers and libraries from the package feeds
- Generate an SDK installer `.sh` script
- Include an environment setup script

---

 **SDK Output Location**:

```bash
tmp/deploy/sdk/
└── poky-glibc-x86_64-core-image-minimal-aarch64-toolchain-<date>.sh
```

---
