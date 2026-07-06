# Chapter 23: Process Management — Understanding and Controlling What Your System Does

## Everything Is a Process

When you open Firefox, you're starting a process. When your system boots, systemd launches dozens of processes. When a web server handles a request, it spawns processes to handle each connection. Even the terminal you're typing in is a process.

A **process** is a running program. A program is a file on disk (like `/usr/bin/firefox`). When you execute that file, the kernel loads it into memory, assigns it a unique ID, gives it CPU time, and tracks everything it does. That living, running instance is a process.

Understanding process management means understanding what your system is doing right now — which programs are consuming resources, how to stop misbehaving applications, how to prioritise important work, and how to keep things running when you close your terminal.

This chapter covers:
- How the kernel creates, schedules, and tracks processes
- Viewing processes with `ps`, `top`, `htop`, and `btop`
- Signals: how processes communicate termination and more
- Killing processes with `kill`, `killall`, and `pkill`
- Priority control with `nice` and `renice`
- Foreground and background jobs with `&`, `jobs`, `fg`, `bg`
- Process trees and parent-child relationships
- cgroups, systemd, and resource limits
- The `/proc` filesystem as a process window

---

## Table of Contents

| Topic | Section |
|---|---|
| What Is a Process? | 1 |
| The /proc Filesystem | 2 |
| ps: Process Snapshots | 3 |
| top and htop: Live Monitoring | 4 |
| Signals: Talking to Processes | 5 |
| Killing Processes | 6 |
| Process Priority: nice and renice | 7 |
| Foreground and Background Jobs | 8 |
| Process Trees and Parent-Child Relationships | 9 |
| cgroups and Resource Limits | 10 |

---

## 1. What Is a Process?

### Programs vs. Processes

| Concept | What It Is | Example |
|---|---|---|
| **Program** | A file on disk — static, inert code | `/usr/bin/firefox` (a 250KB binary) |
| **Process** | A running instance of a program — alive, consuming CPU and memory | PID 4567: firefox using 1.2GB RAM |
| **Thread** | A unit of execution within a process — shares memory with sibling threads | Firefox tab renderer thread |
| **Daemon** | A background process with no controlling terminal — runs continuously | `sshd`, `systemd-journald`, `cupsd` |

A single program can spawn many processes. Firefox opens multiple processes for tabs, GPU acceleration, and sandboxing. The Apache web server forks a process per connection. One program file, many process instances.

### Anatomy of a Process

Every process has:

| Attribute | Description | Example |
|---|---|---|
| **PID** | Unique Process ID number | 4567 |
| **PPID** | Parent Process ID (who spawned it) | 4543 |
| **UID/GID** | User and group running the process | 1000 (john) |
| **State** | What the process is currently doing | Running, Sleeping, Stopped, Zombie |
| **Priority/Nice** | CPU scheduling priority | Nice: 0 (default), -20 (highest), 19 (lowest) |
| **TTY** | Controlling terminal (if any) | pts/0, tty1, ? (none) |
| **Command** | The program being executed | `/usr/bin/firefox` |
| **Environment** | Environment variables inherited from parent | `HOME=/home/john`, `PATH=/usr/bin:...` |
| **File descriptors** | Open files, sockets, pipes | fd 0=stdin, fd 1=stdout, fd 2=stderr |

### Process States

The kernel tracks each process's state:

| State Letter | State Name | Meaning |
|---|---|---|
| `R` | Running | Actively using CPU (or in the run queue waiting) |
| `S` | Interruptible Sleep | Waiting for an event (I/O, timer, signal) — most processes spend most time here |
| `D` | Uninterruptible Sleep | Waiting for disk I/O — cannot be killed until I/O completes |
| `T` | Stopped | Suspended by a signal (SIGSTOP, Ctrl+Z) — can be resumed |
| `Z` | Zombie | Finished execution but parent hasn't read exit status — defunct |

Running (R) ↕ scheduled in/out Interruptible Sleep (S) ← most common — process waiting for input/timer/event ↕ Uninterruptible Sleep (D) ← stuck in kernel I/O — can't kill ↕ Stopped (T) ← suspended by SIGSTOP/Ctrl+Z ↕ Zombie (Z) ← dead but parent hasn't cleaned up

    💡 Why Do Zombies Exist?

    When a process finishes, the kernel keeps its exit status in the process table so the parent can read it. Until the parent calls wait() (or the parent dies and init adopts and cleans up the child), the process lingers as a zombie. Zombies consume no CPU or memory — just a slot in the process table. If zombies accumulate (hundreds), the process table fills up and new processes can't be created.

    To eliminate zombies: kill the parent process, or send it a signal that triggers cleanup.

The Life Cycle

1. fork()   — Parent process calls fork(), creating an exact copy of itself
2. exec()   — Child calls exec(), replacing its memory with a new program
3. run      — Child executes; CPU time allocated by scheduler
4. exit()   — Child finishes, sends SIGCHLD to parent, becomes zombie (Z)
5. wait()   — Parent reads exit status, zombie is reaped (removed)

When you type firefox in the terminal:

    Your shell (bash) calls fork() — creates a clone of itself
    The clone calls exec() — loads the Firefox binary, replacing bash code
    Firefox runs, the shell calls wait() and sleeps until Firefox exits
    Firefox calls exit() when you close it
    Your shell wakes up, reads the exit status, returns to the prompt

PID 1: The Ancestor

Every process traces its ancestry back to PID 1:

# What's PID 1?
ps -p 1
# PID TTY      TIME CMD
#   1 ?        00:00:02 systemd

# On Fedora 44, PID 1 is systemd

systemd is the first process the kernel starts at boot. It launches all other system services, which launch their own children. Every process on the system is a descendant of systemd.

When a process's parent dies, the process is orphaned and adopted by systemd (PID 1), which periodically reaps zombies.
2. The /proc Filesystem
/proc: The Kernel's Window to Userspace

/proc is a virtual filesystem. Its files don't exist on disk — they're generated by the kernel on demand. Every process gets a directory:

# List process directories in /proc:
ls -d /proc/[0-9]* | head -10
# /proc/1    /proc/2    /proc/3  ... /proc/4567  ...

