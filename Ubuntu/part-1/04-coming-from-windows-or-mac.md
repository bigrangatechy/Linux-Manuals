# Chapter 4: Coming from Windows or Mac

## Why This Chapter Exists

If you're reading this manual, you probably already know how to use a computer. You know what a file manager is, where your settings live, how to install apps, and where your documents go. The challenge with switching to Ubuntu isn't that these concepts don't exist — they absolutely do. The challenge is that they look different, are named differently, and sometimes work in subtly different ways.

This chapter is your mental translation layer. We'll take everything you already know from Windows or macOS and map it to its Ubuntu equivalent, so that from your very first boot you know where to look and what to click.

## The Big Picture: Same Goals, Different Tools

Operating systems all solve the same fundamental problems: managing files, running programs, connecting to networks, displaying a desktop, letting you configure your system, and providing a way to install and update software. Windows, macOS, and Ubuntu all do this — they just use different tools with different names.

The most important thing to understand is this: **Ubuntu's desktop is not a clone of Windows or macOS.** It is its own interface with its own design philosophy. Some things will feel instantly familiar. Others will require a small mental adjustment. That's normal — and by the end of this chapter, you'll have the map.

## Ubuntu's Desktop: GNOME 50

Before we start mapping, let's name what you'll be looking at. Ubuntu 26.04 LTS uses **GNOME 50** (codenamed "Tokyo") as its desktop environment. Remember from Chapter 1 that the desktop environment is the interface layer — the dashboard, windows, menus, panels, and built-in apps that you interact with. GNOME is one of several desktop environments available in the Linux world, and it's the one Canonical chose as Ubuntu's default.

GNOME 50 is a significant release. It's the first GNOME version to go **Wayland-only**, dropping the older X11 display system entirely (if you don't know what that means yet, don't worry — we'll cover display servers in Chapter 9). It also ships with a refreshed set of default applications, improved performance, and a more streamlined Settings panel.

> 💡 **A quick orientation before we dive in:**
>
> If you're coming from Windows, GNOME will feel somewhat like a mix between Windows 10/11's taskbar and a macOS-style full-screen app launcher. If you're coming from macOS, GNOME's Activities Overview will feel similar to Mission Control. Neither comparison is exact, but they're useful starting points.

## Interface Mapping: Window by Window

### Taskbar / Dock

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Taskbar (bottom of screen) | Dock (bottom of screen) | **Ubuntu Dock** (left side of screen) |

The **Ubuntu Dock** is a vertical panel on the left side of your screen that shows pinned and running applications. It functions similarly to the Windows taskbar or macOS Dock:

- **Click** an icon to open or switch to that application
- **Right-click** an icon to see options (quit, open new window, unpin)
- Running apps have an indicator dot next to their icon
- The **app grid button** (nine dots at the bottom of the dock) opens the full application launcher

You can move the dock to the bottom or right side if you prefer — we'll cover customization in Chapter 26. By default, it stays on the left.

### Start Menu / Launchpad / Spotlight

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Start Menu | Launchpad / Spotlight | **Activities Overview** |

The **Activities Overview** is Ubuntu's central hub. You access it by:

- Clicking the **Activities** button in the top-left corner
- Pressing the **Super key** (the Windows key on most keyboards, or Command ⌘ on Mac keyboards)
- Moving your mouse to the **top-left hot corner** of the screen

Once in the Activities Overview, you'll see:

- A **search bar** at the top — start typing to find applications, files, and settings (similar to Spotlight on macOS)
- **Open windows** displayed as thumbnails — click one to switch to it (similar to Mission Control on macOS)
- The **app grid** — a full-screen grid of all installed applications (similar to Launchpad on macOS)
- **Workspaces** on the right side — virtual desktops you can switch between (Windows also has Virtual Desktops; macOS has Spaces)

This is probably the single most important thing to learn on your first day. Press the Super key, type what you're looking for, press Enter. That pattern replaces hunting through the Start Menu or Applications folder.

### System Tray / Control Center / Notification Center

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| System tray (bottom-right) | Control Center (top-right) | **Quick Settings** (top-right) |

In the **top-right corner** of your screen, you'll see a cluster of small icons (network, volume, battery, power). Click this area to open the **Quick Settings** menu, which provides:

- Wi-Fi and Bluetooth toggles
- Volume and brightness sliders
- Battery-aware display dimming (new in GNOME 50)
- Dark/light mode toggle
- Power settings (suspend, restart, shut down, log out)
- A gear icon to jump straight to full **Settings**

Notifications also appear here — when you get a system message, app notification, or update alert, it slides in from the top and stacks in this area. Click the clock area to see your notification history and calendar.

