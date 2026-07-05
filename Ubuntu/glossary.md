# Appendix A: Glossary

Terms used throughout this manual, arranged alphabetically. Cross-references are marked with →.

---

**ACL (Access Control List)** — A finer-grained permission system that extends beyond the traditional owner/group/other model, allowing specific permissions for individual users or groups.

**Adwaita** — GNOME's default visual theme and design language. Ubuntu ships with Yaru instead, but Adwaita remains the upstream GNOME look.

**ALSA (Advanced Linux Sound Architecture)** — The kernel-level audio framework on Linux. On Ubuntu 26.04, PipeWire sits above ALSA as the user-facing audio server.

**AppArmor** — Ubuntu's mandatory access control (MAC) system. Restricts what individual programs can do, even if the program is compromised. See Chapter 27.

**App Center** — Ubuntu 26.04's graphical software manager for installing and updating deb packages and snaps.

**apt** — The high-level package management tool for `.deb` packages. Handles dependencies, repositories, and upgrades. See Chapter 18.

**Architecture** — The CPU instruction set a system uses. Most desktop Linux runs on `x86-64` (also called `amd64`); ARM (`aarch64`) is used in Raspberry Pi, Apple Silicon Macs, and some laptops. Ubuntu 26.04 ships `x86-64-v3` optimized packages.

**Associative array** — A key-value data structure in Bash 4.0+. Also called a dictionary or hash map in other languages. See Chapter 24.

**autostart** — Applications or scripts that launch automatically when you log in. Configured via `.desktop` files in `~/.config/autostart/`. See Chapter 25.

**background process** — A process running without occupying the terminal. Started with `&`, resumed with `bg`, or detached with `nohup`/`disown`. See Chapter 23.

**Bash** — The default shell (command interpreter) on Ubuntu. Stands for "Bourne Again Shell." See Chapter 13.

**BIOS (Basic Input/Output System)** — Legacy firmware that initializes hardware before the OS loads. Being replaced by UEFI on modern systems.

**BorgBackup (borg)** — A deduplicating, compressing, encrypting backup tool. Space-efficient for frequent backups. See Chapter 26.

**btop** — A colorful, real-time system resource monitor. Successor to `top` and `htop`. See Chapter 30.

**Btrfs (B-tree file system)** — A modern filesystem supporting snapshots, subvolumes, and built-in RAID. Optional on Ubuntu; ext4 is the default.

**case statement** — A multi-branch control structure in shell scripting, cleaner than long if/elif chains. See Chapter 24.

**cgroup (control group)** — A kernel feature for grouping processes and applying resource limits. systemd uses cgroups to manage services. See Chapter 23.

**Checksum** — A fingerprint of a file used to verify integrity. SHA-256 checksums confirm downloaded ISOs aren't corrupted. See Chapter 5.

**Clipboard** — Temporary storage for copy/cut data. GNOME has a single clipboard; extensions like Clipboard Indicator add history.

**Codename** — Each Ubuntu release has an alliterative animal codename (e.g., 26.04 = "Resolute Raccoon"). See Chapter 3.

**Command line** — The text interface where you type commands. Interchangeable with "terminal" or "shell" in casual usage.

**Cron** — Linux's built-in task scheduler. Runs commands at specified intervals. Configured with `crontab -e`. See Chapter 24.

**Daemon** — A background process that runs continuously, typically providing a service (web server, SSH, etc.). Managed by systemd on Ubuntu.

**dconf** — The low-level configuration storage system behind GNOME's GSettings. Edited with `dconf-editor`. See Chapter 25.

**Dependency** — A package or library required by another package. APT resolves dependencies automatically.

**DEB822** — The new `.sources` file format for APT repositories, replacing the traditional one-line-per-source `sources.list` format. Default on Ubuntu 26.04. See Chapter 18.

**Desktop environment (DE)** — The graphical interface layer: window manager, panels, file manager, settings, and default apps. Ubuntu uses GNOME; Kubuntu uses KDE Plasma. See Chapter 8.

**Device file** — A file in `/dev` representing a hardware device (e.g., `/dev/sda` for a disk, `/dev/null` for a black hole).

