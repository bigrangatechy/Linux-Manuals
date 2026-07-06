# Chapter 22: Hardware and Drivers — Managing Physical Devices

## Hardware Just Works (Until It Doesn't)

One of Linux's greatest strengths is hardware support. Plug in a keyboard, mouse, webcam, or printer, and it usually works instantly — no driver CDs, no installer wizards, no reboots. The Linux kernel includes thousands of drivers built in, and Fedora 44's kernel 7.0 series supports an enormous range of hardware out of the box.

But "just works" can feel like magic when it works and like a mystery when it doesn't. Understanding how Linux detects, drives, and manages hardware transforms that magic into something you can diagnose and control.

This chapter teaches you:
- How the kernel communicates with physical devices
- How to enumerate and identify every piece of hardware in your machine
- How kernel modules (drivers) load and unload
- How firmware differs from drivers
- How udev dynamically manages device nodes in `/dev`
- How to install proprietary drivers (particularly NVIDIA GPUs)
- How to update system firmware (BIOS/UEFI) with fwupd
- How to troubleshoot hardware problems systematically

---

## Table of Contents

| Topic | Section |
|---|---|
| How Linux Sees Hardware | 1 |
| Hardware Discovery Tools | 2 |
| Kernel Modules | 3 |
| Firmware: The Software Inside Hardware | 4 |
| udev: Dynamic Device Management | 5 |
| Graphics Drivers | 6 |
| Audio Subsystem | 7 |
| Input Devices | 8 |
| Power Management and Battery | 9 |
| Hardware Troubleshooting Methodology | 10 |

---

## 1. How Linux Sees Hardware

### The Layered Model

Linux manages hardware through a layered architecture:

┌──────────────────────────────────────────────┐ │ Applications (Firefox, Steam, your code) │ ├──────────────────────────────────────────────┤ │ /dev — Device files (sda, null, input0) │ ├──────────────────────────────────────────────┤ │ udev — Dynamic device node creation │ ├──────────────────────────────────────────────┤ │ Kernel subsystems (ALSA, DRM, net, input) │ ├──────────────────────────────────────────────┤ │ Kernel modules / drivers (nouveau, snd_hda) │ ├──────────────────────────────────────────────┤ │ Firmware (loaded into hardware by kernel) │ ├──────────────────────────────────────────────┤ │ Hardware (GPU, SSD, Wi-Fi card, trackpad) │ └──────────────────────────────────────────────┘

Each layer has a specific role:
Layer	Role	Example
Hardware	Physical silicon	NVIDIA RTX 4070 GPU
Firmware	Low-level code baked into the device	GPU microcode, loaded from /lib/firmware/
Kernel modules	Drivers that talk to hardware	nouveau (open-source NVIDIA driver)
Kernel subsystems	Standardized interfaces	DRM/KMS (display), ALSA (audio), net (networking)
udev	Creates /dev entries dynamically	/dev/nvidia0, /dev/sda, /dev/input/event3
Applications	Use device files to access hardware	Games access GPU via Mesa/Vulkan; audio apps use PipeWire
Drivers vs. Firmware vs. udev

These three concepts are frequently confused:

Drivers (kernel modules) are code that runs ON YOUR CPU. They translate kernel system calls into hardware-specific commands. The driver knows how to tell an NVIDIA GPU "draw a triangle" or a Wi-Fi card "scan for networks."

Firmware is code that runs ON THE HARDWARE DEVICE ITSELF. Many modern devices have their own processors. Your Wi-Fi card, GPU, and even your SSD contain embedded processors that run firmware. The kernel loads firmware into the device at startup — it's essentially "teaching" the hardware how to behave.

udev is the userspace daemon that reacts to hardware events. When you plug in a USB drive, the kernel detects it, loads the driver, and udev creates /dev/sdb so applications can access it. udev also applies naming rules (like calling your network card enp3s0 instead of eth0).

USB drive plugged in
  → Kernel detects new USB device (PCI/USB bus enumeration)
  → Kernel loads usb-storage module (driver)
  → Driver initializes the device, reads partition table
  → udev receives kernel event (uevent)
  → udev creates /dev/sda1 with correct permissions
  → udev runs any matching rules (auto-mount, notify desktop, etc.)
  → File manager shows the drive

    💡 The "It Just Works" Pipeline

    When you plug something in and it works, this entire pipeline executed silently in milliseconds. Understanding each stage means you can diagnose WHERE in the pipeline something broke when it doesn't work.

/sys: The Hardware Window

The /sys filesystem (sysfs) is a virtual filesystem that exposes kernel data structures as files. Every device the kernel knows about appears here:

# List top-level sysfs categories:
ls /sys
# block  bus  class  dev  devices  firmware  fs  kernel  module  power

# Browse device classes (organized by type):
ls /sys/class/
# backlight  drm  graphics  input  net  sound  thermal  tty ...

# Network interfaces:
ls /sys/class/net/
# enp3s0  lo  wlp2s0

# Block devices (storage):
ls /sys/block/
# nvme0n1  sda

# Display controllers (DRM):
ls /sys/class/drm/
# card0  card0-eDP-1  card0-HDMI-A-1  version

# Examine a specific device:
cat /sys/class/net/enp3s0/address
# 00:1a:2b:3c:4d:5e  ← MAC address

cat /sys/class/drm/card0/device/vendor
# 0x10de  ← NVIDIA's PCI vendor ID

Everything in /sys is a real-time view. Reading files queries the hardware; some files are writable and let you change settings:

# Check battery capacity:
cat /sys/class/power_supply/BAT0/capacity
# 87

# Check current brightness:
cat /sys/class/backlight/intel_backlight/brightness
# 450

# Set brightness (carefully):
echo 300 | sudo tee /sys/class/backlight/intel_backlight/brightness

# Check CPU temperature:
cat /sys/class/thermal/thermal_zone0/temp
# 45000  ← in millidegrees Celsius (45°C)

# List all CPU frequencies available:
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
# 3500000 3400000 3200000 3000000 2200000

/dev: Device Nodes

While /sys exposes attributes, /dev provides the actual interface files that programs read/write to:

# Block devices (storage):
ls -la /dev/sd* /dev/nvme*
# /dev/nvme0n1     ← NVMe SSD
# /dev/nvme0n1p1   ← First partition
# /dev/sda         ← SATA/USB drive

# Input devices:
ls /dev/input/
# event0  event1  event2  mice  mouse0

# Audio devices:
ls /dev/snd/
# controlC0  pcmC0D0c  pcmC0D0p  seq  timer

# Special devices:
ls -la /dev/null /dev/zero /dev/random /dev/urandom
# crw-rw-rw- ... /dev/null   ← Discard all data written
# crw-rw-rw- ... /dev/zero   ← Endless stream of zeros
# crw-rw-rw- ... /dev/random  ← Cryptographic random numbers
# crw-rw-rw- ... /dev/urandom ← Faster pseudo-random numbers

Device files have a c (character) or b (block) type prefix:

brw-rw----  root disk  8,   0  /dev/sda     ← b = block device
crw-rw-rw-  root root  1,   3  /dev/null    ← c = character device

The numbers 8, 0 are the major/minor device numbers — the kernel uses these to route I/O to the correct driver. Major number 8 = SCSI/SATA disk driver.
2. Hardware Discovery Tools

Fedora 44 includes several tools for enumerating hardware. Each provides a different view — use them together for a complete picture.
lspci: PCI Bus Enumeration

PCI (Peripheral Component Interconnect) is the backbone for most internal devices: GPUs, network cards, USB controllers, storage controllers, audio chips.

# List all PCI devices:
lspci
# 00:00.0 Host bridge: Intel Corporation 12th Gen Core Processor
# 00:02.0 VGA compatible controller: Intel Corporation UHD Graphics
# 01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060]
# 00:14.0 USB controller: Intel Corporation Alder Lake USB
# 00:1f.3 Audio device: Intel Corporation Alder Lake HD Audio
# 00:1f.6 Ethernet controller: Intel Corporation Ethernet Controller

