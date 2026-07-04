# Chapter 3: Ubuntu Releases and Lifecycle

## Why This Chapter Matters

Imagine buying a new phone, only to discover months later that it no longer receives security updates. You'd feel frustrated — and rightfully so. Knowing how long your system will be supported is one of the most important things about choosing an operating system.

Ubuntu has a well-defined, predictable release cycle. Unlike some operating systems that keep you guessing, Ubuntu tells you exactly when new versions arrive, how long each version is supported, and when you need to upgrade. Understanding this before you install means you'll make smart choices from day one and avoid surprises later.

## How Ubuntu Versions Are Numbered

Ubuntu uses a simple, logical version numbering scheme:

**Year.Month**

- Ubuntu 24.04 was released in **April 2024**
- Ubuntu 24.10 was released in **October 2024**
- Ubuntu 25.04 was released in **April 2025**
- Ubuntu 25.10 was released in **October 2025**
- Ubuntu 26.04 was released in **April 2026**

The format is straightforward: the first two digits represent the year, and the second two represent the month. Once you know this pattern, you can tell at a glance when any Ubuntu version was released.

### Codenames: The Adjective Animal Tradition

Beyond the version number, every Ubuntu release also has a **codename** — and these follow a playful tradition. Each codename consists of an adjective and an animal, with the first letter of both matching, and advancing alphabetically with each release:

- **M**antic **M**inotaur (23.10)
- **N**oble **N**umbat (24.04 LTS)
- **Q**uesting **Q**uokka (25.10)
- **R**esolute **R**accoon (26.04 LTS)
- ...

You'll see these codenames used frequently in documentation, forum posts, and package repositories. When someone says "I'm on Resolute," they mean Ubuntu 26.04 LTS. When a tutorial references "Noble repositories," it's talking about 24.04's software sources. Knowing the codename for your version helps you find relevant solutions when troubleshooting.

> 💡 **Don't worry about memorizing these.**
>
> You'll naturally pick up your own version's codename over time. The alphabetical progression is handy to know because it helps you gauge roughly how old a reference is — if you see a tutorial mentioning "Bionic Beaver" (18.04), you know it's quite old at this point.

## The Two Types of Ubuntu Releases

Ubuntu releases come in two flavors, and the distinction is crucial for beginners:

### LTS: Long Term Support

**LTS releases** are the bedrock of the Ubuntu ecosystem. They are released **every two years**, in April of even-numbered years:

- 18.04 (April 2018)
- 20.04 (April 2020)
- 22.04 (April 2022)
- 24.04 (April 2024)
- **26.04 (April 2026) — current LTS**
- 28.04 (April 2028)
- ...

An LTS release receives:

- **5 years of standard support** — security updates, bug fixes, and hardware enablement for the core system
- **Up to 10+ years of extended support** — through Canonical's Extended Security Maintenance (ESM) program for certain packages (more on this below)
- **Point releases** — periodic refreshes (e.g., 26.04.1, 26.04.2) that bundle accumulated updates into a fresh ISO, so new installations don't require hundreds of post-install updates

LTS releases prioritize **stability**. The software versions included in an LTS release are frozen at release time — they don't get upgraded to newer major versions during the support period. Instead, they receive security patches and bug fixes backported to the version that shipped. This means no surprise breakage from a new version of your desktop environment or a rewritten application, but it also means you won't get brand-new features through the standard update channel.

### Interim (Non-LTS) Releases

Interim releases come out in the odd years and in October of every year — essentially, every release that *isn't* an LTS. They fill the gaps between LTS releases:

- 24.10 (October 2024)
- 25.04 (April 2025)
- 25.10 (October 2025)
- 26.10 (October 2026)
- ...

Interim releases receive:

- **9 months of support** — security updates and bug fixes
- Newer software versions — they include more recent versions of applications, desktop environments, and system tools

Interim releases are essentially stepping stones. They allow the Ubuntu team to ship new features, gather feedback, and stabilize changes before they land in the next LTS. They're useful for:

- Developers who want the latest tools
- Users who want new desktop features
- Testing what's coming before it reaches LTS
- Anyone who doesn't mind upgrading every 6–9 months

After 9 months, an interim release stops receiving updates. You must either upgrade to a newer release or switch to an LTS. If you stay on an unsupported release, your system will continue to run, but you won't receive security patches — which is a risk.

