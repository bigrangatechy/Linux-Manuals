# Chapter 1: What Is Fedora? What Is Linux?

## Before We Begin

If you've never used Linux before, you're in the right place. This manual assumes zero prior knowledge. You don't need to be a programmer, a system administrator, or a "computer person." You just need curiosity and a willingness to learn.

If you're coming from Windows or macOS, many concepts will feel familiar — you still have a desktop, a file manager, a web browser, and settings panels. But underneath, Linux works differently, and understanding those differences is what turns confusion into confidence.

This chapter lays the foundation. We'll cover what an operating system is, what Linux is, where Fedora fits in, and why people choose it. By the end, you'll understand the landscape well enough to follow every subsequent chapter.

---

## What Is an Operating System?

An operating system is the software that sits between you and your hardware. It's the layer that makes a computer usable.

Think of a computer as a restaurant:

- **The hardware** (processor, memory, disk, screen) is the kitchen — the physical equipment that does the actual work.
- **The operating system** is the restaurant manager — it decides which orders go to the kitchen, how ingredients are allocated, who gets served first, and how everything stays organized.
- **Your applications** (browser, email, games) are the menu items — the things you actually want.
- **You** are the customer — you place orders and receive results.

Without an operating system, a computer is just warm silicon. The OS manages memory, draws pixels on your screen, reads and writes files to your disk, handles your keyboard and mouse, connects to networks, and keeps one program from interfering with another. It's the invisible scaffolding that makes everything else possible.

Windows is an operating system. macOS is an operating system. Android and iOS are operating systems. And Linux is an operating system — or more precisely, Linux is the core component of many operating systems.

---

## What Is Linux?

### The Kernel

Strictly speaking, **Linux is a kernel** — the single piece of software at the heart of the system. The kernel is the bridge between hardware and software. It manages memory, handles process scheduling, controls device access, manages filesystems, and provides networking. Everything you do flows through the kernel.

The Linux kernel was created by **Linus Torvalds** in 1991 while he was a university student in Finland. He announced it on a mailing list with what has become one of the most famous messages in computing history:

> "I'm doing a (free) operating system (just a hobby, won't be big and professional like gnu) for 386(486) AT clones."

It was, by his own admission, a hobby project. Today, the Linux kernel runs most of the internet, every Android phone, all of the top 500 supercomputers, SpaceX rockets, Tesla cars, and millions of desktops around the world.

### The Operating System Around the Kernel

But a kernel alone isn't usable. You can't sit down at a kernel and check your email. To build a complete operating system, you need:

- A **shell** (the text interface where you type commands)
- **Core utilities** (programs like `ls`, `cp`, `cat`, `grep` — the basic tools for interacting with the system)
- A **package manager** (software for installing, updating, and removing other software)
- A **display server** (the layer that puts graphics on your screen)
- A **desktop environment** (the graphical interface — windows, panels, menus, icons)
- **Applications** (a web browser, file manager, text editor, and so on)

The kernel is the engine. But you also need a steering wheel, seats, a dashboard, and wheels to have a car.

### Distributions: Complete Linux Operating Systems

Because the kernel alone isn't enough, organizations and communities package the kernel together with everything else you need to form a **complete, usable operating system**. These complete packages are called **distributions** (often shortened to **distros**).

A distribution bundles:

| Component | Purpose |
|---|---|
| Linux kernel | The core — hardware management |
| GNU core utilities | Basic commands (`ls`, `cp`, `rm`, `cat`, etc.) |
| Init system | The first process that starts everything else (Fedora uses systemd) |
| Package manager | Installs and manages software (Fedora uses DNF) |
| Display server | Graphics plumbing (Fedora 44 uses Wayland) |
| Desktop environment | The graphical interface (Fedora Workstation uses GNOME) |
| Default applications | Browser, file manager, text editor, etc. |
| Configuration tools | Settings panels, installers, update managers |

Different distributions make different choices about each of these components. Ubuntu chooses APT and Snap. Fedora chooses DNF and Flatpak. Arch chooses pacman. Debian chooses APT. They all run the Linux kernel, but the surrounding software differs — sometimes slightly, sometimes dramatically.

