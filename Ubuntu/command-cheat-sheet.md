# Appendix B: Command Cheat Sheet

Commands organized by category. Each entry shows the command and a one-line description.

---

## Navigation and Filesystem

| Command | Description |
|---|---|
| `pwd` | Print current directory |
| `ls` | List directory contents |
| `ls -la` | List all files with details (including hidden) |
| `ls -lh` | List with human-readable file sizes |
| `ls -lt` | List sorted by modification time (newest first) |
| `cd /path` | Change to directory |
| `cd ~` or `cd` | Go to home directory |
| `cd ..` | Go up one directory |
| `cd -` | Go to previous directory |
| `pushd /path` | Save current dir and switch to new one |
| `popd` | Return to saved directory |
| `tree` | Show directory tree (install with `apt install tree`) |
| `realink` | Resolve symlinks to their targets |
| `basename /path/file.txt` | Extract filename from path |
| `dirname /path/file.txt` | Extract directory from path |
| `stat file` | Show detailed file metadata |
| `df -h` | Show disk space on all filesystems |
| `du -sh /path` | Show total size of a directory |
| `du -sh /path/*` | Show sizes of all items in a directory |
| `ncdu /` | Interactive disk usage explorer |
| `mount /dev/sdX /mnt` | Mount a device |
| `umount /mnt` | Unmount a device |

## File and Directory Operations

| Command | Description |
|---|---|
| `touch file` | Create empty file or update timestamp |
| `mkdir dir` | Create directory |
| `mkdir -p a/b/c` | Create nested directories |
| `cp src dst` | Copy file |
| `cp -r src dst` | Copy directory recursively |
| `cp -a src dst` | Copy preserving all attributes |
| `mv src dst` | Move or rename file |
| `rm file` | Remove file |
| `rm -r dir` | Remove directory recursively |
| `rm -rf dir` | Force-remove directory (careful!) |
| `ln -s target linkname` | Create symbolic link |
| `ln target linkname` | Create hard link |
| `cat file` | Display file contents |
| `cat > file` | Write typed input to file (Ctrl+D to end) |
| `tac file` | Display file in reverse line order |
| `head -n 20 file` | Show first 20 lines |
| `tail -n 20 file` | Show last 20 lines |
| `tail -f file` | Follow file in real-time (like logs) |
| `less file` | View file interactively (scroll, search) |
| `wc -l file` | Count lines in file |
| `wc -w file` | Count words in file |
| `split -l 1000 file prefix` | Split file into 1000-line chunks |
| `diff file1 file2` | Show differences between files |
| `file filename` | Detect file type |
| `xxd file` | Hex dump of a file |

## Permissions and Ownership

| Command | Description |
|---|---|
| `chmod 755 file` | Set permissions (rwxr-xr-x) |
| `chmod u+x file` | Add execute for owner |
| `chmod g-w file` | Remove write for group |
| `chmod a+r file` | Add read for all |
| `chmod -R 644 dir` | Set permissions recursively |
| `chown user file` | Change file owner |
| `chown user:group file` | Change owner and group |
| `chgrp group file` | Change group ownership |
| `umask 022` | Set default permission mask |
| `getfacl file` | Show ACL permissions |
| `setfacl -m u:user:rwx file` | Set ACL entry for a user |
| `ls -ld dir` | Show directory permissions (not contents) |
| `sudo -l` | List sudo privileges for current user |

## Users, Groups, and Sudo

| Command | Description |
|---|---|
| `whoami` | Print current username |
| `id` | Show UID, GID, and groups |
| `who` | Show logged-in users |
| `w` | Show who is logged in and what they're doing |
| `last` | Show login history |
| `sudo command` | Run command as root |
| `sudo -i` | Open a root shell |
| `sudo -k` | Forget cached sudo password |
| `useradd -m -s /bin/bash username` | Create a new user with home dir |
| `userdel -r username` | Remove user and their home directory |
| `passwd username` | Set/change a user's password |
| `usermod -aG group username` | Add user to a group (append) |
| `usermod -L username` | Lock a user account |
| `usermod -U username` | Unlock a user account |
| `groupadd groupname` | Create a new group |
| `groupdel groupname` | Delete a group |
| `gpasswd groupname` | Administer group membership |
| `su - username` | Switch to another user |

