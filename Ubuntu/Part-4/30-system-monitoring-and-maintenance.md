# Chapter 30: System Monitoring and Maintenance

## Keeping Your System Healthy

You've installed Ubuntu, configured it, secured it, and learned to troubleshoot it. Now comes the ongoing practice: monitoring your system's health and performing routine maintenance to keep it fast, stable, and reliable over the long term.

Linux systems don't degrade the way some operating systems do — there's no registry rot, no mysterious slowdown from accumulated junk. But disks fill up, logs grow, old kernels accumulate, and hardware ages. This chapter covers the tools and habits that keep your Ubuntu 26.04 system running at its best for years.

## System Resource Monitoring

### `free` — Memory at a Glance

free -h

               total        used        free      shared  buff/cache   available
Mem:            15Gi       4.2Gi       3.1Gi       340Mi       8.5Gi        10Gi
Swap:           2.0Gi          0B       2.0Gi

Column	Meaning
total	Total installed RAM
used	Memory currently in use by applications
free	Completely unused memory
shared	Memory used by tmpfs and shared libraries
buff/cache	Memory used by the kernel for disk cache and buffers
available	Realistic estimate of memory available for new applications

    💡 "Free" Memory Is Wasted Memory

    Don't panic when free is low. Linux deliberately uses unused RAM for disk caching (buff/cache) — it speeds up file access. When an application needs memory, the kernel instantly frees cached memory. The number that matters is available, not free.

    A system with 16 GB RAM showing 3 GB free but 10 GB available is healthy. The kernel is using spare memory to cache files, which is exactly what you want.

vmstat — Virtual Memory Statistics

vmstat 5

Updates every 5 seconds:

procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
 r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
 1  0      0 3245672 456784 8504320    0    0    12    28  450 1200 15  4 81  0  0
 0  0      0 3210234 456784 8505560    0    0     8    16  420 1180  8  3 89  0  0

Column	What to Watch
r (run queue)	Processes waiting for CPU — consistently high (>4) means CPU-bound
si/so (swap in/out)	Non-zero values mean the system is actively swapping — slow performance
bi/bo (block in/out)	Disk I/O — high values indicate heavy disk activity
us (user CPU)	Percentage of CPU used by applications
sy (system CPU)	Percentage used by the kernel
id (idle)	High is good — CPU has headroom
wa (I/O wait)	Non-zero means the CPU is waiting for disk — slow disk or failing drive
iostat — Disk I/O Statistics

iostat -x 5

If not installed: sudo apt install sysstat

Device     r/s     w/s     rkB/s    wkB/s   rrqm/s   wrqm/s  %util
nvme0n1   12.50    3.20    240.0     64.0     0.00     0.00   1.23
sda        0.00    0.00      0.0      0.0     0.00     0.00   0.00

The key column is %util — the percentage of time the device was busy. If this is consistently near 100%, your disk is a bottleneck. On NVMe drives, brief spikes are normal; sustained max values indicate a problem.
uptime — How Long and How Busy

uptime

 14:32:01 up 7 days,  3:14,  2 users,  load average: 0.42, 0.38, 0.40

The load average shows three numbers: 1-minute, 5-minute, and 15-minute averages. These represent the average number of processes waiting for CPU time.

    A load average equal to your number of CPU cores means the system is at full capacity
    Below that means there's headroom
    Above that means processes are queuing (waiting for CPU)

Check your core count:

nproc

If nproc returns 8 and your load average is 4.5, your system is at ~56% CPU capacity. If the load is 12 on an 8-core system, processes are waiting.
btop — Real-Time Visual Monitor

sudo apt install btop
btop

btop provides a full-screen, colorful, interactive system monitor with real-time charts for CPU, memory, disk I/O, and network. It's the modern replacement for top and htop.

Key bindings:
Key	Action
q	Quit
Tab	Switch between panels (CPU, MEM, NET, PROC)
Left/Right	Select process
k	Kill selected process
n	Sort by name
c	Sort by CPU usage
m	Sort by memory usage
+/-	Change process priority (nice value)
h	Help
glances — All-in-One Monitor

