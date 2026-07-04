# Chapter 5: Getting Ubuntu

## What You'll Need

Before we download anything, let's make sure you have everything lined up. Here's your checklist:

| Item | Details |
|---|---|
| A USB flash drive | At least **8 GB** capacity. This will be erased, so back up anything on it first. |
| A computer to create the USB | Windows, macOS, or another Linux machine — any will work. |
| An internet connection | The Ubuntu 26.04 ISO is roughly **4–5 GB** to download. |
| The computer you're installing on | This is your target machine. It can be the same one you use to create the USB. |

### System Requirements for Ubuntu 26.04 LTS

Ubuntu 26.04 LTS has modest hardware requirements:

| Component | Minimum | Recommended |
|---|---|---|
| Processor (CPU) | 1 GHz 64-bit (x86-64) | 2 GHz dual-core or faster |
| Memory (RAM) | 2 GB | 8 GB or more |
| Storage | 25 GB free disk space | 50 GB or more |
| Graphics | Any reasonable GPU | GPU with at least 256 MB video memory |
| Boot mode | UEFI (recommended) | UEFI |

A few notes on these numbers:

- **2 GB of RAM is the bare minimum** — the system will boot and run, but it won't be a pleasant experience. Web browsing with multiple tabs, running applications simultaneously, and general responsiveness will be sluggish. Aim for 8 GB if at all possible.
- **25 GB of disk space** gives you room for the system plus a reasonable amount of personal files and installed applications. If you plan to install a lot of software (especially games, media tools, or development environments), budget more.
- **UEFI boot** is the modern boot standard (replacing the old BIOS). Nearly all computers from the last decade support it. We'll check this during installation.
- **ARM processors**: If you're using an ARM-based machine (like a Raspberry Pi or an ARM laptop), you'll need a different ISO. The standard download is for x86-64 (Intel/AMD). We'll cover ARM separately if needed.

> 💡 **Not sure what your specs are?**
>
> - **On Windows**: Press Win+Pause/Break, or right-click "This PC" → Properties
> - **On macOS**: Click the Apple menu → "About This Mac"
>
> If your machine was made in the last 10 years and has at least 4 GB of RAM, you're almost certainly fine.

## Downloading Ubuntu

### Step 1: Get the ISO

The **ISO file** is a single large file that contains a complete, bootable copy of Ubuntu. Think of it as a snapshot of an entire operating system packed into one file. We'll write this file to a USB drive to create installation media.

