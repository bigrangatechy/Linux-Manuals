# Chapter 5: Getting Fedora: Download and Create a Live USB

## What You'll Need

Before we start, gather these items:

| Item | Requirement |
|---|---|
| USB flash drive | 8 GB minimum (16 GB recommended). Everything on it will be erased. |
| Computer with internet | To download the ISO (~2.5 GB) |
| Time | 30–60 minutes (download + USB creation) |
| The computer where you'll install Fedora | This can be the same or a different machine |

> ⚠️ **The USB Will Be Erased**
>
> Every file currently on your USB drive will be destroyed during this process. If the drive contains anything important, copy it elsewhere before you begin. This is not reversible.

---

## Step 1: Download the Fedora 44 ISO

### What Is an ISO?

An **ISO file** (named after the ISO 9660 filesystem standard used on CDs) is a single file that contains a complete, exact image of an optical disc — or in our case, a complete Fedora installation image. It's a snapshot of everything needed to boot and install Fedora, compressed into one file with a `.iso` extension.

You don't open or run an ISO file directly. Instead, you **flash** (write) it onto a USB drive, turning that USB drive into a bootable Fedora installer — equivalent to inserting a Fedora installation DVD, but faster and rewritable.

### Where to Download

The official and only recommended source is the Fedora Project's website:

**→ [fedoraproject.org/workstation/download](https://fedoraproject.org/workstation/download)**

> ⚠️ **Download Only from the Official Site**
>
> Always download Fedora from `fedoraproject.org` or a direct mirror linked from the official site. Third-party download sites may serve modified, outdated, or malware-infected ISOs. The official site provides verified images with checksum signatures.

### Choosing the Right Edition

On the download page, you'll see several options. For this manual, you want:

**Fedora 44 Workstation → x86_64 (64-bit Intel/AMD)**

Here's how to choose:

| Option | Choose If... |
|---|---|
| **x86_64** | You have an Intel or AMD processor (the vast majority of PCs and laptops). This is the default and almost certainly what you want. |
| **aarch64** | You have an ARM-based computer (Raspberry Pi 4/5, Snapdragon X Elite laptop, some Chromebooks, Pinebook). |

Most readers should download the **x86_64** version. If you're unsure which CPU you have:

- **On Windows:** Right-click the Start button → Task Manager → Performance tab → CPU. Look for "Intel" or "AMD."
- **On macOS:** Apple menu → About This Mac. Apple Silicon (M1/M2/M3/M4) is ARM; Intel-based Macs are x86_64. (Note: Fedora on Apple Silicon is community-supported, not official.)

### The Download Itself

Click the download button for **Fedora Workstation 44 (x86_64)**. Your browser will begin downloading a file named something like:

Fedora-Workstation-Live-x86_64-44-1.8.iso

The file is approximately 2.3 GB. Depending on your internet speed, this takes anywhere from a few minutes to an hour.

    💡 Download Tip

    If the download is slow, Fedora automatically redirects you to a nearby mirror server. If it stalls or fails, try again — or use a download manager that supports resuming. Browsers sometimes choke on large downloads; Firefox handles this more gracefully than Chrome.

Also Download the CHECKSUM File

On the same download page (or at the download directory listing), look for a file called Fedora-Spins-44-1.8-x86_64-CHECKSUM or simply CHECKSUM. Download it to the same folder as your ISO.

This file contains the official SHA-256 hash — a cryptographic fingerprint — of the ISO. We'll use it in Step 2 to verify your download wasn't corrupted or tampered with.

If you can't find the CHECKSUM file separately, the download page usually displays the SHA-256 hash directly. Copy it down — you'll need it.
Step 2: Verify the Download (SHA-256 Checksum)
Why Verify?

When you download a 2.3 GB file, any of these can happen:

    Network corruption — a few bytes flip during transfer, silently corrupting the file
    Incomplete download — the browser thinks it's done but the file is truncated
    Malicious interception — someone on your network substitutes a modified ISO (rare, but possible)
    Server compromise — a hacked mirror serves a backdoored image (even rarer, but the checksum catches it)

A checksum is a mathematical fingerprint of a file. You run a program that reads your downloaded ISO and produces a hash — a long string of characters. If even a single byte differs from the original, the hash changes completely. By comparing your hash to the official one, you confirm your download is byte-for-byte identical to what Fedora released.
The SHA-256 Algorithm

Fedora uses SHA-256 (Secure Hash Algorithm 256-bit) — a cryptographic hash function that produces a 64-character hexadecimal string. It's the same standard used for Bitcoin mining and TLS certificates. It's considered computationally infeasible to forge — if your hash matches, your file is authentic.
How to Verify on Linux (Including Fedora Live USB)

If you're already on a Linux system (or if you've booted the live USB and want to verify before installing):