**Ding** — GNOME extension that places a home-folder icon on the desktop. Pre-installed on Ubuntu 26.04.

**Distribution (distro)** — A complete Linux OS package: kernel + tools + desktop + package manager + repositories. Ubuntu, Fedora, Debian, and Arch are all distributions. See Chapter 1.

**DLL** — A Windows shared library. Linux uses `.so` (shared object) files instead.

**DNS (Domain Name System)** — Translates domain names (google.com) to IP addresses (142.250.80.46). See Chapter 21.

**dpkg** — The low-level package manager behind APT. Handles individual `.deb` files without dependency resolution.

**Driver** — Software that lets the kernel communicate with hardware. Most Linux drivers are built into the kernel. NVIDIA drivers are a notable exception. See Chapter 22.

**Dual boot** — Having two operating systems installed on the same computer, choosing which to start at boot. See Chapter 6.

**Environment variable** — A named value available to all processes, set with `export`. Examples: `$PATH`, `$HOME`, `$USER`. See Chapter 19.

**ESM (Extended Security Maintenance)** — Extended security updates for Ubuntu LTS releases beyond standard support. Part of Ubuntu Pro. See Chapter 3.

**ext4** — The default filesystem for Ubuntu desktop installations. Reliable, mature, and well-supported.

**Extension (GNOME Shell)** — A small add-on modifying the GNOME desktop. Managed through the Extensions app. See Chapter 25.

**Fatigue (from terminal)** — The feeling of overwhelm when first learning the command line. Normal and temporary — every Linux user experiences it.

**Filesystem hierarchy** — The standard directory structure on Linux: `/`, `/home`, `/etc`, `/var`, `/usr`, and so on. See Chapter 14.

**Firewall** — A network security barrier controlling incoming and outgoing traffic. Ubuntu uses UFW. See Chapter 27.

**Flatpak** — A cross-distro package format with sandboxing. Installed from Flathub. See Chapter 18.

**Flathub** — The primary Flatpak application repository. See Chapter 18.

**Foreground process** — A process occupying the terminal until it finishes. Moved to background with `Ctrl+Z` and `bg`. See Chapter 23.

**FOSS (Free and Open Source Software)** — Software whose source code is freely available to study, modify, and redistribute. See Chapter 2.

**fstab** — The filesystem table (`/etc/fstab`) defining which partitions mount at boot and where. See Chapter 14.

**getopts** — A shell builtin for parsing command-line options and flags in scripts. See Chapter 24.

**GID (Group ID)** — A numeric identifier for a user group. See Chapter 17.

**GNOME** — The default desktop environment for Ubuntu. Version 50 ships with Ubuntu 26.04. See Chapter 9.

**GNOME Tweaks** — A configuration tool exposing GNOME settings not available in the standard Settings app. See Chapter 25.

**GPL (GNU General Public License)** — A copyleft license requiring derivative works to remain free and open. The Linux kernel uses GPL. See Chapter 2.

**GPG (GNU Privacy Guard)** — An encryption and signing tool. Used for encrypted backups, verifying package signatures, and secure communication.

**gsettings** — The command-line tool for reading and writing GNOME configuration values. See Chapter 25.

**GRUB (Grand Unified Bootloader)** — The bootloader that lets you choose which OS or kernel to start. See Chapter 6.

**GUI (Graphical User Interface)** — Windows, buttons, menus — the visual interface. Contrasted with the command line.

**Hard link** — A directory entry pointing to the same inode as another file. Deleting one doesn't affect the other. See Chapter 15.

**Here document** — A shell syntax for providing multi-line input to a command (`<< EOF ... EOF`). See Chapter 24.

**Here string** — A shell syntax for providing a single string as input to a command (`<<< "text"`). See Chapter 24.

**Home directory** — Your personal space at `/home/username`. Contains your files and per-user configuration (`~/.config`, `~/.local`). Abbreviated as `~`. See Chapter 14.

**HWE (Hardware Enablement)** — Newer kernel and graphics stacks backported to LTS releases for supporting newer hardware.

**Init system** — The first process (PID 1) that starts all other services. Ubuntu uses systemd. See Chapter 23.

**Inode** — A data structure storing a file's metadata (permissions, size, location on disk) but not its name. See Chapter 14.

