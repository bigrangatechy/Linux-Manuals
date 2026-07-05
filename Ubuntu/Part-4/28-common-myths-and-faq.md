# Chapter 28: Common Myths and FAQ

## Separating Fact from Fiction

If you search "should I switch to Linux" online, you'll encounter strong opinions in every direction. Some say Linux is bulletproof and ready for everyone. Others say it's only for hackers and command-line wizards. Most of these claims are outdated, exaggerated, or simply wrong.

This chapter addresses the most common myths and frequently asked questions beginners encounter. Every answer is grounded in the reality of Ubuntu 26.04 as you've experienced it through this manual — not in 2010-era forum posts or sensational YouTube titles.

## Myth: "Linux Is Immune to Viruses"

**The truth:** Linux is not immune to malware, but it's significantly less susceptible than some other operating systems. Here's why:

**Why infections are rare on Linux desktops:**

- **Software repositories.** Most software is installed from Ubuntu's repositories, which are reviewed and signed by Canonical. You're not downloading `.exe` files from random websites.
- **Package signing.** APT verifies cryptographic signatures on every package. Tampered packages are rejected.
- **User separation.** Malware running as your user can't modify system files without your password. A compromised browser can't install a rootkit without `sudo`.
- **AppArmor.** Confined applications (Firefox, snaps) can't access files outside their sandbox even if compromised.
- **Market share.** Desktop Linux has a smaller market share, making it a less attractive target for mass-distribution malware.

**But Linux malware exists.** There are trojans, cryptominers, ransomware, and botnets targeting Linux — especially servers and IoT devices. Desktop infections are rarer, but they happen through:

- Malicious scripts disguised as legitimate commands (`curl | bash` from untrusted sources)
- Compromised PPAs or third-party repositories
- Phishing (social engineering works on any OS)
- Vulnerable services exposed to the internet (unpatched SSH, web servers)

**Bottom line:** You don't need traditional antivirus software on a Ubuntu desktop (more on this below), but you should practice good security hygiene — as covered in Chapter 27.

## Myth: "You Need the Terminal for Everything"

**The truth:** You can use Ubuntu 26.04 entirely through the graphical interface for daily tasks — browsing, email, document editing, media playback, software installation, updates, settings, backups, and system monitoring.

The **App Center** installs software graphically. The **Settings** app configures your system. **Deja Dup** handles backups with a GUI. **Timeshift** creates snapshots with a GUI. **Files** manages your documents. **Software updates** appear as notifications.

The terminal is powerful and often faster — that's why this manual teaches it extensively. But it's not required. Many Ubuntu users never open a terminal and live happily for years.

**Where the terminal genuinely helps:**

- Troubleshooting (reading logs, diagnosing network issues)
- Automation (scripts, cron jobs)
- Advanced configuration (settings not exposed in GUI)
- Remote administration (SSH)
- Batch operations (renaming 500 files, installing 20 packages at once)

Think of the terminal as a power tool: you can build a house with hand tools, but power tools make it faster. This manual teaches both because understanding the terminal makes you a more capable user — not because you're forced to use it.

## Myth: "Linux Is Only for Programmers"

**The truth:** Linux powers smartphones (Android), supercomputers (100% of the top 500), most of the internet's servers, smart TVs, routers, cars, and the International Space Station. It's not a niche tool for developers — it's the most widely deployed operating system family in the world.

On the desktop, Ubuntu is used by writers, artists, teachers, students, scientists, musicians, gamers, retirees, and people who just want a computer that works without forced restarts, telemetry, or subscription nagging. The installation process (Chapter 6) is a few clicks. The desktop (Chapter 9) is familiar to anyone who's used Windows or macOS.

Programming is one *use* of Linux. It's not a *requirement*.

## Myth: "Open Source Is Less Secure Because Anyone Can See the Code"

**The truth:** The opposite is more accurate. This is known as **Linus's Law** (named after Linux creator Linus Torvalds): "Given enough eyeballs, all bugs are shallow."

