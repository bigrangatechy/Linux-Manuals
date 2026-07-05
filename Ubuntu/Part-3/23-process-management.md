# Chapter 23: Process Management

## Every Running Program Is a Process

When you open Firefox, launch a terminal, or run a script, the system creates a **process** — a running instance of a program. Each process gets a unique Process ID (PID), consumes CPU time and memory, and runs until it finishes or is killed.

In Chapter 16, you learned to monitor processes with `top`, `htop`, and `ps`, and to kill them with `kill` and `pkill`. This chapter goes deeper: controlling foreground and background jobs, sending processes specific signals, making processes survive terminal disconnects with `nohup` and `disown`, adjusting process priority with `nice` and `renice`, visualizing the process tree with `pstree`, and understanding how systemd manages long-running services through cgroups.

## The Process Tree

Processes on Linux form a **hierarchy** — a tree structure. Every process has a parent (except the first one). Understanding this hierarchy helps you understand why processes behave the way they do.

### PID 1: systemd

When Linux boots, the first process started is **systemd** (PID 1). It's the ancestor of every other process on the system. systemd starts system services, manages login sessions, and oversees the entire process tree.

### Parent and Child Processes

When a process creates another process, the creator is the **parent** and the new process is the **child.** The child inherits the parent's environment variables, file descriptors, and permissions.

systemd (PID 1) ├── NetworkManager (PID 890) ├── gnome-shell (PID 1200) │ ├── Ptyxis (PID 2500) │ │ └── bash (PID 2501) │ │ └── firefox (PID 3200) │ └── nautilus (PID 1300) ├── snapd (PID 950) └── sshd (PID 880) └── sshd (PID 2100) ← your SSH session └── bash (PID 2101)

This tree shows systemd spawning services, GNOME Shell spawning the terminal, the terminal spawning bash, and bash spawning Firefox. Every process traces back to PID 1.

Viewing the process tree:

pstree

Output:

systemd─┬─NetworkManager───2*[{NetworkManager}]
        ├─accounts-daemon───{accounts-daemon}
        ├─cron
        ├─dbus-daemon
        ├─gnome-shell───19*[{gnome-shell}]
        ├─gnome-terminal-server─┬─bash───firefox───28*[{firefox}]
        │                       └─bash───pstree
        ├─snapd───8*[{snapd}]
        └─sshd───sshd───bash

The ───N*[{name}] notation means "N threads belonging to this process." Firefox, for example, uses many threads for rendering, networking, and JavaScript.

Show PIDs alongside the tree:

pstree -p

Show a specific process and its children:

pstree -p 1200

Shows the subtree rooted at PID 1200 (e.g., GNOME Shell and everything under it).
Process States

Every process is in one of several states:
State Code	State Name	Meaning
R	Running	Currently executing or waiting for CPU time
S	Sleeping	Waiting for an event (input, timer, I/O) — most processes are here
D	Uninterruptible Sleep	Waiting for disk I/O — cannot be killed (even with kill -9) until the I/O completes
Z	Zombie	Finished but parent hasn't collected its exit code — "living dead"
T	Stopped	Suspended (paused by Ctrl+Z or a stop signal)

You see these codes in the STAT column of ps aux:

ps aux | grep firefox

john  3200  2.3  8.1 891234 23456 ?  Sl  14:30  0:05 firefox

The Sl means: S (sleeping) + l (multi-threaded). A process showing R is actively running or queued for CPU.
Zombies Explained

A zombie process is one that has finished executing but whose parent hasn't acknowledged its termination. The process's entry in the process table remains until the parent reads the exit code.

Zombies are harmless — they consume no CPU or RAM, just a process table slot. They usually disappear when the parent process cleans up. If zombies accumulate, it indicates a buggy parent process.

Find zombies:

ps aux | awk '$8 ~ /Z/'

Or:

ps -el | grep Z

To remove a zombie, you can't kill it (it's already dead). Instead, kill or restart its parent process.
Foreground and Background Jobs

When you type a command in the terminal, it runs in the foreground — the terminal is occupied until the command finishes. For long-running commands, this blocks you from doing anything else.
Running a Command in the Background

Append & to run a command in the background:

sleep 60 &

Output:

[1] 5423

    [1] — job number (assigned by the shell)
    5423 — process ID

The terminal is immediately available for more commands. The sleep 60 command runs in the background, and after 60 seconds, you'll see:

[1]+  Done                    sleep 60

The jobs Command

jobs

Lists all background jobs associated with your current terminal session:

[1]-  Running                 sleep 100 &
[2]+  Running                 sleep 200 &

Marker	Meaning
+	The current (most recently used) job — used by default with fg and bg
-	The previous job
Number	Job ID (use with fg %1, bg %2, kill %1)
Suspending a Running Process: Ctrl+Z

Press Ctrl+Z while a foreground process is running:

sleep 300
^Z
[1]+  Stopped                  sleep 300

The process is stopped (paused, not killed). It's now in state T. The terminal is available again.
Resuming in the Background: bg

After suspending with Ctrl+Z, resume the job in the background:

bg

Or specify a job number:

bg %1

The process continues running, but the terminal is free.
Bringing to the Foreground: fg

Bring a background job back to the foreground:

fg

Brings the current job (+) to the foreground. Or specify by job number:

fg %2

Full Workflow Example

# Start a long-running task
$ sleep 300
# Oops — I need the terminal. Suspend it.
^Z
[1]+  Stopped                  sleep 300

# Resume in background
$ bg
[1]+ sleep 300 &

# Do other work
$ ls /var/log

# Check on our job
$ jobs
[1]+  Running                 sleep 300 &

# Bring it back to foreground
$ fg
sleep 300

# Wait for it to finish, or Ctrl+C to kill it

Killing Jobs by Job ID

kill %1

Sends SIGTERM to job 1 (the polite "please stop" signal). Same as kill PID but uses the job number instead.

kill -9 %1

Force kill job 1 immediately.

    💡 Job Numbers vs. PIDs

    Job numbers (%1, %2) are local to your terminal session — they're assigned by the shell. PIDs are system-wide. Both work with kill, but job numbers are only valid in the shell that started the job.

        kill %1 — kills job 1 in THIS terminal
        kill 5423 — kills process 5423 system-wide

Signals: Communicating with Processes

Signals are the mechanism Linux uses to communicate with processes. When you press Ctrl+C, the terminal sends a signal. When you run kill, you send a signal. Understanding signals gives you precise control over process behavior.
Common Signals
Signal	Number	Default Action	When to Use
SIGTERM	15	Terminate	Default for kill — polite shutdown
SIGKILL	9	Kill immediately	When SIGTERM doesn't work — last resort
SIGINT	2	Interrupt	Sent by Ctrl+C — interrupt a foreground process
SIGHUP	1	Hang up	Terminal closed — or reload config for daemons
SIGSTOP	19	Stop (pause)	Sent by Ctrl+Z — pause a process
SIGCONT	18	Continue	Resume a stopped process
SIGUSR1	10	User-defined	Application-specific
SIGSEGV	11	Segmentation fault	Program crashed (memory violation)
SIGALRM	14	Alarm timer	Timed expiration
Sending Signals

Default (SIGTERM):

kill 5423

Specific signal by name:

kill -SIGKILL 5423
kill -SIGHUP 5423

Specific signal by number:

kill -9 5423
kill -1 5423

Kill by name with pkill:

pkill firefox
pkill -9 firefox

Kill by exact name with killall:

killall firefox

Send SIGHUP to reload configuration (not kill):

sudo systemctl reload nginx

or

sudo kill -HUP $(cat /var/run/nginx.pid)

Many daemons (nginx, Apache, sshd) interpret SIGHUP as "reload your configuration file" rather than "shut down."
Signal Behavior Summary
Signal	Catchable?	Cleanup?	Notes
SIGTERM	Yes	Yes	Process can intercept, clean up, then exit
SIGKILL	No	No	Cannot be caught — immediate termination
SIGINT	Yes	Yes	Like SIGTERM but specifically from keyboard interrupt
SIGHUP	Yes	Yes	Often repurposed as "reload"
SIGSTOP	No	N/A	Pauses process — cannot be caught
SIGCONT	No	N/A	Resumes a paused process

The key distinction: SIGKILL and SIGSTOP cannot be caught or ignored by a process. They're enforced by the kernel. Everything else can be intercepted by the program's code, allowing it to save files, close connections, or even ignore the signal entirely.

    💡 Why Not Always Use kill -9?

    kill -9 (SIGKILL) is tempting because it always works — no process can resist it. But it's destructive:

        The process can't save unsaved data or close files properly
        Database connections may be left hanging
        Temporary files may not be cleaned up
        Child processes may be orphaned

    Always try kill (SIGTERM) first. Wait a few seconds. If the process is still running, then escalate to kill -9.