# Show which kernel driver is handling each device (-k):
lspci -k
# 01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060]
#       Subsystem: ... 
#       Kernel driver in use: nouveau          ← Currently active driver
#       Kernel modules: nouveau, nvidia, nvidia_drm  ← Available alternatives

# Verbose output (maximum detail):
lspci -v -s 01:00.0    # -s selects a specific device

# Find just graphics devices:
lspci | grep -iE 'vga|3d|display'
# 00:02.0 VGA compatible controller: Intel Corporation UHD Graphics
# 01:00.0 VGA compatible controller: NVIDIA Corporation GA104

# Machine-readable output:
lspci -mm

Understanding lspci output:

    00:00.0 — Bus:Device.Function address (how the device is physically wired)
    Host bridge — The CPU's connection to the chipset
    VGA compatible controller — GPU
    Audio device — Sound chip
    Ethernet controller — Wired network card
    USB controller — Manages USB ports

lsusb: USB Bus Enumeration

# List all USB devices:
lsusb
# Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
# Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
# Bus 001 Device 002: ID 046d:c534 Logitech, Inc. Unifying Receiver
# Bus 001 Device 003: ID 04f2:b6db Chicony Electronics Co., Ltd Integrated Camera
# Bus 001 Device 004: ID 0bda:8779 Realtek Semiconductor Corp. Bluetooth Radio

# Verbose (detailed info per device):
lsusb -v

# Show a specific device:
lsusb -d 046d:c534 -v
# Detailed info about the Logitech receiver

# Show USB tree (hierarchical view):
lsusb -t
# /:  Bus 02.Port 001: Dev 001, Class=root_hub, Driver=xhci-hcd, 5000M
# /:  Bus 01.Port 001: Dev 001, Class=root_hub, Driver=xhci-hcd, 480M
#     |__ Port 003: Dev 002, If 1, Class=Human Interface Device, Driver=usbhid
#     |__ Port 003: Dev 002, If 0, Class=Human Interface Device, Driver=usbhid

USB Vendor/Product IDs:

    046d:c534 — vendor 046d (Logitech), product c534 (Unifying Receiver)
    Lookup: http://usb-ids.sourceforge.net or the hwdata package

lscpu: CPU Information

# Full CPU details:
lscpu

Architecture:            x86_64
CPU op-mode(s):          32-bit, 64-bit
Address sizes:           46 bits physical, 48 bits virtual
Byte Order:              Little Endian
CPU(s):                  16
On-line CPU(s) list:     0-15
Vendor ID:               AuthenticAMD
Model name:              AMD Ryzen 9 7950X 16-Core Processor
CPU family:              25
Model:                   16
Thread(s) per core:      2
Core(s) per socket:      16
Socket(s):               1
Stepping:                2
Frequency boost:         enabled
CPU max MHz:             5758.6
CPU min MHz:             550.0
BogoMIPS:                11502.39
Virtualization:          AMD-V
L1d cache:               512 KiB (16 instances)
L1i cache:               512 KiB (16 instances)
L2 cache:                16 MiB (16 instances)
L3 cache:                64 MiB (1 instance)
NUMA node(s):            1

Key fields to understand:
Field	Meaning
CPU(s): 16	Total logical cores (physical × threads per core)
Thread(s) per core: 2	Hyperthreading / SMT enabled
Core(s) per socket: 16	Physical cores
CPU max MHz	Maximum clock speed
Virtualization: AMD-V	Hardware virtualization support (needed for VMs)
L3 cache: 64 MiB	Shared cache size
lshw: Comprehensive Hardware Inventory

# Install if not present:
sudo dnf install lshw

# Short summary of all hardware:
sudo lshw -short

H/W path        Device          Class       Description
======================================================
                                system      Micro-Star International Co., Ltd MS-7C37
/0                              bus         PRO X670-P WIFI (MS-7C37)
/0/0                            memory      128GiB System memory
/0/1                            processor   AMD Ryzen 9 7950X
/0/1/0                          memory      512KiB L1 cache
/0/1/1                          memory      16MiB L2 cache
/0/1/2                          memory      64MiB L3 cache
/0/2                            memory      256KiB BIOS
/0/3                            bridge      ASMedia
/0/4                            display     NVIDIA Corporation GA104
/0/5                            bus         AMD
/0/6                            network     Intel Ethernet Controller
/0/7                            display     AMD ATI Raphael
/0/8                            storage     Advanced Micro Devices
/0/9                            multimedia   Advanced Micro Devices
/0/a                           bus         Advanced Micro Devices
/1              enp5s0          network     Ethernet interface
/2              wlp6s0          network     Wireless interface

# Detailed view of a specific class:
sudo lshw -class display
# Shows GPU vendor, driver, clock speeds, VRAM

sudo lshw -class network
# Shows network interface vendor, driver, link state, speed

sudo lshw -class storage
# Shows storage controllers (NVMe, SATA, USB)

# Full HTML report (great for documentation):
sudo lshw -html > hardware_report.html

# JSON output for scripting:
sudo lshw -json | python3 -m json.tool | head -40

hwinfo: Alternative Hardware Probe

# Install:
sudo dnf install hwinfo

# Short overview:
hwinfo --short

# Specific device classes:
hwinfo --cpu
hwinfo --disk
hwinfo --netcard
hwinfo --gfxcard
hwinfo --sound

# Everything (verbose):
hwinfo --all

dmidecode: Hardware from SMBIOS/DMI

dmidecode reads the DMI (Desktop Management Interface) table, which motherboard firmware (BIOS/UEFI) provides:

# Install:
sudo dnf install dmidecode

# Full DMI table:
sudo dmidecode

# Specific types:
sudo dmidecode -t system     # Manufacturer, model, serial number
sudo dmidecode -t baseboard  # Motherboard info
sudo dmidecode -t memory     # RAM slots, installed modules, speeds
sudo dmidecode -t bios       # BIOS version, date, features
sudo dmidecode -t processor  # CPU socket, voltage, external clock

Example:

sudo dmidecode -t memory | head -25
# Handle 0x000C, DMI type 17, 92 bytes
# Memory Device
#   Array Handle: 0x000B
#   Error Information Handle: Not Provided
#   Total Width: 64 bits
#   Data Width: 64 bits
#   Size: 32 GB
#   Form Factor: SODIMM
#   Set: None
#   Locator: ChannelA-DIMM0
#   Bank Locator: BANK 0
#   Type: DDR5
#   Type Detail: Synchronous
#   Speed: 5600 MT/s
#   Manufacturer: Micron
#   Serial Number: 12345678
#   Asset Tag: Unknown
#   Part Number: MTF64C64T8DDR5-5600

lsmem: Memory Layout

# Physical memory layout:
lsmem
# Range                Node  Size     State     Removable
# 0x000000000-0x7ffffffff  0    128G     online   yes

# Online/offline memory regions (relevant for virtual machines):
lsmem --summary
# Memory block size:    1G
# Total online memory:  128G
# Total offline memory: 0B

free and /proc/meminfo: Memory Usage

# Human-readable memory usage:
free -h
#               total   used   free   shared  buff/cache  available
# Mem:           127Gi   12Gi   85Gi    2Gi     30Gi       110Gi
# Swap:          8Gi     0B     8Gi

# Detailed memory statistics:
cat /proc/meminfo
# MemTotal:       133681412 kB
# MemFree:        89123456 kB
# MemAvailable:   115234568 kB
# Buffers:         1234567 kB
# Cached:         30456789 kB
# SwapTotal:       8388608 kB
# ...