**Interim release** — A non-LTS Ubuntu release with 9 months of support, shipping every 6 months between LTS releases. See Chapter 3.

**I/O (Input/Output)** — Data transfer between a process and the outside world: disk, network, keyboard, display.

**IP address** — A numeric identifier for a device on a network. Private IPs (192.168.x.x, 10.x.x.x) are local; public IPs are internet-visible. See Chapter 21.

**Job** — A shell's representation of a background or suspended process. Managed with `jobs`, `fg`, `bg`. See Chapter 23.

**Kernel** — The core of the Linux operating system. Manages hardware, memory, processes, and security. Ubuntu 26.04 uses kernel 7.0. See Chapter 1.

**Kernel module** — Loadable code extending the kernel's capabilities (drivers, filesystem support). Managed with `modprobe`. See Chapter 22.

**KDE Plasma** — A desktop environment emphasizing customization. Used by Kubuntu. Version 6.6 ships with Ubuntu 26.04. See Chapter 9.5.

**kill** — A command for sending signals to processes. Default signal is SIGTERM. See Chapter 23.

**Kubuntu** — An official Ubuntu flavor using KDE Plasma instead of GNOME. See Chapter 8.

**Libadwaita** — GNOME's application framework enforcing consistent visual styling. GTK themes may not fully apply to Libadwaita apps. See Chapter 25.

**Link (symbolic or hard)** — A pointer to another file. Symbolic links (`ln -s`) are shortcuts; hard links (`ln`) point to the same data. See Chapter 15.

**Live USB** — A bootable USB drive running Ubuntu without installing. Used for testing, installation, and rescue. See Chapter 5.

**LN (link)** — The command for creating hard or symbolic links. See Chapter 15.

**Locale** — Language and regional settings (date format, currency, character encoding). See Chapter 6.

**LSB (Linux Standard Base)** — A specification for standardizing Linux distributions.

**LTS (Long Term Support)** — Ubuntu releases supported for 5+ years, aimed at stability. Ubuntu 26.04 LTS is supported through 2031 (standard) or ~2036 (ESM). See Chapter 3.

**LUKS (Linux Unified Key Setup)** — Full disk encryption standard. Optionally TPM-backed on Ubuntu 26.04. See Chapter 6.

**Machine ID** — A unique identifier for the system, stored in `/etc/machine-id`.

**Manifesto (open source)** — See OSI and FOSS. The Open Source Definition and the Free Software Definition are the foundational texts. See Chapter 2.

**Mapfile** — A Bash builtin that reads lines from stdin into an array. Also called `readarray`. See Chapter 24.

**Metadata** — Data about data. File permissions, timestamps, and ownership are file metadata.

**Modprobe** — Command to load or unload kernel modules. See Chapter 22.

**Mount** — Attaching a filesystem so it's accessible at a directory (mount point). See Chapter 14.

**Multipass** — A tool for running Ubuntu VMs quickly on Windows, macOS, or Linux.

**NAT (Network Address Translation)** — A router feature translating private IP addresses to a public one. Provides a natural firewall. See Chapter 21.

**ncdu** — An interactive, terminal-based disk usage analyzer. See Chapter 30.

**NetworkManager** — Ubuntu's default network configuration service. Managed with `nmcli` and `nmtui`. See Chapter 21.

**Nice value** — Process priority from -20 (highest) to 19 (lowest). Default is 0. Adjusted with `nice` and `renice`. See Chapter 23.

**nohup** — Runs a command that ignores SIGHUP, allowing it to survive terminal closure. See Chapter 23.

**Nouveau** — The open-source NVIDIA driver. Works but with limited 3D performance compared to the proprietary driver. See Chapter 22.

**NTFS** — Windows filesystem. Linux can read and write NTFS using `ntfs-3g`. See Chapter 28.

**OOM Killer (Out Of Memory Killer)** — A kernel mechanism that kills processes when the system runs out of memory. See Chapter 30.

**Overlay filesystem** — A union filesystem used by containers and snap packages to layer changes on a read-only base.

**Package** — A bundled software archive with metadata, dependencies, and installation scripts. Formats include `.deb`, snap, flatpak, and AppImage. See Chapter 18.

