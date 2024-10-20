# Linux 5.4 and Yocto 3.1

#intro: Linux has been the mainstay of embedded computing for many years

Buildroot: https://youtu.be/R__wCdZ2VXY?si=VinaAMae5ZytL_iB

# Hardware and software

    - BBB board
    - Raspberry board
    - QEMU (32 bit Arm) => install qemu on ubuntu => sudo apt install qemu qemu-system
    - Yocto Project 3.1 => Compatible Linux ditrobution
    - Buildroot 2020.02 LTS
    - Crosstool-NG 1.24.0
    - U-Boot v2021.01
    - Linux Kernel 5.4

# Section 1: Elements of Embedded Linux

## Chapter 1 - Starting out

#intro: sets the scene by describing the embedded Linux ecosystem and the choices available to you as you start your project.

Five elements of embedded linux

- toolchain: the compiler and other tool needed to create code for your target device
- bootloader: the program to initializes boards and loads the linux kernel
- kernel: this is heard of the system, managing system resources and interfacing with hardware
- root filesystem: contains the lib and programs that are run once the kernel has completed its initialization
- Application: That is the collection of programs specific to your embedded application
  Select hardware of embedded linux
- CPU Architecture => memory management units (MMUs).
- need a reasonable amount of RAM
- non-volatile storage, usually flash memory
- a serial port is very useful, preferably a UART-based serial port. - you need some means of loading software when starting from scratch.
  QEMU
- command to get QEMU

```bash
qemu-system-arm -machine vexpress-a9 -m 256M -drive file=rootfs.ext4,sd -net nic -net use -kernel zImage -dtb vexpress- v2p-ca9.dtb -append "console=ttyAMA0,115200 root=/ dev/mmcblk0" -serial stdio -net nic,model=lan9118 -net tap,ifname=tap0

# to create a network interface named tap0
sudo tunctl -u $(whoami) -t tap0
```

## Chapter 2: Learning about toolchains

#intro: describes the components of a toolchain and shows you how to create a toolchain for cross-compiling code for the target board. It describes where to get a toolchain and provides details on how to build one from the source code.

- A tool called `crosstool-NG` to build tollchain for BBB and QEMU
- Technical requirements
  - to install tool:

```bash
sudo apt-get install autoconf automake bison bzip2 cmake flex g++ gawk gcc gettext git gperf help2man libncurses5-dev libstdc++6 libtool libtool-bin make patch python3-dev rsync texinfo unzip wget xz-utils
```

- Building toolchain using crosstoll-NG
  - to install crosstoll-NG

```bash
git clone https://github.com/crosstool-ng/crosstool-ng.git
cd crosstool-ng
git checkout crosstool-ng-1.24.0
./bootstrap
./configure --prefix=${PWD}
make
make install
```

- building a toolchain for QEMU

```bash
bin/ct-ng distclean
bin/ct-ng arm-unknown-linux-gnueabi
bin/ct-ng build
# The toolchain will be installed in ~/x-tools/arm-unknown-linux-gnueabi.
# add to .bashrc: PATH=~/x-tools/arm-cortex_a8-linux-gnueabihf/bin:$PATH
```

A standard GNU toolchain consists of three main components:

- Binutils: A set of binary utilities including the assembler and the linker. It is available at http://gnu.org/software/binutils.
- GNU Compiler Collection (GCC): These are the compilers for C and other languages, which, depending on the version of GCC, include C++, Objective-C, Objective-C++, Java, Fortran, Ada, and Go. They all use a common backend that produces assembler code, which is fed to the GNU assembler. It is available at http://gcc.gnu.org/.
- C library: A standardized application program interface (API) based on the POSIX specification, which is the main interface to the operating system kernel for applications. There are several C libraries to consider, as we shall see later on in this chapter.
- Building U-Boot

```bash
git clone git://git.denx.de/u-boot.git
cd u-boot
git checkout v2021.01
```

## Chapter 3: All about Bootloaders

#intro: explains the role of the bootloader in loading the Linux kernel into memory, and uses U-Boot as an example. It also introduces device trees as the mechanism used to encode the details of the hardware in almost all embedded Linux systems.

## Chapter 4: Configuring and Building the Kernel

#intro: provides information on how to select a Linux kernel for an embedded system and configure it for the hardware within the device. It also covers how to port Linux to the new hardware.