## Process Management

| Command | Description |
|---|---|
| `ps aux` | List all running processes |
| `ps -ef --forest` | Show process tree |
| `ps -eo pid,ppid,%cpu,%mem,cmd` | Custom process listing |
| `pgrep processname` | Find PID by name |
| `pgrep -a processname` | Find PID with full command line |
| `pstree` | Show process tree visually |
| `pstree -p` | Process tree with PIDs |
| `top` | Real-time process monitor (classic) |
| `btop` | Modern real-time monitor (install with apt) |
| `jobs` | List background jobs in current shell |
| `command &` | Run command in background |
| `Ctrl+Z` | Suspend foreground process |
| `bg` | Resume suspended job in background |
| `fg` | Bring background job to foreground |
| `fg %1` | Bring specific job to foreground |
| `kill PID` | Send SIGTERM to process |
| `kill -9 PID` | Force kill (SIGKILL) |
| `kill -l` | List all signal names and numbers |
| `pkill processname` | Kill by name |
| `killall processname` | Kill all processes by exact name |
| `nice -n 10 command` | Start with low priority |
| `renice -n 5 -p PID` | Change priority of running process |
| `nohup command &` | Run surviving terminal close |
| `disown %1` | Detach job from shell |
| `systemd-cgls` | Show cgroup process tree |
| `uptime` | Show system uptime and load average |

## Package Management (APT)

| Command | Description |
|---|---|
| `sudo apt update` | Refresh package lists from repos |
| `sudo apt upgrade` | Upgrade all installed packages |
| `sudo apt full-upgrade` | Upgrade with dependency changes |
| `sudo apt install pkg` | Install a package |
| `sudo apt install -y pkg` | Install without confirmation |
| `sudo apt remove pkg` | Remove a package (keep config) |
| `sudo apt purge pkg` | Remove package and config files |
| `sudo apt autoremove` | Remove unused dependencies |
| `sudo apt autoremove --purge` | Remove unused deps and their configs |
| `sudo apt clean` | Clear cached .deb files |
| `sudo apt autoclean` | Clear only obsolete cached packages |
| `apt search keyword` | Search for packages |
| `apt show pkg` | Show package details |
| `apt list --installed` | List all installed packages |
| `apt list --upgradable` | List packages with updates available |
| `apt depends pkg` | Show package dependencies |
| `apt rdepends pkg` | Show what depends on this package |
| `apt why pkg` | Show why a package is installed |
| `apt why-not pkg` | Show why a package can't be installed |
| `apt hold pkg` | Prevent a package from being upgraded |
| `apt unhold pkg` | Allow a held package to upgrade |
| `sudo dpkg -i file.deb` | Install a local .deb file |
| `sudo dpkg --configure -a` | Fix interrupted package configuration |
| `sudo apt --fix-broken install` | Fix broken dependencies |
| `sudo add-apt-repository ppa:name/ppa` | Add a PPA |
| `sudo apt edit-sources` | Edit APT sources in editor |
| `sudo apt-mark showhold` | List held packages |

## Package Management (Snap)

| Command | Description |
|---|---|
| `sudo snap install name` | Install a snap |
| `sudo snap install name --classic` | Install with classic confinement |
| `sudo snap remove name` | Remove a snap |
| `snap list` | List installed snaps |
| `sudo snap refresh` | Update all snaps |
| `sudo snap refresh name` | Update a specific snap |
| `snap info name` | Show snap details and channels |
| `sudo snap switch name --channel=edge` | Switch to a different channel |
| `snap changes` | Show recent snap changes |
| `sudo snap revert name` | Revert to previous revision |
| `sudo snap set system refresh.retain=2` | Keep only 2 revisions |
| `sudo snap connect snap:plug snap:slot` | Manually connect interfaces |