When source code is public:
- Security researchers audit it continuously
- Vulnerabilities are found and patched quickly
- No hidden backdoors can survive scrutiny indefinitely
- Fixes come from a global community, not a single vendor's timeline

Proprietary software's "security through obscurity" assumes that hiding code makes it safer. In practice, hidden vulnerabilities remain undiscovered by the vendor but are found by attackers who reverse-engineer the software. The vendor doesn't know about the hole until it's exploited.

Consider: OpenSSL (open source) powers encryption for most of the internet. The Linux kernel (open source) runs most of the world's servers. Apache and Nginx (open source) serve most of the world's websites. These aren't secure *despite* being open — they're secure *because* being open means constant review.

That said, open source is not automatically secure. Small, unmaintained projects with few contributors may have undetected vulnerabilities. The principle holds best for large, active projects — which is why sticking to well-maintained software (Ubuntu repositories, popular Flatpaks) matters.

## Myth: "You Have to Compile Software from Source"

**The truth:** You almost never compile software from source on Ubuntu. This myth comes from the 1990s and early 2000s, when package management was immature and source compilation was common.

Ubuntu 26.04 offers four package formats — all pre-compiled:

| Format | Pre-compiled? | Installation |
|---|---|---|
| APT (`.deb`) | Yes | `sudo apt install firefox` |
| Snap | Yes | `sudo snap install code` |
| Flatpak | Yes | `flatpak install flathub org.gimp.GIMP` |
| AppImage | Yes | Download and run |

Compiling from source is an advanced technique used by developers building their own software, users on unusual architectures, or people who need a specific feature not included in the packaged version. For normal use, it's irrelevant.

## Myth: "Linux Can't Play Games"

**The truth:** Linux gaming has transformed dramatically. The major catalysts:

- **Valve's Proton** (built into Steam) translates Windows games to Linux with high compatibility. Most Steam games now run on Linux out of the box.
- **The Steam Deck** runs Linux (SteamOS, based on Arch). Valve invested heavily in Linux gaming because they wanted a platform independent of Microsoft's ecosystem.
- **Native Linux games** include thousands of titles on Steam, GOG, and itch.io.
- **Emulation** (RetroArch, Dolphin, PCSX2) runs excellently on Linux.
- **Lutris** is a gaming platform that manages Wine, emulators, and native games in one interface.

**What works:** Most Steam library, indie games, older titles, emulators, browser games.

