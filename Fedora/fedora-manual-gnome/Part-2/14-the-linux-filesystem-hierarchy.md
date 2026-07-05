# Chapter 14: The Linux Filesystem Hierarchy

## Everything Is a File

If you remember one thing about Linux from this entire manual, let it be this: **everything is a file.**

On Windows, hardware devices, configuration data, running processes, and network connections are all managed through different APIs — the Registry, the Device Manager, COM objects, Win32 calls. On Linux, they're all files. Your keyboard is a file. Your SSD is a file. A running process's memory map is a file. The kernel's CPU information is a file. Network sockets are files.

This "everything is a file" philosophy — inherited from Unix — means you can interact with the entire system using the same tools you use to read and write text files. `cat`, `less`, `grep`, `echo` — these commands work on documents, hardware, processes, and kernel state alike.

But for this to make sense, you need to understand the **map** — the directory structure that organizes all these files into a coherent, navigable tree. That's the Filesystem Hierarchy Standard, and it's the subject of this chapter.

---

## The Single Tree: No Drive Letters

The most jarring difference for Windows migrants is the absence of drive letters. On Windows:

C:\Users\John\Documents D:\Photos E:\Backup

Each drive gets its own letter and its own root. C:\ and D:\ are completely separate trees.

On Linux, there is one tree. One root. One starting point:

/

Everything — every drive, every partition, every USB stick, every network share — hangs off this single root. A second physical drive doesn't become D:; it gets mounted at a directory within the tree, like /mnt/data or /home or /var.

/
├── home/
│   └── john/
│       └── Documents/     ← could be on a separate SSD
├── var/
│   └── log/               ← could be on a separate HDD
├── mnt/
│   └── backup/            ← external USB drive mounted here
└── ...

The mount system makes physical storage transparent. You navigate from /home/john/Documents to /var/log using cd /var/log — no need to know or care whether those directories are on the same physical disk. The filesystem abstracts the hardware.
Mount Points

A mount point is a directory where a filesystem is attached. When you insert a USB drive, Fedora automatically mounts it at /run/media/username/DRIVE_LABEL/. The files on the USB drive appear under that directory as if they'd always been there.

# See all mounted filesystems:
findmnt

# Or the classic:
mount | column -t

Output shows device, mount point, filesystem type, and options:

SOURCE                     TARGET          FSTYPE  OPTIONS
/dev/nvme0n1p3             /               btrfs   rw,relatime,compress=zstd:1
/dev/nvme0n1p3[/home]      /home           btrfs   rw,relatime,compress=zstd:1
/dev/nvme0n1p2             /boot           ext4    rw,relatime
/dev/nvme0n1p1             /boot/efi       vfat    rw,relatime,fmask=0077
tmpfs                      /tmp            tmpfs   rw,nosuid,nodev
tmpfs                      /run            tmpfs   rw,nosuid,nodev,size=4096M

    💡 What Happened to My Drive?

    If you plug in a USB drive and don't see it, check:

    lsblk          # Shows block devices and their mount points

    If the drive appears in lsblk but has no mount point, GNOME usually auto-mounts it. If not:

    # Find the device name (e.g., /dev/sdb1)
    lsblk

    # Create a mount point and mount manually:
    sudo mkdir /mnt/usb
    sudo mount /dev/sdb1 /mnt/usb

    Your files are now at /mnt/usb/. Unmount before removing:

    sudo umount /mnt/usb

The Root Directory: /

The top of the tree. Every file and directory on your system lives somewhere under /. Let's explore each top-level directory.

ls /

On Fedora 44, you'll see:

afs  bin  boot  dev  etc  home  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

Let's go through each one.
/bin → /usr/bin (The /usr Merge)

Purpose: Essential user command binaries.

Historically, /bin held fundamental commands needed even in single-user or recovery mode: ls, cp, mv, cat, bash, rm. /usr/bin held the rest of the user-installed applications.

On Fedora 44 (and all modern Fedora releases), the /usr merge is complete. /bin is now a symlink to /usr/bin:

ls -ld /bin
# lrwxrwxrwx. 1 root root 7 May 10 09:23 /bin -> usr/bin

Same for /sbin, /lib, and /lib64:

ls -ld /bin /sbin /lib /lib64
# /bin   -> usr/bin
# /sbin  -> usr/sbin
# /lib   -> usr/lib
# /lib64 -> usr/lib64

    💡 Why the /usr Merge?

    In the 1970s, Unix systems had small root disks and large /usr disks. /bin held the minimum needed to boot; /usr/bin held everything else. This made sense when a root disk was 5 MB.

    Today, this split causes problems: package managers must split binaries between two directories, and read-only root filesystems (used in containers and atomic desktops) need everything on one partition. The merge puts all binaries in /usr/bin and makes /bin a symlink for backwards compatibility.

    Fedora completed the merge years ago. The symlinks exist so old scripts and documentation that reference /bin/ls still work — but the actual file lives at /usr/bin/ls.

