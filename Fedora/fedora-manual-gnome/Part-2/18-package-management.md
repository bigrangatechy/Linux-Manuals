# Chapter 18: Package Management — DNF5 and Flatpak from the Command Line

## Software Management Beyond GNOME Software

In Chapter 10, you explored GNOME Software — the graphical application store that makes installing software as simple as clicking a button. It's elegant, user-friendly, and perfect for casual use.

But what if you need to:
- Install fifty packages at once for a fresh system?
- Script an automated setup across multiple machines?
- Remove a stubborn dependency chain without leaving orphaned data?
- Search among thousands of packages for specific capabilities?
- Understand EXACTLY what files a package installs?
- Verify signatures, check provenance, or audit installed versions?

GNOME Software won't help you here. For power, precision, and automation, you need the terminal — specifically **DNF5** (the fifth-generation DNF package manager) and **Flatpak** (the universal Linux app format).

This chapter teaches you both from first principles. You'll understand how Fedora distributes software, why there are two package formats, when to use each, and the complete command-line workflow for installing, updating, searching, querying, and managing every piece of software on your system.

---

## Table of Contents

| Topic | Section |
|---|---|
| Package Management Fundamentals | 1 |
| DNF5: Fedora's Native Package Manager | 2 |
| DNF5 Core Operations | 3 |
| Package Dependencies and Groups | 4 |
| Repository Management | 5 |
| RPM Packages Deep Dive | 6 |
| Flatpak Command-Line Guide | 7 |
| DNF5 vs. Flatpak: When to Use Which | 8 |
| Troubleshooting Package Issues | 9 |
| Automating Package Installation | 10 |

---

## 1. Package Management Fundamentals

### Why Package Managers Exist

Before Linux had package managers, software distribution was manual hell:

1. Download source code
2. Read documentation
3. Run `./configure`
4. Run `make`
5. Run `make install`
6. Hope libraries exist
7. Hope everything compiles
8. Hope nothing breaks

Now imagine maintaining sixty installed programs this way. Updating one might require rebuilding dependencies from scratch. Removing one leaves orphaned files. Installing a missing library causes cascading failures.

Package managers solve this by packaging everything into **precompiled, pre-tested, self-documenting bundles** that:
- Declare their own requirements (dependencies)
- Track every file they install
- Maintain metadata about versions and changelogs
- Handle upgrades cleanly
- Support rollback/recovery

On Fedora, the **RPM Package Manager** (originally Red Hat Package Manager) stores packages in `.rpm` format. **DNF5** (Dandified YUM, version 5) is the modern tool that manages RPM packages from the command line.

### The Package Ecosystem Landscape

Linux has evolved beyond a single package format. Modern distributions support multiple coexisting systems:

| Format | Extension | Pros | Cons | Fedoral Usage |
|---|---|---|---|---|
| **RPM (DNF)** | `.rpm` | Fast, native, tightly integrated, signed | Distribution-specific, older apps | System libraries, core tools |
| **Flatpak** | N/A | Universal across distros, sandboxed, recent apps | Larger size, slower startup | Applications (Firefox, LibreOffice, games) |
| **Snap** | N/A | Universal, auto-updates | Canonical-controlled, no Fedora default | None (not recommended on Fedora) |
| **AppImage** | `.AppImage` | Portable, no installation required | Not in repository, manual management | Occasional standalone apps |

Fedora prioritizes **RPM + Flatpak**:
- DNF handles system packages, libraries, kernels, services
- Flatpak handles desktop applications (browser, office, media)
- Both work alongside each other seamlessly

> 💡 **Why Multiple Formats?**
>
> RPM packages have been tied to Fedora since its inception. But RPMs create dependencies between apps and the OS version. An app built for Fedora 44 might break on Fedora 45. Flatpak solves this by bundling ALL dependencies inside the app container — it runs identically regardless of host OS.

### Metadata and Caches

When you run DNF commands, it uses locally cached metadata downloaded from remote repositories. Understanding this cache helps troubleshoot problems:

```bash
# View current metadata age:
sudo dnf makecache --refresh    # Force refresh
dnf repolist                    # Show repo status

# Cache location:
ls /var/cache/dnf/
# dnf/librepo_cache.db, rpmdb.sqlite, ...

# Clear cache:
sudo dnf clean all              # Wipe all caches

    ⚠️ Cache Staleness

    If DNF reports "package not found" but the package definitely exists upstream, your cache may be outdated. Refresh with sudo dnf makecache or dnf update. The metadata download usually completes in seconds on broadband.

2. DNF5: Fedora's Native Package Manager
What Changed in DNF5?

Fedora 44 ships with DNF5 — a significant rewrite from DNF4 (used in earlier releases):
Aspect	DNF4	DNF5
Language	Python	C++
Speed	Moderate (interpreter overhead)	~5× faster
Memory	Higher consumption	Lower footprint
API	Plugin-based	Cleaner modular design
Commands	Some inconsistencies	Standardized syntax
Progress bars	Basic	Enhanced visual feedback

Most importantly: DNF5 is backwards-compatible. If you've used DNF before, your muscle memory transfers directly. Only edge cases diverge.
Verifying Your DNF Version

dnf --version
# 5.x.x
# fedoraproject.org

which dnf
# /usr/bin/dnf

rpm -q dnf
# dnf-5.x.x.fc44.noarch

If you're seeing version 4.x, you're not yet on Fedora 44 — or something prevented the upgrade. DNF5 became standard with Fedora 41.
DNF Configuration Files

Main configuration lives in /etc/dnf/dnf.conf:

cat /etc/dnf/dnf.conf

[main]
gpgcheck=1              # Verify package signatures
installonly_limit=3     # Keep 3 kernel versions
clean_requirements_on_remove=True  # Auto-remove deps when removing pkg
skip_if_unavailable=False          # Fail if repo unavailable
fastestmirror=True                # Mirror selection optimization
max_parallel_downloads=10         # Parallel downloads
metadata_expire=6h                # Cache validity period
sslverify=1                       # HTTPS verification

Repository-specific settings live in /etc/yum.repos.d/ (yes, still called yum despite being DNF):

ls /etc/yum.repos.d/
# fedora.repo  fedora-updates.repo  fedora-modular.repo  updates.repo

These define where DNF fetches packages from. We'll explore them more in Section 5.
DNF Command Help

# General help:
dnf help

# List available commands:
dnf list commands

# Help for a specific subcommand:
dnf help install
dnf help remove

# Tab completion works!
dnf ins<TAB>    # Completes to: dnf install

DNF supports tab completion out of the box. Install the plugin for enhanced autocomplete:

sudo dnf install dnf-plugins-core dnf-utils

3. DNF5 Core Operations
Installing Packages

Basic installation:

# Install a single package:
sudo dnf install htop

# Interactive confirmation prompt appears; type y/Y

Non-interactive (for scripting):

sudo dnf install -y htop    # Automatically accept

Install multiple packages simultaneously:

sudo dnf install git vim curl wget tree net-tools

Installing specific versions (rarely needed):

# Find available versions:
dnf list gimp --showduplicates

# Install specific version:
sudo dnf install gimp-2.10.34-1.fc44

DNF automatically resolves and installs all required dependencies. Try to uninstall a library DNF thinks is required — it will warn you about dependent packages before proceeding.
Removing Packages

# Remove a single package:
sudo dnf remove htop

# Check what DNF plans to remove:
dnf remove htpool    # Simulate without executing (-n = dry-run)

# Remove package AND unneeded dependencies:
sudo dnf autoremove

# Verify removed:
rpm -q htop        # Outputs: "package htop is not installed"
which htop         # No output

    ⚠️ Removing Core Packages

    Avoid removing critical packages like gnome-shell, kernel, systemd, or base libraries. DNF protects against accidental mass-removals with interactive prompts, but forcing (-y) can damage your system:

    sudo dnf remove gnome-shell -y    # Destroys GUI login
    sudo dnf remove systemd -y        # Breaks boot process

    Be paranoid about what you're deleting.

Updating the System

# Check available updates (don't apply):
dnf check-update

# Apply all updates:
sudo dnf update     # Or: dnf upgrade (synonym)

# Update specific package only:
sudo dnf update firefox

# Clean up after update (remove obsolete packages):
sudo dnf autoremove

Kernel updates trigger GRUB regeneration. Reboot is REQUIRED for kernel changes to take effect:

# After kernel update:
uname -r       # Current running version
rpm -qa kernel # Installed versions (may show multiple)
reboot         # Activate newest kernel

Verify running kernel:

uname -r
# 6.19.0-50.fc44.x86_64

# Check all installed kernels:
rpm -qa kernel | sort -V
# kernel-6.19.0-50.fc44.x86_64
# kernel-6.19.0-55.fc44.x86_64   ← latest

Keep multiple kernels as rollback insurance. Fedora retains three by default (installonly_limit=3 in /etc/dnf/dnf.conf).
Searching Packages

Find packages matching keywords:

# Simple search:
dnf search video

# Exact name match (faster, more precise):
dnf provides */vscode
# Output: .../vscode-release.x86_64 : Microsoft Visual Studio Code

# By capability/file path:
dnf provides /usr/bin/vim
# vim-minimal-9.x.fc44.x86_64 : Minimal Vim editor
# vim-enhanced-9.x.fc44.x86_64 : Advanced Vim editor

# Filter results:
dnf search video | grep -i player
dnf search python | grep -i framework

Advanced search options:

# Search by summary/description:
dnf search description="database client"

# Search by tag/category:
dnf search @multimedia      # All multimedia packages

# Case-insensitive, substring:
dnf search ffmpeg           # Matches ffMPEG, FFmpeg, etc.

# Full-text search index:
dnf search fulltext "video editing"

Listing packages:

# List all available packages (massive output):
dnf list available | head -20

# List installed packages:
dnf list installed | wc -l
# Typically 1500-3000 packages installed on a Fedora system

# Show package info without installing:
dnf info firefox

View detailed package information:

dnf info vlc

Name         : vlc
Epoch        : 3
Version      : 3.0.20
Release      : 2.fc44
Architecture : x86_64
Size         : 26M
Source       : vlc-3.0.20-2.fc44.src.rpm
Summary      : The Swiss Army knife of multimedia playback
URL          : https://www.videolan.org/vlc/
License      : GPL-2.0-or-later
Description  : VLC media player is a free and open source cross-platform
             : multimodal multimedia player and framework...
Dependencies : libavcodec, libpostproc, pulseaudio, ...
Provides     : vlc(x86-64) = 3:3.0.20-2.fc44
Conflicts    : vlc-plugins-extra < 3.0.0

The "Provisional" line shows what the package offers (name/version combo). The "Conflicts" line warns about incompatible packages.
Package History and Rollback

DNF tracks all transaction history:

# View recent transactions:
dnf history

# Detailed view of transaction ID 42:
dnf history info 42

# Undo (rollback) a transaction:
sudo dnf history undo 42

# Redo (restore) a previously undone transaction:
sudo dnf history redo 42

# List all actions by a user/package:
dnf history userlogin john
dnf history packageinfo firefox

Transaction IDs increment monotonically. Undo operations themselves get recorded as separate transactions. You can roll back to any previous state — including reverting a rollback.
4. Package Dependencies and Groups
Understanding Dependencies

Packages rarely exist in isolation. A complex application like LibreOffice depends on dozens of shared libraries. DNF ensures these dependencies are satisfied automatically:

# See required dependencies before installing:
dnf deplist libreoffice-writer

Output:

Package: libreoffice-writer-24.8.0-1.fc44.x86_64
  dependency                             provider
  -------------------------------------- ------
  libc.so.6()(64bit)                     glibc-2.39.fc44.x86_64
  libgtk-3.so.0()(64bit)                 gtk3-3.24.41.fc44.x86_64
  libxml2.so.2()(64bit)                  libxml2-2.12.5.fc44.x86_64
  ... (dozens more)

When you dnf install libreoffice-writer, DNF pulls in ALL dependencies. Removing libreoffice-writer doesn't remove those libraries — other packages might need them. Run dnf autoremove periodically to identify truly orphaned dependencies.
Group Installation

Fedora organizes packages into meta-groups. Installing a group installs ALL member packages at once:

# List available groups:
dnf group list

# Install entire group:
sudo dnf group install "Development Tools"

# Alternative abbreviated form:
sudo dnf groupinstall Development\ Tools

Common groups:
Group	Purpose	Includes
Development Tools	Compilation environment	gcc, gdb, make, automake, kernel-devel
C Development Tools and Libraries	C/C++ development	clang, cmake, valgrind, qt5
Security Tools	Security auditing	wireshark, nmap, tcpdump, openssl-dev
Multimedia Playback	Media codecs	ffmpeg libs, GStreamer plugins, MP3/MKV support
Server Infrastructure	Server utilities	httpd, postfix, mariadb, iptables
Graphical Administration Tools	GUI admin	system-config-* tools, Cockpit web console
Fonts	Typography	Liberation fonts, Noto fonts, fontconfig

View group contents before installing:

dnf group info "Development Tools"

Remove group:

sudo dnf group remove "Development Tools"

Customizing Package Lists

Sometimes you want subsets of groups:

# List optional packages in group:
dnf group info "Development Tools" --verbose

# Install specific subset:
sudo dnf groupinstall "Development Tools" \
  --with-optional=gcc-gfortran \
  --without=debugging-tools

    💡 Groups vs. Personal Preference

    Groups exist for consistency in server deployments. On personal workstations, installing individual packages avoids unnecessary clutter. Use groups liberally for servers; selectively for desktops.

5. Repository Management
What Is a Repository?

A repository ("repo") is a collection of packages hosted on a server with accompanying metadata. DNF fetches from multiple repos configured in /etc/yum.repos.d/:

cat /etc/yum.repos.d/fedora.repo

[fedora]
name=Fedora $releasever - $basearch
enabled=1                           # Include this repo
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
type=rpm-md                         # Repo format
cost=1                              # Priority (lower = preferred)
exclude=*-test                      # Exclude test packages
gpgcheck=1                          # Verify signature
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-primary

[similar format for other repos...]

Variables expanded at runtime:

    $releasever → 44 (current Fedora version)
    $basearch → x86_64 (your architecture)

Enable/Disable Repositories

Check active repos:

dnf repolist

Enable a disabled repo:

sudo dnf config-manager --set-enabled epel

Disable a repo:

sudo dnf config-manager --set-disabled docker-ce-stable

Temporarily skip a repo during one operation:

sudo dnf install some-package --nodocs --enablerepo=\* --disablerepo=docker-ce-stable

Add Third-Party Repositories

Fedora's official repos cover most needs. Occasionally you'll need external sources:
Example: Adding VSCode's Official Repository

# Import Microsoft's signing key:
sudo rpm --import https://packages.microsoft.com/gpg/pgp/key.asc

# Create repo file:
sudo tee /etc/yum.repos.d/vscode.repo > /dev/null <<EOF
[code]
name=Visual Studio Code
baseurl=https://packages.microsoft.com/yumrepos/vscode
enabled=1
gpgcheck=1
gpgkey=https://packages.microsoft.com/gpg/pgp/key.asc
EOF

# Refresh cache:
sudo dnf makecache

# Now install vscode:
sudo dnf install code

Example: Enabling RPM Fusion (Codecs/Proprietary Software)

# Free repo (legal everywhere):
sudo dnf install https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-\$releasever.noarch.rpm

# Nonfree repo (contains patented/proprietary codecs):
sudo dnf install https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-\$releasever.noarch.rpm

# Now install things like NVIDIA drivers, Steam, Spotify:
sudo dnf install spotify steam nvidia-driver

RPM Fusion is essential for multimedia users needing MP3/H.264 decoding. Always verify third-party repos are trustworthy before importing.
Repackage Updates with Local Mirroring

For air-gapped systems or bandwidth conservation, mirror repos locally:

# Install tools:
sudo dnf install createrepo-c

# Mirror specific packages:
mkdir ~/fedora-mirror && cd ~/fedora-mirror
rsync -av rsync://ftp.iij.ad.jp/pub/linux/Fedora/releases/\$releasever/Everything/x86_64/os/ ./

# Create local repo:
createrepo_c .

# Configure local repo in /etc/yum.repos.d/:
tee /etc/yum.repos.d/local-fedora.repo > /dev/null <<EOF
[fedora-local]
name=Local Fedora Mirror
baseurl=file:///home/user/fedora-mirror
enabled=1
gpgcheck=0
EOF

6. RPM Packages Deep Dive

While DNF handles high-level tasks, RPM (Red Hat Package Manager) operates at the lower level. Knowing RPM helps diagnose DNF issues and inspect package internals.
Installing Raw RPM Files

Bypass DNF entirely:

# Download raw RPM:
wget https://example.com/software-1.0-1.fc44.x86_64.rpm

# Install with dependencies checked:
sudo rpm -ivh software-1.0-1.fc44.x86_64.rpm
# -i = install
# -v = verbose
# -h = hash progress bar

# Upgrade existing package:
sudo rpm -Uvh software-1.0-2.fc44.x86_64.rpm

Force installation ignoring conflicts:

sudo rpm -ivh --nodeps software-1.0-1.fc44.x86_64.rpm   # Dangerous!

Only do this if you understand consequences — skipping dependency checks risks broken installations.
Querying RPM Database

Installed packages query:

# List all installed packages:
rpm -qa | wc -l

# Search specific package:
rpm -q firefox
# firefox-126.0-1.fc44.x86_64

# Info about installed package:
rpm -qi firefox

View what files belong to a package:

# List all files provided:
rpm -ql firefox | head -20
# /usr/bin/firefox
# /usr/share/applications/org.mozilla.firefox.desktop
# /usr/share/icons/hicolor/48x48/apps/firefox.png
# ...

# Check owner of specific file:
rpm -qf /usr/bin/firefox
# firefox

Verify package integrity:

# Detect tampered/corrupted files:
sudo rpm -Va
# Only outputs changed files; silent means unchanged

Format codes for customized queries:

# Install scriptlets executed:
rpm -qp --scripts example.rpm

# Package changelog:
rpm -q --changelog firefox | head -30

# Package requires/provides/conflicts:
rpm -qp --requires example.rpm
rpm -qp --provides example.rpm
rpm -qp --conflicts example.rpm

Signing and Verification

All Fedora RPMs carry cryptographic signatures verified via GPG keys. Validate manually:

# Verify signature on local RPM:
rpm -K firefox-126.0-1.fc44.x86_64.rpm
# firefox-126.0-1.fc44.x86_64.rpm: digests signatures OK

# Check GPG key fingerprints:
rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n' | less

If a signature fails validation, DNF aborts the installation with an error message. Never ignore failed verifications.
7. Flatpak Command-Line Guide

Flatpak complements DNF rather than competing with it. While DNF manages OS integration, Flatpak manages applications.
Installing Flatpak Runtime

Flatpak comes pre-installed on Fedora Workstation. Verify:

flatpak --version
# 1.15.x

# Check if Flatpak support is enabled:
flatpak --help-all | grep -i enable

First-time setup:

# Add Flathub (primary Flatpak repository):
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Grant permissions:
flatpak permission-set system.default-dbus yes

Installing Applications

Browse available Flatpak apps:

# List remote references:
flatpak search telegram

# Install:
flatpak install flathub org.telegram.desktop

# Confirm:
org.telegram.desktop (application) from flathub

# Accept silently:
flatpak install -y flathub org.telegram.desktop

Search categories:

flatpak search video
flatpak search music
flatpak search games

# More precise:
flatpak search --columns=name,application,recommends "electron-based"

Managing Installed Flatpaks

List installed Flatpaks:

flatpak list
# Name                                  Application ID                            Version   Branch
# Telegram Desktop                      org.telegram.desktop                      4.8.1     stable
# OBS Studio                            io.obsproject.obs-studio                  30.0.2    stable

# With details:
flatpak list --appstream

Update Flatpaks:

# Check for updates:
flatpak update

# Apply all:
flatpak update -y

# Specific app only:
flatpak update org.telegram.desktop

Remove unused Flatpaks:

# Uninstall application:
flatpak uninstall org.telegram.desktop

# Prune dangling runtimes/languages:
flatpak uninstall --unused

Run applications:

# Launch normally:
flatpak run org.telegram.desktop

# Pass arguments:
flatpak run org.gimp.GIMP /home/user/photo.jpg --batch-mode

Permissions and Sandboxing

Flatpak isolates apps using namespaces. Default permissions deny most filesystem access except ~/Documents, ~/Downloads, etc.

Grant explicit permissions:

# Allow access to home directory:
flatpak override --user --filesystem=home org.telegram.desktop

# Access specific folder:
flatpak override --user --filesystem=~/projects org.gimp.GIMP

# Grant network access (usually allowed by default):
flatpak override --user --socket=x11 io.qtcreator.qtc

Revoke permissions:

flatpak override --unset --filesystem=home org.some-app

# Reset all overrides:
flatpak override --reset org.some-app

View overrides:

flatpak override --show

Flatpak Overrides (Configuration Persistence)

Override configurations survive across updates:

# Set environment variable:
flatpak override --env=GTK_THEME=Adwaita org.gnome.Nautilus

# Disable hardware acceleration:
flatpak override --env=DISABLE_VULKAN_RENDER=1 org.kde.Krita

Apply globally (all users):

sudo flatpak override --filesystem=/srv org.backup-app

8. DNF5 vs. Flatpak: When to Use Which
Decision Matrix
Requirement	Use DNF	Use Flatpak
Install system service (httpd, sshd)	✓	✗
Update kernel	✓	✗
Install development tools (gcc, rustc)	✓	Possible but heavier
Browser application (Firefox, Chrome)	Either	Preferred (isolated)
Office suite (LibreOffice, OnlyOffice)	Either	Preferred (newer versions)
Multimedia codecs	✓ (via RPM Fusion)	Yes
Proprietary software (Spotify, Zoom)	Flatpak	Often only choice
Cross-distro compatibility	✗	✓
System integration (dbus, udev rules)	✓	Limited
Frequent updates (weekly/monthly)	✓	✓ (automatic)
Minimal disk space usage	✓	Heavier (duplicate libs)
Hybrid Approach: Best of Both Worlds

Modern Fedora workflows leverage both:

# System layer via DNF:
sudo dnf install python3 git build-essential

# Applications via Flatpak:
flatpak install flathub org.spotifypublish.Spotify
flatpak install flathub com.discordapp.Discord

# Integration layers:
flatpak install flathub org.freedesktop.Platform
flatpak install flathub org.gtkGLEditor

Both methods install side-by-side. No conflict between DNF and Flatpak managed apps.
Performance Comparison

Loading time differences:
App Type	First Run	Subsequent Runs	Startup Delay
DNF Native	Instant	Instant	Negligible
Flatpak	Slower (namespace spawn)	Fast	~0.5-2 seconds

Disk space tradeoffs:
Method	Single App Storage	Shared Libraries	Overhead per User
DNF	Optimized	Shared system-wide	Minimal
Flatpak	Bundled libs included	Deduplicated per runtime	Higher initially

Net result: DNF for efficiency; Flatpak for portability and freshness.
9. Troubleshooting Package Issues
Dependency Conflicts

Symptom: Package refuses to install due to conflicting requirements.

Resolution strategy:

# Diagnose dependency graph:
sudo dnf distro-sync         # Align packages with release version

# Identify problematic package:
sudo dnf repoquery --depends firefox
# Shows all items required

# Check what provides conflicting dependency:
dnf whatprovides "libxyz.so.1()"
# Returns potential providers

# Install alternate provider:
sudo dnf install alt-provider

Broken transaction repair:

# Fix broken database:
sudo dnf reinstall rpm
sudo rpm --rebuilddb

# Restore transaction logs:
sudo cp /var/lib/dnf/history.2026-07-01T* /var/lib/dnf/history.sql

Corrupt Package Cache

Clear stale metadata:

sudo rm -rf /var/cache/dnf/*
sudo dnf makecache

# Also clear RPM DB cache:
sudo rpm --rebuildindex

Network proxy/authentication issues:

# Configure HTTP proxy for DNF:
echo "proxy=http://proxy.example.com:8080" | sudo tee -a /etc/dnf/dnf.conf

# Proxy authentication:
echo "proxy_username=user" | sudo tee -a /etc/dnf/dnf.conf
echo "proxy_password=pass" | sudo tee -a /etc/dnf/dnf.conf  # Not ideal security!

Flatpak-specific failures:

# Connection timeout:
flatpak remote-modify --update-metadata-interval=60 flathub

# Permission denied errors:
flatpak repair --system

# Outdated runtime mismatch:
flatpak update --appstream
flatpak repair

Signature Verification Failures

Symptom: "GPG key not imported" or "signature invalid."

Resolution:

# Import missing key:
sudo rpm --import URL_TO_KEY

# Check which keys are trusted:
rpm -qa gpg-pubkey | xargs rpm -qi

# Rotate expired keys:
sudo dnf update rpm-gpg-key-fedora

Never disable GPG checking — it's a critical security measure preventing compromised package injection.
10. Automating Package Installation

Kickstart provisioning for reproducible environments:
Kickstart File Example

Create /root/kickstart-packages.ks:

%packages
@^workstation-product-environment
@development-tools
@generic-editor-and-terminals
firewall-config
policycoreutils-python-utils
vim-enhanced
git
curl
wget
htop
neofetch
%end

Execute kickstart:

sudo dnf kickstart -e /root/kickstart-packages.ks

Alternatively, list package names in a text file:

# packages.txt contents:
git
vim
curl
wget
firefox
libreoffice-writer

# Bulk install:
xargs -a packages.txt sudo dnf install -y

Script installation:

#!/bin/bash
PACKAGES=(
    git
    python3-pip
    nodejs
    cargo
    gcc
    make
    qemu-kvm
    virt-manager
)

for pkg in "${PACKAGES[@]}"; do
    echo "Installing $pkg..."
    sudo dnf install -y "$pkg" || { echo "FAILED: $pkg"; exit 1; }
done

echo "Setup complete!"

Make executable:

chmod +x provision.sh
./provision.sh

Automate Flatpak installation similarly:

flatpak_install_list=(
    org.mozilla.firefox
    com.spotify.Client
    org.whatsapp.Web
)

for app in "${flatpak_install_list[@]}"; do
    flatpak install -y flathub "$app" || echo "Failed: $app"
done

    💡 Infrastructure as Code Philosophy

    Scripts documenting exact package lists enable reproducibility. Store these in version control alongside infrastructure-as-code tools like Ansible or Puppet for enterprise-grade provisioning.

Try It Yourself
Exercise 1: Fresh Package Discovery

# Find and install a utility you don't own:
dnf search pdf
# Choose one (poppler, evince, okular)

# Preview before installing:
dnf info poppler
dnf provides */pdfinfo   # Verify what capability you gain

# Install interactively:
sudo dnf install poppler

# Verify success:
pdfinfo --version
which pdfinfo

Exercise 2: Transaction History Exploration

# Review package history:
dnf history

# Select a transaction ID:
SELECTED_ID=$(dnf history | awk '{print $1}' | tail -1)

# Inspect details:
dnf history info $SELECTED_ID

# Simulate undo:
dnf history undo $SELECTED_ID  # Don't confirm unless you mean it

# Check rollback availability:
dnf history undo $SELECTED_ID | grep -i "confirm"

Exercise 3: Repository Manipulation

# Enumerate active repositories:
dnf repolist

# Temporarily skip updates-repo for one install:
sudo dnf install htop --disablerepo=updates

# Verify repo was honored:
dnf install htop -vv | grep -i "repo"

# Re-enable (normal install restores defaults):
sudo dnf install htop

Exercise 4: Flatpak Sandbox Inspection

# Find currently installed Flatpak apps:
flatpak list | wc -l

# Inspect permissions for one:
flatpak info org.gimp.GIMP | grep -A 20 "Permission"

# Temporarily grant additional access:
flatpak override --user --filesystem=~/Projects org.gimp.GIMP

# Test accessibility from within Flatpak:
flatpak run --command=bash org.gimp.GIMP
# Inside shell: ls ~/Projects  <-- Should see mounted

# Remove override afterward:
flatpak override --unset --filesystem=~/Projects org.gimp.GIMP

Exercise 5: RPM Low-Level Commands

# Query Firefox installation:
rpm -q firefox
rpm -qi firefox | head -30

# List files contributed:
rpm -ql firefox | grep bin
rpm -ql firefox | grep share

# Locate origin of a file:
file=$(find /usr -name "nautilus" -type f 2>/dev/null | head -1)
rpm -qf "$file"

# Verify package hasn't been modified:
sudo rpm -Va | grep -i fire
# Silent output = integrity maintained

Exercise 6: Provisioning Script Practice

# Create your personal setup script:
nano ~/my-setup.sh

# Insert:
#!/bin/bash
sudo dnf install -y vim git curl wget jq ansible flatpak

flatpak install -y flathub org.mozilla.firefox com.discordapp.Discord

# Save and make executable:
chmod +x my-setup.sh

# Run (on test VM preferably):
./my-setup.sh

Troubleshooting Reference
Symptom	Probable Cause	Resolution
package not found	Cache stale	sudo dnf makecache then retry
dependency conflict	Version incompatibility	sudo dnf distro-sync or install compatible alternatives
cannot delete package	Dependent packages remain	sudo dnf remove dependents; sudo dnf remove target
GPG signature bad	Key rotated/expired	sudo rpm --import NEW_KEY_URL
transaction lock held	Another DNF session running	Wait or kill zombie: killall -9 dnf
repo unavailable	Network issue or site down	Check internet; wait for mirrors
permission denied installing Flatpak	Missing authorization	flatpak permission-set or re-authenticate
broken transaction	Incomplete DNF operation	sudo dnf reinstall rpm; sudo rpm --rebuilddb
Flatpak app fails to launch	Namespace denial	Check overrides: flatkap override --show
disk space full after install	Leftover packages	sudo dnf clean all; sudo dnf autoremove
What's Next

You now possess complete mastery over Fedora 44's dual-layer software ecosystem: DNF5 (installation, removal, updates, searches, dependency resolution, transaction rollback, repository configuration, RPM inspection), Flatpak (remote management, app isolation, sandbox overrides, permission tuning), hybrid deployment patterns, troubleshooting methodologies, and automation techniques for reproducible provisioning.

Chapter 19 transitions to text manipulation — mastering Vim, Nano, GNU Screen, Emacs, and terminal text editors. Text editors become indispensable companions to package management: scripts require editing, configuration files demand modification, logs call for viewing, and automation demands scripting. Chapter 19 equips you to edit text fluently in the terminal without relying on graphical tools.

Until then, practice package discovery. Install unfamiliar tools, examine dependencies, experiment with repositories, and build comfort navigating DNF5's extensive command set. Software literacy transforms curiosity into competence.

    ✅ Chapter 18 Complete You understand package management rationale (why systems bundle dependencies, track files, validate signatures, manage upgrades), DNF5 fundamentals (version verification, configuration parsing, /etc/dnf/dnf.conf and /etc/yum.repos.d/, tab completion enhancements), core DNF operations (install/remove/update/search/info/query/deplist/autoremove/distro-sync, with and without confirmation flags), dependency resolution (auto-install chains, group awareness, orphan cleanup), group management (@group list/install/info/remove), repository configuration (enable/disable/add-third-party/RPM Fusion setup/metalinks/mirror creation), low-level RPM introspection (installation/upgrading/querying/listing-files/verification/signature-checking/-va integrity scan), Flatpak command-line guide (remote-add/search/install/update/unlist-list/prune/run/override/reset-permissions), DNF vs. Flatpak decision matrices with performance comparisons and hybrid deployment strategies, comprehensive troubleshooting methodologies covering dependency conflicts, corrupt caches, signature failures, broken transactions, and disk space exhaustion, provisioning automation techniques (kickstart scripts, bulk imports, bash loops, Flatpak automation), progressive hands-on exercises exploring the full spectrum of package management functions, and consolidated troubleshooting reference tables for rapid problem diagnosis.