# Each directory is named by PID:
ls /proc/4567/
# attr    cmdline    cwd    exe    fd    maps    status    environ
# fdinfo  io    loginuid    mem    root    statm    task    ...

# What command launched this process?
cat /proc/4567/cmdline | tr '\0' ' '
# /usr/lib/firefox/firefox

# Detailed status:
cat /proc/4567/status

Name:   firefox
Umask:  0022
State:  S (sleeping)
Tgid:   4567
Pid:    4567
PPid:   4543
Uid:    1000
Gid:    1000
Groups: 1000 10 18 135 ...
VmRSS:  1234567 kB     ← Resident memory (RAM actively used)
VmSize: 2345678 kB     ← Virtual memory (total address space)
Threads: 12             ← Number of threads
SigPnd: 0000000000000000
SigBlk: 0000000000000000
SigIgn: 0000000000001000
VoluntaryContextSwitches:   45678
NonvoluntaryContextSwitches: 2345

Useful /proc Entries

# What files has this process opened?
ls -la /proc/4567/fd/
# lrwx------  john  0 -> /dev/pts/0     ← stdin (terminal)
# lrwx------  john  1 -> /dev/pts/0     ← stdout
# lrwx------  john  2 -> /dev/pts/0     ← stderr
# lrwx------  john  3 -> socket:[12345]  ← network socket
# lrwx------  john  4 -> /tmp/profile.cache

# Where is the executable?
ls -la /proc/4567/exe
# /usr/bin/firefox

# What's the current working directory?
ls -la /proc/4567/cwd
# /home/john

# Environment variables:
cat /proc/4567/environ | tr '\0' '\n' | head -10
# HOME=/home/john
# PATH=/usr/local/bin:/usr/bin:...
# DISPLAY=:0
# LANG=en_US.UTF-8

# Memory map (what's loaded into memory):
cat /proc/4567/maps | head -10
# 00400000-00a12000 r-xp 00000000 fd:01  firefox
# 7f9abc123000-7f9abc456000 r--p 00000000  libpthread

# I/O statistics:
cat /proc/4567/io
# rchar: 123456789        ← bytes read (may be cached)
# wchar: 9876543          ← bytes written
# read_bytes: 4567890     ← actual disk reads
# write_bytes: 1234567    ← actual disk writes

System-wide /proc Entries

Not just per-process — /proc also holds system-wide information:

# CPU info:
cat /proc/cpuinfo | head -25

# Memory info:
cat /proc/meminfo | head -10
# MemTotal:       133681412 kB
# MemFree:        89123456 kB
# MemAvailable:   115234568 kB
# Buffers:         1234567 kB
# Cached:         30456789 kB

# Kernel version:
cat /proc/version
# Linux version 6.19.0-50.fc44.x86_64 ...

# Filesystems supported by the kernel:
cat /proc/filesystems

# Mounted filesystems:
cat /proc/mounts | head -5

# Load average (number of processes competing for CPU):
cat /proc/loadavg
# 0.42 0.38 0.31  2/1289  4521
#  ↑     ↑    ↑     ↑      ↑
# 1min  5min 15min  running/total  last PID

# System uptime:
cat /proc/uptime
# 86423.45 84234.12
# ↑          ↑
# uptime   idle time (seconds)

3. ps: Process Snapshots
What ps Does

ps provides a snapshot of running processes at a single moment. Unlike top (which updates continuously), ps captures the state and returns to your prompt.
The Two Syntax Families

ps has two historical implementations with different syntaxes — this confuses everyone initially:
Syntax	Origin	Example
BSD (no dashes)	Berkeley Unix	ps aux
System V / POSIX (with dashes)	AT&T Unix	ps -ef

Both work on Linux. Here's what they mean:

# BSD syntax — "all processes, user-oriented format":
ps aux
# a = all processes (not just yours)
# u = user-oriented format (shows CPU%, MEM%, user)
# x = include processes without a terminal (daemons)

# System V syntax — "every process, full format":
ps -ef
# -e = every process
# -f = full listing (shows PPID, C, STIME)

Reading ps aux Output

ps aux

USER         PID  %CPU  %MEM     VSZ    RSS  TTY   STAT  START  TIME  COMMAND
root           1   0.0   0.0  245632  12340  ?     Ss    Jun15  0:02  /usr/lib/systemd/systemd --system
root           2   0.0   0.0       0      0  ?     S     Jun15  0:00  [kthreadd]
root         456   0.0   0.1   25600   8192  ?     Ss    Jun15  0:01  /usr/lib/systemd/systemd-journald
root         789   0.0   0.0   18632   5120  ?     Ss    Jun15  0:00  /usr/sbin/sshd -D
john        4567   2.3  12.5 2345678 16789000 pts/0 Sl+   09:15  3:42  /usr/lib/firefox/firefox
john        4578   0.0   0.2  123456  32768 pts/0  R+    09:20  0:00  ps aux

Column meanings:
Column	Meaning
USER	Who owns the process
PID	Process ID
%CPU	CPU usage (percentage)
%MEM	Memory usage (percentage of total RAM)
VSZ	Virtual memory size in KiB (total address space, including swapped/shared)
RSS	Resident Set Size in KiB (actual physical memory used)
TTY	Terminal (pts/0 = pseudo-terminal; ? = no terminal/daemon)
STAT	Process state and flags
START	When the process started
TIME	Total CPU time consumed
COMMAND	The command (may be truncated)

STAT codes (can combine multiple letters):
Code	Meaning
R	Running
S	Interruptible sleep
D	Uninterruptible sleep (disk I/O)
T	Stopped (suspended)
Z	Zombie
+	In the foreground process group
s	Session leader (created the session)
l	Multi-threaded
<	High priority (negative nice)
N	Low priority (positive nice)
Useful ps Commands

# Processes for the current user:
ps -u $USER

# Processes for a specific user:
ps -u john

# Show process tree (parent-child relationships):
ps -ejH
# Indented tree view