![[Pasted image 20241014212201.png]]

The main directories of interest are the following:

- arch: Contains architecture-specific files. There is one subdirectory per architecture.
- Documentation: Contains kernel documentation. Always look here first if you want to find more information about an aspect of Linux.
- drivers: Contains device drivers – thousands of them. There is a subdirectory for each type of driver.
- fs: Contains filesystem code.
- include: Contains kernel header files, including those required when building the toolchain.
- init: Contains the kernel startup code.
- kernel: Contains core functions, including scheduling, locking, timers, power management, and debug/trace code.
- mm: Contains memory management.
- net: Contains network protocols.
- scripts: Contains many useful scripts, including the device tree compiler (DTC) which I described in Chapter 3, All About Bootloaders.
- tools: Contains many useful tools, including the Linux performance counters tool, perf, which I will describe in Chapter 20, Profiling and Tracing.

Tool to use menuconfig

```bash
sudo apt install libncurses5-dev flex bison
```

## Chapter 5: Building a Root Filesystem

#intro: introduces the ideas behind the user space part of an embedded Linux implementation by means of a step-by-step guide on how to configure a root filesystem.

To make a minimal root filesystem, you need these components:

- init: This is the program that starts everything off, usually by running a series of scripts. I will describe how init works in much more detail in Chapter 13, Starting Up – The init Program.
- Shell: You need a shell to give you a command prompt but, more importantly, also to run the shell scripts called by init and other programs.
- Daemons: A daemon is a background program that provides a service to others. Good examples are the system log daemon (syslogd) and the secure shell daemon (sshd). The init program must start the initial population of daemons to support the main system applications. In fact, init is itself a daemon: it is the daemon that provides the service of launching other daemons.
- Shared libraries: Most programs are linked with shared libraries and so they must be present in the root filesystem.
- Configuration files: The configuration for init and other daemons is stored in a series of text files, usually in the /etc directory. • Device nodes: These are the special files that give access to various device drivers.
- proc and sys: These two pseudo filesystems represent kernel data structures as a hierarchy of directories and files. Many programs and library functions depend on /proc and /sys.
- Kernel modules: If you have configured some parts of your kernel to be modules, they need to be installed in the root filesystem, usually in /lib/modules/ [kernel version].

Root filesystem:

- /bin: Programs essential for all users
- /dev: Device nodes and other special files
- /etc: System configuration files
- /lib: Essential shared libraries, for example, those that make up the C library
- /proc: Information about processes represented as virtual files
- /sbin: Programs essential to the system administrator
- /sys: Information about devices and their drivers represented as virtual files
- /tmp: A place to put temporary or volatile files
- /usr: Additional programs, libraries, and system administrator utilities, in the /usr/bin, /usr/lib, and /usr/sbin directories, respectively
- /var: A hierarchy of files and directories that may be modified at runtime, for example, log messages, some of which must be retained after boot

Structure root filesystem:

![[Pasted image 20241014220340.png]]

File permission access

![[Pasted image 20241014220521.png]]

![[Pasted image 20241015085456.png]]

## Chapter 6: Select a build system

#intro: covers two commonly used embedded Linux build systems, Buildroot and the Yocto Project, which automate the steps described in the previous four chapters.

Requirement

- A linux-based host system with minimum 60GB(90GB with doc on yocto)
- Etcher for linux

```bash
# install etcher
# Download file .deb at https://github.com/balena-io/etcher/releases/
# Open folder contain file .deb
# And open terminal here
sudo dpkg -i balena-etcher_VERSION_amd64.deb
```

- A microSD card reader and card
- A USB to TTL 3.3V serial cable
- A Raspberry 4
- A 5V 3A USB-C power supply
- An Ethernet cable and port for network connectivity
- A BBB
- A 5V 1A DC power supply

Buildroot

- Overview
  - A toolchain
  - A bootloader
  - A kernel
  - A root filesystem
- Steps
  Therefore, to do its job, a build system has to be able to do the following:

1. Download the source code from upstream, either directly from the source code control system or as an archive, and cache it locally.
2. Apply patches to enable cross compilation, fix architecture-dependent bugs, apply local configuration policies, and so on.
3. Build the various components.
4. Create a staging area and assemble a root filesystem.
5. Create image files in various formats, ready to be loaded onto the target.

