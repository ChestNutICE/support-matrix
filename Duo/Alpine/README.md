---
sys: alpine
sys_ver: 3.21.3
sys_var: null

status: basic
last_update: 2025-05-01
---

# Alpine Linux Milk-V Duo Test Report

## Test Environment

### Operating System Information

- System Version: 3.21.3
- Download Link: 
  - Alpine minirootfs: [https://alpinelinux.org/downloads/](https://dl-cdn.alpinelinux.org/alpine/v3.21/releases/riscv64/alpine-minirootfs-3.21.3-riscv64.tar.gz)
  - Official Buildroot image: [https://github.com/milkv-duo/duo-buildroot-sdk/releases](https://github.com/milkv-duo/duo-buildroot-sdk/releases/download/v1.1.4/milkv-duo-sd-v1.1.4.img.zip)
- Reference Installation Document: 
  - [Alpine Wiki (Installation)](https://wiki.alpinelinux.org/wiki/Installation)
  - [Alpine Wiki (How to make a cross architecture chroot)](https://wiki.alpinelinux.org/wiki/How_to_make_a_cross_architecture_chroot)
  - [Milk-V forum thread](https://community.milkv.io/t/alpine-linux-on-the-duo/700/18)

### Hardware Information

- Milk-V Duo
- A USB-A to C or USB-C to C cable
- A microSD card
- A microSD card reader
- A USB to UART debugger (e.g., CH340, CH341, FT2232, etc.)
- Optional: Milk-V Duo IOB (Baseboard)

## Installation Steps

### Download Alpine minirootfs and Buildroot image

```bash
wget https://dl-cdn.alpinelinux.org/alpine/v3.21/releases/riscv64/alpine-minirootfs-3.21.3-riscv64.tar.gz
tar -xvf alpine-minirootfs-3.21.3-riscv64.tar.gz --one-top-level
wget https://github.com/milkv-duo/duo-buildroot-sdk/releases/download/v1.1.4/milkv-duo-sd-v1.1.4.img.zip
unzip milkv-duo-sd-v1.1.4.img.zip
```

### Prepare rootfs
Note that the obtained Alpine system is only a "minirootfs" without system packages such as OpenRC. We need to install Alpine's base packages using `apk` inside the rootfs in order to make it bootable.

#### Install Alpine's package manager `apk`
Skip this step if using Alpine-based distributions on host.

```bash
wget https://gitlab.alpinelinux.org/api/v4/projects/5/packages/generic/v2.14.10/x86_64/apk.static
chmod +x apk.static
mv apk.static /usr/bin/apk
```

#### Install Alpine's base package `alpine-base` inside minirootfs

(`chroot` is NOT required)

```bash
cd alpine-minirootfs-3.20.3-riscv64
sudo apk add -p . --initdb -U --arch riscv64 --allow-untrusted alpine-base
```

#### Extra setups

1. Edit `./etc/inittab` and add the following line (or uncomment) to enable serial access on `/dev/ttyS0`:
    ```
    ttyS0::respawn:/sbin/getty -L 115200 ttyS0 vt100
    ```
    And comment out the six lines starting with `tty1` - `tty6`.

2. Edit `./etc/passwd`:

    Remove the `x` in `root:x:0:0:root:/root:/bin/sh`.

    (Alternatively, edit `/etc/shadow` and remove the `*` in `root:*::0:::::`). 

3. Enable the OpenRC hostname service (for setting up the hostname correctly)：
   
   ```bash
   ln -s ./etc/init.d/hostname ./etc/runlevels/boot
   ```
   You may enable other OpenRC system services of interest likewise.

### Flash Buildroot image

```bash
cd ..
sudo dd if=milkv-duo-sd-v1.1.3-2024-0930.img of=/dev/your/device bs=4M status=progress
```

Your device should now be able to detect the `rootfs` and `boot` partitions on the SD card. Mount only the `rootfs` partition.

### Replace root on SD card with Alpine rootfs
```bash
rm -rf /path/to/your/mnt/root/*
cp -r /path/to/your/alpine-minirootfs-3.20.3-riscv64/* /path/to/your/mnt/root/
```

### Booting and Logging into the System

Insert SD card onto the board and boot.
Login into the system via serial port at `/dev/ttyUSB0`, baudrate 115200.

Default username: `root`
Default password: none

### Optional post-installation setups
Setup the password and hostname with `passwd` and `hostname` after login. 

Setup the system time with `date -s`, then install `cronyd`:
```bash
apk add cronyd
rc-update add chronyd default
```

It is recommended to enable the following system OpenRC services (though test results show that the system will still be able to boot nevertheless):

```bash
rc-update add bootmisc boot
rc-update add networking boot # make sure /etc/network/interfaces is present
rc-update add devfs sysinit
rc-update add mdev sysinit
rc-update add acpid default
rc-update add killprocs shutdown
rc-update add mount-ro shutdown
rc-update add savecache shutdown
```

## Expected Results

The system should boot normally and allow login through the onboard serial port.

## Actual Results

The system booted successfully, and login through the onboard serial port was also successful.

### Boot Log

```log
   OpenRC 0.55.1 is starting up Linux 5.10.4-tag- (riscv64)

 * Mounting /proc ... [ ok ]
 * Mounting /run ... [ ok ]
 * /run/openrc: creating directory
 * /run/lock: creating directory
 * /run/lock: correcting owner
 * Caching service dependencies ... [ ok ]
 * Clock skew detected with `/etc/init.d'
 * Adjusting mtime of `/run/openrc/deptree' to Wed Apr 30 19:42:03 2025

 * WARNING: clock skew detected!
 * WARNING: clock skew detected!
 * WARNING: clock skew detected!

Welcome to Alpine Linux 3.21
Kernel 5.10.4-tag- on an riscv64 (/dev/ttyS0)

(none) login: root
Welcome to Alpine!

The Alpine Wiki contains a large amount of how-to guides and general
information about administrating Alpine systems.
See <https://wiki.alpinelinux.org/>.

You can setup the system with the command: setup-alpine

You may change this message by editing /etc/motd.

login[206]: root login on 'ttyS0'
(none):~# uname -a
Linux (none) 5.10.4-tag- #1 PREEMPT Fri Nov 22 11:31:04 CST 2024 riscv64 Linux
(none):~# cat /etc/os-release 
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.21.3
PRETTY_NAME="Alpine Linux v3.21"
HOME_URL="https://alpinelinux.org/"
BUG_REPORT_URL="https://gitlab.alpinelinux.org/alpine/aports/-/issues"
(none):~# 
```

## Test Criteria

Successful: The actual result matches the expected result.

Failed: The actual result does not match the expected result.

## Test Conclusion

Test successful.