# Alternative tree view:
pstree
# systemd─┬─ModemManager───2*[{ModemManager}]
#         ├─NetworkManager───2*[{NetworkManager}]
#         ├─sshd
#         └─firefox───12*[{firefox}]

# Custom column selection:
ps -eo pid,ppid,user,%cpu,%mem,stat,start,time,command

# Sort by CPU usage:
ps -eo pid,%cpu,%mem,comm --sort=-%cpu | head -10

# Sort by memory usage:
ps -eo pid,%cpu,%mem,comm --sort=-%mem | head -10

# Find a specific process by name:
ps -C firefox
# PID TTY      TIME CMD
# 4567 pts/0   00:00:03 firefox

# Find processes by partial name:
ps aux | grep -i fire

# Show all threads of a process:
ps -L -p 4567
# Shows each thread with its LWP (Light Weight Process) ID

# Full command line (no truncation):
ps -auxww
# ww = unlimited width (double-w for maximum width)

    💡 grep Trick to Exclude Itself

    When piping ps through grep, grep itself appears in the results:

    ps aux | grep firefox
    # john  4567  ...  /usr/lib/firefox/firefox
    # john  4589  ...  grep --color=auto firefox  ← THIS

    Exclude it with a trick — use a bracket in the pattern:

    ps aux | grep "[f]irefox"
    # john  4567  ...  /usr/lib/firefox/firefox
    # (grep doesn't match itself because "[f]irefox" ≠ "firefox" literally)

4. top and htop: Live Monitoring
top: The Classic Monitor

top shows a continuously updating view of system processes:

top

top - 14:23:45 up 2 days,  4:30,  2 users,  load average: 0.42, 0.38, 0.31
Tasks: 289 total,   1 running, 288 sleeping,   0 stopped,   0 zombie
%Cpu(s):  3.2 us,  1.1 sy,  0.0 ni, 95.5 id,  0.2 wa,  0.0 hi,  0.0 si,  0.0 st
MiB Mem:  130548.3 total,  87015.2 free,  12500.4 used,  31032.7 buff/cache
MiB Swap: 8192.0 total,   8192.0 free,      0.0 used. 112840.7 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
 4567 john      20   0 2287564 1.6g  98432 S   2.3  12.5   3:42.18 firefox
 7890 john      20   0  456789  23456  12345 S   1.5   0.2   0:15.32 gnome-shell
 1234 root      20   0  123456  32768  16384 S   0.3   0.0   0:01.12 systemd-journal
 5678 john      20   0  234567  12345   8192 R   0.0   0.0   0:00.01 top

Top section — system summary:
Line	What It Shows
Load average	1/5/15-minute averages — number of processes wanting CPU time
Tasks	Total/running/sleeping/stopped/zombie counts
%Cpu(s)	CPU breakdown (see below)
MiB Mem	Total/free/used/buff-cache memory
MiB Swap	Swap usage

CPU percentage breakdown:
Value	Meaning
us	User space (applications)
sy	System space (kernel)
ni	Nice (low-priority user processes)
id	Idle (nothing to do)
wa	I/O wait (waiting for disk/network)
hi	Hardware interrupts
si	Software interrupts
st	Stolen (by hypervisor, in VMs)

Bottom section — process list with these important columns:
Column	Meaning
PR	Priority (kernel scheduling priority, lower = higher priority)
NI	Nice value (-20 to 19, lower = higher priority)
VIRT	Virtual memory (total address space)
RES	Resident memory (actual RAM used)
SHR	Shared memory (libraries shared between processes)
S	State (R/S/D/T/Z)
top Interactive Commands
Key	Action
q	Quit
Space	Refresh immediately
P	Sort by CPU% (default)
M	Sort by memory%
N	Sort by PID
T	Sort by CPU time
1	Show per-core CPU breakdown
H	Show individual threads
k	Kill a process (prompts for PID + signal)
r	Renice a process (prompts for PID + nice value)
u	Filter by user
c	Toggle full command line
W	Write current configuration to ~/.toprc
htop: The Enhanced Monitor

htop provides a more user-friendly, colorful interface:

# Install:
sudo dnf install htop

# Run:
htop

    1  [                                                          0.0%]
    2  [||||                                                    ]  2.3%]
  ...
    16 [||                                                      ]  1.1%]
  Mem[||||||||||||||||||||||||                  12.5G/130G]
  Swp[                                            0K/8G]
  Tasks: 289, 2 thr; 1 running
  Load average: 0.42 0.38 0.31
  Uptime: 2 days, 04:30:12

  PID USER      PRI  NI  VIRT   RES   SHR S CPU%  MEM%   TIME+  Command
 4567 john       20   0 2.2G   1.6G  98M  S  2.3  12.5%  3:42.18 /usr/lib/firefox/firefox
 7890 john       20   0 456M  23M   12M   S  1.5   0.2%  0:15.32 gnome-shell

htop Advantages Over top
Feature	top	htop
Mouse support	No	Yes (click to sort, select, kill)
Color	No	Yes (intuitive CPU/memory bars)
Per-core CPU	Toggle with 1	Always visible as bars
Scroll	No	Yes (arrow keys)
Process tree	No	Yes (F5 toggles tree view)
Shortcut keys	Minimal	Full menu at bottom
htop Interactive Commands
Key	Action
F5 / t	Toggle tree view (parent-child hierarchy)
F6	Sort by column
F7 / [	Decrease nice value (higher priority)
F8 / ]	Increase nice value (lower priority)
F9 / k	Kill process
F10 / q	Quit
H	Toggle threads
u	Filter by user
/	Search by name
\	Filter by name
btop: The Modern Choice

For an even more visually rich experience:

sudo dnf install btop
btop

btop offers GPU monitoring, a beautiful TUI, and extensive configuration. It's purely optional but excellent for power users who want the most information-dense view available.
Choosing the Right Monitor
Tool	Best For
top	Available everywhere (no installation)
htop	Daily interactive monitoring
btop	Power users, GPU monitoring, visual flair
ps	Scripting, one-shot snapshots
5. Signals: Talking to Processes
What Signals Are

Signals are software interrupts — tiny messages the kernel delivers to a process. They interrupt whatever the process is doing and force it to respond.