What Lives in /usr/bin?

Thousands of executables. Every command you've learned so far:

which ls        # /usr/bin/ls
which bash      # /usr/bin/bash
which firefox   # /usr/bin/firefox
which dnf       # /usr/bin/dnf

That's over 4,000 binaries on a typical Fedora 44 installation:

ls /usr/bin | wc -l
# Output: 4572 (your number will vary)

/sbin → /usr/sbin (System Binaries)

Purpose: System administration binaries — commands typically run by root or with sudo.

ls -ld /sbin
# /sbin -> usr/sbin

Examples of commands in /usr/sbin:

ls /usr/sbin | head -20
# Output: addpart, blkid, chcpu, crond, dnf5, fdisk, fsck, grub2-mkconfig, ...

These are tools like fdisk (partition management), fsck (filesystem check), grub2-mkconfig (bootloader configuration), and visudo (sudoers editor). Regular users can execute some of them (like fdisk -l to view partitions), but most require root privileges.

    💡 The bin/sbin Distinction Is Fading

    Fedora 41 unified /usr/bin and /usr/sbin into a single directory. So on Fedora 44, /usr/sbin may itself be merged into /usr/bin. The symlink structure preserves compatibility, but the physical separation is largely gone.

    In practice, you don't need to worry about whether a command is in bin or sbin — your $PATH includes both, and the shell finds commands regardless.

/boot

Purpose: Files needed to boot the system.

/boot/
├── efi/              ← EFI System Partition (mounted at /boot/efi)
│   └── EFI/
│       └── fedora/
│           ├── grubx64.efi      ← GRUB bootloader
│           └── shimx64.efi      ← Secure Boot shim
├── grub2/
│   └── grub.cfg       ← GRUB configuration
├── vmlinuz-6.19.0-50.fc44.x86_64     ← Linux kernel image
├── initramfs-6.19.0-50.fc44.x86_64.img  ← Initial RAM filesystem
└── ...

File	What It Is
vmlinuz-*	The compressed Linux kernel. Each kernel version has its own file.
initramfs-*.img	The initial RAM filesystem — loaded into memory before the real root filesystem is mounted. Contains drivers and tools needed to find and mount the root partition.
grub.cfg	The GRUB boot menu configuration. Generated by grub2-mkconfig.
efi/	The EFI System Partition — contains the bootloader executable that UEFI firmware runs directly.

Fedora keeps previous kernels installed as a safety net. If a new kernel causes problems, you select the older one from the GRUB menu. This is why you might see multiple vmlinuz-* and initramfs-* files:

ls /boot/vmlinuz-*
# vmlinuz-6.19.0-50.fc44.x86_64
# vmlinuz-6.19.0-55.fc44.x86_64  ← newer, installed via dnf upgrade

    ⚠️ Don't Manually Edit grub.cfg

    The grub.cfg file is generated by grub2-mkconfig. Manual edits are overwritten on the next kernel update. To make persistent GRUB changes (like default timeout, kernel parameters), edit /etc/default/grub and then regenerate:

    sudo grub2-mkconfig -o /boot/grub2/grub.cfg

/dev — Device Files

Purpose: Device nodes representing hardware and virtual devices.

In Linux, "everything is a file" — and /dev is where the hardware lives as files.

ls /dev | head -30

autofs           cpu_dma_latency  full    hugepages  loop0  ...
block/           disk/            fuse    hwrng      loop1  ...
bsg/             dm-0             hidraw0  input/     loop2  ...
char/            dm-1             hidraw1  kmsg       loop3  ...
console          dri/             hpet    log        mapper/ ...
cpu/             dvd              hugepages/ lp0       mcelog  ...

Important Device Files
Device File	Represents
/dev/nvme0n1	Your NVMe SSD (whole disk)
/dev/nvme0n1p1	First partition on the SSD
/dev/nvme0n1p2	Second partition
/dev/sda	A SATA drive (if present)
/dev/sdb1	First partition on second SATA drive
/dev/null	The "black hole" — discards anything written to it
/dev/zero	Infinite stream of zero bytes
/dev/random	Random number generator
/dev/urandom	Faster (less cryptographically secure) random
/dev/stdin	Standard input (keyboard, pipe)
/dev/stdout	Standard output (screen, pipe)
/dev/stderr	Standard error (screen, pipe)
/dev/dri/card0	Graphics card (DRM)
/dev/video0	Webcam
/dev/input/event*	Keyboard, mouse, touchpad input events
Using Device Files

# Discard output (send to /dev/null):
dnf search something 2>/dev/null    # Suppress error messages

# Generate a 1 MB file of zeros:
dd if=/dev/zero of=test.bin bs=1M count=1

# Generate a random 32-character password:
cat /dev/urandom | tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1

