# Chapter 25: Desktop Customization

## Making Ubuntu Yours

Ubuntu 26.04 ships with GNOME 50 — a clean, modern desktop that works well out of the box. But "works well" isn't the same as "works the way you want." Maybe you want the dock on the bottom. Maybe you want a traditional application menu instead of the full-screen app grid. Maybe you want different fonts, darker themes, or custom keyboard shortcuts.

GNOME is built to be customizable. This chapter covers the tools and techniques to reshape your desktop: GNOME Tweaks, the Extensions app, `gsettings` from the terminal, themes and icons, dock configuration, keyboard shortcuts, startup applications, and performance considerations.

Everything here is reversible. If a change breaks something, you can undo it. Experiment confidently.

## GNOME Tweaks: The Customization Hub

GNOME's built-in Settings app handles the basics — display resolution, Wi-Fi, sound, keyboard layout. For deeper customization, you need **GNOME Tweaks** (formally `gnome-tweaks`).

### Installing GNOME Tweaks

sudo apt install gnome-tweaks

Open it from the app grid (search "Tweaks") or from the terminal:

gnome-tweaks

What GNOME Tweaks Controls

The app is organized into panels on the left:

Appearance:

    Icons — System icon theme (Adwaita, Yaru, custom themes you install)
    Cursor — Mouse cursor theme
    Shell — GNOME Shell theme (requires the User Themes extension — see below)
    Sound — System sound theme
    Background — Desktop wallpaper
    Lock Screen — Lock screen image
    Window Titlebars — Titlebar button placement (left or right), which buttons to show (close, minimize, maximize)
    Applications — GTK theme (controls how application windows look)

Fonts:

    Interface Text — Font used in menus, buttons, and UI elements
    Document Text — Default font for documents
    Monospace Text — Font used in terminal and code editors
    Legacy Window Titles — Font for window titlebars (legacy apps)
    Antialiasing — Font smoothing method (RGBA recommended for LCD screens)
    Hinting — Font hinting (Slight recommended for most displays)

Windows:

    Window Titlebars — Toggle minimize/maximize buttons, button placement
    Focus on Hover — Focus windows when the mouse hovers over them
    Attach Modal Dialogs — Keep dialog windows attached to their parent

Startup Applications:

    Add or remove applications that launch automatically when you log in

Top Bar:

    Clock — Show date, seconds, weekday
    Calendar — Show week numbers
    Power — Show battery percentage (if on a laptop)

Extensions:

    Toggle installed extensions on/off
    Access extension settings (gear icon)

    💡 Tweaks vs. Settings

    GNOME Settings (gnome-control-center) and GNOME Tweaks (gnome-tweaks) complement each other. Settings handles hardware, accounts, and display configuration. Tweaks handles appearance, fonts, window behavior, and extensions. If you can't find a setting in one, check the other.

GNOME Shell Extensions

Extensions are small add-ons that modify the GNOME Shell — the top bar, dock, app grid, window management, and other desktop elements. They're GNOME's primary customization mechanism, allowing functionality that the GNOME project doesn't include by default.
Pre-Installed Extensions on Ubuntu 26.04

Ubuntu ships with several extensions already installed and active:
Extension	What It Does
Ubuntu Dock	The dock (taskbar) on the left side of the screen
AppIndicators	Shows tray icons in the top bar (for apps like Discord, Steam)
Ding	Places a "Files" (home folder) icon on the desktop
Tiling Assistant	Snap windows to halves/quarters of the screen
The Extensions App

Ubuntu 26.04 includes the Extensions app (package: gnome-shell-extension-manager). Open it by pressing Super, typing "Extensions," and pressing Enter.

The Extensions app shows:

    Installed — All extensions on your system, with on/off toggles
    Browse — Search and install extensions from extensions.gnome.org

Toggle extensions on/off with the switch next to each one. Some extensions have a gear icon — click it to access that extension's settings.
Installing Extensions from extensions.gnome.org

The official GNOME Extensions website (extensions.gnome.org) hosts thousands of community-created extensions.

Method 1: Browser Integration (Traditional)

Install the browser connector:

sudo apt install chrome-gnome-shell

Then visit extensions.gnome.org in Firefox or Chrome. You'll be prompted to install a browser add-on. Once installed, each extension page shows an on/off toggle — click it to install and enable the extension.

Method 2: Extensions App (Recommended)