Every signal has:

    A number (e.g., 15)
    A name (e.g., SIGTERM)
    A default action (e.g., terminate, ignore, dump core, stop)

The Most Important Signals
Number	Name	Default Action	When to Use
1	SIGHUP	Terminate	"Hang up" — traditionally meant modem disconnect. Daemons interpret it as "reload configuration"
2	SIGINT	Terminate	Sent by Ctrl+C — "interrupt this program"
3	SIGQUIT	Dump core	Sent by Ctrl+\ — like SIGINT but creates a core dump for debugging
9	SIGKILL	Terminate (uncatchable)	"Kill it NOW" — cannot be caught, blocked, or ignored
15	SIGTERM	Terminate	"Please shut down gracefully" — default for kill command
18	SIGCONT	Resume	Resume a stopped process
19	SIGSTOP	Stop (uncatchable)	Suspend a process — cannot be caught or ignored
20	SIGTSTP	Stop	Sent by Ctrl+Z — "suspend temporarily"

    ⚠️ SIGKILL Is the Nuclear Option

    SIGKILL (signal 9) is the only signal that cannot be caught or ignored. The kernel kills the process immediately — no cleanup, no saving files, no flushing buffers. Use it only when SIGTERM doesn't work.

    If a program is in the middle of writing a file and you send SIGKILL, the file may be corrupted. Always try SIGTERM first. Wait a few seconds. If the process doesn't exit, then escalate to SIGKILL.

Sending Signals with kill

# Default signal is SIGTERM (15) — graceful shutdown:
kill 4567
# Equivalent to: kill -15 4567
# Equivalent to: kill -TERM 4567

# Send SIGKILL (9) — force kill:
kill -9 4567
kill -KILL 4567

# Reload a daemon's configuration (send SIGHUP):
kill -HUP 789
kill -1 789

# List all available signals:
kill -l
# 1) SIGHUP   2) SIGINT   3) SIGQUIT   9) SIGKILL   15) SIGTERM   ...
# There are 62 signals total

# Send a signal to a process GROUP (all children too):
kill -- -4567
# Negative PID means "the process group whose ID is 4567"

Sending Signals from the Terminal
Keystroke	Signal Sent	Effect
Ctrl+C	SIGINT	Interrupt — terminate the foreground process
Ctrl+Z	SIGTSTP	Suspend — stop the foreground process (can resume)
Ctrl+\	SIGQUIT	Quit — terminate with core dump
Ctrl+D	(not a signal)	EOF — closes stdin, causing program to finish reading

# Example: Ctrl+Z suspends a process
sleep 300
# [1]+ Stopped    sleep 300
# Shell prompt returns immediately

# Resume it in the foreground:
fg
# sleep 300 resumes

# Or resume it in the background:
bg
# [1]+ sleep 300 &
# It continues running, shell prompt is free

6. Killing Processes
kill: Kill by PID

# Graceful termination (default):
kill 4567

# Force kill:
kill -9 4567

# Multiple PIDs:
kill 4567 4568 4569

# Verify the process is gone:
ps -p 4567
# (no output = terminated)

killall: Kill by Name

# Kill all processes named "firefox":
killall firefox

# Force kill all firefox processes:
killall -9 firefox

# Kill processes owned by a specific user:
killall -u john firefox

# Ask for confirmation before each kill:
killall -i firefox

# Kill by exact match (case-sensitive):
killall FIREFOX
# Won't match "firefox" — exact name match

# Kill interactively (prompt for each):
killall -i -u john chrome firefox

    ⚠️ killall Is Literal

    killall firefox kills EVERY process whose name is exactly "firefox." If you have 50 Firefox processes (tabs, renderers, GPU processes), they ALL die. This is usually what you want — but be cautious with generic names.

    On some commercial Unix systems, killall literally kills ALL processes the user can signal. The Linux version is safer (matches by name), but always verify what you're killing.

pkill: Kill by Pattern (Recommended)

pkill is more flexible than killall — it matches process names with regex patterns:

# Kill all processes matching "firefox" (partial match):
pkill firefox

# Kill with signal 9:
pkill -9 firefox

# Kill processes owned by a specific user:
pkill -u john firefox

# Kill processes NOT owned by a user:
pkill -u root firefox
# Be VERY careful — kills all root-owned firefox-like processes

# Match the full command line (not just process name):
pkill -f "firefox --new-tab"

# Case-insensitive match:
pkill -i FIREFOX

# Newest process only:
pkill -n firefox

# Oldest process only:
pkill -o firefox

# Show what would be killed without actually killing:
pkill -e firefox
# firefox (PID 4567) killed
# firefox (PID 4569) killed

# Use pgrep to preview what pkill would match:
pgrep firefox
# 4567
# 4569
# 4570

pkill vs. killall vs. kill
Feature	kill	killall	pkill
Identifies by	PID	Exact name	Pattern/regex
Flexibility	Lowest	Medium	Highest
Risk of killing wrong thing	None (specific PID)	Low	Medium (pattern may match unexpectedly)
Scripting	Manual PID needed	Good	Excellent
pgrep counterpart	No	No	Yes (pgrep previews matches)

Recommended workflow:

    pgrep firefox — See what matches
    pkill firefox — Kill if satisfied
    If still running: pkill -9 firefox

Killing Defiant Processes

# Process won't die after SIGKILL? Check its state:
ps -o pid,state,comm -p 4567
# 4567 D firefox
# D = Uninterruptible Sleep (stuck in kernel I/O)
# Cannot be killed until the I/O operation completes

# If a process is stuck in D state, it's waiting for the kernel
# to finish a disk operation. Options:
# - Wait (it may resolve when I/O completes)
# - Force unmount the filesystem it's stuck on
# - Reboot (last resort — D state processes cannot be killed)

7. Process Priority: nice and renice
How Scheduling Works

The Linux scheduler allocates CPU time slices to processes. Not all processes deserve equal treatment — your web browser should get more CPU than a background file indexer.

