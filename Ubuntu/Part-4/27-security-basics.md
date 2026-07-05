# Chapter 27: Security Basics

## Linux Is Secure — But Not Automatically

Linux has a well-deserved reputation for security. Its permission system, built-in firewall framework, mandatory access control, and rapid patching cycle make it fundamentally harder to compromise than many alternatives. But "harder to compromise" isn't the same as "immune." Security is a practice, not a feature.

On a desktop Ubuntu 26.04 system, most security basics are handled for you — firewall, AppArmor, automatic updates, encryption options. This chapter covers what's already protecting you, what you should configure yourself, and the habits that keep your system safe over the long term.

The good news: you've already done several of these steps in Chapter 7's post-install checklist. Now you'll understand *why* each step matters and learn the deeper configuration.

## Ubuntu 26.04's Security Defaults

Ubuntu 26.04 ships with significant security enhancements built in. Here's what's already working before you touch anything:

| Feature | Status | What It Does |
|---|---|---|
| **AppArmor** | Enabled by default | Constrains what programs can do — even if compromised |
| **UFW firewall** | Installed, needs enabling | Blocks unwanted incoming network connections |
| **Automatic security updates** | Enabled by default | Installs security patches without user intervention |
| **Kernel hardening (sysctl)** | Ubuntu-tuned defaults | ASLR, dmesg restriction, symlink protection |
| **Post-quantum crypto** | Default in 26.04 | Quantum-resistant algorithms in kernel, OpenSSL 3.5, OpenSSH 10.2 |
| **Wayland** | Default (only option) | Isolates applications — one app can't keylog another |
| **Snap confinement** | Enforced | Sandboxed applications with restricted access |

### The Security Center App

Ubuntu 26.04 includes a **Security Center** app — a graphical dashboard for reviewing and managing your system's security posture. Open it by pressing Super, typing "Security," and pressing Enter.

The Security Center shows:

- **Firewall status** — On/off toggle, managed through UFW
- **AppArmor status** — Enforcement mode, loaded profiles count
- **Kernel hardening** — Displays the active sysctl protections
- **Permission prompting** — Toggle for AppArmor permission prompts (apps asking for file access)

> ⚠️ **Known Issue: Permission Prompting**
>
> Ubuntu 26.04's new "permission prompting" feature (where apps ask permission before accessing files) has been linked to kernel panics and freezes on some systems, particularly when saving files from Firefox or snap-based browsers. If you experience crashes after enabling this feature, disable it in the Security Center or hold off until the point release (26.04.1) addresses the bug. This is tracked as a known issue in the Ubuntu desktop-security-center project.

## The Firewall: UFW

A **firewall** controls what network traffic can enter and leave your system. Think of it as a bouncer at the door of a building — most people are turned away, but those on the guest list get in.

**UFW** (Uncomplicated Firewall) is Ubuntu's default firewall management tool. It's a simplified front-end for `iptables`/`nftables` — the kernel-level packet filtering framework.

### Checking Firewall Status

sudo ufw status verbose

Output (when enabled):

Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

If the status shows inactive, your firewall is off. Enable it:

sudo ufw enable

Setting Default Policies

The fundamental firewall principle: deny everything incoming, allow everything outgoing. You explicitly open ports only for services that need them.

sudo ufw default deny incoming
sudo ufw default allow outgoing

These commands set the baseline. Incoming connections are blocked by default — no one can reach your system from the network unless you've allowed it. Outgoing connections are allowed — your web browser, email client, and updates work normally.
Allowing Specific Services

Allow SSH (port 22):

sudo ufw allow ssh

Or by port number:

sudo ufw allow 22/tcp

Allow a web server (port 80 and 443):

sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

Allow a specific port range:

sudo ufw allow 8000:8100/tcp

Allow from a specific IP address only:

sudo ufw allow from 192.168.1.50

Allow SSH from your local network only:

sudo ufw allow from 192.168.1.0/24 to any port 22

