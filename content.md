Technical requirements

- A linux base host system
- Buildroot 2020.02.9 LTS release => chapter 6
- Yocto Project dunfell release (minimum 60gb disk space) => chapter 6, 7
- Etcher for linux
- A raspberry pi 4
- A 5V 3A USB-C power supply
- An Ethernet cable and port for network connectivily

In this chapter, we will cover the following topics:
• The observer effect

- examine tools: `perf`, `Ftrace`, `LLTng`, and `BPF`
  • Beginning to profile

Page 648-683

# Profiling

1. Profiling with `top`, `htop`
   1.1 Intro `top`
   1.2 Poor man’s profiler => using gdbserver (chapter 19)
   => Tai
2. Profiling with `perf`
   => Thuy Tien, Mai Lien

# Tracing

3. Introducing Ftrace
   3.1. Intro about Ftrace
   3.2. Preparing to use Ftrace
   3.3. Using Ftrace
   3.4. Dynamic Ftrace and trace filters
   => Tri, Man
4. Using LTTng => LTT (Linux Trace Toolkit)
   4.1. LTTng and the Yocto Project
   4.2. LTTng and Buildroot
   4.3. Using LTTng for kernel tracing
   => Xuan Mai, Huynh
5. Using BPF (Berkeley Packet Filter)
   5.1. Configuring the kernel for BPF with buildroot
   5.2. Build a BCC toolkit with Buildroot
   5.3. Using BPF tracing tools
   => Tien
6. Using Valgrind
   6.1. Callgrind
   6.2. Helgrind
   => Phu
7. Using `strace`
   => Tai
