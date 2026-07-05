# Chapter 6: Installing Fedora

## The Moment of Commitment

You've downloaded the ISO, verified it, created a bootable USB, and explored the live environment. Your hardware works. You're ready.

This chapter walks through the entire installation process — from launching the installer to your first boot into Fedora 44. We'll cover every screen, every choice, and every option. By the end, you'll have a fully installed, running Fedora system.

---

## Before You Begin: Pre-Installation Checklist

Before launching the installer, take five minutes to verify these items:

| Check | Why It Matters |
|---|---|
| **Back up your data** | If you're installing on a computer with an existing OS, back up irreplaceable files. Installation involves partitioning — mistakes are rare but irreversible. |
| **Power source connected** | If you're on a laptop, plug it in. A power failure during installation corrupts the install. |
| **Internet connection (optional)** | Not required, but helpful. The installer can download updates during installation if connected. |
| **Disk space cleared** | At least 20 GB free for a minimal install, 60 GB recommended for comfortable daily use. |
| **USB drive still inserted** | You're installing from the live USB — don't remove it yet. |
| **Time available** | The installation itself takes 15–30 minutes. Plan for an hour total including post-install configuration. |

> 💡 **To Encrypt or Not to Encrypt?**
>
> Fedora offers full disk encryption (LUKS) during installation. If your computer is a laptop, travels, or could be stolen, **enable encryption.** If someone steals your laptop, they can't access your files without the passphrase — even if they remove the drive and read it on another machine.
>
> If your computer is a desktop in a secure location and you're not concerned about physical theft, encryption is optional. It adds a slight performance overhead and requires typing a passphrase at every boot.
>
> This manual recommends enabling encryption for laptops. We'll show you exactly where to enable it during installation.

---

## Launching the Installer

