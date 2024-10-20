## Technical requirement

- A linux-based host system
- Buildroot 2020.02.9 LTS release
- Etcher for Linux
- A micro SD card reader and card
- A Raspberry pi 4
- A 5V 3A USB-C power supply
- An Ethernet cable and port for network connectivity

# Profiling

## Profiling with `top`

- **top**: shows you how much memory is being used, which processes are eating CPU cycles, and how this is spread across different cores and times

### Details

![[Pasted image 20241018003027.png]]

![[Pasted image 20241018003457.png]]

- Component in image:
  - **PID:** Shows task’s unique process id.
  - **PR:** The process’s priority. The lower the number, the higher the priority.
  - **VIRT:** Total virtual memory used by the task.
  - **USER:** User name of owner of task.
  - **%CPU:** Represents the CPU usage.
  - **TIME+:** CPU Time, the same as ‘TIME’, but reflecting more granularity through hundredths of a second.
  - **SHR:** Represents the Shared Memory size (kb) used by a task.
  - **NI:** Represents a Nice Value of task. A Negative nice value implies higher priority, and positive Nice value means lower priority.
  - **%MEM:** Shows the Memory usage of task.
  - **RES:** How much physical RAM the process is using, measured in kilobytes.
  - **COMMAND:** The name of the command that started the process.
- Link Document: https://www.geeksforgeeks.org/top-command-in-linux-with-examples/
- Basic Shortcut:
  - **`q`**: Quit `top`.
  - **`k`**: kill process by enter PID of that process.
  - **`M`**: Sort (memmory).
  - **`P`**: Sort (CPU usage).
  - **`R`**: Swap sort.
  - **`u`**: Show process of a user by enter name of user.
  - z: highlight text in top

### Examples with `top`:

```bash
top -n 10 # stop after 10 times refresh
top -u user_name # show process of user
top -b # Batch Mode: Send output from top to file or any other programs.
top -s # Secure Mode: Use top in Secure Mode
top -c # Command Line: The below command starts top with last closed state.
top -d seconds.tenths # delay time between screen update
```

## The poor man’s profiler

1. Attach to the process using gdbserver (for a remote debug) or gbd (for a native debug). The process stops.
2. Observe the function it stopped in. You can use the backtrace GDB command to see the call stack.
3. Type continue so that the program resumes.
4. After a while, type Ctrl + C to stop it again and go back to step 2.

## Introducing perf

- perf is an abbreviation of the Linux `performance event counter subsystem`, perf_events, and also the name of the command-line tool for interacting with perf_events
- Link Document: https://perf.wiki.kernel.org
- You need a kernel that is confgured for perf_events and you need the perf command cross compiled to run on the target. The relevant kernel confguration is `CONFIG_PERF_EVENTS` present in the menu General setup | Kernel Performance Events And Counters.

## Configuring the kernel for perf

### Building with the Yocto Project

```
EXTRA_IMAGE_FEATURES = "debug-tweaks dbg-pkgs tools-profile" IMAGE_INSTALL_append = "kernel-vmlinux"
```

### Building with Buildroot

- `BR2_LINUX_KERNEL_TOOL_PERF` in Kernel | Linux Kernel Tools. To build packages with debug symbols and install them unstripped on the target, select these two settings.
- `BR2_ENABLE_DEBUG` in the menu Build options | build packages with debugging symbols menu.
- `BR2_STRIP = none` in the menu Build options | strip command for binaries on target.

## Overview about perf

### Detail

![[Pasted image 20241018010829.png]]

### Basic Syntax of the perf

Link Document: https://medium.com/@kuldeepkumawat195/what-is-the-linux-perf-command-8c89db989d8b#:~:text=The%20perf%20command%2C%20also%20known,and%20optimize%20their%20system's%20performance.

```bash
# perf [OPTIONS] SUBCOMMAND [ARGS]
# perf <subcommand> -h
# By default, perf is only accessible to the root usr, to enable normal user
sudo echo 0 > /proc/sys/kernel/perf_event_paranoid
sudo vim /etc/sysctl.conf
# add to file: kernel.perf_event_paranoid = 0
```

- Listing Events

```bash
sudo perf list # The output includes a list of all supported events, regardless of their type.
sudo perf sw # To display software events
```

- Real-Time CPU Profile

```bash
sudo perf top
```

![[Pasted image 20241018013601.png]]
**The output of perf top consists of three columns presented from left to right:**