> 💡 **Analogy: Cars and Platforms**
>
> Think of distributions like car models from different manufacturers. They all have engines, transmissions, and wheels (the Linux kernel, GNU tools, X/Wayland). But a Toyota, a Ford, and a BMW feel different because of the body design, the interior materials, the suspension tuning, and the features included. The engineering underneath is similar; the execution varies.
>
> Ubuntu is like a Toyota Camry — dependable, popular, widely supported, and easy to own. Fedora is more like a BMW — engineered with precision, fast to adopt new technology, and beloved by enthusiasts who want the cutting edge.

---

## Where Does Fedora Come From?

### A Short History

Fedora's story begins with **Red Hat**, one of the most important companies in Linux history.

In 1994, Marc Ewing released **Red Hat Linux** — one of the earliest commercial Linux distributions. In 1995, Bob Young acquired Ewing's business, forming **Red Hat Software**. Throughout the late 1990s and early 2000s, Red Hat Linux was a single product that served both home users and enterprises.

By 2003, Red Hat faced a fork in the road. Enterprise customers needed rock-solid stability and long-term support contracts. Home users and developers wanted fresh software and rapid updates. These two audiences pulled in opposite directions, and Red Hat couldn't serve both with one product.

So Red Hat split:

- **Red Hat Enterprise Linux (RHEL)** — the commercial, enterprise-focused product with long support cycles and paid subscriptions.
- **Fedora** — the community-driven, fast-moving distribution that would serve as the upstream testing ground for new technologies.

Fedora was born on November 6, 2003, with **Fedora Core 1**. The "Core" part of the name referred to the fact that Red Hat provided the core packages, while the community provided the rest. In 2007, starting with Fedora 7, the "Core" designation was dropped — it became simply **Fedora** — reflecting the full merger of Red Hat's and the community's package sets.

### The Red Hat Relationship

Fedora is developed by the community-supported **Fedora Project** and sponsored by **Red Hat** (now part of IBM). This relationship is crucial to understanding what Fedora is and isn't:

- **Fedora is not RHEL.** RHEL is Red Hat's commercial product. Fedora is free, community-driven, and independent.
- **Fedora is upstream of RHEL.** Technologies mature in Fedora, prove themselves, and then flow into CentOS Stream, which feeds into RHEL. Fedora is where new ideas land first.
- **Red Hat employees are major contributors.** Many Fedora developers are Red Hat employees, but the project is open to anyone. Community members and volunteers contribute code, artwork, documentation, QA, and governance.
- **Fedora's independence is real.** Red Hat sponsors Fedora but doesn't dictate its direction. The Fedora Council (an elected community body) governs the project. Red Hat doesn't control what goes in — proposals are debated openly, voted on, and implemented by the community.

