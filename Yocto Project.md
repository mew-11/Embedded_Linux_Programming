# Step build kernel with yocto project

## Installing

- Step 1

```bash
sudo apt update
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 python3-subunit zstd liblz4-tool file locales libacl1 libegl1-mesa libsdl1.2-dev pylint xterm mesa-common-dev vim bzip2
sudo locale-gen en_US.UTF-8
```

```bash
git config --global https.version HTTP/1.1
```

Note: If your build system has the oss4-dev package installed, you might experience QEMU build failures due to the package installing its own custom /usr/include/linux/soundcard.h on the Debian system. If you run into this situation, try either of these solutions:

````bash
# Steps to reproduce:
# 1. Install Ubuntu 20.04 System on an LPAR|z/VM guest|KVM guest from ports.ubuntu.com
# 2. Uncomment all #deb-src lines in /etc/apt/sources.list
#3. Run the following commands:
sudo apt update
sudo apt install build-essential
apt source qemu
sudo apt build-dep qemu
sudo apt remove oss4-dev
`` it`

- package for python

```bash
sudo apt install git make inkscape texlive-latex-extra
sudo apt install python3-sphinx python3-sphinx-rtd-theme

````

```bash
mkdir ~/yocto_build
git clone -b dunfell git://git.yoctoproject.org/poky.git ~/yocto_build
```

```bash
mkdir ~/yocto_build/layer
cd ~/yocto_build/layer
git clone -b dunfell git://git.openembedded.org/metaopenembedded 
git clone -b dunfell git://git.yoctoproject.org/metaraspberrypi

```

```bash
cd ~/yocto_build
source poky/oe-init-build-env
# You must source this script each time you want to work on this project.
# You can choose a different working directory by adding it as a parameter to oe-init-build-env
source poky/oe-init-build-env build-qemuarm # using with qemu
# Option: check task scheduler
```

- In file conf/local.conf => edit `MACHINE ?= "qemuarm"` => to runqemu

```
- Adjust MACHINE
MACHINE ?= "qemuarm"
- Uncomment in file conf/local.conf to improve speed build
INHERIT += "rm_work"
BB_HASHSERVE = "auto"
BB_SIGNATURE_HANDLER = "OEEquivHash"
```

```bash
# using commands
echo "MACHINE = \"raspberrypi4\"" >> conf/local.conf
```

```
In file bblayers.conf

BBLAYERS ?= " \
  /home/user/yocto/poky/meta \
  /home/user/yocto/poky/meta-poky \
  /home/user/yocto/meta-openembedded/meta-oe \
  /home/user/yocto/meta-yocto-bsp \
"
```

Note: using commands `bitbake` to add Layer. There are using for raspberry

```bash
bitbake-layers add-layer ${HOME}/yocto_build/layer/meta-openembedded/meta-oe
bitbake-layers add-layer ${HOME}/yocto_build/layer/meta-openembedded/meta-python
bitbake-layers add-layer ${HOME}/yocto_build/layer/meta-openembedded/meta-multimedia
bitbake-layers add-layer ${HOME}/yocto_build/layer/meta-openembedded/meta-networking
bitbake-layers add-layer ${HOME}/yocto_build/layer/meta-raspberrypi
```

```bash
# to check the Layer
bitbake-layers show-layers
cat conf/bblayers.conf
# list Machine BSP layer raspberry support:
ls ../meta-raspberrypi/conf/machine

```

```bash
bitbake -c listtasks core-image-minimal
# bitbake rpi-test-image --runall=fetch
bitbake core-image-minimali
```

- append ssh-server-openssh

```
EXTRA_IMAGE_FEATURES ?= "debug-tweaks ssh-server-openssh"
```

- if you using QEMU

```bash
# run on qemu
runqemu qemuarm nographic
# if not config MACHINE in file conf/local.conf
runqemu qemux86_64 nographic
```

- if you using raspberry
  Once the image has finished building, there should be a file named rpi-testimage-raspberrypi4-64.rootfs.wic.bz2 in the tmp/deploy/images/raspberrypi4-64 directory:
  Now, write that image to a microSD card using Etcher and boot it on your Raspberry Pi 4:

1. Insert a microSD card into your host machine.
2. Launch Etcher.
3. Click Flash from file from Etcher.
4. Locate the wic.bz2 image that you built for the Raspberry Pi 4 and `bzip2` it.
5. Click Select target from Etcher.
6. Select the microSD card that you inserted in Step 1.
7. Click Flash from Etcher to write the image.
8. Eject the microSD card when Etcher is done flashing.
9. Insert the microSD card into your Raspberry Pi 4.
10. Apply power to the Raspberry Pi 4 by way of its USB-C port.
    Confirm that your Pi 4 booted successfully by plugging it into your Ethernet and observing that the network activity lights blink.

