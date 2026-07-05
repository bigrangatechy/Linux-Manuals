# Chapter 29: Troubleshooting

## The Art of Fixing Problems

Every Linux user troubleshoots. It doesn't matter how careful you are, how stable the distribution is, or how long you've been using it — something will eventually break. A black screen after an update, a Wi-Fi card that mysteriously stops working, a package that refuses to install, a desktop that won't load after login.

The difference between a frustrated user and a confident one isn't the absence of problems — it's having a systematic approach to solving them. This chapter gives you that approach: a methodology, a toolkit, and specific solutions for the most common issues on Ubuntu 26.04.

## The Troubleshooting Mindset

Before diving into specific fixes, adopt these principles:

### 1. Don't Panic, Don't Rush

A broken system feels urgent, but rash actions make things worse. The number one cause of "my system is completely destroyed" is a user who panicked and started running random commands they found online without understanding them.

**Stop. Assess. Understand. Then act.**

### 2. Isolate the Problem

Narrow down what's actually broken:

- "My computer doesn't work" → too vague
- "Wi-Fi connects but DNS doesn't resolve" → actionable
- "Screen is black after login" → a specific symptom with specific causes

Ask: What changed? Was there an update? A new driver? A configuration change? Did I run a command? Correlation isn't always causation, but recent changes are the most likely culprit.

### 3. Read the Error Messages

Error messages are the system talking to you. They feel intimidating because they're full of jargon, but they contain the answer — or at least the keywords you need to search for.

**Don't dismiss error messages. Read them. Copy them. Search them.**

### 4. Change One Thing at a Time

If you change five settings at once and the problem disappears, you don't know which fix worked. If it gets worse, you don't know which change caused it. Make one change, test, then make the next.

### 5. Take a Snapshot First

If the system is still bootable:

sudo timeshift --create --comments "Before troubleshooting"

Now you can experiment freely — if a fix makes things worse, restore the snapshot.
6. Document What You Did

Keep notes on what you try and the results. When you fix it, record the solution. Next time the same issue appears, you'll have the answer ready. This also helps immensely if you need to ask for help — you can say "I already tried X, Y, and Z."
Your Troubleshooting Toolkit

Before problems happen, know these tools:
TTY: The Terminal Behind the Desktop

When your desktop freezes or the display is black, TTY terminals give you text-only access to the system. Ubuntu provides six of them.

Switch to a TTY: Press Ctrl + Alt + F3 (or F4, F5, F6).

Return to the desktop: Press Ctrl + Alt + F2 (on Ubuntu 26.04, the desktop session runs on F2; F1 is the display manager login screen).

From a TTY, you can log in with your username and password, run commands, read logs, restart services, and fix the problem — all without a working graphical interface.

    💡 TTY Keyboard Layout

    TTYs use the default keyboard layout, which may differ from your desktop session if you configured a custom layout. If your password seems wrong in a TTY, check the keyboard layout — special characters may be in different positions.

Recovery Mode

If the system doesn't boot normally, Ubuntu's GRUB boot menu offers recovery options.

Access GRUB menu: Hold Shift (BIOS) or press Esc repeatedly (UEFI) during boot.

Select Advanced options for Ubuntu, then choose the entry with (recovery mode).

The recovery menu offers:
Option	What It Does
resume	Continue normal boot
clean	Free disk space (remove temp files, apt cache)
dpkg	Repair broken packages
fsck	Check and repair filesystem
root	Drop to a root shell for manual repair
The Live USB

Your Ubuntu installation USB (Chapter 5) is a rescue tool. Boot from it, select "Try Ubuntu," and you have a full environment to:

    Mount and repair your installed system's filesystem
    Restore Timeshift snapshots
    Chroot into the installed system to reinstall packages or fix GRUB
    Copy files off a broken system

Log Files: The System's Diary

Linux records everything that happens in log files. When something breaks, the logs tell you what went wrong and when.

systemd journal (the primary log):

journalctl -b

Shows all logs from the current boot. Use journalctl -b -p err to show only errors.

Kernel ring buffer (hardware and boot messages):

dmesg

Specific service logs:

journalctl -u servicename -b

