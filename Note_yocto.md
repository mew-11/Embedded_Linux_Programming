# Summary

1. Basic Yocto Tool

- `Bitbake`, which is task scheduler and executor. `Bitbake` downloads all the sources based on `Metadata` in the form of Layers and Recipes.

2. Kernel

- It is a single binary => to make a workable OS

3. Recipes

- All these hundreds of targets need different Recipes (.bb and .bbappend files).
- They are separated into Layers

4. Metadata

- This is the collection of Recipes, Configuration (.conf) files, and Class files, which include the information for Layers and dependencies, and the git/other repositories links from where to download everything, including the kernel.
- The Metadata also contains a Toolchain and Image definition.

5. Poky

- Poky: To write all this by yourself is a very long and tough process, which is why Yocto has a reference distribution of the whole toolchain and the basic metadata, called Poky.

6. OpenEmbedded

- OpenEmbedded (OE) is a collection of BitBake recipes for thousands of open-source modules, separated into Layers, also referred to as a framework (practically, a super-library). Poky contains at least the OpenEmbedded-core layer by default.

7. BSP Layer

- BSP stands for Board Support Package – the metadata for your HW platform. It includes target CPU architecture, memory, storage definition, peripherals description (like , e.g., Bluetooth, card reader, USB controller on the board, etc.). In our case, we use the meta-raspberrypi, as this is our target. However, if we want, we can relatively easily switch to another platform with similar characteristics by changing the BSP layer and modifying some general project configurations.

8. Target, Tasks, And Steps

Targets, tasks, and steps. As the tasks that BitBake manages are many (we will investigate them later), we have many intermediate steps with intermediate targets. The word target may be a bit confusing. It is used for: the target HW/platform, the target architecture, the target of a task, a build target, etc. If you are new to these concepts, know that each step has a target.

Following is a generic description of the most important build steps performed by a fully configured Yocto project. It is from the very beginning until outputting the final image to be flashed on the target HW, considering that any build step is building for the target CPU architecture:

    1. Based on the metadata BitBake first fetches the sources of all the SW (GNU tools, libraries, applications, the kernel, and all their dependencies).
    2. It then sequentially starts building the dependencies in the correct order, so they are ready for the other SW to be built.
    3. Then it builds the common .so libraries and tools, the higher-level .so libraries and the user applications, and also the kernel.
    4. All the previously built packages are kept in a strict hierarchical structure, so that BitBake knows where to find them.
    5. Now BitBake starts gathering all the binaries to be put into the image.
    6. It finally prepares the image to be flashed.
    7. As the whole process is long, if an error occurs, when restarted it always tries to continue from where it stopped the last time.

9. Host Machine

- It is the PC or server, on which the Yocto project is running when compiling a target image

# Installing and Building

1. Requirement

- On arch

```bash
sudo pacman -S git tar python gcc make chrpath cpio diffstat patch rpcsvc-proto
```

- On other distribution: https://docs.yoctoproject.org/ref-manual/system-requirements.html

2. Yocto Versions And Information, How To Know What Kernel Version Do We Build From Poky

- In directory `metadata` (Kirkstone branch)
  metadata
  ├─ oe
  ├─ poky
  ├─ rpi
  └─ thc
  - oe are all recipes, coming from the OE core for application SW and modules.
  - poky are all recipes, coming from the Poky reference distribution.
  - rpi are all recipes, related to the Raspberry PI.
  - thc are all our custom recipes.

3. The Bitbake build steps

- Part 1:
  - Step 1: `do_fetch`
  - Step 2: `do_unpack`
  - Step 3: `do_patch`
  - Step 4: `do_configure`
  - Step 5: `do_compile`
- Part 2:
  - Step 1: `do_install`
  - Step 2: `do_package`
  - Step 3: `do_package_qa`
  - Step 4: `do_depoly`
  - Step 5: `do_populate_sysroot`
  - Step 6: `do_rootfs`
  - Step 7: `do_build`

# Explain contents of file in yocto project

```bash
cd meta/recipes-core/images
ls -l core* # show all recipes
cat core-image-base.bb
```

```BitBake
SUMMARY = "A console-only image that fully supports the
target device \
hardware."
IMAGE_FEATURES += "splash"
LICENSE = "MIT"
inherit core-image
```

=> This recipe inherits from `core-image` => import the contents of `core-image.bbclass`

```bash
cd ../../classes
cat core-image.bbclass
```