cd ~/Downloads
sha256sum Fedora-Workstation-Live-x86_64-44-1.8.iso

You'll see output like:

2a4a16c009244eb5ab2198700eb04103793b62407e8596f30a3e0cc8ac294d77  Fedora-Workstation-Live-x86_64-44-1.8.iso

Compare this hash to the one in the CHECKSUM file (or on the download page). They must match exactly — all 64 characters.

Automated comparison (if you downloaded the CHECKSUM file):

cd ~/Downloads
sha256sum -c Fedora-Spins-44-1.8-x86_64-CHECKSUM 2>/dev/null | grep workstation

If verification passes, you'll see:

Fedora-Workstation-Live-x86_64-44-1.8.iso: OK

If it fails, the ISO is corrupted — delete it and re-download.
How to Verify on Windows

Windows doesn't include a built-in checksum tool, but you have several options:

Option A: PowerShell (built into Windows 10/11)

    Open PowerShell (Start menu → type "PowerShell" → click it)
    Navigate to your Downloads folder:

    cd $env:USERPROFILE\Downloads

    Run:

    Get-FileHash .\Fedora-Workstation-Live-x86_64-44-1.8.iso -Algorithm SHA256

    Compare the output hash to the official one.

Option B: A graphical tool

Download a free checksum utility like 7-Zip (which adds a right-click "CRC SHA" option) or HashTab (adds a "File Hashes" tab to file properties). These show hashes in a GUI — no terminal needed.

    💡 PowerShell Quirk

    PowerShell's Get-FileHash outputs the hash in uppercase. The official Fedora checksums are lowercase. They're the same value — case doesn't matter in hexadecimal. ABC123 and abc123 are identical hashes.

How to Verify on macOS

macOS includes the shasum command:

    Open Terminal (Applications → Utilities → Terminal, or Spotlight → "Terminal")
    Navigate to your Downloads folder:

    cd ~/Downloads

    Run:

    shasum -a 256 Fedora-Workstation-Live-x86_64-44-1.8.iso

    Compare the output to the official hash.

What If the Hash Doesn't Match?

If your computed hash differs from the official one:

    Don't use the ISO. A corrupted ISO produces a broken installation or a non-booting USB.
    Delete the file. Don't try to "fix" it.
    Re-download. Try a different browser, a wired connection instead of Wi-Fi, or a different mirror.
    If it fails repeatedly, download the CHECKSUM file and verify the ISO's GPG signature (see the sidebar below). This catches the rare case of a compromised checksum.

    💡 Going Further: GPG Signature Verification

    Beyond SHA-256, Fedora digitally signs its CHECKSUM files with GPG (GNU Privacy Guard). This means even the checksum file itself is authenticated — you can verify that the checksum was genuinely produced by Fedora, not substituted by an attacker.

    This is more advanced and requires importing Fedora's GPG public key. Most users don't need this level of verification — SHA-256 is sufficient for peace of mind. But if you're security-conscious or working in a corporate environment, search the Fedora documentation for "verify ISO GPG" for step-by-step instructions.

Step 3: Create the Bootable USB

You have a verified ISO. Now you need to write it to a USB drive, making the drive bootable. Several tools can do this — we'll cover the most reliable ones for each operating system.
Method 1: Fedora Media Writer (Recommended — All Platforms)

Fedora Media Writer is the official tool from the Fedora Project. It's available for Windows, macOS, and Linux. It's the recommended method because:

    It's built by the Fedora team specifically for Fedora ISOs
    It downloads the ISO and verifies the checksum automatically
    It handles partitioning and formatting the USB drive
    It validates the written image after flashing
    It's free and open source

