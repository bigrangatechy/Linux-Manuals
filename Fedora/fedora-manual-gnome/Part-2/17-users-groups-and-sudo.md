# Chapter 17: Users, Groups, and sudo — Managing Multi-User Systems

## Linux Is Multi-User From the Ground Up

Windows and macOS treat multi-user as an add-on — multiple accounts exist, but the system was designed for one person at a keyboard. Linux inherited Unix's DNA: **multiple people, simultaneously, sharing one machine.**

This isn't theoretical. Since the 1970s, Unix systems have handled dozens — sometimes hundreds — of logged-in users at once, each running their own processes, reading their own files, completely isolated from each other. Your Fedora 44 workstation has the same architecture as a university mainframe from 1975.

This design philosophy explains everything you've learned so far:

- **Permissions** exist because multiple users share the same filesystem
- **Ownership** exists because files must belong to specific people
- **Groups** exist because collaboration needs controlled sharing
- **Home directories** exist because every user needs private space
- **sudo** exists because administration must be delegated safely

This chapter ties all those threads together. You'll learn how to create and manage user accounts, configure sudo access, implement password policies, understand the login process, and administer a system where multiple people coexist securely.

---

## Table of Contents

| Topic | Section |
|---|---|
| User Account Fundamentals | 1 |
| Creating User Accounts | 2 |
| Modifying User Accounts | 3 |
| Deleting User Accounts | 4 |
| Group Management | 5 |
| The sudo System | 6 |
| Password Policies and Aging | 7 |
| Switching Identities (su, sudo -i) | 8 |
| The Login Process and PAM | 9 |
| Polkit: Graphical Authorization | 10 |
| Managing Your System Securely | 11 |

---

## 1. User Account Fundamentals

### What Makes a User?

A Linux user is defined by an entry in `/etc/passwd`. Despite the alarming name, this file doesn't contain passwords (those are hashed in `/etc/shadow`). It's a colon-delimited database:

john:x:1000:1000:John Smith:/home/john:/bin/bash ↑ ↑ ↑ ↑ ↑ ↑ ↑ │ │ │ │ │ │ └── Default shell │ │ │ │ │ └───────────── Home directory │ │ │ │ └─────────────────────── GECOS (full name/comments) │ │ │ └────────────────────────────── GID (primary group ID) │ │ └─────────────────────────────────── UID (user ID number) │ └──────────────────────────────────────── Password placeholder (x = see shadow) └──────────────────────────────────────────── Username

Each field's purpose:
Field	Description	Example
Username	Login name, lowercase letters/digits	john
Password	x means hash lives in /etc/shadow	x
UID	Unique numerical user ID	1000
GID	Primary group ID (usually a personal group)	1000
GECOS	Full name, contact info, comments	John Smith
Home	Personal directory path	/home/john
Shell	Default command interpreter	/bin/bash
UID Ranges

Fedora assigns UIDs in specific ranges:
Range	Who	Examples
0	Root (superuser)	root
1–999	System accounts (services/daemons)	apache, nginx, mysql, polkitd
1000–60000	Regular human users	john, mary, alex
65534	Nobody (anonymous/placeholder)	nobody

# View your own entry:
grep "$USER" /etc/passwd

# View root's entry:
grep "^root:" /etc/passwd
# root:x:0:0:root:/root:/bin/bash

# Count regular users (UID 1000+):
awk -F: '$3 >= 1000 {print $1}' /etc/passwd

System Accounts vs. Regular Users

System accounts aren't human — they're identities for services and daemons. When Apache starts, it runs as user apache. When the SSH daemon receives a connection, it handles it as user sshd. This isolation means a compromised service can't wreak havoc as root.

# View system service accounts:
awk -F: '$3 < 1000 && $3 != 0 {print $1, $3}' /etc/passwd | head -15
# bin 1, daemon 2, adm 3, lp 4, sync 5, shutdown 6, halt 7, mail 8, ...

Each system account owns its files and nothing else. The mysql user owns database files in /var/lib/mysql/. The nginx user owns web server worker processes. If an attacker compromises nginx, they get nginx's permissions — limited to web-serving operations, not system-wide control.
The Shadow Password System

Actual password hashes live in /etc/shadow, readable ONLY by root:

# Try as regular user:
cat /etc/shadow
# cat: /etc/shadow: Permission denied

# As root:
sudo cat /etc/shadow | grep john
# john:$6$xyz...$hashed_password_string:20145:0:99999:7:::

Shadow entry format:

john:$6$salt$hash:20145:0:99999:7:::
  ↑          ↑      ↑    ↑    ↑   ↑
  │          │      │    │    │   └── Password expiration warning period (days)
  │          │      │    │    └────── Maximum password age (days, 99999 = never)
  │          │      │    └─────────── Minimum password age (days, 0 = change anytime)
  │          │      └──────────────── Days since epoch (Jan 1, 1970) when password last changed
  │          └─────────────────────── Password hash ($6$ = SHA-512)
  └────────────────────────────────── Username

The $6$ prefix indicates SHA-512 hashing — the strongest algorithm supported. The salt (random characters after $6$) ensures identical passwords produce different hashes.

    💡 Why /etc/shadow Exists

    In early Unix, password hashes were in /etc/passwd, readable by everyone. Users with enough patience could crack each other's passwords. Moving hashes to /etc/shadow — readable only by root — eliminated this attack vector. The /etc/passwd file remains world-readable because many programs need username/UID/home directory information that doesn't require password access.

