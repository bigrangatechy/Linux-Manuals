# Chapter 22: Hardware and Drivers

## What's Inside Your Machine

When you installed Ubuntu, the installer detected your hardware and loaded the appropriate drivers automatically. For most systems, this works flawlessly — your keyboard, trackpad, Wi-Fi, audio, and display all work without you doing anything. This is one of Linux's great achievements: the kernel includes thousands of drivers built right in.

But not every piece of hardware has an open-source driver built into the kernel. Some manufacturers — particularly NVIDIA — keep their best drivers proprietary. Others release driver updates that arrive after Ubuntu's release date. And sometimes hardware misbehaves for reasons that require investigation.

This chapter teaches you how to inventory your hardware, check which drivers are in use, install proprietary drivers when needed, manage kernel modules, and diagnose hardware problems — all from the terminal.

## How Ubuntu Handles Drivers

Unlike Windows, where you install drivers separately for each device (downloading `.exe` files from manufacturer websites), Ubuntu handles drivers differently:

1. **Built-in kernel drivers** — Most drivers are compiled directly into the Linux kernel. When the kernel boots, it detects hardware and loads the appropriate driver automatically. No installation needed.

2. **Kernel modules** — Some drivers are compiled as **modules** — separate files that the kernel loads on demand. Think of modules as drivers that can be loaded or unloaded without rebooting.

3. **Proprietary driver packages** — A few drivers (NVIDIA GPUs, some Wi-Fi chips) are closed-source and can't be included in the kernel by default. Ubuntu packages these as `.deb` packages you install through APT.

The result: on most systems, you never think about drivers. On systems with NVIDIA GPUs or certain Wi-Fi chips, you install one package and you're done.

## Hardware Inventory Commands

Before installing or troubleshooting drivers, you need to know what hardware you have. Ubuntu provides several tools for this.

### `lscpu` — CPU Information

lscpu

Shows your processor's specifications:

Architecture:            x86-64
  CPU op-mode(s):        32-bit, 64-bit
  Address sizes:         46 bits physical, 48 bits virtual
  Byte order:            Little Endian
CPU(s):                  8
  On-line CPU(s) list:   0-7
Vendor ID:               GenuineIntel
  Model name:            Intel(R) Core(TM) i7-12700K
    CPU family:          6
    Model:              151
    Thread(s) per core:  2
    Core(s) per socket:  4
    Socket(s):           1
    Stepping:            2
    CPU max MHz:         5100.0000
    CPU min MHz:         800.0000
    BogoMIPS:            5080.00
    L1d cache:           48 KiB (8 instances)
    L1i cache:           32 KiB (8 instances)
    L2 cache:            1.25 MiB (8 instances)
    L3 cache:            12 MiB (1 instance)

Key fields:
Field	Meaning
CPU(s)	Total logical cores (cores × threads per core)
Model name	Processor model
Thread(s) per core	Hyperthreading (2 = enabled, 1 = disabled or not supported)
Core(s) per socket	Physical cores per CPU
Socket(s)	Number of physical CPU packages
CPU max/min MHz	Clock speed range
L1d/L1i/L2/L3 cache	Cache memory sizes (larger = generally faster)
lspci — PCI Devices

PCI (Peripheral Component Interconnect) is the bus that connects graphics cards, network cards, Wi-Fi adapters, USB controllers, and other internal devices to your system.

lspci

Output:

00:00.0 Host bridge: Intel Corporation 12th Gen Core Processor Host Bridge
00:02.0 VGA compatible controller: Intel Corporation AlderLake-S GT1
00:14.0 USB controller: Intel Corporation Alder Lake-S USB Controller
00:1f.3 Audio device: Intel Corporation Alder Lake-S HD Audio
01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060] (rev a1)
01:00.1 Audio device: NVIDIA Corporation GA106 High Definition Audio (rev a1)
03:00.0 Network controller: Intel Corporation Wi-Fi 6E AX211

Each line shows a device's PCI address, class, and description.

Show which kernel driver is bound to each device:

lspci -k

Output:

00:02.0 VGA compatible controller: Intel Corporation AlderLake-S GT1
        DeviceName: Onboard - Video
        Subsystem: Micro-Star International Co., Ltd. Device 7D80
        Kernel driver in use: i915
        Kernel modules: i915

