# Chapter 20: Searching and Finding Files — Advanced Techniques

## The Needle in the Haystack

Chapter 13 introduced you to `find` and `grep` — the foundational tools for locating files and searching text. You learned enough to get by: find by name, grep for keywords, pipe results between commands.

Now we go deeper. Much deeper.

Real-world systems contain hundreds of thousands of files. A typical Fedora 44 installation has over 300,000 files across the entire filesystem. Your home directory alone might hold 50,000+. Finding a specific configuration file, tracking down which package owns a particular binary, or searching through gigabytes of source code requires precision tools and efficient strategies.

This chapter covers:
- `find` advanced expressions and operators
- `locate` — the pre-indexed fast alternative
- `grep` with regular expressions
- `ripgrep` (`rg`) — the modern, blazing-fast grep replacement
- `fd` — the modern, user-friendly find replacement
- `which`, `whereis`, `type` — executable discovery
- `updatedb` — building and maintaining the locate database
- Searching inside archives without extracting
- Finding duplicate files
- Finding recently changed system files

By the end, you'll locate anything on your system — by name, content, metadata, ownership, or any combination — within seconds.

---

## Table of Contents

| Topic | Section |
|---|---|
| Advanced find Expressions | 1 |
| locate: Pre-Indexed Speed | 2 |
| grep with Regular Expressions | 3 |
| ripgrep (rg): The Modern Search | 4 |
| fd: The Modern find | 5 |
| Executable Discovery Tools | 6 |
| Searching Inside Archives | 7 |
| Finding Duplicate Files | 8 |
| Tracking System Changes | 9 |

---

## 1. Advanced find Expressions

### find Expression Structure

`find` evaluates a sequence of **tests** and **actions** applied to every file it encounters while traversing a directory tree. Understanding the expression structure unlocks its full power:

```bash
find [starting_point...] [expression]

The expression is a combination of:
Component	Purpose	Examples
Options	Affect overall behavior	-maxdepth, -mindepth, -mount
Tests	Filter files by criteria	-name, -type, -size, -mtime, -user
Actions	Do something with matches	-print, -exec, -delete, -ls
Operators	Combine expressions logically	-a (AND), -o (OR), ! (NOT)

The default operator between expressions is AND — if you chain tests without explicit operators, all tests must pass:

# Find: type is regular file AND name ends in .py AND size > 1KB
find /home -type f -name "*.py" -size +1k

Depth Control

By default, find descends into every subdirectory recursively. Control this:

# Only search the current directory (no subdirectories):
find . -maxdepth 1 -type f

# Search up to 3 levels deep:
find / -maxdepth 3 -name "*.conf"

# Skip the top 2 levels (start searching from level 3 onward):
find /var -mindepth 2 -name "*.log"

# Combine both (search levels 2-4 only):
find /var -mindepth 2 -maxdepth 4 -name "*.log"

    💡 Why Depth Control Matters

    Without -maxdepth, find traverses entire filesystem subtrees. On a large system, find / -name "something" can take minutes. Adding -maxdepth 3 limits traversal and returns results in seconds. Always scope your searches to the narrowest useful directory.

Time-Based Tests
Test	Unit	Meaning
-amin n	Minutes	Accessed n minutes ago
-atime n	Days (24h)	Accessed n*24 hours ago
-mmin n	Minutes	Modified n minutes ago
-mtime n	Days (24h)	Modified n*24 hours ago
-cmin n	Minutes	Status changed n minutes ago
-ctime n	Days (24h)	Status changed n*24 hours ago
-newer file	Reference	Modified more recently than file
-newerXY file	Reference	Compare timestamps flexibly

The numeric value n follows special rules:
Format	Meaning
n (exactly)	Exactly n units ago (rarely useful)
+n	More than n units ago (older than)
-n	Less than n units ago (newer than)

# Modified in the last 30 minutes:
find /var/log -mmin -30 -type f

# Modified more than 30 days ago:
find /home -mtime +30 -type f

# Accessed in the last 24 hours:
find /home -atime -1 -type f

# Changed (metadata) in the last 7 days:
find /etc -ctime -7 -type f

# Modified more recently than a reference file:
touch -d "2026-07-01 00:00" /tmp/reference_date
find /home -newer /tmp/reference_date -type f

# Flexibly: modified more recently than /etc/passwd was changed:
find /home -newermc /etc/passwd

The -newerXY syntax:
XY	Comparison
-newermt "date string"	Modified time vs. parsed date
-newerct "date string"	Change time vs. parsed date
-newerat "date string"	Access time vs. parsed date
-newer file	Modified time vs. file's mtime

# Find files modified after July 1st, 2026:
find /home -newermt "2026-07-01" -type f

# Find files modified between two dates:
find /home -newermt "2026-07-01" ! -newermt "2026-07-05" -type f

Size Tests

| Format | Meaning | |---|---|---| | -size n | Exactly n units | | -size +n | Larger than n units | | -size -n | Smaller than n units |

Units:

    c = bytes, k = KiB, M = MiB, G = GiB

# Larger than 1 GB:
find / -type f -size +1G 2>/dev/null

# Between 100 MB and 1 GB:
find /var -type f -size +100M -size -1G

# Exactly zero bytes (empty files):
find /home -type f -size 0

# Small files under 100 bytes:
find /etc -type f -size -100c

Permission and Ownership Tests

# Owned by specific user:
find / -user john 2>/dev/null

# Owned by a user that no longer exists (orphaned):
find / -nouser 2>/dev/null

# In specific group:
find / -group developers 2>/dev/null

# Group no longer exists:
find / -nogroup 2>/dev/null

# Exact permission match (octal):
find /etc -perm 644 -type f

# Files with ALL specified bits set:
find /usr/bin -perm -u+s    # Has setuid bit
find / -perm -o+w -type f 2>/dev/null  # World-writable files

# Files with ANY of specified bits set:
find / -perm /o+w -type f 2>/dev/null  # World-writable (any combination)

# Files with EXACTLY these permissions:
find /etc -perm 640 -type f

Permission test modes:
Mode	Symbolic	Example
-perm nnn	Exact match	-perm 644 = exactly rw-r--r--
-perm -nnn	All bits set	-perm -004 = has at least o+r
-perm /nnn	Any bit set	-perm /222 = has any write bit
Logical Operators

Combine multiple tests:

# AND (implicit — both must match):
find /home -name "*.py" -size +10k

# Explicit AND:
find /home -name "*.py" -a -size +10k

# OR (either matches):
find /home -name "*.jpg" -o -name "*.png"

# NOT (negation):
find /home -name "*.bak" ! -name "*.bak.old"

# Grouping with parentheses (must be escaped!):
find /home \( -name "*.jpg" -o -name "*.png" \) -size +1M

# Complex query: large images OR large videos:
find /home \( -name "*.jpg" -o -name "*.mp4" \) -size +50M

    ⚠️ Parentheses Need Escaping

    In Bash, parentheses have special meaning (subshells). find needs them for grouping. Escape them with backslashes:

    find . \( -name "*.py" -o -name "*.js" \) -type f

    Or quote the entire expression:

    find . "(" -name "*.py" -o -name "*.js" ")" -type f

Actions: Doing Something With Results
-print (Default)

# Implicit — prints path of every match:
find . -name "*.txt"

-printf (Custom Output Formatting)

Format custom output for scripting:

# Size in bytes followed by path:
find /home -type f -printf "%s %p\n" | sort -rn | head -10
# 12345678 /home/john/videos/movie.mp4
# 8765432 /home/john/photos/raw.cr3

# Last modification time as readable date:
find /var/log -type f -printf "%TY-%Tm-%Td %TH:%TM %p\n" | head -10
# 2026-07-05 09:15 /var/log/messages

# File type, permissions, size, path:
find /etc -type f -printf "%y %m %10s %p\n" | head -5
# f 644       1234 /etc/hostname
# f 644        256 /etc/hosts

# Directory tree with indentation:
find /etc/dnf -printf "%y %p\n" | sed 's|^/etc/dnf/|  |'
# d /etc/dnf/
# d /etc/dnf/repos.d
# f /etc/dnf/dnf.conf

Format codes:
Code	Output
%p	Full path
%f	Filename only
%s	Size in bytes
%y	File type (f, d, l, etc.)
%m	Permissions (octal)
%u	Owner name
%g	Group name
%TY	Modification year (4-digit)
%Tm	Modification month (2-digit)
%Td	Modification day (2-digit)
%TH	Modification hour (24h)
%TM	Modification minute
%TS	Modification second
%TT	Full modification timestamp
%n	Number of hard links
%i	Inode number
-exec and -execdir

# Run a command on each match:
find /home -name "*.py" -exec wc -l {} \;
# {} is replaced with the file path, \; terminates the command

# More efficient: batch multiple files into one command:
find /home -name "*.py" -exec wc -l {} +
# + passes all files at once instead of one at a time

# -execdir runs the command from the MATCHED FILE's directory:
find /home -name "*.gitignore" -execdir pwd \;
# Shows the directory containing each .gitignore

-exec vs -execdir:
Feature	-exec	-execdir
Working directory	Current directory of find	Directory containing matched file
Security	Susceptible to path injection	Safer (relative paths)
Path passed to command	Absolute or relative to find's CWD	Basename only (no path prefix)
-delete and -empty

# Delete all matching files:
find /tmp -name "*.tmp" -type f -delete

# Delete empty directories:
find /home -type d -empty -delete

# CRITICAL: always verify with -print BEFORE -delete:
find /tmp -name "*.tmp" -type f -print    # Check first!
find /tmp -name "*.tmp" -type f -delete   # Then delete

    ⚠️ find -delete Ordering

    When using -delete, find implies -depth (processes directory contents before the directory itself). This is correct — it ensures directories are empty before deletion. But if you mix -delete with other actions, ordering matters. Always test with -print or -exec echo first.

-ls

# Long-format listing (like ls -ls):
find /var/log -name "*.log" -ls

Performance Tips

# Prune directories you don't want to search:
find / -path "/proc" -prune -o -path "/sys" -prune -o -name "*.conf" -print
# Skips /proc and /sys entirely, searches everywhere else

# Use -mount to avoid crossing filesystem boundaries:
find / -mount -name "target" -type f
# Won't descend into mounted filesystems like /home or /tmp if they're
# separate mounts

# Combine pruning with specific searches:
find / -path "/proc" -prune -o -path "/sys" -prune -o -path "/run" -prune -o -type f -name "*.service" -print

    💡 Pruning vs. Excluding

    Using -prune is dramatically faster than searching a directory and then filtering out unwanted paths. -prune tells find "don't even look inside this directory." Without it, find searches everything, then discards unwanted results — wasting time.

    When searching from /, always prune /proc, /sys, and /run — they're virtual filesystems with millions of entries that slow find to a crawl.

2. locate: Pre-Indexed Speed
How locate Works

find searches the live filesystem — every directory, every file, in real time. This is thorough but slow. locate takes a different approach: it searches a pre-built database of all file paths on the system.

The trade-off:
Tool	Speed	Freshness	Coverage
find	Slow (real-time traversal)	Perfectly current	Every file that exists
locate	Instant (database lookup)	As recent as last DB update	Everything indexed at last updatedb

locate is orders of magnitude faster than find for simple name searches:

# find would take 30+ seconds:
find / -name "libssl.so*" 2>/dev/null

# locate returns instantly:
locate libssl.so
# /usr/lib64/libssl.so.3
# /usr/lib64/libssl.so.3.2.2

Installing and Configuring locate

Fedora includes locate in the mlocate or plocate package. plocate is the modern, much faster replacement:

# Install plocate (recommended):
sudo dnf install plocate

# Build the database (first time):
sudo plocate-build /usr/sbin/updatedb
# Or simply:
sudo updatedb

How the database is built:

# Updatedb configuration:
cat /etc/updatedb.conf

PRUNEPATHS = "/afs /media /mnt /proc /srv /sys /var/lib/docker"
PRUNEFS = "tmpfs overlay squashfs"
PRUNEBIND_MOUNTDIRS = "yes"

This tells updatedb to skip virtual filesystems, mount points, and directories with transient content — keeping the database lean and relevant.
Using locate

# Basic search:
locate passwd
# /etc/passwd
# /etc/passwd-
# /usr/share/man/man1/passwd.1.gz

# Limit results:
locate -l 10 "*.conf"
# Shows only first 10 matches

# Case-insensitive:
locate -i README
# Matches: README, readme, Readme, etc.

# Only existing files (verify against live filesystem):
locate -e "*.log"
# Filters out files deleted since last updatedb

# Count matches instead of listing:
locate -c "*.jpg"
# 3421

# Show database statistics:
locate -S
# Database /var/lib/plocate/plocate.db
# 287,435 files
# 89,234 directories
# 5,672,114 entries
# Last updated: 2026-07-05 03:00:00

Keeping the Database Current

Fedora's plocate includes a systemd timer that runs updatedb daily:

# Check the timer:
systemctl status plocate-update.timer
# Active: active (enabled)

# View the schedule:
systemctl list-timers plocate-update.timer
# NEXT   LEFT   LAST             PASSED  UNIT
# Tue 2026-07-06 03:00  16h  ...

The database updates automatically. If you need fresher data:

# Manual refresh:
sudo updatedb

    💡 locate vs. find: When to Use Which

    Use locate when:

        You know the filename (or part of it)
        You need results immediately
        The file has existed for more than 1 day (indexed)

    Use find when:

        You need to search by metadata (size, permissions, owner, time)
        The file was just created (locate hasn't indexed it yet)
        You need to act on results (delete, chmod, etc.)
        You need exact real-time filesystem state

    Rule of thumb: locate for "where is this file?", find for "what files match these criteria?"

3. grep with Regular Expressions
grep Refresher

grep searches text. It prints lines matching a pattern:

grep "error" /var/log/messages
# Prints all lines containing "error"

But grep patterns aren't just plain text — they're regular expressions (regex). A regex is a pattern-matching language that describes text structures, not just literal strings.
Regular Expression Primer

Regular expressions are a compact, powerful language for matching text patterns. Learning them transforms how you search and process text. Here's a focused primer for the regex features grep uses.
Literal Characters

The simplest regex matches exact text:

grep "root" /etc/passwd
# Matches any line containing the literal string "root"

The Dot (.)

. matches any single character:

grep "r.o" /etc/passwd
# Matches: root, r.o, r1o, rxo, r-o, r o, r%o
# Does NOT match: ro (no character between r and o)

Character Classes [...]

Match any one character from the set:

grep "[abc]" file.txt
# Matches lines containing a, b, or c anywhere

grep "[A-Z]" file.txt
# Matches lines containing any uppercase letter

grep "[0-9]" file.txt
# Matches lines containing any digit

grep "[a-zA-Z0-9]" file.txt
# Matches any alphanumeric character

# Negated class [^...]:
grep "[^abc]" file.txt
# Matches lines containing any character that is NOT a, b, or c

Anchors ^ and $

Position assertions — they don't match characters, they match locations:

grep "^root" /etc/passwd
# Matches lines STARTING with "root"

grep "bash$" /etc/passwd
# Matches lines ENDING with "bash"

grep "^$" file.txt
# Matches empty lines (start and end with nothing between)

grep "^Error.*failed$" /var/log/messages
# Lines starting with "Error" and ending with "failed"

Quantifiers * + ? {}

How many times a pattern repeats:

# * (zero or more):
grep "ab*c" file.txt
# Matches: ac, abc, abbc, abbbc, ...
# * is greedy — matches as many as possible

# + (one or more) — requires -E flag or egrep:
grep -E "ab+c" file.txt
# Matches: abc, abbc, abbbc, ...
# Does NOT match: ac (needs at least one b)

# ? (zero or one) — requires -E:
grep -E "colou?r" file.txt
# Matches: color, colour

# {n,m} (counted repetitions) — requires -E:
grep -E "a{3}" file.txt    # Exactly 3 a's: aaa
grep -E "a{2,5}" file.txt  # Between 2 and 5 a's
grep -E "a{2,}" file.txt   # 2 or more a's

    💡 Basic vs. Extended Regular Expressions

    grep uses Basic Regular Expressions (BRE) by default. In BRE, quantifiers like +, ?, and {} need backslash escaping: grep "ab\+c".

    Use grep -E (Extended Regular Expressions — ERE) for unescaped quantifiers: grep -E "ab+c".

    Or use egrep — which is simply grep -E:

    egrep "ab+c" file.txt    # Same as grep -E "ab+c" file.txt

Alternation |

Match one pattern OR another (requires -E):

grep -E "cat|dog" file.txt
# Matches lines containing "cat" or "dog"

grep -E "(cat|dog)s?" file.txt
# Matches: cat, cats, dog, dogs

Grouping (...)

Group parts of a regex together:

grep -E "(error|warn).*disk" /var/log/messages
# Matches lines containing "error" or "warn" followed by "disk"

grep Flags Reference

| Flag | Name | Effect | |---|---|---|---| | -i | Ignore case | Case-insensitive matching | | -v | Invert | Print lines that DON'T match | | -r | Recursive | Search through all files in directories | | -l | Files with matches | Print only filenames, not matching lines | | -L | Files without match | Print filenames that DON'T contain pattern | | -n | Line number | Prefix each match with its line number | | -c | Count | Print count of matches per file | | -o | Only matching | Print only the matched portion, not the whole line | | -w | Whole word | Match whole words only | | -x | Whole line | Match entire line exactly | | -A n | After context | Print n lines after each match | | -B n | Before context | Print n lines before each match | | -C n | Context | Print n lines before and after each match | | -E | Extended regex | Use ERE (+ ? {} () | work unescaped) | | -F | Fixed string | Treat pattern as literal text (no regex) | | -e | Pattern | Specify multiple patterns | | -f | File | Read patterns from a file | | --color | Color | Highlight matches in red | | --include | Include filter | Only search files matching glob | | --exclude | Exclude filter | Skip files matching glob | | --exclude-dir | Exclude directory | Skip directories matching glob | | -z | Null separator | Process null-terminated records |
Practical grep Recipes

# Search recursively through all files, showing line numbers:
grep -rn "TODO" ~/projects/

# Case-insensitive search with context:
grep -inC 3 "warning" /var/log/messages
# Shows 3 lines of context around each warning

# Find all files in /etc containing "PermitRootLogin":
grep -rl "PermitRootLogin" /etc/

# Count matching lines per file:
grep -rc "function" ~/projects/*.py

# Extract only the matched text (not whole lines):
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" /var/log/httpd/access.log
# Extracts IP addresses from a web log

# Multiple patterns (OR):
grep -e "error" -e "warning" -e "critical" /var/log/messages

# Invert — find lines that DON'T contain a pattern:
grep -v "^#" /etc/fstab
# Shows all non-comment lines in fstab

# Invert with exclusion — show config file without comments/blanks:
grep -v "^#" /etc/dnf/dnf.conf | grep -v "^$"

# Search only specific file types:
grep -r --include="*.py" "import" ~/projects/
grep -r --include="*.conf" --include="*.cfg" "password" /etc/

# Exclude directories from recursive search:
grep -r --exclude-dir=".git" --exclude-dir="node_modules" "TODO" ~/projects/

# Use fixed strings (no regex interpretation):
grep -F "price: $9.99" catalog.txt
# $ and . are literal, not regex operators

# Read patterns from a file:
echo "error" > patterns.txt
echo "warning" >> patterns.txt
grep -f patterns.txt /var/log/messages

# Process find output through grep (pipeline):
find /var/log -name "*.log" -exec grep -l "segfault" {} +

# Colorized output for readability:
grep --color=auto "root" /etc/passwd

# Null-separated output for safe processing of weird filenames:
find . -name "*.txt" -print0 | grep -z "search term" | xargs -0 ls -l

4. ripgrep (rg): The Modern Search
What Is ripgrep?

ripgrep (rg) is a modern, Rust-based search tool that outperforms grep on large codebases while providing a superior default experience. It is written in Rust, respects .gitignore rules by default, and includes smart case sensitivity.

Install:

sudo dnf install ripgrep

Why ripgrep Over grep?

| Feature | grep | ripgrep | |---|---|---\n| Speed | Fast | Much faster (parallelized) | | gitignore | Ignored | Respected by default | | Binary files | Says "Binary file matches" | Skips automatically | | Hidden files | Depends on flags | Respects config | | Case sensitivity | Manual -i | Smart (auto-case) | | Default output | Matches only | Matches + filename + line numbers | | Unicode | Limited | Full Unicode support | | Color | Manual | Automatic |
Basic Usage

# Basic search (recursive, smart-case, respects .gitignore):
rg "TODO" ~/projects/
# Shows file:line: matched_text automatically

# Filter by file type:
rg "import" -t py ~/projects/
rg "fn " -t rust ~/projects/
rg "function" -t js ~/projects/

# Case-insensitive:
rg -i "error" /var/log/

# Inverted search (lines NOT matching):
rg -v "^#" /etc/fstab

# Show context:
rg -C 3 "error" ~/projects/
# Three lines before and after each match

# Only show filenames with matches:
rg -l "TODO" ~/projects/
# Just lists files containing "TODO"

# Count matches per file:
rg -c "import" ~/projects/

# Fixed string (no regex):
rg -F "price: $9.99" catalog.txt

ripgrep-Specific Features

# Search only specific file types:
rg -t py "import" ~/projects/

# Search MULTIPLE file types:
rg -t py -t js -t ts "function" ~/projects/

# Exclude file types:
rg -T py "function" ~/projects/  # Search everything except .py

# Search hidden files (hidden = files starting with .):
rg --hidden "password" ~/.config/
# By default rg skips hidden files

# Search ALL files, including gitignored:
rg --no-ignore "secret" ~/projects/

# Regex capture groups (show captured groups in output):
rg "import (\w+)" ~/projects/ -r '$1'
# Shows just the imported module names, not full lines

# Replace (dry run — shows what would change):
rg "old_function" ~/projects/ -r "new_function"
# Shows lines with replacement applied

# Show specific types:
rg --type-list
# Lists all supported file types (py, js, ts, rs, c, cpp, etc.)

# Search files modified recently (using find + rg):
find ~/projects -name "*.py" -mtime -7 -print0 | xargs -0 rg "TODO"

ripgrep Configuration

Configure via environment variables or config file:

# Environment variable approach (add to ~/.bashrc):
export RIPGREP_CONFIG_PATH=~/.config/ripgrep/config

# Create config file:
mkdir -p ~/.config/ripgrep
cat > ~/.config/ripgrep/config << 'EOF'
--smart-case
--heading
--colors=match:fg:yellow
--max-columns=200
EOF

5. fd: The Modern find
What Is fd?

fd is a Rust-based replacement for find. It's faster, has sensible defaults, colorized output, and uses a simpler syntax.

Install:

sudo dnf install fd-find
# On Fedora, the command is 'fd' or 'fdfind' — check with:
which fd || which fdfind

Why fd Over find?
Feature	find	fd
Speed	Sequential	Parallelized
Color	None	Automatic
gitignore	Not respected	Respected by default
Syntax	find . -name "*.py"	fd py (much simpler)
Regex	Basic	Modern (Unicode-aware)
Hidden files	Included by default	Excluded by default
Parallel exec	One at a time	Parallel -x
Regex type	POSIX BRE	Rust regex (similar to PCRE)
Extension search	Not supported	-e py filters by extension
Basic Usage

# Find Python files (default searches by name, case-insensitive):
fd py
# Matches: app.py, test_py, python_config.yml

# Search by extension:
fd -e py ""
# All Python files

# Search by full name:
fd -g "README.md"
# Exact match (glob pattern, case-sensitive)

# Search in a specific directory:
fd log /var/log
# Find files named "log" under /var/log

# Case-sensitive:
fd -s README
# Only matches README, not readme

# Include hidden files:
fd -H .bashrc
# Hidden files excluded by default

# Include ignored files (gitignored, etc.):
fd -I node_modules
# By default fd skips gitignored files

# Limit depth:
fd -d 2 "" /etc
# Search up to 2 levels deep in /etc

# Show file type:
fd -t f "" ~/Documents
# Only regular files
fd -t d "" ~/Documents
# Only directories
fd -t l "" /usr/bin
# Only symlinks

fd with -x (Execute) and -x Batch Operations

# Convert all PNG files to JPEG (parallel execution):
fd -e png -x convert {} {.}.jpg
# {} = file path, {.} = path without extension

# Rename all .txt files to .md:
fd -e txt -x mv {} {.}.md

# Compress all log files:
fd -e log -x gzip {}

# Execute a command on each match (like find -exec):
fd -e jpg -x identify

fd Pattern Syntax

fd uses regex by default (unlike find which uses glob patterns):

# Regex match:
fd "test.*\.py$"
# Matches: test_app.py, test_user.py, test_something.py

# Glob mode (for exact glob patterns like find uses):
fd -g "*.py" ~/projects/
# * and ? work like find's -name

6. Executable Discovery Tools

Several commands help you find and understand executables on your PATH:
which

# Find the path of an executable:
which python3
# /usr/bin/python3

which firefox
# /usr/bin/firefox

# Multiple commands:
which python3 git firefox
# /usr/bin/python3
# /usr/bin/git
# /usr/bin/firefox

which searches your $PATH directories in order and returns the first match. If you have two versions of a program installed, which tells you which one runs by default.
whereis

# Find binary, source, and man page locations:
whereis vim
# vim: /usr/bin/vim /usr/share/man/man1/vim.1.gz
#      ↑ binary     ↑ man page

whereis searches standard locations — not just $PATH — so it finds man pages and source files too.
type

# What is "ls" — alias, function, builtin, or executable?
type ls
# ls is aliased to `ls --color=auto`

type cd
# cd is a shell builtin

type python3
# python3 is /usr/bin/python3

type ll
# ll is aliased to `ls -la'