## Configuring tool

### Debugging with yocto

```
In file conf/local.conf
IMAGE_INSTALL_append = " gdbserver"
EXTRA_IMAGE_FEATURES ?= "tools-debug ssh-server-openssh"
```

Note: SDK

### Debugging with VSCODE using yocto

```
In file conf.local.conf
EXTRA_IMAGE_FEATURES ?= "tools-debug ssh-server-openssh"
```

```bash
bitbake core-image-minimal-dev
```

- After build:

  1. Use Etcher to write the resulting core-image-minimal-devraspberrypi4-64.wic.bz2 image from tmp/deploy/images/raspberrypi4-64/ to a microSD card and boot it on your Raspberry Pi 4.
  2. Plug your Raspberry Pi 4 into your local network over Ethernet anduse arp-scan to locate the IP address of your Raspberry Pi 4. We will need this IP address later when Configuring CMake for remote debugging.

  3. Lastly, build the SDK:

```bash
 bitbake -c populate_sdk core-image-minimal-dev
```

- debug on VSCODE

```bash
# install dependencies
sudo apt-get -y update
sudo apt-get -y install build-essential gdb gdb-multiarch git
# install ext
code --install-extension ms-vscode.cpptools
```

```bash
# start debug with a folder
$ mkdir ~/var-hello-world
$ cd ~/var-hello-world
$ code .
```

Next, create the following files:

- main.cpp: Source code for the demo program hello.bin
- Makefile: Makefile to cross compile main.cpp to hello.bin
- .vscode/settings.json: VS Code file to configure global environment variables for the SDK/toolchain
- .vscode/tasks.json: VS Code file to override or add new tasks. Runs Makefile when VS Code build command is executed.
- .vscode/c_cpp_properties.json: VS Code file to configure include path for IntelliSense.

```main.cpp
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Hello, World!\n");
    return 0;
}
```

```Makefile
all: main.cpp
	$(CXX) $(CXXFLAGS) -Og main.cpp -g -o hello.bin
clean:
	rm -f hello.bin
```

```settings.json
{
	"VARISCITE": {
		/* Target Device Settings */
		"TARGET_IP":"192.168.0.141",

		/* Project Settings */
		"PROGRAM":"hello.bin",

		/* Yocto SDK Configuration */
		"ARCH":"aarch64-fslc-linux",
                "OECORE_NATIVE_SYSROOT":"/opt/fslc-xwayland/3.1/sysroots/x86_64-fslcsdk-linux",
                "SDKTARGETSYSROOT": "/opt/fslc-xwayland/3.1/sysroots/aarch64-fslc-linux",

		/* Yocto SDK Constants */
                "CC_PREFIX": "${config:VARISCITE.OECORE_NATIVE_SYSROOT}/usr/bin/${config:VARISCITE.ARCH}/${config:VARISCITE.ARCH}-",
		"CXX": "${config:VARISCITE.CC_PREFIX}g++ --sysroot=${config:VARISCITE.SDKTARGETSYSROOT}",
		"CC": "${config:VARISCITE.CC_PREFIX}gcc --sysroot=${config:VARISCITE.SDKTARGETSYSROOT}",
	}
}
```

```task.json
{
    "version": "2.0.0",
    /* Configure Yocto SDK Constants from settings.json */
    "options": {
        "env": {
            "CXX": "${config:VARISCITE.CXX}",         /* Used by Makefile */
            "CC": "${config:VARISCITE.CC}",           /* Used by Makefile */
        }
     },
     /* Configure integrated VS Code Terminal */
     "presentation": {
        "echo": false,
        "reveal": "always",
        "focus": true,
        "panel": "dedicated",
        "showReuseMessage": true,
    },
    "tasks": [
        /* Configure Build Task */
        {
            "label": "build",
            "type": "shell",
            "command": "make clean; make -j$(nproc)",
            "problemMatcher": ["$gcc"]
        },
    ]
}
```

```c_cpp_properties.json
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",
                "${config:VARISCITE.SDKTARGETSYSROOT}/usr/include/**"
            ]
        }
    ],
    "version": 4
}
```

```bash
scp hello.bin root@<ip addr>:/home/root

```

Launch hello.bin on the target device:

```bash
./hello.bin

```

- details: https://variwiki.com/index.php?title=Yocto_Programming_with_VSCode

### Perf

```
EXTRA_IMAGE_FEATURES = "debug-tweaks dbg-pkgs tools-profile"
IMAGE_INSTALL_append = "kernel-vmlinux"
```

### LTTng

```
IMAGE_INSTALL_append = " lttng-tools lttng-modules lttng-ust"
```

### Note

- Extra strace, ftrace and BPF