### File Explorer / Finder

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| File Explorer | Finder | **Files** (also called Nautilus) |

The **Files** app is Ubuntu's file manager. It's where you browse, copy, move, rename, and organize your documents, downloads, and folders. The name "Nautilus" is the internal project name — you'll see it referenced in documentation and forums, but the app presents itself as "Files" in the interface.

GNOME 50's version of Files brings significant performance improvements:

- Directory listings load up to **5x faster** than previous versions
- Thumbnail generation is up to **10x faster**
- A built-in **dual-pane view** — press **F3** to split the window into two side-by-side panels for easy drag-and-drop between folders
- Improved handling of remote folders (network shares, SSH connections)

The layout will feel familiar:

- **Sidebar** on the left with shortcuts to Home, Documents, Downloads, Pictures, etc. (similar to the sidebar in Finder or the Quick Access panel in Windows File Explorer)
- **Main pane** showing files and folders as icons or a list
- **Breadcrumb navigation** at the top showing your current path
- Right-click for context menus (copy, paste, properties, open in terminal)

One notable change in GNOME 50: the traditional Preferences dialog for Files has been simplified. Many older configuration options have been removed in favor of sensible defaults. If you find yourself wanting more control, alternative file managers like **Nemo** or **Thunar** are available — we'll discuss alternatives in Chapter 26.

> ⚠️ **No more Google Drive integration**
>
> Previous versions of Files included built-in support for mounting Google Drive as a folder. This integration has been removed in GNOME 50. If you relied on this feature, you'll need to use a third-party tool or the web interface. We'll cover alternatives in Chapter 12.

### Settings

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Settings app / Control Panel | System Settings | **Settings** |

Ubuntu's **Settings** app is where you configure your system. GNOME 50 has reorganized this panel with a more streamlined layout:

- **Consolidated sections** — Privacy, Power, and Accessibility have been reorganized for easier navigation
- **Wi-Fi, Bluetooth, Network** — connectivity settings
- **Displays** — resolution, scaling (including improved fractional scaling), arrangement, and variable refresh rate (VRR) support
- **Sound** — input/output devices and volume levels
- **Keyboard, Mouse & Touchpad** — input device settings and shortcuts
- **Background, Appearance** — wallpaper, light/dark mode, and themes
- **Users** — account management, passwords, and login options
- **Online Accounts** — integration with services like Nextcloud
- **Privacy** — screen lock, location services, camera/microphone permissions, and connection sharing
- **Accessibility** — visual aids, keyboard assists, hearing options
- **Well-being** — new in GNOME 50: screen-time limits and bedtime scheduling (primarily useful for family/shared setups)
- **About** — system information, your Ubuntu version, hardware specs

Most of what you'd configure through Windows Settings or macOS System Preferences lives here. One key difference: Ubuntu doesn't have a single "Control Panel" legacy interface the way Windows does. Everything is in one Settings app.

### Security Center (New in 26.04)

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Windows Security / Defender | — | **Security Center** |

Ubuntu 26.04 LTS introduces a new **Security Center** app that consolidates security-related settings into one place:

- System update status
- Firewall configuration
- Application permissions (especially for Snap packages — you now get prompted when an app requests access to things like your camera, microphone, or home directory)
- TPM-backed Full Disk Encryption status
- Overall security posture overview

This is a new addition for 26.04 and represents Ubuntu's continued effort to make security more accessible. Where Windows has Windows Security (formerly Windows Defender) and macOS spreads security across System Settings, Ubuntu now gives you a centralized hub.

We'll cover security in depth in Chapter 20, but it's worth knowing this app exists from day one.

### Software Installation

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Microsoft Store / download .exe | App Store / download .dmg | **App Center** |

The **App Center** is Ubuntu's graphical software manager. It replaces the older "Ubuntu Software" / "Snap Store" application and handles both **Snap** packages and traditional **.deb** packages from the Ubuntu archives. This is a significant improvement over previous versions, where managing .deb packages graphically was limited.

With App Center, you can:

- **Browse and search** for applications by name or category
- **Install** applications with a single click (no need to hunt for download links on websites)
- **Update** installed applications
- **Remove** applications you no longer need
- **Rate and review** apps
- **Switch channels** for Snap packages (e.g., from stable to beta versions)
- **Install local .deb files** — a newer capability that brings App Center closer to a one-stop shop for software management

> ⚠️ **A Note on App Center Quality**
>
> While App Center has improved significantly, it's not perfect. Some users have reported issues with outdated Snap listings, occasional UI glitches, and inconsistent .deb package support. If you ever run into trouble installing something through App Center, don't worry — the terminal offers a more reliable way to install software, and we'll teach you everything you need to know about it starting in Chapter 13. You are never trapped by a graphical tool not working.