> 💡 **The Pipeline Metaphor**
>
> Think of Fedora as a river source. New technologies enter the river in Fedora. They flow downstream to **CentOS Stream** (Red Hat's continuously updated development branch), then arrive in **RHEL** (the commercial product), and eventually reach the countless enterprise systems worldwide that depend on RHEL for stability.
>
> Using Fedora means standing at the source — you see the future of enterprise Linux first.

### The Four Foundations

The Fedora Project is guided by four foundational principles:

1. **Freedom** — Fedora is committed to free and open source software. The project prioritizes software freedom above convenience. This is why Fedora doesn't ship proprietary codecs, drivers, or applications by default — not because they're hard to include, but because they're not free. (Chapter 11 shows you how to add them if you need them.)

2. **Friends** — Fedora is a community. Developers, designers, testers, translators, writers, and users all contribute. The project values collaboration over heroism.

3. **Features** — Fedora aims to showcase the latest that open source has to offer. New technologies land in Fedora quickly — Wayland, systemd, PipeWire, DNF5, and GNOME 50 all appeared in Fedora before most other distributions adopted them.

4. **First** — Fedora strives to be first. First to adopt new upstream releases. First to deprecate old technologies. First to try new approaches. This "features and first" philosophy means Fedora is exciting and current, but it also means things change — which is why this book teaches you adaptability, not just memorization.

These four foundations are why Fedora feels different from Ubuntu. Ubuntu optimizes for ease of use and broad appeal. Fedora optimizes for innovation and software freedom. Both are excellent — they just have different priorities.

---

## Why People Choose Fedora

Different Linux distributions appeal to different people. Here's why people pick Fedora:

### Cutting-Edge Software, Without the Chaos

Fedora walks a middle ground between conservative distributions like Ubuntu LTS (which holds back updates for years) and rolling-release distributions like Arch (which updates constantly, sometimes breaking things). Fedora releases a new version every six months, each containing the latest stable versions of major software — but tested and integrated before release.

You get GNOME 50, the latest kernel, current compilers, and modern toolkits — without rolling the dice every time you update.

### Pure, Unmodified GNOME

If you want the GNOME desktop experience as its creators intended, Fedora Workstation is the gold standard. Ubuntu ships a customized version of GNOME with its own theme (Yaru), a dock pinned to the left, and various modifications. Fedora ships GNOME essentially as GNOME's developers designed it — vanilla, upstream, untouched.

If you've used GNOME on Ubuntu and want to see what GNOME "really" looks like, Fedora is the answer.

### Innovation Leadership

Fedora was:

- **First** to ship Wayland by default (Fedora 25, November 2016 — years before Ubuntu)
- **First** to ship PipeWire as the default audio server (Fedora 34, April 2021)
- **First** to complete the DNF5 migration across the entire system (Fedora 43, October 2025)
- **First** to integrate Whisper AI speech-to-text system-wide (Fedora 44, April 2026)
- **Early** adopter of systemd, Btrfs-by-default, ZRAM swap, and many other technologies that later spread to other distributions

Using Fedora means you're often using the technologies that everyone else will be using in two to three years.

### Developer-Friendly Out of the Box

Fedora ships with developer tools that many other distributions make you install manually:

- **Toolbox** — Built-in containerized development environments. Spin up a throwaway Fedora environment in seconds to test code without touching your host system.
- **DNF5** — The fastest RPM package manager available, with parallel downloads and instant dependency resolution.
- **Modern toolchain** — Current GCC, Python, Rust, Go, Node.js, and more, updated each release cycle.
- **Flatpak integration** — Flathub is a first-class citizen for GUI applications, sandboxed and secure.
- **Nix** — Available in Fedora 44's official repositories, for declarative, reproducible package management.

### Strong Security Defaults

Fedora takes security seriously — sometimes more seriously than convenience:

- **SELinux** is enabled by default in "targeted" mode (a more granular security framework than the AppArmor used by Ubuntu)
- **firewalld** is active by default (Fedora's firewall management tool)
- **Full disk encryption** is offered during installation
- **No network services** are exposed by default — the system is locked down until you explicitly open ports
- **Compile-time hardening** — software is compiled with security features like stack protector and position-independent executables

### The Open Source Purity

Fedora's commitment to free software means the default installation contains no proprietary code — no binary blobs, no closed-source drivers, no patent-encumbered codecs. Some users find this refreshing. Others find it inconvenient (you can't play MP4 videos out of the box without adding RPM Fusion — covered in Chapter 11). Either way, it's a deliberate philosophical choice, not an oversight.

---

## Fedora Editions and Spins

Fedora isn't one product — it's a family. Each member shares the same base system but targets a different use case:

### Editions (Officially Supported)

| Edition | What It Is |
|---|---|
| **Workstation** | The flagship desktop edition. Uses GNOME 50. This is what this manual covers. |
| **Server** | Headless server OS with web-based cockpit management interface. |
| **IoT** | For embedded devices and Internet of Things applications. |
| **Cloud** | For cloud infrastructure (AWS, Azure, GCP images). |
| **CoreOS** | Minimal, container-focused OS for running containerized workloads. |

### Spins (Community-Maintained Alternative Desktops)

If you want Fedora with a different desktop environment, you download a **Spin**:

| Spin | Desktop |
|---|---|
| **Fedora KDE Plasma Desktop** | KDE Plasma 6.6 (covered in the companion manual) |
| **Fedora Xfce Desktop** | Xfce — lightweight, traditional |
| **Fedora Cinnamon Desktop** | Cinnamon — Windows-like layout |
| **Fedora Budgie Desktop** | Budgie — now fully Wayland-native in Fedora 44 |
| **Fedora LXQt Desktop** | LXQt — ultra-lightweight |
| **Fedora i3 Spin** | i3 tiling window manager — for keyboard power users |
| **Fedora Sway Spin** | Sway — Wayland-native tiling window manager |

### Labs (Specialized Software Collections)

Labs are curated collections of software for specific domains:

| Lab | Includes |
|---|---|
| **Astronomy** | Stellarium, Celestia, astrophotography tools |
| **Design Suite** | GIMP, Inkscape, Scribus, Blender |
| **Games** | A collection of open-source games (now on KDE Plasma in F44) |
| **Jam** | Audio production tools — Ardour, LMMS, Hydrogen |
| **Python Classroom** | Teaching environment with Python tools |
| **Scientific** | Computational science tools |
| **Security Lab** | Penetration testing and forensics tools |

### Atomic Desktops (Immutable Systems)

Atomic Desktops are a newer category — immutable, image-based systems where the root filesystem is read-only and updates are applied atomically (all or nothing, never half-applied):

| Atomic Desktop | Desktop | Technology |
|---|---|---|
| **Fedora Silverblue** | GNOME 50 | rpm-ostree |
| **Fedora Kinoite** | KDE Plasma 6.6 | rpm-ostree |
| **Fedora Sway Atomic** | Sway | rpm-ostree |
| **Fedora Budgie Atomic** | Budgie | rpm-ostree |
| **Fedora COSMIC Atomic** | COSMIC | rpm-ostree / bootc |

> ⚠️ **This Manual Covers Traditional Fedora Workstation**
>
> This manual covers the standard **Fedora 44 Workstation** edition — a traditional, mutable system where you can install, modify, and remove packages freely.
>
> Fedora Silverblue (and other Atomic Desktops) work differently — the base system is read-only, and you install applications via Flatpak or layer packages onto the image with `rpm-ostree`. The concepts in this manual still apply (the terminal, the filesystem, GNOME, networking), but the package management chapters would differ significantly. If you've chosen Silverblue, most of this manual still applies — just skip the DNF sections and consult Fedora's Silverblue documentation instead.

---

## How Fedora Differs from Ubuntu

Since this manual series began with Ubuntu, here's a quick comparison for readers who've used Ubuntu (or read the Ubuntu manual) and want to know what's different:

| Aspect | Ubuntu 26.04 LTS | Fedora 44 Workstation |
|---|---|---|
| **Parent organization** | Canonical (UK company) | Red Hat / IBM (US company) |
| **Release cycle** | 6 months + LTS every 2 years (5-year support) | 6 months, ~13 months support per release, no LTS |
| **Kernel** | 7.0 (custom) | 6.19 |
| **Default filesystem** | ext4 | Btrfs (with snapshots) |
| **Package manager** | APT (deb) + Snap | DNF5 (RPM) + Flatpak |
| **Snap support** | Yes (default for many apps) | No — Flatpak instead |
| **Flatpak support** | Available but not default | First-class, integrated |
| **Desktop** | GNOME 50 with Yaru theme, dock mods | Vanilla GNOME 50 (no customization) |
| **App store** | App Center (deb + Snap) | GNOME Software (RPM + Flatpak) |
| **Codec installation** | `ubuntu-restricted-extras` | RPM Fusion repositories |
| **NVIDIA drivers** | Included in repos | RPM Fusion (non-free repo) |
| **Firewall** | UFW (inactive by default) | firewalld (active by default) |
| **MAC security** | AppArmor | SELinux (targeted mode) |
| **Swap** | ZRAM | ZRAM |
| **Display server** | Wayland-only | Wayland-only (X11 removed in F44) |
| **Installer** | Flutter-based | Anaconda (web-based UI) |
| **Philosophy** | Ease of use, broad appeal | Innovation, software freedom, upstream-first |
| **sudo** | Written in Rust | Written in C (traditional sudo) |
| **Developer tools** | Manual installation | Toolbox built-in, Nix in repos |
| **Audio** | PipeWire | PipeWire (first to adopt it) |

Both are excellent distributions. The choice comes down to priorities: Ubuntu for LTS stability and ease; Fedora for freshness and innovation.

---

## Who Is Fedora For?

Fedora is a great choice for:

- **Developers** — current toolchains, Toolbox, Distrobox, and excellent language support
- **GNOME purists** — the cleanest vanilla GNOME experience available
- **Technology enthusiasts** — who want to see and use new features before anyone else
- **Privacy-conscious users** — no telemetry, no proprietary code by default, no tracking
- **Learners** — a friendly, well-documented distro with an active, welcoming community
- **Future RHEL users** — learning Fedora prepares you for the Red Hat ecosystem

Fedora might require more willingness to adapt than Ubuntu LTS — the 6-month release cycle means things change more often. But the Fedora community is excellent, the documentation is thorough, and the system rewards curiosity.

---

## What You'll Need

Before we continue, here's what you need to follow along with this manual:

### A Computer (or a Virtual Machine)

Fedora 44 Workstation requires:

- **Minimum:** 4 GB RAM, 20 GB disk space, 64-bit x86 processor (x86-64-v3 or newer)
- **Recommended:** 8 GB+ RAM, 60 GB+ disk space, dual-core processor or better
- **Architecture:** x86-64 (Intel/AMD) or aarch64 (ARM, e.g., Raspberry Pi 4/5, Snapdragon X Elite)

> 💡 **The 4 GB Minimum**
>
> Fedora technically runs on 4 GB of RAM, but the experience is tight. 8 GB is the practical minimum for comfortable daily use with GNOME 50. If you have less, consider a lighter spin like Xfce or LXQt instead.

### A USB Flash Drive

To install Fedora, you'll need a USB drive with at least 8 GB of capacity. Everything on it will be erased.

### An Open Mind

The biggest obstacle to learning Linux isn't the technology — it's the mental adjustment. Things will look different. Names will be unfamiliar. Some tasks that took one click on Windows will take three steps on Fedora. Some tasks that took ten clicks on Windows will take one command. It's a different way of thinking, not a harder way.

Be patient with yourself. Every Linux user went through the same learning curve. The confusion passes, and what remains is a system you truly understand and control.

---

## What's Next

In Chapter 2, we'll explore the philosophy behind Fedora's existence — free and open source software. You'll learn what "free" means in the software world, why Fedora refuses to ship certain software by default, and how the open source movement shaped the technology landscape you use every day.

---

## Try It Yourself

1. **Visit fedoraproject.org.** Take a look at the Fedora Project homepage. Browse the different editions (Workstation, Server, IoT) and at least two spins (KDE, Xfce). Notice how they all share the same base but target different use cases.

2. **Compare with a friend's computer.** If you know someone running Windows or macOS, compare what's on their desktop with what you'd expect from Fedora. What's the same? What's different? You might be surprised by how much overlaps.

3. **Read the Fedora Magazine.** Visit [fedoramagazine.org](https://fedoramagazine.org). Skim a few recent articles. Don't worry if you don't understand everything — you will soon. Just get a feel for what the Fedora community talks about and cares about.

4. **Join the conversation.** Browse [ask.fedoraproject.org](https://ask.fedoraproject.org) — Fedora's community Q&A site. Read a few questions and answers. You'll see that even experienced users ask "beginner" questions — there's no shame in learning.

> ✅ **Chapter 1 Complete**
> You understand what an operating system is, what the Linux kernel is, what a distribution is, where Fedora comes from (Red Hat, 2003), the four foundations (Freedom, Friends, Features, First), the different Fedora editions and spins, how Fedora compares to Ubuntu, and whether Fedora is right for you.