2. Creating User Accounts
useradd: The Low-Level Tool

useradd is the fundamental user creation command. It doesn't prompt — it expects options:

# Create a user with defaults (creates home directory, copies skeleton files):
sudo useradd -m sarah

# Set password for the new user:
sudo passwd sarah
# New password: ********
# Retype new password: ********
# passwd: all authentication tokens updated successfully.

With options:

# Full-featured user creation:
sudo useradd -m \
  -c "Sarah Connor" \
  -s /bin/bash \
  -G wheel,developers \
  -e 2026-12-31 \
  sarah

Option	Meaning
-m	Create home directory (/home/sarah)
-c	Comment/GECOS field (full name)
-s	Default shell
-G	Supplementary groups (comma-separated)
-g	Primary group (default: creates personal group matching username)
-e	Account expiration date (YYYY-MM-DD)
-u	Specific UID (auto-assigned otherwise)
-d	Custom home directory path
-k	Skeleton directory (default: /etc/skel/)
The Skeleton Directory

When useradd -m creates a home directory, it copies files from /etc/skel/:

ls -la /etc/skel/
# total 32
# drwxr-xr-x. 2 root root 4096 Jul 5 09:00 .
# drwxr-xr-x. 3 root root 4096 Jul 5 09:00 ..
# -rw-r--r--. 1 root root   18 Jul 5 09:00 .bash_logout
# -rw-r--r--. 1 root root 141 Jul 5 09:00 .bash_profile
# -rw-r--r--. 1 root root 492 Jul 5 09:00 .bashrc

These files initialize the new user's shell environment:

    .bashrc — interactive shell configuration (aliases, prompt, etc.)
    .bash_profile — login shell configuration (runs on SSH/login)
    .bash_logout — cleanup commands when logging out

Any files you place in /etc/skel/ are automatically copied to every new user's home directory. Useful for distributing company-wide configurations or welcome documents.
adduser: The Friendlier Alternative

Fedora provides adduser as a symbolic link to useradd:

which adduser
# /usr/sbin/adduser

ls -l /usr/sbin/adduser
# lrwxrwxrwx. 1 root root 7 Jul 5 09:00 adduser -> useradd

On Debian/Ubuntu, adduser is a separate interactive script. On Fedora, it's the same command — no interactive prompting. Always follow useradd -m with passwd to set the initial password.
Default User Creation Settings

Fedora's user creation defaults are defined in /etc/login.defs and /etc/default/useradd:

# View useradd defaults:
sudo useradd -D
# GROUP=100
# HOME=/home
# INACTIVE=-1
# EXPIRE=
# SHELL=/bin/bash
# SKEL=/etc/skel
# CREATE_MAIL_SPOOL=yes

Key settings in /etc/login.defs:

grep -E "^(UID_MIN|UID_MAX|GID_MIN|GID_MAX|PASS_)" /etc/login.defs
# UID_MIN         1000      ← First regular user UID
# UID_MAX         60000     ← Last regular user UID
# GID_MIN         1000      ← First regular group GID
# GID_MAX         60000     ← Last regular group GID
# PASS_MAX_DAYS   99999     ← Max days before password must change
# PASS_MIN_DAYS   0         ← Min days between changes
# PASS_MIN_LEN    5         ← Minimum password length
# PASS_WARN_AGE   7         ← Warning days before expiration

3. Modifying User Accounts
usermod: Modify User Properties

# Change full name (GECOS):
sudo usermod -c "Sarah J. Connor" sarah

# Change default shell:
sudo usermod -s /bin/zsh sarah

# Add to supplementary groups (preserve existing memberships):
sudo usermod -aG wheel,developers sarah
#     ↑ -a = APPEND (without -a, you REPLACE all supplementary groups!)

# Change primary group:
sudo usermod -g developers sarah

# Change username (renames the account):
sudo usermod -l sarah_connor sarah
# Also rename home directory to match:
sudo usermod -d /home/sarah_connor -m sarah_connor

# Lock account (prepend ! to password hash):
sudo usermod -L sarah

# Unlock account:
sudo usermod -U sarah

# Set account expiration date:
sudo usermod -e 2026-12-31 sarah

# Move home directory to new location:
sudo usermod -d /home/newpath -m sarah

    ⚠️ The -aG Trap

    Using -G without -a REPLACES all supplementary group memberships. This is a common mistake:

    # Sarah is in wheel, developers, audio
    # You want to ADD her to video:

    sudo usermod -G video sarah    # WRONG! Removes wheel, developers, audio
    sudo usermod -aG video sarah   # CORRECT! Adds video, keeps the rest

    Always use -aG (append to groups), never bare -G, unless you intend to replace ALL group memberships.

passwd: Change Passwords

# Change your own password:
passwd
# Current password: ********
# New password: ********
# Retype: ********

# Change another user's password (requires sudo):
sudo passwd sarah

# Lock a user's password (prevents login):
sudo passwd -l sarah

# Unlock:
sudo passwd -u sarah

# Force password change on next login:
sudo passwd -e sarah
# Sarah will be prompted to set a new password on next login

# View password status:
sudo passwd -S sarah
# sarah PS 2026-07-05 0 99999 7 -1
#         ↑
#         Status: PS=password set, LK=locked, NP=no password

chage: Password Aging Information

chage (change age) manages password expiration policies:

# View password aging info:
sudo chage -l sarah
# Last password change                    : Jul 05, 2026
# Password expires                        : never
# Password inactive                       : never
# Account expires                          : never
# Minimum number of days between password changes  : 0
# Maximum number of days between password changes  : 99999
# Number of days of warning before password expires : 7