Installing Fedora Media Writer:
Platform	How to Install
Windows	Download from fedoraproject.org/fmw. Run liveusb-creator.exe as administrator.
macOS	Download the .dmg from the same page. Drag to Applications.
Linux (Fedora)	Already installed by default. If not: sudo dnf install mediawriter
Linux (other distros)	Install via Flatpak: flatpak install flathub org.fedoraproject.MediaWriter

Using Fedora Media Writer:

    Insert your USB drive (8 GB+). Note: everything on it will be erased.

    Launch Fedora Media Writer.

    You'll see two main options:
        "Download automatically" — FMW downloads the ISO for you (it picks the latest release) and writes it. This skips Step 1 entirely.
        "Select .iso file" — You point it to the ISO you already downloaded and verified. Use this if you followed Steps 1–2 above.

    Click "Custom image" (or "Select .iso") and browse to your downloaded ISO.

    Select your USB drive from the list of detected drives. Double-check the size and name to make sure you're selecting the USB, not an external hard drive or other storage.

    Click "Write to USB" (or "Create Live USB").

    Wait. The process takes 5–15 minutes depending on your USB speed. FMW will show a progress bar.

    When done, FMW validates the written image. You'll see "Write Complete" when finished.

    ⚠️ Windows USB Corruption Issue

    A well-known issue: after Fedora Media Writer (or any USB flashing tool) writes the ISO, if you let Windows access the USB drive before booting from it, Windows may "repair" or modify the drive's filesystem — causing the Fedora media check to fail at boot.

    The fix: After writing the USB, eject it cleanly (right-click → Eject in File Explorer), then insert it directly into the computer where you'll install Fedora, and boot from it immediately. Don't let Windows "help" by scanning or repairing the drive.

Method 2: balenaEtcher (Cross-Platform Alternative)

balenaEtcher is a popular, graphical USB flashing tool available for Windows, macOS, and Linux. It's not Fedora-specific but works well with Fedora ISOs.

Installing balenaEtcher:
Platform	How to Install
Windows	Download from balena.io/etcher. Run the installer.
macOS	Download the .dmg from the same page.
Linux (Fedora)	Download the RPM from GitHub releases and install: sudo dnf install ./balenaEtcher-*.rpm

Using balenaEtcher:

    Launch balenaEtcher
    Click Flash from file → select your Fedora ISO
    Click Select target → choose your USB drive
    Click Flash!
    Etcher writes the image, then validates it automatically
    You'll see "Flash Complete!" when done

    ⚠️ Etcher and USB "Bricking" Reports

    Some users have reported that balenaEtcher occasionally renders USB drives unusable (the drive stops responding entirely). This is rare and seems hardware-dependent. If it happens, the drive can sometimes be recovered using disk partitioning tools. Fedora Media Writer doesn't have these reports, which is why it's the recommended tool. If you use Etcher and it works, great — but FMW is the safer bet.

Method 3: dd (Linux/macOS Terminal — Advanced)

If you're comfortable with the command line, dd is the simplest method — no extra software needed. It's built into every Linux and macOS system.

    ⚠️ dd Is Dangerous If Misused

    dd will write to whatever device you tell it to — no confirmation, no undo. If you specify the wrong device (e.g., your main hard drive instead of the USB), you will destroy your data. Double-check the device name before pressing Enter.

Step 1: Identify your USB drive.

On Linux:

lsblk

NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda           8:0    1  14.9G  0 disk          ← this is the USB (14.9G, removable)
nvme0n1     259:0    0  468.1G  0 disk
├─nvme0n1p1 259:1    0   600M  0 part /boot/efi
├─nvme0n1p2 259:2    0    70G  0 part /
└─nvme0n1p3 259:3    0  397.5G  0 part /home

Look for a device whose SIZE matches your USB drive and whose TYPE is disk. In this example, sda is the USB (14.9 GB, marked as removable with RM:1).

On macOS:

diskutil list

/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *500.3 GB   disk0
   ...

/dev/disk4 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.5 GB     disk4
   1:                 DOS_FAT_32                         15.5 GB     disk4s1