## Package Management (Flatpak)

| Command | Description |
|---|---|
| `flatpak install flathub name` | Install from Flathub |
| `flatpak uninstall name` | Remove a Flatpak |
| `flatpak list` | List installed Flatpaks |
| `flatpak update` | Update all Flatpaks |
| `flatpak run name` | Run a Flatpak |
| `flatpak info name` | Show Flatpak details |
| `flatpak remote-list` | List configured repositories |
| `flatpak permission-list` | Show Flatpak permissions |
| `flatpak permission-remove name perm` | Remove a Flatpak permission |

## Networking

| Command | Description |
|---|---|
| `ip addr` | Show all IP addresses |
| `ip -br a` | Brief network interface summary |
| `ip link` | Show network interfaces |
| `ip link set dev eth0 up` | Bring an interface up |
| `ip route` | Show routing table |
| `nmcli device status` | List network devices |
| `nmcli connection show` | List saved connections |
| `nmcli device wifi list` | List available Wi-Fi networks |
| `nmcli device wifi connect SSID password PASS` | Connect to Wi-Fi |
| `nmtui` | Text-based network configuration UI |
| `ping -c 4 host` | Send 4 ping packets |
| `hostname` | Show system hostname |
| `hostnamectl set-hostname name` | Set hostname permanently |
| `ss -tlnp` | List listening TCP ports with processes |
| `ss -ulnp` | List listening UDP ports |
| `ss -tunp` | List all TCP/UDP connections |
| `resolvectl status` | Show DNS resolver status |
| `dig domain` | DNS lookup |
| `host domain` | Simple DNS lookup |
| `curl -O URL` | Download a file |
| `curl -s URL` | Download silently (no progress bar) |
| `wget URL` | Download a file |
| `scp file user@host:/path` | Copy file to remote over SSH |
| `scp user@host:/path file` | Copy file from remote |
| `rsync -aAXHv src/ dst/` | Sync directories |
| `rsync -aAXHvz src/ user@host:dst/` | Sync over SSH with compression |
| `sudo ufw status` | Show firewall status |
| `sudo ufw enable` | Enable firewall |
| `sudo ufw allow 80/tcp` | Allow port 80 |
| `sudo ufw limit ssh` | Rate-limit SSH |
| `sudo ufw delete allow 80/tcp` | Remove a rule |
| `sudo ufw status numbered` | List rules with numbers |
| `sudo ufw reset` | Reset firewall to defaults |

## Searching and Finding

| Command | Description |
|---|---|
| `find /path -name "*.txt"` | Find files by name pattern |
| `find /path -type f -size +100M` | Find files larger than 100 MB |
| `find /path -mtime -7` | Find files modified in last 7 days |
| `find /path -user john` | Find files owned by a user |
| `find /path -perm 644` | Find files with specific permissions |
| `find /path -exec cmd {} \;` | Run command on each result |
| `find /path -delete` | Delete all matching files |
| `grep "pattern" file` | Search for text in a file |
| `grep -r "pattern" /path` | Recursive search |
| `grep -i "pattern" file` | Case-insensitive search |
| `grep -n "pattern" file` | Show line numbers |
| `grep -v "pattern" file` | Inverted match (lines NOT matching) |
| `grep -c "pattern" file` | Count matching lines |
| `grep -l "pattern" *` | List files containing the pattern |
| `grep -A2 -B2 "pattern" file` | Show 2 lines after and before match |
| `grep -E "pat1|pat2" file` | Match multiple patterns (extended regex) |
| `plocate filename` | Fast filename search (updated database) |
| `updatedb` | Update the locate database |
| `which command` | Show path to a command |
| `whereis command` | Show binary, source, and man page locations |
| `type command` | Show how a command is interpreted |
| `fd pattern` | Modern find alternative (fdfind on Ubuntu) |
| `rg pattern` | Modern recursive grep (ripgrep) |
| `fzf` | Fuzzy finder for interactive filtering |

