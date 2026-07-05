# Chapter 9: The GNOME 50 Desktop Tour

## Welcome to Your Desktop

You're looking at GNOME 50 — codename "Tokyo" — the desktop environment that Fedora 44 ships as its default interface. GNOME 50 was released on March 18, 2026, after six months of development by the GNOME Project community. It represents the most significant visual and functional evolution of GNOME since the 3.0 redesign of 2011.

Fedora ships **vanilla GNOME 50** — the pure, unmodified upstream GNOME experience. No custom themes, no pre-installed dock extensions, no vendor modifications. What you see is exactly what the GNOME developers designed.

This chapter is a comprehensive tour of every element of the desktop. By the end, you'll know where everything is, what everything does, and — just as importantly — *why* GNOME works the way it does.

---

## The Design Philosophy

Before diving into buttons and menus, you need to understand GNOME's design philosophy. It's different from Windows, macOS, and even other Linux desktops.

GNOME is built around three principles:

### 1. Get Out of the Way

GNOME minimizes chrome — the persistent UI elements that compete for your attention. There's no taskbar, no desktop icons, no system tray clutter, no start menu. The desktop fades into the background. Your applications and content take center stage.

Where Windows and macOS fill your screen with persistent controls, GNOME gives you a mostly empty screen with a single black bar at the top. Everything else appears only when you summon it.

### 2. Keyboard and Gesture First

GNOME assumes you'll use the keyboard and touchpad as primary input devices. The Super key (that's the Windows key on most keyboards, or Command on Apple keyboards) is the gateway to everything. Touchpad gestures — three-finger swipes, pinch-to-zoom — are first-class interactions, not afterthoughts.

This doesn't mean GNOME is unfriendly to mouse users. But power users who embrace keyboard shortcuts and gestures will move significantly faster.

### 3. Focus-Oriented, Not Task-Oriented

GNOME's workspace model encourages you to dedicate full-screen or near-full-screen applications to specific tasks. Rather than cascading overlapping windows (the Windows approach) or permanently docked applications (the macOS approach), GNOME wants you to give each activity its own workspace and switch between them cleanly.

This is a deliberate departure from the traditional desktop metaphor. It can feel jarring at first — especially if you're coming from Windows. Give it a week. Most users find that focus-oriented workspaces genuinely improve concentration.

> 💡 **GNOME Classic: The Transition Path**
>
> If GNOME's workflow feels too alien, Fedora also ships **GNOME Classic** — a session that gives GNOME a traditional two-panel layout (top bar + bottom taskbar) with application menus. Select it from the gear icon on the login screen. It's still GNOME underneath, just presented in a more familiar way. Use it as training wheels while you adjust.

---

## The Top Bar

The **top bar** is the single persistent UI element. It's always visible, spanning the full width of your screen at the top. It's black, semi-transparent, and approximately 32 pixels tall.

┌──────────────────────────────────────────────────────────────────────┐ │ Activities Wed Jul 5 ●●● 🔋 🔊 📶 │ └──────────────────────────────────────────────────────────────────────┘

The top bar has three regions:
Left: Activities

The word "Activities" in the top-left corner is a button. Clicking it opens the Activities Overview — GNOME's central hub (covered in detail below). You can also trigger it by pressing the Super key or by moving your mouse to the top-left hot corner.

On the leftmost edge, you may see indicators for:

    Input method — if you have multiple keyboard layouts, a short label (e.g., "EN" or "JP") indicates the active layout
    External keyboard — GNOME 50 adds improved detection of external keyboards, showing their layout source in the top bar

Center: Clock and Calendar

The center of the top bar shows the current day, date, and time:

Wed Jul 5  14:32

Click the clock to reveal the calendar and notification panel (covered below).

    💡 12-Hour vs. 24-Hour

    The clock format follows your system locale. If you set your language to English (US), you'll see 12-hour AM/PM format. English (UK) or most other locales give you 24-hour format. You can change this in Settings → Date & Time.

Right: System Status

The right side of the top bar shows system status indicators:

●●● 🔋 🔊 📶

Icon	Meaning
●●● (dots)	Keyboard layout indicator (appears when multiple layouts are configured)
🔋	Battery (laptops only; shows charge percentage on hover)
🔊	Volume (click to adjust)
📶	Network (Wi-Fi signal strength, or Ethernet status)
🔆	Brightness (appears on laptops)

Click anywhere in the right region — or press Super+S (new in GNOME 50) — to open the Quick Settings panel.
The Quick Settings Panel

Clicking the right side of the top bar opens the Quick Settings panel — a slide-down menu that provides rapid access to common controls:

┌─────────────────────────────────────────┐
│                                         │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │  🔊      │  │  🔆      │  │  🔋      │ │
│  │ Volume   │  │ Bright-  │  │ Power   │ │
│  │ ━━━━○━━  │  │ ━━○━━━━  │  │ Saver   │ │
│  └─────────┘  └─────────┘  └─────────┘ │
│                                         │
│  ┌─────────────────┐  ┌──────────────┐ │
│  │ Wi-Fi  Connected │  │ Bluetooth Off │ │
│  └─────────────────┘  └──────────────┘ │
│                                         │
│  Power Mode:  ● Balanced                │
│               ○ Performance              │
│               ○ Power Saver              │
│                                         │
│  ────────────────────────────────        │
│  ⚙ Settings          🔒 Lock             │
│  ⏻ Power Off / Log Out                  │
│                                         │
└─────────────────────────────────────────┘

What's Here
Control	Function
Volume slider	Adjust system volume
Brightness slider	Adjust screen brightness (laptops)
Battery indicator	Shows charge level and charging status
Wi-Fi toggle	Turn Wi-Fi on/off; click to see available networks
Bluetooth toggle	Turn Bluetooth on/off; click to pair devices
Power mode	Switch between Balanced, Performance, and Power Saver (from power-profiles-daemon, configured in Chapter 7)
Dark/Light mode	Toggle between light and dark interface theme (if available)
Night Light	Toggle blue-light reduction (also in Settings → Displays)
Settings	Open the full GNOME Settings app
Lock	Lock the screen (Super+L also works)
Power Off / Log Out	Submenu: Power Off, Restart, Suspend, Log Out
GNOME 50 Improvements

GNOME 50 refines the Quick Settings panel with:

    Better tab focus behavior — keyboard navigation through the panel is smoother and more predictable
    Super+S shortcut — new keyboard shortcut to open/close Quick Settings without using the mouse
    Improved Do-Not-Disturb toggle — accessible from the notification area, suppresses notifications more reliably

The Calendar and Notification Panel

Clicking the clock in the center of the top bar opens a different panel — the calendar and notification area:

┌──────────────────────────────────────────┐
│  Notifications                       🌙   │
│                                          │
│  ┌──────────────────────────────────────┐│
│  │ 📧 Mozilla Thunderbird               ││
│  │ Meeting reminder: 3:00 PM            ││
│  │                              now     ││
│  └──────────────────────────────────────┘│
│  ┌──────────────────────────────────────┐│
│  │ 📱 Files                              ││
│  │ Screenshot saved to Pictures         ││
│  │                          12 min ago  ││
│  └──────────────────────────────────────┘│
│                                          │
│  ────────────────────────────────         │
│                                          │
│  July 2026                    ‹  ›       │
│  Mo Tu We Th Fr Sa Su                    │
│     1  2  3  4  5  6                    │
│   7  8  9 10 11 12 13                    │
│  14 15 16 17 18 19 20                    │
│  21 22 23 24 25 26 27                    │
│  28 29 30 31                             │
│                                          │
│  Wed Jul 5                               │
│  ┌──────────────────────────────────────┐│
│  │ 14:00  Team standup                  ││
│  │ 16:00  Code review                    ││
│  └──────────────────────────────────────┘│
│                                          │
│  📅 Open Calendar                        │
└──────────────────────────────────────────┘

Notifications

GNOME 50 groups notifications chronologically. Each notification card shows:

    Application icon — which app sent it
    Title — brief subject
    Body — the message content
    Timestamp — when it arrived ("now", "12 min ago", etc.)

Notifications can be dismissed individually (click the X on hover) or cleared all at once (the "Clear" button at the top of the list).
Do Not Disturb

The moon icon (🌙) at the top toggles Do Not Disturb mode. When enabled:

    Notifications are suppressed (but not lost — they appear when you turn DND off)
    No notification sounds
    No popup banners

GNOME 50 makes the DND toggle clearer and more prominent than in previous versions.
Calendar

Below notifications sits a calendar for the current month. Navigate with the ‹ › arrows. Clicking a date highlights it and shows events from GNOME Calendar for that day. Clicking Open Calendar launches the full GNOME Calendar application.

If you've connected an online account (Google, Nextcloud) in Settings → Online Accounts, calendar events sync automatically and appear here.

    💡 World Clocks

    If you add world clocks in GNOME Clocks (installed by default on Fedora), they appear below the calendar in the panel — handy for tracking time across time zones.

The Activities Overview

The Activities Overview is the heart of GNOME. It's the central hub for launching applications, managing windows, switching workspaces, and searching. It's also the most distinctive aspect of GNOME's design.
Opening the Activities Overview

Three ways to trigger it:
Method	How
Super key	Press the Super key (Windows key). This is the fastest method.
Activities button	Click "Activities" in the top-left corner of the top bar.
Hot corner	Move your mouse pointer to the top-left pixel of the screen.
What You See

