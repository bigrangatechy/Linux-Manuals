# Chapter 10: Software Management Through GNOME Software

## The App Store Concept — Linux Style

On Windows, you install software by downloading `.exe` or `.msi` files from websites, running them, and clicking "Next" through a wizard. On macOS, you drag `.dmg` files into Applications. Both approaches have a fundamental problem: the operating system doesn't know what software you have, where it came from, or whether it's safe.

Linux takes a different approach. Software is distributed through **repositories** — curated, signed, centrally-managed servers that your system trusts. You install software through a package manager that tracks every file, dependency, and version. When you uninstall, the system cleans up properly. When you update, the package manager handles everything in one operation.

**GNOME Software** is the graphical front-end to this system. It's your app store — a place to browse, search, install, update, and remove applications. But unlike the Apple App Store or Microsoft Store, GNOME Software draws from multiple independent repositories simultaneously: Fedora's official RPM repos, RPM Fusion, Fedora's Flatpak repo, and Flathub.

This chapter covers everything you can do through GNOME Software's graphical interface. Chapter 18 covers the same operations from the command line with DNF5 and Flatpak directly.

---

## What GNOME Software Actually Is

GNOME Software (the application is named `gnome-software`) is a **front-end** — a graphical interface that sits on top of two different package systems:

┌─────────────────────────────────────────────────┐ │ GNOME Software (GUI) │ │ Browse, search, install, update, remove │ ├──────────────────────┬──────────────────────────┤ │ PackageKit │ Flatpak (libflatpak) │ │ (RPM front-end) │ (Flatpak front-end) │ ├──────────────────────┼──────────────────────────┤ │ DNF5 + RPM repos │ Flatpak + OSTree repos │ │ (Fedora, RPM │ (Fedora Flatpak, │ │ Fusion) │ Flathub) │ └──────────────────────┴──────────────────────────┘

When you click "Install" in GNOME Software, it determines which package format and repository to use, then delegates to the appropriate backend. You don't need to know which backend is running — but understanding the two formats helps you make informed choices.
The Two Package Formats
Format	Source	Installed Where	Sandbox?	Updates Via
RPM	Fedora repos, RPM Fusion	System directories (/usr, /opt)	No — full system access	DNF5 (through GNOME Software)
Flatpak	Flathub, Fedora Flatpak	~/.local/share/flatpak (user) or /var/lib/flatpak (system)	Yes — isolated in a sandbox	Flatpak (through GNOME Software)