The nice value controls this:
Nice Value	Priority	Meaning
-20	Highest	Reserved for critical system tasks
0	Normal	Default — most processes run here
19	Lowest	"Be nice" — yield CPU to everything else

Only root can set negative nice values (higher priority than normal). Regular users can lower priority (increase nice value from 0 to 19) but cannot raise priority (decrease below 0).
Starting with nice

# Run a command with lower priority (nice = 10):
nice -n 10 ./compile_kernel.sh
# The compilation runs slower, yielding CPU to interactive apps

# Run with lowest priority:
nice -n 19 ./backup_script.sh

# As root, run with higher priority (negative nice):
sudo nice -n -10 ./real_time_process.sh

Changing Priority with renice

# Change a running process's priority:
renice -n 5 -p 4567
# PID 4567: old priority 0, new priority 5

# Renice a process by name (all matching):
sudo renice -n 10 -p $(pgrep firefox)

# Renice all processes owned by a user:
sudo renice -n 5 -u john

# Renice an entire process group:
sudo renice -n 5 -g 4567

Practical Scenarios

# You're compiling a large project but want responsive browsing:
nice -n 19 make -j$(nproc)
# Compilation uses spare CPU only; Firefox stays responsive

# A process is hogging CPU and you don't want to kill it:
renice -n 15 -p 4567
# Throttles it without stopping it

# Give a database high priority:
sudo renice -n -5 -p $(pgrep postgres)

    💡 nice Does NOT Limit CPU Percentage

    nice affects how the scheduler picks the next process to run — it doesn't cap CPU usage. A low-priority process with nothing else competing will still use 100% CPU. To actually limit CPU percentage (e.g., "never exceed 30%"), use cgroups (Section 10) or tools like cpulimit.

8. Foreground and Background Jobs
Job Control Basics

When you run a command in the terminal, it runs in the foreground — your prompt is blocked until the command finishes. Job control lets you push processes to the background and pull them back:

# Start a process in the background (&):
sleep 300 &
# [1] 4567
# [1] = job number, 4567 = PID

# List background jobs:
jobs
# [1]+ Running    sleep 300 &

# Bring it to the foreground:
fg %1
# sleep 300 now runs in foreground — prompt is blocked

# Suspend it (Ctrl+Z):
# ^Z
# [1]+ Stopped    sleep 300

# Resume in background:
bg %1
# [1]+ sleep 300 &

# Kill a background job:
kill %1
# [1]+ Terminated    sleep 300

Job Specifiers
Specifier	Meaning
%n	Job number n (from jobs output)
%% or %+	Current (last referenced) job
%-	Previous job
%name	Job whose command starts with "name"

# Multiple background jobs:
sleep 100 &     # Job [1]
sleep 200 &     # Job [2]
vim file.txt &  # Job [3] (suspended — vim needs a terminal)

jobs
# [1]   Running    sleep 100 &
# [2]-  Running    sleep 200 &
# [3]+  Stopped    vim file.txt

# The + means current job, - means previous job
fg %2     # Bring job 2 to foreground
bg %1     # Resume job 1 in background (already running, no effect)
kill %3   # Kill the suspended vim

nohup: Surviving Terminal Closure

When you close a terminal, the shell sends SIGHUP to all its background jobs, killing them. nohup (no hangup) prevents this:

# Run a command that survives terminal closure:
nohup ./long_running_backup.sh &
# [1] 4567
# nohup: ignoring input and appending output to 'nohup.out'

# Output goes to nohup.out in the current directory
# The process keeps running even after you close the terminal

disown: Detaching a Running Process

Forgot to use nohup? If a process is already running in the background, detach it from the shell:

# Start a background job normally:
./long_task.sh &
# [1] 4567

# Detach it (it won't be killed when terminal closes):
disown %1

# Verify it's detached:
jobs
# (empty — the job is no longer tracked by the shell)
# But ps confirms it's still running:
ps -p 4567

Keeping Long Tasks Running

Three approaches for persistent processes:

# Method 1: nohup (simplest, no output control):
nohup ./backup.sh &
# Output → nohup.out

# Method 2: Redirect output and background:
./backup.sh > /tmp/backup.log 2>&1 &
disown

# Method 3: Use tmux or screen (keeps a full terminal session):
sudo dnf install tmux
tmux
# Run your command inside tmux
# Detach: Ctrl+B then D
# Reattach later: tmux attach
# The process keeps running in the tmux session indefinitely

    💡 tmux vs. nohup

    nohup works for non-interactive scripts. But if your long-running task is interactive (like a software installer that prompts for input), use tmux. tmux keeps a full virtual terminal running on the server — you can detach, close your SSH session, reconnect, and reattach to the same terminal later.

9. Process Trees and Parent-Child Relationships
Understanding Process Trees

Every process (except PID 1) has a parent. This creates a tree:

# Visualize the full process tree:
pstree

systemd─┬─ModemManager───2*[{ModemManager}]
        ├─NetworkManager───2*[{NetworkManager}]
        ├─accounts-daemon───2*[{accounts-daemon}]
        ├─cron
        ├─dbus-broker
        ├─firewalld───4*[{firewalld}]
        ├─fwupd───2*[{fwupd}]
        ├─gnome-shell───8*[{gnome-shell}]
        ├─gssproxy───5*[{gssproxy}]
        ├─login───bash
        ├─pipewire───2*[{pipewire}]
        ├─polkitd───2*[{polkitd}]
        ├─sshd───sshd───sshd───bash───pstree
        ├─systemd-journald
        ├─systemd-logind
        ├─systemd-udevd
        └─wireplumber───2*[{wireplumber}]

The ───2*[{ModemManager}] notation means the process has 2 threads (shown in curly braces).
Examining Parent-Child Relationships

# Show a specific process and its ancestors:
ps -o pid,ppid,comm -p 4567
#  PID  PPID COMMAND
# 4567  4543 firefox
# Firefox's parent is PID 4543

# Follow the parent chain upward:
ps -o pid,ppid,comm -p 4543
#  PID  PPID COMMAND
# 4543  4521 gnome-terminal-server
# Grandparent is the terminal