# Set maximum password age (force change every 90 days):
sudo chage -M 90 sarah

# Set minimum days between changes (prevent immediate re-change):
sudo chage -m 1 sarah

# Set warning period before expiration:
sudo chage -W 14 sarah

# Set account expiration date:
sudo chage -E 2026-12-31 sarah

# Force password change on next login:
sudo chage -d 0 sarah

chsh: Change Default Shell

# Change your own shell:
chsh -l        # List available shells
chsh -s /bin/zsh    # Set zsh as default

# Change another user's shell (requires sudo):
sudo chsh -s /bin/zsh sarah

Available shells are listed in /etc/shells:

cat /etc/shells
# /bin/sh
# /bin/bash
# /usr/bin/sh
# /usr/bin/bash
# /usr/bin/zsh
# /usr/bin/fish
# /usr/bin/tmux

4. Deleting User Accounts
userdel: Remove Users

# Delete user but KEEP their home directory:
sudo userdel sarah

# Delete user AND their home directory:
sudo userdel -r sarah

# Force deletion even if user is currently logged in:
sudo userdel -rf sarah

    ⚠️ Deleting Users Safely

    Before deleting a user:

        Check if they're logged in: who | grep sarah
        Check for running processes: ps -u sarah
        Archive their home directory: tar cJf /backup/sarah_home.tar.xz /home/sarah/
        Check for cron jobs: sudo crontab -l -u sarah
        Check for owned files outside home: sudo find / -user sarah 2>/dev/null
        THEN delete: sudo userdel -r sarah

    Files owned by a deleted user become "orphaned" — their owner UID shows as a number instead of a name:

    # After deleting sarah (UID 1002):
    ls -l /shared/report.pdf
    # -rw-r--r--. 1 1002 developers 1234 Jul 5 report.pdf
    #              ↑
    #              No username matches UID 1002 anymore

    Either reassign ownership or delete these files after account removal.

5. Group Management
Why Groups Matter

