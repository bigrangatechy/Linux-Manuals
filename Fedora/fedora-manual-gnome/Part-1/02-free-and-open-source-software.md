# Chapter 2: Free and Open Source Software: The Philosophy

## Why This Chapter Matters

You might be tempted to skip this chapter. "I just want to use my computer," you think. "Why do I need a philosophy lesson?"

Here's why: Fedora is built on principles that shape every decision the project makes. When you understand those principles, dozens of things about Fedora that might otherwise seem confusing or frustrating suddenly make sense. Why can't I play this video file? Why isn't my Wi-Fi card working out of the box? Why does the App Center offer fewer apps than the Ubuntu App Center? Why does Fedora update so often?

Every one of these answers traces back to the philosophy in this chapter.

This isn't abstract theory. It's the operating manual for understanding Fedora's soul.

---

## What Does "Free" Mean?

In English, the word "free" is famously ambiguous. It can mean:

- **Free as in cost** — you didn't pay money for it (free beer)
- **Free as in liberty** — you have freedom to use, study, modify, and share it (free speech)

Fedora's "free" is the second kind. And the distinction matters enormously.

Consider a piece of software you downloaded for free. It costs nothing. But can you:

- Look at the source code to see how it works?
- Modify that code to fix a bug or add a feature?
- Share your modified version with others?
- Audit the software for security vulnerabilities?
- Verify that the software isn't secretly doing something malicious?

If the answer to any of these is "no," then the software is free of charge but not free in the philosophical sense. It's a gift you can use but can't inspect, modify, or redistribute.

Free and open source software (often abbreviated **FOSS**) is free in both senses — but the liberty is what matters. The cost (or lack of it) is a side effect, not the goal.

> 💡 **The Confusing Terminology**
>
> You'll encounter several overlapping terms in the Linux world:
>
> - **Free Software** — emphasizes the ethical freedom of users (championed by the Free Software Foundation)
> - **Open Source** — emphasizes the practical benefits of open development (championed by the Open Source Initiative)
> - **FOSS** (Free and Open Source Software) — a neutral umbrella term acknowledging both perspectives
> - **FLOSS** (Free/Libre/Open Source Software) — the same thing, with "Libre" added to disambiguate from "free beer" in languages where the word for "free" also means "costless"
>
> These terms are used interchangeably in casual conversation, but they represent slightly different philosophical emphases. Fedora aligns more closely with the "free software" tradition, though it uses the term "open source" in practice.

---

## The Four Freedoms

The concept of free software was formalized in the 1980s by **Richard M. Stallman** (often referred to as RMS), a programmer at MIT's Artificial Intelligence Laboratory. Stallman defined free software using four specific freedoms:

**Freedom 0: The freedom to run the program as you wish, for any purpose.**

No restrictions on what you can do with the software. You can use it at home, at work, to write poetry, to run a business, to analyze data. The software doesn't care and doesn't restrict.

**Freedom 1: The freedom to study how the program works and change it so it does your computing as you wish.**

You can read the source code. You can understand what the program does internally. You can modify it to behave differently. The prerequisite is access to the source code — the human-readable instructions that are compiled into the program you run.

**Freedom 2: The freedom to redistribute copies so you can help your neighbor.**

You can share the software. You can give it to friends, post it online, include it in a larger project. You're not restricted by artificial scarcity, DRM, or licensing fees.

**Freedom 3: The freedom to distribute copies of your modified versions to others.**

Not only can you modify the software, you can share your improvements. This ensures that improvements benefit everyone, not just the person who made them. The community as a whole advances.

If any of these four freedoms is missing or restricted, the software is not free software — regardless of whether it costs money.

---

## The GNU Project and the Free Software Foundation

### The Problem Stallman Saw

In the 1970s and early 1980s, the culture of software at places like MIT's AI Lab was collaborative and open. Researchers shared code freely, improved each other's work, and built on a shared foundation. But a shift was underway in the broader industry.

Software was becoming a product — something to be sold, locked down, and kept secret. Companies began distributing software only in compiled (unreadable) form, with licenses that forbade sharing, modification, or inspection. Printers came with software you couldn't modify to fix bugs. Operating systems were black boxes.

Stallman saw this as an ethical crisis. Software was fundamental to computing, and computing was becoming fundamental to society. If software was locked down, people lost control over their own digital lives.

### The GNU Project (1983)

In September 1983, Stallman announced the **GNU Project** (GNU is a recursive acronym: "GNU's Not Unix"). The goal was ambitious: create a complete, free operating system — every component, from the kernel to the compiler to the text editor — licensed so that users had all four freedoms.