Keeping Processes Alive After Logout: nohup and disown

When you close a terminal or disconnect an SSH session, the shell sends SIGHUP (Signal Hang UP) to all its child processes. This tells them "the terminal is gone, please exit." Most processes comply and terminate.

But sometimes you start a long-running task — a large download, a backup, a database migration — and you need it to continue after you log out.
nohup — Ignore Hang-Up Signals

nohup (No HUP) runs a command that ignores SIGHUP:

nohup ./long-script.sh &

Output:

nohup: ignoring input and appending output to 'nohup.out'
[1] 6543

    nohup makes the process ignore SIGHUP — it won't die when you log out
    Since the terminal is going away, output is redirected to nohup.out in the current directory
    The & puts it in the background so you get your prompt back

Redirecting output explicitly:

nohup ./long-script.sh > /var/log/mytask.log 2>&1 &

    > /var/log/mytask.log — redirect stdout to a log file
    2>&1 — redirect stderr to the same place as stdout
    & — background

This is the standard pattern for running long tasks over SSH: all output goes to a file you can inspect later, and the process survives disconnection.

Verifying the process is still running after re-login:

ps aux | grep long-script

disown — Detaching a Running Job

Sometimes you start a process without nohup, then realize it's going to take hours and you need to close the terminal. disown detaches an existing background job from the shell:

# Started a long task in background
$ ./big-download.sh &
[1] 7890

# Oops, I need to leave. Detach it from this shell.
$ disown %1

After disown, the process is no longer in the shell's job table. When you close the terminal, SIGHUP won't be sent to it. The process keeps running.

Disown all jobs:

disown -a

Disown and keep running:

disown -h %1

The -h flag marks the job to survive SIGHUP but keeps it in the job table (so you can still see it with jobs).

    💡 nohup vs. disown — When to Use Each
    Tool	When Applied	How
    nohup	Before starting the process	nohup command &
    disown	After starting the process	command & then disown

    If you know in advance you'll need the process to survive, use nohup. If you forgot and need to rescue a running process, use disown. Both achieve the same end result — a process that survives terminal closure.

Process Priority with nice and renice

Linux prioritizes processes using a nice value ranging from -20 (highest priority) to 19 (lowest priority). The default is 0.
Nice Value	Priority	Who Can Set It
-20	Maximum priority	Root only
0	Default	Any user
19	Minimum priority	Any user
Starting a Process with nice

nice -n 10 ./render.sh

Starts render.sh with a nice value of 10 — it gets less CPU time than normal-priority processes. Useful for background tasks that shouldn't interfere with interactive work.

sudo nice -n -5 ./critical-task.sh

Starts with priority higher than normal (requires root). Use sparingly — high-priority processes can starve other processes of CPU time.
Changing Priority of a Running Process with renice

renice -n 5 -p 7890

Changes the nice value of PID 7890 to 5 (lower priority).

sudo renice -n -10 -p 7890

Raises priority (requires root).

Renice by user:

sudo renice -n 10 -u john

Changes the priority of all processes owned by user john.

Renice by process group:

renice -n 5 -g 7890

Practical Use Case

You're rendering a video overnight but want it to not impact system responsiveness during the day:

# Start the render with low priority
nice -n 19 ffmpeg -i input.mp4 -c:v libx264 output.mp4 &

# If you need the CPU urgently, lower its priority further
renice -n 19 -p $!

The $! variable holds the PID of the last backgrounded process.
Viewing Processes in Detail
ps Revisited

From Chapter 16, you know ps aux for listing all processes. Here are additional patterns:

Process tree format:

ps -ef --forest

Shows processes in a tree hierarchy, similar to pstree but with more detail.

Custom columns:

ps -eo pid,ppid,user,%cpu,%mem,ni,stat,cmd --sort=-%cpu | head -15

    pid — Process ID
    ppid — Parent Process ID (who spawned it)
    user — Owner
    %cpu — CPU usage
    %mem — Memory usage
    ni — Nice value
    stat — State code
    cmd — Command name
    --sort=-%cpu — Sort by CPU, highest first

Find a process by name:

pgrep firefox

Outputs just the PID(s) — cleaner than ps aux | grep firefox when you only need the number.