# Keep going:
ps -o pid,ppid,comm -p 4521
#  PID  PPID COMMAND
# 4521     1 systemd
# Great-grandparent is systemd (PID 1)

# Or trace in one shot:
pstree -p -s 4567
# systemd(1)───gnome-terminal-(4521)───bash(4543)───firefox(4567)

Why Parent-Child Matters
Scenario	Why It Matters
Killing a parent	Children may become orphaned (adopted by systemd) or killed (if in the same process group)
Debugging a crash	Finding the parent that spawned the crashed process
Security audit	Unexpected parent-child relationships may indicate compromise
Graceful shutdown	Killing the parent daemon often cascades to children
Resource tracking	A parent and all its descendants form a cgroup for resource accounting

# Show all children of a process:
pgrep -P 4521
# 4543
# 4544
# Lists PIDs of all direct children of PID 4521

# Show the full subtree:
pstree -p 4521
# gnome-terminal-server(4521)───bash(4543)───firefox(4567)───15*[{firefox}]

# Kill a process and all its children:
kill -- -4521
# Negative PID = kill the entire process group
# Every member of process group 4521 receives the signal

Session Leaders and Process Groups
Concept	Definition
Session	A collection of process groups. Typically one per login.
Process group	A set of processes that receive signals together (e.g., from a terminal).
Session leader	The process that created the session (usually the login shell).

# Show session/process group info:
ps -eo pid,ppid,pgid,sid,comm | head -10
# PID   PPID  PGID   SID  COMMAND
#   1      0     1     1  systemd
# 4521     1  4521  4521  gnome-terminal-server
# 4543  4521  4543  4543  bash               ← session leader
# 4567  4543  4567  4543  firefox            ← same session, new process group

When you press Ctrl+C in a terminal, the terminal sends SIGINT to the entire foreground process group — not just the foreground process. This is why Ctrl+C kills a command and all its children.
10. cgroups and Resource Limits
What Are cgroups?

cgroups (control groups) are a kernel feature for limiting, accounting, and isolating resource usage of process groups. While nice affects scheduling priority, cgroups enforce hard limits:
Resource	What cgroups Can Limit
CPU	Maximum CPU percentage or bandwidth
Memory	Maximum RAM; trigger OOM killer if exceeded
I/O	Disk read/write bandwidth
Network	Network traffic priority (via tc)
PIDs	Maximum number of processes in the group
cgroups v2 on Fedora 44

Fedora 44 uses cgroups v2 (unified hierarchy), managed entirely through systemd:

# View the cgroup hierarchy:
systemd-cgls
# Control group v2:
# -.scope
# ├─init.scope
# │ └─1 /usr/lib/systemd/systemd --system
# ├─system.slice
# │ ├─NetworkManager.service
# │ │ └─789 /usr/sbin/NetworkManager --no-daemon
# │ ├─sshd.service
# │ │ └─1234 /usr/sbin/sshd -D
# │ ├─firewalld.service
# │ │ └─456 /usr/bin/firewalld
# │ └─...
# └─user.slice
#   └─user-1000.slice
#     ├─session-1.scope
#     │ └─4543 bash
#     │   └─4567 firefox
#     └─user@1000.service
#       └─gnome-shell

Every process belongs to a cgroup. Systemd organizes them into slices:
Slice	Contains
system.slice	System services (sshd, NetworkManager, etc.)
user.slice	User sessions and processes
machine.slice	Containers and VMs
-.slice	Root (contains everything)
Viewing Resource Usage by cgroup

# CPU usage per service:
systemd-cgtop
# Control Group           Tasks   %CPU   Memory    Input/s   Output/s
# /                         289   3.5%   12.5G     0B/s      0B/s
# /user.slice/user-1000...  45   2.8%    8.2G     0B/s      0B/s
# /system.slice             120   0.7%    1.2G     0B/s      0B/s
# /system.slice/firewalld... 4   0.1%   45M       0B/s      0B/s

Setting Resource Limits with systemd-run

# Run a command with a CPU limit (50% of one core):
systemd-run --user --scope -p CPUQuota=50% ./cpu_intensive_task.sh

# Run with a memory limit (500MB):
systemd-run --user --scope -p MemoryMax=500M ./memory_hungry.sh

# Run with both CPU and memory limits:
systemd-run --user --scope \
  -p CPUQuota=25% \
  -p MemoryMax=200M \
  ./limited_task.sh

# Run with a process count limit:
systemd-run --user --scope -p TasksMax=10 ./forking_script.sh

Configuring Service Limits

For systemd services, set limits in the unit file:

# Edit a service's resource limits:
sudo systemctl edit httpd.service

# Override file for httpd.service:
[Service]
CPUQuota=200%        # Allow up to 2 full cores
MemoryMax=2G          # Cap at 2GB RAM
IOWeight=500          # I/O priority (1-10000, default 100)
TasksMax=256          # Max processes/threads

# Apply:
sudo systemctl daemon-reload
sudo systemctl restart httpd.service

# Verify limits are in effect:
systemctl show httpd.service -p CPUQuotaPerSecUSec -p MemoryMax

ulimit: Per-Shell Resource Limits

Before cgroups existed, ulimit provided resource limits. It's still useful for shell sessions:

# View current limits:
ulimit -a
# core file size          (blocks, -c) 0
# data seg size           (kbytes, -d) unlimited
# file size               (blocks, -f) unlimited
# max memory size         (kbytes, -m) unlimited
# open files                      (-n) 1024
# pipe size            (512 bytes, -p) 8
# stack size              (kbytes, -s) 8192
# cpu time               (seconds, -t) unlimited
# max user processes              (-u) 4096
# virtual memory          (kbytes, -v) unlimited

# Change open file limit for this shell session:
ulimit -n 65536

# Limit CPU time to 10 minutes (kills process after 600 seconds):
ulimit -t 600

# These are per-session — add to ~/.bashrc for permanence

Common ulimits that matter:
Limit	Flag	Default	When to Increase
Open files	-n	1024	Database servers, web servers under load
User processes	-u	4096	Development machines, forking apps
Stack size	-s	8192 KB	Deep recursion, large stack frames

