# Buildroot Step

Step 1: install toolchain

```bash
# build tool
sudo apt install sed make binutils build-essential diffutils gcc g++ bash patch gzip bzip2 perl tar cpio unzip rsync file bc findutils libncurses-dev wget libncurses5-dev flex bison
# source fetching tool
sudo apt install wget
# link doc: https://buildroot.org/downloads/manual/manual.html#requirement
```

Step 2: clone repo to build

```bash
git clone git://git.buildroot.net/buildroot -b 2020.02.9
cd buildroot
```

Step 3: configuring

```bash
make clean # clean buildroot
make qemu_arm_versatile_defconfig # build for qemu
make raspberrypi4_64_defconfig # build for raspberry pi 4
make raspberrypi3_64_defconfig # build for raspberry pi 3
make -j(x)
# x is the number of cores
```

Option: Run on QEMU

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

Option: Build root for rasp pi 4
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

# Conifguring Buildroot

## Enable openssh

```bash
touch ssh.sh
chmod 755 ssh.sh
cat <<EOF > ssh.sh
#!/bin/bash
if grep -q "^#.*BR2_PACKAGE_OPENSSH=y" ".config"; then
   sed -i 's/^#\(.*BR2_PACKAGE_OPENSSH=y\)/\1/' ".config"
   echo "removing comment."
elif grep -q "^BR2_PACKAGE_OPENSSH=y" ".config"; then
   echo "enabled."
else
   echo "BR2_PACKAGE_OPENSSH=y" >>".config"
   echo "Added"
fi
EOF
./ssh.sh
```

## Remote debugging using gdbserver

```bash
make menuconfig
```

- Toolchain
  - Build cross gdb for the host
  - Build cross gdb server for the target
- Target | Debugging, profiling amd benchmark | gdb
- Target | Debugging, profiling amd benchmark | gdbserver
- Build options | build packages with debugging symbols

### Step debugging

- Connect GDB and gdbserver

```bash
# On target
gdbserver :1234 ./program # for QEMU
# :1234 is the port number
gdbserver :1234 ./program # for raspberry pi 4
# On host
# copy GDB from your toolchain
aarch64-poky-linux-gdb program
# giving it the IP address or hostname of the target and the port it is waiting on (eg. IP is 192.168.1.128 and port is 1234)
(gdb) target remote 192.168.1.128:1234

# Using for a serial connection
# On target, you tell gdbserver which serial port to use
gdb /dev/ttyO0 ./program
# configure the port baud rate beforehand using
stty -F /dev/ttyO0 115200
# On the host
(gdb) set serial baud 115200
(gdb) target remote /dev/ttyUSB0
```

### Setting the sysroot

TODO: add more info

```bash
# On host
(gdb) set sysroot /home/chris/buildroot/output/staging
(gdb) show directories
aarch64-poky-linux-objdump --dwarf ./program | grep DW_AT_comp_dir
```

## Configuring perf

```bash
make menuconfig
```

- Kernel
  - Linux Kernel Tools
- Build options
  - build packages with debugging symbols
  - strip command for binaries on target

## Configuring Ftrace

```bash
make menuconfig
```

- Kernel Hacking
  - Traces
    - Kernel Function Tracer
    - Kernel Function Graph Tracer
    - enable/disable function tracing dynamically

## Configuring LTTng

```bash
make menuconfig
```

- Kernel Hacking
  - Traces
    - Kernel Function Tracer
- Target packages
  - Debugging, profiling and benchmark
    - lttng-modules
    - lttng-tools
  - Libraries
    - Other, enable lttng-libust

## Configuring BPF

```bash
make menuconfig
```

Before configuring our kernel for BPF, let's select an external toolchain and modify
raspberrypi4_64_defconfig to accommodate BCC

1. Enable use of an external toolchain by navigating to Toolchain | Toolchain type |
   External toolchain and selecting that option.
2. Back out of External toolchain and open the Toolchain submenu. Select the most
   recent ARM AArch64 toolchain as your external toolchain.
3. Back out of the Toolchain page and drill down into System configuration | /dev
   management. Select Dynamic using devtmpfs + eudev.
4. Back out of /dev management and select Enable root login with password. Open
   Root password and enter a non-empty password in the text field.
5. Back out of the System configuration page and drill down into Filesystem images.
   Increase the exact size value to 2G so that there is enough space for the kernel
   source code.
6. Back out of Filesystem images and drill down into Target packages | Networking
   applications. Select the dropbear package to enable scp and ssh access to the target.
   Note that dropbear does not allow root scp and ssh access without a password.
7. Back out of Network applications and drill down into Miscellaneous target
   packages. Select the haveged package so programs don't block waiting for
   /dev/urandom to initialize on the target.
8. Save your changes and exit menuconfig.

```bash
make savedefconfig
make linux-configure
# reconfig for BPF
 make linux-menuconfig
```

To search for a specific kernel configuration option from the interactive menu, hit / and
enter a search string

CONFIG_BPF=y
CONFIG_BPF_SYSCALL=y

CONFIG_NET_CLS_BPF=m
CONFIG_NET_ACT_BPF=m
CONFIG_BPF_JIT=y

TODO: extra at page 673(699)

## Configuring for building a BCC toolkit with Buildroot

- At 674(700)

## Configuring Valgrind

- Valgrind does not require kernel configuration, but it does need debug symbols.
  It is available as a target package in both the Yocto Project and Buildroot

## Configuring strace

- Testing...
  - link: https://buildroot.org/downloads/manual/manual.html#adding-packages