> ⚠️ **For Most Beginners: Choose LTS**
>
> If you're new to Ubuntu, **choose an LTS release.** Plain and simple. Here's why:
>
> - You get 5 years of support without needing to upgrade
> - The software is more tested and stable
> - Most tutorials, documentation, and community answers assume LTS
> - Hardware support is well-established by the time you install
> - You avoid the hassle of frequent major upgrades
>
> Interim releases have their place, but as a beginner, stability and longevity should be your priority. You can always switch to an interim release later if you want to explore newer features.

## The Current LTS: Ubuntu 26.04 Resolute Raccoon

Since this is the version you'll most likely be installing, let's take a look at what Ubuntu 26.04 LTS brings to the table. Released on **April 23, 2026**, Resolute Raccoon is a significant release that introduces several major advancements:

**Desktop and Interface**

- **GNOME 50** — the biggest GNOME milestone in years, featuring variable refresh rate (VRR) support, flawless fractional display scaling, and a fully Wayland-native experience. X11, the legacy display system, has been officially removed.
- **Refreshed default app stack** — new modern applications built on the libadwaita framework, including:
  - **Showtime** — a new video player
  - **Resources** — a new system monitor (replacing GNOME System Monitor)
  - **Loupe** — a new image viewer
  - **Papers** — a new document viewer
  - **Ptyxis** — a new container-aware terminal

**Security**

- **Rust-powered sudo and coreutils** — the classic C-based sudo and core utilities are replaced with memory-safe Rust implementations, improving security at the OS foundation
- **TPM-backed Full Disk Encryption** — hardware-level encryption protection using your computer's Trusted Platform Module
- **Snap permission prompting** — finer-grained control over what Snap applications can access
- **Post-quantum cryptography in OpenSSH** — forward-looking protection against future quantum computing threats
- **Security Center app** — a centralized application for managing your system's security posture

**Performance and Hardware**

- **Linux kernel 7.0** — with day-one support for Intel Nova Lake, AMD Zen 6, and Snapdragon X2 processors
- **x86-64-v3 optimized packages** — the entire package archive is now available in an optimized variant for modern CPUs, providing free performance improvements
- **NVIDIA Wayland improvements** — significant performance fixes for NVIDIA users, with up to 12% higher framerates reported on RTX 5090

**AI Integration**

- **Local AI with Inference Snaps** — run AI models like DeepSeek R1 and Gemma 3 directly on your machine, fully offline and private
- **NVIDIA CUDA and AMD ROCm** available directly in Ubuntu repositories — no need for manual driver setup or third-party downloads

> 💡 **What Does This Mean for You?**
>
> As a beginner, you don't need to understand all of these features yet. The key takeaway is that Ubuntu 26.04 LTS is a substantial, modern release that will serve you well for the next 5 years. Everything in this manual will reference 26.04 LTS, so you'll be working with the current standard.

## Support Timelines in Detail

Let's visualize how support works for your LTS release:

Release: Ubuntu 26.04 LTS (Resolute Raccoon) Released: April 23, 2026

    Standard Support (5 years)
    ┌──────────────────────────────────────────────┐
    │  April 2026                             April 2031  │
    │  ████████████████████████████████████████████ │
    │  Full security updates + bug fixes              │
    └──────────────────────────────────────────────┘
                                │
                                ▼
    Extended Security Maintenance (ESM)
    ┌──────────────────────────────────────────────┐
    │  April 2031                         ~April 2036  │
    │  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
    │  Security-only updates for core packages    │
    └──────────────────────────────────────────────┘

Total potential lifespan: ~10 years

During the standard support period, you receive:

    Security updates for the core system and thousands of packages
    Bug fixes for critical issues
    Hardware enablement stacks (HWE) — updated kernels and graphics drivers that let LTS releases support newer hardware released after the LTS launch. These are delivered via point releases and optional HWE kernel channels

During the ESM period:

    Only security updates for a subset of critical packages (like the kernel, base libraries, and common services) are provided
    ESM is available through Ubuntu Pro (free for personal use on up to 5 machines, more on this below)
    The system remains usable but gradually becomes outdated

Point Releases

Point releases (like 26.04.1, 26.04.2, etc.) are refreshed installer images that include all accumulated updates up to that point. For the current LTS:

    26.04.1 LTS is scheduled for August 4, 2026 — approximately three months after initial release
    Subsequent point releases follow roughly every few months

Point releases matter because:

    New installations start with all updates already applied — no hour-long post-install update session
    LTS-to-LTS upgrades are only offered after the first point release ships, ensuring the new LTS has had time to stabilize
    Hardware enablement — newer kernels and graphics drivers for supporting hardware released after 26.04 launch are bundled in

Ubuntu Pro

