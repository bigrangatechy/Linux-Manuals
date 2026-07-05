# Chapter 11: Multimedia and Codecs

## The Codec Question

If you've just installed Fedora and tried to play an MP4 video, an MP3 audio file, or a YouTube video at 4K, you may have noticed something: it doesn't work. Or it works, but your CPU is at 80% and your laptop fan is screaming. Maybe GNOME Videos refuses to open a file entirely. Maybe a website tells you the video format isn't supported.

This is Fedora's most infamous pain point — and it's entirely intentional. This chapter explains why, and then shows you exactly how to fix it. By the end, every media format will play smoothly, with hardware acceleration offloading the heavy lifting from your CPU to your GPU.

---

## What Is a Codec?

A **codec** (coder-decoder) is a piece of software that compresses and decompresses digital media. Video and audio files are enormous in their raw form — a single minute of uncompressed 1080p video is roughly 4 GB. Codecs shrink this data using clever mathematical algorithms, then decompress it on playback.

Common codecs you'll encounter:

| Codec | Type | Used For | Patent/License Status |
|---|---|---|---|
| **H.264 (AVC)** | Video | Most MP4 files, YouTube 1080p, Blu-ray | Patented — requires licensing |
| **H.265 (HEVC)** | Video | 4K video, modern streaming | Patented — requires licensing |
| **VP9** | Video | YouTube 4K, royalty-free | Royalty-free |
| **AV1** | Video | Newest codec — YouTube, Netflix | Royalty-free (but complex) |
| **AAC** | Audio | Audio in MP4 files, iTunes, streaming | Patented |
| **MP3 (MPEG-2 Audio Layer III)** | Audio | The ubiquitous music format | Patented (patents now expired in most jurisdictions) |
| **Opus** | Audio | VoIP, modern streaming, Discord | Royalty-free |
| **FLAC** | Audio | Lossless audio | Royalty-free |
| **Vorbis** | Audio | Open audio format (Ogg) | Royalty-free |

The key distinction: **royalty-free codecs** (VP9, AV1, Opus, Vorbis, FLAC) can be freely distributed by anyone. **Patented codecs** (H.264, H.265, AAC) require licensing fees — typically paid per unit shipped or through patent pools like MPEG-LA.

---

## Why Fedora Doesn't Ship Codecs

Fedora's stance is principled: it ships only software that is unambiguously free and open source, with no patent or licensing entanglements. This isn't a technical limitation — it's a legal and philosophical position.

The United States has software patents. H.264, H.265, and AAC are covered by patents that require licensing fees. If Fedora (distributed by Red Hat, a US company) shipped these codecs, someone would have to pay those fees. Fedora is a free, community-supported distribution — there's no revenue stream to fund patent licensing.

So Fedora ships **without patented codecs**. The result:

| Media Format | Out of the Box | After This Chapter |
|---|---|---|
| H.264 video (MP4) | ❌ Won't play | ✅ Plays with hardware acceleration |
| H.265/HEVC video | ❌ Won't play | ✅ Plays with hardware acceleration |
| AAC audio | ❌ Won't play | ✅ Plays |
| MP3 audio | ❌ Limited/no playback | ✅ Plays |
| VP9 video (YouTube) | ✅ Plays (software decode) | ✅ Plays with hardware acceleration |
| AV1 video | ✅ Plays (software decode on most hardware) | ✅ Plays with hardware acceleration (if supported) |
| Opus audio | ✅ Plays | ✅ Plays |
| FLAC audio | ✅ Plays | ✅ Plays |

> 💡 **Fedora's ffmpeg-free**
>
> Fedora ships a package called `ffmpeg-free` — a version of the powerful FFmpeg multimedia framework with all patent-encumbered codec code **physically stripped out**. The H.264, H.265, and AAC decoders are not merely disabled — they're compiled out of the binary. This is the nuclear option: no ambiguity, no gray area.
>
> The solution is to **replace** `ffmpeg-free` with the full `ffmpeg` from RPM Fusion, which includes all codecs. Fedora explicitly supports this — RPM Fusion exists precisely for this purpose.

---

## The Multimedia Stack: How Linux Plays Media

Before installing codecs, it helps to understand the layers involved in media playback on Fedora:

┌─────────────────────────────────────────────────┐ │ MEDIA PLAYER (VLC, MPV, GNOME Videos, Firefox) │ ├─────────────────────────────────────────────────┤ │ MULTIMEDIA FRAMEWORK │ │ ├── GStreamer (used by GNOME apps) │ │ └── FFmpeg / libav (used by VLC, MPV, Firefox) │ ├─────────────────────────────────────────────────┤ │ CODECS (decoders and encoders) │ │ ├── H.264, H.265, AAC (from RPM Fusion) │ │ ├── VP9, AV1, Opus (from Fedora) │ │ └── GStreamer plugins (base/good/bad/ugly) │ ├─────────────────────────────────────────────────┤ │ HARDWARE ACCELERATION (VA-API / NVDEC) │ │ └── GPU-specific drivers (Intel/AMD/NVIDIA) │ ├─────────────────────────────────────────────────┤ │ AUDIO SERVER (PipeWire + WirePlumber) │ │ └── Routes decoded audio to speakers/headphones│ ├─────────────────────────────────────────────────┤ │ KERNEL (ALSA + DRM/KMS) │ │ └── Talks to physical hardware │ └─────────────────────────────────────────────────┘

Each layer matters:

    Media player — the application you interact with
    Framework — GStreamer (for GNOME apps) or FFmpeg/libav (for VLC, MPV, Firefox)
    Codecs — the actual decode/encode algorithms
    Hardware acceleration — offloads decoding to the GPU
    Audio server — PipeWire handles all audio routing
    Kernel — ALSA for audio hardware, DRM/KMS for GPU

To get everything working, you need all layers populated. Let's go through them.
Step 1: Replace ffmpeg-free with Full FFmpeg

This is the single most impactful change. The ffmpeg-free package (or its libraries) is used by virtually every media application on Fedora. Replacing it with the full version from RPM Fusion unlocks H.264, H.265, AAC, and dozens of other codec support across the entire system.

sudo dnf swap ffmpeg-free ffmpeg --allowerasing

The --allowerasing flag is necessary because ffmpeg and ffmpeg-free conflict — they provide the same files. DNF5 swaps one for the other.

If the swap reports that ffmpeg doesn't exist, the package name in RPM Fusion may be ffmpeg — ensure RPM Fusion is enabled (Chapter 7, Task 3):

sudo dnf install \
  https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

Then retry the swap.
Also Install libavcodec-freeworld

The libavcodec-freeworld package from RPM Fusion provides additional codec libraries that the Fedora-shipped libavcodec-free deliberately omits:

sudo dnf install libavcodec-freeworld --allowerasing

Again, --allowerasing replaces the stripped version with the full one.
Step 2: Install GStreamer Plugins

GStreamer is the multimedia framework used by GNOME applications — GNOME Videos, GNOME Music, Totem, Cheese, and many third-party GTK apps. GStreamer uses a plugin architecture: the base framework handles playback, but individual codecs are installed as separate plugins.

GStreamer plugins have colorful names that reflect their origin:
Plugin Package	What It Provides	Reputation
gstreamer1-plugins-base	Core formats: Ogg, Vorbis, Theora	"Base" — universally free
gstreamer1-plugins-good	Good-quality, free codecs: FLAC, WAV, JPEG	"Good" — patent-free, high quality
gstreamer1-plugins-bad-free	Less common free codecs	"Bad" — not bad quality, just less tested/license-unclear
gstreamer1-plugins-ugly-free	MP3 and other patent-encumbered codecs	"Ugly" — works fine, but licensing is messy
gstreamer1-libav	FFmpeg-based GStreamer plugin	Bridges FFmpeg codecs into GStreamer

The -free suffix on the "bad" and "ugly" packages means Fedora's version with patent-encumbered parts removed. RPM Fusion provides the full versions:

sudo dnf install \
  gstreamer1-plugins-base \
  gstreamer1-plugins-good \
  gstreamer1-plugins-bad-free \
  gstreamer1-plugins-bad-free-extras \
  gstreamer1-plugins-ugly-free \
  gstreamer1-libav \
  gstreamer1-plugins-ugly \
  gstreamer1-plugins-bad \
  gstreamer1-plugins-good-extras

The packages without -free suffix come from RPM Fusion and include the patented codecs. If RPM Fusion isn't enabled, the last few packages won't be found — enable it first.

    💡 Why the Weird Names?

    GStreamer plugins were named by developers with a sense of humor:

        "Good" = patent-free, well-tested, high quality
        "Bad" = less common, possibly patent-encumbered, still functional
        "Ugly" = definitely patent-encumbered (like MP3), but widely needed

    The names don't reflect quality — they reflect licensing clarity. "Ugly" MP3 playback works perfectly; it's just legally awkward in the US.

Step 3: Install the Multimedia Group

