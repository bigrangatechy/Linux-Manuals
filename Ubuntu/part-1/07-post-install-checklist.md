# Chapter 7: Post-Install Checklist

## The First Hour

You've installed Ubuntu. You're logged in. The desktop is staring back at you. Now what?

A fresh Ubuntu installation is good to go for most basic tasks, but there are several things you should do in your first hour to make sure your system is secure, up to date, and properly configured. Think of this as the "unpack the boxes and set up the essentials" phase of moving into a new home.

This chapter is a checklist — work through it in order. Some steps are quick (a minute or two), others take longer. By the end, you'll have a system that's fully updated, secured, and ready for daily use.

> 💡 **You'll need the terminal for some of these steps.**
>
> Don't worry — we'll cover the terminal in depth starting in Chapter 13. For now, you just need to know how to open it and paste commands. Press the **Super key**, type "Terminal" or "Ptyxis," and press Enter. When a command starts with `sudo`, you'll be asked for your password — type it and press Enter. Nothing will appear on screen as you type your password. This is normal.

## 1. Run System Updates

This is the single most important step. Even though Ubuntu 26.04 was released in April 2026, there have been security patches, bug fixes, and package updates since then. Your installation media — even if it's a recent point release — won't have everything current.

### Using the GUI (Software Updater)

1. Press **Super**, type **"Software Updater"** and open it
2. It will check for updates automatically
3. If updates are available, click **"Install Now"**
4. Enter your password when prompted
5. Wait for the updates to download and install
6. If a kernel update was included, you'll be prompted to restart — do this when convenient

### Using the Terminal

Open your terminal and run:

```bash
sudo apt update

This command refreshes your system's list of available packages — it doesn't actually install anything. Think of it as checking what's new on the shelves.

Then run:

sudo apt full-upgrade

This downloads and installs all available updates. The full-upgrade variant (as opposed to plain upgrade) will also handle package dependencies that need to change during the update — it's the more thorough option.

When it finishes, if a kernel update was installed, restart:

sudo reboot

    💡 What's the difference between apt update and apt upgrade?

        apt update = "Check what updates are available" (refreshes the package index)
        apt upgrade = "Install the available updates"
        apt full-upgrade = "Install updates, including ones that require adding/removing packages" (most thorough)

    You always run update first, then upgrade or full-upgrade. We'll explain APT in detail in Chapter 18.

2. Check for and Install Proprietary Drivers

If you have an NVIDIA or AMD graphics card, or certain Wi-Fi chips, you'll want proprietary drivers for best performance. Ubuntu 26.04 made some changes to where you find these.
The New Way: Settings → About → Additional Drivers

    Open Settings (Super, type "Settings")
    Navigate to About (usually at the bottom of the left sidebar)
    Look for an "Additional Drivers" section or link
    If present, click it — your system will scan for available drivers
    Select the recommended driver and click "Apply Changes"
    Wait for installation to complete
    Restart if prompted

The Classic Way: Install "Software & Updates"

Ubuntu 26.04 no longer includes the "Software & Updates" tool by default (it was removed from fresh installs). This was the traditional tool where you'd find the "Additional Drivers" tab. If you prefer the classic interface — many guides and tutorials still reference it — you can reinstall it:

sudo apt install software-properties-gtk

After installation, press Super, type "Software & Updates," and open it. Click the "Additional Drivers" tab to see and select drivers for your hardware.

Alternatively, you can find it in the App Center by searching for "Software & Updates."
The Terminal Way

You can also check and install drivers from the terminal:

ubuntu-drivers devices

This lists your hardware and any available drivers, marking the recommended ones. To install the recommended drivers automatically:

sudo ubuntu-drivers install

Or to install a specific driver (for example, an NVIDIA driver):

sudo apt install nvidia-driver-595

    ⚠️ Restart After Installing Graphics Drivers

    After installing NVIDIA or AMD drivers, always restart your system. The new driver needs to load at boot time. If you don't restart, you may experience graphical glitches or poor performance until you do.

3. Enable Ubuntu Pro (Free for Personal Use)

As we covered in Chapter 3, Ubuntu Pro extends your security coverage at no cost for personal use on up to 5 machines. It provides:

    Extended Security Maintenance (ESM) — security patches beyond the standard 5-year window
    Livepatch — apply kernel security updates without rebooting
    Broader package coverage for security updates

Enabling Through the Security Center (GUI)

Ubuntu 26.04 includes a new Security Center app where Ubuntu Pro is now managed:

    Press Super, type "Security Center" and open it
    Look for the Ubuntu Pro section
    Click the green "Enable Ubuntu Pro" button
    You'll be redirected to sign in with your Ubuntu One account (free to create if you don't have one)
    Authenticate in your browser
    Your system is automatically attached to the free personal subscription

Enabling Through the Terminal

If you prefer the terminal (or are setting up a server):

sudo pro status

This shows your current Ubuntu Pro status. To attach:

sudo pro attach

Follow the prompts — it will provide a URL to authenticate through your Ubuntu One account.

    ⚠️ Known Issue: "Not available for this version"

    Some users have reported seeing a message that Ubuntu Pro is "not available for this version" on fresh 26.04 installs. This appears to be a temporary issue that Canonical is working to resolve. If you encounter this:

        Check for updates first (sudo apt update && sudo apt full-upgrade) and try again
        Try the terminal method (sudo pro attach)
        If it persists, it may resolve within a few weeks as Canonical updates their Pro backend for the new LTS

    Ubuntu Pro is not required for basic system security — your standard 5-year LTS support is active regardless. Pro adds the extended coverage on top.

4. Enable the Firewall

Ubuntu comes with UFW (Uncomplicated Firewall) pre-installed but disabled by default. A firewall blocks unauthorized incoming connections to your computer. On a desktop behind a home router, this is lower priority than on a server directly exposed to the internet, but it's still good practice to enable it.
Enabling Through the Security Center (GUI)

    Open the Security Center app
    Look for the Firewall section
    Toggle it on
    Enter your password when prompted

Enabling Through the Terminal

sudo ufw enable

This activates the firewall with default rules: block all incoming connections, allow all outgoing connections. This is the correct configuration for a desktop computer — you can still browse the web, download files, and use apps, but nothing can connect to your machine from outside without explicit permission.

To verify it's running:

sudo ufw status

You should see: Status: active

    💡 What This Actually Does

    With the default UFW rules:

        You can browse the web ✓
        You can download files ✓
        You can use email ✓
        You can play online games (most use outgoing connections) ✓
        Someone trying to connect to your computer from outside is blocked ✓

    If you later set up services that need incoming connections (like a web server, SSH, or file sharing), you'll add specific rules to allow them. We'll cover this in Chapter 21 (Networking Basics).

5. Install Multimedia Codecs

If you checked "Install multimedia codecs" during installation, you may already have these. If you didn't, or you're not sure, installing them now ensures your system can play common audio and video formats.

Open your terminal and run:

sudo apt install ubuntu-restricted-extras

This meta-package includes:

    MP3 playback support
    H.264/H.265 video decoding
    Microsoft core fonts (Arial, Times New Roman, etc.)
    Flash player remnants (largely obsolete, but harmless)
    Other commonly restricted media codecs

    ⚠️ EULA Prompt

    During installation, you may see a license agreement screen (for the Microsoft fonts). Use the Tab key to highlight "OK" and press Enter to accept. This is a text-based prompt inside the terminal — you can't click it with your mouse.

If you want to be more selective, you can install individual codec packages instead:

sudo apt install gstreamer1.0-libav gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly ffmpeg

We'll cover codecs in detail in Chapter 11.
6. Set Up Flatpak (Optional but Recommended)

Ubuntu uses Snap as its built-in universal package format, and many apps are available through the App Center as Snaps. However, Flatpak is another popular universal format with a larger community-driven app store called Flathub. Many applications are available on Flathub that aren't in the Snap Store (and vice versa).

Having both Snap and Flatpak gives you the widest selection of software.
Installing Flatpak

sudo apt install flatpak

Enabling the Flathub Repository

sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

Installing the Flatpak Plugin for App Center (Optional)

If you want to browse and install Flatpak apps through the graphical App Center:

sudo apt install gnome-software-plugin-flatpak

After installation, restart your system (or log out and back in) for the changes to take effect:

sudo reboot

    💡 Snap vs. Flatpak — Do I Need Both?

    You don't need both, but having both gives you maximum software availability. We'll cover the differences between Snap, Flatpak, and traditional APT packages in detail in Chapter 18. For now, just know that:

        Snap is built into Ubuntu and works out of the box
        Flatpak + Flathub is a one-command install that expands your options
        They coexist peacefully — having both causes no problems

7. Set Up Backups

This is the step most people skip and later regret. Set up automated backups now, while your system is fresh and your data is minimal. It's much easier to configure a backup strategy with 2 GB of files than with 200 GB.
Using the Built-in Backups App (Deja Dup)

Ubuntu includes a backup tool called Backups (powered by Deja Dup, which uses Duplicity under the hood). It's a simple, graphical backup solution.

    Press Super, type "Backups" and open it
    Click "Create Backup" or go to Settings within the app
    Choose a storage location:
        An external USB drive
        A network share (NAS)
        A cloud service (Nextcloud, Google Drive, etc. — note that Google Drive integration in GNOME Files was removed in 26.04, but Deja Dup's own Google Drive support still works)
        A local folder (less ideal — if the drive fails, you lose both original and backup)
    Select folders to back up — at minimum, select your Home folder, or specific subfolders like Documents, Pictures, and important project files
    Set a schedule — daily or weekly is recommended
    Enable encryption (optional but recommended) — set a passphrase to encrypt your backups
    Click Save and let the first backup run

    ⚠️ The 3-2-1 Rule

    A good backup strategy follows the 3-2-1 rule:

        3 copies of your data (original + 2 backups)
        2 different storage types (e.g., external drive + cloud)
        1 backup stored offsite (in case of theft, fire, or flood)

    Deja Dup handles one backup location. For a full 3-2-1 strategy, consider using Deja Dup for local backups and a cloud service for offsite. We'll cover backup strategies in depth in Chapter 27.

Testing Your Backup

A backup you haven't tested is a hope, not a backup. After your first backup completes:

    Open the Backups app
    Click "Restore"
    Browse the backup and confirm your files are there
    You don't need to actually restore everything — just verify you can see your important files

8. Review Privacy Settings

Take a few minutes to configure your privacy and security preferences:

    Open Settings (Super → "Settings")

    Navigate to Privacy

    Review and configure:
        Screen Lock — set automatic lock after a period of inactivity (5–15 minutes is reasonable)
        Location Services — disable if you don't need location-aware features
        Camera — review which apps have camera access
        Microphone — review which apps have microphone access
        Connectivity — review network sharing options
        Trash — set automatic purging of trash after a certain period (30 days is reasonable)

    Navigate to the Security Center app:
        Verify firewall is enabled (from Step 4)
        Review Snap permission settings — in 26.04, apps now prompt for permissions
        Check disk encryption status

9. Configure Power and Display Settings

Depending on whether you're on a laptop or desktop, configure your power settings for your usage:
Laptops

    Open Settings → Power
    Configure:
        Screen blank — how long before the screen turns off (5–10 minutes on battery, 15+ minutes plugged in)
        Automatic suspend — when to put the computer to sleep
        Battery-aware display dimming — new in GNOME 50, dims the screen when battery is low
        Power button behavior — what happens when you press the power button (suspend, shut down, etc.)

Desktops

    Open Settings → Power
    Configure:
        Screen blank — when the display turns off
        Automatic suspend — usually set to "never" for desktops unless energy saving is important to you

Displays

    Open Settings → Displays
    Verify:
        Resolution is set to your monitor's native resolution
        Scale — if text appears too small or too large, adjust the scaling (GNOME 50 has improved fractional scaling)
        Refresh rate — if your monitor supports higher refresh rates (120 Hz, 144 Hz), select it
        Variable refresh rate (VRR) — new in GNOME 50, enable if your monitor supports it for smoother visuals
        Night light — reduces blue light at night, easier on your eyes

10. Install Essential Applications

Your fresh Ubuntu system comes with a solid set of default apps (Firefox, Files, Settings, etc.), but you'll likely want to add more. Here are common essentials:
Through the App Center (GUI)

Open the App Center (Super → "App Center") and search for:
Category	Recommended App	Notes
Office suite	LibreOffice	Usually pre-installed; if not, install it
Code editor	Visual Studio Code	Available as a Snap or .deb
Image editing	GIMP	Powerful open-source image editor
Vector graphics	Inkscape	Open-source Illustrator alternative
Screenshot tool	GNOME Screenshot	Usually pre-installed; Flameshot is a good alternative
Media player	VLC	Plays virtually every video/audio format
Messaging	Discord, Slack, Telegram	Available as Snaps
Cloud storage	Nextcloud Desktop	If you use Nextcloud
Note-taking	Obsidian	Available as Flatpak or .deb
Through the Terminal (APT)

For system utilities and command-line tools:

sudo apt install curl wget git vim htop tree unzip

These are common utilities you'll likely need at some point:

    curl and wget — download files from the internet via terminal
    git — version control (if you do any programming)
    vim — a powerful text editor (we'll cover it in Chapter 19)
    htop — an interactive process viewer (alternative to Resources for terminal use)
    tree — display directory structures in a tree format
    unzip — extract .zip archives

Through Flatpak

If you enabled Flatpak (Step 6), you can install apps from Flathub:

flatpak install flathub com.spotify.Client
flatpak install flathub com.discordapp.Discord
flatpak install flathub org.videolan.VLC

We'll cover package management in detail in Chapter 18.
11. Clean Up Unnecessary Snaps (Optional)

Ubuntu 26.04 ships with several Snap packages pre-installed that you may not need. Over time, old versions of Snap packages accumulate and consume disk space. To check what's installed:

snap list --all

To remove a specific Snap you don't want:

sudo snap remove --purge packagename

To remove all disabled (old, superseded) Snap versions at once:

LANG=C snap list --all | awk '/disabled/{print $1, $3}' | xargs -rn2 sudo snap remove --purge

    ⚠️ Don't Remove Essential Snaps

    Some Snaps are part of the system infrastructure. Don't remove:

        snapd itself (the Snap daemon)
        core and core18, core20, core22 (base runtime packages)
        Anything you actively use

    When in doubt, leave it. The disk space savings from removing a few unused Snaps are usually small. This step is optional and primarily for users who want a lean system.

12. Create a Restore Point (Optional)

If you want the ability to roll back your system to this clean state, you can set up a snapshot system. Ubuntu doesn't include a built-in "System Restore" like Windows, but there are tools:
Timeshift

Timeshift is a popular system restore tool that takes snapshots of your system (similar to Time Machine on macOS or System Restore on Windows):

sudo apt install timeshift

After installation, open Timeshift from the app menu and configure it to take periodic snapshots. These snapshots capture your system files (not personal data in your home folder) and can be restored if an update or change breaks your system.

    💡 Timeshift vs. Backups

        Timeshift protects your system — if an update breaks your desktop or a driver installation fails, you can roll back
        Deja Dup / Backups protects your personal data — if you accidentally delete a document, you can restore it

    They serve different purposes. Ideally, you use both. We'll cover this in depth in Chapter 27 (Backup and Recovery).

13. Final Reboot and Verification

After completing all the steps above, do one final restart:

sudo reboot

After rebooting, verify that everything is working:

    Login works — type your password, get to the desktop
    Internet works — open Firefox, visit a website
    Audio works — play a YouTube video or music file
    Display looks correct — resolution, scaling, and refresh rate are right
    Graphics are smooth — no tearing, stuttering, or glitches (especially if you installed NVIDIA drivers)
    Firewall is active — sudo ufw status shows "active"
    Updates are current — sudo apt update shows no pending updates (or only minor ones)

Checklist Summary

Here's the complete checklist for quick reference:

    Run system updates (sudo apt update && sudo apt full-upgrade)
    Check and install proprietary drivers (Settings → About → Additional Drivers)
    Enable Ubuntu Pro (Security Center or sudo pro attach)
    Enable the firewall (sudo ufw enable)
    Install multimedia codecs (sudo apt install ubuntu-restricted-extras)
    Set up Flatpak and Flathub
    Set up backups (Backups / Deja Dup app)
    Review privacy settings (Settings → Privacy + Security Center)
    Configure power and display settings (Settings → Power, Displays)
    Install essential applications
    Clean up unnecessary Snaps (optional)
    Set up Timeshift for system snapshots (optional)
    Final reboot and verification

What's Next

Your Ubuntu system is now fully updated, secured, and configured. In Chapter 8, we'll take a proper tour of the GNOME 50 desktop — exploring every corner of the interface, learning keyboard shortcuts, understanding workspaces, and discovering features that make you productive from day one.
Try It Yourself

    Work through the entire checklist. It might take 30–60 minutes depending on your internet speed and how many applications you install.
    Test your backup by restoring a single file from the Backups app.
    Customize your desktop — change your wallpaper (Settings → Background), try dark mode (Settings → Appearance), and experiment with the Activities Overview.
    Note anything that didn't work. If a driver didn't install, Wi-Fi isn't working, or something looks wrong, write it down. We'll address hardware issues in Chapter 22 and troubleshooting in Chapter 30.

