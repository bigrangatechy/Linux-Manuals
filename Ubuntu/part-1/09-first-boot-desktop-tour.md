# Chapter 9: GNOME 50 Desktop Tour (Standard Ubuntu)

## Welcome to Your New Desktop

You've installed Ubuntu. You've run through the post-install checklist. Now you're looking at your fresh GNOME 50 desktop for the first time.

If you're coming from Windows or macOS, some things will feel familiar. Other things won't look the same or work the same way. That's okay. Every computer has a learning curve when you switch operating systems. The goal of this chapter is to eliminate the mystery by taking you systematically through every part of the interface so you know where everything is and how it works.

This tour assumes you're running **standard Ubuntu** with **GNOME 50**. If you're on Kubuntu with KDE Plasma, skip ahead to Chapter 9.5.

> 💡 **Quick Tip: Press the Super Key**
>
> Throughout this chapter, you'll see references to the **Super key**. This is typically the **Windows key** on PC keyboards or the **Command ⌘ key** on Mac keyboards. We'll use "Super" because it sounds the same across both keyboard types. If you press it and nothing happens, check that you have the correct key identified.

## The Three Main Areas of Your Screen

Your GNOME desktop can be divided into three main zones:

| Zone | Location | What It Contains |
|---|---|---|
| **Top Bar** | Across the top edge | Activities button, application title, date/time, quick settings icons |
| **Dock** | Left side (by default) | Pinned applications, running apps, workspace indicators |
| **Desktop Area** | Center | Open windows, wallpaper, no icons by default (unlike Windows) |

Let's explore each zone in detail.

### The Top Bar

The top bar runs across the entire width of your screen. It's always visible (unless you hide it manually). From left to right:

**Left Section:**

1. **Activities Button** — Click here or press the Super key to open the Activities Overview (more on this below)
2. **Application Menu** — When an app has menus available, they appear here instead of the app's own menu bar

**Center Section:**

1. **Date and Time** — Click to show calendar and notification history
2. **Workspace Indicator** — Shows your current workspace number

**Right Section (Quick Settings):**

A cluster of small icons that provide quick access to system controls:

1. **Network/Wi-Fi Icon** — Shows connection status, opens network panel
2. **Volume/Speaker Icon** — Adjust audio volume, shows connected output devices
3. **Battery/Power Icon** (laptops) — Shows battery level, power mode settings
4. **Brightness Icon** — Adjust screen brightness (monitors/laptops with built-in controls)
5. **User Menu** — Lock screen, switch users, log out, shut down/restart

Clicking any of these icons expands a dropdown panel with more detailed controls. The panel also includes:

- **Dark Mode Toggle** — Switch between light and dark themes
- **Wi-Fi Quick Connect** — See available networks, connect directly
- **Bluetooth** — Quick toggle and device list
- **Keyboard Shortcuts Help** — A new GNOME 50 feature showing common shortcuts

> 💡 **Customize Quick Settings**
>
> Click the gear icon in the Quick Settings panel to customize which items appear here. You can reorder, add, or remove toggles based on what you actually use.

### The Ubuntu Dock

On the left side of your screen sits the **Ubuntu Dock** — a vertical panel containing application icons.

**What You'll See:**

- **First section** — Default Ubuntu applications (Files, Firefox, Terminal, etc.)
- **Middle section** — Currently running apps (indicated by a dot beneath the icon)
- **Bottom section** — App grid button (nine dots) to open the full application launcher
- **Separator line** — Between pinned apps and running apps
- **Trash** — At the very bottom (can be moved to the right click menu)

**Basic Interactions:**

| Action | Result |
|---|---|
| Click an app icon | Launches or switches to that app |
| Right-click an app | Context menu (New Window, Quit, Unpin, etc.) |
| Middle-click an app | Opens a new instance (some apps) |
| Drag an icon up/down | Reorder pinned apps |
| Drag an icon off dock | Removes/unpins it |
| Click while app is minimized | Restores/maximizes the window |
| Click already-visible app | Minimizes/hides the window |

**Moving the Dock:**

By default, the dock sits on the left. To move it:

1. Open **Settings** → **Appearance**
2. Look for **Dock Position** options
3. Choose Bottom, Left, or Right

Alternatively, install **GNOME Tweaks** (optional advanced tool):