- Installing
  - Install package dependencies:

```bash
sudo apt update
# build tool
sudo apt install sed make binutils build-essential diffutils gcc g++ bash patch gzip bzip2 perl tar cpio unzip rsync file bc findutils libncurses-dev
# source fetching tool
sudo apt install wget
# link doc: https://buildroot.org/downloads/manual/manual.html#requirement

```

```bash
# clone repo to build
git clone git://git.buildroot.net/buildroot -b 2020.02.9
cd buildroot
```

- Configuring

```bash
make qemu_arm_versatile_defconfig
make -j x
```

- After build
  When it is complete, you will find that two new directories have been created:
  - dl/: This contains archives of the upstream projects that Buildroot has built.
  - output/: This contains all the intermediate and final compiled resources. You will see the following in output/:
  - build/: Here, you will find the build directory for each component.
  - host/: This contains various tools required by Buildroot that run on the host, including the executables of the toolchain (in output/host/usr/bin).
  - images/: This is the most important of all since it contains the results of the build. Depending on what you selected when configuring, you will find a bootloader, a kernel, and one or more root filesystem images.
  - staging/: This is a symbolic link to sysroot of the toolchain. The name of the link is a little confusing because it does not point to a staging area, as I defined it in Chapter 5, Building a Root Filesystem.
  - target/: This is the staging area for the root directory. Note that you cannot use it as a root filesystem as it stands because the file ownership and the permissions are not set correctly. Buildroot uses a device table, as described in the previous chapter, to set ownership and permissions when the filesystem image is created in the image/ directory.
- Running with QEMU

```bash
qemu-system-arm -M versatilepb -m 256 -kernel output/images/zImage \
-dtb output/images/versatile-pb.dtb \
-drive file=output/images/rootfs.ext2,if=scsi,format=raw \
-append "root=/dev/sda console=ttyAMA0,115200" \
-serial stdio -net nic,model=rtl8139 -net user
# create a file run.sh => paste into it
# chmod for run.sh and run it
# login with user `root` with no password
# To close QEMU, either press Ctrl + Alt + 2 to get to the QEMU console and then type quit, or just close the framebuffer window.
```

- Build root for rasp pi 4

```bash
cd buildroot
make clean
make raspberrypi4_64_defconfig
make
```

When the build finishes, the image is written to a file named output/images/ sdcard.img. The post-image.sh script and the genimage-raspberrypi4-64.cfg configuration file used to write the image file are both located in the board/ raspberrypi/ directory. To write sdcard.img onto a microSD card and boot it on your Raspberry Pi 4, follow these steps:

1. Insert a microSD card into your Linux host machine.
2. Launch Etcher.
3. Click Flash from file from Etcher.
4. Locate the sdcard.img image that you built for the Raspberry Pi 4 and open it.
5. Click Select target from Etcher.
6. Select the microSD card that you inserted in Step 1.
7. Click Flash from Etcher to write the image.
8. Eject the microSD card when Etcher is done flashing.
9. Insert the microSD card into your Raspberry Pi 4.
10. Apply power to the Raspberry Pi 4 by way of the USB-C port.

**Yocto Project**
#intro: You can use components from the Yocto Project to design, develop, build, debug, simulate, and test the complete software stack using Linux, the X Window System, GTK+ frameworks, and Qt frameworks.
![[Pasted image 20241014231304.png]]

- Document: https://docs.yoctoproject.org/
  Therefore, the Yocto Project collects several components, the most important of which are as follows: - OE-Core: This is the core metadata, and is shared with OpenEmbedded. - BitBake: This is the task scheduler, and is shared with OpenEmbedded and other projects. - Poky: This is the reference distribution. - Documentation: This is the user's manuals and developer's guides for each component. - Toaster: This is a web-based interface to BitBake and its metadata.
- Package requirements on host

```bash
sudo apt update
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 python3-subunit zstd liblz4-tool file locales libacl1
sudo locale-gen en_US.UTF-8
```

- Installing

![[Pasted image 20241014224757.png]]

```bash
git clone -b $codename git://git.yoctoproject.org/poky.git
# using branch dunfell
```

- Configuring
  As with Buildroot, let's begin with a build for the QEMU Arm emulator