Fedora's multimedia package group bundles the essential media-related packages:

sudo dnf groupinstall multimedia --with-optional

This installs:

    GStreamer plugin sets (both free and RPM Fusion versions)
    ffmpeg-libs and ffmpeg with full codec support
    Audio extraction and conversion tools
    Video transcoding utilities

The --with-optional flag adds optional packages from the group that aren't strictly required but are commonly useful.
Also Install the Sound & Video Group

sudo dnf groupinstall sound-and-video

This adds complementary media packages — format conversion tools, CD/DVD utilities, and additional media-related applications.
Step 4: Enable Cisco OpenH264 for Firefox

Firefox uses its own codec system separate from GStreamer. For H.264 playback in Firefox specifically, Cisco provides an open H.264 implementation that's free to use — Cisco paid the patent fees for everyone.

Fedora includes a repository for this, but it's disabled by default:

sudo dnf config-manager --setopt=fedora-cisco-openh264.enabled=1
sudo dnf install openh264

After installation, Firefox can play H.264 video natively (YouTube 1080p, embedded web video, video calls in browsers).

    💡 This Is Separate from FFmpeg

    Firefox uses its own media pipeline (including the Cisco OpenH264 codec). It doesn't directly use the system FFmpeg. Installing the full ffmpeg from RPM Fusion helps GStreamer-based apps (GNOME Videos, Totem), while the openh264 package specifically enables H.264 in Firefox.

    For hardware-accelerated video decoding in Firefox, see Step 6 below.

Step 5: Verify Codec Installation

After installing everything above, verify that codecs are working:
Test with FFmpeg

ffmpeg -codecs | grep -E "h264|hevc|aac|mp3"

You should see lines like:

DEV.LS h264                 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (encoders: libx264 )
DEV.LS hevc                 H.265 / HEVC (High Efficiency Video Coding) (encoders: libx265 )
DEAIL aac                  AAC (Advanced Audio Coding)

The D means decoding is supported. The E means encoding is supported. If you see these, your codecs are installed correctly.
Test with gst-inspect

gst-inspect-1.0 | grep -E "264|hevc|aac|mp3"

This lists GStreamer plugins. If you see entries for H.264, HEVC, AAC, and MP3, GStreamer has the codecs available.
Play a File

Download or locate a test video file (an MP4 with H.264 video and AAC audio) and try playing it in GNOME Videos:

gnome-videos /path/to/test.mp4

If it plays, your codec installation is complete.
Step 6: Hardware-Accelerated Video Decoding (VA-API)

Playing 4K video on a CPU alone is demanding. Your GPU has dedicated hardware for decoding video — using it reduces CPU usage from 80% to 5-10%, extends laptop battery life, and enables smooth 4K/8K playback.

VA-API (Video Acceleration API) is the standard interface for hardware-accelerated video decoding on Linux. It supports Intel, AMD, and NVIDIA GPUs.
Check Your GPU

lspci | grep -iE "vga|3d|display"

Output examples:

# Intel:
00:02.0 VGA compatible controller: Intel Corporation Iris Xe Graphics

# AMD:
01:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Radeon RX 6700 XT

# NVIDIA:
01:00.0 VGA compatible controller: NVIDIA Corporation GA106 [GeForce RTX 3060]

Install VA-API Utilities and Drivers

sudo dnf install libva-utils libva libva-intel-driver intel-media-driver mesa-va-drivers-freeworld

Package	For
libva-utils	Provides the vainfo command to verify hardware acceleration
libva	The VA-API library
libva-intel-driver	VA-API driver for older Intel GPUs (pre-Broadwell, i965)
intel-media-driver	VA-API driver for newer Intel GPUs (Broadwell+, Gen 8+)
mesa-va-drivers-freeworld	VA-API driver for AMD GPUs (from RPM Fusion — includes full codec support)

For AMD specifically, also install:

sudo dnf install mesa-va-drivers-freeworld mesa-vdpau-drivers-freeworld --allowerasing

The --allowerasing flag replaces Fedora's stripped Mesa drivers with RPM Fusion's full versions that include hardware decoding support.
Verify VA-API

vainfo

Successful output (Intel example):

libva info: VA-API version 1.22.0
libva info: Trying open /usr/lib64/dri/iHD_drv_video.so
libva info: Found init function __vaDriverInit_1_22
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.22 (libva 2.22.0)
vainfo: Driver version: Intel iHD driver for Intel(R) Gen Graphics - 25.1

vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264Main10             : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileVP9Profile0           : VAEntrypointVLD
      VAProfileVP9Profile2           : VAEntrypointVLD
      VAProfileAV1Profile0            : VAEntrypointVLD

If you see VAEntrypointVLD (Video Level Decode) next to H.264, HEVC, VP9, and AV1 profiles, hardware decoding is working. If vainfo returns an error or shows no profiles, your driver isn't loaded — see Troubleshooting below.
NVIDIA-Specific VA-API

NVIDIA requires a special driver package to expose VA-API (NVIDIA uses its own NVDEC API natively, but many Linux apps expect VA-API):

sudo dnf install nvidia-vaapi-driver

This package (from RPM Fusion nonfree) wraps NVIDIA's NVDEC hardware decoder behind the VA-API interface, so standard Linux media players can use it.

After installation, verify:

vainfo

You should see profiles with the NVIDIA driver loaded.
MPV: The Best VA-API Player

MPV is a lightweight, powerful video player that supports VA-API out of the box:

sudo dnf install mpv

Create a configuration file for hardware acceleration:

mkdir -p ~/.config/mpv
cat > ~/.config/mpv/mpv.conf << 'EOF'
# Hardware acceleration
hwdec=vaapi

# High quality video scaling
profile=gpu-hq

# Remember playback position
save-position-on-quit=yes

# Default volume
volume=80
EOF

Now play any video file:

mpv /path/to/video.mkv

MPV uses VA-API hardware decoding automatically. Monitor your CPU usage during playback:

# In another terminal, watch CPU usage
top -o %CPU

If CPU usage during 1080p playback is under 10%, hardware acceleration is working. Without it, CPU usage would be 40-80%.
VLC: Use the Flatpak Version

VLC is the world's most popular media player, but the RPM version on Fedora doesn't support VA-API — it's built against a version of FFmpeg that disables the required APIs.

The Flatpak version of VLC includes working VA-API support:

flatpak install flathub org.videolan.VLC

However, due to Flatpak sandboxing, VLC Flatpak may not automatically detect system VA-API drivers. It needs permission to access the GPU device:

flatpak override --device=all org.videolan.VLC

Or use Flatseal (installed in Chapter 7) to grant device access.

Alternatively, skip VLC and use MPV — it's lighter, faster, and has better hardware acceleration support on Linux.
GNOME Videos (Totem)

GNOME's built-in video player uses GStreamer, so it benefits from the GStreamer plugins installed in Step 2. Hardware acceleration in Totem depends on the gstreamer1-vaapi plugin:

sudo dnf install gstreamer1-vaapi

After installation, GNOME Videos uses hardware decoding for H.264 and H.265 content.
Step 7: Firefox Hardware Acceleration

Firefox deserves its own section because it's the primary way most people consume video (YouTube, Netflix, Twitch). Getting hardware acceleration working in Firefox reduces CPU usage during video playback and extends battery life.
Enable VA-API in Firefox

    Open Firefox
    Type about:config in the address bar
    Accept the risk warning
    Search for and set these preferences:

Preference	Value	Purpose
media.ffmpeg.vaapi.enabled	true	Enable VA-API hardware video decoding
media.hardware-video-decoding.force-enabled	true	Force hardware decoding (helpful on some setups)
gfx.webrender.all	true	Enable WebRender compositor (improves rendering performance)

    Restart Firefox

Verify Firefox Hardware Decoding

    Open a YouTube video at 1080p or 4K
    Right-click the video → "Stats for nerds"
    Look for "Codec" and "Color" information
    In a terminal, monitor GPU usage:

For Intel:

sudo dnf install intel-gpu-tools
sudo intel_gpu_top

For AMD:

sudo dnf install radeontop
sudo radeontop

If you see "Video" or "Video Decode" activity while the YouTube video plays, hardware decoding is active.
Firefox on NVIDIA

NVIDIA's VA-API support through nvidia-vaapi-driver works with Firefox, but requires the nvidia-drm.modeset=1 kernel parameter. Check if it's enabled:

cat /sys/module/nvidia_drm/parameters/modeset

Output should be Y. If it's N, enable it:

sudo grubby --update-kernel=ALL --args='nvidia-drm.modeset=1'
sudo reboot

After reboot, verify Firefox uses VA-API as described above.
PipeWire: The Audio Server

Fedora 44 ships with PipeWire as its audio (and video) server, managed by WirePlumber as the session manager. PipeWire replaced PulseAudio (starting with Fedora 34 in 2021) and is now mature, stable, and superior in every way.
What PipeWire Does