```bash
sudo apt install gnome-tweaks

Then open Tweaks → Appearance → Dock for additional customization.

    ⚠️ Minimize on Click

    By default, clicking an already-open app's dock icon switches to it but doesn't minimize it. If you want the Windows-like behavior (click to toggle minimize), enable it via terminal:

    gsettings set org.gnome.shell.extensions.dash-to-dock middle-click-action 'hide'

The Desktop Area

Here's something that might surprise you: there are no desktop icons in GNOME by default.

Unlike Windows or macOS, where you'd find folders, documents, and shortcuts scattered across the desktop wallpaper, GNOME treats the desktop as... well, empty space. Just your wallpaper and whatever windows you have open.

Why? Because GNOME believes the Activities Overview is a better place to find files and launch apps than cluttered desktop icons. The philosophy is: if you want something, search for it or navigate it through Files — don't scatter it everywhere visually.

If You Want Desktop Icons Anyway:

You can install an extension called "Desktop Icons NG" that restores this functionality:

sudo apt install chrome-gnome-shell
firefox https://extensions.gnome.org/extension/1576/desktop-icons-ng-ding/

(Or search for "desktop icons ng" in GNOME Extensions website). After installation, you'll have standard desktop icons like Home folder, mounted drives, and Trash.

We cover extensions in depth in Chapter 26 (Customization). For now, just know they exist and most aren't necessary for daily use.
The Activities Overview (Your Starting Point)

The Activities Overview is GNOME's central hub. It's the equivalent of the Windows Start Menu + Search combined, plus Task View / Mission Control for managing windows and workspaces.

How to Access It:

    Click the Activities button in the top-left corner
    Or press the Super key
    Or move your mouse to the top-left hot corner of the screen

Once open, you'll see several components:
1. Search Bar

At the top center is a prominent search field. Type anything:

    Application names — Find apps instantly ("Firefox", "Calculator", "Terminal")
    Settings pages — Jump directly to Settings sections ("Display", "Printers")
    Files and folders — Search by name (shows recent and matching files)
    Web searches — Type a query and press Enter to search DuckDuckGo or your configured engine
    Calculations — 5*8 or $10 in EUR performs instant math and unit conversion
    Dictionary lookup — Type a word and get a definition

This is incredibly powerful once you get used to it. Many power users spend their entire session using Activities overview + keyboard navigation instead of browsing through menus.

    💡 Search Tips

        Results are ordered by relevance. Frequently used apps rise to the top.
        Typing partial matches works — "fire" finds Firefox, firewall tools, etc.
        Hold Shift while typing to avoid switching into text input mode within an open window
        You can type directly after pressing Super — no need to click the search bar

2. Application Grid

Below the search results (or behind them) is the full grid of all installed applications. Scroll vertically to browse all categories:

    Featured — Apps promoted by Ubuntu/Canonical
    Productivity — LibreOffice, calendars, notes, etc.
    Utilities — Calculators, archivers, system monitors, terminals
    Games — Pre-installed games
    More Applications — Everything else not categorized above
    Installed from Flatpak/Snap — Third-party packages show up separately

Click the nine-dot button in the Dock to reach this grid directly from the desktop.

    💡 Searching is Faster Than Browsing

    With dozens of apps available, scrolling through grids is inefficient. Train yourself to press Super and start typing immediately. Most people find 90% of their apps within 2–3 keystrokes.

3. Window Thumbnails

Any open windows appear as thumbnails stacked in the center of the screen. Click one to switch to it.

    Drag and drop — Move a window thumbnail to rearrange order
    Swipe gestures — On touchpads, swipe with 3+ fingers between workspaces
    Close windows — Hover over a thumbnail and click the X that appears, or press Ctrl+W from that window's context

4. Workspaces Panel

On the right side of the screen is the workspace indicator showing virtual desktops. Each box represents one workspace.

    Add/remove workspaces — Add automatically when needed, remove when empty
    Move windows between workspaces — Drag and drop thumbnails onto different boxes
    Switch between workspaces — Click a box, or use keyboard shortcuts (below)
    Current workspace highlighted — You can see at a glance where windows live

GNOME defaults to dynamic workspaces — new workspaces are created on demand. You can disable this in extensions if you prefer fixed workspaces.

    💡 Workspaces vs. Alt+Tab

    Workspaces are for separating groups of apps entirely (e.g., "work stuff" on workspace 1, "browser/music" on workspace 2). Alt+Tab cycles through all windows regardless of workspace. Using both together creates an efficient organizational system.

5. Hot Corners

Activate hot corners by moving your cursor to specific screen edges:

    Top-left — Triggers Activities Overview (primary hotspot)
    Top-right — Can trigger a secondary action if enabled
    Other corners — Not active by default but configurable via extensions

To adjust hot corners:

sudo apt install dconf-editor  # Optional advanced tool
dconf-editor > org > gnome > shell > enable-hot-corners

Or simply rely on the Super key to avoid accidental triggers.
Keyboard Shortcuts Worth Learning

GNOME's strengths shine through keyboard navigation. Learn these essential shortcuts early:
Shortcut	Action
Super	Open Activities Overview
Super+A	Show all applications
Super+S	Search in Activities Overview
Alt+Tab	Cycle through open windows
Alt+` (grave accent)	Cycle between windows of the same app
Super+Page Up/Page Down	Move to next/previous workspace
Ctrl+Alt+Arrow Keys	Move current window to adjacent workspace
Super+L	Lock screen
Super+M	Show/hide message area (calendar/notifications)
Super+Up/Down Arrow	Maximize/Restore window
Super+Left/Right Arrow	Tile window to half-screen
Super+C	Show workspace overview
Super+Shift+Q	Close focused window
Super+/	Show keyboard shortcut help overlay