```bash
source poky/oe-init-build-env
# You must source this script each time you want to work on this project.
# You can choose a different working directory by adding it as a parameter to oe-init-build-env
source poky/oe-init-build-env build-qemuarm
# Option: check task scheduler
bitbake -c listtasks core-image-minimal
bitbake -c core-image-minimal
```

Initially, the build directory contains only one subdirectory named conf/, which contains the following configuration files for this project: - local.conf: This contains a specification of the device you are going to build and the build environment. - bblayers.conf: This contains the paths of the meta layers you are going to use. I will describe layers later on.
For now, we just need to set the `MACHINE` variable in /conf/local.conf to `qemuarm` by removing # to uncomment it

- Building
  To actually perform the build, you need to run BitBake, telling it which root filesystem image you want to create. Some common images are as follows: - `core-image-minimal`: This is a small console-based system that is useful for tests and as the basis for custom images. - `core-image-minimal-initramfs`: This is similar to core-imageminimal but built as a ramdisk. - `core-image-x11`: This is a basic image with support for graphics through an X11 server and the xterminal Terminal app. - `core-image-full-cmdline`: This console-based system offers a standard CLI experience and full support for the target hardware.
  When it is complete, you will find several new directories in the build directory, including downloads/, which contains all the source downloaded for the build, and tmp/, which contains most of the build artifacts. You should see the following in tmp/: - work/: This contains the build directory and the staging area for the root filesystem. - deploy/: This contains the final binaries to be deployed on the target: - deploy/images/[machine name]/: Contains the bootloader, the kernel and the root filesystem images ready to be run on the target. - deploy/rpm/: This contains the RPM packages that make up the images. - deploy/licenses/: This contains the license files that are extracted from each package.
  When the build is done, we can boot the finished image on QEMU.
  #TOTO: examine bitbake

To run the QEMU emulation, make sure that you have sourced oe-init-build-env, and then just type this:

```bash
runqemu qemuarm
# In this case, close QEMU using the key sequence Ctrl + A and then x.
```

- #Layers

  - The metadata for the Yocto Project is structured into layers. By convention, each layer has a name beginning with meta. The core layers of the Yocto Project are as follows:
    - meta: This is the OpenEmbedded core and contains some changes for Poky.
    - meta-poky: This is the metadata specific to the Poky distribution.
    - meta-yocto-bsp: This contains the board support packages for the machines that the Yocto Project supports.
  - The list of layers in which BitBake searches for recipes is stored in /conf/bblayers.conf and, by default, includes all three layers mentioned in the preceding list.
  - Link doc: http://layers.openembedded.org/layerindex/branch/master/layers/
  - Adding a layer is as simple as copying the meta directory to a suitable location and adding it to bblayers.conf

- Customizing images via local.conf
  you can simply append to the list of packages to be installed by adding a statement like this:

```
IMAGE_INSTALL_append = " strace helloworld"
```

You can make more sweeping changes via EXTRA_IMAGE_FEATURES. Here is a short list that should give you an idea of the features you can enable:

- dbg-pkgs: This installs debug symbol packages for all the packages installed in the image.
- debug-tweaks: This allows root logins without passwords and other changes that make development easier.
- package-management: This installs package management tools and preserves the package manager database.
- read-only-rootfs: This makes the root filesystem read-only. We will cover this in more detail in Chapter 9, Creating a Storage Strategy.
- x11: This installs the X server.
- x11-base: This installs the X server with a minimal environment.
- x11-sato: This installs the OpenedHand Sato environment.
  Features: https://docs.yoctoproject.org/ref-manual/

- Writing an image recipe
- Creating an SDK

## Chapter 7: Developing with Yocto

#intro: demonstrates how to build system images on top of an existing BSP layer, develop onboard software packages with Yocto's extensible SDK, and roll your own embedded Linux distribution complete with runtime package management.

- Yocto provides Board Support Packages (BSPs) to bootstrap embedded linux
- use `devtool` to rebuilding and redeploying quickly
- `Yocto not only builds Linux images but entire Linux distributions.`
- Content: - Building on top of an existing BSP - Capturing changes with `devtool` - Building your own distro - Provisioning a remote package server
  Started
- Building an existing BSP