When the Activities Overview opens, several things happen simultaneously:

    All open windows shrink into scaled thumbnails, arranged on their respective workspaces
    A search bar appears at the top
    The dash (favorites bar) appears at the bottom
    Workspace thumbnails appear on the right side (vertical strip)

┌──────────────────────────────────────────────────────────┐
│  Type to search...                                       │
│                                                          │
│  ┌───────────────┐  ┌───────────────┐                  │
│  │               │  │               │              ┌───┐│
│  │  Firefox       │  │  Terminal     │              │ 1 ││
│  │               │  │               │              └───┘│
│  │               │  │               │              ┌───┐│
│  └───────────────┘  └───────────────┘              │ 2 ││
│                                                  └───┘│
│                                                          │
│  ┌────────────────────────────────────────────────┐     │
│  │  🦊  📁  📝  ⚙  🖥  🎵  📸  ⊞  ⟳            │     │
│  └────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────┘

The Search Bar

Type immediately after pressing Super — no need to click the search bar first. GNOME 50's search is powerful and instantaneous:
Search Query	Results
Application name (e.g., "firefox")	Matching applications, with icon
File name (e.g., "report.pdf")	Files matching the name
Settings keyword (e.g., "bluetooth")	Relevant Settings panels
Math expression (e.g., "15 * 73")	Calculated result
Software (e.g., "spotify")	If installed: the app. If not: link to install from GNOME Software

Press Enter to launch the top result, or use arrow keys to navigate results and Enter to select.

    💡 Search Providers

    GNOME's search draws from multiple "search providers" — software components that feed results into the unified search. These include:

        Applications (built-in)
        Files (Tracker file indexer)
        Settings (gnome-control-center)
        Calculator (built-in)
        GNOME Software (for installable apps)
        Software Center (installed flatpaks)

    You can toggle which providers contribute to search in Settings → Search.

The Dash

At the bottom of the Activities Overview is the dash — a horizontal bar of application icons. It appears only in the Overview, not on the regular desktop.

┌──────────────────────────────────────────────────┐
│  🦊  📁  📝  ⚙  🖥  🎵  📸  ⊞  ⟳               │
└──────────────────────────────────────────────────┘

Icon	Application
🦊	Firefox (web browser)
📁	Files (Nautilus, file manager)
📝	Text Editor (GNOME Text Editor)
⚙	Settings
🖥	Terminal
🎵	Music (GNOME Music)
📸	Photos / Screenshot
⊞	App grid (show all applications)
⟳	Running applications (appear with a dot under their icon)
Working with the Dash
Action	How
Launch an app	Click its icon
Launch a second window	Middle-click (scroll wheel click) the icon, or Shift+click
Add to favorites	Right-click an app icon → "Pin to Dash" (or drag from app grid)
Remove from favorites	Right-click a pinned icon → "Unpin"
Rearrange favorites	Drag and drop icons into your preferred order
Switch to a running app	Click the running app's icon (has a dot underneath)
Open from keyboard	Super+1 through Super+9 launches the app in that dash position

    ⚠️ Fedora's GNOME Has No Permanent Dock

    Unlike Ubuntu (which adds the Ubuntu Dock extension to make the dash permanently visible on the left side of the screen), Fedora's vanilla GNOME shows the dash only inside the Activities Overview. When you close the Overview, the dash disappears.

    If you want a permanent dock, install the Dash to Dock extension via Extension Manager (Chapter 7 covers this). This is the single most common customization Fedora users make.

The App Grid

Click the ⊞ icon (nine-dot grid) at the right end of the dash to see all installed applications:

┌──────────────────────────────────────────────────┐
│                                                  │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐     │
│  │ 🦊 │ │ 📁 │ │ 📝 │ │ ⚙ │ │ 🖥 │ │ 🎵 │     │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘     │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐     │
│  │ 📸 │ │ 🗓 │ │ 📇 │ │ 🗺 │ │ 🌤 │ │ 🧮 │     │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘     │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐ ┌────┐     │
│  │ 📄 │ │ 🎬 │ │ 📊 │ │ 🛒 │ │ 📦 │ │ 💿 │     │
│  └────┘ └────┘ └────┘ └────┘ └────┘ └────┘     │
│                                                  │
│  Pages:  • ○ ○                          ←→     │
└──────────────────────────────────────────────────┘

The app grid shows all installed applications as icons in a paginated grid. Scroll or use the dots at the bottom to navigate pages.
Action	How
Open an app	Click its icon
Search for an app	Start typing — search activates immediately
Group apps into folders	Drag one app icon onto another to create a folder
Rename a folder	Click the folder name while editing
Move apps between pages	Drag an icon past the edge of the grid
Switch pages	Click the dots, or use left/right arrows

GNOME 50's app grid has smoother page-transition animations than previous versions, and the search integration works seamlessly — typing immediately filters the grid without needing to click into a search bar.
Workspaces