The Kernel driver in use: i915 line tells you the Intel graphics driver (i915) is actively driving this device. If you see Kernel driver in use: nouveau for an NVIDIA card, the open-source Nouveau driver is being used instead of the proprietary NVIDIA driver.

Verbose output for a specific device:

lspci -v -s 01:00.0

Shows detailed information about the NVIDIA GPU at PCI address 01:00.0, including memory regions, capabilities, and driver details.
lsusb — USB Devices

lsusb

Lists all connected USB devices:

Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 001 Device 003: ID 0bda:5688 Realtek Semiconductor Corp. USB Webcam
Bus 001 Device 002: ID 05e3:0610 Generic Multi-Card Reader
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

Each line shows bus number, device number, vendor/product ID, and device name.

Verbose USB information:

lsusb -v

Detailed output — usually piped through less or grep since it's very long:

lsusb -v | less

lsblk — Block Devices (Disks and Partitions)

lsblk

Shows all storage devices in a tree structure:

NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    0 465.8G  0 disk
├─sda1        8:1    0   512M  0 part /boot/efi
├─sda2        8:2    0   100G  0 part /
�└─sda3        8:3    0   365G  0 part /home
sr0          11:0    1  1024M  0 rom
nvme0n1     259:0    0 238.5G  0 disk
├─nvme0n1p1 259:1    0   512M  0 part
└─nvme0n1p2 259:2    0   238G  0 part

Column	Meaning
NAME	Device name
SIZE	Total size
RO	Read-only (1 = read-only, 0 = read-write)
TYPE	disk = physical drive, part = partition, rom = optical drive
MOUNTPOINTS	Where the device is mounted in the filesystem

Show filesystem types:

lsblk -f

Adds filesystem type and UUID columns — useful for identifying what filesystem each partition uses (ext4, btrfs, ntfs, fat32, etc.).
lshw — Comprehensive Hardware Inventory

lshw (list hardware) pulls data from multiple sources — DMI tables, PCI bus, USB bus, kernel interfaces — to produce a complete hardware inventory.

Install if not present:

sudo apt install lshw

Short summary:

sudo lshw -short

Output:

H/W path         Device      Class      Description
=====================================================
                             system     Micro-Star International Co. MS-7D80
/0                         bus        PRO Z690-A WIFI (MS-7D80)
/0/0                       memory     128KiB BIOS
/0/1                       memory     16GiB System Memory
/0/1/0                     memory     8GiB DIMM DDR4 Synchronous
/0/1/1                     memory     8GiB DIMM DDR4 Synchronous
/0/2                       cpu        Intel Core i7-12700K (12th gen)
/0/100                     bridge     12th Gen Core Processor Host Bridge
/0/100/1                   display    AlderLake-S GT1
/0/100/14                  bus        Alder Lake-S USB Controller
/0/100/14.3                network    Wi-Fi 6E AX211
/0/100/1f.3               multimedia  Alder Lake-S HD Audio
/0/1                       storage    NVMe SSD Controller SM981
/0/2                       storage    SATA controller
/1             enp3s0      network    Ethernet controller I225-V

Full hardware details by class:

sudo lshw -class display

Shows detailed information about display devices (GPUs) only — including vendor, product, driver, clock speeds, and memory.

Other useful classes:

sudo lshw -class network    # Network cards
sudo lshw -class storage    # Storage controllers
sudo lshw -class memory     # RAM modules
sudo lshw -class processor  # CPU details

    💡 lshw Requires sudo

    lshw needs sudo to access low-level hardware information. Without it, you'll get incomplete output. Similarly, dmidecode (below) always requires sudo because it reads BIOS/DMI tables.

dmidecode — BIOS-Level Hardware Details

While lshw queries the running system, dmidecode reads the SMBIOS/DMI tables — hardware information stored in your motherboard's firmware. This gives you serial numbers, motherboard models, memory slot details, and BIOS versions.

sudo dmidecode -t system

Output:

BIOS Information
        Vendor: American Megatrends International, LLC.
        Version: 1.40
        Release Date: 03/15/2026
        ROM Size: 32 MB

