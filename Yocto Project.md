# Step build kernel with yocto project

## Installing

- Step 1

```bash
sudo apt update
sudo apt install gawk wget git diffstat unzip texinfo gcc build-essential chrpath socat cpio python3 python3-pip python3-pexpect xz-utils debianutils iputils-ping python3-git python3-jinja2 python3-subunit zstd liblz4-tool file locales libacl1
sudo libegl1-mesa libsdl1.2-dev pylint3 xterm python3-subunit mesa-common-dev
sudo locale-gen en_US.UTF-8
```

Note: If your build system has the oss4-dev package installed, you might experience QEMU build failures due to the package installing its own custom /usr/include/linux/soundcard.h on the Debian system. If you run into this situation, try either of these solutions:

```bash
# Steps to reproduce:
# 1. Install Ubuntu 20.04 System on an LPAR|z/VM guest|KVM guest from ports.ubuntu.com
# 2. Uncomment all #deb-src lines in /etc/apt/sources.list
#3. Run the following commands:
sudo apt update
sudo apt install build-essential
apt source qemu
sudo apt build-dep qemu
sudo apt remove oss4-dev
```

```bash
sudo apt install git make inkscape texlive-latex-extra
sudo apt install python3-sphinx python3-sphinx-rtd-theme

```

```bash
git clone -b dunfell git://git.yoctoproject.org/poky.git
# using branch dunfell
```

```bash
source poky/oe-init-build-env
# You must source this script each time you want to work on this project.
# You can choose a different working directory by adding it as a parameter to oe-init-build-env
source poky/oe-init-build-env build-qemuarm # using with qemu
# Option: check task scheduler
```

- In file conf/local.conf => edit `MACHINE ?= "qemuarm"` => to runqemu

```
Uncomment in file conf/local.conf to improve speed build
BB_HASHSERVE_UPSTREAM = "wss://hashserv.yoctoproject.org/ws"
SSTATE_MIRRORS ?= "file://.* http://cdn.jsdelivr.net/yocto/sstate/all/PATH;downloadfilename=PATH"
BB_HASHSERVE = "auto"
BB_SIGNATURE_HANDLER = "OEEquivHash"
```

Issue with proxy
Link doc: https://wiki.yoctoproject.org/wiki/Working_Behind_a_Network_Proxy

```bash
bitbake -c listtasks core-image-minimal
bitbake core-image-minimal
```

```bash
# run on qemu
runqemu qemuarm
# if not config MACHINE in file conf/local.conf
runqemu qemux86_64
```

## Configuring tool

- Config layer => in file /conf/bblayers.conf

Add layer raspberry

```

```