Workspaces (sometimes called "virtual desktops") are separate, independent desktop screens. Each workspace can hold its own set of windows. You switch between workspaces to change context — like switching physical desks.
Dynamic Workspaces

GNOME uses dynamic workspaces — workspaces are created and destroyed automatically:

    You start with one workspace
    When you move a window to a new workspace (or use the workspace switcher), GNOME creates a second one
    When a workspace becomes empty (all windows closed or moved away), it's automatically removed
    There's always one extra empty workspace available for immediate use

This means you never need to explicitly "create" or "delete" workspaces. They appear and disappear organically as you work.
The Workspace Strip

In the Activities Overview, workspace thumbnails appear as a vertical strip on the right side:

┌─────────────┐
│ Firefox      │  ← Workspace 1 (active, highlighted)
│ Terminal     │
└─────────────┘
┌─────────────┐
│ (empty)      │  ← Workspace 2 (available, ready to use)
└─────────────┘

Each thumbnail shows scaled previews of the windows on that workspace. Click a thumbnail to switch to that workspace.
Moving Windows Between Workspaces
Method	How
Drag in Overview	Open Activities Overview, drag a window thumbnail onto a workspace thumbnail
Keyboard shortcut	Shift+Super+↑ or Shift+Super+↓ moves the focused window up/down a workspace
Right-click title bar	"Move to Workspace Up/Down" or "Move to Workspace N"
Switching Workspaces
Method	How
Keyboard	Super+↑ (next) / Super+↓ (previous), or Ctrl+Alt+← / Ctrl+Alt+→
Touchpad	Three-finger swipe left or right
Overview	Click a workspace thumbnail
Overview arrows	While in Overview, use ← / → arrow keys to move between workspaces (new in GNOME 50)

    💡 Workspaces Are GNOME's Superpower

    Here's a practical workspace layout:

        Workspace 1: Web browser + email (communication)
        Workspace 2: Code editor + terminal (development)
        Workspace 3: LibreOffice (document work)
        Workspace 4: Spotify (music, fullscreen)

    Switch between them with a three-finger swipe or Super+↑/↓. Each context gets full screen real estate. No overlapping windows fighting for space. This is the GNOME workflow in its purest form.

Window Management
Window Anatomy

GNOME 50 windows have a header bar (also called a "client-side decoration" or CSD) that combines the title bar and toolbar into a single element:

┌─────────────────────────────────────────────────┐
│  🦊 Mozilla Firefox           ─  ☐  ✕          │
├─────────────────────────────────────────────────┤
│                                                 │
│              Window content                     │
│                                                 │
└─────────────────────────────────────────────────┘

The header bar contains:

    Application icon/name on the left (click for a menu in some apps)
    Window controls on the right: minimize (─), maximize (☐), close (✕)
    Application widgets in the center (buttons, search fields, etc.)

By default, Fedora's GNOME 50 shows only the close button (✕). Minimize and maximize buttons are hidden — you enable them with GNOME Tweaks (Chapter 7).
Window Actions Without Buttons

Even without visible buttons, you have full window control:
Action	Method
Maximize	Double-click the header bar, or Super+↑
Un-maximize	Double-click the header bar again, or Super+↓
Minimize	Super+H (or Super+↓ with the minimize button enabled)
Close	Super+Q (or Alt+F4, or Ctrl+W in many apps)
Move window	Alt+F7, then arrow keys, then Enter
Resize window	Alt+F8, then arrow keys, then Enter
Snap left half	Super+←
Snap right half	Super+→
Snap to corner	Super+← then Super+↑ (etc.)
Tiling and Snap Assist (New in GNOME 50)

GNOME 50 introduces an improved tiling assistant — a built-in system for arranging windows in halves, quarters, and thirds:

    Half-tiling: Drag a window to the left or right edge of the screen, or press Super+← / Super+→. The window fills exactly half the screen.

    Snap assist: When you tile a window to one half, GNOME 50 shows a preview of remaining open windows on the other half. Click one to fill the opposite side — instant side-by-side layout.

    Quarter-tiling: Drag a window to any corner. The window fills that quarter of the screen. Combine four quarter-tiled windows for a quad layout.

    Edge tiling zones: When dragging a window near a screen edge, a highlight overlay shows where the window will snap.

┌─────────────────┬─────────────────┐
│                 │                 │
│   Firefox       │   Terminal      │
│                 │                 │
│                 ├─────────────────┤
│                 │                 │
│                 │   Text Editor   │
│                 │                 │
└─────────────────┴─────────────────┘

    💡 Tiling vs. Workspaces

    Tiling and workspaces solve the same problem — "too many windows, not enough screen" — differently:

        Workspaces give each window (or set of windows) its own full screen
        Tiling splits one screen among multiple windows

    Use both together: on Workspace 1, tile your editor and terminal side by side. On Workspace 2, give your browser the full screen. The combination is powerful.