# Check GPU:
ls -la /dev/dri/
# crw-rw----. 1 root video 226, 0 Jul 5 09:00 card0
# crw-rw----. 1 root video 226, 128 Jul 5 09:00 renderD128

    💡 Device Files Are Created Dynamically

    The /dev directory on modern Linux is managed by udev (part of systemd). Device files are created and removed automatically as hardware is connected and disconnected. You don't need to manually create device nodes — when you plug in a USB drive, udev creates /dev/sdb1 (or similar) within milliseconds.

/etc — Configuration Files

Purpose: System-wide configuration files. Host-specific settings.

The name /etc has a debated etymology — some say it stood for "et cetera" (everything else), but on modern systems it's commonly understood as "Editable Text Configuration" — because that's exactly what's here: text files you can edit to configure the system.

ls /etc | head -30

adjtime         dnsmasq.conf    hosts           localtime    os-release
aliases         dnf/            hostname        login.defs   pacman.conf
bashrc          environment     hosts.allow     machine-id   passwd
cron.d/         fedora-release  hosts.deny      mime.types   passwd-
...

Important Configuration Files
File	Purpose
/etc/fstab	Filesystem table — defines what gets mounted at boot
/etc/hostname	Computer's hostname
/etc/hosts	Static hostname-to-IP mappings (like a local DNS)
/etc/passwd	User account database (not actual passwords — see below)
/etc/shadow	Encrypted password hashes (readable only by root)
/etc/group	Group definitions
/etc/sudoers	sudo configuration (edit with visudo, never directly)
/etc/dnf/dnf.conf	DNF5 configuration (max_parallel_downloads, etc.)
/etc/os-release	Distribution identity (version, name, codename)
/etc/default/grub	GRUB default settings (timeout, kernel parameters)
/etc/sysconfig/	Legacy system configuration scripts
/etc/systemd/	systemd service configuration
/etc/selinux/	SELinux policy configuration
/etc/firewalld/	Firewall configuration
/etc/ssh/	SSH server/client configuration
/etc/cron.d/, /etc/crontab	Scheduled tasks
/etc/environment	System-wide environment variables
/etc/bashrc	System-wide bash shell configuration
/etc/profile	System-wide login shell configuration
/etc/resolv.conf	DNS resolver configuration (managed by systemd-resolved)
/etc/locale.conf	System locale and language
Viewing System Identity

cat /etc/os-release

NAME="Fedora Linux"
VERSION="44 (Forty Four)"
ID=fedora
VERSION_ID=44
PRETTY_NAME="Fedora Linux 44 (Forty Four)"
ANSI_COLOR="0;38;2;60;110;180"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:44"
HOME_URL="https://fedoraproject.org/"

    💡 Plain Text Configuration

    Nearly every file in /etc is a plain text file. No binary blobs, no Registry hive, no proprietary format. You can read configuration with cat, edit with any text editor, and back up with cp. This is one of Linux's greatest strengths — the entire system configuration is transparent, human-readable, and scriptable.

    That said, never edit /etc/shadow or /etc/sudoers directly. Use passwd to change passwords and visudo to edit sudo configuration. These files have strict syntax requirements that, if violated, can lock you out of your system.

/home — User Home Directories

Purpose: Per-user personal files, settings, and data.

/home/
├── john/                    ← User "john"'s home directory
│   ├── Desktop/
│   ├── Documents/
│   ├── Downloads/
│   ├── Music/
│   ├── Pictures/
│   ├── Videos/
│   ├── .bashrc              ← Bash configuration (personal)
│   ├── .config/             ← Application configuration (XDG)
│   ├── .local/              ← Application data and state
│   ├── .cache/              ← Cached data (safe to delete)
│   └── .ssh/                ← SSH keys and configuration
└── mary/                    ← User "mary"'s home directory
    └── ...

Each user gets a directory under /home/ matching their username. On Fedora's default Btrfs layout, /home is a Btrfs subvolume — it shares storage space with / but can be snapshotted independently.
The XDG Base Directory Specification

Modern Linux applications follow the XDG Base Directory Specification — a standard that defines where applications store configuration, data, and cache files:
Path	Purpose	Example Contents
~/.config/	Application configuration files	~/.config/obs-studio/, ~/.config/flatpak/
~/.local/share/	Application data (persistent)	~/.local/share/TelegramDesktop/, ~/.local/share/flatpak/
~/.cache/	Cached data (regenerable, safe to delete)	~/.cache/thumbnails/, ~/.cache/mozilla/
~/.local/state/	Application state (session, history)	~/.local/state/bash/, ~/.local/state/vim/

Older applications (pre-XDG) dump dot files directly in ~:

~/.bashrc
~/.bash_history
~/.vimrc
~/.gitconfig
~/.mozilla/
~/.ssh/

Over time, the home directory accumulates dot files. List them:

ls -la ~ | grep "^\."

    💡 Dot Files Are Hidden by Default

    Files starting with . are hidden. Use ls -a to see them. In GNOME Files, press Ctrl+H to toggle visibility. These are configuration files and directories — they store your personal settings for applications.

    Never delete dot files you don't understand. Some are critical (.bashrc, .ssh/, .config/). Deleting .config/ wipes all your application settings.