┌─────────────────────────────────────────────────┐
│  APPLICATIONS                                   │
│  (Firefox, Spotify, VLC, Discord, Audacity)    │
├─────────────────────────────────────────────────┤
│  PIPEWIRE                                       │
│  Audio/video routing, mixing, format conversion │
├─────────────────────────────────────────────────┤
│  WIREPLUMBER (Session Manager)                  │
│  Decides which device gets which stream,        │
│  handles device detection, volume control       │
├─────────────────────────────────────────────────┤
│  PIPEWIRE-PULSE (Compatibility Layer)            │
│  Makes PulseAudio apps work with PipeWire       │
├─────────────────────────────────────────────────┤
│  PIPEWIRE-JACK (Compatibility Layer)            │
│  Makes JACK (pro audio) apps work with PipeWire │
├─────────────────────────────────────────────────┤
│  ALSA (Kernel Audio)                            │
│  Talks to the actual sound hardware             │
└─────────────────────────────────────────────────┘

PipeWire handles:

    Audio playback — routing sound from apps to speakers/headphones
    Audio recording — microphone input to recording apps
    Bluetooth audio — A2DP, HSP, HFP profiles
    Video capture — webcam streams, screen sharing
    Low-latency audio — pro audio (replaces JACK)
    Compatibility — PulseAudio and JACK apps work without modification

Verify PipeWire Is Running

systemctl --user status pipewire wireplumber

Both services should show active (running):

● pipewire.service - Multimedia Service
     Loaded: loaded (/usr/lib/systemd/user/pipewire.service; enabled)
     Active: active (running) since ...

● wireplumber.service - Multimedia Service Session Manager
     Loaded: loaded (/usr/lib/systemd/user/wireplumber.service; enabled)
     Active: active (running) since ...

Check Audio Devices

pactl list short sinks

This lists all audio output devices. You should see your speakers, headphones, and any Bluetooth devices.

wpctl status

WirePlumber's status command shows a nicely organized view of all audio and video devices:

Audio
  Sinks:
    * 1. Built-in Audio Analog Stereo [vol: 0.60]
      2. HDMI DisplayPort Audio Stereo [vol: 1.00]
  Sources:
    * 3. Built-in Audio Pro Microphone [vol: 0.85]
Video
  Capture:
    * 4. Integrated Camera [HD]

The * marks the default device. Change defaults with:

# Set default audio output (replace number with your sink ID)
wpctl set-default 1

# Set volume (0 = mute, 1 = 100%, 1.5 = 150%)
wpctl set-volume 1 0.8

Audio in GNOME

GNOME's Settings → Sound panel provides a graphical interface to PipeWire:

    Output device selection (speakers, headphones, HDMI, Bluetooth)
    Input device selection (microphone, headset mic)
    Volume levels for output and input
    Per-application volume (click the volume icon in Quick Settings for per-app controls)

Bluetooth Audio

PipeWire handles Bluetooth audio natively — no special configuration needed:

    Turn on Bluetooth in Quick Settings (or Settings → Bluetooth)
    Put your Bluetooth headphones/speaker in pairing mode
    Click the device name when it appears
    After pairing, audio routes to the Bluetooth device automatically

PipeWire supports high-quality Bluetooth audio codecs:
Codec	Quality	How to Enable
SBC	Standard	Built-in — always available
AAC	Good (Apple ecosystem)	Built-in on Fedora 44
aptX	High	Install pipewire-codec-aptx (if available)
LDAC	Highest (Sony)	Install pipewire-codec-ldac (if available)

# Install additional Bluetooth codec support (if available in repos):
sudo dnf install pipewire-codec-aptx pipewire-codec-ldac

    💡 Which Codec Is Active?

    To see which Bluetooth codec is being used:

    pactl list sinks | grep -i "bluetooth\|codec\|a2dp"

    Look for Active Profile: and Codec: in the output. PipeWire negotiates the best codec both your device and your computer support automatically.

Pro Audio (Low Latency)

For professional audio work (DAWs, live performance, recording studios), PipeWire replaces JACK:

sudo dnf install pipewire-jack-audio-connection-kit

PipeWire can run with very low latency by using the "Pro Audio" profile. In GNOME Settings → Sound, select your audio interface and choose "Pro Audio" as the profile. This gives you raw access to all channels with minimal buffering.

    ⚠️ Pro Audio Caveats

    The "Pro Audio" profile disables hardware mixing and volume control — you get raw device channels with fixed volume. This is desirable for professional use (where you control everything in your DAW) but undesirable for casual listening (no system volume control). Use the "Pro Audio" profile only when working with professional audio software.