Groups are the primary mechanism for collaborative access control. Instead of managing permissions per-user (which doesn't scale), you create groups and manage membership:

                    /shared/projects/
                    Owner: root
                    Group: developers (with setgid)
                    Permissions: rwxrws--- (770)
                        │
            ┌───────────┴───────────┐
            │                       │
     john (member)            mary (member)
     can create files         can create files
     accessible to all        accessible to all
     developers               developers

groupadd: Create Groups

# Create a new group:
sudo groupadd developers

# Create with specific GID:
sudo groupadd -g 5000 developers

# Create a system group (GID below 1000):
sudo groupadd -r webapp

groupmod: Modify Groups

# Rename a group:
sudo groupmod -n devteam developers

# Change GID:
sudo groupmod -g 5001 devteam

groupdel: Delete Groups

sudo groupdel devteam

A group cannot be deleted if it's the primary group of any user. First change those users' primary groups, then delete.
Managing Group Membership

# Add user to a group:
sudo usermod -aG developers sarah

# Alternative: gpasswd (more granular control):
sudo gpasswd -a sarah developers      # Add
sudo gpasswd -d sarah developers      # Remove
sudo gpasswd -A sarah developers      # Make sarah group administrator

# List groups a user belongs to:
groups sarah
# sarah : sarah wheel developers

# View members of a specific group:
getent group developers
# developers:x:5000:john,mary,sarah

    💡 Primary vs. Supplementary Groups

    Every user has exactly ONE primary group (listed in /etc/passwd). They can also belong to multiple supplementary groups (listed in /etc/group).

    When a user creates a file, the file's group is the user's primary group by default — unless the directory has the setgid bit, in which case the file inherits the directory's group.

    Fedora creates a personal group for each user (named after the username) as their primary group. This means files you create are owned by john:john — private by default. Joining supplementary groups like developers or wheel grants additional access without changing your primary group.

The wheel Group: Fedora's Administrators

On Fedora, members of the wheel group can use sudo. This is configured in /etc/sudoers:

sudo grep wheel /etc/sudoers
# %wheel  ALL=(ALL)  ALL

The % prefix means "this is a group, not a username." Adding someone to wheel grants them administrative access:

# Grant sudo access to sarah:
sudo usermod -aG wheel sarah

# Sarah must log out and back in for the change to take effect
# After re-login:
su - sarah -c "groups"
# sarah wheel developers

6. The sudo System
What sudo Actually Does

sudo (superuser do) allows a permitted user to execute a command as root (or another user) without needing the root password. It's safer than logging in as root because:

    Accountability — every sudo command is logged with username, timestamp, and command
    Granular control — specific users can run specific commands, not blanket root access
    Temporary elevation — privileges expire after a timeout (default 15 minutes)
    No shared password — users authenticate with their own password, not a shared root password

┌──────────┐     sudo dnf update      ┌──────────┐     verify password     ┌──────────┐
│  User    │  ──────────────────────► │  sudo    │ ─────────────────────► │ /etc/sudoers │
│  types:  │                         │  checks:│                         │  + /etc/sudoers.d/  │
│  sudo    │                         │  1. User │                         │  rules allow?  │
│  dnf     │                         │  2. Cmd  │                         │  YES → execute as root │
│  update  │                         │  3. Host │                         │  NO  → deny + log │
└──────────┘                         └──────────┘                         └──────────┘

Basic sudo Usage

# Run a single command as root:
sudo dnf update

# Run a command as a different user:
sudo -u mary ls /home/mary/

# Open a root shell (interactive):
sudo -i
# You're now root. Exit with 'exit' or Ctrl+D.

# Open a root shell preserving your environment:
sudo -s

# Run a command with password cached (refresh timer):
sudo -v    # Validates credentials, extends timeout

# Clear cached credentials immediately:
sudo -k

The sudoers File

sudo's configuration lives in /etc/sudoers:

# NEVER edit this file directly — use visudo:
sudo visudo

The format:

user    host=(runas)  commands
%group  host=(runas)  commands

Field	Meaning	Example
who	Username or %groupname	john or %wheel
host	Which hosts this applies to	ALL for any machine
(runas)	Which user to run as	(ALL) = any user, (root) = only root
commands	Allowed commands (full paths)	ALL or /usr/bin/dnf

Common entries:

# The default wheel rule (all members of wheel can run anything):
%wheel  ALL=(ALL)  ALL

# Specific user can run anything:
john  ALL=(ALL)  ALL

# User can only run dnf (package manager):
sarah  ALL=(ALL)  /usr/bin/dnf

# User can run dnf WITHOUT password prompt:
sarah  ALL=(ALL)  NOPASSWD: /usr/bin/dnf

# Group can restart web server only:
%webadmins  ALL=(root)  /usr/bin/systemctl restart httpd

    ⚠️ Always Use visudo

    Never edit /etc/sudoers with a regular text editor. visudo performs syntax checking before saving. A syntax error in sudoers can lock EVERYONE out of root access — including yourself. visudo prevents this by refusing to save invalid configurations.

/etc/sudoers.d/: Drop-In Rules

Fedora uses /etc/sudoers.d/ for modular sudo configuration. Instead of editing the main file, create drop-in files:

# Create a sudoers drop-in for the devops team:
sudo visudo -f /etc/sudoers.d/devops

Inside the file:

# DevOps team can manage services and packages
%devops  ALL=(root)  /usr/bin/systemctl, /usr/bin/dnf, /usr/bin/journalctl

The main /etc/sudoers includes this directory:

sudo grep sudoers.d /etc/sudoers
# @includedir /etc/sudoers.d

Rules in /etc/sudoers.d/ follow the same format. File permissions must be strict:

sudo chmod 440 /etc/sudoers.d/devops

sudo Logging

Every sudo command is logged:

# View sudo history:
sudo journalctl -t sudo

# Or search the audit log:
sudo ausearch -x /usr/bin/sudo

# Sample output:
# Jul 05 10:15:23 hostname sudo[1234]: john : TTY=pts/0 ; PWD=/home/john ; USER=root ; COMMAND=/usr/bin/dnf update
# Jul 05 10:16:45 hostname sudo[1245]: mary : TTY=pts/1 ; PWD=/var/www ; USER=root ; COMMAND=/usr/bin/systemctl restart nginx

This audit trail is invaluable for security investigations and compliance requirements.
sudo Timers and Security

# Default timeout (credentials cached for 15 minutes):
# After timeout, sudo asks for password again

# Extend timeout (in /etc/sudoers via visudo):
Defaults timestamp_timeout=30    # 30 minutes

# Disable caching entirely (require password every time):
Defaults timestamp_timeout=0

# Require password per-terminal (not shared across sessions):
Defaults !tty_tickets    # Old behavior (shares across terminals)
Defaults tty_tickets      # Default: each terminal independent

    💡 Root vs. sudo: Why Fedora Disables Root Login

    Fedora doesn't set a root password by default during installation. Instead, the first user created is added to wheel and uses sudo. This means:

        There's no root password to crack
        All administrative actions are attributed to a named user via sudo logs
        You can't accidentally log in as root (a dangerous habit)

    To switch to a root shell, use sudo -i. You authenticate with YOUR password, not root's.

7. Password Policies and Aging
Why Password Policies Matter

Uncontrolled passwords lead to predictable choices: password123, qwerty, pet names, birth dates. Fedora enforces quality checks and optional aging policies to mitigate this.
PAM Password Quality Module

Password quality is enforced by pam_pwquality (part of PAM — Pluggable Authentication Modules, covered in Section 9). Configuration lives in:

sudo cat /etc/security/pwquality.conf

Default settings on Fedora:

# minlen: minimum password length
minlen = 8

# dcredit: minimum digits (-1 = at least 1)
dcredit = -1

# ucredit: minimum uppercase (-1 = at least 1)
ucredit = -1

# lcredit: minimum lowercase (-1 = at least 1)
lcredit = -1

# ocredit: minimum other characters/symbols (-1 = at least 1)
ocredit = -1

# maxrepeat: maximum consecutive identical characters
maxrepeat = 3

# difok: minimum characters that must differ from old password
difok = 5

When you set a password that fails these checks:

sudo passwd sarah
# New password: abc
# BAD PASSWORD: The password is shorter than 8 characters
# New password: password1
# BAD PASSWORD: The password fails the dictionary check
# New password: Tr0ub4dor&3
# Retype: Tr0ub4dor&3
# passwd: all authentication tokens updated successfully.

Customizing Password Policy

# Edit the quality settings:
sudo nano /etc/security/pwquality.conf

# Example: enforce stronger passwords
minlen = 12
dcredit = -2     # At least 2 digits
ucredit = -1     # At least 1 uppercase
ocredit = -1     # At least 1 symbol
difok = 8        # At least 8 chars different from old password
maxrepeat = 2    # No more than 2 identical consecutive chars
enforce_for_root = 1  # Apply rules to root password too

Password Aging Policies

Forced password rotation is debated in security circles — NIST revised guidance in 2017 suggesting frequent forced changes may REDUCE security (users pick weaker passwords when forced to change often). However, some compliance regimes still require it.

# Set global aging defaults in /etc/login.defs:
sudo nano /etc/login.defs

# PASS_MAX_DAYS   90      # Force change every 90 days
# PASS_MIN_DAYS   1       # Wait at least 1 day between changes
# PASS_WARN_AGE   7       # Warn 7 days before expiration

Per-user aging (overrides defaults):

# Set Sarah's password to expire in 90 days, warn 7 days before:
sudo chage -M 90 -W 7 sarah

# Lock account 30 days after password expires (inactive period):
sudo chage -I 30 sarah

# View the result:
sudo chage -l sarah

Account Lockout Policy

Protect against brute-force attacks by locking accounts after failed attempts:

# Fedora uses pam_faillock for this
# Configure in PAM files (covered in Section 9):
sudo nano /etc/security/faillock.conf

Example settings:

# Lock after 5 failed attempts for 10 minutes:
deny = 5
unlock_time = 600

# Lock root account too (security-hardened environments):
even_deny_root = yes
root_unlock_time = 900

Check lock status:

# View failed login attempts:
sudo faillock --user sarah

# Reset (unlock) an account:
sudo faillock --user sarah --reset

8. Switching Identities
su: Switch User

# Switch to root (requires root password — which may not be set on Fedora):
su -
# Password: [root password]

# Switch to another user (requires their password):
su - sarah
# Password: [sarah's password]

# Run a single command as another user:
su - sarah -c "whoami"
# sarah

The - (or -l/--login) flag matters significantly:
Flag	Environment
su sarah	Keeps YOUR environment, just changes identity
su - sarah	Loads SARAH's environment (clean login shell)

# Without - (inherits your PATH, which may differ):
su sarah
echo $HOME
# /home/john    ← Still your home! Only UID changed.

# With - (proper login):
su - sarah
echo $HOME
# /home/sarah   ← Sarah's home, Sarah's environment

sudo vs. su
Feature	sudo command	su - user
Authentication	YOUR password	TARGET user's password
Duration	Single command	Interactive session
Logging	Logged with your identity	Logged as target user login
Granularity	Configurable per-command	Full session or nothing
Use case	One-off privileged command	Extended work as another user
Practical Identity Switching

# Most common: run one command as root
sudo dnf install htop

# Extended root session (when doing multiple admin tasks):
sudo -i
# Do all your admin work
exit    # Return to regular user

# Test something as another user:
sudo -u sarah ls -l /home/sarah/private/

# Check what sudo privileges you have:
sudo -l
# User john may run the following commands on this host:
#     (ALL) ALL

# Check what another user can sudo:
sudo -l -U sarah

    💡 Prefer sudo Over su

    In professional environments, sudo is strongly preferred:

        No shared root password to compromise
        Full audit trail of who did what
        Fine-grained control over permissions
        Credentials expire (limits exposure window)

    Use su - only when you genuinely need an extended interactive root session and understand the trade-offs.

9. The Login Process and PAM
What Happens When You Log In

The login process involves multiple layers working in sequence:

1. Display Manager (GDM)       Shows login screen
        ↓
2. Username entered             GDM sends to PAM
        ↓
3. PAM authentication           Checks password against /etc/shadow
        ↓
4. PAM account management      Checks account expiration, lockout
        ↓
5. PAM session setup           Sets up environment, logs login
        ↓
6. Shell starts                 ~/.bash_profile, ~/.bashrc executed
        ↓
7. User sees desktop/terminal  Fully logged in

PAM: Pluggable Authentication Modules

PAM is the framework that handles all authentication on Linux. Instead of every program implementing its own password checking, they delegate to PAM.

PAM configuration lives in /etc/pam.d/:

ls /etc/pam.d/ | head -20
# fingerprint-auth  login  other  polkit-1  su  sudo  system-auth  ...

Each file defines authentication rules for a specific service. The most important:

cat /etc/pam.d/system-auth

#%PAM-1.0
auth        required    pam_env.so
auth        sufficient  pam_unix.so try_first_pass nullok
auth        required    pam_deny.so

account     required    pam_unix.so

password    requisite   pam_pwquality.so try_first_pass retry=3
password    sufficient  pam_unix.so sha512 shadow try_first_pass use_authtok
password    required    pam_deny.so

session     required    pam_limits.so
session     required    pam_unix.so

PAM has four management groups:
Group	Purpose	Typical Checks
auth	Verify identity	Password check against /etc/shadow
account	Verify account is usable	Expiration, lockout, access hours
password	Change passwords	Quality enforcement (pwquality)
session	Setup/teardown session	Resource limits, login logging

Control values:
Control	Behavior on Failure
required	Failure reported after all modules run
requisite	Failure stops immediately
sufficient	Success skips remaining modules; failure continues
optional	Result doesn't affect overall outcome

    💡 Don't Edit PAM Files Casually

    PAM misconfiguration can prevent ALL logins — including root. Always back up PAM files before editing:

    sudo cp /etc/pam.d/system-auth /etc/pam.d/system-auth.bak

    If you lock yourself out, you can fix it by booting from a Live USB, mounting your root filesystem, and restoring the backup.

Login Records

Linux tracks logins in several places:

# Who is currently logged in:
who
# john    pts/0   2026-07-05 09:00 (192.168.1.50)

# Same but with login time and idle time:
w
# 09:15:00 up  2:30,  2 users,  load average: 0.34, 0.28, 0.25
# USER     TTY      FROM              LOGIN@   IDLE   WHAT
# john     pts/0    192.168.1.50     09:00    0.00s  w

# History of successful logins:
last | head -10
# john    pts/0   192.168.1.50   Thu Jul 5 09:00   still logged in
# mary    tty1    :0             Wed Jul 4 14:00 - 18:30  (04:30)
# ...

# History of FAILED logins:
sudo lastb | head -10
# (unknown) tty1    :0             Thu Jul 5 03:14 - 03:14  (00:00)
# root     pts/0   10.0.0.5       Thu Jul 5 02:00 - 02:00  (00:00)

# Last time each user logged in:
lastlog
# Username    Port    From      Latest
# root        tty1              2026-06-01 09:00:00
# john        pts/0  192.168.1.50  2026-07-05 09:00:00
# sarah       pts/1  192.168.1.51  2026-07-04 14:00:00

10. Polkit: Graphical Authorization
What Is Polkit?

While sudo handles command-line privilege escalation, Polkit (PolicyKit) handles graphical authorization. When GNOME prompts you for a password to install software, change system settings, or mount a drive — that's Polkit, not sudo.

┌───────────────────────────────────────┐
│         GNOME Settings Panel          │
│    "Authentication Required"           │
│    [Password: ********]                │
│                                       │
│    User: john                         │
│    Action: org.freedesktop.systemd1   │
│    manage-unit-services               │
└───────────────────────────────────────┘

Polkit provides fine-grained authorization for system actions through rules written in JavaScript:

ls /usr/share/polkit-1/actions/
# org.freedesktop.systemd1.policy
# org.freedesktop.NetworkManager.policy
# org.gnome.controlcenter.datetime.policy
# ...

Each action file describes what the action does and who can perform it:

<action id="org.freedesktop.systemd1.manage-units">
  <description>Manage system services</description>
  <message>Authentication is required to manage system services</message>
  <defaults>
    <allow_any>no</allow_any>
    <allow_inactive>no</allow_inactive>
    <allow_active>auth_admin_keep</allow_active>
  </defaults>
</action>

Translation:

    allow_any: Non-graphical sessions → denied
    allow_inactive: Inactive graphical sessions (screen locked, switched away) → denied
    allow_active: Active graphical session → requires admin authentication (auth_admin_keep = remember for a few minutes)

Custom Polkit Rules

Create custom rules in /etc/polkit-1/rules.d/:

// /etc/polkit-1/rules.d/10-admin-users.rules

// Allow wheel group members to manage services without password:
polkit.addRule(function(action, subject) {
    if (action.id == "org.freedesktop.systemd1.manage-units" &&
        subject.isInGroup("wheel")) {
        return polkit.Result.YES;
    }
});

Rule files use JavaScript syntax and are processed in alphabetical order — lower-numbered files run first. The subject object represents the user attempting the action. The action object represents what they're trying to do.

Return values:
Value	Meaning
polkit.Result.YES	Allow without authentication
polkit.Result.NO	Deny
polkit.Result.AUTH_SELF	Require user's own password
polkit.Result.AUTH_ADMIN	Require admin password
polkit.Result.AUTH_SELF_KEEP	Auth once, remember briefly
polkit.Result.AUTH_ADMIN_KEEP	Admin auth, remember briefly

Another example — allow members of libvirt group to manage virtual machines without password prompts:

// /etc/polkit-1/rules.d/20-libvirt.rules
polkit.addRule(function(action, subject) {
    if (action.id.startsWith("org.libvirt") &&
        subject.isInGroup("libvirt")) {
        return polkit.Result.YES;
    }
});

    💡 When Would You Customize Polkit?

    Most Fedora users never touch Polkit rules. The defaults work well. You'd customize them in scenarios like:

        Running a headless server where GUI auth prompts are impractical
        Granting specific groups password-free access to certain operations
        Restricting what non-admin users can do through GNOME settings
        Automation that calls system services requiring Polkit authorization

    Unless you have a specific reason, leave Polkit alone.

11. Managing Your System Securely
The Principle of Least Privilege

Every user account, service, and process should have exactly the permissions it needs — and no more. This is the principle of least privilege, and it's the foundation of Linux security.

Applied practically:
Scenario	Least-Privilege Approach
Web developer needs to restart nginx	Add to nginx group or grant sudo for systemctl restart nginx only — NOT full wheel access
Temporary contractor needs file access	Create account with expiration date (useradd -e 2026-09-30 contractor)
Database backup script	Run as a dedicated backup user, not root
Monitoring agent	Runs as its own system user with minimal file access
Root Lockdown Checklist

# 1. Verify root has no password (locked):
sudo passwd -S root
# root LK ...  ← LK = locked

# 2. Verify root cannot SSH in:
sudo grep "^PermitRootLogin" /etc/ssh/sshd_config
# PermitRootLogin no

# 3. Check who has sudo access:
sudo grep -r "ALL" /etc/sudoers /etc/sudoers.d/ 2>/dev/null | grep -v "^#"

# 4. Audit wheel group membership:
getent group wheel
# wheel:x:10:john,mary

# 5. Review recent sudo activity:
sudo journalctl -t sudo --since "7 days ago"

# 6. Check for accounts with no password:
sudo awk -F: '($2 == "" || $2 == "!") {print $1}' /etc/shadow
# Should return nothing (all accounts have passwords or are locked)

Regular User Auditing

# List all human users (UID >= 1000):
awk -F: '$3 >= 1000 && $3 < 65534 {print $1, $3, $6}' /etc/passwd

# Check for expired passwords:
sudo chage -l $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd) 2>/dev/null

