# Chapter 16: System Monitoring and Maintenance

## Knowing What Your System Is Doing

Your computer is a complex machine. Dozens of processes run simultaneously, consuming CPU cycles, eating memory, reading and writing to disk, communicating over the network — all mostly invisible to you. When everything works well, you don't need to think about it. But when something slows down, freezes, or misbehaves, you need to see inside.

This chapter teaches you how to monitor your system's health from both the graphical interface and the terminal. You'll learn to check CPU usage, memory consumption, disk space, running processes, and system logs. These are the same tools that experienced Linux administrators use every day.

## The Resources App (Graphical)

Ubuntu 26.04 replaces the old GNOME System Monitor with a modern app simply called **Resources**. If you followed the post-install checklist in Chapter 7, it's already installed. If not:

flatpak install flathub net.nokyan.Resources

Open it by pressing Super, typing "Resources," and pressing Enter.
What Resources Shows You

The Resources app presents a clean, real-time dashboard with separate sections for each major subsystem:

CPU:

    Overall CPU utilization percentage (per-core and total)
    Per-core usage graphs
    Clock speed (current frequency)
    Number of processes running

Memory:

    RAM usage (used / total)
    Swap usage
    Cache and buffer allocation

Disk:

    Per-drive read/write speeds
    Total I/O activity
    Storage capacity for each mounted filesystem

Network:

    Download and upload speeds
    Active connections
    Per-interface statistics (Ethernet, Wi-Fi, VPN)

GPU (if detected):

    GPU utilization
    VRAM usage
    Temperature (on supported hardware)

Below the graphs, a process list shows every running application and system process, sortable by CPU usage, memory consumption, or name. Right-click any process to kill it, open its file location, or view its properties.

    💡 Resources vs. Old System Monitor

    The old GNOME System Monitor was functional but hadn't been meaningfully updated in years. Resources is a complete rewrite — cleaner interface, GPU monitoring (the old tool didn't show this), per-disk I/O graphs, and better process management. The old System Monitor is still available if you prefer it:

    sudo apt install gnome-system-monitor

Terminal Monitoring Tools

The graphical Resources app is great for a quick overview. But terminal tools are faster, scriptable, and work even when the graphical interface is struggling. Every Linux administrator relies on these commands.
top — Real-Time Process Monitor

top

top opens a full-screen, continuously updating display of system activity. It's the terminal equivalent of the Resources process list — but it's available on every Linux system, everywhere, always.

What You See:

top - 14:32:07 up  3:25,  2 users,  load average: 0.42, 0.58, 0.61
Tasks: 187 total,   1 running, 186 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.4 id,  0.3 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem :  16384.0 total,   8192.3 free,   4096.5 used,   4095.2 buff/cache
MiB Swap:   8192.0 total,   8192.0 free,      0.0 used.  12288.0 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 1234 username   20   0  891234  23456   8901 S   2.3  0.1   0:05.42 firefox
 1567 username   20   0  234567  12345   2345 S   1.7  0.1   0:03.12 gnome-shell
 1890 username   20   0  123456   8901   1234 S   0.3  0.0   0:01.23 Ptyxis

Reading the Header:
Line	What It Tells You
top - 14:32:07	Current time
up 3:25	System has been running for 3 hours, 25 minutes
2 users	Two user sessions active
load average: 0.42, 0.58, 0.61	System load over 1, 5, and 15 minutes
Tasks: 187 total	187 processes running
%Cpu(s): 3.2 us	3.2% CPU used by user processes
95.4 id	95.4% CPU idle (nothing to do)
MiB Mem: 16384.0 total	16 GB total RAM
8192.3 free	8 GB free RAM

Reading the Process List:
Column	Meaning
PID	Process ID (unique number for each running process)
USER	Who owns the process
PR	Priority (lower = higher priority)
NI	Nice value (-20 to 19; lower = more CPU priority)
VIRT	Virtual memory used
RES	Physical RAM used (the meaningful number)
SHR	Shared memory
S	Status: R=running, S=sleeping, D=uninterruptible sleep, Z=zombie
%CPU	Percentage of CPU being used
%MEM	Percentage of RAM being used
TIME+	Total CPU time consumed
COMMAND	Name of the process