Recommended Media Players

With codecs installed and hardware acceleration working, here are the best media players for Fedora 44:
Video
Player	Source	Hardware Accel	Best For
MPV	DNF (RPM)	VA-API (excellent)	Everything — the best Linux video player
VLC (Flatpak)	Flathub	VA-API (Flatpak only)	Familiarity, DVD menus, streaming
GNOME Videos (Totem)	Built-in	VA-API (via gstreamer1-vaapi)	Simple, integrated with GNOME
Celluloid	Flathub or DNF	VA-API (via mpv backend)	GNOME-style GUI for MPV

Celluloid is worth special mention — it's a GNOME-friendly frontend for MPV's powerful engine:

sudo dnf install celluloid

Or from Flathub:

flatpak install flathub io.github.celluloid_player.Celluloid

Audio
Player	Source	Best For
GNOME Music	Built-in	Simple music browsing, local files
Rhythmbox	DNF	Classic music library management
Audacious	DNF	Winamp-like, lightweight
Spotube	Flathub	Spotify client without ads (uses Spotify API)
Spotify	Flathub	Official Spotify client
Elisa	DNF	KDE music player, simple and elegant
Recommended MPV Configuration

For users who want the best video playback experience, here's an expanded MPV configuration:

mkdir -p ~/.config/mpv
cat > ~/.config/mpv/mpv.conf << 'EOF'
# === Hardware Acceleration ===
hwdec=vaapi
hwdec-codecs=h264,hevc,vp9,av1,mpeg2video,vc1

# === Video Quality ===
profile=gpu-hq
scale=ewa_lanczossharp
cscale=ewa_lanczossharp

# === Audio ===
volume=80
volume-max=100
audio-pitch-correction=yes

# === Subtitles ===
sub-auto=fuzzy
sub-font-size=40
sub-border-size=2
sub-color="#FFFFFFFF"
sub-border-color="#FF000000"

# === Usability ===
save-position-on-quit=yes
keep-open=yes
force-window=immediate
cursor-autohide=500
EOF

This configuration enables hardware acceleration for all major codecs, uses high-quality video scaling, improves subtitle readability, remembers where you stopped watching, and keeps the window open after playback finishes.
DVD and Blu-ray Playback
DVDs

DVD playback requires libdvdcss — a library that decrypts CSS (Content Scramble System) protection on commercial DVDs:

sudo dnf install libdvdcss

This package comes from RPM Fusion. After installation, MPV and VLC can play commercial DVDs:

mpv dvd://

Blu-ray

Blu-ray playback on Linux is more complex due to AACS and BD+ copy protection:

sudo dnf install libaacs libbdplus

These libraries handle Blu-ray decryption, but they require keys that must be obtained separately. Blu-ray support on Linux is spotty — not all discs work. For the most reliable experience, rip Blu-rays with MakeMKV (proprietary, free during beta) and play the resulting files with MPV.
Media Conversion with FFmpeg

FFmpeg isn't just for playback — it's the most powerful media conversion tool available. Now that you have the full version installed, you can convert between formats:
Convert Video Format

# Convert MKV to MP4 (H.264 video, AAC audio)
ffmpeg -i input.mkv -c:v libx264 -c:a aac output.mp4

# Convert to H.265 (smaller file, same quality)
ffmpeg -i input.mp4 -c:v libx265 -c:a aac output.mp4

Extract Audio from Video

# Extract audio as MP3
ffmpeg -i video.mp4 -vn -c:a libmp3lame -q:a 2 audio.mp3

# Extract audio as FLAC (lossless)
ffmpeg -i video.mp4 -vn -c:a flac audio.flac

Resize Video

# Scale to 720p (preserve aspect ratio)
ffmpeg -i input.mp4 -vf scale=-1:720 -c:v libx264 -c:a aac output.mp4

Batch Convert

# Convert all WAV files to MP3 in current directory
for f in *.wav; do
    ffmpeg -i "$f" -c:a libmp3lame -q:a 2 "${f%.wav}.mp3"
done

    💡 FFmpeg Is Not a GUI Tool

    FFmpeg is a command-line tool. If you want a graphical interface for media conversion, install HandBrake (from Flathub):

    flatpak install flathub fr.handbrake.ghb

    HandBrake provides a GUI for video transcoding, with presets for common devices and quality targets.

Troubleshooting Media Issues
"Video won't play at all"

Likely cause: Codecs not installed or ffmpeg-free still active.