Ubuntu Pro is Canonical's extended support and management offering. For personal, non-commercial use on up to 5 machines, it's available for free. Ubuntu Pro extends:

    Extended Security Maintenance (ESM) — security patches beyond the standard 5-year window
    Livepatch — apply kernel security updates without rebooting
    CIS and DISA STIG hardening profiles — security compliance configurations (primarily relevant for enterprise/server environments)
    Enhanced package coverage — security updates for a broader range of packages beyond the base repository

For a beginner on a personal machine, Ubuntu Pro is worth enabling — it adds another layer of long-term security coverage at no cost. We'll walk through enabling it in the Post-Install Checklist chapter.
Upgrades: Moving Between Versions

Eventually, you'll want or need to move to a newer Ubuntu version. There are two kinds of upgrades:
Point Release Upgrades (Minor)

These happen automatically through normal system updates. When Ubuntu 26.04.2 is released, you don't need to do anything special — your 26.04 system simply receives the accumulated updates over time. A point release is just a refreshed installer image for new installations; existing users get the same content through the update process.
Major Version Upgrades

This is when you move from one release to the next — for example, from 26.04 LTS to 28.04 LTS, or from 26.10 to 27.04. The upgrade process replaces large portions of the system with newer versions.

Key behaviors:

    LTS to LTS upgrades are offered automatically after the first point release of the new LTS ships. So 26.04 → 28.04 wouldn't be offered until around July 2028 (three months after 28.04's April 2028 release). This delay ensures the new LTS has had time to stabilize.
    Interim to interim upgrades are offered promptly after the next release ships.
    Interim to LTS upgrades are offered when the LTS releases.
    LTS to interim upgrades are not offered by default — you'd have to opt in manually. This prevents accidental moves from a stable LTS to a short-lived interim release.

The upgrade process itself can be done:

    Graphically: A notification appears prompting you to upgrade. Follow the on-screen wizard.
    From the terminal: Using the do-release-upgrade command (we'll cover this in detail in a later chapter, but for now just know it exists).

    💡 Best Practice for Upgrades

        Always back up your important data before a major upgrade. We'll cover backups in depth in Chapter 27.
        Read the release notes before upgrading — they list known issues and major changes. Find them at discourse.ubuntu.com.
        Upgrade when you have time to troubleshoot if something goes wrong — not right before a deadline or trip.
        For LTS users: wait for the 28.04.1 point release before upgrading — stability is worth the few extra months.

End of Life: What Happens When Support Ends?

When a release reaches End of Life (EOL), several things happen:

    No more security updates — your system becomes increasingly vulnerable over time
    Repositories are archived — the software sources you've been downloading updates from are moved to an old-releases server. Without updating your configuration, apt update (we'll cover this command later) will fail
    The system keeps running — it won't stop working, but it becomes a security liability
    Community support drops off — fewer people will be able to help with issues on an EOL release

If you find yourself on an EOL release, the correct action is to upgrade to a supported release. In most cases, this can be done in place (preserving your files and settings). For very old releases, a clean reinstall may be simpler.
Choosing Your Release: A Quick Decision Guide

Still unsure which release to pick? Here's a simple guide:
Your Situation	Recommendation
New to Linux, want stability	Latest LTS (currently 26.04)
New to Linux, want newest features	Latest LTS (stick with stable while learning)
Developer who needs latest tools	Latest interim or latest LTS with backports
Setting up a server	Latest LTS
Older hardware that works well now	Current LTS (avoid unnecessary upgrades)
Want the safest, longest-lasting option	Latest LTS with Ubuntu Pro enabled

As a beginner, the answer is almost always: Latest LTS, with Ubuntu Pro enabled. This gives you the longest support window, the most stable experience, and the best alignment with community resources.
What's Next

You now understand Ubuntu's versioning scheme, the difference between LTS and interim releases, how long each is supported, and how upgrades work. You also have a snapshot of what 26.04 LTS brings to the table. In Chapter 4, we'll help you bridge the mental gap between your current operating system and Ubuntu — mapping familiar Windows and macOS concepts to their Ubuntu equivalents so you feel oriented from the start.
Try It Yourself

    Visit ubuntu.com/desktop and confirm that Ubuntu 26.04 LTS (Resolute Raccoon) is the current download. Note the version number and codename.
    Calculate your support timeline. Based on 26.04 LTS, work out when standard support ends (April 2031) and when ESM would extend to (~April 2036).
    Think about your own needs. Will you be using Ubuntu on a daily driver laptop, a server, a media center, or something else? Does LTS fit your use case? Write down your reasoning — it'll help when we get to installation.