Interacting with top:
Key	Action
q	Quit
h	Help screen
M	Sort by memory usage
P	Sort by CPU usage (default)
N	Sort by PID
T	Sort by total CPU time
k	Kill a process (prompts for PID)
r	Renice a process (change priority)
1	Show individual CPU cores (instead of combined)
H	Show individual threads

Load Average Explained:

The load average (e.g., 0.42, 0.58, 0.61) represents the average number of processes waiting for CPU time over the last 1, 5, and 15 minutes. Interpretation:
Load Average	Meaning
Below 1.0 (single-core)	System is idle — no backlog
Around 1.0 (single-core)	One core is fully utilized — normal
Above 1.0 (single-core)	Processes are queuing — some slowdown
At core count (multi-core)	All cores fully utilized
Well above core count	System is overloaded — investigate

A load average of 4.0 on an 8-core system is fine. A load average of 4.0 on a 2-core system means the CPU is overloaded. Context matters — always compare to your core count.

    💡 Checking Your Core Count

    Run nproc to see how many CPU cores your system has. Then interpret load average relative to that number.

htop — A Friendlier top

htop is an enhanced version of top with color-coded output, scrollable process lists, and visual bars showing CPU, memory, and swap usage. It's not installed by default:

sudo apt install htop

Then run:

htop

Advantages over top:

    Color-coded bars for each CPU core
    Visual memory and swap meters
    Scrollable and sortable process list (use arrow keys)
    Tree view (press F5) showing parent-child process relationships
    Function key labels at the bottom (F5=tree, F6=sort, F9=kill, F10=quit)
    Mouse support (click column headers to sort)

Common htop interactions:
Key	Action
q or F10	Quit
F5	Toggle tree view
F6	Sort by column
F9	Kill selected process
F4	Filter by name
F3	Search for process
+ / -	Expand/collapse process tree
Space	Tag a process for batch operations

    💡 top vs. htop

    top is everywhere — installed on every Linux system. htop needs installation but is vastly more readable. Install htop for daily use; remember top exists as a fallback.

ps — Process Snapshot

While top and htop show a continuously updating view, ps gives you a one-time snapshot of running processes:

ps aux

This lists all processes for all users. The output is extensive, so it's commonly piped:

ps aux | grep firefox

Finds any Firefox process running. This is the most common way to answer "is [program] running?"

Understanding ps aux columns:
Column	Meaning
USER	Process owner
PID	Process ID
%CPU	CPU usage
%MEM	Memory usage
VSZ	Virtual memory size (KB)
RSS	Resident set size (physical RAM in KB)
TTY	Terminal associated with the process
STAT	Process state (S=sleeping, R=running, etc.)
START	When the process started
TIME	Total CPU time
COMMAND	The command that launched the process

Other useful ps patterns:

ps aux --sort=-%mem | head -10

Top 10 processes by memory usage.

ps -u username

Show only processes owned by a specific user.
kill and pkill — Stopping Processes

When a program freezes or won't respond, you can force it to stop from the terminal.

kill (by PID):

kill 1234

Sends a termination signal to process 1234. This is the polite way — it asks the process to shut down cleanly.

If the process doesn't respond, escalate:

kill -9 1234

-9 (SIGKILL) is the nuclear option — the system kills the process immediately without giving it a chance to clean up. Use this only when a regular kill doesn't work.

Signal hierarchy:
Signal	Number	Meaning
SIGTERM	15 (default)	"Please shut down" — polite, allows cleanup
SIGKILL	9	"Die now" — immediate, no cleanup
SIGHUP	1	"Hang up" — often reloads configuration

    💡 Finding the PID

    To kill a process, you need its PID. Get it with:

    ps aux | grep programname

    Or use pgrep:

    pgrep firefox

    This outputs just the PID(s) — no other information:

    1234
    1567

pkill and killall (by name):

pkill firefox

Kills all processes named "firefox" — no need to find the PID first. Extremely convenient.

killall firefox

Same thing, slightly different command. killall matches exact process names; pkill matches patterns (more flexible).

    ⚠️ Be Specific with pkill

    pkill fire would kill any process with "fire" in its name — Firefox, firewall tools, etc. Always type enough of the name to be unambiguous.

Disk Space Monitoring