# Find users who haven't logged in recently:
lastlog | awk '$2 != "**Never"'

Managing Multiple Users: Best Practices
Practice	Why
Use wheel group for admin delegation	Revoking access = removing from group
Create functional groups (developers, webteam)	Permissions scale without per-user changes
Set account expiration for contractors/temporary staff	Automatic deactivation reduces forgotten accounts
Force password change on first login (passwd -e)	Eliminates risk of known default passwords
Audit sudo logs monthly (journalctl -t sudo)	Detect unauthorized or suspicious activity
Review group membership quarterly	Remove stale access rights
Never share passwords	Each action attributed to individual identity
Disable accounts, don't delete immediately	Preserves audit trail; allows investigation
Guest and Shared Kiosk Accounts

If you need a temporary or guest account:

# Create a guest user with restricted shell and no sudo access:
sudo useradd -m -s /bin/rbash guest
sudo passwd guest

# rbash (restricted bash) prevents:
# - Changing directories with cd
# - Setting PATH
# - Using / in commands
# - Redirecting output

For a kiosk scenario (single-app auto-login):

# Create kiosk user:
sudo useradd -m -s /bin/bash kiosk

# Configure GNOME auto-login for kiosk user:
sudo nano /etc/gdm/custom.conf
# [daemon]
# AutomaticLogin=kiosk
# AutomaticLoginEnable=True

    ⚠️ Auto-Login Security Implications

    Auto-login bypasses authentication entirely. Anyone with physical access gets full access to that user's session and files. Only use auto-login on physically secured machines or for locked-down kiosk accounts — never on laptops or shared computers.