Quick Discovery Commands Summary
Command	Shows	Installed by Default
lspci	PCI devices (GPUs, network, audio, USB controllers)	Yes
lsusb	USB devices	Yes
lscpu	CPU architecture, cores, cache, virtualization	Yes
lsmem	Physical memory layout	Yes
free	Memory usage summary	Yes
lsblk	Block devices (disks, partitions)	Yes
lshw	Comprehensive hardware inventory	No (dnf install lshw)
hwinfo	Detailed hardware probe	No (dnf install hwinfo)
dmidecode	Motherboard/BIOS/RAM slot info from DMI	No (dnf install dmidecode)
inxi	Human-readable system summary	No (dnf install inxi)
3. Kernel Modules
What Are Kernel Modules?

Kernel modules are pieces of code that extend the kernel's capabilities without requiring a reboot. Most hardware drivers are modules — they're loaded when the hardware is detected and unloaded when no longer needed.

Think of modules as plugins for the kernel. The core kernel handles memory, scheduling, and filesystems. Modules add support for specific hardware: a NVIDIA GPU, a Realtek Wi-Fi card, a particular webcam.

Module files live in:

ls /lib/modules/$(uname -r)/
# build  kernel  modules.alias  modules.dep  modules.dep.bin  ...

The actual .ko (kernel object) files:

# Find a specific module file:
find /lib/modules/$(uname -r) -name "nouveau*"
# /lib/modules/6.19.0-50.fc44.x86_64/kernel/drivers/gpu/drm/nouveau/nouveau.ko.xz

# Check kernel version:
uname -r
# 6.19.0-50.fc44.x86_64

lsmod: Listing Loaded Modules

# List all currently loaded modules:
lsmod

Module                  Size  Used by
nvidia_drm             77824  2
nvidia_modeset       1269760  1 nvidia_drm
nvidia              35127296  86 nvidia_modeset
snd_hda_intel          53248  1
snd_hda_codec         184320  1 snd_hda_intel
i915                  2080768  3
nvme                   61440  3
xhci_hcd               196608  1 xhci_pci

Columns:

    Module — Driver name
    Size — Memory consumed (bytes)
    Used by — How many things depend on this module (0 = nothing depends on it; safe to unload)

# Check if a specific module is loaded:
lsmod | grep nvidia
# nvidia_drm  77824  2
# nvidia_modeset  1269760  1 nvidia_drm
# nvidia  35127296  86 nvidia_modeset

# Check if a module is loaded without grep noise:
lsmod | grep -w nouveau
# (empty output = not loaded)

modprobe: Loading and Unloading Modules

# Load a module (and its dependencies):
sudo modprobe nvidia

# Load with specific parameters:
sudo modprobe nouveau modeset=1

# Remove a module (and dependent modules):
sudo modprobe -r nvidia
# Will fail if something is using it

# Force removal (dangerous — can crash the system):
sudo rmmod --force nvidia

# Show what modprobe would do without doing it:
modprobe -v -n nvidia
# insmod /lib/modules/.../nvidia.ko.xz

    ⚠️ Never Force-Remove Active Drivers

    If a module's "Used by" count is > 0, something depends on it. Force-removing an active display driver will freeze your screen. Force-removing a filesystem driver can corrupt data. Always use modprobe -r (which refuses if dependencies exist) rather than rmmod --force.

modinfo: Module Details

# Show information about a module:
modinfo nvidia
# filename:    /lib/modules/.../nvidia.ko.xz
# version:     580.65.05
# license:     NVIDIA
# description: NVIDIA GPU driver
# author:      NVIDIA Corporation
# srcversion:  3B9AC0...
# depends:     drm,drm_kms_helper
# retpoline:   Y
# name:        nvidia
# vermagic:    6.19.0-50.fc44.x86_64 SMP mod_unload
# parm:        NVreg_DynamicPowerManagement:Dynamic power management mode (int)
# parm:        NVreg_PreserveVideoMemoryAllocations:Preserve video memory allocations (int)

# Find modules by pattern:
find /lib/modules/$(uname -r) -name "*.ko*" | xargs basename -a | sed 's/\.ko.*//' | sort | grep -i audio

# Check available parameters for a module:
modinfo -p nouveau
# modeset: bool
# atomic: int
# runpm: int
# vram_pushbuf: bool

# Show dependency chain:
modinfo --depends nvidia
# depends: drm,drm_kms_helper

The parm: lines show configurable parameters. These can be set:

# Temporary (until reboot):
sudo modprobe nouveau modeset=1

# Permanent — create a config file:
sudo nano /etc/modprobe.d/nouveau.conf
# Add: options nouveau modeset=1

Blacklisting Modules

Sometimes you need to prevent a module from loading — for example, to stop the open-source nouveau driver so you can use the proprietary NVIDIA driver instead:

# Create a blacklist file:
sudo nano /etc/modprobe.d/blacklist-nouveau.conf
# Add:
# blacklist nouveau
# options nouveau modeset=0

# Also blacklist in the initramfs (early boot):
sudo dracut --force

# Reboot for changes to take effect:
sudo reboot

    💡 Blacklist vs. Install

    The blacklist directive prevents automatic loading but doesn't prevent manual modprobe. To completely prevent a module from ever loading, add:

    install nouveau /bin/false

    This makes any attempt to load nouveau silently fail. Use this approach when replacing drivers (like nouveau → nvidia).

Automatic Module Loading

Modules are loaded automatically through several mechanisms:
Mechanism	When	Example
udev	When hardware is detected	Plug in webcam → udev loads uvcvideo
Kernel auto-loading	During boot based on detected hardware	GPU detected → kernel loads i915 or nouveau
modules-load.d	During boot, unconditionally	Always load vfio for virtualization passthrough
initramfs	Very early boot (before root filesystem)	Storage drivers needed to find root partition

# Configure unconditional module loading at boot:
sudo nano /etc/modules-load.d/custom.conf
# Add one module name per line:
# vfio
# vfio_iommu_type1
# vfio_pci

# Check what's configured to load at boot:
ls /etc/modules-load.d/

# Check module parameters configured at boot:
ls /etc/modprobe.d/
# blacklist-nouveau.conf
# nvidia.conf
# ...

Rebuilding the Module Dependency Tree

