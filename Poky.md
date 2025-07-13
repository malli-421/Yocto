Downloading Poky Reference Distribution
---
The Poky source code is the Yocto Project's official reference distribution. It provides the foundation for building embedded Linux systems.
To start using Yocto:
```
git clone git://git.yoctoproject.org/poky
cd poky
git checkout scarthgap
```
- Poky is the reference distro and contains:
  - BitBake
  - OpenEmbedded-core
  - Starter metadata and example images
- From here, you can add layers and start building.

Installing Build Host Packages
---
To use Yocto, your Linux development host (like Ubuntu) must have a set of required tools and libraries installed.
- Missing dependencies will cause build errors, such as failing to fetch, compile, or patch source code.
- Installation Command (for Ubuntu):
```
sudo apt update
sudo apt install gawk wget git-core diffstat unzip texinfo gcc \
     build-essential chrpath socat cpio python3 python3-pip \
     python3-pexpect xz-utils debianutils iputils-ping \
     libsdl1.2-dev xterm
```
- This installs tools for:
    - Compilation
    - Network access
    - Script automation and unpacking
    - Cross-build support and scripting
  

Setup Build Environment
---

- we are ready to setup the build environment:
```
source oe-init-build-env
    or
.  oe-init-build-env
```
- It does 3 major things:
  1. Setup the Build Environment Variables
  2. Creates a Build Directory
  3. Make Tools Available

Try to Build the One Package
---
<img src="/Screenshot 2025-07-13 102745.png" alt="Poky.md" width="400"/>











