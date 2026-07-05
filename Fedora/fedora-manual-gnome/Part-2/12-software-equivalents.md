# Chapter 12: Software Equivalents — Finding Linux Replacements

## The Mental Shift

Coming from Windows or macOS? Your brain has muscle memory for application names: Adobe Photoshop, Microsoft Word, Google Chrome, Discord, Steam, Netflix. You know where they live, what they do, how you interact with them.

Linux doesn't work that way. There's no central marketplace where every vendor ships a native app. Instead, the Linux ecosystem is organized around open source projects, community distributions, and repository packaging. Sometimes you'll find an exact equivalent (LibreOffice ≈ Microsoft Office). Sometimes you'll find something even better (GIMP + Krita ≈ Photoshop + Illustrator). And sometimes... there's nothing quite like it.

This chapter maps your Windows/macOS application habits to Linux alternatives. By the end, you'll know what to install when you need specific functionality. We cover productivity, creative work, development, gaming, system utilities, and everything in between.

> 💡 **Two Paths Forward**
>
> For each category, we'll present two options:
> 1. **Exact replacements** — applications that perform similar functions to what you're used to
> 2. **Native paradigms** — ways Linux apps might work differently, often more flexibly
>
> Don't force Linux into Windows-shaped thinking. Sometimes the best replacement isn't the closest clone — it's the tool that integrates naturally with the Linux ecosystem.

---

## Table of Contents

| Category | Windows/macOS Standard | Linux Equivalent | Notes |
|---|---|---|---|
| **Web Browser** | Chrome, Safari | Firefox, Chromium | See Section 1 |
| **Office Suite** | Microsoft Office | LibreOffice, OnlyOffice | See Section 2 |
| **PDF Reader** | Acrobat Reader | Document Viewer, Okular | See Section 3 |
| **Photo Editor** | Photoshop | GIMP, Krita | See Section 4 |
| **Vector Graphics** | Illustrator | Inkscape | See Section 5 |
| **Video Editing** | Premiere Pro, iMovie | DaVinci Resolve, Kdenlive | See Section 6 |
| **Music Production** | Logic, FL Studio | Ardour, LMMS | See Section 7 |
| **Messaging** | WhatsApp, Slack | Element, Telegram, Signal | See Section 8 |
| **Email Client** | Outlook, Apple Mail | Thunderbird, Evolution | See Section 9 |
| **Calendar** | Google Calendar, Outlook | GNOME Calendar, Lightning | See Section 10 |
| **Cloud Storage** | OneDrive, Dropbox | Nextcloud, Syncthing | See Section 11 |
| **Media Player** | VLC, iTunes | MPV, Celluloid, Rhythmbox | See Section 12 |
| **Compression** | WinZip, WinRAR | Archive Manager, p7zip | See Section 13 |
| **Virtual Machine** | VirtualBox, VMware | GNOME Boxes, VirtualBox | See Section 14 |
| **Antivirus** | Norton, McAfee | ClamAV (rarely needed) | See Section 15 |
| **System Monitoring** | Task Manager, Activity Monitor | System Monitor, htop | See Section 16 |
| **Terminal Emulator** | CMD, PowerShell, Terminal | GNOME Terminal, Kitty | See Section 17 |
| **Text Editor** | Notepad, VS Code | GNOME Text Editor, VS Code | See Section 18 |
| **IDE** | Visual Studio | JetBrains, GNStudio | See Section 19 |
| **Codecs/Playback** | QuickTime, Media Player Classic | FFmpeg-based players | Covered in Ch. 11 |
| **Screenshot Tool** | Snipping Tool, Skitch | Screenshot, Flameshot | See Section 20 |
| **Remote Desktop** | RDP, TeamViewer | Remmina, RustDesk | See Section 21 |
| **Password Manager** | LastPass, 1Password | Bitwarden, KeePassXC | See Section 22 |
| **Note Taking** | Evernote, OneNote | Joplin, Obsidian | See Section 23 |
| **Task Management** | Trello, Asana | OpenProject (self-hosted) | See Section 24 |
| **Game Launcher** | Steam | Steam (native) | See Section 25 |
| **System Tweaks** | Registry, Tweaking.com | GNOME Tweaks, RegEDIT | See Section 26 |

Now let's dive into each category.

---

## 1. Web Browsers

### What You Might Use

| Platform | Primary Options |
|---|---|
| **Windows** | Edge, Chrome, Firefox, Brave |
| **macOS** | Safari, Chrome, Firefox, Brave |
| **Linux** | Firefox, Chromium, Brave, Edge (beta) |

### Native Linux Choices

**Firefox (Built-in)** — Fedora ships Firefox as the default browser. It's Chromium-independent, respects privacy by default, blocks trackers out of the box, and has excellent Wayland support. Install add-ons from mozilla.org/addons.