This is much safer than allowing SSH from anywhere — only devices on your home network can attempt to connect.
Rate Limiting (Brute Force Protection)

UFW can limit connection attempts to prevent brute-force attacks:

sudo ufw limit ssh

This denies connections from an IP address that attempts 6 or more connections within 30 seconds. Replace allow ssh with limit ssh for any service exposed to the internet.
Deleting Rules

sudo ufw delete allow 80/tcp

Or use numbered rules:

sudo ufw status numbered

Status: active
     To                         Action      From
     --                         ------      ----
[ 1] 22/tcp                     ALLOW IN    Anywhere
[ 2] 80/tcp                     ALLOW IN    Anywhere
[ 3] 443/tcp                    ALLOW IN    Anywhere

Delete by number:

sudo ufw delete 2

Disabling and Resetting

Temporarily disable (keeps rules):

sudo ufw disable

Reset to factory defaults (removes all rules):

sudo ufw reset

UFW GUI

If you prefer a graphical interface:

sudo apt install gufw

Open "Firewall Configuration" from the app grid. It provides toggles and simple rule creation with the same UFW backend.
Common UFW Rules for Desktop Users

Most desktop users need very few rules:

# Set defaults
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH if you remotely access this machine
sudo ufw limit ssh

# Enable the firewall
sudo ufw enable

A desktop behind a home router already has NAT protection — the router's firewall blocks incoming traffic before it reaches your computer. UFW adds a second layer, protecting you on public Wi-Fi or if the router is misconfigured.

    💡 UFW and Docker

    Docker bypasses UFW rules by manipulating iptables directly. If you run Docker containers with published ports, those ports are accessible regardless of UFW settings. This is a known Docker behavior. If you need Docker + firewall control, research ufw-docker or use Docker's network configurations to restrict access.

SSH Key Management

SSH (Secure Shell) is the protocol for securely accessing a remote machine. Even if you don't run a server, you'll use SSH to connect to remote systems, push code to Git repositories, or manage cloud instances.

SSH supports two authentication methods:

    Password — Type a password each time. Vulnerable to brute-force attacks.
    Key-based — Cryptographic key pair. No password transmitted. Far more secure.

Generating an SSH Key Pair

Ed25519 (recommended — faster, shorter, more secure):

ssh-keygen -t ed25519 -C "john@example.com"

RSA (legacy, larger key — use 4096 bits minimum):

ssh-keygen -t rsa -b 4096 -C "john@example.com"

The -C flag adds a comment (typically your email) to identify the key.

You'll be prompted:

Enter file in which to save the key (/home/john/.ssh/id_ed25519):

Press Enter to accept the default location.

Enter passphrase (empty for no passphrase):

A passphrase encrypts the private key on disk. If someone steals your private key file, they can't use it without the passphrase. Strongly recommended.
The Key Pair

After generation, two files appear in ~/.ssh/:
File	Purpose	Share?
id_ed25519	Private key — never leaves your machine	Never
id_ed25519.pub	Public key — placed on remote servers	Yes

The analogy: the public key is a padlock. You distribute copies of the padlock to anyone. The private key is the only key that opens those padlocks. You never share the private key with anyone.
Copying Your Public Key to a Remote Server

ssh-copy-id user@server.example.com

This appends your public key to the remote server's ~/.ssh/authorized_keys file. After this, you can SSH without a password (only the key passphrase, if you set one).

Manual method (if ssh-copy-id isn't available):

cat ~/.ssh/id_ed25519.pub | ssh user@server "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

Connecting with SSH

ssh user@server.example.com

If this is your first connection, you'll see:

The authenticity of host 'server.example.com (203.0.113.50)' can't be established.
ED25519 key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no)?

Type yes. The server's fingerprint is saved to ~/.ssh/known_hosts. On future connections, SSH verifies the fingerprint matches — if it changes, you'll get a warning (possible man-in-the-middle attack).
Disabling Password Authentication (Server Side)

