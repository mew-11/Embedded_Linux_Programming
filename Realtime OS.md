Technical requirements

- A linux based host system with the minimum of 60GG of available disk spacce
- Buildroot 2020.02.9 LTS release
- Yocto 3.1 (Dunfell) LTS release
- Etcher for Linux
- A micro SD
- BBB
- A 5V 1A DC power supply
- AN Ethernet cable and port for network connectivity

The real-time Linux kernel (PREEMPT_RT)
Getting the PREEMPT_RT patches
The Yocto Project and PREEMPT_RT
```
# in conf/local.conf
PREFERRED_PROVIDER_virtual/kernel = "linux-yocto-rt" COMPATIBLE_MACHINE_beaglebone = "beaglebone"
```