The Root User's Home

The root user doesn't have a directory under /home/. Root's home is /root/:

sudo ls /root/

This is separate from regular user homes for security reasons — if /home is on a separate partition and unmounted, root can still access its own files.
/lib and /lib64 → /usr/lib and /usr/lib64

Purpose: Shared libraries (like Windows DLLs or macOS dylibs).

Part of the /usr merge — /lib is a symlink to /usr/lib:

ls -ld /lib /lib64
# /lib   -> usr/lib
# /lib64 -> usr/lib64

These directories contain shared libraries that multiple programs use simultaneously:

ls /usr/lib64 | grep libssl
# libssl.so.3
# libssl.so.3.x.x

You generally don't interact with these directly. Package managers install and manage them. But understanding they exist helps when troubleshooting "library not found" errors:

# If an app fails with "cannot open shared object file":
ldd /usr/bin/some-app    # Shows which libraries an app needs

/media and /mnt — Mount Points

Purpose: Directories for mounting external filesystems.
Directory	Usage
/media/	Automatic mount point for removable media (USB drives, SD cards, optical discs). GNOME auto-mounts to /run/media/username/LABEL/.
/mnt/	Manual mount point for temporary filesystem mounts. Used by administrators when manually mounting drives, network shares, or temporary filesystems.

# When you plug in a USB drive, GNOME mounts it at:
ls /run/media/$USER/
# Kingston64/  SanDisk128/

    💡 The Difference Between /media and /run/media

    The FHS specifies /media for removable media. In practice, systemd and GNOME mount removable media under /run/media/username/LABEL/ instead. /run is a tmpfs (in-memory filesystem), so these mount points disappear on reboot — which is correct behavior for removable media.

    /media still exists for compatibility but is typically empty on Fedora 44.

/opt — Optional Software

Purpose: Third-party, self-contained application packages.

Software installed in /opt doesn't follow the standard /usr/bin, /usr/lib layout. Instead, each application gets its own directory tree:

/opt/
├── google/
│   └── chrome/
│       ├── google-chrome
│       └── ...
├── teamviewer/
│   └── tv_bin/
│       └── teamviewer
└── vscode/
    └── bin/
        └── code

This approach is used by vendors who want their software to be self-contained — everything (binaries, libraries, resources) lives under one directory. Fedora itself doesn't install anything in /opt by default. It's populated by third-party installers.

    💡 Flatpak and /opt

    Flatpak applications don't use /opt — they install to ~/.local/share/flatpak (user) or /var/lib/flatpak (system). The /opt directory is primarily for legacy vendor installers that predate Flatpak.

/proc — Process and Kernel Information

Purpose: Virtual filesystem exposing kernel and process information as files.

/proc doesn't exist on your disk. It's a virtual filesystem — the kernel generates its contents on the fly when you read from it. Every time you cat a file in /proc, the kernel computes the answer in real time.

ls /proc | head -20

1      1243   2267   cpuinfo    filesystems  ioports   meminfo  ...
10     1244   2268   crypto     fs           irq       modules  ...
104    125    228    devices    interrupts   kallsyms  mounts   ...

Process Directories

Each running process has a directory named after its PID (Process ID):

ls /proc/1
# cmdline  cwd  environ  exe  fd  maps  mem  status  ...

File	What It Shows
/proc/1/cmdline	The command that started PID 1 (systemd)
/proc/1/status	Process status (memory, state, parent)
/proc/1/environ	Environment variables of the process
/proc/1/exe	Symlink to the executable binary
/proc/1/fd/	File descriptors (open files)
System Information Files
File	What It Shows
/proc/cpuinfo	CPU model, cores, cache, flags
/proc/meminfo	Memory usage (RAM, swap, buffers)
/proc/version	Kernel version and compiler
/proc/filesystems	Supported filesystem types
/proc/mounts	Currently mounted filesystems
/proc/ioports	I/O port assignments
/proc/interrupts	IRQ assignments and counts
/proc/cmdline	Kernel boot parameters
Practical Examples

# Check your CPU:
cat /proc/cpuinfo | grep "model name" | head -1
# model name : Intel(R) Core(TM) i7-12700K CPU @ 3.60GHz

# Check memory:
cat /proc/meminfo | head -5
# MemTotal:       16384000 kB
# MemFree:         4200000 kB
# MemAvailable:    8200000 kB
# Buffers:          200000 kB
# Cached:          3800000 kB

# See kernel boot parameters:
cat /proc/cmdline
# BOOT_IMAGE=(hd0,gpt2)/vmlinuz-6.19.0-50.fc44.x86_64 root=/dev/mapper/luks-... ro rd.luks.uuid=... rhgb quiet

# Check what process is using a file:
lsof /path/to/file

    💡 Why /proc Matters

    Many system monitoring tools you use daily — top, htop, free, ps — read their data from /proc. When htop shows CPU usage, it's reading /proc/stat. When free shows memory, it's parsing /proc/meminfo. Understanding /proc means understanding where your tools get their data.

