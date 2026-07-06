# Chapter 24: System Monitoring and Maintenance — Keeping Your System Healthy

## A System Is a Living Thing

A Linux system isn't something you install once and forget. It's a living, breathing collection of processes, logs, packages, files, and hardware that evolves over time. Logs accumulate and fill disks. Packages need security updates. Services crash and need restarting. Filesystems fragment or fill up. Configurations drift.

The difference between a system that runs for months without issues and one that falls apart weekly isn't luck — it's maintenance. This chapter teaches you the tools and habits that keep Fedora 44 healthy over the long haul.

You'll learn:
- Disk usage monitoring and cleanup (`df`, `du`, `ncdu`, `btrfs`)
- Log management with `journalctl` (filtering, rotating, vacuuming)
- System updates with DNF5 (manual and automated)
- Btrfs snapshots and rollback with Snapper
- Service management with `systemctl` (status, restart, enable, edit)
- Scheduled tasks with systemd timers (the modern replacement for cron)
- Performance profiling (`iostat`, `vmstat`, `sar`, `powertop`)
- Routine maintenance checklist

---

## Table of Contents

| Topic | Section |
|---|---|
| Disk Usage Monitoring | 1 |
| Log Management with journalctl | 2 |
| System Updates with DNF5 | 3 |
| Btrfs Snapshots and Rollback | 4 |
| Service Management with systemctl | 5 |
| Scheduled Tasks: systemd Timers | 6 |
| Performance Profiling | 7 |
| Routine Maintenance Checklist | 8 |

---

## 1. Disk Usage Monitoring

### df: Filesystem Overview

`df` (disk free) shows space usage across all mounted filesystems:

```bash
# Human-readable overview:
df -h

Filesystem     Size  Used Avail Use% Mounted on
devtmpfs       4.0M     0  4.0M   0% /dev
tmpfs           63Gi  2.5Gi  61Gi   4% /dev/shm
tmpfs           32Gi  1.8Gi  30Gi   6% /run
efivarfs        128K   58K   66K  48% /sys/firmware/efi/efivars
/dev/nvme0n1p3  468G  123G  322G  28% /
/dev/nvme0n1p1  600M  148M  452M  25% /boot/efi
tmpfs          1.0M     0  1.0M   0% /run/credentials
tmpfs           12Gi   42M   12Gi   1% /tmp
/dev/nvme0n1p4  932G  456G  476G  49% /home

Columns:
Column	Meaning
Filesystem	Device or virtual filesystem
Size	Total capacity
Used	Space consumed
Avail	Space available for use
Use%	Percentage used (watch for >90%)
Mounted on	Where the filesystem is accessible

# Show inode usage (file count, not just bytes):
df -i
# Filesystem      Inodes IUsed IFree IUse% Mounted on
# /dev/nvme0n1p3  3.0M   456K  2.5M   16% /
# If IUse% hits 100%, you can't create new files even if space remains

# Show only specific filesystem type:
df -h -t btrfs    # Btrfs only
df -h -t ext4     # ext4 only
df -h -x tmpfs    # Exclude tmpfs

# Show the filesystem type:
df -Th
# Adds a "Type" column

    💡 Watch Use% Above 90%

    Btrfs behaves differently from ext4 when nearly full. On ext4, fragmentation increases and performance degrades above 90%. On Btrfs, the situation is worse — the filesystem needs free space for metadata operations, and at 95%+ it can become read-only or refuse operations entirely. Keep Btrfs below 80% for optimal performance; consider it critical above 90%.

du: Directory-Level Usage

du (disk usage) measures how much space a directory tree consumes:

# Size of a single directory (human-readable):
du -sh /var/log
# 1.2G  /var/log

# List sizes of subdirectories:
du -sh /var/log/*
# 240M  /var/log/audit
# 450M  /var/log/journal
# 12M   /var/log/dnf.log
# 4.0K  /var/log/README

# Sort by size, show top 10 largest directories:
du -sh /* 2>/dev/null | sort -rh | head -10
# 123G  /home
# 45G   /var
# 12G   /usr
# 3.2G  /opt

# Go deeper — find the largest subdirectories in /var:
du -sh /var/* 2>/dev/null | sort -rh | head -10

# Drill down recursively, sorting:
du -h --max-depth=1 /home/john/ 2>/dev/null | sort -rh
# 45G   /home/john/Downloads
# 12G   /home/john/.cache
# 8.3G  /home/john/Projects
# 2.1G  /home/john/.local
# 500M  /home/john/Documents

# Include hidden files:
du -sh /home/john/.* /home/john/* 2>/dev/null | sort -rh | head -10

# Estimate size of all .log files on the system:
sudo find /var/log -name "*.log" -type f -exec du -ch {} + 2>/dev/null | tail -1

The 2>/dev/null is important — du throws "Permission denied" errors on directories you don't own, flooding your terminal. Redirecting stderr silences them.
ncdu: Interactive Disk Explorer

ncdu (NCurses Disk Usage) provides an interactive, navigable view of disk usage — far more efficient than repeated du commands:

# Install:
sudo dnf install ncdu

# Scan a directory:
ncdu /
# Scans the entire filesystem (takes a moment)

# Scan your home directory:
ncdu ~

ncdu 1.19  Use arrow keys to navigate, press ? for help
--- /home/john -----------------------------------------------------
  45.0GiB [##########] /Downloads
  12.0GiB [##        ] /.cache
   8.3GiB [#         ] /Projects
   2.1GiB [          ] /.local
   500MiB [          ] /Documents
   120MiB [          ] /.config
   45MiB  [          ] /Pictures
   12MiB  [          ] /.thunderbird

ncdu key bindings:
Key	Action
↑ ↓	Navigate up/down
Enter	Enter directory
← or Backspace	Go to parent directory
d	Delete selected file/directory (prompts for confirmation)
n	Sort by name
s	Sort by size (default)
C	Sort by children count
g	Toggle graph/bar display
t	Toggle dirs before files
q	Quit

ncdu is the fastest way to answer "what's eating my disk space?" — drill down with arrow keys, find the culprit, and optionally delete it without leaving the tool.
Btrfs-Specific Disk Tools

Fedora 44 uses Btrfs by default. Btrfs has its own tools for understanding space usage:

# Btrfs filesystem usage overview:
sudo btrfs filesystem df /
# Data, single: total=123.42GiB, used=98.34GiB
# System, DUP: total=8.00MiB, used=16.00KiB
# Metadata, DUP: total=2.00GiB, used=512.00MiB
# GlobalReserve, single: total=512.00MiB, used=0.00B

# Show actual usable space (more accurate than df for Btrfs):
sudo btrfs filesystem show /
# Label: 'fedora_fedora'  uuid: a1b2c3d4-...
#         Total devices 1 FS bytes used 98.34GiB
#         devid    1 size 468.00GiB used 123.42GiB path /dev/nvme0n1p3

# Sync filesystem (force pending operations to complete):
sudo btrfs filesystem sync /

# Show all subvolumes:
sudo btrfs subvolume list /
# ID 256 gen 12345 top level 5 path root
# ID 257 gen 12345 top level 5 path home
# ID 258 gen 12340 top level 5 path .snapshots

    💡 Btrfs and df Discrepancies

    Btrfs often shows different usage in df versus btrfs filesystem df. This is because Btrfs uses CoW (Copy-on-Write) and stores metadata separately. Shared extents from snapshots mean the same physical blocks can be counted for multiple files. When in doubt:

    sudo btrfs filesystem usage /

    This gives the most accurate picture, including how much space is used by snapshots.

Cleaning Up Common Space Hogs

# DNF5 package cache (cached downloaded packages):
sudo dnf clean all
# Removes all cached packages and metadata

# Journal logs (covered in detail in Section 2):
sudo journalctl --vacuum-size=500M

# User cache (~/.cache):
du -sh ~/.cache
rm -rf ~/.cache/thumbnails/      # Thumbnail cache
rm -rf ~/.cache/mozilla/          # Browser cache (restart browser after)

# Trash:
du -sh ~/.local/share/Trash/
# Empty it from Files (nautilus) or:
rm -rf ~/.local/share/Trash/files/* ~/.local/share/Trash/info/*

# Old kernels (Fedora keeps the latest 3 by default):
rpm -q kernel
# kernel-6.19.0-50.fc44.x86_64
# kernel-6.19.0-48.fc44.x86_64
# kernel-6.19.0-45.fc44.x86_64
# DNF5 automatically removes old kernels on update, but manual check:
sudo dnf remove --oldinstallonly

# Flatpak cleanup:
flatpak uninstall --unused
# Removes unused runtimes that are no longer needed by any app

# Systemd journal:
sudo journalctl --disk-usage
sudo journalctl --vacuum-time=7d

Finding the Largest Files on the System

# Top 20 largest files (excluding virtual filesystems):
sudo find / -path /proc -prune -o -path /sys -prune -o -path /run -prune \
  -o -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -20

# Using ncdu (interactive, more practical):
sudo ncdu -x /
# -x stays on the same filesystem (doesn't cross mount points)

2. Log Management with journalctl
What the Journal Is

Fedora 44 uses systemd-journald as its central logging system. Unlike traditional Unix logging (scattered text files in /var/log/), journald collects logs from:

    The kernel (dmesg-style messages)
    All systemd services (stdout/stderr)
    syslog-compatible applications
    Audit messages

Logs are stored in a structured, indexed binary format — not plain text. You can't cat them directly; you use journalctl to query them.

# Check journald is running:
systemctl status systemd-journald
# Active: active (running)

Basic journalctl Queries

# All logs from the current boot:
journalctl -b
# Opens in less (press q to quit, / to search, Space to scroll page)

# Today's logs:
journalctl --since today

# Last hour:
journalctl --since "1 hour ago"

# Specific time range:
journalctl --since "2026-07-06 09:00" --until "2026-07-06 12:00"

# Logs since boot, errors only:
journalctl -b -p err

# Follow logs in real time (like tail -f):
journalctl -f

# Show the last 20 lines:
journalctl -n 20

# Follow with a filter:
journalctl -f -u sshd

Filtering by Priority

Log priorities follow standard syslog levels:
Priority	Name	Meaning
0	emerg	System is unusable
1	alert	Must be acted on immediately
2	crit	Critical conditions
3	err	Error conditions
4	warning	Warning conditions
5	notice	Normal but significant
6	info	Informational
7	debug	Debug-level messages

# Only errors and above:
journalctl -p err

# Warnings and above:
journalctl -p warning

# Critical only:
journalctl -p crit

# Use names or numbers:
journalctl -p 3    # Same as -p err
journalctl -p 3..5 # Range: err through notice

Filtering by Service, Unit, or Binary

# Logs from a specific service:
journalctl -u sshd
journalctl -u NetworkManager
journalctl -u firewalld

# Multiple services:
journalctl -u sshd -u firewalld

# Logs from a specific executable:
journalctl /usr/bin/firefox

# Logs from a specific PID:
journalctl _PID=4567

# Logs from a specific user:
journalctl _UID=1000

# Logs from a specific boot (list boots first):
journalctl --list-boots
# IDX LOG ID    BOOT ID                       FIRST ENTRY                  LAST ENTRY
# -3 b1a2c3...  Mon 2026-07-03 09:15:23 ...  Mon 2026-07-03 17:45:12 ...
# -2 c3d4e5...  Tue 2026-07-04 08:30:00 ...  Tue 2026-07-04 22:15:45 ...
# -1 d5e6f7...  Wed 2026-07-05 07:45:00 ...  Wed 2026-07-05 23:59:59 ...
#  0 e7f8a9...  Thu 2026-07-06 09:00:00 ...  Thu 2026-07-06 14:23:45 ...

# View logs from boot index -1 (previous boot):
journalctl -b -1

# View logs from boot index -2 (two boots ago):
journalctl -b -2

Output Formats

# Default (opens in pager):
journalctl -b

# Plain output (no pager):
journalctl -b --no-pager

# JSON (for scripting/API consumption):
journalctl -b -o json | python3 -m json.tool | head -20

# Short ISO timestamps:
journalctl -b -o short-iso

# Export format (for backup/archival):
journalctl -b -o export > /tmp/boot_log.export

# Just the messages (no metadata):
journalctl -b -o cat
# Cleaner for reading messages without timestamps and unit names

Practical journalctl Recipes

# What happened with SSH today?
journalctl -u sshd --since today

# All errors from the previous boot (troubleshooting boot failures):
journalctl -b -1 -p err

# Real-time monitoring of all services:
journalctl -f

# Real-time monitoring of a specific service:
journalctl -f -u NetworkManager

# What did DNF5 do in the last hour?
journalctl -u dnf5 --since "1 hour ago"

# Kernel messages from current boot:
journalctl -k -b

# Show messages containing a specific string:
journalctl -g "error|fail|panic"
# -g is grep-like (uses regular expressions)

# Show messages from a specific D-Bus path:
journalctl _SYSTEMD_UNIT=gnome-shell.service

# Show the last boot's last 50 lines (what caused a shutdown/reboot):
journalctl -b -1 -n 50 --no-pager

Journal Disk Usage and Cleanup

# Check how much space the journal uses:
journalctl --disk-usage
# Archived and active journals take up 1.2G in total.

# Trim to 500MB (removes oldest archived journals):
sudo journalctl --vacuum-size=500M
# Deleted archived journal /var/log/journal/.../system@... (128.0M)
# Deleted archived journal /var/log/journal/.../user-1000@... (64.0M)
# ...

# Remove logs older than 7 days:
sudo journalctl --vacuum-time=7d

# Remove logs older than 1 second (aggressive — removes all archived):
sudo journalctl --vacuum-time=1s

# Rotate (close current journal file, start a new one):
sudo journalctl --rotate

# Rotate then vacuum (recommended sequence for thorough cleanup):
sudo journalctl --rotate
sudo journalctl --vacuum-size=500M --vacuum-time=7d

# Remove individual journal files manually (last resort):
sudo ls -lhS /var/log/journal/*/  # List by size, largest first
# Only remove files with "@" in the name (archived), never the active one

Configuring Journal Limits

Prevent the journal from growing unbounded by configuring limits:

# Edit the journald configuration:
sudo nano /etc/systemd/journald.conf

[Journal]
# Storage mode: persistent (save to disk), volatile (RAM only), auto (default)
Storage=persistent

# Maximum disk space the journal may use:
SystemMaxUse=500M

# Minimum free disk space to maintain:
SystemKeepFree=1G

# Maximum time to keep entries:
MaxRetentionSec=2week

# Maximum individual journal file size:
SystemMaxFileSize=50M

# Rate-limiting (prevent log flooding):
RateLimitIntervalSec=30s
RateLimitBurst=10000

Alternatively, use a drop-in snippet (preferred — doesn't modify the original config):

sudo mkdir -p /etc/systemd/journald.conf.d/
sudo nano /etc/systemd/journald.conf.d/size-limit.conf

[Journal]
SystemMaxUse=500M
MaxRetentionSec=2week

# Apply:
sudo systemctl restart systemd-journald

Traditional Log Files

Some applications still write to plain-text log files. Fedora 44 retains /var/log/ for compatibility:

ls /var/log/
# audit  btmp  dnf.librepo.log  dnf.log  dnf.rpm.log  journal  lastlog
# maillog  messages  secure  wtmp

# View traditional log files:
sudo tail -f /var/log/messages    # System messages
sudo tail -f /var/log/secure      # Authentication log (SSH, sudo)
sudo tail -f /var/log/dnf.log     # DNF5 package operations

# These are managed by rsyslog (which forwards to journald as well)
systemctl status rsyslog

logrotate: Managing Traditional Log Files

For text logs that grow indefinitely, logrotate automatically rotates, compresses, and deletes old log files:

# Check if logrotate is installed:
rpm -q logrotate

# Main config:
cat /etc/logrotate.conf

# Application-specific configs:
ls /etc/logrotate.d/
# btmp  cups  dnf  wpa_supplicant

# View a sample rule:
cat /etc/logrotate.d/dnf

/var/log/dnf.log {
    missingok
    notifempty
    rotate 4
    size 30M
    compress
}

This rotates dnf.log when it exceeds 30MB, keeps 4 archived copies, compresses old ones, and doesn't error if the file doesn't exist.

Test logrotate manually:

# Debug (show what would happen without doing it):
sudo logrotate -d /etc/logrotate.conf

# Force rotation now:
sudo logrotate -f /etc/logrotate.conf

# Verbose output:
sudo logrotate -v /etc/logrotate.conf

3. System Updates with DNF5
Why Updates Matter

Updates deliver:

    Security patches — vulnerability fixes (most urgent)
    Bug fixes — corrected functionality
    Feature enhancements — improvements to existing packages
    Kernel updates — newer hardware support and performance

Fedora 44's DNF5 handles all of this. Chapter 18 covered DNF5's package management commands. Here we focus on the maintenance workflow.
Manual Updates

# Check for available updates:
dnf check-update

# Update all packages:
sudo dnf upgrade

# Update security patches only:
sudo dnf upgrade --security

# Update and reboot if needed (kernel updates):
sudo dnf upgrade
# If kernel was updated:
sudo reboot

# Preview what would be updated (without installing):
sudo dnf upgrade --downloadonly

Reviewing Update History

# Recent DNF transactions:
dnf history
# ID  Command                          Date             Altered
# 45  upgrade                          2026-07-06 14:00   42
# 44  install akmod-nvidia             2026-07-05 09:15    5
# 43  upgrade                          2026-07-04 03:00   12
# ...

# Details of a specific transaction:
dnf history info 45

# Undo a transaction (rollback):
sudo dnf history undo 45
# Reverses everything transaction 45 did

# Redo a previously undone transaction:
sudo dnf history redo 45

# Show only recent installations:
dnf history list --reverse | head -20

    ⚠️ Undo Caveats

    dnf history undo reverses package installations and removals, but it CANNOT undo:

        Configuration file changes made by post-install scripts
        Database schema migrations
        Data files created or modified by the application

    Use undo cautiously for complex packages. For system-level rollbacks, use Btrfs snapshots (Section 4).

Cleaning Up After Updates

# Remove orphaned dependencies (packages installed as deps but no longer needed):
sudo dnf autoremove

# Clear the DNF cache:
sudo dnf clean all
# Or clean specific caches:
sudo dnf clean packages    # Remove cached .rpm files
sudo dnf clean metadata    # Remove repo metadata

# Check for duplicate packages:
sudo dnf repoquery --duplicates

# Remove old kernel versions (keep latest 3):
sudo dnf remove --oldinstallonly

DNF5 Automatic Updates

Fedora 44 includes dnf5-automatic for scheduled, hands-off updates:

# Install:
sudo dnf install dnf5-automatic

# Configuration:
sudo nano /etc/dnf/automatic.conf

[commands]
# What to do: "default" = upgrade all, "security" = security only
upgrade_type = default

# Whether to download only (no install) or fully install:
download_updates = yes
apply_updates = yes

# When to reboot:
reboot = when-needed
# Options: never, when-changed, when-needed, always

[emitters]
# How to notify:
emit_via = stdio    # Also: motd, email, command
# To enable email:
# emit_via = email

[email]
email_from = root@localhost
email_to = you@example.com
email_host = localhost

[command]
# Run a custom command after update:
command_format = echo "DNF updated on $(hostname)"
command_path = /usr/bin/echo

Enable the timer:

# Enable and start the automatic update timer:
sudo systemctl enable --now dnf-automatic.timer

# Check when it runs next:
systemctl list-timers dnf-automatic.timer
# NEXT                         LEFT   LAST                         PASSED  UNIT
# Mon 2026-07-06 03:00:00 ...  12h    Sun 2026-07-05 03:00:00 ...  12h     dnf-automatic.timer

# Verify the service ran:
journalctl -u dnf-automatic.service --since today

Btrfs and Kernel Updates

On Btrfs, DNF5 with Snapper integration creates pre- and post-snapshot around package transactions automatically (if configured — see Section 4). This means every dnf upgrade has a safety net: if the update breaks your system, you boot into the pre-update snapshot.
4. Btrfs Snapshots and Rollback
Why Snapshots Matter

A Btrfs snapshot is a point-in-time copy of a subvolume. Snapshots are instantaneous and initially consume zero extra space — they share data blocks with the original (CoW). Space grows only as files diverge from the snapshot.

This means you can snapshot your entire root filesystem before a risky update, and if the update breaks anything, you revert to the snapshot in seconds.
Installing Snapper

Snapper is the standard snapshot management tool on Fedora:

# Install Snapper and the GRUB integration:
sudo dnf install snapper grub-btrfs

# Create a configuration for the root filesystem:
sudo snapper -c root create-config /
# This creates /etc/snapper/configs/root

# Verify:
sudo snapper list-configs
# Config | Subvolume
# root  | /

# Enable timeline and cleanup timers:
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer

Snapper Configuration

# View the configuration:
sudo cat /etc/snapper/configs/root

Key settings:

# Timeline snapshots (hourly, daily, etc.):
TIMELINE_CREATE="yes"
TIMELINE_CLEANUP="yes"

# How many to keep:
TIMELINE_LIMIT_HOURLY="10"   # Keep 10 hourly snapshots
TIMELINE_LIMIT_DAILY="7"     # Keep 7 daily snapshots
TIMELINE_LIMIT_WEEKLY="4"    # Keep 4 weekly snapshots
TIMELINE_LIMIT_MONTHLY="0"   # No monthly limit

# Cleanup limits:
NUMBER_LIMIT="10"           # Keep up to 10 numbered (manual/pre/post) snapshots
NUMBER_LIMIT_IMPORTANT="5"  # Keep up to 5 marked important

# Space limits:
SPACE_LIMIT="0.5"           # Max fraction of FS space for snapshots
FREE_LIMIT="0.2"            # Min free space fraction to maintain

Creating and Managing Snapshots

# List existing snapshots:
sudo snapper -c root list
# Type  | #  | Pre # | Date             | User | Cleanup  | Description
# single | 0  |       |                  | root |          | current
# pre    | 1  |       | 2026-07-05 09:15 | root | number   | before nvidia install
# post   | 2  | 1     | 2026-07-05 09:20 | root | number   | after nvidia install
# timeline| 3 |       | 2026-07-06 03:00 | root | timeline | timeline snapshot

# Create a manual snapshot:
sudo snapper -c root create --description "before big upgrade"

# Create a pre/post pair around a command:
sudo snapper -c root create --command "dnf upgrade -y" --description "system upgrade"
# Creates a 'pre' snapshot, runs the command, creates a 'post' snapshot

# Mark a snapshot as important (won't be cleaned up):
sudo snapper -c root modify --userdata "important=yes" 5

# Delete a snapshot:
sudo snapper -c root delete 3

# Delete multiple:
sudo snapper -c root delete 3 4 5

# Manual cleanup (enforces limits from config):
sudo snapper -c root cleanup timeline
sudo snapper -c root cleanup number

Viewing Snapshot Differences

# What changed between snapshot 1 and current state:
sudo snapper -c root status 1..0

# What changed between two snapshots:
sudo snapper -c root status 1..3

# View the diff of a specific file between snapshots:
sudo snapper -c root diff 1..3 /etc/dnf/dnf.conf

# Undo a single file change (revert one file to snapshot state):
sudo snapper -c root undochange 1..0 /etc/dnf/dnf.conf
# Restores /etc/dnf/dnf.conf to the state captured in snapshot 1

# Undo all changes between two snapshots:
sudo snapper -c root undochange 1..3
# WARNING: This reverts ALL files that differ between snapshots 1 and 3

Rolling Back the Entire System

For major failures (system won't boot after an update), you can roll back the entire root subvolume:

# If the system is still running:
sudo snapper -c root rollback 1
# Makes snapshot 1 the new default root subvolume

# Reboot into the rolled-back state:
sudo reboot

# If the system WON'T boot:
# 1. Reboot
# 2. At the GRUB menu, select "Fedora Snapshots" (provided by grub-btrfs)
# 3. Choose the snapshot you want to boot from
# 4. Once booted, make it permanent:
sudo snapper -c root rollback <snapshot_number>
sudo reboot

grub-btrfs Integration

With grub-btrfs installed, snapshots appear in the GRUB boot menu:

# Verify grub-btrfs is running:
sudo systemctl status grub-btrfsd
# Active: active (running)

# Manually update GRUB's snapshot list:
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

# Verify snapshots in GRUB:
ls /boot/grub2/grub.cfg

When you boot from a GRUB snapshot, you're in a read-only snapshot of the root filesystem. This lets you verify everything works before committing to a permanent rollback.
Snapshot Space Management

Snapshots share data initially, but diverge over time. Monitor their impact:

# How much space do snapshots use?
sudo btrfs filesystem usage /

# List snapshots with their exclusive (unique) size:
sudo snapper -c root list --columns type,number,date,size,exclusive
# Type    | # | Date             | Size  | Exclusive
# single  | 0 |                  | 0     | 0
# pre     | 1 | 2026-07-05 09:15 | 45G   | 2.3G
# post    | 2 | 2026-07-05 09:20 | 45G   | 500M
# timeline| 3 | 2026-07-06 03:00 | 45G   | 1.2G

# "Exclusive" = data unique to this snapshot (not shared with others)
# This is the actual space you'd reclaim by deleting this snapshot

5. Service Management with systemctl
systemd: The Master Controller

systemd is PID 1 — the first process to run, the last to exit. It manages every service on Fedora 44. Understanding systemctl is essential for system maintenance.
Checking Service Status

# Status of a service:
systemctl status sshd

● sshd.service - OpenSSH server daemon
     Loaded: loaded (/usr/lib/systemd/system/sshd.service; enabled; preset: enabled)
     Active: active (running) since Mon 2026-07-06 09:15:23 UTC; 5h ago
   Main PID: 1234 (sshd)
      Tasks: 1 (limit: 30891)
     Memory: 4.5M
        CPU: 234ms
     CGroup: /system.slice/sshd.service
             └─1234 /usr/sbin/sshd -D

Jul 06 09:15:23 fedora systemd[1]: Started OpenSSH server daemon.
Jul 06 14:23:45 fedora sshd[5678]: Accepted publickey for john from 192.168.1.50

Status line meanings:
State	Meaning
active (running)	Service is operational
active (exited)	Service ran once and completed (e.g., setup script)
inactive (dead)	Service stopped
failed	Service crashed or failed to start
activating	Service is starting up
deactivating	Service is shutting down

# Quick check (is it running?):
systemctl is-active sshd
# active

# Is it enabled to start at boot?
systemctl is-enabled sshd
# enabled

# Check for failed services:
systemctl --failed
# UNIT          LOAD   ACTIVE SUB    DESCRIPTION
# nginx.service loaded failed failed The NGINX HTTP and reverse proxy server

# List all services:
systemctl list-units --type=service

# List only running services:
systemctl list-units --type=service --state=running

# List enabled services:
systemctl list-unit-files --type=service --state=enabled

Starting, Stopping, Restarting

# Start a service:
sudo systemctl start sshd

# Stop:
sudo systemctl stop sshd

# Restart (stops then starts — brief downtime):
sudo systemctl restart sshd

# Reload (re-reads config WITHOUT stopping — no downtime):
sudo systemctl reload sshd
# Not all services support reload; check the unit file

# Restart but only if already running:
sudo systemctl try-restart sshd

Enabling and Disabling at Boot

# Enable (start at boot):
sudo systemctl enable sshd

# Enable and start in one command:
sudo systemctl enable --now sshd

# Disable (don't start at boot):
sudo systemctl disable sshd

# Disable and stop:
sudo systemctl disable --now sshd

# Mask (prevent manual or dependency-triggered start):
sudo systemctl mask nginx
# If another service tries to start nginx, systemd refuses
# Stronger than disable — disable just removes from boot

# Unmask:
sudo systemctl unmask nginx

Editing Service Configuration

# Edit a service's drop-in override:
sudo systemctl edit sshd.service

This opens an editor with an empty drop-in file. You add only the settings you want to change:

[Service]
# Restart if it crashes:
Restart=always
RestartSec=5s

# Add environment variable:
Environment="OPTIONS=-p 2222"

Save and exit. Then:

# Reload systemd to pick up changes:
sudo systemctl daemon-reload

# Restart the service:
sudo systemctl restart sshd

# Verify the override is active:
systemctl cat sshd.service
# Shows the original unit file PLUS your drop-in

    💡 Drop-In Files vs. Direct Edit

    Never edit /usr/lib/systemd/system/sshd.service directly — package updates will overwrite it. Drop-in overrides in /etc/systemd/system/sshd.service.d/override.conf persist across updates. systemctl edit creates and manages these files automatically.

Viewing Logs for a Service

# Recent logs for a service:
journalctl -u sshd --since today

# Follow live:
journalctl -u sshd -f

# Logs from previous boot (if it crashed):
journalctl -u sshd -b -1

# Last 50 lines:
journalctl -u sshd -n 50 --no-pager

6. Scheduled Tasks: systemd Timers
Timers vs. Cron

Traditional Unix uses cron for scheduled tasks. Fedora 44 supports cron, but systemd timers are the modern, preferred approach:
Feature	cron	systemd timers
Logging	None (silent)	Automatic (journal)
Dependencies	Manual	Service dependencies
Error handling	Manual	OnFailure= chains
State awareness	Runs blindly	Can skip if asleep, run on boot
Precision	Minute-level	Second-level
Management	crontab -e	systemctl
Visibility	Per-user files	systemctl list-timers
Listing Existing Timers

# All active timers:
systemctl list-timers
# NEXT                         LEFT   LAST                         PASSED  UNIT
# Mon 2026-07-06 15:00:00 ...  30min  Mon 2026-07-06 14:00:00 ...  30min   snapper-timeline.timer
# Mon 2026-07-06 03:00:00 ...  12h    Sun 2026-07-05 03:00:00 ...  12h     dnf-automatic.timer
# Tue 2026-07-07 00:00:00 ...  9h     Mon 2026-07-06 00:00:00 ...  15h     snapper-cleanup.timer
# Wed 2026-07-08 03:00:00 ...  1d     Wed 2026-07-05 03:00:00 ...  1d      fwupd-refresh.timer

# All timers (active and inactive):
systemctl list-timers --all

Creating a Custom Timer

Timers always come in pairs: a service (what to do) and a timer (when to do it).

Step 1: Create the service:

sudo nano /etc/systemd/system/backup-home.service

[Unit]
Description=Backup home directory to external drive

[Service]
Type=oneshot
ExecStart=/usr/bin/rsync -a --delete /home/john/ /mnt/backup/john/
# Full path to the script/command is required

Step 2: Create the timer:

sudo nano /etc/systemd/system/backup-home.timer

[Unit]
Description=Run home backup daily at 2 AM

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true
# Persistent = run missed jobs after boot (e.g., if system was off at 2 AM)

[Install]
WantedBy=timers.target

Timer schedule types:
Directive	Meaning	Example
OnCalendar=	Calendar-based schedule	*-*-* 02:00:00 (daily at 2 AM)
OnBootSec=	Time after boot	10min (10 minutes after boot)
OnUnitActiveSec=	Time after last activation	daily (daily since last run)
OnUnitInactiveSec=	Time after deactivation	1h (1 hour after completion)
RandomizedDelaySec=	Random jitter (avoid thundering herd)	10min

Calendar expression format:

DayOfWeek Year-Month-Day Hour:Minute:Second

*-*-* 02:00:00       = Every day at 2:00 AM
*-*-1..7 02:00:00     = 1st-7th of each month at 2 AM (first week)
Mon *-*-* 09:00:00   = Every Monday at 9 AM
*:0/15               = Every 15 minutes
*-*-* *:00:00        = Every hour on the hour

Step 3: Enable and start:

# Reload systemd:
sudo systemctl daemon-reload

# Enable and start the timer (NOT the service):
sudo systemctl enable --now backup-home.timer

# Verify:
systemctl list-timers backup-home.timer
# NEXT: Tue 2026-07-07 02:00:00 ...

# Manually trigger the service (test it):
sudo systemctl start backup-home.service

# Check the result:
journalctl -u backup-home.service --since today

Modifying Timers

# Edit a timer's drop-in:
sudo systemctl edit backup-home.timer
# Change the schedule:
[Timer]
OnCalendar=*-*-* 04:30:00

# Reload:
sudo systemctl daemon-reload
# The timer picks up the new schedule automatically

# Disable a timer (stop scheduled runs):
sudo systemctl disable --now backup-home.timer

# Re-enable:
sudo systemctl enable --now backup-home.timer

Cron (Legacy Reference)

Cron still exists for backward compatibility. If you encounter cron jobs or need a quick one-off schedule:

# Edit your crontab:
crontab -e

# View your crontab:
crontab -l

# System-wide crontab:
cat /etc/crontab

# System-wide cron directories:
ls /etc/cron.daily/
ls /etc/cron.weekly/
ls /etc/cron.monthly/

Crontab format:

# m   h  dom mon dow  command
  0   2  *   *   *    /usr/bin/rsync -a /home/john/ /mnt/backup/
#  │   │  │   │   │
#  │   │  │   │   └── Day of week (0-7, Sunday=0 or 7)
#  │   │  │   └────── Month (1-12)
#  │   │  └────────── Day of month (1-31)
#  │   └──────────── Hour (0-23)
#  └──────────────── Minute (0-59)

    💡 Prefer systemd Timers

    For any new scheduled task, use systemd timers. They provide automatic logging (visible via journalctl -u), error handling, and integration with the rest of the system. Cron is fine for legacy scripts, but systemd timers are the Fedora way.

7. Performance Profiling
iostat: Disk I/O Analysis

# Install:
sudo dnf install sysstat

# Report since boot, then per-second:
iostat
iostat 1 5    # 5 samples, 1 second apart

# Extended statistics (per-device throughput, latency):
iostat -x 1
# Device  rrqm/s  wrqm/s  r/s  w/s  rkB/s  wkB/s  avgrq-sz  avgqu-sz  await
# nvme0n1   0.0     1.2   2.3 4.5  256   512     120         0.02      3.45

# CPU and device report:
iostat -xz 1
# await > 100ms suggests disk bottleneck

Key columns:
Column	Meaning	Concern When
r/s, w/s	Read/write ops per second	—
rkB/s, wkB/s	Read/write throughput	—
await	Average wait time (ms)	> 20ms HDD, > 10ms SSD
%util	Percentage of time device busy	Consistently > 80%
vmstat: Virtual Memory Stats

# Report since boot:
vmstat
# procs --------memory-------- ---swap-- -----io---- -system-- -------cpu-------
#  r  b  swpd  free  buff cache  si  so   bi   bo   in   cs  us sy id wa st
#  1  0     0 85000  2000 30G     0   0   10   20  100 200   3  1 95  0  0

# Continuous monitoring (1-second intervals, 10 samples):
vmstat 1 10

Key columns:
Column	Meaning	Concern When
r	Processes waiting for CPU	> number of cores
b	Processes in uninterruptible sleep	Consistently > 0 (I/O bottleneck)
swpd	Swap used	Growing (RAM pressure)
si, so	Swap in/out per second	Non-zero (actively swapping)
bi, bo	Blocks in/out (disk I/O)	High values + high wa
us	User CPU %	—
sy	System CPU %	> 20% sustained
id	Idle %	Low = CPU-bound
wa	I/O wait %	> 5% = disk bottleneck
sar: System Activity Reporter

sar collects and stores performance data over time — useful for diagnosing intermittent issues:

# Install:
sudo dnf install sysstat

# Enable data collection:
sudo systemctl enable --now sysstat

# CPU usage today:
sar -u
# 09:00:01  CPU  %user  %nice %system %iowait %steal %idle
# 09:10:01  all   3.2    0.0    1.1     0.0      0.0   95.7
# 09:20:01  all   5.1    0.0    1.3     0.2      0.0   93.4

# Memory usage today:
sar -r

# Disk I/O today:
sar -d

# Network stats today:
sar -n DEV

# Specific time range:
sar -u -s 09:00:00 -e 12:00:00

# View data from a specific day:
sar -u -f /var/log/sa/sa05
# sa05 = data from the 5th of the month

powertop: Power Profiling

For laptops, powertop reveals what's consuming battery:

sudo dnf install powertop
sudo powertop

                              PowerTOP
   Summary  Idle stats  Frequency stats  Device stats  Tunables

Power est.    Usage       Events/s    Category       Description
  2.3 W      15.0%                    Device         Display backlight
  0.5 W      100%                     Device         Audio codec hwC0D0
  0.1 W      2.0 pkts/s              Device         Network interface: wlp2s0
  0.0 W                              Process        firefox (4567)
                                                  Wakeups/s  Category  Description
                                                   120.0     Timer     tick_sched_timer
                                                    45.2     Timer     hrtimer_wakeup
                                                    12.3     Interrupt [6] ehci_hcd:usb1

Tuning tab:

Bad    Runtime PM for PCI Device Intel Corporation
Bad    Autosuspend for USB device usbmouse
Good   Wi-Fi power saving for device wlp2s0

Toggle tunables with Enter, or apply all recommended settings:

# Auto-tune all recommendations:
sudo powertop --auto-tune

Quick Performance Snapshot

# CPU:
uptime
# 14:23:45 up 2 days,  4:30, 2 users, load average: 0.42, 0.38, 0.31

# Memory:
free -h

# Disk:
df -h /

# I/O:
iostat -x 1 3

# Top CPU consumers:
ps -eo pid,%cpu,%mem,comm --sort=-%cpu | head -10

# Top memory consumers:
ps -eo pid,%cpu,%mem,comm --sort=-%mem | head -10

# System load from /proc:
cat /proc/loadavg

8. Routine Maintenance Checklist

A healthy Fedora 44 system benefits from periodic attention. Here's a practical maintenance schedule:
Weekly

# 1. Update the system:
sudo dnf upgrade

# 2. Clean DNF cache:
sudo dnf clean all

# 3. Check disk space:
df -h
sudo btrfs filesystem usage /

# 4. Vacuum journal:
sudo journalctl --vacuum-size=500M

# 5. Check for failed services:
systemctl --failed

# 6. Remove orphaned packages:
sudo dnf autoremove

# 7. Clean Flatpak leftovers:
flatpak uninstall --unused

Monthly

# 1. Check for old snapshots:
sudo snapper -c root list
sudo snapper -c root cleanup timeline

# 2. Check firmware updates:
fwupdmgr refresh
fwupdmgr get-updates

# 3. Review installed packages:
rpm -qa | wc -l
dnf history | head -20

# 4. Check large files:
sudo ncdu -x / --exclude /proc --exclude /sys

# 5. Verify disk health (if SMART-supported):
sudo dnf install smartmontools
sudo smartctl -a /dev/nvme0n1 | grep -E "health|temperature|percentage_used"
# For SATA SSDs:
sudo smartctl -H /dev/sda

# 6. Reboot if kernel was updated:
# Check if running kernel matches installed kernel:
uname -r
rpm -q kernel | sort -V | tail -1
# If different, reboot to use the new kernel

Quarterly

# 1. Full system audit:
# Review all enabled services:
systemctl list-unit-files --type=service --state=enabled

# 2. Check SSH config:
sudo cat /etc/ssh/sshd_config | grep -E "PermitRootLogin|PasswordAuthentication"

# 3. Check firewall rules:
sudo firewall-cmd --list-all

# 4. Check for security advisories:
dnf updateinfo list security all

# 5. Review user accounts:
cut -d: -f1,3 /etc/passwd | grep -v nologin | grep -v false

# 6. Full system hardware check:
sudo dmesg | grep -iE "error|fail|warn"
sudo smartctl -a /dev/nvme0n1
sensors

# 7. Rebuild initramfs (after major changes):
sudo dracut --force

# 8. Review snapper config:
sudo snapper -c root list
sudo btrfs filesystem usage /

Maintenance Automation Script

Save this as a weekly manual or cron/timer-triggered script:

#!/bin/bash
# system-health-check.sh — Quick system health snapshot
# Run with: sudo ./system-health-check.sh

echo "=========================================="
echo "    Fedora 44 System Health Report"
echo "    $(date)"
echo "=========================================="

echo ""
echo "--- SYSTEM UPTIME ---"
uptime

echo ""
echo "--- FAILED SERVICES ---"
systemctl --failed || echo "None"

echo ""
echo "--- DISK USAGE ---"
df -h / /home /boot/efi 2>/dev/null
echo ""
sudo btrfs filesystem usage / 2>/dev/null | head -5

echo ""
echo "--- MEMORY ---"
free -h

echo ""
echo "--- TOP CPU CONSUMERS ---"
ps -eo pid,%cpu,%mem,comm --sort=-%cpu | head -5

echo ""
echo "--- TOP MEMORY CONSUMERS ---"
ps -eo pid,%cpu,%mem,comm --sort=-%mem | head -5

echo ""
echo "--- JOURNAL SIZE ---"
journalctl --disk-usage

echo ""
echo "--- SNAPSHOTS ---"
sudo snapper -c root list 2>/dev/null || echo "Snapper not configured"

echo ""
echo "--- PENDING UPDATES ---"
dnf check-update 2>/dev/null | tail -5

echo ""
echo "--- SMART HEALTH ---"
sudo smartctl -H /dev/nvme0n1 2>/dev/null | grep -i "result" || echo "smartctl not available"

echo ""
echo "--- TEMPERATURES ---"
sensors 2>/dev/null | grep -E "Core|Package|temp" | head -10 || echo "lm_sensors not installed"

echo ""
echo "=========================================="
echo "    Health check complete."
echo "=========================================="

Try It Yourself
Exercise 1: Disk Audit

# Check your filesystem space:
df -h

# Find the 10 largest directories in /home:
du -sh /home/*/ 2>/dev/null | sort -rh | head -10

# Use ncdu to explore interactively:
ncdu ~

# Find the 10 largest individual files:
sudo find / -path /proc -prune -o -path /sys -prune -o -path /run -prune \
  -o -type f -printf '%s %p\n' 2>/dev/null | sort -rn | head -10

Exercise 2: Log Exploration

# View boot history:
journalctl --list-boots

# Errors from current boot:
journalctl -b -p err --no-pager

# Last 30 lines of a specific service:
journalctl -u sshd -n 30 --no-pager

# Check journal disk usage:
journalctl --disk-usage

# Vacuum the journal to 500MB:
sudo journalctl --vacuum-size=500M

Exercise 3: Service Management

# List all running services:
systemctl list-units --type=service --state=running

# Check for failed services:
systemctl --failed

# Pick a service and examine it:
systemctl status sshd
systemctl is-enabled sshd
systemctl is-active sshd

# View its logs:
journalctl -u sshd --since today --no-pager | head -20

Exercise 4: Timer Creation

# Create a simple backup service:
sudo nano /etc/systemd/system/test-backup.service
# Add:
# [Service]
# Type=oneshot
# ExecStart=/bin/tar cf /tmp/test-backup.tar /home/$USER/Documents/

# Create the timer:
sudo nano /etc/systemd/system/test-backup.timer
# Add:
# [Unit]
# Description=Test backup timer
# [Timer]
# OnBootSec=2min
# OnUnitActiveSec=daily
# [Install]
# WantedBy=timers.target

# Enable:
sudo systemctl daemon-reload
sudo systemctl enable --now test-backup.timer

# Verify:
systemctl list-timers test-backup.timer

# Trigger manually:
sudo systemctl start test-backup.service
journalctl -u test-backup.service --since "1 minute ago" --no-pager

# Clean up:
sudo systemctl disable --now test-backup.timer
sudo rm /etc/systemd/system/test-backup.{service,timer}
sudo systemctl daemon-reload

Exercise 5: Snapper Setup (if on Btrfs)

# Check if your root is Btrfs:
findmnt -no FSTYPE /

# Install Snapper:
sudo dnf install snapper

# Create configuration:
sudo snapper -c root create-config /

# Take a manual snapshot:
sudo snapper -c root create --description "test snapshot"

# List snapshots:
sudo snapper -c root list

# Delete the test snapshot:
sudo snapper -c root delete <number>

# Enable automatic timeline snapshots:
sudo systemctl enable --now snapper-timeline.timer

Exercise 6: Performance Baseline

# Capture system baseline:
uptime
free -h
df -h /
cat /proc/loadavg

# Disk I/O:
iostat -x 1 3 2>/dev/null || echo "Install sysstat: sudo dnf install sysstat"

# CPU and memory consumers:
ps -eo pid,%cpu,%mem,comm --sort=-%cpu | head -10
ps -eo pid,%cpu,%mem,comm --sort=-%mem | head -10

# Temperatures:
sensors 2>/dev/null || echo "Install lm_sensors: sudo dnf install lm_sensors"

# Save for comparison:
uptime > /tmp/baseline.txt
free -h >> /tmp/baseline.txt
df -h / >> /tmp/baseline.txt
cat /tmp/baseline.txt

Exercise 7: Update Review

# Check for updates without installing:
dnf check-update

# View update history:
dnf history | head -10

# Review a specific transaction:
dnf history info <ID>

# Check for security advisories:
dnf updateinfo list security all 2>/dev/null | head -10

# Clean DNF cache:
sudo dnf clean all

Exercise 8: Run the Health Check Script

# Create the health check script from Section 8:
# (Copy the script content above into a file)

chmod +x system-health-check.sh
sudo ./system-health-check.sh

# Review the output — it's your system's vital signs
# Save it as a baseline:
sudo ./system-health-check.sh > ~/health-baseline-$(date +%Y%m%d).txt

Troubleshooting Reference
Symptom	Likely Cause	First Check
Disk full warnings	Logs, cache, or snapshots	df -h, ncdu /var, journalctl --disk-usage, snapper list
System feels slow	I/O or CPU bottleneck	iostat -x 1, top, vmstat 1
Service won't start	Configuration error or missing dependency	journalctl -u <service> -b, systemctl status <service>
Journal taking too much space	No size limits configured	journalctl --disk-usage, configure /etc/systemd/journald.conf.d/
Updates failing	Corrupted cache or repo metadata	sudo dnf clean all, sudo dnf upgrade
Snapper snapshots too many	Cleanup timer not running or limits too high	snapper list, systemctl status snapper-cleanup.timer
System won't boot after update	Bad kernel or driver	Boot from GRUB snapshot (grub-btrfs), snapper rollback
Btrfs filesystem read-only	Disk nearly full (metadata exhaustion)	btrfs filesystem usage /, delete snapshots, btrfs filesystem resize
High load but low CPU%	I/O wait (disk bottleneck)	top check wa%, iostat -x check await
Failed services at boot	Dependency timing or configuration	systemctl --failed, journalctl -b -p err
DNF automatic not running	Timer disabled or config error	systemctl status dnf-automatic.timer, journalctl -u dnf-automatic
Boot takes long	Many services or slow network mount	systemd-analyze blame, systemd-analyze critical-chain
What's Next

You now possess the complete toolkit for keeping Fedora 44 healthy: disk monitoring (df with -h/-i/-T flags, du with -sh/--max-depth, ncdu interactive explorer, Btrfs-specific btrfs filesystem df/show/usage/subvolume list commands, space management for Btrfs with snapshot-aware cleanup, finding and cleaning common space hogs including DNF cache, journal, user cache, trash, old kernels, Flatpak runtimes), log management (systemd-journald binary structured logging, journalctl with -b boot filtering, --since/--until time ranges, -p priority levels 0-7, -u unit filtering, -f follow, --list-boots, -g grep, -o output formats, --disk-usage, --vacuum-size/--vacuum-time/--rotate cleanup, persistent config in /etc/systemd/journald.conf.d/ with SystemMaxUse/MaxRetentionSec, traditional /var/log/ files, logrotate with /etc/logrotate.d/ rules), system updates (DNF5 dnf upgrade/--security/--downloadonly, dnf history with undo/redo, dnf autoremove/clean/remove --oldinstallonly, dnf5-automatic package with /etc/dnf/automatic.conf for upgrade_type/download_updates/apply_updates/reboot, dnf-automatic.timer systemd timer), Btrfs snapshots and rollback (Snapper installation with create-config, configuration in /etc/snapper/configs/root with TIMELINE_* and NUMBER_* limits, manual snapper create --description, pre/post pairs with --command, snapper list/delete/modify/rollback/undochange, snapper status/diff for viewing changes, snapper cleanup timeline/number, grub-btrfs integration for bootable snapshots, snapshot space analysis with exclusive size tracking), service management (systemctl status/is-active/is-enabled/start/stop/restart/reload, enable --now/disable --now, mask/unmask, --failed, list-units/list-unit-files, systemctl edit drop-in overrides with daemon-reload, systemctl cat for verifying overrides, journalctl -u for per-service logs), scheduled tasks (systemd timers vs cron comparison, systemctl list-timers, creating service+timer pairs with OnCalendar/OnBootSec/OnUnitActiveSec/Persistent, calendar expression syntax, systemctl edit for modifying timers, legacy crontab -e/-l with m h dom mon dow format, /etc/cron.daily/weekly/monthly directories), performance profiling (iostat -x with await/%util analysis, vmstat 1 with r/b/swpd/si/so/wa columns, sar -u/-r/-d/-n historical analysis with sysstat, powertop with auto-tune for laptop power, smartctl -H for disk health, maintenance automation script for weekly health snapshots), routine maintenance schedule (weekly: dnf upgrade, dnf clean, df, journal vacuum, systemctl --failed, autoremove, flatpak cleanup; monthly: snapper cleanup, fwupd, rpm audit, ncdu large files, smartctl, kernel check; quarterly: enabled service review, SSH/firewall config audit, security advisory check, dmesg error scan, dracut rebuild, snapper config review), progressive exercises covering disk auditing, log exploration, service management, timer creation, Snapper setup, performance baselines, update reviews, and the health check script, plus a comprehensive twelve-entry troubleshooting reference table.

Chapter 25 introduces shell scripting — turning the commands you've learned into reusable programs. You'll learn about variables, conditionals, loops, functions, error handling, and practical scripts for system administration. Everything you've typed manually up to this point can be automated, scheduled, and generalized. That's what scripting enables.

Until then, run the health check script on your system. Save the output. In a week, run it again and compare. The differences tell you everything about how your system breathes, grows, and changes over time — and that insight is the foundation of confident system ownership.

    ✅ Chapter 24 Complete You understand disk usage monitoring (df with -h/-i/-T/-t/-x flags, inode exhaustion vs space exhaustion, Btrfs performance thresholds at 80%/90%, du with -sh/--max-depth/2>/dev/null for permission suppression, recursive sorting with sort -rh, ncdu interactive curses explorer with d-delete/s-sort/Enter-drill-down/q-quit, Btrfs-specific tools btrfs filesystem df/show/usage/subvolume list, CoW shared extents explaining df discrepancies, cleaning up space hogs — DNF cache with dnf clean all, journal with journalctl --vacuum-size, user cache ~/.cache/thumbnails/~/.cache/mozilla, trash ~/.local/share/Trash, old kernels with dnf remove --oldinstallonly, Flatpak with flatpak uninstall --unused, finding largest files with find+printf+sort or ncdu -x), log management (systemd-journald as structured binary logging replacing scattered text files, journalctl -b current boot, --since/--until time ranges, -p priority levels emerg through debug, -u unit filtering, -f follow, --list-boots boot index, -n line count, -g grep pattern, -o output formats json/export/cat/short-iso, --no-pager, --disk-usage, --vacuum-size/--vacuum-time/--rotate cleanup, --vacuum-files count limit, journald config in /etc/systemd/journald.conf and /etc/systemd/journald.conf.d/ drop-ins with SystemMaxUse/SystemKeepFree/MaxRetentionSec/SystemMaxFileSize/RateLimit settings, traditional /var/log/ files, logrotate with /etc/logrotate.conf and /etc/logrotate.d/ rules, rotation directives rotate/size/compress/missingok/notifempty, logrotate -d debug and -f force), system updates (DNF5 dnf upgrade/--security/--downloadonly, dnf history with info/undo/redo, undo limitations for config/data/db changes, dnf autoremove/clean all/clean packages/clean metadata, dnf repoquery --duplicates, dnf remove --oldinstallonly for old kernels, dnf5-automatic package with /etc/dnf/automatic.conf configuring upgrade_type/download_updates/apply_updates/reboot/email, systemctl enable --now dnf-automatic.timer, systemctl list-timers verification, journalctl -u dnf-automatic.service review), Btrfs snapshots and rollback (snapshot concept: instant CoW point-in-time copy with zero initial cost and growing divergence, Snapper installation with snapper -c root create-config /, configuration in /etc/snapper/configs/root with TIMELINE_CREATE/CLEANUP/LIMIT_HOURLY/DAILY/WEEKLY/MONTHLY and NUMBER_LIMIT and SPACE_LIMIT/FREE_LIMIT, snapper list/create --description/delete/modify --userdata important=yes, pre/post pairs with create --command, snapper status 1..0 and diff 1..3 /path, undochange 1..0 /path for single-file revert, undochange 1..3 for full revert, rollback N for system-wide, cleanup timeline/number for enforcing limits, snapper-timeline.timer and snapper-cleanup.timer for automatic operation, grub-btrfs for bootable snapshots in GRUB menu, snapshot space analysis with exclusive size tracking, Btrfs+Snopper integration with DNF5 for automatic pre/post-transaction snapshots), service management (systemd as PID 1, systemctl status with active/inactive/failed/activating states, is-active/is-enabled, --failed for checking failures, list-units --type=service and list-unit-files, start/stop/restart/reload/try-restart, enable --now/disable --now, mask/unmask for hard prevention, systemctl edit for drop-in overrides in /etc/systemd/system/service.d/override.conf, daemon-reload after unit file changes, systemctl cat for verifying overrides, journalctl -u service for per-service logs), scheduled tasks (systemd timers vs cron comparison table, systemctl list-timers/--all, creating service+timer pairs with OnCalendar/OnBootSec/OnUnitActiveSec/OnUnitInactiveSec/Persistent/RandomizedDelaySec, calendar expression syntax DayOfWeek Year-Month-Day Hour:Minute:Second, systemctl enable --now timer.timer, manual systemctl start service.service, systemctl edit timer.timer for schedule modification, legacy cron with crontab -e/-l and m h dom mon dow format, /etc/cron.daily/weekly/monthly directories, recommendation to prefer systemd timers for logging/dependencies/error-handling), performance profiling (iostat -x 1 with await/%util/rkB/s columns and threshold guidelines, vmstat 1 N with r/b/swpd/si/so/bi/bo/us/sy/id/wa columns, sar -u/-r/-d/-n with sysstat package and historical data in /var/log/sa/saXX, powertop with auto-tune for laptop power optimization, smartctl -H for disk health, uptime/free -h/cat /proc/loadavg quick snapshot, systemd-analyze blame for boot timing), routine maintenance (weekly: dnf upgrade, dnf clean all, df -h, journal vacuum, systemctl --failed, dnf autoremove, flatpak uninstall --unused; monthly: snapper cleanup, fwupd refresh+get-updates, rpm audit, ncdu large files, smartctl, kernel running vs installed check; quarterly: enabled services review, SSH/firewall config audit, dnf updateinfo security, dmesg error scan, dracut --force rebuild, snapper config review, comprehensive system-health-check.sh automation script), progressive exercises covering disk auditing, log exploration, service management, timer creation, Snapper setup, performance baselines, update reviews, and health check script, plus a comprehensive twelve-entry troubleshooting reference table.