Keyboard Shortcuts: The Essential Reference

GNOME is designed for keyboard users. Memorizing these shortcuts will transform your experience.
Core Navigation
Shortcut	Action
Super	Open/close Activities Overview
Super + Tab	Switch between open applications (hold Super, tap Tab)
Super + ` (backtick)	Switch between windows of the same application
Super + 1–9	Launch/switch to the app in that dash position
Super + ↑	Move to workspace above
Super + ↓	Move to workspace below
Super + ←	Tile window to left half
Super + →	Tile window to right half
Super + S	Open/close Quick Settings (new in GNOME 50)
Super + L	Lock screen
Super + H	Minimize focused window
Super + Q	Close focused window
Super + D	Show desktop (toggle all windows aside)
Alt + F4	Close focused window
Alt + Tab	Cycle through open windows (classic)
Alt + `	Cycle through windows of current app
Ctrl + Alt + ← / →	Switch workspace left/right
Shift + Super + ↑ / ↓	Move window to workspace above/below
PrtSc	Take a screenshot (opens screenshot UI)
Shift + PrtSc	Take a screenshot of a selected region
Alt + PrtSc	Take a screenshot of the focused window
Text Editing
Shortcut	Action
Ctrl+C	Copy
Ctrl+V	Paste
Ctrl+X	Cut
Ctrl+Z	Undo
Ctrl+Shift+Z	Redo (or Ctrl+Y in some apps)
Ctrl+A	Select all
Ctrl+F	Find
Ctrl+S	Save
Ctrl+N	New (new document/window)
Ctrl+O	Open
Ctrl+W	Close tab or document
Ctrl+T	New tab
Ctrl+P	Print
Terminal-Specific
Shortcut	Action
Ctrl+Shift+T	New tab in terminal
Ctrl+Shift+C	Copy from terminal (Ctrl+C would interrupt a process)
Ctrl+Shift+V	Paste into terminal
Ctrl+Shift+N	New terminal window

    💡 Customizing Shortcuts

    All these shortcuts are customizable in Settings → Keyboard → View and Customize Shortcuts. You can remap any of them, add custom shortcuts (e.g., bind a key combination to launch a specific application), or disable shortcuts you don't use.

Touchpad Gestures

GNOME 50 has best-in-class touchpad support under Wayland. If your laptop has a precision touchpad, these gestures work out of the box:
Gesture	Action
Three-finger swipe left/right	Switch between workspaces
Three-finger swipe up	Open Activities Overview
Three-finger swipe down	Close Activities Overview (or show desktop)
Pinch in	Close a window (in Overview)
Two-finger scroll	Scroll in applications
Two-finger pinch	Zoom (in supported apps like Maps, Photos)
Tap	Left-click
Two-finger tap	Right-click
Three-finger tap	Middle-click (paste selection)
Four-finger swipe	Switch between workspaces (alternative to three-finger)

    💡 Touchpad Quality Matters

    GNOME's gestures require a precision touchpad — a multi-touch-capable touchpad that reports finger positions independently. Most laptops from 2018 onward have these. Older laptops with basic Synaptics touchpads may not support multi-touch gestures.

    If gestures aren't working, check Settings → Mouse & Touchpad → ensure "Tap to Click" and natural scrolling are enabled.

Context Menus and Right-Click

Right-clicking (or two-finger tapping on a touchpad) opens a context menu — a popup with actions relevant to what you clicked:
Where	What You Get
Desktop background	Change background, display settings (limited in GNOME — no "create folder" or "arrange icons")
File in Files	Open, cut, copy, rename, properties, compress, move to trash
App icon in dash	New window, pin/unpin, quit
Window header bar	Minimize, maximize, move, resize, always on top, move to workspace
Link in Firefox	Open in new tab, copy link, bookmark link
Selection in text	Copy, paste, look up (if dictionary installed)

    💡 GNOME's Desktop Is Not a File Manager

    On Windows, the desktop is a file folder — you can put files and shortcuts on it. In GNOME, the desktop is a presentation surface, not a file manager. You can't place files or shortcuts on it (without extensions like "Desktop Icons NG").

    This is intentional: GNOME's philosophy is that the desktop should be clean and empty. Use the Activities Overview, dash, and Files app instead.

File Management: GNOME Files (Nautilus)

GNOME's file manager is called Files (its internal name is Nautilus). Open it from the dash (📁 icon) or press Super+E (if configured) or Super, then type "files."
The Files Interface