Once key-based authentication works, disable passwords to eliminate brute-force attacks:

sudo nano /etc/ssh/sshd_config

Set:

PasswordAuthentication no
PubkeyAuthentication yes
PermitRootLogin prohibit-password

Restart SSH:

sudo systemctl restart sshd

    ⚠️ Test Before Closing Your Session

    After changing SSH settings, keep your current session open and test the new configuration from a new terminal:

    ssh user@localhost

    If you can connect, the new settings work. If you can't, you still have your original session to fix the problem. Closing your only session without testing is how people lock themselves out of servers.

SSH Agent: Remembering Your Passphrase

If you set a passphrase on your key, typing it every time is annoying. ssh-agent remembers it for your session:

eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

Enter your passphrase once. Until you log out or reboot, SSH connections use the cached key without prompting.

GNOME automatically starts an SSH agent on login — your key passphrase is prompted once and remembered for the session. You can verify:

echo $SSH_AUTH_SOCK

If this shows a path, the agent is running.
Managing Multiple SSH Keys

If you use different keys for different servers (work, personal, Git hosting), configure ~/.ssh/config:

# Work server
Host work
    HostName server.company.com
    User jsmith
    IdentityFile ~/.ssh/id_work

# Personal server
Host personal
    HostName homeserver.local
    User john
    IdentityFile ~/.ssh/id_ed25519

# GitHub
Host github.com
    User git
    IdentityFile ~/.ssh/id_github

Now you can connect with just the alias:

ssh work
ssh personal
ssh github.com

SSH Key Permissions

SSH refuses to use keys with wrong permissions — this is a security measure, not a bug:

chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
chmod 600 ~/.ssh/authorized_keys
chmod 600 ~/.ssh/config

If SSH complains about "permissions too open," run these commands.
AppArmor: Mandatory Access Control

AppArmor is Ubuntu's mandatory access control (MAC) system. It constrains what programs can do, even if they're compromised. Traditional file permissions (Chapter 16) say "who can access this file." AppArmor says "what this program can do, regardless of who runs it."
How AppArmor Works

An AppArmor profile defines exactly what files, capabilities, and network access a program is allowed. For example, Firefox's profile allows it to read its own configuration, write to your Downloads folder, and access the network — but it can't read /etc/shadow or write to system directories, even if a vulnerability gives an attacker control of the process.

If Firefox gets compromised through a malicious webpage, AppArmor limits the damage to what Firefox is allowed to do. The attacker can't escape the profile to access the rest of your system.
Checking AppArmor Status

sudo aa-status

Output:

apparmor module is loaded.
XX profiles are loaded.
YY profiles are enforces.
ZZ profiles are in complain mode.
X processes have profiles defined.
Y processes are in enforce mode.

Enforcing processes:
   /usr/bin/firefox (XXXX)
   /snap/snapd (XXXX)
   ...

Mode	Meaning
enforce	Profile actively blocks violations — the normal operating mode
complain	Profile logs violations but doesn't block them — for testing new profiles
unconfined	No profile — program runs unrestricted
Viewing Loaded Profiles

ls /etc/apparmor.d/

Each file is a profile. Programs with profiles include Firefox, snap applications, Evince, TCPDump, and many system services.
Switching Modes

Put a profile into complain mode (log only, don't block):

sudo aa-complain /etc/apparmor.d/usr.bin.firefox

Put a profile back into enforce mode:

sudo aa-enforce /etc/apparmor.d/usr.bin.firefox

You generally won't need to modify AppArmor profiles as a desktop user. The profiles shipped with Ubuntu are well-tested. But understanding the concept helps when troubleshooting — if an application behaves strangely (can't access a file it should), check if an AppArmor profile is blocking it:

sudo dmesg | grep apparmor | grep DENIED

This shows recent AppArmor denials — the log tells you what was blocked and why.
Fail2Ban: Brute-Force Protection

