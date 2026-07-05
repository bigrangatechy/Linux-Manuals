# Chapter 8: Understanding Desktop Environments

## The Layer Cake

You've installed Fedora, configured it, and you're using GNOME 50. When you move your mouse, click windows, type in the Activities search, and switch workspaces, you're interacting with software that sits between you and the raw operating system. That software is called a **desktop environment** — and understanding what it is, how it works, and what alternatives exist is fundamental to understanding Linux itself.

On Windows and macOS, the desktop is inseparable from the operating system. Windows *is* the Windows desktop; macOS *is* the macOS desktop. There's no "choose your desktop" option. You get what the vendor ships.

Linux is fundamentally different. The desktop is a user-space application — one of many possible desktops — running on top of the operating system. You can remove it, replace it, or run multiple ones. The kernel, the filesystem, the package manager, the network stack — none of these care which desktop you use. They're layers below.

This chapter explains the architecture, introduces the major desktop environments available for Fedora, and helps you understand what you're looking at when you use GNOME 50.

---

## The Linux Desktop Stack: Top to Bottom

Let's visualize the full software stack on your Fedora system, from your eyeballs down to the silicon:

┌─────────────────────────────────────────────────┐ │ YOU │ ← You, the user ├─────────────────────────────────────────────────┤ │ APPLICATIONS │ ← Firefox, Files, Settings, Terminal │ (Firefox, Nautilus, Gedit, etc.) │ ├─────────────────────────────────────────────────┤ │ DESKTOP ENVIRONMENT │ ← GNOME Shell, panels, workspaces, │ (GNOME Shell, top bar, Activities, etc.) │ notifications, app grid ├─────────────────────────────────────────────────┤ │ WINDOW MANAGER / COMPOSITOR │ ← Mutter (draws windows, composites, │ (Mutter) │ handles input, manages displays) ├─────────────────────────────────────────────────┤ │ DISPLAY SERVER PROTOCOL │ ← Wayland (communication protocol │ (Wayland) │ between compositor and kernel) ├─────────────────────────────────────────────────┤ │ TOOLKITS / LIBRARIES │ ← GTK, libadwaita, Qt, Mesa (OpenGL/Vulkan) │ (GTK 4, libadwaita, Qt, Mesa) │ ├─────────────────────────────────────────────────┤ │ GRAPHICS DRIVERS │ ← Kernel-mode setting (KMS), DRM, │ (Kernel KMS, DRM, GPU drivers) │ Intel/AMD/NVIDIA drivers ├─────────────────────────────────────────────────┤ │ KERNEL │ ← Linux 6.19 (hardware abstraction, │ (Linux 6.19) │ scheduling, memory, filesystems) ├─────────────────────────────────────────────────┤ │ HARDWARE │ ← CPU, RAM, GPU, SSD, network card └─────────────────────────────────────────────────┘

Each layer talks to the one below it. Your click in Firefox travels down through GNOME Shell, through Mutter, through Wayland, through the toolkit, through the kernel, to the hardware — and the visual result travels back up the same chain in milliseconds.

The key insight: every layer above the kernel is replaceable. Swap GNOME for KDE Plasma, and only the top three layers change. The kernel, drivers, toolkit libraries (well, partly), and applications remain the same.
What Is a Desktop Environment?

A desktop environment (DE) is a collection of software that provides the graphical user interface you interact with. It typically includes:
Component	In GNOME 50	What It Does
Shell	GNOME Shell	The top bar, Activities Overview, app grid, workspaces, notifications, search
Window manager	Mutter	Draws window borders, handles moving/resizing, composites the screen
Display manager	GDM (GNOME Display Manager)	The login screen
File manager	Nautilus (GNOME Files)	Browse and manage files
Settings	GNOME Settings (gnome-control-center)	System configuration GUI
Terminal	GNOME Terminal	Command-line interface
Text editor	GNOME Text Editor	Simple text editing
Calculator	GNOME Calculator	Arithmetic
Screenshot tool	Built-in	Capture screen images
Software center	GNOME Software	Install/remove applications
Core apps	Calendar, Contacts, Maps, Weather, Clocks, Music, Photos, Videos	GNOME application suite
Theme engine	libadwaita	Consistent theming for GTK 4 applications
Accessibility	Orca, magnifier, on-screen keyboard	Assistive technologies