┌─────────────────────────────────────────────────────┐
│  ← →  ↑  │ Home                          □  ✕      │
├─────────┬───────────────────────────────────────────┤
│         │                                           │
│ Recent  │  📁 Documents    📁 Downloads              │
│ Home    │  📁 Pictures     📁 Music                  │
│ Desktop │  📁 Videos       📁 Public                 │
│         │  📁 Templates     📁 .config               │
│ Devices │                                           │
│  💿 SSD  │                                           │
│ Network │                                           │
│ Trash   │                                           │
│ Other   │                                           │
└─────────┴───────────────────────────────────────────┘

Sidebar
Entry	Shows
Recent	Recently accessed files across all locations
Home	Your home directory (/home/username)
Desktop	Your desktop folder (exists but isn't displayed on the desktop)
Devices	Connected drives (USB, SD cards, etc.)
Network	Network shares (SMB, NFS — if connected)
Trash	Deleted files (can restore or empty)
Other Locations	Connect to network servers, browse other mounts
GNOME 50 Files Improvements

Nautilus in GNOME 50 receives several meaningful improvements:

    Faster thumbnail generation — image and video thumbnails load more quickly using the Glycin-based loading system
    Reduced memory usage — the file manager uses less RAM, especially with many files open
    Case-insensitive path completion — when typing a path in the location bar, autocomplete now ignores case, matching the way most users expect paths to work
    Smoother scrolling — large directories scroll without lag
    Refined grid/list views — the visual transition between grid and list views is smoother

Files Operations
Operation	How
Open a file	Double-click
Open in terminal	Right-click → "Open in Terminal" (if installed)
Copy	Ctrl+C or right-click → Copy
Cut/Move	Ctrl+X or right-click → Cut, then Ctrl+V in destination
Rename	F2 or right-click → Rename
Delete (trash)	Delete key or right-click → Move to Trash
Delete (permanent)	Shift+Delete
Select multiple	Ctrl+click (individual) or Shift+click (range)
Select all	Ctrl+A
Create folder	Ctrl+Shift+N
New file	Right-click → New Document → (template)
Properties	Right-click → Properties
Path bar	Ctrl+L (shows editable text path instead of breadcrumb buttons)
Hidden files	Ctrl+H (toggle dot-file visibility)
Search	Ctrl+F or start typing
New tab	Ctrl+T
Split pane	F3 (toggle side-by-side panes)

    💡 Hidden Files (Dot Files)

    In Linux, files whose names start with a dot (.) are hidden by default. These are typically configuration files and directories — .config, .bashrc, .local, .ssh. Press Ctrl+H in Files to toggle their visibility. We'll cover dot files in detail in Chapter 14 (Filesystem Hierarchy).

Settings: The Control Center

GNOME Settings (internally gnome-control-center) is the system configuration hub. Open it from the Quick Settings panel (⚙ icon) or from the dash (⚙ icon) or by searching "Settings" in the Overview.
Settings Panels
Panel	What It Controls
Wi-Fi	Wireless network connections, saved networks, hotspot
Network	Wired connections, VPN, proxy, network proxy
Bluetooth	Pair/unpair devices, toggle on/off
Background	Wallpaper, lock screen wallpaper
Appearance	Light/dark mode, accent color (if available), window theme
Displays	Resolution, scaling, refresh rate, night light, multi-monitor arrangement
Sound	Output device, input device, volume levels, system sounds
Power	Sleep timeout, power button behavior, battery percentage, power profile
Keyboard	Keyboard layout, shortcuts, input method
Mouse & Touchpad	Pointer speed, natural scrolling, tap-to-click, two-finger scrolling
Printers	Add/manage printers (CUPS)
Removable Media	Behavior when USB drives, CDs, cameras are connected
Color	Color profiles for monitors (ICC profiles)
Notifications	Per-application notification settings, lock screen notifications, Do Not Disturb
Search	Which search providers are enabled, search order
Apps	Default applications, removable apps, app permissions
Privacy	Location services, camera/microphone access, file history, connectivity checking, problem reporting
Online Accounts	Google, Microsoft, Nextcloud, Fedora account integration
Sharing	Remote desktop, file sharing, media sharing
Users	User accounts, passwords, auto-login
Parental Controls	Screen time limits, bedtime schedules (new in GNOME 50)
Accessibility	Screen reader (Orca), magnifier, on-screen keyboard, visual aids, hearing aids, typing aids, reduced motion
Date & Time	Time zone, automatic time, 12/24-hour format
Region & Language	Display language, input sources, formats
About	System name, hardware info, OS version, disk capacity
GNOME 50 Settings Highlights

Several Settings panels see improvements in GNOME 50:

Displays: Variable Refresh Rate (VRR) and fractional scaling are now enabled by default — no longer hidden behind experimental toggles. This means:

    VRR monitors automatically adjust refresh rate to match content, reducing tearing and saving power
    Fractional scaling (125%, 150%, 175%) works natively on HiDPI displays without blurriness or login tweaks

Accessibility: A new Reduced Motion toggle (distinct from the existing Reduced Animations) lets users minimize UI motion for vestibular comfort. The Orca screen reader receives a major overhaul with global settings and a redesigned preferences window.

Parental Controls: Completely rebuilt. For the first time, parents can:

    Monitor and set screen time limits for child accounts
    Define bedtime schedules that automatically lock the screen
    Extend screen time temporarily with a parent password
    The foundation for web content filtering has been laid (backend service added; UI coming in future releases)

Privacy: Refined screen-time tracking with idle inhibitors — the system more accurately tracks active vs. idle screen time.
The Lock Screen

When you lock the screen (Super+L) or the screen blanks after idle timeout, you see the lock screen:

┌──────────────────────────────────────────────┐
│                                              │
│                                              │
│            Wednesday, July 5                 │
│                14:32                         │
│                                              │
│            🔔  📅                            │
│         (notification previews               │
│          may appear here)                    │
│                                              │
│              ↑                               │
│        (swipe up to unlock)                   │
│                                              │
└──────────────────────────────────────────────┘

The lock screen shows:

    A large clock and date
    Media playback controls (if music is playing)
    Notification previews (unless Do Not Disturb is on)
    Battery status (laptops)

Swipe up (or press any key) to reveal the password entry field. Type your password and press Enter to unlock.

    💡 Lock Screen Notifications

    The lock screen shows notification previews by default. If you don't want private messages visible on the lock screen, disable this in Settings → Notifications → "Lock Screen Notifications."

GNOME 50: What's New — A Summary

Here's a consolidated reference of what changed in GNOME 50 compared to GNOME 49:
Display and Graphics
Feature	Status	Impact
X11 session	Removed	GNOME is Wayland-only. Legacy apps use XWayland.
Variable Refresh Rate (VRR)	Enabled by default	Smoother gameplay, reduced tearing on VRR monitors
Fractional scaling	Enabled by default	HiDPI displays at 125%/150%/175% without blur
HDR screen sharing	New	High dynamic range content in screen sharing/recording
NVIDIA GPU support	Improved	Smoother animations, low-latency cursor for gaming, better external GPU detection
External keyboard layout detection	New	Top bar shows layout of connected external keyboards
Interface
Feature	Status	Impact
Tiling assistant / snap assist	New and improved	Half, quarter, and edge tiling with snap-assist window preview
Quick Settings	Refined	Better keyboard tab-focus, Super+S shortcut
Notifications / Do Not Disturb	Improved	Clearer DND toggle, better notification grouping
Reduced Motion toggle	New	Accessibility option for vestibular comfort
Activities Overview animations	Smoother	Optimized transition animations
App grid page transitions	Smoother	Better page-switch animation
Workspace navigation in Overview	Improved	Arrow keys switch workspaces in Overview
Applications
App	Improvement
Files (Nautilus)	Faster thumbnail loading (Glycin), lower memory usage, case-insensitive path completion
Document Viewer (Evince/Papers)	Ink and text annotation tools directly in the viewer
Calendar	Event attendee lists for public events
Parental Controls	Completely rebuilt: screen time limits, bedtime schedules, web filtering foundation
Orca screen reader	Major overhaul: global settings, redesigned preferences window
System and Infrastructure
Feature	Status	Impact
Headless graphical session	New	gnome-headless-session@user.service for remote desktop/RDP without a physical display
boot_display sysfs support	New	GDM can auto-activate displays on boot (kernel 6.18+)
Screen time tracking	Improved	Idle inhibitors provide accurate active-vs-idle tracking
Hardware-accelerated remote desktop	Improved	Vulkan/VA-API acceleration for remote sessions
Parental controls web filtering	Foundation added	Backend service for content filtering (UI pending)
GNOME Circle: The App Ecosystem

GNOME Circle is an initiative that showcases third-party applications built with GNOME technologies (GTK 4, libadwaita) and adhering to GNOME's design guidelines. These apps look native, integrate well, and are maintained by independent developers.

Fedora 44 ships several GNOME Circle apps by default, and you can discover more through GNOME Software or the GNOME Circle website.

New GNOME Circle additions around the GNOME 50 timeframe include:
App	Purpose
Gradia	Image processing and gradient generation
Constrict	File compression and archive management
Sessions	Session and window state management

These apps appear in GNOME Software alongside official GNOME applications, making the ecosystem feel unified.

    💡 Discovering New Apps

    Open GNOME Software → Browse. The curated selection includes GNOME Circle apps, popular Flatpaks from Flathub, and Fedora-packaged applications. This is your "app store" — explore it to find tools for productivity, creativity, development, and entertainment.

Putting It All Together: A Typical Workflow

Let's trace a realistic workflow to show how GNOME's pieces fit together:
Scenario: Writing a Document While Researching Online

    Boot and log in. Type your LUKS passphrase, reach GDM, type your user password. GNOME 50 desktop appears.

    Open Firefox. Press Super. Type "fire" and press Enter. Firefox opens on Workspace 1.

    Navigate to a research website. Browser fills the workspace.

    Open LibreOffice Writer. Press Super+Tab to switch focus (or press Super to enter Overview, then click Writer — or press Super, type "writer", Enter). Writer opens as a new window.

    Put Writer on a second workspace. Press Shift+Super+↑ to move Writer to Workspace 2. Three-finger swipe (or Super+↑) to follow it. Writer now has a full workspace to itself.

    Research. Three-finger swipe down to return to Workspace 1 with Firefox. Read, copy relevant text (Ctrl+C).

    Write. Three-finger swipe up to Workspace 2 with Writer. Paste (Ctrl+V). Continue writing.

    Side-by-side instead? Go back to Workspace 1, press Super+← to tile Firefox to the left half. Drag Writer from Workspace 2 back to Workspace 1 (or press Shift+Super+↓). Press Super+→ to tile Writer to the right half. Now both are visible side by side.

    Take a screenshot. Press PrtSc. The screenshot UI appears. Select a region, capture it. A notification appears: "Screenshot saved to Pictures."

    Lock for a break. Press Super+L. Screen locks. Come back, swipe up, type password. Your windows are exactly as you left them.

This workflow used: Activities search, workspaces, tiling, three-finger gestures, keyboard shortcuts, notifications, and the lock screen — all GNOME-native, all integrated. No third-party tools, no extensions needed.
What's Next

You now know every element of the GNOME 50 desktop: the top bar, Quick Settings, calendar and notifications, Activities Overview, search, dash, app grid, workspaces, window management, tiling and snap assist, keyboard shortcuts, touchpad gestures, file management, Settings panels, the lock screen, and what's new in this release.

In Chapter 10, we'll explore software management through GNOME's graphical interface — how to install, update, and remove applications using GNOME Software, with both RPM and Flatpak packages. This bridges the gap between the desktop experience you've learned in this chapter and the package management concepts you'll use throughout the rest of this manual.
Try It Yourself

    Master the Overview. Spend five minutes doing nothing but opening the Activities Overview (Super key), searching for apps, launching them, and closing them. Build muscle memory for the Super key — it's the single most important key in GNOME.

    Practice workspaces. Open three applications. Put each on its own workspace using Shift+Super+↑. Now switch between them with Super+↑/Super+↓ and three-finger swipes. Add a fourth app and tile two together on one workspace. Feel how workspaces reduce clutter.

    Use the tiling assistant. Open two applications. Press Super+← to tile one to the left half. Notice the snap-assist preview showing other open windows. Click one to fill the right half. Now you have side-by-side windows without resizing anything manually.

    Customize the dash. Open the app grid (⊞), find three apps you use most, drag them to the dash as favorites. Remove any default favorites you don't use. Now pressing Super shows your most-used apps right at the bottom — one click or Super+number away.

    Explore Settings. Open Settings and click through every panel. Don't change anything yet — just read what's available. Knowing what exists in Settings saves you time later. Pay special attention to Displays (VRR and fractional scaling are now on by default), Accessibility (try the Reduced Motion toggle), and Privacy.

    Take a screenshot. Press PrtSc. Try all three modes: full screen, window, and region. Find the saved screenshots in Pictures/Screenshots.

    Use gestures (if on a laptop). Do everything in exercises 1–4 using only touchpad gestures and the keyboard. Don't touch the mouse. Three-finger swipe up for Overview, left/right for workspaces, tap to click, two-finger scroll. If you practice this for a day, you'll never want to go back to a mouse.

    Try GNOME Classic (optional). Log out. At the login screen, click the gear icon and select "GNOME Classic." Log in. Explore the traditional two-panel layout. When you're done, log out and select "GNOME" again. Understanding both modes helps you choose the one that fits your workflow.

    ✅ Chapter 9 Complete You understand GNOME 50's design philosophy (get out of the way, keyboard/gesture first, focus-oriented workspaces), the top bar (Activities, clock, system status), Quick Settings panel (volume, brightness, Wi-Fi, Bluetooth, power profiles, Super+S shortcut), calendar and notification panel (notifications, Do Not Disturb, calendar, world clocks), the Activities Overview (search, dash, app grid, workspaces), dynamic workspaces and how to navigate them, window management (header bars, tiling, snap assist, keyboard shortcuts), the full keyboard shortcut reference, touchpad gestures, file management with Nautilus (sidebar, operations, GNOME 50 improvements), all Settings panels with GNOME 50 highlights (VRR, fractional scaling, parental controls, accessibility, Reduced Motion), the lock screen, GNOME Circle ecosystem, and a realistic end-to-end workflow demonstrating how these elements work together.