### Task Manager

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Task Manager (Ctrl+Shift+Esc) | Activity Monitor | **Resources** |

Ubuntu 26.04 replaces the old GNOME System Monitor with a new app called **Resources**. It provides a more visual, modern dashboard for monitoring your system:

- **CPU usage** — per-core graphs and total utilization
- **Memory** — RAM usage, swap, and per-process consumption
- **Network** — upload/download activity over time
- **Processes** — running applications and system processes, sorted by resource usage
- **Kill processes** — force-quit unresponsive applications

To open it quickly, press the Super key, type "Resources," and press Enter. You can also force-quit an application by right-clicking its icon in the Dock and selecting "Quit" — if the app is frozen, this sends a terminate signal.

### Terminal

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Command Prompt / PowerShell | Terminal | **Ptyxis** |

Ubuntu 26.04 ships with **Ptyxis** as its default terminal emulator, replacing the older GNOME Terminal. Ptyxis is a modern, container-aware terminal that brings several improvements:

- **Tabbed interface** with a tab overview for managing multiple sessions
- **Named tabs** — you can label tabs to keep track of what you're doing
- **Container support** — integrated support for Podman, Distrobox, and Toolbox (don't worry about what these are yet — they're advanced tools we'll cover much later)
- **Search** within terminal output
- **Refreshed color palette** and modern styling

The terminal is where you'll eventually do most of your power-user work. For now, just know it exists — you can open it by pressing Super, typing "Terminal" or "Ptyxis," and pressing Enter. We'll start using it gently in Chapter 13.

> 💡 **Don't be intimidated by the terminal.**
>
> Many newcomers see the terminal as a scary thing reserved for experts. It's not. It's just another way to talk to your computer — one that happens to be more precise and powerful than clicking through menus. We'll build your terminal skills gradually, starting from absolute zero. By the end of this manual, you'll be comfortable with it.

### Web Browser

| Windows | macOS | Ubuntu (GNOME 50) |
|---|---|---|
| Microsoft Edge | Safari | **Firefox** (pre-installed) + **Web** (available) |

Ubuntu comes with **Mozilla Firefox** pre-installed as the default web browser. Firefox is open source, privacy-focused, and full-featured — comparable to Chrome or Edge in capabilities.

GNOME also includes a lightweight browser called **Web** (internally known as Epiphany), which is simpler and faster but less feature-rich. It's good for quick lookups but most users stick with Firefox or install their preferred browser (Chrome, Brave, etc.) through the App Center.

### Default Application Reference Table

Here's a comprehensive mapping of the applications you'll find on a fresh Ubuntu 26.04 LTS installation:

| Purpose | Windows Equivalent | macOS Equivalent | Ubuntu 26.04 Default |
|---|---|---|---|
| File manager | File Explorer | Finder | Files (Nautilus) |
| Web browser | Microsoft Edge | Safari | Firefox |
| Email client | Outlook / Mail | Mail | Geary |
| Video player | Windows Media Player | QuickTime | Showtime |
| Music player | Windows Media Player | Music | Rhythmbox |
| Image viewer | Photos | Preview | Loupe |
| Photo management | Photos | Photos | Shotwell |
| Document viewer | — | Preview | Papers |
| Text editor | Notepad | TextEdit | Text Editor (GNOME Text Editor) |
| Terminal | Command Prompt / PowerShell | Terminal | Ptyxis |
| System monitor | Task Manager | Activity Monitor | Resources |
| Software manager | Microsoft Store | App Store | App Center |
| Calculator | Calculator | Calculator | Calculator |
| Calendar | — | Calendar | Calendar |
| Settings | Settings / Control Panel | System Settings | Settings |
| Security | Windows Security | — | Security Center |

## File System Mapping

One of the most confusing things for newcomers is that Ubuntu organizes files differently than Windows. macOS users will find it more familiar, since macOS is also built on Unix foundations.

### Where Are My Files?

| Windows | macOS | Ubuntu |
|---|---|---|
| `C:\Users\<username>` | `/Users/<username>` | `/home/<username>` |

Your personal files live in your **Home directory** (`/home/yourusername`). Within it, you'll find familiar folders:

- **Desktop** — what's on your desktop
- **Documents** — your documents
- **Downloads** — files you've downloaded
- **Pictures** — photos and images
- **Music** — audio files
- **Videos** — video files

These names match what you're used to. When you open the Files app, it opens to your Home directory by default.

### No Drive Letters

Windows uses drive letters (`C:`, `D:`, `E:`) to identify storage devices. Ubuntu (and macOS) use a different system called **mount points**.

Instead of `D:\`, a second hard drive or USB stick appears as a folder, typically under `/media/yourusername/drivename`. You access it through the Files app just like any other folder — it shows up in the sidebar under "Other Locations" or "Devices."

We'll cover the file system hierarchy in detail in Chapter 14 — for now, the key takeaway is: **there are no C: or D: drives. Everything is a folder.**

### Installing Software: No .exe Files

On Windows, you download a `.exe` file from a website and run it. On macOS, you download a `.dmg` and drag the app to your Applications folder.

On Ubuntu, the recommended way to install software is through the **App Center** or the **package manager** (APT, which we'll cover in Chapter 18). You don't typically browse the web for installers. Instead:

- Open the App Center
- Search for the application
- Click "Install"
- Done

This is actually safer — the software comes from Canonical's curated repositories, which are checked for malware and digitally signed. You don't risk downloading a fake installer from a random website.

Some applications do come as `.deb` files (similar to `.exe` installers) from developer websites. These can be installed by double-clicking them or through App Center. We'll cover this in Chapter 10.

### Configuration: No Registry

Windows stores system and application settings in a central database called the **Registry**. macOS stores them in preference files (`~/Library/Preferences/`).

Ubuntu stores configuration as **plain text files**, typically in your home directory under hidden folders (folders that start with a dot, like `.config/` or `.local/`). Application settings live alongside their respective data. There is no central registry database.

This means:

- You can read and edit configuration with any text editor
- Backing up your configuration means copying text files
- There's no "registry corruption" scenario
- Configuration is transparent and human-readable

We'll explore this more in later chapters when we start working with the terminal.

## Conceptual Differences Worth Knowing Now

These aren't about specific apps — they're about how the system *thinks* differently:

### Case Sensitivity

On Windows and macOS, filenames are case-insensitive — `Document.txt` and `document.txt` are considered the same file.

On Ubuntu, filenames are **case-sensitive** — `Document.txt` and `document.txt` are two completely different files. This catches many newcomers off guard. Keep it in mind when you're searching for files or typing paths.

### User Accounts and Permissions

On Windows, many people run as an Administrator account by default, meaning every program they launch has full system access. On macOS, users typically have administrator privileges and are prompted for their password when making system changes.

Ubuntu takes this further. **Your normal user account does not have root (administrator) access by default.** When you need to do something that affects the system — install software, change system settings, modify system files — you use a command called **`sudo`** ("superuser do"), which temporarily elevates your privileges and requires your password.

This is a deliberate security design. We'll cover it in detail in Chapter 17, but for now, know that being asked for your password when installing software is **normal and correct** — it's protecting you.

### Software Updates

On Windows, updates are managed by Windows Update and often require reboots. On macOS, updates come through System Settings and also frequently require reboots.

On Ubuntu:

- **System updates** (kernel, core packages, security patches) are managed through the Update Manager or APT
- **Application updates** (for Snap packages) happen automatically in the background
- Many updates don't require a reboot — only kernel updates typically do
- With **Ubuntu Pro's Livepatch** feature, even kernel security patches can be applied without rebooting

You'll see update notifications appear in the top-right of your screen. We'll cover the update process in detail in the Post-Install Checklist chapter.

## What's Next

You now have a mental map between your current operating system and Ubuntu 26.04 LTS. You know where your files are, where your settings live, how to install software, and how the desktop is organized. The transition from here is mostly about muscle memory — using the Activities Overview instead of a Start Menu, looking at the left dock instead of a bottom taskbar, and trusting the App Center instead of browsing for .exe files.

In Chapter 5, we'll get practical: how to actually obtain Ubuntu 26.04 LTS — downloading the right image, verifying it, and preparing installation media.

---

## Try It Yourself

1. **If you have Ubuntu installed already**, try these exercises:
   - Press the **Super key** to open Activities Overview. Search for "Files" and open it.
   - Look at the **top-right corner** of your screen. Click the cluster of icons to open Quick Settings.
   - Press **Super**, type "Settings," and open it. Browse through the sections to get a feel for what's configurable.
   - Open the **App Center** (search for it in Activities) and browse a few categories — see what's available.

2. **If you don't have Ubuntu installed yet**, don't worry — you'll do these exercises after installation. For now:
   - Make a mental note of which Windows or macOS applications you use most. Which ones from the table above replace them?
   - Think about your file organization. Do you rely on multiple drive letters (Windows)? How will you adapt to the mount-point model?

3. **Write down any questions** that came up while reading this chapter. We'll address many of them in the chapters ahead, but identifying your uncertainties now helps you know what to pay attention to.