/sys — Hardware and Driver Information

Purpose: Virtual filesystem exposing kernel device/driver attributes.

Like /proc, /sys (sysfs) is a virtual filesystem generated by the kernel. It provides a structured view of hardware devices, drivers, and kernel subsystems.

ls /sys
# block  bus  class  dev  devices  firmware  fs  kernel  module  power

Exploring Hardware

# List all devices by class:
ls /sys/class/
# backlight  dma  graphics  input  net  powercap  sound  thermal  ...

# Check battery info (laptops):
cat /sys/class/power_supply/BAT0/capacity
# 87

cat /sys/class/power_supply/BAT0/status
# Charging

# Check network interfaces:
ls /sys/class/net/
# enp3s0  lo  wlp2s0

# Check CPU temperature:
cat /sys/class/thermal/thermal_zone*/temp
# 45000    (= 45.0°C, temperatures are in millidegrees Celsius)

Writing to /sys

Some /sys files are writable — writing to them changes kernel behavior:

# Enable a kernel module feature:
echo 1 | sudo tee /sys/module/snd_hda_intel/parameters/power_save

# Set display brightness (laptops):
echo 500 | sudo tee /sys/class/backlight/intel_backlight/brightness

    ⚠️ Be Careful Writing to /sys

    Writing incorrect values to /sys can crash the system or cause hardware malfunction. Read documentation before changing kernel parameters through sysfs. Changes are temporary — they reset on reboot unless made persistent through configuration files or udev rules.

/run — Runtime Data

Purpose: Volatile runtime data for processes and services.

/run is a tmpfs — a filesystem stored in RAM, not on disk. Its contents are recreated on every boot. This is where systemd, services, and applications store runtime state that shouldn't persist across reboots:

/run/
├── media/              ← Auto-mounted USB drives
├── lock/               ← Lock files (prevent duplicate processes)
├── user/               ← Per-user runtime data
│   └── 1000/           ← UID 1000's runtime directory
│       ├── pipewire/   ← PipeWire socket
│       ├── gnupg/      ← GPG agent socket
│       └── keyring/    ← GNOME keyring socket
├── systemd/            ← systemd runtime data
└── udev/               ← udev runtime data

# Verify it's a tmpfs:
findmnt /run
# TARGET  SOURCE  FSTYPE  OPTIONS
# /run    tmpfs   tmpfs   rw,nosuid,nodev,size=4096M,mode=755

    💡 Old Directories Replaced by /run

    Before /run existed, runtime data was scattered across /var/run/ and /var/lock/. On Fedora 44, these are symlinks to /run:

    ls -ld /var/run /var/lock
    # /var/run  -> ../run
    # /var/lock -> ../run/lock

/srv — Service Data

Purpose: Data served by system services (web servers, FTP, etc.).

Currently empty on most Fedora Workstation installations:

ls /srv
# (empty)

If you install a web server (Apache/Nginx), you might place website files here. In practice, many administrators use /var/www/ instead. /srv is the FHS-recommended location, but adoption is inconsistent.
/tmp — Temporary Files

Purpose: Temporary storage for applications and users.

/tmp/
├── systemd-private-...    ← Private systemd service temp dirs
├── snapshot-backup.tar.gz ← User-created temporary file
└── ...

Key characteristics:

    World-writable — any user can create files here
    Cleared on reboot — on Fedora 44, /tmp is a tmpfs (stored in RAM) or is cleaned by systemd-tmpfiles
    No guarantees — applications should not rely on files in /tmp surviving across reboots

# Verify tmp status:
findmnt /tmp
# If "tmpfs": stored in RAM, cleared on reboot
# If absent: on disk, cleaned periodically by systemd-tmpfiles

    💡 Private /tmp for Services

    systemd creates private /tmp directories for services that request them (via PrivateTmp=yes in their service files). This means a service like Apache sees its own /tmp that's isolated from the system /tmp. This is a security feature — a compromised service can't read other services' temporary files.

/usr — The Majority of the System

Purpose: User programs, libraries, documentation, and resources.

This is the largest directory on your system. Almost everything installed by DNF5 ends up here.

/usr/
├── bin/          ← All user command binaries (4,500+ files)
├── sbin/         ← System administration binaries
├── lib/          ← Libraries (32-bit)
├── lib64/        ← Libraries (64-bit)
├── share/        ← Architecture-independent data
│   ├── applications/   ← .desktop files (app shortcuts)
│   ├── icons/          ← Icon themes
│   ├── fonts/          ← System fonts
│   ├── doc/            ← Package documentation
│   ├── man/            ← Man pages
│   └── mime/           ← MIME type associations
├── local/        ← Locally compiled/installed software
│   ├── bin/
│   ├── lib/
│   └── share/
├── include/      ← C/C++ header files (for development)
└── src/          ← Source code (for some packages)

/usr/share — The Data Mine