1. CPU usage associated with a particular function is represented as percentages.
2. The library or program utilizing the function.
3. The symbol and function name, are denoted as [k] for kernel space and [.] for user space.
   **By default, perf top monitors all online CPUs. However, there are additional options that give you more control:** - You can use the -a option to monitor all CPUs, even idle ones. - If you want to monitor specific CPUs instead of all of them, use the -C option followed by a list of comma-separated CPU numbers. - You can also control the sampling frequency of perf top using the -F option.

- CPU Performance Stats:

```bash
sudo perf stat -a sleep 5 # display CPU performance statistics for all standard CPU-wide hardware and software events
```

![[Pasted image 20241018014018.png]]

- CPU Performance for any Command

```bash
sudo perf stat <command> # see performance of command
```

- CPU Performance for any Process

```bash
sudo perf stat -p <PID> sleep 5
```

- Counting System Calls by Type

```bash
sudo perf stat -e 'syscalls:sys_enter_*' -a sleep 5 # To count different system calls made by all processes over a specific period
```

- Recording CPU Cycles

```bash
sudo perf record -e cycles sleep 10 # record and collect CPU cycle information for 10 seconds
```

- Performance Results

```bash
sudo perf report
```

- Customizing Output Format

```bash
sudo perf report --stdio # standard output
sudo perf report --tui # tui output
sudo perf report -n --sort comm,symbol --stdio # includes columns for the sample number, command, and symbol information.
```

- Displaying Trace Output
  The `perf script` command provides a chronological view of recorded events:

```bash
sudo perf script
```

- Trace Header

```bash
sudo perf script --header
```

The “perf script — header” command displays the header information of the file, which includes details about the trace session. This information usually includes the start time of the trace, its duration, CPU-related statistics, and the command that generated the data.

- Dumping Raw Data

```bash
sudo perf script -D
```

- Annotating Data with Source Code

```bash
sudo perf annotate --stdio -v
```

- Profiling Kernel Functions

```bash
sudo perf stat -e cycles:k sleep 5
```

- Generating a Flame Graph

```bash
sudo perf record -F 99 -a -g -- sleep 60
```

# Tracing

The tools we have seen so far all use statistical sampling. You often want to know more about the ordering of events so that you can see them and relate them to each other. Function tracing involves instrumenting the code with tracepoints that capture information about the event, and may include some or all of the following:
• A timestamp
• Context, such as the current PID
• Function parameters and return values
• A callstack
It is more intrusive than statistical profiling and it can generate a large amount of data. The latter problem can be mitigated by applying filters when the sample is captured and later on when viewing the trace.
I will cover three trace tools here: the kernel function tracers Ftrace, LTTng, and BPF.

## Ftrace

Link Document: https://www.kernel.org/doc/html/v5.0/trace/ftrace.html
Follow and record act in kernel and process of user with kernel.

- Preparing to use Ftrace
  Ftrace and its various options are configured in the kernel configuration menu. You will need the following as a minimum: - CONFIG_FUNCTION_TRACER from the Kernel hacking | Tracers | Kernel Function Tracer menu For reasons that will become clear later, you would be well advised to turn on these options as well: - CONFIG_FUNCTION_GRAPH_TRACER in the Kernel hacking | Tracers | Kernel Function Graph Tracer menu - CONFIG_DYNAMIC_FTRACE in the Kernel hacking | Tracers | enable/disable function tracing dynamically menu
- Using Ftrace

```bash
mount -t debugfs none /sys/kernel/debug # convention goes in the /sys/kernel/debug
cat /sys/kernel/debug/tracing/available_tracers # list of tracers available in the kernel => nop (null tracer)
# enable tracer
echo function > /sys/kernel/debug/tracing/current_tracer
echo 1 > /sys/kernel/debug/tracing/tracing_on # start tracing
echo 0 > /sys/kernel/debug/tracing/tracing_on # stop tracing
# read trace buffer from trace file
cat /sys/kernel/debug/tracing/trace # result of tracing

echo > trace # delete data in trace

# filter function
echo 'schedule' > set_ftrace_filter


```

### Dynamic Ftrace and trace filters

To started => Enable CONFIG_DYNAMIC_FTRACE
Link Document: https://medium.com/granulate/dynamic-ftrace-filtering-dececfc3eced

## LTTng

Link document: https://lttng.org/
Record and analyze events happen in kernel and user applications.

### Using LTTng

LTTng consists of three components: - A core session manager - A kernel tracer implemented as a group of kernel modules - A user space tracer implemented as a library
LTTng and the Yocto Project