# Permanently set a limit system-wide:
sudo nano /etc/security/limits.conf
# Add:
# *    soft    nofile    65536
# *    hard    nofile    1048576
# john soft    nproc     8192
# john hard    nproc     16384

Try It Yourself
Exercise 1: Process Survey

# What's running on your system?
ps aux --sort=-%cpu | head -15    # Top 15 by CPU
ps aux --sort=-%mem | head -15    # Top 15 by memory

# View the process tree:
pstree | head -30

# Check your own processes:
ps -u $USER

# Find your shell:
echo "My shell PID is $$"
ps -p $$ -o pid,ppid,comm,args

Exercise 2: /proc Exploration

# Find your shell's PID:
echo $$

# Explore its /proc entry:
PID=$$
cat /proc/$PID/status | head -10
cat /proc/$PID/cmdline | tr '\0' ' '; echo
ls -la /proc/$PID/fd/
cat /proc/$PID/environ | tr '\0' '\n' | head -10

# Check system-wide info:
cat /proc/loadavg
cat /proc/uptime
cat /proc/meminfo | head -5

Exercise 3: Job Control

# Start three background jobs:
sleep 100 &
sleep 200 &
sleep 300 &

# List them:
jobs

# Bring job 2 to the foreground:
fg %2
# Ctrl+Z to suspend it
# ^Z

# Resume in background:
bg %2

# Kill all three:
kill %1 %2 %3

# Verify:
jobs

Exercise 4: Signals

# Start a sleep process:
sleep 500 &
echo "Started with PID $!"

# Send SIGTERM:
kill $!

# If it didn't die (unlikely for sleep), escalate:
# kill -9 $!

# Test Ctrl+Z and Ctrl+C:
sleep 300
# Press Ctrl+Z
# ^Z
# [1]+ Stopped    sleep 300

# Resume:
fg
# Press Ctrl+C
# ^C

Exercise 5: Priority Management

# Start a CPU-consuming process in the background:
yes > /dev/null &
# 'yes' outputs "y" endlessly — uses 100% CPU

# Check its priority:
ps -o pid,ni,comm -p $!

# Lower its priority:
renice -n 10 -p $!

# Verify:
ps -o pid,ni,comm -p $!

# Kill it:
kill $!

# Or use top:
top
# Press 'r', enter the PID of 'yes', enter nice value 10
# Watch its CPU% drop as other processes take priority

Exercise 6: Killing by Name

# Start multiple background processes:
sleep 100 &
sleep 200 &
sleep 300 &
sleep 400 &

# Preview what pkill would match:
pgrep sleep

# Kill them all:
pkill sleep

# Verify:
jobs
# All should be terminated

Exercise 7: cgroup Inspection

# View the cgroup hierarchy:
systemd-cgls | head -40

# View top resource consumers:
systemd-cgtop
# (Press Ctrl+C after viewing)

# Find which cgroup a process belongs to:
cat /proc/$$/cgroup
# 0::/user.slice/user-1000.slice/session-1.scope

# View CPU quota for a service:
systemctl show sshd.service -p CPUQuotaPerSecUSec

Exercise 8: nohup and disown

# Start a background task with nohup:
nohup sleep 120 &
# Output: appending to 'nohup.out'

# Verify it will survive terminal close:
ps -o pid,ppid,comm -p $!

# Detach an already-running background job:
sleep 60 &
disown %1

# Verify it's still running but no longer a job:
jobs          # Should show nothing or other jobs
ps -o pid,comm -p $(pgrep -n sleep)

Troubleshooting Reference
Symptom	Likely Cause	First Check
System feels sluggish	High CPU process	top or htop — look at %CPU column
Out of memory	Process consuming too much RAM	ps aux --sort=-%mem | head, check OOM killer: dmesg | grep -i oom
Process won't die	Stuck in D (uninterruptible) state	ps -o pid,state -p PID — D state can't be killed
Too many processes	Fork bomb or runaway spawning	ps aux | wc -l, check Tasks: in top
"Resource temporarily unavailable"	Process limit (ulimit/pid.max) hit	ulimit -u, cat /proc/sys/kernel/pid_max
Defunct/zombie processes	Parent not reaping children	Find parent: ps -o ppid= -p <zombie_pid>, kill or signal parent
Job killed when terminal closed	Didn't use nohup/disown	Use nohup or disown for persistent processes
Service using too much CPU	Missing resource limits	systemctl edit <service> → add CPUQuota=
Service using too much memory	Missing memory limit	systemctl edit <service> → add MemoryMax=
High load average but low CPU%	I/O bottleneck	Check wa (I/O wait) in top, use iostat
Can't kill process as regular user	Not your process	Check ps -o user -p PID, use sudo kill
Firefox frozen	Unresponsive process	Try kill <PID>, wait 5s, then kill -9 <PID>
Background job disappeared	Lost when shell exited	Check if orphaned: ps -eo pid,ppid,comm | grep <name>
What's Next