type grep
# grep is /usr/bin/grep
# grep is /usr/bin/grep
# grep is /usr/bin/gako grep

type reveals what a command ACTUALLY is — useful when an alias shadows the real command or when a shell builtin (like cd) isn't found by which.
command -v

# Portable version of which:
command -v vim
# /usr/bin/vim

This is a shell builtin that works consistently across all POSIX shells. Prefer it in scripts over which.
7. Searching Inside Archives

Sometimes you need to find a file inside a tarball without extracting it. Or search the contents of compressed logs without decompressing them to disk.
Searching tar Archives

# List contents of a tar archive:
tar tf archive.tar.gz | grep "config"

# Search for a specific file inside an archive:
tar tjf archive.tar.bz2 | grep "README"

# Extract a single file from an archive:
tar xzf archive.tar.gz path/to/single/file.txt

Searching Compressed Logs

# Search inside gzip-compressed files:
zgrep "error" /var/log/messages-20260701.gz
# Works just like grep, but on .gz files

# Search inside bzip2-compressed files:
bzgrep "error" /var/log/messages-20260701.bz2

# Search inside xz-compressed files:
xzgrep "error" /modern-logs.xz
# Search inside zip archives:

Searching Inside RPM Packages

# List files in an installed RPM package:
rpm -ql firefox | grep "desktop"
# Lists all files installed by Firefox, filtered to .desktop files