- add to conf/local.conf

```
IMAGE_INSTALL_append = " lttng-tools lttng-modules lttng-ust"
```

LTTng and Buildroot

- You need to enable the following:
  - BR2_PACKAGE_LTTNG_MODULES in the Target packages | Debugging, profiling and benchmark | lttng-modules menu
  - BR2_PACKAGE_LTTNG_TOOLS in the Target packages | Debugging, profiling and benchmark | lttng-tools menu
- For user space trace tracing, enable this:
  - BR2_PACKAGE_LTTNG_LIBUST in the Target packages | Libraries | Other, enable lttng-libust menu

### Examples

```
Document: https://lttng.org/docs/v2.13/#doc-what-is-tracing
```

## BPF

### Configuring the kernel for BPF

```bash
cd buildroot
make clean
make raspberry4_64_defconfig
make menuconfig

```

Before configuring our kernel for BPF, let's select an external toolchain and modify raspberrypi4_64_defconfig to accommodate BCC: 1. Enable use of an external toolchain by navigating to Toolchain | Toolchain type | External toolchain and selecting that option. 2. Back out of External toolchain and open the Toolchain submenu. Select the most recent ARM AArch64 toolchain as your external toolchain. 3. . Back out of the Toolchain page and drill down into System configuration | /dev management. Select Dynamic using devtmpfs + eudev. 4. Back out of /dev management and select Enable root login with password. Open Root password and enter a non-empty password in the text field. 5. Back out of the System configuration page and drill down into Filesystem images. Increase the exact size value to 2G so that there is enough space for the kernel source code. 6. Back out of Filesystem images and drill down into Target packages | Networking applications. Select the dropbear package to enable scp and ssh access to the target. Note that dropbear does not allow root scp and ssh access without a password. 7. Back out of Network applications and drill down into Miscellaneous target packages. Select the haveged package so programs don't block waiting for /dev/urandom to initialize on the target. 8. Save your changes and exit menuconfig.

```
make savedefconfig
make linux-configure
```

## Valgrind

```
Link document: https://valgrind.org/docs/manual/quick-start.html
To check options:
valgrind --help
valgrind [options] prog-and-args
```

### Callgrind

```bash
valgrind --tool=callgrind <program>
# to see pid file => callgrind.out.<PID>
# you can copy to the host and analyze with callgrind_annotate
```

### Helgrind

```bash
valgrind --tool=helgrind <program>
```

## strace

The `strace` command is a diagnostic tool in Linux that can be used to trace the system calls made by a process. This can be useful for debugging applications, troubleshooting problems, and learning more about how the Linux kernel works.

You can use it to do the following: - Learn which system calls a program makes. - Find those system calls that fail, together with the error code. I find this useful if a program fails to start but doesn't print an error message or if the message is too general. - Find which files a program opens. - Find out which syscalls a running program is making, for example, to see whether it is stuck in a loop
Link document: https://medium.com/@adilrk/the-strace-command-is-a-diagnostic-tool-in-linux-that-can-be-used-to-trace-the-system-calls-made-93f035ae2f2d

Strace command