Here, /dev/disk4 is the USB drive (15.5 GB, external).

Step 2: Unmount the USB drive.

On Linux:

sudo umount /dev/sda

On macOS:

diskutil unmountDisk /dev/disk4

Step 3: Write the ISO.

On Linux:

sudo dd if=Fedora-Workstation-Live-x86_64-44-1.8.iso of=/dev/sda bs=4M status=progress conv=fsync

On macOS:

sudo dd if=Fedora-Workstation-Live-x86_64-44-1.8.iso of=/dev/rdisk4 bs=4m

Parameter explanation:
Parameter	Meaning
if=	Input file — your ISO
of=	Output device — your USB drive (not a partition like sda1; use the whole device sda)
bs=4M	Block size — writes 4 MB at a time (faster than the default 512 bytes)
status=progress	Shows progress (Linux only; macOS shows progress by default)
conv=fsync	Ensures all data is physically written before reporting completion
rdisk4 (macOS)	The r prefix uses raw device access — significantly faster on macOS

Step 4: Wait for completion. Then sync the buffers:

sync

This ensures all data is flushed from memory to the USB drive before you remove it. Wait for sync to finish before unplugging.
Method 4: Ventoy (Advanced — Multi-ISO USB)

Ventoy takes a different approach. Instead of flashing one ISO to a USB drive, you install Ventoy onto the USB once. After that, you simply copy ISO files onto the drive like copying any file. Ventoy presents a boot menu listing all ISOs on the drive, and you pick which one to boot.

This is ideal if you:

    Want to try multiple Linux distributions
    Want to keep both Fedora Workstation and Fedora KDE ISOs handy
    Don't want to re-flash the USB every time

Installing Ventoy (on Windows or Linux):

    Download Ventoy from ventoy.net
    Extract the archive
    Run VentoyGUI.exe (Windows) or ./VentoyGUI.sh (Linux)
    Select your USB drive
    Click Install

Once installed, the USB drive appears as a normal FAT32/exFAT drive. Simply copy ISO files onto it:

USB_DRIVE/
├── Fedora-Workstation-Live-x86_64-44-1.8.iso
├── ubuntu-26.04-desktop-amd64.iso
└── linuxmint-22.3-cinnamon-64bit.iso

Boot from the USB, and Ventoy shows a menu listing all ISOs. Select one and press Enter.

    ⚠️ Ventoy and Fedora's Media Check

    Fedora's boot process includes a built-in media verification step (checking that the USB data matches the expected checksum). Some users report this check failing with Ventoy because Ventoy uses a different boot method than a direct flash. If this happens, press Esc to skip the media check — the ISO is likely fine (you verified it in Step 2). Alternatively, use Fedora Media Writer instead.

Method Comparison
Tool	Platform	Difficulty	Verifies After Write	Multi-ISO	Recommended For
Fedora Media Writer	Win/Mac/Linux	Easy	✅	❌	Most users (best choice)
balenaEtcher	Win/Mac/Linux	Easy	✅	❌	Alternative GUI users
dd	Linux/macOS	Advanced	❌	❌	Terminal-comfortable users
Ventoy	Win/Linux	Medium	❌	✅	Users testing multiple distros
Rufus	Windows only	Easy	❌	❌	Windows users (but see note)

    ⚠️ A Note on Rufus

    Rufus is a popular Windows USB tool. It works with Fedora ISOs but can occasionally produce USBs that fail Fedora's built-in media check, particularly when it writes in "DD Image mode" vs. "ISO Image mode." If you use Rufus and Fedora's media check fails at boot, skip the check (press Esc) or switch to Fedora Media Writer. For Fedora specifically, FMW produces the most reliable results.

Step 4: Boot From the USB

You have a bootable USB drive. Now you need to tell your computer to boot from it instead of its internal hard drive.
Step 4a: Insert the USB

Plug the USB drive into the computer where you want to install Fedora. If it's a desktop, use a rear USB port (directly on the motherboard) rather than a front-panel port or a USB hub — rear ports are more reliable for booting.
Step 4b: Access the Boot Menu or BIOS/UEFI