From the live environment (the GNOME desktop you're currently exploring):

1. Look for an icon on the desktop or in the Activities Overview (press Super) labeled **Install to Hard Drive** or **Install Fedora**

2. Click it. The Anaconda installer launches.

Alternatively, if you don't see the icon:

1. Open a terminal (Super → type "Terminal")
2. Type:
   ```bash
   liveinst

    Press Enter

The Anaconda installer window appears.
Meet Anaconda: The Installer
A Brief History

Anaconda is Fedora's installer — and has been since the beginning. It's also used by RHEL, CentOS, AlmaLinux, and Rocky Linux. It's one of the most powerful Linux installers available, capable of handling everything from simple single-disk desktop installs to complex RAID configurations across dozens of drives.

Historically, Anaconda used a "hub-and-spoke" interface — a main screen with icons for each configuration section (Language, Storage, Network, User Creation, etc.). You could configure them in any order, but you had to visit each one before proceeding. Some users found this flexible; others found it confusing.
The New Web UI (Fedora 44)

Fedora 44 ships with Anaconda's redesigned Web UI — a modern, linear, step-by-step installer that guides you through the process sequentially. Instead of a hub of icons, you're walked through screens one at a time:

    Language selection
    Keyboard layout
    Time zone
    Storage configuration (automatic or custom)
    User account creation
    Root password (optional)
    Review and install

This linear approach is friendlier for newcomers — you're never left wondering "did I miss a section?" because you can't skip ahead without completing each step.

    💡 Old vs. New Anaconda

    If you've installed Fedora before (before version 43), you remember the old hub-and-spoke interface. The Web UI replaces it as the default in Fedora 44. The old interface is still available on the Everything netinstall ISO for users who need advanced features the Web UI doesn't expose yet.

    For this manual, we'll use the default Web UI — it's what you'll see when you launch the installer.

Step 1: Language Selection

The first screen asks you to choose your language and locale.

Welcome to Fedora 44

Language:
┌──────────────────────┐
│ English              │
│ Español              │
│ Français             │
│ Deutsch              │
│ 中文 (Chinese)        │
│ 日本語 (Japanese)     │
│ Português            │
│ ...                  │
└──────────────────────┘

[Continue]

Select your preferred language from the list. This sets the system language and the language of the installer itself.

    💡 Language Can Be Changed Later

    Don't agonize over this choice. You can change the system language after installation in Settings → Region & Language. The installer language is just for the installation process.

Click Continue.
Step 2: Keyboard Layout

Select your keyboard layout. The installer attempts to detect your layout automatically, but verify it:

    English (US) — standard QWERTY, used by most North American users
    English (UK) — slightly different (the " and @ keys are swapped vs. US layout)
    Other layouts — German QWERTZ, French AZERTY, etc.

If you use multiple layouts (e.g., English + another language), you can add both here. GNOME will let you switch between them with a keyboard shortcut after installation.

    ⚡ Test Your Layout

    There's usually a text box on this screen where you can type and verify your keys produce the expected characters. Type a few special characters like ~, |, {, }, @, # to confirm the layout matches your physical keyboard.

Click Continue.
Step 3: Time Zone

Select your time zone from a list or a map:

Region:  [Americas ▼]
City:    [New York ▼]

  or click your location on the map

You can either:

    Select your region and city from the dropdowns, or
    Click on the map to set your location

The time zone affects:

    System clock
    File timestamps
    Cron job scheduling (Chapter 24)
    Calendar appointments
    Log timestamps

    💡 UTC vs. Local Time

    Linux stores the system clock in UTC internally and converts to your local time zone for display. This is the correct approach and avoids problems with daylight saving time transitions.

    If you dual-boot with Windows, Windows may expect the hardware clock to be in local time. Fedora handles this automatically by detecting Windows installations and adjusting accordingly. If you encounter clock discrepancies after dual-booting, see Chapter 29.

Click Continue.
Step 4: Storage Configuration

This is the most important — and potentially intimidating — step. We'll walk through both the automatic and manual options.
Understanding Your Disk

Before choosing, it helps to know what the installer sees. If you have a single disk:

Disk: /dev/nvme0n1
Size: 500 GB
Current contents: (empty or existing partitions)

If you have multiple disks, each will be listed. The installer shows you what it detects so you can make an informed choice.
Option A: Automatic Installation (Recommended for New Users)

The automatic option lets Fedora handle all partitioning. You choose the disk, and Anaconda creates the optimal partition layout:

Storage Configuration

  ● Automatic
    Fedora will create partitions automatically using Btrfs

  ○ Custom
    Manually create, resize, or delete partitions

  ☑ Encrypt my data (LUKS)
    Protects your data with a passphrase

Target Disk: [ /dev/nvme0n1 (500 GB SSD) ▼ ]

[ Continue ]

Steps:

    Select Automatic
    If you want encryption (recommended for laptops), check Encrypt my data
    Select the target disk from the dropdown
    If you selected encryption, you'll be prompted to enter a passphrase

    ⚠️ Choose the Right Disk

    If you have multiple disks, carefully select the correct one. The installer will erase everything on the selected disk. Verify the disk name and size match the disk where you want to install Fedora.

    If the disk has an existing OS you want to keep (dual boot), automatic mode may not be appropriate — use Custom mode to preserve existing partitions.

Encryption Passphrase:

If you enabled encryption, you'll see a dialog:

Disk Encryption Passphrase

Passphrase:     [________________]
Confirm:        [________________]

⚠ Store this passphrase safely. You will need it every time
  you boot your computer. There is no way to recover it.

Choose a strong passphrase — a sentence you can remember but others can't guess. Write it down and store it somewhere physical and secure. If you forget this passphrase, your data is permanently inaccessible.

What Automatic Mode Creates:

On a UEFI system with Btrfs (Fedora 44's defaults), the automatic layout typically creates:
Partition	Mount Point	Size	Filesystem	Purpose
/dev/nvme0n1p1	/boot/efi	600 MB	FAT32 (EFI System Partition)	Bootloader files for UEFI
/dev/nvme0n1p2	/boot	1 GB	ext4	Kernel and initramfs (unencrypted)
/dev/nvme0n1p3	/ (Btrfs root)	Rest of disk	Btrfs (LUKS-encrypted if enabled)	Root filesystem with subvolumes

Inside the Btrfs partition, Fedora creates two subvolumes:
Subvolume	Mount Point	Purpose
root	/	System files, installed software, configuration
home	/home	User files and per-user settings

    💡 Why Btrfs Subvolumes Instead of Partitions?

    Btrfs subvolumes act like separate filesystems (you can snapshot them independently) but share the same storage pool. This means:

        You can't run out of space on /home while / has free space (they share the pool)
        You can snapshot / without affecting /home
        No need to guess partition sizes in advance

    This is a significant advantage over traditional partitioning, where you must allocate fixed sizes at install time and can't easily change them later.

Click Continue to proceed.
Option B: Custom Partitioning (For Dual Boot or Advanced Needs)

Choose Custom if you:

    Are dual-booting and need to preserve existing partitions
    Want a specific partition layout (e.g., separate /var, /tmp)
    Want to use a different filesystem (ext4 instead of Btrfs)
    Need to resize existing partitions to make room for Fedora

The custom partitioning screen shows a visual representation of your disk(s):

Disk: /dev/nvme0n1 (500 GB SSD)

┌─────────────┬──────────────────────────────────────┐
│ EFI (600MB) │ /boot (1GB) │  Free Space (498 GB)    │
└─────────────┴──────────────────────────────────────┘

[+ Create partition]  [- Delete]  [⚙ Reformat]

Creating partitions manually (typical layout):

    EFI System Partition (if not already present from a previous OS install):
        Size: 600 MB
        Filesystem: EFI System Partition (FAT32)
        Mount point: /boot/efi
        Type: EFI System Partition

    Boot partition:
        Size: 1 GB
        Filesystem: ext4
        Mount point: /boot

    Root partition (Btrfs):
        Size: Rest of disk (or whatever space you want)
        Filesystem: Btrfs
        Mount point: /
        (If encrypting: enable LUKS on this partition)

    Within the Btrfs partition, create subvolumes:
        Subvolume root → mount at /
        Subvolume home → mount at /home

    ⚠️ Custom Partitioning and the Web UI

    The new Anaconda Web UI is still maturing. Some advanced partitioning options available in the old interface (like XFS selection, LVM-on-LUKS, and complex subvolume management) may be limited or buggy in the Web UI. If you need advanced partitioning and encounter issues:

        Use the Everything netinstall ISO, which still includes the classic Anaconda interface with full partitioning capabilities
        Or configure the basics in the Web UI and refine after installation with tools like btrfs subvolume commands (Chapter 14, Chapter 26)

Dual-Boot Partitioning:

If you're dual-booting with Windows or another OS:

    Shrink the existing OS partition in Windows Disk Management (or in GNOME Disks) before starting the installer
    In Anaconda's custom mode, select the freed space
    Create Fedora's partitions within that free space — don't touch existing partitions

Typical dual-boot layout:
Partition	OS	Filesystem	Size	Mount Point
/dev/nvme0n1p1	Windows (existing)	NTFS	250 GB	(not mounted in Fedora)
/dev/nvme0n1p2	Windows (existing)	NTFS	200 GB	(not mounted in Fedora)
/dev/nvme0n1p3	Shared (optional)	exFAT	50 GB	/mnt/shared
/dev/nvme0n1p4	Fedora EFI	FAT32	600 MB	/boot/efi
/dev/nvme0n1p5	Fedora boot	ext4	1 GB	/boot
/dev/nvme0n1p6	Fedora root	Btrfs	Remaining	/

    💡 Sharing the EFI Partition

    If Windows is already installed with UEFI, you don't need a separate EFI partition for Fedora. Fedora can share Windows' existing EFI partition — select it in the installer and set its mount point to /boot/efi without reformatting it.

Click Done when partitioning is complete.
Step 5: User Account Creation

The installer asks you to create your user account:

Create User

Full name:    [ John Smith           ]
Username:     [ john                 ]
              (lowercase, no spaces)

Password:     [ ********              ]
Confirm:      [ ********              ]

☑ Make this user administrator
☑ Require a password to log in

Guidelines:
Field	Advice
Full name	Your real name or any display name. Spaces are fine here.
Username	Lowercase letters, numbers, hyphens, underscores. No spaces. This becomes your home directory path (/home/john).
Password	Use a strong passphrase. At least 12 characters. Mix letters, numbers, and symbols.
Make administrator	Check this. This adds you to the wheel group, enabling sudo. Without it, you can't install software or change system settings.
Require password	Leave checked. Disabling it means auto-login without a password — convenient but insecure.

    ⚠️ Remember Your Password

    Unlike Windows, where you can reset a forgotten password with security questions or a Microsoft account, a forgotten Fedora user password (especially with disk encryption) can lock you out permanently. Write it down. Store it securely. The system can't email you a reset link.

Root Password (Optional)

Fedora 44's installer may ask whether you want to set a separate root password.

By default, Fedora locks the root account — meaning root has no password and can't be used to log in directly. Instead, the first user (if in the wheel group) uses sudo for administrative tasks.

Root Account

  ● Lock root account (recommended)
    Administrative access via sudo only

  ○ Set root password
    [_______________]
    [_______________]

Recommendation: Leave root locked. Using sudo is more secure because:

    Every sudo command is logged (you can see who did what and when)
    sudo privileges can be revoked per-user
    No one can log in directly as root (a common attack target)

If you set a root password, someone would need to guess only one password to get full system control. With root locked and sudo in use, they'd need your username and your password.

Click Continue.
Step 6: Installation Review

Before the installer writes anything to your disk, it presents a summary of what it's about to do:

Installation Summary

Language:     English (United States)
Keyboard:     English (US)
Time Zone:    America/New_York (EST)

Storage:
  Disk: /dev/nvme0n1 (500 GB)
  Encryption: LUKS2 enabled
  Filesystem: Btrfs
  
  Partitions to be created:
    /boot/efi  (600 MB, FAT32)
    /boot      (1 GB, ext4)
    /          (Rest, Btrfs, encrypted)
      Subvolume: root → /
      Subvolume: home → /home

User:
  Username: john
  Administrator: Yes
  Root account: Locked

⚠ All data on /dev/nvme0n1 will be erased!

[ Cancel ]  [ Start Installation ]

    ⚠️ This Is the Point of No Return

    Once you click Start Installation, the installer begins writing to your disk. If you chose to erase the entire disk, all data on it is destroyed within seconds. Make absolutely sure:

        You selected the correct disk
        Your backup is complete and verified
        You're ready to commit

If everything looks correct, click Start Installation.
Step 7: Installation Progress

The installer now:

    Partitions the disk — creates the partition table and filesystems
    Enables encryption — sets up the LUKS container if you chose encryption
    Downloads packages — copies Fedora packages from the USB to your disk (if you have internet, it may also download updates)
    Installs the base system — extracts and configures the kernel, systemd, core utilities
    Installs the desktop — GNOME 50, Firefox, LibreOffice, and default applications
    Configures the bootloader — installs GRUB and configures the EFI boot entry
    Creates your user account — sets up /home/john, configures wheel group membership

Installing Fedora 44 Workstation

[████████████████████████░░░░░░░] 72%

Installing: gnome-shell-50.0-1.fc44.x86_64
Packages: 1,247 / 1,723
Time remaining: ~4 minutes

This takes 10–25 minutes depending on your disk speed and CPU.
If Installation Fails

Rare but possible. Common causes:
Failure	Likely Cause	Fix
Disk write error	Faulty drive or bad sectors	Run smartctl on the disk (Chapter 30)
Out of space	Disk too small or partition sizes wrong	Use a larger disk or reduce partition sizes
Package download failure	No internet or unstable connection	The installer uses the USB packages; internet is optional
Kernel panic / freeze	Hardware incompatibility	Try "basic graphics mode" (Chapter 5); check RAM with memtest

If installation fails midway, don't panic — your disk may be in a partially partitioned state. Restart the installer and try again. Anaconda will re-partition from scratch.
Step 8: Installation Complete — First Boot

When the installation finishes, you'll see:

✓ Installation Complete

Fedora 44 Workstation has been successfully installed.

[ Quit ]  [ Reboot System ]

What to do:

    Click Reboot System
    As the computer restarts, remove the USB drive when the screen goes dark (between the BIOS logo and the GRUB menu)
    The computer now boots from your hard drive into Fedora

    ⚠️ Remove the USB at the Right Time

    If you leave the USB inserted, the computer may boot from it again instead of your installed Fedora. Remove it during the reboot — after the BIOS logo disappears but before the GRUB menu appears. If you miss the window and boot back into the live USB, just reboot again and remove the USB sooner.

The LUKS Passphrase Prompt

If you enabled encryption, the first thing you see after the BIOS/UEFI logo is:

Please enter passphrase for disk Fedora (nvme0n1p3):

Type your encryption passphrase and press Enter. Nothing will appear on screen as you type (no asterisks) — this is a security measure. If you make a typo, press Enter and you'll be prompted again.

Once unlocked, the system continues booting normally.

    💡 You'll See This Every Boot

    This passphrase prompt appears every time you turn on your computer. There's no way around it — that's the point of encryption. If someone steals your laptop, they face this prompt and can't proceed without the passphrase.

    Advanced users can set up TPM-backed auto-unlock — the system reads the encryption key from a TPM chip, eliminating the manual passphrase. This binds decryption to your specific hardware. It's more advanced and covered in Fedora's security documentation online.

The GRUB Boot Menu

After the LUKS prompt (or immediately after BIOS if no encryption), you'll see the GRUB boot menu:

GNU GRUB version 2.06

┌────────────────────────────────────────────┐
│ Fedora 44 (Resolute)  ▶                     │
│ Fedora 44 (Resolute) (recovery mode)        │
│                                             │
│                                             │
└────────────────────────────────────────────┘
  Use ↑ and ↓ to select, Enter to boot

If you only have Fedora installed, this menu appears for a few seconds and then auto-boots. If you dual-boot, you'll see entries for both Fedora and your other OS.

    💡 GRUB Timeout

    The GRUB menu auto-boots after 5 seconds. You can press any arrow key to stop the countdown and take your time. Press Enter to boot immediately.

The Login Screen (GDM)

After GRUB loads the kernel, Fedora boots into the GNOME Display Manager (GDM) — the login screen:

┌──────────────────────────────────┐
│                                  │
│         John Smith               │
│      [ Enter Password ]           │
│                                  │
│   🔔  ⚡  🌐                      │
└──────────────────────────────────┘

    Click your username (or it may be pre-selected)
    Type your user password
    Press Enter

The GNOME 50 desktop loads. Welcome to Fedora 44.
Step 9: GNOME Initial Setup

On first login, GNOME may present an Initial Setup wizard — a series of screens guiding you through basic configuration:
Language

Confirm your system language (already set during installation).
Keyboard Layouts

Add additional keyboard layouts if needed. You can switch between layouts with Super+Space.
Privacy Settings

Privacy

☑ Location Services
  Allow applications to determine your location

☐ Problem Reporting
  Automatically send crash reports to help improve software

☐ Connectivity Checking
  Periodically check if you have internet connectivity

Setting	Recommendation
Location Services	Off. Rarely needed on desktop. Enable only if a specific app requires it.
Problem Reporting	On. Crash reports help developers fix bugs. No personal data is included.
Connectivity Checking	Off. Fedora periodically pings a server to check internet status. Privacy-paranoid users disable this.
Online Accounts (Optional)

GNOME offers integration with online accounts (Google, Microsoft, Nextcloud):

Connect Your Online Accounts

  Google    Microsoft    Nextcloud    Fedora
    
  Connect an account to access files, mail,
  contacts, and calendars from your desktop.

You can skip this entirely — it's optional. If you connect a Google account, for example, GNOME Files shows your Google Drive files, GNOME Calendar syncs with Google Calendar, and Evolution/Geary can access Gmail.

    💡 Fedora's Online Accounts vs. Google Drive Integration

    Note: GNOME 50 removed the Google Drive integration from Files (Nautilus) that existed in previous versions. If you relied on Google Drive access through Files, you'll need to use a different method — such as the google-drive-ocamlfuse tool or rclone. See Chapter 21 for cloud storage options.

Software Repositories (Third-Party)

Fedora's initial setup may offer to enable third-party software repositories:

Third-Party Software Repositories

  ☐ Enable RPM Fusion (free)
    Provides additional multimedia codecs and drivers

  ☐ Enable RPM Fusion (nonfree)
    Provides NVIDIA drivers and other proprietary software

  ☐ Enable Flathub
    Provides Flatpak applications from the Flathub repository

Repository	Recommendation
RPM Fusion (free)	Enable. Provides essential codecs for media playback.
RPM Fusion (nonfree)	Enable if you have NVIDIA. Skip if you have Intel/AMD graphics.
Flathub	Enable. This is where most of your GUI applications will come from.

    💡 You Can Enable These Later Too

    If you're unsure, skip this screen. Chapter 7 (Post-Install Checklist) and Chapter 11 (Multimedia and Codecs) walk you through enabling RPM Fusion and Flathub from the terminal with precise control. The initial setup screen is a convenience — the same repos can be added anytime.

You're In

After the initial setup screens, you're at the GNOME 50 desktop. The top bar spans the top of your screen. A clean wallpaper fills the background. Press the Super key to see the Activities Overview.

Take a moment. You've installed Fedora 44. Your system is running, secure, and ready to configure.
Understanding What Was Installed

Let's take stock of what's on your system now.
The Default Partition Layout

If you chose automatic partitioning with encryption, your disk looks like this:
Device	Mount Point	Size	Filesystem	Encrypted	Purpose
/dev/nvme0n1p1	/boot/efi	600 MB	FAT32	No	UEFI bootloader files
/dev/nvme0n1p2	/boot	1 GB	ext4	No	Kernel images, initramfs
/dev/nvme0n1p3	/	Rest	Btrfs	Yes (LUKS2)	Root filesystem
(subvolume root)	/	—	Btrfs	—	System files, software
(subvolume home)	/home	—	Btrfs	—	Your files and settings
Default Software

Fedora 44 Workstation installs a focused set of applications by default:
Category	Application	Notes
Web browser	Firefox	The default and only pre-installed browser
Office suite	LibreOffice	Writer, Calc, Impress (no Base or Draw by default)
File manager	GNOME Files (Nautilus)	
Terminal	GNOME Terminal	(Not Ptyxis — that's Ubuntu 26.04)
Text editor	GNOME Text Editor (gnome-text-editor)	Simple, clean editor
Document viewer	Document Viewer (Evince)	PDFs and other documents
Image viewer	Loupe (or Eye of GNOME)	
Video player	GNOME Videos (Totem)	Limited codec support without RPM Fusion
Music player	GNOME Music	
Calendar	GNOME Calendar	
Contacts	GNOME Contacts	
Maps	GNOME Maps	
Weather	GNOME Weather	
Calculator	GNOME Calculator	
Screenshots	Screenshot tool	Built into GNOME 50
Settings	GNOME Settings	System configuration
Software	GNOME Software	App store for RPM and Flatpak
Disks	GNOME Disks	Disk management
System Monitor	GNOME System Monitor	Process and resource viewer
What's Not Installed

Fedora's minimal, free-software-only default means some things are conspicuously absent:

    No media codecs — can't play MP4, MP3, or most video/audio formats (add via RPM Fusion — Chapter 11)
    No NVIDIA drivers — only the open-source Nouveau driver (add via RPM Fusion — Chapter 22)
    No Steam, Spotify, Discord — install via Flathub from GNOME Software
    No Microsoft fonts — install mscorefonts2 from RPM Fusion if needed
    No Flash — deprecated and unnecessary; Fedora doesn't ship it

Security Defaults

Fedora 44 comes with these security features active:
Feature	Status	What It Does
SELinux	Enforcing (targeted)	Constrains what programs can do
firewalld	Active	Blocks incoming network connections
Disk encryption	Active (if you enabled it)	Protects data at rest
No open ports	All ports closed	No network services exposed
Package signing	Enabled	All packages cryptographically verified
Kernel hardening	Enabled	Stack protection, KASLR, etc.
Verifying Your Installation

After first boot, run these commands in a terminal to confirm everything is set up correctly:
Check the Fedora Version

cat /etc/fedora-release

Fedora release 44 (Forty Four)

Check the Kernel

uname -r

6.19.0-50.fc44.x86_64

Check the Desktop

echo $XDG_CURRENT_DESKTOP

GNOME

Check Filesystems

findmnt --real

TARGET  SOURCE              FSTYPE  OPTIONS
/       /dev/mapper/luks-... btrfs  rw,relatime,compress=zstd:1
├─/home /dev/mapper/luks-.../home btrfs rw,relatime,compress=zstd:1
├─/boot /dev/nvme0n1p2    ext4    rw,relatime
└─/boot/efi /dev/nvme0n1p1 vfat   rw,relatime,fmask=0077,dmask=0077

Check Encryption

lsblk -f

Look for crypto_LUKS in the FSTYPE column — that confirms encryption is active.
Check SELinux

getenforce

Enforcing

Check the Firewall

sudo firewall-cmd --state

running

Check Your User and Groups

id

uid=1000(john) gid=1000(john) groups=1000(john),10(wheel) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023

The wheel group membership confirms you have sudo access.
Check Disk Space

df -h

Ensure your root partition has adequate free space (at least 15 GB free after installation is healthy).
If Something Went Wrong
The System Won't Boot

Symptom: Black screen, GRUB error, or infinite boot loop after installation.

Fix:

    Boot from your Fedora USB drive again
    In the live environment, open GNOME Disks
    Verify your partitions exist and are correctly formatted
    If partitions look correct, the issue is likely the bootloader. Reinstall GRUB:

    # Mount the installed system
    sudo mount /dev/nvme0n1p3 /mnt
    sudo mount /dev/nvme0n1p2 /mnt/boot
    sudo mount /dev/nvme0n1p1 /mnt/boot/efi

    # If encrypted, unlock first:
    sudo cryptsetup luksOpen /dev/nvme0n1p3 fedora-root
    sudo mount /dev/mapper/fedora-root /mnt

    # Chroot and reinstall GRUB
    sudo dnf install --installroot=/mnt -y grub2-efi-x64 shim-x64
    sudo grub2-install --target=x86_64-efi --efi-directory=/mnt/boot/efi --boot-directory=/mnt/boot

    Reboot

Forgot Your Encryption Passphrase

Symptom: You can't boot because you don't remember the LUKS passphrase.

Reality: If you truly forgot it and have no recovery key, your data is gone. LUKS encryption is mathematically unbreakable without the key. This is why we stressed writing it down.

Partial recovery: If you remember part of the passphrase, tools like brute-force scripts exist, but they're impractical for strong passphrases.

Lesson: Always store encryption passphrases in a secure, physical location.
Forgot Your User Password

Symptom: System boots, you see the login screen, but you can't remember your password.

Fix: You can reset it from a root shell:

    Reboot
    At the GRUB menu, press e to edit the boot entry
    Find the line starting with linux and append rw init=/bin/bash at the end
    Press Ctrl+X to boot
    You'll get a root shell (no password needed)
    Reset your user's password:

    passwd john

    If SELinux prevents this, relabel and reboot:

    touch /.autorelabel
    exec /sbin/init

    After reboot and relabel (takes a few minutes), log in with the new password

    ⚠️ This Is Why Root Is Locked

    Notice that the password reset procedure above doesn't require knowing any password — it boots directly into a root shell. This is possible because anyone with physical access to a computer can modify GRUB boot parameters.

    Full disk encryption prevents this. If your disk is encrypted, the attacker can't even mount the filesystem to reset the password without the LUKS passphrase. This is another reason to enable encryption.

What's Next

You have a working Fedora 44 installation. Congratulations — the hardest part is over.

But Fedora isn't quite "ready to use" yet. There are several post-installation tasks that most users perform: updating the system, enabling RPM Fusion for media codecs, installing Flatpak applications, configuring the desktop, and installing drivers.

In Chapter 7, we'll walk through a post-install checklist — the essential first tasks to transform your fresh Fedora installation into a complete, daily-driver system.
Try It Yourself

    Install Fedora. Follow the steps in this chapter to install Fedora 44 Workstation on your computer. Choose automatic partitioning if you're new, and enable encryption if this is a laptop.

    Verify your installation. After first boot, open a terminal and run the verification commands from the "Verifying Your Installation" section. Confirm the version, kernel, filesystem, SELinux, and firewall are all as expected.

    Explore the desktop. Press Super to open Activities. Click through the app grid. Open Settings and browse each panel. Open Firefox and browse to a website. Open Files and navigate your home directory. Get comfortable — this is your desktop now.

    Check your disk layout. Open GNOME Disks and visually examine your partitions. Compare what you see to the partition table in this chapter. Understanding your disk layout helps with future troubleshooting.

    Take a screenshot. Press the Print Screen key. GNOME 50's screenshot tool appears, letting you capture the full screen, a window, or a region. Save it to your Pictures folder. This is your "day one" screenshot — a memento of your first Fedora installation.

    ✅ Chapter 6 Complete You understand how to launch the Anaconda installer from the live environment, the new Web UI's linear step-by-step flow, language/keyboard/timezone selection, automatic vs. custom partitioning, Btrfs subvolumes and why Fedora uses them, LUKS2 full disk encryption, the user account creation process with the wheel group, why the root account is locked by default, the GNOME initial setup wizard, what software is and isn't installed by default, Fedora's security defaults, how to verify your installation with terminal commands, and how to recover from common post-install problems including password resets.