When you install "GNOME," you get all of these as a cohesive package. They share a design language, integrate with each other, and are developed by (mostly) the same community.
The Analogy

Think of a desktop environment as a house interior. The kernel is the foundation and frame — concrete, studs, roof. The display server is the electrical and plumbing infrastructure — hidden but essential. The desktop environment is everything you see and touch — the walls, paint, furniture, appliances, lighting fixtures.

You can live in the same house with a completely different interior. Move out the GNOME furniture, move in KDE Plasma furniture. The foundation hasn't changed. The plumbing works the same. But the living experience is completely different.
What Is a Window Manager?

A window manager (WM) is the component responsible for drawing windows on screen — the borders, title bars, close buttons — and handling their behavior: how they move, resize, overlap, minimize, maximize, snap to edges, and respond to keyboard shortcuts.

Every desktop environment includes a window manager. In GNOME, it's Mutter. In KDE Plasma, it's KWin. In XFCE, it's xfwm4.

But you can also use a standalone window manager without a full desktop environment. This is popular among minimalists and power users who want to assemble their own desktop from components:
Type	Examples	Philosophy
Floating (stacking)	Openbox, Fluxbox, IceWM	Windows can overlap, be moved freely. Traditional behavior.
Tiling	Sway, i3, Hyprland, bspwm	Windows are arranged in non-overlapping tiles. Keyboard-driven.
Dynamic	dwm, awesome	Can switch between tiling and floating modes.

A standalone WM user might use Openbox as their window manager, tint2 as a taskbar, feh for wallpaper, rofi as an app launcher, and lxpanel for a system tray. It's a build-your-own approach — more work, but total control.

    💡 Desktop Environment vs. Window Manager

    A desktop environment is a complete package — window manager plus file manager, settings, panels, and a suite of applications. A window manager is just the piece that manages windows.

    GNOME and KDE Plasma are desktop environments. Openbox and Sway are window managers.

    You can use a WM inside a DE (some people replace Mutter with i3 inside GNOME), or you can use a WM standalone (no DE at all).

What Is a Display Server Protocol?

Between the window manager and the kernel sits the display server protocol — the communication standard that defines how graphical output is drawn to the screen and how input (keyboard, mouse, touch) is delivered to applications.
The Two Protocols
Protocol	Era	Status in Fedora 44
X11 (X Window System)	1984–present	Removed. Fedora dropped the X11 GNOME session entirely.
Wayland	2012–present	The only display protocol. Fedora has defaulted to Wayland since Fedora 25 (2016).
X11: The Grandfather

The X Window System (commonly "X11" or just "X") was created at MIT in 1984 — predating Linux by seven years. It was designed for a world of network transparency: applications could run on one computer and display on another over the network.

X11 is a server-client architecture:

    The X server runs on the machine with the display hardware
    X clients (applications) connect to the server and send drawing commands
    The server renders those commands to the screen and sends input events back

This architecture was flexible and powerful but accumulated decades of complexity. The X server had direct access to all input and output — a security concern. Every application could see every other application's keystrokes and screen content. The codebase was enormous, hard to maintain, and resistant to modern features like HiDPI scaling and per-monitor color management.
Wayland: The Replacement

Wayland is a modern display protocol that addresses X11's shortcomings. Rather than a monolithic server, Wayland defines a protocol where the compositor (which is also the window manager) talks directly to the kernel's graphics drivers via DRM/KMS (Direct Rendering Manager / Kernel Mode Setting).

X11 Architecture:
  App → X server → Kernel → Hardware
        ↑
  (X server has full access to all input/output)

Wayland Architecture:
  App → Compositor (Mutter) → Kernel → Hardware
        ↑
  (Compositor mediates all access; apps are isolated)