Over the next decade, GNU produced an extraordinary amount of high-quality software:

| GNU Component | What It Does |
|---|---|
| **GCC** (GNU Compiler Collection) | Compiles C, C++, and other languages |
| **Bash** (Bourne Again Shell) | The default shell on virtually every Linux system |
| **GNU Coreutils** | `ls`, `cp`, `mv`, `cat`, `chmod`, `dd` — the basic commands you use every day |
| **Glibc** | The C library that nearly every Linux program links against |
| **Emacs** | A legendary text editor (and the subject of an eternal holy war with Vim) |
| **GDB** | The GNU Debugger |
| **make** | Build automation tool |
| **gzip** | Compression utility |

By the early 1990s, GNU had built almost an entire operating system — a shell, core utilities, compilers, a debugger, text editors, and more. But one critical piece was missing: the **kernel**. GNU's own kernel project, called **Hurd**, was architecturally ambitious but perpetually delayed.

### Enter Linux (1991)

In 1991, Linus Torvalds, a Finnish university student, released the Linux kernel. It was pragmatic, functional, and — crucially — licensed under the GNU General Public License (GPL).

The combination of the GNU userland (the shell, utilities, and tools) with the Linux kernel created the first complete, fully free operating system. This is why some people insist on calling the operating system "GNU/Linux" — because the system you use is GNU software running on the Linux kernel. In practice, most people just say "Linux."

### The Free Software Foundation (FSF)

Stallman founded the **Free Software Foundation (FSF)** in 1985 to support the GNU Project and advocate for free software. The FSF remains a vocal advocate for user freedom, campaigning against DRM, proprietary software, and practices it considers harmful to user autonomy.