Running out of disk space causes bizarre problems — failed updates, application crashes, data corruption. Monitoring disk space is essential maintenance.
df — Filesystem Disk Space

df -h

The -h flag (human-readable) shows sizes in GB/MB instead of raw bytes:

Filesystem      Size  Used Avail Use% Mounted on
/dev/sda2       468G   89G  356G  20% /
tmpfs           8.0G  2.1M  8.0G   1% /run
/dev/sda1       511M  6.1M  505M   2% /boot/efi

Column	Meaning
Filesystem	The device/partition
Size	Total capacity
Used	Space currently used
Avail	Space available
Use%	Percentage used
Mounted on	Where in the directory tree this drive lives

Key things to watch:

    Root filesystem (/): If this hits 95%+, urgent action needed
    Boot partition (/boot/efi): Small partition, rarely fills up but check if updates fail

    💡 What Does "tmpfs" Mean?

    tmpfs entries are temporary filesystems stored in RAM — they're not real disks. The /run entry stores runtime data for system services. You don't need to manage these — they're automatically sized and cleared on reboot.

du — Directory Space Usage

While df shows filesystem-level usage, du shows how much space individual directories and files consume:

du -h ~/Downloads

Shows the size of each file and subdirectory inside Downloads, with human-readable sizes.

Most useful pattern — find what's eating your disk:

du -sh ~/* | sort -rh | head -10

    du -sh ~/* — summarize the size of each item in your home directory
    sort -rh — sort by size, largest first (human-readable)
    head -10 — show only the top 10

Output:

4.2G    /home/username/Videos
3.1G    /home/username/Downloads
1.8G    /home/username/.cache
1.5G    /home/username/Downloads
850M    /home/username/.local
...

This immediately shows you what's consuming your space. Is your .cache directory 5 GB? Maybe it's time to clear it. Are your Downloads full of old ISOs? Clean them out.

Going deeper:

du -h --max-depth=1 ~/Downloads | sort -rh

Shows one level of subdirectories inside Downloads — identifying the biggest space consumers within that folder.
Memory Monitoring
free — RAM and Swap at a Glance

free -h

               total        used        free      shared  buff/cache   available
Mem:            16Gi       4.0Gi       8.2Gi       230Mi       4.1Gi        11Gi
Swap:          8.0Gi          0B       8.0Gi

Column	Meaning
total	Total RAM/swap installed
used	Currently in use by applications
free	Completely unused
buff/cache	Used by the system for caching (reclaimable)
available	Realistic estimate of usable memory

    💡 Understanding Linux Memory

    Linux aggressively caches file data in RAM — the buff/cache number. This is not wasted memory. If an application needs RAM, the system instantly releases cached data. The available column tells you the real amount of usable memory — not the free column.

    Many newcomers panic when they see 12 GB "used" on a 16 GB system. But if 8 GB of that is buff/cache, the system is healthy. Only worry about memory when:

        available drops below ~1 GB
        Swap usage is increasing (means the system is using disk as overflow RAM — slow)
        Applications are crashing with "out of memory" errors

swapon — Swap Partition/File Status

swapon --show

NAME      TYPE      SIZE   USED  PRIO
/dev/zram0 partition   8G    0B    -2

Shows swap devices. Ubuntu 26.04 may use zRAM (compressed RAM swap) instead of a disk-based swap partition — faster but uses CPU for compression/decompression.
System Services with systemctl

Ubuntu uses systemd as its init system — the program that starts and manages background services (called "units" or "services"). These include networking, Bluetooth, audio, printing, SSH, and dozens more.
Checking Service Status

systemctl status NetworkManager

Output:

● NetworkManager.service - Network Manager
     Loaded: loaded (/lib/systemd/system/NetworkManager.service; enabled)
     Active: active (running) since Sat 2026-07-04 14:32:07 UTC; 3h ago
   Main PID: 890 (NetworkManager)
     ...

The key line is Active: active (running) — this means the service is healthy. If it says inactive or failed, the service isn't running.
Common systemctl Commands
Command	What It Does
systemctl status servicename	Check if a service is running
systemctl start servicename	Start a stopped service
systemctl stop servicename	Stop a running service
systemctl restart servicename	Restart a service (apply new config)
systemctl enable servicename	Make a service start automatically at boot
systemctl disable servicename	Prevent a service from starting at boot
systemctl is-enabled servicename	Check if a service is set to start at boot
systemctl list-units --type=service	List all services and their status

    💡 Service Names

    Service names are case-sensitive and usually lowercase with hyphens. Common ones:

        NetworkManager — networking
        bluetooth — Bluetooth
        cups — printing
        ssh — SSH server (if installed)
        ufw — firewall
        snapd — Snap package daemon
        timeshift — system snapshots (if installed)

    You can tab-complete service names in most terminals: type systemctl status net + Tab.

    ⚠️ Starting, Stopping, and Disabling Services

    start, stop, restart, enable, and disable require sudo. status and is-enabled work without sudo for most services.

    Be cautious when disabling services — some are critical for system function. If you don't know what a service does, look it up before stopping or disabling it.

System Logs with journalctl

When things go wrong — a service won't start, an application crashes mysteriously, hardware misbehaves — the system logs tell you what happened. Ubuntu uses systemd journal for centralized logging.
Viewing Logs

Current boot (everything since last startup):

journalctl -b

Opens in less — use Spacebar to page through, / to search, q to quit.

Previous boot (if the problem happened before a restart):

journalctl -b -1

-1 means "the boot before the current one." -2 goes two boots back.

Today's logs:

journalctl --since today

Last hour:

journalctl --since "1 hour ago"

Specific time range:

journalctl --since "2026-07-04 09:00:00" --until "2026-07-04 12:00:00"

Filtering Logs

Errors only (priority level 3 and below):

journalctl -p err

Priority levels:
Code	Level	Meaning
0	emerg	System is unusable
1	alert	Action must be taken immediately
2	crit	Critical conditions
3	err	Error conditions
4	warning	Warning conditions
5	notice	Normal but significant
6	info	Informational messages
7	debug	Debug-level messages

Filter by service:

journalctl -u NetworkManager

Shows only logs from NetworkManager.

Filter by executable:

journalctl /usr/bin/firefox

Shows only logs from Firefox.
Following Logs in Real Time

journalctl -f

Continuously displays new log entries as they're generated — the system log equivalent of tail -f. Press Ctrl+C to stop.

Combined filtering with follow:

journalctl -u ssh -f

Follows only SSH service logs in real time. Useful when troubleshooting a specific service.
Disk Usage of Logs

Logs can accumulate over time. Check how much space they use:

journalctl --disk-usage

If logs are consuming too much space, reduce them:

sudo journalctl --vacuum-size=500M

Keeps only the most recent 500 MB of logs, deleting older entries.

sudo journalctl --vacuum-time=7d

Keeps only logs from the last 7 days.
Routine Maintenance Tasks

Monitoring is reactive — checking when something seems wrong. Maintenance is proactive — preventing problems before they occur.
1. Keep Your System Updated

Already covered in Chapters 7 and 10, but it's the single most important maintenance task:

sudo apt update && sudo apt upgrade

Also check for Snap updates (automatic, but you can verify):

snap refresh

2. Clean Up Old Packages

When you install updates, the old package versions are cached on disk. Over time, this can consume gigabytes:

sudo apt autoclean

Removes package files for versions that are no longer downloadable from the repositories.

sudo apt autoremove

Removes dependency packages that were installed as dependencies for other packages that have since been removed. This is surprisingly effective — you'll often recover hundreds of MB.

sudo apt clean

Removes ALL cached package files from /var/cache/apt/archives/. This frees the most space but means packages need to be re-downloaded if you reinstall them.

    💡 Safe Cleanup Order

    Run these in this order periodically (monthly is fine):

        sudo apt autoremove — remove unused dependencies
        sudo apt autoclean — remove obsolete package caches
        sudo journalctl --vacuum-size=500M — trim logs
        Clean old Snap versions (below)

    This routine is safe and recovers meaningful disk space.

3. Clean Up Old Snap Versions

Snap retains old versions of packages after updates. Each old version can consume hundreds of MB:

snap list --all

Shows all installed Snaps, including disabled (old) versions. To remove them:

LANG=C snap list --all | awk '/disabled/{print $1, $3}' | xargs -rn2 sudo snap remove

This one-liner finds all disabled Snap revisions and removes them. It's safe — Snap keeps the current version intact.
4. Clear Thumbnail Cache

GNOME generates thumbnails for images and videos in ~/.cache/thumbnails/. Over time, this directory can grow large:

du -sh ~/.cache/thumbnails/

If it's large (hundreds of MB or more), clear it:

rm -rf ~/.cache/thumbnails/*

Thumbnails regenerate automatically as you browse files — no data is lost.
5. Check Disk Health

Install smartmontools for disk health monitoring:

sudo apt install smartmontools

Check disk health:

sudo smartctl -a /dev/sda

Replace /dev/sda with your disk identifier (from df output). Look for:

    SMART overall-health self-test: PASSED (good) or FAILED (replace immediately)
    Reallocated_Sector_Ct: Should be 0 — non-zero indicates failing sectors
    Temperature_Celsius: Should be under 50°C for HDDs, under 70°C for SSDs

    ⚠️ Disk Failure Is Serious

    If SMART reports anything other than PASSED, back up your important data immediately and replace the drive. Disk failures progress rapidly — don't wait.

System Uptime and Boot Information
uptime

uptime

Output:

 14:32:07 up  3:25,  2 users,  load average: 0.42, 0.58, 0.61

Tells you: current time, how long the system has been running since last boot, number of logged-in users, and load average. Quick health check in one line.
last — Boot and Login History

last reboot | head -10

Shows the last 10 reboots. Useful for understanding crash patterns (did the machine reboot unexpectedly?).

last username | head -10

Shows recent login sessions for a user.
Putting It Together: A Health Check Routine

Run this sequence monthly to keep your system healthy:

# 1. Check disk space
df -h

# 2. Check memory usage
free -h

# 3. Check load average and uptime
uptime

# 4. Find the 10 largest items in your home directory
du -sh ~/* | sort -rh | head -10

# 5. Check for failed services
systemctl --failed

# 6. Check recent system errors
journalctl -p err -b | tail -20

# 7. Update the system
sudo apt update && sudo apt upgrade

# 8. Clean up
sudo apt autoremove && sudo apt autoclean
sudo journalctl --vacuum-size=500M
LANG=C snap list --all | awk '/disabled/{print $1, $3}' | xargs -rn2 sudo snap remove

    💡 Failed Services

    systemctl --failed lists services that attempted to start but failed. This is a quick diagnostic — if nothing appears, all services are healthy. If something fails, investigate with systemctl status servicename and journalctl -u servicename.

What's Next

You now have the tools to monitor CPU, memory, disk, processes, services, and logs — both graphically through Resources and through terminal commands. You can kill misbehaving processes, clean up disk space, check service health, and read system logs to diagnose problems. You also have a monthly maintenance routine to keep everything running smoothly.

In Chapter 17, we'll cover users and groups — understanding the multi-user foundations of Linux, how sudo works in detail, and how to manage user accounts from the terminal.
Try It Yourself

    Monitor with Resources. Open the Resources app. Watch the CPU and memory graphs for a minute. Open Firefox or another app — watch the graphs respond.

    Run top. Press 1 to see individual cores. Press M to sort by memory. Press q to quit. Then install and try htop for comparison.

    Check disk space. Run df -h. Identify your root filesystem and how much space is available. Run du -sh ~/* | sort -rh | head -10 to find the largest items in your home directory.

    Check memory. Run free -h. Compare the used, buff/cache, and available columns. Understand the difference between "used" and "available."

    Check services. Run systemctl --failed. If anything appears, run systemctl status servicename to investigate. Run systemctl status NetworkManager to see a healthy service status.

    Read logs. Run journalctl -p err -b | tail -20. See if any errors have occurred during this boot session. Don't worry if you see some — many are non-critical warnings.

    Perform cleanup. Run sudo apt autoremove and sudo apt autoclean. Check how much space was recovered with df -h before and after.

    Run the health check routine. Execute the monthly health check sequence from this chapter. Note anything unusual for future reference.

    ✅ Chapter 16 Complete You can monitor CPU, memory, disk, and network activity through both graphical tools and terminal commands. You can find and kill processes, check service status, read system logs, perform routine disk cleanup, and run a comprehensive health check. In Chapter 17, we'll dive into user accounts, groups, and how sudo works.