**Parameter expansion** — Bash's built-in string manipulation using `${var...}` syntax. See Chapter 24.

**Partition** — A division of a disk drive, each acting as a separate device. See Chapter 6.

**passwd** — The file `/etc/passwd` storing user account info, and the command to change passwords. See Chapter 17.

**Path** — 1. The colon-separated list of directories (`$PATH`) the shell searches for executables. 2. A file or directory location in the filesystem. See Chapter 19.

**Payload** — The data portion of a package or network packet.

**Permission** — Access rights on a file: read (r), write (w), execute (x) for owner, group, and others. See Chapter 16.

**PID (Process ID)** — A unique number identifying a running process. Assigned by the kernel. See Chapter 23.

**Pipe** — Connecting one command's output to another's input with `|`. See Chapter 15.

**PipeWire** — The audio and video server on Ubuntu 26.04, replacing PulseAudio. See Chapter 11.

**Plex** — A media server. Various alternatives exist on Linux including Jellyfin (open source).

**PPA (Personal Package Archive)** — A third-party APT repository hosted on Launchpad. See Chapter 18.

**Process** — A running instance of a program. Each has a PID, parent, and state. See Chapter 23.

**Prompt** — The text shown before your cursor in the terminal, typically `user@host:~$`. Customizable via `$PS1`. See Chapter 19.

**Proton** — Valve's Wine-based compatibility layer for running Windows games on Linux through Steam. See Chapter 28.

**ProtonDB** — A community database rating Windows game compatibility on Linux. See Chapter 28.

**PTY (pseudo-terminal)** — A virtual terminal that programs like terminal emulators use to communicate with the shell.

**Ptyxis** — Ubuntu 26.04's default terminal emulator, replacing GNOME Terminal. See Chapter 9.

**PulseAudio** — The previous audio server on Ubuntu. Replaced by PipeWire in recent releases.

**Purge** — Removing a package along with its configuration files: `apt purge`. See Chapter 18.

**RCS (Revision Control System)** — An early version control system, superseded by Git and others.

**Recovery mode** — A GRUB boot option providing a minimal repair environment. See Chapter 29.

**Regex (Regular Expression)** — A pattern-matching language for text. Used by `grep`, `sed`, and many other tools. See Chapter 20.

**Repositories (repos)** — Online servers hosting packages. APT downloads from these. See Chapter 18.

**Resolvectl** — The systemd DNS resolution query tool. See Chapter 21.

**root** — 1. The system administrator account (UID 0). 2. The top of the filesystem (`/`). See Chapter 17.

**RPM** — A package format used by Fedora, Red Hat, openSUSE, and others. Not used by Ubuntu.

**Rsync** — A fast, incremental file-copying tool. The backbone of many backup solutions. See Chapter 26.

**Runlevel** — A legacy SysV init concept. Replaced by systemd targets (graphical.target, multi-user.target).

**Sandbox** — An isolated environment restricting what a program can access. Snaps, Flatpaks, and AppArmor all provide sandboxing.

**SATA** — A common disk interface. Being replaced by NVMe for SSDs.

**Script** — A text file containing a series of shell commands, executed as a program. See Chapters 19 and 24.

**SHA-256** — A cryptographic hash function used to verify file integrity. See Chapter 5.

**Shebang** — The `#!/bin/bash` line at the top of a script telling the system which interpreter to use. See Chapter 19.

**Shell** — The command-line interpreter (Bash on Ubuntu). Takes typed commands and executes them. See Chapter 13.

**SIGKILL (signal 9)** — Immediate, unavoidable process termination. Cannot be caught. Last resort. See Chapter 23.

**Signal** — A notification sent to a process. Common signals: SIGTERM (15), SIGKILL (9), SIGHUP (1), SIGINT (2). See Chapter 23.

**Snap** — Canonical's package format with sandboxing and auto-updates. See Chapter 18.

**Snapd** — The background service managing snaps.

**Socket** — A communication endpoint for network or inter-process communication. Listed with `ss`. See Chapter 21.

**Software & Updates** — A legacy settings tool removed from Ubuntu 26.04's default install. Reinstallable via `software-properties-gtk`. See Chapter 9.

