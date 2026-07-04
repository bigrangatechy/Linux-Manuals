# Chapter 10: Software Management (GUI)

## How Software Works on Ubuntu

On Windows, you typically install software by downloading a `.exe` file from a website and running it. On macOS, you download a `.dmg` and drag the app to your Applications folder. In both cases, you're responsible for finding the installer, trusting the source, updating the app manually, and removing it when you're done.

Ubuntu takes a different approach. Software comes from **repositories** — curated, organized collections of applications maintained by Canonical and the community. Think of it like an app store on your phone: you search, tap install, and the system handles downloading, installing, configuring, and updating.

This chapter covers the **graphical** side of software management — everything you can do without touching the terminal. We'll save terminal-based package management for Chapter 18, where you'll learn the full power of APT, Snap, and Flatpak from the command line.

## The Three Package Formats

Before we start clicking buttons, you need to understand that Ubuntu supports three different software packaging formats. Each has different characteristics:

### 1. APT / .deb Packages (Traditional)

These are the classic Ubuntu packages. They come from Canonical's repositories and are managed by **APT** (Advanced Package Tool). When you install a `.deb` package, it integrates tightly with your system.

- **Source**: Ubuntu repositories (curated by Canonical)
- **Updates**: Handled through the system update process
- **Sandboxing**: None — apps have access to your system like any native application
- **Best for**: Core system tools, libraries, and applications that don't need frequent version updates
- **Example**: `git`, `vim`, `htop`, LibreOffice (from repos)

### 2. Snap Packages

Snaps are Canonical's universal package format. A Snap bundles the application and all its dependencies into a single self-contained package that runs identically across different Linux distributions.

- **Source**: Snap Store (maintained by Canonical)
- **Updates**: Automatic in the background
- **Sandboxing**: Yes — Snaps run in confined environments with controlled permissions
- **Best for**: Applications that need to stay current (browsers, editors) or that benefit from sandboxing
- **Example**: Firefox (on Ubuntu, Firefox ships as a Snap by default), Spotify, Slack, Discord

### 3. Flatpak Packages

Flatpaks are a community-driven universal package format, similar in concept to Snaps but maintained by an independent project. They use a central repository called **Flathub**.

- **Source**: Flathub (community-maintained)
- **Updates**: Through system updates or manual commands
- **Sandboxing**: Yes — similar confinement to Snaps, with a different permission model
- **Best for**: Applications not available as Snaps, or when you prefer the Flathub ecosystem
- **Example**: Many open-source apps like OBS Studio, Steam, Blender, Telegram

> 💡 **Do I Need to Understand All Three?**
>
> Not deeply — not yet. The key takeaway is: **Ubuntu's App Center handles Snaps and .deb packages automatically.** Flatpak requires a one-time setup (which you did in Chapter 7's post-install checklist) and then works alongside the App Center through the GNOME Software plugin.
>
> You can have all three formats installed simultaneously without conflicts. The system figures out which format to use based on what you're installing.

## The App Center

The **App Center** is Ubuntu's primary graphical software manager. It's the app store — your one-stop shop for finding, installing, updating, and removing applications.

### Opening the App Center

- Press **Super**, type **"App Center"**, press Enter
- Or click the App Center icon in the Dock (orange shopping bag icon)

### What You'll See

The App Center opens to a browse view with several sections:

- **Featured** — Highlighted and promoted apps
- **Categories** — Browse by type (Development, Games, Graphics, Internet, etc.)
- **Search bar** — Type an app name to find it instantly
- **Installed** — View and manage apps already on your system (new in 26.04: this now includes both Snap and .deb packages)
- **Updates** — See pending updates and apply them

### Installing an Application

1. Use the **search bar** or browse **categories** to find an application
2. Click the app to see its detail page:
   - **Description** — what the app does
   - **Screenshots** — preview the interface
   - **Rating** — community ratings (1–5 stars)
   - **Size** — download size
   - **Version** — current version number
   - **Permissions** — what system resources the app can access (for Snaps)
   - **Channel** — for Snaps, you can choose stable, beta, or edge versions
3. Click **"Install"**
4. Enter your password when prompted
5. Wait for the download and installation to complete
6. The app appears in your application grid and is ready to launch

> 💡 **Snap vs. .deb in the App Center**
>
> Some applications appear in the App Center as both a Snap and a .deb package. When you search for an app, the App Center shows whichever format Canonical recommends — usually the Snap version. You can identify the format by looking at the badge on the app listing (Snap packages show a Snap icon).
>
> If you specifically want the .deb version of an app, you can check the source information on the app's detail page or install it via the terminal (Chapter 18).

### Managing Installed Applications

The **Installed** tab shows everything on your system:

1. Click **"Installed"** in the App Center
2. You'll see a list of all installed applications — both Snaps and .deb packages (new in 26.04)
3. Each entry shows:
   - App name and icon
   - Installed version
   - Available update (if any)
   - Size on disk
4. Click any app to:
   - **Remove** it from your system
   - **View details** — jump to the full app page
   - **Switch channel** (Snaps only) — move between stable/beta/edge versions

### Updating Applications

Updates in the App Center work in two ways:

**Automatic (Snaps):** Snap packages update automatically in the background. You don't need to do anything — when a new version is available, it downloads and installs silently. You may see a notification that an app was updated.

**Manual (.deb packages):** Traditional packages notify you through the **Software Updater** app (see below). You can also check manually:

1. Open App Center → **Updates** tab
2. Review what's available
3. Click **"Update All"** or update individual apps

### Removing Applications

1. Open App Center → **Installed** tab
2. Find the application
3. Click the **trash icon** or **Remove** button
4. Confirm the removal
5. Enter your password if prompted

The app is uninstalled cleanly, along with any dependencies that are no longer needed by other applications.

> ⚠️ **Removing System Components**
>
> The App Center generally prevents you from removing critical system packages. However, be cautious when removing apps you don't recognize — some may be system libraries or components that other software depends on. When in doubt, leave it installed.

### Switching Snap Channels

For Snap packages, you can choose which release channel to track:

- **Stable** — The default. Tested, reliable releases.
- **Candidate** — Pre-release versions being tested before stable. Mostly stable but may have minor issues.
- **Beta** — Development versions with new features. May be buggy.
- **Edge** — Bleeding edge. Use only if you know what you're doing.

To switch channels:

1. Open the app's detail page in App Center
2. Look for the **channel selector** (may be under a settings/gear icon)
3. Select the desired channel
4. The app updates to that channel's current version

> 💡 **When Would I Use Beta or Edge Channels?**
>
> Most users should stick to **Stable**. You might switch to Beta or Edge if:
>
> - A specific bug fix hasn't reached Stable yet and you need it urgently
> - You want to test a new feature before it's widely released
> - A developer asks you to help test a pre-release version
>
> Always switch back to Stable when the feature/fix lands in the Stable channel.

## Software Updater (System Updates)

The **Software Updater** is a separate app from the App Center, focused specifically on system updates (security patches, bug fixes, and package version bumps).

### When It Appears

Software Updater runs automatically in the background. When updates are available:

1. A notification appears in the top-right of your screen
2. You can click it to open Software Updater directly
3. Or search for "Software Updater" in the Activities Overview

### Using Software Updater

1. It automatically scans for available updates
2. Displays a list of packages with available updates, grouped by:
   - **Security updates** (prioritized)
   - **Recommended updates** (bug fixes)
   - **Other updates** (minor version bumps)
3. Click **"Install Now"** to apply all updates
4. Enter your password
5. Wait for download and installation
6. If a kernel update was included, you'll be prompted to restart

### Scheduling Updates

By default, Software Updater checks daily and notifies you when updates are available. You can configure this behavior:

1. Open **Software Updater**
2. Click **Settings** (gear icon)
3. Configure:
   - **Update frequency** — daily, every two days, weekly, etc.
   - **When to install** — immediately, or notify only
   - **Which updates to show** — security only, or all
4. Close the window to save

> 💡 **If You Installed "Software & Updates"**
>
> As mentioned in Chapter 7, Ubuntu 26.04 doesn't include the "Software & Updates" tool by default. If you installed it (`sudo apt install software-properties-gtk`), these settings live there instead, with more options including mirror selection, developer options, and the "Additional Drivers" tab.

## Installing .deb Files from the Internet

Sometimes a developer distributes their application as a `.deb` file on their website rather than through the App Center. This is similar to downloading a `.exe` on Windows — you're responsible for trusting the source.

### Method 1: App Center (Recommended)

Ubuntu 26.04's App Center can install local `.deb` files:

1. Download the `.deb` file from the developer's website
2. **Double-click** the `.deb` file
3. App Center opens, showing package details
4. Click **"Install"**
5. Enter your password
6. Wait for installation to complete

> ⚠️ **"No App Installed for Debian Package Files" Error**
>
> Some users on Ubuntu 26.04 have reported seeing this error when double-clicking a `.deb` file. If this happens to you:
>
> 1. Right-click the `.deb` file → **Open With** → **App Center**
> 2. If App Center isn't listed, use Method 2 (GDebi) below
> 3. Or install from the terminal: `sudo apt install ./filename.deb`

### Method 2: GDebi (Fallback GUI)

**GDebi** is a lightweight, dedicated `.deb` installer that's been around for years. If App Center doesn't handle a `.deb` file, GDebi almost always will:

1. Install GDebi (one-time setup):

sudo apt install gdebi

2. **Right-click** any `.deb` file
3. Select **"Open With"** → **GDebi Package Installer**
4. Click **"Install Package"**
5. Enter your password

GDebi is simpler than App Center for `.deb` files — it shows dependencies, file sizes, and package descriptions without the overhead of the full software center.

### Method 3: Set GDebi as Default

If you frequently install `.deb` files from the web:

1. Right-click any `.deb` file
2. Select **"Properties"**
3. Go to the **"Open With"** tab
4. Select **GDebi Package Installer**
5. Click **"Set as Default"**

From now on, double-clicking any `.deb` file opens GDebi directly.

> ⚠️ **Security Warning: Trust Your Sources**
>
> When you install a `.deb` file from a website, you're bypassing the curated repository system. The package isn't verified by Canonical. Only install `.deb` files from:
>
> - The official website of the software developer
> - Reputable project pages (GitHub releases, official mirrors)
> - Sources you have reason to trust
>
> Never install `.deb` files from random forums, unverified blogs, or file-sharing sites.

## Flatpak and Flathub

If you followed the post-install checklist in Chapter 7, you've already installed Flatpak and enabled Flathub. Flatpak gives you access to a large catalog of applications that may not be available through the App Center.

### Browsing Flathub in Your Browser