```bash
git clone -b dunfell git://git.yoctoproject.org/metaraspberrypi
cd poky
cd  meta/recipes-core/images
ls -l core*
cat core-image-base.bb
cat core-image-minimal.bb
cat core-image-minimal-dev.bb
cd ../../classes
cat core-image.bbclass
# see layer of raspberrypi => how to support wifi and bluetooth is added to core-image-base
cd ../../..
cd meta-raspberrypi/recipes-core/images
ls -l
cat rpi-test-image.bb
cd ../packagegroups
cat packagegroup-rpi-test.bb
cd ../../..
source poky/oe-init-build-env build-rpi
bitbake-layers add-layer ../meta-openembedded/meta-oe
bitbake-layers add-layer ../meta-openembedded/metapython
# add layer python because network and media layer needed this layer
bitbake-layers add-layer ../meta-openembedded/metanetworking
bitbake-layers add-layer ../meta-openembedded/metamultimedia
bitbake-layers add-layer ../meta-raspberrypi
# If bitbake-layers add-layer or bitbake-layers show-layers starts failing due to parse errors, then delete the build-rpi directory and restart this exercise from Step 1.

# Verify all layer have been added to image?
bitbake-layers show-layers

# List the machines supported
ls ../meta-raspberrypi/conf/machine
# add to conf/local.conf => MACHINE = "raspberrypi4-64" => overrides MACHINE ??= "qemux86-64"
# Now, append ssh-server-openssh to the list of EXTRA_IMAGE_FEATURES in your conf/local.conf => EXTRA_IMAGE_FEATURES ?= "debug-tweaks ssh-server-openssh"
bitbake rpi-test-image
# After finished buiding
ls -l tmp/deploy/images/raspberrypi4-64/rpi-test*wic.bz2
# DETAIL at page 204
```

Now, write that image to a microSD card using Etcher and boot it on your Raspberry Pi 4:

1. Insert a microSD card into your host machine.
2. Launch Etcher.
3. Click Flash from file from Etcher.
4. Locate the wic.bz2 image that you built for the Raspberry Pi 4 and open it.
5. Click Select target from Etcher.
6. Select the microSD card that you inserted in Step 1.
7. Click Flash from Etcher to write the image.
8. Eject the microSD card when Etcher is done flashing.
9. Insert the microSD card into your Raspberry Pi 4.
10. Apply power to the Raspberry Pi 4 by way of its USB-C port.
    => Confirm that your Pi 4 booted successfully by plugging it into your Ethernet and observing that the network activity lights blink

- Controlling wifi
- Controlling bluetooth
- Adding a custom layer
- Capturing changes with `devtool`
- Create a new recipe
- Modifying the source built by a recipe
- Upgrading a recipe to a newer version
  #Building your own Distro => https://www.youtube.com/watch?v=R__wCdZ2VXY&t=217s

## Chapter 8: Yocto Under the Hood

#intro: is a tour of Yocto's build workflow and architecture including an explanation of Yocto's unique multi-layer approach. It also breaks down the basics of BitBake syntax and semantics with examples from actual recipe files.

# Yocto - Continue ...

# Section 2: System Architecture and Design Decisions

#intro: we will be able to make informed decisions concerning the storage of programs and data, how to divide work between kernel device drivers and applications, and how to initialize the system

## Chapter 9: Creating a Storage Strategy

#intro: , discusses the challenges created by managing flash memory, including raw flash chips and embedded MMC (eMMC) packages. It describes the filesystems that are applicable to each type of technology.

## Chapter 10: Updating Software in the Feild

#intro:examines various ways of updating the software after the device has been deployed, and includes fully managed Over-the-Air (OTA) updates. The key topics under discussion are reliability and security.

## Chapter 11: Interfacing with Device Drivers

#intro: describes how kernel device drivers interact with the hardware by implementing a simple driver. It also describes the various ways of calling device drivers from user space.

## Chapter 12: Prototying with Breakout Board

#intro: demonstrates how to prototype hardware and software quickly using a pre-built Debian image for the BeagleBone Black together with a peripheral breakout board. You will learn how to read datasheets, wire up boards, mux device tree bindings, and analyze SPI signals.

## Chapter 13: Starting up - The init Program

#intro: explains how the first user space program–init–starts the rest of the system. It describes three versions of the init program, each suitable for a different group of embedded systems, ranging from the simplicity of the BusyBox init, through System V init, to the current state-of-the-art approach, systemd.

