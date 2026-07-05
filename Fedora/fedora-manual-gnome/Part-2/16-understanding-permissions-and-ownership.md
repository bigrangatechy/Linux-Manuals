# Chapter 16: Permissions, Ownership, and Access Control

## Who Can Do What?

Every file and directory on your Fedora system has rules attached to it — rules that determine who can read it, who can modify it, who can execute it, and who can't touch it at all. These rules are called **permissions**, and they're one of the most fundamental security mechanisms in Linux.

On Windows, file permissions are managed through a complex ACL (Access Control List) system buried under right-click → Properties → Security tabs. Most users never interact with it. On macOS, permissions exist but are similarly hidden behind Finder dialogs.

On Linux, permissions are visible every single time you list files:

```bash
ls -l

-rw-r--r--. 1 john john  1234 Jul 5 09:00 notes.txt
drwxr-xr-x. 2 john john  4096 Jul 5 09:30 Documents/
-rwxr-xr-x. 1 john john  2840 Jul 5 08:15 script.sh

That string of letters at the beginning — drwxr-xr-x — is the permission set. It looks cryptic now, but by the end of this chapter you'll read it as fluently as English.

This chapter covers three interconnected systems:

    Traditional Unix permissions — the read/write/execute model used since the 1970s
    Ownership — who owns a file and what group it belongs to
    SELinux — Fedora's mandatory access control system that adds an extra security layer

All three work together. Understanding them transforms you from a user who "just clicks things" into someone who genuinely controls their system.
Table of Contents
Topic	Section
The Permission String Decoded	1
Ownership: Users and Groups	2
Changing Permissions (chmod)	3
Changing Ownership (chown, chgrp)	4
Special Permission Bits	5
Umask: Default Permissions	6
ACLs: Beyond Traditional Permissions	7
SELinux: Fedora's Extra Security Layer	8
Practical Permission Scenarios	9
1. The Permission String Decoded

Every ls -l output begins with a 10-character string (plus an optional SELinux dot). Let's dissect it:

 drwxr-xr-x.
 │└──┬──┘└──┬──┘└──┬──┘│
 │   │     │     │    └── SELinux security context indicator (.)
 │   │     │     └────── Others (everyone else)
 │   │     └──────────── Group
 │   └────────────────── User (owner)
 └──────────────────────── File type

File Type (Character 1)
Symbol	Type	Description
-	Regular file	Text, binary, image, document
d	Directory	Folder containing files
l	Symbolic link	Points to another file
c	Character device	Streams data (keyboard, /dev/null)
b	Block device	Block-based I/O (disk drives, /dev/sda)
s	Socket	Inter-process communication endpoint
p	Named pipe	FIFO data channel
Permission Triplet (Characters 2–10)

After the file type character, nine characters form three triplets — one each for user (owner), group, and others:

Position:  1  2  3  4  5  6  7  8  9  10
           d  r  w  x  r  -  x  r  -  x
           │  │  │  │  │  │  │  │  │  │
           │  │  │  │  │  │  │  │  │  └── Others execute
           │  │  │  │  │  │  │  │  └───── Others write
           │  │  │  │  │  │  └───────── Others read
           │  │  │  │  │  └──────────── Group execute
           │  │  │  │  └─────────────── Group write
           │  │  │  └────────────────── Group read
           │  │  └───────────────────── User (owner) write
           │  └──────────────────────── User (owner) read
           └─────────────────────────── File type (directory)

Each triplet position contains either the letter (r, w, x) or a dash (-) meaning "not granted."
What Each Permission Means

The meaning of r, w, and x depends on whether the file is a regular file or a directory:
Regular Files
Permission	Letter	Effect
Read	r	View file contents (cat, less, grep)
Write	w	Modify file contents (edit, overwrite, append)
Execute	x	Run the file as a program

A file with rw- can be read and edited but not executed as a program. A file with r-x can be read and executed but not modified. A file with rwx can be read, modified, and executed.
Directories

Directory permissions work differently — this trips up many beginners:
Permission	Letter	Effect
Read	r	List directory contents (ls) — see file NAMES
Write	w	Create, delete, or rename files INSIDE the directory
Execute	x	Enter/traverse the directory (cd into it, access files by name)

    💡 Directory Execute Is Not What You Think

    For directories, execute (x) means "enter." Without x on a directory:

        You can't cd into it
        You can't access files inside it even if you know their names
        ls fails with "Permission denied"

    With r but not x: you can see filenames via ls but can't cd in or access file contents.

    With x but not r: you can cd in and access files by exact name, but ls won't list contents — you must already know filenames. This is useful for semi-public drop boxes.

    With rx: normal directory access (browse and enter).

    With rwx: full control — browse, enter, create/delete files.

The Three Audiences

Linux evaluates permissions for three distinct audiences:
Audience	Position	Who They Are
User (owner)	First triplet	The individual user who owns the file
Group	Second triplet	Members of the file's group
Others	Third triplet	Everyone else on the system

Only ONE triplet applies to any given access attempt. Linux checks in order:

    Are you the owner? → Apply user triplet, ignore the rest
    Are you in the group? → Apply group triplet, ignore others
    Otherwise → Apply others triplet

# Example: File owned by john:developers, permissions rw-r--- ---
# -rwxr----- 1 john developers 2048 Jul 5 notes.txt

# If john accesses it: uses rwx (user triplet) → full control
# If mary (member of "developers" group): uses r-- (group triplet) → read only
# If bob (not john, not in developers): uses --- (others triplet) → NO ACCESS

    ⚠️ Permission Order Matters

    If you're the owner, ONLY the user triplet applies — even if group permissions would give you MORE access. Example: file is -rw----r--, owned by you. You get rw- (user triplet). Even though "others" have read access, you as owner do NOT inherit others' permissions. You get exactly what the user triplet specifies — no more, no less.

2. Ownership: Users and Groups

Every file has exactly one owner (a user) and belongs to exactly one group.
Viewing Ownership

ls -l report.pdf
# -rw-r--r--. 1 john john 1234 Jul 5 09:00 report.pdf
#               ↑    ↑    ↑
#               links owner group

Here, john owns the file, and the group is also john (Fedora creates a personal group for each user with the same name by default).
Users

Every user has:

    A username (e.g., john)
    A UID (User ID number, e.g., 1000)
    A home directory (e.g., /home/john)
    A default shell (e.g., /bin/bash)

View user information:

# Your own identity:
id
# uid=1000(john) gid=1000(john) groups=1000(john),10(wheel)

# Check a specific user:
id mary
# uid=1001(mary) gid=1001(mary) groups=1001(mary)

# List all users on the system:
cut -d: -f1 /etc/passwd
# root, bin, daemon, ..., john, mary, ...

# Examine a user's record in /etc/passwd:
grep john /etc/passwd
# john:x:1000:1000:John Smith:/home/john:/bin/bash
#  ↑    ↑    ↑     ↑        ↑         ↑          ↑
#  name pass UID   GID   GECOS field  home       shell

The /etc/passwd format:

username:password:UID:GID:GECOS:home_directory:shell

    The x in the password field means the actual hash is in /etc/shadow (readable only by root)
    GECOS is a legacy field for full name/contact info

Groups

Groups are collections of users. They simplify permission management — instead of granting access to individuals, you grant access to a group and add/remove members.

# See your groups:
groups
# john wheel

# See a specific user's groups:
groups mary
# mary wheel developers

# List all groups:
cut -d: -f1 /etc/group
# root, bin, daemon, ..., john, mary, wheel, developers, ...

# Examine a group's record:
grep wheel /etc/group
# wheel:x:10:john,mary
#         ↑  ↑  ↑
#    group name password members (comma-separated)

Important System Groups on Fedora
Group	Purpose
root	Superuser group (UID 0)
wheel	Administrative group — members can use sudo
audio	Audio device access
video	Graphics/framebuffer access
disk	Raw block device access
network	Network configuration
docker	Docker daemon access (if installed)
libvirt	Virtualization management access
input	Input device access (keyboard, mouse, tablet)
render	GPU render node access (renderD128)

# Add a user to a group (e.g., grant sudo access):
sudo usermod -aG wheel mary

# Remove from group:
sudo gpasswd -d mary wheel

    💡 Changes Require Re-login

    Group membership changes take effect on NEXT login. If you just added yourself to a group, you need to log out and log back in — or use su - $USER to start a new session that picks up the new group membership.

3. Changing Permissions (chmod)

The chmod (change mode) command modifies permission bits. Two methods exist: symbolic (letters) and numeric (octal).
Symbolic Method

Syntax: chmod [who][operator][what] file
Component	Options	Meaning
Who	u	User (owner)
	g	Group
	o	Others
	a	All (u, g, and o)
Operator	+	Add permission
	-	Remove permission
	=	Set exactly (removes others)
What	r	Read
	w	Write
	x	Execute
	X	Execute only if directory or already executable
	s	Setuid/setgid (special — see Section 5)
	t	Sticky bit (special — see Section 5)

Examples:

# Add execute permission for owner:
chmod u+x script.sh
# Before: -rw-r--r--  →  After: -rwxr--r--

# Remove write for others:
chmod o-w report.pdf
# Before: -rw-rw-rw-  →  After: -rw-rw-r--

# Set group to read-only exactly:
chmod g=r config.yml
# Before: -rwxrwxrwx  →  After: -rwxr--rwx

# Add execute for everyone:
chmod a+x program
# Before: -rw-r--r--  →  After: -rwxr-xr-x

# Multiple changes at once:
chmod u+x,g+w,o-r notes.txt
# Owner gets +x, group gets +w, others lose -r

# Recursive (entire directory tree):
chmod -R u+rwX ~/projects/
# X (capital): only add execute to directories and already-executable files

Numeric (Octal) Method

Each triplet maps to a number:
Permission	Binary	Octal
---	000	0
--x	001	1
-w-	010	2
-wx	011	3
r--	100	4
r-x	101	5
rw-	110	6
rwx	111	7

Three digits represent user, group, others:

chmod 755 script.sh     # rwxr-xr-x (owner full, group/others read+execute)
chmod 644 document.txt  # rw-r--r-- (owner read+write, others read-only)
chmod 600 secrets.key   # rw------- (owner only, nothing for others)
chmod 777 shared/       # rwxrwxrwx (EVERYONE full access — rarely appropriate!)
chmod 700 ~/.ssh        # rwx------ (private directory)
chmod 644 ~/.ssh/*.pub  # rw-r--r-- (public keys are readable)
chmod 600 ~/.ssh/id_rsa  # rw------- (private keys must be private!)

Common Permission Presets
Octal	Symbolic	Meaning	Typical Use
755	rwxr-xr-x	Owner full, others read+traverse	Directories, executable scripts
644	rw-r--r--	Owner read+write, others read	Documents, config files
600	rw-------	Owner only	Private files, SSH keys
700	rwx------	Owner only, full	Private directories (~/.ssh)
664	rw-rw-r--	Owner+group write, others read	Collaborative documents
775	rwxrwxr-x	Owner+group full, others read+traverse	Shared project directories
750	rwxr-x---	Owner full, group read+traverse, others nothing	Restricted team directories
640	rw-r-----	Owner read+write, group read, others nothing	Semi-private documents

    💡 Which Method Should You Use?

Situation	Recommended Method
Toggling one permission bit	Symbolic (chmod u+x)
Setting exact full permissions	Numeric (chmod 755)
In scripts (readability)	Numeric (concise, unambiguous)
Recursive directory changes	Symbolic with X flag (chmod -R u+rwX)
Learning/preference	Either — both produce identical results
Recursive Permission Changes

# Change all files AND directories under a path:
chmod -R 644 ~/Documents/

# Problem: 644 removes execute from directories!
# Directories NEED execute (x) to be traversable.
# Fix: use find to apply different permissions to files vs directories:

# Directories get 755, files get 644:
find ~/projects -type d -exec chmod 755 {} +
find ~/projects -type f -exec chmod 644 {} +

# Or with symbolic method:
find ~/projects -type d -exec chmod u+rwx,g+rx,o+rx {} +
find ~/projects -type f -exec chmod u+rw,g+r,o+r {} +

    ⚠️ Recursive chmod Dangers

    Running chmod -R 777 / destroys system security. Always specify a target directory, never the root filesystem. Preview with --reference or test on a small subtree first.

4. Changing Ownership (chown, chgrp)
chown: Change Owner

Only root can change file ownership (regular users cannot give files away):

# Change owner:
sudo chown mary report.pdf
# ls -l → -rw-r--r--. 1 mary john 1234 ...

# Change owner AND group:
sudo chown mary:developers report.pdf
# ls -l → -rw-r--r--. 1 mary developers 1234 ...

# Change only group (shortcut):
sudo chown :developers report.pdf

# Recursive:
sudo chown -R john:john /home/john/restored_backup/

chgrp: Change Group

A focused tool for changing only the group:

# Change group:
sudo chgrp developers project_notes.md

# Recursive:
sudo chgrp -R developers /shared/projects/website/

Regular users CAN change group — but only to a group they're a member of:

# If john is a member of "developers":
chgrp developers mycode.py     # Works — john is in developers
chgrp wheel mycode.py          # Fails — john is not in wheel (unless he is)

Practical Scenarios

# Web server files: owned by root, group www-data:
sudo chown -R root:nginx /var/www/mysite/
sudo chmod -R 750 /var/www/mysite/

# Shared family photos directory:
sudo mkdir /shared/family_photos
sudo chown -R john:family /shared/family_photos/
sudo chmod -R 775 /shared/family_photos/
# Family members can add/view/delete; outsiders see nothing

# Restore home directory ownership after restoring from backup:
sudo chown -R john:john /home/john/
sudo chmod 700 /home/john/.ssh

Using --reference

Copy permissions from an existing file:

# Make newfile match reference file's permissions:
chmod --reference=template.conf newfile.conf
chown --reference=template.conf newfile.conf

5. Special Permission Bits

Beyond the standard rwx triplets, three special bits add powerful — and potentially dangerous — capabilities.
Setuid (Set User ID)

The setuid bit makes an executable run with the privileges of its owner rather than the user running it.

-rwsr-xr-x. 1 root root 34576 Jul 5 /usr/bin/passwd
       ↑
       s = setuid is SET

When a regular user runs passwd, the program needs to modify /etc/shadow (owned by root, readable only by root). The setuid bit makes passwd execute as root temporarily, allowing this modification safely.

# Find all setuid binaries:
find /usr/bin -perm -u+s
# Common results:
# /usr/bin/sudo
# /usr/bin/passwd
# /usr/bin/su
# /usr/bin/newgrp
# /usr/bin/chsh
# /usr/bin/gpasswd

    ⚠️ setuid Security Risks

    A setuid root program runs with FULL root privileges. If such a program has a vulnerability, an attacker exploiting it gains root access. This is why setuid binaries are scrutinized and kept minimal. Never set setuid on scripts or custom programs without understanding the implications.

Setting setuid:

# Symbolic:
chmod u+s myprogram

# Numeric: prepend 4 to octal:
chmod 4755 myprogram    # setuid + 755

# Remove:
chmod u-s myprogram

Setgid (Set Group ID)

Setgid on an executable makes it run with the group of its group owner. Setgid on a directory makes new files created inside inherit the directory's group (instead of the creator's group).

drwxrwsr-x. 2 john developers 4096 Jul 5 /shared/projects/
      ↑
      s = setgid on directory

This is essential for collaborative directories:

# Create shared project directory:
sudo mkdir /shared/projects
sudo chown root:developers /shared/projects
sudo chmod 2775 /shared/projects    # setgid + rwxrwxr-x
#                                              ↑
#                                   2 = setgid prepended

# Now any file created inside inherits "developers" group:
cd /shared/projects
touch newfile.txt
ls -l newfile.txt
# -rw-r--r--. 1 john developers 0 Jul 5 newfile.txt
#                    ↑
#            Group is "developers" (inherited from directory)
#            NOT "john" (creator's personal group)

Without setgid, the file would be owned by john:john, and other developers group members couldn't collaborate without permission adjustments.
Sticky Bit

The sticky bit on a directory prevents users from deleting or renaming files they don't own — even if they have write permission on the directory.

drwxrwxrwt. 2 root root 4096 Jul 5 /tmp
             ↑
             t = sticky bit is SET

/tmp is world-writable (anyone can create files), but the sticky bit ensures only file owners (or root) can delete their own files. Without it, any user could delete any other user's temporary files — chaos.

# Set sticky bit:
chmod +t /shared/upload_area

# Numeric: prepend 1:
chmod 1777 /shared/upload_area

# Remove:
chmod -t /shared/upload_area

Summary of Special Bits
Bit	Symbolic	Numeric Prefix	Effect on Files	Effect on Directories
Setuid	u+s	4	Runs as file OWNER	(rarely used)
Setgid	g+s	2	Runs as file GROUP	New files inherit directory's group
Sticky	+t	1	(legacy, ignored)	Only file owner can delete their files

Combined octal: prepend the special bits digit to the standard three-digit octal:

chmod 4755 program    # setuid + rwxr-xr-x
chmod 2755 directory  # setgid + rwxr-sr-x
chmod 1777 /tmp       # sticky + rwxrwxrwt
chmod 6755 program    # setuid + setgid + rwxr-sr-x (rare)

6. Umask: Default Permissions

When you create a file or directory, Linux assigns default permissions. These are determined by your umask (user file creation mask).
How Umask Works

The umask SUBTRACTS permissions from the base defaults:
Type	Base Permissions	Default umask	Result
File	666 (rw-rw-rw-)	022	644 (rw-r--r--)
Directory	777 (rwxrwxrwx)	022	755 (rwxr-xr-x)

Fedora's default umask for regular users is 022, producing:

    Files: rw-r--r-- (644) — owner can read/write, others can read
    Directories: rwxr-xr-x (755) — owner full, others can read/traverse

# Check current umask:
umask
# 0022

# Set temporary umask (current session only):
umask 077
# Now files created as: 600 (rw-------)
# Directories created as: 700 (rwx------)

# Create a file and verify:
touch test_umask.txt
ls -l test_umask.txt
# -rw-------. 1 john john 0 Jul 5 test_umask.txt  ← 600

Common Umask Values
Umask	File Result	Directory Result	Security Level
022	644 rw-r--r--	755 rwxr-xr-x	Standard (others can read)
077	600 rw-------	700 rwx------	Private (others see nothing)
007	660 rw-rw----	770 rwxrwx---	Group-private (group full, others nothing)
027	640 rw-r-----	750 rwxr-x---	Restrictive (group read, others nothing)
Making Umask Permanent

Add to your shell configuration:

# For bash users:
echo 'umask 077' >> ~/.bashrc
source ~/.bashrc

# System-wide default (affects all users):
sudo nano /etc/bashrc
# Look for existing umask line and change it
# Or add to /etc/profile:
echo 'umask 022' | sudo tee -a /etc/profile

    💡 Files Never Get Execute by Default

    Notice the base for files is 666, not 777. This is deliberate — newly created files should never be executable by default. You explicitly grant execute permission with chmod +x when you want to run something. This prevents accidental execution of data files (like emails or downloaded documents containing malicious code).

7. ACLs: Beyond Traditional Permissions

Traditional Unix permissions have a limitation: exactly three audiences (user, group, others). What if you need to give a SPECIFIC user access to a file without changing ownership or creating a group?

ACLs (Access Control Lists) solve this. They allow per-user and per-group permission overrides on individual files.
Checking ACL Support

Fedora's default Btrfs filesystem supports ACLs natively:

# Check if ACLs are enabled:
getfacl test_file.txt
# If you see extended entries, ACLs are working

Viewing ACLs

# View ACL on a file:
getfacl report.pdf

# File: report.pdf
# owner: john
# group: john
# user::rw-
# group::r--
# other::r--

Without any custom ACLs, this output matches what ls -l shows.
Setting ACLs

# Give user mary read access to a specific file:
setfacl -m u:mary:r report.pdf

# Give group developers read+write:
setfacl -m g:developers:rw shared_doc.md

# Give user bob full access:
setfacl -m u:bob:rwx /shared/projects/website/

# Remove a specific ACL entry:
setfacl -x u:mary report.pdf

# Remove ALL ACLs:
setfacl -b report.pdf

After setting an ACL, ls -l shows a + after the permission string:

ls -l report.pdf
# -rw-r--r--+ 1 john john 1234 Jul 5 report.pdf
#           ↑
#           + indicates ACL entries exist

View the full ACL:

getfacl report.pdf
# File: report.pdf
# owner: john
# group: john
# user::rw-
# user:mary:r--          ← Mary has read access
# group::r--
# other::r--

Recursive ACLs

# Apply ACL to entire directory tree:
setfacl -R -m g:developers:rx /shared/projects/

# Set default ACL (new files created inside inherit these):
setfacl -d -m g:developers:rwx /shared/projects/

Default ACLs are powerful for collaborative directories — new files automatically get the right group permissions without manual chmod calls.
When to Use ACLs
Scenario	Use ACL?
Standard permissions suffice	No — use chmod
Need different permissions for multiple specific users	Yes
Collaborative directory with group inheritance	Consider setgid first, ACLs if more complex
Temporary access grant for one user	Yes — clean removal with setfacl -x

    💡 ACLs and SELinux

    ACLs extend the traditional Unix permission model. SELinux (covered next) is a separate, additional layer. They're complementary, not competing. A process must pass BOTH ACL/permission checks AND SELinux policy checks to access a file.

8. SELinux: Fedora's Extra Security Layer

This is what makes Fedora (and all Red Hat-based distributions) stand apart. SELinux is arguably the most misunderstood component of Linux security. Let's demystify it.
What Is SELinux?

SELinux (Security-Enhanced Linux) is a mandatory access control (MAC) system built into the Linux kernel. It was originally developed by the NSA (National Security Agency) and released to the open source community.

Traditional Unix permissions are discretionary — file owners choose who accesses their files (DAC — Discretionary Access Control). SELinux adds mandatory controls — the system administrator (or policy) defines security rules that even file owners cannot override.

Think of it as a second checkpoint:

Process tries to access file
         │
         ▼
   ┌─────────────────┐
   │  PERMISSION CHECK │  ← Traditional Unix (owner/group/other rwx)
   │  (Discretionary) │
   └────────┬────────┘
            │ PASS
            ▼
   ┌─────────────────┐
   │   SELinux CHECK  │  ← Policy-based (label/type rules)
   │   (Mandatory)    │
   └────────┬────────┘
            │ PASS
            ▼
      ACCESS GRANTED

Both checks must pass. Even if traditional permissions allow access, SELinux can deny it if the policy doesn't permit the interaction.
Why Fedora Uses SELinux

Without SELinux, a compromised process can do enormous damage. Example: if a web server (running as user apache) is hacked, the attacker can:

    Read files accessible to apache user
    Write to any directory apache has write access to
    Potentially escalate privileges

With SELinux, the web server process runs with a type label (like httpd_t). SELinux policy defines EXACTLY what httpd_t can do:

    Read files labeled httpd_sys_content_t (web content)
    Bind to ports 80, 443
    Write to directories labeled httpd_sys_rw_content_t
    Cannot read /home files (labeled user_home_t)
    Cannot read /etc/shadow (labeled shadow_t)
    Cannot bind to non-web ports
    Cannot execute most system binaries

Even if traditional permissions allowed it, SELinux blocks it.
SELinux Modes

SELinux has three operating modes:
Mode	Behavior	Use Case
Enforcing	Denies and logs policy violations	Production (Fedora default)
Permissive	Allows but logs violations (audit log fills)	Debugging/troubleshooting
Disabled	Completely turned off	Not recommended on Fedora

# Check current mode:
getenforce
# Enforcing

# Detailed status:
sestatus
# SELinux status:                 enabled
# SELinuxfs mount:                /sys/fs/selinux
# SELinux root directory:         /etc/selinux
# Loaded policy name:             targeted
# Current mode:                   enforcing
# Mode from config file:          enforcing
# Policy MLS status:              enabled
# Policy deny_unknown status:     allowed

Temporarily Changing Modes

# Switch to permissive (temporary — resets on reboot):
sudo setenforce 0
getenforce
# Permissive

# Switch back to enforcing:
sudo setenforce 1
getenforce
# Enforcing

Permanently Changing Modes

# Edit configuration file:
sudo nano /etc/selinux/config
# Change: SELINUX=enforcing
# To:     SELINUX=permissive

# Reboot required for permanent changes:
sudo reboot

    ⚠️ Never Disable SELinux

    Disabling SELinux is strongly discouraged on Fedora. It weakens system security significantly and some applications may not function correctly when SELinux is re-enabled after being disabled (labels become uninitialized). If you encounter issues, switch to permissive mode temporarily — it logs what would be blocked without actually blocking anything. Fix the underlying issue, then return to enforcing.

SELinux Contexts (Labels)

Every file, process, and port has a SELinux context (also called a label). View it:

# View file contexts:
ls -Z /var/www/html/index.html
# unconfined_u:object_r:httpd_sys_content_t:s0

# View context of running processes:
ps -eZ | grep httpd
# system_u:system_r:httpd_t:s0    1234 ?  00:00:01 httpd

# View your own process context:
ps -eZ | grep $$ | head -1

Context format: user:role:type:level

For most practical purposes, the type field (third part) is what matters:
Common Type	What It Labels
user_home_t	Files in user home directories
httpd_sys_content_t	Web server content (read-only by httpd)
httpd_sys_rw_content_t	Web server content (read-write by httpd)
var_log_t	Log files in /var/log
etc_t	Configuration files in /etc
bin_t	Executable binaries in /usr/bin
usr_lib_t	Libraries in /usr/lib
tmp_t	Temporary files in /tmp
ssh_home_t	SSH-related files in home directories
Changing File Contexts

Sometimes you need to relabel files so SELinux allows specific services to access them:

# Label a file as web content:
sudo semanage fcontext -a -t httpd_sys_content_t "/custom/web/path(/.*)?"
sudo restorecon -Rv /custom/web/path/

# Label a directory for SSH keys:
sudo semanage fcontext -a -t ssh_home_t "/home/john/.ssh/custom_keys(/.*)?"
sudo restorecon -Rv /home/john/.ssh/custom_keys/

The semanage fcontext command makes the label persistent (survives relabeling). The restorecon command applies the label immediately.

# Quick temporary relabel (doesn't survive policy updates):
sudo chcon -t httpd_sys_content_t /var/www/html/newfile.html

# Verify:
ls -Z /var/www/html/newfile.html
# unconfined_u:object_r:httpd_sys_content_t:s0

SELinux Booleans

SELinux policies include boolean toggles — switches that enable/disable specific policy rules without rewriting the entire policy:

# List all booleans with descriptions:
sudo semanage boolean -l

# View current boolean states:
getsebool -a

# Common booleans:
getsebool httpd_can_network_connect
# httpd_can_network_connect --> off

getsebool httpd_enable_homedirs
# httpd_enable_homedirs --> off

Toggle booleans:

# Allow web server to make network connections:
sudo setsebool -P httpd_can_network_connect on

# Allow web server to read home directories:
sudo setsebool -P httpd_enable_homedirs on

# Allow SSH to use pam_namespace:
sudo setsebool -P ssh_use_tcpd on

The -P flag makes the change persistent across reboots.
Troubleshooting SELinux Denials

When SELinux denies access, it logs an AVC (Access Vector Cache) denial. These appear in the audit log:

# View SELinux denials:
sudo ausearch -m avc -ts recent

# Or use the dedicated tool:
sudo sealert -a /var/log/audit/audit.log
# Provides human-readable explanations and suggested fixes

If you suspect SELinux is blocking something:

# 1. Check if SELinux is the culprit:
sudo setenforce 0    # Switch to permissive
# Try the operation again — if it works now, SELinux was blocking it

# 2. Check audit logs for AVC denials:
sudo ausearch -m avc -ts recent

# 3. Generate a human-readable report:
sudo sealert -a /var/log/audit/audit.log

# 4. Apply the suggested fix (often a setsebool or restorecon command)

# 5. Return to enforcing:
sudo setenforce 1

    💡 Don't Immediately Disable SELinux

    When something doesn't work and you find SELinux is involved, resist the urge to disable it permanently. Instead:

        Switch to permissive temporarily
        Identify the denial using ausearch or sealert
        Apply the suggested fix (usually a boolean toggle or file relabel)
        Return to enforcing

    This keeps your system secure while resolving the issue properly.

9. Practical Permission Scenarios

Let's apply everything you've learned to real situations.
Scenario 1: Making a Script Executable

You wrote a backup script. Trying to run it fails:

./backup.sh
# bash: ./backup.sh: Permission denied

Diagnosis and fix:

ls -l backup.sh
# -rw-r--r-- 1 john john 1024 Jul 5 backup.sh
#        ↑
#   No execute permission!

# Add execute for the owner:
chmod u+x backup.sh
# Or more explicitly:
chmod 755 backup.sh

ls -l backup.sh
# -rwxr-xr-x 1 john john 1024 Jul 5 backup.sh
#  ↑
# Now executable

./backup.sh    # Works!

Scenario 2: SSH Key Permissions

SSH refuses to connect, complaining about key file permissions:

ssh -i ~/.ssh/id_rsa server
# Permissions 0644 for '/home/john/.ssh/id_rsa' are too open.

SSH demands private keys be readable ONLY by the owner:

chmod 700 ~/.ssh/
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub

ls -l ~/.ssh/
# -rw------- 1 john john 432 Jul 5 id_rsa
# -rw-r--r-- 1 john john 105 Jul 5 id_rsa.pub

Scenario 3: Sharing a Directory Between Users

John and Mary need a shared directory for collaborative work:

# Create the shared directory:
sudo mkdir /shared/projects

# Create a group and add both users:
sudo groupadd devteam
sudo usermod -aG devteam john
sudo usermod -aG devteam mary

# Set ownership and permissions:
sudo chown root:devteam /shared/projects
sudo chmod 2770 /shared/projects
# 2 = setgid (new files inherit devteam group)
# 770 = rwxrwx--- (owner and group full, others nothing)

# Both users must log out and back in for group membership to take effect

# After re-login, test:
# As john:
cd /shared/projects
echo "hello from john" > john_file.txt

# As mary:
cd /shared/projects
cat john_file.txt         # Should work (group read)
echo "hello from mary" > mary_file.txt
# john can read mary_file.txt too (both in devteam group)

Scenario 4: Web Server Content

Setting up web content for Apache/Nginx:

# Create web directory:
sudo mkdir -p /var/www/mysite

# Set ownership (root owns, nginx/apache group has access):
sudo chown -R root:nginx /var/www/mysite/

# Set permissions:
sudo find /var/www/mysite -type d -exec chmod 750 {} +
sudo find /var/www/mysite -type f -exec chmod 640 {} +
# Directories: rwxr-x--- (nginx group can traverse)
# Files: rw-r----- (nginx group can read)

# Label for SELinux:
sudo semanage fcontext -a -t httpd_sys_content_t "/var/www/mysite(/.*)?"
sudo restorecon -Rv /var/www/mysite/

Scenario 5: Restoring Permissions After a Backup

After restoring files from a backup, ownership and permissions may be wrong:

# Fix home directory ownership:
sudo chown -R john:john /home/john/

# Restore standard permissions:
find /home/john -type d -exec chmod 755 {} +
find /home/john -type f -exec chmod 644 {} +

# Fix critical private directories:
chmod 700 /home/john/.ssh
chmod 600 /home/john/.ssh/id_rsa
chmod 644 /home/john/.ssh/id_rsa.pub
chmod 600 /home/john/.ssh/config

# Restore SELinux contexts:
sudo restorecon -Rv /home/john/

Permission-Related Commands Cheat Sheet
Command	Purpose
ls -l	View permissions and ownership
ls -ld	View directory permissions (not contents)
ls -Z	View SELinux contexts
chmod	Change permissions (mode)
chown	Change owner (and optionally group)
chgrp	Change group
umask	View/set default permission mask
stat	View detailed file metadata including permissions
getfacl	View ACL entries
setfacl	Set/modify ACL entries
getenforce	View SELinux mode
setenforce	Temporarily change SELinux mode
sestatus	Detailed SELinux status
getsebool	View SELinux boolean states
setsebool	Toggle SELinux booleans
semanage fcontext	Manage SELinux file context rules
restorecon	Restore default SELinux labels
chcon	Temporarily change SELinux label
ausearch	Search audit logs for denials
sealert	Generate human-readable SELinux denial reports
Try It Yourself
Exercise 1: Reading Permissions

# Navigate to various directories and decode permission strings:
ls -l /etc/passwd        # What type? Who can read? Write?
ls -ld /tmp              # Sticky bit? Who can create files here?
ls -l /usr/bin/sudo      # Setuid? What does that mean?
ls -ld /home/$USER       # Who can access your home directory?
ls -Z /var/www/html/     # What SELinux type labels web content?

Write down your interpretation for each, then verify using stat and getfacl.
Exercise 2: Changing Permissions

# Create a test script:
echo '#!/bin/bash
echo "Hello from executable script"' > ~/hello.sh

# Try running it (will fail — no execute permission):
~/hello.sh

# Add execute permission:
chmod +x ~/hello.sh

# Run it:
~/hello.sh

# Now remove ALL permissions and observe:
chmod 000 ~/hello.sh
cat ~/hello.sh     # Permission denied!
chmod 644 ~/hello.sh  # Restore read access
cat ~/hello.sh     # Works again

Exercise 3: Ownership and Groups

# View your current user and group info:
id
groups

# Create a file and examine ownership:
touch ~/ownership_test.txt
ls -l ~/ownership_test.txt

# Try changing ownership WITHOUT sudo (should fail):
chown root ~/ownership_test.txt
# chown: changing ownership of 'ownership_test.txt': Operation not permitted

# With sudo:
sudo chown root:root ~/ownership_test.txt
ls -l ~/ownership_test.txt
# Now owned by root — you can still read it (644) but can't write to it

# Restore:
sudo chown $USER:$USER ~/ownership_test.txt

Exercise 4: Special Bits

# Create a test directory and set sticky bit:
mkdir ~/sticky_test
chmod 1777 ~/sticky_test
ls -ld ~/sticky_test
# drwxrwxrwt — note the 't' at the end

# Create two test users (if you have sudo access):
sudo useradd -m alice
sudo useradd -m bob

# As alice, create a file in the sticky directory:
sudo -u alice touch ~/sticky_test/alice_file.txt

# As bob, try to delete alice's file (should FAIL due to sticky bit):
sudo -u bob rm ~/sticky_test/alice_file.txt
# rm: cannot remove 'alice_file.txt': Operation not permitted

# Clean up:
sudo userdel -r alice
sudo userdel -r bob
rm -rf ~/sticky_test

Exercise 5: Umask Experiment

# Check current umask:
umask

# Create a file and note permissions:
touch before.txt
ls -l before.txt

# Change umask to restrictive:
umask 077
touch after.txt
ls -l after.txt
# Note: after.txt has 600 (rw-------) instead of 644

# Change umask to group-collaborative:
umask 007
touch collab.txt
ls -l collab.txt
# Note: collab.txt has 660 (rw-rw----)

# Restore default:
umask 022

# Clean up:
rm before.txt after.txt collab.txt

Exercise 6: SELinux Exploration

# Check SELinux status:
sestatus

# View SELinux contexts on common files:
ls -Z /etc/passwd
ls -Z /var/log/messages
ls -Z /home/$USER/

# View contexts on running processes:
ps -eZ | head -10

# List SELinux booleans related to web serving:
getsebool -a | grep httpd

# If you have any AVC denials, view them:
sudo ausearch -m avc -ts recent 2>/dev/null || echo "No recent AVC denials"

Troubleshooting Common Permission Issues
Symptom	Likely Cause	Fix
"Permission denied" running a script	Missing execute permission	chmod +x script.sh
"Permission denied" accessing a directory	Missing execute (x) on directory	chmod +x directory/
Can't write to a file you own	File is read-only (missing write)	chmod u+w file.txt
SSH rejects your private key	Key permissions too open	chmod 600 ~/.ssh/id_rsa
Can't delete file in shared directory	Sticky bit prevents it (you're not owner)	Ask file owner to delete it, or use sudo
Web server can't read your files	Wrong SELinux context	sudo restorecon -Rv /path/ or sudo chcon -t httpd_sys_content_t /path/
After backup restore, things break	Ownership/permissions lost	chown -R user:user /home/user/ + restorecon -Rv
"Operation not permitted" despite correct permissions	SELinux denying access	Check ausearch -m avc, use sealert for guidance
Can't access files after moving them	Different SELinux context on new location	sudo restorecon -Rv /new/location/
What's Next

You now command the full spectrum of Linux access control: traditional Unix permissions (read/write/execute for user/group/others), ownership management (chown, chgrp), special permission bits (setuid, setgid, sticky), umask default permissions, access control lists for fine-grained per-user overrides, and SELinux mandatory access control — Fedora's powerful additional security layer that keeps processes confined to their intended scope.

Chapter 17 takes us into user and group management — creating new user accounts, configuring sudo access, managing password policies, understanding the login process, and administering multi-user systems. These skills pair directly with what you've learned here: ownership and permissions only make sense when you understand the users and groups they apply to.

Until then, practice reading permission strings everywhere you go. Every ls -l output is a mini security audit. Decode it mentally. The fluency will come faster than you think.

    ✅ Chapter 16 Complete You understand the complete permission string (file type character + three triplets for user/group/others), the different meanings of read/write/execute for files versus directories, permission evaluation order (user first, then group, then others — only one triplet applies), ownership concepts (users with UIDs, groups with GIDs, /etc/passwd and /etc/group formats), changing permissions via both symbolic notation (chmod u+x, g-w, o=r) and numeric octal (755, 644, 600, 700), recursive permission operations with safety considerations, changing ownership with chown and chgrp (including recursive and reference modes), three special permission bits (setuid for privileged execution, setgid for group inheritance on directories, sticky bit for protected shared directories like /tmp), umask default permission calculation and configuration, access control lists (getfacl/setfacl for per-user permission overrides, default ACLs for inherited directory permissions), SELinux fundamentals (mandatory access control, enforcing/permissive/disabled modes, file context labels, boolean toggles, AVC denial troubleshooting with ausearch/sealert, semanage fcontext/restorecon for persistent label management), nine practical real-world permission scenarios with step-by-step solutions, a complete command reference table, and systematic troubleshooting guidance for the most common permission-related problems.

