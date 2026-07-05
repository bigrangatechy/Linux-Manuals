# Chapter 3: Fedora Releases and Lifecycle

## The Six-Month Heartbeat

Fedora releases a new version every six months. Like clockwork — usually. This rhythm is the single most important thing to understand about how Fedora works, and it shapes your experience as a user more than any other factor.

Where Ubuntu offers LTS releases supported for five years (or more with ESM), Fedora takes a different approach: frequent releases, short lifecycles, constant forward motion. There's no LTS safety net. There's no "stick with this version for half a decade" option. Fedora moves, and you move with it.

This sounds intimidating to newcomers — and it's worth addressing that fear honestly. But in practice, the Fedora upgrade process is smooth, well-tested, and usually takes less time than a major Windows update. This chapter explains everything you need to know.

---

## Version Numbering: Simple and Sequential

Fedora uses straightforward sequential numbering. No codenames in the Ubuntu sense (where "Resolute Raccoon" or "Bionic Beaver" are part of the branding). Instead, it's just:

- Fedora 1 (November 2003)
- Fedora 2 (May 2004)
- Fedora 3 (November 2004)
- ...
- Fedora 41 (October 2024)
- Fedora 42 (April 2025)
- Fedora 43 (October 2025)
- Fedora 44 (April 2026) ← **current**

Simple. The number goes up by one each release. There's no year-based scheme (like Ubuntu's 26.04), no marketing names, no animal alliteration.

### A Brief Note on Codenames

Historically, Fedora releases did receive community-chosen codenames (like "Beefy Miracle" for Fedora 16, "Schrödinger's Cat" for Fedora 19, or "Godde" for Fedora 20). These were never used in the way Ubuntu uses codenames — they weren't part of the directory names, repository identifiers, or official branding. They were more like internal nicknames or community in-jokes.

In recent years, the codename tradition has largely faded. Modern Fedora releases are referred to simply by their number: Fedora 42, Fedora 43, Fedora 44. You'll occasionally see a codename referenced in old documentation or community nostalgia, but for practical purposes, the number is the name.

### The "Core" Era

If you read older documentation, you may see references to "Fedora Core 1" through "Fedora Core 6." The "Core" designation indicated that Red Hat provided the core package set, while the community provided the "Extras" repository. Starting with Fedora 7 (May 2007), the two repositories were merged, and the "Core" label was dropped. Since then, it's just "Fedora N."

---

## The Release Calendar

Fedora targets two releases per year:

| Approximate Window | Example Releases |
|---|---|
| **Spring** (April ± 1 month) | Fedora 42 (Apr 2025), Fedora 44 (Apr 2026) |
| **Fall** (October/November ± 1 month) | Fedora 43 (Oct 2025), Fedora 45 (Oct 2026, planned) |

The target dates are set well in advance, but Fedora has a famous (or infamous) tradition: **"It's ready when it's ready."** The project uses a rigorous release process with Go/No-Go meetings. If a release candidate has unresolved blocking bugs, the release slips — typically by one or two weeks.

This means Fedora release dates are approximate, not guaranteed. The Fedora community treats this as a feature, not a bug: a delayed release is better than a broken one. Compare this to some commercial operating systems that ship on a fixed date regardless of quality.

### Recent and Upcoming Releases

| Version | Release Date | End of Life | Kernel |
|---|---|---|---|
| Fedora 41 | October 29, 2024 | December 15, 2025 | 6.11 |
| Fedora 42 ("Adams") | April 15, 2025 | May 27, 2026 | 6.14 |
| Fedora 43 | October 28, 2025 | December 9, 2026 | 6.17 |
| **Fedora 44** | **April 28, 2026** | **~June 2, 2027** | **6.19** |
| Fedora 45 | ~October 2026 (planned) | ~December 2027 | TBD |
| Fedora 46 | ~April 2027 (planned) | ~June 2028 | TBD |

> 💡 **Why the Tildes (~)?**
>
> Fedora's end-of-life date depends on the release date of the version *two releases later*. Fedora 44 loses support approximately four weeks after Fedora 46 is released (roughly June 2027). Since future release dates aren't set in stone, the EOL date shifts accordingly.

---

## Support Lifecycle: ~13 Months

Each Fedora release is maintained for approximately **13 months** from its general availability (GA) date. Here's what that means in practice:

### Phase 1: Active Support (Release → ~13 months)

During active support, a Fedora release receives:

- **Security updates** — patches for vulnerabilities as they're discovered
- **Bug fixes** — fixes for significant bugs reported by users
- **Selected enhancement updates** — minor improvements that don't break compatibility

Updates are delivered through the `updates` repository. Your system checks for and installs them automatically (or notifies you, depending on your settings).

### Phase 2: End of Life (EOL)

When a release reaches EOL (approximately 13 months after release):

- The repositories stop receiving updates
- No more security patches
- No more bug fixes
- The release is archived

At this point, you **must** upgrade to a supported release. Running an EOL Fedora release is a security risk — known vulnerabilities will never be patched.

### The Overlap

Because releases come every six months and each is supported for thirteen, there's always overlap:

Fedora 43 ████████████████████░░░░░░ (Oct 2025 → Dec 2026) Fedora 44 ████████████████████░░░░░░ (Apr 2026 → Jun 2027) Fedora 45 ████████████████████░░ (Oct 2026 → Dec 2027)

At any given time, at least two (usually three) Fedora releases are supported simultaneously. This gives you a comfortable window to upgrade — you're never racing against a deadline the moment a new version drops.
No LTS: What This Means for You

This is the biggest difference from Ubuntu, and it's worth examining thoroughly.
Why Fedora Doesn't Do LTS

Ubuntu's LTS model exists because Canonical (Ubuntu's parent company) serves both desktop users and enterprise/long-term deployments. An LTS release lets Ubuntu satisfy servers, workstations in corporate environments, and cautious users who want stability above all.

Fedora doesn't do LTS because its mission is different. Fedora exists to push the bleeding edge — to be the proving ground for new technologies. An LTS Fedora would contradict its "Features" and "First" foundations. Red Hat has RHEL for long-term stability; Fedora's job is innovation.

Additionally, the 13-month lifecycle is long enough to be practical but short enough to prevent stagnation. Fedora's maintainers don't have to maintain ancient software versions for five years — they get to focus on the current and previous release.
Practical Implications
Concern	Answer
"Do I have to upgrade every six months?"	No. You can upgrade once a year (skipping one release). Since each release is supported for ~13 months, you always have a supported upgrade path.
"Is upgrading scary?"	Not usually. The dnf system-upgrade process is well-tested. It downloads packages, reboots into a special upgrade environment, applies everything, and reboots again. Typically 20–40 minutes.
"What if an upgrade breaks something?"	Fedora maintains the previous release for months after a new one ships. You can stay on the old version while others shake out the bugs, then upgrade when you're ready.
"Do I lose my files when upgrading?"	No. An in-place upgrade preserves your home directory, installed applications, and system configuration.
"Can I skip multiple releases?"	Not safely. Fedora only supports upgrading from N to N+1 or N+2. If you're three releases behind, you'd need to upgrade twice or do a fresh install.
The Upgrade Cadence: Your Options

Option A: Upgrade every release (every ~6 months)

You always have the newest software. Minimal delta between upgrades (fewer packages change per upgrade). Smallest chance of surprises. The downside is more frequent upgrading, but each upgrade is smaller and faster.

Option B: Upgrade every other release (every ~12 months)

You skip one release and jump two versions. More changes per upgrade, but still well within the supported window. This is the sweet spot for many users — once a year, like an annual OS refresh.

Option C: Stay until the last minute