If you run any service exposed to the internet — SSH, a web server, a mail server — automated bots will try to guess passwords. Millions of brute-force attempts hit SSH servers worldwide every day.

Fail2Ban monitors log files for repeated failed authentication attempts and bans the offending IP address by adding a firewall rule. It's automated, self-healing protection.
Installing Fail2Ban

sudo apt install fail2ban

Fail2Ban starts automatically after installation.
Configuration

Fail2Ban's default configuration lives in /etc/fail2ban/jail.conf. Never edit this file directly — it gets overwritten on updates. Create a local override:

sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

Key settings:

[DEFAULT]
bantime  = 1h          # How long to ban (1 hour)
findtime = 10m         # Window for counting failures (10 minutes)
maxretry = 5           # Number of failures before banning

# Enable SSH protection
[sshd]
enabled = true

After editing, restart:

sudo systemctl restart fail2ban

Checking Fail2Ban Status

sudo fail2ban-client status

Status
|- Number of jail:  1
`- Jail list: sshd

Check a specific jail:

sudo fail2ban-client status sshd

Status for the jail: sshd
|- Filter
|  |- Currently failed: 0
|  |- Total failed:     12
|  `- File list:        /var/log/auth.log
`- Actions
   |- Currently banned: 0
   |- Total banned:     3
   `- Banned IP list:   