# List files in an RPM file you haven't installed:
rpm -qlp package.rpm
# Preview contents before installation

# Search for a specific file across all installed packages:
rpm -qf /usr/bin/firefox
# Tells you which package installed this file

Searching Inside Flatpaks

# List files inside a Flatpak application:
flatpak run --command=ls org.mozilla.firefox /app
# Uses the Flatpak's namespace to list internal files

# Search for a specific file inside a Flatpak:
flatpak run --command=find org.mozilla.firefox -- /app -name "*.conf"

# Check Flatpak app data on your filesystem:
ls ~/.local/share/flatpak/app/org.mozilla.firefox/


8. Finding Duplicate Files

Duplicate files waste disk space and create confusion. Linux offers several approaches to detecting them.
Method 1: Hash-Based Deduplication

The most reliable method: compute cryptographic hashes of files and compare them. Identical content produces identical hashes.

# Find files of the same size first (fast filter), then hash only those:
find /home -type f -printf "%s %p\n" | sort -n | uniq -D -w 20 | awk '{$1=""; print substr($0,2)}' | xargs -I {} sha256sum {} | awk '{print $1}' | sort | uniq -D -w 64

This pipeline is complex but demonstrates power. Let's break it down step by step:

    find /home -type f -printf "%s %p\n" — List all files with their sizes
    sort -n — Sort numerically by size
    uniq -D -w 20 — Show only lines where the first 20 characters (the size field) are duplicated
    awk '{$1=""; print substr($0,2)}' — Strip the size, leaving just file paths
    xargs -I {} sha256sum {} — Compute SHA-256 hash of each candidate
    awk '{print $1}' | sort | uniq -D -w 64 — Find matching hashes (identical content)