You now understand process management on Fedora 44: the process lifecycle (fork/exec/wait model, PID 1 as systemd ancestor, process states R/S/D/T/Z with state transitions, orphaned processes adopted by init, zombie cleanup via parent wait()), the /proc filesystem (per-process directories /proc/PID/ with cmdline/status/fd/environ/maps/io files, system-wide entries for cpuinfo/meminfo/loadavg/uptime/mounts, VmRSS vs VmSize distinction), ps command (BSD syntax ps aux vs System V ps -ef, column meanings USER/PID/%CPU/%MEM/VSZ/RSS/TTY/STAT/START/TIME/COMMAND, STAT state codes with modifiers +/s/l/</N, sorting with --sort, process trees with pstree, custom column selection with -eo, the [f]irefox grep trick, thread listing with -L), top (system summary with load average/tasks/CPU breakdown us/sy/ni/id/wa/hi/si/st/memory stats, process list with PR/NI/VIRT/RES/SHR columns, interactive commands P/M/N/T/k/r/1/H/c/u/W), htop (enhanced features, per-core bars, tree view with F5, mouse support, F7-F8 niceness, F9 kill), btop (modern alternative with GPU monitoring), signals (SIGHUP/SIGINT/SIGQUIT/SIGKILL/SIGTERM/SIGCONT/SIGSTOP/SIGTSTP with numbers/names/default actions, SIGKILL as uncatchable nuclear option, Ctrl+C/Ctrl+Z/Ctrl+\ terminal shortcuts, kill -l listing, kill -HUP for daemon reloads), killing processes (kill by PID, killall by exact name with -i/-u flags, pkill by pattern with -f/-i/-n/-o/-e flags, pgrep for previewing, three-tool comparison, defiant D-state processes, recommended pgrep→pkill→pkill -9 workflow), process priority (nice values -20 to 19, root-only negative values, nice -n to start, renice -n to change running processes, by PID/user/group, nice affects scheduling not CPU cap), foreground/background jobs (& background operator, jobs listing, fg/bg %n control, %1/%%/%-/%name specifiers, Ctrl+Z suspension, nohup for surviving terminal close, disown for detaching running processes, tmux for persistent sessions), process trees (parent-child via PPID, pstree visualization with -p/-s flags, pgrep -P for children, process groups and sessions, negative PID kill for entire groups, why Ctrl+C kills process group not just process), cgroups and resource limits (cgroups v2 unified hierarchy, systemd slices system/user/machine, systemd-cgls hierarchy viewer, systemd-cgtop resource monitor, systemd-run with CPUQuota/MemoryMax/TasksMax for ad-hoc limits, systemctl edit for service limits, ulimit -a/-n/-u/-s for per-shell limits, /etc/security/limits.conf for permanent limits, cgroups vs nice distinction: hard limits vs scheduling priority), progressive exercises from process survey through /proc exploration, job control, signals, priority management, killing by name, cgroup inspection, and nohup/disown, plus a comprehensive twelve-entry troubleshooting reference table.

Chapter 24 covers system monitoring and maintenance — keeping your Fedora 44 system healthy over time. You'll learn about disk usage and cleanup (du, df, ncdu), log management with journalctl, system updates and rollback with Btrfs snapshots, service management with systemctl, timed tasks with cron and systemd timers, and performance profiling. These are the skills that turn a fresh install into a system that runs reliably for years.

Until then, watch your system work. Open htop and browse the web, compile code, or play music. See how processes spawn, consume resources, and sleep. The more familiar "normal" looks, the faster you'll spot abnormal.

    ✅ Chapter 23 Complete You understand processes (program vs process vs thread vs daemon, anatomy with PID/PPID/UID/state/priority/TTY/command/environment/file-descriptors, states R/S/D/T/Z with transitions, zombie lifecycle and cleanup, fork/exec/wait model, PID 1 as systemd ancestor, orphan adoption), the /proc filesystem (per-process directories /proc/PID/ with cmdline/status/fd/environ/maps/io/cwd/exe entries, system-wide entries cpuinfo/meminfo/loadavg/uptime/mounts/version/filesystems, VmRSS vs VmSize distinction, I/O statistics, virtual filesystem generated on-demand by kernel), ps (BSD ps aux vs System V ps -ef syntax families, column meanings USER/PID/%CPU/%MEM/VSZ/RSS/TTY/STAT/START/TIME/COMMAND, STAT codes R/S/D/T/Z with modifiers +/s/l/</N, --sort flag, pstree visualization, -eo custom columns, -C by command name, -L for threads, [f]irefox grep trick, pgrep for previewing), top (system summary: load average, Tasks count, %Cpu breakdown us/sy/ni/id/wa/hi/si/st, MiB Mem/Swap with buff/cache and avail; process list: PR/NI/VIRT/RES/SHR/S; interactive keys P/M/N/T/1/H/k/r/c/u/q/W), htop (color bars, per-core visualization, F5 tree view, F6 sort, F7/F8 renice, F9 kill, mouse support, u user filter, / search, \ filter), btop (modern GPU-aware alternative), signals (number/name/default action for SIGHUP/SIGINT/SIGQUIT/SIGKILL/SIGTERM/SIGCONT/SIGSTOP/SIGTSTP, SIGKILL as uncatchable nuclear option with corruption risk, Ctrl+C=SIGINT/Ctrl+Z=SIGTSTP/Ctrl+=SIGQUIT/Ctrl+D=EOF, kill -l listing, kill -HUP for daemon reloads, negative PID for process groups), killing processes (kill by PID with default SIGTERM, kill -9 escalation, killall by exact name with -i/-u/-9 flags and literal-match caution, pkill by regex pattern with -f/-i/-n/-o/-u/-e flags, pgrep for safe previewing, recommended pgrep→pkill→pkill -9 workflow, D-state defiant processes unable to be killed), nice and renice (nice values -20 highest to 19 lowest, root-only negative values, nice -n to start with priority, renice -n to modify running processes by PID/user/group, nice affects scheduler selection not CPU percentage cap, practical scenarios for compilation/database/web browsing), foreground/background jobs (& operator, jobs list, fg %n/bg %n, Ctrl+Z suspension, job specifiers %1/%%/%+/%-/%name, nohup for terminal-independent survival with output to nohup.out, disown for detaching already-running background jobs, tmux recommendation for interactive persistent sessions), process trees (parent-child via PPID chain, pstree -p/-s flags, pgrep -P for direct children, negative PID kill for entire process group, sessions and session leaders, foreground process group receives terminal signals, security and debugging implications), cgroups and resource limits (cgroups v2 unified hierarchy on Fedora 44, systemd slices system.slice/user.slice/machine.slice, systemd-cgls hierarchy viewer, systemd-cgtop resource monitor with CPU/memory/I/O per group, systemd-run --scope with CPUQuota/MemoryMax/TasksMax for ad-hoc resource limiting, systemctl edit for persistent service limits with daemon-reload, ulimit -a/-n/-u/-s for per-shell limits, /etc/security/limits.conf for permanent system-wide limits, cgroups as hard limits vs nice as scheduling priority), progressive exercises from process survey through /proc exploration, job control, signals, priority management, killing by name, cgroup inspection, and nohup/disown, plus a comprehensive twelve-entry troubleshooting reference table.