Command Reference: User and Group Management
Command	Purpose
useradd -m username	Create user with home directory
usermod -aG group username	Add user to supplementary group
usermod -L username	Lock user account
usermod -U username	Unlock user account
usermod -e YYYY-MM-DD username	Set account expiration
usermod -s /bin/shell username	Change default shell
userdel -r username	Delete user and home directory
passwd username	Set/change password
passwd -l username	Lock password
passwd -e username	Force password change on next login
passwd -S username	View password status
chage -l username	View password aging info
chage -M days username	Set max password age
chage -E YYYY-MM-DD username	Set account expiration
chage -d 0 username	Force password change on next login
chsh -s /bin/shell	Change your default shell
groupadd groupname	Create new group
groupmod -n newname oldname	Rename group
groupdel groupname	Delete group
gpasswd -a user group	Add user to group
gpasswd -d user group	Remove user from group
gpasswd -A user group	Make user group administrator
groups username	List user's groups
getent group groupname	View group members
id username	View UID, GID, and group memberships
who	List currently logged-in users
w	List logged-in users with activity
last	View login history
lastb	View failed login attempts
lastlog	View last login for all users
sudo -l	List your sudo privileges
sudo -u user command	Run command as another user
sudo -i	Open interactive root shell
sudo visudo	Edit sudoers file safely
getenforce	Check SELinux mode
faillock --user username	View failed login attempts
faillock --user username --reset	Unlock locked account
Try It Yourself
Exercise 1: User Lifecycle

