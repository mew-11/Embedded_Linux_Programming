Technical requirements

- A linux based host system with the minimum of 60GG of available disk spacce
- Buildroot 2020.02.9 LTS release
- Yocto 3.1 (Dunfell) LTS release
- Etcher for Linux
- A micro SD
- BBB
- A 5V 1A DC power supply
- AN Ethernet cable and port for network connectivity

# Configuring for real-time Linux kernel (PREEMPT_RT)

```bash
# for QEMU
make qemu_arm_versatile_defconfig
# for raspberry pi 4
make raspberrypi4_64_defconfig
make menuconfig
```

Link doc: https://www.stefanocottafavi.com/buildroot_rpi_kernel_rt/
Link doc: https://wiki.linuxfoundation.org/realtime/documentation/start
Link doc: https://www.acontis.com/en/building-a-real-time-linux-kernel-in-ubuntu-preemptrt.html
Link doc: https://ros-realtime.github.io/Guides/Real-Time-Operating-System-Setup/Real-Time-Linux/how_to_build_your_own_linux-real_time_kernel.html
Link doc: https://buildroot.org/downloads/manual/manual.html
Link doc: https://www.instructables.com/Building-GNULinux-Distribution-for-Raspberry-Pi-Us/

# Build real-time Linux kernel (PREEMPT_RT) with Yocto Project

- Add this to file conf/local.conf

BBB:

```
PREFERRED_PROVIDER_virtual/kernel = "linux-yocto-rt"
COMPATIBLE_MACHINE_beaglebone = "beaglebone"
```

Raspberry:

```
PREFERRED_PROVIDER_virtual/kernel = "linux-raspberrypi4-rt"
COMPATIBLE_MACHINE_raspberrypi4 = "raspberrypi4"
```

# Build real-time Linux kernel (PREEMPT_RT) with Buildroot