## Text Editing

| Command | Description |
|---|---|
| `nano file` | Open in nano (simple editor) |
| `vim file` | Open in Vim (modal editor) |
| `vimtutor` | Vim interactive tutorial |
| `select-editor` | Choose default text editor |
| `sed 's/old/new/g' file` | Replace text in file (stream edit) |
| `sed -i 's/old/new/g' file` | Replace text in file (in-place) |
| `awk '{print $1}' file` | Print first column of each line |
| `cut -d: -f1 /etc/passwd` | Extract first field (colon-delimited) |
| `sort file` | Sort lines alphabetically |
| `sort -rn file` | Sort numerically, descending |
| `uniq` | Remove duplicate consecutive lines |
| `tr 'a-z' 'A-Z' < file` | Convert lowercase to uppercase |
| `column -t file` | Align text into columns |
| `paste file1 file2` | Merge files line by line |
| `expand file` | Convert tabs to spaces |
| `unexpand -a file` | Convert spaces to tabs |

## System Information

| Command | Description |
|---|---|
| `uname -a` | Show kernel and system info |
| `uname -r` | Show kernel version |
| `lsb_release -a` | Show Ubuntu version details |
| `hostnamectl` | Show system identity |
| `lscpu` | Show CPU information |
| `lspci -k` | Show PCI devices with drivers |
| `lsusb` | Show USB devices |
| `lsblk -f` | Show block devices with filesystems |
| `lshw -short` | Brief hardware summary |
| `lsmem` | Show memory block information |
| `inxi -F` | Comprehensive system info (install with apt) |
| `dmidecode -t system` | Show hardware (DMI/SMBIOS) info |
| `lsmod` | List loaded kernel modules |
| `modinfo modulename` | Show module info |
| `dmesg` | Show kernel ring buffer |
| `free -h` | Show memory usage |
| `df -h` | Show disk space |
| `du -sh /path` | Show directory size |
| `sensors` | Show temperature sensors |
| `nproc` | Show number of CPU cores |
| `cat /etc/os-release` | Show OS identification |
| `cat /proc/cpuinfo` | Detailed CPU information |
| `cat /proc/meminfo` | Detailed memory information |
| `cat /proc/version` | Kernel version string |

## Systemd and Services

| Command | Description |
|---|---|
| `systemctl status service` | Check service status |
| `sudo systemctl start service` | Start a service |
| `sudo systemctl stop service` | Stop a service |
| `sudo systemctl restart service` | Restart a service |
| `sudo systemctl reload service` | Reload config without full restart |
| `sudo systemctl enable service` | Start at boot |
| `sudo systemctl disable service` | Don't start at boot |
| `sudo systemctl mask service` | Stronger than disable (cannot be started) |
| `sudo systemctl unmask service` | Undo a mask |
| `systemctl is-enabled service` | Check if enabled at boot |
| `systemctl is-active service` | Check if currently running |
| `systemctl list-units --type=service` | List all services |
| `systemctl list-unit-files --type=service` | List all service files |
| `systemctl --failed` | List failed services |
| `systemctl list-timers` | List active timers |
| `systemctl list-jobs` | Show pending systemd jobs |
| `systemd-analyze` | Show boot time breakdown |
| `systemd-analyze blame` | Show slowest boot services |
| `systemd-analyze critical-chain` | Show boot dependency chain |
| `systemd-cgls` | Show process tree by cgroups |
| `journalctl -b` | Show current boot logs |
| `journalctl -b -1` | Show previous boot logs |
| `journalctl -b -p err` | Show current boot errors only |
| `journalctl -u service` | Show logs for a specific service |
| `journalctl -f` | Follow logs in real-time |
| `journalctl --since today` | Show logs from today |
| `journalctl --disk-usage` | Show journal disk space |
| `sudo journalctl --vacuum-size=500M` | Trim journal to 500 MB |
| `sudo journalctl --vacuum-time=30d` | Remove logs older than 30 days |