# Create a new user with full details:
sudo useradd -m -c "Test User" -s /bin/bash testuser
sudo passwd testuser

# Verify the account exists:
grep testuser /etc/passwd
id testuser

# View the home directory created:
ls -la /home/testuser/
# Note the .bashrc, .bash_profile, .bash_logout from /etc/skel/

# Check password status:
sudo passwd -S testuser

# Force password change on next login:
sudo passwd -e testuser

# View aging info:
sudo chage -l testuser

# Lock the account:
sudo usermod -L testuser
sudo passwd -S testuser   # Shows LK status

# Unlock:
sudo usermod -U testuser

# Clean up:
sudo userdel -r testuser
verify = grep testuser /etc/passwd  # Should return nothing

Exercise 2: Group Collaboration

# Create a collaborative group:
sudo groupadd collab

# Create two users and add them to the group:
sudo useradd -m -G collab user_a
sudo useradd -m -G collab user_b
sudo passwd user_a   # Set a password
sudo passwd user_b

# Create a shared directory with setgid:
sudo mkdir /shared_collab
sudo chown root:collab /shared_collab
sudo chmod 2770 /shared_collab

# As user_a, create a file:
sudo -u user_a bash -c 'echo "hello from A" > /shared_collab/file_a.txt'

# Verify group inheritance:
ls -l /shared_collab/file_a.txt
# -rw-r--r--. 1 user_a collab 15 Jul 5 file_a.txt
#                    ↑
#            Group is collab (inherited via setgid)

# As user_b, read user_a's file:
sudo -u user_b cat /shared_collab/file_a.txt
# hello from A

# Clean up:
sudo userdel -r user_a
sudo userdel -r user_b
sudo groupdel collab
sudo rm -rf /shared_collab

Exercise 3: sudo Configuration

# View your current sudo privileges:
sudo -l

# Examine the sudoers file:
sudo visudo
# Find the wheel group rule
# Add a comment if you want (lines starting with #)
# Save and quit — DON'T change anything yet

# Create a drop-in rule for a hypothetical service user:
sudo visudo -f /etc/sudoers.d/webadmin