## Chapter 14: Starting with BusyBox runit (Buildroot)

#intro: shows you how to use Buildroot to divide your system up into separate BusyBox runit services each with its own dedicated process supervision and logging like that provided by systemd.

## Chapter 15: Managing Power

#intro: considers the various ways that Linux can be tuned to reduce power consumption, including dynamic frequency and voltage scaling, selecting deeper idle states, and system suspend. The aim is to make devices that run for longer on a battery charge and also run cooler.

# Section 2 - Continue ...

# Section 3: Writing Embedded Applications

## Chapter 16: Packaging python

#intro: explains what choices are available for bundling Python modules together for deployment and when to use one method over another. It covers pip, virtual environments, conda, and Docker.

- Retracing the origins of Python packaging
- Installing Python packages with pip
- Managing Python virtual environments with venv
- Installing precompiled binaries with conda
- Deploying Python applications with Docker

`Note: Docker is Tool for building, deploying, and running software inside containers`

- venv

```bash
# install python and venv for ubuntu
sudo apt install python3 python3-venv
```

### Docker

- Getting Docker

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable --now docker
sudo usermod -aG docker <username>
```

- Building docker

```bash
docker images
# fetch and build the dockerfile:
git clone https://github.com/tiangolo/uwsgi-nginxflask-docker.git
cd uwsgi-nginx-flask-docker/docker-images
cp python3.8-alpine.dockerfile Dockerfile
# build an image from dockerfile => docker images
```

- Run a docker image

```bash
docker ps # check status of your running container
docker run -d --name my-container -p 80:80 my-image
docker ps
# to stop container
docker stopmy-container
docker ps
```

- Fetching a docker image

```bash
docker pull <image_name>
docker images
docker run -d --name flask-container -p 80:80 <image-name>

```

- Pushing a docker image

```bash
docker login
docker tag my-image:latest <repo>/my-image:latest
docker push <repo>/my-image:latest
```

- Clenaing up

```bash
docker ps -a
docker stop flask-container
docker rm flask-container
# to get id of docker image:
docker images
# delete docker image
docker rmi <image_id>
docker system prune -a
```

### Venv

- Managing python with virtual environments with venv

```bash
# add venv to project folder
python3 -m venv venv
# active venv
source ./venv/bin/activate
# deactive
deactivate
```

### Conda

- Installing precompiled binaries with conda

```bash
# 1. Download Miniconda:
wget https://repo.anaconda.com/miniconda/Miniconda3- latest-Linux-x86_64.sh
# 2. Install Miniconda:
bash Miniconda3-latest-Linux-x86_64.sh
# 3. Update all the installed packages in the root environment: (base)
conda update --all
conda env list # check
conda create --name py377 python=3.7.7 # create other python version and named is py377
source activate py377 # active python 3.7.7 for project folder
conda deactivate # deactive venv if project folder using conda
# eg: You install py277 and py377 and don't use py277
conda remove --name py27 -all # remove py277 using conda
# install package using conda
conda install <package_name>
# update package using conda
conda update <package_name>
# remove package using conda
conda remove <package_name>
# to check info of package using conda
conda info <package_name>
# extras: pahe 555


