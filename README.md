## Yocto Project

==> Role of Yocto Project
- The Yocto Project is a toolset and framework
- The Yocto Project is a open_source Collabrative project, that helps developers build custom Linux-based operating systems for embedded systems (like smart meters, routers, automotive devices, industrial machines, etc.).

==> Core Components and Terminology

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

--> Source code: This is the raw software you want to build into your embedded Linux images.
--> OpenEmbedded: This is the **build system** and the primary software ecosystem used by the Yocto Project to generate embedded Linux distributions.
OpenEmbedded consists of a collection of **scripts and Python libraries** that manage operations on software sources and packages for embedded Linux development.
--> BitBake: This is the **build engine** that powers the OpenEmbedded build system.
--> Poky: Poky is the reference build system provided by the Yocto Project. It includes everything needed to build a custom embedded Linux OS.
--> Custom Layer: This is where your own project code goes.





