While Fedora is not an FSF-aligned distribution (Fedora ships some firmware blobs that the FSF considers non-free, and doesn't maintain FSF endorsement), the FSF's philosophy deeply influences Fedora's commitment to free software by default.

---

## The GNU General Public License (GPL)

Licenses are the legal mechanism that makes free software work. Without a license granting the four freedoms, software isn't free — even if the source code is publicly visible.

The most important license in the free software world is the **GNU General Public License (GPL)**, also written by Stallman. The GPL has gone through several versions:

| Version | Year | Key Feature |
|---|---|---|
| **GPLv1** | 1989 | Established the concept of copyleft — derivative works must also be free |
| **GPLv2** | 1991 | Added the "liberty or death" clause — if you can't distribute freely, you can't distribute at all. The Linux kernel uses GPLv2. |
| **GPLv3** | 2007 | Addressed software patents, tivoization (hardware that locks down modified software), and DRXM anti-circumvention issues |

### Copyleft: The Viral License

The GPL uses a mechanism called **copyleft** — a deliberate inversion of copyright. Instead of using copyright to restrict what users can do, copyleft uses copyright to guarantee they can do everything.

Here's how it works:

1. The original author copyrights their software.
2. They grant everyone the four freedoms via the GPL license.
3. **The catch:** If you modify and distribute GPL-licensed software, your modified version must also be GPL-licensed.

This is sometimes called a "viral" license (a term some find pejorative), because the GPL terms spread to derivative works. If you take a GPL program, modify it, and distribute your version, you can't make it proprietary. Your modifications must also be free.

Copyleft is the reason Linux can't be "taken private." A company can't take the Linux kernel, improve it, and sell a closed-source version. Any modifications they distribute must be published under the GPL. This protects the commons — the shared foundation of free software that everyone builds on.

### Permissive Licenses: The Other Approach

Not all open source licenses use copyleft. **Permissive licenses** grant broad freedom without requiring derivative works to stay open:

| License | Philosophy | Famous Users |
|---|---|---|
| **MIT License** | Do whatever you want, just keep the copyright notice | React, jQuery, Ruby on Rails |
| **BSD License** | Similar to MIT, with variations | FreeBSD, PostgreSQL (historically) |
| **Apache 2.0** | Like MIT/BSD but with explicit patent grants | Kubernetes, Android (Apache components) |

Permissive licenses let you take the code, modify it, and release your version as proprietary software. Apple's macOS incorporates BSD-licensed code. Microsoft has incorporated open source code into Windows under permissive licenses.

### Which Is Better?

The debate between copyleft (GPL) and permissive licenses (MIT/BSD/Apache) is one of the oldest arguments in the open source community. Neither is universally "right":

- **Copyleft advocates** argue that permissive licenses let corporations exploit community work without giving back. They see GPL as protecting the commons.
- **Permissive advocates** argue that forcing openness limits adoption and creates legal friction. They see MIT/BSD as maximizing freedom.

The Linux kernel (GPLv2) uses copyleft. The GNOME Project (GPLv2+) uses copyleft. Fedora uses both GPL'd and permissively-licensed software. The Fedora Project itself doesn't take a position on which license is better — it accepts software under any license that meets its licensing guidelines (roughly: OSI-approved or FSF-approved).

> 💡 **Practical Takeaway**
>
> As a Fedora user, you don't need to understand every license clause. But knowing the GPL exists and what it requires helps you understand why so much Linux software is free: the license legally guarantees it stays that way.

---

## Open Source vs. Free Software: The Split

You might think "free software" and "open source software" are the same thing. In practice, they usually refer to the same software. But the two terms emerged from a philosophical split in the 1990s that still resonates today.

### The Original Movement: Free Software (1980s)

Stallman's free software movement was — and remains — fundamentally ethical. Its core argument: software controls your computing, and if you can't control your software, you don't control your computing. Proprietary software is an injustice. It divides users, keeps them helpless, and prevents community.

This is a moral stance. The Free Software movement doesn't say "free software is better software" — it says "free software is the only ethical choice."

### The Pragmatic Schism: Open Source (1998)

By the late 1990s, the term "free software" had a perception problem. Corporate America associated "free" with "worthless." Executives heard "free software" and thought "no business model, no support, no accountability." And Stallman's uncompromising moral framing made business leaders nervous.

In 1998, a group of developers and advocates — including Eric S. Raymond (author of "The Cathedral and the Bazaar"), Bruce Perens, and Tim O'Reilly — proposed a new label: **"open source."** The term was deliberately chosen to emphasize the practical benefits of open development:

- Better software through more eyeballs (Linus's Law: "Given enough eyeballs, all bugs are shallow")
- Faster innovation through collaboration
- Vendor independence and avoiding lock-in
- Security through transparency

The open source movement deliberately set aside the ethical dimension. It didn't argue that proprietary software was immoral — it argued that open source produces better results. This was a business-friendly message.

### The Two Camps Today

The split persists:

| | Free Software | Open Source |
|---|---|---|
| **Origin** | Richard Stallman / FSF (1980s) | Raymond, Perens et al. / OSI (1998) |
| **Core argument** | Software freedom is an ethical imperative | Open development produces better software |
| **View of proprietary software** | An injustice to users | A legitimate choice, just technically inferior |
| **Tone** | Moral, principled, uncompromising | Pragmatic, business-friendly, inclusive |
| **Representative organization** | Free Software Foundation (FSF) | Open Source Initiative (OSI) |

In practice, the vast majority of software qualifies as both — it's free software (respecting the four freedoms) and open source (meeting the Open Source Definition). The disagreement is about motivation, not about which software is acceptable.

Fedora straddles both worlds. Its **Freedom** foundation echoes the free software ethic. Its **Features** and **First** foundations echo the open source pragmatism. The project maintains strict free software licensing guidelines (aligning with the free software tradition) while enthusiastically promoting practical innovation (aligning with the open source tradition).

> 💡 **Stallman Would Say...**
>
> Richard Stallman famously insists on "GNU/Linux" rather than "Linux" and bristles at "open source" as a dilution of the free software cause. You don't need to agree with him, but understanding his position helps you understand why Fedora is the way it is. The refusal to ship proprietary codecs, the insistence on free firmware, the commitment to open formats — these are free software principles in action.

---

## The Open Source Definition

In 1998, Bruce Perens drafted the **Open Source Definition (OSD)**, adapting it from the Debian Free Software Guidelines. The OSD is a ten-point checklist that defines what qualifies as open source:

1. **Free Redistribution** — Anyone can sell or give away the software. No royalty payments to original authors.
2. **Source Code** — The source must be available, preferably for download. Obfuscated source doesn't count.
3. **Derived Works** — Modifications must be allowed and distributable under the same license.
4. **Integrity of the Author's Source Code** — Modified versions may require a different name or version number (to protect the original author's reputation).
5. **No Discrimination Against Persons or Groups** — The license can't restrict who uses the software.
6. **No Discrimination Against Fields of Endeavor** — The license can't say "for non-commercial use only" or "not for nuclear facilities."
7. **Distribution of License** — Rights attach to everyone who receives the software, automatically. No NDA requirements.
8. **License Must Not Be Specific to a Product** — Rights don't depend on the software being part of a particular distribution.
9. **License Must Not Restrict Other Software** — Can't say "you may only use this with open source software."
10. **License Must Be Technology-Neutral** — No restrictions based on technology or field of use.

Any license meeting all ten criteria is considered open source by the Open Source Initiative (OSI). Fedora's licensing guidelines accept any license that is OSI-approved and/or FSF-approved.

---

## Fedora's Commitment to Free Software

### The Fedora Licensing Guidelines

Fedora maintains strict licensing guidelines that govern what software can be included in the official repositories. In summary:

- All software in Fedora must be under an **OSI-approved open source license** (or a Fedora-acceptable free license)
- The license must allow **modification and redistribution** of both original and modified versions
- **No proprietary licenses** — software under a non-free license cannot be packaged for Fedora's official repos
- The license must be ** Fedora-compatible** — some licenses conflict with each other (e.g., GPLv2-only code can't be combined with GPLv3-only code in a single program)

This is why certain things are conspicuously absent from Fedora by default.

### What Fedora Refuses to Ship (By Default)

| Absent Because... | Examples |
|---|---|
| **Patent-encumbered codecs** | MP3, H.264/H.265 (HEVC), AAC, MPEG video. These formats are patented. Fedora's policy excludes software that requires patent licenses Fedora cannot grant. |
| **Proprietary drivers** | NVIDIA's closed-source GPU drivers. These are distributed under a restrictive license that doesn't meet Fedora's guidelines. |
| **Non-free firmware** | Certain Wi-Fi cards and GPUs require binary-only firmware blobs that aren't freely redistributable. (Fedora has relaxed this somewhat for essential firmware, but the principle remains.) |
| **Closed-source applications** | Spotify, Discord, Zoom, Slack, Steam — these are proprietary. They're available as Flatpaks from Flathub, but not from Fedora's repos. |
| **Non-free media formats** | Some fonts, media samples, or test files have restrictive licenses. |

### The RPM Fusion Solution

Fedora's refusal to ship proprietary code doesn't mean you're stuck without it. **RPM Fusion** is a community-maintained third-party repository that provides exactly what Fedora can't:

- Multimedia codecs (MP3, H.264, AAC, DVD playback)
- NVIDIA proprietary drivers
- Software with patent encumbrances
- Other packages that don't meet Fedora's free software guidelines

RPM Fusion is the most common first stop after a fresh Fedora install. Chapter 11 walks you through enabling it.

> ⚠️ **A Philosophical Choice**
>
> Whether to enable RPM Fusion is a personal choice. Some Fedora users refuse on principle — they use only free formats (Ogg Vorbis, Opus, WebM, AV1) and accept the inconveniences. Others enable RPM Fusion immediately and never think about it again.
>
> There's no wrong answer. Fedora respects your choice either way. The philosophy matters to some people more than others, and both are valid Fedora users.

### Flatpak and Flathub: A Middle Ground

Fedora embraces **Flatpak** — a package format that sandboxes applications and distributes them independently of the system. Flatpaks can contain proprietary software (Spotify, Discord, Zoom) because they're not part of Fedora itself — they're containerized applications running on top of Fedora.

Fedora's **GNOME Software** app integrates Flathub (the primary Flatpak repository) directly. When you search for an app in GNOME Software, results may include both RPM packages from Fedora's repos and Flatpaks from Flathub.

This gives Fedora the best of both worlds: a strict, free-by-default base system, plus easy access to proprietary applications when users want them — clearly separated, sandboxed, and optional.

---

## The Practical Impact on Your Daily Life

You might be wondering: "How does all this philosophy affect me when I'm sitting at my desk?" Here are the concrete ways:

### You Can Audit Everything

If a program on your Fedora system behaves suspiciously, you can trace its source code, examine what it does, and verify its behavior. You can't do this with proprietary software. This is why free software is particularly valued in security research, privacy advocacy, and government contexts.

### No Telemetry by Default

Many proprietary operating systems collect usage data. Fedora doesn't. There's no "share my diagnostics" toggle buried in settings, no background analytics phoning home. Fedora trusts you, and respects your privacy.

### No Vendor Lock-In

Fedora's file formats are open. Your documents, configurations, and data aren't stored in proprietary formats that tie you to one application. If you don't like a program, you can export your data and use a different one.

### You Are the Customer, Not the Product

When a company gives you software for free, you're often the product — your data, attention, and behavior are monetized. Fedora isn't a company, doesn't track you, and has nothing to sell. You are the user, full stop.

### Updates Serve You, Not a Schedule

Proprietary software updates when the vendor decides — sometimes to push new features, sometimes to enforce new restrictions. Fedora updates happen because a security patch was released, a bug was fixed, or a new upstream version is available. The updates serve your interests.

### But: Inconveniences Exist

The philosophy has trade-offs:

- **Media playback:** Out of the box, Fedora can't play MP4 videos or MP3 audio. You need to enable RPM Fusion (Chapter 11).
- **Hardware support:** Some Wi-Fi cards and GPUs need proprietary firmware or drivers that Fedora won't include by default. You may need to install them manually.
- **Application availability:** Some proprietary apps (Adobe Creative Cloud, Microsoft Office) have no Linux version at all — free or otherwise. You'll use alternatives (GIMP, LibreOffice) or web versions.
- **Learning curve:** Fedora expects you to make informed choices. It won't make decisions for you by bundling convenient but non-free software.

---

## The Broader Open Source Ecosystem

Fedora doesn't exist in a vacuum. It's part of a vast ecosystem of projects, organizations, and communities that collaborate on free software:

### Upstream and Downstream

In open source, **upstream** refers to the original source of a project, and **downstream** refers to projects that incorporate that source.

[GNOME Project] → [Fedora] → [CentOS Stream] → [RHEL] (upstream) (we are here) (downstream) (commercial)

Fedora is both upstream and downstream:

    Downstream of: GNOME, the Linux kernel, GNU Project, Python, systemd, and hundreds of other projects. Fedora takes their releases, packages them, and integrates them.
    Upstream of: CentOS Stream and RHEL. Features that land in Fedora may eventually flow to Red Hat's enterprise products.

The "Upstream First" Principle

Fedora follows an "upstream first" philosophy. When Fedora developers fix a bug or add a feature, they submit the change to the upstream project first — not just patch it locally. Only after the upstream project accepts the change does Fedora include it.

This benefits everyone:

    The upstream project gets the improvement
    Other distributions that use the same upstream benefit automatically
    Fedora doesn't carry "Fedora-only" patches that could diverge from the rest of the Linux world
    Bug fixes propagate throughout the entire ecosystem

This contrasts with some distributions that carry large numbers of downstream-only patches. When you use Fedora, you're using software that's as close to what the original developers intended as possible.
The Contribution Model

Anyone can contribute to Fedora. The project has working groups for development, design, infrastructure, marketing, QA, documentation, translation, and community outreach. Contributors range from Red Hat employees to hobbyists to students to retirees.

Common ways to contribute:

    Package maintenance — maintaining RPM packages in Fedora's repositories
    Testing — trying beta releases and reporting bugs
    Documentation — writing or improving guides at docs.fedoraproject.org
    Translation — translating software and documentation into other languages
    Design — creating artwork, icons, and marketing materials
    Infrastructure — helping maintain Fedora's servers and services
    Advocacy — spreading the word, writing blog posts, giving talks

    💡 Fedora Is What You Make It

    Unlike commercial operating systems where users have no voice, Fedora is a community project. If you want a feature added, you can propose it. If you find a bug, you can report it — and watch it get fixed. If you disagree with a decision, you can argue your case in the community forums. This isn't just rhetoric; it's how the project actually operates.

The Cathedral and the Bazaar

In 1997, Eric S. Raymond published an essay titled "The Cathedral and the Bazaar" that became one of the most influential texts in open source history. Raymond contrasted two models of software development:

The Cathedral Model:

    Carefully crafted by a small team of experts
    Released infrequently, polished and complete
    Closed development process between releases
    Example: GNU Emacs, older commercial software development

The Bazaar Model:

    Developed openly and publicly
    Released early and often — "release early, release often"
    Anyone can contribute
    Bugs are shallow with enough eyeballs
    Example: the Linux kernel development process

Raymond argued that the bazaar model — open, collaborative, iterative — produces better software than the cathedral model. The essay influenced Netscape to open-source its browser (which eventually became Mozilla Firefox, the browser shipped with Fedora).

Fedora embodies the bazaar model. Development happens in the open, on public mailing lists, in visible Git repositories, through community discussions. Decisions are documented and archived. You can watch a feature being proposed, debated, implemented, tested, and released — all in public.
How Red Hat Fits In

Red Hat's role deserves clarification because misconceptions are common:
Red Hat Is Not Oracle

Red Hat is a unique technology company. Unlike most software companies, its entire business model is based on open source software. It doesn't sell proprietary software — it sells services, support, and certification around open source software.

Red Hat's revenue comes from:

    Enterprise subscriptions for RHEL (paid support, certified hardware/software compatibility, SLA-backed updates)
    Consulting and training
    Certifications (Red Hat Certified Engineer, etc.)

But the software itself is free. Red Hat's RHEL source code is publicly available (though the binary packages require a subscription). CentOS Stream provides the open development branch for free.
Red Hat Sponsors Fedora

Red Hat sponsors Fedora by employing many of its contributors and providing infrastructure (servers, build systems, bandwidth). But Red Hat does not:

    Sell Fedora (it's free)
    Require any registration or subscription for Fedora
    Insert telemetry or advertising into Fedora
    Dictate Fedora's direction or veto community decisions

Fedora is governed by the Fedora Council — an elected community body. Red Hat employees participate as community members, not as corporate overlords.
The RHEL Pipeline

Fedora's role as RHEL's upstream is sometimes misunderstood. Fedora is not "beta RHEL." It's a fully independent distribution with its own goals, its own community, and its own identity. But technologies that prove successful in Fedora may be adopted by CentOS Stream and eventually RHEL:

[Fedora] → [CentOS Stream] → [RHEL]
  Innovation    Integration    Stabilization
  (~6 months)   (continuous)   (10+ year support)

    Fedora is the fast-moving innovation laboratory
    CentOS Stream is the continuously updated development branch of RHEL
    RHEL is the commercially supported, long-term-stability enterprise product

Using Fedora means you see tomorrow's enterprise Linux today — but you're using Fedora, not a RHEL beta.
Summary: What This Means for You

Let's distill this entire chapter into practical takeaways:

    Fedora is free software. Free as in freedom, not just free as in cost. You can use, study, modify, and share everything in it.

    The four freedoms (use, study, modify, share) are the philosophical foundation. Licenses (especially the GPL) legally guarantee these freedoms.

    Fedora refuses proprietary code by default. Not out of hostility, but out of principle. This means some things don't work out of the box (codecs, some drivers). RPM Fusion and Flathub fill the gaps when you're ready.

    You're in control. No telemetry, no forced updates, no vendor lock-in, no hidden agendas. Your computer does what you tell it to.

    Fedora is upstream-first. Software is as close to what its original developers intended as possible. Fedora contributes improvements back to the community.

    Anyone can contribute. Fedora is a community project, not a corporate product. You can shape its direction.

    The philosophy has trade-offs. You'll occasionally hit inconvenience (needing RPM Fusion for a codec, missing a proprietary app). But you gain transparency, privacy, control, and software you can trust.

What's Next

In Chapter 3, we'll look at Fedora's release cycle — how versions are numbered, when they come out, how long they're supported, and what the upgrade process looks like. You'll understand why Fedora doesn't have an LTS version, and how the six-month cadence affects you as a daily user.
Try It Yourself

    Check the license of a Fedora package. Open a terminal (don't worry — we'll teach you the terminal properly in Chapter 10; for now just follow along) and type:

    rpm -qi bash

    Look for the "License" field. You'll see something like "GPLv3+". That's the GPL version 3 (or later). This means the Bash shell on your system is guaranteed free software.

    Explore the OSI license list. Visit opensource.org/licenses. Browse through the approved licenses. You'll see familiar names (MIT, BSD, Apache, GPL) and many you've never heard of. Each represents a slightly different philosophical or legal approach to software freedom.

    Read the Fedora Licensing Guidelines. Visit docs.fedoraproject.org and search for "licensing." Read the overview — not every detail, just the philosophy. You'll see how seriously Fedora takes this.

    Find a project to admire. Browse gitlab.gnome.org or gitlab.com/fedora. These are the actual source code repositories for GNOME and Fedora. You can read the code, see commit histories, file bug reports, and even propose changes. This transparency is what free software looks like in practice.

    Reflect on the trade-offs. Make a list of three proprietary applications you currently use. For each, search whether a free/open source alternative exists. Would you be willing to switch? What would you lose? What would you gain? There are no wrong answers — this is about understanding your own priorities.

    ✅ Chapter 2 Complete You understand the four freedoms (use, study, modify, share), the difference between free software and open source, the history of the GNU Project and the GPL, what copyleft means, Fedora's strict licensing guidelines, why Fedora doesn't ship proprietary code by default, how RPM Fusion and Flatpak bridge the gap, the upstream-first principle, Red Hat's relationship to Fedora, and the practical impact of all this philosophy on your daily experience.