## Logs and Diagnostics

| Command | Description |
|---|---|
| `journalctl -b -p err` | Current boot errors |
| `journalctl -u service -b` | Service logs for current boot |
| `journalctl -f -u service` | Follow a service's logs live |
| `dmesg` | Kernel ring buffer |
| `dmesg --level=err,warn` | Kernel errors and warnings only |
| `dmesg -w` | Follow kernel messages live |
| `cat /var/log/apt/history.log` | APT installation history |
| `cat /var/log/auth.log` | Authentication events |
| `sudo logrotate -d /etc/logrotate.conf` | Test log rotation (dry run) |
| `sudo smartctl -a /dev/sdX` | SMART disk health details |
| `sudo smartctl -H /dev/sdX` | Quick disk health check |
| `sudo smartctl -t short /dev/sdX` | Run short disk self-test |
| `sudo apparmor_status` or `sudo aa-status` | AppArmor status |
| `sudo dmesg \| grep apparmor \| grep DENIED` | Show AppArmor denials |
| `sudo fail2ban-client status` | Fail2Ban status |
| `sudo fail2ban-client status sshd` | SSH jail status |

## GNOME Customization (gsettings)

| Command | Description |
|---|---|
| `gsettings get schema key` | Read a setting |
| `gsettings set schema key value` | Change a setting |
| `gsettings reset schema key` | Reset to default |
| `gsettings list-keys schema` | List all keys in a schema |
| `gsettings list-recursively schema` | Show all values in a schema |
| `gsettings list-schemas \| grep desktop` | Find schemas |
| `gnome-extensions list` | List installed extensions |
| `gnome-extensions enable uuid` | Enable an extension |
| `gnome-extensions disable uuid` | Disable an extension |
| `gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'` | Enable dark mode |
| `gsettings set org.gnome.shell.extensions.dash-to-dock dock-position 'BOTTOM'` | Move dock to bottom |
| `gsettings set org.gnome.desktop.interface clock-show-seconds true` | Show clock seconds |
| `gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true` | Enable tap-to-click |

## SSH and Security

| Command | Description |
|---|---|
| `ssh user@host` | Connect to remote machine |
| `ssh -p 2222 user@host` | Connect on non-default port |
| `ssh-copy-id user@host` | Copy public key to remote |
| `ssh-keygen -t ed25519 -C "email"` | Generate Ed25519 key pair |
| `ssh-keygen -t rsa -b 4096 -C "email"` | Generate RSA 4096-bit key |
| `ssh-add ~/.ssh/key` | Add key to SSH agent |
| `eval "$(ssh-agent -s)"` | Start SSH agent |
| `ssh-keygen -lf ~/.ssh/key.pub` | Show key fingerprint |
| `sudo systemctl restart sshd` | Restart SSH daemon |
| `sudo ufw status verbose` | Detailed firewall status |
| `sudo ufw allow ssh` | Allow SSH through firewall |
| `sudo ufw limit ssh` | Rate-limit SSH |
| `sudo ufw allow from 192.168.1.0/24` | Allow from a subnet |
| `chmod 700 ~/.ssh` | Set SSH directory permissions |
| `chmod 600 ~/.ssh/id_ed25519` | Set private key permissions |
| `sudo sysctl -a \| grep security` | View security kernel params |
| `sudo sysctl -w param=value` | Set sysctl temporarily |
| `sudo sysctl --system` | Apply sysctl config files |

## Backup and Recovery