Visit **[flathub.org](https://flathub.org)** to browse available applications. Each app has a page with:

- Description and screenshots
- "Install" button (which downloads a `.flatpakref` file)
- Version history and changelog
- License and developer information

### Installing Flatpak Apps

**Method 1: From the Website**

1. Find an app on [flathub.org](https://flathub.org)
2. Click **"Install"**
3. A `.flatpakref` file downloads
4. Double-click the `.flatpakref` file
5. If you installed the GNOME Software Flatpak plugin, it opens and installs the app
6. If not, use the terminal (Chapter 18)

**Method 2: Through GNOME Software / App Center**

If you installed `gnome-software-plugin-flatpak` (from Chapter 7's checklist), Flatpak apps from Flathub appear directly in the App Center alongside Snap and .deb packages. Search for an app, and if it's on Flathub, you'll see it as an install option.

### Flatpak Permissions

Like Snaps, Flatpaks run sandboxed. Some apps request additional permissions (filesystem access, microphone, etc.). When you install a Flatpak, you may see a permissions dialog asking you to approve access.

You can review and modify Flatpak permissions using a tool called **Flatseal**:

1. Install Flatseal from Flathub:

flatpak install flathub com.github.tchx84.Flatseal

2. Open Flatseal
3. Select an app from the list
4. Toggle permissions on/off per app
5. Changes apply immediately

This is useful if an app can't access files it needs, or if you want to revoke permissions from an app you don't fully trust.

### Updating Flatpak Apps

Flatpak apps update through the normal update process if you installed the GNOME Software plugin. You can also update them manually:

1. Open a terminal
2. Run: `flatpak update`
3. Approve the updates

We'll cover this in detail in Chapter 18.

## AppImage Files (Brief Mention)

A fourth software format you might encounter is **AppImage**. These are single-file executables — you download one file, make it executable, and run it. No installation required.

AppImages are less common than Snaps or Flatpaks, but some developers distribute their apps this way. We cover them in detail in Chapter 18.

## Managing Software Sources

Ubuntu pulls software from several **repositories** (software sources). Understanding these helps you know where your software comes from:

### The Four Main Repositories

| Repository | What It Contains | Enabled by Default? |
|---|---|---|
| **Main** | Official Ubuntu software, free and open source, fully supported by Canonical | Yes |
| **Universe** | Community-maintained software, free and open source, not officially supported | Yes |
| **Restricted** | Software with licensing restrictions (proprietary drivers, some codecs) | Yes |
| **Multiverse** | Software with more restrictive licensing (patent encumbrances, etc.) | No (opt-in) |

Most users never need to touch these settings. They're configured automatically during installation. But if you ever can't find a package that should exist, check that all four repositories are enabled.

### Enabling Additional Repositories

If you installed "Software & Updates" (Chapter 7):

1. Open **Software & Updates**
2. Under the **Ubuntu Software** tab, check the boxes for:
- Main
- Universe
- Restricted
- Multiverse (only if you need it)
3. Close and allow the package list to refresh

Through the terminal (for reference — covered in detail in Chapter 18):

```bash
sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo apt update

Third-Party Repositories (PPAs)

PPAs (Personal Package Archives) are third-party repositories hosted on Canonical's Launchpad platform. They allow developers to distribute software outside the main Ubuntu repositories.

You may encounter PPAs when following installation instructions for specific software. They're added via terminal, which we cover in Chapter 18. For now, just know:

    PPAs are a way to get software not in the default repositories
    They're generally safe if they come from a trusted developer or organization
    They can occasionally cause dependency conflicts with system packages
    Always prefer Flatpak, Snap, or the official repositories when available

Practical Examples: Installing Common Applications

Let's walk through installing several popular applications to cement your understanding:
Example 1: Install VLC Media Player (from App Center)

    Open App Center
    Search for "VLC"
    Click the VLC result
    Review the description and screenshots
    Click "Install"
    Enter your password
    Wait for installation
    Press Super, type "VLC," launch it

Example 2: Install Spotify (Snap)

    Open App Center
    Search for "Spotify"
    Click the Spotify result (Snap badge visible)
    Click "Install"
    Enter your password
    Wait — Spotify downloads as a Snap package
    Launch from the app grid

Example 3: Install OBS Studio (from Flathub)

Assuming Flatpak is set up:

    Visit flathub.org in your browser
    Search for "OBS Studio"
    Click "Install"
    The .flatpakref file downloads
    Double-click the .flatpakref file
    The GNOME Software/App Center opens and installs it
    Launch from the app grid

Example 4: Install Google Chrome (.deb from website)

    Visit google.com/chrome in Firefox
    Click "Download Chrome"
    Select the ".deb (For Debian/Ubuntu)" option
    The .deb file downloads to your Downloads folder
    Double-click the .deb file
    App Center (or GDebi) opens
    Click "Install"
    Enter your password
    Wait for installation
    Chrome appears in your app grid

    ⚠️ Google Chrome Adds a Repository

    When you install Chrome from a .deb file, it automatically adds a Google repository to your software sources. This means future Chrome updates will come through Software Updater automatically — you won't need to manually re-download the .deb file. This is standard behavior for many .deb-distributed applications.

Common Problems and Solutions
"I Can't Find an App in the App Center"

If an application isn't showing up:

    Check Flathub — The app may be available as a Flatpak but not as a Snap
    Check the developer's website — They may distribute a .deb file or AppImage directly
    Try the terminal — Some packages are in the repositories but not surfaced in the App Center. sudo apt install packagename may find it
    Enable additional repositories — Universe and Multiverse may need enabling (see above)

"App Center Shows Blank Screen or Won't Load"

This is a known issue with the Snap-based App Center:

    Force restart it:

    sudo snap refresh snap-store

    If that fails, kill and relaunch:

    sudo killall snap-store

    If persistent, reinstall:

    sudo snap remove snap-store && sudo snap install snap-store

    As a last resort, install the classic GNOME Software as an alternative:

    sudo apt install gnome-software

"Installed App Doesn't Appear in the App Grid"

    Press Super and type the app name directly — it may just not be in the grid categorization
    Log out and log back in — some app registrations need a session restart
    For Snap apps: check if the snap installed correctly: snap list
    For Flatpak apps: run flatpak list to verify installation

"Snap App Is Slow to Start"

Snap applications can have slower cold-start times (first launch after boot) because they mount their contained filesystem. This improves on subsequent launches. If it's consistently slow:

    Check if a Flatpak version exists on Flathub — often faster
    Check if a .deb version exists in the App Center
    Disable unnecessary Snap services you don't use

"Disk Space Running Low"

Snap packages retain old versions after updates, consuming disk space. To clean up:

    Open a terminal
    List all Snap versions: snap list --all
    Remove disabled (old) versions: sudo snap remove --purge packagename --revision=old_revision_number
    Or remove all disabled versions at once: LANG=C snap list --all | awk '/disabled/{print $1, $3}' | xargs -rn2 sudo snap remove --purge

We'll cover this in more detail in Chapter 16 (System Monitoring and Maintenance).
What's Next

You now know how to install, update, and remove software graphically on Ubuntu 26.04 — through the App Center for Snaps and .deb packages, through Flathub for Flatpaks, and through direct .deb file installation for developer-distributed software.

In Chapter 11, we'll tackle multimedia codecs — ensuring your system can play every common audio and video format without frustration.
Try It Yourself

    Install an app from the App Center — Try VLC Media Player. Browse the detail page, check permissions, install it, and play a video file.

    Install an app from Flathub — If you set up Flatpak in Chapter 7, install an app from Flathub that isn't in the App Center (like OBS Studio or Blender).

    Install a .deb from a website — Download Google Chrome or another app that distributes a .deb file. Install it using App Center or GDebi.

    Review your installed apps — Open App Center → Installed tab. Remove any pre-installed applications you know you won't use.

    Check for updates — Open Software Updater. Apply any pending updates.