Open the Extensions app, go to the Browse tab, and search for extensions directly. Click "Install" to add them. This avoids the browser integration entirely and is simpler.
Popular Extensions Worth Knowing
Extension	What It Does
User Themes	Enables custom GNOME Shell themes (required for shell theming)
Blur My Shell	Adds blur effects to the top bar, dash, and app grid
Dash to Dock	Enhanced dock with more options than Ubuntu's default
ArcMenu	Traditional application menu (Windows/macOS style)
Just Perfection	Fine-tune GNOME Shell UI — hide elements, resize panel, move clock
GSConnect	KDE Connect integration — sync with Android phone
Clipboard Indicator	Clipboard history with searchable dropdown
Caffeine	Quick toggle to disable screen auto-suspend/screensaver
OpenWeather	Weather indicator in the top bar
Vitals	System monitoring (CPU, temp, RAM) in the top bar

    ⚠️ Extensions and GNOME Version Compatibility

    Each extension declares which GNOME Shell versions it supports. When a new GNOME version ships (like GNOME 50), extension authors must update their extensions. Some extensions break temporarily after major GNOME updates.

    If an extension suddenly stops working after an update, check its page on extensions.gnome.org — the author may not have updated it yet. The Extensions app shows incompatible extensions with a warning icon.

Managing Extensions from the Terminal

List installed extensions:

gnome-extensions list

Show extension info:

gnome-extensions info ubuntu-dock@ubuntu.com

Enable/disable an extension:

gnome-extensions enable ubuntu-dock@ubuntu.com
gnome-extensions disable ubuntu-dock@ubuntu.com

Show which GNOME Shell version you're running:

gnome-shell --version

gsettings: Customization from the Terminal

Behind GNOME's graphical settings lies GSettings — a configuration system storing preferences as key-value pairs. Every setting you change in Tweaks or Settings corresponds to a gsettings command.

Using gsettings lets you script customizations, set up new machines quickly, and access settings that aren't exposed in graphical tools.
Basic gsettings Syntax

gsettings get SCHEMA KEY
gsettings set SCHEMA KEY VALUE
gsettings reset SCHEMA KEY

    SCHEMA — A settings category (e.g., org.gnome.desktop.interface)
    KEY — The specific setting (e.g., gtk-theme)
    VALUE — The new value (strings need quotes)

Useful gsettings Commands

Change the GTK theme:

gsettings set org.gnome.desktop.interface gtk-theme 'Yaru-dark'

Change the icon theme:

gsettings set org.gnome.desktop.interface icon-theme 'Yaru'

Change the cursor theme:

gsettings set org.gnome.desktop.interface cursor-theme 'Yaru'

Enable dark mode (color scheme):

gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

Set the monospace font:

gsettings set org.gnome.desktop.interface monospace-font-name 'Monospace 12'

Show battery percentage on the top bar:

gsettings set org.gnome.desktop.interface show-battery-percentage true

Show the date in the top bar clock:

gsettings set org.gnome.desktop.interface clock-show-date true

Show seconds in the clock:

gsettings set org.gnome.desktop.interface clock-show-seconds true

Show week numbers in the calendar:

gsettings set org.gnome.desktop.calendar show-weekdate true

Enable tap-to-click on touchpads:

gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true

Disable animations (performance):

gsettings set org.gnome.desktop.interface enable-animations false

Discovering Available Settings

List all keys in a schema:

gsettings list-keys org.gnome.desktop.interface

See the current value of all keys in a schema:

gsettings list-recursively org.gnome.desktop.interface

List all schemas:

gsettings list-schemas | grep desktop

Find which schema contains a specific key:

gsettings list-recursively | grep "clock-show"

    💡 Reset Any Setting

    If you change a setting and things break, reset it to default:

    gsettings reset org.gnome.desktop.interface gtk-theme

    Or reset everything in an entire schema:

    gsettings reset-recursively org.gnome.desktop.interface

dconf: The Low-Level Editor

Underneath gsettings is dconf — the actual database storing GNOME settings. The dconf editor provides a graphical tree view of every setting:

sudo apt install dconf-editor
dconf-editor

Navigate the tree to find settings buried deep in GNOME's configuration. Be cautious — changing settings in dconf-editor takes effect immediately and there's no "Apply" button. The reset button is always available if you make a mistake.
Dock Customization

Ubuntu's dock (the taskbar on the left side by default) is highly configurable. While you can change basic settings by right-clicking the dock and selecting "Dock Settings," gsettings gives you full control.
Dock Settings via gsettings

The dock's settings live under the org.gnome.shell.extensions.dash-to-dock schema.