Fix:

# Verify which ffmpeg is installed:
dnf list installed | grep ffmpeg

# If you see ffmpeg-free, swap it:
sudo dnf swap ffmpeg-free ffmpeg --allowerasing
sudo dnf install libavcodec-freeworld --allowerasing
sudo dnf groupinstall multimedia

"Video plays but stutters / CPU at 100%"

Likely cause: Hardware acceleration not working — the CPU is decoding video in software.

Fix:

# Check if VA-API works:
vainfo

# If vainfo errors, check your driver:
# Intel (newer):
sudo dnf install intel-media-driver
# Intel (older, pre-Broadwell):
sudo dnf install libva-intel-driver
# AMD:
sudo dnf install mesa-va-drivers-freeworld --allowerasing
# NVIDIA:
sudo dnf install nvidia-vaapi-driver

# Test with MPV:
mpv --hwdec=vaapi video.mp4

"vainfo: error: Failed to initialise"

Likely cause: Wrong driver loaded, or user not in the video group.

Fix:

# Check group membership:
id | grep video

# If not in video group, add yourself:
sudo usermod -aG video $USER
# Log out and back in for changes to take effect

# Check which driver is being loaded:
vainfo --display drm

# Try specifying the driver:
LIBVA_DRIVER_NAME=iHD vainfo    # Intel (newer)
LIBVA_DRIVER_NAME=i965 vainfo   # Intel (older)
LIBVA_DRIVER_NAME=radeonsi vainfo  # AMD
LIBVA_DRIVER_NAME=nvidia vainfo  # NVIDIA

"No audio / no sound"

Likely cause: Wrong output device selected, muted, or PipeWire issue.

Fix:

# Check PipeWire is running:
systemctl --user status pipewire wireplumber

# Restart PipeWire:
systemctl --user restart pipewire wireplumber

# List audio outputs:
pactl list short sinks

# Check if anything is muted:
pactl list sinks | grep Mute

# Unmute:
pactl set-sink-mute 0 false
pactl set-sink-volume 0 80%

Also check GNOME Settings → Sound to verify the correct output device is selected.
"Bluetooth audio quality is poor"

Likely cause: Using SBC codec instead of a higher-quality codec.

Fix:

# Install additional codecs:
sudo dnf install pipewire-codec-aptx pipewire-codec-ldac

# Check current codec:
pactl list sinks | grep -i codec

# Restart PipeWire:
systemctl --user restart pipewire wireplumber

Then disconnect and reconnect your Bluetooth device — PipeWire should negotiate the highest-quality codec both devices support.
"YouTube 4K stutters in Firefox"

Likely cause: Hardware video decoding not enabled in Firefox.

Fix:

    Go to about:config in Firefox
    Set media.ffmpeg.vaapi.enabled to true
    Set media.hardware-video-decoding.force-enabled to true
    Restart Firefox
    Verify with intel_gpu_top or radeontop — Video decode should show activity during playback

"Subtitles don't display"

Likely cause: Missing subtitle codec or player configuration.

Fix:

# Install subtitle-related packages:
sudo dnf install gstreamer1-plugins-bad gstreamer1-plugins-ugly