sudo apt install glances
glances

Glances shows everything on one screen: CPU, memory, load, network, disk I/O, processes, and more. It can also run as a web server for remote monitoring:

glances -w

Then visit http://localhost:61208 in a browser to view system stats remotely.
GNOME Resources (Built-In)

Ubuntu 26.04 includes the Resources app (replacing the older System Monitor). Open it by pressing Super and typing "Resources." It provides graphical views of CPU, memory, disk, and network usage — ideal for quick checks without opening a terminal.
Disk Usage Analysis
df — Filesystem Disk Space

df -h

Filesystem      Size  Used Avail Use% Mounted on
tmpfs           7.8G  2.3M  7.8G   1% /run
/dev/nvme0n1p2  468G  142G  303G  32% /
tmpfs           7.8G   58M  7.8G   1% /dev/shm
tmpfs           4.0G  4.0K  4.0G   1% /tmp
/dev/nvme0n1p1  511M  6.1M  505M   2% /boot/efi

The -h flag makes sizes human-readable (GB instead of blocks).

Watch the Use% column. If your root partition (/) exceeds 90%, the system can have problems — package installations fail, logs can't write, and some applications crash. Below 80% is comfortable.
du — Directory Disk Usage

Find what's consuming space:

du -sh /home/john/*

4.0K    /home/john/Desktop
2.3G    /home/john/Documents
15G     /home/john/Downloads
4.1G    /home/john/Pictures
8.7G    /home/john/Projects
3.2G    /home/john/.cache
1.5G    /home/john/.local

The -s flag shows summaries (not every file), -h makes it human-readable.

Find the top 20 largest directories on the system:

sudo du -h / 2>/dev/null | sort -rh | head -20

ncdu — Interactive Disk Usage Analyzer

sudo apt install ncdu
ncdu /

ncdu provides an interactive, navigable view of disk usage. Use arrow keys to browse, d to delete files (with confirmation), and q to quit. It's dramatically faster than du for exploring where space has gone.

Scan just your home directory:

ncdu ~

baobab — Graphical Disk Usage Analyzer

For a visual pie-chart representation:

sudo apt install baobab
baobab

Also available as "Disk Usage Analyzer" in the app grid. It shows a color-coded treemap of disk usage, letting you click through directories to find space hogs visually.
Finding Large Files

Files larger than 500 MB:

find / -type f -size +500M 2>/dev/null | head -20

Modified in the last 7 days, larger than 100 MB:

find ~ -type f -mtime -7 -size +100M

Memory and Swap Management
How Ubuntu 26.04 Handles Swap

Ubuntu 26.04 uses ZRAM by default — a compressed block device in RAM that acts as swap space. Instead of writing swapped pages to disk (slow), ZRAM compresses them in memory (fast, but uses CPU).

Check ZRAM status:

sudo zramctl

NAME       ALGORITHM DISKSIZE DATA COMPR TOTAL STREAMS MOUNTPOINT
/dev/zram0 zstd           4G   0B   40B   40B       8 [SWAP]

    DISKSIZE — Maximum swap size (compressed)
    DATA — Uncompressed data currently stored
    COMPR — Compressed size (much smaller — this is the benefit)
    zstd — Compression algorithm (fast and efficient)

Check overall swap:

swapon --show

NAME       TYPE      SIZE USED PRIO
/dev/zram0 partition 4G   0B   -2

Swappiness: When to Swap

The vm.swappiness sysctl parameter controls how aggressively the kernel swaps memory pages to swap space (or ZRAM). Values range from 0 to 100:
Value	Behavior
0	Almost never swap (favors keeping apps in RAM)
10	Conservative swapping (good for desktops)
60	Default for many distributions
100	Aggressive swapping

Check current value:

cat /proc/sys/vm/swappiness

Temporary change:

sudo sysctl vm.swappiness=10

Permanent change:

echo "vm.swappiness=10" | sudo tee /etc/sysctl.d/99-swappiness.conf
sudo sysctl --system

    💡 ZRAM and Swappiness

    With ZRAM as the default swap on Ubuntu 26.04, higher swappiness values are less harmful than with traditional disk swap — data is compressed in RAM, not written to a slow disk. Canonical recommends leaving the default swappiness value (usually 60) when using ZRAM. Only lower it if you notice excessive swapping on a system with plenty of RAM.

Checking for Memory Pressure

If your system feels sluggish and you suspect memory:

sudo dmesg | grep -i "oom\|out of memory\|killed process"

The OOM Killer (Out Of Memory Killer) is a kernel mechanism that kills processes when the system runs out of memory. If you see "Killed process" entries, a process consumed too much RAM and was terminated by the kernel. This indicates you need more RAM, fewer running applications, or a memory leak in an application.

journalctl -b | grep -i "oom"

Disk Health: SMART Monitoring

S.M.A.R.T. (Self-Monitoring, Analysis, and Reporting Technology) is built into most modern drives (HDDs, SSDs, NVMe). It tracks disk health metrics like bad sectors, reallocated sectors, temperature, and power-on hours.
Installing smartmontools

sudo apt install smartmontools

Checking SMART Status

sudo smartctl -a /dev/nvme0n1

Replace nvme0n1 with your drive identifier (check with lsblk).

Quick health check:

sudo smartctl -H /dev/nvme0n1

smartctl 7.4 2023-08-01 r5530 [x86_64-linux-7.0.0] (local build)
Copyright (C) 2002-23, Bruce Allen, Christian Franke, www.smartmontools.org

=== START OF READ SMART DATA SECTION ===
SMART overall-health self-assessment test result: PASSED

PASSED means the drive reports no imminent failure. FAILED means replace the drive immediately.
Key SMART Attributes (for SATA HDDs/SSDs)

sudo smartctl -A /dev/sda

Attribute	What It Means
Reallocated_Sector_Ct	Bad sectors remapped — non-zero is concerning
Current_Pending_Sector	Sectors awaiting reallocation — non-zero is a warning sign
Offline_Uncorrectable	Sectors that can't be read — failing drive
Power_On_Hours	Total hours of operation
Temperature_Celsius	Drive temperature — should be below 50°C
SSD_Life_Left / Wear_Leveling_Count	SSD wear indicator (for NVMe/SSD)
Running SMART Self-Tests

# Short test (2-5 minutes)
sudo smartctl -t short /dev/nvme0n1

# Long test (can take hours — run overnight)
sudo smartctl -t long /dev/nvme0n1

# Check test progress
sudo smartctl -l selftest /dev/nvme0n1

    💡 Schedule Regular SMART Checks

    Add a monthly long test to cron:

    # Run on the 1st of each month at 2 AM
    0 2 1 * * sudo smartctl -t long /dev/nvme0n1

    Check results after a few days with sudo smartctl -l selftest /dev/nvme0n1.

Log Management

System logs grow over time. Without management, they can consume significant disk space.
Journal Size

Check how much space the systemd journal is using:

journalctl --disk-usage

Archived and active journals take up 1.2G in the file system.

Vacuuming the Journal

Reduce to a specific size:

sudo journalctl --vacuum-size=500M

Removes old journal entries until the total size is under 500 MB.

Remove logs older than 30 days:

sudo journalctl --vacuum-time=30d

Remove all but the last 3 boots:

sudo journalctl --vacuum-files=3

Combined (aggressive cleanup):

sudo journalctl --vacuum-size=500M --vacuum-time=14d

Permanent Journal Size Limits

Instead of manual vacuuming, set a permanent limit so the journal self-manages:

sudo nano /etc/systemd/journald.conf.d/size-limit.conf

Content:

[Journal]
SystemMaxUse=500M
SystemMaxFileSize=50M
MaxRetentionSec=30day

Apply:

sudo systemctl restart systemd-journald

The journal will now automatically prune itself to stay under 500 MB and discard entries older than 30 days.
Other Log Files

Besides the journal, some applications maintain their own logs:
Location	Content
/var/log/apt/history.log	APT package operations
/var/log/apt/term.log	APT terminal output
/var/log/auth.log	Authentication events
/var/log/dmesg	Kernel ring buffer (last boot)
/var/log/syslog	Traditional syslog messages
/var/log/kern.log	Kernel messages
logrotate: Automatic Log Rotation

Ubuntu uses logrotate to manage traditional log files. It compresses old logs, rotates them (renames with a date suffix), and deletes very old ones.

Check logrotate configuration:

cat /etc/logrotate.conf
ls /etc/logrotate.d/

Each file in /etc/logrotate.d/ controls rotation for a specific application's logs. Default settings typically keep 4 weeks of rotated logs, compressing old ones with gzip.

Test logrotate (dry run):

sudo logrotate -d /etc/logrotate.conf

Force logrotate:

sudo logrotate -f /etc/logrotate.conf

You generally don't need to modify logrotate configuration as a desktop user — the defaults are sensible. But knowing it exists helps when investigating log-related disk usage.
Boot Performance Analysis
Checking Boot Time

systemd-analyze

Startup finished in 6.234s (firmware) + 2.103s (loader) + 1.215s (kernel) + 4.892s (userspace) = 14.444s

This shows where time is spent:

    firmware — BIOS/UEFI initialization (outside Linux's control)
    loader — GRUB loading the kernel
    kernel — Kernel initialization
    userspace — systemd starting services

Identifying Slow Services

systemd-analyze blame

4.892s snapd.seeded.service
3.102s networkd-dispatcher.service
2.543s fwupd.service
1.876s udisks2.service
1.234s dev-nvme0n1p2.device
0.987s systemd-udevd.service
0.876s NetworkManager-wait-online.service
0.654s gdm.service
...

Services at the top are the slowest to start. Some are unavoidable (disk detection, firmware updates), but some can be optimized.
Critical Path Analysis

systemd-analyze critical-chain

Shows the chain of services that must complete sequentially before the desktop appears:

graphical.target @4.892s
└─gdm.service @4.238s
  └─NetworkManager-wait-online.service @3.362s +876ms
    └─NetworkManager.service @3.110s +252ms
      └─network-pre.target @3.108s

This reveals dependencies — if NetworkManager-wait-online.service is slow, it delays everything that depends on it.
Common Boot Optimization Targets
Service	Can Disable?	Notes
NetworkManager-wait-online.service	Usually yes	Delays boot until network is up — rarely needed for desktops
fwupd.service	No	Firmware update daemon — important for security
snapd.seeded.service	No	Ensures snaps are ready — needed for snap functionality
apport.service	Maybe	Crash reporter — useful for debugging, optional otherwise
avahi-daemon.service	Maybe	mDNS/Bonjour — disable if you don't need local network discovery
bluetooth.service	Maybe	Disable if you don't use Bluetooth
ModemManager.service	Maybe	Disable if you don't have a mobile broadband modem

Disable a service (prevents starting at boot):

sudo systemctl disable NetworkManager-wait-online.service

Mask a service (stronger than disable — prevents activation by any means):

sudo systemctl mask ModemManager.service

    ⚠️ Don't Disable What You Don't Understand

    Boot optimization is a rabbit hole. Disabling the wrong service can break networking, sound, or the display manager. Research each service before disabling it. When in doubt, leave it alone — a few seconds of boot time isn't worth a broken system.

Routine Maintenance Tasks
1. Keep Packages Updated

sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
sudo snap refresh
flatpak update -y

Or the combined one-liner from Chapter 18:

sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo snap refresh && flatpak update -y

Security updates happen automatically. Run this manually every week or two for non-security updates and to clean up unused packages.
2. Clean APT Cache

APT downloads .deb packages to /var/cache/apt/archives/ and keeps them even after installation. Over time, this cache grows.

Check cache size:

du -sh /var/cache/apt/archives/

Clean the cache:

sudo apt clean

Removes all downloaded package files. Packages remain installed — only the cached installers are deleted.

Clean only obsolete packages (older versions still in cache):

sudo apt autoclean

3. Remove Unused Packages

When you uninstall a package, its dependencies often remain. Remove them:

sudo apt autoremove --purge

The --purge flag also removes configuration files for the removed dependencies.
4. Remove Old Kernels

Each kernel update installs a new kernel image but keeps the old ones (for rollback). Over time, multiple old kernels consume disk space and clutter the GRUB menu.

List installed kernels:

dpkg --list 'linux-image-*' | grep ^ii

Current kernel:

uname -r

Remove old kernels (automated):

sudo apt autoremove --purge

On Ubuntu 26.04, apt autoremove is configured to remove old kernels automatically — it keeps the current one and one previous version. If you've manually installed kernels outside the package manager, remove them explicitly:

sudo apt remove --purge linux-image-7.0.0-18-generic

(Never remove the kernel reported by uname -r — that's your running kernel.)
5. Clean Thumbnail and Cache Directories

Over time, your home directory accumulates caches:

# Thumbnail cache (preview images for Files)
rm -rf ~/.cache/thumbnails/*

# General application cache
rm -rf ~/.cache/*

# (Don't worry — applications rebuild caches as needed)

6. Clean Snap Cache

Snaps store revision data. Old revisions are retained for rollback:

sudo snap set system refresh.retain=2

Sets snap to keep only the last 2 revisions (current + 1 previous). The default is 3 on most systems.

List snap disk usage:

du -sh /snap/*

7. Vacuum the Journal

sudo journalctl --vacuum-size=500M --vacuum-time=30d

Run monthly to keep log size in check. Or set a permanent limit as described above.
8. Check Disk Health

sudo smartctl -H /dev/nvme0n1

Run quarterly. If the result is anything other than PASSED, back up your data immediately and replace the drive.
Automated Maintenance Script

Here's a script that performs common maintenance tasks in one run:

#!/bin/bash
set -euo pipefail

# system-maintenance.sh — Routine Ubuntu maintenance
# Run weekly or monthly

echo "╔══════════════════════════════════════════════╗"
echo "║        Ubuntu System Maintenance             ║"
echo "╚══════════════════════════════════════════════╝"
echo ""
echo "Started: $(date)"
echo ""

# Disk space before
echo "─── Disk Space (Before) ───"
df -h / | awk 'NR==1{print} NR==2{print "  Used: "$3" of "$2" ("$5")"}'
echo ""

# Update packages
echo "─── Updating Packages ───"
sudo apt update -qq
UPGRADABLE=$(apt list --upgradable 2>/dev/null | grep -c upgradable)
echo "  $UPGRADABLE packages to upgrade"
sudo apt upgrade -y -qq
echo ""

# Remove unused packages
echo "─── Removing Unused Packages ───"
sudo apt autoremove --purge -y -qq
echo ""

# Clean APT cache
echo "─── Cleaning APT Cache ───"
CACHE_SIZE=$(du -sh /var/cache/apt/archives/ 2>/dev/null | cut -f1)
echo "  Cache size before: $CACHE_SIZE"
sudo apt clean
echo "  Cache cleared"
echo ""

# Refresh snaps
echo "─── Refreshing Snaps ───"
sudo snap refresh 2>/dev/null || echo "  (no snaps to refresh)"
echo ""

# Vacuum journal
echo "─── Vacuuming Journal ───"
JOURNAL_BEFORE=$(journalctl --disk-usage 2>/dev/null | grep -o '[0-9.]*[GKM]')
echo "  Journal size before: $JOURNAL_BEFORE"
sudo journalctl --vacuum-size=500M --vacuum-time=30d > /dev/null 2>&1
JOURNAL_AFTER=$(journalctl --disk-usage 2>/dev/null | grep -o '[0-9.]*[GKM]')
echo "  Journal size after: $JOURNAL_AFTER"
echo ""

# Clean thumbnail cache
echo "─── Cleaning User Cache ───"
CACHE_DIR=~/.cache
if [ -d "$CACHE_DIR" ]; then
    CACHE_USER=$(du -sh "$CACHE_DIR" 2>/dev/null | cut -f1)
    echo "  User cache size: $CACHE_USER"
    rm -rf "$CACHE_DIR"/thumbnails/* 2>/dev/null || true
    echo "  Thumbnails cleared"
fi
echo ""

# Disk health check
echo "─── Disk Health ───"
for disk in $(lsblk -d -n -o NAME | grep -E '^[snv]'); do
    if command -v smartctl > /dev/null 2>&1; then
        HEALTH=$(sudo smartctl -H "/dev/$disk" 2>/dev/null | grep -i "test result" | awk -F': ' '{print $2}')
        if [ -n "$HEALTH" ]; then
            echo "  /dev/$disk: $HEALTH"
        fi
    fi
done
echo ""

# Disk space after
echo "─── Disk Space (After) ───"
df -h / | awk 'NR==1{print} NR==2{print "  Used: "$3" of "$2" ("$5")"}'
echo ""

echo "═══════════════════════════════════════════════"
echo "Maintenance completed: $(date)"
echo "═══════════════════════════════════════════════"

Save as ~/bin/system-maintenance.sh, make executable, and schedule:

chmod +x ~/bin/system-maintenance.sh

Add to crontab (weekly, Sunday at 3 AM):

0 3 * * 0 /home/john/bin/system-maintenance.sh >> /home/john/.local/log/maintenance.log 2>&1

Run manually any time:

~/bin/system-maintenance.sh

Monitoring Checklist

Run through this checklist periodically (monthly is a good cadence):
Check	Command	Healthy Range
Disk space	df -h /	Below 80% used
Memory	free -h	available > 25% of total
Load average	uptime	Below number of CPU cores
Disk health	sudo smartctl -H /dev/sdX	PASSED
Disk I/O	iostat -x 1 3	%util below 80% sustained
Temperature	sensors	CPU below 80°C, disk below 50°C
Journal size	journalctl --disk-usage	Below 1 GB
Failed services	systemctl --failed	None listed
Pending updates	apt list --upgradable	Install regularly
Swap usage	swapon --show	USED should be low normally
Temperature Monitoring

Install hardware sensors:

sudo apt install lm-sensors
sudo sensors-detect --auto

Display readings:

sensors

coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +45.0°C  (high = +100.0°C)
Core 0:        +42.0°C  (high = +100.0°C)
Core 1:        +44.0°C  (high = +100.0°C)
...

nvme-pci-0100
Adapter: PCI adapter
Composite:    +38.9°C  (low  =  -0.1°C, high = +84.8°C)

Monitor continuously:

watch -n 2 sensors

Updates every 2 seconds. Press Ctrl+C to stop.
System Monitoring and Maintenance Cheat Sheet
Task	Command
Memory	
Memory summary	free -h
Virtual memory stats	vmstat 5
ZRAM status	sudo zramctl
Swap status	swapon --show
Check swappiness	cat /proc/sys/vm/swappiness
Set swappiness	sudo sysctl vm.swappiness=10
Check OOM events	sudo dmesg | grep -i oom
Disk Usage	
Filesystem space	df -h
Directory sizes	du -sh /path/*
Interactive disk usage	ncdu /
Largest directories	sudo du -h / | sort -rh | head -20
Large files	find / -type f -size +500M
Disk Health	
SMART health	sudo smartctl -H /dev/sdX
SMART details	sudo smartctl -a /dev/sdX
Short self-test	sudo smartctl -t short /dev/sdX
Long self-test	sudo smartctl -t long /dev/sdX
Test results	sudo smartctl -l selftest /dev/sdX
I/O	
Disk I/O stats	iostat -x 5
Real-time monitor	btop
All-in-one monitor	glances
Logs	
Journal disk usage	journalctl --disk-usage
Vacuum to size	sudo journalctl --vacuum-size=500M
Vacuum by time	sudo journalctl --vacuum-time=30d
Logrotate test	sudo logrotate -d /etc/logrotate.conf
Boot Performance	
Boot time	systemd-analyze
Slow services	systemd-analyze blame
Critical path	systemd-analyze critical-chain
Disable service	sudo systemctl disable servicename
Maintenance	
Update packages	sudo apt update && sudo apt upgrade -y
Remove unused	sudo apt autoremove --purge -y
Clean APT cache	sudo apt clean
Remove old kernels	sudo apt autoremove --purge
Clean thumbnails	rm -rf ~/.cache/thumbnails/*
Snap retain limit	sudo snap set system refresh.retain=2
Full maintenance	~/bin/system-maintenance.sh
Monitoring	
CPU/load	uptime
CPU cores	nproc
Temperatures	sensors
Failed services	systemctl --failed
Real-time stats	btop or glances
GNOME graphical	Resources app
What's Next

You now have a comprehensive toolkit for keeping your Ubuntu system healthy: real-time monitoring with btop and glances, memory and swap analysis with free and vmstat, disk usage hunting with df, du, and ncdu, disk health monitoring with smartctl, log management with journalctl vacuum and logrotate, boot performance analysis with systemd-analyze, routine maintenance tasks, and an automated maintenance script. Combined with the security practices from Chapter 27 and the backup strategy from Chapter 26, your system is protected, monitored, and maintained for the long haul.

In Chapter 31, the final chapter, we'll cover getting help and resources — where to find documentation, how to ask effective questions, community support channels, and the best resources for continuing your Linux education.
Try It Yourself

    Check your system's vitals. Run free -h, df -h, uptime, and sensors. Note your memory available, disk usage percentage, load average, and temperatures. These are your baselines — compare future readings against them.

    Install btop. Run sudo apt install btop && btop. Spend five minutes watching the real-time graphs. Press Tab to switch between panels. Press q to quit.

    Hunt for disk space hogs. Install ncdu: sudo apt install ncdu. Run ncdu ~ in your home directory. Navigate with arrow keys. Find your three largest directories. If any are cache or temporary files, clean them up.

    Check your disk health. Run lsblk to identify your drives, then sudo smartctl -H /dev/sdX (or nvme0nX) for each. All should report PASSED. If any don't, this is urgent — back up and replace.

    Analyze your boot time. Run systemd-analyze and systemd-analyze blame. Identify your three slowest services. Don't disable anything yet — just observe what's happening.

    Vacuum your journal. Check the size with journalctl --disk-usage. If it's over 1 GB, vacuum it: sudo journalctl --vacuum-size=500M. Set a permanent limit by creating the config file from this chapter.

    Run the maintenance script. Copy the system-maintenance.sh script, save it to ~/bin/, make it executable, and run it. Review the before/after disk space difference.

    Schedule weekly maintenance. Add the maintenance script to your crontab: crontab -e, then add the line: 0 3 * * 0 /home/YOURUSERNAME/bin/system-maintenance.sh >> ~/.local/log/maintenance.log 2>&1.

    ✅ Chapter 30 Complete You can monitor system resources (CPU, memory, disk I/O, temperature) with btop, glances, free, vmstat, iostat, and sensors; analyze disk usage with df, du, ncdu, and baobab; manage swap and ZRAM; check disk health with smartctl and SMART self-tests; manage log sizes with journalctl vacuum and logrotate configuration; analyze and optimize boot performance with systemd-analyze; perform routine maintenance (package updates, cache cleaning, old kernel removal, journal vacuuming); run an automated maintenance script; and maintain a monitoring checklist for ongoing system health.