This directory holds architecture-independent data — files that are the same regardless of whether you're on x86_64 or ARM:

# Application launchers (.desktop files):
ls /usr/share/applications/
# firefox.desktop  org.gnome.Nautilus.desktop  spotify.desktop ...

# Where GNOME finds icons:
ls /usr/share/icons/
# Adwaita  hicolor  HighContrast  breeze_cursors

# System fonts:
ls /usr/share/fonts/
# google-noto-sans-cjk-fonts  liberation  dejavu

# Man pages:
ls /usr/share/man/man1/ | head -5
# ls.1.gz  cp.1.gz  mv.1.gz ...

/usr/local — Your Own Software

Software you compile from source or install outside the package manager goes here:

# If you build software from source:
./configure --prefix=/usr/local
make
sudo make install
# Binary lands at /usr/local/bin/myprogram

/usr/local/bin is in your $PATH, so programs installed there are accessible from the terminal. Fedora's package manager never touches /usr/local — it's reserved for you.

    💡 Desktop Files: How GNOME Knows About Apps

    When you install an application, a .desktop file is placed in /usr/share/applications/. This file tells GNOME (and other desktops) the app's name, icon, executable path, and categories. The app grid reads these files to populate its icons.

    For user-specific application shortcuts (like Flatpak overrides), files go in ~/.local/share/applications/.

    You can examine a desktop file:

    cat /usr/share/applications/firefox.desktop

/var — Variable Data

Purpose: Data that changes during system operation — logs, caches, spool files, databases.

/var/
├── log/          ← System and application logs
├── cache/        ← Application caches (DNF cache, man cache)
├── lib/          ← Application state data
│   ├── flatpak/  ← System-wide Flatpak installations
│   └── ...
├── spool/        ← Print queues, mail queues, cron spool
├── tmp/          ← Persistent temporary files (survives reboot, unlike /tmp)
├── www/          ← Web server documents (if installed)
└── ...

/var/log — System Logs

This is where you go when troubleshooting:

ls /var/log/

audit/        btmp         dnf.libcnf.log  journal/  private/  secure
boot.log      cups/        gdm/            lastlog   README     wtmp

Log File/Directory	Contents
/var/log/journal/	systemd journal (binary format, read with journalctl)
/var/log/audit/	SELinux audit logs (audit.log)
/var/log/dnf.libcnf.log	DNF5 operations log
/var/log/boot.log	Boot-time messages (services starting)
/var/log/cups/	Printer logs
/var/log/gdm/	GDM (login screen) logs
/var/log/wtmp	Login/logout history (binary, read with last)
/var/log/btmp	Failed login attempts (binary, read with lastb)

    💡 The Journal Replaced Text Logs

    On older Linux systems, all logs were plain text files in /var/log/. Modern Fedora uses systemd-journald, which stores logs in a binary format under /var/log/journal/. You read them with journalctl, not cat:

    journalctl -b          # All logs since boot
    journalctl -u dnf      # Logs for DNF service
    journalctl --since "1 hour ago"
    journalctl -p err      # Only error-level messages

    Some services still write traditional text logs to /var/log/, but the journal is the primary logging system. Covered in detail in Chapter 24.

/var/cache — Cached Data

du -sh /var/cache/dnf
# 2.3G    ← DNF5's downloaded package cache

Cached data is regenerable — deleting it frees space without losing functionality:

sudo dnf clean all    # Clears DNF cache

Btrfs Subvolumes: How Fedora Organizes the Filesystem

Fedora 44 uses Btrfs as its default filesystem. Btrfs has a feature called subvolumes — internal divisions within a single Btrfs filesystem that behave like separate filesystems but share the same storage pool.
What Fedora Creates by Default

On a default Fedora 44 installation with automatic partitioning:

Physical disk: /dev/nvme0n1p3 (LUKS-encrypted Btrfs partition)
│
├── Btrfs subvolume: root  →  mounted at /
│   └── (entire filesystem: /usr, /var, /etc, /boot, ...)
│
└── Btrfs subvolume: home  →  mounted at /home
    └── (user directories: /home/john, /home/mary, ...)

Both subvolumes share the same physical storage — there's no fixed size allocation. If / needs 80 GB and /home needs 200 GB, they simply use what they need from the shared pool. You never run out of space on one while the other has free space.
Why Subvolumes Instead of Partitions?
Feature	Traditional Partitions	Btrfs Subvolumes
Fixed size	Yes — must guess in advance	No — share a pool
Resizable	Difficult, requires unmounting	Dynamic, automatic
Snapshots	Not possible	Yes — each subvolume independently
Management	Requires partition table changes	Simple commands
Listing Subvolumes

sudo btrfs subvolume list /

ID 256 gen 89432 top level 5 path root
ID 257 gen 89432 top level 5 path home

How Subvolumes Appear in the Filesystem

Subvolumes are mounted via /etc/fstab:

cat /etc/fstab