This shows 12 failed SSH attempts were detected, 3 IPs were banned, and none are currently banned (they've expired or been unbanned).
Unbanning an IP

sudo fail2ban-client unban 192.168.1.100

Or unban all:

sudo fail2ban-client unban --all

    💡 Fail2Ban vs. UFW Rate Limiting

    UFW's limit command provides basic rate limiting for SSH. Fail2Ban is more sophisticated — it parses log files, detects patterns across multiple services, and supports configurable thresholds. For a desktop system, UFW rate limiting is sufficient. For any system exposed to the public internet, install Fail2Ban as well.

Kernel Hardening (sysctl)

The Linux kernel has hundreds of security-related tunable parameters, controlled through sysctl. Ubuntu 26.04 ships with sensible defaults, but understanding them helps you verify your system's posture.
Viewing Current sysctl Values

sudo sysctl -a | grep security

Or check specific values:

sudo sysctl kernel.randomize_va_space

kernel.randomize_va_space = 2

Key Security Parameters
Parameter	Value	What It Does
kernel.randomize_va_space	2	Full ASLR — randomizes memory layout to prevent exploit prediction
kernel.dmesg_restrict	1	Only root can read kernel logs (prevents info leakage)
kernel.kptr_restrict	2	Hide kernel pointers from non-root (prevents kernel address exploits)
fs.protected_symlinks	1	Restrict symlink following (prevents classic symlink attacks)
fs.protected_hardlinks	1	Restrict hard link creation
fs.protected_fifos	2	Restrict FIFO access (Ubuntu 26.04 default)
fs.protected_regular	2	Restrict regular file creation in sticky directories
net.ipv4.conf.all.rp_filter	2	Reverse path filtering — blocks spoofed packets
net.ipv4.tcp_syncookies	1	SYN flood protection
kernel.stacktrace_randomization	1	Randomize kernel stack traces (new in recent kernels)
Modifying sysctl Values

Temporary change (lost on reboot):

sudo sysctl -w kernel.dmesg_restrict=1

Permanent change (survives reboot):

Create a file in /etc/sysctl.d/:

sudo nano /etc/sysctl.d/99-my-hardening.conf

Content:

kernel.randomize_va_space = 2
kernel.dmesg_restrict = 1
kernel.kptr_restrict = 2
fs.protected_symlinks = 1
fs.protected_hardlinks = 1
net.ipv4.tcp_syncookies = 1

Apply:

sudo sysctl --system

This reads and applies all sysctl configuration files. Your custom file loads after the defaults, so it overrides them.

    💡 Don't Over-Harden

    Ubuntu's defaults are well-chosen. Randomly tightening sysctl values can break applications, networking, or container runtimes. Only change parameters you understand and test thoroughly after each change. The defaults on 26.04 already represent a strong security posture.

Automatic Updates

Keeping your system patched is the single most effective security measure. Ubuntu 26.04 enables automatic security updates by default — but verify they're working.
Checking Automatic Update Status

cat /etc/apt/apt.conf.d/20auto-upgrades

Expected content:

APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";

Both values set to "1" mean automatic updates are enabled. "0" means disabled.
Enabling Automatic Updates

If disabled, re-enable:

sudo dpkg-reconfigure -plow unattended-upgrades

Select "Yes" when prompted.
What Gets Updated Automatically

By default, only packages from the security pocket of the Ubuntu archive are automatically installed. Regular updates (bug fixes, feature updates) still require manual sudo apt upgrade.

You can verify what's configured:

cat /etc/apt/apt.conf.d/50unattended-upgrades

Look for the Allowed-Origins section — it lists which repositories are eligible for automatic upgrades.
Reviewing Automatic Update Logs

cat /var/log/unattended-upgrades/unattended-upgrades.log | tail -30

Shows recent automatic update activity — what was installed, when, and any errors.
Disk Encryption

If you enabled TPM-backed Full Disk Encryption or LUKS during installation (Chapter 6), your data is encrypted at rest. If your laptop is stolen, the thief can't read your files without the decryption key.
Verifying Encryption Status

lsblk -f

Look for crypto_LUKS in the FSTYPE column:

NAME        FSTYPE            SIZE
nvme0n1
├─nvme0n1p1 vfat              512M
└─nvme0n1p2 crypto_LUKS       476G
  └─nvme0n1p2_crypt LVM_member 476G
    └─ubuntu--vg-root ext4    476G

If you see crypto_LUKS, your disk is encrypted. If you only see ext4 or btrfs directly, it's not encrypted.
LUKS vs. TPM-Backed FDE
Feature	LUKS (traditional)	TPM-Backed FDE (new in 26.04)
Passphrase	Required at boot	Key sealed in TPM — seamless boot
Security	Password-protects disk	Tied to hardware — can't move drive to another machine
Recovery	Passphrase recovery only	Recovery key required (keep it safe!)
Setup	During install or post-install	During install only

Both provide strong encryption at rest. TPM-backed FDE is more convenient (no boot password) but requires careful recovery key management — if the TPM is cleared or the motherboard is replaced, you need the recovery key to access your data.

    ⚠️ TPM Recovery Key

    If you chose TPM-backed FDE, your system generated a recovery key during installation. Save this key somewhere safe — printed, stored in a password manager, or written down in a secure location. Without it, a motherboard failure means permanent data loss.

Common-Sense Security Habits

Technical measures are only half the equation. Habits matter just as much:
1. Install Software from Trusted Sources
Source	Trust Level	Notes
Ubuntu repositories (apt)	High	Reviewed and signed by Canonical
Snap Store	Medium-High	Sandboxed, but quality varies
Flathub	Medium-High	Sandboxed, community-maintained
PPAs	Variable	Depends entirely on the PPA maintainer
Random .deb downloads	Low	No sandboxing, full system access
`curl	bash` scripts	Very low

The apt repositories and Snap/Flatpak are curated and sandboxed. PPAs and random downloads bypass these protections — only use them from developers you trust.
2. Use Strong, Unique Passwords

Your user password protects sudo access and the lock screen. Use a password manager (like Proton Pass, Bitwarden, or KeePassXC) to generate and store unique passwords. Never reuse passwords across services.
3. Lock Your Screen

Set an automatic lock timeout:

gsettings set org.gnome.desktop.session idle-delay 300

Locks the screen after 5 minutes (300 seconds) of inactivity. Lock manually with Super + L whenever you walk away.
4. Audit Running Services

Every running service is a potential attack surface. Check what's listening on your network:

sudo ss -tlnp

Output shows every program listening for incoming TCP connections. If you see something unexpected, investigate — it might be a legitimate service you forgot, or it might be something you want to disable.

Disable a service you don't need:

sudo systemctl disable --now avahi-daemon

(Avahi is mDNS/Bonjour service discovery — useful on some networks, unnecessary on others. Adjust based on your needs.)
5. Review SSH Access

If you don't need remote SSH access to your desktop, disable it:

sudo systemctl disable --now sshd

If you do need it, ensure it's configured with keys only and rate-limited or behind Fail2Ban.
6. Keep Backups

Security isn't just about preventing access — it's about recovering when things go wrong. Your Chapter 26 backup strategy is part of your security posture. If ransomware or a malicious script encrypts or deletes your files, your backups are your recovery path.
7. Be Careful with sudo

Every sudo command grants root access. Before running a command with sudo, ask yourself:

    Do I understand what this command does?
    Do I trust the source of this command?
    Could this command cause irreversible damage?

The sudo command is a tool, not a ritual. Understanding what you're authorizing is the difference between a power user and a hazard.
8. Browser Security

Your web browser is the most exposed application on your system. It downloads and executes code from the internet constantly. Best practices:

    Keep it updated (automatic updates handle this)
    Use an ad blocker (uBlock Origin)
    Don't install sketchy browser extensions
    Be cautious with downloads — scan with your system's tools if unsure
    Firefox on Ubuntu is confined by AppArmor and runs as a snap — this adds sandboxing on top of normal browser security

Security Audit Script

Run this script to get a quick security posture overview of your system:

#!/bin/bash
# security-audit.sh — Quick security posture check

echo "╔══════════════════════════════════════════╗"
echo "║       Ubuntu Security Audit Report       ║"
echo "╚══════════════════════════════════════════╝"
echo ""

echo "─── System Info ───"
echo "Hostname: $(hostname)"
echo "Kernel: $(uname -r)"
echo "Ubuntu: $(lsb_release -ds 2>/dev/null)"
echo "Uptime: $(uptime -p)"
echo ""

echo "─── Firewall (UFW) ───"
if command -v ufw > /dev/null 2>&1; then
    STATE=$(sudo ufw status | head -1)
    echo "Status: $STATE"
    if echo "$STATE" | grep -q "inactive"; then
        echo "⚠️  Firewall is DISABLED"
    else
        echo "✓ Firewall is enabled"
        sudo ufw status | tail -n +3
    fi
else
    echo "⚠️  UFW not installed"
fi
echo ""

echo "─── AppArmor ───"
if command -v aa-status > /dev/null 2>&1; then
    ENFORCED=$(sudo aa-status | grep "profiles are enforced" | head -1)
    echo "✓ $ENFORCED"
    COMPLAIN=$(sudo aa-status | grep "complain" | head -1)
    [ -n "$COMPLAIN" ] && echo "  $COMPLAIN"
else
    echo "⚠️  AppArmor tools not installed"
fi
echo ""

echo "─── SSH Configuration ───"
if systemctl is-active --quiet sshd 2>/dev/null; then
    echo "SSH daemon: ✓ running"
    PASSAUTH=$(grep -E "^PasswordAuthentication" /etc/ssh/sshd_config 2>/dev/null | awk '{print $2}')
    ROOTLOGIN=$(grep -E "^PermitRootLogin" /etc/ssh/sshd_config 2>/dev/null | awk '{print $2}')
    echo "  Password auth: ${PASSAUTH:-yes (default)}"
    echo "  Root login: ${ROOTLOGIN:-prohibit-password (default)}"
    if [ "${PASSAUTH:-yes}" = "yes" ]; then
        echo "  ⚠️  Password authentication enabled — consider disabling"
    fi
else
    echo "SSH daemon: not running (good for desktop)"
fi
echo ""

echo "─── Automatic Updates ───"
if [ -f /etc/apt/apt.conf.d/20auto-upgrades ]; then
    UNATTENDED=$(grep "Unattended-Upgrade" /etc/apt/apt.conf.d/20auto-upgrades | awk -F'"' '{print $2}')
    if [ "$UNATTENDED" = "1" ]; then
        echo "✓ Automatic security updates enabled"
    else
        echo "⚠️  Automatic security updates disabled"
    fi
else
    echo "⚠️  Auto-upgrades config not found"
fi
echo ""

echo "─── Disk Encryption ───"
if lsblk -o FSTYPE 2>/dev/null | grep -q "crypto_LUKS"; then
    echo "✓ Disk encryption (LUKS) detected"
else
    echo "⚠️  No disk encryption detected"
fi
echo ""

echo "─── Kernel Hardening ───"
ASLR=$(sysctl -n kernel.randomize_va_space 2>/dev/null)
DMESG=$(sysctl -n kernel.dmesg_restrict 2>/dev/null)
KPTR=$(sysctl -n kernel.kptr_restrict 2>/dev/null)
SYMLINK=$(sysctl -n fs.protected_symlinks 2>/dev/null)

echo "ASLR: $([ "$ASLR" = "2" ] && echo '✓ enabled' || echo '⚠️  not fully enabled')"
echo "dmesg restrict: $([ "$DMESG" = "1" ] && echo '✓ enabled' || echo '⚠️  disabled')"
echo "kptr restrict: $([ "$KPTR" = "2" ] && echo '✓ enabled' || echo '⚠️  not fully enabled')"
echo "Symlink protection: $([ "$SYMLINK" = "1" ] && echo '✓ enabled' || echo '⚠️  disabled')"
echo ""

echo "─── Listening Ports ───"
echo "Services listening for incoming connections:"
ss -tlnp 2>/dev/null | tail -n +2 | awk '{print "  " $4 " (" $6 ")"}' || echo "  (unable to query)"
echo ""

echo "─── Failed Login Attempts ───"
FAILED=$(grep "Failed password" /var/log/auth.log 2>/dev/null | wc -l)
if [ "$FAILED" -gt 0 ]; then
    echo "$FAILED failed login attempts logged"
else
    echo "No failed login attempts (or auth.log not available)"
fi
echo ""

echo "═══════════════════════════════════════════"
echo "Audit complete. Review any ⚠️ items above."
echo "═══════════════════════════════════════════"

Save as ~/bin/security-audit.sh, make executable, and run it periodically:

chmod +x ~/bin/security-audit.sh
./security-audit.sh

Security Cheat Sheet
Task	Command
Firewall	
Check status	sudo ufw status verbose
Enable firewall	sudo ufw enable
Set defaults	sudo ufw default deny incoming; sudo ufw default allow outgoing
Allow SSH	sudo ufw allow ssh
Rate-limit SSH	sudo ufw limit ssh
Allow port	sudo ufw allow 80/tcp
Allow from IP	sudo ufw allow from 192.168.1.0/24
Delete rule	sudo ufw delete allow 80/tcp
Numbered rules	sudo ufw status numbered
Reset	sudo ufw reset
SSH Keys	
Generate key	ssh-keygen -t ed25519 -C "email"
Copy to server	ssh-copy-id user@host
Set key perms	chmod 700 ~/.ssh; chmod 600 ~/.ssh/id_ed25519
Start agent	eval "$(ssh-agent -s)"; ssh-add
Check fingerprints	ssh-keygen -lf ~/.ssh/id_ed25519.pub
AppArmor	
Status	sudo aa-status
View denials	sudo dmesg | grep apparmor | grep DENIED
Complain mode	sudo aa-complain /etc/apparmor.d/profile
Enforce mode	sudo aa-enforce /etc/apparmor.d/profile
List profiles	ls /etc/apparmor.d/
Fail2Ban	
Install	sudo apt install fail2ban
Status	sudo fail2ban-client status
Jail status	sudo fail2ban-client status sshd
Unban IP	sudo fail2ban-client unban IP
Restart	sudo systemctl restart fail2ban
Kernel Hardening	
View all	sudo sysctl -a | grep security
Check ASLR	sysctl kernel.randomize_va_space
Set temporary	sudo sysctl -w kernel.dmesg_restrict=1
Set permanent	Edit /etc/sysctl.d/99-hardening.conf, run sudo sysctl --system
Updates	
Check auto-updates	cat /etc/apt/apt.conf.d/20auto-upgrades
Enable auto-updates	sudo dpkg-reconfigure -plow unattended-upgrades
Review update log	cat /var/log/unattended-upgrades/unattended-upgrades.log
Disk Encryption	
Check status	lsblk -f | grep LUKS
General	
Lock screen	Super + L
Auto-lock timeout	gsettings set org.gnome.desktop.session idle-delay 300
Check listening ports	sudo ss -tlnp
Disable service	sudo systemctl disable --now servicename
Security audit	~/bin/security-audit.sh
What's Next

You now understand Ubuntu's security architecture: the UFW firewall for network protection, SSH key authentication for secure remote access, AppArmor for mandatory access control, Fail2Ban for brute-force defense, kernel hardening through sysctl, automatic security updates, disk encryption, and the common-sense habits that tie it all together. You have a security audit script to check your posture and a cheat sheet for quick reference.

In Chapter 28, we'll tackle the myths and misconceptions that surround Linux — addressing the questions beginners hear most often, from "Is Linux immune to viruses?" to "Do I need antivirus software?" and "Will my hardware work?"
Try It Yourself

    Verify your firewall. Run sudo ufw status verbose. If inactive, enable it with sudo ufw enable. Set defaults: sudo ufw default deny incoming and sudo ufw default allow outgoing.

    Generate an SSH key. Run ssh-keygen -t ed25519 -C "your@email.com". Set a passphrase. Verify the files exist in ~/.ssh/. Set correct permissions with chmod 700 ~/.ssh and chmod 600 ~/.ssh/id_ed25519.

    Check AppArmor. Run sudo aa-status. Note how many profiles are enforced. Run sudo dmesg | grep apparmor | grep DENIED — if any denials appear, an AppArmor profile recently blocked something (this is normal and usually correct).

    Install Fail2Ban (if you run SSH). Run sudo apt install fail2ban, then sudo fail2ban-client status. Even on a desktop, seeing the protection in action is educational.

    Audit your listening services. Run sudo ss -tlnp. Review what's listening for incoming connections. If anything surprises you, investigate whether it's needed.

    Verify automatic updates. Run cat /etc/apt/apt.conf.d/20auto-upgrades. Confirm both values are "1". If not, enable them.

    Check disk encryption. Run lsblk -f. Look for crypto_LUKS. If you don't have encryption, consider it for your next reinstall — it's the best protection for a lost or stolen device.

    Run the security audit script. Copy the security-audit.sh script from this chapter. Save it to ~/bin/, make it executable, and run it. Review the output and address any ⚠️ items.

    Review kernel hardening. Run sysctl kernel.randomize_va_space, sysctl kernel.dmesg_restrict, and sysctl fs.protected_symlinks. All should show secure values (2, 1, 1 respectively) on Ubuntu 26.04.

    ✅ Chapter 27 Complete You can configure UFW firewall (defaults, rules, rate limiting, GUI), generate and manage SSH key pairs (Ed25519, passphrase, agent, multi-key config, permissions), harden SSH server settings, understand and check AppArmor (enforce vs. complain, denial logs), install and configure Fail2Ban for brute-force protection, audit and set kernel sysctl parameters, verify automatic security updates, check disk encryption status, follow common-sense security habits, and run a security audit script to assess your overall posture. In Chapter 28, we'll separate Linux myths from reality.

