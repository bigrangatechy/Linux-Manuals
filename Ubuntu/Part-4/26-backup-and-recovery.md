# Chapter 26: Backup and Recovery

## Why Backups Matter

Every Linux user learns this lesson eventually: something will go wrong. A botched update, a mistyped `rm` command, a failing hard drive, a stolen laptop. The question isn't whether disaster will strike — it's whether you'll be prepared when it does.

Backups on Linux fall into two categories:

- **System backups** — The operating system, installed packages, system configuration. You want to restore a working system quickly after a broken update or misconfiguration.
- **Data backups** — Your personal files: documents, photos, projects, dotfiles, application configs. You want these protected against accidental deletion, disk failure, or device loss.

Different tools serve different purposes. This chapter covers the full spectrum: Timeshift for system snapshots, Deja Dup for effortless personal file backups, `rsync` and `tar` for scripted control, and BorgBackup for advanced deduplicated, encrypted archives.

## The 3-2-1 Strategy

Before diving into tools, understand the **3-2-1 backup rule** — a principle that's served IT professionals for decades:

| Rule | Meaning |
|---|---|
| **3** copies of your data | The original plus two backups |
| **2** different storage media | Don't keep both backups on the same drive |
| **1** copy off-site | One backup should be physically elsewhere (external drive stored separately, cloud, remote server)

If your laptop's drive dies, you've lost the original. If your backup drive was sitting right next to the laptop and a power surge fried both, you've lost everything. The off-site copy protects against theft, fire, flood, and localized hardware failure.

## Timeshift: System Snapshots

**Timeshift** creates snapshots of your system files — everything outside your home directory. It's Linux's equivalent of Windows System Restore. If an update breaks your system, a bad driver causes a black screen, or a configuration change leaves your desktop unbootable, Timeshift rolls your system back to a working state in minutes.

### Key Concept: Timeshift Does NOT Back Up Personal Files

By default, Timeshift excludes `/home`. This is by design — system snapshots are for system files, not personal data. Your documents, photos, and browser profiles need a separate backup solution (like Deja Dup, covered next).

You can configure Timeshift to include `/home`, but it's generally discouraged — personal files change frequently, bloat snapshots, and need different backup strategies (incremental, encrypted, off-site).

### Installing Timeshift

sudo apt install timeshift

Open from the app grid (search "Timeshift") or terminal:

sudo timeshift-gtk

Timeshift requires root privileges because it backs up system files.
Choosing a Snapshot Type

On first launch, Timeshift asks you to choose a snapshot backend:
Backend	Requirements	Pros	Cons
RSYNC	Any filesystem (ext4, XFS, etc.)	Works everywhere	Slower, uses more space
BTRFS	Btrfs filesystem	Instant snapshots, zero extra space	Requires btrfs partition

Most Ubuntu 26.04 installations use ext4 by default, so RSYNC is the common choice. If you specifically installed Ubuntu on btrfs, choose BTRFS for near-instant, space-efficient snapshots.
Configuring Timeshift

Step 1: Select Snapshot Location

Choose where snapshots are stored. This should be a separate partition or drive if possible — storing snapshots on the same partition as the system offers no protection if the drive fails.

Step 2: Set Schedule
Schedule	Recommended For
Daily	Most desktop users — keeps 3-5 snapshots
Weekly	Light users or systems that don't change often
Monthly	Servers or rarely modified systems
On Boot	Additional safety before each startup

You can set multiple schedules. A common setup: daily snapshots (keep last 5) plus on-boot snapshots (keep last 3).

Step 3: Define Filter Rules

Timeshift uses include/exclude rules to determine what's captured. By default:

    Included: /, excluding /home, /tmp, /var/tmp, /proc, /sys, /dev
    Excluded: Everything in /home (personal data)

Review the default filters and adjust if needed. For example, you might include /home/john/.config if you want system-relevant dotfiles backed up but exclude everything else under /home.
Creating a Snapshot Manually

From the GUI, click Create. Enter an optional comment:

Before NVIDIA driver update

From the terminal:

sudo timeshift --create --comments "Before NVIDIA driver update" --tags D

