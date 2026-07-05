# Chapter 17: Users, Groups, and sudo

## The Multi-User Foundation

Here's something that might surprise you: Linux was designed from the ground up as a **multi-user operating system.** Windows and macOS started as single-user systems and bolted on multi-user features later. Linux started multi-user and has been that way since day one.

This isn't just historical trivia — it fundamentally shapes how Linux works. Every file has an owner. Every process belongs to a user. Every action is checked against "who are you, and what are you allowed to do?" These checks happen invisibly, thousands of times per second, and they're the reason Linux is considered more secure than single-user operating systems.

This chapter explains the user and group system, how `sudo` works (including Ubuntu 26.04's new Rust-based sudo), and how to manage user accounts from the terminal.

## The Concept of Users

Every person (or service) that interacts with a Linux system does so as a **user**. A user is an identity — a named account with specific permissions and a home directory.

### Types of Users

There are two main categories:

**Regular users** — You, your family members, anyone who logs in to use the computer. Each has:
- A username (e.g., `john`)
- A user ID number (UID) — the system's internal identifier
- A home directory (`/home/john/`)
- A password
- Membership in one or more groups
- Specific permissions (what they can read, write, execute)

**The root user** — Also called the "superuser" or "administrator." Root is a special account with **no restrictions.** Root can read, write, or delete any file on the system, start or stop any service, and override any permission. Root's UID is always 0, and its home directory is `/root/` (not `/home/root/`).

> 💡 **Root vs. Administrator on Windows**
>
> On Windows, the "Administrator" account is powerful but still operates within the Windows security model. On Linux, root is truly unrestricted — there are no guardrails. This is why Linux deliberately discourages logging in as root. Instead, regular users temporarily elevate to root-level privileges using `sudo` (explained below) for individual commands.

### System Users

Beyond regular users and root, Linux has **system users** — accounts created for specific services, not for humans. Examples:

| System User | What It Runs |
|---|---|
| `www-data` | Web server (Apache/Nginx) |
| `mysql` | MySQL database |
| `postgres` | PostgreSQL database |
| `nobody` | Unprivileged sandbox account |
| `syslog` | System logger |
| `snapd` | Snap package daemon |

Each service runs under its own user account so that if it's compromised, the attacker only gains that service's limited permissions — not full system access. This is called **privilege separation**, and it's a core Linux security principle.

### Viewing User Information

**Who am I?**

whoami

Output:

john

Detailed user info:

id

Output:

uid=1000(john) gid=1000(john) groups=1000(john),27(sudo),119(plugdev),132(lpadmin),142(sambashare)

Field	Meaning
uid=1000(john)	Your user ID is 1000; your username is john
gid=1000(john)	Your primary group ID is 1000 (a group named "john")
groups=...	All groups you belong to

Regular users on Ubuntu start at UID 1000. System users typically have UIDs below 1000.

Who's logged in?

who

Lists all currently logged-in users and their terminal sessions.

Everyone's last login:

lastlog

Shows the last login time for every user account on the system.
The /etc/passwd File

All user account information is stored in /etc/passwd. Despite the alarming name, it doesn't contain passwords anymore (those are encrypted in /etc/shadow). It's a plain-text file you can read:

cat /etc/passwd

Each line represents one user:

john:x:1000:1000:John Smith,,,:/home/john:/bin/bash

Field	Value	Meaning
Username	john	Login name
Password placeholder	x	Indicates password is in /etc/shadow
UID	1000	User ID number
GID	1000	Primary group ID
GECOS	John Smith,,,	Full name and comments
Home directory	/home/john	Where the user's personal files live
Shell	/bin/bash	Default shell (or /usr/sbin/nologin for system users)

    ⚠️ Don't Edit /etc/passwd Directly

    While the file is readable, never edit it manually with a text editor. A syntax error in /etc/passwd can lock you out of your system entirely. Always use the user management commands (useradd, usermod, etc.) described later in this chapter.

The Concept of Groups

Groups are collections of users. They simplify permission management — instead of granting access to each user individually, you add users to a group and grant permissions to the group.
How Groups Work

Every user has exactly one primary group — typically a group with the same name as the user. When you create a file, it's owned by your user and your primary group.

Users can also belong to secondary groups — additional groups that grant specific privileges. For example, being in the sudo group lets you use sudo; being in the lpadmin group lets you manage printers.
Common Groups on Ubuntu
Group	What Membership Allows
sudo	Execute commands with administrator privileges
adm	View system logs
cdrom	Access CD/DVD drives
plugdev	Access removable storage (USB drives)
lpadmin	Manage printers
sambashare	Share folders via Samba (Windows file sharing)
docker	Run Docker containers (Chapter 23)
www-data	Web server group
audio	Access audio hardware
video	Access video/display hardware
input	Access input devices (touchpads, tablets)
Viewing Group Information

Your groups:

groups

Output:

john sudo adm cdrom plugdev lpadmin sambashare

Details about a specific group:

getent group sudo

Output:

sudo:x:27:john,mary

This shows the group name, GID (27), and members (john, mary).

All groups on the system:

cat /etc/group

Each line follows the format groupname:x:GID:member1,member2,...
sudo: Temporary Power

sudo (superuser do) is how regular users execute commands with root-level privileges — temporarily, for a single command, without logging in as root.
How sudo Works

When you type:

sudo apt update

    The system checks if your user is in the sudo group (or is explicitly listed in /etc/sudoers)
    If yes, it prompts for your password (not root's password)
    You type your password (nothing appears on screen — this is normal)
    The system verifies the password
    The command runs with root privileges
    After ~15 minutes, the privilege expires and you'll be asked again

    💡 Why Not Just Log In as Root?

    Logging in as root and running all commands with full privileges is dangerous:

        No safety net: Every command is executed with zero restrictions — a typo like rm -rf / would succeed
        No audit trail: You can't tell who ran what — everything shows as "root"
        Accidental damage: Root can overwrite critical system files without warning

    sudo solves all three problems: you're only root for one command at a time, each sudo command is logged with the username, and you must consciously type sudo before dangerous commands.

sudo-rs: The Rust-Based sudo (Ubuntu 26.04)

Ubuntu 26.04 ships with sudo-rs — a complete rewrite of sudo in the Rust programming language. This replaces the traditional C-based sudo that has been the standard for decades.

Why the change?

The original sudo is written in C — a powerful language with a critical weakness: it doesn't protect against memory safety bugs. Over the years, numerous security vulnerabilities in sudo allowed attackers to escalate privileges or crash the system, all caused by memory handling errors that Rust prevents by design.

Rust's compile-time checks eliminate entire categories of bugs (buffer overflows, use-after-free, null pointer dereferences) that have historically been exploited in security-critical software like sudo.

What does this mean for you?

Practically nothing — and that's the point. sudo-rs is designed as a drop-in replacement:

    The command is still sudo — you type it exactly the same way
    The password prompt works identically
    The /etc/sudoers configuration file is the same
    All existing rules and permissions work unchanged
    The 15-minute timeout is the same

The only difference is what's running under the hood — a safer, more robust implementation.

Verifying your sudo version:

sudo --version

If it mentions "sudo-rs" or shows a Rust-based version string, you're on the new implementation. If you ever need to switch back to the classic C-based sudo (unlikely for a beginner):

sudo update-alternatives --set sudo /usr/bin/sudo.ws

    💡 What Is Rust?

    Rust is a modern systems programming language that has been gaining massive adoption in the Linux world. Its key feature is memory safety — the compiler prevents common programming errors that lead to security vulnerabilities. Beyond sudo, Ubuntu 26.04 also ships Rust-based versions of core utilities (coreutils) alongside the traditional C versions. You'll hear more about Rust in the Linux ecosystem as time goes on.

Common sudo Patterns

Run a single command as root:

sudo apt install htop

Open a root shell (interactive):

sudo -i

This drops you into a root shell where every command runs with root privileges until you type exit. Use cautiously.

Run a command as a different user (not root):

sudo -u mary ls /home/mary/

Runs ls /home/mary/ as user mary.

Repeat the last command with sudo:

sudo !!

The !! expands to the previous command. Extremely useful when you forget sudo:

apt update
# Error: "Permission denied. Are you root?"
sudo !!
# Runs: sudo apt update

    💡 The Blank Password Mystery

    When you type your password at a sudo prompt, nothing appears on screen — no asterisks, no dots, no cursor movement. This is intentional. Showing asterisks would reveal the password length to anyone looking at your screen. The terminal is receiving your keystrokes; it's just not displaying them.

What Can Go Wrong with sudo

"username is not in the sudoers file"

This means your user account doesn't have administrator privileges. On Ubuntu, this happens if:

    Your user wasn't created as an administrator during installation
    Someone removed you from the sudo group
    You're logged in as a non-admin user

To fix this, you need to log in as a user with sudo privileges and add your user to the sudo group:

sudo usermod -aG sudo username

If no user has sudo access, you'll need to boot into recovery mode and fix it from a root shell. We cover this in Chapter 30 (Troubleshooting).

"sudo: unable to resolve host"

Your hostname doesn't match your /etc/hosts file. Usually harmless but annoying. Fix by ensuring your hostname is listed:

echo "127.0.1.1 $(hostname)" | sudo tee -a /etc/hosts

Typo'd password too many times

sudo locks you out temporarily after 3 failed attempts (by default). Wait a minute or two and try again. The lockout is per-session, not permanent.
The /etc/sudoers File

The sudoers file controls who can use sudo and what they can do. On Ubuntu, membership in the sudo group grants full sudo access — this is configured in the sudoers file:

%sudo   ALL=(ALL:ALL) ALL

Breaking this down:
Part	Meaning
%sudo	The group named "sudo" (the % means group)
ALL	On all hosts (applies to local machine)
(ALL:ALL)	Can run as any user and any group
ALL	Can run all commands

This line means: "Anyone in the sudo group can run any command as any user on this machine."

    ⚠️ Never Edit sudoers Directly

    Never open /etc/sudoers with nano, vim, or any regular text editor. A syntax error in this file can completely break sudo, locking you out of administrator access permanently.

    Always use:

    sudo visudo

    visudo edits the sudoers file with built-in syntax checking. When you save, it validates the file before applying changes. If there's an error, it refuses to save and shows you the problem.

Granting Limited sudo Access (Advanced)

You can create specific rules for users who need only certain commands:

sudo visudo

Add a line like:

mary ALL=(ALL) /usr/bin/apt update, /usr/bin/apt upgrade

This allows Mary to run sudo apt update and sudo apt upgrade — but nothing else. She can't install packages, remove them, or modify system files.

This level of granularity is rarely needed on personal computers but is common on servers and multi-user systems. We mention it here for awareness; detailed sudoers configuration is covered in Chapter 24 (Security).
su vs. sudo

Linux offers two ways to gain root privileges:
Tool	How It Works	When to Use
sudo command	Runs one command as root using your password	Almost always (Ubuntu default)
su -	Switches to a full root login shell using root's password	Rarely (root may not have a password set)
sudo -i	Opens a root shell using your password	When you need many root commands in sequence

On Ubuntu, the root account has no password set by default — you can't log in as root directly. This is a deliberate security choice. All administrative work goes through sudo, which uses your regular user password.

Switching to another user (if you have permission):

su - mary

Switches to Mary's account. You'd need Mary's password (or to be running as root already).

Switching to root (if root has a password):

su -

On a default Ubuntu system, this fails because root has no password. Use sudo -i instead.
Managing User Accounts

These commands require sudo. They're used when setting up accounts for family members, creating service accounts, or managing multi-user systems.
Creating a New User

sudo adduser sarah

adduser is Ubuntu's friendly wrapper around the useradd command. It interactively prompts for:

    Password (twice for confirmation)
    Full name
    Room number (optional, can be blank)
    Work phone (optional)
    Home phone (optional)
    Other (optional)

It also:

    Creates the home directory (/home/sarah/)
    Copies default config files from /etc/skel/ (skeleton directory)
    Creates a primary group named "sarah"
    Adds the user to common default groups

    💡 adduser vs. useradd

    Ubuntu has two commands for creating users:

        adduser — interactive, user-friendly, creates home directory and sets up groups automatically. Use this one.
        useradd — low-level, non-interactive, requires manual configuration of home directory and groups. It's meant for scripts, not human use.

    If you run useradd sarah by itself, it creates the account but no home directory, no password, and no group membership. adduser sarah does everything correctly. When in doubt, use adduser.

Granting sudo Access to a New User

sudo usermod -aG sudo sarah

    usermod — modify a user account
    -a — append (add to groups, don't replace existing memberships)
    -G sudo — the group to add to
    sarah — the username

The -a (append) flag is critical. Without it, -G sudo would replace all of Sarah's secondary groups with just "sudo," removing her from plugdev, cdrom, etc. Always use -aG together.

Sarah will need to log out and log back in for the group change to take effect. Alternatively, she can run newgrp sudo in her terminal to activate the new group membership immediately.
Changing a User's Password

Your own password:

passwd

Prompts for your current password, then the new password twice. Passwords must meet minimum complexity requirements.

Another user's password (requires sudo):

sudo passwd sarah

Sets a new password for Sarah. Doesn't require Sarah's current password (because you're root).
Modifying User Accounts

Change a username:

sudo usermod -l newname oldname

Change a user's home directory:

sudo usermod -d /home/newdir -m sarah

The -m flag moves existing files to the new location.

Lock an account (prevent login):

sudo usermod -L sarah

Unlock an account:

sudo usermod -U sarah

Set an account expiry date:

sudo usermod -e 2026-12-31 sarah

Sarah's account expires on December 31, 2026. After that date, she can't log in.
Deleting a User

Remove the user but keep their files:

sudo deluser sarah

Remove the user AND delete their home directory:

sudo deluser --remove-home sarah

    ⚠️ Irreversible Deletion

    --remove-home permanently deletes all of the user's personal files. If there's any chance you'll need their data, back up /home/sarah/ first:

    sudo tar czf /root/sarah_backup.tar.gz /home/sarah/

    Then proceed with deluser --remove-home.

File Ownership Revisited

In Chapter 14, you learned about file permissions (read, write, execute). Now let's connect that to users and groups.

Every file and directory on Linux is owned by exactly one user and one group. When you create a file:

touch myproject.txt
ls -l myproject.txt

Output:

-rw-r--r-- 1 john john 0 Jul 4 14:30 myproject.txt

Here:

    Owner: john (your user)
    Group: john (your primary group)
    Permissions: owner can read/write, group can read, others can read

Changing File Ownership

Change the owner:

sudo chown mary myproject.txt

Now Mary owns the file. Her permissions change from "other" to "owner."

Change owner and group:

sudo chown mary:developers myproject.txt

Mary now owns it, and the "developers" group has group-level permissions.

Recursively change ownership of a directory:

sudo chown -R mary:developers /shared/project/

The -R flag applies the change to the directory and everything inside it.
Changing File Group (Without Changing Owner)

sudo chgrp developers myproject.txt

Changes only the group, leaving the owner unchanged. This is useful when you want to grant group access without transferring ownership.
Practical Scenario: Setting Up a Shared Directory

Let's combine everything from this chapter into a realistic scenario:

Goal: Create a shared directory that both you and another user can read and write to.

# 1. Create a group for shared access
sudo groupadd shared

# 2. Add both users to the group
sudo usermod -aG shared john
sudo usermod -aG shared mary

# 3. Create the shared directory
sudo mkdir /shared

# 4. Change ownership to root:shared
sudo chown root:shared /shared

# 5. Give group read/write/execute access
sudo chmod 775 /shared

# 6. Set the group for new files created inside (SGID bit)
sudo chmod g+s /shared

# 7. Verify
ls -ld /shared

Step 6 (chmod g+s) sets the SGID bit — any new file created inside /shared automatically inherits the "shared" group, so both users can always access each other's files. Without this, new files would be owned by whoever created them and their primary group, not "shared."

After step 7, you should see:

drwxrwsr-x 2 root shared 4096 Jul 4 14:30 /shared

Notice the s in the group permission field (rws instead of rwx) — that's the SGID indicator.
User Management Cheat Sheet

For quick reference:
Task	Command
See who you are	whoami
See your groups	groups
See your UID/GID	id
Create a new user	sudo adduser username
Delete a user	sudo deluser username
Delete user and home dir	sudo deluser --remove-home username
Grant sudo access	sudo usermod -aG sudo username
Change password (self)	passwd
Change password (another user)	sudo passwd username
Add user to a group	sudo usermod -aG groupname username
Remove user from a group	sudo gpasswd -d username groupname
Create a new group	sudo groupadd groupname
Delete a group	sudo groupdel groupname
Change file owner	sudo chown username file
Change file group	sudo chgrp groupname file
Open root shell	sudo -i
Repeat last command with sudo	sudo !!
Edit sudoers safely	sudo visudo
What's Next

You now understand the user and group system that underpins all of Linux security. You know how sudo works (including Ubuntu 26.04's Rust-based sudo-rs), how to create and manage user accounts, how to control file ownership, and how to set up shared access between users.

In Chapter 18, we'll return to package management — this time from the terminal. You'll learn APT in depth: installing, updating, searching, and removing packages with precise control. Everything you did graphically in Chapter 10, you'll now be able to do faster and more flexibly from the command line.
Try It Yourself

    Inspect your identity. Run whoami, id, and groups. Understand what each output tells you about your user account.

    Look at system users. Run cat /etc/passwd | less. Scroll through the file. Identify regular users (UID 1000+) versus system users. Notice which ones have /usr/sbin/nologin as their shell — these are service accounts that can't log in interactively.

    Check your sudo version. Run sudo --version. See if it mentions sudo-rs or Rust. You're using the new memory-safe implementation.

    Practice sudo !!. Type apt update (without sudo), see the permission error, then run sudo !! to repeat with sudo. This trick saves enormous time once it becomes muscle memory.

    Create a test user. Run sudo adduser testuser and follow the prompts. Then check /home/testuser/ exists. Add the user to the sudo group with sudo usermod -aG sudo testuser. When done, remove them: sudo deluser --remove-home testuser.

    View the sudoers file (safely). Run sudo visudo. Look at the rules but don't change anything. Find the %sudo line. Press q (or use your editor's quit command) to exit without saving.

    Experiment with file ownership. Create a file with touch test.txt. Check its owner with ls -l test.txt. Change its group with sudo chgrp sudo test.txt. Verify the change. Then change it back.

    ✅ Chapter 17 Complete You understand Linux's multi-user model, how users and groups work, what sudo does (and why Ubuntu 26.04 uses the Rust-based sudo-rs), how to create and manage user accounts, how to control file ownership with chown and chgrp, and how to safely configure administrative privileges with visudo. In Chapter 18, we'll master terminal-based package management with APT.