Move the dock to the bottom:

gsettings set org.gnome.shell.extensions.dash-to-dock dock-position 'BOTTOM'

Options: 'LEFT', 'RIGHT', 'BOTTOM'.

Change the icon size:

gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size 48

Default is 48. Smaller values (32, 36) fit more icons. Larger values (64, 72) are easier to click.

Enable auto-hide:

gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed false

With auto-hide, the dock slides out of view when not in use and reappears when you move the mouse to the screen edge.

Show the dock on all monitors:

gsettings set org.gnome.shell.extensions.dash-to-dock multi-monitor true

Center the dash icons:

gsettings set org.gnome.shell.extensions.dash-to-dock dash-alignment 'CENTER'

Options: 'START', 'CENTER', 'END'.

Move the "Show Apps" button to the top:

gsettings set org.gnome.shell.extensions.dash-to-dock show-apps-at-top true

Change dock transparency:

gsettings set org.gnome.shell.extensions.dash-to-dock transparency-mode 'FIXED'

Options: 'DEFAULT' (adapts to wallpaper), 'FIXED' (constant opacity), 'DYNAMIC' (changes with active window).

Set custom dock background opacity:

gsettings set org.gnome.shell.extensions.dash-to-dock background-opacity 0.5

Values from 0.0 (fully transparent) to 1.0 (fully opaque).

    💡 Discover All Dock Settings

    Run:

    gsettings list-recursively org.gnome.shell.extensions.dash-to-dock

    This lists every dock setting available — you'll find options for pressure sensitivity, scroll action, click behavior, and more.

Themes and Icons

Themes change the visual appearance of applications, windows, icons, and the GNOME Shell itself. Ubuntu 26.04 ships with the Yaru theme — a distinctive orange/purple Ubuntu look.
Theme Types
Theme Type	What It Changes	Set Via
GTK theme	Application windows, buttons, dialogs	Tweaks > Appearance > Applications
Icon theme	Folder, file, and app icons	Tweaks > Appearance > Icons
Cursor theme	Mouse pointer appearance	Tweaks > Appearance > Cursor
GNOME Shell theme	Top bar, app grid, notifications	Tweaks > Appearance > Shell (requires User Themes extension)
Sound theme	System event sounds	Tweaks > Appearance > Sound
Installing Third-Party Themes

Themes are stored in specific directories:
Directory	Purpose
~/.themes/	User-installed GTK and Shell themes
~/.local/share/themes/	Alternative user theme location (newer standard)
~/.icons/	User-installed icon themes
~/.local/share/icons/	Alternative icon location
/usr/share/themes/	System-wide themes (requires sudo to modify)
/usr/share/icons/	System-wide icon themes

Installing a theme from a downloaded archive:

    Download the theme archive (usually a .tar.gz or .zip file)
    Extract it:

    mkdir -p ~/.themes
    tar -xf MyTheme.tar.gz -C ~/.themes/

    Open GNOME Tweaks → Appearance → Applications → select the theme
    For Shell themes, enable the User Themes extension first (Extensions app → search "User Themes" → toggle on)

Installing an icon theme:

mkdir -p ~/.icons
tar -xf MyIcons.tar.gz -C ~/.icons/

Then select it in Tweaks → Appearance → Icons.
Applying Themes from the Terminal

# GTK theme
gsettings set org.gnome.desktop.interface gtk-theme 'MyTheme'

# Icon theme
gsettings set org.gnome.desktop.interface icon-theme 'MyIcons'

# Cursor theme
gsettings set org.gnome.desktop.interface cursor-theme 'MyCursor'

# Shell theme (requires User Themes extension)
gsettings set org.gnome.shell.extensions.user-theme name 'MyShellTheme'

# Dark mode
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'

