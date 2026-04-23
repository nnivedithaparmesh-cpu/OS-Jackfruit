# Multi-Container Runtime

## Team Information

**Name:** Neha K S
**SRN:** PES1UG24CS585

**Name:** Niveditha
**SRN:** PES1UG24CS585

---

## Project Overview

This project implements a lightweight Linux container runtime in C with a long-running supervisor process and a kernel-space memory monitor.

### Main Features

* Run multiple containers at the same time
* Process isolation using namespaces
* Separate filesystem for each container
* CLI commands for management
* Logging using producer-consumer bounded buffer
* Memory monitoring with soft and hard limits
* Scheduler experiments
* Proper cleanup of processes and resources

---

## Files Included

* `engine.c` – User-space runtime and supervisor
* `monitor.c` – Kernel module
* `monitor_ioctl.h` – Shared ioctl definitions
* `cpu_hog.c` – CPU-bound workload
* `io_pulse.c` – I/O-bound workload
* `memory_hog.c` – Memory workload
* `Makefile`

---

## Requirements

Use Ubuntu 22.04 / 24.04 VM.

Install dependencies:

```bash
sudo apt update
sudo apt install -y build-essential linux-headers-$(uname -r)
```

---

## Build Steps

```bash
cd boilerplate
make
```

---

## Root Filesystem Setup

### Create base root filesystem

```bash
mkdir rootfs-base
wget https://dl-cdn.alpinelinux.org/alpine/v3.20/releases/x86_64/alpine-minirootfs-3.20.3-x86_64.tar.gz
tar -xzf alpine-minirootfs-3.20.3-x86_64.tar.gz -C rootfs-base
```

### Create writable copies

```bash
cp -a rootfs-base rootfs-alpha
cp -a rootfs-base rootfs-beta
```

### Copy workloads

```bash
cp cpu_hog rootfs-alpha/
cp cpu_hog rootfs-beta/
chmod +x rootfs-alpha/cpu_hog
chmod +x rootfs-beta/cpu_hog
```

---

## Load Kernel Monitor

```bash
cd boilerplate
sudo insmod monitor.ko
```

---

## Run Instructions

### Terminal 1 – Start Supervisor
><img width="914" height="751" alt="image" src="https://github.com/user-attachments/assets/cde967c8-aea4-4479-8e12-2f66bc064a04" />

>
```bash
cd boilerplate
sudo ./engine supervisor ./rootfs-base
```

### Terminal 2 – Start Containers

```bash
cd boilerplate
sudo ./engine start alpha ./rootfs-alpha /cpu_hog --soft-mib 48 --hard-mib 80
sudo ./engine start beta ./rootfs-beta /cpu_hog --soft-mib 64 --hard-mib 96
sudo ./engine ps
`><img width="898" height="373" alt="image" src="https://github.com/user-attachments/assets/7da6c29c-17b3-48d3-819c-65ab988faa24" />
``

### Terminal 3 – View Logs

```bash
cd boilerplate
tail -f logs/alpha.log
```

or

```bash
tail -f logs/beta.log
```

<img width="882" height="580" alt="image" src="https://github.com/user-attachments/assets/b39d50fa-9a6f-4fb6-94fc-df27ec4b1123" />
---

## Stop Containers

```bash
cd boilerplate
sudo ./engine stop alpha
sudo ./engine stop beta
```

---

## Unload Module

```bash
cd boilerplate
sudo rmmod monitor
```

---

## Supported Commands

```bash
engine supervisor <base-rootfs>
engine start <id> <container-rootfs> <command>
engine run <id> <container-rootfs> <command>
engine ps
engine logs <id>
engine stop <id>
```

---

## Features Implemented

### Task 1: Multi-Container Runtime

* Long-running supervisor
* Multiple containers
* PID, UTS, Mount namespaces
* Separate rootfs
* `/proc` mounted inside containers
* Child process reaping

### Task 2: CLI and IPC

* Start / Stop / PS / Logs commands
* IPC between CLI and supervisor
* Metadata tracking
* Signal handling

### Task 3: Logging System

* Capture stdout and stderr
* Bounded buffer
* Producer-consumer threads
* Per-container log files

### Task 4: Kernel Monitor

* `/dev/container_monitor`
* PID registration using ioctl
* Soft limit warning
* Hard limit process kill

### Task 5: Scheduler Experiments

* CPU-bound and I/O-bound workloads
* Priority comparison using nice values

### Task 6: Cleanup

* No zombie processes
* Threads exit correctly
* File descriptors closed
* Resources freed

---

## Engineering Analysis

### Isolation

Namespaces provide process and filesystem isolation. Each container has its own PID space and root filesystem.

### Supervisor

The supervisor manages all containers, tracks metadata, handles signals, and reaps children.

### IPC and Synchronization

Two IPC paths are used:

* CLI to supervisor
* Container output to logger

Mutex and condition variables protect shared data.

### Memory Management

RSS measures physical memory used by a process.

* Soft limit gives warning
* Hard limit kills process

Kernel enforcement is more reliable than user-space checks.

### Scheduling

Linux scheduler gives fair CPU time. Higher priority workloads get more CPU share.

---

## Design Tradeoffs

| Subsystem  | Choice             | Tradeoff                    |
| ---------- | ------------------ | --------------------------- |
| Filesystem | chroot             | Easier than pivot_root      |
| IPC        | UNIX socket / FIFO | Simpler than shared memory  |
| Logging    | Threads + buffer   | More synchronization needed |
| Monitor    | Kernel module      | More debugging effort       |

---

## Demo Checklist

* Multiple containers running
* `engine ps` output
* Logs generated
* CLI working
* Soft-limit warning
* Hard-limit kill
* Scheduling experiment
* Clean teardown

---

## Conclusion

This project demonstrates practical operating system concepts such as containers, namespaces, IPC, scheduling, synchronization, and kernel memory control.