**Chromium / Google Chrome** — Install either for full Google integration:
```bash
sudo dnf install chromium          # Community-built Chromium
flatpak install flathub com.google.Chrome   # Official Chrome

Brave — Privacy-focused Chromium fork with built-in ad blocking:

sudo dnf copr enable xtraume.brave-browser brave-browser-stable
sudo dnf install brave-browser

Microsoft Edge — Yes, Edge runs on Linux:

curl https://packages.microsoft.com/repos/edge-cli > edge.repo
sudo dnf install microsoft-edge-stable

Recommendation

If you care about privacy and independence from Big Tech: use Firefox with uBlock Origin extension. If you're deep in the Google ecosystem (Chrome sync, Workspace apps): use Chrome. Either works excellently on Linux.

    ⚠️ Google Drive Integration Removed

    GNOME 50 removed the built-in Google Drive integration that existed in previous versions. Files in Google Drive won't appear automatically in Nautilus. Workarounds: use rclone mount, google-drive-ocamlfuse, or access via browser.

2. Office Suites
What You Might Use
Platform	Primary Options
Windows	Microsoft Office, LibreOffice, WPS Office
macOS	Microsoft Office, Pages, LibreOffice
Linux	LibreOffice, OnlyOffice, WPS Office
Native Linux Choices

LibreOffice (Built-in) — Fedora's default office suite. Complete replacement for Microsoft Office: Writer (Word), Calc (Excel), Impress (PowerPoint), Draw (Visio basics). Handles .docx, .xlsx, .pptx files well. Free and open source.

Install additional components:

sudo dnf install libreoffice-base    # Database tool
sudo dnf install libreoffice-math     # Formula editor
sudo dnf install libreoffice-wiki-publisher  # Wiki export

OnlyOffice — Closer MS Office compatibility (format-wise):

flatpak install flathub org.onlyoffice.desktopeditors

Uses .docx, .xlsx, .pptx natively. Better formatting fidelity with complex Microsoft documents. UI resembles modern MS Office ribbon interface.

WPS Office — Another MS-compatible option:

flatpak install flathub wps-office

Free version has some limitations but excellent document rendering. Proprietary closed-source.
Microsoft Office Alternatives That Don't Exist

There's no official Microsoft Office for Linux. No workaround will give you perfect parity. If you absolutely must have Office:

    Use Office Online — office.com runs in browser with nearly all features
    Run Windows inside a VM — heavy but guaranteed compatibility (Chapter 24 covers virtualization)

Recommendation

For most people: LibreOffice handles 95% of use cases perfectly. For businesses exchanging complex documents: use OnlyOffice or run Office Online. For heavy Word/Excel power users: consider staying on Windows or use Wine/Lutris (advanced).
3. PDF Readers
What You Might Use
Platform	Primary Options
Windows	Adobe Acrobat Reader, SumatraPDF
macOS	Preview, Adobe Acrobat Reader
Linux	Document Viewer (Evince/Papers), Okular, Foxit
Native Linux Choices

Document Viewer (Evince/Papers) (Built-in) — Fedora's default PDF viewer. Clean, simple, annotation support (highlight, notes). Good for reading, basic annotating. Part of GNOME Circle.

Okular — More powerful:

sudo dnf install okular

Annotations, forms, signatures, multi-page viewing, DJVU support, EPUB support. KDE-developed but works fine under GNOME.

Foxit Reader — Commercial option:

flatpak install flathub com.foxit.FoxitReader

Feature-rich, fills PDF forms well. Some advanced features require paid license.
Recommendation

For simple viewing: Evince is fast and integrated. For annotations/forms: Okular has more features. Both are free and open source.
4. Photo Editors
What You Might Use
Platform	Primary Options
Windows	Photoshop, Lightroom, Paint.NET, GIMP
macOS	Photoshop, Lightroom, Pixelmator, GIMP
Linux	GIMP, Krita, Darktable, RawTherapee
Native Linux Choices

GIMP (GNU Image Manipulation Program) — Closest Photoshop alternative:

sudo dnf install gimp
# Or Flatpak with nonfree plugin (more recent):
flatpak install flathub org.gimp.GIMP

Layers, masks, filters, brushes, scripting, plugins. Steep learning curve compared to Photoshop but incredibly powerful once mastered.

Krita — Digital painting focus:

sudo dnf install krita

Designed for illustrators and concept artists rather than photo manipulation. Brush engines rival those in Photoshop. Also capable for photo editing.

Darktable — Lightroom competitor for RAW photos:

sudo dnf install darktable

RAW conversion, color grading, non-destructive workflow. Photography-focused. Excellent support for Fujifilm, Canon, Nikon DSLR files.

RawTherapee — Alternative RAW processor:

sudo dnf install rawtherapee

More technical than Darktable. Fine-grained control over demosaicing, noise reduction, sharpening. Great for enthusiasts who want complete control.
What Doesn't Translate Well

Adobe Lightroom — No direct equivalent. Darktable/RawTherapee fill the gap partially, but lack Lightroom's cloud syncing and organizational features. Some photographers use DigiKam instead:

sudo dnf install digikam

Recommendation

For photo editing: GIMP for manipulation, Darktable for RAW processing. For illustration/painting: Krita. For cataloging large photo libraries: DigiKam. These combined exceed Lightroom + Photoshop capabilities, though they demand learning investment.
5. Vector Graphics
What You Might Use
Platform	Primary Options
Windows	Adobe Illustrator, CorelDRAW, Inkscape
macOS	Adobe Illustrator, Affinity Designer, Sketch
Linux	Inkscape, Karbon, Calligra
Native Linux Choice

Inkscape — Professional SVG/vector editor:

sudo dnf install inkscape
# Or Flatpak (newer version):
flatpak install flathub org.inkscape.Inkscape

SVG-native. Comparable to Illustrator for most tasks: bezier curves, layers, text, effects, extensions. Can import/export AI files with mixed success. Export formats include PNG, EPS, PDF, DXF.

Industry-standard vector work happens in Inkscape across design teams worldwide. If you only learn one new tool: make it this one.
What About Sketch/Figma?

Sketch — macOS-only. No equivalent exists.

Figma — Runs in browser at figma.com. Works identically on Linux. No installation needed. Consider switching here anyway if collaborating.
Recommendation

Inkscape is industry-capable. Learn it thoroughly. For collaborative work, use Figma in browser.
6. Video Editing
What You Might Use
Platform	Primary Options
Windows	Premiere Pro, Camtasia, Filmora, DaVinci Resolve
macOS	Final Cut Pro, Premiere Pro, iMovie, DaVinci Resolve
Linux	DaVinci Resolve, Kdenlive, Shotcut, Olive
Native Linux Choices

DaVinci Resolve — Hollywood-grade editor (proprietary):

wget -O resolve.deb https://blackmagicdesign.com/downloads/linux/download/davinci-resolve-fusion
sudo dnf install ./resolve.deb

Free version includes nearly all features. Paid $300 studio adds neural engine, denoise, temporal noise reduction. Hardware requirements steep — needs dedicated GPU and lots of RAM. Best performance on NVIDIA due to CUDA acceleration.

Kdenlive — Open source powerhouse:

sudo dnf install kdenlive
# Or Flatpak (newer version with newer codecs):
flatpak install flathub org.kde.Kdenlive

Multi-track timeline, proxies for smooth editing, keyframes, effects, transitions. Handles 4K surprisingly well on modest hardware. KDE project but works anywhere.

Shotcut — Another solid option:

sudo dnf install shotcut

Broad format support via FFmpeg. Simpler interface than Kdenlive. Good for quick edits.

Olive — Experimental but promising:

flatpak install flathub io.olvide.Olive

Still pre-1.0, but aims to be truly professional node-based editor. Worth watching.
What About Final Cut Pro?

No equivalent. FCP is macOS-exclusive. On Linux: use DaVinci Resolve (professional tier) or Kdenlive (enthusiast tier).
Recommendation

Professionals needing broadcast delivery: DaVinci Resolve (if hardware supports it). Hobbyists/YouTube creators: Kdenlive. Both are free and can produce cinema-quality output.
7. Music Production (DAW)
What You Might Use
Platform	Primary Options
Windows	Cubase, Ableton Live, FL Studio, Logic, Reaper
macOS	Logic Pro, Ableton Live, Cubase, Reaper
Linux	Ardour, REAPER, Bitwig, Cakewalk
Native Linux Choices

Ardour — Most complete Linux-native DAW:

sudo dnf install ardour

Recording, MIDI sequencing, mixing, automation, VST/AU plugin support. Subscription model ($1/mo or lifetime purchase). Industry-used for film scoring and podcast production.

REAPER — Proprietary but affordable:

flatpak install flathub com.reaper.Reaper

60-day fully functional trial, then $60 license (personal use). Extremely efficient, scriptable, stable. Used extensively in game audio, sound design. Linux-first mindset.

LMMS — Beat-making focus:

sudo dnf install lmms

Similar to FL Studio for electronic music production. Built-in instruments, samples, patterns. Learning curve gentler than Ardour.

Bitwig Studio — Modular environment (paid):

Requires subscription at bitwig.com (not in repos)

Excellent cross-platform. Polished, innovative. Costs money but matches Ableton Live quality.
Linux Advantage: PipeWire

Fedora 44's PipeWire stack provides ultra-low-latency audio — beating Windows ASIO drivers in many test cases. Combined with JACK compatibility layer, PipeWire excels for pro audio work. Setup detailed in Chapter 11.
What Doesn't Translate Well

Logic Pro X — macOS-only. Nothing replaces its tight Apple integration.
Recommendation

Serious producers: Ardour or REAPER depending on preference. Electronic musicians: LMMS for beatmaking or invest in Bitwig. Everyone benefits from PipeWire's low latency.
8. Messaging Apps
What You Might Use
Platform	Primary Options
Windows	WhatsApp, Slack, Discord, Teams
macOS	Messages, WhatsApp, Slack, Discord
Linux	Element, Telegram, Signal, Discord
Native/Linux Choices

Element (Matrix protocol) — Open standard messaging:

flatpak install flathub im.riot.Riot
# Or desktop client:
sudo dnf install element-desktop

Decentralized (no single company controls server). End-to-end encryption. Bridges to WhatsApp, Slack, Teams, IRC, XMPP, Telegram in one app. Future-proof protocol.

Telegram — Popular instant messenger:

flatpak install flathub org.telegram.desktop

Fast, feature-rich, secret chats (encryption), channels/bots. Optional encryption unless using "Secret Chat" mode.

Signal — Privacy gold standard:

flatpak install flathub org.signal.Signal

Strongest E2EE implementation possible. Minimal metadata collection. Requires phone number but protects conversation content completely.

Discord — Gaming/community voice/text:

flatpak install flathub com.discordapp.Discord
# Or official RPM:
sudo dnf install discord

Proprietary. Excellent voice chat, screen sharing, bots. Massive user base.

WhatsApp Desktop — Mirror of mobile app:

flatpak install flathub com.whatsapp.Web

End-to-end encryption (but Meta still collects metadata). Dependency on phone connection.
Business Communication

Slack — Enterprise chat:

flatpak install flathub com.slack.Slack

Channel-based team communication, file sharing, integrations. Proprietary.

Microsoft Teams — Corporate collaboration:

flatpak install flathub com.microsoft.Teams

Clunky, resource-heavy, but necessary for organizations using Office 365.
Recommendation

Personal: Element or Signal. Communities/gaming: Discord. Work (enterprise): whatever your organization mandates (Slack, Teams). Avoid Meta-owned apps if prioritizing privacy.
9. Email Clients
What You Might Use
Platform	Primary Options
Windows	Outlook, Thunderbird, Mailbird
macOS	Apple Mail, Outlook, Spark
Linux	Thunderbird, Geary, Evolution
Native Linux Choices

Thunderbird — Mozilla's email client:

sudo dnf install thunderbird
# Or Flatpak (extensions support):
flatpak install flathub org.mozilla.Thunderbird

Open source, tabbed mail, calendar (Lightning extension), encryption (Enigmail/GPG support), themes, massive add-on library. Handles Exchange via OWL add-on (commercial).

Geary (Yelp/Yelp-geary) — GNOME-simple option:

sudo dnf install geary

Minimalist, thread-focused. Good for single-account use. Less customizable than Thunderbird.

Evolution — Outlook competitor for business:

sudo dnf install evolution

Exchange support built-in (OWM), calendars, contacts, tasks, rules, filtering. Heavy-weight but corporate-ready. Integrates with GNOME online accounts.
Webmail

Modern trend leans toward web clients: Gmail, Outlook.com, Proton Mail web interface. All work identically regardless of OS. Many users prefer browser tabs over installed mail programs.
Recommendation

Power users: Thunderbird with add-ons. Simplicity seekers: Geary. Exchange/business environments: Evolution. For privacy-conscious: Proton Mail web client.
10. Calendars
What You Might Use
Platform	Primary Options
Windows	Outlook Calendar, Google Calendar
macOS	Apple Calendar, Google Calendar
Linux	GNOME Calendar, Evolution Calendar
Native Linux Choices

GNOME Calendar (Built-in) — Simple, integrated:

sudo dnf install gnome-calendar

Online account integration (Google, Nextcloud, Outlook). Shows events inline. Clean interface. Basic reminders. Perfect for most personal use cases.

Evolution Calendar — Feature-complete:

Included with sudo dnf install evolution

Groups meetings, recurring events, delegation, time zone handling. Suitable for enterprise scheduling.
Recommendation

Use whichever you prefer. Most calendars sync via CalDAV/WebCal and remain agnostic to OS. Keep your calendar in cloud (Google/Nextcloud) so device changes don't matter.
11. Cloud Storage
What You Might Use
Platform	Primary Options
Windows	OneDrive, Dropbox, Google Drive
macOS	iCloud Drive, Dropbox, Google Drive
Linux	Nextcloud, Syncthing, rclone
Native Linux Choices

Nextcloud — Self-hosted cloud (client available):

flatpak install flathub com.nextcloud.desktopclient

Own your data. Syncs files, contacts, calendar, photos. PHP/JavaScript backend runs on any hosting provider. Replace Google/Microsoft entirely if desired.

Syncthing — Peer-to-peer synchronization:

sudo dnf install syncthing

No central server. Direct peer connections encrypted TLS. Devices sync folders bidirectionally. Ideal for laptop→desktop, home server setups.

rclone — Command-line magic:

sudo dnf install rclone
rclone config               # Set up cloud providers
rclone mount gdrive:/ ~/gdrive --daemon  # Mount drive as filesystem

Mount Amazon S3, Google Drive, Dropbox, Backblaze B2 as local filesystems. Powerful scripting, backups, transfers. CLI-only.
Official Desktop Clients
Service	Availability	Method
Google Drive	❌ Official client discontinued	rclone, web interface
OneDrive	✅ Official Linux client	onedrive package from GitHub
Dropbox	✅ Official Linux client	Available via dropbox.com
iCloud	❌ No Linux support	Workarounds unreliable
Recommendation

Enterprise/self-hosted: Nextcloud. Personal syncing between devices: Syncthing. Power users mounting multiple clouds: rclone.
12. Media Players

Covered extensively in Chapter 11. Brief recap:
Application	Purpose	Source
MPV	Video playback (best)	DNF or Flathub
Celluloid	GNOME wrapper for MPV	DNF or Flathub
VLC (Flatpak)	Universal player	Flathub
Rhythmbox	Music playback/library	Built-in
GNOME Music	Simple music browsing	Built-in

Installation commands repeated for reference:

sudo dnf install mpv celluloid vlc rhythmbox
flatpak install flathub org.videolan.VLC org.musicbrainz.Picard

Refer back to Chapter 11 for codec setup and hardware acceleration configuration.
13. Compression/Archiving
What You Might Use
Platform	Primary Options
Windows	WinZIP, WinRAR, 7-Zip
macOS	The Unarchiver, Keka, StuffIt
Linux	Archive Manager, p7zip, peazip
Native Linux Choices

Archive Manager (File Roller) (Built-in) — GNOME's archiver:

Installed by default

Create and extract ZIP, TAR, GZ, BZ2. GUI-friendly. Drag-and-drop supported.

p7zip-full — Command-line powerhouse:

sudo dnf install p7zip-full

Full 7-zip port. Supports 7Z, ZIP, RAR (read-only), CAB, ARJ, LZH, CHM. Use terminal:

7z a archive.7z folder/        # Create archive
7z x archive.7z                # Extract archive

PeaZip — Full-featured GUI alternative:

flatpak install flathub io.peazip.PeaZip

Supports 180+ formats including RAR, ZIP, TAR, 7Z, ACE, ARJ. Clean interface. Cross-platform.
Handling RAR Files

RAR archives are proprietary (WinRAR developer). Linux can read/write RAR using libarchive and unrar packages:

sudo dnf install unrar unrar-nonfree

Reading RAR requires accepting licensing terms. Installation implies acceptance.
Recommendation

Most users never notice the absence of WinRAR thanks to Archive Manager's simplicity. For command-line automation: p7zip. PeaZip bridges the GUI gap if you miss WinRAR's interface.
14. Virtual Machines
What You Might Use
Platform	Primary Options
Windows	Hyper-V, VirtualBox, VMware Workstation
macOS	Parallels, VMware Fusion, VirtualBox
Linux	GNOME Boxes, VirtualBox, KVM/QEMU
Native Linux Choices

GNOME Boxes — Simplified VMs for beginners:

sudo dnf install gnome-boxes

Intuitive wizard-driven interface. Download ISOs directly within app. Run Windows, Linux, BSD guests easily. Limited customization but extremely accessible.

VirtualBox — Oracle's mainstream hypervisor:

sudo dnf install virtualbox kernel-devel gcc make dkms

Feature-rich: snapshots, shared folders, USB passthrough, headless operation. Hosts both Linux and guest OSes. Mature, widely documented.

KVM/QEMU (libvirt/virt-manager) — Kernel-level native virtualization:

sudo dnf install qemu-kvm libvirt virt-manager
sudo systemctl enable --now libvirtd
sudo usermod -aG libvirt $(whoami)

Best performance (runs VM code in kernel space). Virt-manager provides GUI front-end. Snapshots, cloning, networking, storage pools, CPU pinning, GPU passthrough (advanced). Industry standard for production deployments.
Which Should You Choose?
Use Case	Recommendation
Occasional testing of another OS	GNOME Boxes
Development/testing with snapshots	VirtualBox or KVM
Running Windows for incompatible software	KVM with GPU passthrough (expert)
Server consolidation/hosting	KVM (production)

See Chapter 24 (Virtualization and Containers) for complete tutorial.
15. Antivirus
Reality Check

You generally don't need antivirus on Linux. Why?

    Package repositories deliver signed, verified software
    SELinux confinement restricts app damage
    No registry = malware persistence mechanisms fail
    Users aren't admins = limited blast radius
    Malware writers target Windows (95% of market)

That said, certain use cases benefit from scanning:

ClamAV — Open source scanner (for checking files sent to Windows users):

sudo dnf install clamav clamav-daemon
sudo freshclam                   # Update virus definitions
clamscan -r /home                # Scan recursively

Scan emails, network shares, or downloads meant for Windows recipients.

Sophos — Commercial option:

Download from sophos.com (not in Fedora repos)

Corporate environments with strict compliance standards.
Recommendation

Skip antivirus. Enable automatic updates and practice safe computing. If required for compliance/corporate policy: ClamAV suffices for occasional scans.
16. System Monitoring
What You Might Use
Platform	Primary Options
Windows	Task Manager, Resource Monitor
macOS	Activity Monitor
Linux	System Monitor, htop, btop
Native Linux Choices

System Monitor (Built-in) — GNOME's process/resource viewer:

Installed by default

Processes list (like Task Manager), resource graphs (CPU/memory/network/disks), file system browser. Integrated, familiar, adequate.

htop — Advanced terminal monitor:

sudo dnf install htop
htop           # Launch

Color-coded processes, tree view, searchable, mouse-supported, kill/suspend actions. Far surpasses traditional top utility.

btop — Beautiful next-gen monitoring:

sudo dnf install btop
btop           # Launch

Shows CPU, memory, disk, network, and processes in dashboard layout. Animated graphs. Highly configurable. Visually stunning.
Additional Tools

sudo dnf install iotop         # Disk I/O monitoring
sudo dnf install nethogs       # Per-process network usage
sudo dnf install powertop      # Power consumption analysis (laptops)
sudo dnf install smartmontools # Disk health (smartctl)

Recommendation

Casual monitoring: System Monitor. Terminal enthusiasts: htop. Gamers/power users wanting real-time metrics: btop. All are free and open source.
17. Terminal Emulators
What You Might Use
Platform	Primary Options
Windows	CMD, PowerShell, Windows Terminal
macOS	Terminal.app, iTerm2
Linux	GNOME Terminal, Konsole, Alacritty, Kitty
Native Linux Choices

GNOME Terminal (Built-in) — Default terminal:

Installed by default

Tabs, split panes, profiles, transparency, copy/paste shortcuts. Reliable and sufficient for most users.

Kitty — GPU-accelerated renderer:

sudo dnf install kitty
kitty              # Launch

Ultra-fast, ligature support, image display inside terminal, extensive configurability (~/.config/kitty/kitty.conf). Powershell-like shell management.

Alacritty — Fastest terminal:

sudo dnf install alacritty
alacritty          # Launch

GLSL-rendered terminal. Blazing speed. Configuration file only (no GUI settings). Minimalist power user choice.

Konsole — KDE's feature-packed emulator:

sudo dnf install konsole

Multiple tabs, split views, session management, bookmarks, history search. Works fine under GNOME too.
Shell vs. Terminal

Important distinction:

    Terminal emulator = window program (GNOME Terminal, Kitty)
    Shell = command interpreter (bash, zsh, fish)

They're separate concepts. You can run bash in GNOME Terminal, zsh in Kitty, etc. Most Linux terminals default to bash. Customize shells independently.
Recommendation

Beginners: GNOME Terminal. Performance junkies: Kitty or Alacritty. Feature lovers: Konsole. Experiment freely — none lock you in.
18. Text Editors
What You Might Use
Platform	Primary Options
Windows	Notepad++, Sublime Text, VS Code
macOS	TextEdit, VS Code, Sublime Text
Linux	GNOME Text Editor, VS Code, Vim, Emacs
Native Linux Choices

GNOME Text Editor (Built-in) — Simple modern editor:

Installed by default

Syntax highlighting for common languages, line numbers, autosave, print support. Basic IDE features without bloating. Replacement for oldgedit.

Visual Studio Code — Microsoft's flagship editor:

flatpak install flathub com.visualstudio.code
# Or official repo:
wget https://code.visualstudio.com/docs/?dv=linux64_deb
sudo dnf install code_*.deb

Extensions marketplace, Git integration, debuggers, IntelliSense, remote SSH, containers, Docker. De facto standard for developers.

Neovim/Vim — Modal editing for keyboard masters:

sudo dnf install neovim
nvim              # Launch

Steep learning curve but unbeatable once mastered. Buffer operations, macros, vimscript/lua scripting, LSP support (autocompletion). Extensive plugin ecosystem.

Emacs — Lisp-powered environment:

sudo dnf install emacs-nox
emacs              # Launch (or emacs &)

Editor + email client + calendar + RSS reader + calculator + shell + programming language. Beloved cult following. Org-mode revolutionizes note-taking/planning.
Recommendation

General-purpose coding: VS Code. Keyboard efficiency pursuit: Neovim. Everything-under-one-roof philosophers: Emacs. Casual edits: GNOME Text Editor.
19. IDEs (Integrated Development Environments)

IDEs bundle editor, compiler debugger, profiler, and project tools together.
Language/Framework	Recommended IDE	Source
Java/Kotlin	IntelliJ IDEA, Eclipse	JetBrains/DNF
C/C++	CLion, Qt Creator	JetBrains/DNF
.NET/C#	Rider, MonoDevelop	JetBrains/OpenSource
Python	PyCharm, Spyder	JetBrains/DNF
Web/Javascript	VS Code, WebStorm	DNF/JetBrains
Mobile (Flutter)	Flutter DevTools	Dart SDK
General Multi-Language	NetBeans	DNF
Jetbrains Toolbox

JetBrains products run excellently on Linux:

# Install JetBrains Toolbox App (handles product management):
https://jb.gg/toolbox-download
chmod +x jetbrains-toolbox-*
./jetbrains-toolbox-* --install

Offers IntelliJ, PyCharm, CLion, WebStorm, GoLand, RubyMine, PhpStorm, DataGrip. Free student licenses available. Paid subscriptions ($15–$20/month after first year).
Recommendation

Professional developers investing in productivity: JetBrains suite (justifiable cost). Students/open source contributors: VS Code + extensions (free). Specific frameworks (Qt, Android): check vendor recommendations.
20. Screenshots/Snip & Sketch
What You Might Use
Platform	Primary Options
Windows	Snipping Tool, Snip & Sketch
macOS	Screenshot app, Grab, Skitch
Linux	Screenshot, Flameshot, Shutter
Native Linux Choices

Screenshot Utility (Built-in) — Fedora's tool:

Press PrtSckey triggers it

Capture entire screen, selected region, active window. Delay timer supported. Save as PNG automatically. Simple and effective.

Flameshot — Annotation powerhouse:

sudo dnf install flameshot

Region selection with arrow keys, arrows, boxes, blur, text, numbering, save/upload. Configure hotkeys globally:

# ~/.config/flameshot.ini
[General]
shortcuts/ui_capture=SUPER+SHIFT+S

Assign SUPER+SHIFT+S to launch flameshot gui.
Recommendation

Quick captures: built-in tool. Annotations/sharing: Flameshot. Both fulfill needs previously served by Snipping Tool.
21. Remote Desktop
What You Might Use
Platform	Primary Options
Windows	RDP, TeamViewer, AnyDesk
macOS	Screen Sharing, TeamViewer
Linux	Remmina, RustDesk, x2go
Native Linux Choices

Remmina — Unified RDP/VNC/SSH client:

sudo dnf install remmina remmina-plugin-rdp remmina-plugin-vnc

Connect to Windows RDP servers, Linux VNC sessions, SSH tunnels. Tabbed interface saves multiple configurations. Supports plugins extending protocol coverage.

RustDesk — Open TeamViewer alternative:

flatpak install flathub com.rustdesk.RustDesk

Self-host relay server if desired. End-to-end encryption. Port forwarding not required. Growing popularity as TeamViewer gets expensive.

GNOME Remote Desktop — Built-in host/server capability:

Settings → Sharing → Screen Sharing

Enable VNC/RDP server for inbound connections. Password-protected. Integrate with login manager.
Recommendation

Connecting TO other machines: Remmina. Hosting YOUR machine remotely: GNOME built-in sharing. Wanting TeamViewer replacement: RustDesk.
22. Password Managers
What You Might Use
Platform	Primary Options
Windows/macOS	LastPass, 1Password, Dashlane, Keeper
Linux	Bitwarden, KeePassXC
Native Linux Choices

Bitwarden — Open source vault:

flatpak install flathub com.bitwarden.desktop
# Or browser extension recommended

Syncs across platforms via secure cloud (owned by password manager startup). Free tier unlimited passwords, all devices. Self-host optional for maximal control. Browser extensions excellent.

KeePassXC — Offline vault:

sudo dnf install keepassxc

Single database file (.kbdx) stored locally. No cloud dependency. Strong encryption (AES-256). Browser integration via KeePassXC-Browser extension. Manual synchronization (Dropbox/Syncthing).
Recommendation

Cloud convenience: Bitwarden (industry-trusted). Local control purists: KeePassXC. Avoid LastPass (breaches in 2022 damaged trust irreparably).
23. Note Taking
What You Might Use
Platform	Primary Options
Windows	OneNote, Evernote, Notion
macOS	Notes, Evernote, Bear
Linux	Joplin, Obsidian, CherryTree
Native Linux Choices

Joplin — Evernote replacement:

flatpak install flathub net.joplin.Joplin

Markdown notebooks, plaintext files, encryption, todo lists, sync via Nextcloud/OneDrive/Dropbox/WebDAV. Open source. Cross-platform mobile apps exist.

Obsidian — Knowledge graph builder:

flatpak install flathub md.obsidian.Obsidian

Markdown files stored locally. Links between notes create bidirectional knowledge graph. Plugin ecosystem extends functionality. Closed source but free personally.

CherryTree — Hierarchical notes:

sudo dnf install cherrytree

Nested structure for organizing ideas/code snippets/recipes. Export to HTML/pdf. Single XML file portable backup. Older but battle-tested.
Recommendation

Evernote refugees: Joplin. Think-tank builders connecting ideas: Obsidian. Structured hierarchical databases: CherryTree.
24. Project Management/Tasks

Business task managers rarely translate perfectly to Linux due to web-based deployment models.
What You Might Use
Platform	Primary Options
Windows/macOS	Trello, Asana, Monday.com, ClickUp
Linux	Same apps via web
Reality

All major project management platforms operate entirely in browsers today. Linux has no disadvantage here. Access Trello boards, Asana projects, Monday dashboards through Firefox/Chrome exactly as Windows users do.

Self-hosted alternatives exist:

Taiga — Agile/scrum platform:

docker-compose up (self-hosted setup)

Epics, sprints, backlog management. Open source. Deploy own instance.

Focalboard — Notion-style kanban:

sudo dnf install focalboard-server

MatLab-inspired board creator. Self-hosting friendly. Mattermost ecosystem.
Recommendation

Team collaboration: browser-based platforms (unchanged on Linux). Individual/hobby project tracking: Kanbans.io, Notion web version. Self-hosting enthusiasts: Taiga or Focalboard.
25. Gaming
Game Compatibility Overview

Steam plays beautifully on Linux thanks to Valve's Proton translation layer converting DirectX APIs to Vulkan. Over 20,000 games confirmed working. Anti-cheat remains limiting factor — multiplayer competitive titles often blocked.

sudo dnf install steam
# Then enable Steam Play in Settings → Steam Play → Enable for all titles

ProtonDB ranks individual games compatibility:
Rating	Description
Platinum	Perfect experience out of box
Gold	Minor tweaks needed
Silver	Moderate configuration required
Bronze	Difficult but functional
Borked	Currently broken/not playable

Check protonsdb.com before purchasing expecting Linux support.
Emulation

DOSBox, Dolphin (GameCube/Wii), PCSX2 (PS2), RPCS3 (PS3), Ryujinx (Switch), Yuzu (Switch forks), Citra (3DS), DeSmuME (DS), MAME (Arcade), RetroArch (multi-console frontend) — all available via DNF or Flatpak.

sudo dnf install dosbox dolphin-emulator pcsx2 retroarch

Old consoles emulated flawlessly on decent PCs. RetroArch unifies controllers/themes/input mapping across cores.
Game Libraries

Beyond Steam:
Service	Linux Support
Epic Games Store	✅ Heroic launcher (third-party)
GOG Galaxy	✅ Lutris + winetricks
Ubisoft Connect	✅ Lutris scripts
Battle.net	✅ Bottles + Lutris
Recommendation

Single-player games: Proton enables 80%+ library compatibility. Competitive esports requiring anti-cheat: verify beforehand or stay dual-boot. Retro gaming: emulation fantastic. Always research per-game status at protondb.com.
26. System Tweaks
What You Might Use
Platform	Primary Options
Windows	Registry Editor, CCleaner, Process Hacker
macOS	CleanMyMac, TinkerTool, MacUpdater
Linux	GNOME Tweaks, RegEdit, BleachBit
Native Linux Choices

GNOME Tweaks — Desktop configuration:

sudo dnf install gnome-tweaks

Themes, fonts, cursor themes, extensions, startup apps, windows title buttons. Covers tweaking formerly done via Windows Registry.

RegEDIT — Confusing name, unrelated to Windows Registry:

sudo dnf install regext

Rename files in bulk based on regex patterns. Unrelated to Windows registry editing. Important distinction!

BleachBit — Junk removal/anonymization:

sudo dnf install bleachbit

Clear cache, logs, temporary files, cookies, download history, clipboard contents. Open source. Safe cleaner comparable to CCleaner (which avoided on Windows for questionable practices).
Windows Registry?

Linux has no Windows Registry. System settings distributed across /etc/*, ~/.config/*, dconf (binary blob accessed via gsettings), systemd units, service configs. Each app manages its own state. Decentralization means corruption spreads nowhere — one misconfigured file breaks just that application.
Recommendation

GUI tweaker: GNOME Tweaks. File renaming masochism: RegEDIT. Cleaning cruft: BleachBit. Windows Registry fears: irrelevant on Linux. Peace achieved.
The Bottom Line: Choosing Your Toolkit

Every person adapts differently when migrating operating systems. Follow these principles:
Principles of Successful Migration

    Start with open source alternatives whenever possible — Long-term sustainability, privacy respect, community ownership.

    Accept that some software won't equal — Adobe Creative Cloud, Microsoft Office, Apple ecosystem exclusives. Have fallback plans: web interfaces, Windows VMs, Wine/Bottles, or reconsider necessity.

    Don't cling to brand familiarity blindly — Being loyal to Adobe doesn't guarantee mastery of GIMP workflows. Invest learning time; rewards compound.

    Lean on communities — Reddit threads, forums, Stack Overflow provide faster solutions than searching Google for obscure problems.

    Prioritize interoperability — Choose formats others understand (PNG/JPEG/PDF/ODT/HTML) avoiding proprietary silos trapping you into ecosystems forever.

    Test drive before abandoning old systems — Dual-boot Fedora alongside Windows during transition months. Reduce pressure while establishing new routines.

    Embrace paradigm shifts — Package managers supersede website downloads, Flatpaks sandbox safely, Wayland secures input isolation, PipeWire unifies multimedia pipelines. These aren't quirks — they're improvements.

Try It Yourself

    Audit your current applications. List every Windows/macOS program used weekly. Rank importance Critical/High/Medium/Low.

    Research Linux equivalents. Using this chapter, assign potential replacements. Flag unknowns and seek community feedback asking "Does anyone use [program]? How does it compare?"

    Install one new app category. Pick non-threatening territory (music player, note taking) and try it deeply for 3 days. Give genuine evaluation chance before dismissing unfamiliar workflows.

    Benchmark critical software. Test whether essential professional tools exist satisfactorily. If gaps discovered, evaluate compromises: alternate career paths, virtual machines, browser reliance, continuing partial Windows usage.

    Join migration discussions. Visit r/Fedora, Ask Fedora forum, Ubuntu Forums observing migration stories. Real experiences illuminate pitfalls tutorials ignore.

    Create your own equivalence map. Document winning combinations. Share back with the community helping newcomers navigate similarly.

    ✅ Chapter 12 Complete You mapped 26 software categories from Windows/macOS to Linux equivalents: web browsers (Firefox, Chrome, Brave), office suites (LibreOffice, OnlyOffice), PDF readers (Evince, Okular), photo editors (GIMP, Krita, Darktable), vector graphics (Inkscape), video editors (DaVinci Resolve, Kdenlive), music production (Ardour, REAPER), messaging (Element, Telegram, Discord), email (Thunderbird, Geary), calendars (GNOME Calendar), cloud storage (Nextcloud, Syncthing, rclone), media players (MPV, VLC), compression (Archive Manager, p7zip), virtual machines (Boxes, VirtualBox, KVM), antivirus (ClamAV — rarely needed), system monitors (htop, btop), terminals (Kitty, Alacritty), editors (VS Code, Vim, Emacs), IDEs (JetBrains products), screenshots (Flameshot), remote desktop (Remmina, RustDesk), password managers (Bitwarden, KeePassXC), notes (Joplin, Obsidian), tasks/web platforms, gaming (Steam + Proton), and system tweaks (GNOME Tweaks). Understanding tradeoffs empowers informed decisions balancing functionality, openness, cost, and comfort zones.