Key log file locations:
File	Contents
/var/log/syslog	General system messages
/var/log/auth.log	Authentication events (logins, sudo)
/var/log/dmesg	Kernel ring buffer
/var/log/apt/history.log	APT package operations
/var/log/apt/term.log	APT installation output
/var/log/Xorg.0.log	Display server logs (mostly historical — Wayland)
~/.xsession-errors	Desktop session errors (X11)
~/.local/share/gnome-shell/gnome-shell-extension-errors.log	GNOME extension errors
Troubleshooting by Scenario
Scenario 1: Black Screen After Boot (No Display)

Symptom: System powers on, you hear the startup sound or disk activity, but the screen stays black.

Step 1: Try a TTY. Press Ctrl + Alt + F3. If you see a login prompt, the system is running — only the display is broken. Log in.

Step 2: Check if it's a GPU driver issue.

lspci -k | grep -A 3 -i vga

This shows your GPU and which driver is loaded. If you have an NVIDIA card and see nouveau instead of nvidia:

# Check if the NVIDIA driver is installed
dpkg -l | grep nvidia-driver

# Reinstall if missing
sudo apt update
sudo apt install nvidia-driver-595
sudo reboot

Step 3: Try a different display session. At the login screen, click the gear icon. If "Ubuntu on Wayland" and "Ubuntu on Xorg" are both available, try the other one. If Wayland is broken, Xorg may work (or vice versa).

Note: Ubuntu 26.04 is Wayland-only by default. An Xorg session may not be available unless you install gnome-session-xorg.

Step 4: Check logs for display errors.

journalctl -b | grep -iE "gpu|drm|nvidia|amdgpu|display|wayland|gnome-shell"

Step 5: Boot into recovery mode and reinstall the driver. If the TTY doesn't load either, use recovery mode → root shell:

sudo apt update
sudo apt install --reinstall nvidia-driver-595
sudo reboot

Step 6: Remove problematic drivers. If a recently installed driver caused the issue:

sudo apt purge 'nvidia-*'
sudo apt autoremove
sudo reboot

This falls back to the open-source driver (Nouveau for NVIDIA, amdgpu for AMD, i915 for Intel), which may at least get you a working display to diagnose further.
Scenario 2: System Freezes Completely

Symptom: The desktop stops responding. Mouse doesn't move. Keyboard doesn't respond. Clock is frozen.

Step 1: Try the Magic SysRq Keys.

The Linux kernel responds to special key combinations that work even when everything else is frozen:

Hold Alt + SysRq (SysRq is often the same key as Print Screen), then slowly type:

R E I S U B

Each letter triggers a step:
Key	Action
R	Take keyboard back from the X server / display manager
E	Send SIGTERM to all processes (graceful shutdown of apps)
I	Send SIGKILL to all processes (force kill)
S	Sync filesystems (write data to disk — prevents corruption)
U	Remount filesystems read-only (further protects data)
B	Reboot immediately

Wait a few seconds between each key. This safely reboots a frozen system with minimal data loss risk.

    ⚠️ SysRq May Be Disabled

    Some systems disable the magic SysRq keys. Check with:

    cat /proc/sys/kernel/sysrq

    1 means enabled, 0 means disabled. Enable temporarily:

    echo 1 | sudo tee /proc/sys/kernel/sysrq

    For permanent enable, add kernel.sysrq = 1 to /etc/sysctl.d/99-sysrq.conf.

Step 2: After Reboot, Investigate the Cause.

journalctl -b -1

The -1 flag shows the previous boot's logs — the one that froze. Look for errors around the time of the freeze:

journalctl -b -1 -p err