1. Go to **[ubuntu.com/download/desktop](https://ubuntu.com/download/desktop)**
2. You should see **Ubuntu 26.04 LTS (Resolute Raccoon)** as the current download
3. Click the **Download** button
4. The file you'll receive is named something like: `ubuntu-26.04-desktop-amd64.iso`
5. Save it somewhere you can find it — your Downloads folder is fine

The download is large (around 4–5 GB), so be patient. If your connection is slow, consider using a download manager or leaving it running overnight.

> ⚠️ **Download only from the official source.**
>
> Always download Ubuntu from **ubuntu.com** or an official mirror linked from ubuntu.com. Downloading ISOs from third-party sites risks getting a modified or infected image. The official site is free, fast, and safe.

### What About ARM?

If you're running an ARM-based computer (some newer laptops, Raspberry Pi, etc.), the standard amd64 ISO won't work. Instead:

- Visit **[cdimage.ubuntu.com](https://cdimage.ubuntu.com)** for alternative architectures
- Look for the **arm64** build corresponding to Ubuntu 26.04 LTS
- The rest of the process (verification, USB creation) is the same

For the vast majority of readers on Intel or AMD machines, the standard download from ubuntu.com is correct.

### Ubuntu Flavors (Briefly)

You might notice that ubuntu.com also offers downloads for variants like **Kubuntu**, **Xubuntu**, **Lubuntu**, **Ubuntu MATE**, and others. These are called **Ubuntu flavors** — they use the same underlying Ubuntu system but ship with different desktop environments.

For example:

- **Kubuntu** uses the KDE Plasma desktop (more Windows-like in layout)
- **Xubuntu** uses Xfce (lightweight, good for older hardware)
- **Lubuntu** uses LXQt (even more lightweight)

This manual focuses on standard Ubuntu with GNOME 50. If you later decide a flavor suits you better, everything in this manual still applies — only the desktop interface changes. The terminal, package management, file system, and underlying system are identical.

## Verifying Your Download

This step is optional but **strongly recommended**. When you download a 5 GB file, there's always a small chance it could be corrupted during transfer — a network hiccup, a partial download, or (in rare cases) a malicious intermediary altering the file.

Verification confirms two things:

1. **Integrity** — the file isn't corrupted or incomplete
2. **Authenticity** — the file was actually produced by Canonical and hasn't been tampered with

Ubuntu provides **SHA-256 checksums** and **GPG signatures** for every release. Here's what that means in plain English:

- A **checksum** is like a fingerprint for a file. You run a tool that reads your downloaded file and produces a long string of characters. If this string matches the one Canonical published, your file is byte-for-byte identical to what Canonical released.
- A **GPG signature** proves that the checksum file itself was created by Canonical, not an impostor. It uses cryptographic signing — the same technology that makes HTTPS secure.

### Verification on Windows

1. **Open PowerShell** (search for "PowerShell" in the Start menu)
2. Navigate to your Downloads folder:

cd $env:USERPROFILE\Downloads

3. Calculate the checksum of your ISO:
```powershell
Get-FileHash ubuntu-26.04-desktop-amd64.iso -Algorithm SHA256

    This outputs a long hexadecimal string. Compare it to the official checksum at releases.ubuntu.com/26.04/SHA256SUMS

If the values match, your download is intact. If they don't, re-download the ISO — something went wrong during transfer.

    💡 Going further with GPG verification

    If you want to also verify that the checksum file itself is authentic (not just the ISO), you can use Gpg4win on Windows to import Canonical's signing key and verify the SHA256SUMS.gpg file. This is more involved and optional for most users. The official tutorial is at ubuntu.com/tutorials/how-to-verify-ubuntu.

Verification on macOS

    Open Terminal (Applications → Utilities → Terminal)
    Navigate to your Downloads folder:

    cd ~/Downloads

    Calculate the checksum:

    shasum -a 256 ubuntu-26.04-desktop-amd64.iso

    Compare the output to the official checksum at releases.ubuntu.com/26.04/SHA256SUMS

Verification on Linux

If you already have a Linux machine:

    Open your terminal
    Navigate to where the ISO is saved
    Run:

    sha256sum ubuntu-26.04-desktop-amd64.iso

    Compare the output to the official checksum

Verification Summary

The process boils down to:

You downloaded:  ubuntu-26.04-desktop-amd64.iso
You computed:    a2f5... (64-character hash)
Official says:  a2f5... (64-character hash)
                ✓ Match → Your file is good to go
                ✗ No match → Re-download

Don't skip this step if you can help it. A corrupt ISO is one of the most common causes of installation problems, and verification takes seconds.
Creating Your Bootable USB

Now that you have a verified ISO, you need to write it to a USB drive. This process is sometimes called "flashing" or "burning" an image. The result is a USB drive that your computer can boot from, launching the Ubuntu installer.

    ⚠️ WARNING: This will erase everything on your USB drive.

    The flashing process completely overwrites the USB drive's contents. Copy any important files off the drive before proceeding. There's no way to recover them afterward.

You'll need:

    Your verified ISO file
    A USB drive (8 GB minimum)
    A flashing tool (pick one below based on your current operating system)

Option A: BalenaEtcher (Windows, macOS, Linux)

BalenaEtcher is the simplest, most beginner-friendly tool. It works identically on Windows, macOS, and Linux. If you're unsure which tool to use, pick this one.

    Download BalenaEtcher from etcher.balena.io
    Install and launch it
    Click "Flash from file" and select your Ubuntu ISO
    Click "Select target" and choose your USB drive
    Click "Flash!"
    Wait for the process to complete — Etcher writes the image and then verifies it automatically
    When done, you'll see a "Flash Complete" message

Etcher handles partition schemes and file systems automatically, so there's nothing else to configure. It's genuinely three clicks.

    💡 Etcher also verifies the write.

    One advantage of Etcher is that it verifies the USB after writing, ensuring the written data matches the ISO. This catches issues with faulty USB drives — a surprisingly common problem.

Option B: Rufus (Windows Only)

Rufus is a popular, lightweight Windows tool that offers more configuration options than Etcher. Use this if you're on Windows and want more control, or if Etcher doesn't work for some reason.

    Download Rufus from rufus.ie
    Insert your USB drive
    Launch Rufus
    Configure the following:
        Device: Select your USB drive (double-check it's the right one!)
        Boot selection: Click "SELECT" and choose your Ubuntu ISO
        Partition scheme: GPT (for UEFI systems — this is the modern standard)
        Target system: UEFI (non CSM)
        File system: FAT32 (default)
    Click "START"
    If prompted about write mode, choose "Write in ISO Image mode"
    Confirm the warning that the drive will be erased
    Wait for the process to complete

    ⚠️ If your computer is older (pre-2012)

    Very old computers may use legacy BIOS instead of UEFI. If that's the case, change the partition scheme to MBR and the target system to BIOS (or UEFI-CSM). Most modern computers use UEFI, so GPT is the right choice for most readers.

Option C: Startup Disk Creator (Already on Linux)

If you're already running Ubuntu or another Linux distribution, you have a built-in tool:

    Press the Super key and search for "Startup Disk Creator"
    Insert your USB drive
    The tool should auto-detect your ISO — if not, click "Other" and browse to it
    Select your USB drive
    Click "Make Startup Disk"
    Confirm and wait

Option D: Disks Utility (Already on Linux)

Ubuntu also includes the Disks utility, which can write images:

    Press Super, type "Disks," and open it
    Select your USB drive in the left sidebar (make sure you select the right device)
    Click the menu (three dots or hamburger icon) and select "Restore Disk Image"
    Choose your Ubuntu ISO
    Click "Start Restoring" and confirm

Which Tool Should I Use?
Your Current OS	Recommended Tool	Why
Windows	BalenaEtcher (simple) or Rufus (advanced)	Both work well; Etcher is easier
macOS	BalenaEtcher	Cross-platform, simple, reliable
Linux	Startup Disk Creator or Disks	Already installed, no extra download needed
After Flashing: What's on the USB?

Once the flashing process completes, your USB drive contains a live Ubuntu environment — a complete, bootable copy of Ubuntu that runs entirely from the USB drive without touching your hard drive. This means you can:

    Try Ubuntu before installing — boot from the USB and explore the desktop without making any changes to your computer
    Run Ubuntu on any compatible computer — plug it in, boot, and you're in Ubuntu
    Install Ubuntu — when you're ready, the desktop has an "Install Ubuntu" shortcut

The USB drive is no longer visible as a normal storage device after flashing. It's now a bootable installer. If you want to use it as a regular USB drive again later, you'll need to reformat it (we'll cover that in Chapter 21).
Preparing Your Computer for Installation

Before you boot from the USB, there are a few things to check on your target computer:
Back Up Your Data

If you're installing Ubuntu alongside or replacing an existing operating system, back up your important files first. Installation involves partitioning your disk, and while the process is generally safe, things can go wrong. Copy irreplaceable files (photos, documents, project files) to an external drive or cloud storage.

We'll cover backup strategies in detail in Chapter 27, but for now: if you'd be upset to lose it, back it up.
Disable Fast Startup (Windows Only)

If your computer currently runs Windows, disable Fast Startup (also called Fast Boot). This feature puts Windows into a hibernation-like state instead of fully shutting down, which can interfere with disk operations during Ubuntu installation.

    Open Control Panel → Power Options
    Click "Choose what the power buttons do" (left sidebar)
    Click "Change settings that are currently unavailable" (requires admin privileges)
    Uncheck "Turn on fast startup"
    Save changes

Know Your Boot Key

To boot from your USB drive, you'll need to interrupt the normal boot sequence and tell your computer to boot from the USB instead of the hard drive. Different manufacturers use different keys:
Manufacturer	Common Boot Menu Key	Common BIOS/UEFI Key
Dell	F12	F2
HP	F9	F10
Lenovo	F12, Fn+F12	F1, F2
ASUS	Esc, F8	F2, Del
Acer	F12	F2, Del
MSI	F11	Del
Generic	Esc, F12	F2, Del

The Boot Menu key brings up a one-time boot device selector. The BIOS/UEFI key enters your firmware settings, where you can change the boot order permanently.

    💡 Timing matters.

    You need to press the key immediately after powering on — before the operating system starts loading. Turn on the computer and immediately start tapping the key repeatedly. You may need a few attempts to get the timing right. It's normal.

What You Need in BIOS/UEFI

When you enter your BIOS/UEFI settings, look for these options:

    Boot mode: Set to UEFI (not Legacy/CSM) if available
    Secure Boot: Ubuntu 26.04 supports Secure Boot, so you can leave it enabled. If you encounter issues, you can disable it temporarily.
    Boot order: Set USB drive as the first boot device, or use the one-time boot menu
    Fast Boot: Disable if present (different from Windows Fast Startup — this is a firmware-level setting)

A Note on Secure Boot

Secure Boot is a security feature that prevents unauthorized operating systems from booting. Ubuntu 26.04 LTS includes a signed bootloader, so it works with Secure Boot enabled. You should not need to disable it.

However, if you plan to use proprietary graphics drivers (particularly NVIDIA), you may encounter Secure Boot prompts later. Ubuntu handles this gracefully — it will ask you to set a MOK (Machine Owner Key) password during driver installation and walk you through the enrollment process on the next boot. We'll cover this in Chapter 22 when we discuss hardware and drivers.
Alternative: WSL (Windows Subsystem for Linux)

If you're not ready to commit to a full installation, Ubuntu is also available through WSL (Windows Subsystem for Linux) on Windows 10 and 11. WSL lets you run Ubuntu within Windows, giving you access to the terminal and Linux tools without dual-booting or replacing Windows.

However, WSL has significant limitations for our purposes:

    No desktop environment (WSL2 can run GUI apps, but it's not a full desktop experience)
    No direct hardware access (limited driver support, no low-level control)
    Runs on top of Windows (you're still dependent on Windows for core system functions)

WSL is great for developers who need Linux tools while staying in Windows. But for learning Ubuntu as an operating system — understanding the desktop, managing files, configuring hardware, experiencing the full system — we recommend a proper installation. This manual assumes a real Ubuntu installation.

If you do want to explore WSL anyway, you can install it by opening PowerShell as Administrator and running:

wsl --install -d Ubuntu

Then follow the on-screen prompts. Just be aware that much of this manual (desktop tour, GUI software management, hardware configuration) won't apply to a WSL setup.
What's Next

You should now have:

    ✅ A verified Ubuntu 26.04 LTS ISO file
    ✅ A bootable USB drive with Ubuntu written to it
    ✅ Your target computer prepared (data backed up, Fast Startup disabled, boot key identified)

In Chapter 6, we'll walk through the actual installation process — step by step, screen by screen, from booting the USB to your first login on a fully installed Ubuntu system.
Try It Yourself

    Download the ISO from ubuntu.com/download/desktop. Confirm you're getting Ubuntu 26.04 LTS.
    Verify the download using the checksum method for your operating system. It takes 30 seconds and saves you from potential headaches later.
    Create your bootable USB using the tool that matches your current operating system.
    Back up your important data if you haven't already. Seriously. Do this.
    Find your boot menu key using the table above. Write it down — you'll need it in the next chapter.
