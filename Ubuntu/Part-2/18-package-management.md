# Chapter 18: Package Management from the Terminal

## From Point-and-Click to Command Line

In Chapter 10, you learned to install, update, and remove software through the graphical App Center. That works well — but it's only half the story. The terminal gives you finer control, faster execution, and capabilities the graphical tools simply don't offer: searching by description, listing dependencies, holding packages back from updates, cleaning up orphaned files, and scripting installations.

This chapter teaches you the three terminal-based package managers you'll use on Ubuntu 26.04: **APT** (for traditional `.deb` packages), **Snap** (for Canonical's universal packages), and **Flatpak** (for Flathub's community packages). We'll also cover **AppImage** files.

Everything here builds on Chapters 13–17. You already know `sudo`, navigation, piping, and redirection. Now we apply those skills to software management.

## APT: The Core Package Manager

**APT** (Advanced Package Tool) is the backbone of Ubuntu's software system. It manages `.deb` packages — the traditional Debian/Ubuntu package format. Every time you've typed `sudo apt install something` in earlier chapters, you were using APT.

Ubuntu 26.04 ships with **APT 3.1**, a significant upgrade from the APT 2.x found in Ubuntu 24.04. The new version includes a smarter dependency resolver (called `solver3`), new diagnostic commands (`apt why` and `apt why-not`), and tighter security policies. For you as a user, this means fewer "kept back" packages, better conflict resolution, and clearer error messages.

### `apt` vs. `apt-get`: Which Should I Use?

You'll encounter two commands that look similar: `apt` and `apt-get`. Both work. Here's the difference:

| Command | What It Is | When to Use |
|---|---|---|
| `apt` | Newer, user-friendly front-end | Interactive terminal use (your default) |
| `apt-get` | Older, script-oriented front-end | Shell scripts and automation (Chapter 27) |
| `apt-cache` | Older search/query tool | Replaced by `apt search` and `apt show` |

**Rule of thumb:** Use `apt` when typing commands yourself. Use `apt-get` in scripts. They use the same backend — the difference is in output formatting and a few convenience features.

> 💡 **What About aptitude?**
>
> You may also hear about `aptitude` — another package manager front-end with a text-based fullscreen interface. It's powerful but not installed by default and not necessary for most users. Stick with `apt`.

### Step 1: `apt update` — Refresh the Package List

sudo apt update

Despite the name, this does not update anything. It refreshes APT's local cache of available packages — the list of what's available in the repositories and what versions exist.

Think of it like refreshing a web page: you're downloading the catalog, not buying anything. APT fetches the latest package lists from every enabled repository and updates its index.

Output:

Hit:1 http://archive.ubuntu.com/ubuntu resolute InRelease
Hit:2 http://archive.ubuntu.com/ubuntu resolute-updates InRelease
Hit:3 http://archive.ubuntu.com/ubuntu resolute-security InRelease
Hit:4 http://security.ubuntu.com/ubuntu resolute-security InRelease
Reading package lists... Done
Building dependency tree... Done
All packages are up to date.

Line Prefix	Meaning
Hit	Package list is already current — no changes since last check
Get	New package list downloaded — updates available
Ign	Ignored (temporary server issue, will retry later)
Err	Error — couldn't reach the repository (check network)

    💡 Always Run apt update Before apt install

    If you haven't run apt update recently, APT might try to install an outdated version of a package, or fail because the package was renamed. Make it a habit: sudo apt update && sudo apt install packagename.

Step 2: apt search — Find Packages

apt search video editor

Searches the package lists by both package name and description. No sudo needed — searching is a read-only operation.

Output:

Sorting... Done
Full Text Search... Done
kdenlive/jammy 4:22.08.3-1 amd64
  non-linear video editor

openshot/jammy 2.5.1+dfsg1-2build2 amd64
  video editor, similar to iMovie or Windows Movie Maker

shotcut/jammy 22.09.23+ds-1 amd64
  video and audio editor for Linux

Each result shows:

    Package name and repository
    Version
    Architecture (amd64 = 64-bit)
    Short description

Search by exact name:

apt search "^vlc$"

The ^ and $ are regex anchors — this matches only packages named exactly "vlc."

Search with full descriptions:

apt show vlc

Shows detailed information about a specific package:

Package: vlc
Version: 3.0.21-4build1
Priority: optional
Section: universe/graphics
Origin: Ubuntu
Maintainer: Ubuntu Developers <ubuntu-devel-discuss@lists.ubuntu.com>
Installed-Size: 124 MB
Depends: libc6, libvlc0, vlc-bin, vlc-plugin-base, vlc-plugin-qt, ...
Recommends: vlc-plugin-video-splitter, vlc-plugin-skins2, ...
Suggests: vlc-plugin-access-extra, vlc-plugin-fluidsynth, ...
Description: multimedia player and streamer
  VLC is the VideoLAN project's media player. It plays MPEG, MPEG-2,
  MPEG-4, DivX, MOV, WMV, QuickTime, WebM, MP3, Ogg Vorbis, DVD,
  and many more...
Homepage: https://www.videolan.org/vlc/

Key fields to understand:
Field	What It Tells You
Version	Available version number
Installed-Size	How much disk space it uses when installed
Depends	Required dependencies (must be installed for it to work)
Recommends	Strongly suggested packages (installed by default)
Suggests	Optional companion packages
Description	Detailed explanation of what the package does
Homepage	Project website for more information
Step 3: apt install — Install Packages

sudo apt install vlc

This command:

    Checks dependencies — what other packages does VLC need?
    Downloads VLC and any missing dependencies
    Installs all packages
    Configures them

You'll see a summary before installation:

The following NEW packages will be installed:
  vlc vlc-bin vlc-plugin-base vlc-plugin-qt
4 newly installed, 21 not upgraded.
Need to get 45.3 MB of archives.
After this operation, 124 MB of additional disk space will be used.
Do you want to continue? [Y/n]

Press Y or Enter to proceed. Press n to cancel.

Install multiple packages at once:

sudo apt install htop vim git curl

Install without confirmation prompt:

sudo apt install -y vlc

The -y flag automatically answers "yes" to all prompts. Convenient but risky — always double-check the package list before using -y.

Install a specific version:

sudo apt install vlc=3.0.21-4build1

Reinstall a package (fix corrupted installation):

sudo apt install --reinstall vlc

Install only if it's not already installed:

sudo apt install --no-install-requires vlc

Wait — let me give you the correct flag for that:

sudo apt install --no-install-recommends vlc

The --no-install-recommends flag skips "recommended" packages — installing only the strict minimum. Useful when you want a lean installation or when recommended packages are causing conflicts.

    💡 What Are Dependencies?

    Very few programs are completely self-contained. Most rely on shared libraries — code provided by other packages. For example, many graphical applications depend on libgtk-3 (the GTK toolkit). When you install a package, APT automatically resolves and installs all required dependencies.

    This is one of APT's greatest strengths. On Windows, each installer bundles its own copies of shared libraries (DLL hell). On Ubuntu, libraries are shared across the system, and APT handles the resolution automatically.

Step 4: apt upgrade — Update Installed Packages

After running apt update to refresh the package list:

sudo apt upgrade

This upgrades all installed packages to their latest available versions. It will:

    Download newer versions of packages you have installed
    Install them
    Never remove packages (even if upgrading would require removing a dependency)

You'll see:

Reading package lists... Done
Building dependency tree... Done
Calculating upgrade... Done
The following packages will be upgraded:
  firefox libssl3 openssl python3
4 upgraded, 0 newly installed, 0 to remove, and 0 not upgraded.
Need to get 23.4 MB of archives.
After this operation, 1,024 B of additional disk space will be used.
Do you want to continue? [Y/n]

The combined one-liner you'll use most:

sudo apt update && sudo apt upgrade

The && means "run the second command only if the first succeeds." This is the single most common terminal command on Ubuntu — you'll type it hundreds of times.
Step 5: apt full-upgrade — Upgrade with Dependency Changes

Sometimes an upgrade requires more than just replacing old versions with new ones — it requires installing new dependencies, removing obsolete ones, or switching to different packages entirely. apt upgrade won't do this (it refuses to remove anything). apt full-upgrade will:

sudo apt full-upgrade

This performs the same function as apt upgrade but additionally:

    Installs new dependencies needed by upgraded packages
    Removes packages that conflict with the upgrade
    May remove obsolete packages

On Ubuntu 26.04 with APT 3.1's new solver3, full-upgrade is smarter about resolving conflicts — fewer packages are "kept back" (held from upgrading due to dependency issues).

    💡 When to Use full-upgrade vs. upgrade

        Daily/weekly maintenance: sudo apt update && sudo apt upgrade — safe, won't remove anything
        Major version upgrades or when packages are "kept back": sudo apt update && sudo apt full-upgrade — handles dependency changes

    If apt upgrade shows packages as "not upgraded" or "kept back," run apt full-upgrade to resolve them. Always read what it proposes to remove before pressing Y.

Step 6: apt remove and apt purge — Remove Packages

Remove a package (keep config files):

sudo apt remove vlc

Uninstalls VLC but leaves its configuration files in place. Useful if you might reinstall later and want to keep settings.

Remove a package and its config files:

sudo apt purge vlc

Completely removes the package and its configuration files. More thorough — use this when you want a clean slate.

Remove multiple packages:

sudo apt remove vlc htop vim

Remove unused dependencies:

sudo apt autoremove

When you install a package, APT pulls in its dependencies. When you remove that package, the dependencies stay behind — they're no longer needed but still installed. autoremove cleans them up:

Reading package lists... Done
Building dependency tree... Done
The following packages will be REMOVED:
  libvlc0 vlc-bin vlc-plugin-base vlc-plugin-qt
4 not needed, 0 newly installed, 0 to remove and 0 not upgraded.
Need to get 0 B of archives.
After this operation, 45.3 MB disk space will be freed.
Do you want to continue? [Y/n]

Run autoremove after removing packages to keep your system lean.

    ⚠️ Always Read What Will Be Removed

    Before pressing Y on any remove, purge, or autoremove command, read the list of packages being removed. If you see something unexpected (especially system-critical packages like python3 or gnome-shell), press n and investigate.

Step 7: Listing and Querying Packages

List all installed packages:

apt list --installed

This outputs every package on your system — usually thousands of lines. Pipe it through grep to filter:

apt list --installed | grep python

List packages with available updates:

apt list --upgradable

Shows packages that have newer versions available in the repositories.

Check if a specific package is installed:

apt list --installed | grep vlc

See which version is installed:

apt show vlc | grep "Version"

New in APT 3.1: apt why and apt why-not

Ubuntu 26.04 introduces two diagnostic commands that answer common questions:

Why is this package installed?

apt why curl

Output:

curl
  Depends: libcurl4
  Depends: curl

noble-desktop depends on ubuntu-desktop
  ubuntu-desktop depends on curl

This traces the dependency chain: ubuntu-desktop depends on curl, which is why curl was installed. Useful when you see a package you don't remember installing — apt why tells you what required it.

Why can't I install this package?

apt why-not somepackage

Explains why a package can't be installed — conflicting dependencies, unavailable version, or repository issues. Extremely useful for troubleshooting "package not found" or "unmet dependencies" errors.

    💡 The solver3 Difference

    Previous APT versions would frequently show "The following packages have been kept back:" — meaning some packages couldn't be upgraded due to dependency conflicts. Ubuntu 26.04's new solver3 resolver is significantly better at resolving these situations automatically. You'll see fewer "kept back" messages and smoother upgrades.

Managing Repositories

APT downloads packages from repositories — servers that host package collections. Ubuntu's default repositories are configured automatically during installation, but you may need to add third-party sources.
Viewing Enabled Repositories

apt policy

Shows your repository configuration and priorities:

Package files:
 100 /var/lib/dpkg/status
     release a=now
 500 http://archive.ubuntu.com/ubuntu resolute/universe amd64 Packages
 500 http://archive.ubuntu.com/ubuntu resolute/restricted amd64 Packages
 500 http://archive.ubuntu.com/ubuntu resolute/multiverse amd64 Packages
 500 http://archive.ubuntu.com/ubuntu resolute/main amd64 Packages

Adding Repositories

Enable standard Ubuntu repositories:

sudo add-apt-repository universe
sudo add-apt-repository multiverse
sudo add-apt-repository restricted

These toggle the four main Ubuntu repositories (covered in Chapter 10).

Add a PPA (Personal Package Archive):

sudo add-apt-repository ppa:inkscape.dev/stable
sudo apt update

PPAs are third-party repositories hosted on Launchpad. After adding one, always run apt update to fetch the new package list.

Remove a PPA:

sudo add-apt-repository --remove ppa:inkscape.dev/stable
sudo apt update

The DEB822 Source Format (New in 26.04)

Ubuntu 26.04 adopts the DEB822 format for repository source files by default. Instead of the old single-line format in /etc/apt/sources.list, repository definitions now live in /etc/apt/sources.list.d/ubuntu.sources as structured blocks:

Types: deb
URIs: http://archive.ubuntu.com/ubuntu
Suites: resolute resolute-updates resolute-backports
Components: main universe restricted multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg

This format is more readable and supports richer metadata. As a beginner, you won't need to edit these files directly — add-apt-repository handles it. But if you see a tutorial mentioning /etc/apt/sources.list and yours looks different, the DEB822 format is why.

    ⚠️ Be Careful with Third-Party Repositories

    PPAs and third-party repositories can:

        Introduce unstable packages that break your system
        Override system packages with incompatible versions
        Potentially contain malicious software (rare, but possible)

    Only add repositories from trusted sources (official project websites, recognized developers). When in doubt, prefer Flatpak or Snap over PPAs — they're sandboxed and can't affect system packages.

Installing Local .deb Files with APT

In Chapter 10, you installed .deb files graphically. From the terminal, use APT:

sudo apt install ./package.deb

Note the ./ prefix — it tells APT this is a local file, not a package name in the repositories. Without ./, APT searches the repos and fails.

APT's advantage over dpkg -i (the older method) is dependency resolution: if the .deb file requires other packages, APT downloads and installs them automatically. The older dpkg -i would fail if dependencies weren't met.
Snap: Universal Packages from the Command Line

Snap packages are self-contained, sandboxed applications managed by Canonical's Snap Store. They update automatically in the background and work identically across Linux distributions.
Basic Snap Commands

Search for a Snap:

snap find spotify

Install a Snap:

sudo snap install spotify

Install from a specific channel:

sudo snap install spotify --channel=beta

Channels:

    stable (default) — tested, reliable
    candidate — pre-release, mostly stable
    beta — development version with new features
    edge — bleeding edge, potentially broken

List installed Snaps:

snap list

Update all Snaps manually (they auto-update, but you can force it):

sudo snap refresh

Switch channels:

sudo snap switch spotify --channel=stable

Remove a Snap:

sudo snap remove spotify

Snap Permissions and Connections

Snaps run sandboxed with controlled access to system resources. You can inspect and manage these permissions:

View connections (permissions) for a Snap:

snap connections firefox

Shows which interfaces (permissions) the Snap uses:

Interface       Plug                    Slot                Notes
content         firefox-sandbox         -                   -
desktop         desktop                 :desktop            -
audio-playback  audio-playback          :audio-playback      -
network         network                 :network             -
home            home                    :home               -

Manually connect/disconnect a permission:

sudo snap connect spotify:removable-media :removable-media

Grants Spotify access to USB drives. Disconnect:

sudo snap disconnect spotify:removable-media :removable-media

Managing Snap Versions

Snaps retain old versions after updates. This allows rollback but consumes disk space.

View all revisions of a Snap:

snap list --all | grep spotify

Roll back to a previous version:

sudo snap revert spotify --revision=42

Clean up old versions (from Chapter 16):

LANG=C snap list --all | awk '/disabled/{print $1, $3}' | xargs -rn2 sudo snap remove

Flatpak: Flathub Packages from the Command Line

Flatpak is the community-driven alternative to Snap. If you set up Flatpak in Chapter 7's post-install checklist, you're ready to use it from the terminal.
Basic Flatpak Commands

Search Flathub:

flatpak search blender

Install a Flatpak:

flatpak install flathub org.blender.Blender

The identifier (e.g., org.blender.Blender) is the unique Flatpak ID. The search results show these IDs.

Install without confirmation prompts:

flatpak install -y flathub org.blender.Blender

List installed Flatpaks:

flatpak list

Update all Flatpaks:

flatpak update

Remove a Flatpak:

flatpak uninstall org.blender.Blender

Remove unused Flatpak runtimes (cleanup):

flatpak uninstall --unused

Each Flatpak app depends on a runtime — a shared base layer. When you remove apps, their runtimes may remain. --unused cleans up orphaned runtimes, freeing disk space.
Flatpak Permissions

Like Snaps, Flatpaks run sandboxed. Manage permissions with Flatseal (graphical, Chapter 10) or from the terminal:

View permissions:

flatpak info --show-permissions org.blender.Blender

Override filesystem access:

flatpak override --user --filesystem=/media org.blender.Blender

Grants Blender access to /media (mounted USB drives). The --user flag applies to your user only (no sudo needed).

Reset overrides to defaults:

flatpak override --user --reset org.blender.Blender

AppImage Files

AppImage is a third universal format — a single executable file that contains the entire application. No installation, no package manager, no sandboxing.
Using an AppImage

    Download the .AppImage file from the developer's website
    Make it executable:

    chmod +x application.AppImage

    Run it:

    ./application.AppImage

That's it — no installation needed. The app runs directly from the file.
Managing AppImages

AppImages don't register with the system automatically — they won't appear in your app grid. To fix this, install AppImageLauncher:

sudo add-apt-repository ppa:appimagelauncher-team/stable
sudo apt update
sudo apt install appimagelauncher

After installation, when you double-click an AppImage, AppImageLauncher offers to integrate it into your system — moving the file to a dedicated folder and adding it to your application grid.

    💡 Which Format Should I Use?

    When an application is available in multiple formats, here's the priority order:

        Flatpak (from Flathub) — community-maintained, sandboxed, good permissions model
        Snap — if the Flatpak version is outdated or unavailable
        APT/.deb — for system tools and libraries (these integrate most tightly with the system)
        AppImage — for apps not available in any other format, or when you want a portable single-file executable

    This isn't a rule — just a guideline. Use whatever works best for each application.

Holding Packages: Preventing Updates

Sometimes you need to prevent a specific package from updating — perhaps a new version has a bug, or you need to stay on a specific version for compatibility.

Hold a package:

sudo apt-mark hold vlc

VLC will no longer be upgraded by apt upgrade or apt full-upgrade.

Unhold a package:

sudo apt-mark unhold vlc

List held packages:

apt-mark showhold

    💡 Holding Is Temporary

    Held packages resume normal updates after you unhold them. This isn't a permanent solution — eventually you'll want to update. Use holds to buy time until a bug is fixed in a newer release.

Downloading Packages Without Installing

Sometimes you want to download a .deb file without installing it — perhaps to install on a different machine without internet access.

apt download vlc

Downloads the .deb file to your current directory. No sudo needed — you're only downloading, not installing.

Download source code:

apt source vlc

Downloads the source package (requires deb-src in your sources configuration).
Package Management Cheat Sheet
Task	Command
Refresh package list	sudo apt update
Upgrade all packages	sudo apt upgrade
Full upgrade (with dependency changes)	sudo apt full-upgrade
Install a package	sudo apt install packagename
Install a local .deb file	sudo apt install ./file.deb
Remove a package (keep configs)	sudo apt remove packagename
Remove a package (delete configs)	sudo apt purge packagename
Remove unused dependencies	sudo apt autoremove
Search for packages	apt search keyword
Show package details	apt show packagename
List installed packages	apt list --installed
List upgradable packages	apt list --upgradable
Why is a package installed?	apt why packagename
Why can't it be installed?	apt why-not packagename
Hold a package	sudo apt-mark hold packagename
Unhold a package	sudo apt-mark unhold packagename
Clean cached packages	sudo apt clean
Clean obsolete caches	sudo apt autoclean
Add a PPA	sudo add-apt-repository ppa:name/ppa
Remove a PPA	sudo add-apt-repository --remove ppa:name/ppa
Install a Snap	sudo snap install packagename
List Snaps	snap list
Remove a Snap	sudo snap remove packagename
Refresh all Snaps	sudo snap refresh
Install a Flatpak	flatpak install flathub packagename
List Flatpaks	flatpak list
Update all Flatpaks	flatpak update
Remove a Flatpak	flatpak uninstall packagename
Clean unused Flatpak runtimes	flatpak uninstall --unused
Make AppImage executable	chmod +x file.AppImage
Common Problems and Solutions
"Unable to locate package"

The package name doesn't exist in your enabled repositories. Fixes:

    Run sudo apt update to refresh package lists
    Check spelling: apt search packagename to find the correct name
    Enable universe/multiverse repos (Chapter 10)
    The package may only be available as a Snap or Flatpak — not as an APT package

"Unmet dependencies" / "Broken packages"

Dependency conflicts prevent installation. Try in order:

sudo apt --fix-broken install
sudo apt update
sudo apt full-upgrade

If still broken:

sudo dpkg --configure -a
sudo apt --fix-broken install

"Held broken packages"

APT can't satisfy the dependency requirements. Use the new diagnostic command:

apt why-not packagename

This explains the specific conflict, helping you understand what's blocking installation.
"Lock file /var/lib/dpkg/lock-frontend"

Another package manager is running (Software Updater, App Center, or another terminal session). Wait for it to finish. If it's truly stuck:

sudo rm /var/lib/apt/lists/lock
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock-frontend
sudo dpkg --configure -a

    ⚠️ Only Remove Locks as Last Resort

    Removing lock files should only be done when you're certain no package manager is actually running. If you remove locks while an installation is in progress, you can corrupt your package database. Always wait or reboot first.

Snap "has install饭hooks" or Won't Update

If a Snap is stuck:

sudo snap refresh --list
sudo snap refresh packagename

If a Snap is broken and won't repair:

sudo snap remove packagename
sudo snap install packagename

Flatpak "runtime not found"

A Flatpak app needs a runtime that isn't installed:

flatpak install flathub runtime_name

The error message tells you which runtime is missing. Install it, then retry the app.
The Combined Update Command

Here's the ultimate maintenance one-liner that updates all three package systems:

sudo apt update && sudo apt upgrade -y && sudo snap refresh && flatpak update -y && sudo apt autoremove -y

This:

    Refreshes APT's package list
    Upgrades all APT packages
    Refreshes all Snap packages
    Updates all Flatpak packages
    Removes orphaned dependencies

Run this weekly to keep everything current. You could even alias it (Chapter 19) for convenience.
What's Next

You now have complete control over software management from the terminal. You can install, update, search, remove, hold, and troubleshoot packages across all three package formats. This is one of the most practically useful chapters in the manual — you'll use these commands constantly.

In Chapter 19, we'll learn about the shell itself — how to customize your terminal, create aliases for common commands, use environment variables, and write your first shell script. You're about to make the terminal truly your own.
Try It Yourself

    Refresh and upgrade. Run sudo apt update && sudo apt upgrade. Read the output carefully. Note how many packages were upgraded.

    Search for a package. Run apt search "text editor". Browse the results. Then run apt show on one that interests you to see its details.

    Install something new. Pick a small utility you don't have (e.g., tree — a visual directory viewer). Run sudo apt install tree. Then try it: tree ~

    List your installed packages. Run apt list --installed | wc -l to count how many packages are installed. Then filter: apt list --installed | grep python to see Python-related packages.

    Check for held packages. Run apt-mark showhold. If nothing appears, that's good — no packages are being held back.

    Try apt why. Run apt why curl or apt why wget to trace why a package was installed. Follow the dependency chain.

    Manage Snaps. Run snap list to see your installed Snaps. Run snap find to search for something new.

    Run the combined update. Execute the full update command from the end of this chapter. See how APT, Snap, and Flatpak updates flow together.

    Clean up. Run sudo apt autoremove && sudo apt autoclean. Check disk space before and after with df -h.

    ✅ Chapter 18 Complete You can manage software entirely from the terminal using APT (install, search, upgrade, remove, hold, purge, autoremove), Snap (install, list, refresh, remove, manage channels), and Flatpak (install, list, update, uninstall, manage permissions). You understand dependencies, repositories, PPAs, the new DEB822 format, and can troubleshoot common package management errors. In Chapter 19, we'll customize the shell and write your first script.