```help
Usage: strace [-ACdffhikqqrtttTvVwxxyyzZ] [-I N] [-b execve] [-e EXPR]...
              [-a COLUMN] [-o FILE] [-s STRSIZE] [-X FORMAT] [-O OVERHEAD]
              [-S SORTBY] [-P PATH]... [-p PID]... [-U COLUMNS] [--seccomp-bpf]
              { -p PID | [-DDD] [-E VAR=VAL]... [-u USERNAME] PROG [ARGS] }
   or: strace -c[dfwzZ] [-I N] [-b execve] [-e EXPR]... [-O OVERHEAD]
              [-S SORTBY] [-P PATH]... [-p PID]... [-U COLUMNS] [--seccomp-bpf]
              { -p PID | [-DDD] [-E VAR=VAL]... [-u USERNAME] PROG [ARGS] }

General:
  -e EXPR        a qualifying expression: OPTION=[!]all or OPTION=[!]VAL1[,VAL2]...
     options:    trace, abbrev, verbose, raw, signal, read, write, fault,
                 inject, status, quiet, kvm, decode-fds

Startup:
  -E VAR=VAL, --env=VAR=VAL
                 put VAR=VAL in the environment for command
  -E VAR, --env=VAR
                 remove VAR from the environment for command
  -p PID, --attach=PID
                 trace process with process id PID, may be repeated
  -u USERNAME, --user=USERNAME
                 run command as USERNAME handling setuid and/or setgid
                 USERNAME may be a user name or a UID:GID pair, where UID
                 and GID are numbers. In the latter case, strace does not
                 perform name lookups.
  --argv0=NAME   set PROG argv[0] to NAME

Tracing:
  -b execve, --detach-on=execve
                 detach on execve syscall
  -D, --daemonize[=grandchild]
                 run tracer process as a grandchild, not as a parent
  -DD, --daemonize=pgroup
                 run tracer process in a separate process group
  -DDD, --daemonize=session
                 run tracer process in a separate session
  -f, --follow-forks
                 follow forks
  -ff, --follow-forks --output-separately
                 follow forks with output into separate files
  -I INTERRUPTIBLE, --interruptible=INTERRUPTIBLE
     1, anywhere:   no signals are blocked
     2, waiting:    fatal signals are blocked while decoding syscall (default)
     3, never:      fatal signals are always blocked (default if '-o FILE PROG')
     4, never_tstp: fatal signals and SIGTSTP (^Z) are always blocked
                    (useful to make 'strace -o FILE PROG' not stop on ^Z)
  --kill-on-exit kill all tracees if strace is killed

Filtering:
  -e trace=[!][?]{{SYSCALL|GROUP|all|/REGEX}[@64|@32|@x32]|none},
   --trace=[!][?]{{SYSCALL|GROUP|all|/REGEX}[@64|@32|@x32]|none}
                 trace only specified syscalls.
     groups:     %clock, %creds, %desc, %file, %fstat, %fstatfs %ipc, %lstat,
                 %memory, %net, %process, %pure, %signal, %stat, %%stat,
                 %statfs, %%statfs
  -e signal=SET, --signal=SET
                 trace only the specified set of signals
                 print only the signals from SET
  -e status=SET, --status=SET
                 print only system calls with the return statuses in SET
     statuses:   successful, failed, unfinished, unavailable, detached
  -e trace-fds=SET, --trace-fds=SET
                 trace operations on file descriptors from SET
  -P PATH, --trace-path=PATH
                 trace accesses to PATH
  -z, --successful-only
                 print only syscalls that returned without an error code
  -Z, --failed-only
                 print only syscalls that returned with an error code

Output format:
  -a COLUMN, --columns=COLUMN
                 alignment COLUMN for printing syscall results (default 40)
  -e abbrev=SET, --abbrev=SET
                 abbreviate output for the syscalls in SET
  -e verbose=SET, --verbose=SET
                 dereference structures for the syscall in SET
  -e raw=SET, --raw=SET
                 print undecoded arguments for the syscalls in SET
  -e read=SET, --read=SET
                 dump the data read from the file descriptors in SET
  -e write=SET, --write=SET
                 dump the data written to the file descriptors in SET
  -e quiet=SET, --quiet=SET
                 suppress various informational messages
     messages:   attach, exit, path-resolution, personality, thread-execve
  -e kvm=vcpu, --kvm=vcpu
                 print exit reason of kvm vcpu
  -e decode-fds=SET, --decode-fds=SET
                 what kinds of file descriptor information details to decode
     details:    dev (device major/minor for block/char device files),
                 eventfd (associated eventfd object details for eventfds),
                 path (file path),
                 pidfd (associated PID for pidfds),
                 socket (protocol-specific information for socket descriptors),
                 signalfd (signal masks for signalfds)
  -i, --instruction-pointer
                 print instruction pointer at time of syscall
  -k, --stack-trace[=symbol]
                 obtain stack trace between each syscall
  --stack-trace-frame-limit=limit
                 obtain no more than this amount of frames
                 when backtracing a syscall (default 256)
  -n, --syscall-number
                 print syscall number
  -o FILE, --output=FILE
                 send trace output to FILE instead of stderr
  -A, --output-append-mode
                 open the file provided in the -o option in append mode
  --output-separately
                 output into separate files (by appending pid to file names)
  -q, --quiet=attach,personality
                 suppress messages about attaching, detaching, etc.
  -qq, --quiet=attach,personality,exit
                 suppress messages about process exit status as well.
  -qqq, --quiet=all
                 suppress all suppressible messages.
  -r, --relative-timestamps[=PRECISION]
                 print relative timestamp
     precision:  one of s, ms, us, ns; default is microseconds
  -s STRSIZE, --string-limit=STRSIZE
                 limit length of print strings to STRSIZE chars (default 32)
  --absolute-timestamps=[[format:]FORMAT[,[precision:]PRECISION]]
                 set the format of absolute timestamps
     format:     none, time, or unix; default is time
     precision:  one of s, ms, us, ns; default is seconds
  -t, --absolute-timestamps[=time]
                 print absolute timestamp
  -tt, --absolute-timestamps=[time,]us
                 print absolute timestamp with usecs
  -ttt, --absolute-timestamps=unix,us
                 print absolute UNIX time with usecs
  -T, --syscall-times[=PRECISION]
                 print time spent in each syscall
     precision:  one of s, ms, us, ns; default is microseconds
  -v, --no-abbrev
                 verbose mode: print entities unabbreviated
  --strings-in-hex=non-ascii-chars
                 use hex instead of octal in escape sequences
  -x, --strings-in-hex=non-ascii
                 print non-ASCII strings in hex
  -xx, --strings-in-hex[=all]
                 print all strings in hex
  -X FORMAT, --const-print-style=FORMAT
                 set the FORMAT for printing of named constants and flags
     formats:    raw, abbrev, verbose
  -y, --decode-fds[=path]
                 print paths associated with file descriptor arguments
  -yy, --decode-fds=all
                 print all available information associated with file
                 descriptors in addition to paths
  --decode-pids=pidns
                 print PIDs in strace's namespace, too -Y, --decode-pids=comm
                 print command names associated with PIDs
  --always-show-pid
                 show PID prefix also for the process started by strace

Statistics:
  -c, --summary-only
                 count time, calls, and errors for each syscall and report
                 summary
  -C, --summary  like -c, but also print the regular output
  -O OVERHEAD[UNIT], --summary-syscall-overhead=OVERHEAD[UNIT]
                 set overhead for tracing syscalls to OVERHEAD UNITs
     units:      one of s, ms, us, ns; default is microseconds
  -S SORTBY, --summary-sort-by=SORTBY
                 sort syscall counts by: time, min-time, max-time, avg-time,
                 calls, errors, name, nothing (default time)
  -U COLUMNS, --summary-columns=COLUMNS
                 show specific columns in the summary report: comma-separated
                 list of time-percent, total-time, min-time, max-time,
                 avg-time, calls, errors, name
                 (default time-percent,total-time,avg-time,calls,errors,name)
  -w, --summary-wall-clock
                 summarise syscall latency (default is system time)

Stop condition:
  --syscall-limit=LIMIT
                 Detach all tracees after tracing LIMIT syscalls

Tampering:
  -e inject=SET[:error=ERRNO|:retval=VALUE][:signal=SIG][:syscall=SYSCALL]
            [:delay_enter=DELAY][:delay_exit=DELAY]
            [:poke_enter=@argN=DATAN,@argM=DATAM...]
            [:poke_exit=@argN=DATAN,@argM=DATAM...]
            [:when=WHEN],
  --inject=SET[:error=ERRNO|:retval=VALUE][:signal=SIG][:syscall=SYSCALL]
           [:delay_enter=DELAY][:delay_exit=DELAY]
           [:poke_enter=@argN=DATAN,@argM=DATAM...]
           [:poke_exit=@argN=DATAN,@argM=DATAM...]
           [:when=WHEN],
                 perform syscall tampering for the syscalls in SET
     delay:      microseconds or NUMBER{s|ms|us|ns}
     when:       FIRST[..LAST][+[STEP]]
  -e fault=SET[:error=ERRNO][:when=WHEN], --fault=SET[:error=ERRNO [:when=WHEN]
                 synonym for -e inject with default ERRNO set to ENOSYS.

Miscellaneous:
  -d, --debug    enable debug output to stderr
  -h, --help     print help message
  --seccomp-bpf  enable seccomp-bpf filtering
  --tips[=[[id:]ID][,[format:]FORMAT]]
                 show strace tips, tricks, and tweaks on exit
     id:         non-negative integer or random; default is random
     format:     none, compact, full; default is compact
  -V, --version  print version

```

### Examples:

```bash
strace ls
# The output of `strace` will show a list of all the system calls that were made by the `ls` command
# For example, the `-v` option can be used to print more information about each system call, including the return value and the errno value. The `-f` option can be used to trace the child processes of the specified process. And the `-o` option can be used to save the output to a file.
```

- Run the `strace` command on a running process

```bash
strace -p <PID>
```

### Expand: `ltrace`

# Summary using buildroot to add all tools

```

```