When your computer starts (or restarts), it briefly displays a logo (the manufacturer's logo) before loading the operating system. During this brief window, you can press a specific key to interrupt the normal boot sequence and choose a different boot device.

One-time boot menu keys by manufacturer:
Manufacturer	Boot Menu Key	BIOS/UEFI Key
Dell	F12	F2
HP	F9	F10
Lenovo	F12 (or Novo button)	F1 or F2
ASUS	Esc or F8	F2 or Del
Acer	F12	F2 or Del
MSI	F11	Del
Gigabyte	F12	Del
Toshiba	F12	F2
Samsung	F2 or Esc	F2
Custom build	F11 or F8	Del or F2

How to use the boot menu:

    Shut down the computer completely (not sleep, not hibernate — full shutdown)
    Turn it on and immediately start tapping the boot menu key (F12, F9, Esc, etc.) rapidly
    A menu should appear listing available boot devices
    Select your USB drive using the arrow keys and press Enter
    The Fedora boot screen appears

    💡 Timing the Keypress

    The window for pressing the boot key is narrow — typically 1–3 seconds. If you see the Windows or macOS loading screen, you missed it. Restart and try again. Tap the key repeatedly (2–3 times per second) from the moment you press the power button until the menu appears.

If the Boot Menu Doesn't Appear

If you can't access the boot menu, you may need to enter BIOS/UEFI settings instead:

    Restart and press the BIOS/UEFI key (F2, Del, etc.)
    Navigate to the Boot section
    Change the boot order so that "USB" or "Removable Device" is first
    Save and exit (usually F10)

Step 4c: Secure Boot (Important!)

Secure Boot is a UEFI feature that prevents unsigned or untrusted bootloaders from running. Fedora 44 is signed with a Microsoft-signed shim bootloader, meaning it works with Secure Boot enabled — you should not need to disable it.

However, if your computer refuses to boot from the USB:

    Enter BIOS/UEFI settings
    Find Secure Boot (usually under a "Security" or "Boot" tab)
    Try setting it to Disabled
    Save and try booting again
    After Fedora is installed, you can re-enable Secure Boot — Fedora's bootloader is signed

    💡 Secure Boot and NVIDIA

    Fedora's signed kernel works with Secure Boot enabled. However, if you install NVIDIA's proprietary drivers (Chapter 22), the NVIDIA kernel module needs to be signed for Secure Boot compatibility. Fedora's akmod system handles this automatically — but if you encounter issues with NVIDIA after enabling Secure Boot, see Chapter 22 and Chapter 29 for troubleshooting.

Step 5: The Fedora Boot Screen

When you boot from the USB, you'll see the Fedora boot menu:

Welcome to Fedora-Workstation-Live 44!

1. Start Fedora-Workstation-Live 44
2. Test this media & start Fedora-Workstation-Live 44
3. Troubleshooting

Option 1: Start Fedora-Workstation-Live 44

Boots directly into the live environment. Assumes the USB is fine. Fastest option.
Option 2: Test this media & start Fedora-Workstation-Live 44

Recommended. Runs a checksum verification on the USB drive, confirming the written data matches the original ISO. If the data is corrupted (from a bad flash, a defective USB, or Windows interfering), this catches it before you waste time booting into a broken environment. Takes about 10–30 seconds.
Option 3: Troubleshooting

Provides alternative boot modes:

    Test this media & start Fedora — same as Option 2
    Run a memory test — runs Memtest86+ (diagnoses faulty RAM)
    Boot from local drive — bypasses the USB and boots your installed OS

    💡 Run the Media Test

    Always choose Option 2 (Test this media) at least once. It costs you 15 seconds and prevents the frustration of discovering a corrupted USB after spending 20 minutes trying to figure out why the desktop won't load.

Step 6: The Live Environment

After the media test (or direct boot), Fedora loads into the live environment — a fully functional Fedora desktop running entirely from the USB drive and your computer's RAM.

Nothing is installed. Nothing is written to your hard drive. You can explore the desktop, test your hardware (Wi-Fi, audio, trackpad, display), browse the web, and try applications. When you remove the USB and reboot, your existing operating system is untouched.
What the Live Environment Lets You Do
Activity	How
Explore GNOME 50	Click around, try gestures, open Settings
Test Wi-Fi	Settings → Wi-Fi → connect to a network
Test audio	Play a sound test in Settings → Sound
Test display	Check resolution, scaling, multi-monitor
Test trackpad/mouse	Navigate, click, scroll, try gestures
Browse the web	Open Firefox (pre-installed in the live image)
Try LibreOffice	Write a document, test file compatibility
Check disk detection	Open GNOME Disks to verify your drives are visible
Install Fedora	Double-click "Install to Hard Drive" on the desktop
What the Live Environment Can't Do
Limitation	Why
Save changes	The live environment runs in RAM. When you reboot, everything resets.
Install software permanently	Anything you install (via DNF or GNOME Software) disappears on reboot.
Use proprietary drivers (reliably)	NVIDIA drivers can't load in the live environment without special steps.
Access encrypted drives	You can mount them manually, but it's not automatic.
Run at full speed	Everything runs from USB + RAM, not a fast SSD. The desktop will be slower than after installation.
Try Before You Buy

The live environment is your safety net. Before you install anything, use the live session to verify that your hardware works:

    Wi-Fi connects — Open Settings → Wi-Fi. If your card is detected and connects, you're good.
    Audio plays — Go to Settings → Sound. Click the test button.
    Display looks right — Check resolution, brightness control, and scaling.
    Trackpad works — Two-finger scroll, tap-to-click, multi-touch gestures.
    Keyboard layout is correct — Type a few special characters (~ ` | { }).
    Your hard drive is visible — Open GNOME Disks (press Super, type "Disks"). You should see your internal drives.

If all of these work in the live environment, they'll work after installation. If something doesn't work (e.g., Wi-Fi isn't detected), search for your hardware model plus "Fedora" — there may be a known fix (covered in Chapter 22).

    ⚠️ Live Environment Performance ≠ Installed Performance

    The live environment runs from a USB drive, which is much slower than an SSD. Don't judge Fedora's speed based on the live session. Once installed to your hard drive, Fedora 44 boots in seconds and runs responsively.

When the Live Session Is Over

When you've explored enough:

    To install: Double-click the "Install to Hard Drive" icon on the desktop. This launches the Anaconda installer (covered in Chapter 6).
    To exit without installing: Click the power icon in the top-right corner → Power Off. Remove the USB drive. Your existing operating system boots normally on the next restart.

Troubleshooting Common USB Issues
The USB Won't Boot

Symptoms: You insert the USB, restart, press the boot key, but the USB isn't listed — or selecting it does nothing.

Fixes:

    Try a different USB port. Rear ports on desktops, or USB-A instead of USB-C (some systems don't support USB-C booting reliably).
    Try a different USB drive. Cheap USB drives are notoriously unreliable for booting. A decent-quality drive (SanDisk, Kingston, Samsung) costs a few dollars more and saves hours of frustration.
    Re-flash the USB. The write may have failed. Re-run Fedora Media Writer.
    Check BIOS/UEFI boot order. The USB may need to be explicitly set as the first boot device.
    Disable Fast Boot. Some BIOS settings (like "Fast Boot" or "Ultra Fast Boot") skip USB detection for speed. Disable it.
    Check if UEFI vs. Legacy/CSM. Fedora's USB boots in UEFI mode. If your BIOS is set to Legacy/CSM only, switch to UEFI.

The Media Check Fails

Symptoms: You select "Test this media" and it reports a failure.

Fixes:

    Skip the check (press Esc) if you already verified the SHA-256 hash on the ISO itself. The data on the USB may be fine — the check is just stricter.
    Re-flash with Fedora Media Writer (preferred) instead of another tool.
    Try a different USB drive. The current one may be failing.
    Don't let Windows touch the USB. If you flashed on Windows, eject the USB immediately and don't plug it back into Windows before booting.

The Live Environment Freezes or Crashes

Symptoms: Fedora boots but freezes during the desktop session, or the screen goes black.

Fixes:

    Try the Troubleshooting menu → "Start Fedora in basic graphics mode." This uses a fallback graphics driver. If this works, the issue is likely your GPU driver — particularly with NVIDIA cards. The installed system will work once you install the proper drivers (Chapter 22).
    Try nomodeset boot parameter. At the boot menu, press e to edit the boot entry. Find the line starting with linux and add nomodeset at the end. Press Ctrl+X to boot. This disables kernel mode setting — a diagnostic measure, not a permanent fix.
    Check for faulty RAM. Use the Troubleshooting menu → "Run a memory test." Bad RAM causes random freezes that look like software problems.

USB Drive Becomes Unusable After Flashing

Symptoms: After writing the ISO, the USB drive appears empty, unreadable, or has wrong capacity in Windows/macOS.

Explanation: The ISO was written in "DD" mode, which creates a raw image on the USB. The resulting partition table doesn't look like a normal storage device to Windows/macOS.

Fix: Repartition the drive to restore it as a normal storage device:

    On Windows: Open diskpart (as administrator), select the disk, run clean, then create a new partition with create partition primary, format with format fs=fat32 quick.
    On macOS: Open Disk Utility, select the USB, click "Erase," choose "MS-DOS (FAT)" or "exFAT."
    On Linux: Open GNOME Disks, select the USB, click the menu (☰) → "Format Disk" → "GPT" → Format as "FAT" or "exFAT."

Preparing for Installation

If you've:

    ✅ Downloaded the Fedora 44 Workstation ISO
    ✅ Verified the SHA-256 checksum
    ✅ Written the ISO to a USB drive with Fedora Media Writer
    ✅ Booted from the USB successfully
    ✅ Tested your hardware in the live environment
    ✅ Confirmed everything works

You're ready to install. In Chapter 6, we'll walk through the Anaconda installer step by step — disk partitioning, encryption, user creation, and boot loader setup — all the way through to first boot.

If any hardware didn't work in the live session, don't panic. Most issues (especially GPU-related) are resolved by installing Fedora and then installing the appropriate drivers. Chapter 22 covers hardware and drivers in detail.
Quick Reference: The Complete Process
Step	Action	Tool	Time
1	Download Fedora 44 ISO	Browser → fedoraproject.org	5–30 min
2	Verify SHA-256 checksum	sha256sum / PowerShell / shasum	1 min
3	Write ISO to USB drive	Fedora Media Writer / balenaEtcher / dd	5–15 min
4	Boot from USB	Boot menu (F12/F9/Esc)	1 min
5	Test media (optional but recommended)	Fedora boot menu Option 2	15 sec
6	Explore live environment	GNOME desktop	As long as you like
7	Install to hard drive	Anaconda installer (Chapter 6)	15–30 min
Try It Yourself

    Download the ISO and CHECKSUM. Go to fedoraproject.org/workstation/download. Download the x86_64 Workstation ISO and the CHECKSUM file. Save both to your Downloads folder.

    Verify the checksum. Follow the instructions for your operating system (PowerShell, Terminal, or sha256sum). Confirm the hash matches before proceeding. This habit — verifying downloads — is a security best practice you'll use whenever you download large files.

    Install Fedora Media Writer. Download and install it for your OS. It's a small, clean tool — no adware, no bundled extras.

    Create your bootable USB. Insert an 8 GB+ USB drive (remember: it will be erased), open Fedora Media Writer, select your ISO, select the USB, and click Write. Wait for completion and validation.

    Boot from the USB. Restart your computer, access the boot menu, and select the USB drive. Choose "Test this media & start Fedora-Workstation-Live 44."

    Explore the live desktop. Spend 15–30 minutes in the live environment. Connect to Wi-Fi, test audio, adjust display settings, browse the web. This is your preview of Fedora 44 — everything you see will be available after installation.

    Write down what works and what doesn't. If Wi-Fi connects, note it. If the trackpad gestures work, note it. If something is missing, note it for investigation later. This list will be invaluable during and after installation.

    ✅ Chapter 5 Complete You understand what an ISO file is, where to download Fedora 44 officially, how to verify the SHA-256 checksum on any operating system, how to create a bootable USB using Fedora Media Writer (and alternatives like balenaEtcher, dd, and Ventoy), how to access the boot menu on any computer, how Fedora's Secure Boot compatibility works, what the Fedora boot menu options mean, how to use the live environment to test hardware, and how to troubleshoot common USB and booting issues.