Tags: D (daily), W (weekly), M (monthly), O (on-demand). Tags help organize and prune snapshots.
Restoring from a Snapshot

From the GUI:

    Open Timeshift
    Select the snapshot you want to restore
    Click Restore
    Timeshift shows what will change — review and confirm
    The system restores the files and reboots

From the terminal (e.g., from a TTY when the desktop won't load):

sudo timeshift --restore

This presents an interactive list of snapshots. Select one and confirm.

Restoring to a specific device:

sudo timeshift --restore --snapshot '2026-07-04_18-00-01' --target /dev/sda2

Timeshift from a Live USB (Disaster Recovery)

If your system is completely unbootable:

    Boot from your Ubuntu 26.04 installation USB
    Select "Try Ubuntu"
    Install Timeshift in the live environment:

    sudo apt update && sudo apt install timeshift

    Open Timeshift, select the device where snapshots are stored
    Browse and restore snapshots
    Reboot into your restored system

Timeshift CLI Reference
Command	Purpose
sudo timeshift --create --comments "msg"	Create a snapshot
sudo timeshift --list	List all snapshots
sudo timeshift --restore	Restore (interactive)
sudo timeshift --restore --snapshot 'NAME'	Restore specific snapshot
sudo timeshift --delete --snapshot 'NAME'	Delete a snapshot
sudo timeshift --list-devices	Show available snapshot locations

    💡 Snapshot Before Risky Changes

    Get in the habit of creating a Timeshift snapshot before:

        System upgrades (do-release-upgrade)
        Installing proprietary drivers (especially NVIDIA)
        Editing system files in /etc
        Installing kernel updates
        Changing display server or desktop environment

    A 30-second snapshot can save hours of troubleshooting.

Deja Dup: Personal File Backups

While Timeshift protects your system, Deja Dup (also called "Backups" in the app grid) protects your personal files. It's the GNOME-integrated backup tool — a graphical front-end for the duplicity engine.
Installing Deja Dup

sudo apt install deja-dup

Open from the app grid (search "Backups") or:

deja-dup

Configuring Deja Dup

Step 1: Choose Storage Location

Deja Dup supports multiple backup destinations:
Destination	Use Case
Local folder	External USB drive, secondary internal drive
Network share	NAS, SMB/CIFS mount
Google Drive	Cloud backup (requires Google account sign-in)
Nextcloud	Self-hosted cloud

    💡 External Drive Best Practice

    For local backups, use an external USB drive. Connect it, then select it as the backup destination in Deja Dup. When the drive isn't connected, Deja Dup skips scheduled backups and resumes when it's plugged in again.

Step 2: Select Folders to Back Up

By default, Deja Dup backs up your entire home directory (~). You can:

    Add specific folders (e.g., ~/Documents, ~/Projects)
    Exclude folders (e.g., ~/Downloads, ~/.cache, ~/.local/share/Trash)

Exclude large, regenerable, or cache directories to keep backup sizes manageable:
Exclude	Reason
~/.cache	Cached data — regenerated automatically
~/Downloads	Usually regenerable
~/.local/share/Trash	Deleted files
~/snap	Snap application data (large, app-managed)
~/.local/share/containers	Container images (huge, regenerable)

Step 3: Enable Encryption (Optional but Recommended)

Deja Dup can encrypt backups with a password. If your backup drive is stolen, the data is unreadable without the password. Don't lose this password — encrypted backups cannot be recovered without it.

Step 4: Set Schedule
Option	Meaning
Daily	Backs up every day (recommended for active users)
Weekly	Backs up once a week

Deja Dup keeps:

    All backups from the last 7 days
    One backup per week for the previous 3 weeks
    One backup per month for the previous 6 months

Older backups are automatically pruned.
Restoring Files with Deja Dup

Full restore:

    Open Deja Dup → Restore
    Select the backup location
    Choose a date to restore from
    Choose restore destination (original location or a new folder)

Individual file restore (from Files/Nautilus):

    Navigate to the folder containing the file
    Right-click → Restore Missing Files or Restore Previous Versions
    Deja Dup shows available versions — select and restore

This Nautilus integration is one of Deja Dup's best features — you can recover a single accidentally deleted file without a full restore.
Deja Dup from the Terminal

While Deja Dup is primarily a GUI tool, the underlying duplicity engine can be used from the command line:

Manual backup:

deja-dup --backup

Restore:

deja-dup --restore

Restore specific file:

deja-dup --restore FILE

rsync: The Backbone of Backups

rsync is the most versatile file-copying tool on Linux. It's fast (only transfers changed files), reliable, scriptable, and works over networks. Many backup tools — including Timeshift — use rsync under the hood.
Basic rsync Backup

rsync -aAXHv --delete /home/john/Documents/ /mnt/backup/Documents/

Breaking down the flags:
Flag	Meaning
-a	Archive mode: recursive, preserve permissions, ownership, timestamps, symlinks
-A	Preserve ACLs (Access Control Lists)
-X	Preserve extended attributes
-H	Preserve hard links
-v	Verbose output
--delete	Delete files in destination that no longer exist in source

    ⚠️ Trailing Slash Matters!

    In rsync, the trailing slash changes behavior dramatically:

    # Copies the Documents folder itself into destination
    rsync -av /home/john/Documents /mnt/backup/
    # Result: /mnt/backup/Documents/

    # Copies the CONTENTS of Documents into destination
    rsync -av /home/john/Documents/ /mnt/backup/
    # Result: /mnt/backup/ (with files, not a Documents/ subfolder)

    The general rule: trailing slash on source = contents, no trailing slash = the directory itself. For backups, you usually want the trailing slash on both paths to mirror contents exactly.

rsync Over a Network

Back up to a remote server over SSH:

rsync -aAXHvz --delete /home/john/Documents/ user@server.local:/backup/john/

    -z — Compress data during transfer
    SSH handles authentication and encryption

Pull from remote (restore direction):

rsync -aAXHvz user@server.local:/backup/john/Documents/ /home/john/Documents/

Excluding Files

rsync -aAXHv --delete \
    --exclude='.cache/' \
    --exclude='.local/share/Trash/' \
    --exclude='*.tmp' \
    --exclude='node_modules/' \
    /home/john/ /mnt/backup/home-john/

For complex exclusion lists, use a file:

rsync -aAXHv --delete --exclude-from='/home/john/.backup-excludes' /home/john/ /mnt/backup/home-john/

Contents of ~/.backup-excludes:

.cache/
.local/share/Trash/
*.tmp
node_modules/
.snapshots/
Downloads/

Dry Run: See What Would Happen

Before running any rsync backup (especially with --delete), preview the changes:

rsync -aAXHv --delete --dry-run /home/john/Documents/ /mnt/backup/Documents/

--dry-run (or -n) shows what files would be transferred or deleted without actually doing anything. Always dry-run when using --delete for the first time — it prevents accidental data loss.
A Practical rsync Backup Script

#!/bin/bash
set -euo pipefail

# rsync-backup.sh — Mirror home directory to external drive
# Usage: rsync-backup.sh

SOURCE="${HOME}/"
DEST="/mnt/backup/${USER}/"
LOG="${HOME}/.local/log/backup-$(date +%F).log"
EXCLUDES="${HOME}/.backup-excludes"

mkdir -p "$(dirname "$LOG")"

# Verify destination is mounted
if ! mountpoint -q "/mnt/backup"; then
    echo "Error: Backup drive is not mounted" >&2
    exit 1
fi

echo "=== Backup started: $(date) ===" | tee "$LOG"
echo "Source: $SOURCE" | tee -a "$LOG"
echo "Destination: $DEST" | tee -a "$LOG"

# Dry run first (uncomment to preview)
# rsync -aAXHv --delete --dry-run --exclude-from="$EXCLUDES" "$SOURCE" "$DEST"

# Actual backup
rsync -aAXHv --delete \
    --exclude-from="$EXCLUDES" \
    --log-file="$LOG" \
    "$SOURCE" "$DEST"

echo "=== Backup completed: $(date) ===" | tee -a "$LOG"
echo "Files synced. Size: $(du -sh "$DEST" | cut -f1)" | tee -a "$LOG"

Save as ~/bin/rsync-backup.sh, make executable, and schedule with cron:

0 2 * * * /home/john/bin/rsync-backup.sh

Runs nightly at 2 AM.
tar: Archives for Long-Term Storage

tar creates compressed archive files. Unlike rsync (which mirrors a live copy), tar produces a single, self-contained file that's easy to transfer, store, or encrypt.
Creating a tar Archive

tar -czvpf /mnt/backup/home-john-$(date +%F).tar.gz /home/john/Documents/

Flag	Meaning
-c	Create a new archive
-z	Compress with gzip
-v	Verbose (show files being archived)
-p	Preserve permissions
-f	Output to file (must be followed by filename)

Other compression options:
Flag	Compression	Speed	Ratio
-z	gzip	Fast	Medium
-j	bzip2	Slower	Better
-J	xz	Slowest	Best

For backups, gzip is the usual choice — fast enough for daily use with good compression. Use xz for long-term archival where size matters more than speed.
Extracting a tar Archive

tar -xzvpf /mnt/backup/home-john-2026-07-04.tar.gz -C /tmp/restore/

Flag	Meaning
-x	Extract
-C	Change to directory before extracting
Other flags same as above	
Listing Archive Contents (Without Extracting)

tar -tzf /mnt/backup/home-john-2026-07-04.tar.gz | less

-t lists files in the archive. Useful for verifying what's inside before extracting.
Excluding Files from tar

tar -czvpf backup.tar.gz \
    --exclude='*.cache' \
    --exclude='node_modules' \
    --exclude='.Trash' \
    /home/john/

Encrypting a tar Archive

Combine tar with gpg for encrypted backups:

# Create and encrypt in one step
tar -czvp /home/john/Documents/ | gpg -c --cipher-algo AES256 -o backup-$(date +%F).tar.gz.gpg

# Decrypt and extract
gpg -d backup-2026-07-04.tar.gz.gpg | tar -xzvp -C /tmp/restore/

gpg -c uses symmetric encryption (a passphrase). You'll be prompted for a password. Store the password safely — without it, the backup is unrecoverable.
Incremental tar Backups

tar supports incremental backups — snapshot files that record what changed since the last backup:

# Level 0 (full backup)
tar -czvpf /backup/full-$(date +%F).tar.gz -g /backup/snapshot.db /home/john/

# Level 1 (incremental — only changed files)
tar -czvpf /backup/inc-$(date +%F).tar.gz -g /backup/snapshot.db /home/john/

The -g (or --listed-incremental) flag uses a snapshot file to track changes. Each incremental archive contains only files that changed since the previous backup. The snapshot file must be preserved between runs.

To restore: extract the full backup first, then each incremental in order.
BorgBackup: Deduplicated, Encrypted Backups

For users who want professional-grade backups — deduplication, compression, encryption, and pruning in one tool — BorgBackup (or borg) is the gold standard.
Installing Borg

sudo apt install borgbackup

Key Concepts
Concept	Explanation
Repository	A directory storing Borg archives (the backup storage)
Archive	A named snapshot within a repository (like a Timeshift snapshot)
Deduplication	Only stores changed chunks — identical files across backups are stored once
Compression	Automatically compresses data (lz4, zstd, zlib)
Encryption	AES-256-GCM authenticated encryption built in

Because of deduplication, Borg is incredibly space-efficient. If you have 50 GB of data and back it up daily, each backup might only store a few hundred megabytes of changes — not 50 GB each time.
Creating a Repository

# Initialize a repository on an external drive
borg init --encryption=repokey /mnt/backup/borg-repo

You'll be prompted to set a passphrase. This passphrase is required to access backups — store it safely (password manager, physical notebook, etc.).

The --encryption=repokey option stores the encryption key in the repository (protected by the passphrase). If you lose the passphrase, the data is gone.
Creating a Backup (Archive)

borg create --progress --stats \
    /mnt/backup/borg-repo::$(date +%Y-%m-%d-%H%M) \
    /home/john/Documents/ \
    /home/john/Projects/ \
    /home/john/.config/

This creates an archive named 2026-07-04-1430 inside the repository, containing the specified directories.

Useful borg create flags:
Flag	Purpose
--stats	Show transfer statistics
--progress	Progress bar
--list	List files being processed
--exclude PATTERN	Exclude pattern (can repeat)
--compression zstd	Use zstd compression (fast + efficient)
--exclude-from FILE	Read exclude patterns from a file
Listing Archives

borg list /mnt/backup/borg-repo

Output:

2026-07-04-1430    Thu, 2026-07-04 14:30:12 [abcdef123456...]
2026-07-04-1800    Thu, 2026-07-04 18:00:05 [ghijkl789012...]
2026-07-05-0900    Fri, 2026-07-05 09:00:08 [mnopqr345678...]

Listing Files in an Archive

borg list /mnt/backup/borg-repo::2026-07-04-1430

Restoring from an Archive

Mount the archive as a filesystem (browse and copy individual files):

borg mount /mnt/backup/borg-repo::2026-07-04-1430 /mnt/borg-restore

Now browse /mnt/borg-restore in Files and copy whatever you need. Unmount when done:

borg umount /mnt/borg-restore

Extract an entire archive:

cd /tmp/restore
borg extract /mnt/backup/borg-repo::2026-07-04-1430

Pruning Old Archives

Borg's prune command removes old archives based on retention rules:

borg prune --keep-daily 7 --keep-weekly 4 --keep-monthly 6 \
    /mnt/backup/borg-repo

This retains:

    Last 7 daily archives
    Last 4 weekly archives
    Last 6 monthly archives

Older archives beyond these limits are deleted, but thanks to deduplication, the data chunks unique to deleted archives are cleaned up automatically.
A Complete Borg Backup Script

#!/bin/bash
set -euo pipefail

# borg-backup.sh — Automated Borg backup of home directories
# Usage: borg-backup.sh

REPO="/mnt/backup/borg-repo"
ARCHIVE="$(date +%Y-%m-%d-%H%M)"
SOURCE="${HOME}"
EXCLUDES="${HOME}/.backup-excludes"
LOG="${HOME}/.local/log/borg-$(date +%F).log"

export BORG_PASSPHRASE=""  # Set this or use BORG_PASSCOMMAND
# export BORG_PASSCOMMAND="pass show backup/borg-repo"

mkdir -p "$(dirname "$LOG")"

# Verify repository is accessible
if [ ! -d "$REPO" ]; then
    echo "Error: Borg repository not found at $REPO" >&2
    exit 1
fi

echo "=== Borg backup started: $(date) ===" | tee "$LOG"

# Create the archive
borg create \
    --progress \
    --stats \
    --compression zstd \
    --exclude-from "$EXCLUDES" \
    "${REPO}::${ARCHIVE}" \
    "${SOURCE}" 2>&1 | tee -a "$LOG"

# Prune old archives
borg prune \
    --keep-daily 7 \
    --keep-weekly 4 \
    --keep-monthly 6 \
    --keep-yearly 1 \
    "$REPO" 2>&1 | tee -a "$LOG"

# Check repository integrity (optional, runs periodically)
# Run `borg check` weekly, not daily — it's I/O intensive
if [ "$(date +%u)" = "7" ]; then
    echo "Running repository integrity check..." | tee -a "$LOG"
    borg check --verify-data "$REPO" 2>&1 | tee -a "$LOG"
fi

echo "=== Borg backup completed: $(date) ===" | tee -a "$LOG"

Borg Over SSH (Remote Backups)

Borg supports backing up to a remote server:

# Initialize a remote repository
borg init --encryption=repokey user@server:/backups/borg-repo

# Create a remote backup
borg create --progress --stats \
    user@server:/backups/borg-repo::$(date +%Y-%m-%d-%H%M) \
    /home/john/

The remote server needs borg installed, but all encryption and deduplication happen client-side — the server never sees unencrypted data.

    💡 Borg vs. Other Tools
    Feature	Deja Dup	rsync	tar	Borg
    GUI	Yes	No	No	Limited (Vorta)
    Deduplication	Partial	No	No	Yes
    Encryption	Yes	No (needs gpg)	No (needs gpg)	Built-in
    Compression	Yes	Yes (-z)	Yes	Yes
    Remote support	Limited	Yes (SSH)	Manual	Yes (SSH)
    Pruning	Automatic	Manual	Manual	Built-in
    Learning curve	Easy	Medium	Easy	Steep

    Recommendation: Use Deja Dup for simplicity. Use rsync for mirrors. Use Borg when you want the most space-efficient, encrypted, scheduled backups possible.

Clonezilla: Full Disk Imaging

When you need a byte-for-byte copy of an entire disk — for migrating to a new SSD, creating a bare-metal recovery image, or deploying identical systems — Clonezilla is the tool.

Clonezilla is a live CD/USB (not an installed program). You boot from it, and it creates or restores disk images outside of your running system.

Typical workflow:

    Download Clonezilla ISO from clonezilla.org
    Flash to USB (same process as the Ubuntu ISO in Chapter 5)
    Boot from the Clonezilla USB
    Choose "device-image" → save disk to image, or "device-device" → clone disk to disk
    Follow the guided menus

When to use Clonezilla vs. other tools:
Scenario	Tool
Roll back a broken update	Timeshift
Recover deleted personal files	Deja Dup
Mirror files to an external drive	rsync
Migrate entire disk to new hardware	Clonezilla
Bare-metal disaster recovery	Clonezilla + Borg/rsync
Putting It All Together: A Layered Backup Strategy

Here's a practical, complete backup setup for a desktop Ubuntu 26.04 system:
Layer 1: System Snapshots (Timeshift)

    Purpose: Quick rollback for system breakage
    Schedule: Daily + on-boot
    Retention: 5 daily, 3 boot snapshots
    Scope: System files only (excludes /home)

Layer 2: Personal File Backup (Deja Dup or Borg)

    Purpose: Protect personal data, recover deleted files
    Schedule: Daily
    Destination: External USB drive (Layer 2a) + remote/cloud (Layer 2b)
    Scope: Home directory with exclusions (cache, trash, containers)
    Encryption: Enabled

Layer 3: Off-Site Archive (Monthly tar or Borg push)

    Purpose: Protection against local disasters (theft, fire)
    Schedule: Monthly
    Destination: Remote server, cloud storage, or physical drive stored elsewhere
    Scope: Critical irreplaceable files only (documents, photos, projects — not cache)

Verification Schedule

Backups you've never tested are just files — you don't know if they work until you try restoring.
Frequency	Action
Weekly	Browse a Timeshift snapshot, verify recent system files are included
Monthly	Restore one random file from Deja Dup or Borg backup
Quarterly	Do a full test restore to a VM or spare disk
Disaster Recovery Scenarios
Scenario 1: Broken Update (System Won't Boot)

    Boot from Ubuntu live USB
    Install Timeshift in the live session
    Open Timeshift, select the disk with your snapshots
    Restore the most recent working snapshot
    Reboot — system is back to the pre-update state

Scenario 2: Accidental rm -rf on Important Files

    Don't panic. Stop writing to the disk immediately.
    If the files were in a Timeshift snapshot that included that path — restore from Timeshift
    If using Deja Dup — right-click the parent folder in Files → Restore Missing Files
    If using Borg — mount the latest archive and copy files back
    If no backup exists — stop all disk activity and use photorec or extundelete (recovery tools, best-effort, not guaranteed)

Scenario 3: Dead Hard Drive

    Install Ubuntu on a new drive
    Install Deja Dup (or Borg)
    Connect your backup drive
    Restore from the backup — files are back
    System preferences can be reapplied using the customization script from Chapter 25
    Installed software can be reinstalled — see Appendix B for the cheat sheet

Scenario 4: Migration to a New Computer

    Back up home directory with Borg or rsync to an external drive
    On the new computer, install Ubuntu 26.04
    Restore home directory from the backup
    Run your customization script to apply GNOME settings
    Reinstall applications (apt, snap, flatpak — reference your package list)
    Optionally: use Clonezilla to clone the entire old disk if the hardware is similar

Backup and Recovery Cheat Sheet
Task	Command or Tool
Timeshift	
Create snapshot	sudo timeshift --create --comments "message"
List snapshots	sudo timeshift --list
Restore snapshot	sudo timeshift --restore
Delete snapshot	sudo timeshift --delete --snapshot 'NAME'
List devices	sudo timeshift --list-devices
Deja Dup	
Open GUI	deja-dup
Manual backup	deja-dup --backup
Restore files	deja-dup --restore
Restore single file	deja-dup --restore FILE
rsync	
Mirror directory	rsync -aAXHv --delete SOURCE/ DEST/
Dry run	rsync -aAXHv --delete --dry-run SOURCE/ DEST/
Over SSH	rsync -aAXHvz SOURCE/ user@host:DEST/
With excludes	rsync -aAXHv --exclude-from=FILE SOURCE/ DEST/
tar	
Create archive	tar -czvpf ARCHIVE.tar.gz SOURCE/
Extract archive	tar -xzvpf ARCHIVE.tar.gz -C DEST/
List contents	tar -tzf ARCHIVE.tar.gz
Encrypted archive	tar -czv SOURCE/ | gpg -c -o ARCHIVE.gpg
Borg	
Init repository	borg init --encryption=repokey REPO
Create archive	borg create --progress --stats REPO::NAME PATH
List archives	borg list REPO
List archive contents	borg list REPO::NAME
Mount archive	borg mount REPO::NAME /mnt/point
Extract archive	borg extract REPO::NAME
Prune archives	borg prune --keep-daily 7 --keep-weekly 4 REPO
Check repository	borg check --verify-data REPO
What's Next

You now have a layered backup strategy: Timeshift for instant system rollbacks, Deja Dup for set-it-and-forget-it personal file protection, rsync for mirrored backups, tar for portable archives, and Borg for deduplicated, encrypted, space-efficient backups. You've seen disaster recovery scenarios for broken systems, deleted files, dead drives, and migrations — and you know to test your backups regularly.

In Chapter 27, we'll cover security basics — understanding the Linux permission model in practice, UFW firewall configuration, SSH key management, AppArmor, and common-sense security habits for desktop users.
Try It Yourself

    Set up Timeshift. Install it, run the setup wizard, choose RSYNC mode, select a snapshot location, and configure a daily schedule. Create your first snapshot with the comment "Initial setup."

    Configure Deja Dup. Open the Backups app. Set your home directory as the source, an external drive (or a local folder for testing) as the destination, add exclusions for ~/.cache and ~/Downloads, and run your first backup.

    Practice rsync dry runs. Create a test directory with some files: mkdir -p ~/test-backup/{dir1,dir2} && echo "content" > ~/test-backup/dir1/file.txt. Run a dry-run rsync: rsync -aAXHv --dry-run ~/test-backup/ /tmp/backup-test/. Verify the output looks correct, then run it for real.

    Create a tar archive. Archive your ~/Documents folder (or any test folder): tar -czvpf /tmp/documents-backup-$(date +%F).tar.gz ~/Documents/. List the contents with tar -tzf. Extract to /tmp/restore-test/ and compare with the original.

    Install Borg and create a repository. If you have an external drive or spare partition: borg init --encryption=repokey /tmp/borg-test-repo. Create a small backup: borg create /tmp/borg-test-repo::test ~/Documents/. List archives, mount the archive, and browse the restored files.

    Write your backup script. Adapt the rsync or Borg backup script from this chapter for your own directories and backup destination. Save it in ~/bin/. Schedule it with cron (daily at 2 AM: 0 2 * * * /home/john/bin/backup.sh).

    Test a restore. This is the most important step. Delete a non-critical test file, then restore it from your backup. If you can't restore it, your backup is worthless — fix the issue now, not during an emergency.

    Document your backup setup. Write down (in a text file or notebook): what tools you use, where backups are stored, what's included/excluded, the retention schedule, and any passwords/passphrases (stored securely, not in plaintext on the backed-up machine).

    ✅ Chapter 26 Complete You understand the 3-2-1 backup strategy, can use Timeshift for system snapshots (GUI and CLI, including live USB recovery), Deja Dup for personal file backups with encryption and Nautilus integration, rsync for mirrored backups with dry-run safety, tar for portable archives with optional GPG encryption, BorgBackup for deduplicated and encrypted backups with pruning, and Clonezilla for full disk imaging. You have a layered backup strategy covering system rollback, personal data protection, and off-site archives — plus tested disaster recovery procedures. In Chapter 27, we'll secure your system.