Advantages of Wayland:
Feature	X11	Wayland
Security	Any app can read other apps' input and output	Apps are isolated; can't snoop on each other
Performance	Extra copy through X server	Direct rendering, lower latency
HiDPI	Partial, inconsistent	First-class, per-monitor scaling
Tear-free	Requires vsync hacks	Built-in, guaranteed
Multi-monitor	Mixed DPI was painful	Each monitor scales independently
Codebase	~700K lines, decades old	Lean protocol, compositor-dependent

    💡 Fedora and Wayland: Pioneer

    Fedora has been the leader in Wayland adoption:

        Fedora 25 (2016): First major distro to default to Wayland for GNOME
        Fedora 34 (2021): Wayland became the default for KDE Plasma as well
        Fedora 40 (2024): The legacy X11 GNOME session was removed. Wayland-only.

    Fedora 44 is Wayland-only. There is no X11 fallback. If an application requires X11, it runs through XWayland — a compatibility layer that translates X11 protocol to Wayland. Most users never notice this.

XWayland: The Bridge

Some applications haven't been ported to Wayland native yet. These run through XWayland — an X server that runs inside Wayland. The application thinks it's talking to X11; XWayland translates its output to Wayland compositing.

You can check which applications are running through XWayland:

xlsclients

Or check if a specific app is Wayland-native:

# If this outputs "wayland", the app is native
echo $WAYLAND_DISPLAY

Most major applications are now Wayland-native: Firefox, Chromium, LibreOffice, GNOME apps, VLC, OBS Studio. The main holdouts are older Electron apps and some games — but even these increasingly support Wayland.
Toolkits: How Applications Draw Themselves

