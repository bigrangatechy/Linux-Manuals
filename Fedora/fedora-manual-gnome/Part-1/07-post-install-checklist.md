# Chapter 7: Post-Install Checklist

## Your Fresh System — And What It's Missing

Fedora 44 is installed and booting. You're looking at the GNOME 50 desktop. But right now, your system is in its purest, most stripped-down state — a pristine free-software installation with no proprietary codecs, no third-party applications, no media plugins, and no performance tweaks.

This is by design. Fedora ships clean and lets you decide what to add. This chapter walks through the essential tasks that transform a bare Fedora installation into a complete, functional, daily-driver system.

Think of this as the "unpacking and setup" phase after moving in. The house is built; now you connect the utilities, arrange the furniture, and make it livable.

---

## Task 1: Update the System

This is always the first task. The ISO you installed from was created on a specific date. Since then, packages have been updated — security patches, bug fixes, kernel improvements. Bring your system current before anything else.

```bash
sudo dnf upgrade --refresh

The --refresh flag forces DNF to pull fresh repository metadata (rather than using whatever was cached on the ISO). This ensures you see all available updates.

Dependencies resolved.
===================================================================
 Package                    Arch    Version        Repository  Size
===================================================================
Upgrading:
 kernel                     x86_64  6.19.0-55.fc44 updates      12 M
 kernel-core                x86_64  6.19.0-55.fc44 updates      8.5 M
 kernel-modules             x86_64  6.19.0-55.fc44 updates      45 M
 firefox                    x86_64  138.0-1.fc44   updates      72 M
 gnome-shell                x86_64  50.0-4.fc44    updates      3.2 M
 ... (many more)

Transaction Summary
===================================================================
Upgrade  347 Packages

Total download size: 412 M
Is this ok [Y/n]:

Press Y and Enter. Downloads and installation take 5–20 minutes depending on your internet speed and disk speed.
Reboot After Updates

If a kernel update was included (it almost always is), reboot:

sudo reboot

The new kernel loads on the next boot. Verify:

uname -r

The version number should be higher than what shipped on the ISO.

    💡 Why Kernel Updates Matter

    Unlike Ubuntu, which ships HWE kernel updates periodically, Fedora updates the kernel regularly throughout each release's lifecycle. Each kernel update brings driver improvements, security patches, and hardware support enhancements. Fedora also keeps the previous kernel installed — so if a new kernel causes problems, you can select the old one from the GRUB boot menu.

Task 2: Configure DNF for Speed

Fedora 44 ships with DNF5 — the next-generation package manager that's significantly faster than its predecessor. But out of the box, DNF5 uses conservative defaults. A few configuration tweaks make it dramatically faster.
Enable Parallel Downloads

By default, DNF downloads packages one at a time. On a decent internet connection, downloading 10 packages simultaneously is dramatically faster.

sudo dnf config-manager --setopt=max_parallel_downloads=10

Enable Fastest Mirror

Fedora distributes packages across hundreds of mirror servers worldwide. By default, DNF picks a mirror somewhat randomly. Enabling fastestmirror makes DNF test mirror latency and use the quickest one:

sudo dnf config-manager --setopt=fastestmirror=True

Verify Your Changes

cat /etc/dnf/dnf.conf

You should see:

[main]
max_parallel_downloads=10
fastestmirror=True

    💡 What About Deltarpm?

    Older Fedora guides tell you to enable deltarpm=True to download only the differences between installed and updated packages. This no longer applies. Fedora dropped delta RPM support in Fedora 40. DNF5 doesn't support it. Don't add this option — it does nothing.

    DNF5 is already so fast that the overhead of deltarpms wasn't worth the computational cost of reconstructing full packages from deltas on the client side.

Task 3: Enable RPM Fusion

This is the most important post-install task for most users. RPM Fusion provides the multimedia codecs, drivers, and software that Fedora's free-software policy excludes from the official repositories.
What RPM Fusion Provides
Repository	Contents
RPM Fusion free	Free software that Fedora can't ship due to US patent law (multimedia codecs like MP3, H.264, AAC; ffmpeg with full codec support)
RPM Fusion nonfree	Non-free software that's distributable but not open source (NVIDIA drivers, some firmware, some media tools)
Enabling Both Repositories

sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

This command:

    Detects your Fedora version automatically ($(rpm -E %fedora) expands to 44)
    Downloads and installs the RPM Fusion repository configuration packages
    Imports the GPG keys (you'll be asked to confirm)

After installation, verify:

dnf repolist | grep rpmfusion

You should see:

rpmfusion-free            RPM Fusion for Fedora 44 - Free
rpmfusion-free-updates    RPM Fusion for Fedora 44 - Free - Updates
rpmfusion-nonfree         RPM Fusion for Fedora 44 - Nonfree
rpmfusion-nonfree-updates RPM Fusion for Fedora 44 - Nonfree - Updates

Install Multimedia Codecs

With RPM Fusion enabled, install the essential media codec packages:

sudo dnf groupupgrade core
sudo dnf groupinstall multimedia --with-optional

The core group upgrade pulls in packages that RPM Fusion adds to the core group (like GStreamer plugins). The multimedia group installs a comprehensive set of audio and video codecs:

    GStreamer plugins — for media playback in GNOME apps
    ffmpeg — the universal media framework
    Codec libraries — MP3, AAC, H.264, HEVC, VP9, AV1

Additionally, install these for hardware-accelerated video playback:

# For Intel/AMD GPUs:
sudo dnf install ffmpeg-libs libva libva-utils
sudo dnf swap libva-intel-media-driver intel-media-driver --allowerasing
sudo dnf install mesa-va-drivers-freeworld

# For the open H.264 codec used by Firefox:
sudo dnf config-manager --setopt fedora-cisco-openh264.enabled=1
sudo dnf install openh264

What You Gain

After installing codecs:
Before	After
MP3 files won't play	MP3 playback works in all apps
MP4 videos show black screen	Video playback works
Web videos may stutter	Hardware-accelerated decoding
GNOME Music can't play FLAC	Full audio format support
Thumbnails don't generate for videos	Video thumbnails appear in Files

    ⚠️ RPM Fusion Bug in Early Fedora 44

    Shortly after Fedora 44's release, a packaging bug was identified in the RPM Fusion release package that incorrectly enabled the Rawhide repository (Fedora's development branch) instead of the proper Fedora 44 repository. If you see errors mentioning "Rawhide" when running dnf, check your repository configuration:

    dnf repolist

    If you see rawhide listed, disable it:

    sudo dnf config-manager --set-disabled rawhide

    This bug is expected to be fixed in an updated RPM Fusion release package. Check RPM Fusion's website for the latest version.

Task 4: Enable Flathub

Fedora ships with Flatpak support built in, and the Fedora repository includes a curated set of Flatpaks. But the main event is Flathub — the primary community Flatpak repository with thousands of applications.
Enable Flathub

flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Verify

flatpak remotes

You should see:

Name    Options
fedora  system
flathub system

Restart GNOME Software

After adding Flathub, restart GNOME Software so it picks up the new repository:

gnome-software --quit

Then reopen GNOME Software from the Activities Overview. You'll now see a much wider selection of applications when you search — including proprietary apps like Spotify, Discord, Zoom, and Telegram that aren't in Fedora's repos.

    💡 Fedora's Flatpak vs. Flathub

    Fedora maintains its own Flatpak repository (fedora) with a subset of applications built from Fedora packages. Flathub is a separate, community-maintained repository with a much larger catalog.

    When both repos provide the same app, GNOME Software may prefer the Fedora version. Flathub versions are often newer and more frequently updated. If you want to ensure you're getting Flathub versions, you can set Flathub as the priority:

    flatpak remote-modify flathub --priority=1

Task 5: Install GNOME Tweaks and Extensions Manager

Fedora's GNOME 50 is intentionally minimal — no desktop icons, no minimize/maximize buttons by default, no dock. GNOME Tweaks and Extension Manager let you customize GNOME to your liking.
GNOME Tweaks

sudo dnf install gnome-tweaks

GNOME Tweaks exposes settings that aren't in the standard Settings app:

    Window title bar buttons (add minimize/maximize)
    Theme and icon selection
    Font rendering settings
    Startup applications
    Top bar settings (show seconds, weekday)
    Workspaces configuration

Extension Manager

sudo dnf install extension-manager

Extension Manager (a separate app from GNOME Extensions) lets you browse, install, enable, and configure GNOME Shell extensions:

    Dash to Dock — turns the Activities dash into a permanent dock
    Blur My Shell — adds blur effects to the top bar and overview
    AppIndicator — shows tray icons for apps that use them
    Clipboard Indicator — clipboard history
    Just Perfection — fine-grained GNOME UI customization

Open Extension Manager (Super → "Extension Manager"), browse the "Browse" tab, and install extensions with one click.

    💡 Extensions and GNOME 50

    GNOME 50 is a major release, and some older extensions may not yet be updated for it. Extension Manager will only show extensions compatible with your GNOME version. If an extension you loved doesn't appear, check back in a few weeks — extension authors typically update within a month of a major GNOME release.

Task 6: Restore Window Buttons

By default, GNOME 50 shows only a Close button on windows. No minimize, no maximize. This is intentional — GNOME's design philosophy favors workspace switching and the Activities Overview over minimize/maximize. But if you prefer traditional window controls:

Using GNOME Tweaks:

    Open GNOME Tweaks (Super → "Tweaks")
    Go to Windows → Titlebar Buttons
    Toggle Minimize and Maximize on
    Close Tweaks

Your windows now have Minimize, Maximize, and Close buttons in the top-right corner (or top-left, configurable).

Using the command line:

gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'

To revert to GNOME's default:

gsettings reset org.gnome.desktop.wm.preferences button-layout

Task 7: Check for Firmware Updates

Fedora includes fwupd (firmware update daemon) — a tool that can update firmware on supported hardware (BIOS/UEFI, SSDs, docks, monitors, input devices).
Check for Firmware Updates

fwupdmgr refresh
fwupdmgr get-updates

If updates are available:

sudo fwupdmgr update

Follow the prompts. Some firmware updates require a reboot to complete.
Enable Automatic Firmware Updates

sudo fwupdmgr enable

This configures fwupd to check for and apply firmware updates automatically when available.

    💡 Vendor Firmware Tools

    fwupd works with LVFS (Linux Vendor Firmware Service) participating vendors — including Dell, Lenovo, HP, Logitech, SteelSeries, and others. If your hardware vendor doesn't participate in LVFS, you may need their proprietary tool for firmware updates (e.g., Samsung Magician for Samsung SSDs).

    GNOME Software also surfaces firmware updates — they appear alongside software updates when you open the app.

Task 8: Configure Power Management

Fedora 44 includes power-profiles-daemon by default, which provides three power profiles:
Profile	Behavior	When to Use
balanced	Default. Normal performance and power consumption.	Everyday use
performance	Maximum CPU performance, more power draw, faster battery drain.	Heavy compilation, gaming, rendering
power-saver	Reduces CPU frequency, dims screen, extends battery life.	On battery, trying to stretch runtime
Check Current Profile

powerprofilesctl get

Switch Profiles

powerprofilesctl set performance
powerprofilesctl set balanced
powerprofilesctl set power-saver

GNOME Integration

GNOME 50 has built-in power profile switching in the system menu (top-right corner → click the power/battery icon). You'll see the three profiles as options when on battery power. The Settings → Power panel also shows power mode options.

    ⚠️ Don't Install TLP Alongside power-profiles-daemon

    TLP is a popular power management tool on Linux. However, TLP and power-profiles-daemon conflict with each other — they fight over the same kernel interfaces. Fedora 44 ships with power-profiles-daemon. Don't install TLP unless you first remove power-profiles-daemon.

    For most laptop users, power-profiles-daemon is sufficient. TLP offers more granular control (USB autosuspend, PCIe ASPM, etc.) but requires more configuration knowledge.

Task 9: Set Up Btrfs Snapshots with Snapper

Fedora 44 uses Btrfs as its default filesystem, which supports snapshots — point-in-time copies of the filesystem that can be used to roll back changes. But snapshots aren't automatic out of the box. You need to set up Snapper.
What Snapper Does

    Pre/post snapshots — automatically takes a snapshot before and after every DNF transaction. If an update breaks something, you can roll back.
    Timeline snapshots — takes periodic snapshots (hourly, daily, weekly) so you always have recent restore points.
    Rollback — boot from a previous snapshot to restore the system to a known-good state.

Installation

sudo dnf install snapper python3-dnf5-plugins-snapper

The python3-dnf5-plugins-snapper package hooks Snapper into DNF5, so every dnf install, dnf upgrade, and dnf remove automatically creates before-and-after snapshots.
Configure Snapper for the Root Volume

sudo snapper -c root create-config /

This creates a configuration file at /etc/snapper/configs/root that controls snapshot frequency, retention, and cleanup for the root filesystem (/).
Enable Timeline Snapshots

sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer

The timeline timer creates snapshots on a schedule. The cleanup timer removes old snapshots based on retention rules (keeping a configurable number of hourly, daily, weekly, monthly, and yearly snapshots).
Verify It's Working

sudo snapper ls

You should see at least one snapshot (the initial one created by create-config). Run a DNF transaction and check again:

sudo dnf install nano
sudo snapper ls

You should see pre and post snapshots from the nano installation.
Install grub-btrfs (For Bootable Snapshots)

To boot directly from a snapshot via GRUB (for system rollback), install grub-btrfs:

sudo dnf install grub-btrfs
sudo grub2-mkconfig -o /boot/grub2/grub.cfg

With grub-btrfs, your GRUB boot menu will include entries for each Btrfs snapshot. You can boot into a previous snapshot, verify the system works, and then roll back permanently using:

sudo snapper rollback

    💡 Snapshots Are Not Backups

    Btrfs snapshots are stored on the same disk as the data they snapshot. If your disk fails, both the data and the snapshots are lost. Snapshots protect against software mistakes (bad updates, accidental deletions) but not hardware failure. You still need real backups (Chapter 26) for disaster recovery.

Task 10: Install Essential Applications

Now that RPM Fusion and Flathub are enabled, install the applications you'll use daily. Here's a curated starter set:
From DNF (RPM packages)

# VLC media player (plays virtually anything)
sudo dnf install vlc

# GIMP (image editor)
sudo dnf install gimp

# Inkscape (vector graphics)
sudo dnf install inkscape

# Audacity (audio editor)
sudo dnf install audacity

# htop (terminal process monitor)
sudo dnf install htop

# wget and curl (download tools)
sudo dnf install wget curl

# git (version control)
sudo dnf install git

# Syncthing (file synchronization)
sudo dnf install syncthing

From Flathub (Flatpak applications)

# Spotify
flatpak install flathub com.spotify.Client

# Discord
flatpak install flathub com.discordapp.Discord

# OBS Studio (screen recording/streaming)
flatpak install flathub com.obsproject.Studio

# Telegram
flatpak install flathub org.telegram.desktop

# Bitwarden (password manager)
flatpak install flathub com.bitwarden.desktop

# Thunderbird (email)
flatpak install flathub org.mozilla.Thunderbird

# Bottles (Windows application runner)
flatpak install flathub com.usebottles.bottles

# Extension Manager (if not installed via DNF)
flatpak install flathub com.mattjakeman.ExtensionManager

Or simply open GNOME Software, search for these apps, and click Install.
Fonts (Optional)

If you work with documents created on Windows, install Microsoft-core-compatible fonts:

sudo dnf install mscorefonts2

(Provided by RPM Fusion.) This gives you Arial, Times New Roman, Courier New, Comic Sans, and other common fonts — useful for document compatibility.
Task 11: Enable FUSE for AppImage Support

AppImage is a portable application format — download a single file, make it executable, and run it. No installation needed. Fedora 44 doesn't include FUSE support by default, which AppImages require.

sudo dnf install fuse-libs

Now you can download AppImages from participating projects, make them executable, and run them:

chmod +x SomeApp.AppImage
./SomeApp.AppImage

Task 12: Install NVIDIA Drivers (If Applicable)

If your computer has an NVIDIA GPU, the open-source Nouveau driver is loaded by default. It works for basic display output but lacks 3D acceleration and features like CUDA support.
Check Your GPU

lspci | grep -i vga

If you see NVIDIA, install the proprietary drivers from RPM Fusion:

sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda

The akmod system automatically builds the NVIDIA kernel module for your running kernel. This process takes 2–5 minutes and happens in the background after installation.

For CUDA support (video editing, machine learning, 3D rendering):

sudo dnf install xorg-x11-drv-nvidia-cuda

Reboot after installation:

sudo reboot

After reboot, verify the driver is loaded:

nvidia-smi

This should display a table showing your GPU, driver version, and CUDA version.

    ⚠️ Secure Boot and NVIDIA

    If you have Secure Boot enabled, the NVIDIA kernel module needs to be signed. Fedora's akmod system handles this — but you'll be prompted to create a MOK (Machine Owner Key) password during driver installation. After reboot, a blue MOK management screen appears. Choose "Enroll MOK" → "Continue" → enter the password you set. This enrolls the signing key, allowing the NVIDIA module to load with Secure Boot enabled.

    If you skip the MOK enrollment, the NVIDIA driver won't load, and you'll fall back to Nouveau. See Chapter 22 for full NVIDIA setup details.

    💡 Intel and AMD Users

    If you have an Intel or AMD GPU, you don't need proprietary drivers. The open-source drivers are the official drivers — they're built into the kernel and fully supported. Fedora works perfectly out of the box with Intel and AMD graphics. No post-install driver installation needed.

Task 13: Configure Firefox Hardware Acceleration

Firefox ships with Fedora, but hardware video acceleration isn't always enabled by default. Enabling it reduces CPU usage during video playback (YouTube, Netflix) and improves battery life on laptops.

    Open Firefox
    Type about:config in the address bar
    Accept the risk warning
    Search for and set these preferences:

Preference	Value
media.ffmpeg.vaapi.enabled	true
gfx.webrender.all	true

Restart Firefox. Video playback should now use GPU decoding.

    💡 Verify It's Working

    Open Firefox, play a YouTube video, then open a terminal and run:

    sudo dnf install intel-gpu-tools  # For Intel GPUs
    sudo intel_gpu_top

    If you see "Video" usage in the output, hardware acceleration is working. For AMD GPUs, sudo dnf install radeontop and run radeontop.

Task 14: Configure GNOME Settings

Spend a few minutes in GNOME Settings (Super → "Settings") configuring your system:
Setting	Location	Recommendation
Night Light	Displays → Night Light	Enable. Reduces blue light in the evening.
Touchpad tap-to-click	Mouse & Touchpad	Enable if using a laptop.
Fractional scaling	Displays	Enable on hi-DPI displays for better sizing (new in GNOME 50).
Power button behavior	Power	Set to "Suspend" (laptops) or "Power Off" (desktops).
Automatic suspend	Power → Power Saving	Set timeout (e.g., 20 min on battery, never when plugged in).
File history	Privacy → File History	Enable if you want recent files tracking.
Screen blank	Power → Screen Blank	Set to 5 minutes on battery, 10–15 minutes plugged in.
Date format	Date & Time	Set to 24-hour or 12-hour as preferred.
Default applications	Default Applications	Set your preferred browser, email, calendar, etc.
Set a Wallpaper

Right-click on the desktop → Change Background. Fedora 44 ships with a set of default wallpapers. Choose one, or use your own image.
Task 15: Install Toolbox (Fedora's Signature Tool)

Toolbox is a Fedora-exclusive tool for creating containerized development environments. It uses Podman (a container runtime) to create lightweight, isolated environments that share your home directory but have their own package set.
Installation

sudo dnf install toolbox

Create Your First Toolbox

toolbox create

This creates a container named after your username, running the same Fedora version as your host. Enter it:

toolbox enter

Inside the toolbox, you have a full Fedora environment. You can install packages with dnf install without sudo (since the container runs as your user). Anything you install inside the toolbox stays there — it doesn't affect your host system.
Why Use Toolbox?
Scenario	Benefit
Testing a library or framework	Install it in a toolbox without polluting your host
Running conflicting package versions	Different toolboxes can have different versions
Trying a risky operation	If it breaks the toolbox, just delete it — your host is unaffected
Following a tutorial	Install dependencies in a throwaway toolbox

Toolbox is covered in depth in Chapter 23. For now, just install it and create your first container — it's a taste of Fedora's developer-friendly nature.
Task 16: Set Up Backups

Don't skip this. If you do nothing else from this chapter, set up backups.
Déjà Dup (Graphical, Simple)

Déjà Dup is GNOME's backup tool — simple, encrypted, and integrates with cloud storage or local drives.

sudo dnf install deja-dup

Open Déjà Dup (Super → "Backups"), and configure:

    Storage location — an external drive, network share, or cloud service
    Folders to back up — your home directory, at minimum
    Schedule — daily or weekly automatic backups
    Encryption — enable a passphrase to encrypt your backups

Btrfs Snapshots (Already Set Up in Task 9)

Snapshots protect against software issues (bad updates, accidental deletions) but not hardware failure. They complement, not replace, traditional backups.
Full Backup Coverage

For comprehensive protection, you need both:
Method	Protects Against	Stored Where	Frequency
Btrfs snapshots (Snapper)	Bad updates, accidental deletions	Same disk	Before every DNF transaction + hourly
Déjà Dup backup	Disk failure, theft, disaster	External/cloud	Daily or weekly

Chapter 26 covers backup strategies in detail, including rsync, BorgBackup, and network-based solutions.
Post-Install Checklist: Complete

Here's the consolidated checklist. Tick each one off:
#	Task	Command / Action	Done
1	Update the system	sudo dnf upgrade --refresh	☐
2	Configure DNF speed	max_parallel_downloads=10, fastestmirror=True	☐
3	Enable RPM Fusion	Install rpmfusion-free + nonfree release packages	☐
4	Install codecs	sudo dnf groupinstall multimedia + ffmpeg packages	☐
5	Enable Flathub	flatpak remote-add --if-not-exists flathub ...	☐
6	Install GNOME Tweaks	sudo dnf install gnome-tweaks	☐
7	Install Extension Manager	sudo dnf install extension-manager	☐
8	Restore window buttons	GNOME Tweaks → Windows → Titlebar Buttons	☐
9	Check firmware updates	fwupdmgr refresh && fwupdmgr get-updates	☐
10	Configure power management	Verify power-profiles-daemon, set profile	☐
11	Set up Snapper	Install snapper, create config, enable timers	☐
12	Install essential apps	DNF: vlc, gimp, git, etc. Flatpak: Spotify, Discord, etc.	☐
13	Install FUSE	sudo dnf install fuse-libs	☐
14	NVIDIA drivers (if needed)	sudo dnf install akmod-nvidia	☐
15	Firefox acceleration	Enable vaapi + webrender in about:config	☐
16	Configure GNOME settings	Night Light, touchpad, display scaling, etc.	☐
17	Install Toolbox	sudo dnf install toolbox + toolbox create	☐
18	Set up backups	Install Déjà Dup, configure storage + schedule	☐
A Post-Install Script

If you prefer to automate the essential tasks, here's a script that handles items 1–5, 7, 11, 13, and 17 in one run:

#!/bin/bash
set -euo pipefail

echo "╔══════════════════════════════════════════════╗"
echo "║     Fedora 44 Post-Install Setup Script     ║"
echo "╚══════════════════════════════════════════════╝"
echo ""

# 1. System update
echo "─── Updating System ───"
sudo dnf upgrade --refresh -y
echo ""

# 2. DNF speed tweaks
echo "─── Configuring DNF ───"
sudo dnf config-manager --setopt=max_parallel_downloads=10
sudo dnf config-manager --setopt=fastestmirror=True
echo "  Parallel downloads: 10"
echo "  Fastest mirror: enabled"
echo ""

# 3. RPM Fusion
echo "─── Enabling RPM Fusion ───"
sudo dnf install -y \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf groupupgrade -y core
sudo dnf groupinstall -y multimedia --with-optional
sudo dnf install -y ffmpeg-libs libva libva-utils
echo ""

# 4. Flathub
echo "─── Enabling Flathub ───"
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
echo ""

# 5. GNOME customization tools
echo "─── Installing GNOME Tools ───"
sudo dnf install -y gnome-tweaks extension-manager
echo ""

# 6. Snapper
echo "─── Setting Up Snapper ───"
sudo dnf install -y snapper python3-dnf5-plugins-snapper
sudo snapper -c root create-config /
sudo systemctl enable --now snapper-timeline.timer
sudo systemctl enable --now snapper-cleanup.timer
echo "  Snapper configured for /"
echo ""

# 7. FUSE
echo "─── Installing FUSE ───"
sudo dnf install -y fuse-libs
echo ""

# 8. Toolbox
echo "─── Installing Toolbox ───"
sudo dnf install -y toolbox
echo ""

# 9. Firmware check
echo "─── Checking Firmware ───"
fwupdmgr refresh
fwupdmgr get-updates
echo ""

echo "═══════════════════════════════════════════════"
echo "Essential post-install tasks complete!"
echo ""
echo "Remaining manual tasks:"
echo "  • Install applications (GNOME Software or DNF/Flatpak)"
echo "  • Configure GNOME Settings (Night Light, touchpad, etc.)"
echo "  • Install NVIDIA drivers if needed (Chapter 22)"
echo "  • Configure Firefox hardware acceleration"
echo "  • Set up backups (Déjà Dup)"
echo "═══════════════════════════════════════════════"

Save as ~/fedora-postinstall.sh, make executable, and run:

chmod +x ~/fedora-postinstall.sh
~/fedora-postinstall.sh

    ⚠️ Review Before Running

    Read through any script before executing it. This one does nothing destructive, but understanding what it does is good practice. Adapt it to your needs — remove tasks you don't want, add ones you do.

What's Next

Your Fedora system is now configured with codecs, applications, snapshots, and essential tools. The foundation is solid.

In Chapter 8, we'll step back and explore what a desktop environment is — the distinction between GNOME, KDE Plasma, XFCE, and others. You'll understand how the desktop environment relates to the underlying system, what a window manager does, and why Fedora Workstation uses GNOME by default. This conceptual foundation prepares you for the GNOME 50 desktop tour in Chapter 9.
Try It Yourself

    Run the system update. Execute sudo dnf upgrade --refresh and reboot. Confirm you're running the latest kernel with uname -r.

    Enable RPM Fusion and install codecs. Follow the commands in Task 3. Then test by opening a video file (download a sample MP4 from the web or copy one from another computer). If it plays in GNOME Videos, codecs are working.

    Enable Flathub and install an app. Follow Task 4. Then install something you'll use — Spotify, Discord, or any app from the Flathub catalog. Notice how it appears in your app grid after installation.

    Restore window buttons. Use GNOME Tweaks to add minimize and maximize buttons. Small quality-of-life adjustments like this make GNOME feel like yours.

    Set up Snapper. Follow Task 9. Then install a small package (sudo dnf install sl) and run sudo snapper ls to see the pre/post snapshots. You now have rollback protection for every DNF transaction.

    Install GNOME Tweaks and an extension. Install Extension Manager, browse the catalog, and install one extension that improves your workflow (Dash to Dock, Blur My Shell, or AppIndicator are popular first choices).

    Create a backup. Install Déjà Dup, configure it to back up your home directory to an external drive or cloud location, and run your first backup. Do this today, not "later."

    Take inventory. Open GNOME Software and browse the available apps. Search for each application on your "needed apps" list from Chapter 4. Confirm that alternatives exist for everything you use. Your system is now ready for daily use.

    ✅ Chapter 7 Complete You've updated the system, optimized DNF5 with parallel downloads and fastest mirror, enabled RPM Fusion for codecs and drivers, enabled Flathub for applications, installed GNOME Tweaks and Extension Manager, restored window buttons, checked firmware updates, configured power profiles, set up Btrfs snapshots with Snapper, installed essential applications, enabled FUSE for AppImages, installed NVIDIA drivers (if applicable), configured Firefox hardware acceleration, customized GNOME settings, installed Toolbox, and set up backups. Your Fedora 44 system is now a complete, functional, daily-driver workstation.