Simplified with a script:

#!/bin/bash
# dupfinder.sh — Find duplicate files by content hash
find "$1" -type f -exec sha256sum {} + | sort | awk '{hash=$1; $1=""; if (seen[hash]++) print}'

Usage:

chmod +x dupfinder.sh
./dupfinder.sh /home/john/Documents/

Method 2: Using fdupes

A dedicated tool for duplicate detection:

sudo dnf install fdupes

# Find duplicate files in a directory:
fdupes /home/john/Downloads/

# Recursively:
fdupes -r /home/john/

# Show sizes:
fdupes -S /home/john/Downloads/
# 4096 bytes each
# /home/john/Downloads/copy1.pdf
# /home/john/Downloads/copy2.pdf

# Delete duplicates (prompts for which to keep):
fdupes -r -d /home/john/Downloads/
# [1] /home/john/Downloads/copy1.pdf
# [2] /home/john/Downloads/copy-basedon-1.pdf
# Set 1 of 5, preserve [1 - 2, all]: 1

Method 3: Using jdupes (Faster)

A modern, faster alternative to fdupes with more options:

sudo dnf install jdupes

# Find duplicates:
jdupes /home/john/Pictures/

# Replace duplicates with hard links (zero-cost deduplication!):
jdupes -L /home/john/Pictures/
# Instead of deleting, creates hard links — same inode, no extra space