# export your active environment to a yaml file:
conda env export > my_environment.yaml
conda env create -f my-environment.yaml
```

## Chapter 17: Learning about Processes and Thread

#intro: describes embedded systems from the point of view of the application programmer. This chapter looks at processes and threads, inter-process communications, and scheduling policies.

## Chapter 18: Managing Memory

#intro: introduces the ideas behind virtual memory and how the address space is divided into memory mappings. It also describes how to measure memory usage accurately and how to detect memory leaks.

- A Linux-based host system with gcc, make, top, procps, valgrind, and smem installed

```bash
# check memory of the kernel
CROSS_TOOL vmlinux
cat /proc/meminfo
```

# Section 4: Debugging and Optimizing Performance

#intro: describes how to trace, profile, and debug your code in both the applications and the kernel. The last chapter explains how to design for real-time behavior when required. Detect problems and indentify bottlenecks - debugging: use VSCODE for remote debugging using GDB

## Chapter 19: Debugging with GDB (GNU Project Debugging)

#intro: shows you how to use the GNU debugger, GDB, together with the debug agent, gdbserver, to debug applications running remotely on the target device. It goes on to show how you can extend this model to debug kernel code, making use of the kernel debug stubs with KGDB.

- The GNU debugger
  GCC offers two options for debug symbol => `-g` and `-ggdb`
  Level 0 => 3 - 0 <=> no debug - 1 <=> produces minimal info, includes function names and external variables - 2 <=> This is default and includes info about local variables and line numbers so that you can perform source-level debugging and single-step through the code - 3 <=> This includes extra information which, among other things, means that GDB can handle macro expansions correctly
- Preparing to debug
- Debugging applications
  Setting up the Yocto Project for remote debugging
  include `gdbserver` in the target image => add in `conf/local.conf`

```
IMAGE_INSTALL_append = " gdbserver"
EXTRA_IMAGE_FEATURES ?= "ssh-server-openssh"
```

Alternatively, you can add tools-debug to EXTRA_IMAGE_FEATURES, which will add `gdbserver`, `native gdb`, and `strace` to the target image (I will talk about `strace` in chapter 20)

```
EXTRA_IMAGE_FEATURES ?= "tools-debug ssh-server-openssh"
```

For the second part, you just need to build an SDK as I described in Chapter 6, Selecting a Build System
#note: examine chapter 6 => SDK

```bash
bitbake -c populate_sdk <image>
```

Setting up Buildroot for remote debugging (606)

VISUAL STUDIO CODE

```bash
sudo snap install code --classic
# install toolchain
# clone repo yocto is poky
source poky/oe-init-build-env build-rpi
```

Edit conf/local.conf

```
MACHINE ?= "raspberrypi4-64"
IMAGE_INSTALL_append = " gdbserver"
EXTRA_IMAGE_FEATURES ?= "ssh-server-openssh debug-tweaks"
```

```bash
bitbake core-image-minimal-dev
# lastly => build SDK
bitbake -c populate_sdk core-image-minimal-dev
# Find the SDK installer in tmp/deploy/sdk and run it
./poky-glibc-x86_64-core-image-minimal-dev-aarch64- raspberrypi4-64-toolchain-3.1.5.sh
./opt/poky/3.1.5/environment-setup-aarch64-poky-linux
# install Cmake
sudo apt update
sudo apt install cmake
# started with vscode
mkdir hellogdb
cd hellogdb
mkdir src build
code .
# install some extensions
# C/C++ by Microsoft
# CMake by twxs
# CMake Tools by Microsoft
```

Configuring CMake We need to populate CMakeLists.txt and cross.cmake to cross-compile the hellogdb project with our toolchain:

1. First, copy MELP/Chapter19/hellogdb/CMakeLists.txt to the hellogdb project folder in your home directory.
2. From inside Visual Studio Code, click on the Explorer icon in the upper-left corner of the Visual Studio Window to open the Explorer sidebar.
3. Click on CMakeLists.txt in the Explorer sidebar to view the file's contents. Notice that the project's name is defined as HelloGDBProject and the IP address of the target board is hardcoded as 192.168.1.128.
4. Change that to match the IP address of your Raspberry Pi 4 and save the CMakeLists.txt file.
5. Expand the src folder and click on the New File icon in the Explorer sidebar to create a file named main.c inside the hellogdb project's src directory
6. In file main.c

```C
#include <stdio.h>