Popular Theme Recommendations
Theme	Style	Notes
Yaru (default)	Ubuntu's signature look	Comes pre-installed
Adwaita	Clean GNOME upstream	Light only, pre-installed
Adwaita-dark	Dark variant of Adwaita	Pre-installed
WhiteSur	macOS Big Sur style	GTK + Shell + Icons
Catppuccin	Pastel, soft colors	Popular developer aesthetic
Nordic	Cold blue palette	Calm, minimalist
Orchis	Rounded, material-inspired	Clean and modern

    ⚠️ Theme Compatibility

    GTK themes designed for GTK3 may not fully support GTK4/Libadwaita applications (which GNOME 50 uses extensively). Libadwaita apps (like GNOME's built-in apps) respect the system color scheme (prefer-dark) but may not fully honor custom GTK themes. This is a deliberate GNOME design decision — Libadwaita enforces visual consistency. Third-party applications using older GTK3 or Qt will still honor your GTK theme.

Keyboard Shortcuts

Ubuntu comes with dozens of keyboard shortcuts. You can view and customize them in Settings → Keyboard, but gsettings gives you access to everything.
Default Shortcuts Worth Knowing
Shortcut	Action
Super	Open Activities overview / app grid
Super + D	Show desktop (minimize all)
Super + Left/Right	Snap window to left/right half
Super + Up	Maximize window
Super + Down	Restore/minimize window
Super + 1-9	Switch to workspace 1-9
Super + Shift + Left/Right	Move window to adjacent workspace
Alt + Tab	Switch between windows
Alt + (backtick)	Switch between windows of same app
Ctrl + Alt + T	Open terminal (Ubuntu default)
Super + L	Lock screen
Super + A	Show all applications
Super + V	Show notification/calendar
PrtSc	Full screenshot
Shift + PrtSc	Selected area screenshot
Alt + PrtSc	Screenshot of current window
Ctrl + Alt + Arrow keys	Switch workspace
Customizing Shortcuts via gsettings

Keyboard shortcuts are stored in two main schemas:

    org.gnome.desktop.wm.keybindings — Window management shortcuts
    org.gnome.settings-daemon.plugins.media-keys — System media shortcuts

View current keybindings:

gsettings list-recursively org.gnome.desktop.wm.keybindings

Change a window management shortcut:

gsettings set org.gnome.desktop.wm.keybindings switch-to-workspace-1 "['<Super>1']"

Change the terminal shortcut:

gsettings set org.gnome.settings-daemon.plugins.media-keys terminal "['<Primary><Alt>t']"

Custom keybinding (run a command on a shortcut):

Custom shortcuts require a two-part setup:

# Create a custom shortcut entry
gsettings set org.gnome.settings-daemon.plugins.media-keys custom-keybindings "['/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/']"

# Set the name
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ name 'Open System Monitor'

# Set the command
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ command 'gnome-system-monitor'

# Set the key combination
gsettings set org.gnome.settings-daemon.plugins.media-keys.custom-keybinding:/org/gnome/settings-daemon/plugins/media-keys/custom-keybindings/custom0/ binding '<Primary><Alt>m'

Now Ctrl+Alt+M opens the system monitor.
Key Naming Convention

gsettings uses a specific notation for key combinations:
Notation	Key
<Super>	Super/Windows key
<Primary> or <Control>	Ctrl
<Alt>	Alt
<Shift>	Shift
<Super>1	Super + 1
<Primary><Alt>t	Ctrl + Alt + T
<Super><Shift>Left	Super + Shift + Left arrow

    💡 Graphical Shortcut Editor

    For most users, Settings → Keyboard → View and Customize Shortcuts is easier than gsettings. Use gsettings when you need to script shortcut changes across multiple machines or when a setting isn't visible in the GUI.

Startup Applications

Controlling which applications launch at login.
Graphical Method

Open GNOME Tweaks → Startup Applications. Click the + button to add an application from your installed apps. Toggle switches to enable/disable.
Terminal Method

Startup applications are .desktop files in ~/.config/autostart/:

List startup applications:

ls ~/.config/autostart/

Add a startup application manually:

nano ~/.config/autostart/myscript.desktop

Content:

[Desktop Entry]
Type=Application
Name=My Startup Script
Exec=/home/john/bin/system-report.sh
Terminal=false
X-GNOME-Autostart-enabled=true

    Type=Application — It's an application entry
    Exec= — The command to run
    Terminal=false — Don't open a terminal window (use true if the script needs terminal interaction)
    X-GNOME-Autostart-enabled=true — Enable auto-start

Disable a startup application:

echo "X-GNOME-Autostart-enabled=false" >> ~/.config/autostart/unwanted.desktop

Or simply delete it:

rm ~/.config/autostart/unwanted.desktop

    💡 Delayed Startup

    If an app needs to start slightly after login (to let the desktop settle), add a delay:

    X-GNOME-Autostart-Delay=5

    This delays launch by 5 seconds. Useful for apps that fail when they start too early.

Customization Script: Apply All Settings at Once

If you reinstall Ubuntu or set up a new machine, manually reapplying every customization is tedious. Here's a script that applies common GNOME settings in one shot:

#!/bin/bash
set -euo pipefail

# customize-gnome.sh — Apply my preferred GNOME settings
# Run after a fresh Ubuntu install

echo "Applying GNOME customizations..."

# --- Appearance ---
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
gsettings set org.gnome.desktop.interface gtk-theme 'Yaru-dark'
gsettings set org.gnome.desktop.interface icon-theme 'Yaru'
gsettings set org.gnome.desktop.interface cursor-theme 'Yaru'

# --- Fonts ---
gsettings set org.gnome.desktop.interface font-name 'Ubuntu 11'
gsettings set org.gnome.desktop.interface monospace-font-name 'Ubuntu Mono 13'
gsettings set org.gnome.desktop.interface document-font-name 'Ubuntu 11'

# --- Top Bar ---
gsettings set org.gnome.desktop.interface clock-show-date true
gsettings set org.gnome.desktop.interface clock-show-seconds true
gsettings set org.gnome.desktop.interface clock-show-weekday true
gsettings set org.gnome.desktop.calendar show-weekdate true

# Show battery percentage (laptops)
gsettings set org.gnome.desktop.interface show-battery-percentage true

# --- Touchpad ---
gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true
gsettings set org.gnome.desktop.peripherals.touchpad natural-scroll true
gsettings set org.gnome.desktop.peripherals.touchpad two-finger-scrolling true

# --- Dock ---
gsettings set org.gnome.shell.extensions.dash-to-dock dock-position 'BOTTOM'
gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size 40
gsettings set org.gnome.shell.extensions.dash-to-dock dash-alignment 'CENTER'
gsettings set org.gnome.shell.extensions.dash-to-dock multi-monitor true
gsettings set org.gnome.shell.extensions.dash-to-dock transparency-mode 'DYNAMIC'

# --- Window Management ---
gsettings set org.gnome.desktop.wm.preferences button-layout 'close,minimize,maximize:'

# --- Keyboard Shortcuts ---
# Open terminal with Ctrl+Alt+T
gsettings set org.gnome.settings-daemon.plugins.media-keys terminal "['<Primary><Alt>t']"

# --- Mouse ---
gsettings set org.gnome.desktop.peripherals.mouse natural-scroll false

# --- File Manager (Nautilus) ---
gsettings set org.gnome.nautilus.preferences show-hidden-files false
gsettings set org.gnome.nautilus.preferences click-policy 'double'

echo "Done! Some changes may require restarting GNOME Shell."
echo "Press Alt+F2, type 'r', and press Enter to restart the shell (Wayland requires re-login)."

Save this as ~/bin/customize-gnome.sh, make it executable, and run it after every fresh install:

chmod +x ~/bin/customize-gnome.sh
./customize-gnome.sh

    ⚠️ Restarting GNOME Shell

    On X11 sessions, you can restart GNOME Shell with Alt+F2, type r, press Enter. This reloads the shell without closing your applications. On Wayland (Ubuntu 26.04's default), this doesn't work — you'll need to log out and log back in. Shell theme changes and some extension changes require a shell restart.

Performance Considerations

Customization can affect system performance. Here's what to watch for:
Extensions Impact

Each extension runs JavaScript code inside the GNOME Shell process. Too many extensions — especially ones that constantly poll (weather, system monitors, animations) — can cause:

    Increased RAM usage by the gnome-shell process
    CPU spikes during animations
    Input lag on the top bar
    Reduced battery life on laptops

Check GNOME Shell's resource usage:

top -p $(pgrep gnome-shell)

If gnome-shell is using over 500 MB of RAM or significant CPU, you may have too many extensions or a poorly written one.

Identify problematic extensions:

Disable all extensions:

gsettings set org.gnome.shell disable-user-extensions true

Re-enable them one by one, checking performance after each, to isolate the culprit.
Animations

GNOME's animations look smooth but consume GPU resources. Disabling them improves performance on older hardware:

gsettings set org.gnome.desktop.interface enable-animations false

This removes transitions, fade effects, and window animations. Everything still works — it's just snappier and less pretty.
Performance Tuning Tips
Tip	Command / Action
Disable unnecessary extensions	Extensions app → toggle off what you don't use
Turn off animations	gsettings set org.gnome.desktop.interface enable-animations false
Reduce dock icon size	Lower value = less rendering work
Use fewer workspaces	Settings → Multitasking → Fixed workspaces (fewer = less compositing)
Avoid heavy blur extensions	Blur My Shell is beautiful but GPU-intensive
Monitor gnome-shell memory	Watch top -p $(pgrep gnome-shell)
Customization Cheat Sheet
Task	Command or Action
Install GNOME Tweaks	sudo apt install gnome-tweaks
Install Extensions app	sudo apt install gnome-shell-extension-manager
List extensions	gnome-extensions list
Enable extension	gnome-extensions enable extension@uuid
Disable extension	gnome-extensions disable extension@uuid
Dark mode	gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'
Change GTK theme	gsettings set org.gnome.desktop.interface gtk-theme 'Theme'
Change icon theme	gsettings set org.gnome.desktop.interface icon-theme 'Icons'
Change cursor theme	gsettings set org.gnome.desktop.interface cursor-theme 'Cursor'
Set shell theme	gsettings set org.gnome.shell.extensions.user-theme name 'Shell'
Show date in clock	gsettings set org.gnome.desktop.interface clock-show-date true
Show seconds in clock	gsettings set org.gnome.desktop.interface clock-show-seconds true
Show battery %	gsettings set org.gnome.desktop.interface show-battery-percentage true
Tap-to-click touchpad	gsettings set org.gnome.desktop.peripherals.touchpad tap-to-click true
Natural scrolling	gsettings set org.gnome.desktop.peripherals.touchpad natural-scroll true
Dock to bottom	gsettings set org.gnome.shell.extensions.dash-to-dock dock-position 'BOTTOM'
Dock icon size	gsettings set org.gnome.shell.extensions.dash-to-dock dash-max-icon-size 48
Dock auto-hide	gsettings set org.gnome.shell.extensions.dash-to-dock dock-fixed false
Dock centered	gsettings set org.gnome.shell.extensions.dash-to-dock dash-alignment 'CENTER'
Disable animations	gsettings set org.gnome.desktop.interface enable-animations false
Window button layout	gsettings set org.gnome.desktop.wm.preferences button-layout 'close,minimize,maximize:'
List all gsettings	gsettings list-recursively org.gnome.desktop.interface
Reset a setting	gsettings reset SCHEMA KEY
Install dconf Editor	sudo apt install dconf-editor
List startup apps	ls ~/.config/autostart/
What's Next

You now have the tools to transform Ubuntu's desktop from default to deeply personalized. GNOME Tweaks handles appearance, fonts, and window behavior. The Extensions app adds functionality GNOME doesn't include by default. gsettings and dconf give you terminal-level control over every aspect of the desktop. Themes and icons change the visual identity. Custom keyboard shortcuts and startup applications tailor the experience to your workflow. And the customization script lets you reproduce your setup on any new machine in seconds.

In Chapter 26, we'll cover backup and recovery — protecting your data with rsync, tar, Timeshift system snapshots, and disaster recovery strategies.
Try It Yourself

    Install the customization tools. Run sudo apt install gnome-tweaks gnome-shell-extension-manager dconf-editor. Open GNOME Tweaks and browse through each panel.

    Apply dark mode. Run gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'. Notice the immediate change. Toggle it back with 'default' if you prefer light mode.

    Customize your dock. Try moving it to the bottom, centering the icons, and adjusting the icon size. Find a configuration that feels comfortable.

    Enable the clock date and seconds. Run the two gsettings commands from the Top Bar section. Check the top bar — both should appear immediately.

    Browse GNOME extensions. Open the Extensions app, go to the Browse tab. Search for "Clipboard Indicator" or "Caffeine." Install one and toggle it on. Explore its settings.

    Create a custom shortcut. Use the gsettings custom keybinding sequence to bind Ctrl+Alt+M to opening your system monitor (or another app you use frequently).

    List your startup applications. Check ~/.config/autostart/ for existing entries. Add a .desktop file for a script or app you want at login.

    Save your customization script. Copy the customize-gnome.sh script from this chapter. Modify it with your own preferences (favorite theme, dock position, shortcuts). Save it in ~/bin/. This is your personal Ubuntu setup script for future reinstalls.

    Check performance. Run top -p $(pgrep gnome-shell) and note the memory usage. If you have many extensions enabled, try disabling a few and see if memory drops.

    ✅ Chapter 25 Complete You can install and use GNOME Tweaks, manage GNOME Shell extensions (pre-installed and from extensions.gnome.org), control every desktop setting via gsettings, apply themes and icons, customize the dock, set keyboard shortcuts, manage startup applications, tune performance, and reproduce your entire setup with a single script. In Chapter 26, we'll protect your data with backups and recovery.