Find with details:

pgrep -a firefox

Outputs the PID and full command line:

3200 /usr/lib/firefox/firefox

Find by port:

sudo ss -tlnp | grep :80

Identifies which process is listening on port 80 (useful when something is already using a port you need).
/proc — The Process Filesystem

Linux exposes process information through the /proc filesystem — a virtual filesystem that doesn't exist on disk but is generated in real time by the kernel.

View a process's command line:

cat /proc/3200/cmdline | tr '\0' ' '

(The arguments are null-separated, so tr converts them to spaces.)

View a process's environment variables:

cat /proc/3200/environ | tr '\0' '\n'

View a process's memory maps:

cat /proc/3200/maps

View a process's status (human-readable):

cat /proc/3200/status

Output includes:

Name:	firefox
State:	S (sleeping)
Tgid:	3200
Pid:	3200
PPid:	2501
Uid:	1000
Threads:	28
VmRSS:	891234 kB      ← Physical memory used
VmSize:	2456789 kB     ← Virtual memory size

View a process's open files:

ls -la /proc/3200/fd/

Shows all file descriptors — every open file, socket, and pipe. This is the same information lsof provides, but without needing to install anything.

    💡 /proc Is a Goldmine

    /proc contains real-time information about every running process, the kernel itself (/proc/cpuinfo, /proc/meminfo, /proc/version), and hardware. It's one of Linux's most powerful diagnostic features — everything the kernel knows about processes is exposed as readable files.

systemd and Service Management

Long-running background processes (web servers, databases, SSH daemons) are called daemons or services. On Ubuntu 26.04, these are managed by systemd — the init system that runs as PID 1.
How systemd Manages Services

When you run sudo systemctl start nginx, systemd:

    Reads the service unit file (e.g., /lib/systemd/system/nginx.service)
    Spawns the nginx process in a cgroup (control group)
    Monitors the process — if it crashes, systemd can restart it
    Tracks all child processes spawned by the service

Cgroups are a kernel feature that groups processes hierarchically. When you stop a service, systemd sends SIGTERM to all processes in that service's cgroup — not just the main PID. After a configurable timeout (default 90 seconds), any remaining processes receive SIGKILL.

This means systemd can reliably clean up an entire process tree, even if a service spawns dozens of child processes.
Service Unit Files

Every service has a unit file defining how it runs. Example:

cat /lib/systemd/system/nginx.service

[Unit]
Description=The NGINX HTTP and reverse proxy server
After=syslog.target network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
ExecStart=/usr/sbin/nginx
ExecReload=/usr/sbin/nginx -s reload
ExecStop=/usr/sbin/nginx -s stop
PrivateTmp=true

[Install]
WantedBy=multi-user.target

Section	Purpose
[Unit]	Metadata, dependencies, ordering
[Service]	How to start, stop, reload the service
[Install]	When this service should start (boot level)

You won't need to write unit files as a beginner, but understanding their structure helps when troubleshooting.
systemctl Recap (from Chapter 16)
Command	Purpose
systemctl status nginx	Check if running, recent log output
sudo systemctl start nginx	Start a stopped service
sudo systemctl stop nginx	Stop a running service
sudo systemctl restart nginx	Restart (stop + start)
sudo systemctl reload nginx	Reload configuration without full restart
sudo systemctl enable nginx	Start automatically at boot
sudo systemctl disable nginx	Don't start at boot
systemctl is-enabled nginx	Check if set to start at boot
systemctl is-active nginx	Check if currently running
systemctl list-units --type=service	List all services and states
systemctl --failed	List services that failed to start
systemctl list-unit-files --type=service	List all installed service files
Viewing Service Cgroups

systemd-cgls

Shows the process tree organized by cgroup:

Control group /:
├─session-1.scope
│ └─1200 gnome-shell
│   ├─2500 Ptyxis
│   │ └─2501 bash
│   └─1300 nautilus
├─system.slice
│ ├─nginx.service
│ │ ├─890 nginx
│ │ └─891 nginx (worker)
│ ├─ssh.service
│ │ └─880 sshd
│ └─snapd.service
│   └─950 snapd

This shows how systemd organizes processes into groups. When you stop nginx.service, systemd kills everything under that cgroup — the master process and all worker processes.
Resource Limits with systemd

systemd can limit resources for services using cgroup controllers:

sudo systemctl set-property nginx.service CPUQuota=50% MemoryMax=1G

Limits nginx to 50% of CPU and 1 GB of RAM. The limits persist across restarts.
Practical Scenario: Cleaning Up a Runaway Process

Symptom: Your system is sluggish. A process is consuming 99% CPU.

Step 1: Identify the offender:

ps aux --sort=-%cpu | head -5

USER  PID   %CPU %MEM  COMMAND
john  8520  99.7  4.2  node /home/john/app/server.js

PID 8520 is the problem — a Node.js process eating nearly all CPU.

Step 2: Try a graceful termination:

kill 8520

Wait a few seconds. Check if it's gone:

ps aux | grep 8520

Step 3: If still running, escalate:

kill -9 8520

Step 4: If it was a service that should restart:

sudo systemctl status node-app.service
sudo systemctl restart node-app.service

Step 5: Check logs for why it went rogue:

journalctl -u node-app.service -b --no-pager | tail -50

Process Management Cheat Sheet
Task	Command
List all processes	ps aux
Process tree	pstree
Process tree with PIDs	pstree -p
Custom sorted listing	ps -eo pid,ppid,%cpu,%mem,cmd --sort=-%cpu
Find PID by name	pgrep firefox
Find PID with details	pgrep -a firefox
Start in background	command &
List background jobs	jobs
Suspend foreground	Ctrl+Z
Resume in background	bg or bg %1
Bring to foreground	fg or fg %1
Kill by PID	kill PID
Kill by job number	kill %1
Force kill	kill -9 PID
Kill by name	pkill firefox
Kill all by name	killall firefox
Start with low priority	nice -n 10 command
Change priority	renice -n 5 -p PID
Survive logout (before start)	nohup command > log 2>&1 &
Detach running job	disown %1
View process status	cat /proc/PID/status
View open files	ls -la /proc/PID/fd/
View cgroup tree	systemd-cgls
Service status	systemctl status servicename
List all services	systemctl list-units --type=service
List failed services	systemctl --failed
What's Next

You now understand the full lifecycle of processes on Linux: how they're created, how to move them between foreground and background, how to send them signals for graceful or forced termination, how to adjust their CPU priority, how to make them survive terminal disconnects, how to inspect their internals through /proc, and how systemd manages services through cgroups. This is one of the most practical chapters — these tools are used daily by every Linux administrator.

In Chapter 24, we'll cover shell scripting advanced topics — taking the scripting basics from Chapter 19 to the next level with arrays, case statements, functions with return values, error handling, and practical automation scripts.
Try It Yourself

    View the process tree. Run pstree -p | less. Trace how your terminal session descends from systemd (PID 1). Identify the parent-child chain that leads to your shell.

    Background a process. Run sleep 300 &. Verify it's in the job list with jobs. Practice suspending, resuming with bg, and bringing back with fg. Kill it with kill %1.

    Practice Ctrl+Z workflow. Start sleep 300 (foreground). Press Ctrl+Z to suspend. Run bg to resume in background. Run jobs to verify. Run fg to bring it back. Press Ctrl+C to kill it.

    Explore /proc. Find your shell's PID with echo $$. Then inspect its details:

    cat /proc/$$/status
    cat /proc/$$/cmdline | tr '\0' ' '
    ls -la /proc/$$/fd/

    Adjust process priority. Start a background process: sleep 300 &. Get its PID from the job output. Lower its priority: renice -n 10 -p <PID>. Verify with ps -o pid,ni,cmd -p <PID>.

    Practice nohup. Start a harmless background task: nohup sleep 300 > /dev/null 2>&1 &. Close your terminal. Open a new one. Verify the process is still running: pgrep sleep.

    Explore systemd cgroups. Run systemd-cgls | less. See how systemd organizes your processes. Run systemctl list-units --type=service --state=running to see all running services.

    Kill a process properly. Open a terminal and start sleep 500. In another terminal, find its PID: pgrep sleep. Try kill <PID> (graceful). If that doesn't work (it should for sleep), try kill -9 <PID>.

    ✅ Chapter 23 Complete You understand the process tree, process states, foreground/background job control, signals (SIGTERM vs. SIGKILL vs. SIGHUP), nohup and disown for surviving logout, nice/renice for priority management, /proc for inspecting process internals, and how systemd manages services through cgroups. In Chapter 24, we'll advance your shell scripting skills.