**SSH (Secure Shell)** — An encrypted protocol for remote access. Uses key-based authentication. See Chapter 27.

**Standard error (stderr)** — Stream 2, where error messages go. Redirected with `2>` or `2>&1`. See Chapter 15.

**Standard input (stdin)** — Stream 0, where a program reads input. Redirected with `<`. See Chapter 15.

**Standard output (stdout)** — Stream 1, where a program writes normal output. Redirected with `>` or `|`. See Chapter 15.

**Sticky bit** — A special permission on directories (like `/tmp`) ensuring only the file owner can delete their own files, even if the directory is world-writable. See Chapter 16.

**Subshell** — A child shell process, created with `( )` or command substitution `$( )`.

**sudo** — Temporarily elevates privileges to root for a single command. Stands for "SuperUser DO." Written in Rust on Ubuntu 26.04. See Chapter 17.

**Swap** — Disk or ZRAM space used when RAM is full. Slower than RAM but prevents crashes. See Chapter 30.

**Symbolic link (symlink)** — A file pointing to another file's path. Created with `ln -s`. See Chapter 15.

**sysctl** — A tool for reading and setting kernel parameters at runtime. See Chapter 27.

**systemd** — The init system and service manager on Ubuntu. Runs as PID 1. See Chapter 23.

**systemctl** — The command for managing systemd services. See Chapter 23.

**Tarball** — A tar archive, often compressed (`.tar.gz`, `.tar.xz`). See Chapter 26.

**Terminal emulator** — A graphical program providing a text terminal. Ptyxis on Ubuntu 26.04. See Chapter 9.

**Thread** — A lightweight process sharing memory with its parent. Multi-threaded programs (like Firefox) use many threads.

**Timeshift** — A system snapshot tool for rolling back the OS to a previous state. See Chapter 26.

**TPM (Trusted Platform Module)** — A hardware chip storing encryption keys. Ubuntu 26.04 supports TPM-backed full disk encryption. See Chapter 6.

**TTY (teletypewriter)** — A text-only terminal. Ubuntu provides six TTYs accessible via `Ctrl+Alt+F3`–F6. See Chapter 29.

**UFW (Uncomplicated Firewall)** — Ubuntu's simplified firewall management tool. See Chapter 27.

**UID (User ID)** — A numeric identifier for a user account. Root is UID 0. See Chapter 17.

**umask** — The default permission mask applied to newly created files. See Chapter 16.

**Unix** — The operating system family Linux is modeled after. See Chapter 1.

**Update** — Installing newer versions of installed packages. Contrast with "upgrade" (moving to a new OS release). See Chapter 18.

**Upterm** — See Ptyxis (terminal evolution).

**Userspace** — The part of the system running normal applications, outside the kernel.

**Variables** — Named storage in the shell. Set with `=`, accessed with `$`. See Chapter 19.

**Vim** — A modal text editor. Powerful with a steep learning curve. See Chapter 19.

**Virtual terminal** — See TTY.

**Wayland** — The display server protocol on Ubuntu 26.04 (the only option). Replaces X11 with better security and isolation. See Chapter 8.

**Window manager** — The component that draws window borders, handles dragging, resizing, and focus. Mutter is GNOME's window manager.

**Wine** — A compatibility layer allowing many Windows applications to run on Linux. See Chapter 28.

**WSL (Windows Subsystem for Linux)** — A Microsoft-provided Linux environment running inside Windows. See Chapter 5.

**X11** — The previous display server protocol. Replaced by Wayland on Ubuntu 26.04. See Chapter 8.

**x86-64** — The 64-bit CPU architecture used by most desktop and laptop computers. Also called `amd64`. Ubuntu 26.04 uses `x86-64-v3` optimized packages.

**Yaru** — Ubuntu's default visual theme (orange/purple accent colors). See Chapter 25.

**Zombie process** — A process that has finished but whose parent hasn't collected its exit code. Harmless but indicates a parent bug. See Chapter 23.

**ZRAM** — A compressed swap device in RAM. Default on Ubuntu 26.04. Faster than disk swap. See Chapter 30.

**ZSH** — An alternative shell to Bash. Not installed by default on Ubuntu.