System Information
        Manufacturer: Micro-Star International Co., Ltd.
        Product Name: PRO Z690-A WIFI (MS-7D80)
        Serial Number: To be filled by O.E.M.
        UUID: 03000200-0400-0500-0006-0070-8B2DFE91A3B2
        Wake-up Type: Power Switch

Check memory modules:

sudo dmidecode -t memory

Shows each RAM stick individually — including slot number, type (DDR4/DDR5), speed, manufacturer, serial number, and capacity. Useful for upgrading RAM or diagnosing bad modules.

Processor details:

sudo dmidecode -t processor

Common dmidecode types:
Type Code	Shows
bios	BIOS vendor, version, release date
system	Manufacturer, model, serial number
baseboard	Motherboard info
processor	CPU sockets, cores, speeds
memory	RAM modules and slots
slot	Expansion slots (PCIe, etc.)
inxi — Human-Friendly System Summary (Third-Party)

inxi produces a colorful, readable system summary that's easier to parse than raw hardware tools:

sudo apt install inxi
inxi -Fxz

Output:

System:
  Host: ubuntu-desktop Kernel: 7.0.0-14-generic arch: x86-64 bits: 64
  Desktop: GNOME v: 50.0 Distro: Ubuntu 26.04 LTS (Resolute Raccoon)
Machine:
  Type: Desktop Mobo: Micro-Star model: PRO Z690-A WIFI
  BIOS: AMI v: 1.40 date: 03/15/2026
CPU:
  Info: 8-core Intel Core i7-12700K
  Speed: 4800 MHz min/max: 800/5100
Graphics:
  Device-1: Intel AlderLake-S GT1 driver: i915
  Device-2: NVIDIA GA106 [GeForce RTX 3060] driver: nvidia
Network:
  Device-1: Intel Wi-Fi 6E AX211 driver: iwlwifi
  Device-2: Intel I225-V driver: igc
Drives:
  Local Storage: total: 704.25 GB
  ID-1: /dev/nvme0n1 vendor: Samsung model: SM981 size: 238.5 GB
  ID-2: /dev/sda vendor: Crucial model: MX500 size: 465.8 GB

inxi -Fxz is the command most Linux forum helpers ask you to run when diagnosing hardware issues — it gives a complete system snapshot in one shot. The -F flag means "full output," -x adds extra detail, and -z hides private information like MAC addresses.
Checking Which Drivers Are in Use
lspci -k — Driver per PCI Device

As shown above, lspci -k reveals which kernel driver is bound to each PCI device. This is the quickest way to check if your NVIDIA GPU is using the proprietary driver or the open-source Nouveau:

lspci -k | grep -A 3 -i vga

00:02.0 VGA compatible controller: Intel Corporation AlderLake-S GT1
        Kernel driver in use: i915
        Kernel modules: i915
--
01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060]
        Kernel driver in use: nvidia
        Kernel modules: nvidia, nouveau

Here, the NVIDIA GPU shows Kernel driver in use: nvidia — the proprietary driver is active. If it said Kernel driver in use: nouveau, the open-source driver would be active instead.
lsmod — Loaded Kernel Modules

lsmod

Lists all currently loaded kernel modules (drivers loaded as modules, not built into the kernel):

Module                  Size   Used by
nvidia               24883712  120
nvidia_modeset        1228800  2 nvidia
nvidia_uvm            1265664   0
nvidia_drm             65536   1
i915                 2021376  10
snd_hda_intel          57344   5
iwlwifi               290816   0

The Used by column shows how many other modules depend on this one. A value of 0 means nothing else depends on it — it could theoretically be unloaded.

Filter for a specific driver:

lsmod | grep nvidia

nvidia               24883712  120
nvidia_modeset        1228800  2 nvidia
nvidia_uvm            1265664   0
nvidia_drm             65536   1

If you see these nvidia_* modules loaded, the proprietary NVIDIA driver is active.

Check if Nouveau (open-source NVIDIA) is loaded:

lsmod | grep nouveau

If no output, Nouveau isn't loaded — good if you're using the proprietary driver. If Nouveau IS loaded alongside NVIDIA, that can cause conflicts.
modinfo — Module Details

modinfo nvidia