# Summarize potential savings:
jdupes -S /home/john/Pictures/

    💡 Btrfs Inline Deduplication

    Fedora 44's default Btrfs filesystem supports deduplication at the filesystem level. Tools like bees or jdupes -L (hardlink) can save space without deleting files. Btrfs also supports reflinks (copy-on-write clones) where duplicate data shares storage automatically:

    # Btrfs reflink copy (shares data blocks, instant, zero extra space):
    cp --reflink=always large_video.mp4 copy_for_editing.mp4
    # Both files exist, but storage consumed only once until one is modified

9. Tracking System Changes
What Changed on My System Recently?

Security-conscious users and troubleshooting scenarios often require knowing what files changed recently:

# Files modified in the last 24 hours anywhere in /etc:
sudo find /etc -type f -mtime -1

# Configuration files changed in the last week:
sudo find /etc -type f -mtime -7 -ls

# Files in home directory accessed in the last hour:
find ~ -type f -amin -60

# Packages installed in the last 7 days (via DNF history):
dnf history | head -20
# Look for recent transactions

Using auditd for File Monitoring

For persistent, kernel-level file access monitoring, Fedora includes the audit subsystem:

# Add a watch on a directory (monitors access):
sudo auditctl -w /etc/passwd -p wa -k identity_changes
# -w = watch path, -p = permissions (write/attribute), -k = key (filter name)