**What's challenging:** Games with invasive kernel-level anti-cheat (some multiplayer titles like Valorant, Fortnite). The anti-cheat software is designed for Windows kernel access and doesn't run on Linux. Check compatibility at [protondb.com](https://www.protondb.com) — a community database rating Windows games on Linux through Proton.

## Myth: "Snaps Are Just Bloatware"

**The truth:** Snaps are a packaging format with specific trade-offs. They're neither universally loved nor universally terrible.

**What snaps do well:**
- Sandbox applications for security (restricted file access)
- Include all dependencies (works on any Ubuntu version)
- Atomic updates (rollback if an update breaks something)
- Server/CLI tools (docker, terraform, go — clean and self-contained)

**Legitimate criticisms:**
- Larger download size (bundled dependencies)
- Slower first launch (decompression on cold start)
- The Snap Store is centralized (Canonical controls it)
- Some users dislike the integration with Canonical's infrastructure

**The practical takeaway:** Use snaps when they solve a problem. Use Flatpaks when you prefer Flathub's model. Use APT when the version in the repositories is sufficient. You don't have to pick a side — Ubuntu 26.04 supports all three simultaneously. Chapter 18 covers all of them in detail.

## Myth: "Ubuntu Spies on You"

**The truth:** Ubuntu collects minimal telemetry, and it's transparent and opt-out.

Ubuntu's data collection (if enabled) includes:
- Whether the install succeeded or failed
- Hardware profile (CPU, RAM, GPU — for compatibility testing)
- Whether you chose minimal or normal installation
- Timezone (for mirror selection)

It does **not** collect:
- Your files or documents
- Your browsing history
- Your application usage
- Your personal information

You can disable it during installation (there's a checkbox) or afterward:

sudo ubuntu-report -f send no

Compare this to commercial operating systems that collect usage telemetry, advertising IDs, search history, and voice recordings by default. Ubuntu's approach is minimal and honest.
Myth: "You Need to Reinstall Linux Regularly Like Windows"

The truth: Ubuntu doesn't accumulate the cruft that makes Windows slow down over time. There's no registry bloat, no leftover DLL files, no mysterious slowdown after months of use.

The two reasons people reinstall Windows — accumulated junk and in-place upgrade failures — don't apply to Ubuntu:

    APT and Snap clean up after themselves. sudo apt autoremove removes unused dependencies. sudo apt clean clears cached packages.
    Ubuntu upgrades in place. sudo do-release-upgrade upgrades the entire system without reinstalling. Your files, settings, and applications survive.
    Timeshift snapshots let you roll back any change that causes problems.

Many Ubuntu users have systems that have been upgraded in place across multiple LTS releases — 14.04 → 16.04 → 18.04 → 20.04 → 22.04 → 24.04 → 26.04 — without ever reinstalling. The system stays clean because the package manager tracks everything.
Myth: "Linux Doesn't Support My Hardware"

The truth: Ubuntu 26.04 supports a vast range of hardware out of the box — more than most proprietary operating systems, because most Linux drivers are built into the kernel.

What works without additional drivers:

    Most CPUs (Intel, AMD, ARM)
    Most GPUs (Intel and AMD — full open-source drivers in the kernel)
    Most Wi-Fi cards (Intel, Realtek, MediaTek, Qualcomm)
    Most printers (via CUPS — plug and print for thousands of models)
    Most USB devices (keyboards, mice, drives, cameras, audio interfaces)
    Most keyboards, mice, and monitors
    Bluetooth (most adapters)
    Webcams (UVC-compatible — nearly all modern webcams)

What may need attention:

    NVIDIA GPUs — require proprietary drivers (sudo ubuntu-drivers autoinstall). The open-source Nouveau driver works but has limited performance. Chapter 22 covers this in detail.
    Very new hardware — bleeding-edge hardware released after the kernel version in 26.04 may need a newer kernel (HWE edge kernel).
    Specialized hardware — some scientific instruments, legacy industrial devices, or niche peripherals may lack Linux drivers. Check with the manufacturer.
    Broadcom Wi-Fi — some older Broadcom chips need bcmwl-kernel-source. Ubuntu usually handles this during install if you enable proprietary drivers.

Before buying hardware, check:

    Linux Hardware Database — community-submitted compatibility reports
    Ubuntu Certified Hardware — Canonical's certified device list
    Search the model name + "Linux" or "Ubuntu"

Myth: "Free Means Low Quality"

The truth: Linux and most of its software are free in two senses: free as in freedom (you can study, modify, and share the source code) and free as in price (zero cost). Neither implies low quality.

Companies and individuals invest in open source because:

    It benefits them directly (their product runs on Linux)
    It builds reputation and expertise
    It attracts talent and contributions
    It creates standards everyone can build on

The software powering your Ubuntu system:

    Linux kernel — developed by thousands of paid engineers from Google, Intel, AMD, Red Hat, Canonical, Meta, and more
    GNOME desktop — developed by Red Hat, Canonical, SUSE, and volunteers
    Firefox — developed by Mozilla, a non-profit with hundreds of employees
    LibreOffice — developed by The Document Foundation and contributors
    Python, Rust, Go — backed by major tech companies

Free doesn't mean unsupported. It means the funding model is different — companies pay developers to work on shared infrastructure because everyone benefits, including them.
Myth: "More Terminal Commands Mean More Power"

The truth: The terminal is powerful not because of the number of commands, but because of what commands can do: combine, automate, and operate at a level of precision GUIs can't match.

Learning 50 commands isn't impressive. Understanding 10 commands deeply — and knowing how to combine them with pipes, redirection, and scripts — is far more valuable. This manual taught you the fundamentals thoroughly because mastery of fundamentals beats memorising obscure flags.

The most powerful terminal users don't know every command — they know how to find what they need (man, --help, apropos, web search) and how to compose simple tools into complex workflows.
FAQ: Frequently Asked Questions
"Do I need antivirus software on Linux?"

For most desktop Ubuntu users: no. Here's the nuanced answer:

When you don't need it:

    Desktop system used for normal activities (browsing, documents, media)
    Software installed from Ubuntu repositories, Snap, and Flathub
    Firewall enabled, updates automatic, good security habits

When you might consider it:

    You run a mail server or file server that Windows machines connect to (to scan files passing through)
    You're in a corporate environment that mandates AV on all devices
    You want to scan files downloaded from untrusted sources before sharing them with Windows users

If you want a scanner anyway:

    ClamAV (sudo apt install clamav) is a free, open-source virus scanner for Linux. It scans files on demand, not in real-time (unless configured for that).
    ClamTk is a graphical front-end for ClamAV.

sudo apt install clamav clamtk
sudo freshclam
clamtk

    💡 The Real Threat Model

    On a Linux desktop, the realistic threats aren't viruses in the Windows sense. They are:

        Social engineering (you're tricked into running something malicious)
        Vulnerable services (outdated software, exposed ports)
        Phishing (credential theft through fake websites)
        Supply chain attacks (compromised packages or PPAs)

    None of these are addressed by traditional antivirus. They're addressed by the security practices in Chapter 27: trusted sources, updates, firewall, and awareness.

"Can I run Windows applications on Linux?"

Not directly — Windows programs (.exe files) are compiled for Windows and expect the Windows runtime. But there are options:
Method	How It Works	Reliability
Wine	Compatibility layer that translates Windows API calls to Linux	Varies by application — check winehq.org
Bottles	GUI manager for Wine with preconfigured environments	Good for many apps
CrossOver	Commercial, polished Wine wrapper with support	Best paid option
Proton (Steam)	Wine-based, game-focused, built into Steam	Excellent for many games
VirtualBox/VM	Run a full Windows VM inside Linux	100% compatible, but overhead
Native alternative	Use a Linux equivalent (see Chapter 12 / Appendix C)	Best option when available

For most needs, a Linux-native alternative exists. GIMP replaces Photoshop, LibreOffice replaces Microsoft Office, OBS Studio is cross-platform, and most development tools have Linux versions. Chapter 12 and Appendix C cover software equivalents.
"Is Linux really free? What's the catch?"

There's no catch. Ubuntu is free because:

    It's built on open-source software licensed under GPL and similar licenses
    Canonical (Ubuntu's publisher) makes money from enterprise support, cloud services, and consulting — not from charging desktop users
    The community contributes development time freely

You can use Ubuntu, install it on as many machines as you want, share it, modify it, and run it forever — all legally free. Canonical offers paid support (Ubuntu Pro for enterprises) if you need SLAs and extended support, but the base system is and always will be free.
"What if I break something?"

You have multiple safety nets:

    Timeshift (Chapter 26) — Roll back the entire system to a working state in minutes
    Deja Dup / backups (Chapter 26) — Restore individual files or your entire home directory
    Live USB (Chapter 5) — Boot from the installation USB to repair or reinstall
    The terminal — Most "breaks" are configuration issues fixable by reverting a setting or reinstalling a package

The most common "breaks" and their fixes:

    Changed a gsettings value that broke the desktop → gsettings reset
    Bad extension broke GNOME → Disable from TTY: gnome-extensions disable extension-uuid
    NVIDIA driver issue → sudo apt purge nvidia-* and reinstall from TTY
    Broken package → sudo apt --fix-broken install
    Won't boot → Timeshift restore from live USB

"Can I try Linux without installing?"

Yes. Every Ubuntu ISO includes a live mode — boot from the USB and select "Try Ubuntu." You get a fully functional Ubuntu desktop running entirely from the USB and RAM. Nothing is written to your hard drive. Your Windows or macOS installation is untouched.

Test the desktop, check if your Wi-Fi and printer work, try the applications. When you're satisfied, run the installer from within the live session (or reboot and choose "Install").
"What's the difference between Ubuntu and other distributions?"

All Linux distributions share the same kernel and core utilities. They differ in:

    Package management (APT vs. pacman vs. dnf)
    Desktop environment (Ubuntu uses GNOME; others use KDE, XFCE, Cinnamon, etc.)
    Release cycle (LTS vs. rolling)
    Default software and configuration
    Target audience (beginners, power users, servers, security professionals)

Ubuntu targets mainstream desktop and server users with a balance of ease of use, stability, and modern software. Chapter 3 covers this in depth.
"Should I use LTS or the latest release?"

For most users: LTS (Long Term Support). Ubuntu 26.04 LTS has 5 years of standard support (through April 2031) and up to 10 years with Ubuntu Pro. It's stable, well-tested, and receives security patches without disruptive feature changes.

Interim releases (like 26.10) ship every 6 months between LTS releases, with newer software but only 9 months of support. They're for users who want the latest features and don't mind upgrading twice a year.

This manual focuses on 26.04 LTS because it's the right choice for beginners and most users.
"How do I update everything?"

The combined update command from Chapter 18:

# Update APT packages
sudo apt update && sudo apt upgrade && sudo apt autoremove

# Update snaps
sudo snap refresh

# Update flatpaks
flatpak update

Or the one-liner:

sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y && sudo snap refresh && flatpak update -y

Security updates happen automatically. The commands above catch everything else.
"Will my printer work?"

Almost certainly yes. Ubuntu uses CUPS (Common UNIX Printing System), which supports thousands of printers out of the box — often with better driver-less support than Windows, using standards like IPP Everywhere and AirPrint.

Plug in a USB printer or connect to a network printer, and it typically appears in Settings → Printers automatically. If not, Settings → Printers → Add Printer and search.
"Is my data safe from Microsoft/Google/Apple on Linux?"

Ubuntu doesn't send your data to any corporation. No telemetry beyond the optional install report. No advertising ID. No search history transmitted. No voice recordings. No app usage tracking.

Your files stay on your machine. Your browsing is between you and your browser (Firefox doesn't telemetry-track like some competitors). If you use cloud services (Google Drive, Nextcloud), that's your choice — Linux itself doesn't phone home.
"Do I need to defragment my disk?"

No. Linux filesystems (ext4, btrfs, XFS) are designed differently from NTFS (Windows). They allocate files intelligently and don't fragment under normal use. There's no defrag command because it's unnecessary.

If you're curious:

sudo e4frag /dev/sda2    # Check fragmentation on ext4

Fragmentation above 0-5% is normal and doesn't affect performance. You'll never need to act on it.
"Can I access my Windows files from Linux?"

If you dual-boot (Ubuntu installed alongside Windows), you can mount the Windows partition and read files:

# Find the Windows partition
lsblk -f

# Mount it (replace sda3 with your Windows partition)
sudo mount -t ntfs-3g /dev/sda3 /mnt/windows

# Browse
ls /mnt/windows/Users/YourName/Documents/

Ubuntu can also read BitLocker-encrypted partitions if you have the recovery key (install dislocker).

    ⚠️ Don't write to Windows partitions from Linux while Windows is hibernated. If Windows used "Fast Startup" (a hybrid hibernate), its filesystem is in an inconsistent state. Writing to it from Linux can corrupt the partition. Disable Fast Startup in Windows if you dual-boot.

"How do I get help when I'm stuck?"

Chapter 31 covers this in depth, but the short answer:

    man command — Built-in manual pages
    command --help — Quick usage reference
    Ask Ubuntu (askubuntu.com) — Q&A community
    Ubuntu Forums (ubuntuforums.org) — Discussion community
    Reddit (reddit.com/r/Ubuntu) — Community support
    Ubuntu Community Hub (discourse.ubuntu.com) — Official forums
    Search engines — Most problems have been solved and posted

The key skill is knowing what to search for. Describe the symptom, include "Ubuntu 26.04," and include the exact error message in quotes.
"What does 'sudo' actually stand for?"

SuperUser DO. It temporarily elevates your privileges to root (the system administrator) for a single command. It's not magic — it logs who ran what, when, and requires your password to prevent unauthorized use.

In Ubuntu 26.04, sudo itself is written in Rust — a memory-safe language that eliminates entire classes of security vulnerabilities that C-based tools were susceptible to.
"Can I customize Linux to look like Windows or macOS?"

Yes. Ubuntu's desktop is highly customizable (Chapter 25):

    Windows-like: Install ArcMenu extension (traditional Start menu), move dock to bottom, disable activities overview
    macOS-like: Move dock to bottom, center it, enable Blur My Shell, install WhiteSur or MacTahoe theme, use a global menu extension

Or install a different desktop environment entirely:

    XFCE (Xubuntu) — lightweight, traditional panel layout
    KDE Plasma (Kubuntu) — highly customizable, covered in Chapter 9.5

Debunking in One Line
Myth	Reality
"Linux has no viruses"	Rare, not impossible. Practice good security.
"You need the terminal"	Helpful but optional for daily desktop use.
"Only for programmers"	Used by millions of non-programmers worldwide.
"Open source is insecure"	Open code means more eyes finding bugs faster.
"Must compile from source"	Almost never. Four package formats, all pre-built.
"Can't play games"	Proton runs most Steam games. Check protondb.com.
"Snaps are bloatware"	Trade-offs. Use what fits. They're all valid.
"Ubuntu spies on you"	Minimal, transparent, opt-out telemetry.
"Need to reinstall yearly"	Systems upgrade in place for decades. No registry rot.
"Hardware won't work"	Vast out-of-box support. Check certification.ubuntu.com.
"Free = low quality"	Funded by companies that depend on it.
"Need antivirus"	Usually no. Practice security habits instead.
"Can't run Windows apps"	Wine/Proton/Bottles for many. VM for the rest.
What's Next

Myths fade with experience. The more you use Ubuntu, the more you'll recognise these claims for what they are — outdated assumptions, oversimplifications, or genuine concerns with nuanced answers. You now have the knowledge to evaluate Linux claims critically and explain the reality to others.

In Chapter 29, we'll cover troubleshooting — a systematic methodology for diagnosing and fixing problems when they arise, from boot failures to network issues to package conflicts.
Try It Yourself

    Check a game's compatibility. If you have a Steam game you play on Windows, search for it at protondb.com. See how it runs on Linux through Proton.

    Try a live session. If you have your Ubuntu USB from Chapter 5, boot into "Try Ubuntu" mode. Explore the desktop without touching your installed system. This is what prospective users experience.

    Install ClamAV. Run sudo apt install clamav and sudo freshclam. Scan a directory: clamscan -r ~/Downloads. See for yourself what a Linux virus scanner looks like (spoiler: it's not very exciting because there's rarely anything to find).

    Test Wine. Install sudo apt install wine. Download a simple Windows application (like a small utility or Notepad++) and try running it with wine program.exe. It won't always work perfectly, but it's instructive to see the translation layer in action.

    Share your knowledge. Pick one myth from this chapter that you believed before starting this manual. Write a few sentences explaining why it's wrong, based on your own experience. This is how myths get replaced — person by person, with real understanding.

    Bookmark resources. Bookmark these sites for future reference: protondb.com (game compatibility), winehq.org (app compatibility), certification.ubuntu.com (hardware), askubuntu.com (community help).

    ✅ Chapter 28 Complete You can now distinguish Linux myths from reality — understanding why malware is rare but not impossible, why the terminal is optional, why open source is secure, how gaming works on Linux, what snaps actually are, how Ubuntu handles telemetry, why reinstallation is unnecessary, hardware compatibility reality, and answers to the most common beginner questions. In Chapter 29, we'll build a systematic troubleshooting methodology.