Shows detailed information about a kernel module:

filename:       /lib/modules/7.0.0-14-generic/updates/dkms/nvidia.ko
version:        595.58.03
license:        NVIDIA
author:         NVIDIA Corporation
description:    NVIDIA Linux kernel module
depends:        
name:           nvidia
vermagic:       7.0.0-14-generic SMP mod_unload modversions

Key field: version — tells you the exact driver version installed. Useful when troubleshooting compatibility issues.
Managing Kernel Modules

Kernel modules are driver files located in /lib/modules/$(uname -r)/. The modprobe command loads and unloads them.
Loading a Module

sudo modprobe nvidia

Loads the nvidia module into the running kernel. Takes effect immediately — no reboot needed.
Unloading a Module

sudo modprobe -r nouveau

The -r flag removes (unloads) the nouveau module. Useful when switching from Nouveau to the proprietary NVIDIA driver.
Listing Module Options

modprobe -c | grep nvidia

Shows current configuration options for the NVIDIA module, including any custom parameters set in /etc/modprobe.d/.
Blacklisting Modules

Sometimes you need to prevent a module from loading at boot — for example, preventing Nouveau from loading so it doesn't conflict with the proprietary NVIDIA driver.

Create a blacklist file:

echo "blacklist nouveau" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo "options nouveau modeset=0" | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf

After blacklisting, update the initial RAM disk so the blacklist takes effect at boot:

sudo update-initramfs -u

Reboot for changes to take full effect.

    💡 What Are initramfs and update-initramfs?

    initramfs (Initial RAM Filesystem) is a small temporary filesystem loaded into memory at boot. It contains the drivers needed to access your real filesystem — disk controllers, filesystem drivers, and essential modules. When you blacklist a module or install a new driver, you need to rebuild the initramfs so the boot process picks up the changes:

    sudo update-initramfs -u

    The -u flag means "update" — it rebuilds the initramfs for your current kernel.

Installing Proprietary Drivers

Most hardware works with built-in open-source drivers. The main exceptions are NVIDIA GPUs and some Wi-Fi chips. Ubuntu provides a tool to manage proprietary driver installation.
ubuntu-drivers — The Easy Way

Ubuntu includes a tool called ubuntu-drivers that detects your hardware and recommends appropriate proprietary drivers.

See what drivers are available for your hardware:

ubuntu-drivers devices

Output:

== /sys/devices/pci0000:00/0000:00:01.0/0000:01:00.0 ==
modalias : pci:v000010DEd00002504sv00001462sd00003913bc03sc00i00
vendor   : NVIDIA Corporation
model    : GA106 [GeForce RTX 3060]
driver   : nvidia-driver-580-open - distro non-free
driver   : nvidia-driver-595-open - distro non-free recommended
driver   : nvidia-driver-595 - distro non-free
driver   : xserver-xorg-video-nouveau - distro free builtin

This shows:

    The detected NVIDIA GPU
    Available driver packages, including version numbers
    Which one is recommended (marked with recommended)
    Whether each driver is free (open-source) or non-free (proprietary)
    Whether it's builtin (already in the kernel) or a package to install

Auto-install the recommended driver:

sudo ubuntu-drivers autoinstall

This installs the recommended proprietary driver automatically. After installation, reboot:

sudo reboot

Install a specific driver version:

sudo apt install nvidia-driver-595-open

Choose a specific version if you have reasons to prefer one over the recommended (e.g., a known bug in the newest version, or a feature only in an older one).
NVIDIA Driver Variants on Ubuntu 26.04

Ubuntu 26.04 offers two variants of NVIDIA drivers:
Variant	Package Name	Description
Open	nvidia-driver-595-open	NVIDIA's open-source kernel modules with proprietary userspace. Recommended for most users.
Closed	nvidia-driver-595	Fully proprietary (traditional). Use if the open variant has issues with your specific GPU.