# View audit records for that key:
sudo ausearch -k identity_changes

# Remove the watch:
sudo auditctl -W /etc/audit/audit.rules -k identity_changes

Comparing Snapshots

If you're using Btrfs snapshots (Chapter 14 and 27 cover this), you can compare filesystem states:

# List available snapshots:
sudo btrfs subvolume list /

# If you have snapper configured:
snapper list

# Diff a specific file between current state and a snapshot:
sudo btrfs send /path/to/snapshot/etc/passwd | btrfs receive /tmp/
diff /tmp/passwd /etc/passwd

Tracking Package Changes

# What packages were installed today?
rpm -qa --last | head -20
# Shows packages sorted by install time, newest first

# What packages changed ownership of which files?
rpm -qa --qf '%{NAME} %{INSTALLED:date}\n' | sort -k2 -r | head -20
# Lists packages with installation timestamps

# Verify a package's files haven't been modified:
sudo rpm -V firefox
# Silent output = no modifications detected
# Any output means a file differs from the original package version

# DNF history of specific packages:
dnf history packageinfo firefox

Try It Yourself
Exercise 1: Advanced find Mastery

# Find all Python files in your home directory larger than 5KB:
find ~ -name "*.py" -size +5k -type f

# Find all configuration files modified in the last 7 days:
sudo find /etc -type f -mtime -7 -name "*.conf"

# Find all empty directories under /home:
find ~ -type d -empty

# Find world-writable files in /etc (potential security issue):
sudo find /etc -type f -perm -o+w

# Find files larger than 1GB, sorted by size:
sudo find / -type f -size +1G 2>/dev/null -exec ls -lhS {} + | head -10

# Use pruning to search / only while skipping /proc and /sys:
find / -path "/proc" -prune -o -path "/sys" -prune -o -name "*.service" -print | head -20

Exercise 2: locate Speed Test

# Build the database if not already done:
sudo updatedb

# Race find vs. locate:
time find / -name "libssl.so*" 2>/dev/null
time locate libssl.so
# Note the dramatic speed difference

# Use locate to find all .desktop files:
locate "*.desktop" | head -20

# Find files indexed by name pattern:
locate -i "readme" | head -20

# Verify results are current:
locate -e "*.log" | head -10

Exercise 3: grep Regex Workshop

# Create a test file:
cat > ~/regex_test.txt << 'EOF'
2026-07-05 09:15:23 INFO  Application started on port 8080
2026-07-05 09:15:24 DEBUG Loading configuration from /etc/app.conf
2026-07-05 38
2026-07-05 09:15:26 ERROR Connection refused: 192.168.1.50:3306
2026-07-05 09:15:27 WARN  Retrying connection attempt 1/3
2026-07-05 9:15:28 ERROR Connection refused: 10.0.0.5:5432
EOF

# Extract all IP addresses:
grep -oE "[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+" ~/regex_test.txt

# Find all ERROR lines with context:
grep -nC 2 "ERROR" ~/regex_test.txt

# Find all lines that are NOT comments or blank:
grep -v "^#" /etc/fstab | grep -v "^$"

# Find lines with a specific date pattern (YYYY-MM-DD):
grep -E "^[0-9]{4}-[0-9]{2}-[0-9]{2}" ~/regex_test.txt

# Clean up:
rm ~/regex_test.txt

Exercise 4: ripgrep Practice

# Install ripgrep:
sudo dnf install ripgrep

# Search your home directory for "TODO" comments:
rg "TODO" ~ -t py

# Search /etc for "ENABLED" (case-insensitive):
rg -i "enabled" /etc/ -t conf

# Find all function definitions in Python files:
rg "^def " ~/projects/ -t py --no-heading

# Search for files containing a specific error message:
rg -l "Connection refused" ~/projects/

Exercise 5: fd Practice

# Install fd:
sudo dnf install fd-find

# Find all Python files in your home:
fd -e py "" ~

# Find all configuration files under /etc:
fd -e conf "" /etc

# Find all directories named "test":
fd -t d "^test$" ~/projects/

# Use fd to convert file extensions:
# (Create test files first)
mkdir ~/fd_test && cd ~/fd_test
touch a.txt b.txt c.txt
fd -e txt -x mv {} {.}.md
# All .txt files renamed to .md
ls    # a.md b.md c.md

# Clean up:
rm -rf ~/fd

Exercise 6: Duplicate Detection

# Install jdupes:
sudo dnf install jdupes

# Create test duplicates:
mkdir ~/dup_test
echo "identical content" > ~/dup_test/file1.txt
echo "identical content" > ~/dup_test/file2.txt
echo "different content" > ~/dup_test/file3.txt

# Find duplicates:
jdupes ~/dup_test/

# Replace with hard links (deduplicate without deletion):
jdupes -L ~/dup_test/

# Verify hard linkage:
ls -li ~/dup_test/
# file1.txt and file2.txt should have the same inode number

# Clean up:
rm -rf ~/dup_test

Exercise 7: System Change Tracking

# What config files changed in the last week?
sudo find /etc -type f -mtime -7 -ls

# What packages were recently installed?
rpm -qa --last | head -10

# Verify a core package's integrity:
sudo rpm -V bash
# Silent output = pristine; any output = modification detected

# Check DNF history:
dnf history | head -10

Exercise 8: Searching Archives