When you install new modules (like from RPM Fusion's NVIDIA packages), the module dependency database needs updating:

# Regenerate module dependencies:
sudo depmod -a

# This updates /lib/modules/$(uname -r)/modules.dep
# Usually handled automatically by package managers (dnf triggers depmod)

4. Firmware: The Software Inside Hardware
What Firmware Is and Why It Matters

Many hardware devices contain their own embedded processors — your Wi-Fi card, GPU, SSD, and even your trackpad. These processors run firmware: low-level code that controls the hardware's behavior.

Unlike drivers (which run on your CPU), firmware runs on the device itself. The kernel loads firmware into the device during initialization.

Without the correct firmware:

    Wi-Fi cards won't scan or connect
    GPUs render incorrectly or not at all
    Bluetooth adapters can't pair
    Audio chips produce no sound
    Some storage devices won't initialize

The linux-firmware Package

Fedora 44 ships firmware in the linux-firmware package, installed by default:

# Check if it's installed:
rpm -q linux-firmware
# linux-firmware-20260519-1.fc44.noarch

# See how many firmware files are available:
ls /lib/firmware/ | wc -l
# 8,500+ files

# Browse firmware for a specific vendor:
ls /lib/firmware/nvidia/
ls /lib/firmware/amdgpu/
ls /lib/firmware/iwlwifi-*    # Intel Wi-Fi
ls /lib/firmware/rtw88/       # Realtek Wi-Fi

When you update Fedora, linux-firmware updates regularly — the May 2026 update added firmware for Intel Panther Lake Wi-Fi, AMD GPU ASICs, Realtek Bluetooth, and more. These updates enable newly released hardware to function.
Checking Firmware Loading

# Watch firmware being loaded during boot:
dmesg | grep -i firmware
# [    2.134] iwlwifi 0000:00:14.3: loaded firmware version 89.3.4.1 8a83faa0.0
# [    3.456] amdgpu 0000:01:00.0: firmware: direct-loading firmware amdgpu/navi10_gpu_info.bin
# [    4.123] Bluetooth: hci0: RTL: loading rtl_bt/rtl8761bu_fw.bin

# Check if firmware was found or missing:
dmesg | grep -i "firmware"
# If you see: "Direct firmware load for X failed with error -2"
# That means the firmware file is missing!

fwupd: System Firmware Updates

fwupd is a daemon for updating system firmware (BIOS/UEFI, SSD firmware, dock firmware, etc.) from within Linux — no need to boot into Windows or use BIOS menus.

# Check fwupd status:
systemctl status fwupd
# Active: active (running)

# Refresh firmware database:
fwupdmgr refresh

# List devices that support firmware updates:
fwupdmgr get-devices
# (Lists motherboard/BIOS, SSDs, docks, Thunderbolt controllers, etc.)

# Check for available updates:
fwupdmgr get-updates

# Apply all available updates:
fwupdmgr update

# Update a specific device:
fwupdmgr update --plugins=uefi

fwupd integrates with GNOME Software and KDE Discover — firmware updates may appear in the same update interface as software updates.

# View detailed info about a specific device:
fwupdmgr get-devices --filter=type:system
# System Firmware:
#   Device ID:          a1b2c3d4...
#   Summary:             UEFI BIOS
#   Current version:     1.5.3
#   Vendor:              Lenovo (ME)
#   Update State:        success
#   Flags:               internal|updatable|supported-on-api
#   GUIDs:               ...

    💡 UEFI Capsule Updates

    fwupd uses UEFI capsule updates — a standard mechanism where the OS writes a firmware update file to a special EFI variable, then the UEFI firmware applies it during the next boot. This is why firmware updates require a reboot: the actual flashing happens before the OS loads.

    On some systems, you may need to disable Secure Boot or enroll the fwupd MOK key for capsule updates to work.

5. udev: Dynamic Device Management
What udev Does

udev is the device manager for the Linux kernel. When hardware is added or removed, the kernel generates uevents. udev listens for these events and:

    Creates or removes device nodes in /dev
    Applies persistent naming rules (so devices get consistent names)
    Sets file permissions on device nodes
    Executes custom rules (auto-mount drives, configure permissions, etc.)

# Check udev is running:
systemctl status systemd-udevd
# Active: active (running)

udevadm: Interacting with udev

# Monitor real-time device events:
# (Plug in a USB device while this is running to see events)
udevadm monitor
# KERNEL[12345.678] add      /devices/pci.../usb1/1-3 (usb)
# KERNEL[12345.679] add      /devices/pci.../usb1/1-3/1-3:1.0 (usb)
# UDEV  [12345.680] add      /devices/pci.../usb1/1-3 (usb)
# UDEV  [12345.681] add      /devices/pci.../usb1/1-3/1-3:1.0 (usb)

# Monitor with kernel and udev timing:
udevadm monitor --kernel --udev --subsystem-match=usb

# Query device info:
udevadm info --query=all --name=/dev/sda
# P: /devices/pci.../ata1/host0/target0:0:0/0:0:0:0/block/sda
# N: sda
# S: disk/by-id/ata-Samsung_SSD_870_EVO_500GB_S5...
# S: disk/by-path/pci-0000:00:17.0-ata-1
# E: ID_VENDOR=Samsung
# E: ID_MODEL=SSD_870_EVO_500GB
# E: ID_SERIAL=Samsung_SSD_870_EVO_500GB_S5...
# E: ID_FS_TYPE=ext4

# Trigger device re-detection (rescan):
sudo udevadm trigger

# Trigger specific subsystem:
sudo udevadm trigger --subsystem-match=net

# Test what rules apply to a device:
udevadm test /sys/class/net/enp3s0

Persistent Device Naming

One of udev's most important jobs is persistent naming — ensuring devices get the same name every boot, regardless of detection order.

Network interfaces:
Old Name	Persistent Name	Based On
eth0	enp3s0	PCI bus location (p3 = bus 3, s0 = slot 0)
wlan0	wlp2s0	Wireless, PCI bus 2, slot 0
eth1	eno1	Onboard (built-in) NIC

Storage devices:

# The same USB drive can appear at different /dev/sdX on different boots
# udev creates stable symlinks:
ls -la /dev/disk/by-id/
# lrwxrwxrwx  ... usb-Samsung_Flash_Drive_123456-0:0 → ../../sdb

ls -la /dev/disk/by-uuid/
# lrwxrwxrwx  ... a1b2c3d4-5678-... → ../../nvme0n1p2

ls -la /dev/disk/by-partlabel/
# lrwxrwxrwx  ... EFI\x20System\x20Partition → ../../nvme0n1p1

ls -la /dev/disk/by-path/
# pci-0000:00:17.0-ata-1 → ../../sda

This is why /etc/fstab uses UUIDs instead of /dev/sda1 — UUIDs are stable, device names aren't:

# fstab with UUIDs:
UUID=a1b2c3d4-5678-90ef-1234-567890abcdef  /  ext4  defaults  0 1

# NOT this (unreliable — could change between boots):
/dev/sda2  /  ext4  defaults  0 1

Custom udev Rules

You can write udev rules to automate actions when devices are connected:

# Built-in rules:
ls /usr/lib/udev/rules.d/     # System-provided (don't edit)
ls /etc/udev/rules.d/          # Local rules (your customizations)

Example rule — auto-mount a specific USB drive:

# Create a custom rule:
sudo nano /etc/udev/rules.d/99-usb-drive.rules

# Mount Samsung Flash Drive when connected:
ACTION=="add", SUBSYSTEM=="block", ENV{ID_SERIAL}=="Samsung_Flash_Drive_123456", RUN+="/usr/bin/mount /dev/%k /mnt/flash"
ACTION=="remove", SUBSYSTEM=="block", ENV{ID_SERIAL}=="Samsung_Flash_Drive_123456", RUN+="/usr/bin/umount /mnt/flash"

Reload rules:

sudo udevadm control --reload-rules
sudo udevadm trigger

    💡 Finding udev Attributes

    To write udev rules, you need to know what attributes a device has. Query it:

    udevadm info --attribute-walk --name=/dev/sdb
    # Shows all matching attributes you can use in rules:
    # ATTR{vendor}="Samsung"
    # ATTR{model}="Flash Drive"
    # ATTR{serial}="123456"

6. Graphics Drivers

Graphics drivers are the most common reason Linux users interact with the driver stack. The three GPU vendors have very different driver ecosystems on Linux.
Understanding the Linux Graphics Stack

Application (game, browser, desktop compositor)
        ↓
Mesa / Vulkan loader / OpenGL library
        ↓
DRM / KMS (Direct Rendering Manager / Kernel Mode Setting)
        ↓
Kernel module (driver): i915 / amdgpu / nouveau / nvidia
        ↓
Firmware (loaded into GPU by kernel)
        ↓
GPU Hardware

DRM (Direct Rendering Manager) is the kernel subsystem that arbitrates GPU access between applications. KMS (Kernel Mode Setting) means the kernel sets display resolutions and manages outputs, rather than X11 or Wayland doing it.
AMD Graphics (Easiest)

AMD provides fully open-source drivers integrated into the kernel:
Component	Source	Status
Kernel driver (amdgpu)	Mainline kernel	Built in
Mesa (OpenGL/Vulkan)	Open source	Installed by default
Firmware	linux-firmware	Installed by default

AMD GPUs work out of the box on Fedora 44. No additional installation needed.

# Verify the AMD driver is loaded:
lsmod | grep amdgpu
# amdgpu  1048576  5

# Check GPU info:
lspci -k | grep -A3 -i "vga\|3d\|display"
# Kernel driver in use: amdgpu

# Monitor GPU activity:
sudo dnf install radeontop
radeontop

Intel Graphics (Also Easy)

Intel's open-source driver is also built into the kernel:
Component	Source	Status
Kernel driver (i915 or xe)	Mainline kernel	Built in
Mesa (OpenGL/Vulkan)	Open source	Installed by default
Firmware	linux-firmware	Installed by default

Fedora 44 uses the newer xe driver for Intel GPUs from Tiger Lake onward (Gen12+), and i915 for older generations:

# Check which Intel driver is loaded:
lsmod | grep -E "i915|xe"
# i915  3145728  0

# For Intel Arc GPUs (discrete):
lsmod | grep xe
# xe  2097152  0

NVIDIA Graphics (Requires Work)

NVIDIA is unique among GPU vendors: their best driver is proprietary and closed-source. Fedora ships with the open-source nouveau driver, which works but lacks performance parity, lacks some power management features, and doesn't support CUDA.

Driver options for NVIDIA:
Driver	Source	Performance	CUDA	Recommendation
nouveau	Open source (in kernel)	Lower	No	Acceptable for basic use
nvidia (proprietary)	RPM Fusion	Full	Yes	Required for gaming/CUDA
nvidia-open	NVIDIA (partially open)	Near-full	Yes	Experimental on Fedora 44
Installing Proprietary NVIDIA Drivers via RPM Fusion

Step 1: Enable RPM Fusion repositories:

# Install RPM Fusion free and non-free:
sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Verify:
dnf repolist | grep rpmfusion
# rpmfusion-free
# rpmfusion-free-updates
# rpmfusion-nonfree
# rpmfusion-nonfree-updates

    ⚠️ Update First

    Before installing NVIDIA drivers, run sudo dnf upgrade and reboot. The NVIDIA driver module is compiled against your running kernel. If you install the driver on kernel A, then later update to kernel B, the module won't load on kernel B until it's rebuilt. Starting with a fully updated system avoids this mismatch.

Step 2: Identify your GPU:

# Find your NVIDIA GPU model:
lspci | grep -i nvidia
# 01:00.0 VGA compatible controller: NVIDIA Corporation GA104 [GeForce RTX 3060]

# Or:
sudo lshw -class display | grep -i product

Step 3: Install the driver package:

# For most modern NVIDIA GPUs (RTX 20-series and newer):
sudo dnf install akmod-nvidia

# For specific GPU generations (if the above doesn't match):
# RTX 40/50-series: akmod-nvidia (latest branch)
# GTX 10-series:   akmod-nvidia-535xx (older legacy branch)
# Check RPM Fusion's NVIDIA howto for your specific card:
# https://rpmfusion.org/Howto/NVIDIA

The akmod (AKMOD = Automated Kernel Module) system automatically compiles the NVIDIA driver whenever a new kernel is installed. This is crucial — without it, each kernel update would break your GPU driver.

Step 4: Install CUDA/Vulkan support (optional):

# For CUDA (machine learning, rendering):
sudo dnf install xorg-x11-drv-nvidia-cuda

# For video encoding (NVENC — used by OBS, ffmpeg):
sudo dnf install xorg-x11-drv-nvidia-cuda-libs

Step 5: Handle Secure Boot (if enabled):

# Check if Secure Boot is enabled:
mokutil --sb-state
# SecureBoot enabled

If Secure Boot is enabled, you must enroll the akmod MOK (Machine Owner Key) so the kernel trusts the compiled driver:

# Install mokutil if not present:
sudo dnf install mokutil

# Generate the MOK key (if not already done by akmods):
sudo mokutil --import /etc/pki/akmods/certs/public_key.der
# Enter a password (you'll use it during the next boot)
# The system will prompt you to enroll the key at boot time

# Reboot:
sudo reboot

During boot, a blue MOK Manager screen appears:

┌────────────────────────────────────────┐
│      Enroll MOK                       │
│                                      │
│   [ ] Continue                        │
│   [ ] Enroll MOK                     │  ← Select this
│   [ ] Enroll MOK from Disk            │
│                                      │
└────────────────────────────────────────┘

Select "Enroll MOK," enter the password you set, and the system continues booting with the NVIDIA driver loaded.

Step 6: Verify installation:

# Check the module loaded:
lsmod | grep nvidia
# nvidia_drm       77824  2
# nvidia_modeset  1269760  1 nvidia_drm
# nvidia          35127296  86 nvidia_modeset

# Verify the GPU is recognized:
nvidia-smi

Mon Jul  6 14:23:45 2026
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 580.65.05    Driver Version: 580.65.05   CUDA Version: 13.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
|   0  NVIDIA GeForce RTX 3060  | 00000000:01:00.0  On |                  N/A |
|  30%   35C    P8    12W / 170W|    132MiB / 12288MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+

# Confirm the driver replaced nouveau:
lspci -k | grep -A3 "NVIDIA"
# Kernel driver in use: nvidia   ← Success!
# Kernel modules: nouveau, nvidia, nvidia_drm

NVIDIA Troubleshooting
Symptom	Likely Cause	Fix
Black screen after reboot	Module not built or MOK not enrolled	Boot with nomodeset kernel param, rebuild: sudo akmods --force
"NVIDIA kernel module missing, falling back to nouveau"	Module not built for current kernel	sudo dnf reinstall akmod-nvidia && sudo akmods --force && sudo reboot
nvidia-smi shows "Failed to initialize NVML"	Driver/module version mismatch	Reinstall: sudo dnf reinstall akmod-nvidia, reboot
Screen tearing	Missing modesetting	sudo nano /etc/modprobe.d/nvidia.conf → options nvidia_drm modeset=1
High idle power consumption	Missing Dynamic Power Management	Add options nvidia NVreg_DynamicPowerManagement=0x02 to modprobe.d
CUDA not found	CUDA libraries not installed	sudo dnf install xorg-x11-drv-nvidia-cuda
MOK enrollment failed	Password expired (only valid for one boot)	Re-import key and reboot immediately

    💡 Recovering from a Black Screen

    If the NVIDIA driver leaves you with a black screen:

        Press Ctrl+Alt+F3 to switch to a TTY (text console)
        Log in with your username and password
        Remove the driver: sudo dnf remove akmod-nvidia xorg-x11-drv-nvidia*
        Reboot: sudo reboot
        System will fall back to nouveau

    Alternatively, add nomodeset to the kernel command line in GRUB for a one-time boot with no GPU driver, then troubleshoot from the desktop.

7. Audio Subsystem
The Linux Audio Stack (2026)

Fedora 44 uses PipeWire as its audio/video server, replacing the older PulseAudio:

Application (Firefox, Spotify, Discord)
        ↓
PipeWire (session manager: WirePlumber)
        ↓
ALSA (kernel audio subsystem)
        ↓
Kernel driver (snd_hda_intel, snd_soc, etc.)
        ↓
Firmware (DSP code loaded into audio chip)
        ↓
Audio hardware (speakers, headphones, mic)

Checking Audio Hardware

# List ALSA-recognized sound cards:
cat /proc/asound/cards
#  0 [Generic        ]: HDA-Intel - HD-Audio Generic
#                      HDA Intel PCH at 0xfd6c0000 irq 147

# List playback devices:
aplay -l
# card 0: Generic [HD-Audio Generic], device 0: ALC295 Analog
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0

# List recording devices:
arecord -l

# Check which kernel audio driver is loaded:
lsmod | grep snd
# snd_hda_intel       53248  1
# snd_hda_codec      184320  1 snd_hda_intel
# snd_hda_core        90112  2 snd_hda_intel,snd_hda_codec

# View ALSA mixer levels:
alsamixer
# Interactive TUI — use arrow keys, M to mute/unmute, ESC to exit

PipeWire Tools

# List PipeWire audio devices:
pw-cli list-objects
# Also: wpctl status (WirePlumber CLI)
wpctl status

Audio
 ├─ Devices:
 │   └─ alsa_card.pci-0000_00_1f.3
 │      ├─ Profiles: HiFi (Multimedia), Pro Audio, Off
 │      └─ Active Profile: HiFi (Multimedia)
 ├─ Sinks (outputs):
 │   └─ alsa_output.pci-0000_00_1f.3.HiFi__Speaker
 │      Description: Built-in Audio Stereo
 │      Mute: no
 │      Volume: 0.65
 ├─ Sources (inputs):
 │   └─ alsa_input.pci-0000_00_1f.3.HiFi__Headset_Mic
 │      Description: Built-in Audio Headset Microphone
 │      Mute: no
 │      Volume: 0.40

# Set default output device:
wpctl set-default <device-id>

# Change volume (percent):
wpctl set-volume @DEFAULT_AUDIO_SINK@ 50%

# Mute/unmute:
wpctl set-mute @DEFAULT_AUDIO_SINK@ toggle

# Test audio:
speaker-test -c 2  # Plays test tone on left/right speakers

8. Input Devices
Listing Input Devices

# List all input devices:
cat /proc/bus/input/devices

I: Bus=0011 Vendor=0001 Product=0001 Version=ab83
N: Name="AT Translated Set 2 keyboard"
P: Phys=isa0060/serio0/input0
H: Handlers=sysrq kbd event0 leds
B: PROP=0
B: EV=120013
B: KEY=...

I: Bus=0018 Vendor=06cb Product=ce5f Version=0100
N: Name="SYNA3602:00 06CB:CD8B Touchpad"
P: Phys=i2c-SYNA3602:00
H: Handlers=event4 mouse1
B: PROP=5
B: EV=b

# List evdev devices (event interfaces):
ls /dev/input/
# event0  event1  event2  event3  event4  mice  mouse0  mouse1

# Track input events in real time (useful for debugging):
sudo dnf install evemu
sudo evemu-record /dev/input/event4
# Move your touchpad — see raw event data stream

Trackpad Configuration

GNOME and KDE handle trackpad configuration through their respective settings panels. From the command line:

# Check if libinput (the input driver) is handling your trackpad:
libinput list-devices
# Device:           SYNA3602:00 06CB:CD8B Touchpad
# Kernel:           /dev/input/event4
# Group:            8
# Seat:             seat0, default
# Capabilities:     pointer
# Tap-to-click:     enabled
# Natural scrolling: enabled
# Click method:     button-areas

# Toggle tap-to-click via gsettings (GNOME):
gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true

# Toggle natural scrolling:
gsettings set org.gnome.desktop.peripherals.touchpad natural-scroll true

# Set scroll method:
gsettings set org.gnome.desktop.peripherals.touchpad scroll-method 'two-finger-scrolling'

9. Power Management and Battery
Viewing Power Information

# List power supplies:
ls /sys/class/power_supply/
# AC0  BAT0

# Battery status:
cat /sys/class/power_supply/BAT0/status
# Charging

cat /sys/class/power_supply/BAT0/capacity
# 87

cat /sys/class/power_supply/BAT0/capacity_level
# Normal

cat /sys/class/power_supply/BAT0/manufacturer
# SMP

cat /sys/class/power_supply/BAT0/model_name
# L20M3PG0

cat /sys/class/power_supply/BAT0/technology
# Li-poly

# Full power statistics:
upower -i /org/freedesktop/UPower/devices/battery_BAT0
# native-path:          BAT0
# vendor:               SMP
# model:                L20M3PG0
# serial:               12345
# power supply:         yes
# updated:              Mon 06 Jul 2026 02:34:56 PM UTC
# has history:          yes
# has statistics:       yes
# battery
#   present:             yes
#   rechargeable:        yes
#   state:               charging
#   warning level:       none
#   energy:              45.2 Wh
#   energy-empty:        0 Wh
#   energy-full:         52.0 Wh
#   energy-full-design:  56.0 Wh
#   capacity:            87%
#   cycle count:          124

CPU Frequency Scaling

# Available CPU governors (power profiles):
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
# performance powersave userspace schedutil

# Current governor:
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_governor
# powersave

# Current frequency:
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_cur_freq
# 2200000  (2.2 GHz)

Power Profiles Daemon

Fedora 44 uses power-profiles-daemon for simple power management:

# List available profiles:
powerprofilesctl list
# performance:
#     balanced:
#       Driver:     intel_pstate
#     power-saver:
#       Driver:     intel_pstate

# Check current profile:
powerprofilesctl get
# balanced

# Switch to performance mode:
powerprofilesctl set performance

# Switch to power saving:
powerprofilesctl set power-saver

# This integrates with GNOME's power settings (battery icon → power mode)

Profile	Behavior	Use Case
performance	Higher CPU clocks, more power draw	Gaming, compiling, heavy workloads
balanced	Adaptive scaling	Daily use
power-saver	Lower clocks, reduced background activity	Battery conservation
Thermal Management

# List thermal zones:
ls /sys/class/thermal/
# thermal_zone0  thermal_zone1  cooling_device0

# Check temperatures (millidegrees Celsius):
for zone in /sys/class/thermal/thermal_zone*; do
  echo "$zone: $(cat $zone/temp | awk '{print $1/1000}')°C ($(cat $zone/type))"
done
# /sys/class/thermal/thermal_zone0: 45°C (acpitz)
# /sys/class/thermal/thermal_zone1: 42°C (x86_pkg_temp)

# Check cooling devices:
cat /sys/class/thermal/cooling_device0/type
# Processor

cat /sys/class/thermal/cooling_device0/cur_state
# 0  (0 = no throttling; higher = more throttling)

# Monitor temperature continuously:
watch -n 1 'cat /sys/class/thermal/thermal_zone*/temp'

# For a friendly temperature monitor:
sudo dnf install lm_sensors
sensors
# coretemp-isa-0000
# Package id 0:  +45.0°C  (high = +105.0°C, crit = +135.0°C)
# Core 0:        +42.0°C
# Core 1:        +44.0°C
# ...

Sensor Detection (lm_sensors)

# Run sensor detection (interactive — answers what hardware you have):
sudo sensors-detect
# This probes your motherboard for sensor chips
# It will ask questions — answer "yes" to auto-detection
# At the end, it asks to write config to /etc/sysconfig/lm_sensors

# After detection:
sensors

10. Hardware Troubleshooting Methodology
The Diagnostic Toolkit

Follow this systematic approach when hardware misbehaves:
Step 1: Does the Kernel See It?

# Is the device enumerated on the bus?
lspci | grep -i nvidia        # For PCIe devices
lsusb                          # For USB devices

# Does the kernel have a driver loaded?
lspci -k | grep -A3 "VGA"     # Shows driver in use

# Any errors in the kernel ring buffer?
dmesg | grep -i error
dmesg | grep -i fail
dmesg | grep -i firmware

Step 2: Is the Module Loaded?

# Check if the module is loaded:
lsmod | grep <module_name>

# If not loaded, try loading it:
sudo modprobe <module_name>

# Check for loading errors:
dmesg | tail -20

Step 3: Is the Firmware Available?

# Check for firmware loading errors:
dmesg | grep -i "firmware"

# Check if the firmware file exists:
find /lib/firmware -name "*<device_keyword>*"

# If missing, ensure linux-firmware is up to date:
sudo dnf upgrade linux-firmware

Step 4: Does the Device Node Exist?

# Check /dev for the expected device:
ls /dev/ | grep <expected_name>

# Check /sys for device attributes:
ls /sys/class/<subsystem>/

# Check udev's view:
udevadm info --query=all --name=/dev/<device>

Step 5: Are Permissions Correct?

# Check device node permissions:
ls -la /dev/<device>

# Ensure user is in the right group:
groups   # Check membership
# john disk wheel video audio input

# Add user to a group if needed:
sudo usermod -aG video john
# Log out and back in for changes to take effect

Step 6: What Does the Log Say?

# Kernel messages (hardware, driver, firmware issues):
dmesg | tail -50

# Journalctl (broader system logs):
journalctl -b -p err          # Errors from current boot
journalctl -b -k             # Kernel messages from current boot
journalctl -u systemd-udevd  # udev events
journalctl -f                # Follow logs live

dmesg: The Kernel Ring Buffer

dmesg displays the kernel ring buffer — a circular log of kernel messages since boot. It's the primary tool for hardware diagnosis:

# All kernel messages (most recent at bottom):
dmesg

# Messages since boot, filtered by severity:
dmesg --level=err,warn

# Filter by keyword:
dmesg | grep -i usb
dmesg | grep -i nvidia
dmesg | grep -i thermal
dmesg | grep -i "I/O error"
dmesg | grep -i "hardware error"

# Follow new messages in real time:
sudo dmesg -w

# Show messages from a specific boot:
journalctl -b -k
# Same kernel messages via journalctl

# Timestamp humanization:
dmesg -T
# [Mon Jul  6 14:23:45 2026] USB device connected

Common dmesg patterns for hardware diagnosis:
Pattern	Meaning
firmware: direct-loading firmware X	Firmware loaded successfully ✓
Direct firmware load for X failed with error -2	Firmware file missing ✗
DRM: failed to load gpu firmware	GPU firmware missing
device descriptor read/64, error -110	USB device timeout
I/O error, dev sda, sector 12345	Storage device failure
machine check exception	CPU hardware error (serious)
thermal: thermal_zone0: critical temperature reached	Overheating emergency
nouveau: GPU lockup	NVIDIA/nouveau driver crash
xHCI host controller not responding	USB controller failure
Kernel Parameters for Debugging

When troubleshooting boot or driver issues, you can pass parameters to the kernel via GRUB:

# Edit GRUB entry temporarily:
# Boot → press 'e' on the kernel entry → add to 'linux' line:

Parameter	Effect	Use Case
nomodeset	Disable kernel mode setting (KMS)	Black screen / GPU driver conflict
nouveau.modeset=0	Disable nouveau KMS	Installing NVIDIA proprietary driver
nvidia.modeset=1	Enable NVIDIA KMS	Prevent screen tearing
modprobe.blacklist=nouveau	Prevent nouveau loading	Replacing drivers
acpi=off	Disable ACPI	ACPI-related crashes
pci=noaer	Suppress PCI AER error messages	Noisy non-critical PCI errors
loglevel=8	Show all kernel messages	Detailed boot debugging
systemd.log_level=debug	Verbose systemd logging	Boot/service debugging
Quick Diagnostic Cheat Sheet

# Comprehensive hardware snapshot:
echo "=== CPU ===" && lscpu | head -15
echo "=== MEMORY ===" && free -h
echo "=== PCI DEVICES ===" && lspci
echo "=== USB DEVICES ===" && lsusb
echo "=== BLOCK DEVICES ===" && lsblk
echo "=== NETWORK ===" && ip -br a
echo "=== LOADED MODULES ===" && lsmod | head -20
echo "=== KERNEL ERRORS ===" && dmesg --level=err,warn | tail -20
echo "=== TEMPERATURES ===" && sensors 2>/dev/null || echo "lm_sensors not installed"
echo "=== POWER ===" && upower -i /org/freedesktop/UPower/devices/battery_BAT0 2>/dev/null | grep -E "state|capacity|energy" || echo "No battery"

Try It Yourself
Exercise 1: Hardware Inventory

# Create a complete hardware report:
sudo lshw -html > ~/hardware_report.html
# Open in a browser: xdg-open ~/hardware_report.html

# Save text summary:
sudo lshw -short > ~/hardware_summary.txt
cat ~/hardware_summary.txt

# Get motherboard info:
sudo dmidecode -t baseboard

# Get BIOS info:
sudo dmidecode -t bios

Exercise 2: Module Exploration

# List loaded modules (sorted by size):
lsmod | sort -k2 -rn | head -20

# Find the largest loaded module:
lsmod | sort -k2 -rn | head -1

# Get details about your GPU driver:
modinfo $(lspci -k | grep -A1 "VGA" | grep "Kernel driver" | awk '{print $NF}') 2>/dev/null

# Find modules related to your network card:
lspci -k | grep -A3 -i "ethernet\|network" | grep "Kernel modules"

Exercise 3: sysfs Exploration

# Find your CPU's available frequencies:
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies 2>/dev/null

# Check current temperatures:
for zone in /sys/class/thermal/thermal_zone*; do
  temp=$(cat $zone/temp 2>/dev/null)
  type=$(cat $zone/type 2>/dev/null)
  echo "$type: $(echo "scale=1; $temp/1000" | bc)°C"
done

# List all sysfs device classes:
ls /sys/class/

# Examine your primary disk:
ls /sys/block/nvme0n1/ 2>/dev/null || ls /sys/block/sda/
cat /sys/block/nvme0n1/device/model 2>/dev/null || cat /sys/block/sda/device/model 2>/dev/null

Exercise 4: USB Device Investigation

# List all USB devices:
lsusb

# Get a hierarchical view:
lsusb -t

# Monitor what happens when you plug/unplug a device:
udevadm monitor --subsystem-match=usb

# While monitoring is running, plug in a USB device
# Watch the kernel and udev events flow

# Query details about a specific device:
# (Replace with a device ID from lsusb)
udevadm info --attribute-walk --name=/dev/sdb 2>/dev/null

Exercise 5: NVIDIA Driver Check (if applicable)

# Check what GPU you have:
lspci | grep -iE "vga|3d|display"

# See which driver is currently in use:
lspci -k | grep -A3 -iE "vga|3d|display"

# If you have NVIDIA:
lsmod | grep -E "nvidia|nouveau"

# If nvidia module is loaded:
nvidia-smi 2>/dev/null

# Check firmware loading:
dmesg | grep -i "nvidia\|nouveau\|gpu" | tail -10

Exercise 6: Audio Diagnosis

# List sound cards:
cat /proc/asound/cards

# List audio drivers:
lsmod | grep snd

# Check PipeWire status:
wpctl status 2>/dev/null | head -30

# Test speakers:
speaker-test -c 2 -t sine -f 440 -l 1
# Plays a 1-second 440Hz tone on both channels

Exercise 7: Temperature and Power

# Install lm_sensors:
sudo dnf install lm_sensors

# Detect sensors (answer yes to auto-probing):
sudo sensors-detect

# View temperatures:
sensors

# Check power profile:
powerprofilesctl get

# Check battery health:
upower -i /org/freedesktop/UPower/devices/battery_BAT0 2>/dev/null | grep -E "state|capacity|energy-full"

Exercise 8: Firmware Check

# Check fwupd is running:
systemctl status fwupd

# List devices with firmware update support:
fwupdmgr get-devices

# Refresh firmware catalog:
fwupdmgr refresh

# Check for updates:
fwupdmgr get-updates

Troubleshooting Reference
Symptom	Likely Cause	First Check
No display after boot	GPU driver issue	Boot with nomodeset, check dmesg
Wi-Fi not working	Missing firmware or wrong driver	dmesg | grep -i firmware, lsmod | grep iwl
Bluetooth not detected	Missing firmware	dmesg | grep -i bluetooth, lsmod | grep btusb
No audio	Wrong output selected or driver issue	wpctl status, lsmod | grep snd
Touchpad not working	Missing libinput driver or kernel driver	libinput list-devices, dmesg | grep -i touch
USB device not detected	Power issue or port failure	lsusb, dmesg | tail, try different port
SSD not appearing	NVMe/SATA driver missing	lsblk, lspci | grep -i nvme, dmesg | grep -i nvme
Random freezes/reboots	Hardware fault (thermal, RAM, power)	sensors, dmesg --level=err, memtester
NVIDIA "falling back to nouveau"	Module not built / MOK not enrolled	sudo akmods --force, enroll MOK
Camera not working	uvcvideo driver missing or privacy shutter	ls /dev/video*, dmesg | grep -i uvc
Keyboard backlight not working	Missing system driver	ls /sys/class/leds/, dmidecode -t system
Overheating	Dust, thermal paste, or fan failure	sensors, dmesg | grep -i thermal
High power consumption	No power management or rogue process	powerprofilesctl get, top, powertop
What's Next

You now understand how Fedora 44 manages hardware: the layered model (hardware → firmware → kernel modules → subsystems → udev → /dev → applications), the /sys filesystem for real-time device attribute inspection, the /dev device node system with major/minor numbers and character vs. block devices, comprehensive hardware discovery tools (lspci for PCI bus enumeration with -k driver attribution, lsusb with -t tree view, lscpu for CPU architecture, lshw with -short/-class/-html output, hwinfo for probing, dmidecode for SMBIOS/DMI motherboard/RAM/BIOS data, lsmem and free for memory), kernel modules (lsmod listing with Size/Used-by columns, modprobe loading/unloading with dependencies, modinfo for metadata and parameters, insmod/rmmod low-level alternatives, blacklisting via /etc/modprobe.d/, automatic loading via udev/modules-load.d/initramfs, depmod for dependency regeneration), firmware (linux-firmware package contents in /lib/firmware/, dmesg firmware loading messages, fwupd with fwupdmgr for UEFI capsule updates, MOK key enrollment for Secure Boot), udev (systemd-udevd daemon, udevadm monitor for real-time events, info query, trigger rescans, persistent naming for network/storage devices, custom rules in /etc/udev/rules.d/, attribute-walk for rule writing), graphics drivers (DRM/KMS architecture, AMD amdgpu full open-source, Intel i915/xe full open-source, NVIDIA proprietary via RPM Fusion with akmod-nvidia, MOK enrollment, CUDA/NVENC extras, nouveau fallback, black screen recovery, troubleshooting table), audio subsystem (PipeWire replacing PulseAudio, WirePlumber session manager, ALSA kernel layer, aplay/arecord/alsamixer, wpctl commands, speaker-test), input devices (/proc/bus/input/devices, libinput, evemu-record, GNOME gsettings for trackpad), power management (/sys/class/power_supply, upower battery info, power-profiles-daemon with performance/balanced/power-saver profiles, CPU frequency scaling governors, thermal zones, lm_sensors with sensors-detect), systematic troubleshooting methodology (six-step from bus enumeration through module loading, firmware availability, device node existence, permissions, log analysis, dmesg kernel ring buffer patterns, kernel parameter debugging via GRUB), progressive exercises from hardware inventory through module exploration, sysfs investigation, USB device monitoring, NVIDIA verification, audio diagnosis, temperature monitoring, and firmware updates, plus a comprehensive thirteen-entry troubleshooting reference table.

Chapter 23 dives into process management — how Linux runs programs, schedules CPU time, handles signals, and how you can inspect and control what's happening on your system. You'll learn about ps, top, htop, kill, killall, pkill, nice/renice, jobs, fg/bg, and systemd-cgls. These are the tools that let you understand (and tame) what your system is actually doing at any moment.

Until then, survey your hardware. Run the diagnostic cheat sheet on your machine. Knowing what "healthy" looks like — your temperatures, your loaded modules, your dmesg at boot — makes diagnosing problems dramatically easier when they arise.

✅ Chapter 22 Complete You understand the Linux hardware stack from silicon to application (layered model: hardware → firmware → kernel modules → kernel subsystems like DRM/ALSA/net → udev device management → /dev device nodes → applications), /sys filesystem (virtual filesystem exposing device attributes as readable/writable files, /sys/class organization by type, /sys/block for storage, /sys/devices for physical topology, reading temperatures/brightness/MAC addresses/battery capacity directly), /dev device nodes (character vs block device types, major/minor numbers for driver routing, persistent naming via udev symlinks in /dev/disk/by-id/by-uuid/by-partlabel/by-path), hardware discovery tools (lspci with -k for driver attribution and -mm machine-readable output, lsusb with -t tree hierarchy and vendor/product IDs, lscpu for CPU architecture with cores/threads/cache/virtualization fields, lshw -short/-class/-html/-json for comprehensive inventory, hwinfo for detailed probing, dmidecode -t for SMBIOS/DMI motherboard/RAM/BIOS/processor data, lsmem and free for memory layout, lsblk for block devices), kernel modules (.ko files in /lib/modules/$(uname -r)/, lsmod with Module/Size/Used-by columns showing dependency counts, modprobe for loading with dependencies and -r for removal, modinfo for metadata/version/license/parameters/depends fields, blacklisting via /etc/modprobe.d/ with blacklist directive vs install /bin/false for hard block, automatic loading via udev hardware detection / modules-load.d unconditional loading / initramfs early boot, depmod -a for dependency tree regeneration, AKMOD system for auto-compiling modules on kernel updates), firmware (linux-firmware package with 8,500+ files in /lib/firmware/, dmesg firmware loading messages with "direct-loading firmware" success and "Direct firmware load failed" errors, fwupd daemon with fwupdmgr refresh/get-devices/get-updates/update workflow, UEFI capsule updates requiring reboot, MOK key enrollment for Secure Boot), udev (systemd-udevd daemon processing kernel uevents, udevadm monitor for real-time add/remove events, udevadm info --query for device properties, udevadm trigger for rescans, persistent naming ensuring consistent enp3s0/wlp2s0/eno1 interface names and /dev/disk/by-uuid stability, custom rules in /etc/udev/rules.d/ with ACTION/SUBSYSTEM/ATTR matchers and RUN+ actions, udevadm info --attribute-walk for discovering rule attributes), graphics drivers (DRM/KMS architecture where kernel manages display modesetting, AMD amdgpu fully open-source built-in with Mesa, Intel i915 for older Gen11 and below / xe for Tiger Lake+ and Arc discrete, NVIDIA proprietary via RPM Fusion with akmod-nvidia for automatic kernel module compilation, MOK enrollment process with mokutil --import and blue boot screen, CUDA/NVENC optional packages, nouveau open-source fallback, nomodeset kernel parameter for black screen recovery, comprehensive NVIDIA troubleshooting table for seven common symptoms), audio subsystem (PipeWire replacing PulseAudio with WirePlumber session manager, ALSA kernel layer with snd_hda_intel/snd_hda_codec modules, aplay -l/arecord -l for device listing, alsamixer TUI, wpctl status/set-default/set-volume/set-mute commands, speaker-test for verification), input devices (/proc/bus/input/devices listing with Name/Handlers/Bus/Vendor/Product fields, libinput as the unified input driver, evemu-record for raw event debugging, GNOME gsettings for trackpad tap-to-click/natural-scroll/scroll-method configuration), power management (/sys/class/power_supply for battery status/capacity/technology/manufacturer, upower -i for detailed battery statistics including cycle count and energy-full-design degradation, power-profiles-daemon with performance/balanced/power-saver profiles via powerprofilesctl, CPU frequency scaling governors in /sys/devices/system/cpu/cpufreq/, thermal zones in /sys/class/thermal with millidegree temperatures and cooling devices, lm_sensors package with sensors-detect probing and sensors output), systematic troubleshooting methodology (six steps: bus enumeration with lspci/lsusb → module verification with lsmod/modprobe → firmware availability check with dmesg/find /lib/firmware → device node existence in /dev and /sys/class → permission verification with ls -la and groups → log analysis with dmesg and journalctl), dmesg kernel ring buffer (primary hardware diagnostic tool, --level=err,warn filtering, -T human timestamps, -w follow mode, common diagnostic patterns for firmware failure/USB timeout/I/O error/thermal critical/GPU lockup), kernel parameters for debugging (nomodeset, nouveau.modeset=0, nvidia.modeset=1, modprobe.blacklist, acpi=off, pci=noaer, loglevel=8, systemd.log_level=debug via GRUB 'e' edit), progressive exercises from hardware inventory through module exploration, sysfs investigation, USB device monitoring, NVIDIA verification, audio diagnosis, temperature monitoring, and firmware updates, plus a comprehensive thirteen-entry troubleshooting reference table.