Many more exist. Press Super+/ anytime to see a complete list of GNOME's default shortcuts overlaid on your screen.

    💡 Customize Shortcuts

    Open Settings → Keyboard → View and Customize Shortcuts to remap any shortcut you dislike or create custom ones.

Window Management Basics

Every application you launch creates a window. Here's what GNOME expects you to do with them:
Moving Windows

    Drag the title bar — Anywhere on screen
    Title bar position — Traditionally at the top; some apps place menus there, others keep them separate
    Snap to halves — Drag window to left/right edge until it snaps, then release

Resizing Windows

    Drag the edges/corners — Grab the perimeter and pull inward/outward
    Maximize/Restore — Double-click title bar, press Super+Up/Down, or click maximize/minimize buttons
    Fullscreen — Press F11 or use the app's View menu

Closing Windows

    Click the X in the top-right corner (usually gray circle with white X)
    Press Ctrl+W — Universal close-window shortcut
    Press Alt+F4 — Alternate close-window shortcut
    Right-click app in Dock → Quit — Closes the app completely (not just the window)

Tiling (Half-Screen Layouts)

GNOME supports automatic window tiling for multitasking workflows:

    Drag to edge — Pull a window to the left or right screen edge until it highlights
    Release — Occupies exactly half the screen
    Select second window — Fill remaining space with another window
    Resize divider — Drag the border between tiled windows to adjust proportions

Alternatively, use the shortcuts:

    Super+Left Arrow — Tile left half
    Super+Right Arrow — Tile right half
    Super+Up Arrow — Maximize
    Super+Down Arrow — Restore/Unmaximize

Multiple Monitors

If you have multiple displays:

    Open Settings → Displays
    Arrange monitor layout (drag boxes to match physical arrangement)
    Set primary display (checkmark box)
    Adjust resolution and scaling independently
    Choose mirroring vs. extending

GNOME handles multi-monitor setups robustly. Each monitor can have its own top bar, workspaces flow smoothly across screens, and windows remember which monitor they were last on.
System Notifications

Notifications pop up in the top-right corner, fading away after a few seconds unless dismissed manually.

Clicking the clock/date area in the top bar opens the message tray showing:

    Recent notifications (organized by app)
    Calendar view
    Today's schedule (if synced with online accounts)

Clear notifications individually by hovering and clicking ×, or dismiss all at once.
Managing Permissions

Apps must request permission before sending notifications:

    Open Settings → Notifications
    Review list of requesting apps
    Toggle Allow/Block per app
    Optionally set Do Not Disturb mode (silences all notifications during set hours)

    💡 Do Not Disturb

    Enable Do Not Disturb from Quick Settings or Notification panel. All alerts queue silently and display later — useful for meetings, presentations, or focused work sessions.

Online Accounts Integration

Sync your online services with the system-wide account manager:

    Open Settings → Online Accounts
    Sign in to supported services:
        Google (Drive, Contacts, Calendar)
        Nextcloud
        Microsoft Exchange
        OwnCloud
        Social media (Twitter/X — limited support)

Integrated data appears inside corresponding GNOME apps:

    Contacts sync to Contacts app
    Calendars to Calendar app
    Drive files to Files (remote folders in sidebar)
    Emails to Evolution mail client