RPM packages are the traditional Fedora format. They integrate deeply with the system — shared libraries, system services, file associations. They're maintained by Fedora packagers and signed with Fedora's GPG key (or RPM Fusion's key).

Flatpak packages are containerized applications. Each Flatpak bundles its own dependencies, runs in a restricted sandbox, and can be updated independently of the system. They're maintained by application developers or the Flathub community.

    💡 Which Format Should You Use?

    For most desktop applications, prefer Flatpak when both formats are available. Flatpaks are:

        More up-to-date (developers publish directly to Flathub)
        Sandboxed (limited system access — more secure)
        Independent of system updates (won't break when you upgrade Fedora)
        Larger on disk (each bundles its own dependencies), but this is rarely a concern on modern hardware

    For system-level software (drivers, command-line tools, libraries, fonts), use RPM. Flatpaks aren't suited for software that needs to integrate with the kernel or system services.

Opening GNOME Software

Open GNOME Software from the dash (if pinned) or the app grid. Or press Super, type "software," and press Enter.

┌──────────────────────────────────────────────────────────┐
│  Software                                          🔍    │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │  Editor's │  │  Editor's│  │   New &  │              │
│  │  Choice   │  │  Choice  │  │  Updated │              │
│  │           │  │           │  │           │              │
│  │  [image]  │  │  [image]  │  │  [image]  │              │
│  └──────────┘  └──────────┘  └──────────┘              │
│                                                          │
│  ─── Categories ────────────────────────────────────     │
│                                                          │
│  🎨 Creative   💼 Productivity   🎮 Games               │
│  💬 Social     🔧 Developer      📚 Education            │
│                                                          │
│  ─── Recently Updated ──────────────────────────────     │
│                                                          │
│  [app card]  [app card]  [app card]  [app card]         │
│                                                          │
└──────────────────────────────────────────────────────────┘

GNOME Software opens to the Explore tab, which shows curated content: editor's picks, new and updated apps, and categorized browsing.
The Three Tabs

GNOME Software has three main tabs at the top:
Tab	Purpose
Explore	Browse featured apps, categories, editor's picks. Discovery mode.
Installed	List of everything installed on your system. Update, remove, manage addons.
Updates	Available updates for installed software and the system itself.
Browsing and Searching
Search

Click the 🔍 icon in the top-right corner (or press Ctrl+F) and type your query:

┌──────────────────────────────────────────┐
│  🔍 spotify                              │
├──────────────────────────────────────────┤
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  📦 Spotify                         │ │
│  │  Music streaming                    │ │
│  │  Flathub · Flatpak                  │ │
│  │  ★★★★☆  4.2  (1,847 reviews)      │ │
│  │                          [Install]  │ │
│  └────────────────────────────────────┘ │
│                                          │
│  ┌────────────────────────────────────┐ │
│  │  📦 Spot (TUI Spotify client)       │ │
│  │  Terminal-based Spotify client      │ │
│  │  Fedora · RPM                       │ │
│  │                          [Install]  │ │
│  └────────────────────────────────────┘ │
│                                          │
└──────────────────────────────────────────┘

Search results show:

    Application name and icon
    Short description
    Source — "Flathub · Flatpak" or "Fedora · RPM" (this tells you the format and repository)
    Rating and review count (for Flathub apps)
    An Install button

    💡 Multiple Sources for the Same App

    Some applications are available in both RPM and Flatpak formats. GNOME Software shows each as a separate result. Look at the "Source" label to tell them apart. When in doubt, choose the Flatpak version — it'll be more up-to-date and sandboxed.

    If you don't see the source label, click the app card to open its detail page, which shows the full source information.

Categories

Browse by category from the Explore tab:
Category	Examples
Creative	GIMP, Inkscape, Krita, Blender, OBS Studio
Productivity	LibreOffice, Thunderbird, Todoist, MarkText
Games	Steam, Heroic, Lutris, RetroArch
Social	Discord, Telegram, Element, Signal
Developer	VS Code, GNOME Builder, DBeaver, Postman
Education	Stellarium, GCompris, Anki, Scratch
Utilities	BleachBit, Bottles, Extension Manager
Audio & Video	VLC, Audacity, Kdenlive, HandBrake

Click a category to see a paginated grid of applications in that category.
Editorial Curation

GNOME Software's Explore tab features curated selections that go beyond simple categories:
Section	What It Shows
Editor's Choice	High-quality apps handpicked by GNOME Software editors
New & Updated	Recently added or recently updated applications
Made for GNOME	GNOME Circle apps — built with GTK 4, following GNOME design guidelines
Popular	Most-installed applications across all users

These curated lists help you discover apps you might not know to search for.
Installing Applications
The Installation Flow

    Find the app — search or browse
    Click Install — the button changes to a progress bar
    Authenticate — enter your password (or use Touch ID if configured)
    Wait — GNOME Software downloads and installs the package
    Launch — the button changes to "Open," or find the app in your app grid

┌──────────────────────────────────────────┐
│  Discord                                  │
├──────────────────────────────────────────┤
│                                          │
│         [████████████░░░░] 68%           │
│         Downloading...                    │
│                                          │
└──────────────────────────────────────────┘

For RPM packages, GNOME Software calls PackageKit, which calls DNF5 under the hood. DNF5 resolves dependencies, downloads all required packages, and installs them.

For Flatpak packages, GNOME Software calls libflatpak, which downloads the Flatpak and its runtime (a shared base layer — see below) and installs them.
Flatpak Runtimes

When you install your first Flatpak application, you may notice the download is larger than expected. This is because Flatpaks require a runtime — a shared base layer providing common libraries (GTK, GLib, multimedia frameworks, etc.).
Runtime	Used By	Size
GNOME 50	GNOME-based Flatpaks (Files, Calendar, etc.)	~600 MB
KDE 6.7	KDE-based Flatpaks (Krita, Kdenlive)	~500 MB
Freedesktop 24.08	General-purpose Flatpaks	~400 MB
Freedesktop 25.08	Newer general-purpose Flatpaks	~400 MB

The runtime is installed once and shared by all Flatpaks that depend on it. So your first Flatpak install downloads the app plus the runtime; subsequent installs that use the same runtime download only the app itself.

    💡 "Why Did It Download 500 MB for a Chat App?"

    If you install Discord as your first Flatpak, GNOME Software downloads:

        The Freedesktop runtime (~400 MB)
        The Discord app itself (~150 MB)
        Total: ~550 MB

    If you then install Spotify (also Freedesktop runtime), you only download the Spotify app (~170 MB) — the runtime is already present.

    This is normal and expected. The runtime overhead pays off as you install more Flatpaks.

Installation Permissions

When you install a Flatpak, GNOME Software may show a permissions dialog:

Discord requires access to:

  📁 Your file system (limited)
  📷 Camera
  🎤 Microphone
  🔔 Notifications
  🌐 Network

  [Cancel]  [Install Anyway]

These are the sandbox permissions the app is requesting. Unlike RPM packages (which have full system access), Flatpaks must declare what they need, and you can see (and revoke) these permissions.

You can change Flatpak permissions after installation using Flatseal (available on Flathub):

flatpak install flathub com.github.tchx84.Flatseal

Flatseal provides a GUI to toggle individual permissions for each installed Flatpak — disable camera access for an app that shouldn't have it, grant file system access to an app that needs it, etc.

    💡 Sandboxing Is a Spectrum

    Not all Flatpaks are equally sandboxed:

        Well-sandboxed: Apps that use portals (a secure inter-process communication system) for file access — they can only access files you explicitly open through a file picker dialog
        Loosely sandboxed: Apps that request broad file system access (filesystem=host) — these can read any file in your home directory

    Flatseal lets you see which category each app falls into and tighten permissions if desired.

Application Detail Pages

Click any application in GNOME Software to see its detail page:

┌──────────────────────────────────────────────────────┐
│  ←  Spotify                                           │
│                                                      │
│            ┌──────┐                                  │
│            │  📦   │   Spotify                        │
│            │      │   Music for everyone              │
│            └──────┘   Flathub · Flatpak              │
│                       Version 1.2.45                  │
│                       Updated: 2 days ago             │
│                                                      │
│            ★★★★☆ 4.2  (1,847 reviews)               │
│                                                      │
│            [    Install    ]                          │
│                                                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │                                                │ │
│  │              [Screenshot 1]                    │ │
│  │                                                │ │
│  └────────────────────────────────────────────────┘ │
│  ┌──────────────┐  ┌──────────────┐               │
│  │ [Screenshot 2]│  │ [Screenshot 3]│               │
│  └──────────────┘  └──────────────┘               │
│                                                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  About this app                                      │
│                                                      │
│  Spotify is a digital music service that gives you  │
│  access to millions of songs...                      │
│                                                      │
│  This app is developed in the open.                  │
│                                                      │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Category: Audio & Video                             │
│  Developer: Spotify AB                               │
│  License: Proprietary                                │
│  Size: 172 MB                                        │
│                                                      │
└──────────────────────────────────────────────────────┘

The detail page shows:
Section	Information
Icon and title	App identity
Source	Flathub, Fedora, RPM Fusion — tells you the repository
Version	Current package version number
Updated	When the package was last updated in the repository
Rating	Average rating and review count (Flathub apps only)
Install button	Installs the app
Screenshots	Preview images showing the app in use
Description	What the app does, written by the developer
Metadata	Category, developer name, license, download size
Reviews

Flathub apps in GNOME Software show user ratings and reviews. You can write your own review by clicking the star rating on an app's detail page (you'll need a Flathub account or a GNOME account, depending on the review system).

Reviews help with discovery — apps with higher ratings appear more prominently in search and category listings.
The Installed Tab

Switch to the Installed tab to see everything on your system:

┌──────────────────────────────────────────────────────────┐
│  Installed                                           🔍   │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────────┐│
│  │  🦊 Firefox                          RPM      ⋮      ││
│  │  138.0 · Fedora                                      ││
│  │                                     [Remove]         ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  📦 Spotify                           Flatpak   ⋮    ││
│  │  1.2.45 · Flathub                                    ││
│  │                                     [Remove]         ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  📝 LibreOffice                       RPM      ⋮    ││
│  │  25.2 · Fedora                                       ││
│  │                                  [▼ Add-ons]         ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  🎬 Totem                             RPM      ⋮    ││
│  │  44.0 · Fedora                                        ││
│  │                                     [Remove]         ││
│  └──────────────────────────────────────────────────────┘│
│                                                          │
└──────────────────────────────────────────────────────────┘

Each installed app shows:

    Name and icon
    Version and source (Fedora/RPM, Flathub/Flatpak)
    Remove button
    ⋮ menu (additional actions — see below)
    Add-ons button (if the app has optional components)

The ⋮ Menu

Click the three-dot menu next to an installed app for additional options:
Option	What It Does
Details	Opens the app's detail page
Permissions	(Flatpak only) Opens permission settings
Uninstall	Same as Remove — deletes the package
Shortcuts	Creates a desktop shortcut (limited support)
Add-Ons

Some RPM packages have add-ons — optional components that extend the base package. Click Add-ons to see them:

LibreOffice Add-ons

  ☑ LibreOffice Writer     Installed
  ☑ LibreOffice Calc       Installed
  ☑ LibreOffice Impress    Installed
  ☐ LibreOffice Draw       [Install]
  ☐ LibreOffice Base      [Install]
  ☐ LibreOffice Math      [Install]
  ☑ libreoffice-langpack-en  Installed
  ☐ libreoffice-langpack-es  [Install]

Add-ons are sub-packages — individual RPM components that share the base package's name. This modular approach lets you install only the parts you need.

    💡 Why Some Apps Have Add-Ons and Others Don't

    Add-ons exist when a package is split into multiple RPM sub-packages. This happens when:

        The app is large and users may not need every component (LibreOffice)
        Optional features require additional dependencies (codecs, language packs)
        Plugins or extensions are packaged separately

    Flatpak applications don't have add-ons in GNOME Software — the Flatpak format bundles everything together.

Removing Applications

To uninstall an application:

    Go to the Installed tab
    Find the app
    Click Remove
    Authenticate with your password
    Confirm if prompted

For RPM packages, DNF5 removes the package and any dependencies that were installed exclusively for it and are no longer needed.

For Flatpak packages, the Flatpak is removed. However, the runtime it depended on is NOT removed — other Flatpaks may still need it. Unused runtimes accumulate over time. Clean them up with:

flatpak uninstall --unused

This removes runtimes that no installed Flatpak depends on, freeing disk space.

    ⚠️ Removing vs. Purging

    GNOME Software removes the application's files but may leave user configuration in your home directory (~/.config/appname, ~/.local/share/appname). This is by design — if you reinstall the app later, your settings are preserved.

    To remove configuration files too (a "purge"):

    # For RPM packages:
    sudo dnf remove --noautoerase appname
    rm -rf ~/.config/appname ~/.local/share/appname

    # For Flatpak packages:
    flatpak uninstall --delete-data appname

    Be careful with --delete-data — it permanently removes your app data and settings.

The Updates Tab

GNOME Software's Updates tab shows available updates for:

    Installed RPM packages (from Fedora repos and RPM Fusion)
    Installed Flatpak packages (from Flathub and Fedora Flatpak)
    System updates (kernel, systemd, base libraries)
    Firmware updates (via fwupd)

┌──────────────────────────────────────────────────────────┐
│  Updates                                                 │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  System Updates Available                                │
│                                                          │
│  23 packages can be updated                              │
│  Including: kernel, firefox, gnome-shell                │
│                                                          │
│  [    Restart & Update    ]                              │
│                                                          │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  Application Updates                                     │
│                                                          │
│  ┌──────────────────────────────────────────────────────┐│
│  │  📦 Spotify               1.2.45 → 1.2.48    [Update]││
│  │  Flathub · Flatpak                                  ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  🦊 Firefox               138.0 → 139.0     [Update]││
│  │  Fedora · RPM                                       ││
│  └──────────────────────────────────────────────────────┘│
│  ┌──────────────────────────────────────────────────────┐│
│  │  💾 Firmware               Version 1.2.3     [Update]││
│  │  System Device (SSD)                                ││
│  └──────────────────────────────────────────────────────┘│
│                                                          │
│  [    Update All    ]                                    │
│                                                          │
└──────────────────────────────────────────────────────────┘

System Updates vs. Application Updates

GNOME Software distinguishes between:
Type	Examples	Requires Restart?
System updates	Kernel, glibc, systemd, gnome-shell	Yes — a restart applies the new kernel and core libraries
Application updates	Firefox, Spotify, GIMP	Usually no — the app restarts with the new version
Firmware updates	BIOS/UEFI, SSD firmware	Often yes — depends on the device

When system updates include a new kernel, GNOME Software shows "Restart & Update" instead of "Update." Clicking it applies the updates and immediately reboots the system. This ensures the new kernel loads cleanly.
Update All

Click Update All to install all available updates in one operation. GNOME Software handles RPM and Flatpak updates together, showing a single progress bar.

    💡 How Often Should You Update?

    Fedora pushes updates continuously — security patches land within hours of being reported upstream. There's no monthly "Patch Tuesday."

    A good cadence:

        Daily: Let GNOME Software's automatic background check notify you. Apply application updates when convenient.
        Weekly: Run system updates (including kernel) when you can reboot without disruption.
        Before installing new software: Always update first to ensure dependency versions are current.

    GNOME Software checks for updates automatically every 6 hours when you're online. You'll see a notification when updates are available.

Automatic Updates

GNOME Software can automatically download (but not install) updates in the background:

    Open GNOME Software
    Click the menu (☰, top-right) → Software Update Preferences
    Or go to Settings → Software

Options:
Setting	Recommendation
Automatic Updates	Enable. Downloads updates in the background.
Notify when updates are available	Enable. You choose when to install.

Automatic updates download packages but don't install them until you click "Update All" or "Restart & Update." This is safer than fully automatic installation — you retain control over when changes are applied to your system.
Understanding Sources: Where Your Software Comes From

GNOME Software pulls from multiple repositories. Here's a complete picture of what's available after the post-install configuration from Chapter 7:
RPM Repositories
Repository	Contents	Enabled
fedora	Base packages from the Fedora 44 release ISO	Yes
updates	Security and bug-fix updates since release	Yes
rpmfusion-free	Free software excluded from Fedora (codecs, ffmpeg)	Yes (if enabled in Ch.7)
rpmfusion-free-updates	Updates for RPM Fusion free packages	Yes
rpmfusion-nonfree	Non-free software (NVIDIA drivers, some media tools)	Yes (if enabled in Ch.7)
rpmfusion-nonfree-updates	Updates for RPM Fusion nonfree packages	Yes
fedora-cisco-openh264	Cisco's open H.264 codec for Firefox	Yes (if enabled in Ch.7)
rawhide	Fedora development branch	No — don't enable unless you know what you're doing
Flatpak Repositories
Repository	Contents	Enabled
fedora	Fedora's curated Flatpak subset	Yes (by default)
flathub	Community Flatpak repository — thousands of apps	Yes (if enabled in Ch.7)
Checking Your Repositories

You can see which repositories GNOME Software is pulling from:

# RPM repositories:
dnf repolist

# Flatpak repositories:
flatpak remotes

    ⚠️ The Rawhide Warning

    If you ever see "Rawhide" in your repository list and you didn't intentionally enable it, disable it immediately:

    sudo dnf config-manager --set-disabled rawhide

    Rawhide is Fedora's rolling development branch — packages there are bleeding-edge and may be broken. Running a normal Fedora 44 system with Rawhide enabled will cause severe package conflicts.

Managing Application Sources in GNOME Software

GNOME Software has a "Repositories" or "Sources" settings page where you can enable or disable repositories:

    Open GNOME Software
    Click the ☰ menu (top-right) → Software Repositories (or "Settings")

You'll see a list of all configured repositories, grouped by type:

┌──────────────────────────────────────────────────────┐
│  Software Repositories                                │
├──────────────────────────────────────────────────────┤
│                                                      │
│  RPM Repositories                                    │
│  ☑ Fedora 44 - Base                                  │
│  ☑ Fedora 44 - Updates                               │
│  ☑ RPM Fusion - Free                                 │
│  ☑ RPM Fusion - Free - Updates                       │
│  ☑ RPM Fusion - Nonfree                              │
│  ☑ RPM Fusion - Nonfree - Updates                    │
│  ☑ Cisco OpenH264                                    │
│                                                      │
│  Flatpak Repositories                                │
│  ☑ Fedora                                            │
│  ☑ Flathub                                           │
│                                                      │
└──────────────────────────────────────────────────────┘

Toggle a repository off to stop GNOME Software from showing apps from that source. This can be useful if:

    You want to see only Flatpak apps (disable all RPM repos in the filter)
    You want to exclude nonfree software (disable RPM Fusion nonfree)
    You're troubleshooting and want to isolate a problematic repository

    💡 Disabling vs. Removing

    Toggling a repository off in GNOME Software disables it temporarily — the repo configuration stays on your system, but packages from it won't be shown or installed. You can re-enable it anytime.

    To permanently remove a repository, use the command line:

    sudo dnf config-manager --set-disabled reponame

    Or for Flatpak:

    flatpak remote-delete flathub

Installing Third-Party Repositories

Some software isn't available in Fedora, RPM Fusion, or Flathub. Third-party repositories can be added:
Adding RPM Repositories

Some software vendors provide their own RPM repositories. For example, the Google Chrome repository:

sudo dnf install https://dl.google.com/linux/direct/google-chrome-stable_current_x86_64.rpm

This installs Chrome and automatically adds the Google repository for future updates. After installation, Chrome appears in GNOME Software's Installed tab and receives updates alongside everything else.
Adding COPR Repositories

COPR (Community Projects) is Fedora's community repository system — similar to a PPA on Ubuntu. COPR repos provide packages not in the official Fedora repos, maintained by community members.

# Enable a COPR repository (example):
sudo dnf copr enable username/projectname

    ⚠️ COPR Packages Are Untrusted

    COPR repositories are community-maintained and not reviewed by Fedora. They may contain buggy, outdated, or even malicious software. Only enable COPR repositories from maintainers you trust.

    COPR is not the same as Flathub. Flathub apps are sandboxed Flatpaks; COPR packages are RPMs with full system access. The security model is very different.

Adding Flathub as the Primary Flatpak Source

If you haven't already enabled Flathub (from Chapter 7):

flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo

After enabling, restart GNOME Software to pick up the new repository.
Common Scenarios
Scenario 1: "I Want to Install Spotify"

    Open GNOME Software
    Search "spotify"
    Click the Spotify result (source: Flathub · Flatpak)
    Click Install
    Enter your password
    Wait for download and installation
    Press Super, type "spotify," press Enter

Scenario 2: "I Want to Install a Game"

    Open GNOME Software
    Click Categories → Games
    Browse or search for a specific game (e.g., "steam")
    Click the game's card
    Click Install
    For Steam (Flatpak), it installs to your home directory — no sudo needed for game installations within Steam itself

Scenario 3: "I Want to Install VLC Two Ways"

VLC is available as both an RPM (from RPM Fusion) and a Flatpak (from Flathub). GNOME Software may show both:
Result	Source	Considerations
VLC	Flathub · Flatpak	Sandboxed, self-contained, always latest version. Larger download.
VLC	RPM Fusion · RPM	Integrates with system libraries. Smaller download. May lag behind upstream.

Choose based on preference. If you're unsure, pick the Flatpak version.

    💡 Having Both Isn't a Problem

    You can technically install both the RPM and Flatpak versions of the same application. They install to different locations and don't conflict. However, you'll see two entries in your app grid, which is confusing. Pick one and stick with it.

Scenario 4: "An App Won't Install"

If clicking Install does nothing or shows an error:
Possible Cause	Fix
Repository not enabled	Check Software Repositories in the ☰ menu
RPM Fusion not installed	Follow Chapter 7 Task 3 to enable RPM Fusion
Flathub not enabled	Follow Chapter 7 Task 4 to enable Flathub
Package conflict	Open a terminal, try sudo dnf install packagename to see the detailed error
Network issue	Check your internet connection; try sudo dnf clean all to clear cache
Disk full	Check disk space with df -h — you need at least 2× the package size free
GNOME Software and DNF5: The Relationship

GNOME Software and the terminal's dnf command are two front-ends to the same RPM package system. They're not separate worlds:
Action	GNOME Software	Terminal
Install RPM	Click "Install"	sudo dnf install firefox
Remove RPM	Click "Remove"	sudo dnf remove firefox
Update RPM	Click "Update All"	sudo dnf upgrade
Search RPM	Type in search bar	dnf search firefox
List installed	"Installed" tab	dnf list --installed

They share the same package database. If you install a package via terminal with dnf, it appears in GNOME Software's Installed tab. If you remove an app through GNOME Software, it's gone from the DNF database too.

Similarly, Flatpak operations in GNOME Software are reflected in flatpak list and vice versa.

    💡 When to Use Which

    Use GNOME Software for:

        Browsing and discovering applications
        Reading app descriptions and screenshots
        Managing Flatpak applications
        Quick updates with a visual progress indicator

    Use the terminal (DNF5/Flatpak) for:

        Precise control over what's installed and removed
        Installing packages that aren't in GNOME Software's search results
        Batch operations (install 20 packages at once)
        Troubleshooting (error messages are more detailed)
        Scripting and automation
        Seeing dependency trees and package metadata

    Chapter 18 covers the terminal-based package management in detail.

GNOME Software Limitations

GNOME Software is excellent for everyday use, but it has limitations compared to the command-line tools:
Limitation	Explanation
No dependency visualization	You can't see what else a package needs before installing it. dnf install --assumeno shows the full dependency tree.
No package groups	Fedora's package groups (like "Development Tools" or "Multimedia") aren't exposed in GNOME Software. Use dnf groupinstall in the terminal.
No downgrade option	You can't install an older version of a package through GNOME Software. Use dnf downgrade packagename in the terminal.
No repository priority control	You can't set which repository wins when multiple provide the same package. Use dnf config-manager in the terminal.
Slow with large update sets	GNOME Software can be sluggish when hundreds of packages need updating. dnf upgrade in the terminal is faster.
Limited package information	GNOME Software shows basic metadata. dnf info packagename shows changelogs, build time, signature, and more.
No transaction history	You can't see a log of what was installed/removed when. dnf history shows a full audit trail.

None of these are showstoppers. For daily use, GNOME Software is sufficient and more pleasant to use than the terminal. But when you need control or information, the command-line tools give you everything.
Keeping Your System Healthy
Regular Maintenance Through GNOME Software
Task	Frequency	How
Check for updates	Weekly	Updates tab → Update All
Remove unused apps	Monthly	Installed tab → Remove apps you no longer use
Clean up Flatpak runtimes	Monthly	Terminal: flatpak uninstall --unused
Check firmware	Monthly	Updates tab → look for firmware entries
Review installed apps	Quarterly	Installed tab → audit what's on your system
Disk Space Awareness

Applications consume disk space, and old Flatpak runtimes can accumulate. Check your disk usage periodically:

# Overall disk usage:
df -h

# Flatpak-specific disk usage:
du -sh ~/.local/share/flatpak
du -sh /var/lib/flatpak

# Remove unused Flatpak runtimes and save space:
flatpak uninstall --unused

# Clean DNF cache:
sudo dnf clean all

    💡 The dnf clean all Command

    DNF5 caches downloaded packages and repository metadata in /var/cache/dnf. Over time, this cache can grow to several gigabytes. Running sudo dnf clean all clears it. You won't lose any installed software — only cached download files.

What's Next

You now understand how to install, update, and remove software through GNOME Software — Fedora's graphical app store. You know the difference between RPM and Flatpak packages, how Flatpak runtimes work, how to browse and search, how to manage add-ons and permissions, and how to maintain your system through regular updates.

In Chapter 11, we'll tackle multimedia and codecs — the reason you enabled RPM Fusion back in Chapter 7. We'll cover what codecs are, why Fedora doesn't ship them by default, how to ensure every media format plays correctly, and how to configure hardware-accelerated video playback for smooth performance and longer battery life.
Try It Yourself

    Install your first Flatpak. Open GNOME Software, search for an app you need that isn't already installed (try Obsidian, Bitwarden, or Flatseal). Click Install. Notice how the download includes a runtime on your first Flatpak install.

    Install an RPM from GNOME Software. Search for a tool like "htop" or "neofetch." Install it. Now open a terminal and run htop — the same package you installed through the GUI is available in the terminal. They're the same system.

    Explore the Installed tab. Go through every installed application. Remove anything you don't recognize or don't need. A lean system is a fast system.

    Check for updates. Go to the Updates tab. If updates are available, click Update All. Note the difference between application updates (no restart needed) and system updates (restart required).

    Install Flatseal and review permissions. Install Flatseal from Flathub. Open it and select an installed Flatpak (like Spotify or Discord). Browse the permissions — see what file system access, device access, and network access each app has. Try toggling a permission off and see if the app still works.

    Clean up unused Flatpak runtimes. Open a terminal and run flatpak uninstall --unused. See how much disk space is reclaimed from old runtimes you no longer need.

    Compare the GUI and terminal. Install the same application two ways: first through GNOME Software, then (after removing it) through the terminal with flatpak install flathub <app-id> or sudo dnf install <package>. Notice that the result is identical — GNOME Software and the command line use the same underlying systems.

    ✅ Chapter 10 Complete You understand what GNOME Software is (a graphical front-end to DNF5/RPM and Flatpak), the difference between RPM and Flatpak packages (deep integration vs. sandboxing), how Flatpak runtimes work (shared base layers, installed once), how to browse and search for applications, how to install and remove apps (with authentication), what add-ons are (RPM sub-packages), the Updates tab (system vs. application vs. firmware updates, Restart & Update for kernel updates), automatic update behavior, all the repositories your software comes from (Fedora, RPM Fusion, Flathub, Fedora Flatpak, COPR), how to manage repository sources in GNOME Software, how to add third-party repositories, the relationship between GNOME Software and the terminal (same system, different interfaces), GNOME Software's limitations compared to the command line, and a regular maintenance routine to keep your system healthy and your disk clean.