# /etc/fstab
# Created by anaconda on Mon Apr 28 12:00:00 2026
#
UUID=a1b2c3d4-...  /                      btrfs  subvol=root,compress=zstd:1,...
UUID=a1b2c3d4-...  /home                  btrfs  subvol=home,compress=zstd:1,...
UUID=e5f6g7h8-...  /boot                  ext4   defaults
UUID=i9j0k1l2-...  /boot/efi              vfat   umask=0077,shortname=winnt

Notice that / and /home have the same UUID — they're on the same physical Btrfs partition, just different subvolumes. The subvol=root and subvol=home mount options tell the kernel which subvolume to mount at each mount point.

    💡 Compression: compress=zstd:1

    The compress=zstd:1 mount option in /etc/fstab enables transparent Zstandard compression on your Btrfs filesystem. Files are compressed when written and decompressed when read — automatically, with no user intervention. Level 1 is the fastest preset with reasonable compression ratios.

    Benefits:

        Less disk space used (typically 20-40% savings)
        Faster I/O on many workloads (less data to read/write)
        Zero user intervention — it's transparent

    You can check compression effectiveness:

    sudo btrfs filesystem usage /
    sudo compsize /home

Snapshots and Subvolumes

In Chapter 7, you set up Snapper for automatic Btrfs snapshots. Snapshots are taken at the subvolume level — a snapshot of the root subvolume captures the entire / filesystem at a point in time.

Because /home is a separate subvolume, snapshots of / don't include /home. This is intentional:

    System snapshots (before DNF transactions) protect your system packages — rolling back doesn't wipe your personal files
    Home directory snapshots (if configured separately) protect your data independently

Special Filesystems: tmpfs, procfs, sysfs, devtmpfs

Beyond your physical disk filesystem (Btrfs), Fedora mounts several virtual filesystems at boot:

findmnt -t tmpfs,proc,sysfs,devtmpfs,devpts,cgroup2

Filesystem	Mount Point	Type	Stored Where
tmpfs	/run	RAM	In memory — lost on reboot
tmpfs	/tmp	RAM	In memory — lost on reboot
proc	/proc	Virtual	Generated by kernel on the fly
sysfs	/sys	Virtual	Generated by kernel on the fly
devtmpfs	/dev	Virtual	Managed by kernel/udev
cgroup2	/sys/fs/cgroup	Virtual	Kernel control groups

These don't occupy disk space. They're either in RAM (tmpfs) or generated dynamically by the kernel (proc, sysfs, devtmpfs).
File Types in Linux

Linux has more file types than just "regular file" and "directory":

ls -la / | head -10

The first character of each line tells you the file type:
Symbol	Type	Example
-	Regular file	notes.txt, video.mp4, /usr/bin/ls
d	Directory	Documents/, /etc/
l	Symbolic link (symlink)	/bin → usr/bin
c	Character device	/dev/null, /dev/urandom
b	Block device	/dev/nvme0n1, /dev/sda
s	Socket	/run/user/1000/pipewire-0
p	Named pipe (FIFO)	Rare — created with mkfifo
Symbolic Links

A symlink is a pointer to another file or directory:

# Create a symlink:
ln -s /home/john/Documents/report.pdf ~/Desktop/report_link.pdf

# View it:
ls -l ~/Desktop/report_link.pdf
# lrwxrwxrwx. 1 john john 35 Jul 5 10:00 report_link.pdf -> /home/john/Documents/report.pdf

# The symlink is just a path stored as a file.
# When you access the symlink, the kernel follows the path.

    💡 Symlinks vs. Windows Shortcuts

    A Windows .lnk shortcut is a file that the shell interprets — the OS doesn't know about shortcuts at a low level. A Linux symlink is a kernel-level filesystem feature — every application sees through symlinks transparently. Opening a symlink with any program is identical to opening the original file.

Navigating the Hierarchy: A Practical Tour

Let's put it all together with a guided tour of your filesystem:

# Start at the root:
cd /

# See the top-level directories:
ls -F
# afs/  bin@  boot/  dev/  etc/  home/  lib@  lib64@  media/  mnt/
# opt/  proc/  root/  run/  sbin@  srv/  sys/  tmp/  usr/  var/

# Explore /etc (configuration):
ls /etc/dnf/
# dnf.conf  modules.d  repos.d/

cat /etc/dnf/dnf.conf
# [main]
# max_parallel_downloads=10
# fastestmirror=True

# Explore /var (variable data):
ls /var/log/
du -sh /var/cache/dnf/
# 2.3G

# Explore /usr (programs):
ls /usr/bin/ | wc -l
# 4572

# Explore /proc (kernel info):
cat /proc/cpuinfo | grep "model name" | head -1
cat /proc/meminfo | head -3