int main() {
	printf("Hello Cmake\n");
	return 0;
}
```

7. Copy MELP/Chapter19/hellogdb/cross.cmake to the hellogdb project folder in your home directory.
8. Lastly, click on cross.cmake in the Explorer sidebar to view the file's contents. Notice that the sysroot*target and tools paths defined in cross.cmake point to the /opt/poky/3.1.5 directory where we installed the SDK. Also notice that the values of the CMAKE_C_COMPILE, CMAKE_CXX_COMPILE and CMAKE* CXX_FLAGS variables were derived directly from the environment setup script included with the SDK.

continue... (page 629(657))

- Just-in-time debugging
- Debugging forks and threads
- Core files
- GDB user interfaces
- Debugging kernel code

## Chapter 20: Profiling and Tracing

#intro: covers the techniques available to measure the system performance, starting from whole system profiles and then zeroing in on particular areas where bottlenecks are causing poor performance. It also describes how to use `Valgrind` to check the correctness of an application's use of thread synchronization and memory allocation.

Technical requirements

- A linux base host system
- Buildroot 2020.02.9 LTS release
- Etcher for linux
- A raspberry pi 4
- A 5V 3A USB-C power supply
- An Ethernet cable and port for network connectivily

In this chapter, we will cover the following topics:
• The observer effect

- examine tools: `perf`, `Ftrace`, `LLTng`, and `BPF`
  • Beginning to profile
- started with simple tool => `top`, can use `htop ` to replace `top`, htop has a nicer user interface

# Profiling

Profiling with `top`, `htop`
![[Pasted image 20241017102621.png]]

- At line 2 (busybox) and line 3 (procps)
- Press H to see summary of top (busybox) and press 1 with procps

### The poor man's profiler

- You can profile an application just by using GDB to stop it at arbitrary intervals to see what it is doing. This is the poor man's profiler
- The procedure is simple:

1. Attach to the process using gdbserver (for a remote debug) or GDB (for a native debug). The process stops.
2. Observe the function it stopped in. You can use the backtrace GDB command to see the call stack.
3. Type continue so that the program resumes.
4. After a while, press Ctrl + C to stop it again, and go back to step 2.
   • Introducing `perf` (performance event counter subsystem)
   Link doc: https://perfwiki.github.io/main/
   ![[Pasted image 20241017104707.png]]

## Building with Yocto

Configuring the kernel for perf

- That is `CONFIG_PERF-EVENTS`
- profile using tracepoints-more on this subject later => `CONFIG_DEBUG_INFO`
  Config it in `conf/local.conf` (needed)

```
EXTRA_IMAGE_FEATURES = "debug-tweaks dbg-pkgs tools-profile" IMAGE_INSTALL_append = "kernel-vmlinux"
```

## Building perf with buildroot

=> run menuconfig
Select:

- BR2_LINUX_KERNEL_TOOL_PERF in Kernel | Linux Kernel Tools
  To build packages with debug symbols and install them unstripped on the target, select these two settings:
- BR2_ENABLE_DEBUG in the Build options | build packages with debugging symbols menu
- BR2_STRIP = none in the Build options | strip command for binaries on target menu

# Tracing

### Tracing events

    • A timestamp
    • Context, such as the current PID
    • Function parameters and return values
    • A callstack

Intro about Ftrace
Preparing to use Ftrace
Ftrace and its various options are configured in the kernel configuration menu. You will need the following as a minimum:

- CONFIG_FUNCTION_TRACER from the Kernel hacking | Tracers | Kernel Function Tracer menu
  For reasons that will become clear later, you would be well advised to turn on these options as well:
- CONFIG_FUNCTION_GRAPH_TRACER in the Kernel hacking | Tracers | Kernel Function Graph Tracer menu
- CONFIG_DYNAMIC_FTRACE in the Kernel hacking | Tracers | enable/disable function tracing dynamically menu
  Using Ftrace

```bash
# mount the `debugfs`
# All the controls for Ftrace are in the /sys/kernel/debug/tracing directory
mount -t debugfs none /sys/kernel/debug
# list traces
cat /sys/kernel/debug/tracing/available_tracers
# enable tracing
echo function > /sys/kernel/debug/tracing/current_tracer
echo 1 > /sys/kernel/debug/tracing/tracing_on
sleep 1 # delay 1s
echo 0 > /sys/kernel/debug/tracing/tracing_on
```

=> Dynamic Ftrace and trace filters

- Using LTTng => LTT (Linux Trace Toolkit)
  LTTng and the Yocto Project
  LTTng and Buildroot
  Using LTTng for kernel tracing
- Using BPF (Berkeley Packet Filter)
  Configuring the kernel for BPF with buildroot
  Build a BCC toolkit with Buildroot
  => Using BPF tracing tools
- Using Valgrind - Callgrind - Helgrind
- Using `strace`

## Chapter 21: Real-Time programming

Technical requirements

```
A linux-based host (60gb)
Buildroot 2020.02.9 LTS release
Yocto 3.1 Dunfell LTS release => build kernel real time
Etcher for linux
A microSD
BBB (or Ras)
```

https://triplehelix-consulting.com/yocto-full-demo-gui-raspberry-pi-detailed-manual/