Applications don't draw their buttons, menus, and text directly to the screen. They use a toolkit — a library that provides widgets (buttons, sliders, text fields, scrollbars) and handles rendering them.
GTK (GNOME's Toolkit)

GTK (originally "GIMP Toolkit") is the widget library used by GNOME and many other Linux applications. It's developed primarily by the GNOME project.

    GTK 4 — the current major version, used by GNOME 50 applications
    libadwaita — GNOME's design library built on top of GTK 4, providing the modern GNOME look (rounded corners, specific color palette, smooth animations)

Applications using GTK 4 + libadwaita have a consistent appearance that follows GNOME's design guidelines. They automatically adapt to light/dark mode, support HiDPI, and animate smoothly under Wayland.

Examples: GNOME Files, GNOME Settings, GNOME Text Editor, Firefox (partially), GIMP (migrating), Inkscape (migrating), Transmission.
Qt (KDE's Toolkit)

Qt is the other major Linux toolkit, developed by the Qt Company and used primarily by KDE Plasma. It's a comprehensive C++ framework that includes widgets, networking, multimedia, and more.

    Qt 6 — the current major version, used by KDE Plasma 6 and KDE applications
    Kirigami — KDE's component framework for adaptive applications (analogous to libadwaita)
    Breeze — KDE's default theme and icon set

Examples: Dolphin (KDE file manager), Konsole, Krita, OBS Studio, qBittorrent, Telegram Desktop, Calibre, VLC (uses Qt on Linux).
What This Means for You

Most users don't choose a toolkit — they choose a desktop environment, and the toolkit follows. But understanding toolkits explains why applications from different "worlds" sometimes look different:

    A GTK app running under GNOME looks native (libadwaita theming)
    A Qt app running under GNOME may look slightly different (it uses a GTK theme bridge, but it's not perfect)
    A GTK app running under KDE Plasma may look slightly different (Qt theming dominates)
    Most Flatpak/Flathub apps declare their toolkit, and GNOME Software respects it

    💡 Cross-Toolkit Theming Is Hard

    If you've ever noticed that some applications look "wrong" — different font, odd button shapes, mismatched colors — it's usually because they use a different toolkit than the desktop expects. KDE Plasma does a better job of theming GTK apps than GNOME does of theming Qt apps, because Plasma includes a dedicated GTK integration module.

    This isn't a bug — it's the nature of having multiple independent widget frameworks. The Flatpak ecosystem is gradually reducing this friction by bundling themes with applications.

The Major Desktop Environments

Fedora offers desktop environments through its editions, spins, and labs. Let's meet the major players.
GNOME
Attribute	Details
Default on	Fedora Workstation (the edition you're using)
Window manager	Mutter
Toolkit	GTK 4 + libadwaita
Display server	Wayland only
Philosophy	Minimal, distraction-free, keyboard-centric. Gets out of your way.
Designed by	GNOME Project (a large, distributed community)

GNOME's design philosophy emphasizes focus and simplicity. There's no traditional taskbar or desktop icons by default. The Activities Overview (triggered by the Super key) is the central hub for launching apps, switching windows, and managing workspaces.

GNOME 50 (released March 2026) brought significant visual changes: refined typography, smoother animations, a redesigned quick-settings panel, improved tiling snap assist, and tighter integration with online accounts.

Fedora ships vanilla GNOME — the upstream GNOME experience without modifications. This is notable: Ubuntu modifies GNOME with its Yaru theme, dock-by-default behavior, and custom wallpaper. Fedora gives you GNOME as the GNOME project intended.
KDE Plasma
Attribute	Details
Available via	Fedora KDE Spin (separate ISO)
Window manager	KWin
Toolkit	Qt 6
Display server	Wayland (X11 removed in Plasma 6)
Philosophy	Maximum customization. Every element is configurable.
Designed by	KDE Community

KDE Plasma is the yin to GNOME's yang. Where GNOME removes options to reduce clutter, KDE Plasma adds options to increase control. You can configure panel position, size, opacity, widgets, window rules, shortcuts, compositing effects, and dozens of other aspects.

KDE Plasma 6 (the version shipping with Fedora's KDE spin aligned with F44) is a mature, polished desktop with a traditional layout: a panel at the bottom (configurable), a start-menu-like application launcher, system tray, and desktop icons (optional).

If you're coming from Windows, KDE Plasma will feel most familiar. But GNOME is what Fedora Workstation ships, and this manual focuses on GNOME.
XFCE
Attribute	Details
Available via	Fedora XFCE Spin
Window manager	xfwm4
Toolkit	GTK 3 (migrating to GTK 4)
Philosophy	Lightweight, stable, traditional. Low resource usage.
Designed by	XFCE Team

XFCE is the lightweight champion. It uses minimal RAM and CPU, making it ideal for older hardware (10+ year-old laptops, netbooks) or users who want a snappy, no-frills desktop. It has a traditional panel layout (two panels by default), a basic but functional file manager (Thunar), and modest customization options.
Other Spins and Desktops

Fedora also offers spins for these desktop environments:
Spin	Desktop	Character
Cinnamon	Cinnamon	Windows-like layout. Developed by Linux Mint, but available on Fedora.
MATE	MATE	Continuation of GNOME 2 (the pre-2011 GNOME). Traditional, stable.
LXQt	LXQt	Ultra-lightweight. Uses Qt. For very old hardware.
Budgie	Budgie	Modern, elegant. Developed by Serpent OS project.
i3	i3 (tiling WM)	Keyboard-driven tiling window manager. For power users.
SOAS	Sugar	Designed for children and education. Originally for the OLPC project.

    💡 Spins vs. Workstation: Same Foundation

    Every Fedora spin shares the same underlying system: the same kernel, the same DNF5, the same repositories, the same SELinux, the same firewalld. The only difference is the desktop environment and its associated applications pre-installed.

    This means everything in this manual — the terminal commands, package management, networking, security, filesystem structure — applies identically regardless of which spin you use. The desktop tour (Chapter 9) is GNOME-specific, but Chapters 10 onward are universal.

Fedora's Desktop Ecosystem

Beyond the traditional editions and spins, Fedora offers several variants that rethink the traditional desktop model:
Silverblue (Immutable GNOME)

Fedora Silverblue is an "atomic" variant of Fedora Workstation. Instead of a traditional mutable filesystem where dnf install modifies the system directly, Silverblue uses an immutable base image — the root filesystem is read-only.
Feature	Workstation (traditional)	Silverblue (atomic)
System updates	dnf upgrade modifies system in place	New image deployed atomically; reboot switches
Application installation	dnf install (RPM) or Flatpak	Flatpak (primary), rpm-ostree install (layered)
Rollback	Requires Snapper setup	Built-in: boot previous deployment
Stability	Good, but updates can break things	Very high: atomic updates can't leave system half-broken
Target audience	General users, developers	Developers, enterprise workstations, stability-seekers

Silverblue uses the same GNOME desktop as Workstation — the difference is in the update mechanism, not the user interface. You'd interact with GNOME the same way; the underlying system management is different.
Kinoite (Immutable KDE)

Same concept as Silverblue but with KDE Plasma instead of GNOME. Same atomic update model, same immutable base, different desktop.
Other Atomic Variants
Variant	Desktop	Notes
Sway Atomic	Sway (tiling WM)	Immutable + tiling
Budgie Atomic	Budgie	Immutable + Budgie desktop

These atomic variants are covered briefly for awareness — they represent the cutting edge of Fedora's direction. For most new users, the traditional Workstation edition is the recommended starting point.
Fedora Labs

Fedora Labs are targeted software bundles that can be installed on any Fedora edition or spin. They're not separate desktops — they're curated application sets for specific use cases:
Lab	Purpose
Astronomy	Astrophysics and astronomy tools (Stellarium, KStars, etc.)
Comp Neuro	Computational neuroscience software
Design Suite	Graphic design and creative tools
Games	Gaming-focused software bundle
Python Classroom	Teaching Python programming
Robotics	Robotics simulation and development
Security Lab	Security auditing and forensics tools
Surfer	Web development tools

You can install a lab on your existing system:

sudo dnf groupinstall "Fedora Astronomy"

This doesn't change your desktop — it just installs a set of applications relevant to that field.
How the Desktop Relates to the Underlying System

This is the crucial concept: the desktop environment is just software. It runs as user-space processes on top of the Fedora operating system. It doesn't compile kernels, manage packages, configure networking, or handle security at the system level. Those are handled by:
Concern	Handled By	Desktop's Role
Hardware drivers	Kernel + firmware	Desktop uses them; doesn't manage them
Package management	DNF5 + RPM + Flatpak	GNOME Software is a GUI front-end
Networking	NetworkManager + firewalld	GNOME Settings is a GUI front-end
Security	SELinux + firewalld + kernel hardening	Desktop enforces user-level access
Filesystem	Kernel (Btrfs/ext4) + fdisk/LUKS	GNOME Files is a browser, not a manager
User management	shadow utils (useradd, passwd, etc.)	GNOME Settings is a GUI front-end
Process scheduling	Kernel (CFS/EEVDF)	Desktop is a process itself
Audio	PipeWire + WirePlumber	GNOME Settings configures volume

The desktop is a consumer of system services, not a provider. When you click "Connect" in GNOME Settings for Wi-Fi, GNOME sends a D-Bus message to NetworkManager (a system daemon), which configures the wireless card through the kernel's network subsystem. GNOME just drew the button you clicked.

This separation is why you can replace the desktop without touching the system. Install KDE Plasma alongside GNOME, log out, select KDE from the login screen, and log back in. Same kernel, same packages, same files, same network — completely different desktop.
Display Managers: The Gateway

Before your desktop loads, you interact with a display manager — the login screen. Its job is to authenticate you and then start your chosen desktop session.
GDM (GNOME Display Manager)

Fedora Workstation uses GDM — the GNOME Display Manager. It's what you see when you boot up and type your password.

┌─────────────────────────────────────────────┐
│                                             │
│                                             │
│              John Smith                     │
│           [ • • • • • ]                    │
│           Enter Password                    │
│                                             │
│  🔔  🔋  🔇                          ⚙     │
└─────────────────────────────────────────────┘

GDM does more than just password entry:
Function	How
User authentication	Validates username + password against PAM
Session selection	Gear icon → choose GNOME, GNOME Classic, or other installed sessions
Accessibility	On-screen keyboard, screen reader (Orca), high contrast
Power controls	Shut down, restart, suspend from login screen
Auto-login	Configurable to bypass password (not recommended)
Session Selection

If you have multiple desktop environments installed, GDM lets you choose which one to start. Click the gear icon (⚙) near the sign-in button:

┌─────────────────────────────┐
│  GNOME                      │  ← Default: full GNOME 50 desktop
│  GNOME Classic              │  ← Traditional two-panel layout
│  GNOME on Wayland           │  ← Explicit Wayland (same as GNOME in F44)
│  Plasma (if installed)      │  ← KDE Plasma session
│  XFCE Session (if installed)│  ← XFCE session
└─────────────────────────────┘

    💡 Only One Desktop at a Time

    You can install multiple desktop environments on the same system — GNOME and KDE Plasma, for example. But you can only run one at a time. You choose which one when you log in. Some applications may appear in both environments (a GTK app installed via DNF shows up in both GNOME and KDE menus).

Why Fedora Uses GNOME

You might wonder: why does Fedora Workstation default to GNOME, specifically? There are several reasons:
1. Technical Alignment

Fedora and GNOME are both closely associated with Red Hat. Many Fedora contributors are also GNOME contributors. The technologies are designed with each other in mind: GNOME's Wayland migration was tested on Fedora first, Mutter's latest features land in Fedora before other distros, and GNOME's release schedule aligns with Fedora's.
2. Wayland Leadership

Fedora was the first major distribution to default to Wayland (F25, 2016). GNOME's Mutter was the first production-ready Wayland compositor. The two projects pushed each other forward. This tight coupling means the Wayland experience on Fedora + GNOME is generally the most polished available anywhere.
3. Developer Experience

Fedora Workstation targets developers and workstation users. GNOME's keyboard-centric workflow, clean interface, and integration with developer tools (Terminal, Builder, Ptyxis, Toolbox) make it well-suited for this audience.
4. GNOME Classic Alternative

Fedora ships GNOME Classic as an alternative session — a GNOME configuration that provides a traditional two-panel layout (top bar + bottom taskbar) reminiscent of GNOME 2. This is available from the GDM session selector. It's GNOME under the hood, just configured differently.
5. Accessibility

GNOME has the most comprehensive accessibility suite of any Linux desktop — Orca screen reader, magnifier, on-screen keyboard, high-contrast themes, sticky keys, slow keys. Fedora's commitment to inclusivity aligns with GNOME's accessibility investment.

    ⚠️ Not a Value Judgment

    Fedora's choice of GNOME as the default doesn't mean GNOME is objectively "better" than KDE Plasma, XFCE, or others. It's a decision based on technical alignment, historical relationship, and target audience. Each desktop has strengths:

        GNOME — polish, accessibility, Wayland leadership, focus-oriented design
        KDE Plasma — customization, Windows familiarity, rich feature set
        XFCE — lightweight, stable, traditional
        Cinnamon — Windows-like, comfortable for migrants

    Fedora provides spins for all of these. The "best" desktop is the one that fits your workflow.

Desktop Environments vs. Operating Systems

One more conceptual point that trips up newcomers: Fedora is not GNOME, and GNOME is not Fedora.

    Fedora is the operating system: kernel, packages, repositories, installer, security model, release cycle.
    GNOME is the desktop: shell, windows, panels, applications, settings GUI.

GNOME runs on many distributions (Fedora, Debian, Arch, openSUSE, Ubuntu — though Ubuntu modifies it). Fedora can run many desktops (GNOME, KDE, XFCE, Cinnamon, etc.). The two are independent projects with overlapping communities.

When a GNOME update introduces a new feature, that's GNOME — not Fedora. When Fedora updates the kernel or adjusts SELinux policy, that's Fedora — not GNOME. They're layered, and understanding which layer owns which feature helps you know where to report bugs, where to find documentation, and what to expect in updates.
Practical Implications
Situation	Who Owns It
"The Activities Overview changed"	GNOME (upstream)
"DNF got faster"	Fedora (packaging) / DNF5 (upstream)
"My Wi-Fi won't connect"	Could be: kernel driver (upstream), NetworkManager (upstream), firewalld (upstream), or GNOME Settings (GNOME)
"SELinux blocked something"	Fedora (policy) / SELinux (upstream)
"The app grid looks different"	GNOME (upstream)
"A new kernel caused a regression"	Kernel (upstream) / Fedora (packaging)
"Flatpak isn't working"	Flatpak (upstream) / Fedora (packaging)

This distinction becomes important in Chapter 30 (Troubleshooting) and Chapter 31 (Getting Help) — knowing which project to report a bug to saves time.
What's Next

You now understand the layered architecture of a Linux desktop: applications, desktop environment, window manager, display server protocol, toolkit, drivers, kernel. You know what GNOME is, what alternatives exist, and why Fedora ships GNOME by default. You understand Wayland, toolkits (GTK vs Qt), display managers, and the relationship between desktop environments and the underlying operating system.

In Chapter 9, we'll take a detailed tour of the GNOME 50 desktop — every panel, every menu, every gesture, every keyboard shortcut. You'll learn your way around the desktop you'll be using every day.
Try It Yourself

    Identify your stack. Open a terminal and run these commands to see the layers on your system:

    # Kernel version
    uname -r

    # Display server
    echo $XDG_SESSION_TYPE

    # Desktop environment
    echo $XDG_CURRENT_DESKTOP

    # GNOME Shell version
    gnome-shell --version

    # Mutter version
    mutter --version

    Check for XWayland clients. Run xlsclients in a terminal. These are applications still using the X11 compatibility layer. You'll likely see fewer as time goes on — most apps are Wayland-native now.

    Explore the session selector. Log out of GNOME (top-right menu → Power → Log Out). At the GDM login screen, click your username, then look for the gear icon before entering your password. See what sessions are available. (If you only see "GNOME" and "GNOME Classic," that's expected — you haven't installed other desktops.)

    Look at your processes. Open System Monitor (Super → "System Monitor") and sort by name. Find gnome-shell, mutter (often part of gnome-shell), gdm, pipewire, NetworkManager. These are the daemons and services running your desktop. Each is a separate process — proof that the desktop is modular.

    Visit Fedora Spins. Open Firefox and browse to https://fedoraproject.org/spins. Look at the screenshots of KDE Plasma, XFCE, Cinnamon, and others. Notice how different they look — these are completely different desktop environments running on the same Fedora foundation you're using.

    Install a second desktop (optional, advanced). If you're curious about alternatives, try installing XFCE (lightweight and low-risk):

    sudo dnf groupinstall "XFCE Desktop"

    Log out, select "XFCE Session" from the GDM gear menu, and log in. Explore the different layout, file manager, and settings. When you're done, log out and select GNOME again. Your files, settings, and applications are all still there — only the desktop changed.

    ✅ Chapter 8 Complete You understand the Linux desktop stack (kernel → drivers → display server → toolkit → window manager → desktop environment → applications), the difference between a desktop environment and a window manager, the X11-to-Wayland transition and why Fedora is Wayland-only, toolkits (GTK vs Qt) and why cross-toolkit theming is imperfect, the major desktop environments available for Fedora (GNOME, KDE Plasma, XFCE, Cinnamon, MATE, LXQt, Budgie), Fedora's atomic variants (Silverblue, Kinoite), Fedora Labs, how the desktop relates to the underlying system (it's a consumer of system services, not a provider), display managers and session selection, why Fedora defaults to GNOME, and the important conceptual distinction between "Fedora" the OS and "GNOME" the desktop.