The -open variant uses NVIDIA's open GPU kernel modules. These have been the default recommendation on Ubuntu since they reached feature parity with the closed modules. For most modern GPUs (RTX 20-series and newer), the open variant works identically.

    💡 The Software & Updates GUI Is Gone

    Previous Ubuntu versions included a graphical "Additional Drivers" tool in the Software & Updates app, where you could select NVIDIA drivers from a list. Ubuntu 26.04 dropped this tool from new installs. The ubuntu-drivers command is now the primary way to manage proprietary drivers.

    If you really want the old GUI back:

    sudo apt install software-properties-gtk

    But honestly, ubuntu-drivers devices gives you the same information faster, and sudo ubuntu-drivers autoinstall is quicker than clicking through a GUI.

Verifying NVIDIA Driver Installation

After installing and rebooting:

Check if the module loaded:

lsmod | grep nvidia

You should see nvidia, nvidia_modeset, nvidia_drm, and nvidia_uvm.

Check the NVIDIA driver status:

nvidia-smi

Output:

Sat Jul  4 14:32:07 2026
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 595.58    Driver Version: 595.58.03   CUDA Version: 13.0        |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC|
| Fan  Temp  Perf  Pwr:Usage/Cap|        Memory-Usage | GPU-Util  Compute M.|
|                               |                      |               MIG M.|
|===============================+======================+======================|
|   0  GeForce RTX 3060    Off  | 00000000:01:00.0  On |                  N/A|
| 30%   45C    P0    72W / 170W |    512MiB / 12288MiB |     5%      Default |
+-------------------------------+----------------------+----------------------+

nvidia-smi is the definitive NVIDIA diagnostic tool. It shows:

    Driver version (595.58.03)
    CUDA version supported (13.0)
    GPU model (GeForce RTX 3060)
    Temperature (45°C — should be under 85°C under load)
    Power usage (72W / 170W max)
    VRAM usage (512 MiB / 12288 MiB total)
    GPU utilization (5%)

Check the kernel driver binding:

lspci -k | grep -A 3 -i nvidia

