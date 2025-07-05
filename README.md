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
    2. Explain the Architecture of Yocto Project?
    3. What are layers in Yocto?
---


## Yocto Project Architecture

<prev>
        ![Screenshot 2025-07-03 154231](https://github.com/user-attachments/assets/1a47ef0a-3bc5-4fcd-9756-79a2a123ae4c)

I. Inputs to the OpenEmbedded Build System
---
---
    The OpenEmbedded build system, represented by a blue rectangle, takes several inputs, depicted as brown rectangles:
---
• User Configuration: These are conf files that users configure to set parameters like the machine type for the build and the layers to be included.
• Metadata: This encompasses all the layers and recipe files contained within those layers.
◦ Recipe files describe how BitBake executes tasks for each software package, such as fetching, unpacking, patching, compiling, and installing.
• Machine Configuration: Describes everything related to the platform used in the build and compiler tuning options. Examples include:
          ◦ Architecture
          ◦ Device tree
          ◦ Kernel command line arguments
          ◦ Kernel config file
• Policy Configuration: Customizable settings to change how the build system behaves.
•Source Mirrors: Sources from where the build system retrieves source code. These can be Git repositories, tarball archives, or local sources. The build system downloads software source packages from these mirrors based on configuration.

--------------------------------------------------------------------------------

II. OpenEmbedded Build System Tasks (Inside the Blue Box)
---
---
    Inside the OpenEmbedded build system, various tasks are executed sequentially for each software package required to build an image or SDK. These tasks are depicted as gray boxes:
---

1. Source Fetching:
      ◦ Described in the metadata using Python and Bash scripts within recipes and patches.
      ◦ Downloads or clones source code from specified source mirrors.
      ◦ Archives and Git repositories are stored in a downloads folder.
2. Unpack and Extract:
      ◦ After fetching, sources are unpacked and extracted into different folders.
3. Patch Application:
      ◦ Any modifications to software package sources are typically done via patches for ease of portability and application.
      ◦ The OpenEmbedded system applies all instructed patches at this stage. Patches are often created to migrate software to a specific platform or resolve build errors.
4. Configuration and Compilation:
      ◦ Compilation is typically performed by GCC.
      ◦ If other build systems (e.g., GNU Autoconf) are used, extra configuration is done before compilation to generate makefiles.
      ◦ Compilation then proceeds using make commands and GCC.
      ◦ Binary outputs are deployed to a separate folder.
5. Output Analysis:
      ◦ OpenEmbedded gathers binary outputs and prepares files for package generation.
6. Package Generation:
      ◦ Depending on the desired package manager output, packages are generated (e.g., RPM generation, Deb package outputs).
7. Quality Assurance Checks:
      ◦ Applied to the generated packages and the binaries within the software package archive.
8. Software Package Outputs (First Stage Output):
      ◦ If all checks are complete, the build system produces software package outputs, represented as a green box.
      ◦ This is the first stage of OpenEmbedded's capability, where it creates package feeds or archives containing pre-compiled binaries for a specific platform using configuration files and metadata.
      ◦ These package archives can be copied to an already running Linux system of the same architecture and installed using package manager commands.