Wait until your release is about to hit EOL, then upgrade. This maximizes stability (the release you're on has received 13 months of bug fixes) but means a bigger jump and less runway if something goes wrong.

    💡 Recommendation for New Users

    If you're new to Fedora, don't upgrade during the first two weeks of a new release. Let early adopters find the rough edges. After that window, upgrades are generally smooth. Once you've done one upgrade successfully, you'll understand the process and it stops feeling intimidating.

How Fedora Releases Are Made

Understanding the release process helps you appreciate why Fedora is reliable despite its fast pace. The process involves several stages:
1. Rawhide: The Rolling Development Branch

Rawhide is Fedora's continuously-updating development branch. It's always open, always moving. Packages land in Rawhide as soon as maintainers update them — sometimes within hours of an upstream release.

Rawhide is:

    Rolling — there's no "Rawhide release." It just flows.
    Unstable — things break. APIs change. Libraries disappear.
    Where development happens — new features land here first.
    Not for production — unless you enjoy troubleshooting.

Most Fedora users never touch Rawhide. But it's the engine that feeds the release process. Everything that eventually lands in a stable Fedora release starts in Rawhide.
2. Branched: The Stabilization Phase

About 2–3 months before a target release date, a branch is created from Rawhide. This becomes the upcoming release (e.g., Fedora 44 was branched from Rawhide around January/February 2026).

The branched release enters a stabilization phase:

    Feature freeze — no new features accepted. Only bug fixes and polish.
    String freeze — user-visible text (in dialogs, menus, etc.) is frozen so translators can finalize localized versions.
    Beta release — a testing milestone. The public is invited to try the beta and report bugs.
    Release Candidate (RC) — near-final builds. Each RC is evaluated in a Go/No-Go meeting.

3. Go/No-Go Meetings

Before each release, the Fedora community holds a Go/No-Go meeting — a public, transparent decision-making process. The release team reviews:

    Are there any blocking bugs (release-blocking bugs that must be fixed)?
    Do all the release criteria pass? (Installation works, desktop boots, networking functions, etc.)
    Are the spins and editions also ready?

If any blocking bug is unresolved, the release is declared No-Go and slips by one week (sometimes two). This repeats until everything passes.

This is fundamentally different from commercial OS releases that ship on a marketing-decided date regardless of readiness. Fedora would rather delay than ship broken software.
4. General Availability (GA)

Once Go is declared:

    Final ISO images are built and signed
    Images are distributed to mirror servers worldwide
    The release is announced on the Fedora Magazine
    Repository paths are finalized

At this point, the release is live. Existing Fedora users can upgrade. New users can download the ISO.
5. Post-Release: Updates

After GA, the release enters its ~13-month support window. Updates flow through Bodhi — Fedora's update gating system:

    A maintainer submits a package update to Bodhi
    The update enters a testing period (usually a few days, or faster with enough positive feedback/karma)
    Users who enable the updates-testing repository can try it and provide feedback
    Once the testing period passes (or enough users give it positive "karma"), the update is pushed to the stable repository
    All users receive it

This gating process means that even after release, updates aren't instantaneous — they're tested by real users before reaching everyone. It's a balance between freshness and stability.
Upgrading Between Releases

The upgrade process is one area where Fedora's "no LTS" philosophy could scare newcomers — so let's demystify it completely.
The Command-Line Method (Recommended)

The standard Fedora upgrade process uses dnf system-upgrade:

Step 1: Update your current system fully:

sudo dnf upgrade --refresh

Step 2: Install the upgrade plugin (if not already present):

sudo dnf install dnf-plugin-system-upgrade

Step 3: Download the packages for the new release:

sudo dnf system-upgrade download --releasever=45

Replace 45 with the target version number. This downloads all the packages that will be upgraded — but doesn't install them yet. You can continue using your system during this phase.

If there are dependency conflicts, DNF will tell you. The most common fix is:

sudo dnf system-upgrade download --releasever=45 --allowerasing

The --allowerasing flag tells DNF to remove packages that conflict with the upgrade. Pay attention to what it wants to remove — if it lists something important, investigate before proceeding.

Step 4: Reboot and apply:

sudo dnf system-upgrade reboot

Your system reboots into a special upgrade environment. You'll see a progress bar as packages are installed. This typically takes 15–40 minutes depending on your hardware and how much has changed.

When it's done, the system reboots again into the new release. Your files, settings, and applications are preserved.
The GNOME Software Method (Graphical)

If you prefer a graphical approach, GNOME Software (Fedora's app store) handles upgrades:

    When a new release is available, GNOME Software displays a banner: "Fedora 45 is available"
    Click Download
    After download completes, click Install and Restart
    The system reboots into the upgrade process (same as the CLI method, just triggered from the GUI)

The graphical method does the same thing as the command-line method — it's just wrapped in a friendlier interface.
Before Upgrading: Best Practices
Practice	Why
Back up your data	Upgrades are reliable, but backups are insurance against the unexpected. See Chapter 26.
Update your current system fully	Starting from a fully-patched system minimizes conflicts during upgrade.
Read the release notes	Each release has notes describing major changes, known issues, and things that might require attention. Find them at docs.fedoraproject.org.
Disable third-party repos temporarily	If you've added custom repositories (like RPM Fusion), they might not be ready for the new release yet. DNF will warn you about this.
Ensure sufficient disk space	The upgrade downloads packages (typically 2–4 GB) and needs temporary space to install them. Have at least 10 GB free on your root partition.
Don't rush	Wait 1–2 weeks after GA for early bugs to settle. The release will still be there.
After Upgrading: What to Check

    Verify the version:

    cat /etc/fedora-release

    Should show: Fedora release 44 (Forty Four) (or whatever version you upgraded to).

    Check for leftover packages:

    sudo dnf autoremove

    Removes packages that were dependencies of the old release but are no longer needed.

    Clean the DNF cache:

    sudo dnf clean all

    Re-enable third-party repos: If you disabled RPM Fusion or other repos, re-enable them for the new release version.

    Test critical applications: Launch your browser, email, editor, and any development tools. Make sure everything works.

Fedora vs. Ubuntu: Lifecycle Comparison

Since this manual series started with Ubuntu, here's a direct comparison:
Aspect	Ubuntu 26.04 LTS	Fedora 44
Release frequency	Every 6 months	Every 6 months
LTS?	Yes, every 2 years (5-year support)	No LTS — ever
Support duration	5 years (standard), ~10 years (ESM)	~13 months
Upgrade frequency	Optional for 5 years on LTS	Required at least once per 13 months
Upgrade method	do-release-upgrade (CLI) or Update Manager (GUI)	dnf system-upgrade (CLI) or GNOME Software (GUI)
Risk of falling behind	Low — LTS gives you years	Higher — must upgrade annually
Software freshness	LTS: older but stable. Interim: moderate	Always current — latest stable upstream releases
Kernel updates within release	HWE stacks (occasionally)	Regular kernel updates throughout the release's life
End-of-life behavior	Stops updating, but system keeps working	Repos go away — must upgrade to receive any updates

The fundamental trade-off: Ubuntu LTS gives you years of peace but older software. Fedora gives you fresh software but requires you to participate in the upgrade cycle.

    💡 Is the Upgrade Burden Real?

    In practice, Fedora upgrades are routine. After your first one or two, the process feels like a large system update — because that's essentially what it is. The dnf system-upgrade tool was designed specifically to make this painless. Most users report the entire process taking 20–30 minutes, with minimal or no manual intervention.

    The psychological barrier ("I have to upgrade?!") is larger than the technical reality.

What Happens If You Don't Upgrade?

If your Fedora release reaches end of life:

    Your system keeps working. Fedora doesn't expire, deactivate, or lock you out. The desktop still loads, applications still run, your files are intact.
    You stop receiving updates. No security patches, no bug fixes, no new package versions. Your system is frozen in time.
    Repositories go away. The updates and fedora repositories for that release are moved to the archive. If you try to install new software with dnf install, it may fail with 404 errors until you repoint your repos to the archive.
    Security risk grows. Over time, unpatched vulnerabilities accumulate. The longer you stay on an EOL release, the riskier it becomes.
    You can still upgrade — but it may require extra steps. If you're only one release behind, dnf system-upgrade works fine. If you're two or more behind, you may need to reconfigure repository URLs to point to the archive, or simply do a fresh installation.

Re-enabling Archived Repositories

If you need to install a package on an EOL release (e.g., to get dnf system-upgrade working), you can redirect repositories to the Fedora archive:

sudo sed -i 's|^baseurl=.*|baseurl=https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/44/Everything/x86_64/os/|' /etc/yum.repos.d/fedora.repo

(This example points Fedora 44's repo to the archive. Adjust the version number and architecture as needed.)

This is a rescue technique, not a long-term strategy. The right answer is to upgrade.
Release Codenames and Community Culture

As mentioned, modern Fedora releases use only version numbers. But historically, the community chose codenames through a voting process. Some were memorable:
Version	Codename	Note
Fedora 16	Beefy Miracle	A hot dog. Yes, really. The community voted for it.
Fedora 17	Beefy Miracle (rejected, reused as placeholder)	The naming committee had opinions.
Fedora 19	Schrödinger's Cat	Quantum physics humor.
Fedora 20	Heisenbug	A bug that changes behavior when you try to observe it.
Fedora 21	None — naming discontinued	Fedora shifted to pure version numbering.

The codename tradition reflected Fedora's playful, community-driven culture. While the names are gone, the spirit remains — Fedora is a project that takes its software seriously but doesn't take itself too seriously.
Choosing Your Update Strategy

Based on everything above, here's a practical framework:
For Most Users: Upgrade Once Per Year

Stay on each release for roughly 12 months. Upgrade shortly after the spring or fall release settles (give it 2–3 weeks). This means you:

    Always have a supported system
    Get major software updates once a year (reasonable)
    Minimize upgrade frequency (less disruption)
    Experience a smooth, well-tested upgrade path

For Enthusiasts: Upgrade Every Release

If you enjoy new technology and want the latest features, upgrade every six months. Each upgrade is smaller (less changes between consecutive releases), reducing the chance of issues. You're always on the freshest Fedora.
For Cautious Users: Upgrade Late

Wait until 2–3 months before your release's EOL. This gives the release maximum time to receive bug fixes. The downside is a larger jump to the new release (two versions of changes instead of one).
For No One: Never Upgrade

This isn't an option with Fedora. If you want a "set it and forget it for five years" system, Fedora isn't the right distribution. Ubuntu LTS, Debian, or AlmaLinux (a RHEL rebuild) would serve that need better.
Fedora's Relationship to Other Distributions

To contextualize Fedora's lifecycle, here's how it compares to other distributions:
Distribution	Release Model	Support Duration	Philosophy
Fedora	6-month releases	~13 months per release	Innovation, upstream-first
Ubuntu	6-month releases + LTS every 2 years	5 years (LTS), 9 months (interim)	Broad appeal, ease of use
Debian	Release when ready (every ~2 years)	~3 years (plus LTS extension)	Stability above all
Arch	Rolling release	N/A — always current	Simplicity, user control
AlmaLinux / Rocky Linux	RHEL rebuilds	10+ years per major release	Enterprise stability, free
openSUSE Tumbleweed	Rolling release (tested)	N/A — always current	Balance of freshness and stability
openSUSE Leap	Regular releases	~2.5 years	Stability, enterprise-oriented
Linux Mint	6-month releases + LTS	5 years (LMDE LTS)	User-friendliness, Windows-like

Fedora occupies a unique niche: more current than Debian/Ubuntu LTS, more stable than Arch, and shorter-lived than any of them. It's the distribution for people who want new software from a reputable, well-organized project — without the chaos of a rolling release.
What's Next

In Chapter 4, we'll help you transition from your current operating system. Whether you're coming from Windows 11, Windows 10, or macOS, we'll map the concepts you already know to their Fedora equivalents, identify what transfers and what doesn't, and prepare you mentally for the changes ahead.
Try It Yourself

    Check your Fedora version. Open a terminal and type:

    cat /etc/fedora-release

    You'll see something like Fedora release 44 (Forty Four). That's your current version.

    Also try:

    cat /etc/os-release

    This shows more detailed information, including the version number, codename (if any), and the Fedora home page URL.

    Check your kernel version. Linux kernel versions move quickly in Fedora:

    uname -r

    Fedora 44 ships with 6.19, but kernel updates happen regularly. Your exact version may differ.

    Read the Fedora 44 release notes. Visit docs.fedoraproject.org and find the Fedora 44 release notes. Skim the "Highlights" section. You'll see what changed from Fedora 43 — GNOME 50, DNF5 everywhere, NTSYNC for gaming, Whisper AI, and more.

    Check how long until your release expires. Visit endoflife.date/fedora. This site tracks EOL dates for Fedora and many other distributions. Knowing your timeline helps you plan your upgrade strategy.

    Browse the Fedora Wiki release history. Search for "Fedora release history" and look at the timeline going back to Fedora Core 1 in 2003. Notice how the release cadence has been remarkably consistent — two releases per year, year after year. This consistency is a strength.

    ✅ Chapter 3 Complete You understand Fedora's six-month release cycle, sequential version numbering, the ~13-month support lifecycle, why there's no LTS, the three upgrade strategies (every release, every other release, or late), the release development process (Rawhide → Branched → Beta → RC → Go/No-Go → GA), how the Bodhi update gating system works, how to perform a dnf system-upgrade, what happens at EOL, and how Fedora's lifecycle compares to other major distributions.