# Explore /sys (hardware):
cat /sys/class/net/*/operstate
# up

# Check your Btrfs subvolumes:
sudo btrfs subvolume list /

# Check mounts:
findmnt --real

# Go home:
cd ~

Case Sensitivity

Linux is case-sensitive. These are three different files:

README.txt
Readme.txt
readme.txt

On Windows and macOS (by default), these would conflict. On Linux, they coexist without issue. This extends to commands — LS is not the same as ls:

ls          # Works — lists files
LS          # "command not found" — capital LS doesn't exist

    ⚠️ Case Sensitivity and Web Developers

    If you develop websites, case sensitivity can bite you. A file named Image.PNG referenced as <img src="image.png"> works on Windows (case-insensitive filesystem) but fails on Linux. This is a common source of bugs when deploying code developed on Windows to Linux servers.

Disk Space: Where Did It Go?

When your disk fills up, you need to find the culprit:

# Overall disk usage:
df -h

Filesystem                Size  Used  Avail  Use%  Mounted on
/dev/mapper/luks-...      468G  182G  265G   41%   /
/dev/mapper/luks-.../home  ← (same device, /home is a subvolume)
/dev/nvme0n1p2            960M  280M  630M   31%   /boot
/dev/nvme0n1p1            600M   40M  560M    7%   /boot/efi
tmpfs                     7.8G  2.1G  5.7G   27%   /tmp

# Find largest directories:
sudo du -sh /* 2>/dev/null | sort -rh | head -10

120G  /home
45G   /var
12G   /usr
3.2G  /opt
800M  /boot

# Drill down into /var:
sudo du -sh /var/* 2>/dev/null | sort -rh | head -10

28G   /var/lib
12G   /var/cache
3.5G  /var/log

# Check DNF cache specifically:
sudo du -sh /var/cache/dnf
# 2.3G

# Clean it:
sudo dnf clean all

# Find largest individual files on the system:
sudo find / -type f -size +500M 2>/dev/null | head -20

What's Next

You now understand the complete Linux filesystem hierarchy — from the single root directory through every top-level directory, the /usr merge, virtual filesystems (/proc, /sys, /dev), Btrfs subvolumes, mount points, file types, and disk space analysis. You know where configuration lives (/etc), where programs live (/usr/bin), where logs live (/var/log), and where your personal data lives (/home).

In Chapter 15, we'll deepen your file management skills — advanced operations like copying entire directory trees, creating and extracting archives, comparing files, searching file contents, and understanding inodes, hard links, and the internal mechanics of how Linux tracks files.
Try It Yourself

    Explore the root. Run ls -F / and identify each top-level directory from memory. Then check yourself against the descriptions in this chapter. Which ones are symlinks? (Hint: ls -ld /bin /sbin /lib /lib64.)

    Read your system's identity. Run cat /etc/os-release, cat /etc/hostname, and cat /proc/cmdline. These three files tell you what system you're on, what it's named, and how it was booted.

    Examine /proc. Run cat /proc/cpuinfo | head -20 and cat /proc/meminfo | head -10. Then run ls /proc/$$ (where $$ is your shell's PID) to see what the kernel knows about your current terminal process.

    Explore /sys hardware info. Check your battery (cat /sys/class/power_supply/BAT0/capacity), network (ls /sys/class/net/), and CPU temperature (cat /sys/class/thermal/thermal_zone*/temp). Divide temperature by 1000 to get degrees Celsius.

    List your Btrfs subvolumes. Run sudo btrfs subvolume list /. Then check /etc/fstab to see how they're mounted. Verify that / and /home share the same UUID (same physical partition, different subvolumes).

    Find what's consuming disk space. Run df -h for an overview, then sudo du -sh /* 2>/dev/null | sort -rh | head -10 to find the biggest directories. Drill down into the largest one. Clean the DNF cache with sudo dnf clean all and check the difference.

    Trace a symlink chain. Run ls -ld /bin, then readlink -f /bin/ls to follow the symlink chain from /bin/ls to its real location in /usr/bin/ls. Understanding symlink resolution helps you trace where files actually live.

    View application launchers. Run cat /usr/share/applications/firefox.desktop. This is how GNOME knows about Firefox — its name, icon, executable path, and category. Try creating a custom .desktop file in ~/.local/share/applications/ for a script you use frequently.

    ✅ Chapter 14 Complete You understand the single-root filesystem tree (no drive letters, mount points attach devices at directories), the /usr merge (/bin → /usr/bin symlinks), every top-level directory (/boot, /dev, /etc, /home, /lib, /media, /mnt, /opt, /proc, /root, /run, /srv, /sys, /tmp, /usr, /var) with their purposes and practical examples, the "everything is a file" philosophy (hardware as /dev files, processes as /proc directories, kernel state as /sys attributes), virtual filesystems (tmpfs in RAM, /proc and /sys generated by kernel), the XDG base directory specification (~/.config, ~/.local/share, ~/.cache), Btrfs subvolumes (root and home sharing a pool, mounted via fstab, transparent Zstd compression), file types (regular, directory, symlink, character device, block device, socket, pipe), case sensitivity, disk space analysis tools (df, du, find by size), and how to navigate the entire hierarchy with confidence.