Note: Google Drive integration was removed from GNOME Files in version 50 but still works through integrated web views and browser-based access.

    ⚠️ Privacy Considerations

    Online account syncing transmits credentials and data through third-party services. Review each service's privacy policy before connecting. You can disconnect anytime in Settings → Online Accounts.

Application Lifecycle: Installation vs. Updates vs. Removal

While you'll learn about package management in depth starting in Chapter 18, understanding the basic lifecycle helps orient you:
Installing Apps

    Graphical: App Center → Search → Install
    Flatpak: Install Flatpak plugin → Browse Flathub
    Terminal: sudo apt install packagename

Updating Apps

    Graphical: Software Updater prompts you automatically
    Terminal: sudo apt update && sudo apt upgrade
    Auto-updates: Snap and Flatpak packages self-update in background

Removing Apps

    Graphical: App Center → Installed → Select → Remove
    Terminal: sudo apt remove --purge packagename

App Center shows rating, description, screenshots, and required permissions before installation — similar to mobile app stores.

We'll master this workflow completely in Chapters 10 and 18.
What's Different About GNOME Compared to Windows/macOS?

To reinforce the mental shift, here's a comparison table:
Feature	Windows	macOS	GNOME (Ubuntu)
Start menu/launcher	Start Menu	Spotlight/Launchpad	Activities Overview
Taskbar	Bottom bar	Dock (bottom/top)	Ubuntu Dock (left) + Top bar
File manager icons on desktop	Yes	Yes	No (by default)
App store	Microsoft Store	App Store	App Center + Snaps/Flatpak
Workspace concept	Virtual desktops	Spaces	Dynamic workspaces (prominent)
Customization	High (registry/tools)	Low/Medium (System Settings)	Medium (extensions/settings)
System monitoring	Task Manager	Activity Monitor	Resources/GNOME System Monitor
Settings organization	Scattered (Control Panel/Settings)	System Preferences + Apps	Unified Settings app

None of these approaches are inherently superior — they reflect different philosophies. GNome emphasizes keyboard efficiency, minimalism, and separation of concerns rather than desktop clutter.
Troubleshooting Common Interface Issues

Even with thorough documentation, you might encounter quirks. Here are frequent issues and solutions:
Dock Appears Disappears

Some configurations auto-hide the dock. To restore:

    Settings → Appearance → Dock
    Enable "Keep Dock on Screen" option

Missing Extension Options

If expected features aren't appearing:

sudo apt install gnome-shell-extensions gnome-tweaks

Screen Resolution Looks Wrong

    Settings → Displays
    Change Resolution to native (recommended)
    Adjust Scale percentage (100%, 125%, 150%, etc.)
    Restart if fractional scaling changes didn't apply properly

Touchpad Gestures Don't Work

    Settings → Mouse & Touchpad
    Verify Natural Scrolling preference
    Enable/adjust Tap to Click, Two-finger scrolling
    On laptops with Precision Touchpads, gestures should work by default

Slow Performance

GNOME 50 is reasonably lightweight, but older hardware may struggle:

    Reduce transparency effects: Settings → Accessibility → Visibility → Reduced Transparency
    Disable animated transitions: Use GNOME Tweaks → General → Animations
    Check Resource usage: Open Resources app → Monitor CPU/RAM
    Consider lighter DE alternatives (Xfce, LXQt) if consistently sluggish

We'll dig deeper into troubleshooting in Chapter 30.
What's Next

You've toured the GNOME 50 desktop thoroughly. You understand the interface, know how to navigate it efficiently, and grasp the core concepts distinguishing it from other environments.

In Chapter 10, we'll dive into software management — mastering the App Center, installing applications safely, and understanding the difference between Snaps, Flatpaks, and traditional deb packages.
Try It Yourself

    Practice opening Activities — Press Super key ten times. Build muscle memory.

    Navigate using only keyboard:
        Press Super, type "Settings," press Enter
        Navigate to Display Settings using arrow keys alone
        Return to desktop using Esc

    Create a personal shortcut:
        Go to Settings → Keyboard → View and Customize Shortcuts
        Pick a function (launch calculator, screenshot, etc.)
        Assign a custom key combination you'll actually use

    Organize your workspaces:
        Open Firefox on workspace 1
        Open Terminal on workspace 2
        Practice jumping between them using Page Up/Page Down

    Tile two windows:
        Drag Files to left edge
        Drag Browser to right edge
        Resize divider to give each appropriate space

    Explore your dock:
        Pin three frequently-used apps
        Observe which apps show running indicators
        Drag icons to reorder them