For MPV, subtitles should work automatically. Ensure sub-auto=fuzzy is in your mpv.conf (it's in the recommended config above).
The Complete Codec Setup: All-in-One

If you're setting up a fresh system and want to do everything in one go, here's the complete sequence:

#!/bin/bash
set -euo pipefail

echo "=== Fedora 44 Multimedia Setup ==="

# 1. Swap ffmpeg-free for full ffmpeg
echo "─── Swapping FFmpeg ───"
sudo dnf swap -y ffmpeg-free ffmpeg --allowerasing
sudo dnf install -y libavcodec-freeworld --allowerasing

# 2. Install GStreamer plugins
echo "─── Installing GStreamer Plugins ───"
sudo dnf install -y \
  gstreamer1-plugins-base \
  gstreamer1-plugins-good \
  gstreamer1-plugins-good-extras \
  gstreamer1-plugins-bad-free \
  gstreamer1-plugins-bad-free-extras \
  gstreamer1-plugins-ugly-free \
  gstreamer1-plugins-ugly \
  gstreamer1-plugins-bad \
  gstreamer1-libav \
  gstreamer1-vaapi

# 3. Install multimedia group
echo "─── Installing Multimedia Group ───"
sudo dnf groupinstall -y multimedia --with-optional

# 4. Enable Cisco OpenH264 for Firefox
echo "─── Enabling OpenH264 ───"
sudo dnf config-manager --setopt=fedora-cisco-openh264.enabled=1
sudo dnf install -y openh264

# 5. Install VA-API (hardware acceleration)
echo "─── Installing VA-API Drivers ───"
sudo dnf install -y libva-utils libva
# Intel:
sudo dnf install -y libva-intel-driver intel-media-driver
# AMD:
sudo dnf install -y mesa-va-drivers-freeworld --allowerasing
# NVIDIA (only if NVIDIA GPU present):
# sudo dnf install -y nvidia-vaapi-driver

# 6. Install media players
echo "─── Installing Media Players ───"
sudo dnf install -y mpv celluloid

# 7. DVD playback
echo "─── Installing DVD Support ───"
sudo dnf install -y libdvdcss

# 8. Blu-ray (basic)
echo "─── Installing Blu-ray Support ───"
sudo dnf install -y libaacs libbdplus

echo ""
echo "═══════════════════════════════════"
echo "Multimedia setup complete!"
echo ""
echo "Verify with:"
echo "  ffmpeg -codecs | grep h264"
echo "  vainfo"
echo "  mpv --hwdec=vaapi test.mp4"
echo "═══════════════════════════════════"

Save as ~/fedora-multimedia-setup.sh, make executable, and run:

chmod +x ~/fedora-multimedia-setup.sh
~/fedora-multimedia-setup.sh

    ⚠️ Review Before Running

    This script installs many packages. Read through it, uncomment/comment lines as needed for your hardware (especially the NVIDIA section), and ensure RPM Fusion is already enabled.

What's Next

Your Fedora system can now play every media format — H.264, H.265, AV1, MP3, AAC, FLAC — with hardware acceleration offloading the decode work to your GPU. Your audio is handled by the modern PipeWire/WirePlumber stack, supporting everything from Bluetooth headphones to professional audio interfaces.

In Chapter 12, we'll map your Windows and macOS software to Linux equivalents — helping you find replacements for every application you used on your previous operating system. From office suites to photo editors, from games to system utilities, you'll know exactly what to install and where to find it.
Try It Yourself

    Verify your codecs. Run ffmpeg -codecs | grep -E "h264|hevc|aac" in a terminal. Confirm that H.264, H.265, and AAC show D (decode) support. If they don't, revisit Steps 1–3.

    Test hardware acceleration. Install libva-utils and run vainfo. Confirm that your GPU shows VA-API profiles for H.264 and HEVC. If you see errors, work through the Troubleshooting section.

    Play a video with MPV. Install MPV, create the recommended mpv.conf from this chapter, and play a 1080p or 4K video. Monitor CPU usage with top — it should be under 15% during playback.

    Configure Firefox. Enable VA-API in about:config. Open a 4K YouTube video. Check Stats for nerds (right-click the video). Monitor GPU video decode activity with intel_gpu_top or radeontop.

    Test audio. Play music in your preferred audio player. Plug in headphones — verify audio switches automatically. If you have Bluetooth headphones, pair them and verify audio quality.

    Convert a media file. Use FFmpeg to convert a video file to a different format. Try ffmpeg -i video.mp4 -c:v libx265 -c:a aac smaller.mp4 — compare file sizes. H.265 encoding is slower but produces significantly smaller files.

    Test GStreamer. Run gst-inspect-1.0 | grep -c . to count total GStreamer plugins. A fully configured system should show 200+ plugins. Then try playing a file with gst-launch-1.0 playbin uri=file:///path/to/video.mp4 — if it plays, GStreamer is fully functional.

    ✅ Chapter 11 Complete You understand what codecs are (coder-decoders that compress and decompress media), why Fedora doesn't ship patented codecs (US patent law + free distribution philosophy), the ffmpeg-free vs full ffmpeg swap (replacing stripped codecs with complete ones from RPM Fusion), GStreamer plugins (base/good/bad/ugly naming convention and what each provides), the multimedia package group, Cisco OpenH264 for Firefox, VA-API hardware acceleration (how to install and verify drivers for Intel, AMD, and NVIDIA), PipeWire as Fedora's audio server (replacing PulseAudio, managed by WirePlumber, with PulseAudio and JACK compatibility layers), Bluetooth audio codecs (SBC, AAC, aptX, LDAC), recommended media players (MPV, Celluloid, VLC Flatpak), MPV configuration for optimal playback, DVD and Blu-ray support, FFmpeg for media conversion, and a comprehensive troubleshooting guide for common multimedia issues.

