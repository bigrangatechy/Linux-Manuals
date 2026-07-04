# Chapter 12: Software Equivalents

## "But I Need Photoshop"

This is probably the most common concern people have when considering a switch to Linux. They've spent years learning specific applications — Word, Photoshop, Outlook, Excel — and the prospect of replacing all of them feels overwhelming.

Here's the reality: **most Windows and macOS applications have excellent Linux equivalents.** Some are near-perfect drop-in replacements. Others require a small adjustment period. A few proprietary applications have no direct equivalent, but workarounds exist (browser versions, Wine, virtual machines).

This chapter is your reference guide. Bookmark it — you'll come back to it repeatedly during your first weeks on Ubuntu.

## How to Use This Chapter

Applications are organized by **category**. Each entry shows:

- What you used on Windows/macOS
- The recommended Linux equivalent(s)
- How well the replacement matches the original
- How to install it
- Any caveats or limitations

Rating scale for replacement quality:

| Rating | Meaning |
|---|---|
| ⭐⭐⭐⭐⭐ | Near-perfect replacement — same or better |
| ⭐⭐⭐⭐ | Excellent — minor adjustments needed |
| ⭐⭐⭐ | Good — workflow changes required |
| ⭐⭐ | Functional but different — patience needed |
| ⭐ | Workaround — web app, VM, or limited alternative |

## Office and Productivity

### Office Suite

| Windows/macOS App | Linux Equivalent | Rating | Install Method |
|---|---|---|---|
| Microsoft Word | LibreOffice Writer | ⭐⭐⭐⭐ | Pre-installed / APT |
| Microsoft Excel | LibreOffice Calc | ⭐⭐⭐⭐ | Pre-installed / APT |
| Microsoft PowerPoint | LibreOffice Impress | ⭐⭐⭐⭐ | Pre-installed / APT |
| Microsoft Office (full) | OnlyOffice | ⭐⭐⭐⭐ | Snap / Flathub |

**LibreOffice** comes pre-installed on Ubuntu 26.04. It's a full office suite — Writer (word processor), Calc (spreadsheet), Impress (presentations), Draw (diagrams), Base (database), and Math (formula editor). It opens and saves `.docx`, `.xlsx`, and `.pptx` files with high fidelity.