# Create a test archive:
mkdir ~/archive_test && cd ~/archive_test
echo "config data" > app.conf
echo "documentation" > README.md
tar czf test.tar.gz app.conf README.md

# Search inside without extracting:
tar tf test.tar.gz | grep -i config

# Extract a single file:
tar xzf test.tar.gz app.conf
cat app.conf

# Search compressed file:
gzip app.conf
zgrep "config" app.conf.gz

# Clean up:
rm -rf ~/archive_test

Troubleshooting Common Issues
Symptom	Likely Cause	Resolution
find takes very long	Searching from / without pruning	Add -maxdepth or prune /proc /sys /run
locate can't find a new file	Database not yet updated	Run sudo updatedb
locate shows files that no longer exist	Database is stale	Run sudo updatedb or use locate -e
grep matches unexpected lines	Regex interpreted special chars	Use grep -F for literal strings
grep regex doesn't match	Using BRE instead of ERE	Add -E flag for +, ?, {}, |
rg doesn't search hidden files	Default behavior excludes them	Use rg --hidden
fd misses some files	Respecting .gitignore or hidden by default	Use fd -H (hidden) or fd -I (ignore)
Permission denied flooding output	Searching directories you don't own	Redirect errors: 2>/dev/null
find -exec with spaces in filenames	Shell splits on spaces	Use find -print0 | xargs -0 or -exec ... {} +
Duplicate search takes too long	Hashing every file	Use fdupes or jdupes (size-filter-then-hash)
which shows wrong binary	Path order in $PATH	Check echo $PATH, adjust priority
type shows alias not binary	Shell alias shadows command	Use \command or command -V to see real binary
What's Next

You now possess a comprehensive toolkit for finding anything on your Fedora system: find with advanced expressions (depth control, time/size/permission/ownership tests, logical operators, grouped expressions, custom printf formatting, exec/execdir actions, pruning for performance), locate with pre-indexed speed, grep with full regular expression support, ripgrep for modern parallel searching, fd as a user-friendly find replacement, executable discovery tools (which/whereis/type/command -v), archive searching (tar/zgrep/rpm -ql), duplicate file detection (hash-based, fdupes, jdupes with hardlink deduplication), and system change tracking (find by time, rpm -V, DNF history, auditd watches, Btrfs snapshot diffs).

Chapter 21 moves to networking basics — understanding how your Fedora system connects to networks, configures IP addresses, manages DNS, diagnoses connection problems, and interacts with services across the network. You'll learn about NetworkManager, ip commands, ping, traceroute, ss (socket statistics), curl, and SSH — the foundational skills for any system administration work.

Until then, practice searching. Pick any file you've created during this manual — find it three different ways (find, locate, grep -r). Time yourself. The speed difference will surprise you.

    ✅ Chapter 20 Complete You understand advanced find expressions (options/tests/actions/operators structure, -maxdepth/-mindepth depth control, time-based tests -amin/-atime/-mmin/-mtime/-cmin/-ctime/-newer/-newerXY with +n/-n/exact n rules, size tests with c/k/M/G units, permission tests -perm exact/-all-bits/-any-bits modes, ownership tests -user/-group/-nouser/-nogroup, logical operators -a/-o/! with parenthesized grouping, actions -print/-printf with format codes %p/%f/%s/%y/%m/%u/%g/%TY/%Tm/%Td/%TH/%TM/%n/%i, -exec/-execdir with {} ; and {} + syntax, -delete with safety verification, -ls, pruning -prune for /proc /sys /run, -mount for filesystem boundaries, performance optimization), locate (plocate installation, updatedb with /etc/updatedb.conf PRUNEPATHS/PRUNEFS configuration, database building, -i/-l/-e/-c/-S flags, systemd plocate-update.timer, find-vs-locate speed/use-case comparison), grep with regular expressions (BRE vs ERE distinction, literal/dot/character classes/anchors ^$/quantifiers *+?{}/alternation/grouping, comprehensive flag reference -i/-v/-r/-l/-L/-n/-c/-o/-w/-x/-A/-B/-C/-E/-F/-e/-f/--color/--include/--exclude/--exclude-dir, practical recipes for recursive search/context/case/multiple-patterns/inversion/file-type-filtering/null-separation), ripgrep (installation, grep comparison, smart-case, gitignore respect, -t/-T type filtering, --hidden, --no-ignore, capture groups, replace dry-run, configuration file), fd (installation, find comparison, -e/-g/-s/-H/-I/-d/-t flags, regex vs glob mode, -x parallel execution with {} and {.} substitution, fd vs find syntax comparison), executable discovery (which for PATH search, whereis for binary+man+source, type for alias/builtin/function detection, command -v for portable scripting), searching inside archives (tar tf/list/single-extract, zgrep/bzgrep/xzgrep for compressed files, rpm -ql/-qlp/-qf for package contents, flatpak run --command for sandboxed filesystem access), finding duplicate files (hash-based pipeline, fdupes, jdupes with hardlink deduplication -L, Btrfs reflink copies with cp --reflink), tracking system changes (find by mtime, rpm -qa --last and rpm -V integrity verification, DNF history, auditd watches with auditctl -w/-W/-k, Btrfs snapshot comparison), progressive exercises from find mastery through regex workshops through ripgrep/fd practice through duplicate detection through archive searching, and comprehensive troubleshooting reference table.

