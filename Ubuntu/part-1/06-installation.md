# Chapter 6: Installation

## The Moment of Truth

If you've followed along through Chapters 1–5, you now have a verified Ubuntu 26.04 LTS ISO written to a bootable USB drive, your data is backed up, and you know your boot menu key. In this chapter, we're going to install Ubuntu on your computer — step by step, screen by screen.

Take a breath. This is easier than you think. The Ubuntu 26.04 installer uses a modern **Flutter-based interface** (new as of 25.10, polished for the LTS release) that guides you through the process with clear options and explanations. There are no trick questions or hidden traps. If you can install an app on your phone, you can install Ubuntu.

## Before You Begin: A Quick Recap

Make sure you have these things ready:

- ✅ Bootable USB drive with Ubuntu 26.04 LTS (from Chapter 5)
- ✅ Important data backed up to an external drive or cloud
- ✅ Windows Fast Startup disabled (if coming from Windows — see Chapter 5)
- ✅ Your boot menu key written down (F12, F9, Esc, etc. — see Chapter 5)
- ✅ Your computer plugged into power (don't install on battery)
- ✅ An internet connection available (optional but recommended — the installer can download updates and drivers during installation)

> ⚠️ **About BitLocker (Windows)**
>
> If your Windows installation uses **BitLocker** encryption, you **must** disable it before installing Ubuntu. BitLocker locks the entire disk, and the Ubuntu installer cannot resize or modify BitLocker-encrypted partitions.
>
> To check: Open Windows → Search "BitLocker" → Look at your drive status. If it says "BitLocker on," click "Turn off BitLocker" and wait for decryption to complete (this can take a while depending on drive size).
>
> If you don't use BitLocker, skip this step.

## Step 1: Boot from the USB Drive

1. Insert your **bootable USB drive** into the computer
2. **Restart** (or power on) the computer
3. Immediately start **tapping your boot menu key** (F12, F9, Esc, etc. — whatever applies to your manufacturer)
4. A **boot menu** should appear — a list of devices you can boot from
5. Select your **USB drive** from the list (it may show up as the drive name, "USB," "Removable Device," or "UEFI: [your USB brand]")
6. Press **Enter**

> 💡 **What if the boot menu doesn't appear?**
>
> If you keep tapping the key but Windows (or your current OS) loads instead:
>
> 1. Restart and try the **BIOS/UEFI key** instead (F2, Del, etc. — see Chapter 5's table)
> 2. Once in BIOS/UEFI settings, find the **Boot Order** or **Boot Priority** section
> 3. Move **USB drive** to the top of the list
> 4. Save and exit (usually F10)
> 5. The computer should now boot from the USB
>
> If that still doesn't work, try a different USB port (preferably USB 2.0 if available — some systems have trouble booting from USB 3.0 ports), or recreate your USB with a different flashing tool.

## Step 2: The Boot Screen

Once your computer boots from the USB, you'll see the **Ubuntu boot screen** — a simple screen with the Ubuntu logo and a loading animation. This is the **live environment** loading — a complete Ubuntu system running entirely from your USB drive, without touching your hard drive.

After a few moments (longer on slower systems), you'll be presented with a choice:

- **Try Ubuntu** — Launches the full Ubuntu desktop from the USB drive without installing anything. You can browse around, test if your hardware works, and install later.
- **Install Ubuntu** — Starts the installation process immediately.

> 💡 **Try First If You're Unsure**
>
> If this is your first time with Ubuntu, consider clicking **"Try Ubuntu"** first. Spend 10–15 minutes exploring the desktop:
>
> - Does your Wi-Fi work?
> - Does your screen resolution look correct?
> - Does your trackpad/mouse work properly?
> - Can you hear audio?
>
> If everything works, you can launch the installer from the desktop by double-clicking the **"Install Ubuntu"** icon. If something doesn't work, you may want to research the issue before committing to an installation.

For this walkthrough, we'll assume you've chosen **"Install Ubuntu"** (either from the initial menu or from the live desktop).

## Step 3: Language Selection

The first screen of the installer asks you to select your **language**. This sets the language for the installer interface, your installed system, and your keyboard layout suggestions.

Scroll through the list, find your language, and click **Continue**.

## Step 4: Accessibility Options

Next, you'll see an **accessibility** screen. Ubuntu 26.04 prominently features accessibility options during installation, which is a positive change. Here you can enable:

- **Screen reader** (Orca) — for visually impaired users
- **High contrast** — for better visibility
- **Magnifier** — zoom into parts of the screen
- **On-screen keyboard** — for users who can't use a physical keyboard
- **Sticky keys** — press keyboard shortcuts one key at a time instead of simultaneously

If you don't need any of these, just click **Continue**. If you do, enable what you need — these settings carry over to your installed system.

## Step 5: Keyboard Layout

Select your **keyboard layout**. If your keyboard works correctly when you type in the text box provided, you're good. The installer usually auto-detects your layout based on your language selection, but you can refine it here.

If you use a non-standard layout (DVORAK, a different national layout, etc.), select it from the lists. Click **Continue** when done.

## Step 6: Connect to the Internet

The installer will ask you to **connect to a network**. You can:

- Connect to **Wi-Fi** — select your network from the list and enter the password
- Use a **wired (Ethernet) connection** — plug in the cable, and it connects automatically
- **Skip** — you can install without internet, but this is not recommended

> 💡 **Why connect during installation?**
>
> Connecting to the internet during installation allows Ubuntu to:
>
> - Download and install the **latest updates** — so your system is current from the first boot
> - Download **proprietary drivers** for your hardware (especially graphics and Wi-Fi)
> - Set your **timezone** automatically based on your location
>
> If you skip this, you can do all of these things later, but it's much more convenient to let the installer handle them.

## Step 7: Installation Type — "Interactive"

Ubuntu 26.04's installer presents an **"Interactive"** installation type. This is the guided path designed for most users. It walks you through choosing what software to install, handling drivers, and configuring your disk.

Click **Continue**.

## Step 8: Default Application Selection

The installer asks what software you'd like pre-installed. You'll typically see options like:

- **Normal installation** — Includes a full set of applications: web browser, office suite, media players, games, and utilities. Best for most users.
- **Minimal installation** — Includes only the essentials: web browser and basic utilities. Good if you want a lean system and prefer to install only what you need.

For your first installation, choose **Normal installation**. You can always remove apps you don't want later.

## Step 9: Proprietary Drivers and Media Codecs

You'll see checkboxes for:

- **Install third-party software for graphics and Wi-Fi hardware** — This installs proprietary drivers for NVIDIA/AMD graphics cards, some Wi-Fi chips, and other hardware that requires non-open-source drivers to function optimally.
- **Download updates while installing Ubuntu** — Installs the latest security updates and package updates during the installation process.
- **Install multimedia codecs** — Adds support for common audio and video formats (MP3, H.264, etc.) that aren't included by default due to licensing restrictions.

> 💡 **Should you check these boxes?**
>
> **Yes — check all of them.**
>
> - **Third-party drivers**: If you have an NVIDIA graphics card, this is the easiest way to get your drivers working. Without them, you may get poor performance or low resolution.
> - **Updates**: Gets your system current immediately — saves you a long update session after your first boot.
> - **Codecs**: Without these, many videos and music files won't play. We cover codecs in detail in Chapter 11, but enabling them here is the simplest approach.
>
> These options require an internet connection (which is why we connected in Step 6).

## Step 10: Disk Setup

This is the most important screen of the entire installation. Here you decide how Ubuntu will use your disk(s). The options you see depend on your current setup:

### Option A: Erase Disk and Install Ubuntu

This is the simplest option. It wipes everything on the selected disk and installs Ubuntu as the only operating system.

- **Choose this if**: You're dedicating the entire computer to Ubuntu, or the disk is empty/new
- **What happens**: All existing data and partitions on the disk are destroyed, and Ubuntu is installed fresh

If you have multiple disks, make sure the correct one is selected. The installer shows the disk name and size — double-check it matches the disk you intend to use.

### Option B: Install Alongside Windows (Dual Boot)

If the installer detects an existing Windows installation, it may offer to **install alongside** it. This resizes your Windows partition and installs Ubuntu in the freed space, giving you a boot menu to choose between Windows and Ubuntu each time you start your computer.

- **Choose this if**: You want to keep Windows and also use Ubuntu
- **What happens**: Windows is shrunk (non-destructively), Ubuntu is installed in the new space, and a boot manager (GRUB) is set up to let you choose at startup
- **Requirements**: Enough free space on your Windows partition (at least 25–30 GB), Windows properly shut down (not hibernated), BitLocker disabled

> ⚠️ **Dual Boot Caveats**
>
> Dual booting is generally safe, but there are a few things to know:
>
> - Always back up your Windows data before resizing partitions. While rare, things can go wrong.
> - Make sure Windows is fully shut down, not in Fast Startup hibernation. You disabled Fast Startup in Chapter 5, right?
> - After installation, the **GRUB boot menu** appears each time you start your computer, letting you choose Ubuntu or Windows. You can configure the default OS and timeout later.
> - Windows Updates can occasionally overwrite the GRUB boot menu. If this happens, it's fixable — we'll cover this in Chapter 30 (Troubleshooting).

### Option C: Something Else (Manual Partitioning)

This option gives you full control over partitioning — you create, resize, and assign mount points manually.

- **Choose this if**: You have specific partitioning needs, want to set up separate partitions for `/home` and `/`, want to use LVM, or have a complex multi-disk setup
- **What happens**: You're presented with a partition editor where you can create, delete, and modify partitions

> 💡 **For Beginners: Avoid "Something Else" on Your First Install**
>
> Manual partitioning requires understanding of mount points, file systems, and partition schemes. It's a valuable skill, and we'll cover it in later chapters. For your first installation, use **"Erase disk"** or **"Install alongside."**
>
> If you're curious, the typical manual partitioning layout looks like:
>
> | Partition | Size | Type | Mount Point | File System |
> |---|---|---|---|---|
> | EFI System Partition (ESP) | ~300 MB | EFI System Partition | /boot/efi | FAT32 |
> | Root | 30+ GB | Primary | / | ext4 (or btrfs) |
> | Swap | Variable (or swap file) | — | swap | swap |
> | Home | Remaining space | Primary | /home | ext4 (or btrfs) |

### Encryption Options

Regardless of which disk option you choose (when erasing the disk), the installer offers **encryption**:

- **Encrypt the new Ubuntu installation** — Uses **LUKS** (Linux Unified Key Setup) to encrypt your entire disk. You'll set a **passphrase** that must be entered each time you boot, before the system starts. Without the passphrase, the data on the disk is unreadable — even if someone removes the drive and plugs it into another computer.
- **Use TPM to secure the encryption key** — New in 26.04: If your computer has a **TPM 2.0** chip (most computers from 2018 onward do) and **Secure Boot** is enabled, you can store the encryption key in the TPM hardware. This means you **don't need to type a passphrase at boot** — the TPM unlocks the disk automatically. This is called **TPM-backed Full Disk Encryption (FDE)**.

> ⚠️ **Important Notes on Encryption**
>
> - **If you choose LUKS encryption with a passphrase, DO NOT lose the passphrase.** There is no recovery mechanism. If you forget it, your data is permanently inaccessible. Write it down and store it somewhere safe.
> - **TPM-backed FDE** is convenient but has a consideration: if you change your hardware (motherboard, CPU) or disable Secure Boot, the TPM key becomes invalid and you'll need your recovery passphrase. Always set a passphrase as a fallback, even when using TPM.
> - **Encryption has a slight performance cost** on older hardware, but on modern CPUs (which have hardware AES acceleration), the impact is negligible.
> - **Dual-boot with encryption**: The installer only offers automatic encryption when you choose "Erase disk." If you're dual-booting and want encryption, you'll need to use "Something else" and set up LUKS manually — which is more advanced.

> 💡 **Should You Encrypt?**
>
> For most desktop users on a personal machine, **yes** — encryption is worth it. If your laptop is stolen, the thief can't access your personal files without your passphrase. The minor inconvenience of typing a password at boot is a small price for that peace of mind.
>
> If you choose **TPM-backed FDE**, you get the security without the boot-time password — the best of both worlds, assuming your hardware supports it.

## Step 11: Create Your User Account

Next, you'll set up your user account. The installer asks for:

- **Your name** — Your full name (or any name you like). This is displayed on the login screen and in some applications. It's purely cosmetic.
- **Your computer's name** — The hostname for this machine on your network. This is how your computer identifies itself to other devices. Something like `ubuntu-desktop` or `my-laptop` is fine. Keep it lowercase, no spaces.
- **Pick a username** — This is your login name. It's also the name of your home folder (`/home/yourusername`). Choose something simple and lowercase — no spaces, no special characters. For example: `john`, `sarah`, `alex`.
- **Choose a password** — This is your login password AND your `sudo` password (the one you'll use to install software and make system changes). Make it strong — you'll be typing it often. Use a mix of upper/lowercase letters, numbers, and symbols.
- **Confirm your password** — Type it again to make sure there are no typos.
- **Log in automatically** (checkbox) — If checked, you won't be prompted for a password at startup; the system logs you in directly. Convenient, but less secure. If you've enabled disk encryption, you'll still need to type your encryption passphrase at boot regardless.
- **Require my password to log in** (checkbox, opposite of above) — Recommended for security.

> ⚠️ **Choose Your Username Carefully**
>
> Your username becomes permanent in several ways:
>
> - It's the name of your home directory (`/home/username`)
> - It appears in file paths throughout the system
> - Changing it later is possible but inconvenient (it requires renaming your home directory, updating permissions, and modifying system files)
>
> Pick something you'll be happy with long-term. Short, lowercase, no spaces.

## Step 12: Timezone

If you connected to the internet, the installer auto-detects your **timezone** based on your network location. You'll see a map with your location marked.

Verify the timezone is correct. If it's wrong, click your location on the map or type your city name. Click **Continue**.

## Step 13: Review and Confirm

The installer presents a **summary screen** showing everything you've chosen:

- Language
- Keyboard layout
- Installation type (Normal/Minimal)
- Disk layout
- Encryption status
- Username and computer name
- Timezone

Read through it carefully. This is your last chance to go back and change anything.

> ⚠️ **Final Warning**
>
> If you chose "Erase disk," clicking **Install** will begin writing to your disk. Any data on the target disk will be **permanently deleted**. Make absolutely sure you've selected the right disk and that your backup is complete.
>
> For dual-boot installations, clicking Install will begin resizing your Windows partition and writing Ubuntu to the freed space. Make sure Windows was properly shut down.

When you're confident everything is correct, click **Install**.

## Step 14: Installation Progress

Now the installer works. You'll see a progress bar and a slide show highlighting Ubuntu features. The installation process typically takes **15–30 minutes** depending on:

- Your USB drive speed (USB 3.0 is much faster than 2.0)
- Your disk speed (SSD is much faster than HDD)
- Whether updates are being downloaded
- Whether proprietary drivers are being installed
- Your internet speed (if downloading updates/drivers)

During this time, the installer is:

1. **Partitioning your disk** — creating the filesystem structures
2. **Copying the system files** — writing the Ubuntu image to your disk
3. **Installing the bootloader** (GRUB) — setting up the boot menu
4. **Downloading and applying updates** (if you selected this option)
5. **Installing proprietary drivers** (if you selected this option)
6. **Configuring your user account** — setting up your home directory, password, and permissions
7. **Setting the timezone and locale** — configuring your region settings

> 💡 **Don't Panic If It Seems Stuck**
>
> Sometimes the progress bar stays at a particular percentage for a while — this is normal, especially during the "downloading updates" phase (which depends on your internet speed) or the "installing drivers" phase. As long as the hard drive activity light is blinking or you can see the USB drive is being read, the installation is proceeding. Be patient.

## Step 15: Installation Complete — Reboot

When the installation finishes, you'll see a message: **"Installation is complete. You can continue testing Ubuntu or restart now."**

1. Remove your **USB drive** from the computer
2. Click **"Restart Now"**
3. Your computer will reboot

> ⚠️ **Remove the USB at the right time**
>
> The installer will prompt you to remove the USB drive before restarting. Do it when prompted — not before. If you leave the USB in, the computer may boot from it again instead of your newly installed Ubuntu system. If this happens, just restart again after removing the USB.

## Step 16: First Boot

Your computer restarts. Instead of the USB boot screen, you should now see the **GRUB boot menu** briefly (if dual-booting) or the **Ubuntu login screen** directly (if Ubuntu is the only OS).

### If Dual-Booting

The GRUB menu appears, listing:

- **Ubuntu** — boots into Ubuntu
- **Windows Boot Manager** (or similar) — boots into Windows

Use the arrow keys to select and press Enter. The menu waits a few seconds and auto-selects Ubuntu by default. We'll cover customizing GRUB in Chapter 26.

### The Login Screen

You'll see the **login screen** (called the **display manager**, specifically **GDM** — GNOME Display Manager). It shows:

- The time and date
- Your username (click it)
- A password field

Click your username, type the password you set during installation, and press Enter.

### If You Enabled Disk Encryption

If you chose LUKS encryption, you'll see a prompt **before** the login screen asking for your **disk passphrase**. Type it and press Enter to unlock the disk. Then you'll proceed to the normal login screen.

If you chose **TPM-backed FDE**, the disk unlocks automatically — you'll go straight to the login screen.

### First Boot Welcome Screen

Ubuntu 26.04 LTS displays a **welcome screen** on first login. This typically includes:

- A brief introduction to Ubuntu
- Links to documentation and help resources
- Option to enable **Ubuntu Pro** (free for personal use — extends security updates)
- Tips for getting started
- Privacy settings review (location services, online accounts, etc.)

Take a moment to read through it. We'll cover Ubuntu Pro setup and post-install configuration in detail in **Chapter 7 (Post-Install Checklist)**.

## Step 17: You're In!

Welcome to Ubuntu 26.04 LTS (Resolute Raccoon)! 🎉

You should now be looking at the GNOME 50 desktop. Take a moment to look around:

- The **Ubuntu Dock** is on the left side
- The **Activities** button is in the top-left
- The **clock and quick settings** are in the top-right
- Your **home folder** is accessible through the Files app

Press the **Super key** (Windows key) to open the Activities Overview and search for apps. Try opening **Firefox** and visiting a website to confirm your internet connection works.

## Common Installation Problems and Solutions

### The Computer Won't Boot from USB

- Try a different USB port (USB 2.0 if available)
- Recreate the USB with a different tool (try Rufus if Etcher failed, or vice versa)
- Check BIOS/UEFI boot order — make sure USB is listed above your hard drive
- Disable Secure Boot temporarily in BIOS/UEFI (you can re-enable it after installation)
- Try a different USB drive — some drives have compatibility issues

### Black Screen After Booting from USB

- This is often a **graphics driver issue**, especially with NVIDIA cards
- At the boot menu, press **e** to edit the boot parameters
- Add `nomodeset` after the line starting with `linux`
- Press Ctrl+X to boot with this parameter
- This disables advanced graphics — good enough to complete installation
- We'll cover proper driver installation in Chapter 22

### Installation Freezes or Crashes

- Check your ISO checksum — a corrupt ISO is the most common cause
- Try recreating the USB drive — it may have been written incorrectly
- Test your RAM — use **Memtest86** (often available from the boot menu) to check for memory errors
- Try installing without downloading updates (uncheck the box in Step 9) — if this works, the issue was a network problem during installation

### Wi-Fi Doesn't Work During Installation

- Use a **wired (Ethernet) connection** instead
- Some Wi-Fi cards require proprietary drivers that install after the system is set up
- Complete the installation without Wi-Fi, then install drivers afterward (Chapter 22)

### "No Space Left on Device" Error

- You need at least 25 GB of free space
- If dual-booting, you may need to free up space in Windows first (delete large files, run Disk Cleanup)
- If your disk is very small, consider a **minimal installation**

### GRUB Doesn't Show Windows (Dual Boot)

- This can happen if Windows overwrote GRUB during a Windows Update
- Boot from your Ubuntu USB, select "Try Ubuntu"
- Open a terminal and run `sudo grub-install` and `sudo update-grub`
- Reboot — Windows should now appear in the GRUB menu
- We'll cover this in detail in Chapter 30

## What's Next

Congratulations — you now have a fully installed Ubuntu 26.04 LTS system! But your work isn't quite done. A fresh installation needs some immediate attention: system updates, essential software installation, driver verification, and enabling Ubuntu Pro. In **Chapter 7 (Post-Install Checklist)**, we'll walk through everything you should do in the first hour after installation to make sure your system is secure, up to date, and ready for daily use.

---

## Try It Yourself

1. **Complete the installation** using this chapter as your guide. Don't rush — read each screen carefully.
2. **Log in** and spend 10 minutes exploring the desktop. Open the Activities Overview, browse the app grid, open Files and navigate your home directory.
3. **Check your hardware** — Does Wi-Fi work? Does audio play? Is your screen resolution correct? Make a note of anything that doesn't work — we'll address it in Chapter 7 and Chapter 22.
4. **Connect to the internet** and open Firefox. Browse to a few websites to confirm everything is working.
5. **Write down any questions or concerns** that came up during installation. Bring them to Chapter 7.
