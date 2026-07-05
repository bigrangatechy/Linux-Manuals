# Chapter 4: Coming from Windows or macOS

## You Already Know More Than You Think

Switching operating systems feels like moving to a foreign country. The streets have different names. The currency is unfamiliar. The customs seem strange. But underneath the surface differences, a city is still a city — roads, buildings, shops, traffic lights. The fundamentals haven't changed.

The same is true of operating systems. Whether you're coming from Windows 11, Windows 10, or macOS, you already understand the core concepts: you have a desktop, a taskbar or dock, a file manager, a web browser, settings panels, and applications. Fedora has all of these. The pieces look different, and some are in unexpected places, but the architecture is recognizable.

This chapter is your translation guide. We'll map what you know to what Fedora offers, highlight what transfers smoothly, flag what doesn't, and prepare you for the handful of genuine paradigm shifts that catch newcomers off guard.

---

## The Mental Shift: What Changes and What Doesn't

### What Stays the Same

| Concept | Windows/macOS | Fedora |
|---|---|---|
| Desktop with icons and wallpaper | ✅ | ✅ (GNOME is minimal by default — wallpaper with a top bar) |
| File manager (Explorer/Finder) | ✅ | ✅ (GNOME Files, also called Nautilus) |
| Web browser | ✅ | ✅ (Firefox pre-installed; Chrome, Edge available) |
| Office suite | ✅ (Office/iWork) | ✅ (LibreOffice pre-installed) |
| Email client | ✅ (Outlook/Mail) | ✅ (Geary or Thunderbird — install from GNOME Software) |
| Media player | ✅ (Media Player/QuickTime) | ✅ (GNOME Videos/Totem; VLC available) |
| Settings/Control Panel | ✅ | ✅ (GNOME Settings) |
| App store | ✅ (Microsoft Store/App Store) | ✅ (GNOME Software) |
| Screenshots | ✅ (Snipping Tool/Cmd+Shift+3) | ✅ (Print Screen key) |
| Search | ✅ (Start menu/Spotlight) | ✅ (Activities overview — press Super key) |
| Multiple desktops/workspaces | ✅ (Task View/Mission Control) | ✅ (Built into GNOME, auto-managed) |
| Notifications | ✅ | ✅ (Notification banner at top of screen) |
| Bluetooth, Wi-Fi, printers | ✅ | ✅ (All in GNOME Settings) |

The everyday activities — browsing the web, writing documents, checking email, watching videos, managing files — are fundamentally the same. You'll spend your first day clicking around, finding where things live, and that's normal.

### What Changes

| Concept | Windows/macOS | Fedora | The Difference |
|---|---|---|---|
| Drive letters (C:, D:) | ✅ | ❌ | Linux uses a single filesystem tree starting at `/`. No drive letters. See Chapter 14. |
| Installers (.exe, .dmg, .pkg) | ✅ | ❌ | Fedora uses RPM packages, Flatpaks, and repositories. No double-click installers. |
| Registry | ✅ (Windows) | ❌ | Linux uses individual text configuration files. No central database. |
| Antivirus required | ✅ (Windows) | ❌ | Not necessary on Linux for typical desktop use. See Chapter 27. |
| Forced updates | ✅ (Windows) | ❌ | You choose when to update. Fedora never forces a restart. |
| Telemetry/tracking | ✅ (both) | ❌ | Fedora collects zero data about you. |
| Account login (Microsoft/Apple ID) | ✅ | ❌ | Your Fedora account is local. No cloud account required. |
| File paths with backslash | `C:\Users\name` | `/home/name` | Forward slashes, no drive letter. |
| Taskbar at bottom | ✅ | Top bar only (by default) | GNOME uses a top bar, not a bottom taskbar. |
| Start button | ✅ | Activities button (or Super key) | GNOME's overview replaces the Start menu. |
| Window minimize/maximize buttons | ✅ | Close only (by default) | GNOME 50 hides minimize/maximize by default. They're easily restored. |
| Software from websites | ✅ | ❌ (mostly) | You install software through GNOME Software or DNF, not by downloading from websites. |
| Defragmentation | ✅ (Windows) | ❌ | Fedora uses Btrfs by default, which doesn't fragment. |

### What's Genuinely New (No Equivalent Exists)

These concepts have no direct equivalent in Windows or macOS. They're the ones that require the most mental adjustment:

| Concept | What It Is | Where You'll Learn It |
|---|---|---|
| **Package manager (DNF)** | A centralized system for installing, updating, and removing software from trusted repositories | Chapter 18 |
| **The terminal** | A text interface that's actually powerful and日常 (everyday) — not a relic | Chapter 10, 13 |
| **Root vs. sudo** | The admin account model is different from Windows Administrator or macOS sudo | Chapter 17 |
| **SELinux** | A security system that confines what programs can do, beyond simple file permissions | Chapter 27 |
| **Filesystem hierarchy** | `/home`, `/etc`, `/var`, `/usr` — the standardized directory structure | Chapter 14 |
| **Repositories** | Trusted servers hosting software packages — you don't hunt for installers on the web | Chapter 18 |
| **Flatpak** | Sandboxed applications that run independently of the base system | Chapter 18 |
| **Toolbox** | Containerized development environments — a Fedora signature feature | Chapter 23 |

Don't worry if these are unfamiliar — we'll cover each one in depth when the time comes. For now, just be aware that they exist.

---

## Coming from Windows

### The Desktop Experience

**Windows** gives you a taskbar at the bottom, a Start button in the corner, desktop icons, a system tray, and windows with minimize/maximize/close buttons in the upper right.

**Fedora with GNOME 50** gives you a clean desktop with a top bar. The top bar contains, from left to right:

- **Activities** button (or just press the Super key — that's the Windows key on most keyboards)
- A clock/calendar in the center
- System indicators (Wi-Fi, battery, volume, power) on the right

There is no taskbar at the bottom. There is no Start button. There are no desktop icons by default. Instead, GNOME uses the **Activities Overview** — press the Super key, and your entire screen transforms into a searchable app launcher combined with a workspace switcher.

> 💡 **The Super Key**
>
> "Super" is the Linux term for the key that Windows calls the "Windows key" and macOS calls "Command." On most PC keyboards, it's the key with the Windows logo between Ctrl and Alt. We'll use "Super" throughout this manual.

The GNOME workflow is keyboard-driven and search-focused — closer to macOS Spotlight than to the Windows Start menu, but more integrated into the desktop flow.

### Mapping Windows Concepts to Fedora

| Windows | Fedora Equivalent | Notes |
|---|---|---|
| Start Menu | Activities Overview (Super key) | Type to search apps, files, and settings |
| Taskbar | Top bar + Dash (in Activities) | The Dash (favorites bar) appears in the Activities Overview |
| System Tray | System menu (top-right corner) | Click the cluster of icons in the top-right |
| File Explorer | GNOME Files (Nautilus) | Press Super, type "Files" |
| Control Panel | GNOME Settings | Press Super, type "Settings" |
| Task Manager | GNOME System Monitor / Resources | Press Super, type "System Monitor" or "Resources" |
| Command Prompt / PowerShell | Terminal | Press Super, type "Terminal" (or "Ptyxis" on Ubuntu — Fedora uses GNOME Terminal by default) |
| Snipping Tool | Screenshot tool | Press the Print Screen key |
| Recycle Bin | Trash | Visible in Files and on the desktop (if enabled) |
| C: drive | `/` (the root filesystem) | All storage is under one tree |
| `C:\Users\YourName` | `/home/yourname` | Your personal files and settings |
| `C:\Program Files` | `/usr/bin` and `/usr/lib` | Installed programs (managed by DNF) |
| Windows Update | GNOME Software / `dnf upgrade` | Manual or automatic — your choice |
| Microsoft Store | GNOME Software | Install RPM and Flatpak applications |
| Windows Defender | Not needed | See Chapter 27 for why |
| Driver installation | Mostly automatic | NVIDIA drivers via RPM Fusion (Chapter 22) |
| .exe installers | DNF packages / Flatpaks | Install through GNOME Software or terminal |
| Registry | Text config files in `/etc` and `~/.config` | Human-readable, individually editable |
| Disk Defragmenter | Not needed | Btrfs/ext4 don't fragment the way NTFS does |
| Safe Mode | Rescue mode / Recovery Console | Accessed from GRUB boot menu |

### File Paths

Windows uses backslashes and drive letters:

C:\Users\John\Documents\report.docx

Fedora uses forward slashes and a single root:

/home/john/Documents/report.docx

There is no C: drive. The main drive is simply /. If you plug in a USB drive, it doesn't become E: — it gets mounted at a path like /run/media/john/USBdrive/. You'll interact with it through Files (the file manager), which shows it in the sidebar just like Windows Explorer does.
File Extensions

Windows relies heavily on file extensions to determine what program opens a file: .docx opens Word, .xlsx opens Excel, .exe is a program.

Fedora (like all Linux systems) uses extensions as hints but also examines the file's content (MIME type) to determine what it is. A file without an extension still opens correctly if its contents are recognized. This is why you'll sometimes see files with no extension on Linux — the system knows what they are anyway.
Where's My Antivirus?

This is one of the most common questions from Windows users, so let's address it directly.

Windows needs antivirus software because it's the most-targeted operating system on earth. The economics of malware favor attacking the platform with the most users. Additionally, Windows historically allowed programs deep system access, and its default configuration prioritized convenience over isolation.

Fedora doesn't need antivirus for several reasons:

    Smaller attack surface — fewer desktop Linux users means fewer malware authors targeting Linux
    SELinux — confines what programs can do, even if they're compromised
    Package management — software comes from trusted, signed repositories, not random websites
    User/root separation — you can't modify system files without explicitly elevating with sudo
    Permissions model — files and directories have granular permissions; programs can't write to system areas
    Flatpak sandboxing — containerized apps can't access the wider system

If you work in an enterprise environment that scans email or file servers for Windows malware (using a Linux-based scanner), tools like ClamAV exist. But for personal desktop use, antivirus is unnecessary overhead. We discuss this further in Chapter 27.
What You'll Miss

Be honest with yourself: some things from Windows don't have perfect equivalents on Fedora:

    Microsoft Office — LibreOffice is excellent but isn't identical. Office 365 works in a browser. OnlyOffice is a closer visual match.
    Adobe Creative Cloud — GIMP (Photoshop), Inkscape (Illustrator), Krita (digital painting), and Darktable (Lightroom) are powerful, but the workflow is different. There is no native Linux Premier Pro.
    Windows-specific games — Many games work via Proton (Chapter 28), but some with anti-cheat systems (Valorant, some Call of Duty titles) don't work yet.
    Outlook — Thunderbird or Geary are excellent email clients but feel different.
    Certain peripherals — Some niche hardware (specialized audio interfaces, old printers, proprietary VR headsets) may lack Linux drivers. Check compatibility before switching.

Coming from macOS
The Desktop Experience

macOS gives you a menu bar at the top, a Dock at the bottom, desktop icons, and windows with traffic-light buttons (close, minimize, maximize) in the upper left.

Fedora with GNOME 50 will feel surprisingly familiar in some ways and surprisingly different in others. GNOME's design philosophy was partly inspired by macOS — the top bar, the search-driven workflow, the gesture support, and the emphasis on keyboard navigation all have echoes of macOS.

But Fedora's GNOME is more minimal than macOS. There's no Dock at the bottom by default (the Dash only appears in the Activities Overview). There are no desktop icons by default. There's no global menu bar showing the current app's menus (macOS's top-left app name menu). GNOME apps use header bars (combined title bars with embedded controls) instead of macOS-style menu bars.
Mapping macOS Concepts to Fedora
macOS	Fedora Equivalent	Notes
Spotlight (Cmd+Space)	Activities Overview (Super key)	Type to search apps, files, settings
Dock	Dash (in Activities Overview)	Appears when you press Super; not always visible
Finder	GNOME Files (Nautilus)	Press Super, type "Files"
System Preferences	GNOME Settings	Press Super, type "Settings"
Activity Monitor	System Monitor / Resources	Press Super, type "System Monitor"
Terminal.app	GNOME Terminal	Press Super, type "Terminal"
Screenshot (Cmd+Shift+3/4)	Print Screen key	Full screen or region selection
Trash (Dock icon)	Trash (in Files sidebar)	Same concept, different location
Launchpad	Activities Overview app grid	Press Super, then browse the grid
Mission Control	Activities Overview workspaces	Dynamic workspaces, switch with Super
Time Machine	Déjà Dup / snapper	See Chapter 26 for backup options
Home folder (/Users/name)	/home/name	Same concept, different path
Disk Utility	GNOME Disks (gnome-disk-utility)	Press Super, type "Disks"
App Store	GNOME Software	RPM and Flatpak apps
Preview (image/PDF viewer)	Document Viewer (Evince/Papers)	Opens PDFs, images via Loupe
AirDrop	Not built-in	Use KDEConnect, LocalSend, or web sharing
FaceTime	No native equivalent	Use browser-based Meet, Zoom, or Proton Meet
iCloud	No direct equivalent	GNOME Online Accounts supports Google, Nextcloud
Quick Look (space to preview)	Not built-in	Install a GNOME extension or use file manager preview
Menu bar (global app menu)	Header bars (per-window)	Each app window has its own controls
File Paths

macOS uses forward slashes already, which makes the transition easier:

# macOS
/Users/john/Documents/report.docx

# Fedora
/home/john/Documents/report.docx

The structure is almost identical. Both use a Unix-style filesystem hierarchy. If you've used Terminal.app on macOS, you have a head start — many of the same commands (ls, cd, cat, grep, ssh) work identically on Fedora.
Keyboard Differences

If you're using an Apple keyboard, you'll need to adjust:
Mac Key	Fedora Equivalent
Command (⌘)	Super (Windows key) — or Ctrl, depending on context
Option (⌥)	Alt
Control (⌃)	Ctrl
Fn	Fn (same)
Cmd+C (copy)	Ctrl+C (in most apps) or Super+C (in GNOME Files)
Cmd+V (paste)	Ctrl+V or Super+V
Cmd+Space (Spotlight)	Super (Activities Overview)
Cmd+Tab (app switcher)	Super+Tab (switches between open windows)
Cmd+Q (quit)	Ctrl+Q (in most GNOME apps)
Three-finger swipe (spaces)	Three-finger swipe (workspaces) — same gesture, different OS

    💡 Apple Keyboard Layouts

    If you install Fedora on Apple hardware (MacBook, Mac mini), the keyboard layout may need adjustment. Fedora detects Apple keyboards during installation and usually configures them correctly. If keys feel swapped (Command and Ctrl), go to Settings → Keyboard to remap them. The hid_apple kernel module has parameters like fnmode and swap_opt_cmd that control behavior.

Trackpad Gestures

Fedora 44 with GNOME 50 on Wayland supports multi-touch trackpad gestures:
Gesture	Action
Three-finger swipe up	Open Activities Overview
Three-finger swipe left/right	Switch between workspaces
Pinch (two fingers)	Zoom (in supported apps)
Two-finger scroll	Scroll vertically/horizontally
Tap to click	(Enable in Settings → Mouse & Touchpad)

These gestures are similar to macOS, though not always identical. GNOME's gesture support has improved dramatically in recent releases.
What You'll Miss

Coming from macOS, you'll notice some absences:

    AirDrop — no built-in equivalent. KDEConnect, LocalSend (Flathub), or Warpinator can bridge the gap.
    iCloud integration — no native iCloud support. GNOME Online Accounts works with Google, Nextcloud, and Microsoft, but not Apple.
    Messages (iMessage) — no native iMessage client on Linux. Use web-based messaging or alternative chat apps.
    Final Cut Pro — DaVinci Resolve (free version available for Linux) is the professional alternative.
    Logic Pro — Reaper, Bitwig Studio, or Ardour are the Linux DAW options.
    Continuity features (Handoff, Universal Clipboard, Sidecar) — no native cross-device integration with Apple devices on Fedora.
    Retina display scaling — GNOME 50's fractional scaling (introduced in this release) significantly improves hi-DPI support, but it's still not as seamless as macOS.

The Application Paradigm Shift

This is the single biggest adjustment for users coming from either Windows or macOS. It deserves its own section because misunderstanding it causes the most frustration.
How You Installed Software Before

On Windows or macOS:

    You Google "download [app name]"
    You visit the developer's website
    You download an installer (.exe, .msi, .dmg, .pkg)
    You run the installer and click Next → Next → Finish
    The app installs — sometimes with a toolbar, sometimes with a trial, sometimes with a changed homepage

This model has problems:

    You're trusting the website not to serve a modified installer
    Installers can bundle unwanted software
    You have to manually check for updates by revisiting the website
    Uninstalling may leave behind files and registry entries
    There's no central authority verifying the software

How You Install Software on Fedora

On Fedora:

    You open GNOME Software (press Super, type "Software")
    You search for the app by name
    You click Install
    Done. The system handles download, installation, and registration

Alternatively, from the terminal:

sudo dnf install vlc

Or for a Flatpak:

flatpak install flathub com.spotify.Client

Fedora's software comes from repositories — trusted servers that host packages. When you install a piece of software, Fedora verifies its cryptographic signature. If the signature doesn't match, the installation is refused. This means:

    No fake installers — you can't accidentally download a trojanized version
    No bundled junk — packages contain only the software you requested
    Centralized updates — GNOME Software and DNF update ALL your software in one operation, not each app individually
    Clean removal — sudo dnf remove packagename removes everything cleanly
    Trust model — packages in Fedora's repos are reviewed by Fedora maintainers. Flathub Flatpaks are verified.

    💡 What About Software From Websites?

    Sometimes you do need to install software from outside the repositories — typically proprietary apps like Zoom, Discord, or Spotify that aren't in Fedora's free repos. In these cases, you'll usually install a Flatpak from Flathub (accessible through GNOME Software), or occasionally download an RPM package from the vendor's website.

    The general rule: if it's in GNOME Software, use that. If it's on Flathub, it'll show up in GNOME Software too. Only download installers from websites if neither of those works — and even then, verify you're on the official site.

RPM Fusion: The Gray Area

Some software isn't in Fedora's repos (because it's proprietary or patent-encumbered) but also isn't available as a Flatpak from Flathub. Examples include:

    Multimedia codecs (MP3, H.264, HEVC, AAC, DVD playback)
    NVIDIA proprietary drivers
    Certain media tools (ffmpeg with full codec support)

For these, you enable RPM Fusion — a third-party repository that fills the gap between Fedora's strict free software policy and what users actually need. RPM Fusion is covered in detail in Chapter 11.
The Filesystem Paradigm Shift
No Drive Letters

On Windows:

C:\Users\John\Documents
D:\Photos
E:\USB Drive

Each storage device gets its own letter. Files on different drives are in entirely separate trees.

On Fedora (and all Linux systems), there's a single filesystem tree rooted at /:

/                          ← the root of everything
├── home/
│   └── john/              ← your personal files
│       ├── Documents/
│       ├── Pictures/
│       └── Downloads/
├── etc/                   ← system configuration files
├── usr/                   ← installed programs and libraries
├── var/                   ← logs, caches, databases
├── tmp/                   ← temporary files
└── run/media/john/        ← USB drives and removable media mount here
    └── USBdrive/

A USB drive isn't E: — it's mounted somewhere under /run/media/yourname/. A second internal disk isn't D: — it's mounted wherever you configure it (often under /mnt/ or /media/).

Everything is part of one unified tree. This takes a day or two to get used to, but most users find it cleaner than drive letters once they adjust. Chapter 14 covers the full filesystem hierarchy in detail.
Case Sensitivity

On Windows and macOS (with the default APFS case-insensitive mode), Document.txt and document.txt are the same file. Fedora's filesystem (Btrfs or ext4) is case-sensitive by default:

    Document.txt and document.txt are two different files
    Report, report, and REPORT are three distinct files

This catches newcomers occasionally. It's not a problem once you're aware of it — and many find case-sensitivity useful for distinguishing related files (e.g., README vs. Readme vs. readme).
The Security Model: A Different Philosophy
Windows: Trust by Default

Windows traditionally runs applications with your full user privileges. If you can read a file, so can the program you're running. The User Account Control (UAC) prompt asks for permission when something tries to modify system files — but once an app is running, it generally has access to whatever you can access.

AppLocker and Windows Defender Application Control exist, but they're enterprise features most users never configure.
macOS: Gatekeeper and Notarization

macOS verifies that apps are signed by registered developers and notarized by Apple before allowing them to run. This is a central-authority trust model — Apple is the gatekeeper.

Once running, however, macOS apps generally have access to whatever your user account can access. App sandboxing exists but is opt-in for most apps.
Fedora: Defense in Depth

Fedora layers multiple security mechanisms:
Layer	What It Does
User/root separation	Normal user accounts can't modify system files. Administrative tasks require sudo.
SELinux	Confines what programs can do, even if the user running them has broad access. A compromised web server can't read your home directory, even though the user account theoretically could.
firewalld	Blocks incoming network connections by default. Nothing on your system is network-accessible until you explicitly open a port.
Package signing	All packages in Fedora's repositories are cryptographically signed. Tampering is detected and rejected.
Flatpak sandboxing	Containerized applications run in isolated environments with limited access to the host system.
Compiler hardening	Software is compiled with protections like stack-smashing protection, position-independent executables, and read-only relocations.
No open ports by default	A fresh Fedora installation exposes zero network services to the outside.

This means:

    Even if malware runs on your Fedora system, SELinux limits what it can do
    Even if a web server you're running has a vulnerability, firewalld blocks outside access unless you opened the port
    Even if a Flatpak app is compromised, it can't access your files unless you granted it permission

This layered approach is called defense in depth. No single mechanism is perfect, but together they make Fedora significantly harder to compromise than a default Windows installation.

    ⚠️ SELinux: Don't Disable It

    When something doesn't work on Fedora — a service won't start, a file can't be accessed, an app behaves strangely — a common online suggestion is "disable SELinux." Don't.

    Disabling SELinux removes a critical security layer and masks the real problem. The correct approach is to diagnose why SELinux is blocking the action and adjust the policy if needed. We cover this in Chapter 27. In 90% of cases, the issue is a misconfiguration, not an SELinux overreach.

User Accounts and Administration
Windows Model

Windows has two main account types:

    Administrator — can install software, change settings, modify system files
    Standard User — can use installed software, change personal settings, but can't install system-wide software without an admin password

UAC prompts let administrators temporarily elevate for specific actions.
macOS Model

macOS has:

    Admin users (members of the admin group) who can use sudo for system changes
    Standard users who can't

Fedora Model

Fedora (and all Linux systems) use a more nuanced model:

    root — the system administrator account (UID 0). Has unrestricted power. You don't log in as root.
    Regular users — normal accounts that can use their own files and run installed software
    The wheel group — users in this group can use sudo to temporarily elevate to root privileges for individual commands

By default, the user account you create during installation is placed in the wheel group. This means you can run sudo commands when you need administrative privileges — but you're not running as root all the time.

# You, as a normal user, can do this:
ls /home/john/

# You need sudo for this:
sudo dnf install vlc

This separation means that a compromised application running as your user can't modify system files — it would need to go through sudo, which requires your password.

Chapter 17 covers users, groups, sudo, and the wheel group in detail.
What About My Files?
Can I Access My Windows Files?

If you're dual-booting (running Fedora alongside Windows on the same computer), Fedora can read and write NTFS partitions. Your Windows files are accessible from Fedora — you'll find the Windows partition listed in Files' sidebar after mounting it.

However, writing to NTFS from Linux has historically been less reliable than reading. If you need reliable two-way access, consider:

    Using an exFAT-formatted shared partition (both Windows and Fedora read/write exFAT reliably)
    Storing shared files on a cloud service (Proton Drive, Nextcloud, Google Drive)
    Using a network share (SMB)

See Chapter 28 for more on accessing Windows files and running Windows software.
Can I Access My Mac Files?

If your Mac uses APFS (the default since macOS 10.13), Fedora can mount APFS volumes read-only using the linux-apfs driver (if available in your kernel). Read-write APFS support is limited.

If your Mac uses HFS+ (older Macs), Fedora can read and write HFS+ journals (with some caveats).

In practice, the easiest approach when migrating from macOS is to copy your files to an external drive formatted as exFAT, then copy them onto your Fedora system.
File Compatibility

Common file types and their status on Fedora:
File Type	Windows/macOS	Fedora
.docx, .xlsx, .pptx	Office/iWork	LibreOffice opens and edits natively
.pdf	Acrobat/Preview	Evince (Document Viewer) opens natively
.jpg, .png, .gif, .webp	Photos/Preview	Loupe (image viewer) opens natively
.mp4, .mkv, .webm	Media Player/VLC	Requires codecs (RPM Fusion — Chapter 11)
.mp3, .flac, .ogg	Music/iTunes	Requires codecs (RPM Fusion — Chapter 11)
.zip, .tar.gz	7-Zip/Archive Utility	Archive Manager (File Roller) opens natively
.exe, .msi	Windows installers	Won't run (use Wine — Chapter 28 — for some)
.dmg	macOS disk images	Won't run
.txt, .csv, .md	Notepad/TextEdit	Text Editor opens natively
.html, .css, .js	Browser/Editor	Firefox or any text editor
.psd (Photoshop)	Photoshop	GIMP opens (with some limitations)
.ai (Illustrator)	Illustrator	Inkscape opens (with some limitations)

Most document, image, and media formats work fine. The exceptions are proprietary application-specific formats (Adobe, Microsoft) that need their native applications — which may not have Linux versions.
Migration Checklist

If you're planning a full migration, here's a step-by-step checklist to follow before and after installation:
Before Installation

    Back up everything. Copy your files to an external drive or cloud storage. Don't skip this.
    List your essential applications. Write down every app you use regularly. For each, check if a Linux equivalent exists (see Appendix C or search alternativeto.net).
    Export browser bookmarks. In Firefox/Chrome, export bookmarks as an HTML file. Import them into Firefox on Fedora after install.
    Export email. If using a desktop email client, export your mailboxes. Thunderbird on Fedora can import from Outlook and Apple Mail.
    Check hardware compatibility. Search for your GPU, Wi-Fi card, printer model, and any specialized hardware plus the word "Linux" or "Fedora." Most hardware works; a few edge cases may need attention.
    Create a Fedora live USB. (Covered in Chapter 5.)
    Try Fedora before installing. Boot from the live USB and explore GNOME without touching your installed OS. Test Wi-Fi, audio, display, and trackpad.
    Decide: dual boot or full install. Dual booting lets you keep your old OS. Full install gives you the whole disk. See Chapter 6 for guidance.

After Installation

    Update the system. Run sudo dnf upgrade or open GNOME Software and install all available updates.
    Enable RPM Fusion. Chapter 11 covers this. Needed for media codecs and some drivers.
    Install Flatpak apps. Enable Flathub in GNOME Software and install your favorite apps.
    Configure your browser. Import bookmarks, install extensions, sign in to accounts.
    Set up email. Install Thunderbird or Geary and configure your accounts.
    Copy your files. Transfer documents, photos, music, and other personal files from your backup.
    Configure display and input. Adjust display scaling (especially on hi-DPI screens), set trackpad sensitivity, configure keyboard shortcuts.
    Install NVIDIA drivers if needed. Chapter 22 covers this for systems with NVIDIA GPUs.
    Set up backups. Chapter 26 covers Déjà Dup and other backup tools.
    Explore GNOME. Spend 30 minutes clicking through Settings, trying gestures, and getting comfortable.

Emotional Adjustment: The Honest Truth

Technical preparation is only half the journey. The other half is psychological, and it deserves acknowledgment.
The First Week: Confusion

Days 1–7 will feel disorienting. You'll reach for muscle memory that doesn't work — pressing the wrong keys, looking for the Start menu, not knowing how to change a setting. Every small task seems to take twice as long.

This is normal. Every Linux user went through it. The confusion isn't a sign that you made a mistake — it's a sign that you're learning a new system. Give yourself grace.
Weeks 2–4: Growing Comfort

By the second week, you'll stop reaching for nonexistent menus. You'll develop new muscle memory. You'll discover features that delight you — GNOME's search, workspace switching, the speed of DNF, the cleanliness of the desktop.

You'll also hit your first real snag. A media file won't play. A printer won't connect. A setting won't stick. These moments are frustrating, but each one teaches you something about how the system works. Use the troubleshooting guide in Chapter 29, search Ask Fedora, or ask in the community.
Months 2–3: Fluency

By the second month, the terminal stops being intimidating. You'll know a handful of commands by heart. You'll customize your desktop. You'll install software without thinking about it. You'll stop comparing everything to "how it worked on Windows/Mac" and start appreciating Fedora on its own terms.
Month 6+: Ownership

Eventually — and this is the moment that hooks Linux users forever — you realize you understand your computer. Not just the surface, but the structure. You know where your files are. You know what's running. You know how to change things, fix things, and make the system yours. This feeling of ownership and control is what converts visitors into permanent residents.

    💡 The Dip

    There's a well-known phenomenon in the Linux community called "the dip" — a period after the initial excitement wears off where the system feels harder than expected. Progress seems to stall. You wonder if it's worth the effort.

    It is. The dip is temporary. Push through it, and you emerge with a system you genuinely understand and control. Most users report the dip lasting 2–4 weeks.

Dual Boot or Full Switch?
Dual Boot (Keep Both Operating Systems)

Pros:

    Safety net — you can always boot back to Windows/macOS
    Access to Windows/macOS-only software when needed
    Gradual migration at your own pace

Cons:

    Partition management adds complexity
    Windows updates have been known to overwrite GRUB (the Linux bootloader)
    Rebooting to switch OSes is disruptive
    You never fully commit to learning Fedora

Full Switch (Fedora Only)

Pros:

    Forces full immersion — fastest path to fluency
    Full disk available to Fedora (more space for your files)
    No dual-boot maintenance headaches
    Total commitment means total adaptation

Cons:

    No safety net if you need Windows/macOS software urgently
    Must find alternatives for all your applications upfront
    Hardware surprises (a printer that doesn't work) are more stressful

    💡 The Recommended Path

    For most users: start with a live USB (try Fedora for a few days without installing anything), then dual boot for 1–2 months while you verify all your hardware and software needs are met, then go full Fedora once you're confident.

    For users with a spare computer: install Fedora on the spare machine and use it as a secondary device. When it becomes your primary naturally, you're ready.

Fedora vs. Your Old OS: Quick Reference Card
Task	Windows	macOS	Fedora
Search for apps	Start menu	Spotlight (Cmd+Space)	Activities (Super key)
Open file manager	Win+E	Finder	Super → "Files"
Take screenshot	Win+Shift+S	Cmd+Shift+3/4	Print Screen
Switch apps	Alt+Tab	Cmd+Tab	Super+Tab
Switch desktops	Win+Tab (Task View)	Ctrl+Arrow (Spaces)	Super+Arrow (workspaces)
Open terminal	(not common)	Terminal.app	Super → "Terminal" (or Ctrl+Alt+T)
Install software	Microsoft Store	App Store	GNOME Software
Force quit	Task Manager	Cmd+Opt+Esc	System Monitor
Lock screen	Win+L	Ctrl+Cmd+Q	Super+L
Show desktop	Win+D	F11 (or spread)	Super+H (hide windows)
Change settings	Settings app	System Preferences	Super → "Settings"
Open file location	Right-click → Open file location	Right-click → Show in Enclosing Folder	Right-click → "Open Item Location"
Rename file	F2	Enter	F2
Delete file	Delete key	Cmd+Delete	Delete key
Permanently delete	Shift+Delete	Cmd+Shift+Delete	Shift+Delete
What's Next

In Chapter 5, we'll get Fedora onto a USB drive. You'll learn how to download the correct Fedora 44 Workstation ISO, verify its integrity with a checksum, and create a bootable live USB using tools available on Windows, macOS, and Fedora itself. By the end of the next chapter, you'll have a USB drive that boots Fedora — ready to explore or install.
Try It Yourself

    Make your application list. Write down every application you use at least once a week. Next to each, note whether you know of a Linux equivalent. For the ones you don't, search alternativeto.net filtered by "Linux." Don't worry about installing anything yet — just build the map.

    Audit your hardware. Write down your computer model, CPU, GPU, RAM amount, Wi-Fi card, and any peripherals (printers, scanners, audio interfaces). Search each plus "Fedora" or "Linux." Most hardware works out of the box, but knowing in advance saves surprises.

    Visit GNOME's help documentation. Go to help.gnome.org and skim the GNOME user guide. You'll see screenshots and descriptions of the desktop you'll be using. Familiarity reduces first-day anxiety.

    Browse Flathub. Visit flathub.org and search for the apps on your list. Many will be available as Flatpaks — meaning they'll install easily on Fedora through GNOME Software. This reassures you that your favorite apps are reachable.

    Try Fedora in a virtual machine. If you have Windows Pro or macOS, install VirtualBox or GNOME Boxes (on macOS, download VirtualBox). Download the Fedora 44 ISO and run it in a VM. You won't get full performance, but you'll experience the desktop without touching your installed OS. This is the safest possible way to explore.

    ✅ Chapter 4 Complete You understand what transfers from Windows/macOS and what doesn't, how Fedora's application installation model differs (repositories vs. website installers), the filesystem paradigm shift (no drive letters, single root), the security model (SELinux, firewalld, defense in depth), the user account model (root, wheel group, sudo), file compatibility, how to prepare for migration, the emotional journey of switching, and whether to dual boot or go all in.