> 💡 **Compatibility Tips**
>
> LibreOffice handles most Office documents well, but complex formatting (especially in Excel with macros, or Word documents with intricate layout) may render slightly differently. For maximum compatibility:
>
> - Save documents in `.docx`/`.xlsx`/`.pptx` format (not LibreOffice's native `.odt`/`.ods`/`.odp`) if you share files with Office users
> - Use standard fonts (Arial, Calibri, etc.) — install via `ubuntu-restricted-extras` (Chapter 11)
> - For perfect Office compatibility, try **OnlyOffice** — it uses the same document format as MS Office and renders `.docx` files more faithfully than LibreOffice
>
> **Install OnlyOffice:**
> ```
> sudo snap install onlyoffice-desktopeditors
> ```
> Or from Flathub:
> ```
> flatpak install flathub org.onlyoffice.desktopeditors
> ```

### Microsoft Office Online (Web)

If you need absolute Office fidelity, Microsoft Office Online is free in your browser at **[office.com](https://office.com)** — Word, Excel, PowerPoint, and OneNote all work in Firefox. This requires a free Microsoft account and an internet connection, but for occasional documents, it's the closest you'll get to the real thing.

### Note-Taking

| Windows/macOS App | Linux Equivalent | Rating | Install Method |
|---|---|---|---|
| Microsoft OneNote | Joplin | ⭐⭐⭐⭐ | APT / Snap / Flathub |
| Evernote | Joplin | ⭐⭐⭐⭐ | APT / Snap / Flathub |
| Notion | Notion (web) or AppFlowy | ⭐⭐⭐ | Browser / Flathub (AppFlowy) |
| Obsidian | Obsidian (native) | ⭐⭐⭐⭐⭐ | `.deb` / Flatpak / AppImage |
| Apple Notes | Tomboy Notes / Notes (GNOME) | ⭐⭐⭐ | Pre-installed (GNOME Notes) / APT |

**Joplin** is the recommended replacement for OneNote and Evernote. It's a Markdown-based note-taking app with cloud sync (Nextcloud, Dropbox, OneDrive, WebDAV), end-to-end encryption, and web clipper browser extension. It organizes notes into notebooks and sub-notebooks with tagging support.

sudo apt install joplin

Or from Flathub:

flatpak install flathub net.cozic.joplin_desktop

Obsidian runs natively on Linux — it's not open source, but the desktop app is free and works perfectly. Download the .deb from obsidian.md.
Email
Windows/macOS App	Linux Equivalent	Rating	Install Method
Microsoft Outlook	Thunderbird	⭐⭐⭐⭐	APT / Snap / Flathub
Apple Mail	Thunderbird or Evolution	⭐⭐⭐⭐	APT
Spark Mail	Thunderbird	⭐⭐⭐	APT

Thunderbird is the gold standard for Linux email. It's a full-featured desktop email client by Mozilla (the Firefox organization) with:

    Multiple account support (IMAP, POP, Exchange Web Services)
    Calendar and task integration (via add-ons)
    Built-in RSS feed reader
    Contact management
    Robust spam filtering
    Large extension ecosystem

sudo apt install thunderbird

    💡 Exchange/Office 365 Email

    Thunderbird supports Office 365 and Exchange via IMAP, but calendar sync requires an add-on. For full Exchange integration (calendar, contacts, global address list), consider Evolution (GNOME's email client):

    sudo apt install evolution

    Evolution has built-in Exchange Web Services support and integrates tightly with GNOME's calendar and contacts.

PDF Tools
Windows/macOS App	Linux Equivalent	Rating	Install Method
Adobe Acrobat Reader	Document Viewer (Evince)	⭐⭐⭐	Pre-installed
Adobe Acrobat Pro	Master PDF Editor / LibreOffice Draw	⭐⭐⭐	APT / Flathub
Foxit Reader	Foxit Reader (Linux version)	⭐⭐⭐⭐	.deb from foxit.com
PDF annotator	Xournal++	⭐⭐⭐⭐	APT / Flathub

Ubuntu's default PDF viewer (Document Viewer, also known as Evince) handles reading, bookmarks, and basic annotation. For advanced editing (merge, split, fill forms, add text):

Master PDF Editor:

flatpak install flathub net.codeindustry.MasterPDFEditor

Xournal++ (handwritten annotations, highlighting, drawing on PDFs):

flatpak install flathub com.github.xournalpp.xournalpp

Task Management
Windows/macOS App	Linux Equivalent	Rating	Install Method
Todoist	Todoist (web) / superProductivity	⭐⭐⭐	Browser / Flathub
OmniFocus	Taskwarrior / Super Productivity	⭐⭐⭐	APT / Flathub
Things	Endeavour / Planner	⭐⭐⭐	APT (Endeavour) / Flathub (Planner)

Endeavour is GNOME's default task manager, pre-installed on Ubuntu. It's simple but effective — lists, due dates, priorities, and integration with GNOME's calendar.
Web Browsers
Windows/macOS App	Linux Equivalent	Rating	Install Method
Google Chrome	Google Chrome (native Linux)	⭐⭐⭐⭐⭐	.deb from google.com/chrome
Mozilla Firefox	Firefox (pre-installed Snap)	⭐⭐⭐⭐⭐	Pre-installed
Microsoft Edge	Edge (native Linux)	⭐⭐⭐⭐⭐	.deb from microsoft.com/edge
Brave	Brave (native Linux)	⭐⭐⭐⭐⭐	Snap / .deb
Opera	Opera (native Linux)	⭐⭐⭐⭐	Snap / .deb
Safari	Firefox or Chrome	⭐⭐⭐	N/A (no Safari on Linux)
Vivaldi	Vivaldi (native Linux)	⭐⭐⭐⭐⭐	.deb / Snap

The good news here: almost every major browser has a native Linux version. Chrome, Edge, Brave, Opera, and Vivaldi all run identically to their Windows counterparts.

    💡 Firefox Snap vs. Firefox Flatpak

    On Ubuntu 26.04, Firefox ships as a Snap by default. The Snap version has some limitations (AV1 hardware decoding, see Chapter 11). If you prefer, you can install Firefox as a Flatpak or .deb for better hardware access:

    flatpak install flathub org.mozilla.firefox

Communication
Windows/macOS App	Linux Equivalent	Rating	Install Method
Microsoft Teams	Teams (native Linux client)	⭐⭐⭐⭐	.deb / Snap
Zoom	Zoom (native Linux client)	⭐⭐⭐⭐⭐	.deb from zoom.us
Slack	Slack (native Linux)	⭐⭐⭐⭐⭐	Snap / .deb
Discord	Discord (native Linux)	⭐⭐⭐⭐⭐	Snap / .deb / Flathub
Skype	Skype (native Linux)	⭐⭐⭐⭐	Snap / .deb
Telegram	Telegram (native Linux)	⭐⭐⭐⭐⭐	Snap / APT / Flathub
Signal	Signal (native Linux)	⭐⭐⭐⭐⭐	Snap / Flatpak / APT (via PPA)
WhatsApp	WhatsApp Web / ZapZilla	⭐⭐⭐	Browser / Flathub
FaceTime	Zoom / Jitsi Meet	⭐⭐	Browser / Flathub

Most communication apps have excellent native Linux support. Zoom runs natively and is feature-complete. Discord and Slack are identical to their Windows versions. Signal is fully supported.

Microsoft Teams now has a native Linux client on Ubuntu 26.04 — a significant improvement from the old "Progressive Web App only" era. Install it as a Snap:

sudo snap install teams

Or download the .deb from Microsoft's website.

    ⚠️ WhatsApp

    There's no official WhatsApp desktop app for Linux. Use WhatsApp Web in Firefox (works perfectly), or community-built wrappers like ZapZilla (Flathub) that wrap the web interface in a desktop window with notification support.

Graphics and Design
Photo Editing
Windows/macOS App	Linux Equivalent	Rating	Install Method
Adobe Photoshop	GIMP	⭐⭐⭐	APT / Snap / Flathub
Adobe Lightroom	Darktable / RawTherapee	⭐⭐⭐⭐	APT / Flathub
Affinity Photo	GIMP / Krita	⭐⭐⭐	APT / Flathub
Paint.NET	Pinta	⭐⭐⭐⭐	APT / Flathub
Pixelmator	GIMP / Krita	⭐⭐⭐	APT / Flathub

GIMP (GNU Image Manipulation Program) is the most well-known Linux image editor. It handles photo retouching, composition, and graphic design. The main adjustment from Photoshop: GIMP's interface differs, and some professional CMYK/print features are limited. For web graphics, social media images, and general photo editing, GIMP is more than sufficient.

Darktable is an excellent RAW photo processor — a genuine alternative to Lightroom for photographers. It handles non-destructive editing, catalog management, and RAW development for hundreds of camera models.

sudo apt install gimp darktable

Or from Flathub:

flatpak install flathub org.gimp.GIMP
flatpak install flathub org.darktable.Darktable

Digital Painting and Illustration
Windows/macOS App	Linux Equivalent	Rating	Install Method
Adobe Illustrator	Inkscape	⭐⭐⭐⭐	APT / Flathub
Clip Studio Paint	Krita	⭐⭐⭐⭐	APT / Flathub
Procreate	Krita	⭐⭐⭐	APT / Flathub
CorelDRAW	Inkscape	⭐⭐⭐	APT / Flathub

Krita is a professional-grade digital painting application — arguably better than Photoshop for illustration work. It supports brush stabilizers, vector layers, animation, HDR, and pressure-sensitive tablets (Wacom, Huion, etc.) out of the box.

Inkscape is a vector graphics editor comparable to Illustrator. It uses SVG as its native format and handles logos, icons, illustrations, and diagrams.

sudo apt install krita inkscape

Or from Flathub:

flatpak install flathub org.kde.krita
flatpak install flathub org.inkscape.Inkscape

3D Modeling
Windows/macOS App	Linux Equivalent	Rating	Install Method
Autodesk Maya	Blender	⭐⭐⭐⭐	Snap / Flathub / APT
Cinema 4D	Blender	⭐⭐⭐	Snap / Flathub
AutoCAD	FreeCAD / LibreCAD	⭐⭐⭐	APT / Flathub
SketchUp	Sweet Home 3D (architecture) / Blender	⭐⭐	APT / Flathub

Blender is one of the crown jewels of open source software. It's a professional 3D creation suite — modeling, sculpting, animation, rendering, video editing, simulation, and game creation. Studios use it alongside Maya in production pipelines. It's free, runs natively on Linux, and receives constant updates.

sudo snap install blender --classic

Or:

flatpak install flathub org.blender.Blender

    💡 DaVinci Resolve on Linux

    DaVinci Resolve (professional video editing and color grading) has a native Linux version. It's not open source (free version available), but it's a genuine Hollywood-grade editor running natively. Download the .deb from blackmagicdesign.com. Note: the Linux version has some hardware requirements (NVIDIA GPU recommended).

Video and Audio
Video Editing
Windows/macOS App	Linux Equivalent	Rating	Install Method
Adobe Premiere Pro	Kdenlive / DaVinci Resolve	⭐⭐⭐⭐	Flathub / .deb (Resolve)
Final Cut Pro	Kdenlive / DaVinci Resolve	⭐⭐⭐	Flathub / .deb
iMovie	Kdenlive / Pitivi	⭐⭐⭐⭐	Flathub / APT
Filmora	Shotcut / Kdenlive	⭐⭐⭐⭐	Flathub / APT

Kdenlive is the best free, open-source video editor on Linux. Multi-track editing, effects, transitions, color correction, audio mixing — it handles everything most users need. For professional color grading, DaVinci Resolve (above) is the step up.

flatpak install flathub org.kde.kdenlive

Screen Recording and Streaming
Windows/macOS App	Linux Equivalent	Rating	Install Method
OBS Studio	OBS Studio (native Linux)	⭐⭐⭐⭐⭐	Flathub / APT
Camtasia	OBS Studio / SimpleScreenRecorder	⭐⭐⭐	Flathub / APT
ScreenFlow	OBS Studio / Kooha	⭐⭐⭐	Flathub / APT
XSplit	OBS Studio	⭐⭐⭐⭐	Flathub

OBS Studio runs natively on Linux with identical features to its Windows version — streaming to Twitch/YouTube, multi-scene composition, audio mixing, filters, and recording.

flatpak install flathub com.obsproject.Studio

Audio Editing
Windows/macOS App	Linux Equivalent	Rating	Install Method
Adobe Audition	Audacity	⭐⭐⭐⭐	APT / Flathub
GarageBand	LMMS / Ardour	⭐⭐⭐	APT / Flathub
Logic Pro	Ardour / Bitwig Studio	⭐⭐⭐	APT / Commercial
Ableton Live	Bitwig Studio (native Linux)	⭐⭐⭐⭐	.deb (commercial)
FL Studio	LMMS / Renoise	⭐⭐⭐	APT / Commercial
Reaper	Reaper (native Linux)	⭐⭐⭐⭐⭐	.deb from reaper.fm

Audacity is the go-to free audio editor — recording, editing, effects, noise reduction. It's pre-installed on many systems or available via:

sudo apt install audacity

Reaper deserves special mention: it has a fully native Linux version, is professionally used for music production, and costs $60 for a personal license (with an unlimited evaluation period).
Media Playback

Covered extensively in Chapter 11, but for quick reference:
Windows/macOS App	Linux Equivalent	Rating	Install Method
Windows Media Player	Showtime / VLC	⭐⭐⭐⭐⭐	Pre-installed / APT
QuickTime	Showtime / VLC	⭐⭐⭐⭐⭐	Pre-installed / APT
iTunes	Rhythmbox / Spotify / Music (various)	⭐⭐⭐	Pre-installed / Snap
VLC	VLC (identical)	⭐⭐⭐⭐⭐	APT / Snap / Flathub
File Management and Archiving
Windows/macOS App	Linux Equivalent	Rating	Install Method
File Explorer (Windows)	Files (GNOME) / Dolphin (KDE)	⭐⭐⭐⭐	Pre-installed
Finder (macOS)	Files / Dolphin	⭐⭐⭐⭐	Pre-installed
WinRAR / 7-Zip	File Roller / Ark / p7zip	⭐⭐⭐⭐⭐	Pre-installed / APT
WinSCP	FileZilla / Nautilus (SFTP)	⭐⭐⭐⭐	APT

Ubuntu's built-in archive manager (File Roller for GNOME, Ark for KDE) handles .zip, .tar.gz, .7z, .rar (after installing unrar), and many other formats. No need for WinRAR or 7-Zip — just double-click any archive file.
Cloud Storage
Windows/macOS App	Linux Equivalent	Rating	Install Method
Google Drive	GNOME Online Accounts / Insync	⭐⭐⭐⭐	Pre-installed / .deb (Insync)
Dropbox	Dropbox (native Linux)	⭐⭐⭐⭐⭐	.deb from dropbox.com
Microsoft OneDrive	OneDrive Client (abraunegg)	⭐⭐⭐⭐	APT / Snap
iCloud	Browser / rclone	⭐⭐	Browser
Box	rclone / Browser	⭐⭐⭐	APT / Browser

Dropbox has an official native Linux client that works identically to the Windows version. Install from dropbox.com/install.

Google Drive integrates through GNOME Online Accounts (Settings → Online Accounts → Google), giving you file access through Files (Nautilus). Note: Drive integration in GNOME Files was changed in GNOME 50 — you may need to access Drive files through the browser or use a third-party tool like Insync for full sync.

OneDrive doesn't have an official Linux client, but the community-maintained OneDrive Client for Linux (by abraunegg) provides full two-way sync:

sudo apt install onedrive

After installation, run onedrive and follow the browser authentication prompt. For automatic background sync, enable the systemd service:

systemctl --user enable --now onedrive

    ⚠️ OneDrive Setup Complexity

    The abraunegg OneDrive client is command-line based. There's no GUI by default. Configuration involves editing a config file and authenticating through a browser redirect. It works reliably once set up, but the initial setup requires comfort with the terminal (Chapter 13+). If you need a GUI alternative, rclone can mount OneDrive as a filesystem, though without real-time sync.

Development Tools
Windows/macOS App	Linux Equivalent	Rating	Install Method
Visual Studio	VS Code (native Linux)	⭐⭐⭐⭐⭐	.deb / Snap / Flathub
IntelliJ IDEA	IntelliJ IDEA (native Linux)	⭐⭐⭐⭐⭐	Snap / Flathub / .tar.gz
Eclipse	Eclipse (native Linux)	⭐⭐⭐⭐⭐	APT / Snap
Sublime Text	Sublime Text (native Linux)	⭐⭐⭐⭐⭐	APT / Snap / Flathub
Notepad++	Geany / Kate / VS Code	⭐⭐⭐⭐	APT / Snap
Vim/Neovim	Vim/Neovim (native)	⭐⭐⭐⭐⭐	APT
Git Bash / PowerShell	GNOME Terminal / Ptyxis	⭐⭐⭐⭐⭐	Pre-installed
Docker Desktop	Docker Engine (native)	⭐⭐⭐⭐	APT (see Chapter 23)
PuTTY	SSH (built-in) / Remmina	⭐⭐⭐⭐⭐	APT / Pre-installed

Linux is a developer's paradise — most development tools run natively or were originally designed for Linux. VS Code has a native Linux build with full extension support. Git is built into the system. SSH is built into the system. Docker runs natively without the overhead of a virtual machine (unlike on Windows and macOS).

sudo snap install code --classic  # VS Code
sudo apt install git              # Git (likely pre-installed)

Gaming
Windows/macOS App	Linux Equivalent	Rating	Install Method
Steam (Windows)	Steam (native Linux + Proton)	⭐⭐⭐⭐⭐	.deb from steampowered.com
Epic Games Store	Heroic Games Launcher	⭐⭐⭐	Flathub / .deb
GOG Galaxy	GOG games (via Lutris or direct)	⭐⭐⭐⭐	Flathub (Lutris)
Battle.net	Lutris / Bottles	⭐⭐⭐	Flathub
Origin/EA	Lutris	⭐⭐	Flathub
GeForce Experience	MangoHud + GameMode	⭐⭐⭐	APT
Steam and Proton

Steam runs natively on Linux. Valve (Steam's parent company) developed Proton — a compatibility layer that allows Windows games to run on Linux with minimal effort. Most popular games work; check compatibility at protondb.com before purchasing.

Install Steam:

sudo apt install steam

Or download the .deb from steampowered.com.

In Steam settings, enable Steam Play for all other titles (Settings → Compatibility → Check "Enable Steam Play for all other titles"). This enables Proton for non-native Linux games.
Lutris and Bottles

Lutris is a gaming platform that aggregates games from multiple sources (Steam, GOG, Epic, emulators, Wine) into one library:

flatpak install flathub net.lutris.Lutris

Bottles is a newer, simpler tool for running Windows applications and games through Wine:

flatpak install flathub com.usebottles.bottles

    💡 Anti-Cheat Games

    Some multiplayer games with kernel-level anti-cheat (Valorant, Fortnite, Destiny 2) do not work on Linux because anti-cheat systems block non-Windows environments. Check areweanticheatyet.com before buying games with anti-cheat if you plan to play on Linux.

Password Managers
Windows/macOS App	Linux Equivalent	Rating	Install Method
1Password	1Password (native Linux)	⭐⭐⭐⭐⭐	Snap / .deb
Bitwarden	Bitwarden (native Linux)	⭐⭐⭐⭐⭐	Snap / Flathub / APT
LastPass	Bitwarden / KeePassXC	⭐⭐⭐	Browser / APT
Dashlane	Bitwarden / KeePassXC	⭐⭐⭐	Browser / APT
KeePass	KeePassXC	⭐⭐⭐⭐⭐	APT / Snap / Flathub

Bitwarden has a fully native Linux client (not just a browser extension) and is free/open source. 1Password also has a native Linux app. KeePassXC is the local, offline password manager for those who prefer storing databases locally rather than in the cloud.

sudo snap install bitwarden       # Bitwarden
sudo apt install keepassxc        # KeePassXC

Antivirus and Security
Windows/macOS App	Linux Equivalent	Rating	Install Method
Windows Defender	ClamAV / ClamTk (GUI)	⭐⭐⭐	APT
Malwarebytes	ClamAV / chkrootkit	⭐⭐	APT
Norton / McAfee	Not needed on Linux	N/A	N/A

Here's the honest truth: you generally don't need antivirus software on Linux. The combination of Ubuntu's security model (user separation, sudo requirements, Snap/Flatpak sandboxing) and Linux's small desktop market share means malware targeting Linux desktops is extremely rare.

If you want peace of mind — or if you share files with Windows users and want to scan for Windows malware — install ClamAV:

sudo apt install clamav clamtk

clamtk provides a simple GUI. Scan individual files or directories. This is most useful for scanning files before emailing them to Windows users.

    💡 The Real Security Strategy

    Your best defense on Linux is:

        Keep your system updated (Chapter 7's checklist)
        Enable UFW firewall (Chapter 7)
        Enable Ubuntu Pro for extended security updates (Chapter 7)
        Only install software from trusted sources (App Center, Flathub, official repos)
        Use a password manager (above)

    We cover security in depth in Chapter 24.

When There's No Native Equivalent

Sometimes an application simply doesn't exist on Linux. Here are your options, from easiest to hardest:
Option 1: Use the Web Version

Many apps have web interfaces that work in Firefox — Office Online, Canva, Figma, Notion, Trello, Asana, Gmail, WhatsApp Web. For many users, the web version is sufficient.
Option 2: Wine / Bottles

Wine is a compatibility layer that allows some Windows applications to run on Linux without a Windows license or virtual machine. Bottles (above) provides a friendly GUI for managing Wine installations.

Check compatibility at winehq.org — some apps work flawlessly, others partially, some not at all.
Option 3: Virtual Machine

For applications that absolutely must run on Windows, you can run a full Windows virtual machine inside Ubuntu:

sudo apt install virtualbox

Or use GNOME Boxes (simpler interface, pre-installed on Ubuntu):

sudo apt install gnome-boxes

Install Windows inside the VM, then run your Windows apps there. This requires a Windows license and adequate RAM (at least 8GB total, 4GB allocated to the VM).
Option 4: Dual Boot

If a Windows application is critical for your work and virtualization doesn't perform well enough, dual-boot Windows and Ubuntu. This is covered in Chapter 6 (Installation).
Quick Reference: The Essentials

If you're setting up a new Ubuntu install and want the most common replacements installed immediately:

# Office and productivity
sudo apt install libreoffice thunderbird

# Media
sudo apt install vlc audacity
flatpak install flathub org.kde.kdenlive

# Graphics
sudo apt install gimp inkscape
flatpak install flathub org.kde.krita
flatpak install flathub org.darktable.Darktable

# Communication
sudo snap install discord slack

# Development
sudo snap install code --classic
sudo apt install git

# Gaming
sudo apt install steam

# Password manager
sudo apt install keepassxc

# Note-taking
flatpak install flathub net.cozic.joplin_desktop

    💡 Don't Install Everything at Once

    This list is for reference. Install what you actually need, when you need it. Starting with LibreOffice, VLC, and a browser is enough for most people's first day.

What's Next

You now have a comprehensive map from your old software ecosystem to your new one. Most applications have excellent equivalents — some are even better than the originals. The areas where Linux is weakest (niche professional software like AutoCAD, Adobe suite) have workarounds via web apps, Wine, or virtual machines.

Phase 1 is now complete. Chapters 1 through 12 gave you the foundation: understanding what Linux and Ubuntu are, installing the system, configuring it, navigating the desktop, managing software, and finding your applications.

Phase 2 begins with Chapter 13: Terminal Absolute Basics. This is where many beginners feel intimidated — but it's also where Linux's true power becomes accessible. The terminal isn't something to fear. It's a tool, and we'll learn it from scratch.
Try It Yourself

    Install your three most-used apps. Think about what you used daily on your previous OS. Find their Linux equivalents in this chapter and install them.

    Try OnlyOffice. If you work with Office documents frequently, install OnlyOffice and open a .docx file in both LibreOffice and OnlyOffice. Compare the rendering.

    Check ProtonDB for your games. If you game, visit protondb.com and search for titles in your Steam library. See their Linux compatibility ratings.

    Set up your email. Install Thunderbird, configure your email account, and verify you can send and receive messages.

    Explore Flathub. Browse flathub.org and discover apps you didn't know existed for Linux. You might be surprised at what's available.