# Add this line:
# webadmin ALL=(root) /usr/bin/systemctl restart nginx, /usr/bin/systemctl status nginx

# Set correct permissions:
sudo chmod 440 /etc/sudoers.d/webadmin

# Verify syntax:
sudo visudo -c

# Clean up (if you didn't actually create the webadmin user):
sudo rm /etc/sudoers.d/webadmin

Exercise 4: Login Auditing

# See who's logged in right now:
who
w

# View your own login history:
last $USER | head -10

# Check for failed login attempts:
sudo lastb 2>/dev/null | head -10 || echo "No failed login records"

# View last login for all users:
lastlog | grep -v "Never"

# Check your password aging:
chage -l $USER

Exercise 5: Password Policy Inspection

# View password quality settings:
cat /etc/security/pwquality.conf

# View password aging defaults:
grep -E "^PASS_" /etc/login.defs

# View faillock settings:
cat /etc/security/faillock.conf

# Check if your own password would pass quality checks:
# (Try changing it — the system will reject weak choices)
passwd
# Try: abc         → rejected (too short)
# Try: password123 → rejected (dictionary word)
# Try: MySecure!Pass2026 → accepted
# Cancel with Ctrl+C if you don't want to actually change it

Exercise 6: PAM Exploration (Read-Only)

# List PAM configuration files:
ls /etc/pam.d/

# View the main authentication config:
cat /etc/pam.d/system-auth

# View the sudo PAM config:
cat /etc/pam.d/sudo

# View the login PAM config:
cat /etc/pam.d/login

# Identify which modules handle:
# - Password checking (pam_unix, pam_pwquality)
# - Account expiration (pam_unix)
# - Session setup (pam_limits, pam_unix)

# DON'T modify any PAM files — just observe and understand the structure

Troubleshooting Common Issues
Symptom	Likely Cause	Resolution
User can't log in after password change	Password typo or keyboard layout issue	Reset via root or another sudo user: sudo passwd username
sudo: user is not in the sudoers file	User not in wheel group or sudoers not configured	sudo usermod -aG wheel username (then re-login)
User added to group but still denied access	Group membership changes require new login session	su - username or log out/in to refresh group membership
userdel: user is currently logged in	User has active sessions or running processes	Kill processes (sudo pkill -u username) then retry
userdel: cannot remove entry	User has running cron jobs	Remove cron (sudo crontab -r -u username) then retry
Account locked due to failed attempts	pam_faillock triggered after threshold	sudo faillock --user username --reset
Password expires and user can't change it	Account expired or password too weak	Admin reset: sudo passwd username + sudo chage -d 0 username
Can't edit sudoers — syntax error	Direct edit broke the file	Boot from Live USB, mount root, restore from backup or fix syntax
GNOME prompts for password repeatedly	Polkit auth keep timeout too short	Normal behavior; customize via Polkit rules if needed
New user has no files in home directory	useradd run without -m flag	Recreate with -m, or manually copy from /etc/skel/
What's Next

You now understand the complete user and group management system: creating/modifying/deleting accounts, group membership and collaboration patterns, the sudo privilege escalation system with its configuration and audit trail, password policies and aging controls, identity switching with su and sudo, the PAM authentication framework, Polkit graphical authorization, and security best practices for multi-user administration.

Chapter 18 dives into Fedora 44's package management system — DNF5 and Flatpak. You'll learn how the terminal gives you far more control than GNOME Software for installing, updating, searching, and managing software packages. We'll cover DNF5's commands, repository management, RPM packages, Flatpak from the command line, and how these tools work together to keep your system updated and secure.

Until then, practice auditing your own system. Run who, last, sudo -l, groups, chage -l $USER — get familiar with the user landscape of your machine. Understanding who has access and what they can do is the first step toward secure system administration.

    ✅ Chapter 17 Complete You understand Linux's multi-user architecture (UIDs, GIDs, system vs. regular accounts, the shadow password system separating hashes from the world-readable /etc/passwd), user lifecycle management (useradd with skeleton directory, usermod with -aG group appending, passwd for password management, chage for aging policies, userdel with -r for clean removal), group management (groupadd/groupmod/groupdel, primary vs. supplementary groups, personal groups on Fedora, the wheel group for sudo delegation, setgid directories for collaborative inheritance), the sudo system (privilege escalation with your own password, /etc/sudoers and visudo syntax, /etc/sudoers.d/ drop-in rules, per-command granularity, NOPASSWD directives, sudo logging via journalctl, timestamp caching and timeout configuration), password policies (pam_pwquality for strength enforcement with minlen/dcredit/ucredit/ocredit/difok parameters, /etc/login.defs for aging defaults, per-user chage overrides, pam_faillock for brute-force protection), identity switching (su with/without login shell, sudo -i for interactive root, sudo -u for per-command user switching, sudo -l for privilege auditing), the login process (GDM → PAM → shell startup sequence, PAM's four management groups: auth/account/password/session, control values required/requisite/sufficient/optional, /etc/pam.d/ configuration files), login auditing (who/w/last/lastb/lastlog commands), Polkit graphical authorization (JavaScript rules in /etc/polkit-1/rules.d/, action files defining capabilities, return values YES/NO/AUTH_SELF/AUTH_ADMIN), security best practices (principle of least privilege, root lockdown checklist, regular user audits, guest/kiosk accounts with rbash), a complete command reference table, progressive exercises covering the full user lifecycle, and troubleshooting guidance for common account-related problems.

