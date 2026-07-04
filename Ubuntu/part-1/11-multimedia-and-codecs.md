# Chapter 11: Multimedia and Codecs

## The "Why Won't This Play?" Question

You've just installed Ubuntu. You try to play a video file you downloaded — maybe an `.mp4` or `.mkv` — and nothing happens. Or the audio plays but the video is black. Or the video plays but there's no sound. Or a website tells you a plugin is missing.

This is one of the most common frustrations for new Linux users. The good news: it's completely fixable, usually with a single command. The better news: once fixed, it stays fixed permanently.

This chapter explains *why* this happens, walks you through fixing it, and introduces you to Ubuntu 26.04's built-in media players.

## What Are Codecs?

A **codec** (short for **co**der-**dec**oder) is a piece of software that compresses and decompresses digital media. When you watch a video or listen to music, your computer is using codecs to translate compressed file data into images and sound you can perceive.

Think of a video file as a letter written in a foreign language. The codec is the translator. Without the right translator, you can't read the letter — even though you're holding it.

### Common Video Codecs

| Codec | Typical File Extension | Used For |
|---|---|---|
| H.264 (AVC) | `.mp4`, `.mkv` | Most web video, Blu-ray, streaming |
| H.265 (HEVC) | `.mkv`, `.mp4` | 4K video, modern streaming |
| AV1 | `.mkv`, `.webm` | Next-gen web video (YouTube, Netflix) |
| VP9 | `.webm` | Web video (YouTube) |
| MPEG-2 | `.mpg`, `.vob` | DVDs, old broadcast TV |
| MPEG-4 (Part 2) | `.avi` | Older video files, DivX/XviD |
| WMV | `.wmv` | Windows Media Video |
| Theora | `.ogv` | Open source video format |

### Common Audio Codecs

| Codec | Typical File Extension | Used For |
|---|---|---|
| MP3 | `.mp3` | Universal music format |
| AAC | `.m4a`, `.mp4` | iTunes, streaming, YouTube |
| FLAC | `.flac` | Lossless audio (audiophile quality) |
| Opus | `.opus`, `.ogg` | Voice chat, streaming |
| Vorbis | `.ogg` | Open source audio |
| WAV | `.wav` | Uncompressed audio |
| AC-3 (Dolby Digital) | `.ac3` | DVD and cinema audio |

Your media player needs the correct codec for each format. If a codec is missing, playback fails — or plays partially (audio without video, video without audio, or corrupted output).

## Why Ubuntu Doesn't Include All Codecs by Default

This is the key question, and the answer has nothing to do with technical limitations. It's about **law**.

Many codecs — particularly H.264, H.265, MP3, AAC, and Dolby Digital — are protected by **software patents**. Companies and organizations (like the MPEG-LA, Dolby Laboratories, and Fraunhofer) hold patents on the mathematical algorithms used to encode and decode these formats.

In some countries (notably the United States), using patented software without paying licensing fees is illegal. Ubuntu is developed by Canonical, a company headquartered in the UK but operating globally. To avoid legal liability, Canonical ships Ubuntu without patented codecs enabled by default.

> 💡 **The Patent Landscape Is Changing**
>
> Some older codecs have had their patents expire. For example, the core MP3 patents expired in 2017, meaning MP3 decoding is now freely legal everywhere. However, newer codecs like H.265 and AV1 still have active patent pools.
>
> Ubuntu's conservative approach remains: ship without patented codecs, let users opt in.

### The Good News: Installing Codecs Is Easy

Ubuntu makes a single **meta-package** — a package that doesn't contain software itself but pulls in other packages as dependencies — that installs all the common codecs in one shot. It's called **ubuntu-restricted-extras**.

## Installing ubuntu-restricted-extras

This is the single most important step for multimedia on Ubuntu. If you followed the post-install checklist in Chapter 7, you may have already done this. If not, do it now.

### Step 1: Ensure the Multiverse Repository Is Enabled

The multiverse repository contains software with licensing or patent restrictions — including the codecs. It's usually enabled by default on Ubuntu 26.04, but let's verify:

1. Open a terminal (we'll cover terminals properly in Chapter 13, but for now: press **Super**, type "terminal," press Enter)
2. Type the following command and press Enter:

sudo add-apt-repository multiverse

3. Enter your password when prompted
4. Then update your package list:

sudo apt update


If multiverse was already enabled, the first command will say so and do nothing. That's fine.

### Step 2: Install the Package

sudo apt install ubuntu-restricted-extras

During installation, you'll see a blue/purple screen with a license agreement — this is for the Microsoft Core Fonts package (included in ubuntu-restricted-extras). Don't panic:

    Press Tab to highlight "OK" (it turns red/orange)
    Press Enter
    On the next screen, use the arrow keys to select "Yes" if you agree
    Press Tab to highlight "OK" again
    Press Enter

Installation continues automatically after that.
What This Package Installs

ubuntu-restricted-extras pulls in:

    GStreamer plugins (bad, ugly, libav) — the codec framework for GNOME media players
    FFmpeg libraries — widely-used multimedia processing backend
    Microsoft Core Fonts — Arial, Times New Roman, Courier New, etc. (so documents look right)
    unrar — for extracting .rar archive files
    MP3, AAC, H.264, H.265, WMV, MPEG-2, DivX decoding support
    Additional audio codecs for FLAC, AC-3, and others

After installation, your default media players can handle virtually all common media formats without further configuration.

    ⚠️ The EULA Screen Confuses Everyone

    The Microsoft fonts EULA screen is text-based (not graphical) and catches beginners off guard. The key is using Tab to navigate — there's no clickable "OK" button. This is a legacy interface from the debconf system, which we explain in Chapter 18.

For Kubuntu Users

If you're on Kubuntu, the package is slightly different — it installs KDE-specific codec packages:

sudo apt install kubuntu-restricted-extras

The principle is identical; the package name just targets the KDE multimedia framework instead of GNOME's.
GStreamer: The Engine Behind GNOME Media

You might be wondering: where do codecs actually live? The answer for GNOME-based Ubuntu is GStreamer.

GStreamer is a multimedia framework — essentially a pipeline system for handling audio and video data. When Showtime or Rhythmbox plays a file, they hand it to GStreamer, which selects the appropriate codec plugins to decode it.

GStreamer organizes plugins into four tiers:
Tier	What It Contains	Install Status
Good	Well-maintained, open-source, patent-free codecs (Vorbis, Theora, FLAC)	Included by default
Bad	Plugins of varying license quality (may have patent issues)	Installed via restricted-extras
Ugly	Plugins that definitely have patent issues (MP3, DVD CSS)	Installed via restricted-extras
Libav (formerly FFmpeg)	Broad codec support via the FFmpeg library	Installed via restricted-extras

You normally don't interact with GStreamer directly — it works invisibly behind your media players. But understanding this structure helps when troubleshooting: if a specific format won't play, it's usually because a GStreamer plugin is missing.
Installing Individual GStreamer Plugins (Optional)

If you need a codec that ubuntu-restricted-extras didn't cover (rare, but possible for niche formats), install the plugins individually:

sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav

This ensures all GStreamer codec tiers are present.
Ubuntu 26.04's Default Media Players

Ubuntu 26.04 replaces the old Totem (GNOME Videos) with a modern alternative:
Showtime (Video Player)

Showtime is Ubuntu 26.04's default video player, replacing the aging Totem. It's a clean, modern player designed for GNOME 50 with these features:

    Minimal interface — playback controls fade out while watching, returning on mouse movement
    Adjustable playback speed — slow down or speed up video (0.25x to 2x)
    Multi-language support — select audio tracks and subtitles for multi-track files
    Fullscreen mode — press F11 or click the fullscreen button
    Keyboard navigation — spacebar for play/pause, arrow keys for seek, etc.
    Format support — plays MP4, WebM, MKV, AVI (with codecs installed)

Showtime is intentionally simple — it plays videos well and stays out of your way. It doesn't have features like playlist management or streaming server connections. For those, you'd use a more full-featured player like VLC (below).

    💡 Showtime and MKV Files

    Some users on Ubuntu 26.04 have reported that Showtime struggles with certain .mkv files on fresh installs. This is typically a missing codec issue resolved by installing ubuntu-restricted-extras. If problems persist after installing codecs, VLC is the reliable fallback.

Rhythmbox (Music Player)

Rhythmbox remains Ubuntu's default music and podcast player. It's been the standard GNOME music app for over two decades:

    Music library management — scan folders, organize by artist/album/genre
    Playlist creation — static and smart playlists based on rules
    Podcast subscriptions — subscribe to RSS feeds, auto-download episodes
    Internet radio — stream from SHOUTcast and other radio directories
    Device syncing — sync with portable media players (limited smartphone support)
    CD ripping — convert audio CDs to digital files (requires codecs)
    Format support — MP3, OGG, FLAC, AAC (with codecs installed)

Rhythmbox is straightforward but somewhat dated. Many users eventually switch to alternatives like Spotify (available as a Snap), Elisa (KDE music player), or Lollypop (GNOME-focused music player).
The Reliable Alternative: VLC Media Player

If there's one piece of advice that experienced Linux users give to every newcomer, it's this: install VLC.

VLC Media Player is the Swiss Army knife of media players. It includes its own internal codec library — it doesn't depend on GStreamer or system-installed codecs. This means it plays virtually every format you'll encounter, out of the box, without any additional setup.
Installing VLC

Through the App Center (Chapter 10):

    Open App Center
    Search for "VLC"
    Click Install

Or through the terminal:

sudo apt install vlc

Or as a Flatpak from Flathub:

flatpak install flathub org.videolan.VLC

Why VLC Is Better Than System Players
Feature	Showtime (Default)	VLC
Format support	Depends on installed codecs	Self-contained — plays everything
Subtitle handling	Basic	Extensive (load external subs, sync timing)
Network streaming	No	Yes (open network streams, IPTV)
Playlist management	No	Yes
Screen recording	No	Limited
DVD playback	Requires extra setup	Built in
Audio equalizer	No	Yes (10-band)
Playback speed	0.25x–2x	0.25x–4x
Resource usage	Light	Slightly heavier

This isn't to say Showtime is bad — it's great for casual viewing. But VLC eliminates the "will this play?" anxiety entirely. Many users keep both: Showtime as the default player for double-clicked files, VLC for anything that doesn't work or needs advanced features.
Setting VLC as Default

If you want VLC to handle all your video files by default:

    Right-click any video file (e.g., an .mp4)
    Select "Properties"
    Go to the "Open With" tab
    Select VLC Media Player
    Click "Set as Default"

Now all video files open in VLC automatically.
DVD Playback

Playing commercial DVDs requires a special library called libdvdcss that handles CSS (Content Scramble System) decryption. This library isn't included in ubuntu-restricted-extras because CSS decryption has different legal implications.
Installing libdvdcss

The simplest method:

sudo apt install libdvd-pkg
sudo dpkg-reconfigure libdvd-pkg

This downloads and builds the libdvdcss library. During installation:

    A blue/purple configuration screen appears
    Choose "Yes" to enable automatic building
    Press Tab to highlight OK, press Enter
    The package downloads source code and compiles the library

After this, VLC (and other players with DVD support) can play commercial DVDs.

    ⚠️ Legal Note

    libdvdcss circumvents DVD copy protection. In some jurisdictions, this may violate laws like the DMCA in the United States. Canonical does not ship it for this reason. Installing it is a personal decision — consult your local laws if concerned.

Streaming Media (Netflix, YouTube, Spotify, Amazon Prime)

Streaming services in your browser work differently from local file playback. Here's what to expect:
Browser-Based Streaming (Netflix, YouTube, Amazon Prime)

Browser streaming uses DRM (Digital Rights Management) and HTML5 video codecs. The good news: Firefox on Ubuntu 26.04 handles this automatically — you don't need to install anything extra.

Firefox (which ships as a Snap on Ubuntu) includes:

    Widevine DRM support (Netflix, Amazon Prime, Disney+)
    VP9 decoding for YouTube
    AV1 decoding (software mode in the Snap version; hardware acceleration may not work for AV1 in the Snap)

When you visit Netflix or Prime Video for the first time:

    Firefox detects the site needs DRM
    A prompt appears asking to enable DRM
    Click Enable
    The Widevine plugin downloads automatically
    Refresh the page and start watching

    💡 Firefox Snap and AV1

    The Snap version of Firefox on Ubuntu 26.04 has a known limitation: AV1 hardware decoding doesn't work (it falls back to software decoding, which uses more CPU). This affects YouTube and other sites that serve AV1 streams. If this causes performance issues, the Flatpak version of Firefox supports AV1 hardware acceleration on Intel/AMD GPUs.

Spotify

Spotify is available as a Snap package:

    Open App Center
    Search for "Spotify"
    Click Install

Or via terminal:

sudo snap install spotify

Spotify on Linux is fully featured — same as the Windows and macOS versions. It supports local file playback, playlists, podcasts, and offline downloads.
Other Streaming Services
Service	How to Access	Notes
Netflix	Browser (Firefox)	Requires Widevine DRM
Amazon Prime Video	Browser (Firefox)	Requires Widevine DRM
Disney+	Browser (Firefox)	Requires Widevine DRM
YouTube	Browser (Firefox) or FreeTube Flatpak	FreeTube is a privacy-focused alternative
Spotify	Snap or browser	Desktop app recommended
Tidal	Browser or Tidal HiFi Flatpak	Community-maintained
SoundCloud	Browser	Works in any browser
Twitch	Browser or Streamlink Twitch GUI Flatpak	GUI for low-resource Twitch viewing
Hardware Acceleration (Optional but Helpful)

Hardware acceleration means letting your GPU (graphics processor) handle video decoding instead of your CPU. This reduces power consumption, lowers temperatures, and enables smoother playback of high-resolution video (especially 4K).
Checking If Hardware Acceleration Is Working

Install a diagnostic tool:

sudo apt install vainfo

Then run:

vainfo

This outputs a list of codecs your GPU can decode in hardware. Look for entries like:

    VAProfileH264Decode — H.264 hardware decoding
    VAProfileHEVCMainDecode — H.265/HEVC hardware decoding
    VAProfileAV1Profile0 — AV1 hardware decoding

If you see these, your system supports hardware acceleration. If vainfo returns errors, you may need to install GPU-specific driver packages:
GPU Manufacturer	Driver Package	Notes
Intel (6th gen+)	intel-media-va-driver-non-free or intel-media-va-driver	Non-free version has broader codec support
AMD	mesa-va-drivers	Usually pre-installed with amdgpu driver
NVIDIA	Proprietary NVIDIA driver (from Software & Updates → Additional Drivers)	NVDEC used for hardware decoding
Enabling Hardware Acceleration in Firefox

By default, Firefox on Ubuntu may not use hardware acceleration. To enable:

    Open Firefox
    Type about:config in the address bar
    Accept the risk warning
    Search for media.ffmpeg.vaapi.enabled
    Set it to true
    Restart Firefox

This enables VA-API hardware decoding for H.264, H.265, and VP9 content on supported hardware.

    ⚠️ Firefox Snap Limitations

    Due to Snap sandboxing, the Snap version of Firefox on Ubuntu 26.04 has limited access to hardware acceleration. AV1 hardware decoding in particular doesn't work in the Snap. If you need full hardware acceleration, consider installing Firefox as a Flatpak (which has better GPU access) or as a .deb from Mozilla's PPA.

Converting Media Formats (Bonus)

Sometimes you need to convert between formats — maybe a file won't play, or you need a smaller version. FFmpeg is the tool for this:
Installing FFmpeg

sudo apt install ffmpeg

Common Conversions

Convert AVI to MP4:

ffmpeg -i input.avi -c:v libx264 -c:a aac output.mp4

Extract audio from a video:

ffmpeg -i video.mp4 -vn -c:a copy audio.aac

Convert MP4 to WebM (for web use):

ffmpeg -i input.mp4 -c:v libvpx-vp9 -c:a libopus output.webm

We cover FFmpeg in more depth in Chapter 25 (Advanced Tools and Automation). For now, know that it exists and is incredibly powerful for media manipulation.
Recommended Media Applications

Beyond the defaults, here are applications worth installing for a complete multimedia setup:
Video
App	Purpose	Install Method
VLC	Plays everything, streaming, conversion	App Center / APT / Flatpak
mpv	Lightweight, minimal, powerful	sudo apt install mpv
SMPlayer	GUI frontend for mpv/mplayer	sudo apt install smplayer
HandBrake	Video transcoding/conversion	Flathub
Audio
App	Purpose	Install Method
Audacity	Audio recording and editing	App Center / APT / Flathub
EasyTAG	Edit MP3/FLAC metadata tags	sudo apt install easytag
SoundConverter	Batch convert audio formats	sudo apt install soundconverter
Elisa	Modern KDE music player	sudo apt install elisa
Lollypop	GNOME-focused music player	Flathub
Media Creation
App	Purpose	Install Method
OBS Studio	Screen recording and live streaming	Flathub
Kdenlive	Video editing	Flathub / APT
Blender	3D modeling, animation, video editing	Flathub / Snap
GIMP	Image editing	App Center / Flathub
Inkscape	Vector graphics	Flathub
Troubleshooting Common Media Issues
"The Video Plays but There's No Audio"

The video codec is present but the audio codec is missing. Usually an AAC or AC-3 codec issue:

sudo apt install gstreamer1.0-libav gstreamer1.0-plugins-ugly

Then restart your media player. If using VLC, this shouldn't happen — VLC includes its own codecs.
"MKV Files Won't Play in Showtime"

Matroska (.mkv) is a container format, not a codec — it wraps video and audio streams that could use various codecs. Showtime may not find the right codec for streams inside the container. Fixes:

    Install codecs: sudo apt install ubuntu-restricted-extras
    Install VLC: sudo apt install vlc — set it as default for .mkv files
    Install GStreamer plugins: sudo apt install gstreamer1.0-plugins-bad gstreamer1.0-plugins-ugly gstreamer1.0-libav

"DVD Won't Play"

Commercial DVDs require CSS decryption:

sudo apt install libdvd-pkg
sudo dpkg-reconfigure libdvd-pkg

Then use VLC to play the DVD.
"Streaming Site Says 'Protected Content' or Shows Black Screen"

DRM isn't enabled in Firefox:

    Go to Firefox → Settings → General
    Scroll to Digital Rights Management (DRM) Content
    Check Play DRM-controlled content
    Restart Firefox and reload the page

"Audio Crackling or Distorted"

Could be a sample rate mismatch. Try:

    Open Settings → Sound
    Check output device is correct (not HDMI when you want speakers)
    If still crackling, try changing the audio server configuration:

sudo apt install pavucontrol

Open PulseAudio Volume Control → Configuration tab → Try different profiles for your sound card (e.g., "Analog Stereo Duplex" instead of "Pro Audio").
"Bluetooth Audio Lag"

Bluetooth audio can have noticeable delay, especially with video:

    Install pavucontrol (above)
    Open it → Recording tab
    Find your browser or media player
    Change the capture device from "Monitor of Bluetooth" to "Monitor of Built-in Audio"

This forces the system to use local audio timing instead of Bluetooth timing, reducing desync.
What's Next

You now have a system that plays virtually every common audio and video format — local files, DVDs, streaming services, and web media. Your media players are equipped with the codecs they need, and you know how to troubleshoot when something doesn't work.

In Chapter 12, we'll map out software equivalents — helping you find Linux replacements for every Windows and macOS application you used before switching.
Try It Yourself

    Install codecs — If you haven't already, install ubuntu-restricted-extras. This is the single most impactful step for multimedia.

    Install VLC — Get it from the App Center or terminal. Try playing a video file that previously didn't work.

    Test a streaming service — Open Netflix, YouTube, or Amazon Prime in Firefox. Verify that DRM-protected content plays correctly.

    Check hardware acceleration — Install vainfo and run it. See which codecs your GPU can decode in hardware.

    Play a DVD (if you have a disc drive) — Install libdvd-pkg, then try playing a DVD in VLC.

    Install one creative app — Pick something from the media creation table (HandBrake, Audacity, or OBS Studio). Explore what it can do.