Should show Kernel driver in use: nvidia (not nouveau).
Known NVIDIA Issues on Ubuntu 26.04

    ⚠️ Suspend/Resume Hang with NVIDIA on Kernel 7.0

    A known regression affects Ubuntu 26.04's kernel 7.0 with the NVIDIA open-source driver modules. On some laptops (notably Framework 16 and others using s2idle suspend), the system hangs on lid-close suspend and fails to resume — requiring a hard power-button reset.

    The issue is in the NVIDIA open kernel module's PCI suspend path interacting with kernel 7.0's power management changes. It does not occur on the older 6.17 kernel.

    Workarounds:

        Switch to the 6.17 HWE kernel (if available for your hardware):

        sudo apt install linux-image-6.17.0-generic
        sudo reboot

        Then select 6.17 from the GRUB boot menu.

        Blacklist NVIDIA modules during suspend: Create /etc/systemd/system/nvidia-suspend-fix.service:

        [Unit]
        Description=Unload NVIDIA modules before suspend
        Before=suspend.target

        [Service]
        Type=oneshot
        ExecStart=/bin/modprobe -r nvidia nvidia_drm nvidia_modeset nvidia_uvm
        ExecStop=/bin/modprobe nvidia nvidia_drm nvidia_modeset nvidia_uvm

        [Install]
        WantedBy=suspend.target

        Then:

        sudo systemctl enable nvidia-suspend-fix.service

    Check the Ubuntu bug tracker and NVIDIA open-gpu-kernel-modules GitHub for fixes — this regression is actively tracked (Launchpad bug #2149963).

AMD and Intel Graphics

AMD and Intel GPUs use open-source drivers that are built into the kernel — no proprietary installation needed:
GPU Brand	Driver (built-in)	Verification
Intel	i915	lsmod | grep i915
AMD	amdgpu	lsmod | grep amdgpu
AMD (older)	radeon	lsmod | grep radeon

If you have an AMD or Intel GPU, everything works out of the box. If you experience graphical glitches or poor performance, it's usually a kernel version issue — try the latest HWE kernel or check for a kernel update.
Hardware Logs with dmesg

The kernel logs every hardware event as it happens: driver loading, device detection, errors, and warnings. dmesg (device message) shows these logs.
Viewing Hardware Messages

dmesg | less

Scrolls through all kernel ring buffer messages. Use Spacebar to page forward, q to quit.

Recent messages only:

dmesg | tail -30

Filter for a specific device:

dmesg | grep -i nvidia

Filter for errors:

dmesg --level=err,warn

Shows only error and warning messages — useful for spotting hardware problems.
Following Hardware Events in Real Time

dmesg -w

The -w flag (follow) continuously displays new kernel messages as they're generated — like journalctl -f for kernel events. Press Ctrl+C to stop.

Plug in a USB device while watching dmesg -w to see the kernel detect it in real time:

[  123.456789] usb 1-3: new high-speed USB device number 5 using xhci_hcd
[  123.578901] usb 1-3: New USB device found, idVendor=046d, idProduct=c52b
[  123.578902] usb 1-3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[  123.578903] usb 1-3: Product: USB Receiver
[  123.678904] input: Logitech USB Receiver as /devices/pci0000:00/...
[  123.690123] hid-generic 0003:046D:C52B.0006: input,hidraw3: USB HID

This shows the kernel detecting the USB device, identifying it as a Logitech receiver, and loading the appropriate input driver — all within milliseconds.
Common dmesg Messages
Message	Meaning
New USB device found	A USB device was detected
input: Device as /devices/...	An input device (keyboard, mouse) was registered
EXT4-fs (sda2): mounted filesystem	A filesystem was mounted
DRM: [GPU] initializing	A GPU driver initialized
NVRM: loading NVIDIA driver	NVIDIA driver loaded successfully
nvidia: module verification failed	NVIDIA module signature mismatch — usually harmless on non-Secure-Boot systems
I/O error, dev sda	Disk I/O failure — hardware problem
ata1: link is slow to respond	Storage device responding slowly — failing drive symptom
thermal: CPU temperature above threshold	Overheating — check cooling

    💡 dmesg vs. journalctl

    dmesg shows the kernel ring buffer — messages from the kernel itself about hardware events. journalctl -k shows the same thing through systemd's journal system. They overlap heavily:

        dmesg — Direct kernel messages, always available
        journalctl -k -b — Kernel messages from the current boot, integrated with other system logs

    Use dmesg for quick hardware diagnosis. Use journalctl -k when you need to correlate kernel events with service events.

Storage Health Monitoring
SMART with smartctl

In Chapter 16, we introduced smartmontools for disk health. Let's review with more detail:

sudo apt install smartmontools

Quick health check:

sudo smartctl -H /dev/sda

Output:

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

Detailed report:

sudo smartctl -a /dev/sda

Key things to look for in the full report:
Attribute	Good Value	Concerning Value
SMART overall-health	PASSED	FAILED — replace immediately
Reallocated_Sector_Ct	0	Any non-zero — drive has bad sectors
Power_On_Hours	(informational)	Very high + errors = aging drive
Temperature_Celsius	Under 40°C	Over 50°C — cooling problem
UDMA_CRC_Error_Count	0	Non-zero — faulty SATA cable

Run a self-test:

sudo smartctl -t short /dev/sda

Starts a short self-test (takes 1-2 minutes). Check results afterward:

sudo smartctl -a /dev/sda | grep -A 5 "Self-test"

Practical Scenario: Diagnosing a Non-Working Wi-Fi Adapter

Let's walk through a realistic hardware troubleshooting scenario using the tools from this chapter.

Symptom: Wi-Fi doesn't work after a fresh Ubuntu 26.04 install.

Step 1: Check if the interface exists:

ip link

If no wireless interface (wlp... or wlan...) appears, the kernel doesn't recognize the Wi-Fi adapter.

Step 2: Identify the Wi-Fi chip:

lspci | grep -i network

03:00.0 Network controller: Intel Corporation Wi-Fi 6E AX211

Or for USB Wi-Fi adapters:

lsusb | grep -i wireless

Step 3: Check which driver is loaded:

lspci -k -s 03:00.0

03:00.0 Network controller: Intel Corporation Wi-Fi 6E AX211
        Kernel driver in use: iwlwifi
        Kernel modules: iwlwifi

If Kernel driver in use: is empty, no driver is loaded.

Step 4: Check for driver errors:

dmesg | grep -i iwl

Look for error messages like firmware not found or failed to load. If the firmware is missing:

sudo apt install linux-firmware
sudo modprobe -r iwlwifi
sudo modprobe iwlwifi

Step 5: Check if firmware files exist:

ls /lib/firmware/ | grep iwl

If firmware files are missing, installing the linux-firmware package should fix it.

Step 6: Check for hardware switch:

nmcli radio wifi

If it says disabled, turn it on:

nmcli radio wifi on

Some laptops have a physical Wi-Fi switch or a keyboard Fn combo (Fn+F2, etc.) that disables the radio at the hardware level. Check your laptop's manual.

Step 7: Scan for networks:

nmcli device wifi list

If networks appear, the adapter works — try connecting:

nmcli device wifi connect "YourNetwork" password "yourpassword"

Hardware Management Cheat Sheet
Task	Command
CPU info	lscpu
List PCI devices	lspci
PCI devices with drivers	lspci -k
List USB devices	lsusb
List block devices	lsblk
Block devices with filesystem	lsblk -f
Hardware summary	sudo lshw -short
Hardware by class	sudo lshw -class display
BIOS/system info	sudo dmidecode -t system
Memory module details	sudo dmidecode -t memory
Full system summary	inxi -Fxz
List loaded modules	lsmod
Module details	modinfo modulename
Load a module	sudo modprobe modulename
Unload a module	sudo modprobe -r modulename
Blacklist a module	echo "blacklist name" | sudo tee /etc/modprobe.d/file.conf
Rebuild initramfs	sudo update-initramfs -u
Available drivers	ubuntu-drivers devices
Auto-install recommended driver	sudo ubuntu-drivers autoinstall
Install specific NVIDIA driver	sudo apt install nvidia-driver-595-open
Check NVIDIA status	nvidia-smi
Kernel messages	dmesg | less
Kernel errors only	dmesg --level=err,warn
Follow kernel messages	dmesg -w
Disk health check	sudo smartctl -H /dev/sdX
Disk full report	sudo smartctl -a /dev/sdX
Disk self-test	sudo smartctl -t short /dev/sdX
What's Next

You can now inventory your complete hardware system, check which drivers are in use, install proprietary drivers (especially NVIDIA), manage kernel modules with modprobe, blacklist conflicting drivers, read kernel hardware logs with dmesg, monitor disk health with SMART, and diagnose hardware problems systematically. This is the toolkit that experienced Linux users reach for whenever "something doesn't work."

In Chapter 23, we'll cover process management — understanding how Linux handles running programs, managing background processes, using jobs, fg, bg, nohup, and killing unresponsive applications with precision.
Try It Yourself

    Inventory your system. Run lscpu, lsblk, and lspci. Identify your CPU model, number of cores, storage devices, and PCI devices. You now know more about your hardware than most users ever do.

    Check your drivers. Run lspci -k. For each device, note the Kernel driver in use line. If you have an NVIDIA GPU, check whether it says nvidia or nouveau.

    List loaded modules. Run lsmod | head -20. See how many kernel modules are active. Try lsmod | grep -i video to find display-related modules.

    Get a full system snapshot. Install and run inxi -Fxz. Save the output to a file for future reference:

    sudo apt install inxi
    inxi -Fxz > ~/system-info.txt

    Check for available drivers. Run ubuntu-drivers devices. See if Ubuntu recommends any proprietary drivers for your hardware. If it recommends an NVIDIA driver and you haven't installed it, consider doing so with sudo ubuntu-drivers autoinstall.

    Watch hardware events. Run dmesg -w in one terminal. In another terminal (or by plugging in a USB device), trigger a hardware event. Watch the kernel detect the device in real time.

    Check disk health. If you have smartmontools installed, run sudo smartctl -H /dev/sda (or your primary disk). Confirm the result says PASSED.

    Examine your RAM. Run sudo dmidecode -t memory. Identify how many RAM sticks you have, their capacity, speed, and slot positions.

    ✅ Chapter 22 Complete You can inventory hardware with lscpu, lspci, lsusb, lsblk, lshw, dmidecode, and inxi. You can check which drivers are active with lspci -k and lsmod. You can install proprietary drivers with ubuntu-drivers, manage kernel modules with modprobe, blacklist conflicting drivers, read hardware logs with dmesg, and monitor disk health with SMART. In Chapter 23, we'll master process management.