Common causes of complete freezes:

    GPU driver crash (NVIDIA is the most common culprit)
    Out of memory (OOM killer didn't react in time)
    Hardware failure (overheating, failing RAM, dying drive)
    Kernel bug (rare but possible — check dmesg for kernel oops/panic)

Step 3: Check temperature.

sensors

If temperatures are above 90°C, thermal throttling may be failing. Clean dust from fans, check thermal paste, or reduce CPU frequency.
Scenario 3: Stuck at Boot (Won't Reach Desktop)

Symptom: The system powers on but gets stuck during boot — maybe at the Ubuntu logo, maybe at a text screen with error messages.

Step 1: Read the boot messages.

Press Esc during boot to hide the splash screen and see the actual boot messages. Whatever it's stuck on will be the last line on screen.

Step 2: Boot into recovery mode.

From the GRUB menu: Advanced options → recovery mode → root shell.

Step 3: Check the filesystem. If boot messages mention filesystem errors:

fsck /dev/sda2

Replace /dev/sda2 with your root partition (check with lsblk). Answer y to repair prompts, or use:

fsck -y /dev/sda2

Step 4: Check disk space.

A full disk prevents the system from writing temporary files, causing boot failures:

df -h

If the root partition (/) is at 100%, clear space:

apt clean
journalctl --vacuum-size=100M
rm -rf /tmp/*

Step 5: Check for broken packages.

dpkg --configure -a
apt --fix-broken install

Step 6: Update or remove problematic kernels.

If a kernel update broke boot:

# List installed kernels
dpkg --list | grep linux-image

# In recovery mode, try booting an older kernel from GRUB
# Advanced options → select an older kernel version

# Remove a problematic kernel
apt remove linux-image-7.0.0-XX-generic

Scenario 4: Login Loop (Password Accepted, Returns to Login Screen)

Symptom: You type your password, the screen goes black momentarily, then you're back at the login screen. Repeat forever.

Cause 1: Full disk. If your home partition is 100% full, the desktop can't create session files. From a TTY (Ctrl + Alt + F3):

df -h ~

If full, remove files:

rm -rf ~/.cache/*
sudo apt clean
rm -rf /tmp/$USER/*

Cause 2: Wrong permissions on .Xauthority.

ls -la ~/.Xauthority

The owner should be your user. If it shows root, fix it:

sudo chown $USER:$USER ~/.Xauthority

Cause 3: Broken GNOME Shell or extensions.

Disable all extensions and try logging in:

gsettings set org.gnome.shell disable-user-extensions true

If login works with extensions disabled, re-enable them one by one to find the culprit.

Cause 4: Broken display manager.

sudo systemctl restart gdm3

Or, from a TTY:

sudo systemctl restart display-manager

Scenario 5: No Internet (Wi-Fi or Ethernet)

Step 1: Check the interface.

ip link

Look for your network interface:

    Wired: usually enp3s0, eno1, or eth0
    Wireless: usually wlp2s0, wlan0

If no wireless interface appears, the Wi-Fi driver may not be loaded. Check:

lspci -k | grep -i net
lsmod | grep -i cfg80211

Step 2: Check if the interface is up.

ip link set enp3s0 up     # Wired
ip link set wlp2s0 up     # Wireless

Step 3: Check NetworkManager.

nmcli device status

If NetworkManager is down:

sudo systemctl restart NetworkManager

Step 4: Check IP assignment.

ip addr

No IP address? Request one:

sudo dhclient enp3s0     # Wired DHCP

For Wi-Fi:

nmcli device wifi list
nmcli device wifi connect "SSID" password "yourpassword"

Step 5: Check DNS resolution. If you have an IP but can't load websites:

ping -c 3 8.8.8.8      # Tests raw connectivity
ping -c 3 google.com   # Tests DNS

If 8.8.8.8 works but google.com doesn't, DNS is broken. Fix it:

# Temporarily set DNS
sudo resolvectl dns enp3s0 8.8.8.8 1.1.1.1

Or edit NetworkManager connection:

nmcli connection modify "Connection Name" ipv4.dns "8.8.8.8 1.1.1.1"
nmcli connection up "Connection Name"

Step 6: Check the firewall.

sudo ufw status

If the firewall is blocking DNS or DHCP:

sudo ufw allow out 53/udp     # DNS
sudo ufw allow out 67/udp     # DHCP

Step 7: Check driver-specific issues.

Realtek Wi-Fi cards occasionally regress after kernel updates. Check:

lspci -nn | grep -i net
modinfo rtl8821ce    # Replace with your driver name

Refer to Chapter 22 for detailed Wi-Fi troubleshooting.
Scenario 6: Audio Not Working

Step 1: Check if the system detects audio devices.

aplay -l

Lists all audio playback devices. If nothing appears, the audio driver isn't loaded.

Step 2: Check PipeWire (audio server on Ubuntu 26.04).

systemctl --user status pipewire

If not running:

systemctl --user restart pipewire pipewire-pulse wireplumber

Step 3: Check output device selection.

Open Settings → Sound. Verify the correct output device is selected. Sometimes HDMI audio is selected instead of speakers.

From the terminal:

wpctl status

This shows all audio devices and their statuses. Set the default output:

wpctl set-default <DEVICE_ID>

Step 4: Unmute everything.

alsamixer

Press F6 to select your sound card. Press M to toggle mute on any channel showing MM (muted). Channels should show OO (unmuted). Press Esc to exit.

Step 5: Test audio.

speaker-test -t sine -f 440

Press Ctrl+C to stop. If you hear a tone, audio hardware works — the issue is application-specific.
Scenario 7: Package Manager Errors

Symptom: apt commands fail with errors about locked files, broken dependencies, or held packages.

Problem: Locked dpkg.

If an update was interrupted (power loss, crash, force-closed terminal):

sudo dpkg --configure -a
sudo apt --fix-broken install

Problem: "Could not get lock /var/lib/dpkg/lock"

Another package operation is running. Wait for it to finish, or if you're sure nothing is running:

sudo killall apt apt-get dpkg
sudo rm /var/lib/dpkg/lock*
sudo dpkg --configure -a

    ⚠️ Never Force-Remove Locks Casually

    Removing locks without confirming no process is running can corrupt the package database. Always check first:

    ps aux | grep -E "apt|dpkg"

    Only remove locks if no package operation is actually running.

Problem: Held/broken packages.

sudo apt update
sudo apt -f install
sudo apt --fix-broken install
sudo apt autoremove

If a specific package is broken:

sudo apt remove --purge packagename
sudo apt install packagename

Check why a package can't be installed (new in APT 3.1, Ubuntu 26.04):

apt why-not packagename

Problem: Repository errors.

If a PPA or repository is unreachable:

sudo apt update

Look for Err: or 404 lines. Comment out broken repositories:

sudo nano /etc/apt/sources.list

Or for DEB822-format sources (Ubuntu 26.04 default):

ls /etc/apt/sources.list.d/
sudo nano /etc/apt/sources.list.d/problem-source.sources

Comment out lines with # and run sudo apt update again.
Scenario 8: Slow System Performance

Step 1: Identify the resource hog.

top -o %CPU

Or, if installed:

btop

Sort by CPU to find the process consuming the most resources. If a specific app is the culprit, kill it:

kill PID

Step 2: Check memory.

free -h

If available memory is very low, check what's consuming it:

ps aux --sort=-%mem | head -10

If the system is using swap heavily, consider:

swapon --show

On Ubuntu 26.04, ZRAM is the default swap method. If ZRAM isn't active:

sudo systemctl enable --now systemd-zram-setup@zram0.service

Step 3: Check disk I/O.

iostat -x 1 5

If iostat isn't installed: sudo apt install sysstat. High %util on a device means the disk is saturated. A failing drive can cause extreme slowness — check SMART:

sudo smartctl -a /dev/sda

Step 4: Check GNOME Shell resource usage.

ps aux | grep gnome-shell

If gnome-shell is using excessive memory or CPU, disable extensions (Chapter 25):

gsettings set org.gnome.shell disable-user-extensions true

If performance improves, an extension is the culprit. Re-enable them one by one.

Step 5: Check for overheating.

sensors

High temperatures trigger thermal throttling, reducing CPU speed dramatically. Clean dust, check fans, or improve airflow.
Scenario 9: Application Won't Launch

Step 1: Launch from terminal.

Find the application's command name (right-click the app icon → "Details" in App Center, or check /usr/share/applications/). Then run it from the terminal:

firefox

Error messages appear in the terminal. Common issues:

    Missing dependency
    Permission error
    Configuration file corruption
    Library version conflict

Step 2: Clear application cache/config.

rm -rf ~/.cache/applicationname/
rm -rf ~/.config/applicationname/

Try launching again. If it works, the cache/config was corrupted.

Step 3: Reinstall the application.

# APT
sudo apt reinstall packagename

# Snap
sudo snap remove packagename && sudo snap install packagename

# Flatpak
flatpak uninstall packagename && flatpak install flathub packagename

Step 4: Check AppArmor denials.

sudo dmesg | grep apparmor | grep DENIED

If an AppArmor profile is blocking the app, you may need to adjust the profile or file a bug report.
Scenario 10: Can't Mount a Drive

Step 1: Identify the drive.

lsblk -f

Step 2: Try mounting manually.

sudo mount /dev/sdb1 /mnt

Step 3: Read the error.

"Unknown filesystem type":

sudo apt install ntfs-3g exfat-fuse

"Bad superblock" (filesystem corruption):

sudo fsck /dev/sdb1

"Device is busy": Something is using the drive. Check:

lsof /dev/sdb1
fuser -m /dev/sdb1

Kill the process or unmount properly:

sudo umount /dev/sdb1

If unmount fails:

sudo umount -l /dev/sdb1    # Lazy unmount

Scenario 11: Bluetooth Not Working

Step 1: Check Bluetooth service.

systemctl status bluetooth

If not running:

sudo systemctl start bluetooth

Step 2: Check Bluetooth adapter.

bluetoothctl show

If no adapter is found:

rfkill list bluetooth
rfkill unblock bluetooth

Step 3: Pair a device.

bluetoothctl
[bluetooth]# power on
[bluetooth]# agent on
[bluetooth]# scan on
[bluetooth]# pair XX:XX:XX:XX:XX:XX
[bluetooth]# connect XX:XX:XX:XX:XX:XX
[bluetooth]# trust XX:XX:XX:XX:XX:XX
[bluetooth]# quit

Step 4: Check logs.

journalctl -u bluetooth -b

Scenario 12: Timeshift Restore Fails

Step 1: Try from a live USB.

Boot the Ubuntu installation USB, install Timeshift in the live session, and restore from there (Chapter 26).

Step 2: Check snapshot integrity.

sudo timeshift --list

Verify the snapshot you're trying to restore appears and has files.

Step 3: Check disk space.

df -h

Timeshift needs enough free space to restore. If the disk is nearly full, free space first.

Step 4: Try restoring specific paths.

Instead of a full restore, manually copy specific files from the snapshot:

sudo rsync -aAXHv /timeshift/snapshots/2026-07-04_18-00-01/localhost/etc/ /etc/

The Troubleshooting Decision Tree

When a problem occurs, follow this general path:

1. What changed?
   ├── Recent update? → Check apt history: cat /var/log/apt/history.log
   ├── New driver? → Remove it: sudo apt purge drivername
   ├── New extension? → Disable: gsettings set org.gnome.shell disable-user-extensions true
   ├── Config change? → Revert it
   └── Nothing obvious? → Continue to step 2

2. Can you reach a TTY? (Ctrl+Alt+F3)
   ├── Yes → System is running, display/GUI issue
   │   ├── Check logs: journalctl -b -p err
   │   ├── Restart display manager: sudo systemctl restart gdm3
   │   └── Check GPU: lspci -k | grep -A3 VGA
   └── No → Hardware or kernel issue
       ├── Try recovery mode from GRUB
       ├── Try older kernel from GRUB
       └── Boot live USB to diagnose

3. Is it network-related?
   ├── Check interface: ip link
   ├── Check NetworkManager: systemctl status NetworkManager
   ├── Check DNS: ping 8.8.8.8 vs ping google.com
   └── Check firewall: sudo ufw status

4. Is it a package issue?
   ├── Fix broken: sudo apt --fix-broken install
   ├── Configure pending: sudo dpkg --configure -a
   ├── Check why: apt why-not packagename
   └── Reinstall: sudo apt reinstall packagename

5. Is it a hardware issue?
   ├── Check dmesg: sudo dmesg | grep -i error
   ├── Check temps: sensors
   ├── Check disk SMART: sudo smartctl -a /dev/sda
   └── Check memory: sudo memtest86+ (from boot)

6. Still stuck?
   ├── Restore Timeshift snapshot
   ├── Boot live USB and chroot
   ├── Search with exact error messages
   └── Ask for help (Chapter 31)

Common Quick Fixes

These solve a disproportionate number of problems:

# The "turn it off and on again" of Linux
sudo systemctl restart servicename

# Fix broken packages
sudo dpkg --configure -a && sudo apt --fix-broken install

# Clear GNOME Shell issues
gsettings set org.gnome.shell disable-user-extensions true

# Restart audio
systemctl --user restart pipewire pipewire-pulse wireplumber

# Restart NetworkManager
sudo systemctl restart NetworkManager

# Clean up disk space
sudo apt clean && sudo apt autoremove && sudo journalctl --vacuum-size=200M

# Rebuild initramfs (after kernel or driver changes)
sudo update-initramfs -u

# Update GRUB (after kernel or boot changes)
sudo update-grub

# Reload kernel modules
sudo modprobe -r modulename && sudo modprobe modulename

# Restart the display manager (when desktop is broken but TTY works)
sudo systemctl restart gdm3

Reading Logs Effectively

Logs are your best diagnostic tool. Here's how to use them effectively:
journalctl Essentials

# Current boot, errors only
journalctl -b -p err

# Previous boot (if the problem happened before reboot)
journalctl -b -1

# Previous boot, errors only
journalctl -b -1 -p err

# Specific service
journalctl -u NetworkManager -b

# Last 100 lines
journalctl -n 100

# Follow live (like tail -f)
journalctl -f

# Time range
journalctl --since "today"
journalctl --since "2026-07-04 10:00" --until "2026-07-04 12:00"

# Search for keywords
journalctl -b | grep -i "error\|fail\|warning"

# JSON output (for scripting)
journalctl -o json

Priority Levels
Code	Level	Meaning
0	emerg	System unusable
1	alert	Immediate action required
2	crit	Critical conditions
3	err	Error conditions
4	warning	Warning conditions
5	notice	Normal but significant
6	info	Informational
7	debug	Debug-level messages

Use -p err to filter to level 3 and below (errors and critical). Use -p warning for level 4 and below (includes warnings).
APT History

If an update caused the problem, check what was installed:

# What was installed/upgraded/removed recently
cat /var/log/apt/history.log

# Older history (rotated logs)
zcat /var/log/apt/history.log.*.gz

This shows the exact date, time, and packages affected by each APT operation — invaluable for correlating problems with updates.
dmesg for Hardware Issues

# All kernel messages
dmesg

# Errors and warnings only
dmesg --level=err,warn

# Follow live
dmesg -w

# Filter by device
dmesg | grep -i usb
dmesg | grep -i nvme
dmesg | grep -i snd

When to Restore vs. When to Fix

Sometimes the fastest solution is a Timeshift restore. Other times, fixing the specific issue is better.
Situation	Best Approach
Broke something 10 minutes ago	Timeshift restore — fastest path to working system
Problem has been gradually worsening	Investigate and fix — restoring would mask the root cause
Single application is broken	Fix the application — reinstall or clear cache
System update broke everything	Timeshift restore, then wait for a fixed update
Disk failure	Restore from Deja Dup/Borg after replacing the drive
Intermittent issue (happens randomly)	Investigate logs — restoring won't prevent recurrence
Config change broke desktop	Try reverting the specific setting first, then Timeshift
When to Ask for Help

You've tried everything you can think of. The system is still broken. Now what?

Chapter 31 covers this in depth, but here's the short version:

    Search first. Copy the exact error message into a search engine. Add "Ubuntu 26.04." Someone has likely had this exact problem.

    Gather information. Before posting on a forum, collect:
        Exact error messages (copy-paste, don't paraphrase)
        What you were doing when it happened
        What you've already tried and the results
        Relevant log output (journalctl -b -p err)
        System info (lsb_release -a, uname -r, hardware details)

    Post in the right place.
        Ask Ubuntu — Best for specific technical questions
        Ubuntu Community Hub — Official forums
        Reddit r/Ubuntu — Community support
        Bug reports: Launchpad

    Be patient and courteous. Community members help voluntarily. A well-written question with exact error messages and system details gets answered quickly. "It doesn't work, help!" does not.

Troubleshooting Cheat Sheet
Problem	First Command	Second Command
Black screen	Ctrl+Alt+F3 (TTY)	journalctl -b -p err
System freeze	Alt+SysRq R E I S U B	journalctl -b -1 -p err
Won't boot	GRUB → recovery mode	fsck /dev/sdXN
Login loop	TTY → df -h ~	chown $USER:$USER ~/.Xauthor*
No internet	ip link	nmcli device status
No DNS	ping 8.8.8.8	resolvectl dns IFACE 8.8.8.8
No audio	aplay -l	systemctl --user restart pipewire
Broken packages	sudo dpkg --configure -a	sudo apt --fix-broken install
Slow system	top -o %CPU	free -h
App won't open	Run from terminal	rm -rf ~/.cache/appname/
Drive won't mount	lsblk -f	sudo mount /dev/sdXN /mnt
Bluetooth	systemctl status bluetooth	rfkill unblock bluetooth
GPU issues	lspci -k | grep -A3 VGA	sudo apt purge nvidia-*
Disk full	df -h	sudo apt clean && sudo journalctl --vacuum-size=200M
Check logs	journalctl -b -p err	dmesg --level=err,warn
Check APT history	cat /var/log/apt/history.log	zcat /var/log/apt/history.log.*.gz
Recovery mode	Hold Shift/Esc at boot	Advanced options → recovery
Restore snapshot	sudo timeshift --restore	Boot live USB if system is down
Restart display	sudo systemctl restart gdm3	From TTY
Update initramfs	sudo update-initramfs -u	After kernel/driver changes
Update GRUB	sudo update-grub	After kernel/boot changes
What's Next

You now have a systematic approach to troubleshooting: the mindset (don't panic, isolate, read errors, change one thing, snapshot, document), the toolkit (TTY, recovery mode, live USB, log files), solutions for twelve common scenarios (black screen, freezes, boot failures, login loops, network, audio, packages, performance, applications, mounts, Bluetooth, Timeshift), a decision tree for navigating unknown problems, quick-fix commands, log reading techniques, and guidance on when to restore versus when to fix.

In Chapter 30, the final chapter, we'll cover getting help and resources — where to find documentation, how to ask effective questions, community norms, and the best resources for continued learning.
Try It Yourself

    Practice TTY switching. Press Ctrl+Alt+F3 to switch to a TTY. Log in. Run a few commands (whoami, uname -r, df -h). Press Ctrl+Alt+F2 to return to the desktop. Knowing how to do this under pressure is essential.

    Read your current boot logs. Run journalctl -b -p err. Review what errors appeared during your current boot. Most will be minor (a service timing out, a deprecated config option), but understanding what normal looks like helps you spot abnormal.

    Check APT history. Run cat /var/log/apt/history.log. Review what packages were installed or updated recently. This is your first stop when an update causes problems.

    Simulate a frozen app. Open a terminal and run sleep 300 &. Get its PID: pgrep sleep. Kill it: kill PID. Practice the workflow — you'll use this pattern constantly.

    Try recovery mode. Reboot your system. Hold Shift (or press Esc on UEFI) during boot to access the GRUB menu. Select "Advanced options for Ubuntu." Don't select recovery mode — just observe what's available. Knowing the menu exists is half the battle.

    Test the magic SysRq keys safely. Run cat /proc/sys/kernel/sysrq to check if they're enabled. If so, practice the rhythm: Alt+SysRq then slowly R E I S U B. Don't worry — if your system is fine, REISUB will just reboot it cleanly (like a safe restart).

    Write a diagnostic script. Create a script that gathers all the key troubleshooting information at once: hostname, kernel version, disk usage, memory, top CPU processes, recent errors, network status, and APT history. Save it as ~/bin/diagnose.sh. When something breaks, run it first.

    Take a snapshot NOW. If you haven't already: sudo timeshift --create --comments "Pre-troubleshooting baseline". Every future troubleshooting session should start with a snapshot.

    ✅ Chapter 29 Complete You can systematically troubleshoot Ubuntu 26.04: switch to TTYs, access recovery mode, use a live USB for rescue, read logs with journalctl and dmesg, diagnose and fix black screens, system freezes, boot failures, login loops, network issues, audio problems, package manager errors, performance issues, application failures, mount failures, Bluetooth problems, and Timeshift restore failures. You have a decision tree, quick-fix commands, and know when to restore vs. when to investigate. In Chapter 30, we'll cover getting help and continuing your Linux journey.