| Command | Description |
|---|---|
| `sudo timeshift --create --comments "msg"` | Create system snapshot |
| `sudo timeshift --list` | List snapshots |
| `sudo timeshift --restore` | Restore a snapshot |
| `sudo timeshift --delete --snapshot 'name'` | Delete a snapshot |
| `deja-dup --backup` | Run Deja Dup backup |
| `deja-dup --restore` | Restore from Deja Dup |
| `rsync -aAXHv --delete src/ dst/` | Mirror directories |
| `rsync -aAXHv --dry-run src/ dst/` | Preview what would sync |
| `tar -czvpf archive.tar.gz dir/` | Create compressed archive |
| `tar -xzvpf archive.tar.gz -C /path/` | Extract archive |
| `tar -tzf archive.tar.gz` | List archive contents |
| `tar -czv dir/ \| gpg -c -o backup.gpg` | Create encrypted archive |
| `borg init --encryption=repokey /path` | Initialize Borg repository |
| `borg create --progress REPO::name /src` | Create Borg archive |
| `borg list REPO` | List Borg archives |
| `borg mount REPO::name /mnt` | Mount Borg archive |
| `borg extract REPO::name` | Extract Borg archive |
| `borg prune --keep-daily 7 REPO` | Prune old archives |
| `borg check --verify-data REPO` | Verify repository integrity |

## Shell Scripting

| Command / Syntax | Description |
|---|---|
| `#!/bin/bash` | Shebang for Bash scripts |
| `set -euo pipefail` | Safe scripting header |
| `variable="value"` | Assign a variable |
| `${var:-default}` | Use default if empty |
| `${var##*/}` | Remove longest front match (basename) |
| `${var%/*}` | Remove shortest end match (dirname) |
| `${var/old/new}` | Replace first occurrence |
| `${var//old/new}` | Replace all occurrences |
| `${var^^}` | Uppercase entire string |
| `${var,,}` | Lowercase entire string |
| `${#var}` | String length |
| `${arr[@]}` | All array elements |
| `${#arr[@]}` | Array length |
| `declare -A arr` | Declare associative array |
| `mapfile -t arr < <(command)` | Read command output into array |
| `trap func EXIT` | Run function on script exit |
| `trap 'echo $LINENO' ERR` | Print line on error |
| `command1 \| command2` | Pipe output to input |
| `command > file` | Redirect stdout to file |
| `command 2>&1` | Redirect stderr to stdout |
| `command >> file` | Append stdout to file |
| `command < file` | Read file as stdin |
| `command << EOF ... EOF` | Here document |
| `command <<< "string"` | Here string |
| `cmd1 && cmd2` | Run cmd2 only if cmd1 succeeds |
| `cmd1 \|\| cmd2` | Run cmd2 only if cmd1 fails |
| `crontab -e` | Edit cron jobs |
| `crontab -l` | List cron jobs |
| `crontab -r` | Remove all cron jobs |
| `chmod +x script.sh` | Make script executable |
| `bash -n script.sh` | Check syntax without executing |
| `shellcheck script.sh` | Lint a script (install shellcheck) |

## Maintenance and Monitoring

| Command | Description |
|---|---|
| `sudo apt update && sudo apt upgrade -y` | Update all packages |
| `sudo apt autoremove --purge -y` | Remove unused packages |
| `sudo apt clean` | Clear APT cache |
| `sudo snap refresh` | Update all snaps |
| `flatpak update -y` | Update all flatpaks |
| `sudo journalctl --vacuum-size=500M` | Trim journal |
| `sudo update-initramfs -u` | Rebuild initramfs |
| `sudo update-grub` | Update GRUB bootloader |
| `free -h` | Memory summary |
| `vmstat 5` | Virtual memory stats (every 5s) |
| `iostat -x 5` | Disk I/O stats |
| `uptime` | System load average |
| `sensors` | Temperature readings |
| `sudo smartctl -H /dev/sdX` | Disk health check |
| `swapon --show` | Swap device status |
| `sudo zramctl` | ZRAM status |
| `ncdu /` | Interactive disk usage |
| `systemctl --failed` | List failed services |
| `apt list --upgradable` | List pending updates |
| `dpkg --list 'linux-image-*' \| grep ^ii` | List installed kernels |
| `uname -r` | Current kernel version |
